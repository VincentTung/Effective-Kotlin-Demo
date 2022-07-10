38：使用函数类型来替代接口传递操作

使用函数类型来传递操作
```kotlin
//使用接口 
fun setOnClickListener(listener: OnClick) {
         //...
    }

//使用函数类型
fun setOnClickListener(listener: (View) -> Unit) {
         //...
    }
//调用
setOnClickListener { /*...*/ } 
setOnClickListener(fun(view) { /*...*/ })
setOnClickListener(::println)
setOnClickListener(this::showUsers)
```
