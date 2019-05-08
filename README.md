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



5.Service生命周期（startService与bindService两种启动服务的方式有什么区别）

可以通过context的startService方式启动也可以通过context的bindService方式启动。两种启动方式的生命周期不同。
通过startService()这种方式启动的service，生命周期是这样：调用startService() --> onCreate()--> onStartConmon()--> onDestroy()。这种方式启动的话，需要注意，当我们通过startService被调用以后，多次在调用startService(),onCreate()方法也只会被调用一次，而onStartConmon()会被多次调用，
想停止服务需要调用stopService()的时候，onDestroy()就会被调用，从而销毁服务。如果在Activity中通过startService方法开启一个服务，当Activity退出的时候Service不会销毁，依然在后台运行，只有手动调用stopService或者在应用管理器中关闭Service，服务才会销毁。
通过bindService()方式进行绑定，这种方式绑定service，生命周期走法：bindService-->onCreate()-->onBind()-->unBind()-->onDestroy() 。 onCreate→onBind，只会执行一次，多次调用不生效，Activity退出的时候必须通过unbindService关闭服务，startService结束的时候stopService可以调用多次，只有第一次调用的时候有效，bindService结束的时候unbindService只能调用一次，调用多次应用会抛异常。

6.Service启动流程（区分同进程或非同进程）

非进程：要启动的服务在不同进程中，主要分为五个阶段 1.App向AMS抛出启动Service的消息2.AMS检查启动Service的进程是否存在，如果不存在就先保存Service信息，保存在ServiceRecord对象中，并创建一个新进程
3.新进程启动后，创建ActivityThread对象，并通过AIDL通知AMS，进程启动成功 4.AMS把保存的Service信息发给新进程5.AMS抛出启动服务消息，新进程的ActivityThread接受到消息后调用handleCreateService方法，获取LoadedApk对象(包含了packgeInfo的包信息),然后获取其classLoader反射出service，并调用服务onCreate方法启动服务。

```
 private void handleCreateService(CreateServiceData data) {
        // If we are getting ready to gc after going to the background, well
        // we are back active so skip it.
        unscheduleGcIdler();

        LoadedApk packageInfo = getPackageInfoNoCheck(
                data.info.applicationInfo, data.compatInfo);
        Service service = null;
        try {
            java.lang.ClassLoader cl = packageInfo.getClassLoader();
            service = packageInfo.getAppFactory()
                    .instantiateService(cl, data.info.name, data.intent);
        } catch (Exception e) {
            if (!mInstrumentation.onException(service, e)) {
                throw new RuntimeException(
                    "Unable to instantiate service " + data.info.name
                    + ": " + e.toString(), e);
            }
        }

        try {
            if (localLOGV) Slog.v(TAG, "Creating service " + data.info.name);

            ContextImpl context = ContextImpl.createAppContext(this, packageInfo);
            context.setOuterContext(service);

            Application app = packageInfo.makeApplication(false, mInstrumentation);
            service.attach(context, this, data.info.name, data.token, app,
                    ActivityManager.getService());
            service.onCreate();
            mServices.put(data.token, service);
            try {
                ActivityManager.getService().serviceDoneExecuting(
                        data.token, SERVICE_DONE_EXECUTING_ANON, 0, 0);
            } catch (RemoteException e) {
                throw e.rethrowFromSystemServer();
            }
        } catch (Exception e) {
            if (!mInstrumentation.onException(service, e)) {
                throw new RuntimeException(
                    "Unable to create service " + data.info.name
                    + ": " + e.toString(), e);
            }
        }
    }
```


同进程：1.APP向AMS发送启动Service消息2.服务在AMS中保存3.AMS判断是在同进程启动后直接启动Service。



 


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



