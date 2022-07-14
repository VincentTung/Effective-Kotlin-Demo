11：可读性设计

编程中的一个常见现象就是阅读代码的时间比写代码多很多。写一分钟的代码，十分钟来读懂。如果看起来不可思议，想想你查找错误的时候，花费了多少时间在弄懂代码上了。相信每个人都至少有一次这样的经历，在花费数周查找一个错误，最后的解决方法仅仅是修改了一行代码。当我们需要学习新的API的时候，经常先从阅读代码开始。我们阅读代码通常是为了弄懂逻辑或者实现的过程。编程大部分时候是阅读，而不是写。知道这点，拥有编写易懂代码的思维是显而易见。

**减少认知负荷**

可读性每个人的理解不一样。然而，也有一些规则基于过往经验或者来自认知科学。只要对比一下下面两个实现即可：
```kotlin
// Implementation A 
if (person != null && person.isAdult) {
    view.showPerson(person)
} else {
    view.showError()
} 

// Implementation B
person?.takeIf { it.isAdult }?.let(view::showPerson)?: view.showError()
```
哪个更好，A或B?如果按照单纯的理由来看，代码行数最少的更好，但是这并不是正确答案。我们可以在第一个实现中将代码精简，但是它没有变的更易懂。
这两种结构的可读性如何，取决于我们能够多块理解他们。反过来说，这取决于我们的大脑接受了多少训练来理解每种风格(结构，函数，模式）。对一个Kotllin的初学者来说，A可定是更易懂的。它用来常见的风格(if/else,&&,方法调用）.B的实现风格是典型的Koltin风格(空安全调用?.,takeIf,let ,Elvis操作符?:,函数引用view::showPerson).当然，所有这些风格在Kotlin很常见，所以它们都被有经验的Kotlin开发者数值。当然，两者进行比较很困难。Kotlin对于大多数开发者来说并不是第一语言，相比Kotlin编程，我们在一般编程中有更多的经验。我们并不是只为有经验的开发者编写代码。很有可能，你雇佣的那个经验较少的人不知道 什么是let,takeIf和函数引用。也有可能他们从没看过Elvis操作符这样使用。这些人可能会花费一整天来弄懂这小段代码。此外，即使是有经验的Kotlin开发者，Kotlin也不是他们唯一使用的编程语言。很多阅读你的代码的开发者并不是非常熟悉Kotlin.大脑总是需要花费一点点时间来识别Kotlin的特定风格(格式)。甚至在使用了Kotlin几年后，仍然需要花费少许时间来弄懂第二个实现。每一个不太为人所熟知的格式都伴随着复杂性，且当我们在一行代码里分析所有的含义时，这种复杂性会迅速增长。
请注意，实现A更加容易修改。假如我们需要在if里面添加额外的操作。在实现A中添加是容易的。在实现B中就不能再使用函数引用了。在 实现B中向else里面添加操作也是困难的-需要使用一些函数能够在Elvis操作符右侧持有超过1个表达式 
```kotlin
//Implementation A
if (person != null && person.isAdult) {
    view.showPerson(person)
    view.hideProgressWithSuccess()
} else {
    view.showError()
    view.hideProgress()
}

//Implementation B
person?.takeIf { it.isAdult }
?.let {
    view.showPerson(it)
    view.hideProgressWithSuccess()
} ?: run {
    view.showError()
    view.hideProgress()
}
```
调试实现A也是更加简单的。无需考虑为什么，因为调试工具就是为这种结构设计的。
一般来讲不太常见或"有创造性"的结构通常是不太灵活和适配性差的。假设，举个例子，我们需要添加一个第三分支来person为空的错误或者是未成年的错误。在实现A中由于使用了if/else，我们可以使用Inellij的重构功能轻松的改变if/else,然后轻松的添加额外的分支。在实现B则是痛苦的。甚至可能需要全部重写。
你注意到了吗,实现A和B是指不以同样的方式运行。你能注意到区别吗？回去看看，现在仔细考虑下。
区别是实际上let从lamda表达式返回数据撒谎了。意思是如果showPerson函数能返回null，也能同样能调用showError函数。这一点显然不明显，它教会我们，但我们使用不常见的格式代码，更容易陷入未知的情况（并且不容易发现它们）。
这里通常的做法就是减少认知负荷。我们大脑识别各种模式以及这些模式构建我们对程序是如何工作的认知。当考虑可读性的时候，我们想要缩短这个过程。我们更喜欢于更少的代码，但是也要是常见的代码结构。我们识别熟悉的代码结构当我们经常遇见。我们也总是更喜欢在其他学科所熟悉的结构。

**不要变得极端**

因为刚刚在上述例子中我展示了let是怎样被滥用的，但并不意味着let总要避免使用。let是一种流程的风格用在各种样式的上下文中使用，也确实优化了代码。一个常见的例子就是当我们需要在不为空的时候做操作。智能转换不能使用因为易变属性能被其他线程所修改。处理这种情况的一个好办法就是使用安全调用 let:
```kotlin

class Person(val name: String)

var person: Person? = null
fun printName() {
    person?.let {
        print(it.name)
    }
}
```
这种风格是流行的并且被广泛认知的。还有很多使用let的合理的理由。例如：

- 移动操作到参数的计算之后
- 用它来当做装饰器来包裹一个对象

下面举个例子来展示上面两点（都额外使用了函数引用）
```kotlin
    students.filter
    { it.result >= 50 } .joinToString(separator = "\n")
    {
        "${it.name} ${it.surname}, ${it.result}"
    }.let(::print)
    
    var obj = FileInputStream("/file.gz").let(::BufferedInputStream)
        .let(::ZipInputStream)
        .let(::ObjectInputStream)
        .readObject() as SomeObject
```
在所有的类似情况里我们付出了自己的代价- 这种代码调试起来是困难的，没有经验的kotlin开发者理解起来也很困难。但是我们因此收获了一些，看起来是公平的。问题是当我们并不是出于好的理由来引入了很多的复杂度。
一样东西总有评论它是否有用。如何平衡是一门艺术。好的做法是 通过分辨不同的结构引入的复杂度或者使逻辑更易懂的。特别的是，在当它们一起使用的时候，两个结构的复杂度远超过它们单独的复杂度。

**约定**

我们承认不同的认对可读性的理解是不同的。我们不断的为函数名争论，讨论什么应该是明确的，什么应该不是，应该使用什么格式等等。编程是一门复函表达力的艺术。尽管如此，还是有一些约定需要遵守理解和记忆。
当有人问我见过的kotlin中最坏的东西，我给他们看了这个：
```kotlin
val abc = "A" { "B" } and "C"
print(abc) // ABC
```
要实现个可怕的表达式，需要如下代码
```kotlin
operator fun String.invoke(f: ()->String): String =
this + f()

infix fun String.and(s: String) = this + s
```
这段代码违反很多规则：

- 违反了操作符的含义-invoke不能这么使用。一个String不能被invoke
- "lamda as the last argument"规则的使用是令人疑惑的。它用在函数后面没问题，但是在和invoke操作符配合的时候应该非常小心。
- kotlin已经有支持String拼接的语言特性，无需重复造轮子

每条建议的背后，都有一个能保证好的Kotlin风格的常见规则。我们将在本章中讨论最重要的内容，从第一项开始，即重载操作符。
