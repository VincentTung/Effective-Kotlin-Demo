第4条：不要暴露推断的类型(inferred types)

Kotlin的类型推断是它的亮点之一，以至于Java 10也引入了类型推断。然而使用这个特性却有许多需要注意的地方。最重要的是，需要记住,推断出的赋值类型是表达式右边的确切类型 ，而不是父类型或者接口类型：
```kotlin
open class Animal
class Zebra : Animal()

fun main() {
    var animal = Zebra()
    animal = Animal() // 错误，类型不匹配
}
```

大多数情况下，这不是问题。当遇到严苛的类型限定的时候，我们只需要显示声明类型，问题就会解决
```kotlin
open class Animal
class Zebra : Animal()

fun main() {
    var animal:Animal = Zebra() //明确类型
    animal = Animal() 
}
```
当然，如果不是我们自己的库或者其他模块就没有那么舒服了。在这种情况下，推断类型说明会很危险。来看一下这个例子。
假设你有如下一个接口类用来代表汽车工厂：
```kotlin
 interface CarFactory { 
     fun produce(): Car
 }
```
如果未指定其他内容，则还使用默认Car：
```kotlin
val DEFAULT_CAR: Car = Fiat126P()
```
它是由汽车工厂制造的，所以你将它设置为produce的默认值，你省略了类型，因为你认为DEFAULT_CAR任何时候都是一辆汽车
```kotlin
interface CarFactory {
     fun produce() = DEFAULT_CAR
}
```
同样的，后面有人看到DEFAULT_CAR,认为它的类型可以被推断出来
```kotlin
val DEFAULT_CAR = Fiat126P()
```
现在你所有的工厂都只能生产Fiat126P(菲亚特126P)这一种类型的汽车了。这不行啊。如果是你自己定义的接口，这个问题可能会马上被发现并且轻松的解决。但是如果是外部API,你可能会被愤怒的用户通知。
除此之外，当有人不是很了解API的时候，返回类型是重要的帮助信息，也能用来提高可读性，我们应该显示明确它的类型，特别是对外暴露的API.

**总结**：

一般的规则是：如果是我们不确定的类型，就要明确它。这是重要的信息，并且我们不能隐藏它。此外为了安全起见，在外部API中，我们应该明确类型，不能突然的改变类型。当我们的工程进行迭代的时候，可推断类型会变成阻碍或者轻易被改变类型。

