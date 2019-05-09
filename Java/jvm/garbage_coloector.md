### 垃圾收集器  

### Serial 收集器  
Serial收集器是一个单线程的收集器, 但它的"单线程"的意义并不仅仅说明它只会使用一个CPU或一条收集线程去完成垃圾收集工作,   
更重要的是在它进行垃圾收集时, 必须暂停其他所有的工作线程, 直到它收集结束;  "Stop The World"   
这项工作实际上是由虚拟机在后台自动发起和自动完成的, 在用户不可见的情况下把用户正常工作的线程全部停掉, 这对很多应用来说都是难以接受的;    

实际上到现在为止, 它依然是虚拟机运行在Client模式下的默认新生代收集器;   
它也有着优于其他收集器的地方：简单而高效(与其他收集器的单线程比), 对于限定单个CPU的环境来说, Serial收集器由于没有线程交互的开销,   
专心做垃圾收集自然可以获得最高的单线程收集效率;  所以, Serial收集器对于运行在Client模式下的虚拟机来说是一个很好的选择;   


### ParNew收集器  
ParNew 收集器是 Serial 收集器的多线程版本, 在多核CPU环境下有着比Serial更好的表现, 除了使用多条线程进行垃圾收集之外, 其余行为包括Serial收集器可用的所有控制参数,     
收集算法,  Stop The World,  对象分配规则,  回收策略等都与Serial收集器完全一样；    
ParNew收集器除了多线程收集之外, 其他与Serial收集器相比并没有太多创新之处, 但它却是许多运行在Server模式下的虚拟机中首选的新生代收集器,   
其中有一个与性能无关但很重要的原因是, 除了Serial收集器外, 目前只有它能与CMS收集器配合工作;    


### Parallel Scavenge收集器  
Parallel Scavenge收集器是一个新生代收集器, 它也是使用复制算法的收集器, 又是并行的多线程收集器  
CMS等收集器的关注点是尽可能地缩短垃圾收集时用户线程的停顿时间, 而Parallel Scavenge收集器的目标则是达到一个可控制的吞吐量(Throughput);    
所谓吞吐量就是CPU用于运行用户代码的时间与CPU总消耗时间的比值, 即吞吐量=运行用户代码时间/(运行用户代码时间+垃圾收集时间),   
虚拟机总共运行了100分钟, 其中垃圾收集花掉1分钟, 那吞吐量就是99%;   

停顿时间越短就越适合需要与用户交互的程序, 良好的响应速度能提升用户体验, 而高吞吐量则可以高效率地利用CPU时间, 尽快完成程序的运算任务,   
主要适合在后台运算而不需要太多交互的任务;     


### Serial Old收集器  
老年代单线程收集器, Serial收集器的老年代版本;  

## Parallel Old收集器  
Parallel Scavenge收集器的老年代版本, 并行收集器, 吞吐量优先;  

### CMS收集器  
Concurrent Mark Sweep, 高并发, 低停顿, 追求最短GC回收停顿时间, cpu 占用比较高, 响应时间快, 停顿时间短, 多核cpu 追求高响应时间的选择;  

### G1收集器  


### 垃圾回收器组合  
新生代GC策略	         年老代GC策略	                                                                                  说明    
       Serial	                          Serial Old	                          Serial和Serial Old都是单线程进行GC, 特点就是GC时暂停所有应用线程;   
                                                                                           ---    
       Serial	                    CMS+Serial Old	                      CMS(Concurrent Mark Sweep)是并发GC, 实现GC线程和应用线程并发工作, 不需要暂停所有应用线程; 
                                                                                              另外, 当CMS进行GC失败时, 会自动使用Serial Old策略进行GC; 
                                                                                           ----   
    ParNew                             CMS	                                  使用-XX:+UseParNewGC选项来开启; ParNew是Serial的并行版本, 可以指定GC线程数, 默认GC线程数为CPU的数量;     
                                                                                              可以使用-XX:ParallelGCThreads选项指定GC的线程数;   
                                                                                              如果指定了选项-XX:+UseConcMarkSweepGC选项, 则新生代默认使用ParNew GC策略;        
                                                                                          ---   
    ParNew                         Serial Old	                              使用-XX:+UseParNewGC选项来开启; 新生代使用ParNew GC策略, 年老代默认使用Serial Old GC策略; 
                                                                                          ---       
Parallel Scavenge           Serial Old	                              Parallel Scavenge策略主要是关注一个可控的吞吐量：应用程序运行时间 / (应用程序运行时间 + GC时间),   
                                                                                              可见这会使得CPU的利用率尽可能的高, 适用于后台持久运行的应用程序, 而不适用于交互较多的应用程序;   
                                                                                          ---   
Parallel Scavenge           Parallel Old	                              Parallel Old是Serial Old的并行版本  
     G1GC                            G1GC	                                  -XX:+UnlockExperimentalVMOptions -XX:+UseG1GC        #开启  
                                                                                          -XX:MaxGCPauseMillis =50                   #暂停时间目标  
                                                                                          -XX:GCPauseIntervalMillis =200          #暂停间隔目标  
                                                                                          -XX:+G1YoungGenSize=512m              #年轻代大小  
                                                                                          -XX:SurvivorRatio=6                                #幸存区比例                                                                                                 

                                                                                              
### 参考  
https://www.cnblogs.com/ityouknow/p/5614961.html  
http://www.lifengdi.com/article/10021.html  
