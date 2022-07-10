28.使API稳定
人们喜欢稳定、标准的API的原因：

- 当API改变时候，开发者需要手动升级代码 
- 开发者需要去学习新的API,学习新的知识

使API稳定的最简单的方式就是 ：如果某些个API不稳定，必须在文档中描述清楚。Semantic Versioning (SemVer)版本系统  MAJOR.MINOR.PATCH.

- MAJOR version : 主版本号 ，进行不兼容API的时候需要修改
- MINOR version: 小版本号，当通过兼容方式增加功能时需要修改
- PATCH verson: 补丁版本，bug修改时修改

当增加主版本时候，小版本号和补丁版本需要置为0。增加小版本号时候，补丁版本需要置为0。0.y.z 是初始的开发版本，这时候所有的公共API都不应该被当做是稳定的。0开头的版本，不应当期望它是稳定的。
不应担忧一直处在beta版本。Kotlin到达1.0版本用来5年。对一门开发语言来说，这是非常用的时期，因为在这期间发生了很多改变。
 	当在稳定的API里面引入新的元素时候，由于元素还不是稳定的，应该首先将他们放在其他分支。当你允许用户使用的时候（合并到主分支，并发布release）,可以使用**Expermental(Expermental废弃了，使用RequireOptIn代替)**注解来标注它们是不稳定的。不用担心将元素保持在实验状态上很长时间。缓慢稳定的方式能够帮助我们设计出好的API.

```kotlin
@RequiresOptIn(level = RequiresOptIn.Level.WARNING)
annotation class ExperimentalNewApi

@ExperimentalNewApi
fun startAPI() {

}
```
当我们需要改变部分稳定的API的时候，为了帮助用户过渡到新的API,使用Deprecated注解来提示
```kotlin

    @Deprecated("Use suspending getUsers instead",
                ReplaceWith("getUsers()"))
    fun getUsers(callback: (List<User>) -> Unit) {
        //...
    }
```
这样可以给用户时间过渡到新的API,这个状态也应该持续较长时间。对于广泛使用的API，可以是几年。最后，在发布主版本release后，就可以移除这些废弃的元素了(Deprecated Element)
