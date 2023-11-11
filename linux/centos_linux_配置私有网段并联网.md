

## 1. 创建虚拟机

vcenter 创建两台虚拟机A 、B，如何创建虚拟机请参考[这里](https://ghostwritten.blog.csdn.net/article/details/129310311)

虚拟机 A 具备两个网络接口，外网接口为 `ens192` ip：`192.168.22.6/20`，网关为`192.168.21.1`，另一个接口配置私有网段 `10.60.254.254/16` 

10.60.254.254./16 可用地址范围
![在这里插入图片描述](https://img-blog.csdnimg.cn/424ccab5c4754f10bbcff515e7e7d597.png)


我们在虚拟机添加网卡

![在这里插入图片描述](https://img-blog.csdnimg.cn/fb36257f4286435dbfd890cbb72f010a.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/f419ba4693ea4194bbead79931f9154d.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/befa145a07914827b5861a8cf1fc997a.png)




## 2. 虚拟机 A 配置网络
首先，假设虚拟机 A 的 IP 地址为 192.168.22.6，网关为 192.168.21.1，外网接口为 ens192。将私有网段 10.60.254.254/16 与外网连接起来。

配置静态地址 ens192

```bash
$ cat /etc/sysconfig/network-scripts/ifcfg-ens192
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=eui64
NAME=ens192
DEVICE=ens192
ONBOOT=yes
IPADDR=192.168.22.6
PREFIX=20
GATEWAY=192.168.21.1
DNS1=8.8.8.8
```
配置第二张网卡 ens224

```bash
$ cat /etc/sysconfig/network-scripts/ifcfg-ens224
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=eui64
NAME=ens224
DEVICE=ens224
ONBOOT=yes
IPADDR=10.60.254.254
PREFIX=16
DNS1=8.8.8.8
```
重启生效

```bash
nmcli c reload && nmcli c down ens192 && nmcli c up ens192
nmcli c reload && nmcli c down ens224 && nmcli c up ens224
```


这个命令将会将私有网段 10.60.0.0/24 的数据包通过虚拟路由转发到虚拟机 A 上。

```bash
iptables -t nat -A POSTROUTING -s 10.60.254.254/16 -o ens192 -j MASQUERADE
```

为iptables 持久化
保存防火墙规则

```bash
iptables-save > /etc/sysconfig/iptables
```

设定开机自动恢复iptables规则

```bash
$ vim /etc/rc.d/rc.local
iptables-restore < /etc/sysconfig/iptables
```

> 注意：如果您使用了firewalld服务，则需要停止并禁用它以避免与 iptables NAT规则冲突。您可以通过运行以下命令来停止和禁用firewalld服务：

```bash
sudo systemctl stop firewalld.service
sudo systemctl disable firewalld.service
```


配置路由转发

```bash
$ vim  /etc/sysctl.conf
net.ipv4.ip_forward = 1
```

## 3. 虚拟机 B 分配静态地址
现在，我们可以为虚拟机 B 分配一个静态 IP 地址，让它可以连接到刚才划分出的私有网段中。假设我们需要为虚拟机 B 分配 IP 地址 `10.60.0.2`，可以在虚拟机 B 的网络设置中将其设置为静态地址。


```bash
$ cat /etc/sysconfig/network-scripts/ifcfg-ens192
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=eui64
NAME=ens192
DEVICE=ens192
ONBOOT=yes
IPADDR=10.60.0.2
PREFIX=16
GATEWAY=10.60.254.254
DNS1=8.8.8.8
```

重启生效

```bash
nmcli c reload && nmcli c down ens192 && nmcli c up ens192
```


## 4. 测试 
虚拟机B

```bash
ping 10.60.254.254
ping www.baidu.com
```
虚拟机A

```bash
ping 10.60.0.2
```

参考：
- [Rocky Linux 9.1 新手入门指南](https://ghostwritten.blog.csdn.net/article/details/129644123)
- [CentOS下实现iptables持久化](https://www.linuxprobe.com/centos-iptables.html)




