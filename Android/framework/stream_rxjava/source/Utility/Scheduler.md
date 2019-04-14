```
Observable.just("你好").subscribeOn(Schedulers.io())  
    .unsubscribeOn(AndroidSchedulers.mainThread())  
    .observeOn(AndroidSchedulers.mainThread());  
```
subscribeOn 切换的是调用 subscribeOn 之前的线程;  
observeOn 切换的是调用 observeOn 之后的线程;  
observeOn 之后, 不可再调用 subscribeOn 切换线程;  

只有第一 subscribeOn() 起作用, 所以多个 subscribeOn() 无意义, 无影响, 无副作用;  
observeOn() 可以使用多次, 每个 observeOn() 将导致一次线程切换(), 都是影响其后的操作的线程;  
不论是 subscribeOn() 还是 observeOn(), 每次线程切换, 如果不受到下一个 observeOn() 的干预, 线程将不再改变, 不会自动切换到其他线程;  
如果没有调用过 observableOn, 那么所有的操作, 都是受 subscribeOn 的影响, 也就是第一次调用 subscribeOn 的影响;  

Schedulers#io = IoScheduler  
Schedulers#newThread = NewThreadScheduler    
Schedulers#computation = ComputationScheduler    
Schedulers#trampoline = TrampolineScheduler    
Schedulers#single = SingleScheduler    
AndroidSchedulers#mainThread = HandlerScheduler  

```
private fun testObservableOn() {
    Observable.just<String>("A")
        .subscribeOn(Schedulers.io())  //  上面的 just 发生在 子线程;
        .map {
            //  由于现在没有调用 observeOn, 所以当前线程还在子线程;
            BaseUtil.isUIThread().logW()
        }
        .observeOn(AndroidSchedulers.mainThread())
        .map {
            //  由于刚才调用了 observeOn(mainThread), 所以现在执行在 UI 线程;
            BaseUtil.isUIThread().logW()
        }
        .observeOn(Schedulers.io())
        .map {
            //  由于刚才调用了 observeOn(io), 所以现在执行在子线程;
            BaseUtil.isUIThread().logW()
        }
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe {
            //  由于刚才调用了 observeOn(mainThread), 所以现在执行在 UI 线程;
            BaseUtil.isUIThread().logW()
        }
    Observable.just<String>("A").observeOn(AndroidSchedulers.mainThread())
        .subscribe(object : LiteConsumer<String>() {
            override fun onNext(p0: String?) {
            }
        })
}
```


### 原理  
subscribeOn  
```
Observable#subscribe(Consumer onNext)  
Observable#subscribe(Consumer onNext,  Consumer onError,  Action onComplete, Consumer onSubscribe)  
Observable#subscribe(Observer observer)  
ObservableSubscribeOn#subscribeActual
Scheduler#scheduleDirect  
IoScheduler.EventLoopWorker#schedule  
NewThreadWorker#scheduleActual  
```

observeOn  
```
.observeOn(AndroidSchedulers.mainThread())
    .subscribe{}
```
为什么 subscribe{ } 可以工作在UI线程?    
在 ObservableObserveOn#subscribeActual 方法中, 把下游的 subscribe 调用处封装在一个 runnable, 再通过 handler 执行这个 runnable;  
1.. 先看调用顺序  
```
Observable#subscribe(Consumer onNext)  
Observable#subscribe(Consumer onNext,  Consumer onError,  Action onComplete, Consumer onSubscribe)  
Observable#subscribe(Observer observer)  
ObservableObserveOn#subscribeActual  
ObservableObserveOn.ObserveOnObserver#onNext  
ObservableObserveOn.ObserveOnObserver#schedule  
HandlerScheduler.HandlerWorker#schedule  
Handler#handleCallback
HandlerScheduler.ScheduledRunnable#run  
```
2.. 切换到UI线程  
```
public Disposable schedule(Runnable run, long delay, TimeUnit unit) {
    if (run == null) throw new NullPointerException("run == null");
    if (unit == null) throw new NullPointerException("unit == null");

    if (disposed) {
        return Disposables.disposed();
    }

    run = RxJavaPlugins.onSchedule(run);

    //  A
    ScheduledRunnable scheduled = new ScheduledRunnable(handler, run);

    //  B
    Message message = Message.obtain(handler, scheduled);
    message.obj = this; // Used as token for batch disposal of this worker's runnables.

    if (async) {
        message.setAsynchronous(true);
    }

    //  C
    handler.sendMessageDelayed(message, unit.toMillis(delay));

    // Re-check disposed state for removing in case we were racing a call to dispose().
    if (disposed) {
        handler.removeCallbacks(scheduled);
        return Disposables.disposed();
    }

    return scheduled;
}

```
BC 处代码的实现逻辑, 怎么能将当前事件, 切换到UI线程?  
Message.callback 为什么会工作在UI线程?  
[参见](/Android/basic/handler/library/message_callback.md)    

