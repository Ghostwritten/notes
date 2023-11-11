#  Linux Command netstat 统计网络数据
tags: 监控, 分析, 网络




-----

## 1. 介绍
netstat命令用于显示与IP、TCP、UDP和ICMP协议相关的统计数据，一般用于检验本机各端口的网络连接情况。netstat是在内核中访问网络及相关信息的程序，它能提供TCP连接，TCP和UDP监听，进程内存管理的相关报告。

如果你的计算机有时候接收到的数据报导致出错数据或故障，你不必感到奇怪，TCP/IP可以容许这些类型的错误，并能够自动重发数据报。但如果累计的出错情况数目占到所接收的IP数据报相当大的百分比，或者它的数目正迅速增加，那么你就应该使用netstat查一查为什么会出现这些情况了。

## 2. 格式

```bash
netstat [-acCeFghilMnNoprstuvVwx][-A<网络类型>][--ip]
```

## 3. 参数

```bash
-a或–all 显示所有连线中的Socket。

-A<网络类型>或–<网络类型> 列出该网络类型连线中的相关地址。

-c或–continuous 持续列出网络状态。

-C或–cache 显示路由器配置的快取信息。

-e或–extend 显示网络其他相关信息。

-F或–fib 显示FIB。

-g或–groups 显示多重广播功能群组组员名单。

-h或–help 在线帮助。

-i或–interfaces 显示网络界面信息表单。

-l或–listening 显示监控中的服务器的Socket。

-M或–masquerade 显示伪装的网络连线。

-n或–numeric 直接使用IP地址，而不通过域名服务器。

-N或–netlink或–symbolic 显示网络硬件外围设备的符号连接名称。

-o或–timers 显示计时器。

-p或–programs 显示正在使用Socket的程序识别码和程序名称。

-r或–route 显示Routing Table。

-s或–statistice 显示网络工作信息统计表。

-t或–tcp 显示TCP传输协议的连线状况。

-u或–udp 显示UDP传输协议的连线状况。

-v或–verbose 显示指令执行过程。

-V或–version 显示版本信息。

-w或–raw 显示RAW传输协议的连线状况。

-x或–unix 此参数的效果和指定”-A unix”参数相同。

–ip或–inet 此参数的效果和指定”-A inet”参数相同。
```
## 4. 实例
### 实例1：无参数使用

```bash
$ netstat
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State      
tcp        0    268 192.168.120.204:ssh         10.2.0.68:62420             ESTABLISHED 
udp        0      0 192.168.120.204:4371        10.58.119.119:domain        ESTABLISHED 
Active UNIX domain sockets (w/o servers)
Proto RefCnt Flags       Type       State         I-Node Path
unix  2      [ ]         DGRAM                    1491   @/org/kernel/udev/udevd
unix  4      [ ]         DGRAM                    7337   /dev/log
unix  2      [ ]         DGRAM                    708823 
unix  2      [ ]         DGRAM                    7539   
unix  3      [ ]         STREAM     CONNECTED     7287   
unix  3      [ ]         STREAM     CONNECTED     7286   
```
说明：

从整体上看，netstat的输出结果可以分为两个部分：

 - 一个是Active Internet connections，称为有源TCP连接，其中"Recv-Q"和"Send-Q"指的是接收队列和发送队列。这些数字一般都应该是0。如果不是则表示软件包正在队列中堆积。这种情况只能在非常少的情况见到。
 - 另一个是Active UNIX domain sockets，称为有源Unix域套接口(和网络套接字一样，但是只能用于本机通信，性能可以提高一倍)。

Proto显示连接使用的协议,RefCnt表示连接到本套接口上的进程号,Types显示套接口的类型,State显示套接口当前的状态,Path表示连接到套接口的其它进程使用的路径名。

套接口类型：

```bash
-t ：TCP

-u ：UDP

-raw ：RAW类型

--unix ：UNIX域类型

--ax25 ：AX25类型

--ipx ：ipx类型

--netrom ：netrom类型
```

状态说明：

```bash
LISTEN：侦听来自远方的TCP端口的连接请求

SYN-SENT：再发送连接请求后等待匹配的连接请求（如果有大量这样的状态包，检查是否中招了）

SYN-RECEIVED：再收到和发送一个连接请求后等待对方对连接请求的确认（如有大量此状态，估计被flood攻击了）

ESTABLISHED：代表一个打开的连接

FIN-WAIT-1：等待远程TCP连接中断请求，或先前的连接中断请求的确认

FIN-WAIT-2：从远程TCP等待连接中断请求

CLOSE-WAIT：等待从本地用户发来的连接中断请求

CLOSING：等待远程TCP对连接中断的确认

LAST-ACK：等待原来的发向远程TCP的连接中断请求的确认（不是什么好东西，此项出现，检查是否被攻击）

TIME-WAIT：等待足够的时间以确保远程TCP接收到连接中断请求的确认

CLOSED：没有任何连接状态
```
### 实例2：列出所有端口

```bash
$  netstat -a
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address               Foreign Address             State      
tcp        0      0 localhost:smux              *:*                         LISTEN      
tcp        0      0 *:svn                       *:*                         LISTEN      
tcp        0      0 *:ssh                       *:*                         LISTEN      
tcp        0    284 192.168.120.204:ssh         10.2.0.68:62420             ESTABLISHED 
udp        0      0 localhost:syslog            *:*                                     
udp        0      0 *:snmp                      *:*                                     
Active UNIX domain sockets (servers and established)
Proto RefCnt Flags       Type       State         I-Node Path
unix  2      [ ACC ]     STREAM     LISTENING     708833 /tmp/ssh-yKnDB15725/agent.15725
unix  2      [ ACC ]     STREAM     LISTENING     7296   /var/run/audispd_events
unix  2      [ ]         DGRAM                    1491   @/org/kernel/udev/udevd
unix  4      [ ]         DGRAM                    7337   /dev/log
unix  2      [ ]         DGRAM                    708823 
unix  2      [ ]         DGRAM                    7539   
unix  3      [ ]         STREAM     CONNECTED     7287   
unix  3      [ ]         STREAM     CONNECTED     7286  
```
说明：

显示一个所有的有效连接信息列表，包括已建立的连接（ESTABLISHED），也包括监听连接请（LISTENING）的那些连接。

### 实例3：显示当前UDP连接状况

```bash
$ netstat -nu
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State      
udp        0      0 ::ffff:192.168.12:53392     ::ffff:192.168.9.120:10000  ESTABLISHED 
udp        0      0 ::ffff:192.168.12:56723     ::ffff:192.168.9.120:10000  ESTABLISHED 
udp        0      0 ::ffff:192.168.12:56480     ::ffff:192.168.9.120:10000  ESTABLISHED 
udp        0      0 ::ffff:192.168.12:58154     ::ffff:192.168.9.120:10000  ESTABLISHED 
udp        0      0 ::ffff:192.168.12:44227     ::ffff:192.168.9.120:10000  ESTABLISHED 
udp        0      0 ::ffff:192.168.12:36954     ::ffff:192.168.9.120:10000  ESTABLISHED 
udp        0      0 ::ffff:192.168.12:53984     ::ffff:192.168.9.120:10000  ESTABLISHED 
udp        0      0 ::ffff:192.168.12:57703     ::ffff:192.168.9.120:10000  ESTABLISHED 
udp        0      0 ::ffff:192.168.12:53613     ::ffff:192.168.9.120:10000  ESTABLISHED 
```
### 实例4：显示UDP端口号的使用情况

```bash
$ netstat -apu
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
udp        0      0 0.0.0.0:890             0.0.0.0:*                           726/rpcbind         
udp        0      0 0.0.0.0:sunrpc          0.0.0.0:*                           1/systemd           
udp        0      0 localhost:323           0.0.0.0:*                           760/chronyd         
udp6       0      0 [::]:890                [::]:*                              726/rpcbind         
udp6       0      0 [::]:sunrpc             [::]:*                              1/systemd           
udp6       0      0 localhost:323           [::]:*                              760/chronyd   
```
### 实例5：显示网卡列表

```bash
$ netstat -i
Kernel Interface table
Iface             MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
docker0          1500        0      0      0 0             0      0      0      0 BMU
eth0             1500     1450      0      0 0          1637      0      0      0 BMRU
lo              65536       72      0      0 0            72      0      0      0 LRU
```
### 实例6：显示组播组的关系

```bash
$  netstat -g
\IPv6/IPv4 Group Memberships
Interface       RefCnt Group
--------------- ------ ---------------------
lo              1      all-systems.mcast.net
eth0            1      all-systems.mcast.net
docker0         1      all-systems.mcast.net
lo              1      ff02::1
lo              1      ff01::1
eth0            1      ff02::1:ffc0:4599
eth0            1      ff02::1
eth0            1      ff01::1
docker0         1      ff02::1
docker0         1      ff01::1
```
### 实例7：显示网络统计信息

```bash
$ netstat -s
Ip:
    1254 total packets received
    0 forwarded
    0 incoming packets discarded
    1254 incoming packets delivered
    1140 requests sent out
    16 dropped because of missing route
Icmp:
    37 ICMP messages received
    0 input ICMP message failed.
    ICMP input histogram:
        destination unreachable: 37
    37 ICMP messages sent
    0 ICMP messages failed
    ICMP output histogram:
        destination unreachable: 37
IcmpMsg:
        InType3: 37
        OutType3: 37
Tcp:
    4 active connections openings
    2 passive connection openings
    0 failed connection attempts
    0 connection resets received
    2 connections established
    869 segments received
    1047 segments send out
    0 segments retransmited
    0 bad segments received.
    0 resets sent
Udp:
    163 packets received
    37 packets to unknown port received.
    0 packet receive errors
    339 packets sent
    0 receive buffer errors
    0 send buffer errors
UdpLite:
TcpExt:
    4 TCP sockets finished time wait in fast timer
    24 delayed acks sent
    2 packets directly queued to recvmsg prequeue.
    274 packet headers predicted
    229 acknowledgments not containing data payload received
    249 predicted acknowledgments
    TCPRcvCoalesce: 3
    TCPAutoCorking: 8
    TCPOrigDataSent: 850
    TCPHystartTrainDetect: 1
    TCPHystartTrainCwnd: 18
IpExt:
    InBcastPkts: 147
    InOctets: 111816
    OutOctets: 767452
    InBcastOctets: 14228
    InNoECTPkts: 1260
```
说明：

按照各个协议分别显示其统计数据。如果我们的应用程序（如Web浏览器）运行速度比较慢，或者不能显示Web页之类的数据，那么我们就可以用本选项来查看一下所显示的信息。我们需要仔细查看统计数据的各行，找到出错的关键字，进而确定问题所在。

### 实例8：显示监听的套接口

```bash
$ netstat -l
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 localhost:smtp          0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:sunrpc          0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN     
tcp6       0      0 localhost:smtp          [::]:*                  LISTEN     
tcp6       0      0 [::]:sunrpc             [::]:*                  LISTEN     
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN     
udp        0      0 0.0.0.0:890             0.0.0.0:*                          
udp        0      0 0.0.0.0:sunrpc          0.0.0.0:*                          
udp        0      0 localhost:323           0.0.0.0:*                          
udp6       0      0 [::]:890                [::]:*                             
udp6       0      0 [::]:sunrpc             [::]:*                             
udp6       0      0 localhost:323           [::]:*                             
raw6       0      0 [::]:ipv6-icmp          [::]:*                  7          
Active UNIX domain sockets (only servers)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  2      [ ACC ]     STREAM     LISTENING     23300    /var/run/docker.sock
unix  2      [ ACC ]     STREAM     LISTENING     17690    /run/dbus/system_bus_socket
unix  2      [ ACC ]     STREAM     LISTENING     23342    /var/run/docker/libcontainerd/docker-containerd.sock
```
### 实例9：显示所有已建立的有效连接

```bash
$ netstat -n
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0     52 192.168.211.15:22       192.168.211.1:49878     ESTABLISHED
tcp        0      0 192.168.211.15:22       192.168.211.1:64318     ESTABLISHED
Active UNIX domain sockets (w/o servers)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  2      [ ]         DGRAM                    19782    /var/run/chrony/chronyd.sock
unix  2      [ ]         DGRAM                    12252    /run/systemd/shutdownd
unix  3      [ ]         DGRAM                    9450     /run/systemd/notify
```
### 实例10：显示关于以太网的统计数据

```bash
$ netstat -e
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       User       Inode     
tcp        0     52 localhost.localdoma:ssh 192.168.211.1:49878     ESTABLISHED root       40280     
tcp        0      0 localhost.localdoma:ssh 192.168.211.1:64318     ESTABLISHED root       25954     
Active UNIX domain sockets (w/o servers)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  2      [ ]         DGRAM                    19782    /var/run/chrony/chronyd.sock
unix  2      [ ]         DGRAM                    12252    /run/systemd/shutdownd
unix  3      [ ]         DGRAM                    9450     /run/systemd/notify
```
### 实例11：显示关于路由表的信息

```bash
$ netstat -r
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
default         gateway         0.0.0.0         UG        0 0          0 eth0
172.17.0.0      0.0.0.0         255.255.0.0     U         0 0          0 docker0
192.168.211.0   0.0.0.0         255.255.255.0   U         0 0          0 eth0
```

### 实例12：列出所有 tcp 端口

```bash
$ netstat -at
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 localhost:smtp          0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:sunrpc          0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN     
tcp        0     52 localhost.localdoma:ssh 192.168.211.1:49878     ESTABLISHED
tcp        0      0 localhost.localdoma:ssh 192.168.211.1:64318     ESTABLISHED
tcp6       0      0 localhost:smtp          [::]:*                  LISTEN     
tcp6       0      0 [::]:sunrpc             [::]:*                  LISTEN     
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN     
```
###  实例13：统计机器中网络连接各个状态个数

```bash
$ netstat -a | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
LISTEN 6
ESTABLISHED 2
```
### 实例14：把状态全都取出来后使用uniq -c统计后再进行排序

```bash
$ netstat -nat |awk '{print $6}'|sort|uniq -c
      1 established)
      2 ESTABLISHED
      1 Foreign
      6 LISTEN

netstat -nat |awk '{print $6}'|sort|uniq -c|sort -rn
      6 LISTEN
      2 ESTABLISHED
      1 Foreign
      1 established)

```
### 实例15：找出程序运行的端口

```bash
$ netstat -ap | grep ssh
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN      1116/sshd           
tcp        0     52 localhost.localdoma:ssh 192.168.211.1:49878     ESTABLISHED 3406/sshd: root@pts 
tcp        0      0 localhost.localdoma:ssh 192.168.211.1:64318     ESTABLISHED 1716/sshd: root@pts 
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN      1116/sshd           
unix  2      [ ]         DGRAM                    25487    1716/sshd: root@pts  
unix  2      [ ]         DGRAM                    40335    3406/sshd: root@pts  
unix  3      [ ]         STREAM     CONNECTED     22118    1116/sshd 
```
### 实例17：在 netstat 输出中显示 PID 和进程名称

```bash
$ netstat -pt
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0     52 localhost.localdoma:ssh 192.168.211.1:49878     ESTABLISHED 3406/sshd: root@pts 
tcp        0      0 localhost.localdoma:ssh 192.168.211.1:64318     ESTABLISHED 1716/sshd: root@pts 
```
说明：

netstat -p 可以与其它开关一起使用，就可以添加 “PID/进程名称” 到 netstat 输出中，这样 debugging 的时候可以很方便的发现特定端口运行的程序。
### 实例18：找出运行在指定端口的进程

```bash
$ netstat -anpt | grep ':22'
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1116/sshd           
tcp        0     52 192.168.211.15:22       192.168.211.1:49878     ESTABLISHED 3406/sshd: root@pts 
tcp        0      0 192.168.211.15:22       192.168.211.1:64318     ESTABLISHED 1716/sshd: root@pts 
tcp6       0      0 :::22                   :::*                    LISTEN      1116/sshd   
```
参考：

 - [每天一个linux命令（56）：netstat命令](https://www.cnblogs.com/peida/archive/2013/03/08/2949194.html)
 - [How to Use netstat on Linux](https://www.howtogeek.com/513003/how-to-use-netstat-on-linux/)
 - [20 Netstat Commands for Linux Network Management](https://www.tecmint.com/20-netstat-commands-for-linux-network-management/)
 - [netstat(8) - Linux man page](https://linux.die.net/man/8/netstat)

