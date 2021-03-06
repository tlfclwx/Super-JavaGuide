# linux相关
> 软硬链接是什么

硬连接的nodeid跟文件一样，就是一个文件，往里面写东西也会同时出现在原文件中。这两个都是指向同一块硬盘中的区块。

软连接保存了其代表文件的绝对路径。如果把原文件删了，软连接就失效了。而硬连接不会失效。

> 什么是僵尸进程

当子进程比父进程结束的早，而父进程又没有SIGCHLD来回收子进程，那么子进程会一直在进程列表中保留一个位置。

僵尸进程是非常特殊的一种，它已经放弃了几乎所有内存空间，没有任何可执行代码，也不能被调度，仅仅在进程列表中保留一个位 置，记载该进程的退出状态等信息供其他进程收集。除此之外，僵尸进程不再占有任何内存空间。

如果这时父进程结束了， 那么init进程自动会接手这个子进程，为它收尸，它还是能被清除的。但是如果如果父进程是一个循环，不会结束，那么子进程就会一直保持僵尸状态，这就是 为什么系统中有时会有很多的僵尸进程。

> 僵尸进程的危害

占用了进程号，而进程号是有限的

> 父子进程的如果父进程先结束了怎么办，如果子进程先结束了怎么办？ 

如果父进程先结束，那么子进程就成了==孤儿进程==。自动成为进程1（init进程）的子进程，init进程会循环地wait()它的已经退出的子进程。==因此孤儿进程不会有什么危害==

如果子进程比父进程先结束，如果父进程没有显示调用wait函数的话，会直到父进程结束时才会回收子进程的资源。这样的子进程就成了==僵尸进程==。==父进程一直循环，然后不wait子进程，而产生的僵尸进程会占用进程号，产生危害==

> linux的进程id上限

pid_max (/proc/sys/kernel/pid_max)

这个值表示进程ID的上限。为了兼容旧版，默认为32768（即两个字节）。2^15次，最前面那位表示正负。

> 子进程如何通知父进程自己结束了 

当父进程调用wait时会被阻塞住，直到收到子进程的SIGCHLD信号，就会回收相关资源！

子进程通过SIGCHLD信号

> fork之后父子进程的内存关系 

(1)首先我们可以确定父子进程的代码段是相同的，所以代码段是没必要复制的，因此内核将代码段标记为只读，这样父子进程就可以安全的共享此代码段了。fork之后在进程创建代码段时，新子进程的进程级页表项都指向和父进程相同的物理页帧

(2)而对于父进程的数据段，堆段，栈段中的各页，由于父子进程要相互独立，所以我们采用写实复制的技术，来最大化的提高内存以及内核的利用率。刚开始，内核做了一些设置，令这些段的页表项指向父进程相同的物理内存页。调用fork之后，内核会捕获所有父进程或子进程针对这些页面的修改企图(说明此时还没修改)并为将要修改的页面创建拷贝。系统将新的页面拷贝分配给被内核捕获的进程，还会对子进程的相应页表项做适当的调整，现在父子进程就可以分别修改各自的上述段，不再互相影响了
![](https://gitee.com/super-jimwang/img/raw/master/img/20210315172306.png)

> 什么是写时复制技术

fork（）会产生一个和父进程完全相同的子进程，但子进程在此后多会exec系统调用，出于效率考虑，linux中引入了“写时复制“技术，也就是只有进程空间的各段的内容要发生变化时，才会将父进程的内容复制一份给子进程。在fork之后exec之前两个进程用的是相同的物理空间（内存区），子进程的代码段、数据段、堆栈都是指向父进程的物理空间，也就是说，两者的虚拟空间不同，但其对应的物理空间是同一个。当父子进程中有更改相应段的行为发生时，再为子进程相应的段分配物理空间。

> linux常用指令

```
cd 进入指定的目录

cd .. 返回上一级目录

ls 查看当前目录下的所有的目录与文件名

touch filename 表示创建一个文件

mkdir dirname 表示创建一个目录

rm filename 表示删除一个文件

mv 移动文件或者重命名

lshw： 查看硬件信息

lscpu：查看cpu信息

df： 查看磁盘空间

ps: 查看当前文件的进程 进程号

kill：杀死

top：根据消耗的资源,从上之下排序
```

> 什么是mmap

是一种内存映射文件的方式。

即将一个文件或者其它对象映射到进程的地址空间，实现文件磁盘地址和进程虚拟地址空间中一段虚拟地址的一一对映关系。实现这样的映射关系后，进程就可以采用指针的方式读写操作这一段内存，而系统会自动回写脏页面到对应的文件磁盘上，即完成了对文件的操作而不必再调用read,write等系统调用函数。相反，内核空间对这段区域的修改也直接反映用户空间，从而可以实现不同进程间的文件共享。

==如果有多个进程访问同一个文件, 则每一个进程在自己的地址空间都包含有该文件的副本,这不必要地浪费了存储空间。mmap解决了这个问题==
![](https://gitee.com/super-jimwang/img/raw/master/img/20210314095926.png)

==进程A和进程B都将该页映射到自己的地址空间, 当进程A第一次访问该页中的数据时, 它生成一个缺页中断. 内核此时读入这一页到内存并更新页表使之指向它.以后, 当进程B访问同一页面而出现缺页中断时, 该页已经在内存, 内核只需要将进程B的页表登记项指向次页即可==
![](https://gitee.com/super-jimwang/img/raw/master/img/20210314095957.png)

==一定时间后系统会自动回写脏页面到对应磁盘地址，也即完成了写入到文件的过程==

> namespace是什么

namespace 是 Linux 内核用来隔离内核资源的方式。通过 namespace 可以让一些进程只能看到与自己相关的一部分资源，而另外一些进程也只能看到与它们自己相关的资源，这两拨进程根本就感觉不到对方的存在。具体的实现方式是把一个或多个进程的相关资源指定在同一个 namespace 中。

> 守护线程什么时候结束

守护线程的作用是在后台运行任务,只要还有一个以上非守护线程没有结束(即便此时主线程已结束),程序就不会结束

> 操作系统提供了哪些接口

命令接口、程序接口、图形界面接口

1.命令接口
提供一组命令供用户直接或间接操作。
根据作业的方式不同，命令接口又分为联机命令接口和脱机命令接口。
2.程序接口
程序接口由一组系统调用命令组成，提供一组系统调用命令供用户程序使用。
3.图形界面接口
通过图标 窗口 菜单 对话框及其他元素,和文字组合,在桌面上形成一个直观易懂 使用方便的计算机操作环境.