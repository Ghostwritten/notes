#  Linux Commnad iptables 防火墙




------------
[!NOTE|style:flat|lable:Mylable|iconVisibility:hidden]
iptables service 在 /etc/sysconfig/iptables 中储存配置,而 firewalld将配置储存在
/usr/lib/firewalld/ 和 /etc/firewalld/ 中的各种XML

## 1. 默认的四张表

```bash
1、filter:用于防火墙，默认有INPUT/OUTPUT/FORWARD三条链
2、nat:网络地址转换，PREROUTING/POSTROUTING/OUTPUT三条链
3、mangle：流量整形，五条链
4、raw：用于状态跟踪
```

## 2. 默认的五条规则链

```bash
1、INPUT: 如果一个数据包的目的地址是LINUX本身，则进入INPUT链
2、OUTPUT: 源地址是LINUX本身
3、FORWARD: 数据包从一块网卡接收，从另一块网卡发出，经过LINUX的包，进入这条链
4、PREROUTING: 路由前
5、POSTROUTING: 路由后
```

##  3. 选项

```bash
-t, --table table 对指定的表 table 进行操作， table 必须是 raw， nat，filter，mangle 中的一个。如果不指定此选项，默认的是 filter 表。

# 通用匹配：源地址目标地址的匹配
-p：指定要匹配的数据包协议类型；
-s, --source [!] address[/mask] ：把指定的一个／一组地址作为源地址，按此规则进行过滤。当后面没有 mask 时，address 是一个地址，比如：192.168.1.1；当 mask 指定时，可以表示一组范围内的地址，比如：192.168.1.0/255.255.255.0。
-d, --destination [!] address[/mask] ：地址格式同上，但这里是指定地址为目的地址，按此进行过滤。
-i, --in-interface [!] <网络接口name> ：指定数据包的来自来自网络接口，比如最常见的 eth0 。注意：它只对 INPUT，FORWARD，PREROUTING 这三个链起作用。如果没有指定此选项， 说明可以来自任何一个网络接口。同前面类似，"!" 表示取反。
-o, --out-interface [!] <网络接口name> ：指定数据包出去的网络接口。只对 OUTPUT，FORWARD，POSTROUTING 三个链起作用。

# 查看管理命令
-L, --list [chain] 列出链 chain 上面的所有规则，如果没有指定链，列出表上所有链的所有规则。

# 规则管理命令
-A, --append chain rule-specification 在指定链 chain 的末尾插入指定的规则，也就是说，这条规则会被放到最后，最后才会被执行。规则是由后面的匹配来指定。
-I, --insert chain [rulenum] rule-specification 在链 chain 中的指定位置插入一条或多条规则。如果指定的规则号是1，则在链的头部插入。这也是默认的情况，如果没有指定规则号。
-D, --delete chain rule-specification -D, --delete chain rulenum 在指定的链 chain 中删除一个或多个指定规则。
-R num：Replays替换/修改第几条规则

# 链管理命令（这都是立即生效的）
-P, --policy chain target ：为指定的链 chain 设置策略 target。注意，只有内置的链才允许有策略，用户自定义的是不允许的。
-F, --flush [chain] 清空指定链 chain 上面的所有规则。如果没有指定链，清空该表上所有链的所有规则。
-N, --new-chain chain 用指定的名字创建一个新的链。
-X, --delete-chain [chain] ：删除指定的链，这个链必须没有被其它任何规则引用，而且这条上必须没有任何规则。如果没有指定链名，则会删除该表中所有非内置的链。
-E, --rename-chain old-chain new-chain ：用指定的新名字去重命名指定的链。这并不会对链内部造成任何影响。
-Z, --zero [chain] ：把指定链，或者表中的所有链上的所有计数器清零。

-j, --jump target <指定目标> ：即满足某条件时该执行什么样的动作。target 可以是内置的目标，比如 ACCEPT，也可以是用户自定义的链。
-h：显示帮助信息；
--state 匹配一组连接状态
--string 匹配应用层数据字节序列
--comment 注释数据
--probability 0.50000000000  -j KUBE-SEP-ID6YWIT3F6WNZ47P  使连接有50%的概率进入到KUBE-SEP-ID6YWIT3F6WNZ47P链
```

## 4. 规则

### 4.1 编写规则：


### 4.2 添加规则:

 - -A 在链的末尾追加一条规则
 - -I 在链的开头插入一条规则
 - -L 列出所有的规则条目

### 4.3 查看规则:

 - -n 以数字形式显示地址，端口等信息
 - -line-numbers 查看规则时，显示规则的序号

### 4.4 删除规则：

 - -D 删除链内指定序号的一条规则
 - -F 清空所有的规则
 - -P 为指定的链设置默认规则

## 5.  iptables实例

1、拒绝PING防火墙本身

```bash
$ iptables  -F
$ iptables  -A  INPUT  -p  icmp  -j  DROP
```

2、ping  linux的IP地址
3、把防火墙规则再清空，把拒绝ICMP的目标规则改为REJECT，再次ping测试，看结果。
4、查看防火墙规则

```bash
$  iptables  -nvL
```

5、允许指定IP地址的主机PING防火墙

```bash
$  iptables  -I  INPUT  1  -s  192.168.194.1  -p  icmp  -j  ACCEPT
```

6、查看防火墙规则，每个规则注明序号

```bash
$  iptables  -nL  --line-numbers
```

7、删除防火墙的INPUT链中第二条规则

```bash
$ iptables  –D  INPUT  2
```

8、把OUTPUT的默认规则改为DROP

```bash
$  iptables  -P  OUTPUT  DROP
```

```bash
# 限制syn并发数为每秒1次
$ iptables -A INPUT -p tcp --syn -m limit --limit 1/s -j ACCEPT

# 限制单个IP在60秒新建立的连接数为10
$ iptables -I INPUT -p tcp --dport 80 --syn -m recent --name SYN_FLOOD --update --seconds 60 --hitcount 10 -j REJECT
```

### 5.1  规则的基本匹配条件

1、允许特定IP地址访问LINUX的telnet服务

```bash
$  yum  install  -y  telnet-server
$ service  xinetd  start
$ chkconfig  telnet  on
$ iptables  -P  INPUT  DROP
$ iptables  -A  INPUT  -s  192.168.194.1  -p  tcp  --dport  23  -j  ACCEPT
```

2、继续第1步。向INPUT链插入规则，作为第一条。从eth0网卡收到的、访问telnet服务的数据包，拒绝。

```bash
$ iptables  -I  INPUT  -i  eth0  -p  tcp  --dport  23  -j  REJECT
```

### 5.2 打开LINUX路由功能（转发功能）
IS: 中间系统，路由器。
ES: 终端系统，主机系统。
1、临时打开路由功能

```bash
$ echo  1  > /proc/sys/net/ipv4/ip_forward
$ cat  /proc/sys/net/ipv4/ip_forward
1
```

2、永久打开转发功能

```bash
$  echo  ‘echo  1  > /proc/sys/net/ipv4/ip_forward’  >>  /etc/rc.local
或
$  vim  /etc/sysctl.conf
net.ipv4.ip_forward = 1
$ sysctl  -p
```


### 5.3 使用FORWARD链
1、拓扑

2、拒绝192.168.195.0/24网段访问192.168.194.0/24网段的telnet服务

```bash
$ iptables -A FORWARD -s 192.168.195.0/24 -d 192.168.194.0/24 -p tcp --dport 23  -i  eth1  -o  eth0  -j  REJECT
```

3、从拒绝SSH协议通过防火墙

```bash
$ iptables  -A  FORWARD  -p  tcp  --dport  22  -j  REJECT
```

4、不是192.168.195.0/24网段的主机访问SSH服务，可通过

```bash
$ iptables  -I  FORWARD  !  -s  192.168.195.0/24  -p  tcp  --dport 22  -j  ACCEPT
```

5、防火墙拒绝icmp的请求

```bash
$ iptables  -A  INPUT  -p  icmp  --icmp-type  echo-request  -j  REJECT
```

6、允许192.168.195.0/24网段进行PING

```bash
$ iptables  -I  INPUT  -s  192.168.195.0/24  -p  icmp  --icmp-type  echo-request  -j ACCEPT
```

7、防火墙拒绝发送echo-reply

```bash
$ iptables  -A  OUTPUT  -p  icmp  --icmp-type  echo-reply  -j  REJECT
```

8、允许防火墙回应192.168.195.0/24网段的ping

```bash
$ iptables  -I  OUTPUT  -s  192.168.195.0/24  -p  icmp  --icmp-type  echo-reply  -j ACCEPT
```

9、防火墙拒绝192.168.195.0/24对其进行TCP连接。
检查SYN/ACK/RST/FIN四个位置，其中SYN被置位

```bash
$ iptables  -I  INPUT  -s  192.168.195.0/24  -p  tcp  --tcp-flags  SYN,ACK,RST,FIN  SYN  -j  REJECT
```

### 5.4 扩展匹配


###  5.5 按数据包速率和状态匹配
`-m state --state <状态>` ，网络连接的五种状态

 - `NEW`，请求建立连接的包、完全陌生的包
 - `ESTABLISHED`，将要或已经建立连接的包
 - `RELATED`，与已知某个连接相关联的包
 - `INVALID`，无对应连接，以及连接无效的包
 - `UNTRACKED`，未跟踪状态的包

```bash
-m limit --limit 匹配速率 如： -m limit --limit 50/s -j ACCEPT
-m state --state 状态 如： -m state --state INVALID,RELATED -j ACCEPT
```

2、192.168.194.0/24作为内网，192.168.195.0/24作为外网。从内到外的访问不受限制，从外到内的主动连接全部拒绝。

```bash
$ iptables  -A  FORWARD  -s  192.168.194.0/24  -j  ACCEPT
```

```c
$ iptables  -A  FORWARD  -d  192.168.194.0/24  -j  REJECT
```

```bash
$ iptables  -I  FORWARD  2  -d  192.168.194.0/24  \
 -m  state  --state  ESTABLISHED,RELATED  -j  ACCEPT
```
3、减轻DOS（拒绝服务）攻击
4、免状态跟踪

###  4.6 还可以限制链接数
-m connlimit --connlimit-above n 限制为多少个

```bash
  //表示限制链接数最大为9个   
 iptables -I FORWARD -p tcp -m connlimit --connlimit-above 9 -j DROP      

//表示访问80端口服务最大为10个链接  
iptable -A INPUT -p tcp -- dport 80 -m connlimit --connlimit-above 10 -j REJECT
```


###  4.7 模拟随机丢包率

```bash
iptables -A FORWARD -p icmp -m statistic --mode random --probability 0.31  -j REJECT   //表示31%的丢包率

-m random --average 5 -j DROP 表示模拟丢掉5%比例的包
```

##  6. raw
追踪数据包
```bash
# 在宿主机上执行
$ iptables -t raw -A OUTPUT -p icmp -j TRACE
$ iptables -t raw -A PREROUTING -p icmp -j TRACE
```





更多阅读：

 - [iptables 命令](https://wangchujiang.com/linux-command/c/iptables.html)
 - [iptables-extensions](https://ipset.netfilter.org/iptables-extensions.man.html)
 - [iptables(8) - Linux man page](https://linux.die.net/man/8/iptables)
 - [Sysadmin tools: How to use iptables](https://www.redhat.com/sysadmin/iptables)

