### Thread  
Thread有两种写法, 继承Thread 和 Runnable, Runnable 主要是为了解决Java不能多继承、又要实现多线程的问题；  
sleep 是让线程暂停, 但是依然占用CPU的时间片；  
wait 是让线程等待, 也就是等待CPU的时间片；  
getId 就是获取线程的编号 id；  
stop 停止线程, 一定别去调用它, 它即将被废弃了；  
yield 让出CPU时间片, 但是合适让出, 不确定；就算让出了, 可能也会立刻又获取了；  

❀ java 线程阻塞的代价  
java 的线程是映射到操作系统原生线程之上的, 如果要阻塞或唤醒一个线程就需要操作系统介入, 需要在户态与核心态之间切换,  
这种切换会消耗大量的系统资源, 因为用户态与内核态都有各自专用的内存空间, 专用的寄存器等,   
用户态切换至内核态需要传递给许多变量, 参数给内核, 内核也需要保护好用户态在切换时的一些寄存器值, 变量等, 以便内核态调用结束后切换回用户态继续工作;  
如果线程状态切换是一个高频操作时, 这将会消耗很多 CPU 处理时间;  
如果对于那些需要同步的简单的代码块, 获取锁挂起操作消耗的时间比用户代码执行的时间还要长, 这种同步策略显然非常糟糕的;  
synchronized 会导致争用不到锁的线程进入阻塞状态, 所以说它是 java 语言中一个重量级的同步操纵, 被称为重量级锁, 为了缓解上述性能问题;  
JVM 从 1.5 开始, 引入了轻量锁与偏向锁, 默认启用了自旋锁, 他们都属于乐观锁;  
 

❀ 抛异常停止线程  
```
public static void main(String[] args) throws InterruptedException {
    LocalThread localThread = new LocalThread();
    localThread.start();
    Thread.sleep(3000);
    localThread.interrupt();
    Thread.sleep(10000);
}

private static final class LocalThread extends Thread {
    @Override
    public void run() {
        super.run();
        try {
            for (int i = 0; i < 2000; i++) {
                if (isInterrupted()) {
                    LogTrack.w("中断点, 抛异常停止线程 ");
                    throw new InterruptedException("");
                }
                LogTrack.w("hello " + i);
                Thread.sleep(1000);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```