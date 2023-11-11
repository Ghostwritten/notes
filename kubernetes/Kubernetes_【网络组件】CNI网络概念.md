![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401111439531.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_centor)





---
本文将在介绍技术原理和相应术语的基础上，再集中探索与详细对比目前最流行的CNI插件：Flannel、Calico、Weave和Canal，对比介绍它们的原理、使用方法、适用场景和优缺点等。


## 1. Docker网络模式

容器网络是容器选择连接到其他容器、主机和外部网络（如Internet）的机制。容器的runtime提供了各种网络模式，每种模式都会产生不同的体验。例如，Docker采用插件化的网络模式，默认提供bridge、host、none、overlay、maclan和Network plugins这几种网络模式，运行容器时可以通过–network参数设置具体使用那一种模式：

 1. `bridge`：这是Docker默认的网络驱动，此模式会为每一个容器分配Network Namespace和设置IP等，并将容器连接到一个虚拟网桥上，每个容器可以通过IP地址相互连接。如果未指定网络驱动，这默认使用此驱动。
 2. `host`：此网络驱动直接使用宿主机的网络，没有隔离。
 3. `none`：此驱动不构造网络环境。采用了none网络驱动，那么就只能使用loopback网络设备，容器只能使用127.0.0.1的本机网络。
 4. `overlay`：此网络驱动可以使多个Docker daemons连接在一起，并能够使用swarm服务之间进行通讯。也可以使用overlay网络进行swarm服务和容器之间、容器之间进行通讯，
 5. `macvlan`：此网络允许为容器指定一个MAC地址，允许容器作为网络中的物理设备，这样Docker
    daemon就可以通过MAC地址进行访问的路由。对于希望直接连接网络网络的遗留应用，这种网络驱动有时可能是最好的选择。
 6. `Network plugins`：可以安装和使用第三方的网络插件。可以在Docker Store或第三方供应商处获取这些插件:flannel等。

在默认情况，Docker使用bridge网络模式，bridge网络驱动的示意图如下，我们先以bridge模式对Docker的网络进行说明。

自定义网桥：用户定义的网桥，具有更多的灵活性、隔离性和其他便利功能。
Docker还可以让用户通过其他驱动程序和插件，来配置更高级的网络（包括多主机覆盖网络）。

### 1.2 bridge网络

 1. 安装Docker时，创建一个名为docke0的虚拟网桥，虚拟网桥使用“`10.0.0.0 -10.255.255.255`   
    “、”`172.16.0.0-172.31.255.255`″和“`192.168.0.0——192.168.255.255`”这三个私有网络的地址范围。
 2. 通过 `ifconfig` 命令可以查看docker0网桥的信息；
 3. 通过 `docker network inspect bridge` 可以查看网桥的子网网络范围和网关；
 4. 运行容器时，在宿主机上创建虚拟网卡`veth pair`设备，veth pair设备是成对出现的，从而组成一个数据通道，数据从一个设备进入，就会从另一个设备出来。将veth pair设备的一端放在新创建的容器中，命名为`eth0`；另一端放在宿主机的docker0中，以`veth`为前缀的名字命名。通过 `brctl show` 命令查看放在docker0中的veth pair设备；
 5. `bridge`的docker0是虚拟出来的网桥，因此无法被外部的网络访问。因此需要在运行容器时通过`-p`和`-P`参数对将容器的端口映射到宿主机的端口。实际上Docker是采用 NAT的 方式，将容器内部的服务监听端口与宿主机的某一个端口port 进行绑定，使得宿主机外部可以将网络报文发送至容器

```bash
通过-P参数，将容器的端口映射到宿主机的随机端口：
$ docker run -P {images}
通过-p参数，将容器的端口映射到宿主机的制定端口：
$ docker run -p {hostPort}:{containerPort}
```
可以修改docker的默认网段：

```bash
第一步 删除原有配置
sudo service docker stop
sudo ip link set dev docker0 down
sudo brctl delbr docker0
sudo iptables -t nat -F POSTROUTING

第二步 创建新的网桥
sudo brctl addbr docker0
sudo ip addr add 172.17.10.1/24 dev docker0
sudo ip link set dev docker0 up

第三步 配置Docker的文件
注意： 这里是 增加下面的配置
vi /etc/docker/daemon.json
##追加的即可
cat /etc/docker/daemon.json  
{"registry-mirrors": ["http://224ac393.m.daocloud.io"],
            "bip": "172.17.10.1/24"
}
$ systemctl  restart  docker
```


## 2. CNI介 绍
首先我们介绍一下什么是 CNI，它的全称是 Container Network Interface，即容器网络的 API 接口，它是一种标准的设计，为了让用户在容器创建或销毁时都能够更容易地配置容器网络。在本文中，我们将集中探索与对比目前最流行的CNI插件：Flannel、Calico、Weave和Canal（技术上是多个插件的组合）。这些插件既可以确保满足Kubernetes的网络要求，又能为Kubernetes集群管理员提供他们所需的某些特定的网络功能。


CNI的初衷是创建一个框架，用于在配置或销毁容器时动态配置适当的网络配置和资源。下面链接中的CNI规范概括了用于配制网络的插件接口，这个接口可以让容器运行时与插件进行协调：

[https://github.com/containern...](https://github.com/containernetworking/cni/blob/master/SPEC.md)

插件负责为接口配置和管理IP地址，并且通常提供与IP管理、每个容器的IP分配、以及多主机连接相关的功能。容器运行时会调用网络插件，从而在容器启动时分配IP地址并配置网络，并在删除容器时再次调用它以清理这些资源。

运行时或协调器决定了容器应该加入哪个网络以及它需要调用哪个插件。然后，插件会将接口添加到容器网络命名空间中，作为一个veth对的一侧。接着，它会在主机上进行更改，包括将veth的其他部分连接到网桥。再之后，它会通过调用单独的IPAM（IP地址管理）插件来分配IP地址并设置路由。

## 3. 如何使用 CNI
在Kubernetes中，kubelet可以在适当的时间调用它找到的插件，来为通过kubelet启动的pod进行自动的网络配置。
基本的使用方法为：

 1. 首先在每个结点上配置 CNI 配置文件(`/etc/cni/net.d/xxnet.conf`)，其中 xxnet.conf是某一个网络配置文件的名称；
 2. 安装 CNI 配置文件中所对应的二进制插件；
 3. 在这个节点上创建 Pod 之后，Kubelet 就会根据 CNI 配置文件执行前两步所安装的 CNI 插件；上步执行完之后，Pod 的网络就配置完成了。
 
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210406115029415.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
在集群里面创建一个 Pod 的时候，首先会通过 `apiserver` 将 Pod 的配置写入。apiserver 的一些管控组件（比如 `Scheduler`）会调度到某个具体的节点上去。`Kubelet` 监听到这个 Pod 的创建之后，会在本地进行一些创建的操作。当执行到创建网络这一步骤时，它首先会读取刚才我们所说的配置目录中的配置文件，配置文件里面会声明所使用的是哪一个插件，然后去执行具体的 CNI 插件的二进制文件，再由 CNI 插件进入 Pod 的网络空间去配置 Pod 的网络。配置完成之后，Kuberlet 也就完成了整个 Pod 的创建过程，这个 Pod 就在线了。

## 4. 哪个 CNI 插件适合我
这就要从 CNI 的几种实现模式说起。我们需要根据不同的场景选择不同的实现模式，再去选择对应的具体某一个插件。
通常来说，CNI 插件可以分为三种：**Overlay、路由及 Underlay**。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021040615100496.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

 1. `Overlay` 模式的典型特征是容器独立于主机的 IP 段，这个 IP段进行跨主机网络通信时是通过在主机之间创建隧道的方式，将整个容器网段的包全都封装成底层的物理网络中主机之间的包。该方式的好处在于它不依赖于底层网络；
 2. `路由模式`中主机和容器也分属不同的网段，它与 Overlay模式的主要区别在于它的跨主机通信是通过路由打通，无需在不同主机之间做一个隧道封包。但路由打通就需要部分依赖于底层网络，比如说要求底层网络有二层可达的一个能力；
 3. `Underlay`模式中容器和宿主机位于同一层网络，两者拥有相同的地位。容器之间网络的打通主要依靠于底层网络。因此该模式是强依赖于底层能力的。

其他相关选择CNI因素
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210406151231304.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)


## 5. 如何理解集群网络

> CIDR：无类别域间路由（Classless Inter-Domain Routing、CIDR）是一个用于给用户分配IP地址以及在互联网上有效地路由IP数据包的对IP地址进行归类的方法

以 flannel 为例，深入分析阿里云 Kubernetes 集群网络的实现方法，一个是网络的搭建过程，另外一个是基于网络的通信。

分三种情况来理解：**集群配置，节点配置以及 Pod 配置**。与这三种情况对应的，其实是对集群网络 IP 段的三次划分：首先是集群 CIDR，接着为每个节点分配 podCIDR（即集群 CIDR 的子网段），最后在 podCIDR 里为每个 Pod 分配自己的 IP。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210406144151278.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)


### 5.1 初始阶段
集群的创建，基于云资源 VPC 和 ECS，在创建完 VPC 和 ECS 之后，我们基本上可以得到如下图的资源配置。我们得到一个 VPC，这个 VPC 的网段是 `192.168.0.0/16`，我们得到若干 ECS，他们从 VPC 网段里分配到 IP 地址。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210406144232368.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

### 5.2 集群阶段
在以上出初始资源的基础上，我们利用集群创建控制台得到集群 `CIDR`。这个值会以参数的形式传给集群节点 provision 脚本，并被脚本传给集群节点配置工具 `kubeadm`。kubeadm 最后把这个参数写入集群控制器静态 Pod 的 yaml 文件 `kube-controller-manager.yaml`。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210406144519502.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
集群控制器有了这个参数，在节点 kubelet 注册节点到集群的时候，集群控制器会为每个注册节点，划分一个子网出来，即为每个节点分配 `podCIDR`。如上图，Node B 的子网是 `172.16.8.1/25`，而 Node A 的子网是 `172.16.0.128/25`。这个配置会记录到集群 node 的 podCIDR 数据项里。

### 5.3 节点阶段
经过以上集群阶段，Kubernetes 有了集群 `CIDR`，以及为每个节点划分的 `podCIDR`。在此基础上，集群会下发 `flanneld` 到每个阶段上，进一步搭建节点上，可以给 Pod 使用的网络框架。这里主要有两个操作，

 - **第一个是集群通过 `Cloud Controller Manager` 给 VPC配置路由表项**。路由表项对每个节点有一条。每一条的意思是，如果 VPC 路由收到目的地址是某一个节点 podCIDR 的 IP地址，那么路由会把这个网络包转发到对应的 ECS 上。
 - **第二个是创建虚拟网桥 `cni0`，以及与 cni0 相关的路由**。这些配置的作用是，从阶段外部进来的网络包，如果目的 IP 是podCIDR，则会被节点转发到 cni0 虚拟局域网里。

注意：实际实现上，cni0 的创建，是在第一个使用 Pod 网络的 Pod 被调度到节点上的时候，由下一节中 `flannal cni` 创建的，但是从逻辑上来说，**cni0 属于节点网络**，不属于 Pod 网络，所以在此描述。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210406144834717.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
### 5.4 Pod 阶段
在前边的三个阶段，集群实际上已经为 Pod 之间搭建了网络通信的干道。这个时候，如果集群把一个 Pod 调度到节点上，kubelet 会通过 `flannel cni` 为这个 Pod 本身创建`网络命名空间和 veth 设备`，然后，把其中一个 `veth` 设备加入到 cni0 虚拟网桥里，并为 Pod 内的 veth 设备配置 ip 地址。这样 Pod 就和网络通信的干道连接在了一起。这里需要强调的是， `flanneld` 和这一节的 `flannel cni` 完全是两个组件。`flanneld` 是一个 `daemonset` 下发到每个节点的 pod，它的作用是搭建网络（干道），而 flannel cni 是节点创建的时候，通过 kubernetes-cni 这个 rpm 包安装的 cni 插件，其被 kubelet 调用，用来为具体的 pod 创建网络（分枝）。

理解这两者的区别，有助于我们理解 flanneld 和 flannel cni 相关的配置文件的用途。比如`/run/flannel/subnet.env`，是 `flanneld` 创建的，为 flannel cni 提供输入的一个环境变量文件；又比如`/etc/cni/net.d/10-flannel.conf`，也是 flanneld pod（准确的说，是 pod 里的脚本 install-cni）从 pod 里拷贝到节点目录，给 `flannel cni` 使用的子网配置文件。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210406145342696.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 6. 如何理解集群网络通信类型
以上完成 Pod 网络环境搭建。基于以上的网络环境，Pod 可以完成四种通信：本地通信，同节点 Pod 通信，跨节点 Pod 通信，以及 Pod 和 Pod 网络之外的实体通信。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210406145728960.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
### 6.1 pod内部通信
Kubernetes创建Pod时，首先会创建一个pause容器，为Pod指派一个唯一的IP地址。然后，以`pause`的网络命名空间为基础，创建同一个Pod内的其它容器（`–net=container:xxx`）。因此，同一个Pod内的所有容器就会共享同一个网络命名空间，在同一个Pod之间的容器可以直接使用`localhost`即loopback 设备进行通信。
### 6.2 不同Pod中容器之间通信
是 cni0 虚拟网桥内部的通信，这相当于一个二层局域网内部设备通信。

### 6.3 跨节点通信
发送端数据包，通过 cni0 网桥的网关，流转到节点上，然后经过节点 eth0 发送给 VPC 路由。这里不会经过任何封包操作。当 VPC 路由收到数据包时，它通过查询路由表，确认数据包目的地，并把数据包发送给对应的 ECS 节点。而进去节点之后，因为 flanneld 在节点上创建了真的 cni0 的路由，所以数据包会被发送到目的地的 cni0 局域网，再到目的地 Pod。

flannel的支持多种网络模式，常用用都是`vxlan`、`UDP`、`hostgw`、`ipip`以及`gce`和`阿里云`等。

vxlan和UDP的区别是vxlan是内核封包，而UDP是flanneld用户态程序封包，所以UDP的方式性能会稍差；

hostgw模式是一种主机网关模式，容器到另外一个主机上容器的网关设置成所在主机的网卡地址，这个和calico非常相似，只不过calico是通过BGP声明，而hostgw是通过中心的etcd分发，所以hostgw是直连模式，不需要通过overlay封包和拆包，性能比较高，但hostgw模式最大的缺点是必须是在一个二层网络中，毕竟下一跳的路由需要在邻居表中，否则无法通行。


## 7. 术 语

在对CNI插件们进行比较之前，我们可以先对网络中会见到的相关术语做一个整体的了解。不论是阅读本文，还是今后接触到其他和CNI有关的内容，了解一些常见术语总是非常有用的。

一些最常见的术语包括：

 - 第2层网络： OSI（Open Systems Interconnections，开放系统互连）网络模型的“数据链路”层。第2层网络会处理网络上两个相邻节点之间的帧传递。第2层网络的一个值得注意的示例是以太网，其中MAC表示为子层。
 - 第3层网络： OSI网络模型的“网络”层。第3层网络的主要关注点，是在第2层连接之上的主机之间路由数据包。IPv4、IPv6和ICMP是第3层网络协议的示例。
 - VXLAN：代表“虚拟可扩展LAN”。首先，VXLAN用于通过在UDP数据报中封装第2层以太网帧来帮助实现大型云部署。VXLAN虚拟化与VLAN类似，但提供更大的灵活性和功能（VLAN仅限于4096个网络ID）。VXLAN是一种封装和覆盖协议，可在现有网络上运行。
 - Overlay网络：Overlay网络是建立在现有网络之上的虚拟逻辑网络。Overlay网络通常用于在现有网络之上提供有用的抽象，并分离和保护不同的逻辑网络。
 - 封装：封装是指在附加层中封装网络数据包以提供其他上下文和信息的过程。在overlay网络中，封装被用于从虚拟网络转换到底层地址空间，从而能路由到不同的位置（数据包可以被解封装，并继续到其目的地）。
 - 网状网络：网状网络（Mesh network）是指每个节点连接到许多其他节点以协作路由、并实现更大连接的网络。网状网络允许通过多个路径进行路由，从而提供更可靠的网络。网状网格的缺点是每个附加节点都会增加大量开销。
 - BGP：代表“边界网关协议”，用于管理边缘路由器之间数据包的路由方式。BGP通过考虑可用路径，路由规则和特定网络策略，帮助弄清楚如何将数据包从一个网络发送到另一个网络。BGP有时被用作CNI插件中的路由机制，而不是封装的覆盖网络。


参考连接：
[https://www.infoq.cn/article/6mdfWWGHzAdihiq9lDST？utm_source=related_read&utm_medium=article](https://www.infoq.cn/article/6mdfWWGHzAdihiq9lDST?utm_source=related_read&utm_medium=article)
[https://blog.csdn.net/hguisu/article/details/92637760](https://blog.csdn.net/hguisu/article/details/92637760)








