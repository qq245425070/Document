```
public WeakReference(T referent, ReferenceQueue<? super T> q) {
    super(referent, q);
}
```
LeakCanary 主要利用了弱引用的对象, 当GC回收了这个对象后, 会被放进 ReferenceQueue 中;  
在页面消失, 也就是 activity.onDestroy 的时候, 判断利用 idleHandler 发送一条延时消息, 5秒之后,  
分析 ReferenceQueue 中存在的引用, 如果当前 activity 仍在引用队列中, 则认为可能存在泄漏, 再利用系统类 VMDebug 提供的方法, 获取内存快照,  
找出 GC roots 的最短强引用路径, 并确定是否是泄露, 如果泄漏, 建立导致泄露的引用链;  

```
ActivityLifecycleCallbacks{
    @Override public void onActivityDestroyed(Activity activity) {
        //  最终会调用  com.squareup.leakcanary.RefWatcher#ensureGone  
        refWatcher.watch(activity);
    }
}
com.squareup.leakcanary.RefWatcher
Retryable.Result ensureGone(final KeyedWeakReference reference, final long watchStartNanoTime){
    //  1.. 从 retainedKeys 移除掉已经被会回收的弱引用  
    removeWeaklyReachableReferences();
    //  3.. 若当前引用不在 retainedKeys, 说明不存在内存泄漏
    if (gone(reference)) {
        return DONE;
    }
    //  4.. 触发一次gc
    gcTrigger.runGc();
    //  5.. 再次从 retainedKeys 移除掉已经被会回收的弱引用
    removeWeaklyReachableReferences();
    if (!gone(reference)) {
        //  存在内存泄漏  
        long startDumpHeap = System.nanoTime();
        long gcDurationMs = NANOSECONDS.toMillis(startDumpHeap - gcStartNanoTime);
        //  获得内存快照  
        File heapDumpFile = heapDumper.dumpHeap();
        if (heapDumpFile == RETRY_LATER) {
            // Could not dump the heap.
            return RETRY;
        }
        long heapDumpDurationMs = NANOSECONDS.toMillis(System.nanoTime() - startDumpHeap);
        
        HeapDump heapDump = heapDumpBuilder.heapDumpFile(heapDumpFile).referenceKey(reference.key)
          .referenceName(reference.name)
          .watchDurationMs(watchDurationMs)
          .gcDurationMs(gcDurationMs)
          .heapDumpDurationMs(heapDumpDurationMs)
          .build();
        
        heapdumpListener.analyze(heapDump);
    }
    return DONE;
}  
private boolean gone(KeyedWeakReference reference) {
    return !retainedKeys.contains(reference.key);
}
private void removeWeaklyReachableReferences() {
    // WeakReferences are enqueued as soon as the object to which they point to becomes weakly
    // reachable. This is before finalization or garbage collection has actually happened.
    KeyedWeakReference ref;
    while ((ref = (KeyedWeakReference) queue.poll()) != null) {
        retainedKeys.remove(ref.key);
    }
}
```
内存快照  
```
android.os.Debug#dumpHprofData(String fileName){
    dalvik.system.VMDebug.dumpHprofData(fileName);   
}
```
### 参考  
https://www.jianshu.com/p/0df3fdd365ee  
