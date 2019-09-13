#### ActivityManagerService  
工作在 SystemServer 进程;  
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
AMP 是 Client 端, AMN 则是 Server 端, 而 Server 端中具体的功能都是由 AMN 的子类 AMS 来实现的, 因此AMP就是AMS在Client端的代理类;   
AMN 又实现了Binder类, 这样AMP可以和AMS就可以通过Binder来进行进程间通信;  

ActivityManager 通过 AMN 的 getDefault 方法得到 AMP, 通过 AMP 就可以和 AMN 进行通信, 也就是间接的与 AMS 进行通信;  
除了 ActivityManager, 其他想要与 AMS 进行通信的类都需要通过AMP;  

#### 在 Launcher 中点击  App 的图标后, 发生了什么  
点击桌面 App 图标, Launcher 所在的进程通过 Binder IPC 向 system_server 进程发起 startActivity 请求;  
system_server 进程接收到请求后, 如果发现目标 app 进程并没有在运行, 就会通过 socket 向 zygote 进程发送创建 app 进程的请求;  
zygote 进程收到请求后, 就会 fork 出 App 进程, 并调用 ActivityThread.main 方法, 在 main 方法里面创建 ActivityThread 对象, 并创建 Application 对象;  
App 进程启动后, 会通过 Binder IPC 向 system_server 进程发起 attachApplication 请求;  
system_server 进程在收到请求后, 进行一系列准备工作后, 再通过 binder IPC 向 App 进程发起 scheduleLaunchActivity 请求;  
App 进程的 ApplicationThread(binder 线程) 收到请求后, 通过 handler 向主线程发送 LAUNCH_ACTIVITY 消息;  
主线程在收到 Message 后, 通过发射机制创建目标 Activity, 并回调 Activity.onCreate()等方法;  
到此, App 便正式启动, 开始进入 Activity 生命周期, 执行完 onCreate/onStart/onResume方法, UI 渲染结束后便可以看到 App 的主界面;  

startActivity 流程, 详见[链接](/Android/basic/context/Activity.md)  

