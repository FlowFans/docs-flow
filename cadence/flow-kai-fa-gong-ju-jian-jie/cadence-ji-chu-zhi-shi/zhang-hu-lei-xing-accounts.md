# 账户类型（Accounts）

每个帐户都可以通过两种类型进行访问：

*  **公共帐户**`PublicAccount` ，代表**帐户**的公共可用部分

例如：  任何代码都可以使用内置的`getAccount`函数获取指定地址的`PublicAccount`的对象：

{% code title="script.cdc" %}
```javascript

pub fun main() {

    let acct1 = getAccount(0x01)   // 获取公共账户 0x01 对象
 
    log(acct1.storageUsed)         // 已使用容量
    log(acct1.storageCapacity)     // 账户容量
}
```
{% endcode %}

*  **授权帐户**`AuthAccount`，代表**帐户**的授权部分

 `AuthAccount`可以完全访问其[存储](https://docs.onflow.org/cadence/language/accounts/#account-storage)，公钥和代码。 只有已[签名的交易](https://docs.onflow.org/cadence/language/accounts/transactions)才能获得`AuthAccount`帐户的。对于事务的每个脚本签名者，将`AuthAccount`传递到事务的`prepare`阶段。

{% code title="transaction.cdc" %}
```javascript

transaction() {
    // 前置条件
    prepare(signer: AuthAccount) {
        let account = AuthAccount(payer: signer) // 获取授权账户
        
        log(account.storageUsed)     // 已使用容量
        log(account.storageCapacity) // 账户容量
    }
}
```
{% endcode %}



