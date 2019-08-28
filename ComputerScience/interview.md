### 综合  
ConcurrentHashMap;  
thread 之间通信, 怎么通信;  
四大组件, 原理;  
view 测量-绘制原理;  
okHttp 怎么做的3次握手, 什么时候做, socket有哪些api;  
除了 thread.start, 你还用过哪些 api, 对 join 有了解吗?  
Glide 是不是后进来的任务, 优先执行? 

HTTP, keep-Alive 怎么实现的 长连接 和 复用;  
FragmentPagerAdapter 缓存机制,  怎么做 Fragment 的缓存;  
清单文件注册的广播, 什么时候回生效;  
清单文件, 什么时候被解析, 什么时候生效;  
bindService 和 startService 的区别, 在 生命周期上的表现;  
IPC, 有哪些, binder 机制的了解有多少;  
有哪些 OOM, 除了 bitmap string+string, 还有别的么? 
thread 申请过多, 会不会造成 OOM, 为什么?  
你有过哪些性能优化的案例, 能不能再说一些有深入性的案例;  


红黑树, 你了解多少, 时间复杂度 O(lg n), 是怎么推导出来的;  

对象的比较, equals 和 hashCode, 如果, 只重写了 equals, 没有重写 hashCode, a1==a2, 这样的运算, 系统会怎么处理, 为什么?   
hashCode 的重写规则, 哪些时候, 会用到 hashCode, 简单说一下;  

System#arraycopy  实现机制, 时间复杂度  

https://juejin.im/post/5d48e9c36fb9a06af13d50f9  
https://github.com/JsonChao/Awesome-Android-Interview
https://github.com/w4lle/developnote  
https://github.com/JasonWu1111/Android-Review
https://www.jianshu.com/p/089861329c1a  
https://www.jianshu.com/p/0f82b0650909  
https://www.jianshu.com/p/c70989bd5f29  
https://github.com/jwasham/coding-interview-university/blob/master/translations/README-cn.md    
https://github.com/francistao/LearningNotes   
https://github.com/zmywly8866/Worth-Reading-the-Android-technical-articles  
http://whuhan2013.github.io/blog/2016/05/31/job-find-summary/  
http://blog.csdn.net/chivalrousman/article/details/51553114  
https://github.com/suzeyu1992/repo  
https://github.com/karmalove/AndroidInterview  
https://github.com/JackyAndroid/AndroidInterview-Q-A  
http://www.jianshu.com/nb/3450453  
http://www.jianshu.com/p/1e415787efc1  
https://github.com/kdn251/interviews/blob/master/README-zh-cn.md  
http://blog.csdn.net/xiaole0313/article/details/62056612  
https://github.com/CyC2018/Interview-Notebook  
https://github.com/andreis/interview  
https://github.com/Mr-YangCheng/ForAndroidInterview  
https://github.com/Snailclimb/Java-Guide  
https://github.com/CyC2018/CS-Notes  
https://github.com/kdn251/interviews/blob/master/README-zh-cn.md   
https://github.com/Snailclimb/JavaGuide  
https://www.jianshu.com/p/9502b0f9313e  
https://www.jianshu.com/p/a0bc97527d24  
https://www.jianshu.com/p/089861329c1a  
https://www.jianshu.com/p/97d337bd3a9d  
https://www.jianshu.com/p/73e539f0d6e4  
https://juejin.im/post/5c952b5a6fb9a0710c7039b5  
https://www.jianshu.com/p/e6702d61eec9  
https://www.jianshu.com/p/ea4f5d685a9a  
https://juejin.im/post/5caacb2af265da24d320bca9  
https://github.com/interviewandroid/AndroidInterView  
https://github.com/Blankj/AndroidOfferKiller  
https://github.com/hadyang/interview  
https://github.com/Ellen2018/AndroidFaceInterview  

### Java  
ConcurrentHashMap;  
什么年龄代, 用什么回收策略, 什么回收器;  
TreeMap;  

synchronized 与 Lock的区别;  
为什么要有 工作内存 和 主存;  
String 为什么要设计成不可变的;  
泛型, 如果不想 类型擦除, 怎么办;  

说说你对Java反射的理解; 

内部类的作用;  
Java深拷贝和浅拷贝;  
Java finally与return执行顺序;  
类加载机制;  

责任链设计模式, 手写;  
https://github.com/crossoverJie/Java-Interview  

###  Android  
LinearLayout, FrameLayout, RelativeLayout 哪个效率高, 为什么;  

横竖屏切换, 会触发 Activity 什么生命周期;  

图片编解码, BitmapFactory.createBitmap 的时候, 就会涉及编解码;  
LinearLayout 包裹 A, B 两个 Button, 在 AButton 上按下, 一直移动, 手指扫过 BButton, 整个事件传递的过程, MoveEvent 的数值;  

Jvm、Dalvik、Art和对比;  

Application 的 Context 和 Activity 的 Context;  

Service 的生命周期;  
BroadcastReceiver;  
Binder机制及底层实现;  
什么是AIDL 以及如何使用;  
BroadcastReceiver, LocalBroadcastReceiver 区别  

你用过MD, 你知道怎么定义一个Behavior吗?  
https://www.jianshu.com/p/82d18b0d18f4  

AlertDialog, popupWindow, 在 Activity 中的区别;  

多线程断点续传原理;  


Https请求慢的解决办法, DNS, 携带数据, 直接访问IP;  

1.. IPC机制, Binder和匿名共享内存等;  
2.. 四大组件启动, 工作原理;  
3.. View系统, 绘制原理, 事件分发;  
4.. 动画框架, 原理;  
5.. 多线程机制, 消息机制 AsyncTask, Thread / Handler;  
6.. 系统启动过程, system_server启动过程;  
7.. Window系统, Window创建过程;  
8.. 资源管理系统, 资源加载机制等;  

项目相关(Android)  
RecyclerView 的 item 包含一个 横向的列表:  
1.. 如果 item 是 recyclerView , 上下滑动的时候, 会不会卡顿, 为什么;  
2.. 你打算, 怎么处理;  用 LinearLayout 通过 add 和 remove;  
2.. 你打算, 怎么处理;  用 LinearLayout 通过 visible 和 gone;  
如果有 RecyclerView 嵌套, 推荐用 recyclerViewPool;  

图片加解码;  
如果 用列表页面, 播放视频, 怎么做, 可以保证流畅度  占用内存小;  
WebView 怎么解决加载缓慢, 和 缓存问题, 导致数据不能及时刷新;  

https://github.com/Mr-YangCheng/ForAndroidInterview  
https://github.com/Blankj/AndroidOfferKiller  
https://github.com/JackyAndroid/AndroidInterview-Q-A  
https://github.com/francistao/LearningNotes  
https://github.com/leerduo/InterviewQuestion  
https://github.com/Mr-YangCheng/ForAndroidInterview  
http://www.jianshu.com/p/89f19d67b348  
https://github.com/suzeyu1992/repo  
https://www.jianshu.com/p/735be5ece9e8  
https://github.com/Freelander/Android_Data  
http://www.jianshu.com/nb/3450453  
https://github.com/LRH1993/android_interview  
https://juejin.im/post/59e54b9051882578cb511f00  
http://blog.csdn.net/axi295309066/article/details/51275470  
https://www.jianshu.com/p/b5ba11275a6d  
https://www.jianshu.com/p/dfa6d4caedad  
https://www.jianshu.com/p/3df3d2974234    
https://www.jianshu.com/p/a22450882af2  
https://www.jianshu.com/p/fb815eaf628f  
https://juejin.im/post/5aa721936fb9a028d4443d8b  
http://blog.csdn.net/qian520ao/article/details/79601179  
https://github.com/MindorksOpenSource/android-interview-questions  
https://www.jianshu.com/p/c70989bd5f29  
https://github.com/AweiLoveAndroid/CommonDevKnowledge  
https://github.com/guoxiaoxing/android-interview  
https://juejin.im/post/5b97ab465188255c865e030a  
https://juejin.im/post/5b8f15e26fb9a01a031b12d9  
https://github.com/hadyang/interview  
https://www.jianshu.com/p/cf5092fa2694  
https://github.com/huannan/AndroidReview  
https://www.jianshu.com/p/ea4f5d685a9a  


### 算法  
时间复杂度, 推导;  
两个排序数组归并为一个;  
写出你所知道的排序算法及时空复杂度, 稳定性;  
求1000以内的水仙花数以及40亿以内的水仙花数;  
猫扑素数;  
单词反转; 
字符串压缩abbcddde,压缩后ab2cd3e;  
合并多个单有序链表（假设都是递增的）;   
深度优先, 广度优先;  
两个不重复的数组集合中, 求共同的元素;  

二叉树 遍历;  
二叉树 求 最小 公共节点;  
数组 合并;  
随机数组, 包含正负, 找最大子数组;  
一个有序数组, 把奇数放在左边, 偶数放在右边, 怎么实现;  
Stack max()  O(1),  怎么实现? 用一个辅助的 stack;  
实现 0-1000 的数组, 随机乱序, 怎么实现;  
4.找到链表倒数第k个节点
5.二分查找数组中的k, 输出第一个
1、用栈实现队列的功能;  
2、分别用递归和遍历实现二叉树的深度计算;  

1、单链表, 输入一个value, 比如3, 那么把链表中小于等于3的值放在链表左边, 其它放在右边
要求, 元素相对位置不变, 时间复杂度O（1）
2、检测一个二叉树是平衡二叉树  
[4, 5], [5, 8] [6, 10] [11, 15], [14, 15], [17, 18] 区间合并  [4,10] [11, 15] [17, 18]  

10 个数, 循环报数, 是 3 的出队;  

https://github.com/helen-x/AndroidInterview/blob/master/algorithm/swordForOffer/合并两个排序链表.md  
https://github.com/helen-x/AndroidInterview/blob/master/algorithm/swordForOffer/跳台阶.md  
https://github.com/helen-x/AndroidInterview/blob/master/algorithm/swordForOffer/变态跳台阶.md  
https://github.com/helen-x/AndroidInterview/blob/master/algorithm/swordForOffer/数组中出现次数超过一半的数字.md  
https://github.com/helen-x/AndroidInterview/blob/master/algorithm/swordForOffer/二维数组中的查找.md  
https://github.com/helen-x/AndroidInterview/blob/master/algorithm/swordForOffer/二叉树的镜像.md  

### 计算机基础  
utf-8 编码中的中文占几个字节; int型几个字节;  
RSA, AES加密;  
http://www.jianshu.com/p/ef892323e68f  



