47.考虑使用inline类

不仅可以内联函数，还可以用该值替换持有单个值的对象。 这种可能性在 Kotlin 1.3 中作为实验性引入，为了使其成为可能，我们需要将 inline 修饰符放在具有**单个主构造函数属性**的类之前：
```kotlin
inline class Name(private val value: String) {
    // ...
}
```
这样的类将尽可能替换为它所拥有的值：
```kotlin
// Code
val name: Name = Name("Marcin") 
// 编译的时候会被替换如下：
val name: String = "Marcin"
```
来自此类的方法将被评估为静态方法：
```kotlin
inline class Name(private val value: String) {
    // ...

    fun greet() {
        print("Hello, I am $value")
    }
}

// Code
val name: Name = Name("Marcin")
name.greet()

// 编译时将被替换如下:
val name: String = "Marcin"
Name.`greet-impl`(name)
```
此类类中的方法将被评估为我们可以使用内联类对某种类型（如上例中的 String）进行包装，而不会产生性能开销（第 45 条：避免不必要的对象创建）。 内联类的两个特别流行的用途是：

- 表示计量单位
- 使用类型来保护用户免受误用

让我们分别讨论它们。静态方法：

**指明计量单位**

假设你需要用一个方法来设置一个定时器：
```kotlin
interface Timer {
    fun callAfter(time: Int, callback: ()->Unit)
}
```
这次是什么时候？ 可能是以毫秒、秒、分钟为单位的时间……目前还不清楚，很容易出错。 一个严重的错误。 这种错误的一个著名例子是坠入火星大气层的火星气候轨道器。 其背后的原因是用于控制它的软件是由一家外部公司开发的，它产生的输出与 NASA 预期的不同。 它产生的结果以磅力秒 (lbf·s) 为单位，而 NASA 预计为牛顿-秒 (N·s)。 任务总成本为 3.276 亿美元，完全失败。 如你所见，测量单位的混淆可能非常昂贵。
开发人员建议度量单位的一种常见方法是将其包含在参数名称中：
```kotlin
interface Timer { 
    fun callAfter(timeMillis: Int, callback: ()->Unit)
}
```
它更好，但仍然为错误留下了一些空间。 使用函数时，属性名称通常不可见。 另一个问题是，当返回类型时，以这种方式指示类型更加困难。 在下面的例子中，时间是从decisionAboutTime返回的，它的度量单位根本没有指明。它可能以分钟为单位返回时间，然后时间设置将无法正常工作。
```kotlin

interface User {
    fun decideAboutTime(): Int
    fun wakeUp()
}

interface Timer {
    fun callAfter(timeMillis: Int, callback: () -> Unit)
}

fun setUpUserWakeUpUser(user: User, timer: Timer) {
    val time: Int = user.decideAboutTime()
    timer.callAfter(time) {
        user.wakeUp()
    }
}

```
我们可能会在函数名称中引入返回值的度量单位，例如将其命名为decisionAboutTimeMillis，但这样的解决方案相当少见，因为每次使用它都会使函数变长，并且它说明了这些低级信息 即使我们不需要知道它。解决这个问题的更好方法是引入更严格的类型，以防止我们误用更多的泛型类型，并且为了使它们高效，我们可以使用内联类：
```kotlin
inline class Minutes(val minutes: Int) {
    fun toMillis(): Millis = Millis(minutes * 60 * 1000)
    // ...
}

inline class Millis(val milliseconds: Int) {
    // ...
}

interface User {
    fun decideAboutTime(): Minutes
    fun wakeUp()
}

interface Timer {
    fun callAfter(timeMillis: Millis, callback: () -> Unit)
}

fun setUpUserWakeUpUser(user: User, timer: Timer) {
    val time: Minutes = user.decideAboutTime()
    timer.callAfter(time) { // ERROR: Type mismatch
        user.wakeUp()
    }
}
```
这会强制我们使用正确的类型：
```kotlin
fun setUpUserWakeUpUser(user: User, timer: Timer) {
    val time = user.decideAboutTime()
    timer.callAfter(time.toMillis()) {
        user.wakeUp()
    }
}
```
它对公制单位特别有用，因为在前端，我们经常使用像素、毫米、dp 等多种单位。为了支持对象创建，我们可以定义类似 DSL 的扩展属性（你可以将它们设为内联函数）：
```kotlin
inline val Int.min
    get() = Minutes(this)

inline val Int.ms
    get() = Millis(this)

val timeMin: Minutes = 10.min
```


**保护我们免受类型误用**

在 SQL 数据库中，我们经常通过 ID 来识别元素，这些 ID 都是数字。 因此，假设你在系统中有一个学生成绩。 它可能需要引用学生、教师、学校等的 id：
```kotlin
@Entity(tableName = "grades")
class Grades(
    @ColumnInfo(name = "studentId")
    val studentId: Int,
    @ColumnInfo(name = "teacherId")
    val teacherId: Int,
    @ColumnInfo(name = "schoolId")
    val schoolId: Int,
// ...
)
```
问题是以后很容易误用所有这些 id，并且上输入系统并不能保护我们，因为它们都是 Int 类型。解决方案是将所有这些整数包装到单独的内联类中：

```kotlin
inline class StudentId(val studentId: Int)
inline class TeacherId(val teacherId: Int)
inline class SchoolId(val studentId: Int)

@Entity(tableName = "grades")
class Grades(
    @ColumnInfo(name = "studentId")
    val studentId: StudentId,
    @ColumnInfo(name = "teacherId")
    val teacherId: TeacherId,
    @ColumnInfo(name = "schoolId")
    val schoolId: SchoolId,
    // ...
)


```
现在那些 id 使用将是安全的，同时，数据库将正确生成，因为在编译期间，所有这些类型无论如何都会被 Int 替换。 这样，内联类允许我们在以前不允许的地方引入类型，并且由于这一点，我们有更安全的代码而没有性能开销。

**内联类和接口**

内联类就像其他类一样可以实现接口。 这些接口可以让我们以我们想要的任何度量单位正确地打发时间。
```kotlin
inline class Minutes(val minutes: Long) : TimeUnit {
    override val millis: Long get() = minutes * 60 * 1000
    // ...
}

inline class Millis(val milliseconds: Long) : TimeUnit {
    override val millis: Long get() = milliseconds
}

fun setUpTimer(time: TimeUnit) {
    val millis = time.millis
    //...
}

setUpTimer(Minutes(123))
setUpTimer(Millis(456789))
```
问题是当一个对象通过接口使用时，它不能被内联。 因此在上面的示例中，使用内联类没有任何优势，因为需要创建包装对象才能让我们通过此接口呈现类型。 当我们通过接口呈现内联类时，这些类不是内联的。

**类型别名（Typealias）**

Kotlin typealias 允许我们为类型创建另一个名称：
```kotlin
typealias NewName = Int
val n: NewName = 10
```

命名类型是一种有用的功能，尤其是在我们处理长且可重复的类型时。例如，命名可重复的函数类型是一种流行的做法：

```kotlin
typealias ClickListener =
            (view: View, event: Event) -> Unit

class View {
    fun addClickListener(listener: ClickListener) {}
    fun removeClickListener(listener: ClickListener) {}
    //...
}
```

但是需要理解的是，类型别名并不能以任何方式保护我们免受类型滥用。 他们只是为一种类型添加一个新名称。 如果我们将 Int 命名为 Millis 和 Seconds，我们会产生一种错觉，即输入系统保护我们，而它没有：

```kotlin
typealias Seconds = Int
typealias Millis = Int

fun getTime(): Millis = 10

fun setUpTimer(time: Seconds) {}

fun main() {
    val seconds: Seconds = 10
    val millis: Millis = seconds // No compiler error

    setUpTimer(getTime())
}
```
在上面的示例中，如果没有类型别名，将更容易找到问题所在。 这就是为什么不应该以这种方式使用它们的原因。要指示度量单位，请使用参数名称或类。 名称更便宜，但类提供更好的安全性。 当我们使用内联类时，我们会从这两种选择中得到最好的——它既便宜又安全。

**总结**

内联类让我们在没有性能开销的情况下包装类型。 多亏了这一点，我们通过使我们的输入系统保护我们免受值滥用来提高安全性。 如果你使用含义不明确的类型，尤其是可能具有不同度量单位的类型，请考虑使用内联类对其进行包装。
