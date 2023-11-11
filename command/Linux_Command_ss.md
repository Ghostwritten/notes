#  Linux Command ss
tags: 网络, 分析



--------------
##  1. 简介
`ss` 是 Socket Statistics 的缩写。ss 命令可以用来获取 socket 统计信息，它显示的内容和 netstat 类似。但 ss 的优势在于它能够显示更多更详细的有关 TCP 和连接状态的信息，而且比 netstat 更快。当服务器的 socket 连接数量变得非常大时，无论是使用 netstat 命令还是直接 `cat /proc/net/tcp`，执行速度都会很慢。ss 命令利用到了 TCP 协议栈中 tcp_diag。tcp_diag 是一个用于分析统计的模块，可以获得 Linux 内核中第一手的信息，因此 ss 命令的性能会好很多


##  2. 参数
常用选项

```bash
-h, –help 帮助
 -V, –version 显示版本号
 -t, –tcp 显示 TCP 协议的 sockets
 -u, –udp 显示 UDP 协议的 sockets
 -x, –unix 显示 unix domain sockets，与 -f 选项相同
 -n, –numeric 不解析服务的名称，如 “22” 端口不会显示成 “ssh”
 -l, –listening 只显示处于监听状态的端口
 -p, –processes 显示监听端口的进程(Ubuntu 上需要 sudo)
 -a, –all 对 TCP 协议来说，既包含监听的端口，也包含建立的连接
 -r, –resolve 把 IP 解释为域名，把端口号解释为协议名称
 -o, –options 显示时间信息
 -m, –memory 显示 socket 使用的内存 
 -i, –info 显示更多 TCP 内部的信息
 -s 显示概要信息
```
## 3.示例

### 3.1 默认显示
如果不添加选项 ss 命令默认输出所有建立的连接(不包含监听的端口)，包括 `tcp, udp, and unix socket` 三种类型的连接：

```c
[root@master ~]# ss | head -n 5
Netid  State      Recv-Q Send-Q Local Address:Port                 Peer Address:Port                
u_str  ESTAB      0      0       * 49344                 * 49345                
u_str  ESTAB      0      0       * 49311                 * 49312                
u_str  ESTAB      0      0       * 49341                 * 49342                
u_str  ESTAB      0      0       * 49312                 * 49311
```
### 3.2 查看主机监听的端口

```bash
# 显示端口的服务名
[root@master ~]# ss -tl
State      Recv-Q Send-Q                                                     Local Address:Port                                                                      Peer Address:Port                
LISTEN     0      128                                                                    *:sunrpc                                                                               *:*                    
LISTEN     0      128                                                                    *:ssh                                                                                  *:*                    
LISTEN     0      100                                                            127.0.0.1:smtp                                                                                 *:*                    
LISTEN     0      128                                                                   :::sunrpc                                                                              :::*                    
LISTEN     0      128                                                                   :::ssh                                                                                 :::*                    
LISTEN     0      100                                                                  ::1:smtp                                                                                :::*       
#显示端口号             
[root@master ~]# ss -tnl
State      Recv-Q Send-Q                                                       Local Address:Port                                                                      Peer Address:Port              
LISTEN     0      128                                                                      *:111                                                                                  *:*                  
LISTEN     0      128                                                                      *:22                                                                                   *:*                  
LISTEN     0      100                                                              127.0.0.1:25                                                                                   *:*                  
LISTEN     0      128                                                                     :::111                                                                                 :::*                  
LISTEN     0      128                                                                     :::22                                                                                  :::*                  
LISTEN     0      100                                                                    ::1:25                                                                                  :::*      
```
### 3.3 通过 -r 选项解析 IP 和端口号

```bash
[root@master ~]# ss -tlr
State      Recv-Q Send-Q                                                     Local Address:Port                                                                      Peer Address:Port                
LISTEN     0      128                                                                    *:rpc.portmapper                                                                       *:*                    
LISTEN     0      128                                                                    *:ssh                                                                                  *:*                    
LISTEN     0      100                                                            localhost:smtp                                                                                 *:*                    
LISTEN     0      128                                                                   :::rpc.portmapper                                                                      :::*                    
LISTEN     0      128                                                                   :::ssh                                                                                 :::*                    
LISTEN     0      100                                                            localhost:smtp                                                                                :::*  
```

### 3.4 使用 -p 选项查看监听端口的程序名称

```bash
[root@master ~]# ss -tlp
State      Recv-Q Send-Q                                                                               Local Address:Port                                                                                                Peer Address:Port                
LISTEN     0      128                                                                                              *:sunrpc                                                                                                         *:*                     users:(("rpcbind",pid=5502,fd=4),("systemd",pid=1,fd=44))
LISTEN     0      128                                                                                              *:ssh                                                                                                            *:*                     users:(("sshd",pid=6477,fd=3))
LISTEN     0      100                                                                                      127.0.0.1:smtp                                                                                                           *:*                     users:(("master",pid=6890,fd=13))
LISTEN     0      128                                                                                             :::sunrpc                                                                                                        :::*                     users:(("rpcbind",pid=5502,fd=6),("systemd",pid=1,fd=46))
LISTEN     0      128                                                                                             :::ssh                                                                                                           :::*                     users:(("sshd",pid=6477,fd=4))
LISTEN     0      100                                                                                            ::1:smtp                                                                                                          :::*                     users:(("master",pid=6890,fd=14))
```
###  3.5 -a –all 对 TCP 协议来说，既包含监听的端口，也包含建立的连接

```bash
[root@master ~]# ss -tna 
State      Recv-Q Send-Q                                                                                 Local Address:Port                                                                                                Peer Address:Port              
LISTEN     0      128                                                                                                *:111                                                                                                            *:*                  
LISTEN     0      128                                                                                                *:22                                                                                                             *:*                  
LISTEN     0      100                                                                                        127.0.0.1:25                                                                                                             *:*                  
ESTAB      0      52                                                                                    192.168.211.12:22                                                                                                 192.168.211.1:62054              
LISTEN     0      128                                                                                               :::111                                                                                                           :::*                  
LISTEN     0      128                                                                                               :::22                                                                                                            :::*                  
LISTEN     0      100                                                                                              ::1:25                                                                                                            :::*    
```
### 3.6 dst/src dport/sport 语法
可以通过 dst/src/dport/sprot 语法来过滤连接的来源和目标，来源端口和目标端口。

匹配远程地址和端口号

```bash
$ ss dst 192.168.1.5
$ ss dst 192.168.119.113:http
$ ss dst 192.168.119.113:443
```

匹配本地地址和端口号

```bash
$ ss src 192.168.119.103
$ ss src 192.168.119.103:http
$ ss src 192.168.119.103:80
```

将本地或者远程端口和一个数比较

可以使用下面的语法做端口号的过滤：

```bash
$ ss dport OP PORT
$ ss sport OP PORT
```

OP 可以代表以下任意一个：

```bash
# <= or le : 小于或等于端口号
# >= or ge : 大于或等于端口号
# == or eq : 等于端口号
# != or ne : 不等于端口号
# < or gt : 小于端口号
# > or lt : 大于端口号


ss  sport = :http
ss  dport = :http
ss  dport \> :1024
ss  sport \> :1024
ss sport \< :32000
ss  sport eq :22
ss  dport != :22
ss  state connected sport = :http
ss \( sport = :http or sport = :https \)
ss -o state fin-wait-1 \( sport = :http or sport = :https \) dst 192.168.1/24
```
### 3.7 用TCP 状态过滤Sockets

```bash
ss -4 state closing
# ss -4 state FILTER-NAME-HERE
# ss -6 state FILTER-NAME-HERE
# FILTER-NAME-HERE 可以代表以下任何一个：
# established、 syn-sent、 syn-recv、 fin-wait-1、 fin-wait-2、 time-wait、 closed、 close-wait、 last-ack、 listen、 closing、
# all : 所有以上状态
# connected : 除了listen and closed的所有状态
# synchronized :所有已连接的状态除了syn-sent
# bucket : 显示状态为maintained as minisockets,如：time-wait和syn-recv.
# big : 和bucket相反.
```
###  3.8 显示TCP连接

```bash
[root@localhost ~]# ss -t -a
State       Recv-Q Send-Q                            Local Address:Port                                Peer Address:Port
LISTEN      0      0                                             *:3306                                           *:*
LISTEN      0      0                                             *:http                                           *:*
LISTEN      0      0                                             *:ssh                                            *:*
LISTEN      0      0                                     127.0.0.1:smtp                                           *:*
ESTAB       0      0                                112.124.15.130:42071                              42.156.166.25:http
ESTAB       0      0                                112.124.15.130:ssh                              121.229.196.235:33398
```

 查看TCP的连接状态

```bash
[root@localhost ~]# ss  -tan|awk 'NR>1{++S[$1]}END{for (a in S) print a,S[a]}'
LISTEN 7
ESTAB 31
TIME-WAIT 28
```

###  3.9 显示 Sockets 摘要

```bash
[root@localhost ~]# ss -s
Total: 172 (kernel 189)
TCP:   10 (estab 2, closed 4, orphaned 0, synrecv 0, timewait 0/0), ports 5

Transport Total     ip        IPv6
*         189       -         -
RAW       0         0         0
UDP       5         5         0
TCP       6         6         0
INET      11        11        0
FRAG      0         0         0
```

列出当前的established, closed, orphaned and waiting TCP sockets

###  3.10 列出所有打开的网络连接端口

```bash
[root@localhost ~]# ss -l
Recv-Q Send-Q                                 Local Address:Port                                     Peer Address:Port
0      0                                                  *:3306                                                *:*
0      0                                                  *:http                                                *:*
0      0                                                  *:ssh                                                 *:*
0      0                                          127.0.0.1:smtp                                                *:*
```
###  3.11 查看进程使用的socket

```bash
[root@localhost ~]# ss -pl
Recv-Q Send-Q                                          Local Address:Port                                              Peer Address:Port
0      0                                                           *:3306                                                         *:*        users:(("mysqld",1718,10))
0      0                                                           *:http                                                         *:*        users:(("nginx",13312,5),("nginx",13333,5))
0      0                                                           *:ssh                                                          *:*        users:(("sshd",1379,3))
0      0                                                   127.0.0.1:smtp                                                         *:*        us
```

找出打开套接字/端口应用程序

```bash
[root@localhost ~]# ss -pl | grep 3306
0      0                            *:3306                          *:*        users:(("mysqld",1718,10))
```
###  3.12 显示所有UDP Sockets

```bash
[root@localhost ~]# ss -u -a
State       Recv-Q Send-Q                                     Local Address:Port                                         Peer Address:Port
UNCONN      0      0                                                      *:syslog                                                  *:*
UNCONN      0      0                                         112.124.15.130:ntp                                                     *:*
UNCONN      0      0                                            10.160.7.81:ntp                                                     *:*
UNCONN      0      0                                              127.0.0.1:ntp                                                     *:*
UNCONN      0      0                                                      *:ntp                                                     *:*
```
###  3.13 所有端口为 22（ssh）的连接

```bash
[root@localhost ~]# ss state all sport = :ssh
Netid State      Recv-Q Send-Q     Local Address:Port                      Peer Address:Port
tcp   LISTEN     0      128                    *:ssh                                  *:*
tcp   ESTAB      0      0          192.168.0.136:ssh                      192.168.0.102:46540
tcp   LISTEN     0      128                   :::ssh                                 :::*
```

参考：

 - [ss(8) — Linux manual page](https://man7.org/linux/man-pages/man8/ss.8.html)
 - [An Introduction to the ss Command](https://www.linux.com/topic/networking/introduction-ss-command/)
 - [How to Use the ss Command on Linux](https://www.howtogeek.com/681468/how-to-use-the-ss-command-on-linux/)
