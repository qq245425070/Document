### 使用说明  
apply方法法人入口参数是T， 出口参数是Observable<R>   
所以map做的事情是，T  to Observable<R>  
```
@CheckReturnValue
@SchedulerSupport(SchedulerSupport.NONE)
public final <R> Observable<R> flatMap(Function<? super T, ? extends ObservableSource<? extends R>> mapper) {
    return flatMap(mapper, false);
}

public interface Function<T, R> {
    R apply(@NonNull T t) throws Exception;
}
```

```
observable.flatMap{ source ->
    return Observable.just().delay(random);
}
```
如果上游事件, 发送多个数据, flatMap会响应多次, 下游收到的事件是无序的; 
具体解释:   
假设上游事件, 发送数据是 "A", "B", "C";  
flatMap 产生的数据是 "1".delay(200ms), "2".delay(100ms), "3".delay(80ms);  
那么下游收到的数据, 顺序是 "3", "2", "1";  
如果想让下游收到的数据是有序的, 可以使用 concatMap;  

### 示例分析  
```
Observable.just("AA", "AA1", "AA2", "AA3")
    .flatMap(new Function<Object, ObservableSource<?>>() {
        @Override
        public ObservableSource<?> apply(Object o) throws Exception {
            if ("AA".equals(o)) {
                return Observable.just("CC: "+o).delay(1, TimeUnit.SECONDS);
            }
            if ("AA1".equals(o)) {
                return Observable.just("CC: "+o).delay(3, TimeUnit.SECONDS);
            }
            if ("AA2".equals(o)) {
                return Observable.just("CC: "+o).delay(4, TimeUnit.SECONDS);
            }
            if ("AA3".equals(o)) {
                return Observable.just("CC: "+o).delay(2, TimeUnit.SECONDS);
            }
            return Observable.just("CC: "+o).delay(new Random(4).nextInt(), TimeUnit.MINUTES);
        }
    })
    .subscribe(new Consumer<Object>() {
        @Override
        public void accept(Object o) throws Exception {
            System.out.println(o+", "+ TimeUtil.currentTimeMillis());
        }
    });
```
如果是 flatMap, 回调函数里面是任务并发执行;  
如果是 concatMap, 回调函数里面的任务是顺序执行, 必须"AA"的 delay(1秒)执行完, 才会执行"AA1"的 delay(3秒);  

### 原理  
flatMap 的整体流向是 upstream.flatMap{ downstream }.subscribe{ print result }  
上游可能会产生多个事件, 也就是 onNext 会执行多次, 每一次多会引起下游, 做出相应;  
假设上游每 1s 执行一次 onNext, 下游每 2s 执行一次 onNext, 那么每次都是上游 onNext 执行完, 下游 onNext 才开始执行;  
依次为一个周期, 一个周期结束, 才会执行下一个周期, 如果下游处理较慢, 上游会阻塞, 一直到下游 onNext 执行完, 才会开始新的一个周期;  
所以, 这里就有一个线程同步的问题;  

### 参考  
https://www.jianshu.com/p/610170c8f3af  
https://www.jianshu.com/p/1fa72188268d  
