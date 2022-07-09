第8条：恰当的处理空值

null意味着缺少值。对于属性来说，它可能意味着值没有被设置或者被移除了。当一个函数返null的时候，根据这个函数可能有不同的意思:

- String.toIntOrNull() 当字符串不能被正确的解析成Int的时候返回null
- Iterable<T>.firstOrNull(() ->Boolean) 当没有元素符合表达式的条件的时候返回null

 **在这些和所有其它情况中，null的含义应该是尽可能清晰的**。这是因为可空值必须被处理，API的用户需要决定如何使用它。
```kotlin
val printer: Printer? = getPrinter()
printer.print() // Compilation Error

printer?.print() // Safe call
if (printer != null) printer.print() // Smart casting
printer!!.print() // Not-null assertion
```
一般来说，有3种处理可空值的方式：

- 处理控制使用空安全调用?. 、智能转换、Elvis操作符等等
- 抛出一个错误
- 重构函数或者属性，使它不为空

让我们一个个来讨论下：

**安全的处理空值**

像之前提到的，处理空值最安全和最流行的做法就是使用空安全调用和智能转换：
```kotlin
printer?.print() // 空安全调用
if (printer != null) printer.print() // 智能转换
```
在上面两种情况中，print函数只有在printer不为空的时候才会调用。从应用用户角度来说这是最安全的方式。对于开发者也很方便。那怪这就是为什么我们处理空值的最流行的方式。
Kotlin比其他语言咋处理空值上提供更广泛的支持。一个流行的实践就是使用Elvis操作符，对于可空类型提供一个默认值。它允许任意的表达式，包括return和throw在它的右侧:
```kotlin
val printerName1 = printer?.name ?: "Unnamed"
val printerName2 = printer?.name ?: return
val printerName3 = printer?.name ?:
    throw Error("Printer must be named")
```
许多对象需要额外的处理。例如，常见的要求使用空的集合代替null,在Collection<T>？中有一个扩展函数orEmpty，它返回不为空的List<T>.也有相似的函数String?.
智能转换也是Kotlin的一个特性，下面讨论函数中的智能转换：
```kotlin
println("What is your name?")
val name = readLine()
if (!name.isNullOrBlank()) {
    println("Hello ${name.toUpperCase()}") 
}

val news: List<News>? = getNews()
if (!news.isNullOrEmpty()) {
    news.forEach { notifyUser(it) }
}
```
所有这些操作都应该被Kotlin开发者知道，它们提供了处理空值的有用方式。

**抛出错误**

安全处理的一个问题就是如果printer有时是null的时候，我们并不会被通知，但是print不会调用。这样我们可能隐藏了重要信息。如果我们期望printer绝对不是null,当print函数没有调用的时候我们会很吃惊。这将导致相当难于发现的错误。当我们特别关心不常见的情况的时候，最好抛出错误来提醒开发者出现了意料之外的情况。使用throw，和使用 ！！，requireNotNull,checkNotNull或者错误抛出函数一样：
```kotlin
fun process(user: User) {
    
    requireNotNull(user.name)
    
    val context = checkNotNull(context)
    val networkService =
    getNetworkService(context) ?:
    throw NoInternetConnection()
    
    networkService.getData { data, userData ->
        show(data!!, userData!!)
    }
}
```


**不为空断言！！的问题**

处理可空值的最简单的方式就是使用不为空断言！！。它在概念上和Java中的一些事情相似-当我们错误的认为有些东西不为空的时候，就会抛出NPE.**不为空断言！！是一种懒操作。抛出了一个通用的异常，并没有解释什么。同时简短，容易被滥用、错用**。不为空断言！！经常被用在可空类型被期望不是空的时候。问题是，即使如果它现在不被期望，在将来也会，这个操作符只是悄悄的隐藏了可空性。
一个简单的例子就是一个函数，在4个参数中查找最大的那个。常见的实现就是将4个参数放入一个list,然后使用max函数找出最大的那个。问题是如果list是空的时候，它会返回null。然而开发者知道list不会空，并会用不为空断言！！：
```kotlin
fun largestOf(a: Int, b: Int, c: Int, d: Int): Int =
    listOf(a, b, c, d).max()!!
```
有人可能会重构这个函数，来接收任意参数，但会忽略集合为空的的事实：
```kotlin
fun largestOf(vararg nums: Int): Int =
    nums.max()!!
    
largestOf() // NPE
```
关于可空性的信息是无声的，当它很重要的时候可能被轻易忽略。变量也相似，当你有个变量需要一会才能赋值，但是确定要在第一次调用前赋好值。设置为null，然后使用！！，是错误的做法。
```kotlin
class UserControllerTest {
   
    private var dao: UserDao? = null
    private var controller: UserController? = null
    
    @BeforeEach
    fun init() {
        dao = mockk()
        controller = UserController(dao!!)
    }
    
    @Test
    fun test() {
        controller!!.doSomething()
    }
}
```
每次这样做是令人苦恼的，同时这也阻止了这些属性将来实际有null含义的可能性。后面，我们会看到 正确的处理这种情况的做法就是使用lateinit或者Delegates.notNull.
**没人可以预言代码将来会怎么要演变，如果你使用！！或者明确的抛出错误，你应该假设有一天它将抛出错误**。抛出的异常用来表明一些未预料或者错误的东西发生了。**然而，明确的错误比一般的NPE更能说明问题，它们几乎总是首选** 。
非空断言！！的罕见情况是 使得与一个不能正确表达可空性的库进行交互的结果变得确实有意义。当你与为 Kotlin 设计的 API 进行交互时，这不应该成为一种规范。
通常，我们应该避免使用！！。这个建议被我们社区广泛支持。许多团队都有不使用的规定。有得设置了静态检测分析器，当检测到有使用的时候，就会抛出错误。我认为这样处理太极端了，但我同意！！常常会导致代码异味。**这样的操作符看起来不是巧合，看上去喊叫着“小心”或者“这里有问题”**。

**避免无意义的可空性**

可空性是一种成本，因为它需要正确处理，我们希望在**不需要它的时候避免可空性**。null可能传递着一条重要的信息，但是当它看上去对其他开发者是无意义的时候，我们应该避免这些状况。然后他们会忍不住使用！！或者被迫重复那些容易弄乱代码的通用安全处理机制。应该避免对用户不合理的空安全性。最重要的方法如下：

- 类提供了函数的变种，有的返回是预期的，有的考虑可能是缺少值，是可空的或者是返回一个密封类。一个简单的例子就是List<T>的get和getorNUll.这些将在第7条中讲解。
- 当一个值肯定是在使用之前但在类创建期间之后设置的时候，使用lateinit属性或notNull委托 
- 不要返回null，应该使用一个空的集合代替。当我们处理一个集合的时候，比如 List<Int>?或者Set<String>?,null的含义和一个空集合不同。它表示没有集合出现。表明缺少元素，使用一个空的集合。
- Nullable enum和None enum 是两个不同的意思。null是一个需要被单独处理的特殊信息，但是它没有出现在枚举定义中，你可以添加它到任何使用的地方。

我们来深刻的讨论下lateinit属性和notNull代理

**lateinit属性和notNull代理**

在工程中有些属性在类的创建的时候不能初始化是不少见的，但是肯定在第一次使用前初始化。一个典型的例子就是当一个属性在一个函数里面设置，这个函数比其他函数都先调用，想Junit5里面的@BeforeEach:
 
```kotlin
class UserControllerTest {
    
    private var dao: UserDao? = null
    private var controller: UserController? = null
   
    @BeforeEach
    fun init() {
        dao = mockk()
        controller = UserController(dao!!)
    }
    
    @Test
    fun test() {
        controller!!.doSomething()
    }
    
}
```
当我们需要使用这些属性时，将它们从空值转换为非空值是非常不可取的 .它也是无意义的，因为我们期望这些值在测试之前设置。这个问题的正确解决方式使使用lateinit修饰符，让我们稍后初始化这些属性：
```kotlin
class UserControllerTest {
    private lateinit var dao: UserDao
    private lateinit var controller: UserController
   
    @BeforeEach
    fun init() {
        dao = mockk()
        controller = UserController(dao)
    }
   
    @Test
    fun test() {
        controller.doSomething()
    }
}
```
lateinit的成本就是，如果我们是错的，我们想要在它初始化之前获取值的时候，会抛出一个异常。
听起来有点吓人，但实际上是想要的-我们应该仅当在确定它第一次使用前肯定会初始化的时候使用lateinit.如果我们是错的，我们想要被告知。 lateinit和nullable的主要区别是：

- 不需要每次“解包”（unpack）属性到不为空
- 如果我们需要使用null来表示有意义的东西，在将来我们可以轻松的将它变为nullable
- 一旦属性被初始化了，就不能回到未初始化状态了

**当我们肯定属性在第一次使用前会被初始化的时候，lateinit是一个好的实践**。我们主要处理这些情况：类有生命周期，并且我们在第一个被调用的方法中设置属性，以便在后面的方法中使用它。例如当在Android的Activity中的onCreate方法中设置对象的时候，在一个iOS的UIViewController的viewDidAppear，在一个React的React.Component的componentDidMound.
有一种情况不能使用lateinit,就是 当我们需要初始化的属性是如下类型：JVM的原始类型，像Int,Long,Double或Boolean.这时我们需要使用稍微慢点的，但是支持这些类型的Delegates.notNull：
```kotlin
 class DoctorActivity: Activity() {
     private var doctorId: Int by Delegates.notNull()
     private var fromNotification: Boolean by
         Delegates.notNull()
 
     override fun onCreate(savedInstanceState: Bundle?) {
         super.onCreate(savedInstanceState)
         doctorId = intent.extras.getInt(DOCTOR_ID_ARG)
         fromNotification = intent.extras
             .getBoolean(FROM_NOTIFICATION_ARG)
     }
 }
```
这种情况也经常被属性代理代替，像上面的例子中，在onCreate中我们需要读取参数，可以使用代理，延迟初始化这些属性：
```kotlin
class DoctorActivity: Activity() {
    private var doctorId: Int by arg(DOCTOR_ID_ARG)
    private var fromNotification: Boolean by
    arg(FROM_NOTIFICATION_ARG)
}
```
属性代理模式将详细在 21条进行描述。它之所以流行的原因就是可以安全的帮我们避免可空性。
