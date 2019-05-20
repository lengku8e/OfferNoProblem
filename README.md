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

1.接口的意义
规范、扩展、回调

2.抽象类的意义
为其子类提供一个公共的类型 封装子类中得重复内容 定义抽象方法，子类虽然有不同的实现 但是定义是一致的

3.内部类的作用
* 内部类可以用多个实例，每个实例都有自己的状态信息，并且与其他外围对象的信息相互独立。
*   在单个外围类中，可以让多个内部类以不同的方式实现同一个接口，或者继承同一个类。
*   创建内部类对象的时刻并不依赖于外围类对象的创建。
*   内部类并没有令人迷惑的“is-a”关系，他就是一个独立的实体。
*   内部类提供了更好的封装，除了该外围类，其他类都不能访问

4.父类的静态方法能否被子类重写，为什么？
不能。子类继承父类后，用相同的静态方法和非静态方法，这时非静态方法覆盖父类中的方法（即方法重写），父类的该静态方法被隐藏（如果对象是父类则调用该隐藏的方法），另外子类可继承父类的静态与非静态方法，至于方法重载我觉得它其中一要素就是在同一类中，不能说父类中的什么方法与子类里的什么方法是方法重载的体现

5.举1-2个排序算法，并使用java代码实现（美团）

[排序算法](http://blog.csdn.net/qy1387/article/details/7752973)

6.列举java的集合的继承关系
 ![](https://github.com/lengku8e/OfferNoProblem/blob/master/app/src/main/res/mipmap-xhdpi/collection.png)

7.java虚拟机的特性

Java语言的一个非常重要的特点就是与平台的无关性。而使用Java虚拟机是实现这一特点的关键。一般的高级语言如果要在不同的平台上运行，至少需要编译成不同的目标代码。而引入Java语言虚拟机后，Java语言在不同平台上运行时不需要重新编译。Java语言使用模式Java虚拟机屏蔽了与具体平台相关的信息，使得Java语言编译程序只需生成在Java虚拟机上运行的目标代码（字节码），就可以在多种平台上不加修改地运行。Java虚拟机在执行字节码时，把字节码解释成具体平台上的机器指令执行。

8.哪些情况下的对象会被垃圾回收机制处理掉

Java 垃圾回收机制最基本的做法是分代回收。内存中的区域被划分成不同的世代，对象根据其存活的时间被保存在对应世代的区域中。一般的实现是划分成3个世代：年轻、年老和永久。内存的分配是发生在年轻世代中的。当一个对象存活时间足够长的时候，它就会被复制到年老世代中。对于不同的世代可以使用不同的垃圾回收算法。进行世代划分的出发点是对应用中对象存活时间进行研究之后得出的统计规律。一般来说，一个应用中的大部分对象的存活时间都很短。比如局部变量的存活时间就只在方法的执行过程中。基于这一点，对于年轻世代的垃圾回收算法就可以很有针对性。


9.进程和线程的区别
简而言之,一个程序至少有一个进程,一个进程至少有一个线程。
线程的划分尺度小于进程，使得多线程程序的并发性高。
另外，进程在执行过程中拥有独立的内存单元，而多个线程共享内存，从而极大地提高了程序的运行效率。
线程在执行过程中与进程还是有区别的。每个独立的线程有一个程序运行的入口、顺序执行序列和程序的出口。但是线程不能够独立执行，必须依存在应用程序中，由应用程序提供多个线程执行控制。
从逻辑角度来看，多线程的意义在于一个应用程序中，有多个执行部分可以同时执行。但操作系统并没有将多个线程看做多个独立的应用，来实现进程的调度和管理以及资源分配。这就是进程和线程的重要区别。
进程是具有一定独立功能的程序关于某个数据集合上的一次运行活动,进程是系统进行资源分配和调度的一个独立单位.
线程是进程的一个实体,是CPU调度和分派的基本单位,它是比进程更小的能独立运行的基本单位.线程自己基本上不拥有系统资源,只拥有一点在运行中必不可少的资源(如程序计数器,一组寄存器和栈),但是它可与同属一个进程的其他的线程共享进程所拥有的全部资源.
一个线程可以创建和撤销另一个线程;同一个进程中的多个线程之间可以并发执行.
进程和线程的主要差别在于它们是不同的操作系统资源管理方式。进程有独立的地址空间，一个进程崩溃后，在保护模式下不会对其它进程产生影响，而线程只是一个进程中的不同执行路径。线程有自己的堆栈和局部变量，但线程之间没有单独的地址空间，一个线程死掉就等于整个进程死掉，所以多进程的程序要比多线程的程序健壮，但在进程切换时，耗费资源较大，效率要差一些。但对于一些要求同时进行并且又要共享某些变量的并发操作，只能用线程，不能用进程。如果有兴趣深入的话，我建议你们看看《现代操作系统》或者《操作系统的设计与实现》。对就个问题说得比较清楚。

10.Java中==和equals的区别，equals和hashCode的区别

[Java中==和equals的区别，equals和hashCode的区别](http://blog.csdn.net/tiantiandjava/article/details/46988461)

11.常见的排序算法时间复杂度（小米）

 ![](https://github.com/lengku8e/OfferNoProblem/blob/master/app/src/main/res/mipmap-xhdpi/time.png)

   
12.HashMap的实现原理（美团）
*   HashMap概述：    HashMap是基于哈希表的Map接口的非同步实现。此实现提供所有可选的映射操作，并允许使用null值和null键。此类不保证映射的顺序，特别是它不保证该顺序恒久不变。
*   HashMap的数据结构： 在java编程语言中，最基本的结构就是两种，一个是数组，另外一个是模拟指针（引用），所有的数据结构都可以用这两个基本结构来构造的，HashMap也不例外。HashMap实际上是一个“链表散列”的数据结构，即数组和链表的结合体。HashMap底层就是一个数组结构，数组中的每一项又是一个链表。当新建一个HashMap的时候，就会初始化一个数组。

13.java state模式

[state模式](http://www.jdon.com/designpatterns/designpattern_State.htm)

14.java中int char long各占多少字节数

byte 位数 8 字节数 1

short 16 2

int 32 4

long 64 8

float 32 4

double 64 8

char 16 2


15.java int与integer的区别

[int与integer的区别](http://www.cnblogs.com/shenliang123/archive/2011/10/27/2226903.html)

16.string stringbuffer stringbuilder 区别

* String 字符串常量
StringBuffer 字符串变量（线程安全）
StringBuilder 字符串变量（非线程安全）
简要的说， String 类型和 StringBuffer 类型的主要性能区别其实在于 String 是不可变的对象, 因此在每次对 String 类型进行改变的时候其实都等同于生成了一个新的 String 对象，然后将指针指向新的 String 对象，所以经常改变内容的字符串最好不要用String ，因为每次生成对象都会对系统性能产生影响，特别当内存中无引用对象多了以后,JVM 的 GC 就会开始工作，那速度是一定会相当慢的。
而如果是使用 StringBuffer 类则结果就不一样了，每次结果都会对 StringBuffer 对象本身进行操作，而不是生成新的对象，再改变对象引用。所以在一般情况下我们推荐使用 StringBuffer ，特别是字符串对象经常改变的情况下。而在某些特别情况下， String 对象的字符串拼接其实是被 JVM 解释成了 StringBuffer 对象的拼接，所以这些时候 String 对象的速度并不会比 StringBuffer 对象慢，而特别是以下的字符串对象生成中， String 效率是远要比 StringBuffer 快的：
String S1 = “This is only a” + “ simple” + “ test”;
StringBuffer Sb = new StringBuilder(“This is only a”).append(“ simple”).append(“ test”); 你会很惊讶的发现，生成 String S1 对象的速度简直太快了，而这个时候 StringBuffer 居然速度上根本一点都不占优势。其实这是 JVM 的一个把戏，在 JVM 眼里，这个  String S1 = “This is only a” + “ simple” + “test”; 其实就是：  String S1 = “This is only a simple test”; 所以当然不需要太多的时间了。但大家这里要注意的是，如果你的字符串是来自另外的 String 对象的话，速度就没那么快了，譬如：  String S2 = “This is only a”; String S3 = “ simple”; String S4 = “ test”; String S1 = S2 +S3 + S4; 这时候 JVM 会规规矩矩的按照原来的方式去做
在大部分情况下 StringBuffer > String
* StringBuffer
Java.lang.StringBuffer线程安全的可变字符序列。一个类似于 String 的字符串缓冲区，但不能修改。虽然在任意时间点上它都包含某种特定的字符序列，但通过某些方法调用可以改变该序列的长度和内容。
可将字符串缓冲区安全地用于多个线程。可以在必要时对这些方法进行同步，因此任意特定实例上的所有操作就好像是以串行顺序发生的，该顺序与所涉及的每个线程进行的方法调用顺序一致。
StringBuffer 上的主要操作是 append 和 insert 方法，可重载这些方法，以接受任意类型的数据。每个方法都能有效地将给定的数据转换成字符串，然后将该字符串的字符追加或插入到字符串缓冲区中。append 方法始终将这些字符添加到缓冲区的末端；而 insert 方法则在指定的点添加字符。
例如，如果 z 引用一个当前内容是“start”的字符串缓冲区对象，则此方法调用 z.append("le") 会使字符串缓冲区包含“startle”，而 z.insert(4, "le") 将更改字符串缓冲区，使之包含“starlet”。
在大部分情况下 StringBuilder > StringBuffer
java.lang.StringBuilder
java.lang.StringBuilder一个可变的字符序列是5.0新增的。此类提供一个与 StringBuffer 兼容的 API，但不保证同步。该类被设计用作 StringBuffer 的一个简易替换，用在字符串缓冲区被单个线程使用的时候（这种情况很普遍）。如果可能，建议优先采用该类，因为在大多数实现中，它比 StringBuffer 要快。两者的方法基本相同。

17.Java多态、Java多态性理解、Java中多态性的实现、什么是多态

面向对象的三大特性：封装、继承、多态。从一定角度来看，封装和继承几乎都是为多态而准备的。这是我们最后一个概念，也是最重要的知识点。
多态的定义：指允许不同类的对象对同一消息做出响应。即同一消息可以根据发送对象的不同而采用多种不同的行为方式。（发送消息就是函数调用）
实现多态的技术称为：动态绑定（dynamic binding），是指在执行期间判断所引用对象的实 际类型，根据其实际的类型调用其相应的方法。
多态的作用：消除类型之间的耦合关系。

现实中，关于多态的例子不胜枚举。比方说按下 F1 键这个动作，如果当前在 Flash 界面下弹出的就是 AS 3 的帮助文档；如果当前在 Word 下弹出的就是 Word 帮助；在 Windows 下弹出的就是 Windows 帮助和支持。同一个事件发生在不同的对象上会产生不同的结果。 下面是多态存在的三个必要条件，要求大家做梦时都能背出来！

多态存在的三个必要条件 一、要有继承； 二、要有重写； 三、父类引用指向子类对象。

多态的好处：

* 可替换性（substitutability）。多态对已存在代码具有可替换性。例如，多态对圆Circle类工作，对其他任何圆形几何体，如圆环，也同样工作。
* 可扩充性（extensibility）。多态对代码具有可扩充性。增加新的子类不影响已存在类的多态性、继承性，以及其他特性的运行和操作。实际上新加子类更容易获得多态功能。例如，在实现了圆锥、半圆锥以及半球体的多态基础上，很容易增添球体类的多态性。
* 接口性（interface-ability）。多态是超类通过方法签名，向子类提供了一个共同接口，由子类来完善或者覆盖它而实现的.
* 灵活性（flexibility）。它在应用中体现了灵活多样的操作，提高了使用效率。
* 简化性（simplicity）。多态简化对应用软件的代码编写和修改过程，尤其在处理大量对象的运算和操作时，这个特点尤为突出和重要。
Java中多态的实现方式：接口实现，继承父类进行方法重写，同一个类中进行方法重载。

18.什么导致线程阻塞

线程的阻塞
为了解决对共享存储区的访问冲突，Java 引入了同步机制，现在让我们来考察多个线程对共享资源的访问，显然同步机制已经不够了，因为在任意时刻所要求的资源不一定已经准备好了被访问，反过来，同一时刻准备好了的资源也可能不止一个。为了解决这种情况下的访问控制问题，Java 引入了对阻塞机制的支持.
阻塞指的是暂停一个线程的执行以等待某个条件发生（如某资源就绪），学过操作系统的同学对它一定已经很熟悉了。Java 提供了大量方法来支持阻塞，下面让我们逐一分析。

*   sleep() 方法：sleep() 允许 指定以毫秒为单位的一段时间作为参数，它使得线程在指定的时间内进入阻塞状态，不能得到CPU 时间，指定的时间一过，线程重新进入可执行状态。 典型地，sleep() 被用在等待某个资源就绪的情形：测试发现条件不满足后，让线程阻塞一段时间后重新测试，直到条件满足为止。
*   suspend() 和 resume() 方法：两个方法配套使用，suspend()使得线程进入阻塞状态，并且不会自动恢复，必须其对应的resume() 被调用，才能使得线程重新进入可执行状态。典型地，suspend() 和 resume() 被用在等待另一个线程产生的结果的情形：测试发现结果还没有产生后，让线程阻塞，另一个线程产生了结果后，调用 resume() 使其恢复。
*   yield() 方法：yield() 使得线程放弃当前分得的 CPU 时间，但是不使线程阻塞，即线程仍处于可执行状态，随时可能再次分得 CPU 时间。调用 yield() 的效果等价于调度程序认为该线程已执行了足够的时间从而转到另一个线程.
*   wait() 和 notify() 方法：两个方法配套使用，wait() 使得线程进入阻塞状态，它有两种形式，一种允许 指定以毫秒为单位的一段时间作为参数，另一种没有参数，前者当对应的 notify() 被调用或者超出指定时间时线程重新进入可执行状态，后者则必须对应的 notify() 被调用.
初看起来它们与 suspend() 和 resume() 方法对没有什么分别，但是事实上它们是截然不同的。区别的核心在于，前面叙述的所有方法，阻塞时都不会释放占用的锁（如果占用了的话），而这一对方法则相反。
上述的核心区别导致了一系列的细节上的区别。
首先，前面叙述的所有方法都隶属于 Thread 类，但是这一对却直接隶属于 Object 类，也就是说，所有对象都拥有这一对方法。初看起来这十分不可思议，但是实际上却是很自然的，因为这一对方法阻塞时要释放占用的锁，而锁是任何对象都具有的，调用任意对象的 wait() 方法导致线程阻塞，并且该对象上的锁被释放。而调用 任意对象的notify()方法则导致因调用该对象的 wait() 方法而阻塞的线程中随机选择的一个解除阻塞（但要等到获得锁后才真正可执行）。
其次，前面叙述的所有方法都可在任何位置调用，但是这一对方法却必须在 synchronized 方法或块中调用，理由也很简单，只有在synchronized 方法或块中当前线程才占有锁，才有锁可以释放。同样的道理，调用这一对方法的对象上的锁必须为当前线程所拥有，这样才有锁可以释放。因此，这一对方法调用必须放置在这样的 synchronized 方法或块中，该方法或块的上锁对象就是调用这一对方法的对象。若不满足这一条件，则程序虽然仍能编译，但在运行时会出现IllegalMonitorStateException 异常。
wait() 和 notify() 方法的上述特性决定了它们经常和synchronized 方法或块一起使用，将它们和操作系统的进程间通信机制作一个比较就会发现它们的相似性：synchronized方法或块提供了类似于操作系统原语的功能，它们的执行不会受到多线程机制的干扰，而这一对方法则相当于 block 和wakeup 原语（这一对方法均声明为 synchronized）。它们的结合使得我们可以实现操作系统上一系列精妙的进程间通信的算法（如信号量算法），并用于解决各种复杂的线程间通信问题。
* 关于 wait() 和 notify() 方法最后再说明两点：
第一：调用 notify() 方法导致解除阻塞的线程是从因调用该对象的 wait() 方法而阻塞的线程中随机选取的，我们无法预料哪一个线程将会被选择，所以编程时要特别小心，避免因这种不确定性而产生问题。
第二：除了 notify()，还有一个方法 notifyAll() 也可起到类似作用，唯一的区别在于，调用 notifyAll() 方法将把因调用该对象的 wait() 方法而阻塞的所有线程一次性全部解除阻塞。当然，只有获得锁的那一个线程才能进入可执行状态。
谈到阻塞，就不能不谈一谈死锁，略一分析就能发现，suspend() 方法和不指定超时期限的 wait() 方法的调用都可能产生死锁。遗憾的是，Java 并不在语言级别上支持死锁的避免，我们在编程中必须小心地避免死锁。
以上我们对 Java 中实现线程阻塞的各种方法作了一番分析，我们重点分析了 wait() 和 notify() 方法，因为它们的功能最强大，使用也最灵活，但是这也导致了它们的效率较低，较容易出错。实际使用中我们应该灵活使用各种方法，以便更好地达到我们的目的。

19.抽象类接口区别

*   默认的方法实现 抽象类可以有默认的方法实现完全是抽象的。接口根本不存在方法的实现
*   实现 子类使用extends关键字来继承抽象类。如果子类不是抽象类的话，它需要提供抽象类中所有声明的方法的实现。 子类使用关键字implements来实现接口。它需要提供接口中所有声明的方法的实现
*   构造器 抽象类可以有构造器 接口不能有构造器
*   与正常Java类的区别 除了你不能实例化抽象类之外，它和普通Java类没有任何区 接口是完全不同的类型
*   访问修饰符 抽象方法可以有public、protected和default这些修饰符 接口方法默认修饰符是public。你不可以使用其它修饰符。
*   main方法 抽象方法可以有main方法并且我们可以运行它 接口没有main方法，因此我们不能运行它。
*   多继承 抽象类在java语言中所表示的是一种继承关系，一个子类只能存在一个父类，但是可以存在多个接口。
*   速度 它比接口速度要快 接口是稍微有点慢的，因为它需要时间去寻找在类中实现的方法。
*   添加新方法 如果你往抽象类中添加新的方法，你可以给它提供默认的实现。因此你不需要改变你现在的代码。 如果你往接口中添加方法，那么你必须改变实现该接口的类。


20.容器类之间的区别

[容器类之间的区别](http://www.cnblogs.com/yuanermen/archive/2009/08/05/1539917.html)

[容器类之间的区别]
(http://www.cnblogs.com/yuanermen/archive/2009/08/05/1539917.html http://alexyyek.github.io/2015/04/06/Collection/http://tianmaying.com/tutorial/java_collection)

21.java 内部类

[java 内部类](http://www.cnblogs.com/chenssy/p/3388487.html)

22.Java中hashmap和hashtable的区别

[hashmap和hashtable的区别](http://www.233.com/ncre2/JAVA/jichu/20100717/084230917.html)

23.ArrayMap VS HashMap

[hashmap和hashtable的区别](http://lvable.com/?p=217)





### 资料
java四大名著 [链接:https://pan.baidu.com/s/1ip4VEdbCDp0FV3G0Cr5CYw  密码:msx5](java四大名著)


## 网络


## 数据结构


## android架构


## 设计模式


## 性能调优


## 跨平台


## 三方开源库原理及使用



