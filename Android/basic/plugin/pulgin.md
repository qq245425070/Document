在 Java 程序中, JVM 虚拟机通过类加载器 ClassLoader 来加载 class 文件;  
Android 与 java 类似, 只不过 Android 使用的是 dalvik/Art 虚拟机来运行 .dex 文件;  
.dex 文件本质上是 .class 文件打包优化而得到的;  
Android 的类加载器, 包括系统和自定义两种;   
系统 classLoader 包括 BootClassLoader, PathClassLoader, DexClassLoader;  

BootClassLoader 是 ClassLoader 的内部类, 用于预加载 preload()常用类以及一些系统 Framework 层级需要的类;  
PathClassLoader 加载系统类和应用程序的类, 如果是加载非系统应用程序类, 则会加载 data/app/目录下的 dex 文件以及包含 dex 的 apk 文件或 jar 文件;  
    可以加载 /data/app 目录下的 apk, 这也意味着, 它只能加载已经安装的 apk;   
DexClassLoader 可以加载自定义的 dex 文件以及包含 dex 的 apk 文件或 jar 文件, 也支持从 SD 卡进行加载;  
URLClassLoader 可以加载 java 中的 jar,但是由于 dalvik 不能直接识别 jar, 所以此方法在 android 中无法使用, 尽管还有这个类  

### 参考  
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
Android动态加载   http://www.androidblog.cn/index.php/Index/detail/id/16#  
Android动态加载   http://blog.csdn.net/u013478336/article/details/50734108（已看，最简单入门）  
Android动态加载   http://www.cnblogs.com/over140/archive/2011/11/23/2259367.html  
Android动态加载   http://blog.csdn.net/u010687392/article/details/47121729?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io（入门详解版本）  
Android动态加载   https://segmentfault.com/a/1190000004062866#articleHeader9（入门基础）  
Android动态加载   https://segmentfault.com/a/1190000004062866（深入L1理解）  
Android动态加载   https://segmentfault.com/a/1190000004062972（深入L2 代理Activity）  
Android动态加载   https://segmentfault.com/a/1190000004077469（深入L2 动态创建Activity）  
Android动态加载   http://www.jianshu.com/p/353514d315a7  

Android插件化开发指南  

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

废物  
https://juejin.im/post/5bf22bb5e51d454cdc56cbd5   
https://juejin.im/post/5c5100c1e51d4550f31755b6  


