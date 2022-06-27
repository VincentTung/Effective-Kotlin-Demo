第1条：限制可变性

在 Kotlin 中，我们以模块的形式设计程序，每个模块都由不同种类的元素组成，例如类、对象、函数、类型别名和顶级属性。 其中一些元素可以保持状态，例如通过具有读写的属性 var 或可变对象组成：
```kotlin
 var a = 10 
 val list: MutableList<Int> = mutableListOf()
```
当一个元素持有状态时，它的行为方式不仅取决于你如何使用它，还取决于它的历史。具有状态的类的一个典型示例是具有一些货币余额的银行帐户：
```kotlin
class BankAccount {
    var balance = 0.0
    private set
    
    fun deposit(depositAmount: Double) {
        balance += depositAmount
    } 
    
    @Throws(InsufficientFunds::class)
    fun withdraw(withdrawAmount: Double) {
        if (balance < withdrawAmount) {
            throw InsufficientFunds()
        }
        balance -= withdrawAmount
    }
}

class InsufficientFunds : Exception()

val account = BankAccount()
println(account.balance) // 0.0
account.deposit(100.0)
println(account.balance) // 100.0
account.withdraw(50.0)
println(account.balance) // 50.0
```
这里 BankAccount 有一个状态，表示该帐户上现在存在多少钱。持有状态是是一把双刃剑。一方面它非常有用，因为它可以表示随时间变化的元素，但另一方面，状态管理很困难，因为：

1. 拥有很多可变地方的程序理解和调试起来是很困难的。这些可变地方之间的关系需要被理解，如果有很多可变地方，追踪它们是如何改变的将是很困难的。具有许多相互依赖的可变地方的类通常很难理解和修改。在出现意外情况或错误时尤其成问题。
1. 可变性使得对代码的推理变得更加困难。 不可变元素的状态是明确的。 可变状态更难理解。 很难推断它的值是什么，因为它可能随时发生变化，并且仅仅因为我们在某个时刻检查过它并不意味着它仍然是相同的。
1. 它需要在多线程程序中进行适当的同步。每个可变的地方都是潜在的冲突。
1. 可变元素更难测试。 我们需要测试每一个可能的状态，可变性越多，需要测试的状态就越多。 更重要的是，我们需要测试的状态数量通常会随着同一对象或文件中可变点的数量呈指数增长，因为我们需要测试可能状态的所有不同组合。
1. 当状态发生变化时，通常需要通知其他一些类有关此更改。 例如，当我们将一个可变元素添加到排序列表中时，一旦元素更改，我们需要再次对该列表进行排序。

对于在更大团队中工作的开发人员来说，状态一致性问题和具有更多可变点的项目日益复杂是很熟悉的。 让我们看一个管理共享状态有多难的例子。 看看下面的片段。 它显示了多个线程试图修改相同的属性，但是由于冲突，其中一些操作将丢失。
```kotlin
var num = 0

for (i in 1..1000) {
    thread {
        Thread.sleep(10)
        num += 1
    }
} 

Thread.sleep(5000)
print(num) // Very unlikely to be 1000
// Every time a different number
```
当我们使用 Kotlin 协程时，由于涉及的线程较少，因此冲突较少，但仍然会发生：
```kotlin
suspend fun main() {
    var num = 0
    coroutineScope {
        for (i in 1..1000) {
            launch {
                delay(10)
                num += 1
            }
        }
    }

    print(num) // Every time a different number
}
```
在实际的项目中，我们通常不能只丢失一些操作，因此我们需要像下面介绍的那样实现适当的同步。 尽管实现正确的同步很困难，而且我们拥有的突变点越多，就越难。限制可变性确实有帮助。
```kotlin
val lock = Any()
var num = 0
for (i in 1..1000) {
    thread {
        Thread.sleep(10)
        synchronized(lock) {
            num += 1
        }
    }
}
Thread.sleep(1000)
print(num) // 1000
```
可变性的缺点如此之多，以至于有些语言根本不允许状态突变。 这些是纯粹的函数式语言。 一个著名的例子是 Haskell。 但是，此类语言很少用于主流开发，因为在可变性如此有限的情况下很难进行编程。可变状态是表示现实世界系统状态的一种非常有用的方式。 我建议使用可变性，但要谨慎并明智地决定我们的可变点应该在哪里。 好消息是 Kotlin 很好地支持限制可变性。
**在Kotlin中限制可变性**
Kotlin被设计用来支持限制可变性。生成不可变对象或保持属性不可变很容易。这是这种语言的许多特性和特征的结果，但最重要的是：

- 制度属性val
- 可变集合和只读集合之间的分离
- 在data类中复制

让我们一个一个讨论它们
**只读属性val**
在Kotlin中 ，我们可以让任意属性 只读val(像value)或者读写 var(像variable)。只读属性val不允许重复赋值：
```kotlin
val a = 10
a = 20 // ERROR
```
请注意，只读属性不一定是不可变的，也不一定是最终的。 只读属性可以保存可变对象：
```kotlin
val list = mutableListOf(1,2,3)
list.add(4)

print(list) // [1, 2, 3, 4]
```
也可以使用可能依赖于另一个属性的自定义 getter 来定义只读属性
```kotlin
var name: String = "Marcin"
var surname: String = "Moskała"

val fullName
get() = "$name $surname"

fun main() {
    println(fullName) // Marcin Moskała
    name = "Maja"
    println(fullName) // Maja Moskała
}
```
请注意，这是可能的，因为当我们自定义 getter 时，每次我们要求值时都会调用它。
```kotlin
fun calculate(): Int {
    print("Calculating... ")
    return 42
}
val fizz = calculate() // Calculating...
val buzz
    get() = calculate()
    
fun main() {
    print(fizz) // 42
    print(fizz) // 42
    print(buzz) // Calculating... 42
    print(buzz) // Calculating... 42
}
```

Kotlin 中的属性是默认封装的，并且它们可以具有自定义访问器（getter 和 setter）这一特性在 Kotlin 中非常重要，因为它在我们更改或定义 API 时为我们提供了灵活性。 将在第 16 条中详细描述：属性应该代表状态，而不是行为。 核心思想是 val 不提供可变点，因为当 var 既是 getter 又是 setter 时，它只是底层的 getter。 这就是为什么我们可以用 var 覆盖 val：
```kotlin
interface Element {
    val active: Boolean
}

class ActualElement: Element {
    override var active: Boolean = false
}
```
只读属性 val 的值可以更改，但此类属性不提供可变点，这是我们需要同步或推理程序时问题的主要来源。 这就是为什么我们通常更喜欢 val 而不是 var。
尽管请记住 val 并不意味着不可变。 它可以由 getter 或 delegate 定义。 这个事实给了我们更多改变的自由。 虽然当我们不需要它时，应该首选最终属性。 对它们进行推理更容易，因为它们的定义旁边有状态说明。 Kotlin 也更好地支持它们。 例如，它们可以被智能转换：
```kotlin
val name: String? = "Márton"
val surname: String = "Braun"

val fullName: String?
    get() = name?.let { "$it $surname" }

val fullName2: String? = name?.let { "$it $surname" }

fun main() {
    if (fullName != null) {
        println(fullName.length) // ERROR
    }
    
    if (fullName2 != null) {
        println(fullName2.length) // Márton Brau
    }
}
```
fullName 不可能进行智能转换，因为它是使用 getter 定义的，因此它可能会在检查期间给出不同的值，并且在稍后使用期间会给出不同的值（例如，如果其他线程会设置名称）。 非本地属性只有在它们是最终的并且没有自定义 getter 时才能被智能转换。
**可变集合和只读集合之间的分离**
类似地，正如 Kotlin 将读写和只读属性分开一样，Kotlin 将读写和只读集合分开。 这要归功于集合层次结构的设计方式。 看一下在 Kotlin 中展示集合层次结构的图表。 在左侧，您可以看到只读的Iterable、Collection、Set和List接口。 这意味着他们没有任何允许修改的方法。 在右侧，您可以看到表示可变集合的 MutableIterable、MutableCollection、MutableSet 和 MutableList 接口。 请注意，每个可变接口都扩展了相应的只读接口，并添加了允许可变的方法。 这类似于属性的工作方式。 只读属性仅表示 getter，而读写属性同时表示 getter 和 setter。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/10378482/1656248518700-4e7b6f95-98dd-470b-aa53-bf4439852549.png#clientId=ue384994b-4d15-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=314&id=ue28d787f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=628&originWidth=1130&originalType=binary&ratio=1&rotation=0&showTitle=false&size=392317&status=done&style=none&taskId=ud32cf7e4-9bb2-4386-92c5-3d13ca74bc0&title=&width=565)
只读集合不一定是不可变的。 很多时候它们是可变的，但它们不能被改变，因为它们隐藏在只读接口后面。 例如，Iterable<T>.map 和 Iterable<T>.filter 函数将 ArrayList（一个可变列表）作为 List（一个只读接口）返回。 在下面的代码片段中，可以看到来自 标准的 Iterable<T>.map 的简化实现。
```kotlin
inline fun <T, R> Iterable<T>.map(
    transformation: (T) -> R
): List<R> {
    
    val list = ArrayList<R>()
    for (elem in this) {
        list.add(transformation(elem))
    } 
    return list
    
}
```
使这些集合接口成为只读而不是真正不可变的设计选择非常重要。 它给了我们更多的自由。 在底层，只要满足接口，任何实际的集合都可以返回。 因此，我们可以使用平台特定的集合。
这种方法的安全性接近于拥有不可变集合所获得的安全性。 唯一的风险是当开发人员试图“破解系统”并执行向下转换时。 这是 Kotlin 项目中绝不应该允许的事情。 我们应该能够相信，当我们以只读方式返回列表时，它将仅用于读取它。 这是合同的一部分。 在本书的第 2 部分中了解更多信息
向下转换集合不仅违反了它们的约定，并且依赖于实现而不是我们应该的抽象，而且它也是不安全的，并且可能导致令人惊讶的后果。 看看这段代码：
```kotlin
val list = listOf(1,2,3) 

// DON’T DO THIS!
if (list is MutableList) {
    list.add(4)
}
```

此操作的结果是特定于平台的。 在 JVM 上 listOf 返回一个实现 Java List 接口的 Arrays.ArrayList 实例。 这个 Java List 接口具有 add 或 set 等方法，因此它转换为 Kotlin MutableList 接口。但是，Arrays.ArrayList 没有实现其中的一些操作。 这就是为什么上面代码的结果如下：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/10378482/1656249008284-c8e46a4c-ca77-49d0-98fb-afc0f73a3fb7.png#clientId=ue384994b-4d15-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=115&id=uda8c2fb8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=230&originWidth=960&originalType=binary&ratio=1&rotation=0&showTitle=false&size=117636&status=done&style=none&taskId=u4d15c4ae-123a-4940-a822-a276b7c2fcd&title=&width=480)
虽然不能保证从现在起一年后它会如何表现。 基础集合可能会发生变化。 它们可能会被 Kotlin 中实现的真正不可变集合所取代，而根本不实现 MutableList。 没有什么是保证的。 这就是为什么在 Kotlin 中永远不应该将只读集合向下转换为可变集合的原因。 如果您需要从只读更改为可变，您应该使用 List.toMutableList 函数，该函数创建一个您可以修改的副本：

```kotlin
val list = listOf(1, 2, 3) 

val mutableList = list.toMutableList()
mutableList.add(4)
```

这种方式不会破坏任何合同，而且对我们来说也更安全，因为我们可以放心，当我们将某些内容公开为 List 时，它不会从外部修改。
**在data类中复制**
有很多理由更喜欢不可变对象 - 不改变其内部状态的对象，如 String 或 Int。 除了已经提到的我们通常更喜欢较少可变性的原因之外，不可变对象还有其自身的优势：

1. 它们更容易推理，因为它们的状态在创建后保持不变。
1. 不变性使程序更容易并行化，因为共享对象之间没有冲突。
1. 对不可变对象的引用可以被缓存，因为它们不会改变。
1. 我们不需要在不可变对象上制作防御性副本。当我们复制不可变对象时，我们不需要将其设为深拷贝。
1. 不可变对象是构造其他对象的完美材料。既可变又不可变。我们仍然可以决定可变性发生在哪里，并且更容易对不可变对象进行操作。
1. 我们可以添加它们来设置或将它们用作map中的key，与不应该以这种方式使用的可变对象相反。这是因为这两个集合都在 Kotlin/JVM 的底层使用了哈希表，当我们修改已经分类到哈希表的元素时，它的分类可能不再正确，我们将无法找到它。这个问题将在第 41 条：尊重 hashCode 的约定中详细描述。对集合进行排序时，我们遇到了类似的问题。
```kotlin
val names: SortedSet<FullName> = TreeSet()
val person = FullName("AAA", "AAA")
names.add(person)
names.add(FullName("Jordan", "Hansen"))
names.add(FullName("David", "Blanc"))

print(names) // [AAA AAA, David Blanc, Jordan Hansen]
print(person in names) // true

person.name = "ZZZ"
print(names) // [ZZZ AAA, David Blanc, Jordan Hansen]
print(person in names) // false
```
在最后一次检查中，即使该人在此集合中，collection 也返回 false。 找不到它，因为它的位置不正确。
如 你所见，可变对象更危险且更难以预测。 另一方面，不可变对象的最大问题是数据有时需要更改。 解决方案是不可变对象应该具有在某些更改后生成对象的方法。 例如， Int 是不可变的，它有许多方法，如 plus 或 minus 不会修改它，而是在此操作后返回一个新的 Int。 Iterable 是只读的，像 map 或 filter 这样的集合处理函数不会修改它，而是返回一个新的集合。 这同样适用于我们的不可变对象。 例如，假设我们有一个不可变类 User，我们需要允许其姓氏改变。 我们可以使用 withSurname 方法来支持它，该方法会生成一个更改了特定属性的副本：
```kotlin
class User(
    val name: String,
    val surname: String
) {
    fun withSurname(surname: String) = User(name, surname)
} 

var user = User("Maja", "Markiewicz")
user = user.withSurname("Moskała")
print(user) // User(name=Maja, surname=Moskała)
```
编写这样的函数是可能的，但如果我们需要为每个属性编写一个函数，那也是乏味的。 data修饰符来救援了。 它生成的方法之一是复制。 它创建一个新实例，默认情况下，所有主要构造函数属性都与前一个相同。 也可以指定新值。 复制和其他由数据修饰符生成的方法在第 37 条：使用数据修饰符表示一捆数据中有详细描述。 这是一个简单的例子，展示了它是如何工作的：
```kotlin
data class User(
    val name: String,
    val surname: String
)

var user = User("Maja", "Markiewicz")
user = user.copy(surname = "Moskała")
print(user) // User(name=Maja, surname=Moskała)
```
这是一个优雅且通用的解决方案，支持使数据模型类不可变。 当然，这种方式比只使用可变对象效率低，但它具有不可变对象的所有优点，默认情况下应该首选。
**不同的可变点**
假设我们需要表示一个可变list。 我们有两种方法可以实现。 通过使用可变集合或使用读写属性 var：
```kotlin
val list1: MutableList<Int> = mutableListOf()
var list2: List<Int> = listOf()
```
这两个属性都可以修改，但方式不同：
```kotlin
list1.add(1)
list2 = list2 + 1
```
这两种方式也可以用加号赋值运算符替换，但它们中的每一种都被转换为不同的行为：
```kotlin
list1 += 1 // Translates to list1.plusAssign(1)
list2 += 1 // Translates to list2 = list2.plus(1)
```
这两种方式都是正确的，各有利弊。 它们都有一个可变点，但它位于不同的位置。 在第一个可变发生在具体的列表实现上。 我们可能依赖于它在多线程的情况下具有适当的同步这一事实，但这样的假设也是危险的，因为它并不能真正得到保证。 在第二种情况下，我们需要自己实现同步，但整体安全性更好，因为可变点只有一个属性。 但是，在缺乏同步的情况下，请记住我们仍然会丢失一些元素：
```kotlin
var list = listOf<Int>()
for (i in 1..1000) {
    thread {
        list = list + i
    }
}
Thread.sleep(1000)
print(list.size) // Very unlikely to be 1000,
// every time a different number, like for instance 911
```
使用可变属性而不是可变list允许我们在定义自定义设置器或使用委托（使用自定义设置器）时跟踪此属性的变化。 例如，当我们使用可观察委托时，我们可以记录list的每次更改：
```kotlin
var names by Delegates.observable(listOf<String>()) {
    _, old, new ->
    println("Names changed from $old to $new") 
} 

names += "Fabio"
// Names changed from [] to [Fabio]
names += "Bill"
// Names changed from [Fabio] to [Fabio, Bill]
```
为了使可变集合成为可能，我们需要一个特殊的可观察集合实现。 对于可变属性的只读集合，控制它们的变化方式也更容易——只有一个 setter 而不是多个方法改变这个对象，我们可以将其设为私有：
```kotlin
var announcements = listOf<Announcement>()
    private set
```
简而言之，使用可变集合是一个稍微快一点的选择，但是使用可变属性可以让我们更好地控制对象的变化方式。
请注意，最糟糕的解决方案是同时拥有一个可变属性和一个可变集合：
```kotlin
// Don’t do that
var list3 = mutableListOf<Int>()
```
我们需要同步它可以可变的两种方式（通过属性更改和内部状态更改）。 此外，由于模棱两可，使用 plus-assign 更改它是不可能的：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/10378482/1656250853012-fca70780-207f-4f81-8bf2-ab66195ab05b.png#clientId=ue384994b-4d15-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=122&id=ua1d9d1f8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=244&originWidth=1110&originalType=binary&ratio=1&rotation=0&showTitle=false&size=138006&status=done&style=none&taskId=u0369cf1d-d26f-4749-b5a3-41ac2e321a7&title=&width=555)
一般规则是不应该创造不必要的方式来改变状态。 每一种改变状态的方式都是有代价的。 它需要被理解和维护。 我们更喜欢限制可变性。
**不要泄漏可变点**
当我们暴露一个构成状态的可变对象时，这是一种特别危险的情况。 看看这个例子：
```kotlin
data class User(val name: String)

class UserRepository {
    private val storedUsers: MutableMap<Int, String> = mutableMapOf()

    fun loadAll(): MutableMap<Int, String> {
        return storedUsers
    }

//...
}
```
可以使用 loadAll 来修改 UserRepository 私有状态：
```kotlin
val userRepository = UserRepository()

val storedUsers = userRepository.loadAll()
storedUsers[4] = "Kirill"
//...

print(userRepository.loadAll()) // {4=Kirill}
```
当这种修改是偶然的时，尤其危险。 我们有两种方法可以处理它。 第一个是复制返回的可变对象。 我们称之为防御性复制。 当我们处理标准对象时，这可能是一种有用的技术，在这里由data修饰符生成的副本非常有用：
```kotlin
class UserHolder {
    private val user: MutableUser()

    fun get(): MutableUser {
        return user.copy()
    } 
    
    //...

}
```
尽管只要有可能，我们更喜欢限制可变性，对于集合，我们可以通过将这些对象向上转换为它们的只读超类型来做到这一点：
```kotlin
data class User(val name: String)

class UserRepository {
    private val storedUsers: MutableMap<Int, String> =
    mutableMapOf()

    fun loadAll(): Map<Int, String> {
        return storedUsers
    }

    //...
}
```
**总结**

在本章中，我们学习了为什么限制可变性和偏爱不可变对象很重要。 我们已经看到 Kotlin 为我们提供了许多支持限制可变性的工具。 我们应该使用它们来限制可变点。 简单的规则是：

-  首选 val 而不是 var。
-  更喜欢不可变属性而不是可变属性。
- 更喜欢不可变的对象和类而不是可变的。
- 如果需要更改它们，请考虑使它们成为不可变的数据类，并使用复制。
- 当持有状态时，更喜欢只读而不是可变集合。
-  明智地设计你的可变点，不要产生不必要的可变点。
-  不要暴露可变对象。

这些规则有一些例外。 有时我们更喜欢可变对象，因为它们更有效。 只有在我们代码的性能关键部分（第 3 部分：效率）中才应该首选此类优化，当我们使用它们时，我们需要记住，在为多线程准备可变性时需要更多注意。 基线是我们应该限制可变性。
