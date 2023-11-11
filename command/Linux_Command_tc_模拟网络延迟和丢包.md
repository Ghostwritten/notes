#  Linux Command tc 模拟网络延迟和丢包


![在这里插入图片描述](https://img-blog.csdnimg.cn/a847d39afe1145298a5c2ba2a59acd51.png)


## 1. 介绍
Linux 操作系统中的流量控制器 TC(Traffic Control) 用于Linux内核的流量控制，它利用**队列**规定建立处理数据包的队列，并定义队列中的数据包被发送的方式，从而实现对流量的控制。TC 模块实现流量控制功能使用的队列规定分为两类，一类是无类队列规定，另一类是分类队列规定。无类队列规定相对简单，而分类队列规定则引出了分类和过滤器等概念，使其流量控制功能增强。

 

无类队列规定是对进入网络设备（网卡）的数据流不加区分统一对待的队列规定。使用无类队列规定形成的队列能够接收数据包以及重新编排、延迟或丢弃数据包。这类队列规定形成的队列可以对整个网络设备（网卡）的流量进行整形，但不能细分各种情况。常用的无类队列规定主要有 `pfifo_fast`（先进先出）、`TBF`（令牌桶过滤器）、`SFQ`（随机公平队列）、`ID`（前向随机丢包）等等。这类队列规定使用的流量整形手段主要是排序、限速和丢包。

 

分类队列规定是对进入网络设备的数据包根据不同的需求以分类的方式区分对待的队列规定。数据包进入一个分类的队列后，它就需要被送到某一个类中，也就是说需要对数据包做分类处理。对数据包进行分类的工具是过滤器，过滤器会返回一个决定，队列规定就根据这个决定把数据包送入相应的类进行排队。每个子类都可以再次使用它们的过滤器进行进一步的分类。直到不需要进一步分类时，数据包才进入该类包含的队列排队。除了能够包含其他队列规定之外，绝大多数分类的队列规定还能够对流量进行整形。这对于需要同时进行调度（如使用SFQ）和流量控制的场合非常有用。

Linux流量控制的基本原理如下图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/8486d35d243f45928ba0fb9d446643ee.png)
接收包从输入接口（Input Interface）进来后，经过流量限制（Ingress Policing）丢弃不符合规定的数据包，由输入多路分配器（Input De-Multiplexing）进行判断选择。如果接收包的目的地是本主机，那么将该包送给上层处理，否则需要进行转发，将接收包交到转发块（Forwarding Block）处理。转发块同时也接收本主机上层（TCP、UDP等）产生的包。转发块通过查看路由表，决定所处理包的下一跳。然后，对包进行排列以便将它们传送到输出接口（Output Interface）。一般我们只能限制网卡发送的数据包，不能限制网卡接收的数据包，所以我们可以通过改变发送次序来控制传输速率。Linux流量控制主要是在输出接口排列时进行处理和实现的。

## 2. 规则
### 2.1 流量控制方式

流量控制包括一下几种方式：`SHAPING`、`SCHEDULING`、`POLICING`、`DROPPING`；
 
- `SHAPING`（限制）
当流量被限制时，它的传输速率就被控制在某个值以下。限制值可以大大小于有效带宽，这样可以平滑突发数据流量，使网络更为稳定。SHAPING（限制）只适用于向外的流量。

 - `SCHEDULING`（调度）
通过调度数据包的传输，可以在带宽范围内，按照优先级分配带宽。SCHEDULING（调度）也只适用于向外的流量。

-  `POLICING`（策略）
SHAPING（限制）用于处理向外的流量，而POLICING（策略）用于处理接收到的数据。

 - `DROPPING`（丢弃）
如果流量超过某个设定的带宽，就丢弃数据包，不管是向内还是向外。


### 2.2 流量控制处理对象
流量的处理由三种对象控制，它们是：`qdisc`（排队规则）、`class`（类别）和`filter`（过滤器）。

`qdisc`（排队规则）是 queueing discipline的简写，它是理解流量控制（traffic control）的基础。无论何时，内核如果需要通过某个网络接口发送数据包，它都需要按照为这个接口配置的qdisc（排队规则）把数据包加入队列。然后，内核会尽可能多的从qdisc里面取出数据包，把它们交给网络适配器驱动模块。最简单的qdisc是`pfifo`他不对进入的数据包做任何的处理，数据包采用先进先出的方式通过队列。不过，它会保存网络接口一时无法处 理的数据包。

qddis（排队规则）分为 `CLASSLESS QDISC`和 `CLASSFUL QDISC`；

**CLASSLESS QDISC** （无类别QDISC）包括：

- `[ p | b ]fifo`,使用最简单的qdisc（排队规则），纯粹的先进先出。只有一个参数：`limit` ，用来设置队列的长度，pfifo是以数据包的个数为单位；bfifo是以字节数为单位。

- `pfifo_fast`，在编译内核时，如果打开了高级路由器（Advanced Router）编译选项，`pfifo_fast` 就是系统的标准qdisc(排队规则)。它的队列包括三个波段（band）。在每个波段里面，使用先进先出规则。而三个波段（band）的优先级也不相同，`band 0` 的优先级最高，`band 2`的最低。如果band 0里面有数据包，系统就不会处理band 1 里面的数据包，band 1 和 band 2 之间也是一样的。数据包是按照服务类型（Type Of Service，TOS ）被分配到三个波段（band）里面的。

 - `red`，red是Random Early Detection（随机早期探测）的简写。如果使用这种qdsic，当带宽的占用接近与规定的带宽时，系统会随机的丢弃一些数据包。他非常适合高带宽的应用。

 - `sfq`，sfq是Stochastic Fairness Queueing 的简写。它会按照会话（session --对应与每个TCP 连接或者UDP流）为流量进行排序，然后循环发送每个会话的数据包。
 - `tbf`，tbf是 Token Bucket Filter 的简写，适用于把流速降低到某个值。


 **CLASSLESS QDISC** （无类别QDISC）的配置

 如果没有可分类qdisc，不可分类qdisc 只能附属于设备的根。它们的用法如

```bash
 $ tc qdisc add dev DEV root QDISC QDISC_PARAMETERS 
```
要删除一个不可分类qdisc，需要使用如

```bash
$ tc qdisc del dev DEV root 
```

 

一个网络接口上如果没有设置`qdisc`，`pfifo_fast`就作为缺省的`qdisc`。

  **CLASSFUL QDISC**(可分类 QDISC)包括：
  
- `CBQ`，CBQ是 `Class Based Queueing`（基于类别排队）的缩写。它实现了一个丰富的连接共享类别结构，既有限制（shaping）带宽的能力，也具有带宽优先级别管理的能力。带宽限制是通过计算连接的空闲时间完成的。空闲时间的计算标准是数据包离队事件的频率和下层连接（数据链路层）的带宽。

- `HTB`，HTB是`Hierarchy Token Bucket` 的缩写。通过在实践基础上的改进，它实现一个丰富的连接共享类别体系。使用HTB可以很容易地保证每个类别的带宽，虽然它也允许特定的类可以突破带宽上限，占用别的类的带宽。HTB可以通过TBF（Token Bucket Filter）实现带宽限制，也能够划分类别的优先级。

- `PRIO`，PRIO qdisc 不能限制带宽，因为属于不同类别的数据包是顺序离队的。使用PRIO qdisc 可以很容易对流量进行优先级管理，只有属于高优先级类别的数据包全部发送完毕，参会发送属于低优先级类别的数据包。为了方便管理，需要使用iptables 或者 ipchains 处理数据包的服务类型（Type Of Service，TOS）。


## 3. 操作原理
类（class）组成一个树，每个类都只有一个父类，而一个类可以有多个子类。某些qdisc （例如：CBQ和 HTB）允许在运行时动态添加类，而其它的qdisc（例如：PRIO）不允许动态建立类。允许动态添加类的qdisc可以有零个或者多个子类，由它们为数据包排队。此外，每个类都有一个叶子qdisc，默认情况下，这个也在qdisc有可分类，不过每个子类只能有一个叶子qdisc。 当一个数据包进入一个分类qdisc，它会被归入某个子类。我们可以使用一下三种方式为数据包归类，不过不是所有的qdisc都能够使用这三种方式。

如果过滤器附属于一个类，相关的指令就会对它们进行查询。过滤器能够匹配数据包头所有的域，也可以匹配由ipchains或者iptables做的标记。 

树的每个节点都可以有自己的过滤器，但是高层的过滤器也可以一直接用于其子类。如果数据包没有被成功归类，就会被排到这个类的叶子qdisc的队中。相关细节在各个qdisc的手册页中。

## 4. 命名规则
所有的`qdisc`、类、和过滤器都有ID。ID可以手工设置，也可以由内核自动分配。ID由一个主序列号和一个从序列号组成，两个数字用一个冒号分开。

- `qdisc`，一个qdisc会被分配一个主序列号，叫做句柄（handle），然后把从序列号作为类的命名空间。句柄才有像1:0 一样的表达方式。习惯上，需要为有子类的qdisc显式的分配一个句柄。

- `类（Class）`，在同一个qdisc里面的类共享这个qdisc的主序列号，但是每个类都有自己的从序列号，叫做类识别符（classid）。类识别符只与父qdisc有关，与父类无关。类的命名习惯和qdisc相同。

 - `过滤器（Filter）`，过滤器的ID有三部分，只有在对过滤器进行散列组织才会用到。详情请参考tc-filtes手册页。


## 5. 单位
tc命令所有的参数都可以使用浮点数，可能会涉及到以下计数单位。

带宽或者流速单位：
- kbps                              千字节/秒

- mbps                             兆字节/秒

- kbit                                KBits/秒

- mbit                               MBits/秒

- bps或者一个无单位数字  字节数/秒

数据的数量单位：

- kb或者k                        千字节

- mb或者m                      兆字节

- mbit                              兆bit

- kbit                               千bit

- b或者一个无单位数字    字节数

 

时间的计量单位：

- s、sec或者secs                                     秒

- ms、msec或者msecs                           分钟

- us、usec、usecs或者一个无单位数字   微秒


## 6. 安装
 `TC`是linux自带的模块，一般不需要安装，TC要求内核2.4.18以上。注意：64位机器上，或需先执行下面命令，做个软链接：`ln -s /usr/lib64/tc /usr/lib/tc`

## 7. 命令

tc可以使用以下命令对qdisc、类和过滤器进行操作：

- `add`， 在一个节点里加入一个qdisc、类、或者过滤器。添加时，需要传递一个祖先作为参数，传递参数时既可以使用ID也跨越式直接传递设备的根。如果要建立一个qdisc或者过滤器，可以使用句柄（handle）来命名。如果要建立一个类，可以使用类识别符（classid）来命名。

- `remove`， 删除由某个句柄（handle）指定的qdisc，根qdisc（root）也可以删除。被删除qdisc上所有的子类以及附属于各个类的过滤器都会被自动删除。

- `change`， 以替代的方式修改某些条目。除了句柄（handle）和祖先不能修改以外，change命令的语法和add命令相同。换句话说，change命令不能指定节点的位置。

- `replace`， 对一个现有节点进行近于原子操作的删除/添加。如果节点不存在，这个命令就会建立节点。

- `link`， 只适用于qdisc，替代一个现有的节点。

名称：

- linux TC(8) ： tc - 显示/操作流量控制设置

命令的格式：

```bash
tc qdisc [ add | change | replace | link | delete ] dev DEV [ parent qdisc-id | root ] [ handle qdisc-id ] qdisc [ qdisc specific parameters ]

tc class [ add | change | replace | delete ] dev DEV parent qdisc-id [ classid class-id ] qdisc [ qdisc specific parameters ]

tc filter [ add | change | replace | delete ] dev DEV [ parent qdisc-id | root ] protocol protocol prio priority filtertype [ filtertype specific parameters ] flowid flow-id

tc [ FORMAT ] qdisc show [ dev DEV ]

tc [ FORMAT ] class show dev DEV

tc filter show dev DEV

tc [ -force ] [ -OK ] -b[atch] [ filename ]

FORMAT := { -s[tatistics] | -d[etails] | -r[aw] | -p[retty] | -i[ec] }
```

## 8. 常用命令
1. 模拟延迟传输：

```bash
$ tc qdisc add dev eth0 root netem delay 100ms
```

该命令将 eth0 网卡的传输设置为延迟 100 毫秒发送，更真实的情况下,延迟值不会这么精确，会有一定的波动，后面用下面的情况来模拟出带有波动性的延迟值


2. 模拟延迟波动：

```bash
$ tc qdisc add dev eth0 root netem delay 100ms 10ms
```

该命令将 eth0 网卡的传输设置为延迟 100ms ± 10ms (90 ~ 110 ms 之间的任意值)发送。 还可以更进一步加强这种波动的随机性


3. 延迟波动随机性：

该命令将 eth0 网卡的传输设置为 100ms ,同时,大约有 30% 的包会延迟 ± 10ms 发送。

```bash
$ tc qdisc add dev eth0 root netem delay 100ms 10ms 30%
```


4. 模拟网络丢包：

```bash
$  tc qdisc add dev eth0 root netem loss 1%
```

该命令将 eth0 网卡的传输设置为随机丢掉 1% 的数据包


5. 网络丢包成功率：

```bash
$ tc qdisc add dev eth0 root netem loss 1% 30%
```

该命令将 eth0 网卡的传输设置为随机丢掉 1% 的数据包,成功率为 30%


6. 删除相关配置（将之前命令中的 add 改为 del 即可删除配置）：

```bash
$ tc qdisc del dev eth0 root netem delay 100ms
```

7. 模拟包重复：

```bash
$ tc qdisc add dev eth0 root netem duplicate 1%
```

该命令将 eth0 网卡的传输设置为随机产生 1% 的重复数据包


8. 模拟包损坏：

```bash
$ tc qdisc add dev eth0 root netem corrupt 0.2%
```

该命令将 eth0 网卡的传输设置为随机产生 0.2% 的损坏的数据包 。 (内核版本需在 2.6.16 以上)

 

9. 模拟包乱序：

```bash
$ tc qdisc change dev eth0 root netem delay 10ms reorder 25% 50%
```

该命令将 eth0 网卡的传输设置为:有 25% 的数据包(50%相关)会被立即发送,其他的延迟10 秒。

新版本中,如下命令也会在一定程度上打乱发包的次序:# tc qdisc add dev eth0 root netem delay 100ms 10ms

 

10. 查看网卡配置：

```bash
$ tc qdisc show dev eth0
```

该命令将 查看并显示 eth0 网卡的相关传输配置


11. 查看丢包率：

```bash
$ tc -s qdisc show dev eth0
```

## 9. 案例
如何使用tc模拟网络延迟和丢包

修改网络延时：  `sudo tc qdisc add dev eth0 root netem delay 1000ms`

查看流量管理：`tc qdisc show`

删除策略：`sudo tc qdisc del dev eth0 root netem delay 1000ms`

验证效果：`ping 192.168.102.124 -c 20`

 修改丢包率：`sudo tc qdisc add dev eth0 root netem loss 10%`

删除策略：`sudo tc qdisc del dev eth0 root netem loss 10%`


配置网络超时

```bash
[root@dev-xx-xx ~]# tc qdisc del dev eth0 root netem delay 100ms
RTNETLINK answers: Invalid argument
[root@dev-xx-xx ~]# tc qdisc show
qdisc mq 0: dev eth0 root
qdisc pfifo_fast 0: dev eth0 parent :1 bands 3 priomap 1 2 2 2 1 2 0 0 1 1 1 1 1 1 1 1
qdisc pfifo_fast 0: dev eth0 parent :2 bands 3 priomap 1 2 2 2 1 2 0 0 1 1 1 1 1 1 1 1
qdisc pfifo_fast 0: dev eth0 parent :3 bands 3 priomap 1 2 2 2 1 2 0 0 1 1 1 1 1 1 1 1
qdisc pfifo_fast 0: dev eth0 parent :4 bands 3 priomap 1 2 2 2 1 2 0 0 1 1 1 1 1 1 1 1
[root@dev-xx-xx ~]# tc qdisc add dev eth0 root netem delay 100ms
[root@dev-xx-xx ~]# ping 192.168.102.124
PING 192.168.102.124 (192.168.102.124) 56(84) bytes of data.
64 bytes from 192.168.102.124: icmp_seq=1 ttl=64 time=0.074 ms
64 bytes from 192.168.102.124: icmp_seq=2 ttl=64 time=0.066 ms
64 bytes from 192.168.102.124: icmp_seq=3 ttl=64 time=0.080 ms
64 bytes from 192.168.102.124: icmp_seq=4 ttl=64 time=0.043 ms
64 bytes from 192.168.102.124: icmp_seq=5 ttl=64 time=0.084 ms
64 bytes from 192.168.102.124: icmp_seq=6 ttl=64 time=0.094 ms
^C
--- 192.168.102.124 ping statistics ---
12 packets transmitted, 12 received, 0% packet loss, time 11131ms
rtt min/avg/max/mdev = 0.043/0.081/0.107/0.018 ms
[root@dev-xx-xx ~]# tc qdisc del dev eth0 root netem delay 100ms
[root@dev-xx-xx ~]# tc qdisc del dev eth0 root netem delay 100ms
RTNETLINK answers: Invalid argument
```
配置网络丢包率

```bash
[root@dev-xx-xx ~]# tc qdisc del dev eth0 root netem loss 10%
RTNETLINK answers: Invalid argument
[root@dev-xx-xx ~]# tc qdisc add dev eth0 root netem loss 10%
[root@dev-xx-xx ~]# tc qdisc show
qdisc netem 8005: dev eth0 root refcnt 5 limit 1000 loss 10%
[root@dev-xx-xx ~]# ping 192.168.102.124 -n 20
PING 20 (0.0.0.20) 56(124) bytes of data.
^C
--- 20 ping statistics ---
21 packets transmitted, 0 received, 100% packet loss, time 20650ms
 
[root@dev-xx-xx ~]# ping 192.168.102.124 -c 20
PING 192.168.102.124 (192.168.102.124) 56(84) bytes of data.
64 bytes from 192.168.102.124: icmp_seq=1 ttl=64 time=0.101 ms
64 bytes from 192.168.102.124: icmp_seq=2 ttl=64 time=0.062 ms
64 bytes from 192.168.102.124: icmp_seq=3 ttl=64 time=0.098 ms
64 bytes from 192.168.102.124: icmp_seq=4 ttl=64 time=0.098 ms
64 bytes from 192.168.102.124: icmp_seq=5 ttl=64 time=0.062 ms
64 bytes from 192.168.102.124: icmp_seq=6 ttl=64 time=0.088 ms
64 bytes from 192.168.102.124: icmp_seq=7 ttl=64 time=0.045 ms
64 bytes from 192.168.102.124: icmp_seq=8 ttl=64 time=0.070 ms
64 bytes from 192.168.102.124: icmp_seq=9 ttl=64 time=0.062 ms
64 bytes from 192.168.102.124: icmp_seq=10 ttl=64 time=0.066 ms
64 bytes from 192.168.102.124: icmp_seq=11 ttl=64 time=0.088 ms
64 bytes from 192.168.102.124: icmp_seq=12 ttl=64 time=0.070 ms
64 bytes from 192.168.102.124: icmp_seq=13 ttl=64 time=0.089 ms
64 bytes from 192.168.102.124: icmp_seq=14 ttl=64 time=0.087 ms
64 bytes from 192.168.102.124: icmp_seq=15 ttl=64 time=0.054 ms
64 bytes from 192.168.102.124: icmp_seq=16 ttl=64 time=0.085 ms
64 bytes from 192.168.102.124: icmp_seq=17 ttl=64 time=0.064 ms
64 bytes from 192.168.102.124: icmp_seq=18 ttl=64 time=0.124 ms
64 bytes from 192.168.102.124: icmp_seq=19 ttl=64 time=0.063 ms
64 bytes from 192.168.102.124: icmp_seq=20 ttl=64 time=0.108 ms
 
--- 192.168.102.124 ping statistics ---
20 packets transmitted, 20 received, 0% packet loss, time 19000ms
rtt min/avg/max/mdev = 0.045/0.079/0.124/0.020 ms
[root@dev-xx-xx ~]# tc qdisc del dev eth0 root netem loss 10%
[root@dev-xx-xx ~]# ping 192.168.102.124 -c 20
PING 192.168.102.124 (192.168.102.124) 56(84) bytes of data.
64 bytes from 192.168.102.124: icmp_seq=1 ttl=64 time=0.041 ms
64 bytes from 192.168.102.124: icmp_seq=2 ttl=64 time=0.132 ms
64 bytes from 192.168.102.124: icmp_seq=3 ttl=64 time=0.344 ms
64 bytes from 192.168.102.124: icmp_seq=4 ttl=64 time=0.404 ms
64 bytes from 192.168.102.124: icmp_seq=5 ttl=64 time=0.086 ms
64 bytes from 192.168.102.124: icmp_seq=6 ttl=64 time=0.088 ms
64 bytes from 192.168.102.124: icmp_seq=7 ttl=64 time=0.063 ms
64 bytes from 192.168.102.124: icmp_seq=8 ttl=64 time=0.109 ms
64 bytes from 192.168.102.124: icmp_seq=9 ttl=64 time=0.064 ms
64 bytes from 192.168.102.124: icmp_seq=10 ttl=64 time=0.092 ms
64 bytes from 192.168.102.124: icmp_seq=11 ttl=64 time=0.044 ms
64 bytes from 192.168.102.124: icmp_seq=12 ttl=64 time=0.066 ms
64 bytes from 192.168.102.124: icmp_seq=13 ttl=64 time=0.094 ms
64 bytes from 192.168.102.124: icmp_seq=14 ttl=64 time=0.097 ms
64 bytes from 192.168.102.124: icmp_seq=15 ttl=64 time=0.108 ms
64 bytes from 192.168.102.124: icmp_seq=16 ttl=64 time=0.043 ms
64 bytes from 192.168.102.124: icmp_seq=17 ttl=64 time=0.093 ms
64 bytes from 192.168.102.124: icmp_seq=18 ttl=64 time=0.056 ms
64 bytes from 192.168.102.124: icmp_seq=19 ttl=64 time=0.093 ms
64 bytes from 192.168.102.124: icmp_seq=20 ttl=64 time=0.039 ms
 
--- 192.168.102.124 ping statistics ---
20 packets transmitted, 20 received, 0% packet loss, time 18999ms
rtt min/avg/max/mdev = 0.039/0.107/0.404/0.093 ms
[root@dev-xx-xx ~]#
```


参考：
- [linux下使用tc(Traffic Control) 流量控制命令模拟网络延迟和丢包](https://www.cnblogs.com/yulia/p/10346339.html) 
