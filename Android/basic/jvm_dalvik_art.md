### ART  
Android Runtime (ART) 是 Android 上的应用和部分系统服务使用的托管式运行时;  
ART 及其前身 Dalvik 最初是专为 Android 项目打造的;作为运行时的 ART 可执行 Dalvik 可执行文件并遵循 Dex 字节码规范;  
ART 和 Dalvik 是运行 Dex 字节码的兼容运行时, 因此针对 Dalvik 开发的应用也能在 ART 环境中运作;不过, Dalvik 采用的一些技术并不适用于 ART;  

ART 功能   
以下是 ART 实现的一些主要功能;  
❀ 预先 (AOT) 编译  
ART 推出了预先 (AOT) 编译, 可提高应用的性能;  
ART 还具有比 Dalvik 更严格的安装时验证;  
在安装时, ART 使用设备自带的 dex2oat 工具来编译应用;该实用工具接受 DEX 文件作为输入, 并针对目标设备生成已编译的应用可执行文件;  
该实用工具应能够毫不费力地编译所有有效的 DEX 文件;  但一些后期处理工具（尤其是执行模糊处理的工具）可能会生成被 Dalvik 容忍而被 ART 拒绝的无效文件;    

❀ 优化的垃圾回收  
垃圾回收 (GC) 可能会损害应用的性能, 从而导致显示不稳定、界面响应速度缓慢以及其他问题;ART 通过以下几种方式优化垃圾回收：  
采用一个而非两个 GC 暂停  
在 GC 保持暂停状态期间并行处理  
采用总 GC 时间更短的回收器清理最近分配的短时对象这种特殊情况  
优化了垃圾回收人机工程学, 能够更加及时地进行并行垃圾回收, 这使得 GC_FOR_ALLOC 事件在典型用例中极为罕见  
压缩 GC 以减少后台内存使用和碎片    

❀ 支持更多调试功能  
ART 支持许多新的调试选项, 特别是与监控和垃圾回收相关的功能;例如, 您可以：  
查看堆栈跟踪中保留了哪些锁, 然后跳转到持有锁的线程;
询问指定类的当前活动的实例数、请求查看实例, 以及查看使对象保持有效状态的参考;
过滤特定实例的事件（如断点）;
查看方法退出（使用“method-exit”事件）时返回的值;
设置字段观察点, 以在访问和/或修改特定字段时暂停程序执行;  

❀ 优化了异常和崩溃报告中的诊断详细信息  
当发生运行时异常时, ART 会为您提供尽可能多的上下文和详细信息;ART 会提供更多异常详细信息;  
ART 还通过纳入 Java 和原生堆栈信息在应用原生代码崩溃报告中提供优化的上下文信息;  

### jvm、dalvik、art 对比；  
dvm是基于寄存器的虚拟机, 而jvm是基于栈结构的虚拟机      
❀ dalvik虚拟机 与 JVM, 内存模型  
dalvik虚拟机是基于寄存器的, jvm是基于虚拟栈的；  

基于栈的指令集最主要的优点是可移植性强, 主要的缺点是执行速度相对会慢些；  
而由于寄存器由硬件直接提供, 所以基于寄存器指令集最主要的优点是执行速度快, 主要的缺点是可移植性差;  

❀ dvm速度快  
java虚拟机基于栈结构, 程序在运行时虚拟机需要频繁的从栈上读取写入数据, 这个过程需要更多的指令分派与内存访问次数, 会耗费很多CPU时间;  
dvm虚拟机基于寄存器, 虽然牺牲了一些平台无关性, 但是它在代码的执行效率上要更胜一筹;  

❀  指令数小  
dvm基于寄存器, 所以它的指令是二地址和三地址混合, 指令中指明了操作数的地址；  
jvm基于栈, 它的指令是零地址, 指令的操作数对象默认是操作数栈中的几个位置;  

这样带来的结果就是dvm的指令数相对于jvm的指令数会小很多, jvm需要多条指令而dvm可能只需要一条指令;  

❀  为什么jvm不是基于寄存器结构的  
jvm基于栈带来的好处是可以做的足够简单, 真正的跨平台, 保证在低硬件条件下能够正常运行;  
而dvm操作平台一般指明是ARM系统, 所以采取的策略有所不同;  

需要注意的是dvm基于寄存器, 但是这也是个映射关系, 如果硬件没有足够的寄存器, dvm将多出来的寄存器映射到内存中;  

### dvm的堆结构相对于JVM的堆结构  

Dalvik将堆分成了Active堆和Zygote堆, 这里大家只要知道Zygote堆是Zygote进程在启动的时候预加载的类、资源和对象；  
除此之外的所有对象都是存储在Active堆中的;对于为何要将堆分成Zygote和Active堆, 这主要是因为Android通过fork方法创建到一个新的Zygote进程,   
为了尽可能的避免父进程和子进程之间的数据拷贝, fork方法使用写时拷贝技术, 写时拷贝技术简单讲就是fork的时候不立即拷贝父进程的数据到子进程中,   
而是在子进程或者父进程对内存进行写操作时是才对内存内容进行复制, Dalvik的Zygote堆存放的预加载的类都是Android核心类和java运行时库,   
这部分内容很少被修改, 大多数情况父进程和子进程共享这块内存区域;  
通常垃圾回收重点对Active堆进行回收操作, Dalvik为了对堆进行更好的管理创建了一个Card Table、两个Heap Bitmap和一个Mark Stack数据结构;  

 
### dalvik虚拟机执行的是.dex文件, jvm执行的是.class文件；    

JAVA程序经过编译, 生成JAVA字节码保存在class文件中, JVM通过解码class文件中的内容来运行程序;  
而DVM运行的是Dalvik字节码, 所有的Dalvik字节码由JAVA字节码转换而来, 并被打包到一个DEX（Dalvik Executable）可执行文件中,   
DVM通过解释DEX文件来执行这些字节码;  

dalvik打包.class文件, 方法数受限：多个class文件变成一个dex文件所带来的问题就是方法数超过65535时报错, 由此引出MultiDex技术；  

class文件去冗余：class文件存在很多的冗余信息, dex工具会去除冗余信息, 例如：多个class中的字符串常量合并为一个,   
每个class文件基本都有该字符常量, 存在很大的冗余, 并把所有的.class文件整合到.dex文件中;减少了I/O操作, 提高了类的查找速度;  

  
### dvm 与 art 虚拟机运行字节码 问题  
ART虚拟机直接执行本地机器码, 而Dalvik虚拟机运行的dex字节码需要通过解释器执行;  
在Android5.0中, ART取代了Dalvik虚拟机；  
❀ art虚拟机直接执行本地机器码, 而Dalvik虚拟机运行的dex字节码需要通过解释器执行；  

安卓运行时从Dalvik虚拟机替换成ART虚拟机, 并不要求开发者重新将自己的应用直接编译成目标机器码, 应用程序仍然是一个包含dex字节码的apk文件,   
这主要得益于AOT技术, 也就是在APK运行之前, 就对其包含的Dex字节码进行翻译, 得到对应的本地机器指令, 于是就可以在运行时直接执行了;  

Dalvik是依靠一个Just-In-Time（JIT）解释器去解释字节码, 运行时的代码都需要通过一个解释器在用户的设备上运行, 这一机制并不高效,   
但让应用能更容易在不同硬件和架构上运行;  

art应用安装的时候, 把dex中的字节码编译成本地机器码, 之后每次打开应用, 执行的都是本地机器码;  
取掉了运行时的解释执行, 效率更高, 启动更快;  

所以art占用的内存空间要比dalvik占用的大, 大约10-20%用于存储预编译生成的机器码；    
art应用在安装的时候, 会比dalvik应用慢一点的；  

❀ ART的运行原理：  
在Android系统启动过程中创建的Zygote进程利用ART运行时导出的Java虚拟机接口创建ART虚拟机;  
APK在安装的时候, 打包在里面的classes.dex文件会被工具dex2oat翻译成本地机器指令, 最终得到一个ELF格式的oat文件;  
APK运行时, 上述生成的oat文件会被加载到内存中, 并且ART虚拟机可以通过里面的oatdata和oatexec段找到任意一个类的方法对应的本地机器指令来执行,   
oatdata用来生成本地机器指令的dex文件内容, oatexec含有生成的本地机器指令;  

❀ JIT 与 AOT  

首先了解JIT（Just In Time, 即时编译技术）和AOT(Ahead Of Time, 预编译技术)两种编译模式;  
最早的时候, javac把程序源码编译成JAVA字节码, JVM通过逐条解释字节码将其翻译成对应的机器指令, 逐条读入, 逐条解释翻译,   
执行速度必然比C/C++编译后的可执行二进制字节码程序慢, 为了提高执行速度, 就引入了JIT技术, JIT会在运行时分析应用程序的代码,   
识别哪些方法可以归类为热方法, 这些方法会被JIT编译器编译成对应的汇编代码, 然后存储到代码缓存中, 以后调用这些方法时就不用解释执行了,   
可以直接使用代码缓存中已编译好的汇编代码;这能显著提升应用程序的执行效率;  
安卓Dalvik虚拟机在2.2中增加了JIT, 相对的AOT就是指C/C++这类语言, 编译器在编译时直接将程序源码编译成目标机器码, 运行时直接运行机器码;  


### ART与DVM在GC上的变化  
与 Dalvik 相比, ART 垃圾回收计划在很多方面都有一定的改善：  
与 Dalvik 相比, 暂停次数从 2 次减少到 1 次;Dalvik 的第一次暂停主要是为了进行根标记, 即在 ART 中进行并发标记, 让线程标记自己的根, 然后马上恢复运行;    
与 Dalvik 的另一个主要区别在于 ART GC 引入了移动垃圾回收器;  使用移动 GC 的目的在于通过堆压缩来减少后台应用使用的内存;  
目前, 触发堆压缩的事件是 ActivityManager 进程状态的改变;当应用转到后台运行时, 它会通知 ART 已进入不再“感知”卡顿的进程状态;  

### Dalvik可执行文件体积更小  
class文件中包含多个不同的方法签名, 如果A类文件引用B类文件中的方法, 方法签名也会被复制到A类文件中,   
在虚拟机加载类的连接阶段将会使用该签名链接到B类的对应方法, 也就是说, 多个不同的类会同时包含相同的方法签名,   
同样地, 大量的字符串常量在多个类文件中也被重复使用, 这些冗余信息会直接增加文件的体积,   
而JVM在把描述类的数据从class文件加载到内存时, 需要对数据进行校验、转换解析和初始化, 最终才形成可以被虚拟机直接使用的JAVA类型,   
因为大量的冗余信息, 会严重影响虚拟机解析文件的效率;为了减小执行文件的体积, 安卓使用Dalvik虚拟机,   
SDK中有个dx工具负责将JAVA字节码转换为Dalvik字节码,   dx工具对JAVA类文件重新排列, 将所有JAVA类文件中的常量池分解, 消除其中的冗余信息,   
重新组合形成一个常量池, 所有的类文件共享同一个常量池, 使得相同的字符串、常量在DEX文件中只出现一次, 从而减小了文件的体积;  


### 参考  
https://www.zhihu.com/question/319688949/answer/648358786  
https://source.android.google.cn/devices/tech/dalvik/    
http://blog.csdn.net/evan_man/article/details/52414390  
http://blog.csdn.net/jason0539/article/details/50440669    
https://www.cnblogs.com/qiaoyanlin/p/6743010.html  
https://source.android.google.cn/devices/tech/dalvik/gc-debug  
https://www.jianshu.com/p/a85bc59d6549  
http://loody.github.io/2017/03/30/Dalvik%20vs%20ART/  
https://www.jianshu.com/p/58f817d176b7  
https://zh.wikipedia.org/wiki/Dalvik%E8%99%9A%E6%8B%9F%E6%9C%BA  
https://zh.wikipedia.org/wiki/Java%E8%99%9A%E6%8B%9F%E6%9C%BA  
https://zh.wikipedia.org/wiki/Android_Runtime  
https://www.infoq.cn/article/2016%2F04%2Fandroid-n-aot-jit  
https://www.jianshu.com/p/92227738f270  
https://www.jianshu.com/p/bdb6c29aca83  
http://www.cnblogs.com/lao-liang/p/5111399.html  
https://www.infoq.cn/article/android-in-depth-dalvik  
https://www.zhihu.com/question/29406156  




