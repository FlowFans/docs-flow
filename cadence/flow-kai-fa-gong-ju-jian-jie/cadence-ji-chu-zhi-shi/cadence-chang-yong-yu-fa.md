# Cadence 常用语法

1. 注释 和 变量命名

```javascript
    // （双斜杠单行注释） 
    /* （多行注释） 构造函数，初始化 */
    
    //  使用 let 关键字声明常量。使用 var 关键字声明变量。 
    //  类型注释： 确保安全性
    pub let greeting: String
```

  2. 数据类型

```javascript
 数据类型： 非资源类型，资源类型
 
 AnyStruct 和 AnyResource
 
 AnyStruct： 是所有非资源类型的顶级类型，即所有非资源类型都是它的子类型。
 
 AnyResource： 是所有资源类型的顶部类型。
```

  3.  可选类型的表示： `？` 

```javascript
let a: Int? = nil
```

可选有两种情况：要么有值，要么什么都没有。用nil表示。 例如，Int是一个非可选整数，并且Int?是一个可选整数，即为空或整数。 目的： 主要解决大多数编程语言中的 “空”值问题。

    4.  Nil-Coalescing 联合运算符:   `??` 

```javascript
    let a: Int? = nil
    let b: Int = a ?? 42
    *只能应用于具有可选类型的值
```

   5. 强制展开符号：   !  

```javascript
let a: Int? = nil
let b: Int = a!
// 程序直接终止 因为 `a` 是空值.
 
let c: Int? = 3
let d: Int = c!
// `d` 初始化赋值 3 因为 `c` 不是 nil.

*只能应用于具有可选类型的值
```

  6. 移动运算符:   `<-`  

```javascript
let b <- create SomeResource() // 资源实例化
```

  7. 强制分配运算符:    `<-!` 

```text
如果变量为nil，则将资源类型的值分配给可选类型的变量。
如果分配给该变量的值不为nil，则程序的执行将中止。

目的： 保证资源的唯一确定性
*强制分配运算符仅用于 资源类型和移动运算符（<-）
```

 8. Never 类型及交换符

```javascript
fun crashAndBurn(): Never {
    panic("An unrecoverable error occurred")
}
字符串、数组、字典

运算符：交换运算符： <->
var a = 1
var b = 2
a <-> b
// `a` 是 `2`， `b` 是 `1`
```

  9. 函数先决条件和后置条件：

```javascript
fun factorial(_ n: Int): Int {
        pre {
            // 要求参数 n 大于等于 0
            n >= 0:
                "factorial is only defined for integers greater than or equal to zero"
        }
        post {
            // 确保 result 值大于等于 1
            result >= 1:
                "the result must be greater than or equal to 1"
        }

        if n < 1 {
            return 1
        }

        return n * factorial(n - 1)
    }

factorial(5)  // is `120`

// Run-time error: The given argument does not satisfy
// the precondition `n >= 0` of the function, the program aborts.
//
factorial(-2)

目的：验证程序的一致性
```

 10.  复合类型： 复合类型只能在合同内声明，而不能在其他地方声明

```javascript
分别有两种复合类型。它们的用法和行为不同。

1. 结构（struct）被复制，它们是值类型。

2. 资源（resource）是移动的，它们是线性类型，必须只使用一次。

当需要对所有权进行建模时（一个值恰好存在于一个位置，并且不应丢失），
资源很有用。

结构不是表示此所有权的理想方法，因为它们已被复制。
这将意味着存在某些资产的多个副本浮动的风险，
这破坏了这些资产具有真实价值所需的稀缺性要求。

结构对于表示可以按逻辑方式分组在一起，
但没有价值或不需要拥有或转让的信息表示有用。

例如，结构可用于包含与公司部门相关的信息，
但是资源将用于表示已分配给该组织用于支出的资产。

资源的嵌套仅允许在其他资源类型内，或者在数据结构（如数组和字典）中，
但在结构中则不允许，因为那样可以复制资源。

pub struct SomeStruct {
    // 结构 声明
}

let a = SomeStruct()  // 实例化

pub resource SomeResource {
    // 资源 声明
}

let b <- create SomeResource() // 实例化

资源是一次只能在一个位置上存在的类型，必须只能使用一次。

必须使用create关键字创建（实例化）资源。

在具有在范围资源（变量，常量参数）的函数的结束时，
资源必须被任一移动或破坏。

当用作常量或变量的初始值，分配给其他变量，
作为参数传递给函数以及从函数返回时，它们将被移动。

可以使用关键字显式销毁资源destroy。

访问字段或调用资源的功能不会移动或破坏它。

移动资源时，在移动之前引用资源的常量或变量将变为无效。
一个无效的资源不能再使用。

为了使资源类型的用法和行为明确，@必须在变量或常量声明，
参数和返回类型的类型注释中使用前缀。

为使资源移动明确，<-必须在资源是常量或变量的初始值。
```

11.  资源不可丢失性：资源对象不能超出范围并动态丢失

```javascript
程序必须明确销毁它或将其移至另一个上下文。
 
//  只声明变量，未使用，报错

let c <- create SomeResource(value: 10)

   
为了明确表明类型是资源类型，并且必须遵循与资源相关联的规则，
必须 @ 在所有类型注释中为该类型添加前缀，例如变量声明，参数或返回类型。
 
let someResource: @SomeResource <- create SomeResource(value: 5)
 
资源必须只使用一次。
 
pub fun forgetToUse(resource: @SomeResource) {
    // 传参却未使用， 报错
}


     
let res <- create SomeResource()

use(resource: <-res)

use(resource: <-res)
 
// 两次调用同一个 资源，报错
```

12. 资源的变量的替换： 无法直接赋值，将导致资源丢失

```javascript
 pub resource R {}
 
 var x <- create R()
 var y <- create R()
 // 不能这样赋值会报错， 资源x 将丢失
 x <- y

// 使用 交换符号 <->  （最佳实践如下）

var replacement <- create R()
x <-> replacement
      
// `x` 是新资源， `replacement` 是旧的资源

// 或者可以简写成
let oldX <- x <- create R()
        
// oldX 旧资源需要被销毁
destroy oldX
```

13. 资源的销毁: 析构函数

资源可能具有析构函数，该析构函数在破坏资源时执行。析构函数没有参数，也没有返回值，并使用destroy名称进行声明。

```javascript
var destructorCalled = false

pub resource Resource {
    // 当资源被销毁时，调用该函数
    destroy() {
        destructorCalled = true
    }
}

let res <- create Resource()
destroy res
// 销毁后 `destructorCalled` 变为 `true`
```

14. 嵌套资源

```javascript
// 子资源
pub resource Child {
    let name: String
    init(name: String){
        self.name = name
    }
}
// 父资源 内包含 子资源
pub resource Parent {
    let name: String
    var child: @Child
    
    init(name: String, child: @Child) {
        self.name = name
        self.child <- child
    }
 
     destroy() {
        // 销毁子资源
        destroy self.child
    }
}
```

15. 继承和抽象类型

有没有支持继承。继承是其他编程语言中的常见功能，它允许将一种类型的字段和功能包括在另一种类型中。

取而代之的是遵循“组成继承”的原则，即从多个单独的部分组成功能的想法，而不是构建继承树。

此外，也不支持抽象类型。抽象类型是其他编程语言中的常见功能，它阻止创建该类型的值，而仅允许创建子类型的值。另外，抽象类型可以声明函数，但是省略它们的实现，而是需要子类型来实现它们。





