以下分析基于 api 26, 不同的 api, 交互细节不一致, 导致一些类名, 方法明, 字段名不一样, 不过整体的原理是一样的;  
ActivityManagerService 工作在 SystemServer 进程;  
SystemServer 进程 #run 方法 ⤑ #startBootstrapServices 方法 ⤑  启动 ActivityManagerService;  
在 SystemServer.java 的 run方法, ActivityManagerService 向 Native 的 ServiceManager 注册服务;  
ActivityManagerService 是服务端对象, 负责系统中所有 Activity 的生命周期;  
创建 ActivityManager, ProcessRecord, ServiceManager;  
start 方法中, 反射调用 ActivityThread.main方法;  

createSystemContext  
初始化系统上下文对象 mSystemContext, 并设置默认的主题,mSystemContext 实际上是一个 ContextImpl 对象;    
调用 ActivityThread.systemMain()的时候, 会调用 ActivityThread.attach(true), 而在attach()里面,  
则创建了 Application 对象, 并调用了 Application.onCreate();  

内部关系  
AMP 是 AMN 的内部类, 它们都实现了 IActivityManager 接口, 这样它们就可以实现代理模式, 具体来讲是远程代理, AMP 和 AMN 是运行在两个进程的;  
AMP 是 Client 端, AMN 则是 Server 端, 而 Server 端中具体的功能都是由 AMN 的子类 AMS 来实现的, 因此 AMP 就是 AMS 在 Client 端的代理类;   
AMN 又实现了 Binder 类, 这样 AMP 可以和 AMS 就可以通过 Binder 来进行进程间通信;  

ActivityManager 通过 AMN 的 getDefault 方法得到 AMP, 通过 AMP 就可以和 AMN 进行通信, 也就是间接的与 AMS 进行通信;  
除了 ActivityManager, 其他想要与 AMS 进行通信的类都需要通过 AMP;  

#### 在 Launcher 中点击  App 的图标后, 发生了什么  
点击桌面 App 图标, Launcher 所在的进程通过 Binder IPC 向 system_server 进程发起 startActivity 请求;  
system_server 进程接收到请求后, 如果发现目标 app 进程并没有在运行, 就会通过 socket 向 zygote 进程发送创建 app 进程的请求;  
zygote 进程收到请求后, 就会 fork 出 App 进程, 并调用 ActivityThread.main 方法, 在 main 方法里面创建 ActivityThread 对象, 并创建 Application 对象;  
App 进程启动后, 会通过 Binder IPC 向 system_server 进程发起 attachApplication 请求;  
system_server 进程在收到请求后, 进行一系列准备工作后, 再通过 binder IPC 向 App 进程发起 scheduleLaunchActivity 请求;  
App 进程的 ApplicationThread(binder 线程) 收到请求后, 通过 handler 向主线程发送 LAUNCH_ACTIVITY 消息;  
主线程在收到 Message 后, 通过发射机制创建目标 Activity, 并回调 Activity.onCreate()等方法;  
到此, App 便正式启动, 开始进入 Activity 生命周期, 执行完 onCreate/onStart/onResume方法, UI 渲染结束后便可以看到 App 的主界面;  

### startActivity 流程  
![流程图](/Android/basic/ImageFiles/start_ac_001.png)  

之前, ActivityManagerNative.getDefault() 得到一个 ActivityManagerProxy 对象, 是 client 端的代表;  
AMP 通过 binder 将数据传输到 activityManagerService, 后面程序进入 system_server 进程, 开始继续执行;  
ActivityThread  
App 的真正入口, 当开启 App 之后, 会调用 main()开始运行, 开启消息循环队列, 启动 UI 线程;  
ActivityThread.ApplicationThread  
ApplicationThread 是 ActivityThread 的内部类, 继承于 IApplicationThread.Stub, 也就是 Binder;  
用来完成 ActivityManagerService 与 ActivityThread 之间的交互;  
在 ActivityManagerService 需要管理相关 Application 中的 Activity 的生命周期时, 通过 ApplicationThread 的代理对象与 ActivityThread 通讯;  
ApplicationThreadProxy  
是 ApplicationThread 在服务器端的代理, 负责和客户端的 ApplicationThread 通讯;  AMS 就是通过该代理与 ActivityThread 进行通信的;  
ActivityThread.H  
H 其实是一个 Handler, 也是 ActivityThread 的一个内部类, 运行在主线程;  
Instrumentation  
管理一个活动的生命周期;  
Instrumentation 是 android 系统中启动 Activity 的一个实际操作类, 也就是说 Activity 在应用进程端的启动实际上就是 Instrumentation 执行的;   
每一个应用程序只有一个 Instrumentation 对象, 每个 Activity 内都有一个对该对象的引用;  Instrumentation 可以理解为应用进程的管家,  
ActivityThread 要创建或暂停某个 Activity 时, 都需要通过 Instrumentation 来进行具体的操作;  
Instrumentation#newActivity();  
Instrumentation#newApplication();  
ActivityStack  
管理一个活动栈  
Activity 在 AMS 的栈管理, 用来记录已经启动的 Activity 的先后关系, 状态信息等;  通过 ActivityStack 决定是否需要启动新的进程;  
ActivityStackSupervisor  
管理所有的活动栈  
ActivityRecord  
ActivityStack 的管理对象, 每个 Activity 在 AMS 对应一个 ActivityRecord, 来记录 Activity 的状态以及其他的管理信息;  其实就是服务器端的 Activity 对象的映像;  
ActivityStarter  
根据 intent, flags 找到 activity, stack  
TaskRecord  
AMS 抽象出来的一个"任务"的概念, 是记录 ActivityRecord 的栈, 一个"Task"包含若干个 ActivityRecord;  AMS 用 TaskRecord 确保 Activity 启动和退出的顺序;  
ActivityManagerService  
管理所有的活动;  
位于 system_server 进程, 从 ActivityManagerService 提供的接口来看, 它负责管理 Activity 的启动和生命周期;  
ActivityManagerProxy  
是 ActivityManagerService 在普通应用进程的一个代理对象;  
已经被废弃, 通过 AIDL 生成的对象;  
它只负责准备相关的数据发送到 system_process 进程去处理 startActivity;  
应用进程通过 ActivityManagerProxy 对象调用 ActivityManagerService 提供的功能;  
应用进程并不会直接创建 ActivityManagerProxy 对象,  
而是通过调用 ActivityManagerNative 类的工具方法 getDefault 方法得到 ActivityManagerProxy 对象;  
也就是 通过 ActivityManager#getService 方法得到 ActivityManagerProxy 对象;  
#### 函数调用栈  
Activity.startActivity  
Instrumentation#execStartActivity  
ActivityManagerProxy#startActivity  
无论是通过 launcher 来启动 Activity 还是通过其他 Activity 来启动另一个 Activity, 都需要通过 IPC 调用 ActivityManagerService 的 startActivity 的方法;  
也就是说, ActivityManagerService 调用 ApplicationThread 的 scheduleLaunchActivity 方法, scheduleLaunchActivity 通过 handler 发送消息;  
在 activityThread 的 performLaunchActivity 中创建 activity, 并创建 PhoneWindow 对象, 通过 activity 的 attach 方法, 把 phoneWindow 对象传递给 activity;  
如果 Application 未创建, ActivityManagerService 所在的 system_server 进程;  
创建并初始化 Application 对象, 参见 [在 Launcher 中点击  App 的图标后, 发生了什么]  
经过 IPC 调用, 启动 Activity 的指令来到了 ActivityManagerService, 紧接着 AMS 调用 startActivityAsUser 着手 Activity 的启动工作;  


ActivityStarter#startActiviytMayWait  
AMS 有一个 ActivityStack, 负责 Activity 的调度工作, 比如维护回退栈, 但 ActivityStack 内的 Activity 是抽象成 ActivityRecord 来管理的, Activity 对象不会存在于 AMS 当中;  

ActivityStarter#startActivityUncheckedLocked  
这个方法会根据 Activity 启动信息(提取封装到 ActivityInfo 类中)中的 launchMode, flag 等属性来调度 ActivityStack 中的 Task 和 ActivityRecord;  
因此这个方法是理解 Activity 启动模式的关键;  

ActivityStack#resumeTopActivityInnerLocked  
这个方法内部会把前台处于 Resume 状态的 Activity 变成 Pause 状态后才会继续启动 Activity 的逻辑;  
将一个 Activity 变成 Pause 状态需要经历的调用于后面的启动调用非常相似;  

ActivityStack#startSpecificActivityLocked  
这里最后会调用 AMS 的 startProcessLocked, 这个方法会先判断是否已经存在相应的进程, 如果不存在则通过远程调用 Zygote 进程来孵化出新的应用进程;  
zygote 进程孵化出新的应用进程后, 会执行 ActivityThread 类的 main 方法;  
在该方法里会先准备好 Looper 和消息队列, 然后调用 attach 方法将应用进程绑定到 ActivityManagerService, 然后进入 loop 循环, 不断地读取消息队列里的消息, 并分发消息;  
这个过程在 Android 的消息机制里已经非常熟悉了, 其中 attach 方法在与 AMS 绑定的过程中会调用 attachApplicationLocked 方法;  
attachApplicationLocked 方法有两个重要的函数调用 thread.bindApplication 和 mMainStack.realStartActivityLocked;  
thread.bindApplication 将应用进程的 ApplicationThread 对象绑定到 ActivityManagerService, 也就是说获得 ApplicationThread 对象的代理对象;  
mMainStack.realStartActivityLocked 通知应用进程启动 Activity  

ActivityStack#realStartActivityLocked  
app.thread 其实就是 ApplicationThread 在 AMS 的代理对象, 实际上是调用 ApplicationThread#scheduleLaunchActivity;  
接下来 Activity 的启动工作就交给应用进程来完成了, 别忘了这时候的 Activity 对象还没被创建呢;  

ActivityThread#performLaunchActivity  
通过类加载器加载 Activity 对象;  
创建 ContextImpl 对象并调用 activity 的 attach 方法, 把上下文变量设置进 activity 中, 创建 Window 对象和设置 WindowManager;  
回调 onCreate,onStart 和 onRestoreInstanceState 方法;  

ActivityThread#handleResumeActivity  
回调 Activity 的 onResume 方法;  
调用 WindowManager 的 addView 方法, 将前面在 attach 方法内创建出来的 window 对象添加到 WindowManager 当中;  

