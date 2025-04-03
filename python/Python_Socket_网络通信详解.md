#  Python 模块 Socket 网络通信



----
## 1. socket介绍
Python 提供了两个基本的 socket 模块：

 - Socket 它提供了标准的BSD Socket API。
 - SocketServer 它提供了服务器重心，可以简化网络服务器的开发。

## 2. Socket 类型
套接字格式：`socket(family, type[,protocal])` 使用给定的套接族，套接字类型，协议编号（默认为0）来创建套接字。
| socket 类型               | 描述                                                                                                            |
|-------------------------|---------------------------------------------------------------------------------------------------------------|
| socket\.AF\_UNIX        | 用于同一台机器上的进程通信（既本机通信）                                                                                          |
| socket\.AF\_INET        | 用于服务器与服务器之间的网络通信                                                                                              |
| socket\.AF\_INET6       | 基于IPV6方式的服务器与服务器之间的网络通信                                                                                       |
| socket\.SOCK\_STREAM    | 基于TCP的流式socket通信                                                                                              |
| socket\.SOCK\_DGRAM     | 基于UDP的数据报式socket通信                                                                                            |
| socket\.SOCK\_RAW       | 原始套接字，普通的套接字无法处理ICMP、IGMP等网络报文，而SOCK\_RAW可以；其次SOCK\_RAW也可以处理特殊的IPV4报文；此外，利用原始套接字，可以通过IP\_HDRINCL套接字选项由用户构造IP头 |
| socket\.SOCK\_SEQPACKET | 可靠的连续数据包服务                                                                                                    |
创建TCP Socket：

```bash
sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
```

创建UDP Socket：

```bash
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
```
## 3. Socket 函数

 - TCP发送数据时，已建立好TCP链接，所以不需要指定地址，而UDP是面向无连接的，每次发送都需要指定发送给谁。
 - 服务器与客户端不能直接发送列表，元素，字典等带有数据类型的格式，发送的内容必须是字符串数据。


### 3.1 服务器端 Socket 函数
| Socket 函数            | 描述                                                                        |
|----------------------|---------------------------------------------------------------------------|
| s\.bind\(address\)   | 将套接字绑定到地址，在AF\_INET下，以tuple\(host, port\)的方式传入，如s\.bind\(\(host, port\)\) |
| s\.listen\(backlog\) | 开始监听TCP传入连接，backlog指定在拒绝链接前，操作系统可以挂起的最大连接数，该值最少为1，大部分应用程序设为5就够用了          |
| s\.accept\(\)        | 接受TCP链接并返回（conn, address），其中conn是新的套接字对象，可以用来接收和发送数据，address是链接客户端的地址。    |

### 3.2 客户端 Socket 函数
| Socket 函数                 | 描述                                                                         |
|---------------------------|----------------------------------------------------------------------------|
| s\.connect\(address\)     | 链接到address处的套接字，一般address的格式为tuple\(host, port\)，如果链接出错，则返回socket\.error错误 |
| s\.connect\_ex\(address\) | 功能与s\.connect\(address\)相同，但成功返回0，失败返回errno的值                              |
### 3.3 公共 Socket 函数
| Socket 函数                                   | 描述                                                                                                              |
|---------------------------------------------|-----------------------------------------------------------------------------------------------------------------|
| s\.recv\(bufsize\[, flag\]\)                | 接受TCP套接字的数据，数据以字符串形式返回，buffsize指定要接受的最大数据量，flag提供有关消息的其他信息，通常可以忽略                                               |
| s\.send\(string\[, flag\]\)                 | 发送TCP数据，将字符串中的数据发送到链接的套接字，返回值是要发送的字节数量，该数量可能小于string的字节大小                                                       |
| s\.sendall\(string\[, flag\]\)              | 完整发送TCP数据，将字符串中的数据发送到链接的套接字，但在返回之前尝试发送所有数据。成功返回None，失败则抛出异常                                                     |
| s\.recvfrom\(bufsize\[, flag\]\)            | 接受UDP套接字的数据u，与recv\(\)类似，但返回值是tuple\(data, address\)。其中data是包含接受数据的字符串，address是发送数据的套接字地址                       |
| s\.sendto\(string\[, flag\], address\)      | 发送UDP数据，将数据发送到套接字，address形式为tuple\(ipaddr, port\)，指定远程地址发送，返回值是发送的字节数                                           |
| s\.close\(\)                                | 关闭套接字                                                                                                           |
| s\.getpeername\(\)                          | 返回套接字的远程地址，返回值通常是一个tuple\(ipaddr, port\)                                                                        |
| s\.getsockname\(\)                          | 返回套接字自己的地址，返回值通常是一个tuple\(ipaddr, port\)                                                                        |
| s\.setsockopt\(level, optname, value\)      | 设置给定套接字选项的值                                                                                                     |
| s\.getsockopt\(level, optname\[, buflen\]\) | 返回套接字选项的值                                                                                                       |
| s\.settimeout\(timeout\)                    | 设置套接字操作的超时时间，timeout是一个浮点数，单位是秒，值为None则表示永远不会超时。一般超时期应在刚创建套接字时设置，因为他们可能用于连接的操作，如s\.connect\(\)                  |
| s\.gettimeout\(\)                           | 返回当前超时值，单位是秒，如果没有设置超时则返回None                                                                                    |
| s\.fileno\(\)                               | 返回套接字的文件描述                                                                                                      |
| s\.setblocking\(flag\)                      | 如果flag为0，则将套接字设置为非阻塞模式，否则将套接字设置为阻塞模式（默认值）。非阻塞模式下，如果调用recv\(\)没有发现任何数据，或send\(\)调用无法立即发送数据，那么将引起socket\.error异常。 |
| s\.makefile\(\)                             | 创建一个与该套接字相关的文件                                                                                                  |

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c3589a5a96d310bd90440b58cbde8238.png)
## 4. Socket 编程思想
### 4.1 操作ip
```powershell
127.0.01 本地测试ip
0.0.0.0  局域网可用ip
192.168.1.0 网段
192.168.1.1  网关
192.168.1.255 广播地址
```

```powershell
In [1]: import socket
#获取主机名
In [2]: socket.gethostname()
Out[2]: 'localhost.localdomain'
# 通过主机名解析ip
In [3]: socket.gethostbyname('localhost.localdomain')
Out[3]: '127.0.0.1'

In [4]: socket.gethostbyname('localhost')
Out[4]: '127.0.0.1'

In [3]: socket.gethostbyname('www.baidu.com')
Out[3]: '180.101.49.11'

#十进制转换二进制
In [5]: socket.inet_aton('192.168.1.190')
Out[5]: '\xc0\xa8\x01\xbe'

#二进制转换十进制
In [8]: socket.inet_ntoa('\xc0\xa8\x01\xbe')
Out[8]: '192.168.1.190'

#十进制转换二进制
In [11]: socket.inet_pton(socket.AF_INET, '192.168.1.190')
Out[11]: '\xc0\xa8\x01\xbe'

#二进制转换十进制
In [15]: socket.inet_ntop(socket.AF_INET, '\xc0\xa8\x01\xbe')
Out[15]: '192.168.1.190'

```
### 4.2 操作端口
应用层端口：1--65535
众所周知的服务端口：1--255 
系统端口：256--1023


```powershell
In [16]: socket.getservbyname('mysql')
Out[16]: 3306

In [17]: socket.getservbyname('http')
Out[17]: 80
```


### 4.3 TCP 服务器 
1、创建套接字，绑定套接字到本地IP与端口

```python
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.bind()
```

2、开始监听链接

```python
s.listen()
```

3、进入循环，不断接受客户端的链接请求

```python
While True:
    s.accept()
```

4、接收客户端传来的数据，并且发送给对方发送数据

```python
s.recv()
s.sendall()
```

5、传输完毕后，关闭套接字

```python
s.close()
```

### 4.4 TCP 客户端 
1、创建套接字并链接至远端地址

```python
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect()
```

2、链接后发送数据和接收数据

```python
s.sendall()
s.recv()
```

3、传输完毕后，关闭套接字
## 5. 练习

### 5.1 示例1
服务器端代码

```python
import socket

HOST = '192.168.1.100'
PORT = 8001

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.bind((HOST, PORT))
s.listen(5)

print 'Server start at: %s:%s' %(HOST, PORT)
print 'wait for connection...'

while True:
    conn, addr = s.accept()
    print 'Connected by ', addr

    while True:
        data = conn.recv(1024)
        print data

        conn.send("server received you message.")

# conn.close()
```



客户端代码

```python
import socket
HOST = '192.168.1.100'
PORT = 8001

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((HOST, PORT))

while True:
    cmd = raw_input("Please input msg:")
    s.send(cmd)
    data = s.recv(1024)
    print data

    #s.close()
```

```powershell
$ python soc2.py
Please input msg:ls
server received you message.
Please input msg:hello
server received you message.
```
```powershell
$ python test1.py
Server start at: 192.168.211.15:80
wait for connection...
Connected by  ('192.168.211.15', 44079)
ls
hello

```

### 5.2 示例2
#### 打印程序服务端
server2.py

```python
#!/usr/bin/env python3

import socket

HOST = '127.0.0.1'  # 标准的回环地址 (localhost)
PORT = 65432        # 监听的端口 (非系统级的端口: 大于 1023)

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.bind((HOST, PORT))
    s.listen()
    conn, addr = s.accept()
    with conn:
        print('Connected by', addr)
        while True:
            data = conn.recv(1024)
            if not data:
                break
            conn.sendall(data)
```
解说：

 - `socket.socket()` 创建了一个 socket 对象，并且支持 context manager type，你可以使用 with语句，这样你就不用再手动调用 s.close() 来关闭 socket 了。
 

```python
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    pass  # Use the socket object without calling s.close().
```

 - `bind()` 用来关联 socket 到指定的网络接口（IP 地址）和端口号，bind() 方法的入参取决于 socket地址族，在这个例子中我们使用了 socket.AF_INET (IPv4)，它将返回两个元素的元组：(host, port)。host可以是主机名称、IP 地址、空字符串，如果使用 IP 地址，host 就应该是 IPv4 格式的字符串，127.0.0.1 是标准的IPv4 回环地址，只有主机上的进程可以连接到服务器，如果你传了空字符串，服务器将接受本机所有可用的 IPv4 地址 端口号应该是1-65535 之间的整数（0是保留的），这个整数就是用来接受客户端链接的 TCP 端口号，如果端口号小于1024，有的操作系统会要求管理员权限。
 
 - `listen()` 方法调用使服务器可以接受连接请求，这使它成为一个「监听中」的 socket
listen() 方法有一个 backlog 参数。它指定在拒绝新的连接之前系统将允许使用的 未接受的连接 数量。从 Python 3.5 开始，这是可选参数。如果不指定，Python 将取一个默认值。如果你的服务器需要同时接收很多连接请求，增加 backlog 参数的值可以加大等待链接请求队列的长度，最大长度取决于操作系统。比如在 Linux 下，参考 [/proc/sys/net/core/somaxconn](https://serverfault.com/questions/518862/will-increasing-net-core-somaxconn-make-a-difference/519152)
 - `accept()` 方法阻塞并等待传入连接。当一个客户端连接时，它将返回一个新的 socket 对象，对象中有表示当前连接的 conn和一个由主机、端口号组成的 IPv4/v6 连接的元组，这里必须要明白我们通过调用 accept() 方法拥有了一个新的 socket对象。这非常重要，因为你将用这个 socket 对象和客户端进行通信。和监听一个 socket 不同的是后者只用来授受新的连接请求。
 - 从 `accept()` 获取客户端 socket 连接对象 conn 后，使用一个无限 while 循环来阻塞调用`conn.recv()`，无论客户端传过来什么数据都会使用 `conn.sendall()` 打印出来，如果 conn.recv()方法返回一个空 byte 对象（b''），然后客户端关闭连接，循环结束，with 语句和 conn 一起使用时，通信结束的时候会自动关闭socket 链接。

#### 打印程序客户端

```powershell
#!/usr/bin/env python3

import socket

HOST = '127.0.0.1'  # 服务器的主机名或者 IP 地址
PORT = 65432        # 服务器使用的端口

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.connect((HOST, PORT))
    s.sendall(b'Hello, world')
    data = s.recv(1024)

print('Received', repr(data))
```
运行：
第一步

```powershell
$ python3.8 server1.py
```

第二步：

```powershell
$ python3.8 client2.py 
Received b'Hello, world'
```


第三步

```powershell
$ python3.8 server1.py
Connected by ('127.0.0.1', 47816)
```
参考：

 - [Python 中的 Socket 编程（指南）](https://keelii.com/2018/09/24/socket-programming-in-python/)
 - [Python Socket 编程详细介绍](https://gist.github.com/kevinkindom/108ffd675cb9253f8f71)

