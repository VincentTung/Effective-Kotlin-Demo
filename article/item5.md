第5条：明确你对参数和状态的期望

当我们对参数和状态有期望的时候，要尽快声明。在kotlin中主要使用以下方式：

- require block - 一种对参数指定期望的通用方法
- check block-一种对状态指定期望的通用方法
- assert block-一种判断某些条件是否为true的通用方法
- Elvis操作符搭配return或者throw
            - 


下面是使用这些机制一个例子：
```kotlin
// Part of Stack<T>
fun pop(num: Int = 1): List<T> {
    require(num <= size) {
        "Cannot remove more elements than current size"
    } 
    check (isOpen) { "Cannot pop from closed stack" }
    val ret = collection.take(num)
    collection = collection.drop(num)
    assert(ret.size == num)
    return ret
}
```
虽然这种方式不能让我们摆脱在文档中表明限定条件，但是它确实有很多帮助。这种声明性的检查有很多优点：

- 限定条件对那些甚至没看过API文档的人也是可见的。
- 如果不满足条件，函数会抛出异常，而不是发生其他意外行为。重要的是，异常是在修改状态前抛出来的，所以不会出现有些地方修改了，而其地方没修改的情况。这种情况是危险且难以处理的。多亏了断言式检查，使得错误不能够忽略，程序变得更加稳定。
- 代码在某种程度上是自检的。当代码中具备检查条件的时候，依赖单元测试的次数就少了。
- 上面所有的检查都依赖于只能转换。所以多亏了智能转换，不需要过多的转换要求。

让我们来讨论下各种不同的检查，以及为什么需要它们。从最常见的一个开始：参数检查

**参数**

当你定义一个带参数的函数的时候，很少看到对参数进行一些限定，因为现有类型系统无法来表现出来。我们来看看几个例子：

- 当你计算一个数的阶乘的时候，应该要求这个数是正整数。
- 当你查找集群时，你可能需要点列表不为空
- 当要给一个用户发送邮件的时候，应该要求这个用户有邮箱，并且邮箱地址的格式是正确的（假设用户在使用函数之前应该检查邮箱地址是否正确）
- Kotlin中来声明这些要求条件最常见和最直接的方式就是用 require函数，require函数检查参数是否满足条件，如果不满足就会抛出异常。
```kotlin
fun factorial(n: Int): Long {
    require(n >= 0)
    return if (n <= 1) 1 else factorial(n - 1) * n
}

fun findClusters(points: List<Point>): List<Cluster> {
    require(points.isNotEmpty())
    //...
}

fun sendEmail(user: User, message: String) {
    requireNotNull(user.email)
    require(isValidEmail(user.email))
    //...
}
```
注意这些需求很明显，因为它们被定义在了函数的最开头了。这使得阅读这些函数的人很清楚的注意到了它们(因为不是每个人都会阅读方法体，这些需求需要在文档中描述)
这些异常无法被忽略，因为当断言无法满足的时候，require函数会抛出IllegalArgumentException。当reqire函数被放置在函数开头的时，我们知道如果一个参数不正确，函数会立即停止。当发现会导致失败的奇怪数值的时候，异常将被清晰的抛出，而我们必须处理这个异常，不能忽略。从另一方面来说，我们应该在函数开头合适对参数要求进行限定，然后可以假设这些要求会被满足。
也可以在lamda表达式里为异常指定一个条消息：
```kotlin
fun factorial(n: Int): Long {
    require(n >= 0) { "Cannot calculate factorial of $n because it is smaller than 0" }
    return if (n <= 1) 1 else factorial(n - 1) * n
}
```
require函数用来对函数参数进行条件限定。
另一种常见情况是我们需要检查当前的状态值是否符合预期，这时我们可以使用check函数。

**状态State**

只允许在特定的条件下使用一些函数，这种情况并不少见。来看几个例子：

-  一些函数需要一个被初始化的对象
- 必须在用户登录后，才能执行的一些操作
- 函数可能需要打开一个对象？？Functions might require an object to be open

检查状态是否满足预期的值的标准方式就是使用check 函数：
```kotlin
fun speak(text: String) {
    check(isInitialized)
//...
}
fun getUserInfo(): UserInfo {
    checkNotNull(token)
    //...
}

fun next(): T {
    check(isOpen)
//...
}
```
check函数的作用和require函数相似，但是不符合预期的值的时候它抛出的是IllegalStateException。它用来检查状态是否正确。可以像require一样来自定义异常消息。一般将check放在函数开头。尽管一些状态检查是本地的需求，但是后面依然可以使用check。
check函数特别用于当我们怀疑开发者不按照规定在错误的时机进行函数调用的时候。与其相信开发者会遵守规则，不如检查状态且抛出合适的异常。当我们不相信自己是否正确的实现状态处理的时候，应该使用check函数。尽管如此，大多数的检查主要是用来测试自己的实现，这个时候可以使用assert函数。

**断言Assertions**

我们可以通过一些条件为true来判断一个函数实现的是否正确。例如当一个函数要求返回10个元素，我们可能期望它返回10个元素。我们期望某些条件是true，但并不意味着我们总是对的，我们自己也会犯错。我们可能会在实现逻辑的时候犯错，也可能有人修改了我们使用的函数，导致我们自己的函数不能再正常运行。也许因为函数重构导致我们的函数不能正确的工作。这些问题最常见的解决方式就是编写单元测试来检查我们的期望和实际是否相符：
```kotlin
class StackTest {
    @Test
    fun `Stack pops correct number of elements`() {
        val stack = Stack(20) { it }
        val ret = stack.pop(10) 7 assertEquals(10, ret.size)
    }
    //...
}
```
单元测试应该是我们检查实现正确性的最基本方法，但请注意，弹出列表大小与所需大小匹配这一事实对该函数相当普遍。 在几乎每个 pop 调用中添加这样的检查会很有用。 对这种单一用途只进行一次检查是相当幼稚的，因为可能存在一些边缘情况。 更好的解决方案是在函数中包含断言：

```kotlin
fun pop(num: Int = 1): List<T> {
    //...
    assert(ret.size == num)
    return ret 
}
```
这些条件目前仅在 Kotlin/JVM 中启用，除非使用 -ea JVM 选项启用，否则不会检查它们。我们宁愿将它们视为单元测试的一部分 - 它们检查我们的代码是否按预期工作。 默认情况下，它们不会在生产中引发任何错误。 它们仅在我们运行测试时默认启用。 这通常是需要的，因为如果我们犯了错误，我们可能宁愿希望用户不会注意到。 如果这是一个可能的严重错误并且可能产生重大后果，请改用检查。 在函数中而不是在单元测试中进行断言检查的主要优点是：

-  断言使代码自检并导致更有效的测试。
-  检查每个实际用例而不是具体案例的期望。
-  我们可以使用它们在确切的执行点检查某些内容。
- 我们使代码尽早失败，更接近实际问题。 多亏了这一点，我们还可以轻松找到意外行为开始的地点和时间。

请记住，要使用它们，我们仍然需要编写单元测试。 在标准应用程序运行中，assert 不会抛出任何异常。
这样的断言是 Python 中的一种常见做法。 在Java中没有那么多。 在 Kotlin 中随意使用它们来使你的代码更可靠。
 
**可空性和智能转换**

require 和 check 都有 Kotlin 契约，声明当函数返回时，它的谓词在此检查后为真。
```kotlin
public inline fun require(value: Boolean): Unit {
    contract {
        returns() implies value
    } 
    require (value) { "Failed requirement." }
}
```
在这些块中检查的所有内容都将在稍后在同一函数中被视为 true。 这适用于智能转换，因为一旦我们检查某事是否为真，编译器就会如此对待它。 在下面的例子中，我们要求一个人的着装是一件连衣裙。之后，假设着装属性是最终的，它将被智能投射到连衣裙。
```kotlin
fun changeDress(person: Person) {
    require(person.outfit is Dress)
    val dress: Dress = person.outfit
    //...
}
```
当我们检查某些东西是否为空时，这个特性特别有用：
```kotlin
class Person(val email: String?)

fun sendEmail(person: Person, message: String) {
    require(person.email != null)
    val email: String = person.email
    //...
}
```
对于这种情况，我们甚至有特殊的函数：requireNotNull 和 checkNotNull。 它们都具有智能转换变量的能力，并且它们也可以用作表达式来“解包”它：
```kotlin
class Person(val email: String?)

fun validateEmail(email: String) { /*...*/
}

fun sendEmail(person: Person, text: String) {
    val email = requireNotNull(person.email)
    validateEmail(email)
    //...
}

fun sendEmail(person: Person, text: String) {
    requireNotNull(person.email)
    validateEmail(person.email)
    //...
}
```
对于可空性，在右侧使用带有 throw 或 return 的 Elvis 运算符也很流行。 这样的结构也具有很高的可读性，同时，它让我们在决定我们想要实现的行为方面有更大的灵活性。 首先，我们可以很容易地使用 return 来停止一个函数，而不是抛出一个错误：
```kotlin
fun sendEmail(person: Person, text: String) {
    val email: String = person.email ?: return
    //...
}
```
如果我们需要在一个属性错误地为空的情况下执行多个操作，我们总是可以通过将 return 或 throw 包装到 run 函数中来添加它们。 如果我们需要记录函数停止的原因，这样的功能可能会很有用：
```kotlin
fun sendEmail(person: Person, text: String) {
    val email: String = person.email ?: run {
        log("Email not sent, no email address") 
        return
    }
    //...
}
```
带有 return 或 throw 的 Elvis 运算符是一种流行且惯用的方式来指定在变量可空性的情况下应该发生什么，我们应该毫不犹豫地使用它。 同样，如果可能，请在函数的开头保留此类检查，以使其可见和清晰。

**总结**

指定你的期望：

- 让它们更显眼。
- 保护你的应用程序稳定性。
- 保护你的代码正确性。
- 智能转换变量。

我们为此使用的四种主要机制是：

-  require block——一种指定参数期望的通用方法。
-  check block - 一种指定状态期望的通用方法。
-  assert block- 如果某事为真，则在测试模式下进行测试的通用方法。
- 带有返回或抛出的Elvis 运算符。

你也可以使用 throw 来抛出不同的错误。
