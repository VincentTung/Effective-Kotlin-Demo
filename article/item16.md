16.属性应该描述状态，而不是行为

kotlin的属性看起来和java的成员变量相似，但是确代表了不同的概念
```kotlin
//var类型 读写属性， 有setter getter方法，通过filed指向这个属性存储的数据
var name: String? = null
    get() = field?.uppercase(Locale.getDefault())
    set(value) {
        if (!value.isNullOrBlank()) {
            field = value
        }
    }
//val类型 只读属性 ，只有getter方法
val fullName: String? = null
    get() {
        if (field?.isEmpty() == true) return "none"
        return field
    }

```
属性扩展
```kotlin
val Context.inflater: LayoutInflater
    get() {
        return getSystemService(Context.LAYOUT_INFLATER_SERVICE) as LayoutInflater
    }
```
属性代表了访问方式，而不是成员变量。 setter、getter里面应该只是对状态的操作，而不应该是负责的业务逻辑
那些情况下应该使用函数而不是属性？

- 操作复杂度超过O(1)
- 牵扯到业务逻辑
- 存在不确定性
- 涉及到类型转换 Int.toDouble()
- getter方法不应该改变状态
