# java基础面试题

> 分析一下代码，变量的分配空间在哪

```java
int rows = nums.length;
int cols = nums[0].length;
int[][] dp = new int[rows][cols];
int MAX = Integer.MIN_VALUE;
```

rows、cols、MAX是分配在栈上（都是32位，4字节），dp的数组分配在堆上，dp引用分配在栈上（存的是地址，根据操作系统的位数判断，如果32位就4字节，64位就8字节）

> java程序是如何运行的？

先编译成字节码，然后交给jvm运行

> 什么时候会进行类加载

以下情况会触发类的初始化：

遇到new，getstatic，putstatic，invokestatic这4条指令；
使用java.lang.reflect包的方法对类进行反射调用；
初始化一个类的时候，如果发现其父类没有进行过初始化，则先初始化其父类（注意！如果其父类是接口的话，则不要求初始化父类）；
当虚拟机启动时，用户需要指定一个要执行的主类（包含main方法的那个类），虚拟机会先初始化这个主类；
当使用jdk1.7的动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果REF_getstatic，REF_putstatic,REF_invokeStatic的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则先触发其类初始化；

> 线程什么时候会进入阻塞状态

①线程通过调用sleep方法进入睡眠状态；

②线程调用一个在I/O上被阻塞的操作，即该操作在输入输出操作完成之前不会返回到它的调用者；

③线程试图得到一个锁，而该锁正被其他线程持有；

④线程在等待某个触发条件；

> ==和equal的区别

\=\=：表示两个变量的值是否相等，比较两个基本数据类型的数据或者引用变量，用\=\=。
equals:用于比较两个独立对象的内容是否相同。字符串的比较也用equals。

> int 和 Integer的区别

Int是Java的8中基本数据类型之一，integer是int的封装类。Int类型的默认值为0，integer默认值为null，所以区别在于，integer能区分出null值和0的区别。

> 重载和重写的区别？

重载（Overload）：函数名相同，参数不同。可以改变参数的个数和类型。==光返回值类型不同不行！！==
重写（Override）：和父类的的方法名称、参数完全相同。

>String和StringBuffuer、StringBuilder的区别？

String:字符串数值不可变；
StringBuffer：字符串可修改，可以动态构造字符数据。StringBuffer类是可以通过Append()来修改值。线程安全。
StringBuilder：线程不安全。

三者在执行速度方面的比较：StringBuilder >  StringBuffer  >  String
对于三者使用的总结：
1.如果要操作少量的数据用 = String　　
2.单线程操作字符串缓冲区下操作大量数据 = StringBuilder
3.多线程操作字符串缓冲区下操作大量数据 = StringBuffer

> java中有几种方法实现一个线程？用什么关键字修饰同步方法？

第一种：继承Thread类。
```java
Thread t = new Thread(){
	public void run(){
		//要执行的任务
	}
}
// 启动线程
t.setName("thread1");
t.start();
```
第二种：实现Runable接口。
```java
Runnable runnable = new Runnable(){
  public void run(){
    //要执行的任务
  }
};
// 创建线程对象
Thread t = new Thread(runnable);
t.start();
```
第三种：线程池创建多线程。
第四种：实现Callable接口，重写call函数（
继承Thread类实现多线程，重写run方法时没有返回值也不能抛出异常，使用Callable接口就可以解决这个问题。

用synchronized 关键字修饰同步方法。

>stop()和suspend()方法为何不推荐使用？那该如何终止线程？

suspend在调用后，线程不会释放已经占有的资源比如锁，而是带着资源进入睡眠状态，十分容易引发死锁。同样stop方法在终结一个线程时，不能保证线程资源的正常释放，通常没有基于线程完成资源释放的机会。

使用中断方法interupt或者cancel标志位方法都有机会让线程取释放自己占有的资源，这种方法更安全和优雅。

> Callable接口和Runnable接口的不同之处：

1.Callable规定的方法是call，而Runnable是run
2.call方法可以抛出异常，但是run方法不行
3.Callable对象执行后可以有返回值，运行Callable任务可以得到一个Future对象，通过Future对象可以了解任务执行情况，可以取消任务的执行，而Runnable不可有返回值
）

> sleep()、wait()和yield()有什么区别？

sleep和yield都是Thread的静态方法，wait是Object的方法

sleep是线程被调用时，让出CPU，进入waited_waiting，但是占着资源（比如锁），事件到了自动进入就绪状态。

wait是进入等待池等待，需要notify和synchronized配合使用，让出系统资源和cpu。

yield与sleep()不同的是，当前线程执行yield()后，也就是告诉CPU，当前线程已经执行的差不多了，线程调度器可以将当前的CPU让给那些比当前线程优先级更高的线程或者优先级和当前线程同级的线程去执行，如果没有这样的线程，那么当前线程继续执行，如果有这样的线程，那么当前线程进入就绪状态。

> 请对比synchronized与java.util.concurrent.locks.Lock的异同？

主要相同点：Lock能完成synchronized所实现的所有功能
主要不同点：Lock有比synchronized更精确的线程语义和更好的性能。synchronized会自动释放锁，而Lock一定要求程序员手工释放，并且必须在finally从句中释放。

那为什么还要用synchronized？
- 因为软件测试里讲过，为了降低出现错误的可能性，synchronized是自动加锁释放，而lock需要手动。

> String  s =new  String (“syz”);创建了几个String Object?

1.如果String常量池(常量缓冲区)中，已经创建"xyz"，则不会继续创建，此时只创建了一个对象new String("xyz")；
2.如果String常量池中，没有创建"xyz"，则会创建两个对象，一个对象的值是"xyz"，一个对象new String("xyz")。

> java的四种权限作用域是什么？说一下其范围和默认的作用域是什么？

|作用域	|当前类	|同一包（package）|	子孙类|	其他包|
|--|--|--|--|--|
|public|	Y	|Y	|Y	|Y|
|protected|	Y|	Y|	Y|	N|
|default	|Y	|Y	|N	|N|
|private|	Y|	N|	N|	N|

默认情况下就是default，当前类和同一个包内可以使用。

==protected的情况下，A包中的test1类有一个protected属性x，B包中的test2类中new了一个test1对象，无法访问x属性，因为其他包无法访问。==

> forward和redirect两种跳转方式的区别是什么

1.从地址栏显示来说
forward是服务器请求资源,服务器直接访问目标地址的URL,把那个URL的响应内容读取过来,然后把这些内容再发给浏览器.浏览器根本不知道服务器发送的内容从哪里来的,所以它的地址栏还是原来的地址.
redirect是服务端根据逻辑,发送一个状态码,告诉浏览器重新去请求那个地址.所以地址栏显示的是新的URL.

2.从数据共享来说
forward:转发页面和转发到的页面可以共享request里面的数据.
redirect:不能共享数据.

3.从运用地方来说
forward:一般用于用户登陆的时候,根据角色转发到相应的模块.
redirect:一般用于用户注销登陆时返回主页面和跳转到其它的网站等.

4.从效率来说
forward:高.
redirect:低.

本质上说, 转发是服务器行为，重定向是客户端行为。其工作流程如下：
转发过程：==客户浏览器发送http请求----》web服务器接受此请求--》调用内部的一个方法在容器内部完成请求处理和转发动作----》将目标资源发送给客户；在这里，转发的路径必须是同一个web容器下的url==，其不能转向到其他的web路径上去，中间传递的是自己的容器内的request。在客户浏览器路径栏显示的仍然是其第一次访问的路径，也就是说客户是感觉不到服务器做了转发的。转发行为是浏览器只做了一次访问请求。

重定向过程：==客户浏览器发送http请求----》web服务器接受后发送302状态码响应及对应新的location给客户浏览器--》客户浏览器发现是302响应，则自动再发送一个新的http请求，请求url是新的location地址----》服务器根据此请求寻找资源并发送给客户。== 在这里 location可以重定向到任意URL，既然是浏览器重新发出了请求，则就没有什么request传递的概念了。在客户浏览器路径栏显示的是其重定向的路径，客户可以观察到地址的变化的。重定向行为是浏览器做了至少两次的访问请求的。

> HashMap和HashTable的区别

HashMap：实现了Map接口，允许空(null)键值(key),由于非线程安全，在只有一个线程访问的情况下，效率高于Hashtable。
Hashtable：不能将null作为key或者value。方法是同步的，线程安全。==方法上都标记了synchronized==

> List、Set和Map的区别

List:是存储单列数据的集合，存储有顺序，允许重复。继承Collection接口。
Set: 是存储单列数据的集合。继承Collection接口。不允许重复。
Map:存储键和值这样的双列数据的集合，存储数据无顺序，键(key)不能重复，值(value)。可以重复。

> java创建对象的方式有哪些

1.使用new关键字
2.使用反射机制创建对象：
(1)使用Class类的newInstance方法
(2)java.lang.reflect.Constructor类里也有一个newInstance方法可以创建对象。
3.使用clone方法：先实现Cloneable接口并实现其定义的clone方法
4.使用反序列化

> error和exception的区别。exception的分类

Error（错误）表示系统级的错误和程序不必处理的异常，是java运行环境中的内部错误或者硬件问题。比如：内存资源不足等。对于这种错误，程序基本无能为力，除了退出运行外别无选择，它是由Java虚拟机抛出的。
Exception（违例）表示需要捕捉或者需要程序进行处理的异搜索常，它处理的是因为程序设计的瑕疵而引起的问题或者在外的输入等引起的一般性问题，是程序必须处理的。
Exception又分为运行时异常，受检查异常。

- 运行时异常，表示无法让程序恢复的异常，导致的原因通常是因为执行了错误的操作，建议终止程序，因此，编译器不检查这些异常。
- 受检查异常，是表示程序可以处理的异常，也即表示程序可以修复（由程序自己接受异常并且做出处理）， 所以称之为受检查异常。

> Error、运行时异常和受查异常有哪些

**运行时异常**
NullPointerException - 空指针引用异常
ClassCastException - 类型强制转换异常。
IllegalArgumentException - 传递非法参数异常。
ArithmeticException - 算术运算异常
ArrayStoreException - 向数组中存放与声明类型不兼容对象异常
IndexOutOfBoundsException - 下标越界异常
NegativeArraySizeException - 创建一个大小为负数的数组错误异常
NumberFormatException - 数字格式异常
SecurityException - 安全异常
UnsupportedOperationException - 不支持的操作异常

**受查异常**
EOFException 文件已结束异常
FileNotFoundException  文件未找到异常
SQLException  操作数据库异常
IOException  输入输出异常
NoSuchMethodException  方法未找到异常

**Error**
OutOfMemory
StackOverflow

> JDBC使用步骤的过程

```java
// 1.注册驱动
Class.forName("com.mysql.jdbc.Driver");
// 2.获取连接
String url = "jdbc:mysql://localhost:3306/wyj";
String user = "root";
String password = "12345678";
conn = DriverManager.getConnection(url, user, password);
// 3.获取数据库操作对象
stmt = conn.createStatement();
// 4.创建statement
String sql = "insert into dept (deptno,dname,loc) values (50,'人事部','北京')";
// 5.执行sql
int count = stmt.executeUpdate(sql);
System.out.println(count==1?"保存成功":"保存失败"); //因为insert只影响一条记录

// 6.处理查询结果集

// 7. 关闭jdbc对象
stmt.close();
conn.close();
```

> 抽象类和接口的区别

抽象类：用abstract修饰，抽象类不能创建实例对象。抽象方法必须在子类中实现，不能有抽象构造方法或者抽象静态方法。
接口：抽象类的一种特例，接口中的方法必须是抽象的。
两者的区别：
1. 抽象类可以有构造方法，接口没有构造方法
2. 抽象类可以有普通成员变量，接口没有普通成员变量。
3. 抽象类可以有非抽象的普通方法，接口中的方法必须是抽象的。
4. 抽象类中的抽象方法访问类型可以是public，protected，接口中抽闲方法必须是public类型的。
5. 抽象类可以包含静态方法，接口中不能包含静态方法。
6. 一个类可以实现多个接口，但是只能继承一个抽象类。
7. 接口中基本数据类型的数据成员，都默认为static和final，抽象类则不是。

> hashCode与equals的区别和联系

一、equals方法的作用
1、默认情况（没有覆盖equals方法）下equals方法都是调用Object类的equals方法，而Object的equals方法主要用于判断对象的内存地址引用是不是同一个地址（是不是同一个对象）。
2 、要是类中覆盖了equals方法，那么就要根据具体的代码来确定equals方法的作用了，覆盖后一般都是通过对象的内容是否相等来判断对象是否相等。

二、Hashcode（）方法：
1、我们并没有覆盖equals方法只覆盖了hashCode方法，两个对象虽然hashCode一样，但在将stu1和stu2放入set集合时由于equals方法比较的两个对象是false，所以就没有在比较两个对象的hashcode值。
2、覆盖一下equals方法和hashCode方法，stu1和stu2通过equals方法比较相等，而且返回的hashCode值一样，所以放入set集合中时只放入了一个对象。
3、我们让两个对象equals方法比较相等，但hashCode值不相等试试，虽然stu1和stu2通过equals方法比较相等，但两个对象的hashcode的值并不相等，所以在将stu1和stu2放入set集合中时认为是两个不同的对象。

> HashSet是如何比较是否属于同一个对象的？

先判断hashcode是否相同，如果相同再判断equals方法

> String是否重写了toString()?

toString是对自我描述的方法：getClass().getName() + "@" +Integer.toHexString(hashCode());
而String重写了，直接输出的是对象存储的字符串。

> 什么是反射？以及四种反射获取

JAVA 反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为 java 语言的反射机制。

1.知道具体类的情况下可以使用：
`Class alunbarClass = TargetObject.class;`

2.通过 Class.forName()传入类的路径获取：
`Class alunbarClass1 = Class.forName("cn.javaguide.TargetObject");`

3.通过对象实例instance.getClass()获取：
`Employee e = new Employee();
Class alunbarClass2 = e.getClass();`

4.通过类加载器xxxClassLoader.loadClass()传入类路径获取
`class clazz = ClassLoader.LoadClass("cn.javaguide.TargetObject");`
这种方式获取的class对象不会进行初始化。意味着不进行包括初始化等一些列步骤，静态块和静态对象不会得到执行

> final的含义，用在类上，变量上和方法上的含义

用在类上表明不可被继承，所有的成员方法都会被指定为final

用在方法上表示不能被重写

用在了变量上，表明是常量，不能被修改。

> static的四种场景

- **修饰成员变量和成员方法:** 被 static 修饰的成员属于类，不属于单个这个类的某个对象，被类中所有对象共享，可以并且建议通过类名调用。被static 声明的成员变量属于静态成员变量，静态变量 存放在 Java 内存区域的方法区。调用格式：类名.静态变量名 类名.静态方法名()
- **静态代码块:** 静态代码块定义在类中方法外, 静态代码块在非静态代码块之前执行(静态代码块—>非静态代码块—>构造方法)。 该类不管创建多少对象，静态代码块只执行一次.
  - 静态代码块对于定义在它之后的静态变量，可以赋值，但是不能访问.
  ```java
  public class demo {
      static {
          i = 3; //可以赋值
          System.out.println(i);// 出错。不能访问
      }
      private static int i;
  }
  ```
- **静态内部类（static修饰类的话只能修饰内部类）：** 静态内部类与非静态内部类之间存在一个最大的区别: 非静态内部类在编译完成之后会隐含地保存着一个引用，该引用是指向创建它的外围类，但是静态内部类却没有。没有这个引用就意味着：1. 它的创建是不需要依赖外围类的创建。2. 它不能使用任何外围类的非static成员变量和方法。
  - 会加载，但是不会初始化，内部静态类不会自动初始化，只有调用静态内部类的方法，静态域，或者构造方法的时候才会加载静态内部类。
  - 因此可以用这个特性来实现单例模式
  ```java
  package Test;
    public class Single {
        
        private Single() {}
        
        static class SingleHolder {
            
            private static final Single instance=new Single(); 
            
        }
        public static Single getinstance() {
            return SingleHolder.instance;
        }
    
        public static void main(String[] args) {
            
            Single a=Single.getinstance();
            Single b=Single.getinstance();
            Single c=Single.getinstance();
            System.out.println(a.toString());
            System.out.println(b.toString());
            System.out.println(c.toString());
        }
    
    }
  ```


- **静态导包(用来导入类中的静态资源，1.5之后的新特性):** 格式为：import static 这两个关键字连用可以指定导入某个类中的指定静态资源，并且不需要使用类名调用类中静态成员，可以直接使用类中静态成员变量和成员方法。

> java中的BIO

![](https://gitee.com/super-jimwang/img/raw/master/img/20210224160851.png)
采用 BIO 通信模型 的服务端，通常由一个独立的 Acceptor 线程负责监听客户端的连接。我们一般通过在while(true) 循环中服务端会调用 accept() 方法等待接收客户端的连接的方式监听请求，请求一旦接收到一个连接请求，就可以建立通信套接字在这个通信套接字上进行读写操作，此时不能再接收其他客户端连接请求，只能等待同当前连接的客户端的操作执行完成， 不过可以通过多线程来支持多个客户端的连接，如上图所示。

如果要让 BIO 通信模型 能够同时处理多个客户端请求，就必须使用多线程（主要原因是socket.accept()、socket.read()、socket.write() 涉及的三个主要函数都是同步阻塞的），也就是说它在接收到客户端连接请求之后为每个客户端创建一个新的线程进行链路处理，处理完成之后，通过输出流返回应答给客户端，线程销毁。这就是典型的 一请求一应答通信模型 。

> java中的伪异步IO

为了解决同步阻塞I/O面临的一个链路需要一个线程处理的问题，后来有人对它的线程模型进行了优化一一一后端通过一个线程池来处理多个客户端的请求接入，形成客户端个数M：线程池最大线程数N的比例关系，其中M可以远远大于N.通过线程池可以灵活地调配线程资源，设置线程的最大值，防止由于海量并发接入导致线程耗尽。
![](https://gitee.com/super-jimwang/img/raw/master/img/20210224161053.png)
采用线程池和任务队列可以实现一种叫做伪异步的 I/O 通信框架，它的模型图如上图所示。当有新的客户端接入时，将客户端的 Socket 封装成一个Task（该任务实现java.lang.Runnable接口）投递到后端的线程池中进行处理，JDK 的线程池维护一个消息队列和 N 个活跃线程，对消息队列中的任务进行处理。由于线程池可以设置消息队列的大小和最大线程数，因此，它的资源占用是可控的，无论多少个客户端并发访问，都不会导致资源的耗尽和宕机。

伪异步I/O通信框架采用了线程池实现，因此避免了为每个请求都创建一个独立线程造成的线程资源耗尽问题。不过因为它的底层仍然是同步阻塞的BIO模型，因此无法从根本上解决问题。

> java中的NIO与IO的不同

NIO中的N可以理解为Non-blocking，不单纯是New。它支持面向缓冲的，基于通道的I/O操作方法。 NIO提供了与传统BIO模型中的 Socket 和 ServerSocket 相对应的 SocketChannel 和 ServerSocketChannel 两种不同的套接字通道实现,两种通道都支持阻塞和非阻塞两种模式。

- **IO是阻塞的，NIO是非阻塞的**
  - 单线程中从通道读取数据到buffer，同时可以继续做别的事情，当数据读取到buffer中后，线程再继续处理数据。写数据也是一样的。另外，非阻塞写也是如此。一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。
- **IO 面向流(Stream oriented)，而 NIO 面向缓冲区(Buffer oriented)**
  - 在NIO厍中，所有数据都是用缓冲区处理的。在读取数据时，它是直接读到缓冲区中的; 在写入数据时，写入到缓冲区中。任何时候访问NIO中的数据，都是通过缓冲区进行操作。
- **NIO通过Channel进行读写**
  - 通道是双向的，可读也可写，而流的读写是单向的。无论读写，通道只能和Buffer交互。因为 Buffer，通道可以异步地读写。
- **IO没有选择器，而NIO有**
  - 选择器用于使用单个线程处理多个通道。因此，它需要较少的线程来处理这些通道。线程之间的切换对于操作系统来说是昂贵的。 因此，为了提高系统效率选择器是有用的。

> IO多路复用为什么要配合非阻塞的读写IO？

因为当用select时，通知可读后，不一定真的可读。那么如果调用阻塞io，整个线程就都阻塞了。

select通知后，内核是收到了数据，但是在实际运行中可能出现各种意外导致不能被正常送到用户，比如：经过校验是坏数据，丢弃；锁没加好，数据被另外一个线程拿走……这样还是可能发生阻塞。

> JAVA中NIO如何读取数据和写数据

selector.select监视文件描述符，阻塞。
如果有准备就绪的就返回key。创建channel，通过缓冲区把数据读进来

通常来说NIO中的所有IO都是从 Channel（通道） 开始的。

从通道进行数据读取 ：创建一个缓冲区，然后请求通道读取数据。
从通道进行数据写入 ：创建一个缓冲区，填充数据，并要求通道写入数据。

> JAVA中NIO的三个核心组件

- selector用来选择感兴趣的事件，然后使用select进行监视
- Channel 通道，与缓冲区交互
- Buffer 一个通道可以对应多个buffer

在selector上注册channel，每个channel表示一个感兴趣事件fd，一个channel可以对应多个buffer，channel从buffer中读写数据。

```java
// channel是一个文件，从buffer中读数据，写入channel
public void openAndWrite() throws IOException {
    FileChannel channel = FileChannel.open(Paths.get("my.txt"), StandardOpenOption.CREATE, StandardOpenOption.WRITE);
    ByteBuffer buffer = ByteBuffer.allocate(64);  //使用allocate函数分配缓冲区空间
    buffer.putChar('A').flip();  //flip()函数将缓冲区由写模式转变为读模式
    channel.write(buffer);  //从buffer中读数据到channel
}
```

> finally 代码块和 finalize()方法有什么区别?

finally 代码块和 finalize()方法有什么区别?
无论是否抛出异常,finally 代码块都会执行,它主要是用来释放应用占用的资源。
finalize()方法是 Object 类的一个 protected 方法,它是在对象被垃圾回收之前由 Java 虚拟机来调用的。不要依赖使用finalize方法，因为很难知道这个方法什么时候被调用。

> throw 和 throws 有什么区别?

throw 关键字用来在程序中明确的抛出异常,相反,throws 语句用来表明方法不能处理的异常。throws用在声明方法的时候，表示该方法可能抛出的异常，交给上层调用catch来处理

> 泛型T和?的区别

T代表某一类具体对象，list里只能存放一类。？是占位不知道list里会存放多少种类型的数据

> try-catch块中存在return语句，是否还执行finally块，如果执行，说出执行顺序

try-catch中存在return时，会在return前把finally执行掉。
如果finally中有return会真的return，返回的数值是finally中修改的。
如果finally中没有return,==返回的数值仍是try中的数值。==

> **String.intern()的理解**

一、new String都是在堆上创建字符串对象。当调用 intern() 方法时，编译器会将字符串添加到常量池中（stringTable维护），并返回指向该常量的引用。
![](https://gitee.com/super-jimwang/img/raw/master/img/20210224163725.png)

二、通过字面量赋值创建字符串（如：String str=”twm”）时，会先在常量池中查找是否存在相同的字符串，若存在，则将栈中的引用直接指向该字符串；若不存在，则在常量池中生成一个字符串，再将栈中的引用指向该字符串。
![](https://gitee.com/super-jimwang/img/raw/master/img/20210224163856.png)

三、常量字符串的“+”操作，编译阶段直接会合成为一个字符串。如string str=”JA”+”VA”，在编译阶段会直接合并成语句String str=”JAVA”，于是会去常量池中查找是否存在”JAVA”,从而进行创建或引用。

四、对于final字段，编译期直接进行了常量替换（而对于非final字段则是在运行期进行赋值处理的）。
final String str1=”ja”;
final String str2=”va”;
String str3=str1+str2;
在编译时，直接替换成了String str3=”ja”+”va”，根据第三条规则，再次替换成String str3=”JAVA”

五、常量字符串和变量拼接时（如：String str3=baseStr + “01”;）会调用stringBuilder.append()在堆上创建新的对象。

六、JDK 1.7后，intern方法还是会先去查询常量池中是否有已经存在，如果存在，则返回常量池中的引用，这一点与之前没有区别，区别在于，如果在常量池找不到对应的字符串，则不会再将字符串拷贝到常量池，而只是在常量池中生成一个对原字符串的引用。简单的说，就是往常量池放的东西变了：原来在常量池中找不到时，复制一个副本放到常量池，1.7后则是将在堆上的地址引用复制到常量池。
![](https://gitee.com/super-jimwang/img/raw/master/img/20210224164414.png)


代码理解
String s = new String("1");①
s.intern();②
String s2 = "1";③
System.out.println(s == s2);④

String s3 = new String("1") + new String("1");⑤
s3.intern();⑥
String s4 = "11";⑦
System.out.println(s3 == s4);⑧

输出：
JDK1.6以及以下：false false
JDK1.7以及以上：false true

分析：
1.6以前，常量池在方法区，是直接复制字符串过去的。①在常量池创建了"1"，也在堆中创建了String对象，因此②没有影响==因为intern只在常量池中没有该字符串的时候生效==，③是字面量赋值，直接指向了常量池，因此④是false。⑤由于常量池已经有"1"了，因此只在堆中创建String，⑥由于常量池没有"11"，因此创建"11"，⑦指向常量池，因此⑧false==因为7是指向常量池的，而6是指向堆中的对象的==。
1.7以后，⑥在常量池中生成⑤对象的引用，⑦指向他，因此⑤=\=⑦。==7指向堆中的6==


String s = new String("1");①
String s2 = "1";②
s.intern();③
System.out.println(s == s2);④

String s3 = new String("1") + new String("1");⑤
String s4 = "11";⑥
s3.intern();⑦
System.out.println(s3 == s4);⑧

输出：
JDK1.6以及以下：false false
JDK1.7以及以上：false false

分析：
此处⑦没有作用，因为⑥已经在常量池中生成了"11"。

> 什么时候finally不会执行

如果在try块或者catch块中调用了退出虚拟机的方法（即System.exit();）
那么finally中的代码不会执行，不然无论在try块、catch块中执行任何代码，
出现任何情况，异常处理的finally中的代码都要被执行的。

> 一个“.java”源文件中是否可以包括多个类（不是内部类）？有什么限制？

一个“.java”源文件中可以包括多个类（不是内部类）！
单一个文件中只能有一个public类，并且该public类必须与文件名相同

> **java 数组调用clone方法，返回的对象是深拷贝还是浅拷贝**

一维数组深拷贝（重新分配内存，并复制值）
二维数组浅拷贝（只传递引用）
注：若要实现二维数组的深拷贝，可以把二维数组内的每个数组分别用clone()方法复制。

> 反射可以调用private方法吗

可以
```java
public class ReflectDemo {
    public static void main(String[] args) throws Exception {
        Method method = PackageClazz.class.getDeclaredMethod("privilegedMethod", new Class[]{String.class,String.class});   
        method.setAccessible(true);
        method.invoke(new PackageClazz(), "452345234","q31234132");
    }
}

class PackageClazz {
    private void privilegedMethod(String invokerName,String adb) {
        System.out.println("---"+invokerName+"----"+adb);
    }
} 
```

> 可以用反射设置对象的private属性吗，并且该属性没有set方法

可以
```java
public void fieldTest() throws Exception {
  User user = userClass.newInstance();
  // 获取当前类所有属性
  Field fields[] = userClass.getDeclaredFields();
  // 获取公有属性(包括父类)
  // Field fields[] = cl.getFields();
  // 取消安全性检查,设置后才可以获取或者修改private修饰的属性，也可以单独对某个属性进行设置
  Field.setAccessible(fields, true);`
  Field fieldUserName = userClass.getDeclaredField("name");
  // 取消安全性检查,设置后才可以获取或者修改private修饰的属性，也可以批量对所有属性进行设置
  fieldUserName.setAccessible(true);
  fieldUserName.set(user, "韦德");
}
```

> 基础变量的内存大小

char 16bit
byte 8bit
short 16bit
int 32bit
long 64bit
float 32bit
double 64bit
boolean 没有明确定义

> NIO 主要解决的问题

如果采用阻塞式IO，单线程情况下，处理者线程可能阻塞在其中一个套接字的read上，导致另一个套接字即使准备好了数据也无法处理，这个时候解决的方法就是针对每一个套接字，都新建一个线程处理其数据读取。

NIO
如果将套接字读操作换成非阻塞的，那么只需要一个线程就可以同时处理套接字，每次检查一个套接字，有数据则读取，没有则检查下一个，因为是非阻塞的，所以执行read操作时若没有数据准备好则立即返回，不会发生阻塞。

> 静态代理写一个

```java
1.定义发送短信的接口
public interface SmsService {
    String send(String message);
}
public class SmsServiceImpl implements SmsService {
    public String send(String message) {
        System.out.println("send message:" + message);
        return message;
    }
}
public class SmsProxy implements SmsService {
​
    private final SmsService smsService;
​
    public SmsProxy(SmsService smsService) {
        this.smsService = smsService;
    }
​
    @Override
    public String send(String message) {
        //调用方法之前，我们可以添加自己的操作
        System.out.println("before method send()");
        smsService.send(message);
        //调用方法之后，我们同样可以添加自己的操作
        System.out.println("after method send()");
        return null;
    }
}
public class Main {
    public static void main(String[] args) {
        SmsService smsService = new SmsServiceImpl();
        SmsProxy smsProxy = new SmsProxy(smsService);
        smsProxy.send("java");
    }
}
```

> JDK动态代理模式写一个

```java
1. 定义被代理类接口和实现
2. 自定义InvocationHandler并重写Invoke方法
3. 通过Proxy.newProxyInstance方法创建代理类

1.定义发送短信的接口
public interface SmsService {
    String send(String message);
}
public class SmsServiceImpl implements SmsService {
    public String send(String message) {
        System.out.println("send message:" + message);
        return message;
    }
}
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
​
/**
 * @author shuang.kou
 * @createTime 2020年05月11日 11:23:00
 */
public class DebugInvocationHandler implements InvocationHandler {
    /**
     * 代理类中的真实对象
     */
    private final Object target;
​
    public DebugInvocationHandler(Object target) {
        this.target = target;
    }
​
​
    public Object invoke(Object proxy, Method method, Object[] args) throws InvocationTargetException, IllegalAccessException {
        //调用方法之前，我们可以添加自己的操作
        System.out.println("before method " + method.getName());
        Object result = method.invoke(target, args);
        //调用方法之后，我们同样可以添加自己的操作
        System.out.println("after method " + method.getName());
        return result;
    }
}
​
public class JdkProxyFactory {
    public static Object getProxy(Object target) {
        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(), // 目标类的类加载
                target.getClass().getInterfaces(),  // 代理需要实现的接口，可指定多个
                new DebugInvocationHandler(target)   // 代理对象对应的自定义 InvocationHandler
        );
    }
}
SmsService smsService = (SmsService) JdkProxyFactory.getProxy(new SmsServiceImpl());
smsService.send("java");
```

> CGLIB动态代理写一个

```java

public class HelloService {
 
    public HelloService() {
        System.out.println("HelloService构造");
    }
 
    /**
     * 该方法不能被子类覆盖,Cglib是无法代理final修饰的方法的
     */
    final public String sayOthers(String name) {
        System.out.println("HelloService:sayOthers>>"+name);
        return null;
    }
 
    public void sayHello() {
        System.out.println("HelloService:sayHello");
    }
}

public class MyMethodInterceptor implements MethodInterceptor{
 
    /**
     * sub：cglib生成的代理对象
     * method：被代理对象方法
     * objects：方法入参
     * methodProxy: 代理方法
     */
    @Override
    public Object intercept(Object sub, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("======插入前置通知======");
        Object object = methodProxy.invokeSuper(sub, objects);
        System.out.println("======插入后者通知======");
        return object;
    }
}


public class Client {
    public static void main(String[] args) {
        // 代理类class文件存入本地磁盘方便我们反编译查看源码
        System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, "D:\\code");
        // 通过CGLIB动态代理获取代理对象的过程
        Enhancer enhancer = new Enhancer();
        // 设置enhancer对象的父类
        enhancer.setSuperclass(HelloService.class);
        // 设置enhancer的回调对象
        enhancer.setCallback(new MyMethodInterceptor());
        // 创建代理对象
        HelloService proxy= (HelloService)enhancer.create();
        // 通过代理对象调用目标方法
        proxy.sayHello();
    }
}
```

> 32位操作系统会为每个进程分配多大的内存空间？为什么能为每个进程分配那么多虚拟内存空间？地址空间是连续的吗？

3GB的进程空间 1GB留给内核
因为有2的32次方个地址，每个地址能存1字节。在内存中，数据存储的最小单位是1个字节
不连续

> 底层如何实现重载和重写？

**重载**： 静态分配，在编译的时候就决定了
**重写**： 动态分配
动态分配并不会频繁操作去搜索合适的目标方法，而是通过在类的方法区中建立一个虚方法表（Virtual Method Table=vtable）来代替元数据查找以提高性能。Father与Son的虚方法表如下图（Vtable中存放各个方法的实际入口地址）

![](https://gitee.com/super-jimwang/img/raw/master/img/20210317192829.png)

　解释1：如果子类没有重写方法，执行的是父类的方法：这是因为子类中没有被重写，那么子类的虚方法表里面的地址入口与父类相同方法的地址入口是一致的，都指向父类的实现入口。即：Father与Son的choiceA与choiceB方法都指向了Father相同方法中的入口地址。

　　解释2：如果子类重写父类方法，那么执行的是子类的方法，这是因为子类方法表中地址将会被替换为指向子类实现版本的入口地址。即：Son的choiceA与choiceB方法被重写了，则指向了Son实现版本的入口地址，并没有指向Father的箭头。

　　解释3：Father与Son从Object继承来的方法都没有重写，故都会指向Object的数据类型；

> 所有类都继承Object类这句话对吗

不对。

在创建一个A类型对象的时候,如果该对象没有继承类,那么在编译时,编译器会自动让该A类继承于object类,如果A类继承了父类B,这个B父类上面没有父类了,则B继承于object类,A直接使用B继承的object类的成员,但是如果B类上面还有很多的父类,那么从B类开始一直到顶级父类的直接子类处都不会继承object类,只有顶级父类自己继承于object类,因为这个继承体系中的每一层都有自己的父类,所有都不会直接继承object类,只有顶级父类可以直接继承object类,然后提供给每一层的子类对象去使用

> Object类都有哪些方法？.

- hashcode()
- equals()
- notify()
- wait()
- clone()
- finalize()

> lambda的底层原理

**静态方法中使用lambda时**

生成内部类，生成一个私有方法，在内部类中调用这个私有方法。

```java
public class Main {
    public String sayHello(Function<String,String> f){
        return f.apply("aa");
    }
    public static void main(String argv[]){
        Main main=new Main();
        System.out.println(main.sayHello(x->("start "+x+" end")));
    }
}

public class Main {
	//自动生成内部类
    final class Main$$Lambda$1 implements Function {
        private Main$$Lambda$1() {
        }
        public Object apply(Object var1) {
            return Main.lambda$main$0((String)var1); // 调用类的静态方法
        }
    }
    //自动生成静态方法
    private static java.lang.String lambda$main$0(java.lang.String var){
        //lambda表达式的逻辑
        return "start "+var+" end";
    }
    public String sayHello(Function<String,String> f){
        return f.apply("aa");
    }
    public static void main(String argv[]){
        Main main=new Main();
        System.out.println(main.sayHello(main.new Main$$Lambda$1()));
    }
}

```

**实例方法中使用lambda时**
```java
public class Main {
    public String sayHello(Function<String,String> f){
        return f.apply("aa");
    }
    public static void main(String argv[]){
        Main main=new Main();
        main.process();
    }
    //lambda表达式位于实例方法process里面
    public void process(){
        System.out.println(sayHello(x->{
            return "start "+x+" end";
        }));
    }
}

final class Main$$Lambda$1 implements Function {
    private final Main arg$1; // 有一个实例对象
    private Main$$Lambda$1(Main var1) {
        this.arg$1 = var1;
    }
    private static Function get$Lambda(Main var0) { // 新增的私有静态方法。
        return new Main$$Lambda$1(var0);
    }
    @Hidden
    public Object apply(Object var1) {
        return this.arg$1.lambda$process$0((String)var1); //调用实例的方法。而在静态方法中，是调用类的静态方法
    }
}

```

**两种方法中使用lambda的不同**

- 实例方法中用lambda，apply()方法里面，调用的是Main对象的实例方法，不再是静态方法；
- 实例方法中用lambda，新增了一个私有静态方法get$Lambda()；
- 实例方法中用lambda，新增了一个类为Main的私有属性，通过构造方法初始化该属性值。

> java中Integer的是属于什么类型的拷贝

浅拷贝
```java
public static void main(String[] args) {
    Integer a = new Integer(10);
    Integer b = a;
    Integer c= 10;
    int d = 10;
    System.out.println(a==b);//true
    System.out.println(b==c);//false 两个Integer比较，引用地址不同
    System.out.println(b==d);//true Integer会自动拆箱成int类型，因此10==10
}
```

> 基本数据类型为什么还有包装类

让基本数据类型拥有对象的特性，比如在哈希表插入的时候需要有hashcode，基本数据类型是没有的。所有都是相应的包装类提供的

> 基础类型的赋值是原子的吗，包装类呢？

在java中基本类型的大部分赋值操作是原子性的,但是long和double除外,因为jvm将long和double会产生字撕裂的情况,jvm将long和double读取和写入当作分离的两次32位操作来执行,这样多线程可能产生不一致的情况出现.解决办法就是加上volatile.

包装类如Integer是不原子的。

> java内部类是否会生产class文件

会。不论是内部类还是匿名内部类都会生产class文件。

```java
public class MyClass {

    private MyInterface myInterface = new MyInterface() {
        @Override
        public void onTaskClick() {

        }
    };

    public interface MyInterface {
        void onTaskClick();
    }
}
```
![](https://gitee.com/super-jimwang/img/raw/master/img/20210402134902.png)
```java
classMyClass$1

 implements MyClass.MyInterface

{

  MyClass$1(MyClass paramMyClass) {}

 public void onTaskClick() {}

}

```

> int类型的最大值是多少

2147483647。对应0x7fffffff。为什么是7？因为最高位用来表示正负号。

> CGLib动态代理的底层原理

- CGLib采用了非常底层的字节码技术，其原理是通过目标类的字节码为一个类创建子类，并在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑。
- 底层使用字节码处理框架ASM，来转换字节码并生成新的类。
- 更详细一点说，代理类将目标类作为自己的父类并为其中的每个非final委托方法创建两个方法：
  - 一个是与目标方法签名相同的方法，它在方法中会通过super调用目标方法；
  - 另一个是代理类独有的方法，称之为Callback回调方法，它会判断这个方法是否绑定了拦截器（实现了MethodInterceptor接口的对象），若存在则将调用intercept方法对目标方法进行代理，也就是在前后加上一些增强逻辑。intercept中就会调用上面介绍的签名相同的方法。==回调函数就是拦截器中要执行的函数==

# socket编程

客户端有socket，服务端有一个用于监听的socket和一个用于跟客户端通信的socket



> NIO编程的流程说一下

服务端：
- 获取ServerChannel对象，并设置非阻塞绑定监听端口
- 创建Selector对象（select或者poll）
- 通过ServerChannel在selector中注册感兴趣事件（先注册OP_ACCEPT），监听客户端连接服务器事件
- while循环，通过select开始监听，直到出现了感兴趣事件
- 获取selector的selectionkeys,存的就是需要处理的事件，就是连接请求
- 处理事件
  - 如果是Acceptable的，就是连接请求
    - ServerChannel accept表示等待客户端连接，如果是非阻塞会立即返回为null的socketChannel(用于与客户端通信)，并将selector注册到socketChannel上，感兴趣事件为OP_READ。开始监听客户端的写事件
  - 如果是Readable，写请求
    - 必须监听了OP_READ才行
    - 开辟buffer空间
    - channel读取buffer
- 处理完毕，从key中移除当前事件

