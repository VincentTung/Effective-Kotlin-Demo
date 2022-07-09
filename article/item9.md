9.使用use关闭资源

一些资源不会自动关闭，当我们不在使用它们的时候需要手动调用一次close方法。Java的标准库中，使用的Kotlin/Jvm中，包含很多这种资源，比如：

- InpurtStream和OutputStream
- java.sql.Connection
- java.io.Reader(FileReader,BufferedReader,CSSParser）
- java.new.Socket和java.util.Scanner

所有这些资源都实现了Closeable接口，Closeable继承了AutoCloseable
问题是在所有的实际情况中，我们需要确保在需要资源的时候调用close方法，这么做是因为资源是很昂贵的并且它们不会轻易的自动关闭。之前，为了确保能够关闭资源，我们一般会使用类似try-finally的代码块，并且调用close，如下

```kotlin

fun countCharactersInFile(path: String): Int {
    val reader = BufferedReader(FileReader(path))
    try {
        return reader.lineSequence().sumBy { it.length }
    } finally {
        reader.close()
    }
}
```
这种结构的代码是负责和不正确的。说它不正确是因为 close方法会抛出错误，且这些错误没有被捕获。如果我们同时遇到在try方法体和finally块中都有错误的情况，只有一个错误会被正确的传播出去。这种表现不是我们期望的。针对这种情况的处理实现，是繁琐的，但是它被提取到了标准库中的use函数。use函数用来合适的关闭资源和处理异常。

```kotlin
fun countCharactersInFile(path: String): Int {
    val reader = BufferedReader(FileReader(path))
    reader.use { reader ->
        return reader.lineSequence().sumBy { it.length }
    }
}
```

receiver当做参数传入lamda,函数可以简化如下

```kotlin
fun countCharactersInFile(path: String): Int {
    BufferedReader(FileReader(path)).use { reader ->
        return reader.lineSequence().sumBy { it.length }
    }
}
```

这种支持通常是应对文件的需要，常见的文件一行行读取的情况，可以使用标准库中的useLine函数。useLine可以提供一行行string的序列，并且在处理完毕的时候关闭底层的reader:

```kotlin
fun countCharactersInFile1(path: String): Int =
    File(path).useLines { lines ->
        return lines.sumBy { it.length }
    }
```
