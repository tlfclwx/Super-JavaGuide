> ThreadLocal的原理说一下

ThreadLocal可以理解为TreadLocalMap的封装，ThreadLocal中存了ThreadLocalMap，然后根据currentThread获取对应的map，每一个map中都存了键值对，key就是ThreadLocal，不同的线程有不同的ThreadLocal。

看一下ThreadLocal中的set方法，把对象自己作为key存进map

ThreadLocalMap是一个Entry数组。Entry内是一个弱引用key和强引用value
```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}

// 静态内部类
static class ThreadLocalMap {
        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }

        ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
            table = new Entry[INITIAL_CAPACITY];
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            setThreshold(INITIAL_CAPACITY);
        }
}
```


ThreadLocal为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。

> ThreadLocal的key为什么用弱引用

什么是弱引用？
==Key使用强引用：也就是上述说的情况，引用ThreadLocal的对象被回收了，ThreadLocal的引用ThreadLocalMap的Key为强引用并没有被回收，如果不手动回收的话，ThreadLocal将不会回收那么将导致内存泄漏。==
==Key使用弱引用：引用的ThreadLocal的对象被回收了，ThreadLocal的引用ThreadLocalMap的Key为弱引用，如果内存回收，那么将ThreadLocalMap的Key将会被回收，ThreadLocal也将被回收。value在ThreadLocalMap调用get、set、remove的时候就会被清除。==

> 那么在threadLocalMap调用get方法时，发生了gc，弱引用对象threadLocal是否会因为被回收掉，而get不到。

==不会，因为堆中有一个强引用在，只有在不在用threadLocal了，把这个强引用设为null后，ThreadLocalMap中的弱引用才会被回收。（看下一题的图）==

> ThreadLocal的内存泄漏问题

==首先来说，如果把ThreadLocal置为null，那么意味着Heap中的ThreadLocal实例不在有强引用指向，只有弱引用存在，因此GC是可以回收这部分空间的，也就是key是可以回收的。但是value却存在一条从Current Thread过来的强引用链。因此只有当Current Thread销毁时，value才能得到释放。==

==因此，只要这个线程对象被gc回收，就不会出现内存泄露，但在threadLocal设为null和线程结束这段时间内不会被回收的，就发生了我们认为的内存泄露。最要命的是线程对象不被回收的情况，比如使用线程池的时候，线程结束是不会销毁的，再次使用的，就可能出现内存泄露。==

==着重理解这张图==
![](https://gitee.com/super-jimwang/img/raw/master/img/20210314160741.png)

> ThreadLocalMap和HashMap的不同

hash计算方式不同，另外ThreadLocalMap解决冲突的方法是开放地址法。