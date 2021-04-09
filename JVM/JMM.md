# JMM

> JMM是什么


JMM是一种规范，目的是解决由于多线程通过共享内存进行通信时，存在的本地内存数据不一致、编译器会对代码指令重排序、处理器会对代码乱序执行等带来的问题。

作用于本地内存和主存之间
![](https://gitee.com/super-jimwang/img/raw/master/img/20210314100503.png)

> JMM的特性

- 原子性
  - 提供了两个高级的字节码指令monitorenter和monitorexit。也就是synchronized。代码块内是原子性的
- 可见性
  - 通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值
  - volatile（读写屏障），synchronized（JVM规定，加锁前同步，解锁后同步）和final两个关键字也可以实现可见性
- 有序性
  - synchronized和volatile来保证多线程之间操作的有序性
  - volatile关键字会禁止指令重排。synchronized关键字保证同一时刻只允许一条线程操作。