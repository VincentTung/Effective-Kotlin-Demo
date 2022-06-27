27.使用抽象来保护代码免受更改
 
当我们将实际代码隐藏在诸如函数或类之类的抽象后面时，我们不仅可以保护用户免受这些细节的影响，而且还可以让自己在以后自由地更改这些代码。 通常甚至没有用户知道它。 例如，当你将排序算法提取到函数中时，可以稍后优化其性能而无需更改其使用方式。
想想前面提到的汽车比喻，汽车制造商和机械师可以改变汽车引擎盖下的一切，只要操作保持不变，用户就不会注意到。 这为制造商提供了制造更环保汽车或添加更多传感器以使汽车更安全的自由。
在本项目中，我们将看到不同类型的抽象如何通过保护我们免受各种变化的影响而给予我们自由。 我们将研究三个实际案例，最后讨论在许多抽象中找到平衡点。 让我们从最简单的抽象类型开始：常量值。

**常量**

字面常量值很少是不言自明的，当它们在我们的代码中重复时尤其成问题。 将值移动到常量属性中不仅可以为该值分配一个有意义的名称，还可以帮助我们更好地管理何时需要更改该常量。 让我们看一个密码验证的简单示例：
```kotlin
fun isPasswordValid(text: String): Boolean {
    if(text.length < 7) return false
    //...
}
```
数字 7 可以根据上下文来理解，但如果将其提取为常量会更容易：
```kotlin
const val MIN_PASSWORD_LENGTH = 7

fun isPasswordValid(text: String): Boolean {
    if(text.length < MIN_PASSWORD_LENGTH) return false
    //...
}
```
这样，修改最小密码大小就更容易了。 我们不需要理解验证逻辑，相反，我们可以改变这个常量。 这就是为什么提取多次使用的值特别重要的原因。 例如，可以同时连接到我们的数据库的最大线程数：
```kotlin
val MAX_THREADS = 10
```
提取后，您可以在需要时轻松更改它。 只是
想象一下，如果这个数字遍布整个项目，改变它会有多难。
如你所见，提取常量：

- 命名它
-  帮助我们在未来改变值

对于不同类型的抽象，我们也会看到类似的结果。

**函数**

想象一下，你正在开发一个应用程序，并且你注意到你经常需要向用户显示一条 toast 消息。 这是以编程方式执行此操作的方式：
```kotlin
Toast.makeText(this, message, Toast.LENGTH_LONG).show()
```

![image.png](https://cdn.nlark.com/yuque/0/2022/png/10378482/1656289874597-cf7828aa-eb6c-48c8-b60f-0c9c98d075ea.png#clientId=ufaee83c9-8859-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=548&id=u8d264948&margin=%5Bobject%20Object%5D&name=image.png&originHeight=548&originWidth=781&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11790&status=done&style=none&taskId=u558bc906-7e94-4cb5-b7d4-502f7520065&title=&width=781)


我们可以将这个常用算法提取成一个简单的扩展函数来显示 toast：
```kotlin
fun Context.toast(
    message: String,
    duration: Int = Toast.LENGTH_LONG
) {
    Toast.makeText(this, message, duration).show()
}

// Usage
context.toast(message)

// Usage in Activity or subclasses of Context
toast(message)
```
这种变化帮助我们提取了一个通用算法，这样我们就不需要每次都记住如何显示toast。 如果显示toast的方式通常是变化的（不太可能），这也会有所帮助。 尽管有些变化我们还没有准备好。
如果我们必须将向用户显示消息的方式从 toasts 更改为snackbars（一种不同类型的消息显示）怎么办？ 一个简单的答案是，提取此功能后，我们只需更改此函数内部的实现并重命名即可。

```kotlin
fun Context.snackbar(
    message: String,
    length: Int = Toast.LENGTH_LONG
) {
    //...
}
```
![image.png](https://cdn.nlark.com/yuque/0/2022/png/10378482/1656290894697-76fce173-1116-472d-9504-376432367a73.png#clientId=ufaee83c9-8859-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=519&id=u21088014&margin=%5Bobject%20Object%5D&name=image.png&originHeight=519&originWidth=585&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11358&status=done&style=none&taskId=ucde0bc60-ed3c-4b50-8745-de9ac5ccc60&title=&width=585)

这个解决方案远非完美。首先，即使只在内部使用，重命名函数也可能很危险。特别是如果其他模块依赖于这个函数。下一个问题是参数不能那么容易地自动更改，因此我们仍然坚持使用 toast API 来声明消息持续时间。这是非常有问题的。当我们显示一个snackbar 
时，我们不应该依赖Toast 中的一个字段。另一方面，将所有用法更改为使用 Snackbar 的枚举也会有问题：

```kotlin
fun Context.snackbar(
    message: String,
    duration: Int = Snackbar.LENGTH_LONG
) {
    //...
}
```

当我们知道消息的显示方式可能会改变时，我们知道真正重要的不是消息的显示方式，而是我们想要向用户显示消息的事实。 我们需要的是一种更抽象的方法来显示消息。 考虑到这一点，程序员可以将 toast 显示隐藏在更高级别的函数 showMessage 后面，这将独立于 toast 的概念：

```kotlin
fun Context.showMessage(
    message: String,
    duration: MessageLength = MessageLength.LONG
) {
    val toastDuration = when(duration) {
        SHORT -> Toast.LENGTH_SHORT
        LONG -> Toast.LENGTH_LONG
    }
    Toast.makeText(this, message, toastDuration).show()
}

enum class MessageLength { SHORT, LONG }
```

这里最大的变化是名称。 一些开发人员可能会忽略此更改的重要性，说名称只是一个标签，并不重要。 从编译器的角度来看，这种观点是有效的，但从开发人员的角度来看是无效的。一个函数代表一个抽象，这个函数的签名告诉我们这是什么抽象。 一个有意义的名字非常重要。
函数是一个非常简单的抽象，但也非常有限。 函数不保持状态。 函数签名的更改通常会影响所有用法。 一种更强大的抽象实现的方法是使用类。

**类**

以下是我们如何将消息显示抽象到一个类中：
```kotlin
class MessageDisplay(val context: Context) {
    fun show(
        message: String,
        duration: MessageLength = MessageLength.LONG
) {
        val toastDuration = when(duration) {
            SHORT -> Toast.LENGTH_SHORT
            LONG -> Toast.LENGTH_LONG
        }
        Toast.makeText(context, message, toastDuration)
            .show()
    }
}

enum class MessageLength { SHORT, LONG }

// Usage
val messageDisplay = MessageDisplay(context)
messageDisplay.show("Message")
```

类比函数更强大的关键原因在于它们可以保持状态并公开许多函数（类成员函数称为方法）。 在这种情况下，我们在类状态中有一个上下文，它是通过构造函数注入的。 使用依赖注入框架，我们可以委托类创建：
```kotlin
@Inject lateinit var messageDisplay: MessageDisplay
```
此外，我们可以模拟该类来测试依赖于指定类的其他类的功能。 这是可能的，因为我们可以模拟类用于测试目的：
```kotlin
val messageDisplay: MessageDisplay = mock()
```
此外，可以添加更多方法来设置消息显示
```kotlin
messageDisplay.setChristmasMode(true)
```
正如你所看到的，类给了我们更多的自由。 但它们仍然有其局限性。 例如，当一个类是 final 时，我们知道它的类型下的确切实现。 我们对开放类有更多的自由，因为我们可以为子类服务。 不过，这种抽象仍然强烈地绑定到这个类。 为了获得更多自由，我们可以使其更加抽象并将此类隐藏在接口后面

**接口**

阅读 Kotlin 标准库，你可能会注意到几乎所有内容都表示为接口。 看看几个例子：

-  listOf 函数返回List，它是一个接口。 这类似于其他工厂方法（我们将在条款 33，考虑工厂方法而不是构造函数中解释它们）。
-  Collection处理函数是Iterable或Collection上的扩展函数，返回List、Map等，都是接口。
-  属性委托隐藏在同样是接口的 ReadOnlyProperty 或 ReadWriteProperty 后面。 实际的类通常是private的。 函数lazy 也将接口Lazy 声明为其返回类型。

库创建者通常会限制内部类的可见性并从接口后面公开它们，这是有充分理由的。这样，库创建者可以确保用户不会直接使用这些类，因此只要接口保持不变，他们就可以毫无顾虑地更改其实现。这正是这个项目背后的想法——通过将对象隐藏在接口后面，我们抽象出任何实际的实现，我们迫使用户只依赖于这个抽象。这样我们减少了耦合。
在 Kotlin 中，返回接口而不是类还有另一个原因 - Kotlin 是一种多平台语言，相同的 listOf 为 Kotlin/JVM、Kotlin/JS 和 Kotlin/Native 返回不同的列表实现。这是一种优化——Kotlin 通常使用特定于平台的原生集合。这很好，因为它们都尊重 List 接口。
让我们看看如何将这个想法应用到我们的消息显示中。这就是我们将类隐藏在接口后面时的样子：

```kotlin
interface MessageDisplay {
    fun show(
        message: String,
        duration: MessageLength = LONG
) 
}

class ToastDisplay(val context: Context): MessageDisplay {

    override fun show(
        message: String,
        duration: MessageLength
    ) {
        val toastDuration = when(duration) {
            SHORT -> Toast.LENGTH_SHORT
            LONG -> Toast.LENGTH_LONG
        }
        Toast.makeText(context, message, toastDuration)
            .show()
    }
}

enum class MessageLength { SHORT, LONG }
```
作为回报，我们得到了更多的自由。 例如，我们可以注入在平板电脑上显示toast和在手机上显示snackbar的类。 也可以在 Android、iOS 和 Web 之间共享的公共模块中使用 MessageDisplay。 然后我们可以为每个平台提供不同的实现。 例如，在 iOS 和 Web 中，它可以显示警告。
另一个好处是用于测试的接口伪造比类模拟更简单，并且不需要任何模拟库：
```kotlin
val messageDisplay: MessageDisplay = TestMessageDisplay()
```
最后，声明与使用更加分离，因此我们可以更自由地更改实际类，如 ToastDisplay。 另一方面，如果我们想改变它的使用方式，我们就需要改变 MessageDisplay 接口和所有实现它的类。
**下一个ID**
让我们再讨论一个例子。 假设我们在项目中需要一个唯一的 ID。 一个非常简单的方法是拥有一个顶级属性来保存下一个 ID，并在我们需要新 ID 时递增它：
```kotlin
var nextId: Int = 0 

// Usage

val newId = nextId++
```
看到这种用法在我们的代码中蔓延应该会引起一些警告。
如果我们想改变创建 ID 的方式怎么办。 老实说，这种方式远非完美：

-  每当我们冷启动程序时，我们都会从 0 开始。
-  它不是线程安全的。

假设现在我们接受这个解决方案，我们应该通过将 ID 创建提取到一个函数中来保护自己免受更改：

```kotlin
1 private var nextId: Int = 0 
fun getNextId(): Int = nextId++

// Usage
val newId = getNextId()
```
请注意，此解决方案仅保护我们免受 ID 创建更改的影响。 我们仍然容易发生许多变化。 最大的一个是ID类型的变化。 如果有一天我们需要将 ID 保存为 String 怎么办？ 另请注意，看到 ID 表示为 Int 的人可能会使用一些类型相关的操作。 例如，使用比较来检查哪个 ID 较旧。 这样的假设可能会导致严重的问题。 为了防止这种情况发生并让我们以后轻松更改 ID 类型，我们可以将 ID 提取为一个类：
```kotlin
data class Id(private val id: Int)

private var nextId: Int = 0
fun getNextId(): Id = Id(nextId++)
```
再一次，很明显，更多的抽象给了我们更多的自由，但也使定义和用法更难定义和理解

**抽象带来自由**

我们提出了一些引入抽象的常用方法：

-  提取常数
- 将行为包装到函数中
-  将函数包装到一个类中
-  在接口后面隐藏一个类
-  将通用对象包装成专业对象

我们已经看到每种方法如何给我们不同的自由。 请注意，还有更多可用的工具。 仅举几个：

- 使用泛型类型参数
-  提取内部类
- 限制创建，例如通过工厂方法强制创建对象⁴

另一方面，抽象也有其阴暗面。 它们给了我们自由和拆分代码，但它们通常会使代码更难理解和修改。 让我们谈谈抽象的问题。

**抽象的问题**

添加新的抽象需要代码的读者学习或已经熟悉特定的概念。当我们定义另一个抽象时，这是我们项目中需要理解的另一件事。当然，当我们限制抽象可见性（第 30 条：最小化元素可见性）或定义仅用于具体任务的抽象时，问题就不那么严重了。这就是模块化在大型项目中如此重要的原因。我们需要了解定义抽象是有这个成本的，我们不应该默认抽象一切。
我们可以无限地提取抽象，但很快这将弊大于利。这个事实在 FizzBuzz 企业版项目⁴⁴ 中被戏仿，作者表明，即使对于像 Fizz Buzz⁴⁵ 这样简单的问题，人们也可以提取大量可笑的抽象，使这个问题非常难以使用和理解。在编写本书时，共有 61 个类和 26 个接口。所有这些都是为了解决一个通常需要不到 10 行代码的问题。当然，在任何级别应用更改都很容易，但另一方面，了解这段代码的作用以及它是如何做到的却非常困难
![image.png](https://cdn.nlark.com/yuque/0/2022/png/10378482/1656290809700-64985869-ddca-45fd-8aea-cd580f2e5c9e.png#clientId=ufaee83c9-8859-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=705&id=u3a4399e4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=705&originWidth=679&originalType=binary&ratio=1&rotation=0&showTitle=false&size=134803&status=done&style=none&taskId=uffc208c1-52a9-4bb5-8fce-eeea99dc04f&title=&width=679)

**FizzBuzz 企业版本类结构的一部分。 在这个项目的描述中，你可以找到讽刺的理由“这个项目是一个例子，说明如果流行的 FizzBuzz 游戏受到企业软件的高质量标准的约束，它是如何构建的。”**

另一方面，当我们需要考虑的事情较少时，当我们使用太多抽象时，就很难理解我们行为的后果。 有人可能会使用 showMessage 函数，认为它仍然显示 toast，当它显示一个snackbar时，我们可能会感到惊讶。 看到显示了意外的 toast 消息的人可能会查找 Toast.makeText 并在查找它时遇到问题，因为它是使用 showMessage 显示的。 太多的抽象使我们的代码更难理解。 当我们不确定自己行为的后果是什么时，它也会让我们感到焦虑。
为了理解抽象，示例非常有帮助。 文档中的单元测试或示例展示了如何使用元素，使抽象对我们来说更加真实。 出于同样的原因，我在本书中为我提出的大多数想法提供了具体的例子。 很难理解抽象的描述。 也很容易误解他们。

**如何平衡**

经验法则是：每个级别的复杂性都给了我们更多的自由和组织我们的代码，但也让我们更难理解我们项目中真正发生的事情。两个极端都不好。最好的解决方案总是介于两者之间，它到底在哪里，这取决于许多因素，例如：

-  团队规模
-  团队经验
- 项目规模
-  功能集
-  领域知识

我们不断在每个项目中寻求平衡。找到适当的平衡几乎是一门艺术，因为它需要在数百甚至数千小时的架构和编码项目中获得直觉。
以下是我可以给出的一些建议：

-  在拥有更多开发人员的大型项目中，以后更改对象的创建和使用要困难得多，因此我们更喜欢更抽象的解决方案。此外，模块或部分之间的分离在那时特别有用。
-  当我们使用依赖注入框架时，我们不太关心创建的难度，因为无论如何我们可能只需要定义一次创建。
- 测试或制作不同的应用程序变体可能需要我们使用一些抽象。
-  当你的项目较小且处于试验阶段时，可以享受直接进行更改的自由，而无需处理抽象。虽然当它变得严重时，尽快改变它。

我们需要不断思考的另一件事是可能会发生什么变化以及每次变化的几率是多少。例如，toast 显示的 API 发生更改的可能性很小，但我们需要更改显示消息的方式的合理可能性。我们可能需要模拟这种机制吗？有一天你会需要一种更通用的机制吗？还是一种可能独立于平台的机制？这些概率不是0，那么它们有多大呢？观察这些年来事物的变化给了我们越来越好的直觉。

**总结**

抽象不仅是为了消除冗余和组织我们的代码。 当我们需要更改代码时，它们也会帮助我们。 尽管使用抽象更难。 它们是我们需要学习和理解的东西。 当我们使用抽象结构时，也更难理解后果。 我们需要了解使用抽象的重要性和风险，我们需要在每个项目中寻找平衡点。 抽象太多或太少都不是理想的情况。
