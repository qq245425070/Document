### Thread  
Thread 有两种写法, 继承 Thread 和 Runnable, Runnable 主要是为了解决 Java 不能多继承, 又要实现多线程的问题;  

sleep  
让出 CPU 的使用, 让其他线程有机会执行, 继续持有锁;  
sleep 的时间到了, 如果获得的 CPU 资源, 会继续执行;  

wait  
让出 CPU 的使用, 让其他线程有机会执行, 进入挂起队列, 释放锁;  
wait 必须在同步环境下使用, 例如 synchronized 方法或者代码块;  
需要其他线程, 调用 notifyAll, 将其唤醒;  

yield  
让出 CPU 的使用, 让同优先级的线程有机会执行, 如果没有同等优先权的线程, 那么 yield 方法将不会起作用;  


getId 就是获取线程的编号 id;   
stop 停止线程, 一定别去调用它, 它即将被废弃了;   

❀ java 线程阻塞的代价  
java 的线程是映射到操作系统原生线程之上的, 如果要阻塞或唤醒一个线程就需要操作系统介入, 需要在户态与核心态之间切换,  
这种切换会消耗大量的系统资源, 因为用户态与内核态都有各自专用的内存空间, 专用的寄存器等,   
用户态切换至内核态需要传递给许多变量, 参数给内核, 内核也需要保护好用户态在切换时的一些寄存器值, 变量等, 以便内核态调用结束后切换回用户态继续工作;  
如果线程状态切换是一个高频操作时, 这将会消耗很多 CPU 处理时间;  
如果对于那些需要同步的简单的代码块, 获取锁挂起操作消耗的时间比用户代码执行的时间还要长, 这种同步策略显然非常糟糕的;  
synchronized 会导致争用不到锁的线程进入阻塞状态, 所以说它是 java 语言中一个重量级的同步操纵, 被称为重量级锁, 为了缓解上述性能问题;  
JVM 从 1.5 开始, 引入了轻量锁与偏向锁, 默认启用了自旋锁, 他们都属于乐观锁;  

### join  
先看一个 demo, 就是需要 run thread1 和 thread2, 但是在 thread2.run 方法里面, 首先调用 thread1.join,  
意思就是, thread1 的 run 方法执行完, 才会执行 thread2.run 后续的代码块,  
看着好像是 thread2 挂起了, 等 thread1.run 方法结束, 再回退到 thread1.run 的挂起点, 继续之前的字节码行号指示器,  继续运行;  
事实上不是, 仅仅是 while(true){ wait(0) };  
 ```
public class Test {
    public static void main(String[] args) {
        Thread thread1 = new Thread("t-test-1") {
            @Override
            public void run() {
                super.run();
                LogTrack.w("hello, test1 run");
                ThreadUtil.sleep(500);
                LogTrack.w("hello, test1 run 2");
            }
        };

        Thread thread2 = new Thread("t-test-2") {
            @Override
            public void run() {
                super.run();
                try {
                    thread1.join();
                } catch (Exception e) {
                    e.printStackTrace();
                }
                LogTrack.w("hello, test2 run");
            }
        };
        LogTrack.w("main 1");
        thread1.start();
        thread2.start();
        LogTrack.w("main 2");
    }
}
```
实现原理  
thread1.join();  
join(0);   
```
public final synchronized void join(long millis)
throws InterruptedException {
    long base = System.currentTimeMillis();
    long now = 0;
    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }
    if (millis == 0) {
        while (isAlive()) {
            wait(0);
        }
    } else {
        while (isAlive()) {
            long delay = millis - now;
            if (delay <= 0) {
                break;
            }
            wait(delay);
            now = System.currentTimeMillis() - base;
        }
    }
}
```


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