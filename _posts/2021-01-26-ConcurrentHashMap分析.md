# ConcurrentHashMap

ConcurrentHashMap的主要结构跟HashMap类似，依旧是Node数组加上链表结构。但是ConcurrentHashMap能够实现多线程下的同步操作。

主要字段：

```java
/**
 * 桶数组。这个数组的初始值为null，使用懒惰初始化，即当第一次put操作的时候进行初始化。
 * 桶数组的大小总是2的幂。通过iterator直接访问。
 */
transient volatile Node<K,V>[] table;

/**
 * 用来在扩容操作中存放新数组，默认值为null，只有当扩容是才是non-null值
 */
private transient volatile Node<K,V>[] nextTable;

/**
 * 一个用来控制table 初始化和扩容操作的变量。 当sizeCtl为负数时，表示table正在初始化或者扩容，
 * -1 表示正在初始化，否则-(1 + 正在扩容的线程)。当sizeCtl为0时，表示table == null, 使用默认
 * 的初始化大小进行初始化。如果sizeCtl为整数，且table == null, sizeCtl表示table的初始化大小
 * 当table != null。sizeCtl表示下一次扩容的阈值。
 */
private transient volatile int sizeCtl;
```

首先介绍一些必须的辅助函数

1. charAt函数根据tab数组和给定的i得到对应index=i的Node。

```java
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
    // 使用Unsafe模块的本地方法getReferenceAcquire方法得到对应tab数组的指定offset的值，
    // 且保证这个值是最新的。
	return (Node<K,V>)U.getReferenceAcquire(tab, ((long)i << ASHIFT) + ABASE);
}
```

2. casTabAt函数在tab数组的index=i的位置上进行CAS操作

```java
static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                        Node<K,V> c, Node<K,V> v) {
    // 使用Unsafe模块的compareAndSetReference函数，这个函数检查tab数组指定offset位置的值，
    // 如果这个值等于c，那么将这个值赋值为v，否则啥也不做
    return U.compareAndSetReference(tab, ((long)i << ASHIFT) + ABASE, c, v);
}
```

3. setTabAt函数在tab数组的对应位置上进行set操作

```java
static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
        U.putReferenceRelease(tab, ((long)i << ASHIFT) + ABASE, v);
    }
```



## put函数

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    // spread函数用来计算hash值 (h ^ (h >>> 16)) & HASH_BITS;这里实际就是将hashcode的
    // 高位值和低位值进行异或操作。
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh; K fk; V fv;
        // 如果table == null，还没有进行初始化，那么此时调用initTable进行table初始化
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        // 如果已经初始化，那么使用tabAt函数得到对应的Node节点。如果这个Node节点为null，
        // 说明这个桶为空，那么此时可以使用CAS操作尝试将新创建的Node放在桶中。如果CAS成功则跳出
        // 循环，失败则进入下一次循环继续CAS(自旋)。 
        
        // 空桶不加锁的原因在于ConcurrentHashMap在涉及同步机制的时候，设计在每个桶的头节点上
        // 加锁。如果此时为空，此时没有其他线程能够持有锁，所以使用CAS使得只有一个线程能够插入
        // 头节点。
        
        // 为什么当桶不为空的时候不能使用CAS保证同步呢？因为如果桶不为空且不加锁，那么桶中的所
        // 有元素都能被并发修改。所以在头节点上加锁保证每次只有一个线程能够修改这个桶中的元素
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value)))
                break;                   // no lock when adding to empty bin
        }
        // 如果这个Node头节点的hash值等于MOVED(-1), 说明这个tab正在进行扩容操作，那么当前线程
        // 可以调用helpTransfer来协助扩容。
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        // 如果头节点的hash值匹配且只在对应val==null时插入，且节点的key与参数key相同，说明头
        // 节点匹配。
        else if (onlyIfAbsent // check first node without acquiring lock
                 && fh == hash
                 && ((fk = f.key) == key || (fk != null && key.equals(fk)))
                 && (fv = f.val) != null)
            return fv;
        // 否则
        else {
            V oldVal = null;
            // 在头节点上加锁。Synchronized在jvm上能够进行锁优化。
            synchronized (f) {
                // 再次检查这个桶头节点是否被修改，因为在Synchronized之前，没有同步机制，
                // 所以在f赋值后，f处的值可能被修改或者移除
                if (tabAt(tab, i) == f) {
                    // 如果f节点的hash值大于0，说明节点是正常的链表节点
                    if (fh >= 0) {
                        binCount = 1;
                        // 这里遍历Node链表
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            // 如果e遍历节点匹配
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    // 赋值
                                    e.val = value;
                                break;
                            }
                            // 如果链表中没有节点的key与给定key匹配
                            // 这时要创建新Node节点，然后尾插。
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key, value);
                                break;
                            }
                        }
                    }
                    // 如果此时桶中的元素是红黑树的结构
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                    else if (f instanceof ReservationNode)
                        throw new IllegalStateException("Recursive update");
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    // 这里查看是否需要扩容
    addCount(1L, binCount);
    return null;
}
```

### initTable函数

```java
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    
    while ((tab = table) == null || tab.length == 0) {
        // 如果sizeCtl小于0，说明此时正在进行扩容操作，所以当前线程自旋
        if ((sc = sizeCtl) < 0)
            Thread.yield(); // lost initialization race; just spin
        // sizeCtl为0或者为正数时，尝试使用CAS来更新sizeCtl的值，这里this是地址，SIZECTL为
        // sizeCtl字段的offset，如果当前sizeCtl的值等于sc，那么修改为-1，表示正在扩容，如果不
        // 是，则自旋
        else if (U.compareAndSetInt(this, SIZECTL, sc, -1)) {
            // CAS成功
            try {
                // 如果此时tab==null，说明tab还没有初始化，此时进行初始化操作
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    // 分配Node数组
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2);
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

### transfer方法

1. 首先需要把老数组的值全部拷贝到扩容后的新数组上，先从数组的队尾开始拷贝
2. 拷贝数组的桶时，先把原数组的桶锁住，保证原数组桶上不能操作，成功拷贝到新数组时，把原数组桶的头节点赋值为转移节点
3. 这时如果有新数组正好需要put到这个桶，发现桶的头节点为转移节点，就会一直等待，所以在扩容完成之前，该桶上不会发生操作
4. 从数组的尾部拷贝到头部，每拷贝成功一次，就把原数组中的桶的头节点设置为转移节点
5. 直到所有桶上的数据都拷贝到新数组，直接把新数组赋值给tab，拷贝完成。

```java
// 扩容函数，主要有两部，第一步创建新的空数组，第二步移动拷贝每个元素到新数组中
private final void transfer(ConcurrentHashMap.Node<K,V>[] tab, ConcurrentHashMap.Node<K,V>[] nextTab) {
    // 老数组的长度
    int n = tab.length, stride;
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
        stride = MIN_TRANSFER_STRIDE; // subdivide range
    // 如果nextTab为空，那么nextTab要复制tab并扩容为之前的两倍
    if (nextTab == null) {            // initiating
        try {
            @SuppressWarnings("unchecked")
            // 分配两倍与原大小的新数组
            ConcurrentHashMap.Node<K, V>[] nt = (ConcurrentHashMap.Node<K, V>[]) new ConcurrentHashMap.Node<?, ?>[n << 1];
            nextTab = nt;
        } catch (Throwable ex) {      // try to cope with OOME
            sizeCtl = Integer.MAX_VALUE;
            return;
        }
        nextTable = nextTab;
        transferIndex = n;
    }
    // 新数组的长度
    int nextn = nextTab.length;
	// 代表转移节点，如果原数组上是转移节点，说明该节点正在被扩容
    ConcurrentHashMap.ForwardingNode<K, V> fwd = new ConcurrentHashMap.ForwardingNode<K, V>(nextTab);
    boolean advance = true;
    boolean finishing = false; // to ensure sweep before committing nextTab
    // 无限自旋，i的值会从原数组的最大值开始，慢慢递减到0
    for (int i = 0, bound = 0; ; ) {
        ConcurrentHashMap.Node<K, V> f;
        int fh;
        while (advance) {
            int nextIndex, nextBound;
            // 结束循环的标志
            if (--i >= bound || finishing)
                advance = false;
            // 表示拷贝完成
            else if ((nextIndex = transferIndex) <= 0) {
                i = -1;
                advance = false;
            } 
            // 每次减少i的值
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
            // 如果拷贝结束，直接赋值，每次拷贝完一个桶，就在原节点上放转移节点，所以拷贝完成
            // 的原数组的桶上不会再出现变化。因为原数组节点一旦标记为转移节点，是不会再发生任何
            // 改变
            if (finishing) {
                nextTable = null;
                table = nextTab;
                sizeCtl = (n << 1) - (n >>> 1);
                return;
            }
            if (U.compareAndSetInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                    return;
                finishing = advance = true;
                i = n; // recheck before commit
            }
        } 
        // 如果tab数组该桶的位置为空，那么直接在这里放转移节点，避免其他节点插入值
        else if ((f = tabAt(tab, i)) == null)
            advance = casTabAt(tab, i, null, fwd);
        // 如果正在进行扩容
        else if ((fh = f.hash) == MOVED)
            advance = true; // already processed
        else {
         // 在桶的头节点上加锁
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    ConcurrentHashMap.Node<K, V> ln, hn;
                    if (fh >= 0) {
                        int runBit = fh & n;
                        ConcurrentHashMap.Node<K, V> lastRun = f;
                        for (ConcurrentHashMap.Node<K, V> p = f.next; p != null; p = p.next) {
                            int b = p.hash & n;
                            if (b != runBit) {
                                runBit = b;
                                lastRun = p;
                            }
                        }
                        if (runBit == 0) {
                            ln = lastRun;
                            hn = null;
                        } else {
                            hn = lastRun;
                            ln = null;
                        }
                        for (ConcurrentHashMap.Node<K, V> p = f; p != lastRun; p = p.next) {
                            int ph = p.hash;
                            K pk = p.key;
                            V pv = p.val;
                            if ((ph & n) == 0)
                                ln = new ConcurrentHashMap.Node<K, V>(ph, pk, pv, ln);
                            else
                                hn = new ConcurrentHashMap.Node<K, V>(ph, pk, pv, hn);
                        }
                        setTabAt(nextTab, i, ln);
                        setTabAt(nextTab, i + n, hn);
                        setTabAt(tab, i, fwd);
                        advance = true;
                    } else if (f instanceof ConcurrentHashMap.TreeBin) {
                        ConcurrentHashMap.TreeBin<K, V> t = (ConcurrentHashMap.TreeBin<K, V>) f;
                        ConcurrentHashMap.TreeNode<K, V> lo = null, loTail = null;
                        ConcurrentHashMap.TreeNode<K, V> hi = null, hiTail = null;
                        int lc = 0, hc = 0;
                        for (ConcurrentHashMap.Node<K, V> e = t.first; e != null; e = e.next) {
                            int h = e.hash;
                            ConcurrentHashMap.TreeNode<K, V> p = new ConcurrentHashMap.TreeNode<K, V>
                                    (h, e.key, e.val, null, null);
                            if ((h & n) == 0) {
                                if ((p.prev = loTail) == null)
                                    lo = p;
                                else
                                    loTail.next = p;
                                loTail = p;
                                ++lc;
                            } else {
                                if ((p.prev = hiTail) == null)
                                    hi = p;
                                else
                                    hiTail.next = p;
                                hiTail = p;
                                ++hc;
                            }
                        }
                        ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                (hc != 0) ? new ConcurrentHashMap.TreeBin<K, V>(lo) : t;
                        hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                (lc != 0) ? new ConcurrentHashMap.TreeBin<K, V>(hi) : t;
                        setTabAt(nextTab, i, ln);
                        setTabAt(nextTab, i + n, hn);
                        setTabAt(tab, i, fwd);
                        advance = true;
                    } else if (f instanceof ConcurrentHashMap.ReservationNode)
                        throw new IllegalStateException("Recursive update");
                }
            }
        }
    }
}
```

## get方法

```java
public V get(Object key) {
    ConcurrentHashMap.Node<K, V>[] tab;
    ConcurrentHashMap.Node<K, V> e, p;
    int n, eh;
    K ek;
    // 计算hash
    int h = spread(key.hashCode());
    if ((tab = table) != null && (n = tab.length) > 0 &&
			// 计算对应桶的位置
            (e = tabAt(tab, (n - 1) & h)) != null) {
        // 如果头节点hash值匹配
        if ((eh = e.hash) == h) {
            // 如果头节点的key相等，直接返回头节点的值
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        } else if (eh < 0)
            // 小于0说明时数节点
            return (p = e.find(h, key)) != null ? p.val : null;
        // 否则说明为链表，进行遍历搜索
        while ((e = e.next) != null) {
            if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```
