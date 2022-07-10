第22条：在实现通用算法的时候使用泛型

我们可以给函数传递一个值作为参数，同样，也给你传递一个类型作为类型参数。接收类型参数的函数被称作泛型函数。一个例子就是标准库中的filter函数，它有类型参数T:
```kotlin
inline fun <T> Iterable<T>.filter(
    predicate: (T) -> Boolean
): List<T> {
    val destination = ArrayList<T>()
    for (element in this) {
        if (predicate(element)) {
            destination.add(element)
        }
    }
    return destination
}
```
对于编译器来说，类型参数是有用的，因为它们允许编译器来检查，进一步正确地推断类型，这使得我们的程序更加安全，对开发者更加友好。例如，当使用filter的时候，在lamda表达式里，编译器知道参数的类型和集合的元素类型是一样的，所以这保护了我们合法的使用，IDE给我们有用的提示。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/10378482/1655954732542-982cb12a-3a65-47f1-86fe-d4598c278d6a.png#clientId=uc77660fd-0ffb-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=293&id=u148ed921&margin=%5Bobject%20Object%5D&name=image.png&originHeight=293&originWidth=576&originalType=binary&ratio=1&rotation=0&showTitle=false&size=115375&status=done&style=none&taskId=u6ccc9dce-b378-43f5-8797-17f21b1228b&title=&width=576)

泛型主要被引入到类和接口中,允许创建只有具体类型的集合，像List<String>或者Set<User>.这些类型在编译期间会丢失，但是当我们开发的时候，编译器会强制我们只能传入正确的类型。例如当我们向MutableList<Int>里添加数据的时候只能Int.同时，多亏了它们，当我们从Set<User>中获取元素的时候，编译器知道返回的类型是User.这种方式,类型参数在静态类型语言中帮助我们很多.Kotlin 对尚未被很好理解的泛型提供了强大的支持,根据我的经验，即使是经验丰富的 Kotlin 开发人员也有 他们的知识空白，尤其是关于差异修饰符（variance modifiers）的知识.所以让我们在本文中讨论 Kotlin 泛型最重要的方面,第23条将讲解考虑是使用泛型的差异类型

**泛型约束**

类型参数的一个重要特点是它们能被约束为某一个特定类型的子类型。我们过在冒号后放置超类型设置这个约束。这个类型可以包括前面的类型参数：
```kotlin
fun <T : Comparable<T>> Iterable<T>.sorted(): List<T> {
    /*...*/
}

fun <T, C : MutableCollection<in T>>
Iterable<T>.toCollection(destination: C): C {
        /*...*/
} 

class ListAdapter<T: ItemAdaper>(/*...*/) { /*...*/ }
```
具有约束的一个重要结果是这种情况的实例可以使用该类型提供的所有方法。这样，当T被约束为Iterable<T>的子类型的时候，我们知道可以掉遍历一个T类型的实例，通过迭代器返回的元素将是Int类型的。当我们约束为Comparable<T>的时候，我们知道则会种类型可以进行比较。另一种流行的约束选择就是Any,意味着一个类型，能是任何不为空类型：
```kotlin
inline fun <T, R : Any> Iterable<T>.mapNotNull(
    transform: (T) -> R?
): List<R> {
    return mapNotNullTo(ArrayList<R>(), transform)
}
```
在极少数情况下，我们可能需要设置多个顶级的限制，可以使用 where 来设置更多的约束：
```kotlin
fun <T: Animal> pet(animal: T) where T: GoodTempered {
 /*...*/
} 

// OR

fun <T> pet(animal: T) where T: Animal, T: GoodTempered {
/*...*/
}
```


**总结**

类型参数是Kotlin类型系统中重要的一部分。我们使用它们来实现类型安全的泛型算法或者泛型对象。类型参数可以被约束为某一个特性类型的子类型。这时，我们可以安全使用这个类型体提供的方法。
