46.写带函数类型参数的函数的时候使用inline修饰符

你可能已经注意到，几乎所有 Kotlin 标准库中的 高阶函数都有一个 inline 修饰符。 你有没有问过自己为什么要这样定义它们？ 例如，下面是 Kotlin 标准库中的重复函数是如何实现的：

```kotlin
inline fun repeat(times: Int, action: (Int) -> Unit) {
    for (index in 0 until times) {
        action(index)
    } 
}
```

这个 inline 修饰符的作用是在编译过程中，这个函数的所有使用都替换为它的主体。 此外，repeat 内的所有函数参数调用都替换为这些函数体。因此，以下重复函数调用：

```kotlin
 repeat(10) {
     print(it)
}
```

在编译期间将替换为以下内容：

```kotlin
for (index in 0 until 10) {
    print(index)
}
```
与正常函数的执行方式相比，这是一个重大变化。 在普通函数中，执行跳转到这个函数体，调用所有语句，然后跳回到调用函数的地方。 用 body 替换调用是一种截然不同的行为。
这种行为有几个优点：

1. 类型参数可以被具体化
1. 带函数参数的函数内联时速度更快
1. 允许非本地值返回

使用此修饰符也有一些代价。 让我们回顾一下内联修饰符的所有优点和成本。

**类型参数可以被具体化**

Java 在旧版本中没有泛型。 它们于 2004 年在 J2SE 5.0 版本中添加到 Java 编程语言中。 但是它们仍然不存在于 JVM 字节码中。 因此，在编译期间，泛型类型会被删除。 例如 List<Int> 编译为 List。 这就是我们无法检查对象是否为 List<Int> 的原因。 我们只能检查它是否是一个列表。

```kotlin
any is List<Int> // Error
any is List<*> // OK
```
![image.png](https://cdn.nlark.com/yuque/0/2022/png/10378482/1656392936077-1306dbb0-9bc9-4637-b1a5-42dea922a3b6.png#clientId=u0ff6f2f4-948b-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=117&id=uafc5217e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=117&originWidth=460&originalType=binary&ratio=1&rotation=0&showTitle=false&size=16976&status=done&style=none&taskId=ub347bea6-16b5-45df-b2c6-a3dbde1c8e0&title=&width=460)
因为这些原因，我们不能在一个类型参数上操作：
```kotlin
fun <T> printTypeName() {
    print(T::class.simpleName) // ERROR
}
```
我们可以通过内联函数来克服这个限制。 函数调用被它的主体替换，所以类型参数的使用可以被类型参数替换，通过使用 reified 修饰符：
```kotlin
inline fun <reified T> printTypeName() {
    print(T::class.simpleName)
} 
// Usage
printTypeName<Int>() // Int
printTypeName<Char>() // Char
printTypeName<String>() // String
```
在编译过程中，printTypeName 的主体替换了用法，类型参数替换了类型参数：
```kotlin
print(Int::class.simpleName) // Int
print(Char::class.simpleName) // Char
print(String::class.simpleName) // String
```
reified 是一个有用的修饰符。 例如，它在标准库 中的 filterIsInstance 中用于仅过滤特定类型的元素：
```kotlin
class Worker
class Manager

val employees: List<Any> =
    listOf(Worker(), Manager(), Worker())

val workers: List<Worker> =
    employees.filterIsInstance<Worker>()
```

**具有函数参数的函数在内联时更快**

更具体地说，所有函数在内联时都会稍微快一些。 无需执行跳转并跟踪回栈。 这就是为什么在标准库 中经常使用的小函数经常被内联：

```kotlin
inline fun print(message: Any?) {
    System.out.print(message)
}
```

当一个函数没有任何函数参数时，这种差异很可能是微不足道的。 这就是 IntelliJ 给出这样警告的原因：

![image.png](https://cdn.nlark.com/yuque/0/2022/png/10378482/1656393412085-c40c32ad-f00b-48a9-a83b-127796aae094.png#clientId=u0ff6f2f4-948b-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=105&id=uce0cf702&margin=%5Bobject%20Object%5D&name=image.png&originHeight=105&originWidth=827&originalType=binary&ratio=1&rotation=0&showTitle=false&size=20934&status=done&style=none&taskId=u1664dc5e-e2bb-4278-bf05-e9cb8eeed79&title=&width=827)
要了解原因，我们首先需要了解将函数作为对象进行操作的问题所在。 这些使用函数字面量创建的对象需要以某种方式保存。 在 Kotlin/JS 中，这很简单，因为 JavaScript 将函数视为一等公民。因此在 Kotlin/JS 中，它要么是函数，要么是函数引用。在 Kotlin/JVM 中，需要使用 JVM 创建一些对象 、匿名类或普通类。 因此下面的 lambda 表达式：

```kotlin
val lambda: ()->Unit = {
 // code
}
```
会编译成一个类， 一个 JVM 匿名类：
```kotlin
// Java
Function0<Unit> lambda = new Function0<Unit>() {
    public Unit invoke() {
        // code
    } 
};
```
或者它可以编译成一个在单独文件中定义的普通类：
```kotlin
// Java
// Additional class in separate file
public class Test$lambda implements Function0<Unit> {
    public Unit invoke() {
        // code
    } 
} 

// Usage
Function0 lambda = new Test$lambda()
```

这两个情况之间没有显着差异。请注意，函数类型转换为 Function0 类型。 这就是 JVM 中没有参数的函数类型被编译成的。其他函数类型也是如此：

- ()->Unit 编译为 Function0<Unit>
- ()->Int 编译为 Function0<Int>
- (Int)->Int 编译为 Function1<Int, Int>
- (Int, Int)->Int 编译为 Function2<Int, Int, Int>

所有这些接口都是由 Kotlin 编译器生成的。 你不能在 Kotlin 中显式使用它们，因为它们是按需生成的。 我们应该改用函数类型。 尽管知道函数类型只是接口，但您会发现一些新的可能性：
```kotlin
class OnClickListener: ()->Unit {
    override fun invoke() {
        // ...
    }
}
```
如第 45 条：避免不必要的对象创建，将函数体包装到对象中会减慢代码速度。这就是为什么在下面的两个函数中，第一个会更快：
```kotlin
inline fun repeat(times: Int, action: (Int) -> Unit) {
    for (index in 0 until times) {
        action(index)
    }
}

fun repeatNoinline(times: Int, action: (Int) -> Unit) {
    for (index in 0 until times) {
        action(index)
    }
}
```

差异是可见的，但在实际示例中很少显示出来。然而如果我们设计好我们的测试 ，可以清楚地看到这种差异：

```kotlin
@Benchmark
fun nothingInline(blackhole: Blackhole) {
    repeat(100_000_000) {
        blackhole.consume(it)
    }
}

@Benchmark
fun nothingNoninline(blackhole: Blackhole) {
    noinlineRepeat(100_000_000) {
        blackhole.consume(it)
    }
}
```

第一个平均占用我的计算机 189 毫秒。 第二个平均需要 447 毫秒。 这种差异来自这样一个事实，即在第一个函数中，我们只迭代数字并调用一个空函数。 在第二个函数中，我们调用了一个迭代数字并调用一个对象的方法，而这个对象调用了一个空函数。 所有这些差异都来自于我们使用了一个额外的对象这一事实（第 45 条：避免不必要的对象创建）。
举一个更典型的例子，假设我们有 5000 种产品，我们需要总结所购买产品的价格。 我们可以通过以下方式简单地做到这一点：

```kotlin
users.filter { it.bought }.sumByDouble { it.price }
```

在我的机器中，计算平均需要 38 毫秒。 如果 filter 和 sumByDouble 函数不是内联的，那会是多少？在我的机器上平均 42 毫秒。 这看起来不是很多，但每次使用方法进行集合处理时，都会有 ∼10% 的差异。
当我们在函数文字中捕获局部变量时，内联函数和非内联函数之间更显着的区别就会显现出来。 捕获的值也需要包装到某个对象中，并且无论何时使用它，都需要使用该对象来完成。例如，在以下代码中：

```kotlin
var l = 1L
noinlineRepeat(100_000_000) {
 l += it
}
```

局部变量不能真正直接在非内联 lambda 中使用。 这就是为什么在编译期间，a 的值将被包装到一个引用对象中：

```kotlin
val a = Ref.LongRef()
a.element = 1L
noinlineRepeat(100_000_000) {
    a.element = a.element + it
}
```
这是一个更明显的区别，因为通常，此类对象可能会被多次使用：每次我们使用由函数字面量创建的函数时。 例如，在上面的例子中，我们使用了两次。 因此，额外的对象使用将发生 2 * 100 000 000。要了解这种差异，让我们比较以下函数：

```kotlin
@Benchmark
// On average 30 ms
fun nothingInline(blackhole: Blackhole) {
    var l = 0L
    repeat(100_000_000) {
        l += it
    } 
    blackhole.consume(l)
}

@Benchmark
// On average 274 ms
fun nothingNoninline(blackhole: Blackhole) {
    var l = 0L
    noinlineRepeat(100_000_000) {
        l += it
    }
    blackhole.consume(l)
}
```
我机器上的第一个需要 30 毫秒。 第二个274ms。 这来自于函数被编译为对象，并且需要包装局部变量的事实的累积效应。 这是一个显着的区别。 因为在大多数情况下，我们不知道如何使用带有函数类型参数的函数，所以当我们定义带有此类参数的实用函数时，例如用于集合处理，最好将其内联。 这就是为什么 stdlib 中大多数带有函数类型参数的扩展函数都是内联的。

**允许返回飞本地值**

先前定义的 repeatNoninline 看起来很像一个控制结构。 只需将其与 if 或 for 循环进行比较：

```kotlin
if(value != null) {
    print(value)
} 

for (i in 1..10) {
    print(i)
} 

repeatNoninline(10) {
    print(it)
}
```
虽然一个显着的区别是内部不允许返回值：

```kotlin
fun main() {
    repeatNoinline(10) {
        print(it)
        return // ERROR: Not allowed
    } 
}
```
这是函数文字被编译成的结果。 当我们的代码位于另一个类中时，我们不能从 main 返回。内联函数文字时没有这样的限制。 无论如何，代码都将位于 main 函数中。
```kotlin
fun main() {
    repeat(10) {
        print(it)
        return // OK
    } 
}
```
多亏了这一点，函数的外观和行为更像控制结构：
```kotlin
fun getSomeMoney(): Money? {
    repeat(100) {
        val money = searchForMoney()
        if(money != null) return money
    } 
    return null
}
```

**inline修饰符的成本**

inline是一个有用的修饰符，但它不应该在任何地方使用。内联函数不能递归。 否则，它们将无限地替换它们的调用。 循环周期特别危险，因为目前它在 IntelliJ 中没有显示错误：
```kotlin
inline fun a() { b() }
inline fun b() { c() }
inline fun c() { a() }
```
内联函数不能使用具有更严格可见性的元素。 我们不能在公共内联函数中使用私有或内部函数或属性。 事实上，内联函数不能使用任何具有更严格可见性的东西：
```kotlin
internal inline fun read() {
    val reader = Reader() // Error
    // ...
}

private class Reader {
    // ...
}
```
这就是为什么它们不能用于隐藏实现，因此它们很少在类中使用。
最后，它们使我们的代码增长。 要查看比例，假设我非常喜欢打印 3。我首先定义了以下函数：
```kotlin
 inline fun printThree() {
     print(3) 
 }
```
我想调用它 3 次，所以我添加了这个函数：
```kotlin
inline fun threePrintThree() {
    printThree()
    printThree()
    printThree()
}
```
仍不满意，又定义了如下函数：
```kotlin
inline fun threeThreePrintThree() {
    threePrintThree()
    threePrintThree()
    threePrintThree()
}

inline fun threeThreeThreePrintThree() {
    threeThreePrintThree()
    threeThreePrintThree()
    threeThreePrintThree()
}
```
它们都编译成什么？ 前两个非常可读：
```kotlin
inline fun printThree() {
    print(3) 
} 

inline fun threePrintThree() {
    print(3)
    print(3)
    print(3)
}
```
后面两个编译成了如下的样子：
```kotlin
inline fun threeThreePrintThree() {
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
}

inline fun threeThreeThreePrintThree() {
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
    print(3)
}
```
这是一个抽象的例子，但它显示了内联函数的一个大问题：当我们过度使用它们时，代码增长得非常快。 我在一个真实的项目中遇到了这个问题。 有太多内联函数相互调用是危险的，因为我们的代码可能会开始呈指数增长.

**Crossinline和noinline**

在某些情况下，我们想要内联一个函数，但由于某种原因，我们不能内联所有函数类型的参数。 在这种情况下，我们可以使用以下修饰符：

- crossinline - 这意味着函数应该被内联，但不允许非本地返回。 当此函数用于另一个不允许非本地返回的范围时，我们会使用它，例如在另一个未内联的 lambda 中。
-  noinline - 这意味着这个参数根本不应该被内联。 它主要用于当我们将此函数用作另一个未内联函数的参数时。

```kotlin
inline fun requestNewToken(
    hasToken: Boolean,
    crossinline onRefresh: () -> Unit,
    noinline onGenerate: () -> Unit
) {
    if (hasToken) {
        httpCall("get-token", onGenerate) // We must use
        // noinline to pass function as an argument to a
        // function that is not inlined
    } else {
        httpCall("refresh-token") {
            onRefresh() // We must use crossinline to
            // inline function in a context where
            // non-local return is not allowed
            onGenerate()
        }
    }
}

fun httpCall(url: String, callback: () -> Unit) {
    /*...*/
}
```
很高兴知道这两个修饰符的含义是什么，但我们可以不用记住它们，因为 IntelliJ IDEA 在需要它们时会建议它们：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/10378482/1656412202629-2e20cdba-7be6-492d-8b85-af18858182c8.png#clientId=u48877a94-2938-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=238&id=uf0e8e3d5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=238&originWidth=781&originalType=binary&ratio=1&rotation=0&showTitle=false&size=45038&status=done&style=none&taskId=u0e479c5a-9cb3-46e7-bf96-8b0e1e57889&title=&width=781)

**总结**

我们使用内联函数的主要情况是：

-  非常常用的功能，例如打印。
- 需要将具体类型作为类型参数传递的函数，例如filterIsInstance。
- 当我们使用函数类型的参数定义顶级函数时。 尤其是辅助函数，如集合处理函数（如 map、filter、flatMap、joinToString）、范围函数（如also、apply、let）或顶级实用程序函数（如重复、运行、与）。

我们很少使用内联函数来定义 API，我们应该小心一个内联函数调用其他内联函数的情况。 请记住，代码增长会累积。
