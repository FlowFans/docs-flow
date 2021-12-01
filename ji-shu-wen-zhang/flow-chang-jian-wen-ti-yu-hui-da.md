---
description: 关于Flow的常见问题与回答
---

# Flow  常见问题与回答

**Flow是什么？**&#x20;

Flow 是一个快速、去中心化且开发者友好的区块链。由加密猫团队开发， 它是整个消费者应用程序生态系统的基础，覆盖游戏、收藏品以及与之交互的各类应用程序。Flow基于一个全新的架构，在不牺牲去中心化－且不分片－的情况下，实现主流应用程序所要求的性能。这意味着Flow链上的开发者可以构建安全、可组合的应用程序，从而为世界范围内数十亿的消费者提供新的可能性。



**Flow  Token 是什么？**&#x20;

FLOW 是 Fl​​ow 网络的原生货币，是网络上用于质押、治理、支付交易费用和本金储备资产的专属代币。



**Flow 是否去中心化？**

当然。通过让参与维护网络安全的共识过程更加容易，福洛甚至会比现有的区块链网络更加去中心化。



**Flow 和其它的区块链相比，有什么不同？**

福洛从一开始就明确要支持游戏和消费者应用程序，并具有可扩展到百万活跃用户所必需的吞吐量。实现这些目标需要一系列重大的技术创新：

* 流水线架构将通常由单个矿工或验证者完成的工作分配给五类不同的节点，从而显著减少重复工作并提高效率；
* 一项我们命名为加密知识专有证明（SPoCKs）的加密技术用于解决验证者的困境；
* 所有智能合约共享单一状态，确保每笔交易都有完整的ACID保证。这允许了智能合约之间的大量交互（“可组合性”），并为基于福洛开发的应用创造了强大的网络效应；



**从以太坊上把应用迁移到Flow 的难度大吗？**

当前，所有基于以太坊的智能合约和去中心化应用（“Dapps”）都具有两个重要特征：它们的架构设计假定了ACID的开发环境；它们都是用以太坊虚拟机（EVM）的编程语言Solidity编写。

第一个特征也适用Flow：Flow上的所有Dapps和智能合约都可以假设单一共享状态空间，并且无需调整架构以支持一个分片的环境或异步函数调用。

第二个特征则不适用：尽管以太坊虚拟机相比非可编程的区块链已经有了巨大的改进，但以太坊本身也正朝着更加灵活和高性能的编程模型发展。Flow不直接支持EVM。



**扩容性不可能三角怎么解决？根据这个理论，安全性、去中心化和可扩展性三者不可兼得！**

扩容性不可能三角是Vitalik Buterin提出的一个[重要推论](https://github.com/ethereum/wiki/wiki/Sharding-FAQ#this-sounds-like-theres-some-kind-of-scalability-trilemma-at-play-what-is-this-trilemma-and-can-we-break-through-it)，它尚未得到正式证明，但几乎可以肯定对于同质化的区块链设计是符合的。如果区块链网络中每一个节点都扮演相同的角色，则必须至少牺牲其中一个维度。\
\
福洛并不“打破”或证伪这个不可能三角推论，而是绕过它。诀窍就在于，如果我们让不同的节点扮演不同的角色，我们就可以为系统中每个部分选择正确的取舍。\
\
共识节点最容易遭受拜占庭攻击，福洛最大程度保证其_安全性_ 和 _去中心化_。当然，这限制了它们的_可扩展性_，但这实际上并不是问题，因为我们不要求共识节点完成任何计算量大的工作。\
\
另一方面，我们提高了执行节点的 _可扩展性_ ，以显著提高吞吐量。这影响了这些节点的_安全性_和_去中心化_ ，我们通过由高度_安全_ 和_去中心化_ 的验证节点，对交易的每个步骤进行确认，来解决这一问题。\
\
对每一类节点，这个不可能三角推论都是适用的，但是合并后，系统每一部分的弱点都能为其他部分的优势所弥补。



**如何查看Flow钱包的交易记录？**

Flow 有自己的区块链浏览器，在此处输入钱包地址：[https://flowscan.org/](https://flowscan.org)&#x20;





**存放 FLOW 的最好的用户友好的钱包有哪些？**

您可以使用 Blocto、Ledge Finoa 或 Kraken。有关可用钱包的更多信息，请参阅 [https://docs.onflow.org/flow-token/available-wallets/](https://docs.onflow.org/flow-token/available-wallets/)



**Flow 钱包创建后有0.001 Flow Token是怎么回事？**&#x20;

这是账户创建时说提供的最低要求，您不能低于该金额，因为您的帐户将无法支付其存储费用，因为这是所需的存储费用最低余额。现在可能存在一个问题，即帐户意​​外地低于该余额，但该帐户的任何未来交易都将失败，除非添加更多 FLOW。



**Flow 上的生态项目有哪些 ？**&#x20;

可以在 Flowverse 找到，[https://www.flowverse.co/projects](https://www.flowverse.co/projects)&#x20;



&#x20;**Flow 生态的投资者有哪些？**&#x20;

[https://www.flowverse.co/investors](https://www.flowverse.co/investors)



#### 如何在 Flow 中创建账户？ <a href="#how-are-accounts-created-in-flow" id="how-are-accounts-created-in-flow"></a>

通过生成私钥和公钥，然后提交初始化帐户存储并为帐户分配密钥的交易来创建帐户。创建新帐户的帐户必须提供称为存储存款的付款才能创建新帐户。有关此过程的更多上下文，请参阅[Flow-SDK](https://github.com/onflow/flow-go-sdk#creating-an-account)文档。



#### 允许多个账户签署交易有什么意义？ <a href="#what-is-the-point-of-allowing-multiple-accounts-to-sign-transactions" id="what-is-the-point-of-allowing-multiple-accounts-to-sign-transactions"></a>

允许交易的多个签名者的主要目的是能够访问他们的私人帐户存储。每个帐户都有一个私有`[account.storage](http://account.storage)`对象，只能在`prepare`由它签名的交易块内访问。如果多个账户签署了一个交易，那么该交易就可以与它们的两个存储对象进行交互。它可以用它做很多不同的事情，包括传输任意资源、访问它们的私有功能以及实现内置于协议中的多重签名钱包。



#### Flow 是否有公开可用的 API 端点（gRPC、JSON-RPC ..）我们可以将交易发送到？ <a href="#is-there-a-publicly-available-api-endpoint-grpc-json-rpc-we-can-send-the-transactions-to" id="is-there-a-publicly-available-api-endpoint-grpc-json-rpc-we-can-send-the-transactions-to"></a>

要将交易发送到网络，您必须将交易提交到公共访问节点才能包含在内。有关更多信息，请参阅官方文档dapp部署指南。



**Flow 的智能合约语言是什么？**&#x20;

是一种新的编程语言 Cadence&#x20;



#### 为什么要创建一种新的编程语言？ <a href="#why-create-a-new-programming-language" id="why-create-a-new-programming-language"></a>

我们相信，目前没有一种语言可以为去中心应用程序提供安全性、安全性、性能和易用性的最佳组合。



#### 我可以在 Cadence 中生成随机数吗？ <a href="#can-i-generate-random-numbers-in-cadence" id="can-i-generate-random-numbers-in-cadence"></a>

您可能知道，在区块链环境中生成随机数很困难。区块链和执行环境的完全开放性意味着任何人都可以查看生成随机数的算法。也没有一个好的熵源不能在所有情况下都被欺骗。Cadence 包含一个`unsafeRand`生成随机数的函数，该随机数是伪随机的，但并非在所有情况下都可以安全使用。我们还致力于设计更安全的方案以用于智能合约。



#### Oracle 将如何在 Cadence 中工作？ <a href="#how-will-an-oracle-work-in-cadence" id="how-will-an-oracle-work-in-cadence"></a>

预言机可以像在其他区块链环境中一样在 Cadence 中工作。你可以制作一个智能合约来注册事件，并为一个账户提供授权资源，oracle 可以发送交易来记录链下事件。



#### 如何连接和查询接入节点？那里有哪些可用的数据？ <a href="#how-can-i-connect-to-and-query-the-access-nodes-what-data-is-available-there" id="how-can-i-connect-to-and-query-the-access-nodes-what-data-is-available-there"></a>

在协议层面，您可以通过 GRPC 连接到一个接入节点。我们提供 JavaScript 和 Golang SDK 来为您执行此操作。

连接到访问节点后，您可以获取有关帐户、合约、区块、集合、交易和事件的信息。您还可以执行脚本来查询 Flow 区块链的当前状态。



#### 什么是FCL？ <a href="#what-is-fcl" id="what-is-fcl"></a>

FCL（流客户端库）使应用程序能够轻松地与所有兼容 FCL 的钱包和其他服务（例如配置文件、私人信息、通知）集成。这为开发人员提供了一个强大的基础，可以用现有的构建块来组合他们的应用程序。FCL 目前支持浏览器应用，并将扩展到其他平台。

使用 FCL，您可以：

* 集成所有兼容钱包，无需任何自定义代码或代码注入
* 验证用户
* 查询用户的Flow账号
* 发送交易（例如初始化资源、发送资产、购买等）
* 通过钱包集成签署交易以避免密钥管理（对非托管应用程序特别有用）



#### 如何访问Flow主网？ <a href="#how-do-i-access-mainnet" id="how-do-i-access-mainnet"></a>

您可以在[https://docs.onflow.org/access-api](https://docs.onflow.org/access-api)找到有关访问 Flow 主网的详细信息。

访问是通过 GRPC 接口进行的。交易也可以使用 GRPC [SendTransaction 调用](https://docs.onflow.org/access-api#sendtransaction)提交。用任何语言编写的 GRPC 客户端都应该能够通过 Access API 进行对话。我们有[Go SDK](https://github.com/onflow/flow-go-sdk)和[FCL-JS（流客户端库）](https://github.com/onflow/fcl-js)，如果您分别使用基于 Go 或基于 JS 的客户端，则可以使用它们。



#### Flow 的 Gas 费用是多少？ <a href="#how-much-will-gas-cost-on-flow" id="how-much-will-gas-cost-on-flow"></a>

从太平洋时间 2021 年 10 月 16 日上午 7:30（UTC 下午 2:30）开始，提交给网络的每笔交易都将收取 0.00001 FLOW 的固定费用。交易费用可能会发生变化。如果确实发生了更改，将在实施之前通知社区。这些费用用于防止垃圾邮件，并且对于所有交易都是低且固定的。



#### 开发人员什么时候可以在主网上部署？如何为主网创建我的第一个帐户？ <a href="#when-will-devs-be-able-to-deploy-on-mainnet-how-can-i-create-my-first-account-for-mainnet" id="when-will-devs-be-able-to-deploy-on-mainnet-how-can-i-create-my-first-account-for-mainnet"></a>

在测试期间，您需要将您的合同提交给 Flow 团队进行审核，然后您就可以将其提交给网络。一旦您在 devnet 上测试了您的 dapp 并且 Flow 团队审查了您的合同，您就可以部署了！

信息在这里：[https](https://docs.onflow.org/dapp-deployment) : [//docs.onflow.org/dapp-deployment](https://docs.onflow.org/dapp-deployment)



#### 有测试网/开发网吗？ <a href="#is-there-a-testnetdevnet" id="is-there-a-testnetdevnet"></a>

在 testnet/devnet 上有一个供您开发的访问节点。您可以在此处了解更多信息[https://docs.onflow.org/dapp-deployment/testnet-deployment#accessing-flow-testnet](https://docs.onflow.org/dapp-deployment/testnet-deployment#accessing-flow-testnet)



#### 密钥和帐户如何在 Flow 上工作？ <a href="#how-do-keys-and-accounts-work-on-flow" id="how-do-keys-and-accounts-work-on-flow"></a>

帐户是使用关联的密钥创建的。一个帐户上可以有多个密钥。要从账户执行交易，总共需要签署 1000 个权重密钥。该帐户包含 FLOW 余额字段。当交易流动时，该余额由协议更新。该帐户还用于存放存储和合约代码。

FLOW 支持多种签名方案来为账户添加密钥。

详情：[https](https://docs.onflow.org/concepts/accounts-and-keys) : [//docs.onflow.org/concepts/accounts-and-keys](https://docs.onflow.org/concepts/accounts-and-keys)



#### 如何查看账户拥有哪些资源？ <a href="#how-can-i-see-what-resources-an-account-has" id="how-can-i-see-what-resources-an-account-has"></a>

Flow 尚未提供检查帐户上所有资源的功能，但可以执行 Cadence 脚本来检查已知存储路径上的资源。



#### 可以查询块范围之间的事件吗？ <a href="#can-you-query-events-between-a-block-range" id="can-you-query-events-between-a-block-range"></a>

您可以查询访问 API 以获取块范围的事件。请参阅此处的访问 API 规范：[https](https://docs.onflow.org/access-api) : [//docs.onflow.org/access-api](https://docs.onflow.org/access-api)。



#### 是否有关于如何访问Flow测试网的教程？从头开始，获取测试网、Flow Token等？ <a href="#is-there-a-tutorial-about-how-to-access-flow-testnet-from-scratch-getting-testnet-flow-token-etc" id="is-there-a-tutorial-about-how-to-access-flow-testnet-from-scratch-getting-testnet-flow-token-etc"></a>

是：[https](https://docs.onflow.org/dapp-deployment/testnet-deployment/) : [//docs.onflow.org/dapp-deployment/testnet-deployment/](https://docs.onflow.org/dapp-deployment/testnet-deployment/)



#### &#x20;<a href="#can-you-query-events-between-a-block-range" id="can-you-query-events-between-a-block-range"></a>



