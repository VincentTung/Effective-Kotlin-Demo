48.消除过时的对象引用

习惯于具有自动内存管理的语言的程序员很少考虑释放对象。 例如，在 Java 中，垃圾收集器 (GC) 完成了所有工作。 尽管完全忘记内存管理会导致内存泄漏——不必要的内存消耗——并且在某些情况下会导致OutOfMemoryError。 最重要的一条规则是我们不应该保留对不再有用的对象的引用。特别是，如果这样的对象在内存方面很大或者可能有很多这样的对象的实例。
 在 Android 中，有一个常见的初学者错误，开发人员愿意从任何地方自由访问 Activity（类似于桌面应用程序中的窗口的概念），将其存储在静态或伴随对象属性中（经典地，在 静态字段）：
```kotlin

class MainActivity : Activity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        //...
        activity = this
    }
    //...

    companion object {
        // 不要这样做，会导致内存泄露
        var activity: MainActivity? = null
    }
}
```
只要我们的应用程序正在运行，在伴随对象中保存对活动的引用不会让垃圾收集器释放它。 Activity是重物，所以它是一个巨大的内存泄漏。 有一些方法可以改进它，但最好不要静态地持有这些资源。 正确管理依赖项，而不是静态存储它们。 另外，请注意，当我们持有一个存储对另一个对象的引用的对象时，我们可能会处理内存泄漏。 就像在下面的示例中一样，我们拥有一个捕获对 MainActivity 的引用的 lambda 函数：
```kotlin
class MainActivity : Activity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        //...

        // Be careful, we leak a reference to `this`
        logError = {
            Log.e(
                this::class.simpleName,
                it.message
            )
        }
    }

    //...

    companion object {
        // DON'T DO THIS! A memory leak
        var logError: ((Throwable) -> Unit)? = null
    }
}
```

尽管内存问题可能要微妙得多。 看看下面的堆栈实现⁵：

```kotlin
class Stack {
    private var elements: Array<Any?> =
        arrayOfNulls(DEFAULT_INITIAL_CAPACITY)
    private var size = 0
    fun push(e: Any) {
        ensureCapacity()
        elements[size++] = e
    }

    fun pop(): Any? {
        if (size == 0) {
            throw EmptyStackException()
        }
        return elements[--size]
    }

    private fun ensureCapacity() {
        if (elements.size == size) {
            elements = elements.copyOf(2 * size + 1)
        }
    }

    companion object {
        private const val DEFAULT_INITIAL_CAPACITY = 16
    }
}
```
你能在这里发现问题吗？ 花点时间考虑一下。
问题是当我们pop时，我们只是减少大小，但我们没有释放数组上的元素。 假设我们在堆栈上有 1000 个元素，我们一个接一个地弹出几乎所有元素，现在我们的大小等于 1。我们应该只有一个元素。 我们只能访问一个元素。 尽管我们的堆栈仍然拥有 1000 个元素并且不允许 GC 销毁它们。所有这些对象都在浪费我们的内存。 这就是为什么它们被称为内存泄漏。 如果这种泄漏累积起来，我们可能会面临 OutOfMemoryError。 我们如何修复这个实现？ 一个非常简单的解决方案是在不再需要对象时在数组中设置 null：

```kotlin
fun pop(): Any? {
    if (size == 0)
        throw EmptyStackException()
    val elem = elements[--size]
    elements[size] = null
    return elem
}
```
这是一个罕见的例子，也是一个代价高昂的错误，但有些日常使用的对象我们也可以从这条规则中获利，或者可以获利。 假设我们需要一个 mutableLazy 属性委托。 它应该像惰性一样工作，但它也应该允许属性状态是易变的。 我可以使用以下实现来定义它：

```kotlin
fun <T> mutableLazy(initializer: () -> T):
        ReadWriteProperty<Any?, T> = MutableLazy(initializer)

private class MutableLazy<T>(
    val initializer: () -> T
) : ReadWriteProperty<Any?, T> {
    private var value: T? = null
    private var initialized = false

    override fun getValue(
        thisRef: Any?,
        property: KProperty<*>
    ): T {
        synchronized(this) {

            if (!initialized) {
                value = initializer()
                initialized = true
            }
            return value as T
        }
    }

    override fun setValue(
        thisRef: Any?,
        property: KProperty<*>,
        value: T
    ) {
        synchronized(this) {
            this.value = value
            initialized = true
        }
    }
}
```

使用：
```kotlin
var game: Game? by mutableLazy { readGameFromSave() }

fun setUpActions() {
    startNewGameButton.setOnClickListener {
        game = makeNewGame()
        startGame()
    }
    resumeGameButton.setOnClickListener {
        startGame()
    }
}


```

上面 mutableLazy 的实现是正确的，但是有一个缺陷：初始化器在使用后没有被清理。 这意味着只要对 MutableLazy 实例的引用存在，即使它不再有用，它就会被持有。 这就是 MutableLazy 实现可以改进的方式：

```kotlin
fun <T> mutableLazy(initializer: () -> T):
        ReadWriteProperty<Any?, T> = MutableLazy(initializer)

private class MutableLazy<T>(
    var initializer: (() -> T)?
) : ReadWriteProperty<Any?, T> {
    private var value: T? = null
    override fun getValue(
        thisRef: Any?,
        property: KProperty<*>
    ): T {
        synchronized(this) {
            val initializer = initializer
            if (initializer != null) {
                value = initializer()
                this.initializer = null
            }
            return value as T
        }
    }

    override fun setValue(
        thisRef: Any?,
        property: KProperty<*>,
        value: T
    ) {
        synchronized(this) {
            this.value = value
            this.initializer = null
        }

    }
}

```

当我们将 initializer 设置为 null 时，之前的值可以被 GC 回收。
这种优化有多重要？ 为很少使用的对象烦恼并不那么重要。 有句话说，过早的优化是万恶之源。 虽然，在不需要花费太多成本的情况下，将未使用的对象设置为 null 是件好事。 特别是当它是一个可以捕获许多变量的函数类型时，或者当它是一个未知类（如 Any 或泛型类型）时。 例如，上面示例中的 Stack 可能被某人用来存放重物。 这是一个通用工具，我们不知道它将如何使用。 对于这样的工具，我们应该更关心优化。在创建库时尤其如此。 例如，在 Kotlin 标准库的惰性委托的所有 3 个实现中，我们可以看到初始化器在使用后被设置为 null：
```kotlin
private class SynchronizedLazyImpl<out T>(
    initializer: () -> T, lock: Any? = null
) : Lazy<T>, Serializable {
    private var initializer: (() -> T)? = initializer
    private var _value: Any? = UNINITIALIZED_VALUE
    private val lock = lock ?: this
    override val value: T
        get() {
            val _v1 = _value
            if (_v1 !== UNINITIALIZED_VALUE) {
                @Suppress("UNCHECKED_CAST")
                return _v1 as T
            }

            return synchronized(lock) {
                val _v2 = _value

                if (_v2 !== UNINITIALIZED_VALUE) {
                    @Suppress("UNCHECKED_CAST") (_v2 as T)
                } else {
                    val typedValue = initializer!!()
                    _value = typedValue
                    initializer = null
                    typedValue
                }
            }
        }

    override fun isInitialized(): Boolean =
        _value !== UNINITIALIZED_VALUE

    override fun toString(): String =
        if (isInitialized()) value.toString()
        else "Lazy value not initialized yet."

    private fun writeReplace(): Any =
        InitializedLazyImpl(value)
}

```

一般规则是，当我们持有状态时，我们的头脑中应该有内存管理。 在更改实现之前，我们应该始终为我们的项目考虑最佳权衡，不仅要考虑内存和性能，还要考虑我们解决方案的可读性和可扩展性。 通常，可读代码在性能或内存方面也会做得相当好。 不可读的代码更有可能隐藏内存泄漏或浪费的 CPU 功率。 尽管有时这两个值是对立的，但在大多数情况下，可读性更为重要。 当我们开发一个库时，性能和内存往往更为重要。
我们需要讨论一些常见的内存泄漏源。 首先，缓存包含可能永远不会使用的对象。这是缓存背后的想法，但是当我们遇到内存不足错误时，它对我们没有帮助。 解决方案是使用**软引用**。如果需要内存，GC 仍然可以收集对象，但通常它们会存在并会被使用。
一些对象可以使用弱引用来引用。 例如屏幕上的对话框。 只要显示出来，无论如何都不会被垃圾收集。 一旦它消失了，我们无论如何都不需要引用它。 它是使用弱引用引用的对象的完美候选者。
最大的问题是内存泄漏有时很难预测，并且直到应用程序崩溃时才会显现出来。 对于 Android 应用程序尤其如此，因为与其他类型的客户端（如桌面）相比，内存使用限制更严格。 这就是我们应该使用特殊工具搜索它们的原因。 最基本的工具是堆分析器。 我们还有一些库可以帮助搜索数据泄漏。 例如，一个流行的 Android 库是 LeakCanary，它会在检测到内存泄漏时显示通知。
重要的是要记住手动释放对象的需要非常罕见。 在大多数情况下，这些对象无论如何都会被释放，这要归功于作用域，或者当持有它们的对象被清理时。 避免混乱内存的最重要方法是在局部范围内定义变量（第 2 项：最小化变量范围），并且不要在顶级属性或对象声明（包括伴随对象）中存储可能的大量数据。

