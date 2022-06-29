31.定义文档的契约

再想一想显示第 27 条中用来提示消息的函数：使用抽象来保护代码免受更改：

```kotlin
fun Context.showMessage(
    message: String,
    length: MessageLength = MessageLength.LONG
) {
    val toastLength = when(length) {
        SHORT -> Toast.LENGTH_SHORT
        LONG -> Toast.LENGTH_LONG
    } 
    Toast.makeText(this, message, toastLength).show()

}

enum class MessageLength { SHORT, LONG }
```

我们提取它是为了让自己可以自由地更改消息的显示方式。 但是，它没有很好的记录。另一个开发人员可能会阅读它的代码并假设它总是显示一个 toast。 这与我们想要通过命名实现的相反 ，它以某种方式不建议具体的消息类型。 为了清楚起见，最好添加一个有意义的 KDoc 注释来解释该函数的预期内容。
```kotlin
/**
 * Universal way for the project to display a short
 * message to a user.
 * @param message The text that should be shown to
 * the user
 * @param length How long to display the message.
 */
fun Context.showMessage(
    message: String,
    duration: MessageLength = MessageLength.LONG
) {
    val toastDuration = when (duration) {
        SHORT -> Toast.LENGTH_SHORT
        LONG -> Toast.LENGTH_LONG
    }
    Toast.makeText(this, message, toastDuration).show()
}

enum class MessageLength { SHORT, LONG }
```

在许多情况下，有些细节根本无法通过名称清楚地推断出来。 例如，powerset，即使它是一个定义明确的数学概念，也需要解释，因为它不是那么广为人知，解释也不够清楚：

```kotlin
/**
 * Powerset returns a set of all subsets of the receiver
 * including itself and the empty set
 */
fun <T> Collection<T>.powerset(): Set<Set<T>> = 
    if (isEmpty()) setOf(emptySet())
    else take(size - 1)
    .powerset()
    .let { it + it.map { it + last() } }
```

指定这些元素的顺序。 作为用户，我们不应该依赖这些元素的排序方式。 隐藏在这个抽象背后的实现可以在不改变这个函数的外观的情况下进行优化：

```kotlin
/**
 * Powerset returns a set of all subsets of the receiver
 * including itself and empty set
 */
fun <T> Collection<T>.powerset(): Set<Set<T>> =
    powerset(this, setOf(setOf()))

private tailrec fun <T> powerset(
    left: Collection<T>,
    acc: Set<Set<T>>
): Set<Set<T>> = when {
    left.isEmpty() -> acc
    else -> {
        val head = left.first()
        val tail = left.drop(1)
        powerset(tail, acc + acc.map { it + head })
    }
}
```
一般的问题是，当行为没有记录并且元素名称不明确时，开发人员将依赖于当前的实现而不是我们打算创建的抽象。 我们通过描述可以预期的行为来解决这个问题。

**约定**

每当我们描述一些行为时，用户都将其视为一种承诺，并据此调整他们的期望。我们将所有这些预期的行为称为元素的契约。就像在现实生活中的元素一样，另一方希望我们遵守它，在这里，用户也希望我们在契约稳定后保留该元素（第 28 条：指定 API 稳定性）。
在这一点上，定义契约可能听起来很可怕，但实际上，这对双方都很好。当合约被明确指定时，创建者不需要担心类是如何被使用的，用户也不需要担心某些东西是如何在幕后实现的。用户可以在不了解实际实现的情况下依赖此合约。对于创作者来说，只要元素得到满足，契约就可以自由改变一切。用户和创建者都依赖于合约中定义的抽象，因此他们可以独立工作。只要遵守契约，一切都会完美无缺。这对双方来说都是一种安慰和自由。
如果我们不签订契约怎么办？在用户不知道他们能做什么和不能做什么的情况下，他们将依赖于实施细节。不知道用户依赖什么的创建者要么被阻止，要么冒着破坏用户实现的风险。如您所见，指定元素很重要。

**定义一个契约**

我们如何定义契约？ 有多种方法，包括：

-  名称——当名称与某个更一般的概念相关联时，我们希望该元素与该概念一致。 例如，当你看到 sum 方法时，无需阅读其注释即可了解其行为方式。 这是因为求和是一个定义明确的数学概念。
- 注释和文档——最强大的方式，因为它可以描述所需的一切。
- 类型 - 类型说明了很多关于对象的信息。 每种类型都指定了一组通常定义良好的方法，并且某些类型在其文档中还具有设置责任。 当我们看到一个函数时，关于返回类型和参数类型的信息是非常有意义的。

**我们需要注释吗？**

回顾历史，看到社区中的意见如何波动是令人惊讶的。在 Java 还年轻的时候，有一个非常流行的识字编程概念。它建议在评论中解释所有内容。十年后，我们可以听到对评论的强烈注释和强烈的声音，我们应该忽略注释并专注于编写可读的代码（我相信最有影响力的书是 Robert C. Martin 的 Clean Code）。
没有极端是健康的。我绝对同意我们应该首先专注于编写可读的代码。虽然需要理解的是元素（函数或类）之前的注释可以在更高的层次上描述它，并设置它们的契约。此外，注释现在经常用于自动生成文档，这通常被视为项目中的真实来源。当然，我们通常不需要评论。例如，许多功能是不言自明的，不需要任何特殊描述。例如，我们可以假设 product 是程序员知道的一个清晰的数学概念，并且不做任何评论：

```kotlin
fun List<Int>.product() = fold(1) { acc, i -> acc * i }
```

明显的注释只会分散我们的注意力。 不要编写只描述函数名和参数清楚表达的内容的注释。 下面的示例演示了一个不必要的注释，因为可以从方法的名称和参数类型推断出功能：

```kotlin
// Product of all numbers in a list
fun List<Int>.product() = fold(1) { acc, i -> acc * i }
```

我也同意，当只需要组织我们的代码，而不是在实现中的注释时，我们应该提取一个函数。 看看下面的例子：

```kotlin
fun update() {
    // Update users
    for (user in users) {
        user.update()
    } 
    
    // Update books
    for (book in books) {
        updateBook(book)
    }
}
```
函数更新显然由可提取的部分组成，并且注释表明可以用不同的解释来描述这些部分。 因此，最好将这些部分提取到单独的抽象中，例如实例方法，并且它们的名称足以清楚地解释它们的含义（就像第 26 条中的那样：每个函数都应该按照单个抽象级别编写）。

```kotlin
    fun update() {
        updateUsers()
        updateBooks()
    }

    private fun updateBooks() {
        for (book in books) {
            updateBook(book)
        }
    }

    private fun updateUsers() {
        for (user in users) {
            user.update()
        }
    }
```

尽管注释通常有用且重要。 要查找示例，请查看 Kotlin 标准库中的几乎所有公共函数。 他们有明确的契约，提供了很大的自由。 例如，看一下函数 listOf：

```kotlin
/**
 * Returns a new read-only list of given elements.
 * The returned list is serializable (JVM).
 * @sample samples.collections.Collections.Lists
.readOnlyList
 */
public fun <T> listOf(vararg elements: T): List<T> =
    if (elements.size > 0) elements.asList()
    else emptyList()
```

它所承诺的只是它返回在 JVM 上只读且可序列化的 List。 没有其他的。 该列表不需要是不可变的。没有承诺具体的类。 这个契约是简约的，但满足大多数 Kotlin 开发人员的需求。 还可以看到它指向示例用途，这在我们学习如何使用元素时也很有用。

**KDOc格式**

当我们使用注释记录函数时，我们呈现该注释的官方格式称为 KDoc。 所有 KDoc 注释都以 /** 开头并以 */ 结尾，内部所有行通常以 * 开头。 那里的描述是用 KDoc markdown 写的。
此 KDoc 注释的结构如下：

- •文档文本的第一段是元素的摘要描述。
- 第二部分是详细说明。
-  每下一行都以一个标签开始。 这些标签用于引用一个元素来描述它。

以下是支持的标签：

- @param <name> - 记录函数的值参数或类、属性或函数的类型参数。
- @return - 记录函数的返回值。
- @constructor - 记录类的主要构造函数。
- @receiver - 记录扩展函数的接收者。
- @property <name> - 记录具有指定名称的类的属性。用于在主构造函数上定义的属性。
- @throws <class>、@exception <class> - 记录可由方法抛出的异常。
- @sample <identifier> - 将具有指定限定名称的函数体嵌入到当前元素的文档中，以展示如何使用该元素的示例。
- @see <identifier> - 添加到指定类或方法的链接
- @author - 指定被记录元素的作者。
- @since - 指定引入文档元素的软件版本。
- @suppress - 从生成的文档中排除元素。可用于不属于模块官方 API 但仍必须在外部可见的元素.

在描述和描述标签的文本中，我们可以链接类、方法、属性或参数。 当我们想要与链接元素的名称不同的描述时，链接位于方括号或双方括号中:

```kotlin
 /**
 * This is an example descriptions linking to [element1],
 * [com.package.SomeClass.element2] and
 * [this element with custom description][element3]
 */

```

Kotlin 文档生成工具将理解所有这些标签。 官方称为Dokka。 他们生成可以在线发布并呈现给外部用户的文档文件。 以下是带有简短描述的示例文档：

```kotlin
/**
 * Immutable tree data structure.
 * Class represents immutable tree having from 1 to
 * infinitive number of elements. In the tree we hold
 * elements on each node and nodes can have left and
 * right subtrees...
 * @param T the type of elements this tree holds.
 * @property value the value kept in this node of the tree.
 * @property left the left subtree.
 * @property right the right subtree.
 */
class Tree<T>(
    val value: T,
    val left: Tree<T>? = null,
    val right: Tree<T>? = null
) {
    /**
     * Creates a new tree based on the current but with
     * [element] added.
     * @return newly created tree with additional element.
     */
    operator fun plus(element: T): Tree {
        ...
    }
}
```

请注意，并非所有内容都需要描述。 最好的文档是简短的，并且准确地描述了可能不清楚的地方.

**类型系统和期望**

类型层次结构是有关对象的重要信息来源。接口不仅仅是我们承诺实现的方法列表。类和接口也可以有一些期望。如果一个类承诺了一个期望，那么它的所有子类也应该保证这一点。这个原则被称为里氏替换原则，它是面向对象编程中最重要的规则之一。它通常被翻译为“如果 S 是 T 的子类型，则类型 T 的对象可以被类型 S 的对象替换，而不会改变程序的任何所需属性”。一个简单的解释为什么它很重要是每个类都可以用作超类，因此如果它的行为不像我们期望它的超类表现的那样，我们可能会以意想不到的失败告终。在编程中，孩子应该始终满足父母的契约。
该规则的一个重要含义是我们应该为开放函数正确指定契约。例如，回到我们的汽车比喻，我们可以使用以下接口在代码中表示汽车：

```kotlin
interface Car {
    fun setWheelPosition(angle: Float)
    fun setBreakPedal(pressure: Double)
    fun setGasPedal(pressure: Double)
}

class GasolineCar : Car {
    // ...
}

class GasCar : Car {
    // ...
}

class ElectricCar : Car {
    // ...
}
```

这个接口的问题是它留下了很多问题。 setWheelPosition 函数中的角度是什么意思？ 用什么单位来衡量。 如果有人不清楚油门和刹车踏板的作用怎么办？ 使用 Car 类型实例的人需要知道如何使用它们，并且所有品牌在将它们用作 Car 时都应该有相似的行为。 我们可以通过文档解决这些问题：
```kotlin
interface Car {
    /**
     * Changes car direction.
     
     * @param angle Represents position of wheels in
     * radians relatively to car axis. 0 means driving
     * straight, pi/2 means driving maximally right,
     * -pi/2 maximally left.
     * Value needs to be in (-pi/2, pi/2)
     */
    fun setWheelPosition(angle: Float)

    /**
     * Decelerates vehicle speed until 0.
     *
     * @param pressure The percentage of brake pedal use.
     * Number from 0 to 1 where 0 means not using break
     * at all, and 1 means maximal pedal pedal use.
     */
    fun setBreakPedal(pressure: Double)

    /**
     * Accelerates vehicle speed until max speed possible
     * for user.
     *
     * @param pressure The percentage of gas pedal use.
     * Number from 0 to 1 where 0 means not using gas at
     * all, and 1 means maximal gas pedal use.
     */
    fun setGasPedal(pressure: Double)
}
```
现在所有的汽车都设定了一个标准来描述它们应该如何表现。
标准库和流行库中的大多数类都为其子级定义和描述良好的契约和期望。我们也应该为我们的元素定义契约。 这些契约将使那些接口真正有用。它们将使我们能够自由地使用以契约保证的方式实现这些接口的类。

**泄漏的实现**

实施细节总是泄漏。在汽车中，不同类型的发动机的行为略有不同。我们仍然可以驾驶汽车，但我们可以感受到不同。没关系，因为合同中没有说明。
在编程语言中，实现细节也会泄露。例如，使用反射调用函数是可行的，但它比普通函数调用慢得多（除非它被编译器优化）。我们将在有关性能优化的章节中看到更多示例。尽管只要一种语言按照它的承诺工作，一切都很好。我们只需要记住并应用好的做法。
在我们的抽象中，实现也会泄漏，但我们仍然应该尽可能地保护它。我们通过封装来保护它，可以说是“你可以做我允许的事情，仅此而已”。封装的类和函数越多，我们在它们内部的自由度就越大，因为我们不需要考虑如何依赖于我们的实现。

**总结**

当我们定义一个元素，尤其是外部 API 的一部分时，我们应该定义一个契约。 我们通过名称、文档、评论和类型来做到这一点。 契约规定了对这些要素的期望。 它还可以描述应该如何使用元素。
契约让用户对元素现在和未来的行为方式充满信心，并且让创作者可以自由更改合约中未指定的内容。 契约是一种协议，只要双方都尊重，它就行得通。
