41.遵守hashcode的约定

两个equals的元素，hashcode是一样的
hash函数的要求 1.快 2.不相等的元素返回不同的hash值，或者能够尽量将碰撞限制在最小值
不能将易变得元素用于依赖hash机制的数据结构 ，如set map等
hashcode的约定

- 同一个对象，多次调用hashcode返回值都是一个
- 两个equals的对象，hashcode值相等

在kotlin中，一般在需要自定义equals方法的时候，顺便自定义hashcode,可以借助于Objects.hash()方法
```kotlin
Objects.hash(timeZone, millis)
```
data class会自动实现hashcode方法
