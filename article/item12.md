12：操作符的含义应该和它的函数名保持一致

操作符重载是一个强健的特性，同时和其他强健的特性一样，也是危险的。这样就可以限制每个操作符的用途。
在编程中，能力越大，责任越大。作为一个培训师，我经常见到有些人第一次发现操作符重载时候是如何得意忘形的。例如，一个需要计算数字阶乘的练习：
```kotlin
fun Int.factorial(): Int = (1..this).product()

fun Iterable<Int>.product(): Int =
    fold(1) { acc, i -> acc * i }
```
由于这个函数是作为扩展函数定义的，所以它使用起来很方便：
```kotlin
print(10 * 6.factorial())
```
一个数学家肯定知道阶乘的特殊符号，就是在数字后面放一个叹号
```kotlin
10 * 6!
```
kotlin中没有类似的操作符，但是我们可以使用操作符重载**not**代替：
```kotlin
operator fun Int.not() = factorial()

print(10 * !6) // 7200
```
但是应该这么做嘛？答案是NO. 你只需要去读一下函数的声明，注意函数的名字是not.由于名字的暗示，not不应该这么使用。它表明了逻辑操作，而不是一个数字阶乘。这种使用方式，会造成困惑和误导的。在kotlin中，所有的操作符都只是对应相应名字函数的语法糖，如下表所示。每个操作符都可已被使用函数表达式的函数代替调用。下面的代码会如何？
```kotlin
print(10 * 6.not()) // 7200
```

![image.png](https://cdn.nlark.com/yuque/0/2022/png/10378482/1655020864450-9721d129-6443-4fa7-82b5-95c788c6ed94.png#clientId=u5d8a227a-8170-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=486&id=ufda01218&margin=%5Bobject%20Object%5D&name=image.png&originHeight=972&originWidth=696&originalType=binary&ratio=1&rotation=0&showTitle=false&size=267307&status=done&style=none&taskId=u1e1e6545-a37c-4a8c-b6fc-268715bac56&title=&width=348)

Kotlin长得操作符含义应该保持一致。这是一个非常重要的设计决定。一些语言，比如Scala,给你无线的操作符重载能力。这种自由已知是被一些开发者过渡滥用的。阅读代码的时候，遇到第一次使用的不熟悉的库的时候，即便它有命名良好的函数名和类，也是困难的。现在设想下，操作符被用来做其他含义的操作，只有熟悉范畴理论的开发者才知道那种。这将变得难于理解。你可能需要将每个操作分开单独弄懂，记住它在特定上下文的含义，在脑子中拼接起来弄懂这段代码的含义。在Kotlin中不会遇到这种问题，因为每个操作符都有具体的含义。举个例子，当你见到如下表达式：
```kotlin
x + y == z
```
你应该知道与下面的含义一样
```kotlin
x.plus(y).equal(z)
```
如果plus声明了一个可为空的返回值，也可以如下：
```kotlin
(x.plus(y))?.equal(z) ?: (z === null)
```
这些函数具有具体的名字并且我们希望所有的函数做它们名字象征的操作。使用not操作符来返回阶乘明显就是对这种规则的破坏，不应该再次发生。

**未知情况**

最大的问题是不清楚某些用法是否符合约定.例如，当我们需要重复三次函数的是什么意思？对于一些人来说，他是明白的，意思是实现一个调用三次的函数：
```kotlin
operator fun Int.times(operation: () -> Unit): ()->Unit =
 { repeat(this) { operation() } }
 
 val tripledHello = 3 * { print("Hello") }
 
 tripledHello() // Prints: HelloHelloHello
```
对于另外一些人来说，可能意味着需要调用函数3次：
```kotlin
operator fun Int.times(operation: ()->Unit) {
 repeat(this) { operation() }
 } 

3 * { print("Hello") } // Prints: HelloHelloHello
```
当意义不明确时，最好采用描述性好的扩展函数。如果我们想要让它们像操作符一样使用，可以使用infix:
```kotlin
infix fun Int.timesRepeated(operation: ()->Unit) = {
    repeat(this) { operation() }
}

val tripledHello = 3 timesRepeated { print("Hello") }
tripledHello() // Prints: HelloHelloHello
```
有时，最好使用一个顶级函数代替。重复函数3次已经实现且在标准库中了：
```kotlin
repeat(3) { print("Hello") } // Prints: HelloHelloHello
```

**合适打破这个规则是可以的？**

有一个很重要的情况，当在一种奇怪的方式下使用操作符重载是可以的：当我们设计了一个DSL.一个经典的HTML DSL例子：
```kotlin
body {
    div {
        +"Some text"
    } 
}
```
可以看到向元素种插入文本使用了String.unaryPlus.这是可以接受的，因为明显的是它是DSL的一部分。在这种特定的下上文中，看到这种使用这种不同规则的情况不奇怪。

**总结**

认真的使用操作符重载。函数的名字应该和它的表现一致。避免操作符含义不明确的情况。通过使用一个具有描述性好的名字的普通函数来表达清楚含义。如果你希望具有操作符式样的表达式，就是用infix修饰符或者顶级函数。
