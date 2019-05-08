OfferNoProblem
==============
## 仓库介绍

  这个仓库会分模块更新android面试相关的技术知识，欢迎纠正与更新。
## android
1.Binder介绍

分为Client和Server两个进程，发消息的作为client接受的作为server，使用时Server需要在ServiceManeger中注册
而Client会对ServerManager进行查询，两个进程之间的通信通过binder驱动完成。

2.Binder使用过程

例如有两个进程A（Client）和B(Server),A要访问B进程中的b()方法,因为处于不同进程，无法直接调用，此时需要借助binder实现，
。首先，B需要在ServiceManager中注册，然后，A通过SM获取到一个代理Proxy，该Proxy包含同名b()方法，A可以直接调用Proxy的b()方法，
最后，SM会调用B的b()方法并把结果返回给A。

3.叙述app启动流程

[从源码分析app启动过程](https://www.jianshu.com/p/602aec6f1209)

4.反射有哪些了解？

在反射中，PathClassLoader DexClassLoader 以及其父类 BaseDexClassLoader扮演了重要的角色。他们的基类即是ClassLoader。
###### DexClassLoader和PathClassLoader区别
构造函数不同，DexClassLoader的 optimizedDirectory 字段非空，而PathClassLoader为空。其中 optimizedDirectory 是用来缓存需要加载的dex文件，所以DexClassLoader可以加载外部的dex文件，
PathClassLoader只能加载已经安装的apk的dex

 ![](https://github.com/lengku8e/OfferNoProblem/blob/master/app/src/main/res/mipmap-xhdpi/classLoader.png?raw=true)





 


### 资料
android入门资料 [链接:https://pan.baidu.com/s/1X5nY9EmS5LnQLuH4EXrqBA  密码:7z4x](android入门资料)
## java
### 资料
java四大名著 [链接:https://pan.baidu.com/s/1ip4VEdbCDp0FV3G0Cr5CYw  密码:msx5](java四大名著)


## 网络


## 数据结构


## android架构


## 设计模式


## 性能调优


## 跨平台


## 三方开源库原理及使用



