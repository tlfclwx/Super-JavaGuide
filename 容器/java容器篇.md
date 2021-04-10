[toc]

# java容器篇

> 哪些类实现了collection接口

除了map，其他的都实现了。
- List
  - ArrayList
  - LinkedList 双向链表
  - Vector
  - Stack
- Queue
  - PriorityQueue
  - Deque
  - ArrayQue
- Set
  - HashSet 基于HashMap实现
  - LinkedHashSet 基于LinkedHashMap实现
  - SortedSet
  - TreeSet 红黑树，有序

> ArrayList扩容的过程，以及涉及的函数

首先是`add`函数
内部调用`ensureCapacityInternal(size+1);`

然后`ensureCapacityInteral`函数内
检查最小需要容量与默认容量的关系，哪一个大，哪一个是minCapacity。就是为了应对空数组的第一次扩容的。
调用`ensureExplicitCapacity(minCapacity);`

`ensureExplicitCapacity`内
判断最小需要容量是否大于当前数组的长度。如果是的，进行`grow(minCapacity)`扩容。也就是在这个函数里，判断是否需要扩容。

`grow`函数
新的大小为旧的1.5倍（偶数1.5，奇数1.5左右）。然后拿新的大小跟最小需要容量minCapacity比较，如果还是不够。那么就扩容到minCapacity大小。
拿新的大小跟MAX_ARRAY_SIZE比较。如果比他还大，调用`hugeCapacity`
使用`Arrays.copyOf(elementData, newCapacity);`创建一个新的数组

`hugeCapacity`内
如果minCapacity小于MAX_ARRAY_SIZE，就返回这个MAX_ARRAY_SIZE，否则返回MAX_VALUE. 两者差了8.

> ArrayList的实现与继承关系

实现了List, RandomAccess, Clonable和Serializable
继承了AbstractList

这里的RandomAccess知识一个标记，没有任何方法

> ArrayList和Vector的区别

ArrayList是List的主要实现类，是线程不安全的。
Vector是List的古老实现类，是线程安全的

> ArrayList和LinkedList的区别

线程安全： 两个都是线程不安全的

底层数据结构：前者是Object数组，后者是双向链表(1.7以前是循环双向链表)

插入和删除是否受元素位置的影响：前者受影响，需要把插入位之后的后移，后者也受，需要找到插入位置。

是否支持随机访问：前者支持，后者不支持。

内存空间浪费：前者会因为多分配的数组空间而浪费，后者因为每个节点要存指针而浪费

> ArrayList的初始化容器大小是多少

如果是有参构造，并且参数不是0，那么就是10。 如果是0，就是0.
如果是无参构造，创建出来的也是空数组，只有当第一次插入的时候，会扩容成10.

> ArrayList扩容每次扩大多少，有没有别的情况。

一般情况下，每次扩容是1.5倍，但是受奇偶数影响，因为用的是>>右移，所以奇数的时候，是1.5倍左右。
还有情况是超出最大容量MAX_ARRAY_SIZE了，会根据minCapacity最小容量需求判断。最多扩容到MAX_VALUE，是MAX_ARRAY_SIZE+8。

> System.copyOf和System.arraycopy的区别

arraycopy需要目标数组，将原数组拷贝到自定义的数组里或者原数组，而且可以选择拷贝的起点和长度以及放入数组中的位置

copyOf是系统自动在内部心间一个数组，并返回该数组

> hashmap1.8之前与之后的数据结构是什么

1.8之前是数组+链表，用链表解决冲突问题。

1.8之后当链表长度大于阈值（默认是8）就会转换成红黑树（==如果当前数组长度小于64，会先选择数组扩容==），以减少搜索时间

链表是一个Node链表，是一个静态类
```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;
    //....
}
```

> 如何计算hash，如何确定存放的数组位置。抖动函数？

1.8之后的hash计算方法是key.hashcode()的高16位异或低16位。

存放的位置通过(n-1)&hash来计算。其中n表示数组的长度

> hashmap的默认数组长度是多少

16

> hashmap链表长度大于多少会转换成红黑树，小于多少又会转换回来？

8和6

> hashmap中加载因子是什么

默认0.75，指的是数据条目是桶数量的0.75倍时，会扩容！

> hashmap的put方法1.8和1.7的区别

1.8是插入到链表尾部，或者插入红黑树，而1.7是插入链表头部。

1.7的头插法会有循环引用问题。

> hashmap一次扩容多少

两倍

> hashmap扩容后，数据如何迁移？

JDK7中，HashMap的内部数据保存的都是链表。因此逻辑相对简单：在准备好新的数组后，map会遍历数组的每个“桶”，然后遍历桶中的每个Entity，重新计算其hash值（也有可能不计算），找到新数组中的对应位置，以头插法插入新的链表。

JDK8中，数组长度变为原来的2倍，表现在二进制上就是多了一个高位参与数组下标确定。此时，一个元素通过hash转换坐标的方法计算后，恰好出现一个现象：最高位是0则坐标不变，最高位是1则坐标变为“10000+原坐标”，即“原长度+原坐标”。==比如原来的n-1的二进制是1111，现在翻倍了就是11111.因此只需要判断多出来的最高位是0是1即可==

> LinkedList实现了哪些接口

Deque（队列）和List

> LinkedList是线程安全的吗？

不是

> 如何让LinkedList线程安全

Collections.synchronizedList(new LinkedList());

> LinkedList获取头节点的四个方法，讲一下有什么不同

getFirst(),element(),peak(),peakFirst()

element就是调用了getFirst，peak和peakFirst实现方法一样。

element如果没有头节点则抛出一场。

peak没有头节点则返回null

> Collection接口下的集合有哪些，说一下其用处

List
- ArrayList 底层就是动态数组
- Vector 数组
- LinkedList，是双向链表，1.6以前是循环双向链表

Set
- HashSet，基于HashMap实现
- LinkedHashSet
- TreeSet（有序的）：红黑树

Map
- HashMap1.7 数组+链表。1.8数组+链表+红黑树
- LinkedHashMap：在HashMap基础上加了指针，可以保持插入顺序
- HashTable：数组+链表
- TreeMap：红黑树

> 什么是Set的无序性和不可重复性

1、什么是无序性?无序性不等于随机性 ，无序性是指存储的数据在底层数组中并非按照数组索引的顺
序添加 ，而是根据数据的哈希值决定的。

2、什么是不可重复性?不可重复性是指添加的元素按照 equals()判断时 ，返回 false，需要同时重写
equals()方法和 HashCode()方法。

> HashMap的长度为什么是2的幂次方

数组下标的计算方法是“ (n - 1) & hash ”。(n 代表数组⻓ 度)。这也就解释了 HashMap 的⻓度为什么是 2 的幂次方。

这个是为了对哈希值进行取模。但是n-1&能够取模的前提是n是2的幂次方



> Arrays.sort()使用的是什么算法

如果长度<47使用插入排序，<286使用快排，大于286时，先判断数据是否有序，如果是降序组太多的就会转会快排，否则用归并排序。

> ArrayList和LinkedList插入一条数据时谁的效率高？插入上万条数据时谁的效率高？

插一条数据分在什么位置插，如果末尾则差不多，如果中间则LinkedList；插入很多条的话考虑到ArrayList会扩容，LinkedList高

>String不可变的原理是什么？为什么要设计成不可变的？

char数组用final修饰的；如果可变，字符串常量池引用会混乱；String缓存了自己的hash，如果可变，但是hash不会变，在HashMap、HashSet中会出现问题；String经常用作参数，如果可变则不安全。



> 讲一下HashSet的底层实现。是如何保证不重复的

HashSet内部其实就是new了一个HashMap。所以数据结构跟HashMap一摸一样。

HashSet的add方法如下
```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```
可以看到他把add对象当作key给了hashmap。那么hashmap先通过计算hashcode，在计算equals，如果都相同，那么就认为是同一个key，对旧值进行覆盖。因此保证了不重复。




> 如何遍历hashmap

map.Entry 获得全部键值对。这里是Map接口
map.KeySet 获取键，这里是map对象
map.values

> hashmap的内存泄漏说一下

如果put的对象的hashcode会改变，那么就会发生内存泄漏问题。无法再找到之前插入的那个hashcode对象了。


> hashmap链表转红黑树为什么定为8

红黑树中的TreeNode是链表中的Node所占空间的2倍，虽然红黑树的查找效率为o(logN)，要优于链表的o(N)，但是当链表长度比较小的时候，即使全部遍历，时间复杂度也不会太高。

Java的源码贡献者在进行大量实验发现，hash碰撞发生8次的概率已经降低到了0.00000006，几乎为不可能事件，如果真的碰撞发生了8次，那么这个时候说明由于元素本身和hash函数的原因，此时的链表性能已经已经很差了，操作的hash碰撞的可能性非常大了，后序可能还会继续发生hash碰撞。所以，在这种极端的情况下才会把链表转换为红黑树，链表转换为红黑树也是需要消耗性能的，为了挽回性能，权衡之下，才使用红黑树，提高性能的，大部分情况下hashMap还是使用链表

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