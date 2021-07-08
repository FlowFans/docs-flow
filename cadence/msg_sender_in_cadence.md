---
description: >-
  原始出处：https://joshuahannan.medium.com/how-do-i-check-msg-sender-in-cadence-capabilities-e5a9703e2ca0
---

# 在Cadence中我该如何查询msg.sender?Capabilities!

Hello! 我是 Josh Hannan，DapperLabs Flow团队的智能合约工程师。距离[上次发表文章](https://joshuahannan.medium.com/optionals-in-cadence-not-optional-fb39bb4b0081)已经有一段时间了。在过去的一段时间，Flow团队一直都很忙碌，但我准备从现在起定期发表文章。Flow团队的进展很快，我们有很多有意思的产出！下面是我最近正在从事的工作。

## 时间点智能合约（Epoch Smart Contracts）

你有没有读过[Flow质押智能合约\(Flow Staking Smart Contract\)](https://github.com/onflow/flow-core-contracts/blob/master/contracts/FlowIDTableStaking.cdc)？没有的话，我建议先去读一下文档，了解质押合约是如何运行的。这并不是通常意义下的NFT合约，也许你会发现对你有帮助的一些有意思的设计思路\(Design Pattern\)和工具。

Flow质押智能合约只是Flow用来管理时间点\(Epoch\)的众多Cadence合约中的一个，时间点\(Epoch\)指的是为期一周的时间段，在这段时间里，节点身份表\(Node Identity Table\)会被设定，并且节点会在运行网络的同时为下一个时间点\(Epoch\)做准备。这部分协议的核心逻辑是由智能合约处理的，我们不久将会把这部分合约部署到主网。

我们正在编写详尽的文档，感兴趣的话，下面是我提出来的几份Flow改进提案\(FLIPs, Flow Improvement Proposals\)。

* [时间点集群数量证书智能合约](https://github.com/onflow/flow/blob/josh/epochflip/flips/20210113-quorum-certificate.md)（Epoch Cluster Quorum Certificate smart contract）
* [时间点分布式秘钥生成智能合约](https://github.com/onflow/flow/blob/josh/epochflip/flips/20210322-dkg.md)\(Epoch Distributed Key Generation \(DKG\) smart contract\)
* [时间点生命周期智能合约](https://github.com/onflow/flow/blob/josh/epochflip/flips/20210322-dkg.md)\(Epoch Lifecycle smart contract\)

如果想了解更多或者给予反馈意见的话，我们在Github上也有一个开放的包含全部代码和测试的功能分支。

{% embed url="https://github.com/onflow/flow-core-contracts/pull/63" %}

## 我所有的文章的集中所在地！

Dapper Labs的团队非常友好的帮我创建了一个包含我所有这个系列文章列表的官方博客！

{% embed url="https://www.onflow.org/post/flow-blockchain-cadence-programming-language" %}

团队会在我每一次发表新文章后更新这个列表，所以你总是可以通过这个官方博客找到我所有的文章。喜欢的话可以添加个书签方便以后查阅啊！

如果你还没有Flow或者Cadence的开发经验的话，我强烈建议读一读我的文章，可以从第一篇开始读起！

## 权限控制

每一个开始使用Cadence编程的Solidity用户都会问到“我该怎么查询`msg.sender`?” 。在以太坊中，查询`msg.sender`是用来验证调用函数\(function\)的账户信息并且合理修改这个函数的状态\(behavior\)的。这么做对在以太坊上的身份\(identity\)，权限\(permissions\)，所有权\(ownership\)，安全\(security\)都很重要。ERC-20合约可以让用户在发送代币\(token\)时通过查询`msg.sender`查看要发送的是什么代币。治理合约可以通过查询`msg.sender`来注册访问者的投票，以防再次投票。除此之外，还有很多别的使用案例。

Cadence中没有`msg.sender`，并且在交易层\(transaction-level\)Cadence也不支持单独验证调用人的身份。这并不是因为这个功能还没有被添加进去。而是因为，在Cadence中，交易\(transaction\)可以由多人共同签名，所以调用`msg.sender`也就没什么意义了。

所有的这些意味着，在Solidity中通过单一中心合约查询`msg.sender`的任务将需要采用不同的方式才能在Cadence中实现。Cadence的设计初衷不同，Cadence采用的是一种[基于Capability的安全模型\(Capability-based security model\)](https://en.wikipedia.org/wiki/Capability-based_security)，而不是[基于访问控制表的模型\(Access-Control List-based model\)](https://en.wikipedia.org/wiki/Access-control_list)。我们相信这样的设计能够为程序的鲁棒性和安全性带来显著的优势。

Flow团队中的一位合约工程师，Rhea Myers，创建了一篇文档，用以说明我们为什么不能在Cadence中使用`msg.sender`，我们的替代方案是什么，以及一些不同使用场景下的例子。Capability安全\(Capability Security\)是Cadence最为重要的设计之一，如果你想要编写你自己的智能合约的话，了解这一特性是非常非常重要的。

下面是Rhea的文档链接：

{% embed url="https://docs.onflow.org/cadence/msg-sender/" %}

在我接下来的文章中，我将会选取几个Rhea文档中提到的例子并展开说明，我也会用一步一步的教程向大家展示在不同的智能合约中如何使用Capability。

## 结论

希望这些信息能对你有所帮助！一如既往的，有任何问题或者建议，都通过下面的这些渠道联系我们！

Flow Discord: [https://discord.gg/flow](https://discord.gg/flow)

Flow Forum: [https://forum.onflow.org](https://forum.onflow.org/latest)

Flow Github: [https://github.com/onflow/flow](https://github.com/onflow/flow)

下次见！

非常感谢Rhea Myers给予的启发，介绍，和我今天所讲的大部分内容！
