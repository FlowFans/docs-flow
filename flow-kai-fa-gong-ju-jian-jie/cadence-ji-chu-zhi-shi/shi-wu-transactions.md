# 事务 （Transactions）

交易（事务）是由一个或多个[帐户](https://docs.onflow.org/cadence/language/accounts)签名 并发送到链中与之交互的对象。

Transactions 的结构如下：

首先，事务可以使用导入语法从外部帐户导入任意数量的类型。

四个可选的主要阶段：准备阶段，前置条件，执行阶段，后置条件。顺序执行。

```javascript
import FungibleToken from 0x01

transaction {
    // 变量定义
    let localVar: Int

    prepare(signer1: AuthAccount, signer2: AuthAccount) {
        // 准备阶段: 当您的交易需要访问签名帐户的私有AuthAccount对象时，将使用该阶段。
        
        // 仅在prepare阶段内可以直接访问签名帐户。
    }

    pre {
        // 前置条件: 用于检查在执行其余事务之前是否存在明确的条件。
        
        // 一个常见的示例是在帐户之间转移令牌之前检查必要的余额。
    }

    execute {
        // 执行阶段: execute阶段完全按照其说的去做，它执行事务的主要逻辑。
    }

    post {
        // 后置条件: post阶段内部的语句用于验证您的事务逻辑是否已正确执行。它包含零个或多个条件检查。
        
        // 如果任何条件检查结果为false，则事务将失败并被完全还原。
    }
}
```

