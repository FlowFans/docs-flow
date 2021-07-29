# Cadence的学习经验总结



​ 学习Cadence，首先需要理解资源这一概念，这点与传统的Solidity等编程语言有很大的不同。Cadence是一种面向资源的编程语言，通过保证资源（和相关资产）一次只能存在于一个位置，不能被复制，从而为数字所有权创建安全的声明性模型，使其不能意外丢失或删除。

​ flow认为传统的智能合约的中心化账本设计有很多缺点，尤其是在跟踪属于单个帐户的多个资产的所有权时要找出一个账户拥有的所有资产，您必须枚举可能包含该账户的所有可能的智能合约，并搜索该账户是否拥有这些资产。

​ 但在像Cadence这种面向资源的语言面前，所有权并不集中在单个中央智能合约的存储中。相反，每个账户拥有并存储自己的资产，资产可以在账户之间自由转移，无需通过中央智能合约进行控制。

![fow\_account](/home/wtom/图片/fow_account.png)

​ 在Flow里，所有的persistent state与其接口都存储在账户中，而其他用户想要调用它们可以通过transactions执行相关代码来进行调用。

​ 以Hello World为例，可以在Playground里看到该教程。需要注意的是，Playground里提供了多账户，这极大的方便了合约的交互。在这里，我们以用户0X02的资源演示为例，

​ 部分contract内容：

```text
pub resource HelloAsset {
    pub fun hello(): String {
        return "Hello, World!"
    }
}
```

​ 在这里，resource代表这是一种资源类型，每一个资源都存在于一个特定的地点且不能被复制，也很难丢失，与传统语言相比，flow更适合对NFT进行编程与开发。

​ 在flow中，想要与资源交互，需要发送transactions：

```text
import HelloWorld from 0x02

// This transaction calls the "hello" method on the HelloAsset object
// that is stored in the account's storage by removing that object
// from storage, calling the method, and then putting it back in storage

transaction {

    prepare(acct: AuthAccount) {

        // load the resource from storage, specifying the type to load it as
        // and the path where it is stored
        let helloResource <- acct.load<@HelloWorld.HelloAsset>(from: /storage/Hello)

        // We use optional chaining (?) because the value in storage
        // may or may not exist, and thus is considered optional.
        log(helloResource?.hello())

        // Put the resource back in storage at the same spot
        // We use the force-unwrap operator `!` to get the value
        // out of the optional. It aborts if the optional is nil
        acct.save(<-helloResource!, to: /storage/Hello)
    }
}
```

​ 该transaction将HelloWorld从用户0x02引入，并将HelloAsset从storage移除，然后调用hello\(\)函数存储到HelloAsset资源中。

​ script运行不需要任何账户权限，但是不能对区块链执行任何写入操作，只能读取帐户的状态。

​ 例如：

```text
import HelloWorld from 0x02

pub fun main() {

    // Cadence code can get an account's public account object
    // by using the getAccount() built-in function.
    let helloAccount = getAccount(0x02)

    // Get the public capability from the public path of the owner's account
    let helloCapability = helloAccount.getCapability<&HelloWorld.HelloAsset>(/public/Hello)

    // borrow a reference for the capability
    let helloReference = helloCapability.borrow()
        ?? panic("Could not borrow a reference to the hello capability")

    // The log built-in function logs its argument to stdout.
    //
    // Here we are using optional chaining to call the "hello"
    // method on the HelloAsset resource that is referenced
    // in the published area of the account.
    log(helloReference.hello())
}
```

​ 该脚本可以调用0X02里的hello函数并显示打印结果。

​ 总的来说，Cadence是一个包含contract，transaction，script三个部分的面向资源的编程语言。

​

