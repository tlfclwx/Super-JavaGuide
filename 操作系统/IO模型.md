# IO模型
## 同步异步、阻塞非阻塞的理解
阻塞： 发生动作的时候，用户线程不动，一直等待动作返回为止，才会继续运行。

非阻塞： 发生动作的时候，会立即空返回。同步非阻塞中用户线程多次轮训，发出相同请求，直到有返回值完成动作为止。

同步： 用户线程必须完成了请求的动作才会继续执行下一个指令

异步： 用户发起请求后，会继续执行其他指令，不会等待请求完成。异步的一种实现方式就是多线程

## 同步阻塞IO (blocking IO) 也叫做BIO

![image-20210202201654267](https://gitee.com/super-jimwang/img/raw/master/img/20210202201654.png)

用户线程通过系统调用read发起IO读操作，由用户空间转到内核空间。内核等到数据包到达后，然后将接收的数据拷贝到用户空间，完成read操作。

==**整个读取过程，用户线程是被阻塞的，导致在发起IO请求时，不能做任何事情，对CPU资源利用不够**==

## 同步非阻塞IO (Non-blocking IO) 也叫做NIO

同步非阻塞IO是在同步阻塞IO的基础上，将socket设置为NONBLOCK。这样做用户线程可以在发起IO请求后可以立即返回。

![image-20210202201839390](https://gitee.com/super-jimwang/img/raw/master/img/20210202201839.png)

由于socket是非阻塞的方式，因此用户线程发起IO请求时立即返回。但并未读取到任何数据，==**用户线程需要不断地发起IO请求，直到数据到达后，才真正读取到数据，继续执行。**==

==**不断的轮询，重复请求，会消耗大量的cpu资源**==

==**个人理解：一个NIO可以通过socket监视很多的用户，当一个发送请求了，就会一对一的非阻塞处理，轮询。**==


## IO多路复用

IO多路复用是一种同步IO模型，实现一个线程可以监控多个文件句柄。一旦某个文件句柄就绪，就能通知应用程序进行相应的IO操作。没有文件句柄时就会发生阻塞，交出cpu。多路指多个文件句柄，复用指同一个线程

一句话解释：单线程或单进程同时监控若干个文件描述符是否可以执行IO操作的能力



![image-20210202202204434](https://gitee.com/super-jimwang/img/raw/master/img/20210202202204.png)

IO多路复用模型是建立在内核提供的多路分离函数select基础之上的，使用select函数可以避免同步非阻塞IO模型中轮询等待的问题。

==**用户首先将需要进行IO操作的socket添加到select中，然后阻塞等待select系统调用返回。当数据到达时，socket被激活，select函数返回。用户线程正式发起read请求，读取数据并继续执行。**==

==但是好处在于一个线程可以设置多个监听socket，不断的调用select函数，从而激活不同的socket，达到在同一个线程内同时处理多个IO请求的目的==



### 使用Reactor设计模式的多路复用

![image-20210202202741071](https://gitee.com/super-jimwang/img/raw/master/img/20210202202741.png)

通过Reactor的方式，可以将用户线程轮询IO操作状态的工作统一交给handle_events事件循环进行处理。==**用户线程注册事件处理器之后可以继续执行做其他的工作（异步），而Reactor线程负责调用内核的select函数检查socket状态。当有socket被激活时，则通知相应的用户线程（或执行用户线程的回调函数），执行handle_event进行数据读取、处理的工作。**==

由于select是阻塞的，而用户线程异步，因此也称为**异步阻塞模型**

### select函数接口、使用示例
[视频](https://www.bilibili.com/video/BV1qJ411w7du?from=search&seid=12365697407458547972)
接口

```c
#include <sys/select.h>
#include <sys/time.h>

#define FD_SETSIZE 1024
#define NFDBITS (8 * sizeof(unsigned long))
#define __FDSET_LONGS (FD_SETSIZE/NFDBITS)

// 数据结构 (bitmap) 底层就是数据
typedef struct {
    unsigned long fds_bits[__FDSET_LONGS];
} fd_set;

// API
int select(
    int max_fd, 
    fd_set *readset,  // 读 写 异常三张bitmap
    fd_set *writeset, 
    fd_set *exceptset, 
    struct timeval *timeout
)                              // 返回值就绪描述符的数目

FD_ZERO(int fd, fd_set* fds)   // 清空集合
FD_SET(int fd, fd_set* fds)    // 将给定的描述符加入集合
FD_ISSET(int fd, fd_set* fds)  // 判断指定描述符是否在集合中 
FD_CLR(int fd, fd_set* fds)    // 将给定的描述符从文件中删除
```

示例

```c
int main() {
  /*
   * 这里进行一些初始化的设置，
   * 包括socket建立，地址的设置等,
   */

  fd_set read_fs, write_fs;
  struct timeval timeout;
  int max = 0;  // 用于记录最大的fd，在轮询中时刻更新即可

  // 初始化比特位
  FD_ZERO(&read_fs);
  FD_ZERO(&write_fs);

  int nfds = 0; // 记录就绪的事件，可以减少遍历的次数
  while (1) {
    // read_fd就是监视的文件描述符010011这就表示第二个第五第六个文件被监视
    // 1.阻塞获取 将所有的 fd_set 传入内核 让它帮我们监听是否有事件准备就绪
    // 2.每次需要把多有的fd_set从用户态拷贝到内核态  read_fd write_fd...
    // 3.nfds 表示内核返回给你有多少个 fd 准备就绪了，并将有数据的fd在bitmap中置位
    // 这里就是使用select 返回的是就绪描述符的数目。
    // select是阻塞函数，如果一直没有数据进来就会一直阻塞在这一行
    nfds = select(max + 1, &read_fd, &write_fd, NULL, &timeout); // 这里的max+1主要是为了告诉内核最大fd的，以便内核监听
    
    // 1.用户态的进程是不知道到底哪个 fd 准备就绪的，那么它就需要再遍历一遍之前需要遍历所有的 fd
    // 2.判断有无读写事件发生 看下哪几个数据准备就绪了，然后进行处理
    for (int i = 0; i <= max && nfds; ++i) {
      if (i == listenfd) {
         --nfds;
         // 这里处理accept事件

         FD_SET(i, &read_fd);//将客户端socket加入到集合中
      }
      if (FD_ISSET(i, &read_fd)) {
        --nfds;
        // 这里处理read事件
      }
      if (FD_ISSET(i, &write_fd)) {
         --nfds;
        // 这里处理write事件
      }
    }
  }
}
```

**缺点**

- 单个进程所打开的FD是有限制的，通过FD_SETSIZE设置，默认1024。==select中底层是一个long数组，因此数组是有长度限制的。==
- 每次调用select，都需要把fd_set（文件描述符标记的bitmap）从用户态拷贝到内核态，这个开销在fd很多时会很大。
- 另外数组不可重用。内核直接在fdset数组中标记，所以不可重用。
- 唤醒的时候由于进程不知道是哪个 fd 已经就绪，需要进行一次遍历，采用轮询的方法，效率较低（高并发时）`for (int i = 0; i <= max && nfds; ++i)`

### poll函数

功能与select类似. ==注意poll底层是链表，链表内是pollfd==，不是数组！！！

接口

```c
#include <poll.h>
// 数据结构
// 没有使用bitmap带来了一定好处
struct pollfd {
    int fd;                         // 需要监视的文件描述符
    short events;                   // 需要内核监视的事件
    short revents;                  // 实际发生的事件
};

// API
int poll(struct pollfd fds[], nfds_t nfds, int timeout);
```



示例

```c
// 先宏定义长度
#define MAX_POLLFD_LEN 4096  

int main() {
  /*
   * 在这里进行一些初始化的操作，
   * 比如初始化数据和socket等。
   */

  int nfds = 0;
  pollfd fds[MAX_POLLFD_LEN];
  memset(fds, 0, sizeof(fds));
  fds[0].fd = listenfd;
  fds[0].events = POLLRDNORM;
  int max  = 0;  // 队列的实际长度，是一个随时更新的，也可以自定义其他的
  int timeout = 0;

  int current_size = max;
  while (1) {
    // 1. 阻塞获取  每次需要把pollfd数组从用户态拷贝到内核态
    // 2. 当内核监听到有事件就绪的时候，会把pollfd结构体中的revents置成内核监听事件，并返回就绪数目
    nfds = poll(fds, max+1, timeout);
    if (fds[0].revents & POLLRDNORM) {
        // 这里处理accept事件
        connfd = accept(listenfd);
        //将新的描述符添加到读描述符集合中
    }
    // 每次需要遍历所有fd，判断有无读写事件发生
    for (int i = 1; i < max; ++i) {
      // 可以修改fds[i].fd 和 fds[i].revents = 0 实现复用
      if (fds[i].revents & POLLRDNORM) { 
         sockfd = fds[i].fd
         if ((n = read(sockfd, buf, MAXLINE)) <= 0) {
            // 这里处理read事件
            if (n == 0) {
                close(sockfd);
                fds[i].fd = -1;
            }
         } else {
             // 这里处理write事件     
         }
         if (--nfds <= 0) {
            break;       
         }
      }
    }
  }
}
```

如果有就绪的poll是会改变其结构体pollfd内的events置成内核监听事件

**优点**

- 相比select 文件描述符的数量已不再受限制。底层是链表
- 可以修改fds[i].fd文件描述符 和 fds[i].revents = 0 实现复用



**缺点**

- 每次调用poll，都需要把pollfd集合从用户态拷贝到内核态，这个开销在fd很多时会很大
- 对fd集合扫描时仍然是线性扫描，采用轮询的方法，效率较低（高并发时）`for (int i = 1; i < max; ++i)`

### epoll函数

接口

```c
#include <sys/epoll.h>

// 数据结构
// 每一个epoll对象都有一个独立的eventpoll结构体
// 用于存放通过epoll_ctl方法向epoll对象中添加进来的事件
// epoll_wait检查是否有事件发生时，只需要检查eventpoll对象中的rdlist双链表中是否有epitem元素即可
struct eventpoll {
    /*红黑树的根节点，这颗树中存储着所有添加到epoll中的需要监控的事件*/
    struct rb_root  rbr;
    /*双链表中则存放着将要通过epoll_wait返回给用户的满足条件的事件*/
    // 就绪队列
    struct list_head rdlist;
};

// API

// 内核中加一个 eventpoll 对象，把所有需要监听的 fd 都放到这个对象中
int epoll_create(int size); 
// epoll_ctl 负责把 fd 增加、删除到内核红黑树
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
// epoll_wait 负责检测红黑树，就绪则放到可读队列，没有可读 fd 则阻塞进程
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);

struct epoll_event {
    __uint32_t events; // 回调事件
    epoll_data_t data; // 存储fd等
};

```

示例

```c
int main(int argc, char* argv[])
{
   /*
   * 在这里进行一些初始化的操作，
   * 比如初始化数据和socket等。
   */

    // 内核中创建eventpoll对象
    epfd=epoll_create(256);
    // 需要监听的epfd 放到 eventpoll中
    epoll_ctl(epfd,EPOLL_CTL_ADD,listenfd,&ev);
 
    while(1) {
      // 阻塞获取 准备好了的 epfd。
      // 内核监测到就绪的事件之后，把时间放到rdlist
      nfds = epoll_wait(epfd,events,20,0);
      // 这里就直接对收到就绪的rdlist 进行遍历  不需要再遍历所有的 fd
      // 是怎么做到的呢，下面继续分析
      for(i=0;i<nfds;++i) {
          if(events[i].data.fd==listenfd) {
              // 这里处理accept事件
              connfd = accept(listenfd);
              // 接收新连接写到内核对象中
              epoll_ctl(epfd,EPOLL_CTL_ADD,connfd,&ev);
          } else if (events[i].events&EPOLLIN) {
              // 这里处理read事件
              read(sockfd, BUF, MAXLINE);
              //读完后准备写
              epoll_ctl(epfd,EPOLL_CTL_MOD,sockfd,&ev);
          } else if(events[i].events&EPOLLOUT) {
              // 这里处理write事件
              write(sockfd, BUF, n);
              //写完后准备读
              epoll_ctl(epfd,EPOLL_CTL_MOD,sockfd,&ev);
          }
      }
    }
    return 0;
}
```

epolln内部存了一个列表，所以不需要遍历所有的fd了，只需要遍历这个列表就行了` for(i=0;i<nfds;++i) `

## 异步IO

![image-20210202203301515](https://gitee.com/super-jimwang/img/raw/master/img/20210202203301.png)

异步IO模型中，用户线程直接使用内核提供的异步IO API发起read请求，且发起后立即返回，继续执行用户线程代码。不过此时用户线程已经将调用的AsynchronousOperation和CompletionHandler注册到内核，然后操作系统开启独立的内核线程去处理IO操作。当read请求的数据到达时，由内核负责读取socket中的数据，并写入用户指定的缓冲区中。最后内核将read的数据和用户线程注册的CompletionHandler分发给内部Proactor，Proactor将IO完成的信息通知给用户线程（一般通过调用用户线程注册的完成事件处理函数），完成异步IO。

==**在多路复用中，用户线程是收到Reactor的通知，再去拷贝数据，而在异步IO中，内核会直接读取socket中的数据，拷贝到指定缓冲区，再通知用户。**==



> epoll 水平触发和边缘触发的区别


水平触发
1. 对于读操作
只要缓冲内容不为空，LT模式返回读就绪。

2. 对于写操作
只要缓冲区还不满，LT模式会返回写就绪。

边缘触发
1. 对于读操作
（1）当缓冲区由不可读变为可读的时候，即缓冲区由空变为不空的时候。
（2）当有新数据到达时，即缓冲区中的待读数据变多的时候。

1. 对于写操作
（1）当缓冲区由不可写变为可写时。
（2）当有旧数据被发送走，即缓冲区中的内容变少的时候。


> IO多路复用用来解决什么问题

单线程就可以处理多个文件描述符的输入问题，而不需要每个文件描述符分配一个线程。

> 讲一下select的原理，缺点

有一个max最大值，然后用max开一个fd_set的long数组用来保存标记。通过select方法讲fd_set由用户态拷贝到内核态。阻塞。一旦由数据到了，select方法返回，并且会在fd_set中标记有数据的文件描述符。然后开始遍历fd_set，遇到了标记就处理。


缺点：
- 有max的最大监视限制，默认是1024
- 每次都需要把fd_set从用户态拷贝到内核态
- 每次都需要轮询fd_set 
- fd_set不可复用

> 讲一下poll的原理，缺点

有pollfd结构体，其中fd就是文件描述符，还有fd.events表示关心的事件还有一个revents，用于如果有数据就设置它（这是与select的不同，select中直接修改了原来的fd，导致不能重用）
poll阻塞，然后有数据来了，就返回，然后就是遍历pollfds链表，检查revents，如果有就读，并且把它重置为0。那么每次的pollfds都是一样的，就可以重复利用了

改进：
- 用pollfds链表，所以就等于没有max限制了
- 实现了重用

缺点：
- 还是需要每次都拷贝进内核态
- 还是需要线性扫描数组

> epoll的原理和改进

内部是红黑树

epoll_create: 在内核cache里建了个红黑树用于存储以后epoll_ctl传来的socket外，还会再建立一个rdllist双向链表，用于存储准备就绪的事件

epoll_ctl: 操作epfd树。注册 修改 删除。有一个epoll_event参数，里面有events，就是让内核检测的兴趣事件。`epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)`

![](https://gitee.com/super-jimwang/img/raw/master/img/20210309190917.png)
这个是通过epoll_create创建的根节点和epoll_ctl插入的一个节点。这个插入的节点就是结构体`epoll_event`

epoll_wait: 观察这个rdllist双向链表里有没有数据即可，`epoll_wait(int epfd, struct epoll_event* events, int maxevents, int timeout)`
这里的events是一个数组。把rdllist上的epoll_event一个一个copy到数组里。

红黑树上的节点，一旦IO就绪，就会调用回调函数，加入rdllist中，等待wait使用。

> epoll水平触发和边缘触发的应用场景

关于读事件，如果业务可以保证每次都可以读完，那就可以使用边缘触发，否则使用水平触发。

对于写事件，如果一次性可以写完那就可以使用水平触发，写完删除写事件就可以了；但是如果写的数据很大也不在意延迟，那么就可以使用边缘触发。

> select底层的数据结构是什么

==就是一个long数组，一个long是64位，每一位都代表了一个文件描述符==

```java

#define FD_SETSIZE 1024
#define NFDBITS (8 * sizeof(unsigned long))
#define __FDSET_LONGS (FD_SETSIZE/NFDBITS)

// 数据结构 (bitmap) 底层就是数据
typedef struct {
    unsigned long fds_bits[__FDSET_LONGS]; // long是8字节64为的，因此数组长度是1024/64=16
} fd_set;
```

> poll的底层数据结构，为什么他没有最大连接数限制？

它没有最大连接数的限制，原因是它是基于链表来存储的.
```java
struct pollfd {
    /* 想要监听的文件描述符 */
    int fd;
    /* 关心的events */
    short events;
    /* 就绪的events */
    short revents;
};

struct poll_list {
    struct poll_list *next;
    int len;
    struct pollfd entries[0];
};
```