
### LockSupport  
park()  
park(Thread.currentThread())  
阻塞住当前线程;   

unpark()  
unpark(Thread.currentThread());  
唤醒当前线程;   

如果先执行 unpark 再执行 park 就不会阻塞住线程;   
