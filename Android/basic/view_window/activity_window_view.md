[Activity-Window-View层级嵌套结构](../context/ImageFiles/awv_001.jpg)    

PhoneWindow 是在 Activity#attach 方法中创建的;  

Activity#setContentView 方法, 包含以逻辑:   
DecorView 就是一个 FrameLayout;  
DecorView 是在 PhoneWindow 的成员变量, generateDecor 方法中创建;  
DecorView 包含 titleBar 和 mContentParent;  
PhoneWindow#setContentView 把 xml 对应的 View 添加到 mContentParent 上;  

ViewRootImpl 内部持有 mView 作为根视图;  
ActivityThread#handleResumeActivity 方法:    
WindowManagerGlobal 在把 DecorView 关联到 ViewRootImpl#mView 上;  
再调用 ViewRootImpl#requestLayout, 触发 View树的遍历;  


Activity#attach  
```
final void attach(Context context, ActivityThread aThread,
        Instrumentation instr, IBinder token, int ident,
        Application application, Intent intent, ActivityInfo info,
        CharSequence title, Activity parent, String id,
        NonConfigurationInstances lastNonConfigurationInstances,
        Configuration config, String referrer, IVoiceInteractor voiceInteractor,
        Window window, ActivityConfigCallback activityConfigCallback) {
    attachBaseContext(context);
    //  window 初始化  
    mWindow = new PhoneWindow(this, window, activityConfigCallback);
}
```

每一个 Activity 都包含了唯一一个 PhoneWindow, 这个就是 Activity 根 Window;  
在它上面可以增加更多其他的 Window, 如dialog等;  

为什么要设计Activity-View-Window?  
View 是视图,  window是面板, 视图需要挂在到面板上, Activity负责管理面板的生命周期;  

Activity 有存在的必要吗?  
Window 已经是系统管理的窗口界面。那么为什么还需要Activity呢?  
其实, 本质上讲, 我们要显示一个窗口出来, 的确可以不需要 Activity, 悬浮窗口可以不依赖 Activity;  
Android 中的应用中, 对各个窗口的管理相当复杂, 包括窗口的入栈出栈, 焦点问题, 状态管理, 内存回收等等问题, 为了简化开发难度, 引入 Activity 的概念;  

Window 是什么? 它的职能是什么?  
Activity 内部持有 PhoneWindow 对象, 并通过 Window 的 addView(), removeView(), updateViewLayout() 等方法来管理 View 的;  
只给 Window 提供可以在这块区域上绘制图形的 Surface对象, 换句话说, 站在系统的角度上看, 系统是"不知道"有View对象的;  

Activity 的启动需要通过 AMS 完成;  
Window 的添加过程需要通过 WindowSession 完成;  

DecorView  
PhoneWindow  
WindowManagerService  
WindowManager  

### DecorView添加到Window上  
ActivityThread#handleResumeActivity  
```
@Override
public void handleResumeActivity(IBinder token, boolean finalStateRequest, boolean isForward,
        String reason) {
    final ActivityClientRecord r = performResumeActivity(token, finalStateRequest, reason);
    final Activity a = r.activity;
    final int forwardBit = isForward
            ? WindowManager.LayoutParams.SOFT_INPUT_IS_FORWARD_NAVIGATION : 0;
    if (r.window == null && !a.mFinished && willBeVisible) {
        //  获得当前Activity的PhoneWindow对象
        r.window = r.activity.getWindow();
        //  获得当前phoneWindow内部类DecorView对象
        View decor = r.window.getDecorView();
        decor.setVisibility(View.INVISIBLE);
        //  得当当前 Activity 的 WindowManagerImpl 对象
        ViewManager wm = a.getWindowManager();
        WindowManager.LayoutParams l = r.window.getAttributes();
        a.mDecor = decor;
        if (a.mVisibleFromClient) {
            if (!a.mWindowAdded) {
                //  标记根布局DecorView已经添加到窗口
                a.mWindowAdded = true;
                //  将根布局DecorView添加到当前Activity的窗口上面
                wm.addView(decor, l);
            } else {
                a.onWindowAttributesChanged(l);
            }
        }
        
    } else if (!willBeVisible) {
        r.hideForNow = true;
    }
}
```
WindowManagerImpl#addView  
WindowManagerGlobal#addView  
```
public void addView(View view, ViewGroup.LayoutParams params,
        Display display, Window parentWindow) {
    synchronized (mLock) {
        //  获得 ViewRootImpl 对象 root
        root = new ViewRootImpl(view.getContext(), display);
        try {
            //  将 DecorView 关联到 ViewRootImpl 上;
            root.setView(view, wparams, panelParentView);
        } catch (RuntimeException e) {
            throw e;
        }
    }
}
```
ViewRootImpl#setView  
```
public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView) {
    synchronized (this) {
        if (mView == null) {
            //  将顶层视图DecorView赋值给全局的mView
            mView = view;
            //  标记已添加DecorView
            mAdded = true;  
            // 请求布局
            requestLayout();
        }
    }
}
```
requestLayout 的原理  
[链接](/Android/basic/view_window/invalidate_requestLayout.md)  
### 参考  
https://www.jianshu.com/p/8766babc40e0  


