# NFT 实例讲解

## [Cadence 投票实例 -- 每张票都是唯一的NFT](https://play.onflow.org/424af532-7a83-40c2-a16d-475cb51d32ee?type=account&id=0)

### 1. 创建投票合约 （Contract）

{% code title="ApprovalVoting.cdc" %}
```javascript

/*
*
*   在本例中，我们希望创建一个简单的批准投票实例 
*   投票站 将选票发送到指定地址
*
*   1. 管理员部署智能合约，开启投票
*   2. 初始化提案,初始化后的建议数组不能被修改。
*   3. 然后他们将票给用户
*   4. 每个用户都可以投票批准任何数量的提案。
*
*/

pub contract ApprovalVoting {

    // 待批准的提案清单
    pub var proposals: [String]

    // 每项提案的投票数
    pub let votes: {Int: Int}

    //发送给用户的资源。
    //当用户获得一个选票对象时，他们调用' vote '函数
    //包含他们的投票，然后将其投到智能合约中
    //使用' cast '函数将他们的投票包含在投票中
    pub resource Ballot {

        // 所有提案的数组
        pub let proposals: [String]

        // 对应于投票后提案中的数组索引
        pub var choices: {Int: Bool}

        init() {
            self.proposals = ApprovalVoting.proposals
            self.choices = {}

            // 将每个选项设置为false
            var i = 0
            while i < self.proposals.length {
                self.choices[i] = false
                i = i + 1
            }
        }

        // 修改选票
        // 表明它投票支持哪些提案
        pub fun vote(proposal: Int) {
            pre {
                self.proposals[proposal] != nil: "不能投票，选项不存在"
            }
            self.choices[proposal] = true
        }
    }

    // 由投票管理员控制的资源
    // 初始化提案并向选民分发选票资源
    pub resource Administrator {

        // 函数初始化投票的所有提案
        pub fun initializeProposals(_ proposals: [String]) {
            pre {
                ApprovalVoting.proposals.length == 0: "提案只能被初始化1次！"
                proposals.length > 0: "不能初始化空的提案！"
            }
            ApprovalVoting.proposals = proposals

            // 将每个选票的计数设为零
            var i = 0
            while i < proposals.length {
                ApprovalVoting.votes[i] = 0
                i = i + 1
            }
        }

        // 管理员调用这个函数来创建一个新的Ballo
        // 可以被转移到另一个用户
        pub fun issueBallot(): @Ballot {
            return <-create Ballot()
        }
    }

    // 用户将他们的选票移动到合同中的这个函数
    // 它的选票被清点，选票被销毁
    pub fun cast(ballot: @Ballot) {
        var index = 0
        // 检查选票
        while index < self.proposals.length {
            if ballot.choices[index]! {
                // 如果通过，统计票数
                self.votes[index] = self.votes[index]! + 1
            }
            index = index + 1;
        }
        // 销毁选票，因为它已经计分
        destroy ballot
    }

    // 通过设置提案和投票为空来初始化合同
    // 创建一个新的管理资源放入存储
    init() {
        self.proposals = []
        self.votes = {}

        self.account.save<@Administrator>(<-create Administrator(), to: /storage/VotingAdmin)
    }
}
```
{% endcode %}

### 2. 初始化投票提案 \(Transaction1\)

```javascript
import ApprovalVoting from 0x01

// 该交易允许管理员调用投票合同
// 创建新的投票提案，并将其保存到智能合约

transaction {
    prepare(admin: AuthAccount) {

        // 借用 管理资源的引用
        let adminRef = admin.borrow<&ApprovalVoting.Administrator>(from: /storage/VotingAdmin)!

        // 调用initializeProposals函数
        // 创建一个字符串数组的 提案数组
        adminRef.initializeProposals(
            ["篮球", "赛车", "音乐"]  // 在 Flow 上喜欢哪个 NFT 领域 ？
        )

        log("提案初始化!")
    }

    post {
        ApprovalVoting.proposals.length == 3    // 三个提案
    }

}
```

### 3.  分发票给投票人 （Transaction2）

```javascript
import ApprovalVoting from 0x01

 
// 创建一个新的选票并将其存储在一个选民的帐户中
// 投票者和管理员都必须签署事务
// 它可以访问它们的存储

transaction {
    prepare(admin: AuthAccount, voter: AuthAccount) {

        // 借用 管理资源的引用
        let adminRef = admin.borrow<&ApprovalVoting.Administrator>(from: /storage/VotingAdmin)!

        // 通过调用issueBallot创建一个新的选票
        // admin Reference的函数
        let ballot <- adminRef.issueBallot()

        // 将选票存储在投票人的帐户存储中
        voter.save<@ApprovalVoting.Ballot>(<-ballot, to: /storage/Ballot)

        log("选票转让给投票人")
    }
}
```

### 4. 持票人投票 （Transaction3）

```javascript
import ApprovalVoting from 0x01

// 该事务允许投票者选择他们想要投的票
// 通过使用castVote函数来转换该投票

transaction {
    prepare(voter: AuthAccount) {

        // 把投票人的选票存起来
        let ballot <- voter.load<@ApprovalVoting.Ballot>(from: /storage/Ballot)!

        // 对提案进行投票
        ballot.vote(proposal: 1)  // 0.篮球 1.赛车 2.音乐

        // 通过将投票提交给智能合约进行投票
        ApprovalVoting.cast(ballot: <-ballot)

        log("投票和统计...")
    }
}
```

### 5. 查看投票结果 （Scrpit）

```javascript
import ApprovalVoting from 0x01

// 这个脚本允许任何人阅读每个提案的计票结果
 
pub fun main() {

    log("在 Flow 上你喜欢哪个 NFT 领域 ？")
    log("----投票结果----")
    // 访问合约的公共字段来记录
    // 提案名称和投票数

    
    log(ApprovalVoting.proposals[0])
    log("总票数:")
    log(ApprovalVoting.votes[0])
    
    log(ApprovalVoting.proposals[1])
    log("总票数:")
    log(ApprovalVoting.votes[1])
  
    log(ApprovalVoting.proposals[2])
    log("总票数:")
    log(ApprovalVoting.votes[2])
   

}
```

