### synchronized  
修饰实例方法, 多个线程, 访问同一个实例的, 加锁方法时, 会出现锁的竞争;  
修饰静态方法, 多个线程, 访问类的加锁方法时, 会出现锁的竞争;  
修饰代码块, 多线程访问到同一个代码块时, 会出现竞争的问题;  

类锁  
```
synchronized(A.class) {
    …
}
```
不同线程中访问这些此⽅法时, 都要对该类对象加锁;  
每个类只有一个类锁;  

对象锁  
```
synchronized(A.this) {
    …
}
```
不同的线程访问该⽅法时, 要对访问的对象加锁;  
每个对象有其独立的对象锁, 互不干扰;  

### 实现原理  

同步代码块:  
同步代码块, 是使用 monitorenter 和 monitorexit 指令实现的;  
monitorenter 指令插入到, 同步代码块的开始位置, monitorexit 指令插入到, 同步代码块的结束位置;  
JVM 需要保证每一个 monitorenter 都有一个 monitorexit 与之相对应;  
任何对象都有一个 monitor 与之相关联, 当且一个 monitor 被持有之后, 他将处于锁定状态;  

简单来说在JVM中 monitorenter 和 monitorexit 字节码, 依赖于底层的操作系统的 Mutex Lock (互斥锁) 来实现的;  
但是由于使用 Mutex Lock 需要将当前线程挂起, 并从用户态切换到内核态来执行, 这种切换的代价是非常昂贵的;  

同步方法:  
在字节码层面, 并没有任何特别的指令, 来实现被 synchronized 修饰的方法,  
而是在 Class 文件的方法表中, 将该方法的 access_flags 字段中的 synchronized 标志位置 1,  
表示该方法是同步方法, 并使用调用该方法的对象, 或该方法所属的 Class, 在JVM的内部对象表示 Klass 做为锁对象;  

synchronized 是基于, 进入和退出, 管程(Monitor)对象实现;  
在 JVM 中, 对象在内存中的布局, 分为3块区域: 对象头, 实例变量, 填充数据;  
头对象:  
实现 synchronized 的锁对象的基础;  
一般而言, synchronized 使用的锁对象, 是存储在 Java 对象头里的;    
每个对象都存在着一个 monitor 与之关联;  
对象与其 monitor 之间的关系有存在多种实现方式;  
例如 monitor 可以与对象一起创建销毁, 当线程试图获取对象锁时自动生成, 但当一个 monitor 被某个线程持有后, 它便处于锁定状态;  
在 Java虚拟机(HotSpot)中, monitor 是由 ObjectMonitor 实现的;  
ObjectMonitor 中有两个队列, _WaitSet 和 _EntryList, 用来保存 ObjectWaiter 对象列表;  
_owner 指向持有 ObjectMonitor 对象的线程;  
当多个线程同时访问,一段同步代码时, 首先会进入 _EntryList 集合,   
当线程获取到对象的 monitor 后, 进入 _Owner 区域, 并把 monitor 中的 owner 变量设置为当前线程, 同时 monitor 中的计数器 count 加1;  
若当前线程调用 wait() 方法, 将释放当前持有的 monitor, owner 变量恢复为 null, count 自减1, 同时该线程进入 WaitSet 集合中等待被唤醒;  
若当前线程执行完毕, 也将释放 monitor(锁) 并复位变量的值, 以便其他线程进入获取 monitor(锁);  

由此看来, monitor 对象存在于每个 Java 对象, 的对象头中(存储的指针的指向);  
synchronized 的锁, 便是通过这种方式获取锁的, 也是为什么 Java 中任意对象可以作为锁的原因;  
同时也是 notify/notifyAll/wait 等方法存在于顶级对象 Object 中的原因;  


synchronized 的可重入性  
从互斥锁的设计上来说, 当一个线程, 试图操作一个, 由其他线程持有的,对象锁的,临界资源时, 将会处于阻塞状态;  
但当一个线程,再次请求自己持有的,对象锁, 的临界资源时, 这种情况属于重入锁;  
请求将会成功, 在 java 中 synchronized 是基于原子性的内部锁机制, 是可重入的,   
因此在一个线程调用 synchronized 方法的同时, 在其方法体内部调用该对象另一个 synchronized 方法,   
也就是说, 一个线程, 得到一个对象锁后, 再次请求该对象锁, 是允许的, 这就是 synchronized 的可重入性;  

wait  
wait()总是在一个循环中被调用, 挂起当前线程来等待一个条件的成立,  wait 调用会一直等到其他线程调用 notifyAll()时才返回;  
当一个线程在执行 synchronized 的方法内部, 调用了 wait()后, 该线程会释放该对象的锁,  然后该线程会被添加到, 该对象的等待队列中(waiting queue),  
只要该线程在等待队列中,  就会一直处于闲置状态, 不会被调度执行;  
要注意 wait()方法会强迫线程先进行释放锁操作, 所以在调用 wait()时, 该线程必须已经获得锁, 否则会抛出异常;  
由于 wait()在 synchronized 的方法内部被执行, 锁一定已经获得,  就不会抛出异常了;  

notify  
当一个线程调用一个对象的 notify()方法时, 调度器会从所有处于该对象等待队列(waiting queue)的线程中取出任意一个线程, 将其添加到入口队列( entry queue) 中;  
然后在入口队列中的多个线程就会竞争对象的锁, 得到锁的线程就可以继续执行, 如果等待队列中(waiting queue)没有线程, notify()方法不会产生任何作用;  
notify() 和 notifyAll()工作机制一样, 区别在于 notifyAll()会将等待队列(waiting queue), 中所有的线程都添加到入口队列中(entry queue);  
注意, notifyAll() 比 notify() 更加常用, 因为notify()方法只会唤起一个线程, 且无法指定唤醒哪一个线程,  
所以只有在多个执行相同任务的线程在并发运行时, 我们不关心哪一个线程被唤醒时, 才会使用notify();  

### synchronized 优化  
❀ 偏向锁:  
有些时候, 在整个同步周期内是没有竞争的, 在这时只有一个线程在运行, 没有其他线程在和它争抢 cpu;  
那么这个时候进行的加锁, 释放锁, 重入锁等等操作, 都是多余且降低性能的, 所以这个时候就进入了偏向锁的状态;  
在这个过程中, 线程可以忽略这些锁, 不会进行锁的操作, 就是好像在偏向这个线程一样, 只有初始化的时候使用一次锁的操作;  
也就是整个过程只有进入偏向锁这个状态使用 CAS 切换了下状态, 其他时候任何的锁和 CAS 都不会做, 偏向锁就是力取在无竞争的情况下将同步都去掉;  
如果此时再有其他线程竞争锁, 那么偏向锁会膨胀为轻量级锁;  

偏向锁的核心思想是, 如果一个线程获得了锁, 那么锁就进入偏向模式, 此时 Mark Word 的结构也变为偏向锁结构;  
当这个线程再次请求锁时, 无需再做任何同步操作, 即获取锁的过程, 这样就省去了大量有关锁申请的操作, 从而也提升了性能;  
所以, 对于没有锁竞争的场合, 偏向锁有很好的优化效果, 毕竟极有可能连续多次是同一个线程申请相同的锁;  
但是对于锁竞争比较激烈的场合, 偏向锁就失效了, 因为这样场合极有可能每次申请锁的线程都是不相同的;  
因此这种场合下不应该使用偏向锁, 否则会得不偿失, 需要注意的是, 偏向锁失败后, 并不会立即膨胀为重量级锁, 而是先升级为轻量级锁;  

判断是否为可偏向状态?  
偏向锁, 获取过程, 判断线程 id 是否指向当前线程;  
是, 即表示这个偏向锁就是这个线程持有,  直接执行代码块;  
否, 通过 CAS 操作竞争锁;  
    竞争成功,  则设置线程 ID 为当前线程, 并执行代码块;  
    竞争失败, 说明多线程竞争啦, 问题严重了, 当偏向锁到达安全点时, 将偏向锁升级为轻量锁;  

偏向锁, 获取过程,         
当偏向锁遇到其他线程, 尝试竞争时, 持有偏向锁的线程会释放, 并升级为轻量锁;   
到达安全点, 暂停拥有偏向锁的线程, 判断锁对象是否处于被锁的状态, 撤销偏向锁后恢复到未锁定(标志位为"01"), 或轻量级锁(标志位为"00")的状态;  

❀ 轻量锁:  
倘若偏向锁失败, 虚拟机并不会立即升级为重量级锁, 它还会尝试使用一种称为轻量级锁的优化手段(1.6之后加入的);  
此时 Mark Word 的结构也变为轻量级锁的结构, 轻量级锁能够提升程序性能的依据是 "对绝大部分的锁, 在整个同步周期内都不存在竞争", 注意这是经验数据;   
需要了解的是, 轻量级锁所适应的场景是线程交替执行同步块的场合, 如果存在同一时间访问同一锁的场合, 就会导致轻量级锁膨胀为重量级锁;  

❀ 自旋锁:  
轻量级锁失败后, 虚拟机为了避免线程真实地在操作系统层面挂起, 还会进行一项称为自旋锁的优化手段;  
毕竟操作系统实现线程之间的切换时需要从用户态转换到核心态, 这个状态之间的转换需要相对比较长的时间, 时间成本相对较高;  
这是基于在大多数情况下, 线程持有锁的时间都不会太长, 因此自旋锁, 会假设, 在不久将来, 当前的线程可以获得锁;  
因此虚拟机会让当前想要获取锁的线程做几个空循环(这也是称为自旋的原因), 一般不会太久, 可能是50个循环或100循环;   
在经过若干次循环后, 如果得到锁, 就顺利进入临界区, 如果还不能获得锁, 那就会将线程在操作系统层面挂起, 这就是自旋锁的优化方式;  
这种方式确实也是可以提升效率的, 最后没办法也就只能升级为重量级锁了;  


锁消除:  
消除锁是虚拟机另外一种锁的优化, 这种优化更彻底, Java虚拟机在JIT编译时(可以简单理解为当某段代码即将第一次被执行时进行编译, 又称即时编译);  
通过对运行上下文的扫描, 去除不可能存在共享资源竞争的锁, 通过这种方式消除没有必要的锁, 可以节省毫无意义的请求锁时间;  
如下 StringBuffer的 append 是一个同步方法, 但是在 add 方法中的 StringBuffer 属于一个局部变量, 并且不会被其他线程所使用;  
因此 StringBuffer 不可能存在共享资源竞争的情景, JVM会自动将其锁消除;  

总结  
简单来讲, 单线程时, 使用偏向锁;  
如果这个时候, 又来了一个线程访问这个代码块, 那么就要升级为轻量锁;  
如果这个线程在访问代码块同时, 又来了一个线程来访问这个代码块, 那么就要升级为重量锁了;  

### 锁.锁的概念  
悲观锁的实现方式: 自旋锁, 自适应自旋锁, 偏向锁, 轻量级锁, 重量级锁;
乐观锁的实现方式: 版本号机制, CAS机制;  

公平锁: 按照申请锁的顺序, 等待获取锁;  
非公平锁: 抢占式, 谁抢到谁占有锁;  
[详见](ReentrantLock.md)  

独享锁, 是指该锁一次只能被一个线程所持有;  
共享锁, 是指该锁可被多个线程所持有;  
ReentrantLock 是独享锁, ReadWriteLock 中, 其读取锁是共享锁, 其写入锁是独享锁;  
Synchronized 是独享锁;  

```
可重入锁 / 不可重入锁
独享锁 / 共享锁
互斥锁 / 读写锁
分段锁
```

❀ 乐观锁  
乐观锁是一种乐观思想, 即认为读操作多, 遇到并发写的可能性低, 每次去拿数据的时候都认为别人不会修改, 所以不会上锁,   
但是在更新的时候会判断一下在此期间别人有没有去更新这个数据,  
在写入时, 先读出当前版本号, 然后加锁操作, 比较跟上一次的版本号, 如果一样则更新, 如果失败则要重执行, 复读-比较-写的操作;  
java 中的乐观锁基本都是通过 CAS 操作实现的, CAS 是一种更新的原子操作, 比较当前值跟传入值是否一样, 一样则更新, 否则失败;  

❀ 悲观锁  
悲观锁是就是悲观思想, 即认为写操作多, 遇到并发写的可能性高, 每次去拿数据的时候都认为别人会修改, 所以每次在读写数据的时候都会上锁,  
这样别人想读写这个数据就会 block 直到拿到锁, java 中的悲观锁就是 synchronized,   
AQS 框架下的锁则是先尝试 cas 乐观锁去获取锁, 获取不到, 才会转换为悲观锁, 如 ReentrantLock;  
### cas.Unsafe  
CAS:   compare and swap  或者 compare and set 硬件同步原语, 比较并交换;  
synchronized 用的是悲观锁, cas 是乐观锁的一种实现形式, 在多核 CPU 机器上会有比较好的性能;  
CAS 只能保证一个共享变量的操作的原子性(原子性操作+原子性操作≠原子操作), 如果要保持多个共享变量的操作的原子性, 就必须使用锁;  

给定一个内存地址, 一个期望值, 一个新值;  
如果内存地址里面的数值, 和期望值相等, 把内存值设置成新值, 返回 true;  
如果内存地址里面的数值, 和期望值不一样, 说明其他线程在操作, 返回 false;  

ABA 解决方案  
ABA 问题可以通过版本号来解决, 每次修改操作都添加一个版本号, 例如刚才的取款操作加个版本号 1, 在存款操作执行后版本号+1, 变为2;  
取款的第二次请求执行时就会判断版本号不是1, 执行失败;  
ABA问题, 原子变量 AtomicStampedReference, AtomicMarkableReference 用于解决 ABA 问题;  


AtomicReference#compareAndSet  
expectedValue: 预期值;  
newValue: 新值;  
当内存值和预期值相等, 把内存值设置为新值, 并返回 true;  
当内存值和预期值不等, 直接并返回 false;  

compareAndSwapInt  
unsafe.compareAndSwapInt(this, valueOffset, expect, update);   
意思是如果 valueOffset 位置包含的值与 expect 值相同, 则更新 valueOffset 位置的值为 update, 并返回 true, 否则不更新, 返回 false;  
```
/**
* 比较 obj 的 offset 处内存位置中的值和期望的值, 如果相同则更新, 此更新是不可中断的;  
* 
* @param obj 需要更新的对象
* @param offset obj 中整型 field 的偏移量
* @param expect 希望 field 中存在的值
* @param update 如果期望值 expect 与 field 的当前值相同, 设置 filed 的值为这个新值
* @return 如果 field 的值被更改返回 true
*/
public native boolean compareAndSwapInt(Object obj, long offset, int expect, int update);
```
### 示例  
```
static final class Entity {
    private static int h;
    synchronized static void sTest() {
        LogTrack.w("静态 方法");
        ThreadUtil.sleep(2000);
        h = 2;
    }

    synchronized static void sTest2() {
        LogTrack.w("静态 方法");
        ThreadUtil.sleep(2000);
        h = 2;
    }

    synchronized void test() {
        LogTrack.w("非静态 方法");
        ThreadUtil.sleep(2000);
        h = 1;
    }

    synchronized void test2() {
        LogTrack.w("非静态 方法");
        ThreadUtil.sleep(2000);
        h = 3;
    }
}
```
假设有多个 thread, 只有一个 Entity 的实例, 那么 在
同时 访问 test, test2, 会发生锁的竞争;  
同时访问 sTest, sTest2, 会发生锁的竞争;  
同时访问 sTest 和 test, 不会发生锁的竞争;  
因为, 静态的同步方法, 锁的对象是 Class, 非静态的同步方法, 锁的对象是当前类的实例;  

### 参考  
http://cmsblogs.com/?p=2071  
https://www.zhihu.com/question/53826114/answer/160222185  
http://www.cnblogs.com/paddix/p/5367116.html  
http://www.cnblogs.com/paddix/p/5405678.html  
https://github.com/farmerjohngit/myblog/issues/12  
https://blog.csdn.net/javazejian/article/details/72828483  
http://www.cnblogs.com/javaminer/p/3889023.html  
https://juejin.im/post/5c37377351882525ec200f9e  
https://blog.csdn.net/lengxiao1993/article/details/52296220  
https://www.jianshu.com/p/f4454164c017  
http://www.open-open.com/lib/view/open1352431526366.html  
https://blog.csdn.net/javazejian/article/details/72828483  
https://juejin.im/post/5d5374076fb9a06ac76da894  
https://blogs.oracle.com/dave/biased-locking-in-hotspot  

cas.unsafe  
https://www.jb51.net/article/136718.htm  
https://juejin.im/post/5c7a86d2f265da2d8e7101a1  
http://www.tianxiaobo.com/2018/05/15/Java-中的-CAS-原理分析/  


