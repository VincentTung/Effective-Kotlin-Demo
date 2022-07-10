33.考虑工厂方法代替构造函数

 创建类实例的方法不仅仅通过构造函数，还有别的方法，比如工厂方法(Factory Funtions)

 工厂方法创建实例的优点：

- 函数有自己的名字 可以明确构建的含义、用途
- 可以返回子类型
- 每次调用的时候都不必要求都创建一个新的实例  常用于有缓存机制的逻辑。 Connections.createOrNull() 不满足创建条件下 会返回null
- 可以返回不存在对象。  常用于使用注解处理器生成对象的时候
- 在类外定义工厂方法，可以控制它的可见性
- 工厂方法可以是inline的，类型可以泛型化
- 可以构造很复杂的对象
- 构造函数需要理解通用父类的构造函数或者主构造函数，工厂方法可以推迟

  需要清楚的是，工厂方法内部依然需要使用一个构造函数,工厂函数的主要目标是次构造函数

kotlin常见的工厂函数：

- 伴生对象工厂函数 Companion object factory function
- 扩展工厂函数 Extension factory function
- 顶级工厂函数 Top-level factory functions
- Facke constrouctors
- 工厂类的方法  Methods on a factory classes

**伴生对象工厂方法**

```kotlin
//类中 
class MyLinkedList<T>(
        val head: T,
        val tail: MyLinkedList<T>?
    ) {
        companion object {
            fun <T> of(vararg elements: T): MyLinkedList<T>? {
                /*...*/
            }
        }
    }
 
 //接口中定义
  class MyLinkedList<T>(
        val head: T,
        val tail: MyLinkedList<T>?
    ) : MyList<T> {
        // ...
    }

    interface MyList<T> {
        // ...

        companion object {
            fun <T> of(vararg elements: T): MyList<T>? {
                // ...
            }
        }
    }
```
 关于伴生对象方法命名常见命名规则

- **from **- 参数只有一个，其返回相同的类型：  val date: Date = Date.from(instant)
- **of **-  一个聚合函数，有多个参数，返回相同类型： val faceCards: Set<Rank> = EnumSet.of(JACK, QUEEN, KING)
- **valueOf** - from和of的替代 val prime: BigInteger = BigInteger.valueOf(Integer.MAX_- VALUE)
- **instance**或者**getInstance** - 只用于单例模式，通常每次调用返回的实例是相同的
- **createInstance**或者**newInstance**- 和getInstance不同，每次都产生新的实例
- **getType**- 类似getInstance 但是用于不同的类之间，返回类型就是getType名字指定的类型  val fs: FileStore = Files.getFileStore(path)
- **newType**- 类似newInstance但是用于不同的之间 val br: BufferedReader = Files.newBufferedReader(path)

compaion object不一定非要定义在类中，可以用继承的方式定义在类外：

```kotlin
abstract class ActivityFactory {
    abstract fun getIntent(context: Context): Intent
    fun start(context: Context) {
        val intent = getIntent(context)
        context.startActivity(intent)
    }

    fun startForResult(
        activity: Activity, requestCode:
        Int
    ) {
        val intent = getIntent(activity)
        activity.startActivityForResult(
            intent,
            requestCode
        )
    }
}

class Main4Activity : AppCompatActivity() {
    //...
    companion object : ActivityFactory() {
        override fun getIntent(context: Context): Intent =
            Intent(context, MainActivity::class.java)
    }
}
```


