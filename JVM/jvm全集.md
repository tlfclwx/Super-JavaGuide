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

## 1.概述

在本篇文章中，你将掌握最常用的 JVM 参数配置。如果对于下面提到了一些概念比如堆、

## 2.堆内存相关

>Java 虚拟机所管理的内存中最大的一块，Java 堆是所有线程共享的一块内存区域，在虚拟机启动时创建。**此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都在这里分配内存。**
>

### 2.1.显式指定堆内存`–Xms`和`-Xmx`

与性能有关的最常见实践之一是根据应用程序要求初始化堆内存。如果我们需要指定最小和最大堆大小（推荐显示指定大小），以下参数可以帮助你实现：

```
-Xms<heap size>[unit] 
-Xmx<heap size>[unit]
```

- **heap size** 表示要初始化内存的具体大小。
- **unit** 表示要初始化内存的单位。单位为***“ g”*** (GB) 、***“ m”***（MB）、***“ k”***（KB）。

举个栗子🌰，如果我们要为JVM分配最小2 GB和最大5 GB的堆内存大小，我们的参数应该这样来写：

```
-Xms2G -Xmx5G
```

> 指定栈的内存大小

- `-Xss`

### 2.2.显式新生代内存(Young Ceneration)

根据[Oracle官方文档](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/sizing.html)，在堆总可用内存配置完成之后，第二大影响因素是为 `Young Generation` 在堆内存所占的比例。默认情况下，YG 的最小大小为 1310 *MB*，最大大小为*无限制*。

一共有两种指定 新生代内存(Young Ceneration)大小的方法：

**1.通过`-XX:NewSize`和`-XX:MaxNewSize`指定**

```
-XX:NewSize=<young size>[unit] 
-XX:MaxNewSize=<young size>[unit]
```

举个栗子🌰，如果我们要为 新生代分配 最小256m 的内存，最大 1024m的内存我们的参数应该这样来写：

```
-XX:NewSize=256m
-XX:MaxNewSize=1024m
```

**2.通过`-Xmn<young size>[unit] `指定**

举个栗子🌰，如果我们要为 新生代分配256m的内存（NewSize与MaxNewSize设为一致），我们的参数应该这样来写：

```
-Xmn256m 
```

GC 调优策略中很重要的一条经验总结是这样说的：

> 将新对象预留在新生代，由于 Full GC 的成本远高于 Minor GC，因此尽可能将对象分配在新生代是明智的做法，实际项目中根据 GC 日志分析新生代空间大小分配是否合理，适当通过“-Xmn”命令调节新生代大小，最大限度降低新对象直接进入老年代的情况。

另外，你还可以通过**`-XX:NewRatio=<int>`**来设置新生代和老年代内存的比值。

比如下面的参数就是设置新生代（包括Eden和两个Survivor区）与老年代的比值为1。也就是说：新生代与老年代所占比值为1：1，新生代占整个堆栈的 1/2。

```
-XX:NewRatio=1
```

### 2.3.显示指定永久代/元空间的大小

**从Java 8开始，如果我们没有指定 Metaspace 的大小，随着更多类的创建，虚拟机会耗尽所有可用的系统内存（永久代并不会出现这种情况）。**

JDK 1.8 之前永久代还没被彻底移除的时候通常通过下面这些参数来调节方法区大小

```java
-XX:PermSize=N //方法区 (永久代) 初始大小
-XX:MaxPermSize=N //方法区 (永久代) 最大大小,超过这个值将会抛出 OutOfMemoryError 异常:java.lang.OutOfMemoryError: PermGen
```

相对而言，垃圾收集行为在这个区域是比较少出现的，但并非数据进入方法区后就“永久存在”了。

**JDK 1.8 的时候，方法区（HotSpot 的永久代）被彻底移除了（JDK1.7 就已经开始了），取而代之是元空间，元空间使用的是直接内存。**

下面是一些常用参数：

```java
-XX:MetaspaceSize=N //设置 Metaspace 的初始（和最小大小）
-XX:MaxMetaspaceSize=N //设置 Metaspace 的最大大小，如果不指定大小的话，随着更多类的创建，虚拟机会耗尽所有可用的系统内存。
```

## 3.垃圾收集相关

### 3.1.垃圾回收器

为了提高应用程序的稳定性，选择正确的[垃圾收集](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)算法至关重要。

JVM具有四种类型的*GC*实现：

- 串行垃圾收集器
- 并行垃圾收集器
- CMS垃圾收集器
- G1垃圾收集器

可以使用以下参数声明这些实现：

```
-XX:+UseSerialGC
-XX:+UseParallelGC
-XX:+USeParNewGC
-XX:+UseG1GC
```

有关*垃圾回收*实施的更多详细信息，请参见[此处](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/jvm/JVM%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.md)。

### 3.2.GC记录

为了严格监控应用程序的运行状况，我们应该始终检查JVM的*垃圾回收*性能。最简单的方法是以人类可读的格式记录*GC*活动。

使用以下参数，我们可以记录*GC*活动：

```
-XX:+UseGCLogFileRotation 
-XX:NumberOfGCLogFiles=< number of log files > 
-XX:GCLogFileSize=< file size >[ unit ]
-Xloggc:/path/to/gc.log
```



## 推荐阅读

- [CMS GC 默认新生代是多大？](https://www.jianshu.com/p/832fc4d4cb53)
- [CMS GC启动参数优化配置](https://www.cnblogs.com/hongdada/p/10277782.html)
- [从实际案例聊聊Java应用的GC优化-美团技术团队](https://tech.meituan.com/2017/12/29/jvm-optimize.html)
- [JVM性能调优详解](https://www.choupangxia.com/2019/11/11/interview-jvm-gc-08/) （2019-11-11）
- [JVM参数使用手册](https://segmentfault.com/a/1190000010603813)

# 运行时数据区
> 运行时数据区内有哪些东西

1.8以前：
线程共享的有堆和方法区（永久代是其实现方式）
线程独立的有本地方法栈、虚拟方法栈和程序计数器

1.8以后
线程共享的有堆
线程独立的有本地方法栈、虚拟方法栈和程序计数器
方法区的实现方法变为元空间放在内存中

> 程序计数器有什么用？生命周期说一下

通过改变程序计数器可以依次读取执行指令，从而实现代码的流控制
另外，每个线程都有一个独立的程序计数器，互不影响。

随着线程的创建而创建，线程消失而消失

> 虚拟机栈有什么用？是如何运作的？

虚拟机栈内是栈帧。
每调用一次函数，对应的栈帧就会被压入虚拟机栈。里面保存了局部变量表，操作数栈、动态链接、方法出口等信息。

> 堆内是存什么的

对象实例和数组

> 所有的对象实例都分配在堆上吗

不是。jdk1.7开始有逃逸分析，如果对象引用没有被返回或者未被外面使用，那么对象可以直接分配在栈上

> 堆内存是如何分布的

年轻代、老年代和永久代（也就是方法区）
其中，年轻代的Eden survivor0 survivor1的比例是8:1:1
年轻代和老年代的比例是1:2

> 年轻代什么时候会变成老年代？

每经过一次垃圾回收，年龄+1（从Eden到s区，年龄初始化为1）。年龄到15就会被晋升到老年代。
有别的情况：
会把年轻代中的所有按年龄排序，如果某个年龄的大小超过了s区的一半时，就取这个年龄和15的最小值作为晋升年龄

>  方法区存储了哪些信息

加载的类的信息，常量，静态变量

> 为什么要将永久代替换为元空间

因为永久代设置空间大小是很难确定的。空间小容易FullGC和OOM，分配大了就会浪费
而元空间是用本地空间

> 说一说运行时常量池的变化

1.7以前，运行时常量池包含字符串常量
1.7 字符串常量池被移到了堆中
1.8 运行时常量池还在方法区中，只不过变成了元空间，到内存中去了

> 讲一下java对象创建的过程

1. 类加载检查
检查能否在常量池中定位到这个类的符号引用，检查是否被加载、验证、准备、解析和初始化

2. 内存分配
为新生对象分配内存，可以使用 指针碰撞 或 空闲列表法，取决于堆是否规整

3. 初始化零值
将分配到的内存空间初始化为0值

4. 设置对象头
设置一些信息在对象头中，比如属于哪个类，年龄等

5. 执行init方法
按照程序员的意愿进行初始化

> 虚拟机对对象初始化时，如何保证线程安全

采用两种方式：
1. CAS+失败重试
2. TLAB，每个线程私有的缓存空间，如果能存的下就存，存不下在放到堆中。TLAB也在堆中。

> 对象的内存分布是怎么样的

对象头、实例数据和对齐填充

> 虚拟机中对象是如何访问定位的

栈上的reference数据来操作堆上的具体对象
有两种方法：使用句柄和直接指针
1. 句柄：reference指向句柄，句柄中存着到对象实例的地址和到对象类型的地址

2. 直接指针：reference直接指向对象实例数据，然后实例数据有指针指向对象类型数据


> 哪些包装类在常量池中有缓存数据，缓存数值是多少

Byte[-128,127]
Short[-128,127]
Integer[-128,127]
Long[-128,127]
Character[0,127]
Boolean直接返回True or False

> Integer i1 =33;
> Integer i2 = 33;
> i1==i2的结果是什么
> 
> Integer i3 = new Integer(33);
> i3==i2的结果是什么

true，因为能缓存到常量池中

false，因为new Integer是创建对象

> 讲一下JVM中两种内存分配的方法

指针碰撞：
内存规整的情况下使用，用一个指针表明左边是用过的，右边是没用过的

空闲列表：
堆内存不规整的时候用，用列表记录哪些内存可用

注意：
在操作系统中，是用位图或者链表法。

# 类加载器

> jvm内置的类加载器说一下，各负责加载什么

BootstrapClassLoader(启动类加载器)：负责加载%JAVA_HOME%/lib目录下的jar包和类或者被-Xbootclasspath参数指定的路径中的所有类

ExtensionClassLoader(扩展类加载器)：加载%JRE_HOME%/lib/ext目录下的包和类，或者java.ext.dirs系统变量指定目录下的jar包

AppClassLoader(应用程序类加载器)：加载classpath下的jar包和类

> 双亲委派机制讲一下

如果要加载一个类，会先判断是否已经加载。如果为加载，会把请求委派给父加载器，如果父加载器还有父，则让父的父加载。如果父为null，则给BootstrapClassLoader。如果父无法处理，才由儿子来处理

> 双亲委派机制的好处

1. 避免了重复类的加载（相同文件，被不同的类加载器加载了，就是不同的类）
2. 保证java核心代码不会被篡改（总是先由最上层来加载）

> 什么时候需要破坏双亲委派机制

因为在某些情况下父类加载器需要委托子类加载器去加载class文件。受到加载范围的限制，父类加载器无法加载到需要的文件，以Driver接口为例，由于Driver接口定义在jdk当中的，而其实现由各个数据库的服务商来提供，比如mysql的就写了MySQL Connector，那么问题就来了，DriverManager（也由jdk提供）要加载各个实现了Driver接口的实现类，然后进行管理，但是DriverManager由启动类加载器加载，只能记载JAVA_HOME的lib下文件，而其实现是由服务商提供的，由系统类加载器加载，这个时候就需要启动类加载器来委托子类来加载Driver实现，从而破坏了双亲委派，这里仅仅是举了破坏双亲委派的其中一个情况。

==由启动类加载器加载的接口要加载其实现类时，由于是服务商提供的，只能委托给应用程序类加载器来加载==

# 类加载过程
> 类的生命周期说一下

加载、连接（验证，准备，解析）、初始化、使用、卸载

> 类的加载过程具体说一下

类的加载包括了加载、连接（验证，准备，解析）、初始化

1. 加载
- 通过全类名获取定义此类的二进制字节流
- 将字节流所代表的静态存储结构转换为运行时数据区的结构
- 在内存中生成一个代表该类的Class对象，作为方法区这些数据的访问入口

2. 验证
- 文件格式验证
- 元数据验证
- 字节码验证
- 符号引用验证

3. 准备
- 为类变量（static）分配内存并设置类变量初始值
- 准备阶段初始化为0值，等到初始化阶段才会赋值。==但是有特殊情况，如果是final static的变量，那么准备阶段就会被赋值了==

4. 解析
- 将常量池内的符号引用替换为直接引用的过程
- 符号引用就是一组符号来描述目标，可以是任何字面量。直接引用就是直接指向目标的指针。

5. 初始化
- 调用`<clint>`方法，就是执行静态代码块
- 只有以下6中情况会发生初始化
  - 遇到new、getstatic、putstatic、invokestatic
  - 反射，比如class.forName(...)，newInstance等
  - 初始化一个类，其父类还没初始化，先让其父类初始化
  - 包含main方法的主类，会先初始化
  - MethodHandle和VarHandle可以看作是轻量级的反射，要用这两个，需要先使用findStaticVarHandle来初始化要调用的类
  - 当一个接口定义了JDK8中新的default接口方法时，如果实现类发生了初始化，那该接口要在其之前被初始化

> 卸载发生的条件

- 该类的所有的实例对象都被GC了，堆中不存在该类的任何对象
- 该类没有在其他地方被引用
- 该类的类加载器的实例已被GC

> 卸载是什么意思

该类的class对象被GC



# 垃圾回收机制

> 讲一下堆常见的分配策略

- 对象优先在Eden区分配
  
- 第一次垃圾回收后，如果s区放不下，那么只能通过==分配担保机制==把新生代的对象提前转移到老年代
  
- 大对象直接进入老年代
  - 新生代如果放不下，直接放到老年代
  - ==如果在分配对象到Eden区的时候发现放不下，触发了GC，还是放不下，那么就会老年代尝试存放，如果老年代也放不下会触发FGC，如若还是放不下直接OOM。==

- 长期存活的对象将进入老年代
  - 大于默认年龄的会进入老年代，一般的垃圾回收器是15，而CMS是6
  - 动态年龄，按照年龄从小到大排序，对占用的大小进行累计，如果累计的某个年龄大小超过了s区的一半，那么取这个年龄和默认年龄的最小值，作为晋升年龄
    ```
    年龄1的对象占用了33%
    年龄2的对象占用33%
    年龄3的对象占用34%。
    年龄2和3都会晋升到老年代
    ```

> JVM中有几种GC讲一下

分为两大类：
- Partial GC
  - Young GC：只收集年轻代的GC
  - Old GC： 只收集老年代的GC
  - Mixed GC: 收集整个年轻代和部分老年代
- Full GC
  - 收集整个堆的，包括年轻代，老年代和永久代

> 说一下各GC的触发条件
- young GC：eden区分配满了的时候。==注意s区满了是不会触发gc的==
- full GC：当要触发young GC时，根据以前的统计数据，young GC平均晋升的大小比目前老年代剩余空间大，那么就触发full GC，不会再发生young GC了
- 如果有永久代的话，永久代剩余空间不足时，会触发full GC
- 老年代空间不足也会触发full GC

> 如何判断一个对象已经无效

- 引用计数法
  - 每被引用一次+1，引用失效-1，引用次数为0的就无效了。
  - 但是存在无法解决相互引用的问题。比如A引用B，B引用A。而这俩没有其他的引用，应该被回收掉，但是此时无法回收。

- 可达性分析算法
  - 从GC Root的对象作为起点，开始搜索引用链，如果一个对象没有到GC Root的引用链，那就是不可用的。
  - 可以作用GC Roots的对象包括
    - 虚拟机栈中引用的对象
    - 本地方法栈中引用的对象
    - 方法区中类静态属性引用的对象
    - 方法区中常量引用的属性
    - 所有被同步锁持有的对象

> 说一下几种引用的情况

- 强引用
  - 大部分引用都是强引用
  - 无论如何垃圾回收器都不会回收
- 软引用
  - 如果内存空间足够，就不会回收，如果不足，就会回收这些引用
- 弱引用
  - 一旦被发现是弱引用，不管内存是否足够，马上将它回收
- 虚引用
  - 就跟没有引用一样
  - 虚引用主要用来跟踪对象被垃圾回收器回收的活动。

> 不可达的对象一定会被回收吗

不一定，在真正被回收之前需要标记两次。
- 第一次是检查这些被标记的是否要进行finalize方法，如果对象没有覆盖finalize方法或者已经执行过了，那么就会被回收
- 第二次：如果这个对象被判定为有必要执行finalize()方法，那么这个对象将会放置在一个叫做F-Queue的队列之中，触发finalize方法，它们有可能重新被引用而逃离死亡。

但是虚拟机并不承诺会等待finalize的结果。如果一个对象在finalize()方法中执行缓慢，或者发生了死循环。

> 如何判断常量是废弃常量，谁负责回收它。

运行时常量池负责回收废弃常量。

如果一个常量没有对象引用它了，就会被回收。

> 如何判断一个类是无用类，谁负责回收它

方法区主要负责回收无用的类。 三个条件
- 该类的所有实例都已经被回收
- 加载该类的classLoader已经被回收
- 该类对应的Class对象没有在任何地方被引用

> 讲一下有哪几种垃圾回收算法，讲一下缺点

- 标记-清除算法：先把不需要回收的对象标记出来，统一回收没有标记的对象
  - 效率问题
  - 空间问题，会产生碎片
- 标记-复制算法：把内存分为两个大小一样的块，标记不需要回收的，复制到另一边，然后清空这一边
  - 浪费空间
- 标记-整理算法：让所有存活的对象向一段移动，然后清理掉端边界以外的内存
  - 算法复杂度大，步骤复杂
- 分代收集算法
  - 根据对象存活周期的不同，将内存分为年轻代，老年代和永久代
  - 年轻代有大量的对象死去，就用标记-复制算法
  - 老年代存活率高用标记-清除或标记-整理

> 为什么HotSpot要分为年轻代和老年代

因为使用了分代收集算法，可以根据对象存活的时间，选择不同的算法

> 讲一下有哪些垃圾收集器

- Serial收集器
  - 单线程，必须停止所有工作线程stop the world
  - 年轻代使用标记-复制算法，老年代使用标记-整理算法
- ParNew收集器
  - 就是并行的Serial，多线程版本。
  - 年轻代标记-复制，老年代标记-整理
- Parallel Scavenge收集器
  - 关注吞吐量的多线程收集器
  - 年轻代标记-复制，老年代标记-整理
  - JDK8默认收集器
- Serial Old
  - Serial的老年代版本
- Parallel Old 
  - Parallel Scavenge的老年代版本
- CMS
  - 是并发收集器，实现最短停顿时间
  - 是一种标记-清除算法，会有空间碎片
  - 四个步骤
    - 初始标记：stw，暂停所有的其他线程，记录下与root相连的对象，速度很快
    - 并发标记：同时启动GC和用户线程，记录Roots的可达对象
    - 重新标记：stw，修正并发阶段因为用户行为导致的变动。并发
    - 并发清除：开启用户线程，并发清除垃圾
![](https://gitee.com/super-jimwang/img/raw/master/img/20210301153937.png)

> 如果频繁发生full gc是什么原因

可能有内存泄漏了

> 举几个会发生内存泄漏的例子

被静态集合引用的对象。`static List<> ls; ls.add(a)`。因为静态对象的生命周期跟JVM一样，JVM不结束静态集合就不会销毁。 

hash会改变的对象。比如set.add(这个对象)，然后对象的hash变了，这时候想去set里删除这个对象是不行的。

> 哪些东西会被当作垃圾回收？

不在引用链上的所有对象，以及在链上的软、弱、虚引用对象。