> concurrenthashmap如何计数

在1.7中 源码如下：
![](https://gitee.com/super-jimwang/img/raw/master/img/20210314105439.png)
可以发现设定了一个RETRIES_BEFORE_LOCK.会先循环几次，每次统计所有segment的modCount为sum。modCount每次在做修改的时候会+1.因此如果两次循环的sum一样，说明这段时间内没有变化，那么size计算准确返回。否则超过了RETRIES_BEFORE_LOCK次数，会把segment全锁了，然后统计size返回。

1.8中
有一个long的baseCount和一个CounterCell[] counterCells数组。求size的时候调用`sumCount`
```java
final long sumCount() {
    CounterCell[] as = counterCells; CounterCell a;
    long sum = baseCount;
    if (as != null) {
        for (int i = 0; i < as.length; ++i) {
            if ((a = as[i]) != null)
                sum += a.value;
        }
    }
    return sum;
}
```
可以看出就是累加baseCount和counterCells。
counterCells 数组未初始化
a. CAS 一次 baseCount
b. 如果 CAS 失败，则调用 fullAddCount 方法

counterCells 数组已初始化
a. CAS 一次当前线程探针哈希到的数组元素
b. 如果 CAS 失败，则调用 fullAddCount 方法

每次concurrentHashMap有修改的时候都会调用`addCount`方法，该方法内如果counterCells未初始化，就通过CAS操作尝试在baseCount中操作，如果counterCells已经初始化就通过hash到一个CounterCell数组的一个位置，进行+操作。

`fullAddCount`中
线程探针哈希值的初始化。
counterCells 数组的初始化和扩容。
counterCells 元素的初始化。
将 size 的变化，写入 counterCells 中的某一个元素。

会初始化counterCells。

==核心思路==
如果没有冲突发生，只将 size 的变化写入 baseCount。
一旦发生冲突，就用一个数组（counterCells）来存储后续所有 size 的变化。这样，线程只要对任意一个数组元素写入 size 变化成功即可，数组长度越长，线程发生冲突的可能性就越小。

> ConcurrentHashMap在1.8中 触发扩容的情况说一下

- 添加新元素后，达到了负载因子，扩容
- putall方法，数据放不下，扩容
- 某个Node内的链表比8长，但是总的Node数小于64.扩容

> ConcurrentHashMap如果在扩容的时候，读写操作怎么办

写操作
```java
else if ((fh = f.hash) == MOVED)
    tab = helpTransfer(tab, f);
```
这里MOVED=-1，因此当hashcode==-1的时候，表明在扩容，写线程会帮忙扩容。


> ConcurrentHashMap get的时候会加锁吗？为什么

不会。

因为Node.val是被volatile修饰了的，保证可见性。

> ConcurrentHashMap 和 HashTable的区别

- 底层不一样，ConcurrentHashMap.17以前是segment分段数组+HashEntry+链表，1.8之后是Node数组+链表。而HashTable是数组+链表
- ==线程安全方面：ConcurrentHashMap1.7以前是锁一个segment，1.8开始是锁Node的头节点,通过synchronized锁住一个node。所以一部分锁了，其他部分还是可以访问的。而HashTable是直接锁整个Table，其他线程不能访问了==
- ==ConcurrentHashMap通过volatile保证读操作的可见性，读的时候不需要上锁，而HashTable中get方法和put方法，都用synchronized修饰了。==

```java
//1.8中的concurrenthashmap
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,
                            new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            // 锁住了node
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                    (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                            value, null);
                                break;
                            }
                        }
                    }
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
    addCount(1L, binCount);
    return null;
}
```

> concurrenthashmap 1.8中Node合适创建？扩容机制

扩容类似hashmap，初始大小16，扩容因子0.75.

是惰性初始化，也就是说分配到的Node可能还没有被初始化，第一次put的时候才去初始化。

> ConcurrentHashMap 1.7中 segment扩容机制

不能扩容

> ConcurrentHashMap是如何具体实现线程安全的

1.7:
首先将数据分为一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据
时，其他段的数据也能被其他线程访问。Segment就是一把ReentrantLock
ConcurrentHashMap 是由 Segment 数组结构和 HashEntry 数组结构组成。
Segment 实现了 ReentrantLock ,所以 Segment 是一种可重入锁，扮演锁的⻆色。 HashEntry 用于 存储键值对数据。

1.8:
ConcurrentHashMap 取消了 Segment 分段锁，采用 CAS 和 synchronized 来保证并发安全。数据 结构跟 HashMap1.8 的结构类似，数组+链表/红黑二叉树。Java 8 在链表⻓度超过一定阈值(8)时将 链表(寻址时间复杂度为 O(N))转换为红黑树(寻址时间复杂度为 O(log(N)))
synchronized 只锁定当前链表或红黑二叉树的首节点，这样只要 hash 不冲突，就不会产生并发，效 率又提升 N 倍。

> ConcurrentHashMap 是怎么样一个存储结构

![](https://gitee.com/super-jimwang/img/raw/master/img/20210228154856.png)
在1.7中，ConcurrentHashMap是由多个Segment组合，而每一个Segment对应的hashEntry数组，类似于一个Segment内存了一个HashMap。每一个数组内存的是链表，解决冲突用。

而在1.8中，ConcurrentHashMap使用Node节点+链表和红黑树实现。这就类似1.8中的HashMap
![](https://gitee.com/super-jimwang/img/raw/master/img/20210228154950.png)

> ConcurrentHashMap1.7中，默认的Segment个数是多少，负载因子是多少，初始化的数组长度是多少？

默认的Segment是16，也就是对应着16个线程的并发，负载因子是0.75，每个segment的初始化数组长度是2.

> ConcurrentHashMap1.7的put流程说一下

根据hash值计算出key的位置，获取指定位置的segment.以16个segment为例，高四位判断属于哪一个segment

如果指定位置u的segment为空，就需要初始化这个segment

- 检查位置u的segment是否为null
- 为null继续初始化，使用Segment[0]的容量和负载因子创建一个HashEntry数组
- 再次经检查位置u的Segment是否为null
- 使用创建的HashEntry初始化Segment，创建Segment `s`
- 自旋判断位置uSegment是否为null，使用CAS在这个位置u的Segment赋值为Segment `s `

完成初始化，在Segment中插入值

获取ReentrantLock独占锁

计算put要放置的hashEntry

如果没有这个hashEntry：
- 如果当前容量大于扩容阈值，小于最大容量，进行扩容。每次扩大2倍，跟hashmap一样
- ==头插法==插入

如果hashEntry存在
- 查看是否有一样的key，进行替换
- 没有一直的就头插法插入

> ConcurrentHashMap1.7中segment的位置是怎么算的

以16为例，那么mask就是1111.

使用mask&对象.hash的头四位。
```java
// hash 值无符号右移 28位(初始化时获得)，然后与 segmentMask=15 做与运算
// 其实也就是把高4位与segmentMask(1111)做与运算
int j = (hash >>> segmentShift) & segmentMask;
```

> ConcurrentHashMap1.8如何初始化

根据sizeCtl来判断，如果<0，说明另外线程成功执行CAS，正在进行初始化，那么就让出当前CPU。

> ConcurrentHashMap1.8实现put的流程

计算hashcode

判断是否需要初始化Node（==注意因为是惰性初始化，第一次put时候才会去初始化所有的Node，即使已经扩容==）
- 如果是null，就要通过CAS和自旋锁进行初始化

如果Node为空，则尝试使用CAS写入数据，失败则自旋保证成功

如果当前位置的hashcode==-1则需要扩容

如果都不满足，那么就用synchronized锁住Node头，写入数据

如果数量大于8就转换为红黑树

> ConcurrentHashMap1.8中的get方法说一下

```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    int h = spread(key.hashCode());
    if ((tab = table) != null && (n = tab.length) > 0 && (e = tabAt(tab, (n - 1) & h)) != null) {
        //如果所要找的元素就在数组上，直接返回结果
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
        //遇到 hash < 0 可能能有两种情况：
        //    (1) 该节点是TreeBin节点(红黑树代理节点，其hash值固定为：-2);
        //    (2) ConcurrentHashMap正在扩容当中，并且该hash桶已迁移完毕，该位置被放置了FWD节点;
        //1、如果是TreeBin节点，调用TreeBin类的find方法，具体是以链表方式遍历还是红黑树方式遍历视情况而定(后面细说)
        //2、如果正在扩容中，则跳转到扩容后的新数组上去查找，TreeBin和Node节点都有对应的find方法，具体什么节点类型则调用对应节点类型的find方法(后面细说)
        else if (eh < 0)
            return (p = e.find(h, key)) != null ? p.val : null;
        //如果前面两种情况都不满足，说明该hash桶上面连接的是普通链表结构，使用while循环去遍历该链表节点
        while ((e = e.next) != null) {
            if (e.hash == h && ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```