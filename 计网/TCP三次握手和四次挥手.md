# TCP三次握手和四次挥手

TCP是可靠的双向通道传输协议，所以需要三次握手建立连接和四次挥手关闭连接来保证信息传输的可靠性。



### 报文标识位

先来了解一下TCP在握手连接时候的报文用到的标志位。

- SYN：**建立连接消息的标志**
- ACK（Acknowledgment Number）：**应答消息的标志**
- FIN：连接关闭的标志



### 三次握手

![image-20210209213041707](https://gitee.com/super-jimwang/img/raw/master/img/20210209213041.png)

第一次握手：建立连接。客户端发送SYN连接请求报文段，将SYN位置为1，Sequence Number(seq)置为x；然后，客户端进入**SYN_SEND状态**，等待服务器的确认；



第二次握手： 服务器收到客户端的SYN报文段。服务器需要对这个SYN报文段进行确认，设置为ack=x+1(seq+1)；同时，自己还要发送SYN+ACK报文段，来应答建立连接请求，将SYN+ACK位置为1，seq置为y；服务器端将上述所有信息放到一个报文段（即SYN+ACK报文段）中，一并发送给客户端，此时服务器进入**SYN_RECV状态**；



第三次握手： 客户端收到服务器的SYN+ACK报文段。然后设置ack=seq+1，向服务器发送ACK报文段，这个报文段发送完毕以后，客户端和服务器端都进入ESTABLISHED状态，完成TCP三次握手。



ack总是回复信息的seq+1. 如果是回复的信息，ACK就是1，标示位表示是回复。



### 为什么要三次握手

为了防止已失效的连接请求报文段突然又传送到了服务端，因而产生错误。如果只有两次握手，client发送连接请求后不再ACK服务端的SYN。此时client由于自身的原因判断建立连接失败，可能会重复建立TCP连接，而服务端却会认为那些被client丢弃的TCP连接依然有效，会浪费服务端资源。

举例：**比如假设只有两次握手，客户端向服务端发起了连接请求，但是由于网络延迟，被滞留了，很久以后才到达了服务端，服务端因为是新的链接请求，就返回确认报文，并由于只有两次握手，服务端开始等待客户端的数据，因此浪费了资源**



## 四次挥手过程

![image-20210209213541539](https://gitee.com/super-jimwang/img/raw/master/img/20210209213541.png)

第一次挥手： client发送关闭连接请求。向server发送一个FIN报文段，设置Sequence Number；此时，client进入FIN_WAIT_1状态；这表示client没有数据要发送给server了；



第二次挥手： server收到了client发送的FIN报文段，向client返回ACK报文段, 设置ack=seq+1，seq=v；client进入FIN_WAIT_2状态；server告诉client，我“同意”你的关闭请求，但是你要等一下，我还有东西要给你；



第三次挥手： server向client发送FIN+ACK报文段，设置ack=seq+1，seq=w；确认没有数据要传输了，可以进行关闭连接，同时server进入LAST_ACK状态，等待client确认收到最后的数据；==**注意，这里的ack=seq+1，seq是客户端发送的第一次挥手**==



第四次挥手： client收到server发送的FIN报文段，向server发送ACK报文段，然后client进入TIME_WAIT状态；server收到主机1的ACK报文段以后，就关闭连接；==**此时，client等待2MSL后依然没有收到回复，则证明Server端已正常关闭，那好，client也可以关闭连接了。**== 是server先关闭，后才是client关闭



### 为什么需要四次挥手

TCP是全双工模式，这就意味着，当主机1发出FIN报文段时，只是表示主机1已经没有数据要发送了，主机1告诉主机2，它的数据已经全部发送完毕了；但是，这个时候主机1还是可以接受着来自主机2的数据；当主机2返回ACK报文段时，表示它表示已经知道主机1没有数据发送了，但主机2还是有数据要发送数据到主机1的；当主机2也发送了FIN报文段时，这个时候就表示主机2也没有数据要发送了，就会告诉主机1，我也没有数据要发送了，之后彼此就会愉快的中断这次TCP连接。

### TIME_WAIT和CLOSE_WAIT的区别

CLOSE_WAIT(服务端)是被动关闭形成的；当对方close socket而发送FIN报文过来时，回应ACK之后进入CLOSE_WAIT状态。随后检查是否存在未传输数据，如果没有则发起第三次挥手，发送FIN报文给对方，进入LAST_ACK状态并等待对方ACK报文到来

TIME_WAIT(客户端)是主动关闭连接方式形成的；处于FIN_WAIT_2状态时，收到对方FIN报文后进入TIME_WAIT状态；之后再等待两个MSL(Maximum Segment Lifetime:报文最大生存时间)



### 为什么需要等待2MSL后，client才关闭

MSL(Maximum Segment Lifetime)：报文段最大生存时间，它是任何报文段被丢弃前在网络内的最长时间。

原因有二：

- 保证TCP协议的全双工连接能够可靠关闭
- 保证这次连接的重复数据段从网络中消失

==**第一点：由于网络的不稳定，可能client最后回复的fin报文没有被server正常接收到，那么这个时间最大为MSL。当没接受时，server会再次发送FIN，那么这个时间最大也为MSL。因此要等2MSL后才可以关闭。**==

==**第二点：确保这次链接中的数据从网络消失。如果client直接close，后又向server发起了新的链接，那么可能会时相同端口号的，可能会收到上一次链接中的数据。**==



## SYN Flood

SYN Flood 伪造 SYN 报文向服务器发起连接，服务器在收到报文后用 SYN_ACK 应答，此应答发出去后，不会收到 ACK 报文，造成一个半连接

若攻击者发送大量这样的报文，会在被攻击主机上出现大量的半连接，耗尽其资源，使正常的用户无法访问，直到半连接超时

> TCP是如何保证可靠传输

1. 应用数据被分割成TCP认为最合适发送的数据块
2. TCP给发送的每一个包进行编号，接收方对数据包进行排序再给应用层
3. 校验和，对验证有错的丢弃，并不确认收到此报文
4. TCP的接收端会丢弃重复的数据
5. 流量控制：使用滑动窗口，提示发送方降低发送的速率
6. 拥塞控制
7. ARQ协议，每发完一个分组就停止发送，等待对方确认。收到确认后再发下一组
8. 超时重传：超时还未确认的，就会在发送一次。

> 什么是ARQ协议

Automatic Repeat-reQuest。自动重传请求
它又分为停止等待ARQ协议和连续ARQ协议。

> 停止等待ARQ协议原理说一下

每发完一个分组就停止发送，等待对方确认ACK。然后再发送

> 连续ARQ协议原理说一下

发送方维持一个发送窗口，凡位于发送窗口内的分组可以连续发送出去，而不需要等待对方确认。接收方采用累计确认，对按序到达的最后一个分组发送确认，表明到达这个分组为止的所有分组都已经确认收到了。

> time_wait状态只会出现在客户端吗

服务端也会，如果服务端是主动关闭的一方。

> 服务端主动关闭会有什么问题？如何解决

因为time_wait等2MSL，客户端的一个连接对应服务端的一个fd，而fd是有限的，因此如果有大量的time_wait浪费了fd，新来的连接就无法连接了。

服务端为了解决这个TIME_WAIT问题，可选的方式有3种：
（1）保证由客户端主动发起关闭

（2）关闭的时候使用RST方式

（3）对处于TIME_WAIT状态的TPC允许重用