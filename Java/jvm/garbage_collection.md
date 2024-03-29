JVM 怎么判断对象是否已死, 引用计数算法 和 可达性分析算法;  

### 对象存活分析  
❀ 引用计数算法  
很多教科书判断对象是否存活的算法是这样的: 给对象中添加一个引用计数器,   
每当有一个地方引用它时, 计数器值就加1; 当引用失效时, 计数器值就减1;   
任何时刻计数器为 0 的对象就是不可能再被使用的;  

客观地说, 引用计数算法的实现简单, 判定效率也很高, 在大部分情况下它都是一个不错的算法;  
但是, 至少主流的Java虚拟机里面没有选用引用计数算法来管理内存,   
其中最主要的原因是它很难解决对象之间相互循环引用的问题;  

举个简单的例子, 对象objA 和 objB 都有字段 instance, 赋值令 objA.instance=objB 及 objB.instance=objA,   
除此之外, 这两个对象再无任何引用, 实际上这两个对象已经不可能再被访问, 但是它们因为互相引用着对方,   
导致它们的引用计数都不为 0, 于是引用计数算法无法通知 GC 收集器回收它们;  

❀ 可达性分析算法  
通过一些列的称为"GC Roots"的对象作为起始点, 从这些节点开始向下搜索, 搜索所走过的路径称为引用链;  
当一个对象到 GC Roots 没有任何引用链相连时, 就是从GC Roots 到这个对象是不可达, 则证明此对象是不可用的, 所以它们会被判定为可回收对象;  

可以作为GC Roots的对象包括下面几种:  
由系统类加载器(system class loader), 加载的对象, 通过用户自定义的类加载器加载的类不是 GC Root;  
正在运行的方法中使用的对象;  
正在运行的 native 方法中使用的对象;  
引用数据类型的, 静态常量-变量;  

生存还是死亡    
即使在可达性分析算法中不可达的对象, 也并非是"非死不可"的;  
要真正宣告一个对象死亡, 至少要经历两次标记过程:   
如果对象在进行可达性分析后发现没有与 GC Roots 相连接的引用链, 那它将会被第一次标记并且进行一次筛选;   
当对象没有覆盖 finalize()方法, 或者 finalize()方法已经被虚拟机调用过, 那么这个对象将会放置在一个叫做F-Queue的队列之中;  
稍后 GC 将对 F-Queue 中的对象进行第二次小规模的标记;   
如果对象要在 finalize()中成功拯救自己, 只要重新与引用链上的任何一个对象建立关联即可,   
譬如把自己(this关键字)赋值给某个类变量或者对象的成员变量, 那在第二次标记时它将被移除出"即将回收"的集合;   
如果对象这时候还没有逃脱, 那基本上它就真的被回收了;   
另外一个值得注意的地方是, 代码中有两段完全一样的代码片段, 执行结果却是一次逃脱成功, 一次失败,   
这是因为任何一个对象的 finalize()方法都只会被系统自动调用一次, 如果对象面临下一次回收, 它的finalize()方法不会被再次执行,   

### 垃圾收集算法

标记-清除算法, 复制算法, 标记-整理算法, 分代收集算法   

❀ 标记-清除算法  
先标记, 后清除;  
最基础的收集算法是"标记-清除"(Mark-Sweep)算法, 如同它的名字一样, 算法分为"标记"和"清除"两个阶段;   
首先标记出所有需要回收的对象 , 在标记完成后统一回收所有被标记的对象;   
不足:   
效率问题: 标记和清除两个过程的效率都不高  
空间问题: 标记清除之后产生大量不连续的内存碎片, 空间碎片太多可能会导致以后程序运行过程中需要分配较大对象时, 
无法找到足够的连续内存而不得不提前触发另一次垃圾收集动作;   
  
❀ 复制算法  
内存分两半, 只用一半, 把存活的放到另一半上, 清理一下;  
目的: 为了解决效率问题;   
将可用内存按容量大小划分为大小相等的两块, 每次只使用其中的一块; 当一块内存使用完了, 就将还存活着的对象复制到另一块上面,   
然后再把已使用过的内存空间一次清理掉; 这样使得每次都是对整个半区进行内存回收, 内存分配时也就不用考虑内存碎片等复杂情况; 
缺点: 将内存缩小为了原来的一半;   

现代的商业虚拟机都采用这种收集算法来回收年轻代, IBM公司的专门研究表明, 是年轻代中对象98%对象是"朝生夕死"的,   
所以不需要按照1: 1的比例来划分内存空间, 而是将内存分为较大的 Eden 空间和两块较小的 Survivor 空间, 每次使用 Eden 和其中两块 Survivor;   
HotSpot 虚拟机中默认 Eden 和 Survivor 的大小比例是 8:1:1;   

❀ 标记-整理算法  
活跃的移动到一端, 然后清理边界以外的;  
复制收集算法在对象存活率较高时, 就要进行较多的复制操作, 效率就会变低;  根据年老代的特点, 提出了"标记-整理"算法;    
标记过程仍然与"标记-清除"算法一样, 但后续步骤不是直接对可回收对象进行清理, 而是让所有存活的对象都向一端移动, 然后直接清理掉边界以外的内存;   

###  分代收集算法  
按照 Oracle JMC 的中文翻译, 是 年轻代, 年老代, 永生代;  
一般是把 Java 堆分为年轻代和年老代, 这样就可以根据各个年代的特点采用最适当的收集算法;   
在年轻代中, 每次垃圾收集时都发现有大批对象死去, 只有少量存活, 那就选用复制算法;   
在年老代中, 因为对象存活率高, 没有额外空间对它进行分配担保, 就必须采用"标记-清除"或"标记-整理"算法来进行回收;   

Java虚拟机的堆内存共划分为: 年轻代(Young Generation), 年老代(Tenured Generation), 永生代(Permanent Generation);    
新生对象被分配到 Young Generation 的 Eden 区, 大对象直接被分配到 Tenured Generation;  
当 Eden 区没有足够空间进行分配时, 虚拟机将发起一次 Minor GC;   
所谓的大对象是指, 需要大量连续内存空间的 Java 对象, 最典型的大对象就是那种很长的字符串以及数组, 直接分配到年老代, 是为了让年轻代有更大的内存空间, 减少 GC;  
大对象对虚拟机的内存分配来说就是一个坏消息, 经常出现大对象容易导致内存还有不少空间时就提前触发垃圾收集以获取足够的连续空间来存储它们;  
长期存活的对象将进入年老代, 如果对象在 Eden 出生并经过第一次 Minor GC 后仍然存活, 并且能被 Survivor 容纳的话, 将被移动到 Survivor 空间中, 并且对象年龄设为1;  
对象在 Survivor 区中每 "熬过" 一次 Minor GC, 年龄就增加1岁, 当它的年龄增加到一定程度(默认为15岁), 就将会被晋升到年老代中;  
动态对象年龄判定, 为了能更好地适应不同程序的内存状况, 如果在 Survivor 空间中相同年龄所有对象大小的总和大于 Survivor 空间的一半, 年龄大于或等于该年龄的对象就可以直接进入年老代;   
空间分配担保, 在发生 Minor GC 之前, 虚拟机会先检查年老代最大可用的连续空间是否大于年轻代所有对象总空间, 如果这个条件成立, 那么 Minor GC 可以确保是安全的;   
因此当出现大量对象在 Minor GC 后仍然存活的情况(最极端的情况就是内存回收后年轻代中所有对象都存活), 就需要年老代进行分配担保, 把 Survivor 无法容纳的对象直接进入年老代;   
 
❀ 年轻代  
年轻代分为:  1个 Eden 区和2个 Survivor 区(分别叫 from 和 to), 默认比例为 8:1:1;  
一般情况下, 新创建的对象都会被分配到 Eden 区, 这些对象经过第一次 Minor GC 后, 如果仍然存活, 将会被移到 Survivor 区;   
因为年轻代中的对象大多数都很快被回收, 所以在年轻代的垃圾回收算法使用的是复制算法, 复制算法的基本思想就是将内存分为两块, 
每次只用其中一块, 当这一块内存用完, 就将还活着的对象复制到另外一块上面; 复制算法不会产生内存碎片; 

❀ 永生代    
用于存放静态文件, 如今Java类, 方法等; 持久代对垃圾回收没有显著影响, 但是有些应用可能动态生成或者调用一些class, 例如Hibernate等,   
在这种时候需要设置一个比较大的持久代空间来存放这些运行过程中新增的类; 持久代大小通过-XX:MaxPermSize=<N>进行设置;   

### GC.分类    

年轻代 GC(Minor GC): 指发生在年轻代的垃圾收集动作, 因为 Java 对象大多都具备朝生夕灭的特性, 所以 Minor GC 非常频繁, 一般回收速度也比较快;  
年老代 GC(Major GC/Full GC): 指发生在年老代的 GC, 出现了 Major GC, 经常会伴随至少一次的 Minor GC,  Major GC 的速度一般会比 Minor GC 慢 10 倍以上;  
❀ Minor GC  
从年轻代空间(包括 Eden 和 Survivor 区域)的内存回收, 被称为 Minor GC;  
1.. 当 JVM 无法为一个新的对象分配空间时会触发 Minor GC, 比如当 Eden 区满了, 所以分配率越高, 越频繁执行 Minor GC;  
2.. 会触发 stop-the-world, 对系统几乎无影响;  

Major GC 是清理年老代;  
Full GC 是清理整个堆空间—包括年轻代和年老代;  
1.. 会触发 stop-the-world, 对系统几乎无影响;  

许多 Major GC 是由 Minor GC 触发的, 所以很多情况下将这两种 GC 分离是不太可能的;  
由于 Major GC 会检查所有存活的对象, 因此会花费更长的时间, 圾回收期间让应用反应迟钝;  

对于 Major GC 和 Full GC,  在最初的标记阶段, 会出现 stop-the-world, 停止所有应用程序的线程, 然后开始标记;  
并行执行标记和清洗阶段, 这些都是和应用程序线程并行的;  
最后 Remark 阶段, 会再次暂停所有的事件;  
并行执行清理操作, 不会停止其他线程;  

Full GC  
对整个堆进行整理, 包括 Young, Old, Permanent;  
在对 JVM 调优的过程中, 很大一部分工作就是对于 Full GC 的调节; 有如下原因可能导致 Full GC:   
年老代(Old Generation)被写满;  
持久代(Permanent Generation)被写满;   
System.gc()被显示调用;   
上一次 GC 之后 Heap 的各域分配策略动态变化;  

1.. Full GC == Major GC 指的是对年老代/永生代的 stop the world 的 GC;  
2.. Full GC 的次数 = 年老代 GC 时 stop the world 的次数;  
3.. Full GC 的时间 = 年老代 GC 时 stop the world 的总时间;  
4.. CMS 不等于 Full GC, 我们可以看到 CMS 分为多个阶段, 只有 stop the world 的阶段被计算到了 Full GC 的次数和时间, 而和业务线程并行的 GC 的次数和时间则不被认为是 Full GC;  
5.. Full GC 本身不会先进行 Minor GC, 我们可以配置, 让 Full GC 之前先进行一次 Minor GC, 因为年老代很多对象都会引用到年轻代的对象, 先进行一次 Minor GC 可以提高年老代 GC 的速度;  
      比如年老代使用 CMS 时, 设置 CMSScavengeBeforeRemark 优化, 让 CMS remark 之前先进行一次 Minor GC;  

### 手动尝试垃圾回收  
System.gc();  //  告诉垃圾收集器打算进行垃圾收集, 而垃圾收集器进不进行收集是不确定的 
System.runFinalization();  //  强制调用已经失去引用的对象的 finalize 方法  
```
public void runGc() {
    Runtime.getRuntime().gc();
    this.enqueueReferences();
    System.runFinalization();
}

private void enqueueReferences() {
    try {
        Thread.sleep(100L);
    } catch (InterruptedException var2) {
        throw new AssertionError();
    }
}
```

### 垃圾收集器  

❀ Serial 收集器  
Serial 收集器是一个单线程的收集器, 但它的"单线程"的意义并不仅仅说明它只会使用一个 CPU 或一条收集线程去完成垃圾收集工作,   
更重要的是在它进行垃圾收集时, 必须暂停其他所有的工作线程, 直到它收集结束;  "Stop The World"   
这项工作实际上是由虚拟机在后台自动发起和自动完成的, 在用户不可见的情况下把用户正常工作的线程全部停掉, 这对很多应用来说都是难以接受的;    

实际上到现在为止, 它依然是虚拟机运行在 Client 模式下的默认年轻代收集器;   
它也有着优于其他收集器的地方: 简单而高效(与其他收集器的单线程比), 对于限定单个 CPU 的环境来说, Serial 收集器由于没有线程交互的开销,   
专心做垃圾收集自然可以获得最高的单线程收集效率;  所以, Serial 收集器对于运行在 Client 模式下的虚拟机来说是一个很好的选择;   

❀ ParNew 收集器  
ParNew 收集器是 Serial 收集器的多线程版本, 在多核 CPU 环境下有着比 Serial 更好的表现, 除了使用多条线程进行垃圾收集之外, 其余行为包括 Serial 收集器可用的所有控制参数,     
收集算法,  Stop The World,  对象分配规则,  回收策略等都与 Serial 收集器完全一样;     
ParNew 收集器除了多线程收集之外, 其他与 Serial 收集器相比并没有太多创新之处, 但它却是许多运行在 Server 模式下的虚拟机中首选的年轻代收集器,   
其中有一个与性能无关但很重要的原因是, 除了 Serial 收集器外, 目前只有它能与 CMS 收集器配合工作;    


❀ Parallel Scavenge 收集器  
Parallel Scavenge 收集器是一个年轻代收集器, 它也是使用复制算法的收集器, 又是并行的多线程收集器;  
CMS 等收集器的关注点是尽可能地缩短垃圾收集时用户线程的停顿时间, 而 Parallel Scavenge 收集器的目标则是达到一个可控制的吞吐量(Throughput);    
所谓吞吐量就是 CPU 用于运行用户代码的时间与 CPU 总消耗时间的比值, 即吞吐量 = 运行用户代码时间/(运行用户代码时间+垃圾收集时间);   
虚拟机总共运行了 100 分钟, 其中垃圾收集花掉1分钟, 那吞吐量就是 99%;   
停顿时间越短就越适合需要与用户交互的程序, 良好的响应速度能提升用户体验, 而高吞吐量则可以高效率地利用 CPU 时间, 尽快完成程序的运算任务,   
主要适合在后台运算而不需要太多交互的任务, Parallel Scavenge 被称为吞吐量优先的收集器;     
Parallel Scavenge 可以设置最大-垃圾收集停顿-时间, 

❀ Serial Old 收集器  
年老代单线程收集器, Serial 收集器的年老代版本;  

❀ Parallel Old 收集器  
Parallel Scavenge收集器的年老代版本, 并行收集器, 吞吐量优先;  

❀ CMS 收集器  
Concurrent Mark Sweep, 高并发, 低停顿, 追求最短 GC 回收停顿时间, cpu 占用比较高, 响应时间快, 停顿时间短, 多核 cpu 追求高响应时间的选择;  
初始化标记,  仍需要 stop the world, 初始化标记只是标记一下 GC roots 能直接关联到的对象, 速度很快;    
并发标记,  
重新标记,  仍需要 stop the world, 是为了修正并发标记期间因用户继续操作导致标记变动的那一部分,  这个阶段停顿时间会比初始化标记长一些, 比并发标记时间短一些;    
并发清除,   
在并发阶段, 虽然用户线程不会停下来, 但是它会占据 CPU 资源, 导致 CPU 的吞吐量会降下来;  
只针对年老代进行垃圾回收, 能与 CMS 搭配使用的年轻代垃圾收集器有 Serial 收集器和 ParNew 收集器;  

❀ G1收集器  
Garbage first 是面向服务端的垃圾回收器, 与 CMS 相比:  
并发与并行, 利用多个或者多核 CPU 来缩短 stop the world 的时间, 在其他垃圾回收器原本需要停下来的 GC 动作, G1 仍可以通过并发的方式, 让 java 程序继续执行;   
分代收集, G1 不需要同其他回收器配合就能完成独立管理整个 GC 堆;  
空间整合, G1 整体上看是采用标记-整理的算法实现的, 在运行期间不会产生内存空间碎片, 利于程序长时间运行;  
可预测的停顿, G1 除了降低克顿意外, 还能设置, 在 M 毫秒的时间片内, 消耗在垃圾收集的时间不得超过 N 毫秒;   
初始化标记,  STW  
并发标记,  标记线程与应用程序线程并行执行;  
最终标记, STW  
筛选回收,   

❀ 垃圾回收器组合  
年轻代GC策略	         年老代GC策略	                                                                                  说明    
       Serial	                          Serial Old	                          Serial和Serial Old都是单线程进行GC, 特点就是GC时暂停所有应用线程;   
                                                                                           ---    
       Serial	                    CMS+Serial Old	                      CMS(Concurrent Mark Sweep)是并发GC, 实现GC线程和应用线程并发工作, 不需要暂停所有应用线程; 
                                                                                              另外, 当CMS进行GC失败时, 会自动使用Serial Old策略进行GC; 
                                                                                           ----   
    ParNew                             CMS	                                  使用-XX:+UseParNewGC选项来开启; ParNew是Serial的并行版本, 可以指定GC线程数, 默认GC线程数为CPU的数量;     
                                                                                              可以使用-XX:ParallelGCThreads选项指定GC的线程数;   
                                                                                              如果指定了选项-XX:+UseConcMarkSweepGC选项, 则年轻代默认使用ParNew GC策略;        
                                                                                          ---   
    ParNew                         Serial Old	                              使用-XX:+UseParNewGC选项来开启; 年轻代使用ParNew GC策略, 年老代默认使用Serial Old GC策略; 
                                                                                          ---       
Parallel Scavenge           Serial Old	                              Parallel Scavenge策略主要是关注一个可控的吞吐量: 应用程序运行时间 / (应用程序运行时间 + GC时间),   
                                                                                              可见这会使得CPU的利用率尽可能的高, 适用于后台持久运行的应用程序, 而不适用于交互较多的应用程序;   
                                                                                          ---   
Parallel Scavenge           Parallel Old	                              Parallel Old是Serial Old的并行版本  
     G1GC                            G1GC	                                  -XX:+UnlockExperimentalVMOptions -XX:+UseG1GC        #开启  
                                                                                          -XX:MaxGCPauseMillis =50                                                      #暂停时间目标  
                                                                                          -XX:GCPauseIntervalMillis =200                                             #暂停间隔目标  
                                                                                          -XX:+G1YoungGenSize=512m                                                 #年轻代大小  
                                                                                          -XX:SurvivorRatio=6                                                                 #幸存区比例                    

### 参考  
http://www.importnew.com/26821.html  
https://www.cnblogs.com/ityouknow/p/5614961.html  
GC 分类  
http://www.importnew.com/15820.html  
http://www.importnew.com/14086.html  
https://blog.csdn.net/iter_zc/article/details/41825395  
https://www.cnblogs.com/ityouknow/p/5614961.html  
http://www.lifengdi.com/article/10021.html  
gcRoot  
https://www.yourkit.com/docs/java/help/gc_roots.jsp  