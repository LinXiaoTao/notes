[WIKI](https://zh.wikipedia.org/wiki/%E4%BC%A0%E8%BE%93%E6%8E%A7%E5%88%B6%E5%8D%8F%E8%AE%AE)

传输控制协议（Transmission Control Protocol）是一种面向连接的、可靠的、基于字节流的传输层通信协议，由 IETF 的 RFC 739 定义。

应用层向 TCP 层发送用于网间传输的、用 8 位字节表示的数据流，然后 TCP 把数据分区成适当的报文段（通常受到该计算机连接的网络的数据链路层的最大传输单元（MTU）的限制）。之后 TCP 把结果包传给 IP 层，由它来通过网络将包传送给接收端实体的 TCP 层。TCP 为了保证不发生丢包，就给每个包一个序号，同时序号也保证了传送到接收端实体的包的按序接收。然后接受端实体对已成功收到的包发送一个相应的确认（ACK）；如果发送端在合理的往返时延（RTT）内未收到确认，那么对应的数据包就被假设为已丢失，将会被进行重传。TCP 用一个校验和函数来检验数据是否有错误；在发送和接受时都要计算校验和。

TCP连接包括三个状态：连接创建、数据传送和连接终止。操作系统将TCP连接抽象为[套接字](https://zh.wikipedia.org/wiki/Berkeley%E5%A5%97%E6%8E%A5%E5%AD%97)的编程接口给程序使用，并且要经历一系列的[状态](https://zh.wikipedia.org/wiki/%E4%BC%A0%E8%BE%93%E6%8E%A7%E5%88%B6%E5%8D%8F%E8%AE%AE#.E7.8A.B6.E6.80.81.E7.BC.96.E7.A0.81)改变。

### 连接创建

TCP用三次握手（three-way handshake）过程创建一个连接。

一对终端同时初始化一个它们之间的连接是可能的。但通常是由一端打开一个[套接字](https://zh.wikipedia.org/wiki/Berkeley%E5%A5%97%E6%8E%A5%E5%AD%97)（[socket](https://zh.wikipedia.org/wiki/Socket)）然后监听来自另一方的连接，这就是通常所指的被动打开（passive open）。服务器端被被动打开以后，用户端就能开始创建主动打开（active open）。

1. 客户端通过向服务器端发送一个[SYN](https://zh.wikipedia.org/w/index.php?title=SYN&action=edit&redlink=1)来创建一个主动打开，作为三路[握手](https://zh.wikipedia.org/wiki/%E6%8F%A1%E6%89%8B_(%E6%8A%80%E6%9C%AF))的一部分。客户端把这段连接的序号设定为随机数**A**。
2. 服务器端应当为一个合法的SYN回送一个SYN/ACK。ACK的确认码应为**A+1**，SYN/ACK包本身又有一个随机序号**B**。
3. 最后，客户端再发送一个[ACK](https://zh.wikipedia.org/wiki/ACK)。当服务端受到这个ACK的时候，就完成了三路握手，并进入了连接创建状态。此时包序号被设定为收到的确认号**A+1**，而响应则为**B+1**。

![三次握手](https://upload.wikimedia.org/wikipedia/commons/thumb/3/3f/Connection_TCP.png/440px-Connection_TCP.png)

### 数据传输

在 TCP 的数据传送状态，很多重要的机制保证了 TCP 的可靠性和强壮性。它们包括：使用序号，对收到的 TCP 报文段进行排序以及检测重复的数据；使用校验和来检测报文段的错误；使用确认和计时器来检测和纠正丢包或延时。

#### 序列号和确认

在 TCP 的连接创建状态，两个主机的 TCP 层间要交换*初始序号*（ISN：initial sequence number）。

TCP 报文的接收者为了确保可靠性，在接收到一定数量的连续字节流后才发送确认。这是对 TCP 的一种扩展，通常称为选择确认（Selective Acknowledgement）。选择确认使得 TCP 接收者可以对乱序到达的数据块进行确认。每一个字节传输过后，ISN 号都会递增 1。

通过使用序号和确认号，TCP 层可以把收到的报文段中的字节按正确的顺序交付给应用层。序号是 32 位的无符号数，在它增大到232-1时，便会回绕到 0。对于 ISN 的选择是 TCP 中关键的一个操作，它可以确保强壮性和安全性。

