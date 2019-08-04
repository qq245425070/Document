[classLoader](/Java/jvm/object_info.md)  
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
https://github.com/Omooo/Android-Notes/blob/master/blogs/Android/热修复.md   
https://juejin.im/post/5a0ad2b551882531ba1077a2  
https://blog.csdn.net/u010299178/article/details/52031505  



https://yq.aliyun.com/articles/231111  
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

废物  
https://juejin.im/post/5bf22bb5e51d454cdc56cbd5   
https://juejin.im/post/5c5100c1e51d4550f31755b6  
https://tech.meituan.com/2016/09/14/android-robust.html  
https://blog.csdn.net/u010299178/article/details/52031505  
https://segmentfault.com/a/1190000004062866  
https://www.jianshu.com/p/b1e7b6326330  
