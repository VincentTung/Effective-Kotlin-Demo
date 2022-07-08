第7条：当可能缺少返回值时，首选null返回值或者Failure返回值

有时候，一个函数不能产生它想要的返回值。常见的情况如下：

- 尝试从服务器获取数据，但是现在网络连接有问题
- 尝试获取第一个符合标准的元素，但是列表中没有合适的元素
- 尝试解析文本获取实例 ，但是文本是错误的格式。

有两种机制来处理上述情况：

- 返回null值或者一个表示失败的密封类（常常起名为Failure）
- 抛出一个异常Exception

在上面两点中有个很重要的区别.异常不是用来传递消息的一种标准方式。所有的异常表明了不正确、特殊的情况，应当被处理。我们应该只对特殊的例外的情况使用异常。主要的原因如下：

- 对大部分的编程者来说，exception传播的方式可读性差，而且很容易在代码中被忽略。
- 在Kotlin中的所有的异常都是未被检查的。用户没有被强迫、甚至鼓励取处理异常。它们通常没有很好的被文档描述。当我们使用api时，它们并不是真正可见的。
- 因为异常是为特殊情况设计的，所以JVM实现者几乎没有动力使它们像显式测试一样快
- 将代码防止try-catch里面会抑制编译器可能执行的一些优化

从另一方讲，null或者Failure都是用来表明预期错误的绝佳方式。它们是易懂、高效的且能够用常用方式来处理的。这就是为什么当错误是预期的时候，我们应该倾向于返回null 或者Failure，当错误不在预期内，应该抛出异常。下面看例子
```kotlin
inline fun <reified T> String.readObjectOrNull(): T? {
    //...
    if (incorrectSign) {
        return null
    }  //...
    return result
}

inline fun <reified T> String.readObject(): Result<T> {
    //...
    if (incorrectSign) {
        return Failure(JsonParsingException())
    }
    //...
    return Success(result)
}

sealed class Result<out T>
class Success<out T>(val result: T) : Result<T>()
class Failure(val throwable: Throwable) : Result<Nothing>()

class JsonParsingException : Exception()

```

用这种方式表明错误，更容易处理，也更难忽略。当选择返回null值的时候，处理这些值的调用者可以用各种空安全操作，比如 空安全调用或者Elvis操作符

```kotlin
val age = userText.readObjectOrNull<Person>()?.age ?: -1
```
当我们决定返回 像Result一样的 联合类型的时候，使用者要用when表达式来处理
```kotlin
val personResult = userText.readObject<Person>()
val age = when(personResult) {
    is Success -> personResult.value.age
    is Failure -> -1 
}
```
用这种方式来进行错误处理，不仅仅比时间用try-catch高效，而且也更易使用和理解。一个异常会被错过处理，并停止整个程序。然而 null值或者密封的result类需要被明确的处理，但是不会打断程序流。
对比 空值和 密封result类，当我们在处理失败的情况需要传递额外的消息时候，更倾向于密封的reslut类，然后才是null值。记住Failure能够携带你需要的任何数据。（更容易扩展？？）

常见的有两种类型的函数：一种预期会发生错误，另一种将错误当做意外情况进行对待。List就是一个很好的例子，两种类型它都有：

- get函数用来获取指定位置的元素，如果取不到值，就会抛出IndexOutOfBoundsException
- getOrNull函数当越界调用的时候返回null

其他相应的操作也有类似的补充，比如 getOrDefault函数在某些情况下是有用的，但是一般能够轻易的被getOrNul和Elvis操作符 ?:代替

这是种好的做法，开发者明白他们可以安全的获取元素，不用被强迫去处理空值，同时，如果有任何疑惑，可以使用getOrNull 并且正确的处理缺少值。
