35.考虑为负责的对象创建定义一个DSL
Domain Specific Language (**DSL**)领域特定语言
DSL常用于定义更复杂的对象或者对象的层次结构。DSL的定义不是很简单，但是一旦定义了，由于DSL隐藏了复杂性和模板，调用者可以更清晰的表达他的意图。
```kotlin
//DSL定义html
body {
     div {
        a("https://kotlinlang.org") {
             target = ATarget.blank
             +"Main site"
            } 
      } 
     +"Some content"
 }
```
DSL也经常用来定义数据或者配置。可以使用Gradle DSL定义Gradle配置

**定义自己的DSL**

弄懂如何定义自己的DSL，关键点是理解 函数类型里面的receiver概念。先介绍下函数类型的概念：函数类型是表示可用作函数的对象的类型。
几个函数类型的例子

-   ()->Unit - 无参无返回值
- (Int)->Unit - 一个Int参数无返回值
- (Int)->Int -  一个Int参数且返回值Int. 
- (Int, Int)->Int - 两个Int参数且返回值Int. 
- (Int)->()->Unit - 一个Int参数返回另一个函数.另一个函数无参数且
- (()->Unit)->Unit - 参数是另一个函数，无返回值。另一个函数无参无返回值

创建函数类型实例的基本方法：

- 使用lamda表达式
- 使用匿名函数
- 使用函数引用
```kotlin

fun plus(a: Int, b: Int) = a + b
//在函数声明中定义类型，可以在lamda中推断出参数
val plus1: (Int, Int) -> Int = { a, b -> a + b }
val plus2: (Int, Int) -> Int = fun(a, b) = a + b
//函数引用
val plus3: (Int, Int) -> Int = ::plus
//在lamda中定义参数类型，可以推断出函数类型
val plus4 = { a: Int, b: Int -> a + b }
val plus5 = fun(a: Int, b: Int) = a + b

```
匿名扩展函数以及匿名扩展函数类型
**匿名扩展函数类型**又被称为**带有receiver的函数类型**
```kotlin

//扩展函数定义
fun Int.myPlus(other: Int) = this + other
//匿名扩展函数类型
val myPlus1 = fun Int.(other: Int) = this + other
val myPlus2:Int.(Int) ->Int = fun Int.(other:Int) = this + other
val myPlus3:Int.(Int)->Int = {this + it}
```
带有receiver的函数类型最重要的特征是：通过this来改变指向的对象，例如apply函数中，this用来更便捷的调用对象的属性和方法。
**带有receiver的函数类型是构建kotlin DSL的基础**。
