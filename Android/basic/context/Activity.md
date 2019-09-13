[关于展示吐司和通知](library/open_toast_notification.md)  
[Activity 启动模式与任务栈](library/launchMode.md)  
[清单文件 Activity 标签属性](library/manifest_tag.md)  
[状态栏高度 24dp](ImageFiles/status_bar_height.png)  
[Activity 常见方法](library/function.md)  
[Fragment 重叠异常](library/solution_001.md)  
不管 Activity 是不是被回收, 只要执行 onStop 就一定会执行 onSaveInstanceState;  

Activity 会通过 android:id 逐个恢复 View 的 State;  
也就是说, 如果 android:id 为空, View 将不具备恢复 State 的能力了;  
所有的自定义控件, 都应该实现 State 相关方法, onSaveInstanceState and onRestoreInstanceState;  
一旦 Fragment 从回退栈出来, Fragment 本身还在, View 却是重新创建的;  
但是给 TextView, EditText 设置 android:freezeText="true" 会让其在 Fragment 内, 自动保存 State;  

启动一个 Activity 和 Fragment, 他们的生命周期方法, 调用顺序;  
切换横竖屏;  屏幕锁;  
[链接](library/lifecycle_sample.md)  

### Fragment  

setRetainInstance  
Fragment 具有属性 retainInstance, 默认值为 false;  
当设备旋转时, fragment 会随托管 activity 一起销毁并重建;  

如果 retainInstance 属性值为 false, FragmentManager 会立即销毁该 fragment 实例;  
随后, 为适应新的设备配置, 新的 Activity 的新的 FragmentManager 会创建一个新的 fragment 及其视图;  

如果 retainInstance 属性值为 true, 则该 fragment 的视图立即被销毁, 但 fragment 本身不会被销毁;  
为适应新的设备配置, 当新的 Activity 创建后, 新的 FragmentManager 会找到被保留的 fragment, 并重新创建其视图;   
一旦发生 Activity 重启现象, Fragment 会跳过 onDestroy 直接进行 onDetach(界面消失, 对象还在),  
而 Fragment 重启时也会跳过 onCreate, 而 onAttach 和 onActivityCreated 还是会被调用;  
需要注意的是, 要使用这种操作的 Fragment 不能加入 backStack 后退栈中;  
并且, 被保存的 Fragment 实例不会保持太久, 若长时间没有容器承载它, 也会被系统回收掉的;  

[FragmentManagerImpl, Api21](library/FragmentManagerImplApi21.md)  
getActivity()空指针问题  
❀ 搞清楚, 为什么 f.mActivity() 为空  
如果 app 长时间在后台运行, 再次进入 app 的时候可能会出现 crash, 而且 fragment 会有重叠现象;  
如果系统内存不足, 切换横竖屏, app 长时间在后台运行, Activity 都可能会被系统回收然后重建,  
但 Fragment 并不会随着 Activity 的回收而被回收, 创建的所有 Fragment 会被保存到 Bundle 里面, 从而导致 Fragment 丢失对应的 Activity;  
Fragment 放在 ViewPager 中, ViewPager 只预加载三个, 在跳转到未被预加载的 Item 的时候, 目标 Fragment 也重新创建, 这个时候, 通过 getActivity()获取不到 context;  
❀ 解决办法  
Fragment 中维护一个 全局的 Activity 对象, 在 onAttach 方法中 给其赋值, 在 onDetach 中, 把它置空;  
管理好 Fragment 的生命周期, 在 onCreate 开始监听, 在 onDestroy 时, 解除监听, 忽略掉耗时操作的回调;  



#### commit  

❀ commit()  
在主线程中异步执行, 其实也是 Handler 抛出任务, 等待主线程调度执行;  
commit() 需要在宿主 Activity 保存状态之前调用, 否则会报错;  
这是因为如果 Activity 出现异常需要恢复状态, 在保存状态之后的 commit() 将会丢失, 这和调用的初衷不符, 所以会报错;  

❀ commitAllowingStateLoss()  
commitAllowingStateLoss() 也是异步执行, 但它的不同之处在于, 允许在 Activity 保存状态之后调用, 也就是说它遇到状态丢失不会报错;  

❀ commitNow()  
commitNow() 是同步执行的, 立即提交任务;  
FragmentManager.executePendingTransactions() 也可以实现立即提交事务;  
但我们一般建议使用 commitNow(), 因为另外那位是一下子执行所有待执行的任务, 可能会把当前所有的事务都一下子执行了, 这有可能有副作用;  
此外, 这个方法提交的事务可能不会被添加到 FragmentManger 的后退栈, 因为你这样直接提交, 有可能影响其他异步执行任务在栈中的顺序;  
和 commit() 一样, commitNow() 也必须在 Activity 保存状态前调用, 否则会抛异常;  

attach 与 detach  
transaction.attach(fragment); 对应 onCreateView-onViewCreated-onActivityCreated-onStart-onResume  
transaction.detach(fragment);  对应 onPause-onStop-onDestroyView  


#### transaction  
1.. replace  加入回退栈, Fragment 不销毁, 但是切换回销毁视图和重新创建视图;  
2.. replace  未加回退栈, Fragment 销毁掉;  
3.. hide. show. Fragment 不销毁, 也不销毁视图, 隐藏和显示不走生命周期;  

replace, AFragment 加入回退栈  
在同一个位置, 第一次 replace AFragment, 第二次 replace BFragment;  
```
A: onAttach -> onCreate -> onCreateView -> onActivityCreated -> onStart -> onResume;  
A: onPause -> onStop -> onDestroyView  
B: onAttach -> onCreate -> onCreateView -> onActivityCreated -> onStart -> onResume;  
```

replace, AFragment 未加回退栈  
在同一个位置, 第一次 replace AFragment, 第二次 replace BFragment;  
```
A: onAttach -> onCreate -> onCreateView -> onActivityCreated -> onStart -> onResume;  
A: onPause -> onStop -> onDestroyView -> onDestroy -> onDetach  
B: onAttach -> onCreate -> onCreateView -> onActivityCreated -> onStart -> onResume;  
```


detach 与 attach  
fragmentTransaction.detach(fragmentA);  
```
onPause -> onStop -> onDestroyView  
```
fragmentTransaction.attach(fragmentA);  
```
onCreateView -> onActivityCreated -> onStart -> onResume  
```

#### fragmentManager  

popBackStack(String tag,int flags)  
```
如果  tag = null, flags = 0, 弹出回退栈中最上层的那个 fragment  
如果  tag = null, flags = 1, 弹出回退栈中所有 fragment  
如果  tag != null, flags = 0, 弹出该 fragment 以上的所有 Fragment, 不包括 tag  
如果  tag != null, flags = 1, 弹出该 fragment 以上的所有 Fragment, 包括 tag  
原本 D -> C -> B -> A ;  
执行  
```

未挂载异常  
```
//  "Fragment " + this + " not attached to Activity"  
transaction.add(routerFragment, RouterConfig.FM_TAG);  
transaction.commitAllowingStateLoss();  
manager.executePendingTransactions();  
```

###  参考  
https://inthecheesefactory.com/blog/fragment-state-saving-best-practices/en  
https://github.com/nuuneoi/StatedFragment  

❀ startActivity  
https://github.com/interviewandroid/AndroidInterView/blob/master/android/ams.md  

https://www.jianshu.com/p/a768810c3ff8  
https://blog.csdn.net/stonecao/article/details/6591847  
https://blog.csdn.net/qq_23547831/article/details/51224992  
http://gityuan.com/2016/03/12/start-activity/  
https://github.com/yipianfengye/androidSource/blob/master/14 activity 启动流程.md  
http://aspook.com/2017/02/10/Android-Instrumentation 源码分析(附 Activity 启动流程)/  
深入理解 Android 内核设计思想  
Android 开发艺术探索  
https://blog.csdn.net/pihailailou/article/details/78545391  
https://blog.csdn.net/itachi85/article/details/64123035  
https://blog.csdn.net/qian520ao/article/details/81908505  
https://blog.csdn.net/luoshengyang/article/details/6689748  
https://blog.csdn.net/luoshengyang/article/details/6703247  
https://juejin.im/post/5c4180566fb9a049a62cdfd7  
https://juejin.im/post/5c469b23f265da614933efe8  
https://juejin.im/post/5c483eaff265da61327fa0e3  
https://blog.csdn.net/AndrExpert/article/details/81488503  
https://www.jianshu.com/p/a72c5ccbd150  
https://lrh1993.gitbooks.io/android_interview_guide/content/android/advance/app-launch.html  
https://blog.csdn.net/afu10086/article/details/80140817  

❀ Activity 参考  
http://liuwangshu.cn/framework/ams/2-activitytask.html  
http://liuwangshu.cn/framework/component/6-activity-start-1.html  
http://liuwangshu.cn/framework/component/7-activity-start-2.html  
https://developer.android.com/training/basics/activity-lifecycle/recreating?utm_campaign=adp_series_processes_012016&utm_source=medium&utm_medium=blog  


❀ fragment 参考  
http://toughcoder.net/blog/2015/04/30/android-fragment-the-bad-parts/  
http://www.jianshu.com/p/825eb1f98c19  
https://github.com/AlanCheen/Android-Resources/blob/master/Fragment.md  
https://github.com/JustKiddingBaby/FragmentRigger  
http://blog.csdn.net/u011240877/article/details/78132990#fragment-的使用  
https://www.jianshu.com/p/f2fcc670afd6  
https://www.jianshu.com/p/d9143a92ad94  
https://www.jianshu.com/p/fd71d65f0ec6  
https://www.jianshu.com/p/38f7994faa6b  
https://www.jianshu.com/p/9dbb03203fbc  
https://www.jianshu.com/p/78ec81b42f92  
https://www.jianshu.com/p/c12a98a36b2b  
http://toughcoder.net/blog/2015/04/30/android-fragment-the-bad-parts/  
https://github.com/YoKeyword/Fragmentation/blob/master/README_CN.md  
https://meta.tn/a/15e2d3292a521d700b4fef2f4ebaaa331b8df551431a766ff85b9a5b50c851fc  
https://meta.tn/a/b5b52d1cf21bb9f929cfc2b78b03927c97e74d076d094b12656c6c8c661d3072  

❀ getActivity()空指针问题  参考  
http://blog.csdn.net/goodlixueyong/article/details/48715661  

❀ application 参考  
http://www.jianshu.com/p/f665366b2a47  

❀ context 参考  
http://liuwangshu.cn/framework/context/2-activity-service.html  
https://blog.csdn.net/guolin_blog/article/details/47028975  


❀ onActivityResult  
请放弃使用 类似的库, 因为在 页面被回收, 页面重启后, 回调不会被执行的;   
https://github.com/VictorAlbertos/RxActivityResult  
https://github.com/florent37/InlineActivityResult  
https://github.com/NateWickstrom/RxActivityResult  
https://github.com/nekocode/RxActivityResult  

❀ 启动模式 IntentFilter  
https://juejin.im/post/5c5d85da6fb9a049fd104d8f  

