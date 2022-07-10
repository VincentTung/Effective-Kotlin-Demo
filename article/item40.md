40.遵守equals的约定

kotlin中所有的类都继承自Any,所有都有以下方法

- equals 
- hashCode 
-  toString

kotlin中有两种类型的比较

- 结构化比较 ，使用equals或者==    ，a ==b 就是 a.equals(b)
- 引用比较， 使用===，比较两个引用是否指向同一个对象

因为继承Any,默认kotlin的equals方法也是判断两个比较的对象是不是同一个（即是不是指向同一个对象），使用data class会自动重写equals方法，实现比较每个成员变量的值是否相同
通常很少需要自己重写equals方法，但是遇到如果只有某个单一属性决定两个对象是否相同，就需要重写
```kotlin
   class User(  val id: Int,
                 val name: String,
                 val surname: String
    ) {
       //userid相同，就认为是同一个用户
        override fun equals(other: Any?): Boolean =
            other is User && other.id == id
        override fun hashCode(): Int = id
    }
```
 关于equals的约定：

- 自反性，对于非空值x: x.equals(x)= true
- 对称性，对于非空值x,y: 如果x.equals(y)= true,那么 y.equals(x)= true
- 传递性：对于非空值x,y: 如果 x.equals(y)=true,y.equals(z)=true,那么x.equals(z) = true
- 一致性：(幂等性)，对于非空值x,y;如果x.equals(y) = true,那么调用多次依然为true
- 非空值.eqausl(空值） = false 

破坏对称性的做法就是在不同类型之间使用equals，所以禁止在不同类型之间使用equals
不同的类型也会破坏传递性
尽量使用data class 的equals，非必要不需要自己实现equals方法
