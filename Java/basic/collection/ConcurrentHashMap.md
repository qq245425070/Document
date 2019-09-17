Node<K,V>[] table;  
ConcurrentHashMap 在 java8 已经摒弃了 Segment 的概念, 而是直接使用 Node 数组+链表+红黑树的数据结构来实现;  
使用 synchronized 和 CAS 控制并发操作;  

### 名词解释  
static final int MOVED     = -1;  //  表示正在转移  
static final int TREEBIN   = -2;  //  表示已经转换成树  
private transient volatile Node<K,V>[] nextTable;  //  转移的时候用的数组  

private transient volatile int sizeCtl;    
负数代表正在进行初始化或扩容操作;    
-1 代表正在初始化;    
-N 表示有 N-1 个线程正在进行扩容操作;    
0 代表 hash 表还没有被初始化;    
当为正数时 , 表示初始化或者下一次进行扩容的大小;    

private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;    
2^15-1, help resize 的最大线程数    

private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;    
32-16=16, sizeCtl 中记录 size 大小的偏移量;    
  
ForwardingNode    
一个特殊的 Node 节点 , hash 值为 -1, 其中存储 nextTable 的引用 , 只有 table 发生扩容的时候 , ForwardingNode 才会发挥作用;    
作为一个占位符放在 table 中表示当前节点为 null 或则已经被移动;    

CAS  
在 ConcurrentHashMap 中, 大量使用了 U.compareAndSwapXXX 的方法, 这个方法是利用一个 CAS 算法实现无锁化的修改值的操作, 他可以大大降低锁代理的性能消耗;   
### 扩容函数 transfer   

支持多线程进行扩容操作, 并没有加锁 , 这样做的目的不仅仅是为了满足 concurrent 的要求, 而是希望利用并发处理去减少扩容带来的时间影响;     
单线程扩容的大体思想就是遍历, 复制的过程; 首先根据运算得到需要遍历的次数 i, 然后利用 tabAt 方法获得 i 位置的元素:     
  
如果这个位置为空, 就在原 table 中的 i 位置放入 forwardNode 节点, 这个也是触发并发扩容的关键点;     
如果这个位置是 Node 节点(fh>=0), 如果它是一个链表的头节点, 就构造一个反序链表, 把他们分别放在 nextTable 的 i 和 i+n 的位置上    
如果这个位置是 TreeBin 节点(fh<0), 也做一个反序处理, 并且判断是否需要 untreefi, 把处理的结果分别放在 nextTable 的 i 和 i+n 的位置上    
遍历过所有的节点以后就完成了复制工作, 这时让 nextTable 作为新的 table, 并且更新 sizeCtl 为新容量的 0.75倍, 完成扩容;     
多线程遍历节点, 处理了一个节点, 就把对应点的值 set 为 forward, 另一个线程看到 forward, 就向后继续遍历, 再加上给节点上锁的机制, 就完成了多线程的控制;   
这样交叉就完成了复制工作; 而且还很好的解决了线程安全的问题;     


### putVal  
如果没有初始化就先调用 initTable()方法来进行初始化过程;  
如果没有 hash 冲突就直接 CAS 插入;  
如果还在进行扩容操作就先进行扩容;  
如果存在 hash 冲突 , 就加锁来保证线程安全 , 这里有两种情况 , 一种是链表形式就直接遍历到尾端插入 , 一种是红黑树就按照红黑树结构插入 ,     
最后一个如果该链表的数量大于阈值 8, 就要先转换成黑红树的结构 , break 再一次进入循环    
如果添加成功就调用 addCount()方法统计 size, 并且检查是否需要扩容    
```
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    //  再次 hash, hash 值均匀分布, 减少 hash 冲突;    
    int hash = spread(key.hashCode());  
    int binCount = 0;  //  用来计算在这个节点总共有多少个元素，用来控制扩容或者转移为树
    //  类似于while(true), 死循环, 保证插入成功  
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh; K fk; V fv;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();  //  如果 hash 表为空, 初始化 hash表  initTable;  
        
        //   table[i] 为空, 利用 CAS 在 table[i] 头结点直接插入, 
        //  如果插入成功, 退出插入操作;  
        //  如果插入失败, 则有其他节点已经插入, 继续下一步;  
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {  
             //   如果 hash 值对应的位置, 没有数据, 用 CAS 将 value 放进去, 插入成功, 结束 putVal;
            if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value)))
                break;                   // no lock when adding to empty bin
        }
        
        //  需要扩容 , 如果 table[i] 不为空, 且 table[i] 的 hash 值为 -1, 则有其他线程在执行扩容操作, 帮助他们一起扩容, 提高性能  
        else if ((fh = f.hash) == MOVED)  
            tab = helpTransfer(tab, f);
        
        //  key value都存在, 表示重复,  结束 putVal;  
        else if (onlyIfAbsent && fh == hash &&  // check first node
                 ((fk = f.key) == key || fk != null && key.equals(fk)) &&
                 (fv = f.val) != null)      
            return fv;
        
        //  如果以上条件都不满足, 也就是存在 hash 冲突, 那就要进行加锁操作, 锁住链表或者红黑树的头结点;  
        else {
            V oldVal = null;
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    //  fh >= 0, 表示该节点是链表结构, 将该节点插入到链表尾部  
                    if (fh >= 0) {  
                        binCount = 1;
                        //  先查找链表中是否出现了此 key, 如果出现, 则更新 value, 并跳出循环;  
                        //  否则将节点加入到链表末尾并跳出循环;  
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            //  key相同, 替换原先的value
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                //  仅 putIfAbsent()方法中 onlyIfAbsent 为 true  
                                if (!onlyIfAbsent)  
                                    e.val = value;  //  putIfAbsent() 包含 key 则返回 get, 否则 put 并返回 ;  
                                break;
                            }
                            Node<K,V> pred = e;
                            //  在链表尾, 插入新的节点  
                            if ((e = e.next) == null) {  
                                pred.next = new Node<K,V>(hash, key, value);
                                break;
                            }
                        }
                    }  
                    //  如果是树节点, 这个过程采用同步内置锁实现并发  
                    else if (f instanceof TreeBin) {  
                        Node<K,V> p;
                        binCount = 2; 
                        //  插入节点, 并旋转红黑树  
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                    else if (f instanceof ReservationNode)  //  如果是 预留节点
                        throw new IllegalStateException("Recursive update");
                }
            }
            //  到此时, 已将键值对插入到了合适的位置, 检查链表长度是否超过阈值, 若是, 则转变为红黑树结构
            if (binCount != 0) {
                //  同一个 hash 位置, 链表的长度 >=8  就要树化  
                if (binCount >= TREEIFY_THRESHOLD)  
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    //  count+1, 如有必要, 则扩容  
    addCount(1L, binCount);  //  统计size, 并且检查是否需要扩容
    return null;
}
```
1.. 如果待插入的键值对中 key 或 value 为 null, 抛出异常, 结束, 否则执行 2  
2.. 如果 table 为 null, 则进行初始化操作 initTable(), 否则执行3   
3.. 如果 table[i] 为空, 则用 CAS 在 table[i] 头结点直接插入, 
      如果 CAS 执行成功, 退出插入操作, 执行步骤 7;  
      如果 CAS 失败, 则说明有其他节点已经插入, 执行 4;  
4.. 此时判断, hash 值是否为 MOVED(-1), 如果是则说明, 有其他线程在执行扩容操作, 帮助他们一起扩容, 来提高性能, 如果没有在扩容, 那么执行 5;  
5.. 判断 hash 的值, 如果 >=0, 则在链表合适的位置插入, 否则查看 table[i] 是否是红黑树结构, 如果是, 则在红黑树适当位置插入;  
      到此时, 键值对已经顺利插入, 接下来执行 6;  
6.. 如果 table[i] 节点数 binCount 不为 0, 判断它此时的状态, 是否需要转变为红黑树;  
7.. 执行 addCount(1L,  binCount);  

### initTable  
```
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {   //  hash表为空, 进行初始化  
        if ((sc = sizeCtl) < 0)  //  sizeCtl<0表示其他线程已经在初始化了或者扩容了, 挂起当前线程 
            Thread.yield(); 
        //  如果该线程获取了初始化的权利, 则用 CAS 将 sizeCtl 设置为 -1, 表示本线程正在初始化
        else if (U.compareAndSetInt(this, SIZECTL, sc, -1)) {  
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2);  //  记录下次扩容的大小, 即 sc = 0.75n
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```
### helpTransfer  
```
final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
    Node<K,V>[] nextTab; int sc;
    //  如果 table 不是空, 且 node 节点是转移类型, 数据检验;  
    //  且 node 节点的 nextTable(新 table) 不是空, 同样也是数据校验
    if (tab != null && (f instanceof ForwardingNode) &&
        (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) {  //  新的table nextTba已经存在前提下才能帮助扩容
        int rs = resizeStamp(tab.length);
        //  如果 nextTab 没有被并发修改, 且 tab 也没有被并发修改, 且 sizeCtl  < 0 (说明还在扩容)
        while (nextTab == nextTable && table == tab &&
               (sc = sizeCtl) < 0) {
            //  如果 sizeCtl 无符号右移  16 不等于 rs (sc前 16 位如果不等于标识符, 则标识符变化了) 
            //  或者 sizeCtl == rs + 1 , 扩容结束了, 不再有线程进行扩容, 默认第一个线程设置 sc ==rs 左移 16 位 + 2,  
            //  当第一个线程结束扩容了, 就会将 sc 减一, 这个时候 sc 就等于 rs + 1;  
            //  或者 sizeCtl == rs + 65535  , 如果达到最大帮助线程的数量65535;
            //  或者转移下标正在调整 , 扩容结束;  
            if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                sc == rs + MAX_RESIZERS || transferIndex <= 0)
                break;
            //  如果以上都不是, 将 sizeCtl + 1, 表示增加了一个线程帮助其扩容
            if (U.compareAndSetInt(this, SIZECTL, sc, sc + 1)) {
                transfer(tab, nextTab);  //  调用扩容方法  
                break;
            }
        }
        return nextTab;
    }
    return table;
}
```
### transfer  
把数组中的节点复制到新的数组的相同位置, 或者移动到扩张部分的相同位置;  
在这里首先会计算一个步长, 表示一个线程处理的数组长度, 用来控制对 CPU 的使用;  
每个 CPU 最少处理 16 个长度的数组元素, 也就是说, 如果一个数组的长度只有 16, 那只有一个线程会对其进行扩容的复制移动操作;  
扩容的时候会一直遍历, 知道复制完所有节点, 没处理一个节点的时候会在链表的头部设置一个 fwd 节点, 这样其他线程就会跳过他;  
复制后在新数组中的链表不是绝对的反序的;  
```
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
    int n = tab.length, stride;
    //  每核处理的量小于16, 则强制赋值16
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
        stride = MIN_TRANSFER_STRIDE; // subdivide range
    //  如果复制的目标 nextTab 为 nul l的话, 则初始化一个 table 两倍长的 nextTab
    //  此时 nextTable 被设置值了(在初始情况下是为null的)
    //  因为如果有一个线程开始了表的扩张的时候, 其他线程也会进来帮忙扩张;
    //  而只是第一个开始扩张的线程需要初始化下目标数组
    if (nextTab == null) {            // initiating
        try {
            @SuppressWarnings("unchecked")
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
            nextTab = nt;
        } catch (Throwable ex) {      // try to cope with OOME
            sizeCtl = Integer.MAX_VALUE;
            return;
        }
        nextTable = nextTab;
        transferIndex = n;
    }
    int nextn = nextTab.length;
    //  ForwardingNode 是一个标示类, 它的 hash 字段值为  -1(MOVED), 
    //  如果看到节点为  ForwardingNode 类表示这个位置已经被处理过了, 
    //  这个位置上面的数据已经被搬走了, 不需要处理了;  
    //  advance 为 true 表示当前节点已经处理完了, 可以继续处理下一个节点;  
    ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
    boolean advance = true;  //  当advance == true时, 表明该节点已经处理过了
    boolean finishing = false; //  在完成之前重新在扫描一遍数组, 看看有没完成的
    for (int i = 0, bound = 0;;) {
        Node<K,V> f; int fh;
        while (advance) {
            int nextIndex, nextBound;
            if (--i >= bound || finishing)
                advance = false;
            else if ((nextIndex = transferIndex) <= 0) {
                i = -1;
                advance = false;
            }
            //  用CAS计算得到的transferIndex  
            else if (U.compareAndSetInt
                     (this, TRANSFERINDEX, nextIndex,
                      nextBound = (nextIndex > stride ?
                                   nextIndex - stride : 0))) {
                bound = nextBound;
                i = nextIndex - 1;
                advance = false;
            }
        }
        if (i < 0 || i >= n || i + n >= nextn) {
            int sc;
            //  已经完成所有节点复制了
            if (finishing) {
                nextTable = null;
                table = nextTab;
                sizeCtl = (n << 1) - (n >>> 1);  //  sizeCtl阈值为原来的1.5倍  
                return;
            }
            //  CAS 更扩容阈值, 在这里面sizectl值减一, 说明新加入一个线程参与到扩容操作
            if (U.compareAndSetInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                    return;
                finishing = advance = true;
                i = n; // recheck before commit
            }
        }
        //  遍历的节点为null, 则放入到ForwardingNode 指针节点
        else if ((f = tabAt(tab, i)) == null)
            advance = casTabAt(tab, i, null, fwd);
        //  f.hash == -1 表示遍历到了ForwardingNode节点, 意味着该节点已经处理过了
        //  这里是控制并发扩容的核心
        else if ((fh = f.hash) == MOVED)
            advance = true; // already processed
        else {
            //  fh >= 0 节点的 hash 值大于 0, 表示是链表;  红黑树的 hash 值似是负数, 
            //  将迁移过的数组赋值为 ForwardingNode 节点, 记录为已迁移的状态;  
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    Node<K,V> ln, hn;
                    if (fh >= 0) {
                         //  构造两个链表  一个是原链表  另一个是原链表的反序排列
                        int runBit = fh & n;
                        Node<K,V> lastRun = f;
                        //  lastRun 表示的是需要复制的最后一个节点
                        //  每当新节点的 hash&n -> b 发生变化的时候, 就把 runBit 设置为这个结果 b
                        //  这样 for 循环之后, runBit 的值就是最后不变的 hash&n 的值
                        //  而 lastRun 的值就是最后一次导致 hash&n 发生变化的节点(假设为p节点)
                        //  为什么要这么做呢, 因为 p 节点后面的节点的 hash&n 值跟 p节点是一样的,
                        //  所以在复制到新的 table 的时候, 它肯定还是跟 p 节点在同一个位置
                        //  在复制完 p 节点之后, p 节点的 next 节点还是指向它原来的节点, 就不需要进行复制了, 自己就被带过去了
                        //  这也就导致了一个问题就是复制后的链表的顺序并不一定是原来的倒序
                        for (Node<K,V> p = f.next; p != null; p = p.next) {
                            int b = p.hash & n;
                            if (b != runBit) {
                                runBit = b;
                                lastRun = p;
                            }
                        }
                        if (runBit == 0) {
                            ln = lastRun;
                            hn = null;
                        }
                        else {
                            hn = lastRun;
                            ln = null;
                        }
                        //  构造两个链表, 顺序大部分和原来是反的
                        //  分别放到原来的位置和新增加的长度的相同位置(i/n+i)
                        for (Node<K,V> p = f; p != lastRun; p = p.next) {
                            int ph = p.hash; K pk = p.key; V pv = p.val;
                            if ((ph & n) == 0)
                                //  假设 runBit 的值为0, 则第一次进入这个设置的时候,  
                                //  相当于把旧的序列的最后一次发生 hash 变化的节点
                                //  该节点后面可能还有 hash 计算后同为0的节点,  
                                //  设置到旧的 table 的第一个 hash 计算后为0的节点下一个节点
                                //   并且把自己返回, 然后在下次进来的时候把它自己设置为后面节点的下一个节点
                                ln = new Node<K,V>(ph, pk, pv, ln);
                            else
                                //  假设 runBit 的值不为0, 则第一次进入这个设置的时候相当于, 把旧的序列的最后一次发生 hash 变化的节点
                                //  该节点后面可能还有 hash 计算后同不为 0 的节点, 设置到旧的 table 的第一个 hash 计算后不为 0 的节点下一个节点
                                //  并且把自己返回，然后在下次进来的时候把它自己设置为后面节点的下一个节点
                                hn = new Node<K,V>(ph, pk, pv, hn);
                        }
                        //  在nextTable i 位置处插上链表
                        setTabAt(nextTab, i, ln);
                        //  在nextTable i + n 位置处插上链表
                        setTabAt(nextTab, i + n, hn);
                        //  在table i 位置处插上ForwardingNode 表示该节点已经处理过了
                        setTabAt(tab, i, fwd);
                        //  advance = true 可以执行--i动作, 遍历节点
                        advance = true;
                    }
                    //  如果是TreeBin, 则按照红黑树进行处理, 处理逻辑与上面一致
                    else if (f instanceof TreeBin) {
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        TreeNode<K,V> lo = null, loTail = null;
                        TreeNode<K,V> hi = null, hiTail = null;
                        int lc = 0, hc = 0;
                        for (Node<K,V> e = t.first; e != null; e = e.next) {
                            int h = e.hash;
                            TreeNode<K,V> p = new TreeNode<K,V>
                                (h, e.key, e.val, null, null);
                            if ((h & n) == 0) {
                                if ((p.prev = loTail) == null)
                                    lo = p;
                                else
                                    loTail.next = p;
                                loTail = p;
                                ++lc;
                            }
                            else {
                                if ((p.prev = hiTail) == null)
                                    hi = p;
                                else
                                    hiTail.next = p;
                                hiTail = p;
                                ++hc;
                            }
                        }
                        //  扩容后树节点个数若<=6, 将树转链表
                        ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                            (hc != 0) ? new TreeBin<K,V>(lo) : t;
                        hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                            (lc != 0) ? new TreeBin<K,V>(hi) : t;
                        setTabAt(nextTab, i, ln);
                        setTabAt(nextTab, i + n, hn);
                        setTabAt(tab, i, fwd);
                        advance = true;
                    }
                }
            }
        }
    }
}

```
### treeifyBin  
````
private final void treeifyBin(Node<K,V>[] tab, int index) {
    Node<K,V> b; int n;
    if (tab != null) {
        //  如果整个 table 的数量小于 64, 就扩容至原来的一倍, 不转红黑树了
        //  因为这个阈值扩容可以减少 hash 冲突, 不必要去转红黑树
        if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
            tryPresize(n << 1);
        else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
            synchronized (b) {
                if (tabAt(tab, index) == b) {
                    TreeNode<K,V> hd = null, tl = null;
                    for (Node<K,V> e = b; e != null; e = e.next) {
                        TreeNode<K,V> p =
                            new TreeNode<K,V>(e.hash, e.key, e.val,
                                              null, null);
                        if ((p.prev = tl) == null)
                            hd = p;
                        else
                            tl.next = p;
                        tl = p;
                    }
                    //  通过TreeBin对象对TreeNode转换成红黑树
                    setTabAt(tab, index, new TreeBin<K,V>(hd));
                }
            }
        }
    }
}

````

### tryPresize  
```
private final void tryPresize(int size) {
    //  给定的容量若 >=MAXIMUM_CAPACITY 的一半, 直接扩容到允许的最大值, 否则调用 tableSizeFor 函数扩容 
    int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY :
        //  tableSizeFor(count) 的作用是找到大于等于 count 的最小的 2 的幂次方;  
        tableSizeFor(size + (size >>> 1) + 1);
    int sc;
    while ((sc = sizeCtl) >= 0) {  //  只有大于等于 0 才表示该线程可以扩容, 具体看 sizeCtl 的含义
        Node<K,V>[] tab = table; int n;
        //  如果数组还没有初始化, 先初始化 tab  
        //  使得 sizeCtrl 为数组长度的 3/4
        //  如果第一次 put 的时候不是 put 单个元素, 而是调用 putAll 方法直接 put 一个 map 的话, 在 putAll 方法中没有调用 initTable 方法去初始化 table;  
        //  而是直接调用了 tryPresize 方法, 所以这里需要做一个是不是需要初始化 table 的判断;
        if (tab == null || (n = tab.length) == 0) {  //  没有被初始化
            n = (sc > c) ? sc : c;
            //  期间没有其他线程对表操作, 则 CAS 将 SIZECTL 状态置为 -1, 表示正在进行初始化  
            if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    if (table == tab) {
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = nt;
                        sc = n - (n >>> 2);  //  无符号右移2位, 此即 0.75*n
                    }
                } finally {
                    sizeCtl = sc;
                }
            }
        }
        //  一直扩容到的 c 小于等于 sizeCtl, 或者数组长度大于最大长度的时候;    
        else if (c <= sc || n >= MAXIMUM_CAPACITY)
            break;
        else if (tab == table) {  //  table 不为空, 且在此期间其他线程未修改 table  
            int rs = resizeStamp(n);
            //  如果正在扩容Table的话, 则帮助扩容, 否则的话，开始新的扩容
            //  在 transfer 操作, 将第一个参数的 table 中的元素, 移动到第二个元素的 table 中去; 
            //  虽然此时第二个参数设置的是 null, 但是在 transfer 方法中, 当第二个参数为 null 的时候, 会创建一个两倍大小的 table;  
            if (sc < 0) {
                Node<K,V>[] nt;
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                    transferIndex <= 0)
                    break;
                //  transfer 的线程数加一, 该线程将进行 transfer 的帮忙
                //  在 transfer 的时候, sc 表示在 transfer 工作的线程数
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    transfer(tab, nt);
            }
            //  没有在初始化或扩容, 则开始扩容
            else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                         (rs << RESIZE_STAMP_SHIFT) + 2))
                transfer(tab, null);
        }
    }
}
```

```
//  当扩容到n时，调用该函数返回一个标志位;  
//  numberOfLeadingZeros 返回 n 对应 32 位二进制数左侧 0 的个数, 如 9(1001) 返回 28  
//  RESIZE_STAMP_BITS=16,  因此返回值为: (参数n的左侧0的个数)|(2^15)  
static final int resizeStamp(int n) {
    return Integer.numberOfLeadingZeros(n) | (1 << (RESIZE_STAMP_BITS - 1));
}
```
### addCount
```
private final void addCount(long x, int check) {
    CounterCell[] as; long b, s;
    //  更新baseCount, table的数量, counterCells表示元素个数的变化
    if ((as = counterCells) != null ||
        !U.compareAndSetLong(this, BASECOUNT, b = baseCount, s = b + x)) {
        CounterCell a; long v; int m;
        boolean uncontended = true;
        //  如果多个线程都在执行, 则CAS失败, 执行fullAddCount, 全部加入count
        if (as == null || (m = as.length - 1) < 0 ||
            (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
            !(uncontended =
              U.compareAndSetLong(a, CELLVALUE, v = a.value, v + x))) {
            fullAddCount(x, uncontended);
            return;
        }
        if (check <= 1)
            return;
        s = sumCount();
    }
    //  check>=0表示需要进行扩容操作
    if (check >= 0) {
        Node<K,V>[] tab, nt; int n, sc;
        while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
               (n = tab.length) < MAXIMUM_CAPACITY) {
            int rs = resizeStamp(n);
            if (sc < 0) {
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                    transferIndex <= 0)
                    break;
                if (U.compareAndSetInt(this, SIZECTL, sc, sc + 1))
                    transfer(tab, nt);
            }
            else if (U.compareAndSetInt(this, SIZECTL, sc,
                                         (rs << RESIZE_STAMP_SHIFT) + 2))
                transfer(tab, null);
            s = sumCount();
        }
    }
}

```
### get
```
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    int h = spread(key.hashCode());  //  计算两次hash  
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {  //  读取首节点的Node元素
        //  搜索到的节点 key 与传入的 key 相同且不为 null, 直接返回这个节点;  
        if ((eh = e.hash) == h) {  //  如果该节点就是首节点就返回
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
        //  hash值为负值表示正在扩容, 这个时候查的是 ForwardingNode 的 find 方法来定位到 nextTable来查找, 查找到就返回
        else if (eh < 0)
            return (p = e.find(h, key)) != null ? p.val : null;
        while ((e = e.next) != null) {  //  既不是首节点也不是ForwardingNode, 那就往下遍历
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```
1.. 根据 key 调用 spread 计算 hash 值, 并根据计算出来的 hash 值计算出该 key 在 table 出现的位置i;  
2.. 检查 table 是否为空, 如果为空, 返回 null, 否则进行3;  
3.. 检查 table[i] 处桶位不为空, 如果为空, 则返回 null, 否则进行4;  
4.. 先检查 table[i] 的头结点的 key 是否满足条件, 是则返回头结点的 value, 否则分别根据树, 链表查询;  

### replaceNode  
```
final V replaceNode(Object key, V value, Object cv) {
    int hash = spread(key.hashCode());
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0 ||
            (f = tabAt(tab, i = (n - 1) & hash)) == null)
            break;
        //  如果检测到其它线程正在扩容, 则先帮助扩容, 然后再来寻找, 可见扩容的优先级之高; 
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            boolean validated = false;
            synchronized (f) {
                //  重新检查, 避免由于多线程的原因 table[i] 已经被修改
                if (tabAt(tab, i) == f) {
                    //  链表节点
                    if (fh >= 0) {
                        validated = true;
                        for (Node<K,V> e = f, pred = null;;) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                V ev = e.val;
                                if (cv == null || cv == ev ||
                                    (ev != null && cv.equals(ev))) {
                                    oldVal = ev;
                                    if (value != null)
                                        e.val = value;
                                    else if (pred != null)
                                        pred.next = e.next;
                                    else
                                        setTabAt(tab, i, e.next);
                                }
                                break;
                            }
                            pred = e;
                            if ((e = e.next) == null)
                                break;
                        }
                    }
                    else if (f instanceof TreeBin) {
                        validated = true;
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        TreeNode<K,V> r, p;
                        if ((r = t.root) != null &&
                            (p = r.findTreeNode(hash, key, null)) != null) {
                            V pv = p.val;
                            if (cv == null || cv == pv ||
                                (pv != null && cv.equals(pv))) {
                                oldVal = pv;
                                if (value != null)
                                    p.val = value;
                                else if (t.removeTreeNode(p))
                                    setTabAt(tab, i, untreeify(t.first));
                            }
                        }
                    }
                }
            }
            if (validated) {
                if (oldVal != null) {
                    if (value == null)
                        addCount(-1L, -1);
                    return oldVal;
                }
                break;
            }
        }
    }
    return null;
}
```
1.. 先根据 key 的 hash 值计算书其在 table 的位置 i;  
2.. 检查 table[i] 是否为空, 如果为空, 则返回 null, 否则进行 3;  
3.. 在 table[i] 存储的链表(或树)中开始遍历比对寻找, 如果找到节点符合 key 的, 则判断 value 是否为 null 来决定是否是更新 oldValue 还是删除该节点;  
### 常见问题  
#### 为什么线程同步用的是 synchronized 而不是锁  
为什么是 synchronized, 而不是可重入锁   
1.. 减少内存开销, 假设使用可重入锁来获得同步支持, 那么每个节点都需要通过继承 AQS 来获得同步支持, 但并不是每个节点都需要获得同步支持的,   
只有链表的头节点(红黑树的根节点)需要同步, 这无疑带来了巨大内存浪费;   
2.. 获得 JVM 的支持 可重入锁毕竟是 API 这个级别的, 后续的性能优化空间很小, synchronized 则是 JVM 直接支持的,   
JVM 能够在运行时作出相应的优化措施: 锁粗化, 锁消除, 锁自旋等等;  

#### 这三个方法又是分别在什么情况下进行调用的呢?  
tryPresize 是在 treeifyBin 和 putAll 方法中调用, treeifyBin 主要是在put添加元素完之后;  
判断该数组节点相关元素是不是已经超过 8 个的时候, 如果超过则会调用这个方法来扩容数组或者把链表转为树;  
helpTransfer 是在当一个线程要对 table 中元素进行操作的时候, 如果检测到节点的 HASH 值为 MOVED 的时候, 就会调用 helpTransfer 方法;  
在 helpTransfer 中再调用 transfer 方法来帮助完成数组的扩容;  
addCount 是在当对数组进行操作, 使得数组中存储的元素个数发生了变化的时候会调用的方法;  

####  那么, 多个线程又是如何同步处理的呢?  
在 ConcurrentHashMap 中, 同步处理主要是通过 Synchronized 和 unsafe 两种方式来完成的;  
在取得 sizeCtl, 某个位置的 Node 的时候, 使用的都是 unsafe 的方法, 来达到并发安全的目的;  
当需要在某个位置设置节点的时候, 则会通过 Synchronized 的同步机制来锁定该位置的节点;  
在数组扩容的时候, 则通过处理的步长和 fwd 节点来达到并发安全的目的, 通过设置 hash 值为 MOVED;  
当把某个位置的节点复制到扩张后的 table 的时候, 也通过 Synchronized 的同步机制来保证现程安全;  

### 参考  
https://www.cnblogs.com/zerotomax/p/8687425.html  
https://juejin.im/entry/592a39820ce4630057778b80  
https://my.oschina.net/hosee/blog/639352  
https://my.oschina.net/hosee/blog/675884  
https://www.cnblogs.com/study-everyday/p/6430462.html  
https://blog.csdn.net/u010412719/article/details/52145145  
https://www.jianshu.com/p/c0642afe03e0  
https://www.jianshu.com/p/23b84ba9a498  
https://juejin.im/post/5c8276216fb9a049d51a4cd6  

http://www.importnew.com/26049.html  
http://blog.csdn.net/u010723709/article/details/48007881  
http://blog.csdn.net/fjse51/article/details/55260493  
https://www.cnblogs.com/everSeeker/p/5601861.html   
https://javadoop.com/post/hashmap  
https://www.jianshu.com/p/d10256f0ebea  
https://www.cnblogs.com/wenbochang/p/8484779.html  
https://blog.csdn.net/makeliwei1/article/details/81287855  
https://www.jianshu.com/p/c0642afe03e0  
http://www.jianshu.com/p/e694f1e868ec  
https://my.oschina.net/liuxiaomian/blog/880088  
http://cmsblogs.com/?p=2283  
https://bentang.me/tech/2016/12/01/jdk8-concurrenthashmap-1/  
https://blog.csdn.net/u011392897/article/details/60479937  
https://blog.csdn.net/u010412719/article/details/52145145  




