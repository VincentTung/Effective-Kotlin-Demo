42.遵守compareTo的约定

  compareTo方法不是Any的方法，是kotlin的操作符  > 、 <、  >= 、<=
  实现compareTo方法（借助于compareValuesBy方法来对比多个属性）
```kotlin
class User(
    private val name: String,
    private val surname: String
) : Comparable<User> {
    override fun compareTo(other: User): Int =
        compareValuesBy(this, other, { it.surname }, {
            it.name
        })
}
```
compareTo方法返回值定义：

- 如果a = b,返回0
- 如果a>b，返回正数
- 如果a<b, 返回负数
