窗口和图形系统 - Window and View Manager System;  
显示合成系统 - Surface Flinger;  
用户输入系统 - InputManager System;  
应用框架系统 - Activity Manager System;  
一个 Activity 对应一个 WindowManagerImpl, 一个DecorView(ViewRoot), 以及一个ViewRootImpl, 而 WindowManagerGlobals 是一个全局对象, 一个进程只有一 个;  
WindowManager  是一个接口类, 定义了一些接口来管理 Activity 里的窗口, WindowManager 是 Android 应用进程空间里的一个对象, 不提供 IPC 服务;  
WindowManagerService  是 SystemServer 进程里的一个 Service;  

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

