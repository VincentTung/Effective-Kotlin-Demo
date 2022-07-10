37：使用data修饰符来修饰数据集

使用data修饰符，会生成下面几个函数
• toString 
• equals and hashCode 
• copy 
• componentN (component1, component2, etc.)
toStirng方法会输出data class的属性名字和值 例如  
```kotlin
print(player) // Player(id=0, name=Gecko, points=9999)
```
