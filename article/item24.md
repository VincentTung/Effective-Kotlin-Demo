24. 逆变 协变

变化修饰符的作用：泛型约束，确定反省对象之间的关系
不限修饰符、协变修饰符、逆变修饰符
**Variance modifier：out in**
**out** 协变：子类->父类
**in**逆变：父类->子类

 ![image.png](https://cdn.nlark.com/yuque/0/2022/png/10378482/1654079400548-d8bf7512-6662-4bb6-80ca-45ccb221db83.png#clientId=uc42d2995-eb72-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=251&id=u3df9296c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=388&originWidth=660&originalType=url&ratio=1&rotation=0&showTitle=false&size=213038&status=done&style=none&taskId=u0f95be64-4681-4379-93fa-82354255314&title=&width=427)

变化修饰符的位置：

- 声明处：类或者接口定义的地方，影响所有类、接口使用给的地方

   
```kotlin
 // Declaration-side variance modifier
class Box<out T>(val value: T)
```

- 使用处：修饰一个具体的变量

```kotlin
class Box<T>(val value: T)
val boxStr: Box<String> = Box("Str") 
// Use-side variance modifier
val boxAny: Box<out Any> = boxStr
```

有三种变化修饰符：

- 默认修饰符是不变的， 如果在Cup<T>中，类型参数T是不变的并且A是B的子类型，那么Cup<A>和Cup<B>之间没有关系
- out修饰符使得类型参数协变。如果在Cup<T>中，类型参数T是不变的并且A是B的子类型，那么Cup<A>是Cup<B>的子类型。协变类型用在out-position
- in修饰符使得类型参数逆变。如果在Cup<T>中，类型参数T是不变的且A是B的子类型，那么Cup<B>是Cup<A>的子类型。逆变类型用在in-position

在kotlin中

- List、Set、Map的类型参数是协变的(out修饰符)  Array、MutableList、MutableMaps是不变的(无变化修饰符)
- 函数类型参数是逆变的(in修饰符)  返回类型是协变的(out修饰符)
- 使用协变(out修饰符) 作用于返回值
- 使用逆变(in修饰符)作用于参数
