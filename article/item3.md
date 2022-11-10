第3条：尽快消除平台类型（ platform types）
 
kotlin中引入空安全机制是令人惊喜的。Java因为空指针异常（NPE）而被开发社区熟知，而Kotlin的空安全机制使得NPE很少或者完全不会发生。没有什么事情是绝对安全的，一门语言也很难保证有充足的空安全-比如Java、C 、Kotlin。设想一下，在java中我们定义了返回值为String类型的方法。那么在Kotlin中应该使用什么类型？
如果使用了@Nullable注解，我们假设它是可空的，那么就使用String?.如果使用了@NotNull注解，那么我们信任这个注解，就使用String。但是，如果返回的类型没有注解或者使用的不是这些注解，应该怎么办呢？
```java
// Java
public class JavaTest { 
    public String giveName() {
        // ...
    } 
}
```
人们可能会说应该把它看做是可空类型。这是一种安全的做法，因为在Java中，任何东西都是可空的。然而，我们经常知道一些东西不是null,所以需要代码中的很多地方在尾部使用不为空的符号!!。
真正的问题是什么时候我们需要从Java中获取泛型。假设一个Java API返回了 一个List<User>,且没有注解。如果Kotlin默认假设是可空的类型，我们又知道list和这些user不是空的，那么不仅需要使用!!，还要过滤空值：
```java
 // Java
public class UserRepo { 
    
    public List<User> getUsers() {
    //***
    }  
}
```
```kotlin
// Kotlin
val users: List<User> = UserRepo().users!!.filterNotNull()
```
如果换成是一个函数返回的是一个List<List<User>>呢?
情况变得复杂了：
```kotlin
val users: List<List<User>> = UserRepo().groupedUsers!!.map { it!!.filterNotNull() }
```
List至少有像map和filterNotNull的函数。其他泛型，可空性(nullability)将会是一个大问题。这就是为什么被默认当做是可空类型，来自Java的一个类型，具有未知的可空性，在kotlin中是一种特殊的类型。它被称作是平台类型。

**平台类型**-来自另外一种语言的类型且具有未知的可空性。

平台类型被单个感叹号标记!，在类型后面，比如 String!。可是这种标记不能用于代码中。平台类型是无指示性的-意思是不能在语言中明确的把它们写下来。当一个平台类型赋值给了一个Kotlin的变量或者属性时，它可以被推断出来，但是不能被明确的设置。反而，我们可以选择期望的类型：可空类型或者不可空类型.
```java
// Java
public class UserRepo { 
     public User getUser() {
    //...
    } 
}
```
```kotlin
// Kotlin
val repo = UserRepo()
val user1 = repo.user // Type of user1 is User!
val user2: User = repo.user // Type of user2 is User
val user3: User? = repo.user // Type of user3 is User?
```
多亏了这种事实，从Java中获取泛型不再是问题：
```kotlin
val users: List<User> = UserRepo().users
val users: List<List<User>> = UserRepo().groupedUsers
```
问题是因为我们假设的某些不为空的值可能是空的，所以仍然存在危险。这就是为什么出于安全的原因，我总是建议当从Java中拿到平台类型的时候务必要认真处理。记住即使一个函数现在没有返回空值，也不意味着将来不会。如果一个设计者没有使用注解明确或者注释描述，他们可能会发生改变并且没有任何声明。
如果你有一些需要和Kotlin交互的Java代码，尽可能的引入@Nullable 和@NotNull
```java
// Java
import org.jetbrains.annotations.NotNull;

public class UserRepo {
    public @NotNull User getUser() {
     //...
    } 
}
```
当我们想去更好的支持Kotlin开发者的时候，这是最重要的一步。在Kotlin成为第一开发语言后，在Android API中引入注解许多类型是最重要的变化。这使得Android API Kotlin友好化。
注意 支持许多不同的注解，包括如下：

- JetBrains (@Nullable and @NotNull from org.jetbrains.annotations) 
- Android (@Nullable and @NonNull from androidx.annotation as well as from com.android.annotations and from the support library android.support.annotations) 
- JSR-305 (@Nullable, @CheckForNull and @Nonnull from javax.annotation) 
- JavaX (@Nullable, @CheckForNull, @Nonnull from javax.annotation) 
- FindBugs (@Nullable, @CheckForNull, @PossiblyNull and @NonNull from edu.umd.cs.findbugs.annotations) 
- ReactiveX (@Nullable and @NonNull from io.reactivex.annotations) 
- Eclipse (@Nullable and @NonNull from org.eclipse.jdt.annotation) 
- Lombok (@NonNull from lombok)

或者，你可以使用 using JSR 305的@ParametersAreNonnullByDefault 的注解指定在Java所有的类型默认都是不为空的。
在Kotlin代码中，同样也有一件事情可以做。出于安全的原因，我推荐尽可能快的剔除平台类型。为了理解为什么，思考一下statedType和platformType函数在例子中的不同处：
```java
 // Java
public class JavaClass {
        public String getValue() {
            return null; 
        } 
}
```
```kotlin
// Kotlin
fun statedType() {
    val value: String = JavaClass().value
    //...
    println(value.length)
}

fun platformType() {
    val value = JavaClass().value
    //...
    println(value.length)
}
```
上面的两种情况，开发者都假设getValue不会返回空值，但是这是错误的。结果是两个函数都会出现NPE,但是两个发生错误的地方却有不同。
在statedType中，NPE将会在我们从Java获取值的同一个被抛出。这很明显，错误的声明了一个不为空的类型，且拿到了空值。只需要修改它，调整剩余的代码部分。
在platformType中，NPE将在我们把值当做不为空来使用的时候抛出。可能是从一些更复杂的表达式的中间部分开始的。被当做是平台类型的变量可以被当做可空、亦或不可空来对待。这种变量可能前几次使用是安全的，然后就变得不安全，抛出了NPE.当我们使用这种属性的时候，类型系统并不能帮助我们。Java中也是同样的情况，但是在Kotlin中我们并不希望在使用一个对象的时候可能会产生NPE。很有可能有人迟早会不安全的使用它，最后因为一个运行时的异常而停止，而导致这些的原因也可能不会轻易被发现。
```kotlin
// Java
public class JavaClass {
    public String getValue() {
        return null; 
    } 
}
```
```kotlin
// Kotlin

fun statedType() {
    val value: String = JavaClass().value // NPE
    //...
    println(value.length)
}

fun platformType() {

    val value = JavaClass().value
    //...
    println(value.length) // NPE
}

```
更为危险的是，平台类型可以被传播的更深。例如，在接口中想要暴露一个平台类型：
```kotlin
interface UserRepo {
    fun getUserName() = JavaClass().value
}
```
这种情况，方法的返回类型被推断为平台类型。意味着，任意使用者仍然可以决定它是空或者不是。一个使用者可能在某处定义它可空的，而另一个人可能定义为不可空：
```kotlin
class RepoImpl: UserRepo {
    override fun getUserName(): String? {
        return null
    }
}

fun main() {
    val repo: UserRepo = RepoImpl()
    val text: String = repo.getUserName() // NPE in runtime
    print("User name length is ${text.length}")
}
```
传递平台类型是造成灾难的根源。这将是有问题的，出于安全，应该尽可能的剔除出去。这种情况，IDEA Intellij用一个warning来帮助我们：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/10378482/1655542189060-3e1cc09e-2ce2-46e6-92e4-462752cd411c.png#clientId=uf1e4d90f-2da5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=78&id=euH9K&margin=%5Bobject%20Object%5D&name=image.png&originHeight=97&originWidth=925&originalType=binary&ratio=1&rotation=0&showTitle=false&size=29321&status=done&style=none&taskId=ubbeec932-a16b-4b90-9f66-d1b51686718&title=&width=740)

**总结：**

来自另一种语言的类型拥有未知的可空性，被称作是平台类型。由于它们是危险的，我们应该尽可能的剔除它们，不能传递它们。在暴露的Java中的构造函数、方法、变量中使用注解来指定可空性，也是很好的做法。这对使用这些元素的Java和Kotlin开发者来说都将是宝贵的信息。
