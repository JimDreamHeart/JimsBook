# HTTP、TCP和Socket

----
## HTTP

### 主要特点
  * 简单快速：客户向服务器请求服务时，只需传送请求方法和路径。请求方法常用的有GET、HEAD、POST。每种方法规定了客户与服务器联系的类型不同。由于HTTP协议简单，使得HTTP服务器的程序规模小，因而通信速度很快。
  * 灵活：HTTP允许传输任意类型的数据对象。正在传输的类型由Content-Type加以标记。
  * 无连接：无连接的含义是限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开连接。采用这种方式可以节省传输时间。
  * 无状态：HTTP协议是无状态协议。无状态是指协议对于事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就较快。
  * 支持B/S及C/S模式。

### 关于HTTP
TCP协议对应于传输层，而HTTP协议对应于应用层，从本质上来说，二者没有可比性。  
  * Http协议是建立在TCP协议基础之上的，当浏览器需要从服务器获取网页数据的时候，会发出一次Http请求。
  * Http会通过TCP建立起一个到服务器的连接通道，当本次请求的数据完毕后，Http会立即将TCP连接断开，这个过程是很短的。所以Http连接是一种短连接，是一种无状态的连接。所谓的无状态，是指浏览器每次向服务器发起请求的时候，不是通过一个连接，而是每次都建立一个新的连接。如果是一个连接的话，服务器进程中就能保持住这个连接并且在内存中记住一些信息状态。而每次请求结束后，连接就关闭，相关的内容就释放了，所以记不住任何状态，成为无状态连接。

关于Keep-Alive：  
  * 从HTTP/1.1起，默认都开启了Keep-Alive，保持连接特性，简单地说，当一个网页打开完成后，客户端和服务器之间用于传输HTTP数据的TCP连接不会关闭，如果客户端再次访问这个服务器上的网页，会继续使用这一条已经建立的连接Keep-Alive不会永久保持连接，它有一个保持时间，可以在不同的服务器软件（如Apache）中设定这个时间。虽然这里使用TCP连接保持了一段时间，但是这个时间是有限范围的，到了时间点依然是会关闭的，所以我们还把其看做是每次连接完成后就会关闭。后来，通过Session, Cookie等相关技术，也能保持一些用户的状态。但是还是每次都使用一个连接，依然是无状态连接

为什么Http是无状态的短连接呢？而TCP是有状态的长连接？Http不是建立在TCP的基础上吗，为什么还能是短连接？
  * Http就是在每次请求完成后就把TCP连接关了，所以是短连接。而我们直接通过Socket编程使用TCP协议的时候，因为我们自己可以通过代码区控制什么时候打开连接什么时候关闭连接，只要我们不通过代码把连接关闭，这个连接就会在客户端和服务端的进程中一直存在，相关状态数据会一直保存着。

### 请求方法
根据HTTP标准，HTTP请求可以使用多种请求方法。  
HTTP1.0定义了三种请求方法： GET, POST 和 HEAD方法。  
HTTP1.1新增了五种请求方法：OPTIONS, PUT, DELETE, TRACE 和 CONNECT 方法。  
  * GET->请求指定的页面信息，并返回实体主体。
  * HEAD->类似于get请求，只不过返回的响应中没有具体的内容，用于获取报头
  * POST->向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST请求可能会导致新的资源的建立和/或已有资源的修改。
  * PUT->从客户端向服务器传送的数据取代指定的文档的内容。
  * DELETE->请求服务器删除指定的页面。
  * CONNECT->HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器。
  * OPTIONS->允许客户端查看服务器的性能。
  * TRACE->回显服务器收到的请求，主要用于测试或诊断。

### 工作原理
HTTP 请求/响应的步骤：  
#### 客户端连接到Web服务器
  * 一个HTTP客户端，通常是浏览器，与Web服务器的HTTP端口（默认为80）建立一个TCP套接字连接。
#### 发送HTTP请求
  * 通过TCP套接字，客户端向Web服务器发送一个文本的请求报文，一个请求报文由请求行、请求头部、空行和请求数据四部分组成。
#### 服务器接受请求并返回HTTP响应
  * Web服务器解析请求，定位请求资源。服务器将资源复本写到TCP套接字，由客户端读取。一个响应由状态行、响应头部、空行和响应数据4部分组成。
#### 释放连接[TCP连接](http://www.jianshu.com/p/ef892323e68f)
  * 若connection 模式为close，则服务器主动关闭TCP连接，客户端被动关闭连接，释放TCP连接;若connection 模式为keepalive，则该连接会保持一段时间，在该时间内可以继续接收请求;
#### 客户端浏览器解析HTML内容
客户端浏览器首先解析状态行，查看表明请求是否成功的状态代码。然后解析每一个响应头，响应头告知以下为若干字节的HTML文档和文档的字符集。客户端浏览器读取响应数据HTML，根据HTML的语法对其进行格式化，并在浏览器窗口中显示。  
例如：在浏览器地址栏键入URL，按下回车之后会经历以下流程：  
  * 浏览器向 DNS 服务器请求解析该 URL 中的域名所对应的 IP 地址;
  * 解析出 IP 地址后，根据该 IP 地址和默认端口 80，和服务器建立TCP连接;
  * 浏览器发出读取文件(URL 中域名后面部分对应的文件)的HTTP 请求，该请求报文作为`TCP 三次握手`的第三个报文的数据发送给服务器;
  * 服务器对浏览器请求作出响应，并把对应的 html 文本发送给浏览器;
  * 释放TCP连接;
  * 浏览器将该 html 文本并显示内容;


## TCP协议
TCP协议主要是在传输层，三次握手四次挥手。
### 三次握手
  * 1.客户端尝试连接服务器，向服务器发送syn包（同步序列编号Synchronize Sequence Numbers），syn=j，客户端进入SYN_SEND状态等待服务器确认；
  * 2.服务器接收客户端syn包并确认（ack=j+1），同时向客户端发送一个SYN包（syn=k），即SYN+ACK包，此时服务器进入SYN_RECV状态；
  * 客户端收到服务器的SYN+ACK包，向服务器发送确认包ACK(ack=k+1），此包发送完毕，客户端和服务器进入ESTABLISHED状态，完成三次握手。

### 四次挥手
  * Client发送一个FIN，用来关闭Client到Server的数据传送，Client进入FIN_WAIT_1状态；
  * Server收到FIN后，发送一个ACK给Client，确认序号为收到序号+1（与SYN相同，一个FIN占用一个序号），Server进入CLOSE_WAIT状态；
  * Server发送一个FIN，用来关闭Server到Client的数据传送，Server进入LAST_ACK状态；
  * Client收到FIN后，Client进入TIME_WAIT状态，接着发送一个ACK给Server，确认序号为收到序号+1，Server进入CLOSED状态，完成四次挥手。


## Scoket
是支持TCP/IP协议的网络通信的基本操作单元。  
它是网络通信过程中端点的抽象表示，包含进行网络通信必须的五种信息：连接使用的协议，本地主机的IP地址，本地进程的协议端口，远地主机的IP地址，远地进程的协议端口。

### 作用
  * 应用层通过传输层进行数据通信时，TCP和UDP会遇到同时为多个应用程序进程提供并发服务的问题。
  * 为了区别不同的应用程序进程和连接，许多计算机操作系统为应用程序与TCP／IP协议交互提供了称为套接字(Socket)的接口，区分不同应用程序进程间的网络通信和连接。

### 原理
生成套接字，主要有3个参数：通信的目的IP地址、使用的传输层协议(TCP或UDP)和使用的端口号。  
Socket可以看成在两个程序进行通讯连接中的一个端点，一个程序将一段信息写入Socket中，该Socket将这段信息发送给另外一个Socket中，使这段信息能传送到其他程序中。  

#### Socket连接的实现方式
要通过互联网进行通信，至少需要一对套接字，一个运行于客户机端，称之为`ClientSocket`，另一个运行于服务器端，称之为`ServerSocket`。  
根据连接启动的方式以及本地套接字要连接的目标，套接字之间的连接过程可以分为三个步骤：  
  * 服务器监听：是服务器端套接字并不定位具体的客户端套接字，而是处于等待连接的状态，实时监控网络状态。
  * 客户端请求：是指由客户端的套接字提出连接请求，要连接的目标是服务器端的套接字。为此，客户端的套接字必须首先描述它要连接的服务器的套接字，指出服务器端套接字的地址和端口号，然后就向服务器端套接字提出连接请求。
  * 连接确认：是指当服务器端套接字监听到或者说接收到客户端套接字的连接请求，它就响应客户端套接字的请求，建立一个新的线程，把服务器端套接字的描述发给客户端，一旦客户端确认了此描述，连接就建立好了。而服务器端套接字继续处于监听状态，继续接收其他客户端套接字的连接请求。

#### Socket与TCP/IP的关系
  * 创建Socket连接时，可以指定使用的传输层协议，Socket可以支持不同的传输层协议（TCP或UDP），当使用TCP协议进行连接时，该Socket连接就是一个TCP连接。
  * socket则是对TCP/IP协议的封装和应用；TPC/IP协议是传输层协议，主要解决数据如何在网络中传输；HTTP是应用层协议，主要解决如何包装数据。
