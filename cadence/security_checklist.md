# Cadence 合约安全自查清单

[原文来源「cadence-contract-security-internal-review-checklist」](https://forum.onflow.org/t/cadence-contract-security-internal-review-checklist/2355)

本检查表由 bluesign 的 Ken Huang（英文名 Deniz），和 flowstarter 的 Julien 共同编写。

本检查表是作为内部审计使用，并不能取代第三方的审计。我们强烈建议项目团队在主网部署代码之前，聘请第三方对代码进行审计。Flow 团队目前也在审计代码。

## 1. 访问修饰符 （Access Modifier） 检查

Cadence 变量、常量和函数都有访问控制修饰符来定义读和写的范围。

如果修饰符使用不正确，将存在安全风险。审计师可以重点检查访问修饰符是否使用正确。

**`let`会让 `array` 或 `dictionary` 也有写入权限，这是个很常见的错误**

公共（public）的 数组（array） 或 字典（dictionary） 字段不能直接覆盖重写，但其成员可以被访问和改写。如果这些字段被错误地设置为 public，这有可能导致合约的安全漏洞。确保 合约（contract）、结构（struct） 或 资源（resource） 中的任何数组或字典字段都是 access(contract)、access(account)或 access(self)，除非它们需要被有意公开。

Example:

```cadence
pub contract Array {
  // array is intended to be initialized to something constant
  pub let shouldBeConstantArray: [Int]
}
```

任何人都可以发送这样的交易（transaction）来修改它。

```cadence
import Array from 0x01

transaction {
  execute {
    Array.shouldbeConstantArray[0] = 1000
  }
}
```

### Public 的管理员资源（Admin Resource） 创建函数是不安全的

合约里的创建资源的 public 函数可以被任何账户调用。

如果该资源提供了对管理员功能的访问，那么该函数就不应该是 public 的。

为了解决这个问题，在合约初始化时，使用该资源创建单一实例，然后链接（link）到管理员账户存储的一个私有路径（PrivatePath）上。如果创建资源的代码足够复杂，需要有自己的函数，那么创建资源的函数应该使用 access(contract) 作为其访问修饰符。

### Public 的 Capability 字段是安全漏洞

public 字段的值可以被复制。Capabilities 是值类型。任何收到 Capability 的人都可以使用它。因此，如果他们可以从 public 字段复制它，他们就可以调用它所暴露的函数。

如果一个 Capability 被以这种方式存储，这几乎肯定不是你想要的。为了让其他人访问一个 Capability，请把它放在一个账户的公共区域中。

### 用户可能会重新绑定受限制的 Public Capability 路径

用户账户上 public 的 Capability 是一个标准资源（如 Vault）的变体，它有额外的逻辑实现来限制或改变其行为，可以由该用户在同一路径上用另一个没有这些变体的资源来替代。

为了解决这个问题，确保在任何访问它的代码中仔细指定你所期望的 Capability 的类型。

### 额外参考

作为额外的参考，下表总结了变量声明、常量声明和字段的权限范围：

| 声明类型 | 访问修饰符              | 读范围                                              | 写范围            |
| :------- | :---------------------- | :-------------------------------------------------- | :---------------- |
| `let`    | `priv` / `access(self)` | Current and inner                                   | _None_            |
| `let`    | `access(contract)`      | Current, inner, and containing contract             | _None_            |
| `let`    | `access(account)`       | Current, inner, and other contracts in same account | _None_            |
| `let`    | `pub`,`access(all)`     | **All**                                             | _None_            |
| `var`    | `access(self)`          | Current and inner                                   | Current and inner |
| `var`    | `access(contract)`      | Current, inner, and containing contract             | Current and inner |
| `var`    | `access(account)`       | Current, inner, and other contracts in same account | Current and inner |
| `var`    | `pub` / `access(all)`   | **All**                                             | Current and inner |
| `var`    | `pub(set)`              | **All**                                             | **All**           |

对于函数，下表按范围的顺序定义了可见性：

| 访问修饰符              | 访问范围                                            |
| :---------------------- | :-------------------------------------------------- |
| `priv` / `access(self)` | Current and inner                                   |
| `access(contract)`      | Current, inner, and containing contract             |
| `access(account)`       | Current, inner, and other contracts in same account |
| `pub` / `access(all)`   | **All**                                             |

结构、资源、事件和合约的声明只能是 pub 。然而，即使声明/类型是公开的，资源也只能从它们所声明的合约内部创建。

## 2. Capability 访问控制检查

Capabilities 标识为路径，并链接（link）到 target 路径，而不是直接链接到一个对象。Capabilities 要么是公共的（任何用户都可以获得访问权），要么是私有的（必须有授权用户的访问权）。公共 Capabilities 是使用公共路径（PublicPath）创建的，也就是说，它们的域是公共的。创建后，它们可以从授权账户（AuthAccount）和公共账户（PublicAccount）获得。
私有 capabilities 是使用私有路径（PrivatePath）创建的，也就是说，它们的域是私有的。创建后，它们可以从授权账户（AuthAccount）获得，但不能从公共账户（PublicAccount）获得。
一旦一个 capability 被创建并获得，它可以被借用（borrow）来获得对存储对象的引用。当一个 capability 被创建时，一个类型（type）被指定，决定了该 capability 可以被借用的类型。这**允许暴露或隐藏存储对象的某些功能**。
可以使用授权账户（AuthAccount）的取消链接功能来删除 Capabilities。
审计师可以检查该 capability 是否应被定义为私有而不是公共的，以及当不再需要时是否应使用 unlink 函数来删除该 Capability。
Capabilities（即使它们是私有的，也不应该存储在可公开访问的地方）。它们可以被复制（因为它们是结构）。
确保给每个 capability 分配一个独特的私有路径，这样你就可以在以后撤销它（unlink）。

使用 [capability receiver](https://docs.onflow.org/cadence/design-patterns/)是一个好的模式。
合约开发者必须意识到，由用户账户提供的 Capability 有可能被解除链接，导致借用该资源的代码失败。如果功能是必要的，应该提供备用方案。另外，一个 Capability 可以被替换成不同的类型，它可以通过 check()，但在 borrow()上却不能被转换。同样，如果代码的功能是必要的，应该提供备用方案。

## 3. 交易（transaction）的四个阶段：准备（prepare）、先决条件（pre）、执行（execute）和后决（post）条件 检查

在 Flow 中，有 3 种类型的代码可以与 Flow 区块链互动。合约（Contract）、交易（Transaction）和脚本（Scripts）。合约被部署到 Flow 区块链上，如果执行或调用，可以改变区块链状态。交易没有部署到链上，可以与部署在 Flow 区块链上的合约互动，改变链的状态。脚本可以读取区块链的状态，但不会改变或写入区块链的状态。

因此，对于你的项目，如果你确实有将被你的前端代码（如 ReactApp）调用的交易代码，最好仔细检查，看看是否有任何潜在的安全问题，并检查后置条件，看看交易是否符合你定义的业务逻辑。当然，请牢记，你的合约需要对所有的正反交易进行测试，以防止黑客的交易利用你合同中的安全漏洞。**一个可能的情况是让你的合约设置允许的交易的钱包或地址白名单**

Blocto 钱包对此所做的是提供警告。目前，所有的交易都显示了大量的代码，这对于非技术性用户来说无疑是有点可怕的。他们的解决方案更多的是隐藏这一点，而是显示“这是一个由团队创建的交易，已经被其他用户运行了 XXXX 次”或类似的东西，或是“一个交易是在他们的白名单中”。

[https://github.com/portto/flow-transactions/tree/main/transactions/BloctoSwap](https://github.com/portto/flow-transactions/tree/main/transactions/BloctoSwap)

一个交易有 4 个可选的阶段，用以下代码说明。

审计师可以检查代码块是否处于正确的阶段。
`Post{ }` 很少使用，但它是交易中最重要的部分之一。(对用户来说）也是 "合约函数"。

在后置条件中，你需要检查不变量，也需要检查余额（执行后的总余额或 self.balance），以及其他后置条件。例如，你应该检查执行结果的后置条件。如果你用 10FLOW 购买 1 个 ID 为 X 的 NFT，余额应该少 10FLOW，并且你应该在你的收藏中拥有购买的 ID 为 X 的 NFT。

```cadence
transaction {
prepare(signer1: AuthAccount) {
// Access signing accounts for this transaction.
//
// Avoid logic that does not need access to signing accounts.
//
// Signing accounts can't be accessed anywhere else in the transaction.
}

    pre {
        // Define conditions that must be true
        // for this transaction to execute.
    }

    execute {
        // The main transaction logic goes here, but you can access
        // any public information or resources published by any account.
    }

    post {
        // Define the expected state of things
        // as they should be after the transaction is executed.
        //
        // Also used to provide information about what changes
        // this transaction will make to accounts in this transaction.
    }

}
```

## 4. 角色检查

可以使用 3 个不同的角色来建立一个 Cadence 交易：提议者(proposer)、授权者(authorizer)和支付者(payer)，如下所示:

```shell
flow transactions build ./transaction.cdc "Meow" \
  --authorizer alice \
  --proposer bob \
  --payer charlie \
  --filter payload --save built.rlp:
```

审计师可以与项目方合作，以 "职责分离 "和 "最小权限 "的安全原则定义，以确定授权者、提议者和付款人的角色。

这里需要注意的是，当签名 envelope 时，"payer" 同时也是 "authorizer"，所以如果你作为 "payer "为别人签名，要小心检查，他们不会把你设置为 "authorizer "以获得对你账户的访问。(authorizer 也是如此（作为 proposer）。把检查你所签名的内容作为经验法则）。

## 5. 从 Resources 中产生的 Events 可能并不是独一无二的

合约上创建资源的公共函数可以被任何账户调用。

如果该资源有触发事件（emit event）的功能，任何账户都可以创建该资源的实例并触发这些事件。

如果这些事件是为了表明使用该资源的单一实例（例如 admin 对象、switchboard 或 registry）所采取的行动，或通过特定工作流程创建的实例，那么这将使事件日志的搜索和管理更加困难，因为来自其他实例的事件可能与你想要检查的事件混在一起。

为了解决这个问题，如果应该只有一个资源实例，它应该在合约的初始化过程中被创建并链接到管理员账户存储的公共路径上。

## 6. 最好是对值类型字段进行命名

当合约、资源和脚本都必须重复引用相同的常量值时，手动输入这些值是一个潜在的错误来源。与其在函数、事务和脚本中手动输入这些值，不如将这些值一次性存储在合约的字段中，然后通过这些字段访问它们。这就是命名值字段模式。
在负责该值的合约中添加一个 access(all) 字段，例如一个 Path，并在合同的初始化中设置它。然后通过这个公共字段来引用该值，而不是手动指定它。

## 7. 业务逻辑安全检查

根据业务逻辑，审计师可以检查以下的安全方面：

- [与时间有关的逻辑或与顺序有关的逻辑](https://docs.onflow.org/cadence/measuring-time/)

如果你的合约代码可以容忍大约 10 秒的变化，就使用区块的时间戳。如果不能，你可能需要考虑更奇特的解决方案（如时间预言机等）。

无论你使用哪种方法，都要注意不要将任何关于区块生产率的假设以不能更新的硬编码方式编写，不管是链上还是链下。

链上拍卖和类似机制应该始终有一个扩展机制。如果有人在最后一刻出价（这在区块生产攻击中更容易做到），拍卖的结束时间就会延长（如果需要）到最后一次出价后的 N 分钟。(10 分钟，30 分钟，一个小时) 。随着 N 的增加，这变得更加安全。N=5 应该是绰绰有余的。以目前 Flow 区块链的参数来看。

- 随机性

cadence 中没有可靠的随机性（不安全的随机性既不安全也不一致，任何依赖随机性的业务逻辑都应该是链外的）。
在未来，最好是利用 Chainlink VRF 来实现随机性。
[chainlink-vrf](https://docs.chain.link/docs/chainlink-vrf/)

- Withdraw 和 Deposit 相关逻辑

- 价格信息

## 8. Flow 对于测试的要求

Flow 团队 有 [high level guidance for contract testing](https://docs.onflow.org/dapp-deployment/contract-testing/)，简单地说，需要以下内容：

- 合约的每一个公开暴露的功能及其资源都应该有单元测试，在正确的输入情况下检查成功，在不正确的输入情况下检查失败。这些测试应该能够用 Flow 模拟器在本地运行，不需要或只需要最小的额外资源或配置，且只需要一个命令。
- 每个使用合约的用户场景或工作流程都应该有一个集成测试，以确保完成它所需的一系列步骤能用测试数据成功完成。
- 在批准部署到主网之前的手动测试，代码必须部署在测试网上并使用两周（需要使用负载测试来产生负载）。
- 尽量接近它在主网中使用的条件模拟，如果它将被部署到主网。

目前有一些 JavaScript 和 Go 测试框架，你可以用来为你的合约代码创建测试：

[https://github.com/onflow/flow-js-testing](https://github.com/onflow/flow-js-testing)

[https://github.com/onflow/flow-core-contracts/tree/master/lib/go/test](https://github.com/onflow/flow-core-contracts/tree/master/lib/go/test)

[https://github.com/onflow/flow-nft/tree/master/lib/go/test](https://github.com/onflow/flow-nft/tree/master/lib/go/test)

[https://github.com/onflow/flow-ft/tree/master/lib/go/test](https://github.com/onflow/flow-ft/tree/master/lib/go/test)

## 9. 压力测试和 DoS 测试

系统的可用性是安全的一个方面。可以进行压力测试以查看可用性和响应时间。 Flow 团队有 [相关指导](https://docs.onflow.org/dapp-deployment/testnet-testing/)
在你的应用程序被批准和部署后，将开始一段时间的观察。在此期间，Flow 团队将记录你的应用程序如何处理流量，以确保没有任何可能影响更大范围网络或你的应用程序的用户体验的问题。
在观察期间，Flow 团队希望看到你的应用程序处理非微不足道的流量。如果 Flow 团队没有看到足够的流量，他们可能会推迟主网的批准。
包括你自己的自动化负载测试或负载测试指标可以帮助你的应用程序更快地获得主网的批准。Flow 团队对可能导致问题的模式有一定的了解，所以如果你不确定，请在实施你的想法之前与他们取得联系

如果你进行了更大的测试，请在 Discord 上给 Flow 团队发消息（你可以在#developers 频道上标记@Flow 团队）。你可能也想使用模拟器来进行压力测试。你可以在模拟器上配置一个人工的区块延迟，以模拟真实的网络延时。

请记住，DoS 攻击是严重的安全问题，DoS 风险应与业务逻辑一起审查。

另外，存储在 Flow 中是有成本的，不要让人们在你的账户上存储东西，尽量把存储推给用户的账户。

链下处理索引。例如：如果你正在制定一个拍卖合同，不要存储所有的出价，只存储中标价，其余的用事件来通知其他网络。
