13：避免返回或者操作Unit?

函数返回Unit?的好处是能够使用Elvis操作符或者进行空安全调用

![image.png](https://cdn.nlark.com/yuque/0/2022/png/10378482/1654675481009-bf5f8e28-f1fb-4e63-be9c-40253ade5a14.png#clientId=u98ba80bc-e735-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=175&id=u5bd98630&margin=%5Bobject%20Object%5D&name=image.png&originHeight=175&originWidth=364&originalType=binary&ratio=1&rotation=0&showTitle=false&size=15769&status=done&style=none&taskId=u7d875a2d-6bc4-446e-a25b-195171901ab&title=&width=364)

但是Unit?降低了可读性，所以尽量避免操作和返回Unit?
