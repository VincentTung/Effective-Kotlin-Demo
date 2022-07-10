36:组合优于继承

继承保持了多态性
继承的缺点：

- 只能继承一个类
- 从父类继承不必要的方法、属性
- 使用父类功能的需要跳转到父类阅读代码来理解工作原理

组合避免了从父类继承不必要的方法，避免了复杂的类继承层次结构，更稳固也更灵活
组合的缺点：

-  破坏多态性：解决方法 使用代理模式
- 用于组合的类需要严格使用
```kotlin
    class CounterSet<T>(
        private val innerSet: MutableSet<T> = mutableSetOf()
    ) : MutableSet<T> by innerSet {
        var elementsAdded: Int = 0
            private set

        override fun add(element: T): Boolean {
            elementsAdded++
            return innerSet.add(element)
        }

        override fun addAll(elements: Collection<T>): Boolean {
            elementsAdded += elements.size
            return innerSet.addAll(elements)
        }
    }
```
