


---------------
## 1. 回顾
DNS 可以提供域名和 IP 地址的映射关系，也是一种常用的全局负载均衡（GSLB）实现方法。

通常，需要暴露到公网的服务，都会绑定一个域名，既方便了人们记忆，也避免了后台服务 IP 地址的变更影响到用户。不过要注意，DNS 解析受到各种网络状况的影响，性能可能不稳定。比如公网延迟增大，缓存过期导致要重新去上游服务器请求，或者流量高峰时 DNS 服务器性能不足等，都会导致 DNS 响应的延迟增大。

此时，可以借助 nslookup 或者 dig 的调试功能，分析 DNS 的解析过程，再配合 ping 等工具调试 DNS 服务器的延迟，从而定位出性能瓶颈。通常，你可以用缓存、预取、HTTPDNS 等方法，优化 DNS 的性能。

上一节我们用到的 ping，是一个最常用的测试服务延迟的工具。很多情况下，ping 可以帮我们定位出延迟问题，不过有时候， ping 本身也会出现意想不到的问题。这时，就需要我们抓取 ping 命令执行时收发的网络包，然后分析这些网络包，进而找出问题根源。

## 2. 包分析
[tcpdump](https://ghostwritten.blog.csdn.net/article/details/117995196) 和 Wireshark 就是最常用的网络抓包和分析工具，更是分析网络性能必不可少的利器。

 - tcpdump 仅支持命令行格式使用，常用在服务器中抓取和分析网络包。
 - Wireshark 除了可以抓包外，还提供了强大的图形界面和汇总分析工具，在分析复杂的网络情景时，尤为简单和实用。


因而，在实际分析网络性能时，先用 tcpdump 抓包，后用 Wireshark 分析，也是一种常用的方法。今天，我就带你一起看看，怎么使用 tcpdump 和 Wireshark ，来分析网络的性能问题。

## 3. 案例准备

 - Ubuntu 18.04
 - 机器配置：2 CPU，8GB 内存
 - 预先安装 tcpdump、Wireshark 等工具，如：

```bash
# Ubuntu
apt-get install tcpdump wireshark

# CentOS
yum install -y tcpdump wireshark
```
由于 Wireshark 的图形界面，并不能通过 SSH 使用，所以我推荐你在本地机器（比如 Windows）中安装。你可以到 [https://www.wireshark.org/](https://www.wireshark.org/) 下载并安装 Wireshark。

##  4. 再探 ping

```bash
# ping 3 次（默认每次发送间隔1秒）
# 假设DNS服务器还是上一期配置的114.114.114.114
$ ping -c3 geektime.org
PING geektime.org (35.190.27.188) 56(84) bytes of data.
64 bytes from 35.190.27.188 (35.190.27.188): icmp_seq=1 ttl=43 time=36.8 ms
64 bytes from 35.190.27.188 (35.190.27.188): icmp_seq=2 ttl=43 time=31.1 ms
64 bytes from 35.190.27.188 (35.190.27.188): icmp_seq=3 ttl=43 time=31.2 ms

--- geektime.org ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 11049ms
rtt min/avg/max/mdev = 31.146/33.074/36.809/2.649 ms
```
不过要注意，假如你运行时发现 ping 很快就结束了，那就执行下面的命令，再重试一下。至于这条命令的含义，稍后我们再做解释。

```bash
# 禁止接收从DNS服务器发送过来并包含googleusercontent的包
$ iptables -I INPUT -p udp --sport 53 -m string --string googleusercontent --algo bm -j DROP
```
根据 ping 的输出，你可以发现，`geektime.org` 解析后的 IP 地址是 `35.190.27.188`，而后三次 ping 请求都得到了响应，延迟（RTT）都是 30ms 多一点。但汇总的地方，就有点儿意思了。3 次发送，收到 3 次响应，没有丢包，但三次发送和接受的总时间居然超过了 11s（11049ms），这就有些不可思议了吧。会想起上一节的 DNS 解析问题，你可能会怀疑，这可能是 DNS 解析缓慢的问题。但到底是不是呢？再回去看 ping 的输出，三次 ping 请求中，用的都是 IP 地址，说明 ping 只需要在最开始运行时，解析一次得到 IP，后面就可以只用 IP 了。

我们再用 nslookup 试试。在终端中执行下面的 nslookup 命令，注意，这次我们同样加了 time 命令，输出 nslookup 的执行时间：


```bash
$ time nslookup geektime.org
Server:    114.114.114.114
Address:  114.114.114.114#53

Non-authoritative answer:
Name:  geektime.org
Address: 35.190.27.188


real  0m0.044s
user  0m0.006s
sys  0m0.003s
```
可以看到，域名解析还是很快的，只需要 44ms，显然比 11s 短了很多。到这里，再往后该怎么分析呢？其实，这时候就可以用 tcpdump 抓包，查看 ping 在收发哪些网络包。我们再打开另一个终端（终端二），SSH 登录案例机器后，执行下面的命令：

```bash
$ tcpdump -nn udp port 53 or host 35.190.27.188
```
当然，你可以直接用 tcpdump 不加任何参数来抓包，但那样的话，就可能抓取到很多不相干的包。由于我们已经执行过 ping 命令，知道了 geekbang.org 的 IP 地址是 35.190.27.188，也知道 ping 命令会执行 DNS 查询。所以，上面这条命令，就是基于这个规则进行过滤。我来具体解释一下这条命令。

 - -nn ，表示不解析抓包中的域名（即不反向解析）、协议以及端口号。
 - udp port 53 ，表示只显示 UDP 协议的端口号（包括源端口和目的端口）为 53 的包。
 - host 35.190.27.188 ，表示只显示 IP 地址（包括源地址和目的地址）为 35.190.27.188 的包。
 - 这两个过滤条件中间的“ or ”，表示或的关系，也就是说，只要满足上面两个条件中的任一个，就可以展示出来。

接下来，回到终端一，执行相同的 ping 命令：

```bash
$ ping -c3 geektime.org
...
--- geektime.org ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 11095ms
rtt min/avg/max/mdev = 81.473/81.572/81.757/0.130 ms
```
命令结束后，再回到终端二中，查看 tcpdump 的输出：

```bash
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
14:02:31.100564 IP 172.16.3.4.56669 > 114.114.114.114.53: 36909+ A? geektime.org. (30)
14:02:31.507699 IP 114.114.114.114.53 > 172.16.3.4.56669: 36909 1/0/0 A 35.190.27.188 (46)
14:02:31.508164 IP 172.16.3.4 > 35.190.27.188: ICMP echo request, id 4356, seq 1, length 64
14:02:31.539667 IP 35.190.27.188 > 172.16.3.4: ICMP echo reply, id 4356, seq 1, length 64
14:02:31.539995 IP 172.16.3.4.60254 > 114.114.114.114.53: 49932+ PTR? 188.27.190.35.in-addr.arpa. (44)[添加链接描述](https://www.ietf.org/rfc/rfc1035.txt)
14:02:36.545104 IP 172.16.3.4.60254 > 114.114.114.114.53: 49932+ PTR? 188.27.190.35.in-addr.arpa. (44)
14:02:41.551284 IP 172.16.3.4 > 35.190.27.188: ICMP echo request, id 4356, seq 2, length 64
14:02:41.582363 IP 35.190.27.188 > 172.16.3.4: ICMP echo reply, id 4356, seq 2, length 64
14:02:42.552506 IP 172.16.3.4 > 35.190.27.188: ICMP echo request, id 4356, seq 3, length 64
14:02:42.583646 IP 35.190.27.188 > 172.16.3.4: ICMP echo reply, id 4356, seq 3, length 64
```
这次输出中，前两行，表示 tcpdump 的选项以及接口的基本信息；从第三行开始，就是抓取到的网络包的输出。这些输出的格式，都是 `时间戳` `协议` `源地址`.`源端口` `> 目的地址`.`目的端口` `网络包详细信息`（这是最基本的格式，可以通过选项增加其他字段）。

前面的字段，都比较好理解。但网络包的详细信息，本身根据协议的不同而不同。所以，要理解这些网络包的详细含义，就要对常用网络协议的基本格式以及交互原理，有基本的了解。当然，实际上，这些内容都会记录在 IETF（ 互联网工程任务组）发布的 [RFC](https://tools.ietf.org/rfc/index)（请求意见稿）中。比如，第一条就表示，从本地 IP 发送到 114.114.114.114 的 A 记录查询请求，它的报文格式记录在 RFC1035 中，你可以点击这里查看。在这个 tcpdump 的输出中，

 - **36909+ 表示查询标识值，它也会出现在响应中，加号表示启用递归查询**。
 - A? 表示查询 A 记录。
 - geektime.org. 表示待查询的域名。
 - 30 表示报文长度。

接下来的一条，则是从 114.114.114.114 发送回来的 DNS 响应——域名 geektime.org. 的 A 记录值为 35.190.27.188。
第三条和第四条，是 ICMP echo request 和 ICMP echo reply，响应包的时间戳 14:02:31.539667，减去请求包的时间戳 14:02:31.508164 ，就可以得到，这次 ICMP 所用时间为 30ms。这看起来并没有问题。

**但随后的两条反向地址解析 PTR 请求，就比较可疑了。因为我们只看到了请求包，却没有应答包。仔细观察它们的时间，你会发现，这两条记录都是发出后 5s 才出现下一个网络包，两条 PTR 记录就消耗了 10s。**

再往下看，最后的四个包，则是两次正常的 ICMP 请求和响应，根据时间戳计算其延迟，也是 30ms。到这里，**其实我们也就找到了 ping 缓慢的根源，正是两次 PTR 请求没有得到响应而超时导致的。PTR 反向地址解析的目的，是从 IP 地址反查出域名，但事实上，并非所有 IP 地址都会定义 PTR 记录，所以 PTR 查询很可能会失败。**

所以，在你使用 ping 时，如果发现结果中的延迟并不大，而 ping 命令本身却很慢，不要慌，有可能是背后的 PTR 在搞鬼。知道问题后，解决起来就比较简单了，只要禁止 PTR 就可以。还是老路子，执行 man ping 命令，查询使用手册，就可以找出相应的方法，即加上 -n 选项禁止名称解析。比如，我们可以在终端中执行如下命令：


```bash
$ ping -n -c3 geektime.org
PING geektime.org (35.190.27.188) 56(84) bytes of data.
64 bytes from 35.190.27.188: icmp_seq=1 ttl=43 time=33.5 ms
64 bytes from 35.190.27.188: icmp_seq=2 ttl=43 time=39.0 ms
64 bytes from 35.190.27.188: icmp_seq=3 ttl=43 time=32.8 ms

--- geektime.org ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 32.879/35.160/39.030/2.755 ms
```
你可以发现，现在只需要 2s 就可以结束，比刚才的 11s 可是快多了。到这里， 我就带你一起使用 tcpdump ，解决了一个最常见的 ping 工作缓慢的问题。案例最后，如果你在开始时，执行了 iptables 命令，那也不要忘了删掉它：

```bash
$ iptables -D INPUT -p udp --sport 53 -m string --string googleusercontent --algo bm -j DROP
```
过，删除后你肯定还有疑问，明明我们的案例跟 Google 没啥关系，为什么要根据 googleusercontent ，这个毫不相关的字符串来过滤包呢？实际上，如果换一个 DNS 服务器，就可以用 PTR 反查到 35.190.27.188 所对应的域名：


```bash
 $ nslookup -type=PTR 35.190.27.188 8.8.8.8
Server:  8.8.8.8
Address:  8.8.8.8#53
Non-authoritative answer:
188.27.190.35.in-addr.arpa  name = 188.27.190.35.bc.googleusercontent.com.
Authoritative answers can be found from:
```
你看，虽然查到了 PTR 记录，但结果并非 geekbang.org，而是 `188.27.190.35.bc.googleusercontent.com`。其实，这也是为什么，案例开始时将包含 googleusercontent 的丢弃后，ping 就慢了。因为 iptables ，实际上是把 PTR 响应给丢了，所以会导致 PTR 请求超时。tcpdump 可以说是网络性能分析最有效的利器。接下来，我再带你一起看看 tcpdump 的更多使用方法。

## 5. tcpdump

我们知道，tcpdump 也是最常用的一个网络分析工具。它基于 [libpcap](https://www.tcpdump.org/)  ，利用内核中的 `AF_PACKET` 套接字，抓取网络接口中传输的网络包；并提供了强大的过滤规则，帮你从大量的网络包中，挑出最想关注的信息。

tcpdump 为你展示了每个网络包的详细细节，这就要求，在使用前，你必须要对网络协议有基本了解。而要了解网络协议的详细设计和实现细节， [RFC](https://www.rfc-editor.org/rfc-index.html) 当然是最权威的资料。

不过，RFC 的内容，对初学者来说可能并不友好。如果你对网络协议还不太了解，推荐你去学`《TCP/IP 详解》`，特别是第一卷的 TCP/IP 协议族。这是每个程序员都要掌握的核心基础知识。

再回到 tcpdump 工具本身，它的基本使用方法，还是比较简单的，也就是 `tcpdump [选项] [过滤表达式]`。当然，选项和表达式的外面都加了中括号，表明它们都是可选的。

查看 [tcpdump](https://www.tcpdump.org/manpages/tcpdump.1.html) 的 手册  ，以及 [pcap-filter](https://www.tcpdump.org/manpages/pcap-filter.7.html) 的手册，你会发现，tcpdump 提供了大量的选项以及各式各样的过滤表达式。不过不要担心，只需要掌握一些常用选项和过滤表达式，就可以满足大部分场景的需要了。

为了帮你更快上手 tcpdump 的使用，我在这里也帮你整理了一些最常见的用法，并且绘制成了表格，你可以参考使用。首先，来看一下常用的几个选项。在上面的 ping 案例中，**我们用过  -nn 选项，表示不用对 IP 地址和端口号进行名称解析**。其他常用选项，我用下面这张表格来解释。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/82913ecaaaad40685cdd5f070df53d5d.png)
接下来，我们再来看常用的过滤表达式。刚刚用过的是  udp port 53 or host 35.190.27.188 ，表**示抓取 DNS 协议的请求和响应包，以及源地址或目的地址为 35.190.27.188 的包**。其他常用的过滤选项，我也整理成了下面这个表格。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/45a94ec3ba1851388e856d56c81861b7.png)
最后，再次强调 tcpdump 的输出格式，我在前面已经介绍了它的基本格式：

```bash
时间戳 协议 源地址.源端口 > 目的地址.目的端口 网络包详细信息
```
中，网络包的详细信息取决于协议，不同协议展示的格式也不同。所以，更详细的使用方法，还是需要你去查询 tcpdump 的 man 手册（执行 [man tcpdump](https://www.tcpdump.org/manpages/tcpdump.1.html) 也可以得到）。不过，讲了这么多，你应该也发现了。tcpdump 虽然功能强大，可是输出格式却并不直观。特别是，当系统中网络包数比较多（比如 PPS 超过几千）的时候，你想从 tcpdump 抓取的网络包中分析问题，实在不容易。对比之下，Wireshark 则通过图形界面，以及一系列的汇总分析工具，提供了更友好的使用界面，让你可以用更快的速度，摆平网络性能问题。接下来，我们就详细来看看它。

##  6. Wireshark
Wireshark 也是最流行的一个网络分析工具，它最大的好处就是提供了跨平台的图形界面。跟 tcpdump 类似，Wireshark 也提供了强大的过滤规则表达式，同时，还内置了一系列的汇总分析工具。比如，拿刚刚的 ping 案例来说，你可以执行下面的命令，把抓取的网络包保存到 `ping.pcap` 文件中：


```bash
$ tcpdump -nn udp port 53 or host 35.190.27.188 -w ping.pcap
```
接着，把它拷贝到你安装有 Wireshark 的机器中，比如你可以用 scp 把它拷贝到本地来：

```bash
$ scp host-ip/path/ping.pcap .
```
然后，再用 Wireshark 打开它。打开后，你就可以看到下面这个界面：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/776457e30527737e15873efea3e19cf8.png)
从 Wireshark 的界面里，你可以发现，它不仅以更规整的格式，展示了各个网络包的头部信息；还用了不同颜色，展示 `DNS 和 ICMP` 这两种不同的协议。你也可以一眼看出，中间的两条 PTR 查询并没有响应包。

接着，在网络包列表中选择某一个网络包后，在其下方的网络包详情中，你还可以看到，这个包在协议栈各层的详细信息。比如，以编号为 5 的 PTR 包为例：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/45243715e78409c83f1c2bd6ef3217cd.png)
你可以看到，IP 层（Internet Protocol）的源地址和目的地址、传输层的 UDP 协议（Uder Datagram Protocol）、应用层的 DNS 协议（Domain Name System）的概要信息。继续点击每层左边的箭头，就可以看到该层协议头的所有信息。

比如点击 DNS 后，就可以看到 Transaction ID、Flags、Queries 等 DNS 协议各个字段的数值以及含义。

当然，Wireshark 的功能远不止如此。接下来我再带你一起，看一个 HTTP 的例子，并理解 TCP 三次握手和四次挥手的工作原理。

这个案例我们将要访问的是 http://example.com/ 。进入终端一，执行下面的命令，首先查出 example.com 的 IP。然后，执行 tcpdump 命令，过滤得到的 IP 地址，并将结果保存到 web.pcap 中。


```bash
$ dig +short example.com
93.184.216.34
$ tcpdump -nn host 93.184.216.34 -w web.pcap
```
实际上，你可以在 host 表达式中，直接使用域名，即 `tcpdump -nn host example.com -w web.pcap`。

```bash
$ curl http://example.com
```
最后，再回到终端一，按下 Ctrl+C 停止 tcpdump，并把得到的 web.pcap 拷贝出来。使用 Wireshark 打开 web.pcap 后，你就可以在 Wireshark 中，看到如下的界面：

![最后，再回到终端一，按下 Ctrl+C 停止 tcpdump，并把得到的 web.pcap 拷贝出来。使用 Wireshark 打开 web.pcap 后，你就可以在 Wireshark 中，看到如下的界面：](https://i-blog.csdnimg.cn/blog_migrate/bad366877310489d72999b3c6f2a4c77.png)
由于 HTTP 基于 TCP ，所以你最先看到的三个包，分别是 TCP 三次握手的包。接下来，中间的才是 HTTP 请求和响应包，而最后的三个包，则是 TCP 连接断开时的三次挥手包。

从菜单栏中，点击 Statistics -> Flow Graph，然后，在弹出的界面中的 Flow type 选择 TCP Flows，你可以更清晰的看到，整个过程中 TCP 流的执行过程：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4c0a44f75db477129f4324db8574f1c2.png)
这其实跟各种教程上讲到的，TCP 三次握手和四次挥手很类似，作为对比， 你通常看到的 TCP 三次握手和四次挥手的流程，基本是这样的：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3324110ac4a73ae1e0c91541a25b1aac.png)
不过，对比这两张图，你会发现，这里抓到的包跟上面的四次挥手，并不完全一样，实际挥手过程只有三个包，而不是四个。其实，之所以有三个包，是因为服务器端收到客户端的 FIN 后，服务器端同时也要关闭连接，这样就可以把 ACK 和 FIN 合并到一起发送，节省了一个包，变成了“三次挥手”。

而通常情况下，服务器端收到客户端的 FIN 后，很可能还没发送完数据，所以就会先回复客户端一个 ACK 包。稍等一会儿，完成所有数据包的发送后，才会发送 FIN 包。这也就是四次挥手了。抓包后， Wireshark 中就会显示下面这个界面（原始网络包来自 Wireshark [TCP 4-times close](https://wiki.wireshark.org/TCP%204-times%20close) 示例，你可以点击 这里 下载）：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dc883fff49588a4a8d6671526058183c.png)

