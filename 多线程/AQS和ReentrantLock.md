# AQS和ReentrantLock
AQS使用一个int成员变量来表示同步状态，通过内置的FIFO队列来完成获取资源线程的排队工作。

state=0的时候表示锁空闲，为1表示被占用

AQS支持两种同步方式：

　　1.独占式

　　2.共享式

> 同步器的使用

　　1.使用者继承AbstractQueuedSynchronizer并重写指定的方法。（这些重写方法很简单，无非是对于共享资源state的获取和释放）

　　2.将AQS组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写的方法。

可重写的方法包括：
protected boolean tryAcquire(int arg) : 独占式获取同步状态，试着获取，成功返回true，反之为false

　　　　protected boolean tryRelease(int arg) ：独占式释放同步状态，等待中的其他线程此时将有机会获取到同步状态；

　　　　protected int tryAcquireShared(int arg) ：共享式获取同步状态，返回值大于等于0，代表获取成功；反之获取失败；

　　　　protected boolean tryReleaseShared(int arg) ：共享式释放同步状态，成功为true，失败为false

　　　　protected boolean isHeldExclusively() ： 是否在独占模式下被线程占用。

> AQS底层原理

AQS内部有一个FIFO的队列（称为CLH队列）。该队列由一个一个的Node结点组成，每个Node结点维护一个prev引用和next引用，分别指向自己的前驱和后继结点。AQS维护两个指针，分别指向队列头部head和尾部tail。

当线程获取资源失败（比如tryAcquire时试图设置state状态失败），会被构造成一个结点加入CLH队列中，同时当前线程会被阻塞在队列中（通过LockSupport.park实现，其实是等待态）。当持有同步状态的线程释放同步状态时，会唤醒后继结点，然后此结点线程继续加入到对同步状态的争夺中。

> AQS实现公平锁和非公平锁不同的地方

公平锁和非公平锁只有两处不同：

1. 非公平锁在调用 lock 后，首先就会调用 CAS 进行一次抢锁，如果这个时候恰巧锁没有被占用，那么直接就获取到锁返回了。
2. 非公平锁在 CAS 失败后，和公平锁一样都会进入到 AQS的acquire 方法，AQS的acquire方法会调用tryAcquire，tryAcquire被ReentrantLock重写（==注意：公平锁和非公平锁的tryAcquire实现不同==），在 tryAcquire 方法中，如果发现锁这个时候被释放了（state == 0），非公平锁会直接 CAS 抢锁，但是公平锁会判断等待队列是否有线程处于等待状态，如果有则不去抢锁，乖乖排到后面。
3. 在进入队列后，阻塞前，两种锁都会判断自己是否是首节点，如果是的化，还会再抢一次锁，又失败了，才会挂起线程

公平锁和非公平锁就这两点区别，前两次 CAS 都不成功，那么后面非公平锁和公平锁是一样的，都要进入到阻塞队列等待唤醒。

==ReentrantLock内有两个内部类，NonFairSync和FairSync。两个都重写了lock方法和tryAcquire方法。非公平的在lock中会抢一次锁再调用acquire。然后在NonFairTryRequire中进入队列后，阻塞前，如果释放了锁还会抢一次，而公平锁的tryRequire中，会同时判断是否释放以及链表中有无节点。==

==ReentrantLock内非公平锁的实现==
```java
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
      // 进入队列之后，不管是否有前驱，再抢一次。
      // 进入队列的代码在AQS中
      // tryAcquire返回false的话，在AQS中这个线程就会被进入阻塞态了
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
static final class NonfairSync extends Sync {
    private static final long serialVersionUID = 7316153563782823691L;

    /**
      * Performs lock.  Try immediate barge, backing up to normal
      * acquire on failure.
      */
    final void lock() {
      // 调用lock的时候就会先抢一次！
        if (compareAndSetState(0, 1))
            setExclusiveOwnerThread(Thread.currentThread());
        else
            acquire(1);
    }

    protected final boolean tryAcquire(int acquires) {
        return nonfairTryAcquire(acquires);
    }
}
```

==公平锁的实现==
```java
static final class FairSync extends Sync {
    private static final long serialVersionUID = -3000897897090466540L;

    final void lock() {
      // 调用lock直接进入acquire！
        acquire(1);
    }

    /**
      * Fair version of tryAcquire.  Don't grant access unless
      * recursive call or no waiters or is first.
      */
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
          // 这里的第一个判断非公平锁没有，就是判断队列中有无前驱
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
}
```

> 讲一下基于AQS，reentrantlock的几个特征

- 可重入
  - 如果当前线程等于owner线程，则发生锁重入，把state++。释放的时候status--。如果status=0才会释放锁。
- 公平锁
  - 非公平锁：lock中抢一个，tryAcquire中抢一次，然后进入阻塞跟公平锁一样，按顺序被唤醒
  - 公平锁：检查AQS队列中是否有前驱节点，没有才去竞争
- 条件变量
  - 每个条件变量其实对应着一个等待队列，其实现类是ConditionObject
  - 开始Thread-0持有锁，调用await，进入conditionObject的addConditionWaiter流程，把自己加入到ConditionObject的等待队列中。然后释放锁，唤醒CLH队列的下一个节点
  - 调用signal后，将节点从ConditionObject队列删除，加入到CLH尾部

> reentrantlock的抢占的锁是什么

就是一个volatile int state。先判断state是否为0，为0表示没被占用，那么就用cas去抢。如果当前线程就是state抢占线程，那么state+1，重入。

```java
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
              // 这一个方法就是抢锁
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    // 重入
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```