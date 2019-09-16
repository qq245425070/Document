### ContentProvider  

内容提供者, 是 Android 四大组件之一;   
内容提供者, 只是适合于多进程模式下的, 中间层, 数据的搬运工, 并不是数据源本身, 数据源本身可能是db, file等;   
内容提供者, 基于Binder实现;   
[ContentProvider 用法](ContentProvider/InitFun.md)  


相关知识  
[统一资源标识符 URI](/ComputerScience/network/URI.md)   
辅助类 ContentResolver  
辅助类 ContentUris  
辅助类 UriMatcher  
辅助类 ContentObserver  

### 原理  
进程创建的时候, 
context.getContentResolver().query()  
```
android.app.ContextImpl.java  
android.app.ContextImpl;  
android.content.ContentResolver;  
android.app.ContextImpl.ApplicationContentResolver;    
android.content.ContentResolver#query(android.net.Uri, java.lang.String[], android.os.Bundle, android.os.CancellationSignal){
    IContentProvider unstableProvider = acquireUnstableProvider(uri);
}

android.app.ContextImpl.ApplicationContentResolver#acquireUnstableProvider{
    return mMainThread.acquireProvider(c,
                    ContentProvider.getAuthorityWithoutUserId(auth),
                    resolveUserIdFromAuthority(auth), false);
}
public final IContentProvider acquireProvider(Context c, String auth, int userId, boolean stable) {
        final IContentProvider provider = acquireExistingProvider(c, auth, userId, stable);
        if (provider != null) {
            return provider;
        }
        ContentProviderHolder holder = null;
        try {
            holder = ActivityManager.getService().getContentProvider(getApplicationThread(), auth, userId, stable);
        } catch (RemoteException ex) {
            throw ex.rethrowFromSystemServer();
        }
        if (holder == null) {
            return null;
        }
        holder = installProvider(c, holder, holder.info, true /*noisy*/, holder.noReleaseNeeded, stable);
        return holder.provider;
    }
```
android.app.ActivityThread#handleBindApplication  
```
    if (!data.restrictedBackupMode) {
        if (!ArrayUtils.isEmpty(data.providers)) {
            installContentProviders(app, data.providers);
            mH.sendEmptyMessageDelayed(H.ENABLE_JIT, 10*1000);
        }
    }
```
ContentProvider 读取数据, 使用了匿名共享内存, CursorWindow 就是匿名共享内存;  




### 参考  
http://www.jianshu.com/p/ea8bc4aaf057  
http://blog.csdn.net/hehe26/article/details/51784355  
http://blog.csdn.net/liuhe688/article/details/7050868  
http://blog.csdn.net/dmk877/article/details/50387741  

uri  
http://blog.csdn.net/harvic880925/article/details/44679239  
http://www.cnblogs.com/fsjohnhuang/p/4280369.html  
http://blog.csdn.net/whyrjj3/article/details/7852800  

ContentObserver  
http://blog.csdn.net/qinjuning/article/details/7047607  


其他  
http://blog.csdn.net/coder_pig/article/details/47858489  
https://github.com/Triple-T/simpleprovider  

原理  
https://blog.csdn.net/tianmi1988/article/details/51077378  
https://www.jianshu.com/p/37f366064b98  
https://www.jianshu.com/p/37f366064b98  
https://www.jianshu.com/p/9fdc894fb97c    
https://www.jianshu.com/p/3c81df444034  
https://blog.csdn.net/Innost/article/details/47254697  


