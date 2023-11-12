---
title: "传输控制协议（TCP)知识点总结"
date:  2023-05-03T23:00:11+08:00
draft: false
# summary: "博客的概述" # 文章简单描述，会展示在主页
# categories:
# - category 1
# - category 2

tags:
- 计算机网络

---

# 传输控制协议（TCP)知识点总结

>维基百科:
>
>传输控制协议（TCP）是Internet协议套件中的主要协议之一。它起源于最初的网络实现中，它补充了Internet协议（IP）。因此，整个套件通常称为TCP/IP。TCP在通过IP网络通信的主机之间提供可靠的、有序的、经过检查的字节流传输。主要的互联网应用程序，如万维网、电子邮件、远程管理和文件传输，都依赖于TCP，它是TCP/IP套件的传输层的一部分。SSL/TLS通常运行在TCP之上。
>
>TCP是面向连接的，客户端和服务器之间必须建立连接后才能发送数据。在建立连接之前，服务器必须在等待（被动打开）来自客户端的连接请求。三次握手（主动打开）、重传和错误检测增加了可靠性，但也延长了延迟时间。不需要可靠数据流服务的应用程序可以使用用户数据报协议（UDP），它提供了一个无连接的数据报服务，优先考虑时间而非可靠性。TCP采用网络拥塞避免。但是，TCP存在漏洞，包括拒绝服务攻击、连接劫持、TCP否决和重置攻击。
>
>数据在TCP层称为流（Stream），数据分组称为分段（Segment）。作为比较，数据在IP层称为Datagram，数据分组称为分片（Fragment）。 UDP 中分组称为Message。

## 介绍
传输控制协议(`TCP`)是一种在`IP`之上使用的传输协议，用于确保数据包的可靠传输。

`TCP`拥有一些机制，可以解决数据包传输的传递中出现的许多问题，例如数据包丢失、数据包乱序、数据包重复和数据包损坏。

由于`TCP`是`IP`之上最常用的协议，因此`Internet`协议栈有时也称为`TCP/IP`。

## 数据包格式

使用`TCP/IP`发送数据包时，每个`IP`数据包的数据部分被格式化为`TCP`段。

![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets%E6%9C%AA%E5%91%BD%E5%90%8D%E6%96%87%E4%BB%B6.png)


## TCP连接的建立和关闭

### 三次握手

任何TCP连接在建立之前都需要经过三次握手，确保通信双方都能正确的接收和发送数据，这也是TCP可靠性的保障之一。
让我们来研究一下TCP三次握手的过程：

下图是使用`wireshark`捕获到的三次握手的过程（服务端监听的端口为`8888`）：

![wireshark三次握手](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230503172623.png)

1. 首先客户端(图中端口14243)先向服务端（图中端口8888）发送一个同步位SYN被设置为1的TCP报文段，用来表明这是一个TCP连接请求报文段，并将Seq需要字段设置为x(上图中的连接的x是0)，表明客户端选择x作为这个连接的初始序号。发送该报文段后，客户端进入同步发送状态（SYN-SENT）。
2. 然后服务端收到客户端的TCP连接请求报文段后，服务端同意连接后会向请求的客户端回复一个请求确认报文，这个报文的同步位SYN和确认位ACK都被置为1，序号Seq设置为y(上图是y=0)，确认号字段ack被设置为x+1（对于图中连接来说就是x+1=0+1=1）。发送该报文段后，服务端进入同步接收状态（SYN-RECEIVED）。
3. 最后，在客户端收到服务端发送的请求确认报文段后会回复一个确认报文段，该报文的确认位ACK置1，序号seq置为x+1,确认号字段ack置为y+1。发送该报文段后，客户端进入已建立连接状态（ESTABLISHED）
4. 服务端接收到客户端发出的确认报文段后也进入已建立连接状态（ESTABLISHED）。

>注意：TCP规定SYN被设置为1的报文段不能携带数据而且会消耗掉一个序号。在三次握手的第三次回复中，该报文段没有设置SYN为1，所以它是可以携带数据的。但如果不携带数据则不消耗序号，这时下一次发送的报文段的Seq还是x+1。

下图展示了三次握手的过程：
![TCP三次握手](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230503180039.png)
### 四次挥手

TCP在关闭连接时会进行四次挥手，这确保了数据能够完整地传输并且双方都能正常关闭连接。

下图是`wireshark`捕获的一次TCP连接的四次挥手记录

![wireshark四次挥手](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230503172607.png)

1. 首先假设挥手的发起者是客户端（图中端口4676），则客户端会发送一个终止位FIN和确认位ACK置为1的报文段，表示客户端想要释放该TCP连接，并将序号Seq设置为u(即该连接请求断开之前已传送过的数据的最后一个字节的序号加1，图中Seq为6)，ack置为v(即该连接请求断开之前已收到的数据的最后一个字节的序号加1，图中ack为14)。发送该报文段后，客户端进入终止等待1状态（FIN-WAIT-1）。
2. 之后，服务端收到该连接终止请求的报文段，服务端会立即回复一个确认报文段，该报文段的确认位ACK置为1，序号Seq置为v，确认号字段ack置为u+1。发送该确认报文段后，服务端进入关闭等待状态（CLOSE-WAIT）。
3. 在服务端发送确认报文段后，表明客户端到服务端的连接就被释放了，现在该连接处于半关闭状态，客户端不再向服务端发送数据，但是服务端如果还有数据要发送，客户端仍然会接收，这个状态会持续到服务端确认没有数据传输后，主动发出终止连接的报文段。
4. 在客户端收到服务端发来的确认报文段后，会进入终止等待2状态（FIN-WAIT-2）。该状态会一直等待服务端发出的终止连接的报文段。
5. 在服务端确认没有数据要发送时，会通知终止连接，并发出一个终止连接的报文段。该报文段的终止位FIN和确认位ACK置为1，序号Seq置为w(如果服务端在半关闭状态下发送了数据那么Seq就会变化，如果没有发送数据，那么Seq=v)，确认号字段ack还是置为u+1。发送该报文段后，服务端进入最后确认状态（LAST-ACK）。
6. 客户端在接收到服务端发来的终止连接的报文段后，会回复一个确认报文段，该报文段确认位ACK置为1，序号Seq置为u+1，确认号字段ack置为w+1。该报文段发送后，客户端进入时间等待状态（TIME-WAIT）。
7. 服务端在收到客户端发来的确认报文段后，会进入已关闭状态（CLOSED）。而客户端需要等待2MSL的时间后才会从时间等待状态（TIME-WAIT）进入已关闭状态（CLOSED）。（MSL（Maximum Segment Lifetime)即报文段最大生存时间，RFC793建议为2分钟）。

>注意：TCP规定终止位FIN置为1的报文段即使不携带数据，也要消耗掉一个序号

下图展示了四次挥手的过程：
![TCP四次挥手](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230503220501.png)

### 一张图展示这些过程

![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230503180728.png)

## TCP的特点

### 流量控制

TCP会对数据的发送和接收进行流量控制，通过`滑动窗口`机制来限制发送方的发送速度，确保接收方能够处理接收到的数据，避免数据的丢失或拥塞。

推荐视频:[https://www.bilibili.com/video/BV1c4411d7jb?p=60](https://www.bilibili.com/video/BV1c4411d7jb?p=60)

### 拥塞控制

TCP会对网络拥塞情况进行探测和调整，根据网络拥塞情况动态调整发送方的发送速度，以避免网络拥塞引起数据的丢失或延迟。

推荐视频：[https://www.bilibili.com/video/BV1c4411d7jb?p=61](https://www.bilibili.com/video/BV1c4411d7jb?p=61)

### 检测丢失的报文段

TCP连接使用超时检测丢失的TCP报文段。

发送方在发送TCP报文段后，会启动一个计时器并将该TCP报文段放入重传队列中。如果计时器超时，发送方仍未收到接收方发来的确认（ACK）报文段，它会重新发送该TCP报文段。

### 处理乱序数据

TCP的接收端可以按序号重组数据，并将已经接收到的数据发送给应用程序，保证应用程序接收到的数据的顺序和发送端发送的顺序一致。

### 处理损坏的数据报文段

TCP报文段在传输过程中可能出现一些差错。发送方给接收方发送TCP报文段，接收方会根据报文段首部中的校验和字段值，检查该报文段是否无误，如果没问题，接收方会发送一个确认报文段给发送方，如果报文段出现问题，接收方会丢弃该报文段，不向发送方发送确认报文段，这就会使得发送方对该报文段的超时重发。



## TCP（Transmission Control Protocol）和UDP（User Datagram Protocol）对比

1. tcp是面向连接的；udp是无连接的
>这里的连接指的是逻辑连接，也称为会话。在TCP协议中，连接是指通过三次握手建立的逻辑连接，它包括发送方和接收方之间的会话状态信息。在连接建立后，发送方和接收方之间可以进行数据的可靠传输，并且连接状态信息会一直保留在双方的内存中，直到连接被显式地关闭。在UDP协议中，由于没有连接的概念，因此每个数据包都是独立的，发送方和接收方之间没有任何状态信息。
2. tcp每一条连接只能有两个端点，是一对一通信；udp可以一对一，一对多和多对多通信，即支持单播、多播和广播。
>因为UDP协议本身没有提供可靠性保障和连接状态维护的机制，因此UDP协议允许将数据包发送到多个目标主机。TCP协议是一种面向连接的协议，必须在双方建立连接之后才能进行数据传输，因此TCP协议只支持点对点通信，即一对一通信。
3. tcp面向字节流；udp直接将应用层交付的报文打包
>因为TCP协议将应用程序交付的数据视为一连串的无结构字节流进行传输，而不是将数据分割成具有独立含义的报文进行传输。TCP协议会在传输过程中将字节流分割成一系列大小不等的数据块（即TCP段），并在接收方重新组装成原始的字节流。这种字节流传输的方式可以保证数据的可靠性和有序性，但也可能导致数据包的边界模糊，需要应用程序自己进行处理。
>
>相比之下，UDP协议直接将应用层交付的报文打包成UDP数据包进行传输，每个UDP数据包都是独立的，具有独立的含义和边界。UDP协议不提供数据的可靠性和有序性保障，因此应用程序需要自己处理数据包的丢失、重复和乱序等问题。
4. tcp是可靠传输，使用流量控制和拥塞控制；udp尽最大努力交付，不是可靠传输，不使用流量控制和拥塞控制。

5. tcp由于提供可靠传输，其报文段首部最小20字节，最大60字节；udp首部开销比较小，只有8字节。
6. tcp适用于要求可靠传输的场景（文件传输等）；udp适用于实时应用（视频会议、IP电话等）


## 其他知识

1. 在TCP连接中，服务端使用一个五元组（`源IP地址，源端口，目的IP地址，目的端口，协议类型`）来唯一标识客户端的连接。当服务端监听一个端口时，它会接受来自任何源IP地址和源端口号的连接请求，但是每个连接都必须具有唯一的目的IP地址、目的端口号和协议类型。


## 疑问

### 服务器一个端口可以连接多个客户端，反过来客户端可以连接多个服务器吗？
>在我的印象中，好像建立的TCP连接都是一个服务端监听一个端口号，可以与多个客户端建立TCP连接，然后每个客户端要与服务器连接也需要选择一个本地端口与服务端连接。但是按五元组（`源IP地址，源端口，目的IP地址，目的端口，协议类型`）来区分不同连接来说，服务端可以区分不同的客户端，那反过来客户端应该也是可以区分服务端的。


既然有疑惑那就实验一下

实验：
>前提说明：
>1. 我测试使用的是JDK8
>2. 系统为`windows11系统`上

>下面通过开启两个“服务端”监听两个不同端口，然后使用一个客户端连接这两个“服务端”。


服务端代码：
```java
import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;

public class JavaServer implements Runnable {

    private Socket socketClient;

    public JavaServer(Socket socketClient) {
        this.socketClient = socketClient;
    }

    public static void main(String[] args) throws Exception {
        //从运行程序的命令行参数中获取端口
        int port = Integer.parseInt(args[0]);

        try (ServerSocket socket = new ServerSocket(port)) {

            System.out.println("Server listening at " + port);
            while (true) {
                Socket client = socket.accept();
                System.out.println("accept:" + client.getRemoteSocketAddress());
                new Thread(new JavaServer(client)).start();
            }
        }

    }


    @Override
    public void run() {
        try {

            // 获取客户端输入流和输出流
            InputStream inputStream = socketClient.getInputStream();
            OutputStream outputStream = socketClient.getOutputStream();

            // 定义一个缓冲区，用于读取客户端发送的数据
            byte[] buffer = new byte[1024];

            // 循环读取客户端发送的数据，并原样写回客户端
            int length;
            while ((length = inputStream.read(buffer)) != -1) {
                System.out.println("client said:" + new String(buffer, 0, length));
                outputStream.write(buffer, 0, length);
                outputStream.flush();
            }
            System.out.println("client close");
            // 关闭客户端连接
            socketClient.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

客户端代码：
```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.Socket;

public class JavaClient {
    public static void main(String[] args) {
        try {
             Socket socket = new Socket();
            socket.bind(new InetSocketAddress(8080));
            socket.connect(new InetSocketAddress("127.0.0.1", 8888));
            socket.connect(new InetSocketAddress("127.0.0.1", 8889));
            System.out.println("连接成功，开始循环");
            while (true) {

            }
        } catch (IOException e) {
            e.printStackTrace();
            System.out.println("陷入循环");
            while (true) {

            }
        }

    }
}
```

启动两个服务器分别监听 8888 和 8889 端口
```bash
java JavaServer 8888
```
和
```bash
java JavaServer 8889
```

之后启动客户端连接这个两个端口。

不幸的是客户端启动抛出异常了，客户端运行输出如下：
```
java.net.SocketException: already connected
	at java.net.Socket.connect(Socket.java:586)
	at java.net.Socket.connect(Socket.java:555)
	at JavaClient.main(JavaClient.java:14)
陷入循环
```
查看TCP连接情况：

![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230510173014.png)

发现只有8888建立了连接，8889没有建立连接。

通过查看java 中connect的源码，发现他会判断这个Socket对象是否已经连接建立连接了，如果已经建立连接了则会抛出异常。
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230510174809.png)

既然不能再同一个socket对象上调用connect，那么不同对象是否可以呢？修改客户端代码如下：
```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.Socket;

public class JavaClient {
    public static void main(String[] args) {
        try {
            Socket socket1 = new Socket();
            socket1.bind(new InetSocketAddress(8080));
            socket1.connect(new InetSocketAddress("127.0.0.1", 8888));
            System.out.println("socket1 connect");

            Socket socket2 = new Socket();
            socket2.bind(new InetSocketAddress(8080));
            socket2.connect(new InetSocketAddress("127.0.0.1", 8889));
            System.out.println("socket2 connect");

            System.out.println("连接成功，开始循环");
            while (true) {

            }
        } catch (IOException e) {
            e.printStackTrace();
            System.err.println("陷入循环");
            while (true) {

            }
        }

    }
}
```
服务端依旧和第一次测试的一样监听8888和8889端口，启动客户端后的输出如下：
```
socket1 connect
java.net.BindException: Address already in use: JVM_Bind
	at java.net.DualStackPlainSocketImpl.bind0(Native Method)
	at java.net.DualStackPlainSocketImpl.socketBind(DualStackPlainSocketImpl.java:102)
	at java.net.AbstractPlainSocketImpl.bind(AbstractPlainSocketImpl.java:513)
	at java.net.PlainSocketImpl.bind(PlainSocketImpl.java:180)
	at java.net.Socket.bind(Socket.java:661)
	at JavaClient.main(JavaClient.java:20)
陷入循环
```
查看TCP连接情况：
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230510175514.png)
发现依旧只有第一个连接成功。
不过这次报错是`Address already in use`,难道是说不能在同一个地址上绑定多个socket。
通过追踪报错来源（查看了一些相关的native方法的源码），如下：
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230510181817.png)
是一个`WSAEADDRINUSE`的错误代码，查看微软的文档[https://learn.microsoft.com/en-us/windows/win32/winsock/windows-sockets-error-codes-2](https://learn.microsoft.com/en-us/windows/win32/winsock/windows-sockets-error-codes-2)

![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230510182209.png)

翻译如下：
```
地址已被使用。

通常，每个套接字地址（协议/IP地址/端口）只允许使用一次。
如果应用程序尝试将套接字绑定到已被现有套接字使用的IP地址/端口，或者一个没有正确关闭的套接字，或者正在关闭过程中的套接字，则会发生此错误。
对于需要将多个套接字绑定到相同端口号的服务器应用程序，请考虑使用setsockopt（SO_REUSEADDR）。
客户端应用程序通常根本不需要调用bind，connect会自动选择未使用的端口。
当使用通配符地址（涉及ADDR_ANY）调用bind时，WSAEADDRINUSE错误可能会延迟到特定地址提交之前发生。
这可能会发生在稍后调用另一个函数时，包括connect、listen、WSAConnect或WSAJoinLeaf。
```
从上面的文档可以看到，通常情况套接字地址至只允许被使用一次。文档也提到如果需要将多个套接字绑定到相同端口号，可以使用`setsockopt（SO_REUSEADDR）`,这个倒是提供了一个方向。在Java的socket提供的api中确实也有这个选项，接下来修改一下客户端代码如下：
```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.Socket;

public class JavaClient {
    public static void main(String[] args) {
        try {
            Socket socket1 = new Socket();
            socket1.setReuseAddress(true);
            socket1.bind(new InetSocketAddress(8080));
            socket1.connect(new InetSocketAddress("127.0.0.1", 8888));
            System.out.println("socket1 connect");

            Socket socket2 = new Socket();
            socket2.setReuseAddress(true);
            socket2.bind(new InetSocketAddress(8080));
            socket2.connect(new InetSocketAddress("127.0.0.1", 8889));
            System.out.println("socket2 connect");

            System.out.println("连接成功，开始循环");
            while (true) {

            }
        } catch (IOException e) {
            e.printStackTrace();
            System.err.println("陷入循环");
            while (true) {

            }
        }

    }
}

```

代码中通过调用`setReuseAddress`设置为`true`来开启这个选项。令人失望的是和前一个测试的报错一样，没有任何变化。
看一下这个方法的注释：
```
Enable/disable the SO_REUSEADDR socket option.
When a TCP connection is closed the connection may remain in a timeout state for a period of time after the connection is closed (typically known as the TIME_WAIT state or 2MSL wait state). For applications using a well known socket address or port it may not be possible to bind a socket to the required SocketAddress if there is a connection in the timeout state involving the socket address or port.
Enabling SO_REUSEADDR prior to binding the socket using bind(SocketAddress) allows the socket to be bound even though a previous connection is in a timeout state.
When a Socket is created the initial setting of SO_REUSEADDR is disabled.
The behaviour when SO_REUSEADDR is enabled or disabled after a socket is bound (See isBound()) is not defined.
```
翻译如下:
```
启用/禁用 SO_REUSEADDR 套接字选项。
当一个 TCP 连接关闭时，连接可能会在连接关闭后的一段时间内保持超时状态（通常称为 TIME_WAIT 状态或 2MSL 等待状态）。对于使用公知的套接字地址或端口的应用程序,如果涉及套接字地址或端口的连接处于超时状态,则可能无法将套接字绑定到所需的SocketAddress。

在使用 bind(SocketAddress)绑定套接字之前启用 SO_REUSEADDR 允许绑定套接字，即使先前的连接处于超时状态也是如此。

创建Socket时,SO_REUSEADDR的初始设置为禁用。

未定义套接字绑定后启用或禁用 SO_REUSEADDR 时的行为（请参阅 isBound()）。
```

似乎在这个注释中我们只能得出在一个TCP连接还处于超时状态的时候，这个连接占用的地址端口想要被另外的套接字使用就需要在另外的套接字中设置启用`SO_REUSEADDR`，可是我测试的是同时存在两条TCP连接，并不会使得另外一条连接处于超时状态。

而且这个和`WSAEADDRINUSE`错误文档中说明的可以使用setsockopt（SO_REUSEADDR）将多个套接字绑定到相同端口号不太相符。
在看native源码的时候我发现了一个参数

![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230511162939.png)

这个`exclBind`参数看起来像是排他绑定的意思
接着看一下`NET_WinBind`的实现如下：
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230511163308.png)

再看看`setExclusiveBind`方法

![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230511163502.png)

看得出来这个方法的做的事情就是获取指定`socketfd`的`SO_REUSEADDR`是否启用(将获取结果存储在变量`parg`中)，没有启用则设置`SO_EXCLUSIVEADDRUSE`选项，这个选项看意思就是排他的意思，应该只允许绑定一个套接字到指定的地址端口。

到目前知道了在java调用`bind0`的时候会根据`exclBind`来确定是否调用`setExclusiveBind`设置排他选项，而且最后还会根据是否启用了`SO_REUSEADDR`来决定要不要启用`SO_EXCLUSIVEADDRUSE`。

知道了这些，回去继续调试跟踪java的源代码发现默认情况下的`exclBind`是`true`。
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230511165257.png)
再看看这个属性还有哪里可以被赋值，最后看到这个`exclusiveBind`在一个静态代码块中有相关赋值操作：
```
static {
        java.security.AccessController.doPrivileged( new PrivilegedAction<Object>() {
                public Object run() {
                    version = 0;
                    try {
                        version = Float.parseFloat(System.getProperties().getProperty("os.version"));
                        preferIPv4Stack = Boolean.parseBoolean(
                                          System.getProperties().getProperty("java.net.preferIPv4Stack"));
                        exclBindProp = System.getProperty("sun.net.useExclusiveBind");
                    } catch (NumberFormatException e ) {
                        assert false : e;
                    }
                    return null; // nothing to return
                } });

        // (version >= 6.0) implies Vista or greater.
        if (version >= 6.0 && !preferIPv4Stack) {
                useDualStackImpl = true;
        }

        if (exclBindProp != null) {
            // sun.net.useExclusiveBind is true
            exclusiveBind = exclBindProp.length() == 0 ? true
                    : Boolean.parseBoolean(exclBindProp);
        } else if (version < 6.0) {
            exclusiveBind = false;
        }
    }
```

根据上面的代码可以看出`exclusiveBind`可以通过`exclBindProp`赋值，这个`exclBindProp`就是`sun.net.useExclusiveBind`这个系统属性。

而且在`exclusiveBind`为`true`时并没有真正的设置`SO_REUSEADDR`选项，可看下面的代码：
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230511174402.png)

知道了如何赋值，那么我们可以在运行代码时将属性设置为`false`即可，客户端代码不变。

运行命令如下：
```bash
java -Dsun.net.useExclusiveBind=false JavaClient
```

运行结果输出：
```
socket1 connect
socket2 connect
连接成功，开始循环
```

查看TCP连接情况：
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets20230511170256.png)

发现`8080`成功连接上了`8888`和`8889`端口并建立了tcp连接。



