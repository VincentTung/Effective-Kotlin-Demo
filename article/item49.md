49：数据多次操作的时候使用Sequence

下面文字来自网络，非本书内容  
Iterable和Sequence的区别（[https://zhuanlan.zhihu.com/p/488164661](https://zhuanlan.zhihu.com/p/488164661)）
1.Iterable是即时的、Sequence是惰性的：Iterable会要求尽早的计算结果，因此在多步骤处理链的每一环都h会有新的集合产生；后者会尽可能的延迟计算结果，
     Sequence处理的中间函数不进行任何计算。相反，他们返回一个新Sequence的，用新的操作装饰前一个，
     所有的这些计算都只是在类似toList的终端操作符调用时候执行
     2.区分中间操作符和末端操作符的方式也很简单：如果操作符返回的是一个Sequence类型的数据，它就是中间操作符

**Iterable是计时的，操作调用后会立即处理，Sequence的中间函数不进行计算，只是返回一个新的Sequence，直到调用终端操作符才会执行**
**Iterable每次调用操作符都会产生中间集合，Sequence不会产生，性能好一些。**
**sequence终端操作符：**

- toHashSet
- toList
- toMutableList
- toSet
- associateXXX

什么时候使用sequence？
集合的处理超过一步，且元素个数超过100，用sequece会有40%-60%的性能提升
什么时候sequence不是快的，不宜使用？
 stored操作的时候，collection比sequence块。在sequence避免调用sorted

kotlin Sequence debug插件 Kotlin Sequence Debugger
```kotlin
        //iterable
        (1..100).filter {
            Log.d(tag, "filter:$it")
            it % 5 == 0
        }.map {
            "hello$it"
            Log.d(tag, "map:$it")
        }.take(5)

        //sequence
        (1..100).asSequence().filter {
            Log.d(tag, "filter:$it")
            it % 5 == 0
        }.map {
            "hello$it"
            Log.d(tag, "map:$it")
        }.take(5).toList()
```
