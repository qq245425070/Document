### merge  
并发无序  
```
    Observable<String> just1 = Observable.just("1", "2", "3", "4", "5", "6").delay(20, TimeUnit.MILLISECONDS);
    Observable<String> just2 = Observable.just("11", "22", "33", "44", "55", "66").delay(40, TimeUnit.MILLISECONDS);

    List<ObservableSource<String>> sourcesList = new ArrayList<>();
    sourcesList.add(just1);
    sourcesList.add(just2);
    Observable.merge(just1, just2);  
    
    Observable.merge(sourcesList)
            .subscribe(new LiteObserver<Object>() {
                @Override
                public void onNext(Object result) {
                    LogTrack.w(result);
                }
            });
```
### 原理  
```
    Observable.merge(Observable.just(""), Observable.just(""))
            .subscribe(new BaseObserver<Object>() {
                @Override
                public void onNext(Object o) {
                    super.onNext(o);
                }
            });
    public static <T> Observable<T> merge(ObservableSource<? extends T> source1, ObservableSource<? extends T> source2) {
        return fromArray(source1, source2).flatMap((Function)Functions.identity(), false, 2);
    }

```
ObservableFromArray  
```
public final class ObservableFromArray<T> extends Observable<T> {
    @Override
    public void subscribeActual(Observer<? super T> observer) {
        FromArrayDisposable<T> d = new FromArrayDisposable<T>(observer, array);

        observer.onSubscribe(d);

        if (d.fusionMode) {
            return;
        }
        d.run();
    }
}
```

