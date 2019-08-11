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
使用 DexClassLoader 加载dex 数组, patch.dex 最优先加载, 例如 A.class 存在于 patch.dex 和 class.dex 中,   
那么系统中只会存在一个 A.class, 就是 patch.dex 中的 class;  
如果 A.class 和 B.class 以前在一个 dex 文件中, 现在 A.class 在 patch.dex 中, 那么, 如果 A 类和 B 类有相互调用的时候, 就是出现 与校验的问题, 下面具体讲解;  
基于 multiDex 实现的热更新, 必须在重启之后才会生效;  
简单来讲, 热更新的原理就是 dex 插桩;  
java 文件编译成 .class 文件, dx.bat 再打包成 patch.dex;  

❀ CLASS_ISPREVERIFIED 问题  
在 apk 安装的时候, 虚拟机会将 dex 优化成 odex 后才拿去执行, 在这个过程中会对所有 class 进行校验;    
校验方式, 假设 A 类的 static 方法, private方法, 构造函数, override 方法中直接引用到 B 类, 如果 A 类和 B 类在同一个 dex 中, 那么 A 类就会被打上 CLASS_ISPREVERIFIED 标记;  
被打上这个标记的类不能引用其他 dex 中的类, 否则就会报错;  
在我们的 Demo 中, MainActivity 和 Cat 本身是在同一个 dex 中的, 所以 MainActivity 被打上了 CLASS_ISPREVERIFIED,  
而我们修复 bug 的时候却引用了另外一个 dex 的 Cat.class, 所以这里就报错了;  
而普通分包方案则不会出现这个错误, 因为引用和被引用的两个类一开始就不在同一个 dex 中, 所以校验的时候并不会被打上 CLASS_ISPREVERIFIED;  
补充一下第二条: A 类如果还引用了一个 C 类, 而 C 类在其他 dex 中, 那么 A 类并不会被打上标记,  
换句话说, 只要在 static 方法, 构造方法, private方法, override 方法中直接引用了其他 dex 中的类, 那么这个类就不会被打上 CLASS_ISPREVERIFIED 标记;  

在 Dalvik虚拟机下, 执行 dexopt 时, 会对类进行扫描, 如果类里面所有直接依赖的类都在同一个 dex 文件中, 那么这个类就会被打上 CLASS_ISPREVERIFIED 标记,  
如果一个类有 CLASS_ISPREVERIFIED 标记, 那么在热修复时, 它加载了其他 dex 文件中的类, 会报经典的Class ref in pre-verified class resolved to unexpected implementation异常;  
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
假如需要启动插件中的 AActivity, Intent 传值的时候, 启动的是 SubActivity, 在真正 new 出来 Activity 的时候, 利用 Intent 的传入参数, 创建目标 AActivity;  
 
Service ContentProvider 的启动方式, 也是通过 SubService 的形式来完成的;  
BroadCastReceiver 是通过解析清单文件, 将静态注册转为动态注册;  
 
 
 
### 虚拟机  
❀ 编译技术  
JIT just in time, 及时编译技术, JIT 会在运行时分析应用程序的代码, 识别哪些方法可以归类为热方法,   
    这些方法会被 JIT 编译器编译成对应的汇编代码, 然后存储到代码缓存中, 以后调用这些代码时就不用解释执行了, 
    可以直接使用代码缓存中已编译好的汇编代码, 能显著提升应用程序的执行效率;  
AOT Ahead Of Time, 预编译技术, 编译器在编译时直接将源程序编译成目标机器码, 运行时直接运行机器码;  
Dalvik 采用的是 JIT 技术, 而 ART 采用的是 AOT 技术;   
ART 在应用程序安装的时候, 就已经将所有的字节码编译成了机器码, 运行的时候直接运行的是机器码;  
Dalvik 则是在应用程序运行的时候, 实时将字节码编译成机器码;  
ART 与 Dalvik 相比, 在安装的时候, 需要把所有的字节码编译成机器码, 所以安装的时间会比较长一些, 同时占用内存更多一些;  
ART 与 Dalvik 相比, 省去了运行时将字节码编译成机器码的过程, 极大地提升了应用程序的运行效率;  

❀ 关于 GC  
Dalvik 在 gc 时, 会挂起所有线程, 然后进行垃圾回收, 完成后恢复所有线程;  
需要注意的是, gc 与应用程序不是并发执行的, 应用程序如同停止了一样, 如果 gc 频繁时间过长都会导致 app 卡顿;  
ART 在垃圾回收的时候, 部分过程并发执行, 缩短了垃圾回收时间;  

Dalvik 虚拟机垃圾回收采用了 "标记-清除" 算法, 它是一种比较高效的算法, 但是会造成可用内存块的不连续, 碎片化严重;  
ART 在内存分配上做了优化, 开辟了一块名为 Large Object Space 的内存区域, 专门用来存放大对象,   
    同时还引入了moving collector 技术, 专门用来将 gc 后不连续的物理内存块对齐, 解决了内存碎片化严重的问题;  
    
### 参考  

热更新;热修复;   
https://github.com/Omooo/Android-Notes/blob/master/blogs/Android/热修复.md   
https://juejin.im/post/5a0ad2b551882531ba1077a2  
https://yq.aliyun.com/articles/231111  
https://yq.aliyun.com/live/313  
https://github.com/WeMobileDev/article/blob/master/微信Android热补丁实践演进之路.md  


http://weishu.me/2016/01/28/understand-plugin-framework-overview/  
http://weishu.me/2016/01/28/understand-plugin-framework-proxy-hook/  
http://weishu.me/2016/03/21/understand-plugin-framework-activity-management/  
http://weishu.me/2016/04/05/understand-plugin-framework-classloader/  
http://weishu.me/2016/04/12/understand-plugin-framework-receiver/  
http://weishu.me/2016/05/11/understand-plugin-framework-service/  
http://weishu.me/2016/07/12/understand-plugin-framework-content-provider/  

反射原理  http://javawebsoa.iteye.com/blog/1512798  
类加载器  https://www.ibm.com/developerworks/cn/java/j-lo-classloader/  
类加载器  http://blog.csdn.net/zdwzzu2006/article/details/2253982  
Android动态加载   
http://www.androidblog.cn/index.php/Index/detail/id/16#  
http://blog.csdn.net/u013478336/article/details/50734108（已看，最简单入门）  
http://www.cnblogs.com/over140/archive/2011/11/23/2259367.html  
http://blog.csdn.net/u010687392/article/details/47121729  （入门详解版本）  
https://segmentfault.com/a/1190000004062866#articleHeader9（入门基础）  
https://segmentfault.com/a/1190000004062866（深入L1理解）  
https://segmentfault.com/a/1190000004062972（深入L2 代理Activity）  
https://segmentfault.com/a/1190000004077469（深入L2 动态创建Activity）  
http://www.jianshu.com/p/353514d315a7  
https://www.jianshu.com/p/b65e5da3dff2  
https://www.jianshu.com/p/e179fcc97666  


插件化
https://www.jianshu.com/p/b6d0586aab9f  
https://www.kymjs.com/code/2016/05/04/01/  
https://juejin.im/post/59752eb1f265da6c3f70eed9  

https://github.com/ManbangGroup/Phantom  
https://github.com/tiann/understand-plugin-framework  
http://weishu.me/2016/01/28/understand-plugin-framework-overview/  
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

GC  
https://yq.aliyun.com/articles/179805  


热修复比较    
http://w4lle.com/2017/05/04/hotpatch-summary/  

instant run 原理  
https://blog.csdn.net/nupt123456789/article/details/51828701  
https://blog.csdn.net/xiangzhihong8/article/details/64906131  

废物  
https://juejin.im/post/5bf22bb5e51d454cdc56cbd5   
https://juejin.im/post/5c5100c1e51d4550f31755b6  
https://tech.meituan.com/2016/09/14/android-robust.html  
https://blog.csdn.net/u010299178/article/details/52031505  
https://segmentfault.com/a/1190000004062866  
https://www.jianshu.com/p/b1e7b6326330  

