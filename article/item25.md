25.不同平台之间通过提取通用方法来重用

科技公司很少只为一个单独的平台开发应用，他们宁愿为2个或者更多的平台开发产品，如今，这些产品依赖于在不同的平台上运行的一些程序。考虑下客户端和服务端通过网络呼叫进行交互。由于他们需要交互，有些相似的地方是能重用的。同一产品不同平台的实现有更多相同的地方。特别是他们的业务逻辑基本是一样的。这些项目可以从共享代码中显著获利。

**全栈开发**

很多公司基于web开发。他们的产品是网页，但是大多数时候，这些产品需要一个后台应用。在网页上，JavaScirpts是国王,它几乎垄断了这个平台。在后端，一个非常流行的选择就是Java.因为这些语言是不同的，所以常见的是后端和web开发是分开的。然而，事情是可以改变的。现在Ktolin能够代替Java进行后端开发。例如Srping,最流行的Java 框架，Kotlin是一等公民。Kotlin能够被用在任意Java框架，来代替java。同样的也有很多Kotlin后端框架，比如Ktor.这就是为什么很多后端工程从Java移植到了Kotlin。Kotlin厉害的一部分就是可以编译成JavaScript.已经有很多Kotlin/Js库，我们可以使用Kotlin来编写各种的web应用。例如，我们可以使用React框架和Kotlin/JS来编写一个web前端。这样允许我们的后端和web端都使用Kotlin来编写。更好的是，我们可以同时编译成JVM字节码和JavaScript.这些可以共享的部分 ，例如唯一的工具类、API定义、公共的抽象等等，这些我们都可能想要重用。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/10378482/1656033877135-3ac073ad-bc11-47f1-8f3f-0b349a76fb04.png#clientId=uca2ff425-0beb-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=280&id=u71a2361e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=280&originWidth=411&originalType=binary&ratio=1&rotation=0&showTitle=false&size=18582&status=done&style=none&taskId=u5e2a5c9c-0e1e-4b56-bbec-1d7366ae848&title=&width=411)

**移动端开发**

这种能力在移动端更加重要。我们极少只为Android平台开发。有事候，我们可以没有服务器，但同样得需要实现一个iOS应用。为每个编写的应用使用不同语言和工具。最后Android和iOS版本非常相似。它们经常只是设计不同，但是内部的逻辑经常基本一模一样。使用Kotlin的多平台能力，我们能够只需实现一次逻辑，在两个平台上重用。我们可以编写一个通用的模块，在那里实现业务逻辑。无论如何，业务逻辑应该是独立于框架和平台的。共有逻辑可以使用纯Kotlin编写或者使用其他模块，然后它可以被用在不同平台上。这种经验和我们在Android工程里使用相同的部分是相似的。
在Android里，它可以直接使用，因为Android同样使用了Gradle来构建。
对于 iOS，我们使用 Kotlin/Native 将这些通用部分编译为 Objective-C 框架，然后使用 LLVM 编译为原生代码。然后我们可以再Xcdoe中通过Swift或者其他APp代码使用。或者，我们可以使用Kotlin/Native实现我们整个应用。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/10378482/1656035497429-a6375e56-b9dd-4050-b206-f168a672d7a7.png#clientId=uee543791-5a54-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=275&id=ubcb06ae8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=275&originWidth=502&originalType=binary&ratio=1&rotation=0&showTitle=false&size=16685&status=done&style=none&taskId=ua539fa5b-e7b2-4d8b-a547-0dc85de09ff&title=&width=502)

**库**

定义通用模块对于库来说是一个强有力的工具。不是强依赖平台的部分可以被轻松的移动到通用模块，允许用户在所以可以在JVM上运行的语言中调用，JavaScript或者native语言（JavaScript, CoffeeScript, TypeScript, C, Objective-C, Swift, Python,C#等等）

**概括**

我们可以一起使用所有这些平台。使用Kotlin,我们能够为几乎全部的流行的设备和平台开发，并且在它们之间重用我们想要的代码。几个我们能用Kotlin编写例子：

- 后端使用Kotlin/JVM,例如Spring或者Ktor
- 使用Kotlin/JS开发网页，例如React
- Kotlin/JVM开发Android，使用Android SdK
- iOS Frameworks 可以使用OC 或Swfit,使用Kotlin/Native
- 使用Kotlin/JVM的桌面程序，如 TornadoFX
- Kotlin/Native 中的 Raspberry Pi、Linux 或 Mac OS 程序

如下是一个可视化的典型应用程序：

![image.png](https://cdn.nlark.com/yuque/0/2022/png/10378482/1656058711463-2a3033bd-cef5-423a-bba8-2444ac378ca1.png#clientId=uee543791-5a54-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=302&id=u0181b5e2&margin=%5Bobject%20Object%5D&name=image.png&originHeight=302&originWidth=504&originalType=binary&ratio=1&rotation=0&showTitle=false&size=28787&status=done&style=none&taskId=u091c56b0-932b-4c7d-a41c-9f827c08e0d&title=&width=504)

我们现在仍在学习怎么组织我们的代码，使代码能够安全的重用，高效的长时间运行。但很高兴知道这种方法给我们的可能性。我们能够使用通用模块在不同平台间重用。这是一个消除冗余和重用通用逻辑和通用算法的强大工具。
