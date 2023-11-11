#  Linux Command ip
tags: 网络


##  1. ip
```bash
$  ip [option] [动作] [命令]**
```

参数：
option ：设定的参数，主要有：
    

 - -s ：显示出该设备的统计数据(statistics)，例如总接受封包数等；   动作：就是是可以针对哪些网络参数进行动作，包括有：

       link ：关于设备 (device) 的相关设定，包括 MTU, MAC 地址等等
       addr/address ：关于额外的 IP 设定，例如多 IP 的实现等等；
       route ：与路由有关的相关设定 
*******************************************************************

## 2. ip link

关于设备(device) 的相关设定： `ip link` 
ip link 可以设定与设备 (device) 有关的相关设定，包括 MTU 以及该网络设备的 MAC 等等， 当然也可以启动 (up) 或关闭 (down) 某个网络设备。整个语法是这样的：

```bash
$ ip [-s] link show <== 单纯的查阅该设备相关的信息
$ ip link set [device] [动作与参数]
```

参数：

 - show：仅显示出这个设备的相关内容，如果加上 -s 会显示更多统计数据；
 - set ：可以开始设定项目， device 指的是 eth0, eth1 等等设备代号；

动作与参数：包括以下动作：
   

 1. up|down ：启动 (up) 或关闭 (down) 某个设备，其他参数使用预设的以太网参数；
 1. address ：如果这个设备可以更改 MAC ，用这个参数修改；
  
 1. name     ：给予这个设备一个特殊的名字；
 1. mtu      ：设置最大传输单元。

```bash
$ ip link show 
$ ip -s link show eth0 
ip link set eth0 up$ 启动eth0这个设备
$  ip link set eth0 down      $ 关闭eth0这个设备
$ ip link set eth0 mtu 1000  $ 更改 MTU为1000 bytes，单位就是 bytes
```

使用 ip link show 可以显示出整个设备的硬件相关信息，如上所示，包括 MAC地址、MTU等等， 比较有趣的应该是那个 sit0 的设备了，那个 sit0 的设备是将IPv4 和 IPv6 的封包进行转换， 对于我们仅使用 IPv4 的网络是没有作用的。 lo 及 sit0 都是主机内部自行设定的。 而如果加上 -s 的参数后，则这个网卡的相关统计信息就会被列出来， 包括接收 (RX) 及传送 (TX) 的封包数量等等，详细的内容与 ifconfig 所输出的结果相同。

更新网卡的 MTU 使用 ifconfig 也可以实。如果是要更改『网卡代号、 MAC 地址的信息』的话，那可就得使用 ip了，设定前需要先关闭该网卡，否则会不成功
修改网卡代号、MAC 等参数

```bash
$ ip link set eth0 name vbird $不会成功
$ ip link set eth0 down       <==关闭设备
$ ip link set eth0 name vbird <==重新设定
$ ip link show                <==查看信息
```

因为我们的 ifcfg-eth0 还是使用原本的设备代号！避免有问题，要改回来

```bash
$ ip link set vbird name eth0 <==设备改回来 
$ ip link set eth0 address aa:aa:aa:aa:aa:aa
$ ip link show eth0
```

关闭一个网络设备

```bash
$ ip link set dev ens33 down$ 等同于ifconfig ens33 down
```

修改网络设备的MTU

```bash
$ip link set dev ens33 mtu 1500
```

## 3. ip address

关于额外的 IP 相关设定： ip address 
如果说 ip link 是与 OSI 七层模型的第二层数据链路层有关的话，那么 ip address (ip addr) 就是与第三层网络层有关的了。主要是在设定与 IP 有关的各项参数，包括 netmask, broadcast 等等。

```bash
$ ip address show   <==查看IP参数
$ ip address [add|del] [IP参数] [dev 设备名] [相关参数]
```

参数：

 - show  ：单纯的显示出设备的 IP 信息；

add|del ：进行相关参数的增加 (add) 或删除 (del) 设定，主要有：
IP 参数 ：主要就是网域的设定，例如 192.168.100.100/24 之类的设定；
dev ：这个 IP 参数所要设定的设备，例如 eth0, eth1 等等；
 相关参数：如下所示：
        broadcast：设定广播位址，如果设定值是 + 表示让系统自动计算；
        label    ：该设备的别名，例如eth0:0；
        scope    ：这个设备的领域，通常是以下几个大类：
                   global ：允许来自所有来源的连线；
                   site   ：仅支持IPv6 ，仅允许本主机的连接；
                   link   ：仅允许本设备自我连接；
                   host   ：仅允许本主机内部的连接；
                   所以当然是使用 global 了。预设也是 global ！

```bash
$ ip address show    $显示出所有设备的 IP 参数
$ ifconfig
$ ip address add 192.168.50.50/24 broadcast + > dev eth0 label eth0:vbird                                   新增一个设备，名称假设为 eth0:vbird 
$ ip address show eth0
inet 192.168.1.100/24 brd 192.168.1.255 scope global eth0
  inet 192.168.50.50/24 brd 192.168.50.255 scope global eth0:vbird
看上面的输出多出了一行，增加了新的设备，名称是 eth0:vbird
# 至于那个 broadcast + 也可以写成 broadcast 192.168.50.255 
$ ip address del 192.168.50.50/24 dev eth0  $将刚刚的设备删除 
```

## 4. ip route 
这个项目就是路由的查看与设定。事实上ip route 的功能几乎与 route 这个命令一样，但是，它还可以进行额外的参数设置，例如 MTU 的规划等等，相当的强悍啊！
```bash
$ ip route show <==单纯的显示出路由的设定
$ ip route [add|del] [IP或网域] [via gateway] [dev 设备]
```

参数：

 - show ：单纯的显示出路由表，也可以使用 list ；
 - add|del ：增加 (add) 或删除 (del) 路由；
 - IP或网域：可使用 192.168.50.0/24 之类的网域或者是单纯的 IP ；
 - via：从那个 gateway 出去，不一定需要；
 - dev：由那个设备连出去，需要；
 - mtu：可以额外的设定 MTU 的数值；

```bash
$ ip route show  $显示出目前的路由资料
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.2
169.254.0.0/16 dev eth1 scope link
default via 192.168.1.254 dev eth1 
```
```bash

 - proto：此路由的路由协定，主要有 redirect, kernel, boot, static, ra 等， 其中 kernel指的是直接由核心判断自动设定。
 - scope：路由的范围，主要是 link ，是与本设备有关的直接连接。

增加路由，主要是本机直接可沟通的网域
$ ip route add 192.168.5.0/24 dev eth0
# 针对本机直接沟通的网域设定好路由，不需要透过外部的路由器
$ ip route show
192.168.5.0/24 dev eth0 scope link

增加可以通往外部的路由，需透过 router ；
$ ip route add 192.168.10.0/24 via 192.168.5.100 dev eth0
$ ip route show
192.168.5.0/24 dev eth0 scope link。。。。。
192.168.10.0/24 via 192.168.5.100 dev eth0

增加预设路由
$ ip route add default via 192.168.1.2 dev eth0
# 那个 192.168.1.2 就是我的预设路由器(gateway)

删除路由
$ ip route del 192.168.10.0/24
$ ip route del 192.168.5.0/24 
```

## 5. ip netns
1.增加虚拟网络命名空间

```bash
$ ip netns add net0
```

2.显示所有的虚拟网络命名空间

```bash
# ip netns list
net0
也可通过查看/var/run/netns目录下的文件来list
EULER:~$ ls /var/run/netns/
net0
```
3.进入虚拟机网络环境


```bash
ip netns exec net0 `command`
如
$ ip netns exec net0 bash $打开虚拟网络环境net0的bash窗口
$ ip addr$显示所有虚拟网络环境的设备
$ exit$退出该网络虚拟环境
exit
```

4.增加一对veth虚拟网卡

```bash
$ ip link add type veth
```

5.将veth0添加到net0虚拟网络环境

```bash
ip link set veth0 netns net0
```

6.将虚拟网卡veth1改名并添加到net1虚拟网络环境中

```bash
ip link set dev veth1 name net1-bridge netns net1
```

7.设置虚拟网络环境net0的veth0设备处于激活状态

```bash
ip netns exec net0 ip link set veth0 up
```

8.为虚拟网络环境net0的veth0设备增加IP地址

```bash
ip netns exec net0 ip address add 10.0.1.1/24 dev veth0
ip link set br0 netns ns1$将br0添加到ns1虚拟网络环境
ethtool -k br0   $查看设备转移
ip netns exec netns1 ethtool -S veth1  $查询veth设备对端口在设备列表中的序列号
```


```bash
ip r
ip l
ip -6 a
```

 - IP 命令[3] 类似于 ifconfig，常用于配置静态 IP 地址、路由 & 默认网关，等等。
 - ifconfig 命令因为多年没有维护而被遗弃了，即使它仍然在大多数 Linux 发行版上可获得。
 - ifconfig 命令已经被 ip 命令替代了，ip 命令是非常强大的，只要一个命令就能执行几个网络管理任务。
 - ip 命令工具附带在 `iproute2` 包中。在主要的 Linux 发行版中都默认预装了 iproute2 。

如果没有，你可以在你的终端中在包管理器的帮助下通过指定 iproute2 来安装它。

```bash
$ ip r
或
$ ip route
或
$ ip route show
default via 192.168.1.1 dev wlp8s0 proto dhcp metric 600
192.168.1.0/24 dev wlp8s0 proto kernel scope link src 192.168.1.6 metric 600
```

##  6. 更多ip相关命令

 - ip
 - ip6tables
 - ip6tables-restore
 - ip6tables-save
 - ipcalc
 - ipcmk
 - ipcrm
 - ipcs
 - ipmaddr
 - iprconfig
 - iprdbg
 - iprdump
 - iprinit
 - iprsos
 - iprupdate
 - ipset
 - iptables
 - iptables-restore
 - iptables-save


参考：

 - [How to find your IP address in Linux](https://opensource.com/article/18/5/how-find-ip-address-linux)
 - [Ways to Find IP address in Linux](https://www.howtouselinux.com/post/check-ip-address-in-linux)
 - [Linux ip Command Examples](https://www.cyberciti.biz/faq/linux-ip-command-examples-usage-syntax/)

