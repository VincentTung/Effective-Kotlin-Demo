第2条：最小化变量的作用域

当我们定义状态的时候，喜欢缩小变量和属性的作用域，通过以下方法：

- 使用局部变量代替属性
- 尽可能的在较小的作用域里使用变量，比如，一个变量仅在循环里使用，那么久将他定义在循环内部

元素的作用域就是一个电脑程序中这个元素可见的一块区域。在Kotlin中，作用域几乎总是有花括号创建的。我们通常能访问作用域外的元素。看下这个例子：

```kotlin
val a = 1 
fun fizz() {
    val b = 2
    print(a + b)
}
val buzz = {
    val c = 3
    print(a + c)
}
// Here we can see a, but not b nor c
```

上面例子中，函数fizz和buzz的作用域里，可以访问外部作用域的变量。然而，在外部作用力，不能访问定义在这些函数里面的变量。下面这个例子是如何限定变量作用域的：

```kotlin
// Bad 
var user: User
for (i in users.indices) {
    user = users[i]
    print("User at $i is $user") 
    } 

// Better
for (i in users.indices) {
    val user = users[i]
    print("User at $i is $user")
}

// Same variables scope, nicer syntax 
for ((i, user) in users.withIndex()) {
    print("User at $i is $user")
}
```

在第一例子中，user变量不仅在for循环中能够访问，外面也能够访问。在第二个、第三个例子中，我们限定user变量的作用域为for循环的内部。
同样的，我们会有作用域嵌套作用域，最好尽可能将变量定义在最小范围内。
为什么我们喜欢这样做有很多原因，但是最重要的一个是：**当我们收紧一个变量的作用域的时候，会使我们的程序更容易追踪和管理**。当我们分析代码的时候，需要思考这里的元素时哪些。越多的元素需要处理，编程就变得更加困难。你的程序越简单，崩溃的可能性就越小。我们喜欢不可变属性或者对象也是这个原因。
**思考一下，对于易变属性，在一个更小的作用域里追踪它们是如何改变的是更加简单的**。这样解释和更改它们的表现是更加简单的。
另一个问题是**拥有较广作用域的变量可能会被别的开发者过度使用**。例如，一个人可以这样解：如果在遍历中，有一个被用来赋值给下一个元素的变量，在循环结束的时候，list的最后一个元素应该持有这个变量的引用。这样解释，会导致可怕的滥用，像在遍历后使用这个变量和最后的元素做一些操作。这将是糟糕的，因为别的尝试去弄懂这个值需要理解整个流程。这增加了不必要的复杂性。
**不论一个变量是只读的或者可读写的，我们总是喜欢这个变量在定义的时候初始化完毕**。不要强迫一个开发者去查看变量是在哪儿定义的。可以使用控制结构语句，比如if,when,try-catch或者Elvis操作符作作为表达式来支持：

```kotlin
// Bad
val user: User
if (hasValue) {
    user = getValue()
}else { 
    user = User()
 }

// Better
val user: User = if(hasValue) {
    getValue()
} else {
    User()
}
```

如果需要设置多个属性，解够声明(destructuring declaations)可以帮到我们：

```kotlin
    // Bad
    fun updateWeather(degrees: Int) {
        val description: String
        val color: Int
        if (degrees < 5) {
            description = "cold" 
            color = Color . BLUE
        } else if (degrees < 23) {
            description = "mild"
            color = Color . YELLOW
        } else {
            description = "hot" 
            color = Color . RED
        }
    // ...
    }

    // Better
    fun updateWeather(degrees: Int) {
        val (description, color) = when {
            degrees < 5 -> "cold" to Color.BLUE
            degrees < 23 -> "mild" to Color.YELLOW
            else  -> "hot" to Color.RED
        }
    // ...
    }
```

最后，太广的变量作用域是危险的。让我们来讨论一个常见的危险.

**捕捉（缓存？？）**

当我教授关于Kotlin协程的内容的时候，其中一练习是实现埃拉托斯特尼筛法来使用使用序列构建起找到质数。算法在概念上很简单：

1. 准备一个数字2开始的List
1. 拿到第一个数字，它是质数
1. 对于剩下的数，筛选出所有能被这个质数整除的数

这个算法的一个简单实现如下：
```kotlin
var numbers = (2..100).toList()
val primes = mutableListOf<Int>()
while (numbers.isNotEmpty()) {
    val prime = numbers.first()
    primes.add(prime)
    numbers = numbers.filter { it % prime != 0 }
} 
print(primes) // [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31,
// 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97]
```

挑战是如何让产生一个潜在的无线质数序列。如果你想要挑战自己，停下来并尝试去实现它。
解决方法可能长这样：

```kotlin
val primes: Sequence<Int> = sequence {
    var numbers = generateSequence(2) { it + 1 } 
    
    while (true) {
        val prime = numbers.first()
        yield(prime)
        numbers = numbers.drop(1)
            .filter { it % prime != 0 }
    }
}

print(primes.take(10).toList())
// [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]
```

然而，总有人想要尝试“优化”它，不在每个每个循环里创建变量，将质数提取成一个易变的变量：

```kotlin
val primes: Sequence<Int> = sequence {
    var numbers = generateSequence(2) { it + 1 } 
    
    var prime: Int
    while (true) {
        prime = numbers.first()
        yield(prime)
        numbers = numbers.drop(1)
            .filter { it % prime != 0 }
    }
}
```

问题是这样实现导致不能正确的运行了。前10个yielded的数字：

```kotlin
print(primes.take(10).toList())
// [2, 3, 5, 6, 7, 8, 9, 10, 11, 12]
```

停下来，尝试去解释原因：
这样的运行结果的原因是我们捕获了prime.因为使用了序列，所以过滤是懒加载的。在每一步中，我们添加越来越的过滤器。在这个“优化过”的版本中，我们总是添加引用易变属性pimary的过滤器。这就是为什么过滤器没有正确运行。所以我们得到了连续的数字，去掉就好了。我们应该意识到意外捕获的问题，因为这种情况会发生 。为了避免这种情况，我们应该避免变量的可变性，并选择更小的变量作用域。
**
总结**

因为很多原因，我们喜欢尽可能将变量定义在小的作用域内。此外，相比var我们更喜欢使用val来定义本地变量。我们总是应该注意在lamda中变量容易被捕获（缓存？？）。这些简单的规则能够从麻烦中拯救我们。
