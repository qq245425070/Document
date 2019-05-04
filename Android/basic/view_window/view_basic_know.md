自定义控件相关 基础知识   

FrameLayout onMeasure 对子View onMeasure 执行2次;  
LinearLayout onMeasure 对子View onMeasure 执行2次;  
RelativeLayout onMeasure 对子View onMeasure 执行4次;  

### onLayout layout  
继承 View, 可以重写 onLayout 和 layout;  
继承 ViewGroup, 只可以重写 onLayout, layout 方法是 final 类型的,  不可以重写;  

onLayout 收到的是, 在父窗体的坐标;  
自定义 ViewGroup 需要在 onMeasure 里面调用 measureChildren 或者, 子 view.measure 方法, 否则, 子 view 的 onMeasure 不会被执行;  

###  getWidth 与 getMeasuredWidth 
getMeasuredWidth 是measure阶段获得的View的原始宽度;  
getWidth 是layout阶段完成后，其在父容器中所占的最终宽度;  

### 修改ScrollView 的阴影效果  
```
在xml中添加：
//  删除android ScrollView边界阴影  
android:fadingEdge="none"
//  删除ScrollView拉到尽头（顶部、底部），然后继续拉出现的阴影效果  
android:overScrollMode="never"
 
scrollView.setHorizontalFadingEdgeEnabled(false);  
 
```
### merge  
使用merge标签, 可以减少过度绘制, 但是问题是, 如果不加上配置, 你看到的preview 是比较奇怪的;  
使用 tools:parentTag 属性, 可以解决这个问题;  
```
<?xml version="1.0" encoding="utf-8"?>
<merge xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    tools:ignore="HardcodedText"
    tools:parentTag="android.widget.RelativeLayout">

    <TextView
        android:id="@+id/title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:paddingLeft="@dimen/padding_horizontal"
        android:paddingTop="48dp"
        android:paddingRight="@dimen/padding_horizontal"
        android:textColor="@color/text_color"
        android:textSize="32sp"
        android:textStyle="bold"
        tools:text="Marquee" />

    <TextView
        android:id="@+id/subtitle"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@+id/title"
        android:paddingLeft="@dimen/padding_horizontal"
        android:paddingTop="4dp"
        android:paddingRight="@dimen/padding_horizontal"
        android:text="hello"
        android:textSize="16sp" />

    <Space
        android:layout_width="0dp"
        android:layout_height="24dp" />

</merge>
```

### 系统屏幕密度  
```
ldpi---120---240X320分辨率
mdpi---160---320X480分辨率
hdpi---240---480X800分辨率
xhdpi---320---720X1280分辨率
xxhdpi---480---1080X1920分辨率
```

### View#post  
```
override fun onResume() {
    super.onResume()
    
    //  0  
    LogTrack.w(recyclerView.measuredHeight)

    recyclerView.post {
        //  精确值  
        LogTrack.w(recyclerView.measuredHeight)
    }

    Handler().postDelayed(Runnable {
        //  0  
        //  和 delay 的时间, 有关系, 但是 delay 多久, 无法预测   
        LogTrack.w(recyclerView.measuredHeight)
    }, 10)
}
```
❀ 参考  
https://blog.csdn.net/scnuxisan225/article/details/49815269  
https://www.jianshu.com/p/7f5342ceba34  

### 参考  
https://juejin.im/post/5c0e1dca6fb9a049c30b12e9  
https://hk.saowen.com/a/185130ca1a868cf4f6746bc998c897fd3ec1d189930d53bf91d000bdcaa6da85  
https://hk.saowen.com/a/88c274f18d44b90a649a68fb39eba3b6ae1e03fd9469bc6177e6d8117f5c0088  

onMeasure  
https://huangtianyu.gitee.io/2018/01/03/自定义控件View之onMeasure调用时机源码分析  

为什么 onMeasure 执行多次  
https://blog.csdn.net/jewleo/article/details/39547631  

Android 屏幕刷新机制  
https://www.jianshu.com/p/0d00cb85fdf3   

touchDelegate  
https://www.jianshu.com/p/cb5181418c7a  
