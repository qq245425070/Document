[自定义LayoutManager](library/layoutmanager.md)   
[对源码的认识](library/recycler_source.md)   

取消上下拉, 阴影效果  
```
recyclerView.setOverScrollMode(View.OVER_SCROLL_NEVER);  
```
### 对.RecyclerView.的优化  
createViewHolder   bindViewHolder 不要有耗时操作;  
点击事件的回调在 createViewHolder 里面初始化, position 在 bindViewHolder 里面传递;  
数据的刷新操作可以用 notifyItemRange 的形式, 做差异化更新, 而不是全量更新;  
减少布局嵌套, 尽量让 itemView 的高度固定, 调用 RecyclerView.setHasFixedSize(true);  来避免requestLayout浪费资源;  
设置滑动监听, RecyclerView.addOnScrollListener(listener); 减少不必要的耗时操作, 例如视图消失停止动画;  
如果不必要 item 刷新动画, 最好关闭它;  
多个 RecycledView 可以共享 RecycledViewPool;  
尽量避免 RecycleView 嵌套 RecycleView, 如果一定要嵌套的话, 子 View里面的 itemView 一样的话, 可以共用 RecycledViewPool;  

#### 特别的优化  
```
binding.fragmentCandidateBrowseList.setItemViewCacheSize(30);
binding.fragmentCandidateBrowseList.setDrawingCacheEnabled(true);
binding.fragmentCandidateBrowseList.setDrawingCacheQuality(View.DRAWING_CACHE_QUALITY_HIGH);
```
#### 额外预加载  
```
参考  
https://stackoverflow.com/questions/39374227/how-to-load-all-the-images-in-the-background-of-a-recyclerview-in-android/39375267#39375267  

Hi I am working on a Recycler view with 1000 records, where i need to display remote images. Iam loading 20 records at a time.    
6 records can be shown at a time on the screen (based on the screen size of the android device).   
The recycler view is making a req to the backend for the image when the row is shown on the screen,    
which is resulting in taking some time to display it on the screen (I am using picasso library for lazy loading).    
Can you please suggest me how to cache all 20 images in the back ground at a time,   
this way i can show the image immediately to the user when he scrolls down. (this is what our requirement)  


PreCachingLayoutManager layoutManager = new  
PreCachingLayoutManager(getActivity());
layoutManager.setOrientation(LinearLayoutManager.VERTICAL);
layoutManager.setExtraLayoutSpace(DeviceUtils.getScreenHeight(getActivity()));
recyclerView.setLayoutManager(layoutManager);


public class PreCachingLayoutManager extends LinearLayoutManager {
private static final int DEFAULT_EXTRA_LAYOUT_SPACE = 600;
private int extraLayoutSpace = -1;
private Context context;

public PreCachingLayoutManager(Context context) {
    super(context);
    this.context = context;
}

public PreCachingLayoutManager(Context context, int extraLayoutSpace) {
    super(context);
    this.context = context;
    this.extraLayoutSpace = extraLayoutSpace;
}

public PreCachingLayoutManager(Context context, int orientation, boolean reverseLayout) {
    super(context, orientation, reverseLayout);
    this.context = context;
}

public void setExtraLayoutSpace(int extraLayoutSpace) {
    this.extraLayoutSpace = extraLayoutSpace;
}

@Override
protected int getExtraLayoutSpace(RecyclerView.State state) {
    if (extraLayoutSpace > 0) {
        return extraLayoutSpace;
    }
    return DEFAULT_EXTRA_LAYOUT_SPACE;
}
}

```
### 禁用刷新动画  
```
/**
 * 打开局部刷新动画
 */
public void openItemAnimator() {
    isDefaultAnimatorOpen = true;
    this.getItemAnimator().setAddDuration(120);
    this.getItemAnimator().setChangeDuration(250);
    this.getItemAnimator().setMoveDuration(250);
    this.getItemAnimator().setRemoveDuration(120);
    ((SimpleItemAnimator) this.getItemAnimator()).setSupportsChangeAnimations(true);
}

/**
 * 关闭局部刷新动画
 */
public void closeItemAnimator() {
    isDefaultAnimatorOpen = false;
    this.getItemAnimator().setAddDuration(0);
    this.getItemAnimator().setChangeDuration(0);
    this.getItemAnimator().setMoveDuration(0);
    this.getItemAnimator().setRemoveDuration(0);
    ((SimpleItemAnimator) this.getItemAnimator()).setSupportsChangeAnimations(false);
}
```  
### 参考  
http://www.cnblogs.com/dasusu/p/7746946.html  
ListView 与 RecyclerView 简单对比  
https://blog.csdn.net/shu_lance/article/details/79566189  

RecyclerView的拖拽怎么实现的  
https://blog.csdn.net/aiynmimi/article/details/77744610   

RecyclerView 优化  
https://blankj.com/2018/09/29/optimize-recycler-view/  
