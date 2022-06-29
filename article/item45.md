45.避免不必要的对象创建

对象创建有时会很昂贵，而且总是要付出一些代价。 这就是为什么避免不必要的对象创建可能是一项重要的优化。 它可以在许多层面上完成。 例如，在 JVM 中，可以保证字符串对象将被运行在同一虚拟机中的任何其他代码重用，而这些代码恰好包含相同的字符串文字:

```kotlin
val str1 = "Lorem ipsum dolor sit amet"
val str2 = "Lorem ipsum dolor sit amet"
print(str1 == str2) // true
print(str1 === str2) // true
```

装箱原始类型（整数、长整数）在 JVM 中也可以在它们很小的时候重用（默认情况下，整数缓存保存范围从 -128 到 127 的数字）
```kotlin
val i1: Int? = 1
val i2: Int? = 1
print(i1 == i2) // true
print(i1 === i2) // true, because i2 was taken from cache
```
引用相等表明这是同一个对象。 如果我们使用小于 -128 或大于 127 的数字，则会产生不同的对象：
```kotlin
val j1: Int? = 1234
val j2: Int? = 1234 
print(j1 == j2) // true
print(j1 === j2) // false
```
请注意，在底层可空类型用于强制 Integer 而不是 int 。 当我们使用 Int 时，它通常被编译为原始 int，但是当我们将其设为可空或将其用作类型参数时，则使用 Integer 代替。 这是因为原始类型不能为空，也不能用作类型参数。 知道这种机制是在语言中引入的，你可能想知道它们有多么重要。 对象创建成本高吗？

**对象的创建是昂贵的吗？**

将某些东西包装到一个对象中需要 3 部分成本：

-  对象占用额外空间。在现代 64 位 JDK 中，对象具有 一个 12 字节的标头，填充为 8 字节的倍数，因此最小对象大小为 16 字节。对于 32 位 JVM，开销为 8 个字节。此外，对象引用也占用空间。通常，在 32 位平台或 64 位平台上最多为 -Xmx32G 的引用为 4 个字节，超过 32Gb (-Xmx32G) 为 8 个字节。这些是相对较小的数字，但它们可能会增加可观的成本。当我们考虑像整数这样的小元素时，它们会有所作为。 Int 作为一个原始类型适合 4 个字节，当作为我们今天主要使用的 64 位 JDK 上的包装类型时，它需要 16 个字节（它适合头部后面的 4 个字节）+它的引用需要 4 或 8 个字节。最后，它需要 5 或 6 倍的空间。
- 封装元素时，Access 需要额外的函数调用。这又是一个很小的功能使用成本。
- 对象占用额外空间。在现代 64 位 JDK 中，对象有一个 12 字节的头，填充为 8 字节的倍数，因此最小对象大小为 16 字节。对于 32 位 JVM，开销为 8 个字节。此外，对象引用也占用空间。通常，在 32 位平台或 64 位平台上最多为 -Xmx32G 的引用为 4 个字节，超过 32Gb (-Xmx32G) 为 8 个字节。这些是相对较小的数字，但它们可能会增加可观的成本。当我们考虑像整数这样的小元素时，它们会有所作为。 Int 作为一个原始类型适合 4 个字节，当作为我们今天主要使用的 64 位 JDK 上的包装类型时，它需要 16 个字节（它适合头后面的 4 个字节）+它的引用需要 4 或 8 个字节。最后，它需要 5 到 6 倍的空间。

```kotlin

class A

private val a = A()

// Benchmark result: 2.698 ns/op
fun accessA(blackhole: Blackhole) {
    blackhole.consume(a)
}

// Benchmark result: 3.814 ns/op
fun createA(blackhole: Blackhole) {
    blackhole.consume(A())
}

// Benchmark result: 3828.540 ns/op
fun createListAccessA(blackhole: Blackhole) {
    blackhole.consume(List(1000) { a })
}

// Benchmark result: 5322.857 ns/op
fun createListCreateA(blackhole: Blackhole) {
    blackhole.consume(List(1000) { A() })
}
```

通过消除对象，我们可以避免所有三个成本。 通过重用对象，我们可以消除第一个和第三个。 知道了这一点，你可能会开始考虑限制代码中不必要对象的数量。 让我们看看我们可以做到这一点的一些方法。

**对象声明**

重用对象而不是每次都创建它的一种非常简单的方法是使用对象声明（单例）。 看一个例子，假设你需要实现一个链表。 链表可以是空的，也可以是包含一个元素并指向其余元素的节点。 这是它的实现方式：
```kotlin

sealed class LinkedList<T>

class Node<T>(
    val head: T,
    val tail: LinkedList<T>
) : LinkedList<T>()

class Empty<T> : LinkedList<T>()

// Usage
val list: LinkedList<Int> =
    Node(1, Node(2, Node(3, Empty())))

val list2: LinkedList<String> =
    Node("A", Node("B", Empty()))
```
这个实现的一个问题是，每次我们创建一个列表时，我们都需要创建一个 Empty 实例。 相反，我们应该只拥有一个实例并普遍使用它。 唯一的问题是泛型类型。 我们应该使用什么泛型类型？我们希望这个空列表是所有列表的子类型。 我们不能使用所有类型，但我们也不需要。 一个解决方案是我们可以将其设为 Nothing 列表。 Nothing 是每种类型的子类型，因此一旦这个列表是协变的（out 修饰符），LinkedList<Nothing> 将成为每个 LinkedList 的子类型。 使类型参数协变在这里真正有意义，因为列表是不可变的，并且这种类型仅用于 out 位置（第 24 条：考虑泛型类型的方差）。 所以这是改进的代码：
```kotlin
sealed class LinkedList<out T>
    
class Node<out T>(
    val head: T,
    val tail: LinkedList<T>
) : LinkedList<T>()

object Empty : LinkedList<Nothing>()

// Usage
val list: LinkedList<Int> =
    Node(1, Node(2, Node(3, Empty)))

val list2: LinkedList<String> =
    Node("A", Node("B", Empty))
```
这是一个经常使用的有用技巧，尤其是当我们定义不可变的密封类时。 不可变，因为将它用于可变对象可能会导致与共享状态管理相关的微妙且难以检测的错误。 一般规则是不应该缓存可变对象（第 1 项：限制可变性）。 除了对象声明之外，还有更多方法可以重用对象。 另一种是带有缓存的工厂函数。

**带有缓存的工厂函数**

每次我们使用构造函数时，都会有一个新对象。 尽管使用工厂方法时不一定正确。 工厂函数可以有缓存。 最简单的情况是工厂函数总是返回相同的对象。 例如，标准库中的 emptyList 是如何实现的：
```kotlin
fun <T> emptyList(): List<T> = EmptyList
```
有时我们有一组对象，我们返回其中一个。例如，当我们使用 Kotlin 协程Dispatchers.Default 中的默认调度程序时，它有一个线程池，每当我们使用它启动任何东西时，它都会 在一个不使用的地方启动它。 同样，我们可能有一个与数据库的连接池。 当对象创建量很大时，拥有一个对象池是一个很好的解决方案，我们可能需要同时使用一些可变对象。
也可以对参数化工厂方法进行缓存。 在这种情况下，我们可能会将对象保存在map中：
```kotlin
private val connections: MutableMap<String, Connection> =
    mutableMapOf<String, Connection>()

fun getConnection(host: String) =
    connections.getOrPut(host) { createConnection(host) }
```
缓存可用于所有纯函数。 在这种情况下，我们称之为记忆。 例如，这是一个根据定义计算某个位置的斐波那契数的函数：
```kotlin
private val FIB_CACHE: MutableMap<Int, BigInteger> =
    mutableMapOf<Int, BigInteger>()

fun fib(n: Int): BigInteger = FIB_CACHE.getOrPut(n) {
    if (n <= 1) BigInteger.ONE else fib(n - 1) + fib(n - 2) 
}
```
现在我们的方法在第一次运行时几乎和线性解决方案一样有效，如果已经计算过了，它会立即给出结果。 下表给出了此示例与经典线性斐波那契实现在示例机器上的比较。 下面还介绍了我们与之比较的迭代实现。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/10378482/1656463240924-3f0bed4c-349d-4763-a6a1-2b5d577a406b.png#clientId=uc727ef1a-cf21-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=143&id=u0032910a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=143&originWidth=615&originalType=binary&ratio=1&rotation=0&showTitle=false&size=12262&status=done&style=none&taskId=u8bbf7291-8b39-407f-93e4-ecdf3180c4a&title=&width=615)
```kotlin

fun fibIter(n: Int): BigInteger {
    if (n <= 1) return BigInteger.ONE
    var p = BigInteger.ONE
    var pp = BigInteger.ONE
    for (i in 2..n) {
        val temp = p + pp
        pp = p
        p = temp
    }
    return p
}
```

你可以看到第一次使用此函数比使用经典方法要慢，因为检查值是否在缓存中并将其添加到那里会产生额外的开销。 添加值后，检索几乎是瞬时的。
但是它有一个明显的缺点：我们要保留和使用更多的内存，因为 Map 需要存储在某个地方。 如果这在某个时候被清除，一切都会好起来的。 但请考虑到对于垃圾收集器 (GC)，缓存与将来可能需要的任何其他静态字段之间没有区别。 即使我们不再使用 fib 函数，它也会尽可能长时间地保存这些数据。 有帮助的一件事是使用可以在需要内存时由 GC 删除的软引用。 它不应该与弱引用混淆。 简单来说，区别在于：

-  弱引用不会阻止垃圾收集器清理值。 因此，一旦没有其他引用（变量）使用它，该值将被清除。
- 软引用也不能保证 GC 不会清理该值，但在大多数 JVM 实现中，除非需要内存，否则不会清理该值。 当我们实现缓存时，软引用是完美的。

这是一个示例属性委托（详见第 21 条：使用属性委托来提取公共属性模式），它按需创建一个映射并让我们使用它，但不会阻止垃圾收集器在需要内存时回收此映射（完整实现应该 包括线程同步）：
```kotlin

private val FIB_CACHE: MutableMap<Int, BigInteger> by
SoftReferenceDelegate { mutableMapOf<Int, BigInteger>() }

fun fib(n: Int): BigInteger = FIB_CACHE.getOrPut(n) {
    if (n <= 1) BigInteger.ONE else fib(n - 1) + fib(n - 2)
}

class SoftReferenceDelegate<T : Any>(
    val initialization: () -> T
) {
    private var reference: SoftReference<T>? = null

    operator fun getValue(
        thisRef: Any?,
        property: KProperty<*>
    ): T {
        val stored = reference?.get()
        if (stored != null) return stored
        val new = initialization()
        reference = SoftReference(new)
        return new
    }
}
```

设计好缓存并不容易，归根结底，缓存始终是一种权衡：内存的性能。 记住这一点，并明智地使用缓存。 没有人愿意从性能问题转向内存不足问题。

**重量实例的提升**

一个非常有用的性能技巧是将重量实例提升到外部范围。 当然，如果可能，我们应该将所有繁重的操作从集合处理函数提升到一般处理范围。例如，在这个函数中，我们可以看到我们需要为 Iterable 中的每个元素找到最大元素：
```kotlin
fun <T: Comparable<T>> Iterable<T>.countMax(): Int =
    count { it == this.max() }
```
更好的解决方案是将最大元素提取到 countMax 函数的级别：
```kotlin
fun <T: Comparable<T>> Iterable<T>.countMax(): Int {
    val max = this.max()
    return count { it == max }
}
```
这个解决方案对性能更好，因为我们不需要在每次迭代时都在接收器上找到最大元素。 请注意，它还通过在扩展接收器上调用 max 来提高可读性，因此它在所有迭代中都是相同的。
将值计算提取到外部范围以不计算它是一种重要的做法。 这听起来很明显，但并不总是那么清楚。 看看这个函数，我们使用正则表达式来验证字符串是否包含有效的 IP 地址：
```kotlin
fun String.isValidIpAddress(): Boolean {
    return this.matches(
        "\\A(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\\.){ 3 }(?:25[0-5]|2[0-4][0-9]|[01] 4 ?[0-9][0-9]?)\\z".toRegex())
}
// Usage
print("5.173.80.254".isValidIpAddress()) // true
```
这个函数的问题是我们每次使用它时都需要创建Regex对象。 这是一个严重的缺点，因为正则表达式模式编译是一项复杂的操作。 这就是为什么这个函数不适合在我们代码的性能受限部分重复使用。 虽然我们可以通过将正则表达式提升到顶层来改进它：
```kotlin
private val IS_VALID_EMAIL_REGEX = "\\A(?:(?:25[0-5]
|2[0-4][0-9]|[01]?[0-9][0-9]?)\\.){3}(?:25[0-5]|2[0-4] 3 [0-9]|[01]?[0-9][0-9]?)\\z".toRegex()

fun String.isValidIpAddress(): Boolean =
    matches(IS_VALID_EMAIL_REGEX)
```
如果这个函数和其他一些函数在一个文件中，并且我们不想在不使用的情况下创建这个对象，我们甚至可以懒惰地初始化正则表达式：
```kotlin
private val IS_VALID_EMAIL_REGEX by lazy {
    "\\A(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\\.){ 3 }(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\\z".toRegex()
}
```
当我们处理类时，使属性变得惰性也很有用。

**延迟初始化**

通常当我们需要创建一个重类时，最好是懒惰地去做。 例如，假设 A 类需要重的 B、C 和 D 实例。 如果我们只是在创建类的过程中创建它们，那么 A 的创建将会非常繁重，因为它需要创建 B、C 和 D，然后是它的主体的其余部分。 对象创建的重量只会累积。

```kotlin
class A { 2 val b = B()
    val c = C()
    val d = D()
    
    //...
}
```

不过有治愈的方法。 我们可以延迟初始化那些重量的对象：
```kotlin
class A {
    val b by lazy { B() }
    val c by lazy { C() }
    val d by lazy { D() }

    //...
}
```
然后每个对象将在其第一次使用之前被初始化。这些对象的创建成本将分散而不是累积。
请记住，这把剑是双刃剑。 你可能会遇到对象创建可能很繁重的情况，但需要方法尽可能快。 假设 A 是后端应用程序中响应 HTTP 调用的控制器。 它启动很快，但第一次调用需要所有重物初始化。 因此，第一次调用需要等待更长的时间才能得到响应，这与我们的应用程序运行多长时间无关。 这不是我们想要的行为。这也可能会干扰我们的性能测试。

**使用原语言（？？）**

在 JVM 中，我们有一个特殊的内置类型来表示数字或字符等基本元素。 它们被称为原语，Kotlin/JVM 编译器尽可能在后台使用它们。尽管在某些情况下需要使用包装类。 两种主要情况是：

1. 当我们对可空类型进行操作时（原语不能为空）
1. 当我们使用类型作为泛型时

简而言之：

![image.png](https://cdn.nlark.com/yuque/0/2022/png/10378482/1656465477289-18589e9c-59a2-4e26-a116-2429e08b76e2.png#clientId=uc727ef1a-cf21-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=101&id=uc25e5e7e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=101&originWidth=461&originalType=binary&ratio=1&rotation=0&showTitle=false&size=5808&status=done&style=none&taskId=u91e76bac-d6ad-475f-a36a-6b15a44f7bb&title=&width=461)

知道了这一点，你可以优化代码以在底层使用原语而不是包装类型。 这种优化主要在 Kotlin/JVM 和一些 Kotlin/Native 风格上有意义。 在 Kotlin/JS 上根本没有。 还需要记住，仅当对数字的操作重复多次时才有意义。与其他操作相比，原始类型和包装类型的访问都相对较快。 当我们处理一个非常大的集合时（我们将在第 51 条中讨论它：考虑使用原语进行性能关键处理的数组）或当我们对一个对象进行密集操作时，差异就会显现出来。 还要记住，强制更改可能会导致代码可读性降低。这就是为什么我建议仅针对我们代码和库中的性能关键部分进行此优化。 你可以使用分析器找出对性能至关重要的因素。
举个例子，假设你为 Kotlin 实现了一个标准库，并且你想要引入一个函数，该函数将返回最大元素或如果该迭代为空则返回 null。 你不想多次迭代可迭代对象。 这不是一个小问题，但可以通过以下函数解决：
```kotlin
fun Iterable<Int>.maxOrNull(): Int? {
    var max: Int? = null
    for (i in this) {
        max = if (i > (max ?: Int.MIN_VALUE)) i else max
    }
    return max
}

```
这种实现有严重的缺点：
1.我们需要在每一步都使用一个Elvis操作符
2. 我们使用一个可以为空的值，所以在 JVM 的底层，会有一个 Integer 而不是 int。
解决这两个问题需要我们使用while循环来实现迭代
```kotlin

fun Iterable<Int>.maxOrNull(): Int? {
    val iterator = iterator()
    if (!iterator.hasNext()) return null
    var max: Int = iterator.next()
    while (iterator.hasNext()) {
        val e = iterator.next()
        if (max < e) max = e
    }
    return max
}

```
对于从 1 到 1000 万个元素的集合，在我的电脑中，优化实现耗时 289 ms，而前一个耗时 518 ms。 这几乎快了 2 倍，但请记住，这是一个旨在显示差异的极端案例。 在对性能不重要的代码中，这种优化很少是合理的。尽管如果你实现一个 Kotlin 标准库，那么一切都是性能关键的。 这就是这里选择第二种方法的原因：
```kotlin
/**
 * Returns the largest element or `null` if there are
 * no elements.
 */
public fun <T : Comparable<T>> Iterable<T>.max(): T? {
    val iterator = iterator()
    if (!iterator.hasNext()) return null
    var max = iterator.next()
    while (iterator.hasNext()) {
        val e = iterator.next()
        if (max < e) max = e
    }
    return max
}
```

**总结**

在本项目中，你已经看到了避免创建对象的不同方法。其中一些在可读性方面很便宜：那些应该自由使用。 例如，从循环或函数中取出重物通常是一个好主意。 这是一个很好的性能实践，而且由于这种提取，我们可以命名这个对象，这样我们的函数就更容易阅读了。 可能应该跳过那些更严格或需要更大更改的优化。 我们应该避免过早的优化，除非我们在我们的项目中有这样的指导方针，或者我们开发了一个可能以谁知道什么方式使用的库。 我们还学习了一些可用于优化代码的性能关键部分的优化。
