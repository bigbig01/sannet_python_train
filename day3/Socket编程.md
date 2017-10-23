# Socket编程

## 面向连接与无连接

无论使用哪一种地址家族，套接字的类型只有两种。一种是**面向连接**的套接字，即在通信之前一定要建立一条连接，就像跟朋友打电话一样，这种方式提供了顺序的、可靠的、不会重复的数据传输，而且也不会被加上数据边界。这也意味着，每一个要发送的数据，可能会被拆分成多份，每一份都会不多不少地正确到达目的地，然后被重新按顺序拼装起来，传给正在等待的应用程序。实现这种连接的主要协议就是**传输控制协议（TCP）**。
与虚电路完全相反的是数据报型的**无连接**套接字，意味着，无需建立连接就可以进行通讯。但这时，数据到达的顺序、可靠性及不重复性就无法保证了。数据报会保留数据边界，这就表示，数据是整个发送的，不会像面向连接的协议那样被先拆分成小块。实现这种连接的主要协议就是用户数据报协议（UDP）。

## socket模块

在网络编程当中的一个基本组件就是套接字（socket）。套接字主要是两个程序之间的“信息通道”。程序可能（通过网络连接）分布在不同的计算机上，通过套接字相互发送信息。在Python中的大多数的网络编程都**隐藏了socket模块的基本细节**，并且不直接和套接字交互。

套接字分为两个：服务器套接字和客户机套接字。

```
socket(family, type, protocol = 0)
```

一个套接字就是一个 socket 模块中的socket类的实例。它的实例化需要3个参数：
第一个参数是地址族（**默认是socket.AF\_INET**）：

```
socket.AF_INET			:IPv4
socket.AF_INET6			:IPv6
socket.AF_UNIX			:UNIX系统进程间通信
socket.AF_NETLINK
```

第二个参数是流`socket.SOCK_STREAM`或数据报`socket.SOCK_DGRAM`套接字。

第三个参数是使用的协议。通常对于一个普通的套接字，不需要提供任何参数，其默认值为0。

创建一个TCP套接字

	tcp_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

创建一个UDP套接字

	udp_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

## 套接字对象内建方法

> bind(address)

使用bind方法绑定一个socket到本地地址。对IP socket来说，地址的格式是(host, port)一个元祖。

> listen(backlog)

使用listen函数来使server监听绑定的地址，backlog表示在拒绝连接之前，操作系统可以挂起的最大连接数量，该值至少为1，大部分应用程序设为5就可以了。当服务端在处理一个请求的同时又来一个请求，操作系统会挂起这个请求。直到前面的一个请求处理完成之后，才会继续处理这个请求。

> accept()

服务器端套接字开始监听后，它就可以接受客户端的连接。这个步骤使用 accept方法来完成，这个方法会**阻塞**直到有客户端连接，然后该方法就返回一个格式为（client，address）的元祖，**client是一个客户端套接字**，用于后续的通信。这种场景类似于当一个客户打电话进来，总机接了电话，然后把电话转到合适的人那里来处理客户的请求。
**这种形式的服务器端编程称为阻塞或者是同步网络编程。**

> connect(addr)

客户端套接字使用connect方法连接到服务器。如果连接出错，返回socket.error错误。

> connect_ex(address)

连接成功时返回0，连接失败时返回错误编码。

> close()

客户端socket在完成之后需要关闭。

> recv(bufsize[, flag])

Receive up to buffersize bytes from the socket.  For the optional flags
argument, see the Unix manual.  When no data is available, block until
at least one byte is available or until the remote end is closed.  When
the remote end is closed and all data is read, return the empty string.

> recvfrom(bufsize[, flag])

与recv()类似，但返回值是(data, address)。其中data是包含接收数据的字符串，address是发送数据的套接字地址。

> send(string[, flag])

将string中的数据发送到连接的套接字，返回值是要发送的字节数量，该数量可能小于string的字节大小。

> sendall(string[, flag])

将string中的数据发送到连接的套接字，但在返回之前会尝试发送所有数据。成功返回None，失败则抛出异常。

> sendto(stringp[, flag], address)

将数据发送到套接字，address是形式为(ip\_addr, port)的元祖，指定远程地址。返回值是发送的字节数。该函数主要用于UDP协议

> setblocking(flag)

如果flag为0，则将套接字设置为非阻塞模式，否则，套接字将为阻塞模式（默认值）。在非阻塞模式下，如果recv()调用没有发现任何数据或者send()调用无法立即发送数据，那么将引发socket.error异常。在阻塞模式下，这些调用在处理之前都将被阻塞。

> settimeout(timeout)

设置套接字操作的超时时间，timeout是一个浮点数，单位是秒。值为None表示没有超时时间。一般，超时时间应该在刚创建套接字时设置，因为它们可能用于连接的操作（如connect）

> getpeername()

返回连接套接字的远程地址，返回值通常是元祖(ipaddr, port)

> getsockname()

返回套接字自己的地址。通常是一个元祖(ipaddr, port)

> fileno()

套接字的文件描述符

## 应用实例：

### 创建一个TCP服务器

```python
#!/usr/bin/env python

import socket

host = 'localhost'
port = 12345

tcp_server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
tcp_server.bind((host, port))
tcp_server.listen(5)

while True:
    print "waiting for connection..."
    client, address = tcp_server.accept()
    print "a connection from: ", address

    while True:
        recv_data = client.recv(1024)
        if recv_data == 'exit':
            break
        print recv_data
    client.close()

tcp_server.close()
```

创建一个TCP客户端

```python
#!/usr/bin/env python

import socket

host = 'localhost'
port = 12345

tcp_client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
tcp_client.connect((host, port))

while True:
    input_data = raw_input('> ')
    tcp_client.send(input_data)
    if input_data == 'exit':
        break

tcp_client.close()
```

### 创建一个UDP服务器

```python
#!/usr/bin/env python

import socket

host = 'localhost'
port = 12345

udp_server = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
udp_server.bind((host, port))

while True:
    data, addr = udp_server.recvfrom(1024)
    print 'received from ', addr, ':', data

udp_server.close()
```

**创建一个UDP客户端**

```
#!/usr/bin/env python

import socket

host = 'localhost'
port = 12345

udp_client = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

while True:
    data = raw_input('> ')
    udp_client.sendto(data, (host, port))
    if data == 'exit':
        break

udp_client.close()
```

## SocketServer模块

SocketServer模块是标准库中很多服务器框架的基础，这些服务器框架包括BaseHTTPServer、SimpleHTTPServer、CGIHTTPServer、SimpleXMLRPCServer和DocXMLRPCServer，所有这些服务器框架都为基础服务器增加了特定的功能。

1. BaseServer
2. TCPServer/UDPServer：基本的网络同步TCP/UDP服务器
3. ThreadingTCPServer/ThreadingUDPServer：ThreadingMixIn和TCPServer/UDPServer的组合
4. StreamRequestHandler/DatagramRequestHandler：用于TCP/UDP服务器的服务处理工具

**使用TCPServer创建同步socket：**

```python
#!/usr/bin/env python

import socket
from SocketServer import TCPServer, StreamRequestHandler

class MyRequestHandler(StreamRequestHandler):
    def handle(self):
        print '...connected from: ', self.client_address
        print self.request.recv(1024)

tcp_server = TCPServer(('localhost', 12345), MyRequestHandler)
print "waiting for connection"
tcp_server.serve_forever()
```

**使用ThreadingTCPServer创建异步socket：**

```python
#!/usr/bin/env python

import socket
from SocketServer import ThreadingTCPServer, StreamRequestHandler
import time

class MyRequestHandler(StreamRequestHandler):
    def handle(self):
        print 'connection from: ', self.client_address
        print self.request.recv(1024)
        time.sleep(3)
        print 'hello'

tcp_server = ThreadingTCPServer(('localhost', 12345), MyRequestHandler)
tcp_server.serve_forever()
```
