简单描述  
HashMap 基于数组, 链表, 红黑树;  
HashMap 同时允许 key, value 都可以为空;  
HashMap 不是 线程安全的, 可以用 synchronized 实现线程安全, 在高并发下, 应该用 ConcurrentHashMap, 建议不要再使用 Hashtable 了;  
HashMap 的默认容量是16;  

数组的每个元素, 称之为箱子(bucket), 箱子里存放着桶(bin);  
桶的数据结构有 2 种可能: 链表, 红黑树;  
HashMap 使用 key 的 hashCode 来寻找存储位置;  
不同的 key 可能具有相同的 hashCode, 这时候就出现哈希冲突了, 也叫做哈希碰撞, 为了解决哈希冲突;   
常见办法有开放地址方法, 链地址方法, HashMap 的实现上选取了链地址方法;  
也就是将哈希值一样的, 但是 equals 是 false 的, entry 保存在同一个数组项里面;   
每一个数组项当做一个桶, 桶里面装的 entry 的 key 的 hashCode 是一样的;  
### Put 操作
如果在创建 HashMap 时, 默认情况下 capacity = 16, loadFactor = 0.75;  
第一次 put 的时候 , 数组 tab 是 null, 需要进行初始化 , 调用 resize 方法进行初始化, 使得 threshold = 16 * 0.75 = 12;  
如果 hashMap 总元素的个数大于 threshold, 则进行扩容;  
如果遇到 hash 冲突的时候, 同一个 hash 位置, 会是一个链表, 要存放第8个节点的时候, 会将链表结构树化;  
在树化的时候, 检查如果 tab 的长度小于 64(MIN_TREEIFY_CAPACITY), 那么会停止树化, 而是选择调用 resize 进行扩容, 使得 capacity = 32, threshold = 24;  
假设仍在上一个 hash 位置, 发生冲突, 则会继续进行树化, 再树化的时候, 会检查 tab 的长度小于 64, 再次扩容, 使得 capacity = 64, threshold = 48;   
假设仍在上一个 hash 位置, 发生冲突, 则会继续进行树化, 再树化的时候, 这个时候, tab 的长度大于等于 64 了, 则真的是需要树化了;  
假设仍在上一个 hash 位置, 发生冲突, 因为上一次冲突已经发生了树化, 所以这次的 node 是 treeNode, 一直在树上挂在节点,  
         一直到 hashMap 总元素的个数大于 threshold = capacity * 0.75, 则会进调用 resize 方法进行扩容;  
在扩容的时候, 会检查箱子中, 桶的数量小于 6 的时候,  就会从树变成链表;  

put 操作的过程  
如果是 key 已存在则修改旧值, 并返回旧值;  
如果 key 不存在, 则执行插入操作, 返回 null;   
如果是插入操作还要 modCount++;   
put 操作, 当发生碰撞时, 如果是使用链表处理冲突, 执行的尾插法;   
这个跟 ConcurrentHashMap 不同, ConcurrentHashMap 执行的是头插法; 因为其 HashEntry 的 next 是 final 的;   
### 怎么计算 hash 表位置  
```
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    }
}        
```
总体来讲就是, 
1.. 计算 h = key.hashCode();  
2.. h >>> 16;  无符号项右位移 16位,  右边的低位被丢弃, 左边的高位补 0;  
3.. 再 把之前的 hashCode 异或 上 位移之后的结果值;  
4.. 取当前的hash数组的长度 n, 当前的 index = (n-1) & hash;  

### 扩容   
因为 hash 值的计算规则是固定的, 所以在扩容的时候, 不需要再一次计算 hash 值;  
而是需要 把老的 hash数组的元素, 迁移到新的 hash 数组;  
hash 数组的索引值, 还是 (length -1) & hash 值;  
扩容过程中判断, hash & length 是 0 还是 1, 如果是 0 , 则待在原来的位置, 如果是 1 , 移动到新的位置;  

### 几个常量的解释  
❀ size(元素的个数)  
size 表示 HashMap 中存放 KV 的数量(为链表和树中的 KV 的总和);  

❀ capacity(容量)  
capacity 就是指 HashMap 中桶的数量, 默认值为 16, 之后是 2 倍, 容量都是 2 的幂;  

❀ loadFactor(装载因子)  
装载因子用来衡量 HashMap 满的程度, loadFactor 的默认值为0.75f;  
计算 HashMap 的实时装载因子的方法为: size/capacity, 而不是占用桶的数量去除以 capacity;  

❀ threshold  
threshold 表示当 HashMap 的元素的个数大于 threshold 时会执行 resize 操作;   
threshold = capacity * loadFactor  
1.. 默认容量  
```
/**
 * The default initial capacity - MUST be a power of two.
 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```
默认初始化的容量为16, 必须是2的幂;  
2.. 最大容量  
```
/**
 * The maximum capacity, used if a higher value is implicitly specified
 * by either of the constructors with arguments.
 * MUST be a power of two <= 1<<30.
 */
static final int MAXIMUM_CAPACITY = 1 << 30;
```
最大容量是 2^30  
3.. 装载因子  
```
/**
 * The load factor used when none specified in constructor.
 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```
默认的装载因子是0.75  
4.. 由链表转换成树的阈值 TREEIFY_THRESHOLD  
```
/**
 * The bin count threshold for using a tree rather than list for a
 * bin.  Bins are converted to trees when adding an element to a
 * bin with at least this many nodes. The value must be greater
 * than 2 and should be at least 8 to mesh with assumptions in
 * tree removal about conversion back to plain bins upon
 * shrinkage.
 */
static final int TREEIFY_THRESHOLD = 8;
```
一个桶中 bin(箱子 )的存储方式由链表转换成树的阈值;  
即当桶中 bin 的数量超过 TREEIFY_THRESHOLD 时使用树来代替链表;  
默认值是 8;  
5.. 由树转换成链表的阈值 UNTREEIFY_THRESHOLD   
```
/**
 * The bin count threshold for untreeifying a (split) bin during a
 * resize operation. Should be less than TREEIFY_THRESHOLD, and at
 * most 6 to mesh with shrinkage detection under removal.
 */
static final int UNTREEIFY_THRESHOLD = 6;
```
一个桶中 bin(箱子 )的存储方式由树转换成链表的阈值;    
当执行 resize 操作时 , 当桶中 bin 的数量少于 UNTREEIFY_THRESHOLD 时使用链表来代替树;    
默认值是 6;     
6.. 表扩容的阈值 MIN_TREEIFY_CAPACITY  
```
/**
 * The smallest table capacity for which bins may be treeified.
 * (Otherwise the table is resized if too many nodes in a bin.)
 * Should be at least 4 * TREEIFY_THRESHOLD to avoid conflicts
 * between resizing and treeification thresholds.
 */
static final int MIN_TREEIFY_CAPACITY = 64;
```
当桶中的 bin 被树化时 , 最小的 hash 表容量;  
当 HashMap 中箱子的数量大于 TREEIFY_THRESHOLD, 小于 MIN_TREEIFY_CAPACITY 时 , 会执行 resize 扩容操作;  
当 HashMap 中箱子的数量大于 MIN_TREEIFY_CAPACITY 时 ,  如果某个桶中的箱子数量大于 8 会被树化;  
当 HashMap 中箱子数量 size  * 0.75 大于 threshold 的时候 , 就会扩容;  

### 参考  
http://yikun.github.io/2015/04/01/Java-HashMap工作原理及实现/  
https://juejin.im/post/5d5d25e9f265da03f66dc517  

HashMap类的注释翻译  
http://blog.csdn.net/fan2012huan/article/details/510859243  

HashMap中capacity, loadFactor, threshold, size等概念的解释  
http://blog.csdn.net/fan2012huan/article/details/51087722  

HashMap的扩容及树化过程  
http://blog.csdn.net/fan2012huan/article/details/51088211  

HashMap源码注解 之 常量定义(一)  
http://blog.csdn.net/fan2012huan/article/details/51094454  

HashMap源码注解 之 成员变量(二)  
http://blog.csdn.net/fan2012huan/article/details/51094525  

HashMap源码注解 之 内部数据结构 Node (三)  
http://blog.csdn.net/fan2012huan/article/details/51097264  

HashMap源码注解 之 静态工具方法hash(), tableSizeFor()(四)  
http://blog.csdn.net/fan2012huan/article/details/51097331  

HashMap源码注解 之 get()方法(五)  
http://blog.csdn.net/fan2012huan/article/details/51130576  

HashMap源码注解 之 put()方法(六)  
http://blog.csdn.net/fan2012huan/article/details/51233378  

HashMap源码注解 之 resize()方法(七)  
http://blog.csdn.net/fan2012huan/article/details/51233540  

http://www.jianshu.com/p/aa017a3ddc40  
http://www.jianshu.com/p/e2f75c8cce01  
http://blog.csdn.net/lianhuazy167/article/details/66967698  

hash 索引计算-扰动函数  
https://www.cnblogs.com/zhengwang/p/8136164.html  
https://my.oschina.net/u/232911/blog/2254278  

