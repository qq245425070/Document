[关于展示吐司和通知](library/open_toast_notification.md)  
[Activity 启动模式与任务栈](library/launchMode.md)  
[清单文件 Activity 标签属性](library/manifest_tag.md)  
[状态栏高度 24dp](ImageFiles/status_bar_height.png)  
[Activity 常见方法](library/function.md)  
[Fragment 重叠异常](library/solution_001.md)  
不管 Activity 是不是被回收, 只要执行 onStop 就一定会执行 onSaveInstanceState;  

Activity 会通过 android:id 逐个恢复 View 的 State;  
也就是说, 如果 android:id 为空, View 将不具备恢复 State 的能力了;  
所有的自定义控件, 都应该实现 State 相关方法, onSaveInstanceState and onRestoreInstanceState;  
一旦 Fragment 从回退栈出来, Fragment 本身还在, View 却是重新创建的;  
但是给 TextView, EditText 设置 android:freezeText="true" 会让其在 Fragment 内, 自动保存 State;  

启动一个 Activity 和 Fragment, 他们的生命周期方法, 调用顺序;  
切换横竖屏;  屏幕锁;  
[链接](library/lifecycle_sample.md)  

### Fragment  

setRetainInstance  
Fragment 具有属性 retainInstance, 默认值为 false;  
当设备旋转时, fragment 会随托管 activity 一起销毁并重建;  

如果 retainInstance 属性值为 false, FragmentManager 会立即销毁该 fragment 实例;  
随后, 为适应新的设备配置, 新的 Activity 的新的 FragmentManager 会创建一个新的 fragment 及其视图;  

如果 retainInstance 属性值为 true, 则该 fragment 的视图立即被销毁, 但 fragment 本身不会被销毁;  
为适应新的设备配置, 当新的 Activity 创建后, 新的 FragmentManager 会找到被保留的 fragment, 并重新创建其视图;   
一旦发生 Activity 重启现象, Fragment 会跳过 onDestroy 直接进行 onDetach(界面消失, 对象还在),  
而 Fragment 重启时也会跳过 onCreate, 而 onAttach 和 onActivityCreated 还是会被调用;  
需要注意的是, 要使用这种操作的 Fragment 不能加入 backStack 后退栈中;  
并且, 被保存的 Fragment 实例不会保持太久, 若长时间没有容器承载它, 也会被系统回收掉的;  

[FragmentManagerImpl, Api21](library/FragmentManagerImplApi21.md)  
getActivity()空指针问题  
❀ 搞清楚, 为什么 f.mActivity() 为空  
如果 app 长时间在后台运行, 再次进入 app 的时候可能会出现 crash, 而且 fragment 会有重叠现象;  
如果系统内存不足, 切换横竖屏, app 长时间在后台运行, Activity 都可能会被系统回收然后重建,  
但 Fragment 并不会随着 Activity 的回收而被回收, 创建的所有 Fragment 会被保存到 Bundle 里面, 从而导致 Fragment 丢失对应的 Activity;  
Fragment 放在 ViewPager 中, ViewPager 只预加载三个, 在跳转到未被预加载的 Item 的时候, 目标 Fragment 也重新创建, 这个时候, 通过 getActivity()获取不到 context;  
❀ 解决办法  
Fragment 中维护一个 全局的 Activity 对象, 在 onAttach 方法中 给其赋值, 在 onDetach 中, 把它置空;  
管理好 Fragment 的生命周期, 在 onCreate 开始监听, 在 onDestroy 时, 解除监听, 忽略掉耗时操作的回调;  



#### commit  

❀ commit()  
在主线程中异步执行, 其实也是 Handler 抛出任务, 等待主线程调度执行;  
commit() 需要在宿主 Activity 保存状态之前调用, 否则会报错;  
这是因为如果 Activity 出现异常需要恢复状态, 在保存状态之后的 commit() 将会丢失, 这和调用的初衷不符, 所以会报错;  

❀ commitAllowingStateLoss()  
commitAllowingStateLoss() 也是异步执行, 但它的不同之处在于, 允许在 Activity 保存状态之后调用, 也就是说它遇到状态丢失不会报错;  

❀ commitNow()  
commitNow() 是同步执行的, 立即提交任务;  
FragmentManager.executePendingTransactions() 也可以实现立即提交事务;  
但我们一般建议使用 commitNow(), 因为另外那位是一下子执行所有待执行的任务, 可能会把当前所有的事务都一下子执行了, 这有可能有副作用;  
此外, 这个方法提交的事务可能不会被添加到 FragmentManger 的后退栈, 因为你这样直接提交, 有可能影响其他异步执行任务在栈中的顺序;  
和 commit() 一样, commitNow() 也必须在 Activity 保存状态前调用, 否则会抛异常;  

attach 与 detach  
transaction.attach(fragment); 对应 onCreateView-onViewCreated-onActivityCreated-onStart-onResume  
transaction.detach(fragment);  对应 onPause-onStop-onDestroyView  


#### transaction  
1.. replace  加入回退栈, Fragment 不销毁, 但是切换回销毁视图和重新创建视图;  
2.. replace  未加回退栈, Fragment 销毁掉;  
3.. hide. show. Fragment 不销毁, 也不销毁视图, 隐藏和显示不走生命周期;  

replace, AFragment 加入回退栈  
在同一个位置, 第一次 replace AFragment, 第二次 replace BFragment;  
```
A: onAttach -> onCreate -> onCreateView -> onActivityCreated -> onStart -> onResume;  
A: onPause -> onStop -> onDestroyView  
B: onAttach -> onCreate -> onCreateView -> onActivityCreated -> onStart -> onResume;  
```

replace, AFragment 未加回退栈  
在同一个位置, 第一次 replace AFragment, 第二次 replace BFragment;  
```
A: onAttach -> onCreate -> onCreateView -> onActivityCreated -> onStart -> onResume;  
A: onPause -> onStop -> onDestroyView -> onDestroy -> onDetach  
B: onAttach -> onCreate -> onCreateView -> onActivityCreated -> onStart -> onResume;  
```


detach 与 attach  
fragmentTransaction.detach(fragmentA);  
```
onPause -> onStop -> onDestroyView  
```
fragmentTransaction.attach(fragmentA);  
```
onCreateView -> onActivityCreated -> onStart -> onResume  
```

#### fragmentManager  

popBackStack(String tag,int flags)  
```
如果  tag = null, flags = 0, 弹出回退栈中最上层的那个 fragment  
如果  tag = null, flags = 1, 弹出回退栈中所有 fragment  
如果  tag != null, flags = 0, 弹出该 fragment 以上的所有 Fragment, 不包括 tag  
如果  tag != null, flags = 1, 弹出该 fragment 以上的所有 Fragment, 包括 tag  
原本 D -> C -> B -> A ;  
执行  
```

未挂载异常  
```
//  "Fragment " + this + " not attached to Activity"  
transaction.add(routerFragment, RouterConfig.FM_TAG);  
transaction.commitAllowingStateLoss();  
manager.executePendingTransactions();  
```
### startActivity 流程  
![流程图](ImageFiles/start_ac_001.png)  
以下分析基于 api 26  
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
创建并初始化 Application 对象, 参见 [在 Launcher 中点击  App 的图标后, 发生了什么](/Android/basic/ipc_service/system_zygote_binder.md)  
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
###  参考  
https://inthecheesefactory.com/blog/fragment-state-saving-best-practices/en  
https://github.com/nuuneoi/StatedFragment  

❀ startActivity  
https://github.com/interviewandroid/AndroidInterView/blob/master/android/ams.md  

https://www.jianshu.com/p/a768810c3ff8  
https://blog.csdn.net/stonecao/article/details/6591847  
https://blog.csdn.net/qq_23547831/article/details/51224992  
http://gityuan.com/2016/03/12/start-activity/  
https://github.com/yipianfengye/androidSource/blob/master/14 activity 启动流程.md  
http://aspook.com/2017/02/10/Android-Instrumentation 源码分析(附 Activity 启动流程)/  
深入理解 Android 内核设计思想  
Android 开发艺术探索  
https://blog.csdn.net/pihailailou/article/details/78545391  
https://blog.csdn.net/itachi85/article/details/64123035  
https://blog.csdn.net/qian520ao/article/details/81908505  
https://blog.csdn.net/luoshengyang/article/details/6689748  
https://blog.csdn.net/luoshengyang/article/details/6703247  
https://juejin.im/post/5c4180566fb9a049a62cdfd7  
https://juejin.im/post/5c469b23f265da614933efe8  
https://juejin.im/post/5c483eaff265da61327fa0e3  
https://blog.csdn.net/AndrExpert/article/details/81488503  
https://www.jianshu.com/p/a72c5ccbd150  
https://lrh1993.gitbooks.io/android_interview_guide/content/android/advance/app-launch.html  
https://blog.csdn.net/afu10086/article/details/80140817  

❀ Activity 参考  
http://liuwangshu.cn/framework/ams/2-activitytask.html  
http://liuwangshu.cn/framework/component/6-activity-start-1.html  
http://liuwangshu.cn/framework/component/7-activity-start-2.html  
https://developer.android.com/training/basics/activity-lifecycle/recreating?utm_campaign=adp_series_processes_012016&utm_source=medium&utm_medium=blog  


❀ fragment 参考  
http://toughcoder.net/blog/2015/04/30/android-fragment-the-bad-parts/  
http://www.jianshu.com/p/825eb1f98c19  
https://github.com/AlanCheen/Android-Resources/blob/master/Fragment.md  
https://github.com/JustKiddingBaby/FragmentRigger  
http://blog.csdn.net/u011240877/article/details/78132990#fragment-的使用  
https://www.jianshu.com/p/f2fcc670afd6  
https://www.jianshu.com/p/d9143a92ad94  
https://www.jianshu.com/p/fd71d65f0ec6  
https://www.jianshu.com/p/38f7994faa6b  
https://www.jianshu.com/p/9dbb03203fbc  
https://www.jianshu.com/p/78ec81b42f92  
https://www.jianshu.com/p/c12a98a36b2b  
http://toughcoder.net/blog/2015/04/30/android-fragment-the-bad-parts/  
https://github.com/YoKeyword/Fragmentation/blob/master/README_CN.md  
https://meta.tn/a/15e2d3292a521d700b4fef2f4ebaaa331b8df551431a766ff85b9a5b50c851fc  
https://meta.tn/a/b5b52d1cf21bb9f929cfc2b78b03927c97e74d076d094b12656c6c8c661d3072  

❀ getActivity()空指针问题  参考  
http://blog.csdn.net/goodlixueyong/article/details/48715661  

❀ application 参考  
http://www.jianshu.com/p/f665366b2a47  

❀ context 参考  
http://liuwangshu.cn/framework/context/2-activity-service.html  
https://blog.csdn.net/guolin_blog/article/details/47028975  


❀ onActivityResult  
请放弃使用 类似的库, 因为在 页面被回收, 页面重启后, 回调不会被执行的;   
https://github.com/VictorAlbertos/RxActivityResult  
https://github.com/florent37/InlineActivityResult  
https://github.com/NateWickstrom/RxActivityResult  
https://github.com/nekocode/RxActivityResult  

❀ 启动模式 IntentFilter  
https://juejin.im/post/5c5d85da6fb9a049fd104d8f  

