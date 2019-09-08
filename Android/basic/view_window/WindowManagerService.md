android.view.Window  
com.android.internal.policy.PhoneWindow  
android.view.WindowManager  
android.view.WindowManagerImpl  
android.view.WindowManagerGlobal  
com.android.server.wm.Session  
com.android.server.wm.WindowManagerService  
[DecorView 添加到 Window 上](activity_window_view.md)  
```
android.view.WindowManagerGlobal{
    private static IWindowSession sWindowSession;
    private static IWindowManager sWindowManagerService;
    //  就是 windowMannagerServiceProxy  
    sWindowManagerService = IWindowManager.Stub.asInterface(ServiceManager.getService("window"));
    //  sWindowSession  负责 viewRootImpl  与 windowMangerService 进行通信;  
    IWindowManager windowManager = getWindowManagerService();
                        sWindowSession = windowManager.openSession(
                                new IWindowSessionCallback.Stub() {
                                    @Override
                                    public void onAnimatorScaleChanged(float scale) {
                                        ValueAnimator.setDurationScale(scale);
                                    }
                                },
                                imm.getClient(), imm.getInputContext());
}

com.android.server.wm.Session#addToDisplay{
        return mService.addWindow(this, window, seq, attrs, viewVisibility, displayId,
                outContentInsets, outStableInsets, outOutsets, outInputChannel);
}
```
在 system_server 启动的时候, 会注册 windowManagerService, 
### 参考  
https://www.cnblogs.com/samchen2009/p/3364327.html  
https://www.cnblogs.com/samchen2009/p/3367496.html  
https://www.cnblogs.com/samchen2009/p/3315993.html  
https://www.jianshu.com/p/effaff9ab9f2  
https://www.jianshu.com/p/3528255475a2  

