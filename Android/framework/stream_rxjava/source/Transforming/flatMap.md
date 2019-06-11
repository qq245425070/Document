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
如果是flatMap, 回调函数里面是任务并发执行;  
如果是concatMap, 回调函数里面的任务是顺序执行, 必须"AA"的delay(1秒)执行完, 才会执行"AA1"的delay(3秒);  

### 参考  
https://www.jianshu.com/p/610170c8f3af  
https://www.jianshu.com/p/1fa72188268d  
