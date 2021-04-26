# 引用 （References）

可以创建对对象（即资源或结构）的引用。引用可用于访问字段并在引用的对象上调用函数。

引用可以**复制**， 即它们是值类型。

 引用是通过使用`&`运算符创建的，其后是对象，`as`关键字后面是访问的类型。

```javascript
let counter <- create Counter(count: 42)
let counterRef: &Counter = &counter as &Counter
```

引用 可能是经过**授权的**或**未经授权的**。

 授权引用具有`auth`修饰符，即完整语法为`auth &T`，而未授权引用则没有修饰符。

 引用是短暂的，即它们无法[存储](https://docs.onflow.org/cadence/language/accounts#account-storage)。相反，请考虑[存储功能并](https://docs.onflow.org/cadence/language/capability-based-access-control)在需要时[借用它](https://docs.onflow.org/cadence/language/capability-based-access-control)。 



