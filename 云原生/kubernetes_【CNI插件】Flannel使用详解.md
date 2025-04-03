




-----
## 1. 介绍
链接：[https://github.com/flannel-io/flannel](https://github.com/flannel-io/flannel)

CoreOS开发的项目Flannel，可能是最直接和最受欢迎的CNI插件。它是容器编排系统中最成熟的网络结构示例之一，旨在实现更好的容器间和主机间网络。随着CNI概念的兴起，Flannel CNI插件算是早期的入门。

与其他方案相比，Flannel相对容易安装和配置。它被打包为单个二进制文件flanneld，许多常见的Kubernetes集群部署工具和许多Kubernetes发行版都可以默认安装Flannel。Flannel可以使用Kubernetes集群的现有etcd集群来使用API存储其状态信息，因此不需要专用的数据存储。

Flannel配置第3层IPv4 overlay网络。它会创建一个大型内部网络，跨越集群中每个节点。在此overlay网络中，每个节点都有一个子网，用于在内部分配IP地址。在配置pod时，每个节点上的Docker桥接口都会为每个新容器分配一个地址。同一主机中的Pod可以使用Docker桥接进行通信，而不同主机上的pod会使用flanneld将其流量封装在UDP数据包中，以便路由到适当的目标。

Flannel有几种不同类型的后端可用于封装和路由。默认和推荐的方法是使用VXLAN，因为VXLAN性能更良好并且需要的手动干预更少。

总的来说，Flannel是大多数用户的不错选择。从管理角度来看，它提供了一个简单的网络模型，用户只需要一些基础知识，就可以设置适合大多数用例的环境。一般来说，在初期使用Flannel是一个稳妥安全的选择，直到你开始需要一些它无法提供的东西。

## 2. Flannel实现原理
### 2.1 原理说明
Flannel为每个host分配一个subnet，容器从这个subnet中分配IP，这些IP可以在host间路由，容器间无需使用nat和端口映射即可实现跨主机通信

每个subnet都是从一个更大的IP池中划分的，flannel会在每个主机上运行一个叫flanneld的agent，其职责就是从池子中分配subnet

Flannel使用etcd存放网络配置、已分配 的subnet、host的IP等信息

Flannel数据包在主机间转发是由backend实现的，目前已经支持UDP、VxLAN、host-gw、AWS VPC和GCE路由等多种backend

### 2.2 数据转发流程
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4239ec05b7763433d359d798b0646cf5.png)

 1. 容器直接使用目标容器的ip访问，默认通过容器内部的eth0发送出去。
 2. 报文通过veth pair被发送到vethXXX。
 3. vethXXX是直接连接到虚拟交换机docker0的，报文通过虚拟bridge docker0发送出去。
 4. 查找路由表，外部容器ip的报文都会转发到flannel0虚拟网卡，这是一个P2P的虚拟网卡，然后报文就被转发到监听在另一端的flanneld。
 5. flanneld通过etcd维护了各个节点之间的路由表，把原来的报文UDP封装一层，通过配置的iface发送出去。
 6. 报文通过主机之间的网络找到目标主机。
 7. 报文继续往上，到传输层，交给监听在8285端口的flanneld程序处理。
 8. 数据被解包，然后发送给flannel0虚拟网卡。
 9. 查找路由表，发现对应容器的报文要交给docker0。
 10. docker0找到连到自己的容器，把报文发送过去。


## 3. Flannel安装配置

### 3.1 环境准备
| 节点名称  | IP地址          | 软件环境                |
|-------|---------------|---------------------|
| etcd1 | 192.168.211. 61| etcd、flannel、docker |
| etcd2 | 192.168.211.61 | etcd、flannel、docker |
| etcd3 | 192.168.211.61 | etcd、flannel、docker |

### 3.2 安装etcd

关于etcd的安装使用已经在「[etcd使用入门](https://www.hi-linux.com/posts/40915.html)」和「[通过静态发现方式部署etcd集群](https://www.hi-linux.com/posts/49138.html)」中做了比较详细的讲解，如果你还不会安装etcd可先阅读下这两篇文章。这里就不再重复讲解了。

### 3.3 安装flannel

三个节点都需安装配置flannel，这里以etcd1节点为例。
下载地址：[https://github.com/flannel-io/flannel/releases](https://github.com/flannel-io/flannel/releases)

flannel和etcd一样，直接从官方下载二进制执行文件就可以用了。当然，你也可以自己编译。

```bash
$ curl -L https://github.com/coreos/flannel/releases/download/v0.13.0/flannel-v0.13.0-linux-amd64.tar.gz -o flannel.tar.gz
$ mkdir -p /opt/flannel
$ tar xzf flannel.tar.gz -C /opt/flannel
flanneld
mk-docker-opts.sh
README.md
```
解压后主要有`flanneld`、`mk-docker-opts.sh`这两个文件，其中flanneld为主要的执行文件，`sh`脚本用于生成Docker启动参数。

### 3.4 配置flannel

由于flannel需要依赖`etcd`来保证集群IP分配不冲突的问题，所以首先要在etcd中设置 flannel节点所使用的IP段。

```bash
$ etcdctl --endpoints "http://etcd1.hi-linux.com:2379" \
set /coreos.com/network/config '{"NetWork":"10.0.0.0/16", "SubnetMin": "10.0.1.0", "SubnetMax": "10.0.20.0"}'

{"NetWork":"10.0.0.0/16", "SubnetMin": "10.0.1.0", "SubnetMax": "10.0.20.0"}
```
`flannel`预设的`backend type`是`udp`，如果想要使用`vxlan`作为`backend`，可以加上backend参数：

```bash
$ etcdctl --endpoints "http://etcd1.hi-linux.com:2379" \
set /coreos.com/network/config '{"NetWork":"10.0.0.0/16", "Backend": {"Type": "vxlan"}}'
```
`flannel backend`为`vxlan`比起预设的`udp`性能相对好一些。

```bash
[root@master flannel]# etcdctl --endpoints "http://etcd1.hi-linux.com:2379" get  /coreos.com/network/config 
{"NetWork":"10.0.0.0/16", "Backend": {"Type": "vxlan"}}
```

### 3.5 启动flannel
命令行方式运行

```bash
$ /opt/flannel/flanneld --etcd-endpoints="http://etcd1.hi-linux.com:2379" --ip-masq=true >> /var/log/flanneld.log 2>&1 &
```
### 3.6 后台服务方式运行
给flannel创建一个systemd服务，方便以后管理。创建flannel配置文件:

```bash
$ cat <<EOF | sudo tee /etc/systemd/system/flanneld.service
[Unit]
Description=Flanneld
Documentation=https://github.com/coreos/flannel
After=network.target
Before=docker.service

[Service]
User=root
ExecStart=/opt/flannel/flanneld \
--etcd-endpoints="http://etcd1.hi-linux.com:2379,http://etcd2.hi-linux.com:2379,http://etcd3.hi-linux.com:2379" \
--iface=192.168.2.210 \
--ip-masq
Restart=on-failure
Type=notify
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```
注意：`--iface`参数为要绑定的网卡的IP地址，请根据实际情况修改。

### 3.7 启动flannel服务

```bash
$ systemctl start flanneld
```
flannel启动过程解析

flannel服务需要先于Docker启动。flannel服务启动时主要做了以下几步的工作：

 - 从etcd中获取network的配置信息。
 - 划分`subnet`，并在etcd中进行注册。
 - 将子网信息记录到`/run/flannel/subnet.env`中。

### 3.8 验证flannel网络

在etcd1节点上看etcd中的内容

```bash
$ etcdctl  --endpoints "http://etcd1.hi-linux.com:2379" ls /coreos.com/network/subnets

/coreos.com/network/subnets/10.0.2.0-24
```
查看`flannel0`的网络情况：

```bash
$ ifconfig flannel0
flannel0  Link encap:UNSPEC  HWaddr 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00
          inet addr:10.0.2.0  P-t-P:10.0.2.0  Mask:255.255.0.0
          UP POINTOPOINT RUNNING NOARP MULTICAST  MTU:1472  Metric:1
          RX packets:85 errors:0 dropped:0 overruns:0 frame:0
          TX packets:75 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:500
          RX bytes:7140 (7.1 KB)  TX bytes:6300 (6.3 KB)
```
可以看到`flannel0`网卡的地址和etcd中存储的地址一样，这样flannel网络配置完成。

### 3.9 配置Docker

在各个节点安装好以后最后要更改Docker的启动参数，使其能够使用flannel进行IP分配，以及网络通讯。

flannel运行后会生成一个环境变量文件，包含了当前主机要使用`flannel`通讯的相关参数。

#### 3.9.1 查看flannel分配的网络参数

```bash
$ cat /run/flannel/subnet.env

FLANNEL_NETWORK=10.0.0.0/16
FLANNEL_SUBNET=10.0.2.1/24
FLANNEL_MTU=1472
FLANNEL_IPMASQ=true
```

#### 3.9.2 创建Docker运行参数
使用flannel提供的脚本将`subnet.env`转写成Docker启动参数，创建好的启动参数位于`/run/docker_opts.env`文件中。

```bash
$ /opt/flannel/mk-docker-opts.sh -d /run/docker_opts.env -c
$ cat /run/docker_opts.env
DOCKER_OPTS=" --bip=10.0.2.1/24 --ip-masq=false --mtu=1472"
```
#### 3.9.3 修改Docker启动参数
修改docker的启动参数，并使其启动后使用由flannel生成的配置参数，修改如下:


```bash
# 编辑 systemd service 配置文件
$ vim /lib/systemd/system/docker.service
# 在启动时增加flannel提供的启动参数
ExecStart=/usr/bin/dockerd $DOCKER_OPTS
# 指定这些启动参数所在的文件位置(这个配置是新增的，同样放在Service标签下)
EnvironmentFile=/run/docker_opts.env
```

然后重新加载systemd配置，并重启Docker即可


```bash
$ systemctl daemon-reload
$ systemctl restart docker
```

此时可以看到docker0的网卡ip地址已经处于flannel网卡网段之内。


```bash
$ ifconfig flannel0
flannel0  Link encap:UNSPEC  HWaddr 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00
          inet addr:10.0.2.0  P-t-P:10.0.2.0  Mask:255.255.0.0
          UP POINTOPOINT RUNNING NOARP MULTICAST  MTU:1472  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:500
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

$ ifconfig docker0
docker0   Link encap:Ethernet  HWaddr 02:42:cf:87:3c:f7
          inet addr:10.0.2.1  Bcast:0.0.0.0  Mask:255.255.255.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

到此节点etcd1的flannel安装配置完成了，其它两节点按以上方法配置完成就行了。

## 4. 测试flannel
三台机器都配置好了之后，我们在三台机器上分别开启一个docker容器，测试它们的网络是否可相互联通的。

etcd1


```bash
$ docker run -it  busybox sh

# 查看容器IP
$ cat /etc/hosts
10.0.2.2	9de86bfde6cc
```

etcd2


```bash
$ docker run -it  busybox sh

# 查看容器IP
$ cat /etc/hosts
10.0.5.2	9ddd4a4e455b
```

etcd3


```bash
$ docker run -it  busybox sh

# 查看容器IP
$ cat /etc/hosts
10.0.6.2	cbb0d891f353
```

从不同宿主机容器到三台宿主机

```bash
/ # ping -c3 192.168.2.210
PING 192.168.2.210 (192.168.2.210): 56 data bytes
64 bytes from 192.168.2.210: seq=0 ttl=64 time=0.089 ms
64 bytes from 192.168.2.210: seq=1 ttl=64 time=0.065 ms

/ # ping -c5 192.168.2.211
PING 192.168.2.211 (192.168.2.211): 56 data bytes
64 bytes from 192.168.2.211: seq=0 ttl=63 time=1.712 ms
64 bytes from 192.168.2.211: seq=1 ttl=63 time=0.356 ms
64 bytes from 192.168.2.211: seq=2 ttl=63 time=2.201 ms

/ # ping -c3 192.168.2.212
PING 192.168.2.212 (192.168.2.212): 56 data bytes
64 bytes from 192.168.2.212: seq=0 ttl=63 time=0.467 ms
64 bytes from 192.168.2.212: seq=1 ttl=63 time=0.477 ms
64 bytes from 192.168.2.212: seq=2 ttl=63 time=0.532 ms
```

从容器到到跨宿主机容器

```bash
/ # ping -c3  10.0.5.2
PING 10.0.5.2 (10.0.5.2): 56 data bytes
64 bytes from 10.0.5.2: seq=0 ttl=60 time=0.692 ms
64 bytes from 10.0.5.2: seq=1 ttl=60 time=0.565 ms
64 bytes from 10.0.5.2: seq=2 ttl=60 time=1.135 ms

/ # ping -c3  10.0.6.2
PING 10.0.6.2 (10.0.6.2): 56 data bytes
64 bytes from 10.0.6.2: seq=0 ttl=60 time=0.678 ms
64 bytes from 10.0.6.2: seq=1 ttl=60 time=0.907 ms
64 bytes from 10.0.6.2: seq=2 ttl=60 time=1.272 ms

/ # ping -c3 10.0.2.2
PING 10.0.2.2 (10.0.2.2): 56 data bytes
64 bytes from 10.0.2.2: seq=0 ttl=60 time=0.644 ms
64 bytes from 10.0.2.2: seq=1 ttl=60 time=0.915 ms
64 bytes from 10.0.2.2: seq=2 ttl=60 time=1.032 ms
```

测试容器到到跨宿主机容器遇到一个坑，开始怎么都不通，后找到原因是宿主机iptables给阻挡掉了。附：Ubuntu一键清除iptables规则脚本


```bash
$ cat clear_iptables_rule.sh

#!/bin/bash

iptables -F
iptables -X
iptables -Z
iptables -P INPUT ACCEPT
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT
```
参考连接：
[奇妙的 Linux 世界](https://www.hi-linux.com/posts/30481.html)


