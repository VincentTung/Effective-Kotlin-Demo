15条：考虑显示的引用接收者receiver

我们可能会选择更长的结构来使某些内容变得明确,一个常见地方是，当我们想要强调函数或属性是从接收器获取而不是本地变量或顶级变量时。在最基本的情况下，它意味着对与方法关联的类的引用。
```kotlin
class User: Person() {
    private var beersDrunk: Int = 0
    fun drinkBeers(num: Int) {
        // ...
        this.beersDrunk += num
        // ...
    }
}
```
同样，我们可能会明确的引用一个扩展接收者（一个扩展函数 ）来让它使用更加明确。请比较下面没有明确接受者的快速排序算法：
```kotlin
fun <T : Comparable<T>> List<T>.quickSort(): List<T> {
    if (size < 2) return this
    val pivot = first()
    val (smaller, bigger) = drop(1)
        .partition { it < pivot }
    return smaller.quickSort() + pivot + bigger.quickSort()
}
```
使用了明确接收者的
```kotlin
fun <T : Comparable<T>> List<T>.quickSort(): List<T> {
    if (this.size < 2) return this
    val pivot = this.first()

    val (smaller, bigger) = this.drop(1)
        .partition { it < pivot }
    return smaller.quickSort() + pivot + bigger.quickSort()
}
```
两个函数的用法相同：
```kotlin
listOf(3, 2, 5, 1, 6).quickSort() // [1, 2, 3, 5, 6]
listOf("C", "D", "A", "B").quickSort() // [A, B, C, D]
```


**多个接收者**

当我们在一个包含多个接收者的范围里的时候，使用明确的接收者是特别有帮助的。我们在使用apply、with或者run函数的时候经常会遇到这样的情况。这种情况是危险的，应该避免。使用明确的接收者能够使对象的使用更加安全。为了弄懂这个问题，请看下面的代码：
```kotlin
class Node(val name: String) {
    
    fun makeChild(childName: String) =
    create("$name.$childName")
        .apply { print("Created ${name}") }
        
    fun create(name: String): Node? = Node(name)
}

fun main() {
    val node = Node("parent")
    node.makeChild("child")
}
```
结果是什么？现在停下来，在看答案前花点时间尝试回答这个问题。
或许，期望的结果应该是"Created parent.child",但实际的结果是"Created parent".为什么呢？来研究一下，现在在name前明确的使用接收者：

```kotlin
class Node(val name: String) {
    fun makeChild(childName: String) =
        create("$name.$childName")
            .apply { print("Created ${this.name}") }
        // Compilation error
    
    fun create(name: String): Node? = Node(name)
}
```
问题是在apply里面的类型是Node?,所以不能直接调用。我们需要解包(unpack)，比如使用空安全调用.如果这么做了，结果就是正确的了：
```kotlin
class Node(val name: String) {
    
    fun makeChild(childName: String) =
    create("$name.$childName")
        .apply { print("Created ${this?.name}") }

    fun create(name: String): Node? = Node(name)
} 

fun main() {
    val node = Node("parent")
    node.makeChild("child")
    // Prints: Created parent.child

}
```
这是一个关于apply的错误使用例子。但我们使用also代替,代用参数name，不会有这个问题.also强制我们以显示接收者的方式来显示引用函数的及守着。通常 当我们操作一个可空值得时候，使用also和let做额外的操作是更好的选择。
```kotlin
class Node(val name: String) {
    
    fun makeChild(childName: String) =
    create("$name.$childName").also { print("Created ${it?.name}") }
    
    fun create(name: String): Node? = Node(name)
}
```
当接收者不清楚的时候，我们要不避免它，要不就使用明确的接收者。当使用没有明确标签的receiver的时候，意味着将使用最近的那个。这时候明确的使用它是有用的。下面是这两种情况的使用例子：
```kotlin
class Node(val name: String) {
    
    fun makeChild(childName: String) =
    create("$name.$childName").apply {
        print("Created ${this?.name} in "+
                " ${this@Node.name}")
    }
    
    fun create(name: String): Node? = Node(name)
}

fun main() {
    val node = Node("parent")
    node.makeChild("child")
    // Created parent.child in parent
}
```
这样明确了我们想要的接收者。这样做将提供重要的信息，不仅避免了错误，也增加了可读性。

**DSL标记**

在这种情况下我们经常在具有不同接收器的非常嵌套的作用域上进行操作，我们根本不需要使用显式接收者。我讲的是Kotlin的DSL.我们不需要明确的使用接收者，因为DSL就是为这种情况设计的。然而，在DSL中，从
作用域外部偶然调用函数是特别危险的。想一下，一个简单的HTML DSL，我们用它来完成一个HTML表格：
```kotlin
table {
    tr {
        td { +"Column 1" } 
        td { +"Column 2" } 
    }
   
    tr {
        td { +"Value 1" }
        td { +"Value 2" }
    }
 }
```
注意，默认情况下，在每个作用域中，我们也可以使用来自外部作用域的接收者的方法。我们可能会利用这个事实来搞乱 DSL：
```kotlin
table {
    tr {
        td { +"Column 1" }
        td { +"Column 2" }
        tr {
            td { +"Value 1" }
            td { +"Value 2" }
        } 
    }

}
```
为了限制这种用法，有一个特别的元注解来限制作用域外的接收者的隐式调用，它就是DslMarker.当我们在一个注解上使用它，然后在一个用作构建器的类上使用这个注解时，在这个构建器内隐式接收器使用是不可能的。以下是如何使用 DslMarker 的示例：
```kotlin
@DslMarker
annotation class HtmlDsl

fun table(f: TableDsl.() -> Unit) { /*...*/ }
@HtmlDsl
class TableDsl { /*...*/ }
```
这样做，就可以阻止我们隐式的使用外部的接收者：
```kotlin
table {
    tr {
        td { +"Column 1" }
        td { +"Column 2" }
        tr { // COMPILATION ERROR
            td { +"Value 1" }
            td { +"Value 2" }
        }
    }
}
```
使用外部的接收者的函数需要明确的接收者：
```kotlin
table {
    tr {
        td { +"Column 1" }
        td { +"Column 2" } 
        this@table.tr {
            td { +"Value 1" }
            td { +"Value 2" }
        } 
    }
}
```
DSL标记是一种重要的机制，我们使用它来强制使用最近的接收者或显式接收者。但是，无论如何，最好不要在 DSL 中使用显式接收器。尊重 DSL 设计并相应地使用它。

**总结**

不要仅仅因为可以更改范围接收者就去做。有太多的接收者都为我们提供了我们可以使用的方法，这可能会让人感到困惑。明确的参数或引用通常更好。当我们确实更改接收器时，使用显式接收者可以提高可读性，因为它阐明了函数的来源。当有许多接收者时，我们甚至可以使用标签来说明函数来自哪一个。如果要强制使用外部范围的显式接收者，可以使用 DslMarker 元注释。
