listView 的 adapter 需要重写 getView 方法, 入口参数 convertView;  
假设一屏展示 N 条, 那么 convertView 初始化 N 次, 最终初始化 N + 1 次;  
之后, 无论怎样滑动, 都不需要再初始化 convertView;  
listView 的缓存机制, 依赖于 RecycleBin 去实现,  他的内部有一些字段:  
mActiveViews : View[];  缓存屏幕上可见的视图;  
mScrapViews: ArrayList<View>[];  这是一个数组, 每一种布局类型的视图都有一个自己的 list 缓存;  
mCurrentScrap: ArrayList<View>;  当前布局类型下的缓存;  
实际上, listView 只有两级缓存, mActiveViews 和 mScrapViews;  
假设一屏展示6条数据, 初始状态展示 0..5, 之后向上滑动, 第6条出现, 再次创建 convertView;  
再次滑动, 第0条划出屏幕, view0 进入 mScrapViews;  
再次滑动, 第7条出现, 复用 mScrapViews 中的 view0;   
再次滑动, 第1条划出屏幕, view1 进入 mScrapViews;  
再次滑动, 第8条出现, 复用 mScrapViews 中的 view1;   
以此类推 ....  
### 参考  
https://juejin.im/post/5a52b0e15188257345015ad3  
