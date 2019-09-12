窗口和图形系统 - Window and View Manager System;  
显示合成系统 - Surface Flinger;  
用户输入系统 - InputManager System;  
应用框架系统 - Activity Manager System;  
一个 Activity 对应一个 WindowManagerImpl, 一个DecorView(ViewRoot), 以及一个ViewRootImpl, 而 WindowManagerGlobals 是一个全局对象, 一个进程只有一 个;  
WindowManager  是一个接口类, 定义了一些接口来管理 Activity 里的窗口, WindowManager 是 Android 应用进程空间里的一个对象, 不提供 IPC 服务;  
WindowManagerService  是 SystemServer 进程里的一个 Service;  
WindowToken  是在 WindowManagerService 中定义的一个基类, 它是用来标识某一个窗口;  
和下面的 appWindowToken 相比,  它不属于某个特定的 Activity, 比如说输入法窗口, 状态栏窗口等等;  
appWindowToken  它是用来标识 app, 用来标识某个具体的 Activity;  
ApplicationToken  指的是 ActivityRecord 类里的 Token 子类, appWindowToken 里的 appToken 也就是它;  
appToken  和 applicationToken 是一个意思;  

当一个 Window 加入 WindowManagerService 管理时, 必须指定他的 Token 值, WindowManagerService 维护着一个 Token 与 WindowState 的键值 Hash 表;  
Activity 创建, ViewRootImpl 将窗口注册到 WindowManager Service;  
WindowManager Service 通过 SurfaceFlinger 的接口创建了一个 Surface Session 用于接下来的 Surface 管理工作;  
由 Surface Flinger 传上来的 VSYNC 事件到来, Choreographer 会运行 ViewRootImpl 注册的 Callback函数, 这个函数会最终调用 performTraversal 遍历 View 树里的每个 View;  
在第一个 VSYNC 里, WindowManager Service 会创建一个 SurfaceController 对象, ViewRootImpl 根据 Parcel 返回的该对象生成了 Window 对应的 Surface 对象;  
通过这个对象, Canvas 可以要求 Surface Flinger 分配 OpenGL 绘图用的 Buffer;  
View 树里的每个 View 会根据需要依次执行 measure(), layout() 和 draw() 操作;  
Android 3.0 之后引入了硬件加速机制, 为每个 View 生成 DisplayList, 并根据需要在 GPU 内部生成 Hardware Layer, 从而充分利用 GPU 的功能提升图形绘制速度;  
当某个 View 发生变化, 它会调用 invalidate() 请求重绘, 这个函数从当前 View 出发, 向上遍历找到 View Tree 中所有 Dirty 的 View 和 ViewGroup;   
根据需要重新生成 DisplayList, 并在 drawDisplayList() 函数里执行 OpenGL 命令将其绘制在某个 Surface Buffer上;   
最后, ViewRootImpl 调用 eglSwapBuffer 通知 OpenGL 将绘制的 Buffer 在下一个 VSync 点进行显示;  

android.view.Window  是一个抽象类  
com.android.internal.policy.PhoneWindow  继承于 Window  
android.view.ViewManager  是一个接口  
android.view.WindowManager  是一个接口, 继承于 ViewManager  
android.view.WindowManagerImpl  
android.view.WindowManagerGlobal  在整个 app 进程中, 单例存在  {  
    创建 ViewRootImpl 对象;  
}
com.android.server.wm.Session  继承于 IWindowSession.Stub  
每一个应用进程都有一个唯一的 Session 对象与 WMS 通信;  
com.android.server.wm.WindowManagerService 继承于 IWindowManager.Stub  
com.android.server.policy.WindowManagerPolicy  
android.view.ViewParent  
```
android.view.ViewRootImpl 实现 ViewParent  {  
    mWindowSession = WindowManagerGlobal.getWindowSession();  
    final W mWindow = new W(this);  
    android.view.ViewRootImpl.W extends IWindow.Stub {  }
}
```
android.view.IWindowSession  
[app wms ipc](../ImageFiles/app_wms_ipc_001.png)  
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
https://blog.51cto.com/lindt/1864591  
https://www.jianshu.com/p/9e244d13b866  


