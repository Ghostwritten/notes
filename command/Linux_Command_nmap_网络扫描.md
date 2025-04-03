

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a39e302a58654ce085cd5849deba5337.jpeg)


tags: 网络 




![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/15fd06247963e65764bf262a0e9a4c42.webp?x-image-process=image/format,png#pic_center)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2a3193faee3e9a945297787096b0430e.jpeg)

##  简介

[Nmap](https://nmap.org/)（“ Network Mapper ”）是一个用于网络探索和安全审计的开源工具。它旨在快速扫描大型网络，尽管它对单个主机运行良好。Nmap 以新颖的方式使用原始 IP 数据包来确定网络上可用的主机、这些主机提供的服务（应用程序名称和版本）、它们运行的​​操作系统（和操作系统版本）、数据包过滤器/防火墙的类型正在使用中，以及其他数十种特性。虽然 Nmap 通常用于安全审计，但许多系统和网络管理员发现它对日常任务非常有用，例如网络清单、管理服务升级计划以及监控主机或服务正常运行时间。

Nmap 的输出是扫描目标的列表，每个目标的补充信息取决于使用的选项。该信息中的关键是“有趣的端口表”。 该表列出了端口号和协议、服务名称和状态。状态为 open、filtered、 closed或unfiltered。 Open 表示目标机器上的应用程序正在侦听该端口上的连接/数据包。 Filtered 表示防火墙、过滤器或其他网络障碍正在阻塞端口，因此 Nmap 无法判断它是 open还是closed。 Closed 端口没有应用程序监听它们，尽管它们可以随时打开。端口分类为 unfiltered 当它们响应 Nmap 的探针时，但 Nmap 无法确定它们是打开还是关闭。Nmap 报告状态组合 open|filtered 和closed|filtered 当它无法确定两种状态中的哪一种描述一个端口时。当请求版本检测时，端口表还可以包括软件版本详细信息。当请求 IP 协议扫描 ( -sO) 时，Nmap 提供有关支持的 IP 协议而不是侦听端口的信息。

除了有趣的端口表之外，Nmap 还可以提供有关目标的更多信息，包括反向 DNS 名称、操作系统猜测、设备类型和 MAC 地址。

基本功能有三个：

 - 是扫描主机端口，嗅探所提供的网络服务
 - 探测一组主机是否在线
 - 推断主机所用的操作系统，到达主机经过的路由，系统已开放端口的软件版本

##  原理
![原理](https://i-blog.csdnimg.cn/blog_migrate/54fb225009a537379644bd49c82449a1.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9c0a84a0f29b68fd1e7d8568879ae12b.png)
##  端口状态
然许多端口扫描器传统上将所有端口归为打开或关闭状态，但 Nmap 更加精细。它将端口分为六种状态：`open`、 `closed`、`filtered`、 `unfiltered`、 `open|filtered`、 或 `closed|filtered`。

这些状态不是端口本身的固有属性，而是描述了 Nmap 如何看待它们。例如，来自与目标相同的网络的 Nmap 扫描可能会将端口显示135/tcp为打开，而同时使用来自 Internet 的相同选项的扫描可能会将该端口显示为filtered.

 - `open` ： 应用程序在该端口接收 TCP 连接或者 UDP 报文。
 - `closed` ：关闭的端口对于nmap也是可访问的， 它接收nmap探测报文并作出响应。但没有应用程序在其上监听。
 - `filtered` ：由于包过滤阻止探测报文到达端口，nmap无法确定该端口是否开放。过滤可能来自专业的防火墙设备，路由规则或者主机上的软件防火墙。
 - `unfiltered` ：未被过滤状态意味着端口可访问，但是nmap无法确定它是开放还是关闭。 只有用于映射防火墙规则集的 ACK扫描才会把端口分类到这个状态。
 - `open | filtered` ：无法确定端口是开放还是被过滤，开放的端口不响应就是一个例子。没有响应也可能意味着报文过滤器丢弃了探测报文或者它引发的任何反应。UDP，IP协议,FIN, Null等扫描会引起。
 - `closed|filtered`：（关闭或者被过滤的）：无法确定端口是关闭的还是被过滤的。


##  选项
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4974e9026111409f0a10c06c30f28c65.png)
##  基本扫描
| Goal                           | Command                                 | Example                                                   |
|--------------------------------|-----------------------------------------|-----------------------------------------------------------|
| Scan a Single Target           | nmap [target]                           | nmap 192.168.0.1                                          |
| Scan Multiple Targets          | nmap [target1, target2, etc             | nmap 192.168.0.1 192.168.0.2                              |
| Scan a Range of Hosts          | nmap [range of ip addresses]            | nmap 192.168.0.1-10                                       |
| Scan an Entire Subnet          | nmap [ip address/cdir]                  | nmap 192.168.0.1/24                                       |
| Scan Random Hosts              | nmap -iR [number]                       | nmap -iR 0                                                |
| Excluding Targets from a Scan  | nmap [targets] – exclude [targets]      | nmap 192.168.0.1/24 –exclude 192.168.0.100, 192.168.0.200 |
| Excluding Targets Using a List | nmap [targets] – excludefile [list.txt] | nmap 192.168.0.1/24 –excludefile notargets.txt            |
| 全面扫描     | nmap -A [target]                        | nmap -A 192.168.0.1                                       |
| Scan an IPv6 Target            | nmap -6 [target]                        | nmap -6 1aff:3c21:47b1:0000:0000:0000:0000:2afe           |

```bash
nmap  -A  192.168.0.12
```

扫描结果：目标主机MAC地址、设备类型、操作系统、中央处理单元、主机详细资料、网络距离等

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4c496c14e738725469658e8141bf8b8c.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d053ebf409f75ce3149d11119765cab1.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/559009fb97eb48cce814068b1feaf7dd.png)

##  发现扫描
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9729d8ff278ee714d7d0f36fbaffb56e.png)
| Goal                     | Command           | Example              |
|--------------------------|-------------------|----------------------|
| 执行仅 Ping 扫描 | nmap -sP [target] | nmap -sP 192.168.0.1 |
| Don’t Ping               | nmap -PN [target] | nmap -PN 192.168.0.1 |
| TCP SYN Ping             | nmap -PS [target] | nmap -PS 192.168.0.1 |
| TCP ACK Ping             | nmap -PA [target] | nmap -PA 192.168.0.1 |
| UDP Ping                 | nmap -PU [target] | nmap -PU 192.168.0.1 |
| SCTP INIT Ping           | nmap -PY [target] | nmap -PY 192.168.0.1 |
| ICMP Echo Ping           | nmap -PE [target] | nmap -PE 192.168.0.1 |
| ICMP Timestamp Ping      | nmap -PP [target] | nmap -PP 192.168.0.1 |
| CMP Address Mask Ping    | nmap -PM [target] | nmap -PM 192.168.0.1 |
| IP Protocol Ping         | nmap -PO [target] | nmap -PO 192.168.0.1 |


| ARP Ping                       | nmap -PR [target]                    | nmap -PR 192.168.0.1                        |
|--------------------------------|--------------------------------------|---------------------------------------------|
| Traceroute（跟踪路由）                     | nmap –traceroute [target]            | nmap –traceroute 192.168.0.1                |
| Force Reverse DNS Resolution（强制反向 DNS 解析）   | nmap -R [target]                     | nmap -R 192.168.0.1                         |
| Disable Reverse DNS Resolution（禁用反向 DNS 解析） | nmap -n [target]                     | nmap -n 192.168.0.1                         |
| Alternative DNS Lookup（替代 DNS 查找）         | nmap –system-dns [target]            | nmap –system-dns 192.168.0.1                |
| Manually Specify DNS Server(s)（手动指定 DNS 服务器） | nmap –dns-servers [servers] [target] | nmap –dns-servers 201.56.212.54 192.168.0.1 |
| Create a Host List （创建主机列表）            | nmap -sL [targets]                   | nmap -sL 192.168.0.1/24                     |

| Goal                      | Command                          | Example                            |
|---------------------------|----------------------------------|------------------------------------|
| TCP SYN Scan              | nmap -sS [target]                | nmap -sS 192.168.0.1               |
| TCP Connect Scan          | nmap -sT [target]                | nmap -sT 192.168.0.1               |
| UDP Scan                  | nmap -sU [target]                | nmap -sU 192.168.0.1               |
| TCP NULL Scan             | nmap -sN [target]                | nmap -sN 192.168.0.1               |
| TCP FIN Scan              | nmap -sF [target]                | nmap -sF 192.168.0.1               |
| Xmas Scan                 | nmap -sX [target]                | nmap -sX 192.168.0.1               |
| TCP ACK Scan              | nmap -sA [target]                | nmap -sA 192.168.0.1               |
| Custom TCP Scan           | nmap –scanflags [flags] [target] | nmap –scanflags SYNFIN 192.168.0.1 |
| IP Protocol Scan          | nmap -sO [target]                | nmap -sO 192.168.0.1               |
| Send Raw Ethernet Packets（发送原始以太网数据包） | nmap –send-eth [target]          | nmap –send-eth 192.168.0.1         |
| Send IP Packets           | nmap –send-ip [target]           | nmap –send-ip 192.168.0.1          |


### 禁用DNS名称解析
最后，您可以通过使用 -n 参数禁用反向 DNS 解析来加速 Nmap 扫描。如果您想扫描大型网络，这将非常有用。例如，要关闭上面提到的基本 ping 扫描的 DNS 解析，添加 -n：

```bash
nmap -sp -n 192.100.1.1/24
```

###  无ping扫描
通常用于防火墙禁止ping的情况下，它能确定正在运行的机器。

默认情况下，nmap只对正在运行的主机进行高强度的探测，如端口扫描，版本探测或者操作系统探测。

用-P0禁止主机发现会使nmap对每一个特定的目标IP地址进行所要求的扫描，这可以穿透防火墙，也可以避免被防火墙发现。

```bash
nmap -P0 192.168.0.12
```

##  端口扫描
| 目标       | 命令                                  | 例子                                                            |
|----------|-------------------------------------|---------------------------------------------------------------|
| 执行快速扫描   | nmap -F [目标]                        | nmap -F 192.168.0.1                                           |
| 扫描特定端口   | nmap -p [端口] [目标]                   | nmap -p 21-25,80,139,8080 192.168.1.1                         |
| 按名称扫描端口  | nmap -p [端口名称] [目标]                 | nmap -p ftp,http* 192.168.0.1                                 |
| 按协议扫描端口  | nmap -sU -sT -p U: [端口],T:[端口] [目标] | nmap -sU -sT -p U:53,111,137,T:21- 25,80,139,8080 192.168.0.1 |
| 扫描所有端口   | nmap -p '*' [目标]                    | nmap -p '*' 192.168.0.1                                       |
| 扫描顶部端口   | nmap –top-ports [数字] [目标]           | nmap –top-ports 10 192.168.0.1                                |
| 执行顺序端口扫描 | nmap -r [目标]                        | nmap -r 192.168.0.1                                           |

##  版本检测
| 目标           | 命令                      | 例子                                |
|--------------|-------------------------|-----------------------------------|
| 操作系统检测       | nmap -O [目标]            | nmap -O 192.168.0.1               |
| 提交 TCP/IP 指纹 | www.nmap.org/submit/    |                                   |
| 指纹           |                         |                                   |
| 尝试猜测未知操作系统   | nmap -O –osscan 猜测 [目标] | nmap -O –osscan-guess 192.168.0.1 |
| 服务版本检测       | nmap -sV [目标]           | nmap -sV 192.168.0.1              |
| 版本扫描故障排除     | nmap -sV –版本跟踪 [目标]     | nmap -sV –version-trace 192.168.0.1        |
| 执行 RPC 扫描    | nmap -sR [目标]           | nmap -sR 192.168.0.1              |


##  防火墙规避技术
| 目标        | 命令                                  | 例子                                 |
|-----------|-------------------------------------|------------------------------------|
| 增加数据包     | nmap -f [目标]                        | nmap -f 192.168.0.1                |
| 安抚特定的 MTU | nmap –mtu [MTU] [目标]                | nmap –mtu 32 192.168.0。            |
| 使用诱饵      | nmap -D RND:[数字] [目标]               | nmap -D RND:10 192.168.0.1         |
| 僵尸扫描      | nmap -sI [僵尸] [目标]                  | nmap -sI 192.168.0.38              |
| 手动指定源端口   | nmap –source-port [端口] [目标]         | nmap –source-port  10 192.168.0.1           |
| 附加随机数据    | nmap –data-length [大小] [目标]         | nmap –data-length 2 192.168.0.1           |
| 随机化目标扫描顺序 | nmap –randomize-hosts [目标]          | nmap --randomize-ho 192.168.0.1-20 |
| 欺骗 MAC 地址 | nmap –spoof-mac [MAC\|0\|vendor] [目标] | nmap –spoof-mac Cis 192.168.0.1    |
| 发送错误校验和   | nmap –badsum [目标]                   | nmap –badsum 192.168.0.1           |


##  故障排除和调试
| 目标         | 命令                      | 例子                       |
|------------|-------------------------|--------------------------|
| 获得帮助       | nmap -h                 | nmap -h                  |
| 显示 Nmap 版本 | nmap -V                 | nmap -V                  |
| 详细输出       | nmap -v [目标]            | nmap -v 192.168.0.1      |
| 调试         | nmap -d [目标]            | nmap -d 192.168.0.1      |
| 显示端口状态原因   | nmap –reason [目标]       | nmap –reason 192.168.0.1     |
| 仅显示打开的端口   | nmap –open [目标]         | nmap –open 192.168.0.1     |
| 跟踪数据包      | nmap –packet-trace [目标] | nmap –packet-trace  192.168.0.1  |
| 显示主机网络     | nmap –iflist            | nmap –iflist             |
| 指定网络接口     | nmap -e [接口] [目标]       | nmap -e eth0 192.168.0.1 |


##  NMAP 脚本
| 目标       | 命令                                        | 例子                                                |
|----------|-------------------------------------------|---------------------------------------------------|
| 执行单个脚本   | nmap –script [script.nse] [目标]            | nmap –script banner.nse 192.168.0.1                    |
| 执行多个脚本   | nmap –script [表达式] [目标]                   | nmap –script 'http-*' 192.168.0.1                 |
| 脚本类别     | 所有, 身份验证, 默认, 发现, 外部, 侵入性, 恶意软件, 安全, vuln |                                                   |
| 按类别执行脚本  | nmap –script [类别] [目标]                    | nmap –script ‘not intrusive’ 192.168.0.1                 |
| 执行多个脚本类别 | nmap – 脚本 [category1,category2,etc]       | nmap –script ‘default or safe’ 192.168.0.1                |
| 脚本疑难解答   | nmap –script [脚本] –脚本跟踪 [目标]              | nmap –script banner.nse –script-trace 192.168.0.1 |
| 更新脚本数据库  | nmap –脚本更新数据库                             | nmap –script-updatedb                                   |

参考：

 - [Nmap命令详解](https://www.jianshu.com/p/4030c99fcaee)
 - [NMAP Cheat Sheet](https://www.tutorialspoint.com/nmap-cheat-sheet)
 - [nmap命令-----基础用法](https://www.cnblogs.com/nmap/p/6232207.html)
 - [nmap 官方](https://nmap.org/book/toc.html)
 - [How to use Nmap to scan for open ports](https://www.techtarget.com/searchsecurity/feature/How-to-use-Nmap-to-scan-for-open-ports)
 - [官网 kali](https://www.kali.org/)
 - [How to Use Nmap: Commands and Tutorial Guide](https://www.varonis.com/blog/nmap-commands)
 - [8 Nmap Commands That You Should Know About](https://www.interviewbit.com/blog/nmap-commands/)
 - [python nmap网络扫描详解](https://ghostwritten.blog.csdn.net/article/details/106022722)
 - 
