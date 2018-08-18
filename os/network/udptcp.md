# UDP
## 面向无连接
类比于TCP的三次握手连接，UDP只需要将数据封装成UDP数据报，然后封装成IP数据报的方式进行传输。（那么，如果数据报过长，怎么办？UDP依赖于IP层提供的数据分片支持，将数据进行切片封装）

## UDP数据分片
UDP本身并不做分组，UDP数据报容易产生IP分组（IP数据包），分组依据依靠于不同两路的MTU，每个IP分组头部都相同，除了偏移量和标志位，而且UDP首部只在第一个数据分组中。这种分片方式是在传输过程的路由器中执行，对主机起始端不可见（由于IP数据分组，只要有一个分组错误，就重传整个IP数据包，而TCP数据分段可以只传中间错误的段，所以TCP尽量避免IP分片）。

> IP在分片前是数据报，在分片后则叫分组（实际传输的数据单元）。

## 差错检验
UDP的校验和是可选的，通过校验和可以判断数据包是否出现差错。（如果关闭，可以依赖于链路层校验，而由于路由器也会发生差错，这种问题不会被检查出来）而当UCP数据包产生差错时，就直接被丢弃了，IP层检测到错误之后，也会做这样的处理。

> 从这一点来看，可以看出UDP是不可靠的。

## 拥塞控制
UDP是不存在拥塞控制的，当Linux缓冲队列满时，抛弃数据报（可能不产生ICMP差错，因为可能丢失的是第一个IP分组，丢失源主机的地址和端口号）。UDP只是尽最大努力交付（不可靠），吞吐量不受拥挤控制算法的调节，只受应用软件生成数据的速率、传输带宽、源端和终端主机性能的限制。
> 第一次UDP传输之前，会发送ARP指令，如果因为IP分片，造成多个IP请求，会导致多个ARP指令。

## 总结
UDP不可靠的传输，一般是通过应用层（或者Socket底层实现）控制超时重传（定时器）机制。支持一对一、一对多、多对一和多对多的交互通信。

# TCP
## 连接建立与终止
- 三次握手的主要原因：
    - 防止过期的连接请求再次传递到服务端，当客户端连接服务端的请求SYN，由于网络原因导致滞留后，发送给服务端，而其实客户端因为超时重发机制，早已经重发SYN，但是此时服务端会认为这是客户端新连接的请求，则会发送SYN + ACK。如果是二次连接，当客户端接收到，认为无效请求，不予理睬；服务端则认为已经建立请求，开始发送数据。
   
- 四次挥手的主要原因：主动关闭的一方，等待被关闭的另一方发送剩余数据。

- TIME_WAIT时间原因：
    - 确保最后一个确认报文段能够到达。如果 B 没收到 A 发送来的确认报文段，那么就会重新发送连接释放请求报文段，A 等待一段时间就是为了处理这种情况的发生。
    - 等待一段时间是为了让本连接持续时间内所产生的所有报文段都从网络中消失，使得下一个新的连接不会出现旧的连接请求报文段。

> TIME_WAIT状态也称为2MSL等待状态，每个具体TCP实现，必须选择一个报文段中最大生成时间MSL（指报文在丢弃前在网络内最长的时间），这个时间内，可能存在一个未到达的报文，都被丢弃。

## 分段
TCP依赖于MSS进行分段，在SYN建立连接时加入的可选字段，通告MSS，MSS表示TCP传往另一端的最大数据块长度。一般情况，MSS只能代表链路起始的限制，如果中间某段链路减少，也会发生分片（只能通过MTU发现机制解决）。

## 差错检验
TCP头中自带16位校验和，能够检验TCP传输的数据是否正确。

### 超时-重传
超时重传机制，关键之处在于超时和重传的策略。（如何决定超时间隔和如何确定重传频率）
1. 重传定时器；
2. 坚持定时器(persist)；
3. 保活定时器（keep alive）；
4. 2MSL定时器(Time Wait)；

超时重传机制，最重要的在于过期时间的估计，过期时间过长，影响效率；过期时间过短，增加负担，所以应该是动态变化的。实现的方式（注意超时计算一般应用于第一次，之后，通过指数退避2^n的方式计算）如下：
- 加权移动平均，将两部分加权求和，一部分，来自于之前的RTT；另一部分，来自于刚测量得出的RTT；
- 基于均值和方差计算RTO。

> 具体RTT的估计，是每次发送数据的时候，启动一个滴答计时器（500ms/滴答），每次数据传输间隔包含的滴答数量，则为RTT的值，此外，当滴答计时器已经启动，则之后的TCP段不计算RTT。（超时重传时，不更新RTT，为解决重传的多义性）

## 拥塞控制
TCP采用滑动窗口协议进行流量控制，在发送端采用拥塞窗口cwnd（表示发送方感受到的网络阻塞的估计），在接收端采用通告窗口（表示接受缓冲区剩余的空间），最终发送的TCP段的数量是两者的最小值。

### 基于字节流的TCP
- Delayed ACK <br>
TCP给ACK发送统一安排计时器，如果需要发送ACK，则隔200ms发送一起，在200ms内，我发有数据需要发送，则用ACK捎带机制，将数据传输过去；无，则只将ACK信息发送。（如果给每一个包设定计时器，耗费过多资源）

- Nagle算法，一个TCP连接上最多只有一个未被确认的未完成的小分组，在该分组确认到达之前不能发送其他小分组。

### 基于成块数据的TCP
#### 滑动窗口协议
    - 窗口属于发送和接受的缓存一部分，用于暂存字节流。
    - 窗口的大小，根据接收端返回的ACK的序号作为起始值，窗口大小作为偏移量；
    - 当接收到ACK时，（发送）窗口左边沿会合拢（左边沿不会扩张，因为左移的ACK会被认为是重复并抛弃）；
    - 当（接受）窗口中的数据发送确认并被应用程序接受，右边沿会张开；
    - 发送窗口内的字节都允许被发送，接收窗口内的字节都允许被接收。如果发送窗口左部的字节已经发送并且收到了确认，那么就将发送窗口向右滑动一定距离，直到左部第一个字节不是已发送并且已确认的状态；接收窗口的滑动类似，接收窗口左部字节已经发送确认并交付主机，就向右滑动接收窗口。

> 滑动窗口协议的好处，可以一次性发多个数据段，不需要等待ACK，提高数据吞吐量，同时，根据窗口的调整，可以进行一定程度的拥塞控制，滑动窗口是根据字节序列进行排序的。

#### 拥塞控制算法
- 慢启动，拥塞窗口(cwnd)的大小初始化为1，之后随着接收到的ACK数量增加cwnd的值，呈指数级上升（cwnd表示当前能够一次发送的TCP段的数量）。

- 拥塞避免算法，用于处理分组丢失情况的方法。假设由分组损坏造成的丢失非常少，而分组丢失的表现：超时重传和重复确认。同时，需要为每个连接维护两个变量，一个是拥塞窗口(cwnd)和一个慢启动门限(sshtresh)。具体工作流程如下：
    - 初始化，cwnd = 1，sshtresh = 65536(门限设置最高)；
    - TCP输出不能超过cwnd和通告窗口大小；
    - 当拥塞发生（数据包丢失）时，sshtresh设置为当前窗口大小的一半（拥塞窗口和通告窗口的最小值，但最少2个报文段）。此外，如果是超时引起的，则将cwnd设置为1；
    - 当新数据被被确认时，增加cwnd，增加方式依赖于慢启动还是拥塞避免。如果cwnd <= ssthresh，则正在进行慢启动，反之进行拥塞避免。（sshthresh记录了原来cwnd一半的大小）；拥塞避免采用加性增加的方式增长，线性方式增加。
    
- 快速重传和快速恢复。建立在之前的基础上，接收到重复ACK原因有两个，一个，可能是报文段重排（后面的先到）；第二个，报文段丢失。那么，1-2个重复ACK，则认为报文段重排；而3个或3个以上，非常可能是报文段丢失，则会触发快速重传机制，无需等待计时器溢出。之后，触发拥塞避免的第三步，cwnd并不会设置成1，具体流程如下：
    - 接收到第三个重复ACK，则将sshtresh设置为cwnd的一半，重传丢失的报文段，将cwnd设置为sshtresh加上3倍报文段大小；
    - 之后，每次接受到另一个重复ACK时，cwnd增加1个报文段大小并发送1个分组（未确认的字节数 < cwnd新值）；
    - 当下一个确认新数据ACK到达时（包含所有未确认的段），设置cwnd为sshtres（当前速率减半）。

#### ICMP差错控制
- 收到ICMP源站抑制差错，cwnd设置为1个报文段，发起慢启动（慢启动门限不变）；
- 主机不可达/网络不可达，维持连接，记录出错信息，之后如果超时，则将出错信息打印出来；

> 注意cwnd和ssthresh都是通过字节为单位进行维护的。

# 两者比较
- 数据发生错误？ UDP直接根据可选的校验和进行判断（没有，则通过其他层数据校验），但是没有重传机制；TCP通过ACK指定某个报文段，来提示发送端出现问题的报文段，然后，发送端通过超时重传机制重发TCP段；
- 数据发送拥塞？ UDP如果在接收端的缓冲区已满的情况下，直接丢弃；TCP会根据发送端和接收端两侧都提供拥塞控制方法；



> 网络传输的次序，是big endian字节序。


# Reference
- [[通俗易懂]深入理解TCP协议](http://www.52im.net/thread-515-1-1.html)
- 《TCP/IP详解：卷1》