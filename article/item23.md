23.避免参数名和成员变量名重复

重名的函数参数名，会遮蔽成员变量
```kotlin
class Forest(val name: String) {
    fun addTree(name: String) {
     // ...
    } 
}
```
