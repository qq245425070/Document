[classLoader](/Java/jvm/object_info.md)  
[robust](/Android/framework/plugin_hotfix/robust.md)  
[VirtualApp](/Android/framework/plugin_hotfix/VirtualApp.md)  
[VirtualApk](/Android/framework/plugin_hotfix/VirtualApk.md)  

在 Java 程序中, JVM 虚拟机通过类加载器 ClassLoader 来加载 class 文件;  
Android 与 java 类似, 只不过 Android 使用的是 dalvik/Art 虚拟机来运行 .dex 文件;  
.dex 文件本质上是 .class 文件打包优化而得到的;  
Android 的类加载器, 包括系统和自定义两种;   
系统 classLoader 包括 BootClassLoader, PathClassLoader, DexClassLoader;  
```
ClassLoader  
    | BootClassLoader  
    | SecureClassLoader  
        | URLClassLoader  
    | BaseDexClassLoader  
        | DexClassLoader  
        | PathClassLoader  
```
BootClassLoader   是 ClassLoader 的内部类, 用于预加载 preload()常用类以及一些系统 Framework 层级需要的类;  
PathClassLoader    用于加载data/app下的, 也就是已安装的 apk 中的 apk, dex, class 文件;  
DexClassLoader     可以加载外部的dex 的 apk, dex, class 文件, 也支持从 SD 卡进行加载;  
URLClassLoader     可以加载 java 中的 jar,但是由于 dalvik 不能直接识别 jar, 所以此方法在 android 中无法使用, 尽管还有这个类;  

Hot Swap, Warm Swap, Cold Swap  
基于 native 层, 函数调用指向新的指针;  

### instant run 原理  
每一个方法, 都有一个钩子 change 变量, 类型是 IncrementalChange 这个接口, 如果代码有变动, 会生成一个实现类, 再根据要改动的方法, 生成一个对应的实现方法;  
判断如果 change 不为空, 就执行 change 里面方法, 如果 change 为空, 就执行自己的逻辑;  

### 热更新  
主要分为 java 层实现, 和 native 层实现两种, java 层实现主要分为基于 multiDex 和 instantRun;  
使用 DexClassLoader 加载 dex 数组, patch.dex 最优先加载, 例如 A.class 存在于 patch.dex 和 class.dex 中;   
那么系统中只会存在一个 A.class, 就是 patch.dex 中的 class;  
如果 A.class 和 B.class 以前在一个 dex 文件中, 现在 A.class 在 patch.dex 中, 那么, 如果 A 类和 B 类有相互调用的时候, 就是出现 与校验的问题, 下面具体讲解;  
基于 multiDex 实现的热更新, 必须在重启之后才会生效;  
简单来讲, 热更新的原理就是 dex 插桩;  
java 文件编译成 .class 文件, dx.bat 再打包成 patch.dex;  

CLASS_ISPREVERIFIED 问题  
在 dalvik 虚拟机上, 安装 apk 的过程中, 会有一个验证优化 dex 的机制, 就是 dexOpt(Optimised Dex), 这个过程会生成 odex 文件, 当然 odex 文件也是属于 dex 文件;  
执行 odex 的效率会比直接执行 dex 文件的效率要高很多;   
运行 Apk 的时候, 直接加载 odex 文件, 从而避免重复验证和优化, 加快了 apk 的响应时间;  
在 apk 安装的时候, 虚拟机会将 dex 优化成 odex 后才拿去执行, 在这个过程中会对所有 class 进行校验;    
校验方式, 假设 A 类的 static 方法, private方法, 构造函数, override 方法中直接引用到 B 类, 如果 A 类和 B 类在同一个 dex 中, 那么 A 类就会被打上 CLASS_ISPREVERIFIED 标记;  
被打上这个标记的类不能引用其他 dex 中的类, 否则就会报错;  
在我们的 Demo 中, MainActivity 和 Cat 本身是在同一个 dex 中的, 所以 MainActivity 被打上了 CLASS_ISPREVERIFIED;  
而我们修复 bug 的时候却引用了另外一个 dex 的 Cat.class, 所以这里就报错了;  
而普通分包方案则不会出现这个错误, 因为引用和被引用的两个类一开始就不在同一个 dex 中, 所以校验的时候并不会被打上 CLASS_ISPREVERIFIED;  
A 类如果还引用了一个 C 类, 而 C 类在其他 dex 中, 那么 A 类并不会被打上标记,  
换句话说, 只要在 static 方法, 构造方法, private方法, override 方法中直接引用了其他 dex 中的类, 那么这个类就不会被打上 CLASS_ISPREVERIFIED 标记;  
在 Dalvik虚拟机下, 执行 dexopt 时, 会对类进行扫描, 如果类里面所有直接依赖的类都在同一个 dex 文件中, 那么这个类就会被打上 CLASS_ISPREVERIFIED 标记;  
如果一个类有 CLASS_ISPREVERIFIED 标记, 那么在热修复时, 它加载了其他 dex 文件中的类, 会报经典的 Class ref in pre-verified class resolved to unexpected implementation 异常;  
通过在 android7.0 8.0 上进行热修复, 也没有遇到这个异常, 猜测这个问题只属于 android5.0以前, 因为 android5.0 后新增了 art;  
很多热更新的解决方案, 都存在这个问题, 但是美团的 robust 和 阿里的 Sophix 不存在, robust 的解决办法很巧妙, sophix 则是 native 方式解决, 所以不存在此问题;  
```
Dexposed                阿里               开源          实时修复  
Andfix                      阿里               开源           实时修复  
阿里百川 Hotfix      阿里               未开源       实时修复  
QQ 空间                   QQ 空间        未开源       冷启动修复  
QFix                          手Q                开源           冷启动修复  
Robust                     美团               开源           实时修复  
Nuwa                        大众点评       开源           冷启动修复    
RocooFIx                  百度金融       开源           冷启动修复  
Aceso                        蘑菇街          开源           实时修复  
Amigo                        饿了么         开源           冷启动修复  
Tinker                         微信             开源           冷启动修复  
Sophix                        阿里             未开源       实时修复+ 冷启动修复  
```

### 插件化  
第一代  dynamic-load-apk  DroidPlugin  
第二代  VirtualApk  Small  RePlugin 
第三代  VirtualApp  Atlas  
 
多 dex 模式, 对于每一个插件, 都会生成一个 DexClassLoader, 当加载该插件中的类, 需要通过对应的 DexClassLoader;  
这样, 不同的插件中的类是隔离的, 当不同的插件, 都引用了同一个库, 但是版本号不一致时, 是不会有问题的, RePlugin 就是采用的这种方案.  

单 dex 模式, 将每个插件的 pathList 合并到主工程的 DexClassLoader 中, 这样的好处是, 不同的插件之间互相调用, 不需要通过中介来完成, 直接调用即可;  
 
怎么启动插件中的 Activity, 因为插件中的 Activity 并没有在清单文件中注册, 也就是如何绕过系统的检查;  
解决问题的办法有很多种, 例如在清单文件中预先配置 SubActivity(就是占位符), 在 Application 初始化的时候, 利用反射系统的 Instrumentation,   
Instrumentation 是 ActivityThread 的字段, ActivityThread 对象在内存中只有一份;  
假如需要启动插件中的 AActivity, Intent 传值的时候, 启动的是 SubActivity, 在真正 new 出来 Activity 的时候, 利用 Intent 的传入参数, 创建目标 AActivity;  

Service ContentProvider 的启动方式, 也是通过 SubService 的形式来完成的;  
BroadCastReceiver 是通过解析清单文件, 将静态注册转为动态注册;  

1.. 关于 Activity 插件化  
四大组件的通信, 全都是通过 ActivityManagerService 来管理的, 所以想要做到欺上瞒下, 就要 hook 住 AMS;  
在启动 Activity 的时候, componentName 是清单文件中预留的 SubActivity, 但是会传入参数, 告知系统真正想要启动的是 AActivity;  
在 AMS 受到请求的时候, 会检查 componentName 对应的 Activity 是否在清单文件中注册, 如果是则会调起 Activity 对应的 ApplicationThread, 之后通过 handler 转发给 activityThread;  
在 activityThread 中通过 Instrumentation 来解析 intent 并创建 Activity, 所以 hook 住 ActivityThread 里面的 Instrumentation 就可以启动真正的 Activity 了;  

1.1.. launchModel 启动模式  
正常来讲, 连续进入 5-6 级页面已经是极限了, 如果需要进入 10 级以上页面, 那真是产品设计缺陷, 不在技术上解决, 没有用户能接受层级这么深入;  
每种 LaunchMode 声明十个 StubActivity 绝对能满足需求了;  

1.2.. 页面生命周期  
目标 Activity 是具有正常的生命周期的;  
AMS 要启动的是 SubActivity, 我们在 activityThread 中, 通过启动传入的参数, 改成了目标 Activity,  那么为什么还能正确的完成对 目标 Activity 生命周期的回调呢?  
AMS 与 ActivityThread 之间对 Activity 生命周期的交互, 并没有直接使用 Activity 对象进行交互, 而是使用一个 token 来标识, 这个 token 是 binder 对象, 因此可以方便地跨进程传递;  
Activity 里面有一个成员变量 mToken 代表的就是它, token 可以唯一地标识一个 Activity 对象, 它在 Activity 的 attach 方法里面初始化;  
在 AMS 处理 Activity 的任务栈的时候, 使用这个 token 标记 Activity;  
因此, 在 ActivityThread 执行回调的时候, 能正确地回调到目标 Activity 相应的方法;  

1.3.. 管理资源文件  
context 引用资源文件, 是通过 Resources 类来完成的;  
Resources 对资源的管理是通过 AssetManager 来完成的;  
AssetManager 有有一个方法 addAssetPath, app 启动的时候, 会把当前 apk 的路径传过去, 这样 AssetManager 就可以管理当前 apk 的所有资源文件了;  
所以通过反射得到 AssetManager 对象, 并 调用 addAssetManagerPath 方法, 把 plugin 的路径添加进去, 这样就可以获取到 plugin 里面的资源了;  

2.. 关于 Service 插件化  
四大组件的通信, 都是通过 ActivityManagerService 来管理的;  
在启动 Service 的时候, componentName 是清单文件中预留的 SubService, 但是会传入参数, 告知系统真正想要启动的是 Service;  
在 AMS 受到请求的时候, 会检查 componentName 对应的 Service 是否在清单文件中注册, 如果是则会调起 Service 对应的 ApplicationThread, 之后通过 handler 转发给 activityThread;  
在 activityThread 中直接 new 出来 Service, 这样就启动了 Service;  
所以 hook 朱 activityThread 的 H 类, 就可以解析参数, 启动真正的 Service 了;  
和 Activity 不同的是 Service 并没有lunchModel 的概念, 因为 Service 本身是单例存在的;  
所以只用一个 SubService 是应付不了 Service 插件化的问题;  

3.. 关于 BroadcastReceiver 插件化  
四大组件的通信, 都是通过 ActivityManagerService 来管理的;  
广播有注册, 发送, 接收这几个概念, 注册的过程最终会将广播接收器的信息注册到 AMS;  
发送广播也是通过 AMS 查找 action 相关的注册者是谁, AMS 会回调 action 对应的 applicationThread, 然后再回调对应的接收器的 onReceive 方法;  
广播又区分静态注册和动态注册, 区别就在于静态广播需要在清单文件注册, 动态广播用 java 代码注册;  
最简单的解决方案就是, 静态注册转动态注册;  


4.. 关于 ContentProvider 插件化  

### 虚拟机  
❀ 编译技术  
JIT just in time  
在程序运行起来之后, 把字节码编译成机器码, 然后执行;   
提高实时运行效率, 但是打开 app 的时候变慢了;  
JIT 可以理解为, 为了提高运行效率, 实时的把字节码编译成机器码, 缓存起来, 下次运行到相同代码逻辑的时候, 运行的是机器码, 提高了运行效率;  
但是, 字节码编译成机器码的时候, 也是耗时的, 所以 dalvik 只对一些次数较多的代码, 做优化;  
但是, 编译好的机器码, 缓存在内存中, 每次进程重启, 都需要重新编译, 这样打开 app, 就会变慢;  

AOT Ahead Of Time  
在程序安装的时候, 把字节码编译成机器码, 然后执行;  
提高实时运行效率, 打开 app 也变快了, 但是安装 app 的时候, 变慢了;  
2.2 以后 Dalvik 采用的是 JIT 技术, 而 5.0 以后 ART 采用的是 AOT 技术;   
android 7.0 以后, app 安装的时候, 不会把字节码编译成机器码, 这样缩短了 app 安装时间;  
app 首次运行的时候, dex 文件先通过解析器被直接执行;  
对于热点函数, 会被编译成机器码, 提高运行效率(JIT);  
保存 JIT 编译后的代码信息, 存储在 jit code cache 中并生成 profile 文件以记录热点函数的信息;  
手机进入空闲状态的时候, 系统会扫描 App 目录下的 profile 文件并执行 AOT 过程进行编译;  

❀ 关于 GC  
Dalvik 在 gc 时, 会挂起所有线程, 然后进行垃圾回收, 完成后恢复所有线程;  
需要注意的是, gc 与应用程序不是并发执行的, 应用程序如同停止了一样, 如果 gc 频繁时间过长都会导致 app 卡顿;  
ART 在垃圾回收的时候, 部分过程并发执行, 缩短了垃圾回收时间;  

Dalvik 虚拟机垃圾回收采用了 "标记-清除" 算法, 它是一种比较高效的算法, 但是会造成可用内存块的不连续, 碎片化严重;  
ART 在内存分配上做了优化, 开辟了一块名为 Large Object Space 的内存区域, 专门用来存放大对象;   
同时还引入了moving collector 技术, 专门用来将 gc 后不连续的物理内存块对齐, 解决了内存碎片化严重的问题;  
    
### 参考  
热更新;热修复;   
https://github.com/GitLqr/HotFixDemo  
https://yq.aliyun.com/articles/231111  
https://github.com/WeMobileDev/article/blob/master/微信Android热补丁实践演进之路.md  

http://weishu.me/2016/01/28/understand-plugin-framework-overview/  
http://weishu.me/2016/01/28/understand-plugin-framework-proxy-hook/  
http://weishu.me/2016/03/21/understand-plugin-framework-activity-management/  
http://weishu.me/2016/04/05/understand-plugin-framework-classloader/  
http://weishu.me/2016/04/12/understand-plugin-framework-receiver/  
http://weishu.me/2016/05/11/understand-plugin-framework-service/  
http://weishu.me/2016/07/12/understand-plugin-framework-content-provider/  

反射原理  
http://javawebsoa.iteye.com/blog/1512798  
类加载器  
https://www.ibm.com/developerworks/cn/java/j-lo-classloader/  
类加载器  
http://blog.csdn.net/zdwzzu2006/article/details/2253982  
Android动态加载   
http://www.androidblog.cn/index.php/Index/detail/id/16#  
已看, 最简单入门  
http://blog.csdn.net/u013478336/article/details/50734108
  
http://www.cnblogs.com/over140/archive/2011/11/23/2259367.html  
入门详解版本  
http://blog.csdn.net/u010687392/article/details/47121729  
入门基础  
https://segmentfault.com/a/1190000004062866#articleHeader9    
深入 L1 理解  
https://segmentfault.com/a/1190000004062866   
深入 L2 代理 Activity   
https://segmentfault.com/a/1190000004062972    
深入 L2 动态创建 Activity  
https://segmentfault.com/a/1190000004077469    
http://www.jianshu.com/p/353514d315a7  
https://www.jianshu.com/p/b65e5da3dff2  
https://www.jianshu.com/p/e179fcc97666  


插件化  
https://github.com/tiann/understand-plugin-framework  
http://weishu.me/2016/05/11/understand-plugin-framework-service/  
http://weishu.me/2016/07/12/understand-plugin-framework-content-provider/  
https://segmentfault.com/a/1190000004062866  
http://liuwangshu.cn/application/classloader/1-java-classloader-.html  
http://liuwangshu.cn/application/classloader/2-android-classloader.html
http://liuwangshu.cn/application/hotfix/1-code-repair.html  
https://www.jianshu.com/p/c58804962f73  
https://www.jianshu.com/p/a620e368389a  
https://www.jianshu.com/p/96a72d1a7974  

http://www.cnblogs.com/lanrenxinxin/p/4712224.html  
https://www.jianshu.com/p/704cac3eb13d  
http://weishu.me/2016/03/21/understand-plugin-framework-activity-management/  
https://juejin.im/post/5c1b20e26fb9a049c30b33d7  
https://github.com/SusionSuc/AdvancedAndroid/tree/master/插件化  

虚拟机  
https://blog.csdn.net/jason0539/article/details/50440669  
https://yq.aliyun.com/articles/663995  
https://juejin.im/post/5d4bdb23e51d453c2577b747  
https://github.com/interviewandroid/AndroidInterView/blob/master/android/dalvik.md  
https://github.com/interviewandroid/AndroidInterView/blob/master/android/artordalvik.md  

GC  
https://yq.aliyun.com/articles/179805  


热修复比较    
http://w4lle.com/2017/05/04/hotpatch-summary/  

instant run 原理  
https://blog.csdn.net/nupt123456789/article/details/51828701  
https://blog.csdn.net/xiangzhihong8/article/details/64906131  

tinker 原理  
https://github.com/interviewandroid/AndroidInterView/blob/master/tencent/tinker.md  


废物  
https://juejin.im/post/5bf22bb5e51d454cdc56cbd5   
https://juejin.im/post/5c5100c1e51d4550f31755b6  
https://tech.meituan.com/2016/09/14/android-robust.html  
https://blog.csdn.net/u010299178/article/details/52031505  
https://segmentfault.com/a/1190000004062866  
https://www.jianshu.com/p/b1e7b6326330  
https://www.jianshu.com/p/b6d0586aab9f  




