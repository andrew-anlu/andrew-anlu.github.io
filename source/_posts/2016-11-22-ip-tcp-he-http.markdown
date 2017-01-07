---
layout: post
title: "IP  TCP 和HTTP"
date: 2016-11-22 10:04:46 +0800
comments: true
categories: iOS
---


当app和服务器进行通信的额时候，大多数情况下，都是采用http协议。http最初是为web浏览器而定制的，如果在浏览器输入 `http://www.baidu.com`。浏览器就会通过http协议和baidu所对应的服务器进行通信。
<!--more-->

HTTP是运行在应用层上的应用协议，而不同的层级上都有相应的协议在运行。层级的堆栈关系一般可以这么描述：

```
Application Layer -- e.g. HTTP
----
Transport Layer -- e.g. TCP
----
Internet Layer -- e.g. IP
----
Link Layer -- e.g. IEEE 802.2
```

所谓的[OSI(Open Systems Interconnnection),开放式系统互联](https://en.wikipedia.org/wiki/OSI_model)模型定义了七层结构。本文会关注应用层(application layer)传输层(transport layer)和网络层(internet layer),他们分别代表了典型的HTTP的应用的HTTP,TCP,以及IP.在ip之下的是数据连接和物理层级，比如像Ethernet的实现之类的东西

如上文所述，我们只关注应用层，传输层和网络层部分，更确切的说，着重探讨一种特殊的混合模式：基于IP的TCP,以及基于TCP实现的HTTP.这就是我们每天使用的app的基本网络配置。

通过本文，希望大家能够对HTTP工作原理有一个细致的了解，知道一些常见的HTTP问题产生的原因，从而能在实践中尽量避免这些问题的发生。

其实在互联网上传递数据的方式并不只是HTTP一种，HTTP之所以被广泛使用的原因是其非常稳定，易用，即便是防火墙一般也是允许HTTP协议穿透的。

接下来，我们从最低的一层谈起，说说IP网络协议。



### IP网络协议(IP_Internet Protocol)

TCP/IP 中的Ip是[网络协议(Internet Protocol)](https://en.wikipedia.org/wiki/Internet_Protocol)的缩写。从字面意思便知，它是互联网众多协议的基础。

IP实现了分组交换网络，在协议下，机器被叫做主机(host)，IP协议明确了host之间的资料包（数据包）的传输方式。

所谓数据包是指一段二进制的数据，其中包含了发送源主机和目标主机的信息。IP网络负责源主机和目标主机之间的数据包传输。IP协议的特点是best effort(尽力服务，其目标是提供有效服务并尽力传输)。这意味着，在传输过程中，数据包可能会丢失，也有可能被重复传送导致目标主机收到多个同样的数据包

IP网络中的主机都配有自己的地址，被称为IP地址。每个数据包中都包含了源主机和目标主机的IP地址，IP协议负责路径计算，即IP数据包在网络中的传输时，数据包所经过的灭一个主机几点都会读取数据包中的目标主机地址信息，以便选择朝着什么地方传输数据包


今天，绝大多数的数据包仍旧是IPv4的，每一个IPV4的地址的长度是32位的，常见采用dotted-decimal(点分十进制)表示法，具体形式如下：198.51.100.42

新的IPv6标准也正在逐渐推广中。它有更大的地址空间：长度为128位，这使得数据包在网络中传输中传输时的寻址更容易一些。另外，由于有更多的地址可以分配，注入网络地址转换等问题也迎刃而解。IPv6的表示形式为:八组十六进制数以冒号分割，比如：2001:0db8:85a3:0042:1000:8a2e:0370:7334。

## IP Header
一个ip数据包通常包含header（报头信息）和payload（有效载荷）

payload中的内容即是要传输的真正信息，而header承载的是与传输数据有关的元数据(metadata)



### IPV4 Header

ipv4的header信息内容如下：

```
IPv4 Header Format
Offsets  Octet    0                       1                       2                       3
Octet    Bit      0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31|
 0         0     |Version    |IHL        |DSCP            |ECN  |Total Length                                   |
 4        32     |Identification                                |Flags   |Fragment Offset                       |
 8        64     |Time To Live           |Protocol              |Header Checksum                                |
12        96     |Source IP Address                                                                             |
16       128     |Destination IP Address                                                                        |
20       160     |Options (if IHL > 5)                                                                          |
```

header长度为20字节（不包含极少用到的可选信息）

header信息中最关键的是源和目标IP地址，除此之外，版本信息是4，代表ipv4.protocol(协议区)代表payload采用的传输协议。TCP的协议号是6.Total Length(总长度区)表明了header加payload整个数据包的大小

### Ipv6 Header

ipv6的地址长度为128位，IPv6的header信息内容如下：

```
Offsets  Octet    0                       1                       2                       3
Octet    Bit      0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31|
 0         0     |Version    |Traffic Class         |Flow Label                                                 |
 4        32     |Payload Length                                |Next Header            |Hop Limit              |
 8        64     |Source Address                                                                                |
12        96     |                                                                                              |
16       128     |                                                                                              |
20       160     |                                                                                              |
24       192     |Destination Address                                                                           |
28       224     |                                                                                              |
32       256     |                                                                                              |
36       288     |                                                                                              |
```


Ipv6 header采用固定长度40字节。经过多年来对Ipv4使用的总结，如今Ipv6的header信息简化了很多。


除了源和目标地址这种必备信息外，IPv6提供专门的next header区域来指明紧跟header的数据是什么。也就是说，Ipv6允许在数据包中将header链接起来，每一个被链接的Ipv6 header 都会有一个next header 字段，直到到达实际的payload数据，比如说，当next header的值为6(TCP的协议号)时，数据包的其它信息就是tcp协议要传输的数据

## Fragmentation(数据分片)
由于底部链路层对所传输的数据帧有最大长度限制，所以有时候IPV4需要对所传数据包进行分片。具体表现为，如果数据包尺寸超过了索要经过的数据链路层的最大传输限制，路由就会对数据包进行分片，当分片数据到达目标主机后，可以根据分片信息进行数据重组。当然，数据发送源有权决定路由是否启用对传输数据包进行分片，加入所传输的数据超过了输送限制，又禁止了路由分片，发送源会收到[ICMP](https://en.wikipedia.org/wiki/Internet_Control_Message_Protocol)的数据帧超长报告信息

在Ipv6中，如果数据包超限制，路由会直接丢弃数据包并且向发送源回传ICMP6的数据帧超长报告信息。源和目标两端会基于这个特性进行[路径MTU发现](https://en.wikipedia.org/wiki/Path_MTU)，以此寻找两端之间最大传输单元所在的路由，找到MTU路由后，仅当上层数据包的最小payload确实超过了MTU,IPV6才会进行分片传输，对于IPV6下的tCp来说，这不会造成什么问题。

## TCP-传输控制协议(Transmission Control Protocol)
TCP层位于IP层之上，是最受欢迎的因特网通讯协议之一，人们通常用TCP/IP来泛指整个因特网协议族。

刚刚提到，IP协议允许两个主机之间传送单一的数据包，为了保证对所传送数据包达到尽力服务的目的，最终的传输结果可能是数据包乱序，重复甚至丢包。

TCP是基于IP 层的协议。但是TCP是可靠地，有序的，有错误检查机制的基于字节流传输的协议。这样当两个设备上的应用通过TCP来传递数据的时候，总能够保证目标接收方收到的数据的顺序和内容与发送方所发出的是一致的。TCP做的这些事看起来很平常，但是比起IP层的粗犷处理方式已经是显著的进步了。

应用程序之间可以通过TCP建立链接，TCP建立的是双向连接，通信双方可以同时进行数据的传输。连接的双方都不需要操心数据是否分块，或者是否采用了尽力服务等。TCP会确保所传输的数据的正确性，即接受方收到的数据与发出方的数据一致。

HTTP是典型的TCP应用，用户浏览器(应用1)与web服务器(应用2)建立连接后，浏览器可以通过连接发送服务请求，web服务器可以通过同样的连接请求作出响应。

同一个host主机上可以有多个应用同时使用TCP协议。TCP用不同的端口来区分应用，作为连接的两端，发送源和接收目标分别拥有自己的IP地址和端口号，凭借这样一对IP地址和端口号，就可以唯一标识一个连接。

使用HTTPS的web服务器会监听443端口，浏览器作为发送源会启用一个临时端口结合自己的IP地址与目标服务器对应的端口和IP地址建立TCP连接。

TCP在IPv4和Ipv6上是无差别运行的。所以如果ipv4和protocol活IPv6的Next Header的协议号被设置为6，表示执行TCP协议。

## TCP Segments (TCP 报文段)
主机之间传输的数据流一般会先被分块，再转为为TCP报文段，最终会生成IP数据包中的payload的载荷数据


每个TCP报文段都有header信息和对应的载荷payload.payload信息就是待传输的数据块。TCP报文段的header信息中主要包含的是源和目标端口号，至于说源和目标的IP地址信息则已经包含在IP header信息中了。

TCP的报文段header信息中还有报文序列号，确认号等其他一些用于管理连接的信息。

所谓序列号信息，其实就是为每个报文段分配的唯一编号，第一个报文段的序列号是随机的，比如1721092979，其后的每一个报文段的序列号都以此号为基础以此加1，1721092980，1721092981 等等。至于确认号，是目标端反馈给源的确认信息，通知源目前已经接到那些报文段了。由于TCP是双向的，所以数据和确认信息发送也都是双向的。

## TCP连接

连接管理是TCP的核心功能之一，而且协议需要解决由于IP层采用不可靠传输引发的一系列复杂问题。下面会分别介绍TCP的连接简历，数据传输以及连接终止的详细过程。

TCP连接全过程的状态变化是复杂的，但是大多数情况下还是比较简单的。


### 连接建立

TCP连接都是简历在两个主机之间的。所以，每个连接建立过程中都存在两个角色：一端（例如web服务器）监听连接，另一端(例如应用)主动连接正在监听的一端(web服务器).服务器端的这种监听行为被称为passive open(被动打开)。客户端主动连接服务器的行为称为active open(主动打开)

TCP会通过三次握手来完成连接建立，具体过程是这样的：

1. 客户端首先向服务器端发送一个SYn包和一个随机序列号A
2. 服务器端收到回回复客户端一个SYn_ACK包以及各一个确认号（用于确认收到SYN）A+1,同时再发送一个随机序列号B
3. 客户端收到后会发送一个ACK包以及确认号（用于确认收到SYN-ACK）B+1和序列号A+1给服务端

SYN是Synchronize sequence numbers（同步序列号）的缩写。两端在传递数据时，所传递的每个TCP报文段都有一个序列号。就是利用这种机制，TCP可以确保分块传输的数据包最终都已正确的个数和顺序抵达目标端。在正式传输开始之前，源和目标端需要同步确认第一个报文的序列号。

ACK 是acknowledgment(确认)的缩写。当某一端接到了报文包后，通过回传已报文序列号来确认接收到报文这件事。

运行如下语句：

```
curl -4 http://www.apple.com/contact/
```

这是通过`curl`命令与www.apple.com 的 80端口创建一个TCPl连接

www.apple.com所在服务器23.63.125.15（注意，整个 IP 不是固定的）会监听80端口。我们自己的IP地址是`10.0.1.6`，启用的临时端口52181(这个端口是从可用端口中随机选择的)。利用tcpdump(1)输出的三次握手过程是这样的、

```
% sudo tcpdump -c 3 -i en3 -nS host 23.63.125.15
18:31:29.140787 IP 10.0.1.6.52181 > 23.63.125.15.80: Flags [S], seq 1721092979, win 65535, options [mss 1460,nop,wscale 4,nop,nop,TS val 743929763 ecr 0,sackOK,eol], length 0
18:31:29.150866 IP 23.63.125.15.80 > 10.0.1.6.52181: Flags [S.], seq 673593777, ack 1721092980, win 14480, options [mss 1460,sackOK,TS val 1433256622 ecr 743929763,nop,wscale 1], length 0
18:31:29.150908 IP 10.0.1.6.52181 > 23.63.125.15.80: Flags [.], ack 673593778, win 8235, options [nop,nop,TS val 743929773 ecr 1433256622], length 0
```

这里的信息量很大，下面要逐个分析一下：

最左边的是系统时间。当执行命令的时间是晚上18:31。后面的IP代表的是这些都是IP协议数据包。

接下来看这段10.0.1.6.52181 > 23.63.125.15.80，这一对是源和目标端的IP地址+端口。第一行和第三行是客户端发向服务端的信息，第二行是服务端发向客户端的。tcpdump会自动把端口号加到IP地址后头，比如 10.0.1.6.52181 表示 IP 地址为 10.0.1.6，端口号为 52181。

`Flags `表示TCP报文段header信息中的一些缩写标识：`S`代表SYN,`.`代表ACK,`P`代表PUSH,F是FIN.还有一些其他的标识，这边就不罗列了。注意上面三行Flags中先是携带SYN,接着是SYN-ACK,最后是ACK,这就是三次握手确认的全过程。

另外，第一行中客户端发送了一个随机序列号1721092979（就是上文中所说的A）给服务器，第二行展示的是服务器回传给客户端的确认号`1721092980 (A+1)`和一个随机序列号 673593777 (B).最后在第三行，客户端将自己的确认号 673593778 (B+1) 发送给服务端.

## 其他选项

当然，在连接建立过程中还会配置一些其他信息。比如第一行中客户端发送的内容:

```
[mss 1460,nop,wscale 4,nop,nop,TS val 743929763 ecr 0,sackOK,eol]
```

还有第二行服务端发送的：

```
[mss 1460,sackOK,TS val 1433256622 ecr 743929763,nop,wscale 1]
```

其中`TS val / ecr`是TCP用来创建RTT 往返时间(round-trip time)的。TS val 是发送方的时间戳，ECR 是相应应答时间戳，通常情况下就是发送方收到的最后时间戳。TCP以RTT作为其拥塞控制算法 (congestion-control algorithms) 的依据

连接的两端都发送 sackOK.这样会启用选择性确认(Selective Acknowledagement)机制，使连接双方能够确认收到的字节返回。一般情况下，确认机制只是确认接受方已收到的数据的字节总数。


`mss`选项声明了最大报文长度，表示接收端希望接收的单个报文的最大长度。wscale是窗口放大因子。


## 数据传输

一旦建立了连接，双方就可以互发数据了。发送端所发出的每个报文段都有一个序列号，这个序列号与当下已传送的字节总数有关。接收端会针对已接收的数据包向源端发送确认报文，确认信息同样是由报文header所携带的ACK

假设现在传送的信息是除最后一个报文5字节外，其它都是10字节。具体是这样的：


```
host A sends segment with seq 10
host A sends segment with seq 20
host A sends segment with seq 30    host B sends segment with ack 10
host A sends segment with seq 35    host B sends segment with ack 20
                                    host B sends segment with ack 30
                                    host B sends segment with ack 35
```

整个机制是双向运转的，A主机会持续的发送数据包。B收到数据包后会向A发送确认信息。A发送数据包的过程不需要等待B的确认。


TCP将流量控制和其它一系列复杂机制结合起来进行拥塞控制。需要处理以下问题：针对丢失的报文采用重发机制，同时还需要动态的调整发送报文的频率。

流量控制的原则是发送方发送数据的速度不能比接收方处理数据的速度快。接收方，也就是所谓的接收窗口会告知发送方自身接收窗口数据缓冲区的大小。从上面tcpdump的输出看，窗口大小是 `win 65535 `,`wscale`（窗口放大因子）是4.这些数字的意思是说：`10.0.1.6`主机的接收窗口大小是4＊64 kB = 256 kB,`23.63.125.15`主机的`win`是`14480 `,`wscale `是1，接收窗口约为14kb.总之，不管哪一方作为数据接收方，都会向对方通报自己的接收窗口的大小。


拥塞控制要更负责一些，所有拥塞控制的目标都是要计算出当前网络中数据传输的最佳速率。所谓最佳速率就是要达到一种微妙的平衡。一方面，是希望速度越快越好，另一方面，速度快意味着数据传输多，这样处理性能会打打打折甚至崩溃。而这种超负荷崩溃是分组交换网络的固有特点。当负载过大，数据包之间会产生拥塞，直接导致丢包率急速上升。

拥塞控制还需要充分考虑对流量的影响。发送方要时刻关注来自接收方的确认信息。要做到这点并不简单，有的时候还需要一定的妥协。要知道底部IP协议数据包是无序传输的，数据包会丢失也会重复。发送方需要评估RTT往返时间，然后基于RTT去确定是否收到了接收方的确认信息。重发数据包也有很大代价，除了连接延迟问题，网络的负载也会发生明显的波动。导致TCP需要不停的去适应当前网络情况。

更重要的是，TCP连接本身是易变的。除了数据传输，连接的两端还会不时的发送一些提醒和确认信息以便可以适当的调整状态来维持连接。


基于这种一直在相互协调中的连接关系，TCP连接往往会是短暂而低效的。在简历连接的初期，TCP协议算法还不能完全了解当前网络状况。而在连接将要结束的时候，反馈给发送方的信息又可能不充分，这样就很难对连接状况作出实时的合理的评估。

之前展示了客户单和服务端之间交换的三段报文，再看看关于连接的其它信息：

```
18:31:29.150955 IP 10.0.1.6.52181 > 23.63.125.15.80: Flags [P.], seq 1721092980:1721093065, ack 673593778, win 8235, options [nop,nop,TS val 743929773 ecr 1433256622], length 85
18:31:29.161213 IP 23.63.125.15.80 > 10.0.1.6.52181: Flags [.], ack 1721093065, win 7240, options [nop,nop,TS val 1433256633 ecr 743929773], length 0
```


客户端`10.0.1.6`发送的第一段报文长度是85 bytes(http请求)。由于在上一个报文发送后没有收到来自服务端的信息，所以ACK确认号值不变。

服务端 `23.63.125.15`只是对接收客户端的数据进行确认回复，没有向客户端发送数据，所以length为0，由于当前连接是采用选择性确认，所以序列号和确认好之间的字节长度从1721092980到1721093065，也就是85 bytes.接收方发送的ACK确认号是1721093065,这代表目前已接收的数据确认累计到1721093065字节了。至于说为什么数字会如此之大，这要说道初次握手时发出的随机数，数字的范围和那个初始数字是相关的。


这种模式会一直持续到全部数据传送完成：

```
18:31:29.189335 IP 23.63.125.15.80 > 10.0.1.6.52181: Flags [.], seq 673593778:673595226, ack 1721093065, win 7240, options [nop,nop,TS val 1433256660 ecr 743929773], length 1448
18:31:29.190280 IP 23.63.125.15.80 > 10.0.1.6.52181: Flags [.], seq 673595226:673596674, ack 1721093065, win 7240, options [nop,nop,TS val 1433256660 ecr 743929773], length 1448
18:31:29.190350 IP 10.0.1.6.52181 > 23.63.125.15.80: Flags [.], ack 673596674, win 8101, options [nop,nop,TS val 743929811 ecr 1433256660], length 0
18:31:29.190597 IP 23.63.125.15.80 > 10.0.1.6.52181: Flags [.], seq 673596674:673598122, ack 1721093065, win 7240, options [nop,nop,TS val 1433256660 ecr 743929773], length 1448
18:31:29.190601 IP 23.63.125.15.80 > 10.0.1.6.52181: Flags [.], seq 673598122:673599570, ack 1721093065, win 7240, options [nop,nop,TS val 1433256660 ecr 743929773], length 1448
18:31:29.190614 IP 23.63.125.15.80 > 10.0.1.6.52181: Flags [.], seq 673599570:673601018, ack 1721093065, win 7240, options [nop,nop,TS val 1433256660 ecr 743929773], length 1448
18:31:29.190616 IP 23.63.125.15.80 > 10.0.1.6.52181: Flags [.], seq 673601018:673602466, ack 1721093065, win 7240, options [nop,nop,TS val 1433256660 ecr 743929773], length 1448
18:31:29.190617 IP 23.63.125.15.80 > 10.0.1.6.52181: Flags [.], seq 673602466:673603914, ack 1721093065, win 7240, options [nop,nop,TS val 1433256660 ecr 743929773], length 1448
18:31:29.190619 IP 23.63.125.15.80 > 10.0.1.6.52181: Flags [.], seq 673603914:673605362, ack 1721093065, win 7240, options [nop,nop,TS val 1433256660 ecr 743929773], length 1448
18:31:29.190621 IP 23.63.125.15.80 > 10.0.1.6.52181: Flags [.], seq 673605362:673606810, ack 1721093065, win 7240, options [nop,nop,TS val 1433256660 ecr 743929773], length 1448
18:31:29.190679 IP 10.0.1.6.52181 > 23.63.125.15.80: Flags [.], ack 673599570, win 8011, options [nop,nop,TS val 743929812 ecr 1433256660], length 0
18:31:29.190683 IP 10.0.1.6.52181 > 23.63.125.15.80: Flags [.], ack 673602466, win 7830, options [nop,nop,TS val 743929812 ecr 1433256660], length 0
18:31:29.190688 IP 10.0.1.6.52181 > 23.63.125.15.80: Flags [.], ack 673605362, win 7830, options [nop,nop,TS val 743929812 ecr 1433256660], length 0
18:31:29.190703 IP 10.0.1.6.52181 > 23.63.125.15.80: Flags [.], ack 673605362, win 8192, options [nop,nop,TS val 743929812 ecr 1433256660], length 0
18:31:29.190743 IP 10.0.1.6.52181 > 23.63.125.15.80: Flags [.], ack 673606810, win 8192, options [nop,nop,TS val 743929812 ecr 1433256660], length 0
18:31:29.190870 IP 23.63.125.15.80 > 10.0.1.6.52181: Flags [.], seq 673606810:673608258, ack 1721093065, win 7240, options [nop,nop,TS val 1433256660 ecr 743929773], length 1448
18:31:29.198582 IP 23.63.125.15.80 > 10.0.1.6.52181: Flags [P.], seq 673608258:673608401, ack 1721093065, win 7240, options [nop,nop,TS val 1433256670 ecr 743929811], length 143
18:31:29.198672 IP 10.0.1.6.52181 > 23.63.125.15.80: Flags [.], ack 673608401, win 8183, options [nop,nop,TS val 743929819 ecr 1433256660], length 
```

## 终止连接

最终连接会终止(或结束)，连接的每一端都会发送FIN标识给另一端来声明结束传输，接着另一端会对收到的FIN进行通知，当连接两端均发送完各自FIN和做出相应的确认后，连接将会彻底关闭：

```
18:31:29.199029 IP 10.0.1.6.52181 > 23.63.125.15.80: Flags [F.], seq 1721093065, ack 673608401, win 8192, options [nop,nop,TS val 743929819 ecr 1433256660], length 0
18:31:29.208416 IP 23.63.125.15.80 > 10.0.1.6.52181: Flags [F.], seq 673608401, ack 1721093066, win 7240, options [nop,nop,TS val 1433256680 ecr 743929819], length 0
18:31:29.208493 IP 10.0.1.6.52181 > 23.63.125.15.80: Flags [.], ack 673608402, win 8192, options [nop,nop,TS val 743929828 ecr 1433256680], length 0
```

这里值得注意的是第二行，`23.63.125.15`发送了FIN，同时在这个报文信息中还对第一行中的另一端发送的FIN予以ACK(以.代表)确认。


## HTTP-超文本传输协议（Hypertext Transfer Protocol）

1989年，Tim Berners lee 在[CERN](https://en.wikipedia.org/wiki/CERN)担任软件咨询师的时候，开发了一套程序，奠定了万维网的基础。HyperText Transform Protocol（超文本转移协议，即HTTP）,是用于从WWW服务器传输超文件到本地浏览器的传送协议。[RFC 2616](https://tools.ietf.org/html/rfc2616) 定义了今天普通使用的一个版本:HTTP 1.1

### 请求与响应

HTTP采用简单的请求和响应机制，在safari输入` http://www.apple.com `时，会向www.apple.com所在的服务器发送一个http请求。服务器会在对请求作出一个响应，将请求结果信息返回给Safari.


每一个请求都有一个对应的响应信息。请求和响应遵从同样的格式，第一行是请求行或者响应状态行，接下来是header信息，header信息之后会有一个空行。空行之后是body请求信息体

### 一个简答请求
当safari加载html页面[http://www.objc.io/about.html](http://www.objc.io/about.html)的时候，先是发送HTTP请求到`www.objc.io`,请求的内容是:

```
GET /about.html HTTP/1.1
Host: www.objc.io
Accept-Encoding: gzip, deflate
Connection: keep-alive
If-None-Match: "a54907f38b306fe3ae4f32c003ddd507"
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
If-Modified-Since: Mon, 10 Feb 2014 18:08:48 GMT
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_2) AppleWebKit/537.74.9 (KHTML, like Gecko) Version/7.0.2 Safari/537.74.9
Referer: http://www.objc.io/
DNT: 1
Accept-Language: en-us
```

第一样是请求行，它包含三部分信息：动作，资源信息，还有HTTP的版本

本例中，动作是GET,所谓动作也就是常说的HTTP请求方法， 资源信息表明所请求的资源。例子中的资源信息是 /about.html,这表示我们想get服务器的在/about.html位置中的文档。当前HTTP版本是HTTP/1.1

接下来10行是HTTP header信息。跟着是一行空行。例子中请求没有body信息。

header的作用是向服务器传递一些额外的辅助信息，它的内容比较宽泛，维基百科中又[常用HTTP header关键字](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields)信息的清单。例子中的header信息:`Host: www.objc.io `表示告诉服务器，本次请求的服务器名称是什么。这样可以让同一个服务器处理针对多个域名的请求。

下面是一些常见的header信息：

```
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us
```
服务器可能具备返回多种媒体类型的能力，Accept表示safari希望接收的媒体格式类型，text/html是胡亮媒体，也被称为MIME类型或者是内容类型(Content-TYpes).`q=0.9`表示Safari对给定媒体类型的优先级要求。Accept_Lanaguage 代表Safari希望接收的自然语言清单。这回要求服务器尽可能的根据清单要求去匹配相应的语言。

```
Accept-Encoding: gzip, deflate
```

通过这个header,Safari告诉服务器可以对响应boday做压缩处理，如果header信息中没有设置压缩标识，那么服务器就必须返回没有压缩过的信息。压缩可以大大减少数据的传输量，在文本信息(比如HTML)中尤为明显。

```
If-Modified-Since: Mon, 10 Feb 2014 18:08:48 GMT
If-None-Match: "a54907f38b306fe3ae4f32c003ddd507"
```

这两行信息表明Safari已经对请求结果做过缓存，如果服务器上的请求内容在2月10号以后发生过变化或者是etag与`a54907f38b306fe3ae4f32c003ddd507 `不匹配，这就表示请求结果与当前缓存信息不一致，需要服务器返回最新的请求结果。

user-agent 是告知服务器当前发送请求的客户端类型。



### 一个简单响应

作为上面的请求的响应，服务器的返回是：

```
HTTP/1.1 304 Not Modified
Connection: keep-alive
Date: Mon, 03 Mar 2014 21:09:45 GMT
Cache-Control: max-age=3600
ETag: "a54907f38b306fe3ae4f32c003ddd507"
Last-Modified: Mon, 10 Feb 2014 18:08:48 GMT
Age: 6
X-Cache: Hit from cloudfront
Via: 1.1 eb67cb25620df959ba21a943fbc49ef6.cloudfront.net (CloudFront)
X-Amz-Cf-Id: dDSBgR86EKBemW6el-pBI9kAnuYJEaPQYEqGmBnilD12CbixCuZYVQ==
```

第一行是状态航，它包括HTTP版本，状态码(204)和状态信息

HTTP定义了一系列状态码，他们各有用途，本利中的304表示所请求的信息自上次访问以来没有变化

响应中没有包含body信息。也就说服务器通知客户端：你的版本已经是最新的了，可以直接使用当前缓存信息


### 关闭缓存

用curl 发送一个请求:

```
% curl http://www.apple.com/hotnews/ > /dev/null
```

`curl`没有使用本地缓存，整个请求会是这样:

```
GET /hotnews/ HTTP/1.1
User-Agent: curl/7.30.0
Host: www.apple.com
Accept: */*
```

这个请求与之前Safari发的请求很类似，但是curl请求的header信息中没有`If-None-Match`,所以服务器必须将请求结果返回。

此处`curl`头信息中声明的`Accept: */*`表示可以接收任何媒体类型。

来自www.apple.com的响应：

```
HTTP/1.1 200 OK
Server: Apache
Content-Type: text/html; charset=UTF-8
Cache-Control: max-age=424
Expires: Mon, 03 Mar 2014 21:57:55 GMT
Date: Mon, 03 Mar 2014 21:50:51 GMT
Content-Length: 12342
Connection: keep-alive

<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en-US" lang="en-US">
<head>
    <meta charset="utf-8" />
```
后面还会有一些，现在收到的响应里body中包含了HTML文档信息

Apple服务器响应的状态码是200,这是标准的表示HTTP请求成功的状态码。

服务器同时还告知响应媒体类型是 `text/html`;字符集`charset=utf-t`；内容长度：`Content-length:12342`,还代表了body信息的大小

## HTTPS-安全的HTTP

[Transport Layer Security](https://en.wikipedia.org/wiki/Transport_Layer_Security)(安全传输层协议，TLS)是一种基于TCP的加密协议，它支持两件事情：传输的两端可以互相验证对方的身份，以及加密所传输的数据。基于TLS的HTTP请求就是HTTPS

用https去替代http,在安全方面会有显著的提升。也许你还会采用一些其他的安全措施，总之这都会为安全通信提供保障。


### TLS 1.2
如果服务器支持的话，你应该讲`TLSMinmumSupportedProtocol`设置为`kTLSProtocol12 `,以要求使用[`TLS1.2`](https://en.wikipedia.org/wiki/Transport_Layer_Security#TLS_1.2)版本，这能有效的防御[中间人攻击](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)

## 证书锁定(Certificate Pinning)
如果不确定数据接收方的身份，那么即便对所传输数据进行加密也没什么意义。服务器的证书可以表明服务器的身份，只允许好持有某个特定证书的一方建立连接，就是[证书锁定](https://en.wikipedia.org/wiki/Certificate_pinning#Certificate_pinning)


如果一个客户端通过TLS和服务器建立连接，操作系统会验证服务器证书的有效性。当然，有很多手段可以绕开这个校验，最直接的是在iOS设备上安装证书并且将其设置为可信的，这种情况下，实施[中间人攻击](https://en.wikipedia.org/wiki/Certificate_pinning#Certificate_pinning)也不是什么难事。

可以使用证书锁定来规避这种风险。当监理TLS连接后，应立即检查服务器的证书，不仅要验证证书的有效性，还需要确定证书和其持有者是否匹配。考到应用和服务器需要同时升级证书的要求，这种方式比较适合应用在访问自家服务器的情况下。


为了实现证书锁定，在建立连接的过程中需要对服务器进行信任检查，每当通过NSURLSession创建了连接，`NSUrlSession`的代理就会收到一个
`-URLSession:didReceiveChallenge:completionHandler:`的调用。传递的参数`NSURLAuthenticationChallenge `有一个属性`protectionSpace `，它是`NSURLProtectionSpace `的实例，它有一个`serverTrust `属性。

`serverTrust `是一个`SecTrustRef `对象，`Security`框架提供了很多方法用于验证`SecTrustRef `。`AFNetworking`项目中的`AFSecurityPolicy `就是一个不错的使用。一如既往的提醒大家，如果要自己构建全验证相关的代码，请一定要认真做好代码审查，千万不要再出现诸如`goto fail`这类bug


## 综合讨论

现在大家多IP，TCP，和http的工作原理有了一定的了解了。下面说说还可以做些什么

### 有效地使用连接

TCP连接容易在两个时点出现问题：初始设置，以及通过连接传输的最后一部分报文

### 建立连接

连接设置可能会非常耗时，正如前文所说，TCPj建立连接的过程中需要进行三次握手，这个过程中本身没有太多的数据需要传递，但是，对于移动网络来说，从手机端向服务器端发送一个数据包普遍需要250ms,也就是四分之一秒。推及到三次握手，也就是说在还没有传送任何数据之前，光建立连接就花费了750ms


https的情况更夸张，由于Https是基于TLS的http,而http又基于TCP,TCP连接就要执行三次握手，然后到了TLS层还会再握手三次，估算一下，建立一个HTTPS连接的耗时只要是创建一个https连接的两倍。如果RTT时间是500ms(假设单程250ms)，HTTPS建立连接累计总耗时1.5秒


不管建立连接后传递多少数据，建立连接本身都太过耗时了。


另一个影响TCP连接的因素是传送大规模的数据。如果要在网络情况未知的条件下传送报文，TCP需要侦测当前网络的能力。换句话说，TCP得花费一定的事件去计算此网络最佳传输速率。上文提到过，TCP需要逐步调整以便找到最佳速度。这种算法成为[慢启动](https://en.wikipedia.org/wiki/Slow-start)。还有一点值得注意，慢启动策略在那些数据链路层传输质量较差的网络环境中的表现更差，无线网络就是典型的例子


### 结束连接

另一个问题主要存在于数据传输的最后阶段。每当客户端发起HTTP请求某些资源的时候，服务器会持续的向客户端发送TCP报文数据，客户端收到数据后会给服务器反馈ACK报文数据，客户端收到数据后会给服务器反馈ACK确认信息。假如某个报文在传输过程中发生丢包，那么服务器也就不会收到该包的确认ACK.一旦服务器发现现有数据包没有ACK反馈，就会触发快速重传。

每当某个数据包丢失，数据接收方在收到下个数据包后发出的确认ACK与所接收的前一个数据包的确认ACK相同。那么数据发送方自然就会收到重复的ACK.除了报文丢失，还有很多网络状况会导致重复ACK的问题。一般情况下，如果数据发送方连续收到3个重复的ACK就会立即进行快速重发


这所导致的问题将发生在数据传输的收尾阶段。如果发送方完成数据发送，接受方自然会停止发送ACK确认。在最后四个报文传输的过程中，快速重发算法是没有办法处理这四个报文的数据包丢失问题的（以为不会收到三个相同的确认ACK,所以不能界定传输丢包），在常规的网络环境下，四个数据包相当于5.7kb的数据规模。总之，在这最后的5.7kb的传输过程中，快速重发机制是无效的。针对这种情况，TCP会启用其它机制来侦测丢包问题。对于这种情况，重传操作可能要消耗几秒钟去执行，这并不奇怪



### 长连接和管线化

HTTP有两种策略来解决这些问题，最简单的是[HTTP持久连接(persistent connection)](https://en.wikipedia.org/wiki/HTTP_persistent_connection)，也被称为长连接。具体就是,每当HTTP完成一组请求-响应后，还会继续复用相同的TCP连接。而HTTPS会复用同样的TLS连接：

```
open connection
client sends HTTP request 1 ->
                            <- server sends HTTP response 1
client sends HTTP request 2 ->
                            <- server sends HTTP response 2
client sends HTTP request 3 ->
                            <- server sends HTTP response 3
close connection
```

第二部就是利用了[HTTP管线](https://en.wikipedia.org/wiki/Http_pipelining)处理，即允许客户端利用同样的连接并行发送多个请求，也就是说无需等待上一个请求的响应完成可以发下一个请求。这表示能同时处理请求和响应，请求处理的顺序采用先进先出的原则，响应结果会按照请求发出的顺序依次返回给客户端


稍微简化一下，看起来会是这样：

```
open connection
client sends HTTP request 1 ->
client sends HTTP request 2 ->
client sends HTTP request 3 ->
client sends HTTP request 4 ->
                            <- server sends HTTP response 1
                            <- server sends HTTP response 2
                            <- server sends HTTP response 3
                            <- server sends HTTP response 4
close connection
```

注意，服务器发出的响应是实时的，不会等到接收完全请求才处理。

可以利用这个特点来提升TCP的效率，只需要在简建立连接初始阶段执行握手，而后一直复用同样的连接。这样TCP就可以最大限度的利用带宽，此种情况下，拥塞控制也会随之提升。因为快速重复机制无法处理的最末四个报文丢失情况只会发生在使用本地连接的最后一个请求-响应中，而不是像之前那样每一个请求-响应都需要建立自己的连接，每个连接中都可能出现最后四个报文丢失的问题。


HTTP管线化对高网络延迟连接的通讯性能提升尤为显著，在你的iPhone没有通过wifi访问网络的时候，此类网络连接就属于高延迟范畴。实际上，有调差显示，在移动网络环境下，[SPDY](https://en.wikipedia.org/wiki/SPDY)的通讯性能并不优于HTTP管线。

[RFC 2616](https://tools.ietf.org/html/rfc2616)指明，在与同一个服务器通讯的时候，如果启用了http管线，建议启用两个连接。按照说明所述，这样能获得最优的响应效率，能最大限度避免拥塞，增加更多的连接也不会再对性能有什么明显改善。

遗憾的是，还有相当多的服务器不支持管线话，由于这个原因，HTTP管线在`NSURlSession`中默认是关闭的，如果想要开启HTTP管线，需要将`NSURLSessionConfiguration `中的`HTTPShouldUsePipelining `设置为YES.另外，建议服务器最好还是支持管线化


### 超时处理

我们都有在网络不太好的情况下使用app的经历，很多app大概15s左右就会结束请求并且反馈一个超时信息，这种设计其实是很不好有的。应该给用户一个他们理解的友好提示，比如：“你好，现在网络状态不太好，你需要多等一会”。但是即便网络状况良好，只要连接还在，TCP都会保证将请求发出去并且会一直等待响应的返回，只是时间长短问题。


从另一个角度来说，在较慢的网络中，请求-响应的RTT时间可能会有17秒。如果15秒就决定终止请求，就算用户有足够的耐心，他们也没积水等到想要的操作结果，反过来，如果我们给出用户响应的提示信息，而他们又刚好愿意多等一会，用户可能会更喜欢使用这样的应用。


一直以来都有一种误解，用重发请求来解决上面的问题，注意，这不是问题的关键，因为TCP有自己的重复机制。


正确的处理方式应该是：没放发起一个请求的时候，同时启动一个10秒计时器，如果请求在10秒之内返回，就把计时器关掉。如果超过10秒，可以给用户一个提示：“网络不好请稍后”。我建议再给用户一个取消按钮，让他们可以自行选择等待还是取消请求，当然提示信息的具体内容和是否配备取消按钮，这个可以视各app的情况去决定。总而言之，开发者最好不要直接替用户做决定，比如直接终止他们的请求。

只要连接双方的IP地址是不变的，可用的，连接就一定是活跃的。如果把iPhone从wifi连接切换到3G网络，这样连接就会变得不可用，因为的IP地址发生了变化，基于原IP地址创建的路由自然是失效的。

## 缓存

