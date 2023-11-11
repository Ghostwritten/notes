traceroute详解

## 1.traceroute基本概念

　　traceroute (Windows系统下是tracert) 命令利用ICMP 协议定位您的计算机和目标计算机之间的所有路由器。TTL值可以反映数据包经过的路由器或网关的数量，通过操纵独立ICMP呼叫报文的TTL值和观察该报文被抛弃的返回信息，traceroute命令能够遍历到数据包传输路径上的所有路由器。traceroute是一条缓慢的命令，因为每经过一台路由器都要花去大约10到15秒。

## 2. traceroute工作原理及详细过程

　　traceroute是用来侦测主机到目的主机之间所经路由情况的重要工具，也是最便利的工具。尽管ping工具也可以进行侦测，但是，因为ip头的限制，ping不能完全的记录下所经过的路由器，所以traceroute正好就填补了这个缺憾。traceroute的原理是非常非常的有意思，它收到目的主机的IP后，首先给目的主机发送一个TTL=1的UDP数据包，而经过的第一个路由器收到这个数据包以后，就自动把TTL减1，而TTL变为0以后，路由器就把这个包给抛弃了，并同时产生 一个主机不可达的ICMP数据报给主机。主机收到这个数据报以后再发一个TTL=2的UDP数据报给目的主机，然后刺激第二个路由器给主机发ICMP数据报。如此往复直到到达目的主机。这样，traceroute就拿到了所有的路由器ip。从而避开了ip头只能记录有限路由IP的问题。有人要问，我怎么知道UDP到没到达目的主机呢？这就涉及一个技巧的问题，TCP和UDP协议有一个端口号定义，而普通的网络程序只监控少数的几个号码较小的端口，比如说80,比如说23,等等。而traceroute发送的是端口号大于30000(真变态)的UDP报，所以到达目的主机的时候，目的主机只能发送一个端口不可达的ICMP数据报给主机。主机接到这个报告以后就知道，主机到了，所以，说traceroute是一个骗子一点也不为过:)。其详细过程如下：
将传递到目的IP地址的ICMP Echo消息的TTL值被设置为1，该消息报经过第一个路由器时，其TTL值减去1，此时新产生的TTL值为0。
由于TTL值被设置为0，路由器判断此时不应该尝试继续转发数据报，而是直接抛弃该数据报。由于数据报的生存周期（TTL值）已经到期，这个路由器会发送一个一个ICMP时间超时，即TTL值过期信息返回到客户端计算机。
此时，发出traceroute命令的客户端计算机将显示该路由器的名称，之后可以再发送一个ICMP Echo消息并把TTL值设置为2。
第1个路由器仍然对这个TTL值减1，然后，如果可能的话，将这个数据报转发到传输路径上的下一跳。当数据报抵达第2个路由器，TTL值会再被减去1，成为0值。
第2个路由器会像第1个路由器一样，抛弃这个数据包，并像第1个路由器那样返回一个ICMP消息。
该过程会一直持续，traceroute命令不停递增TTL值，而传输路径上的路由器不断递减该值，直到数据报最终抵达预期的目的地。
当目的计算机接收到ICMP Echo消息时，会回传一个ICMP Echo Reply消息。

## 3.traceroute常用命令

 　　traceroute的用法为: Traceroute [options] <IP-address or domain-name> [data size]
　　[options]的内容有:
[-n]:显示的地址是用数字表示而不是符号
[-v]:长输出
[-p]:UDP端口设置(缺省为33434)
[-q]:设置TTL测试数目(缺省为3)
[-t]:设置测包的服务类型
　　[data size]:每次测试包的数据字节长度(缺省为38)

traceroute 命令的基本用法是，在命令提示符后键入 “tracert host_name” 或 “tracert ip_address”，其中，tracert 是 traceroute 在 Windows 操作系统上的称呼。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200220112251308.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
输出有 5 列：

第一列是描述路径的第 n 跳的数值，即沿着该路径的路由器序号；
第二列是第一次往返时延；
第三列是第二次往返时延；
第四列是第三次往返时延；
第五列是路由器的名字及其输入端口的 IP 地址。

如果源从任何给定的路由器接收到的报文少于 3 条（由于网络中的分组丢失），traceroute 在该路由器号码后面放一个星号，并报告到达那台路由器的少于 3 次的往返时间。

此外，tracert 命令还可以用来查看网络在连接站点时经过的步骤或采取哪种路线，如果是网络出现故障，就可以通过这条命令查看出现问题的位置。
 

## 4.一些注意点

并不是所有网关都会如实返回ICMP超时报文。出于安全性考虑，大多数防火墙以及启用了防火墙功能的路由器缺省配置为不返回各种ICMP报文，其余路由器或交换机也可能被管理员主动修改配置变为不返回 ICMP报文。因此traceroute程序不一定能拿到所有的沿途网关地址。所以，当某个TTL值的数据包得不到响应时，并不能停止这一追踪过程，程序仍然会把TTL递增而发出下一个数据包。这个过程将一直持续到数据包发送到目标主机，或者达到默认或用参数指定的追踪限制（maximum_hops）才结束追踪。依据上述原理，利用了UDP数据包的traceroute程序在数据包到达真正的目的主机时，就可能因为该主机没有提供UDP服务而简单将数据包抛弃，并不返回任何信息。为了解决这个问题，traceroute故意使用了一个大于30000的端口号，因UDP协议规定端口号必须小于30000，所以目标主机收到数据包后唯一能做的事就是返回一个“端口不可达”的ICMP报文，于是主叫方就将端口不可达报文当作跟踪结束的标志。
使用UDP的traceroute，失败还是比较常见的。这常常是由于，在运营商的路由器上，UDP与ICMP的待遇大不相同。为了利于troubleshooting，ICMP ECHO Request/Reply 是不会封的，而UDP则不同。UDP常被用来做网络攻击，因为UDP无需连接，因而没有任何状态约束它，比较方便攻击者伪造源IP、伪造目的端口发送任意多的UDP包，长度自定义。所以运营商为安全考虑，对于UDP端口常常采用白名单ACL，就是只有ACL允许的端口才可以通过，没有明确允许的则统统丢弃。比如允许DNS/DHCP/SNMP等。
总结一下，traceroute主要利用IP数据包的TTL字段值 + ICMP来实现，它发送的用于探测网络路径的数据包的IP之上的协议可以是 UDP、TCP或ICMP。不同模式下，探测过程中设计的数据包如下：
UDP模式：UDP探测数据包（目标端口大于30000） + 中间网关发回 ICMP TTL 超时数据包 + 目标主机发回ICMP Destination Unreachable 数据包
TCP模式：TCP [SYN]探测数据包（目标端口为Web服务的80） + 中间网关发回 ICMP TTL 超时数据包 + 目标主机发回TCP [SYN ACK] 数据包
ICMP模式：ICMP Echo (ping) Request 探测数据包 + 中间网关发回ICMP TTL超时数据包 + 目标主机发回ICMP Echo (ping) reply 数据包
traceroute出现*的分析：源发出ICMP Request，第一个request的TTL为1，第二个request的TTL为2，以后依此递增直至第30个；中间的router送回ICMP TTL-expired ( ICMP type 11) 通知source，（packet同时因TTL超时而被drop)，由此source知晓一路上经过的每一个router；最后的destination送回ICMP Echo Reply（最后一跳不会再回ICMP TTL-expired）。所以中间任何一个router上如果封了ICMP Echo Request, traceroute就不能工作；如果封了type 11(TTL-expired), 中间的router全看不到，但能看到packet到达了最后的destination；如果封了ICMP Echo Reply，中间的全能看到，最后的destination看不到。

## 5.思考【测试大型网络的路由】

（1）多尝试几次 “ping www.sina.com.cn” 操作，比较得到的新浪网的 IP 地址。如果两次 ping 得到的 IP 地址不同，试考虑其中的原因（如考虑到负载均衡）。然后，针对这些不同的 IP 地址，执行 “tracert ip_address” 命令，观察分析输出的结果是否有差异。

（2）对于大型网络中的某站点进行 traceroute 测试，记录测试结果。观察其中是否出现第 n 跳的时延小于第 n-1 跳的时延情况。试分析其中原因（提示：可分别考虑时延的各个构成成分在总时延中所起的作用）。

（3）在一天的不同时段内，用 traceroute 程序多次测试从固定主机到远程固定 IP 地址的主机的路由。试分析比较测量数据，观察该路由是否有变化？如果有变化，该变化频繁吗？
