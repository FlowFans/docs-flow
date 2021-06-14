# 合约 （Contracts）

Cadence中的合同是位于Flow中帐户的合同存储区域中的接口，结构，资源，数据（其状态）和代码（其功能）的类型定义的集合。

在Cadence中，必须在合同中定义所有复合类型，例如结构，资源，事件和这些类型的接口。

 **可以使用**[**授权帐户**](https://docs.onflow.org/cadence/language/accounts)**的`contracts` 对象来创建，更新和删除合同。**

 使用`contract`关键字声明合同。关键字后跟合同名称。

```javascript
pub contract SomeContract {
    // ...
}
```

合同的另一个重要特征是，合同中声明的资源和事件的实例只能在同一合同中声明的函数或类型内创建/发出。

无法在合同外部创建资源和事件的实例。

```javascript
pub contract HelloWorld {

    // 定义变量
    pub let greeting: String

    // 定义函数
    pub fun hello(): String {
        return self.greeting
    }
    
    // 构造函数
    init() {
        self.greeting = "Hello World!"
    }
}
```



