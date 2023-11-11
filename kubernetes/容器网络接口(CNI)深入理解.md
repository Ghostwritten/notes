

-----

##  1. CNI 简介
容器网络的配置是一个复杂的过程，为了应对各式各样的需求，容器网络的解决方案也多种多样，例如有Flannel，Calico，Kube-OVN，Weave等。同时，容器平台/运行时也是多样的，例如有Kubernetes，OpenShift，rkt等。如果每种容器平台都要跟每种网络解决方案一一对接适配，这将是一项巨大且重复的工程。当然，聪明的程序员们肯定不会允许这样的事情发生。想要解决这个问题，我们需要一个抽象的接口层，将容器网络配置方案与容器平台方案解耦。

CNI（Container Network Interface）就是这样的一个接口层，它定义了一套接口标准，提供了规范文档以及一些标准实现。采用CNI规范来设置容器网络的容器平台不需要关注网络的设置的细节，只需要按CNI规范来调用CNI接口即可实现网络的设置。

CNI最初是由CoreOS为rkt容器引擎创建的，随着不断发展，已经成为事实标准。目前绝大部分的容器平台都采用CNI标准（rkt，Kubernetes，OpenShift等）。本篇内容基于CNI最新的发布版本v0.4.0。

值得注意的是，Docker并没有采用CNI标准，而是在CNI创建之初同步开发了CNM（Container Networking Model）标准。但由于技术和非技术原因，CNM模型并没有得到广泛的应用。

### 1.1 设计考量
CNI 设计的时候考虑了以下问题：

 - 容器运行时必须在调用任何插件之前为容器创建一个新的网络命名空间。
 - 然后，运行时必须确定这个容器应属于哪个网络，并为每个网络确定哪些插件必须被执行。
 - 网络配置采用 JSON 格式，可以很容易地存储在文件中。网络配置包括必填字段，如 name 和 type以及插件（类型）。网络配置允许字段在调用之间改变值。为此，有一个可选的字段 args，必须包含不同的信息。
 - 容器运行时必须按顺序为每个网络执行相应的插件，将容器添加到每个网络中。
 - 在完成容器生命周期后，运行时必须以相反的顺序执行插件（相对于执行添加容器的顺序）以将容器与网络断开连接。
 - 容器运行时不能为同一容器调用并行操作，但可以为不同的容器调用并行操作。
 - 容器运行时必须为容器订阅 ADD 和 DEL 操作，这样 ADD 后面总是跟着相应的 DEL。 DEL 可能跟着额外的DEL，但是，插件应该允许处理多个 DEL（即插件 DEL 应该是幂等的）。
 - 容器必须由 ContainerID 唯一标识。存储状态的插件应该使用（网络名称，容器 ID）的主键来完成。
 - 运行时不能调用同一个网络名称或容器 ID 执行两次 ADD（没有相应的 DEL）。换句话说，给定的容器 ID必须只能添加到特定的网络一次。

## 2. CNI是怎么工作的
CNI的接口并不是指HTTP，gRPC接口，CNI接口是指对可执行程序的调用（exec）。这些可执行程序称之为CNI插件，以Kubernetes为例，Kubernetes节点默认的CNI插件路径为`/opt/cni/bin`，在Kubernetes节点上查看该目录，可以看到可供使用的CNI插件：

```bash
$ ls /opt/cni/bin/  
bandwidth  bridge  dhcp  firewall  flannel  host-device  host-local  ipvlan  loopback  macvlan  portmap  ptp  sbr  static  tuning  vlan
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/0bac3764443a427fb8e115b6ad122fcf.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
CNI通过JSON格式的配置文件来描述网络配置，当需要设置容器网络时，由容器运行时负责执行CNI插件，并通过CNI插件的标准输入（stdin）来传递配置文件信息，通过标准输出（stdout）接收插件的执行结果。图中的 `libcni` 是CNI提供的一个go package，封装了一些符合CNI规范的标准操作，便于容器运行时和网络插件对接CNI标准。

举一个直观的例子，假如我们要调用bridge插件将容器接入到主机网桥，则调用的命令看起来长这样：

```bash
# CNI_COMMAND=ADD 顾名思义表示创建。
# XXX=XXX 其他参数定义见下文。
# < config.json 表示从标准输入传递配置文件
CNI_COMMAND=ADD XXX=XXX ./bridge < config.json
```


### 2.1 插件类型

容器运行时通过设置环境变量以及从标准输入传入的配置文件来向插件传递参数。

Main：接口创建

 - `bridge`：创建网桥，并添加主机和容器到该网桥
 - `ipvlan`：在容器中添加一个 ipvlan 接口
 - `loopback`：创建一个回环接口
 - `macvlan`：创建一个新的 MAC 地址，将所有的流量转发到容器
 - `ptp`：创建 veth 对
 - `vlan`：分配一个 vlan 设备

IPAM：IP 地址分配

 - `dhcp`：在主机上运行守护程序，代表容器发出 DHCP 请求
 - `host-local`：维护分配 IP 的本地数据库

Meta：其它插件

 - `flannel`：根据 flannel 的配置文件创建接口
 - `tuning`：调整现有接口的 sysctl 参数
 - `portmap`：一个基于 iptables 的 portmapping 插件。将端口从主机的地址空间映射到容器。

### 2.2 环境变量

 - `CNI_COMMAND`：定义期望的操作，可以是ADD，DEL，CHECK或VERSION。
 - `CNI_CONTAINERID`：容器ID，由容器运行时管理的容器唯一标识符。
 - `CNI_NETNS`：容器网络命名空间的路径。（形如 /run/netns/[nsname]）。
 - `CNI_IFNAME`：需要被创建的网络接口名称，例如eth0。
 - `CNI_ARGS`：运行时调用时传入的额外参数，格式为分号分隔的key-value对，例如FOO=BAR;ABC=123
 - `CNI_PATH`：CNI插件可执行文件的路径，例如`/opt/cni/bin`。

### 2.3 配置文件

文件示例：

```bash
{
  "cniVersion": "0.4.0", // 表示希望插件遵循的CNI标准的版本。
  "name": "dbnet",  // 表示网络名称。这个名称并非指网络接口名称，是便于CNI管理的一个表示。应当在当前主机（或其他管理域）上全局唯一。
  "type": "bridge", // 插件类型
  "bridge": "cni0", // Bridge插件的参数，指定网桥名称。
  "ipam": { // IP Allocation Management，管理IP地址分配。
    "type": "host-local", // IPAM插件的类型。
    // IPAM定义的参数
    "subnet": "10.1.0.0/16",
    "gateway": "10.1.0.1"
  }
}
```
公共定义部分：

配置文件分为公共部分和插件定义部分。公共部分在CNI项目中使用结构体NetworkConfig定义：

```go
type NetworkConfig struct {
   Network *types.NetConf
   Bytes   []byte
}
...
// NetConf describes a network.
type NetConf struct {
   CNIVersion string `json:"cniVersion,omitempty"`

   Name         string          `json:"name,omitempty"`
   Type         string          `json:"type,omitempty"`
   Capabilities map[string]bool `json:"capabilities,omitempty"`
   IPAM         IPAM            `json:"ipam,omitempty"`
   DNS          DNS             `json:"dns"`

   RawPrevResult map[string]interface{} `json:"prevResult,omitempty"`
   PrevResult    Result                 `json:"-"`
}
```

 - `cniVersion`：表示希望插件遵循的CNI标准的版本。
 - `name`：表示网络名称。这个名称并非指网络接口名称，是便于CNI管理的一个表示。应当在当前主机（或其他管理域）上全局唯一。
 - `type`：表示插件的名称，也就是插件对应的可执行文件的名称。
 - `Bridge`：该参数属于bridge插件的参数，指定主机网桥的名称。
 - `IPAM`：表示IP地址分配插件的配置，`ipam.type`则表示IPAM的插件类型。

更详细的信息，可以参考官方文档：
[https://github.com/containernetworking/cni/blob/spec-v0.4.0/SPEC.md#network-configuration](https://github.com/containernetworking/cni/blob/spec-v0.4.0/SPEC.md#network-configuration)

插件定义部分：

上文提到，配置文件最终是传递给具体的CNI插件的，因此插件定义部分才是配置文件的“完全体”。公共部分定义只是为了方便各插件将其嵌入到自身的配置文件定义结构体中，举`Bridge`插件为例：

```go
type NetConf struct {
 types.NetConf // <-- 嵌入公共部分
        // 底下的都是插件定义部分
 BrName       string `json:"bridge"`
 IsGW         bool   `json:"isGateway"`
 IsDefaultGW  bool   `json:"isDefaultGateway"`
 ForceAddress bool   `json:"forceAddress"`
 IPMasq       bool   `json:"ipMasq"`
 MTU          int    `json:"mtu"`
 HairpinMode  bool   `json:"hairpinMode"`
 PromiscMode  bool   `json:"promiscMode"`
 Vlan         int    `json:"vlan"`

 Args struct {
  Cni BridgeArgs `json:"cni,omitempty"`
 } `json:"args,omitempty"`
 RuntimeConfig struct {
  Mac string `json:"mac,omitempty"`
 } `json:"runtimeConfig,omitempty"`

 mac string
}
```
各插件的配置文件文档可参考官方文档：[https://www.cni.dev/plugins/current/](https://www.cni.dev/plugins/current/)

### 2.4 插件操作类型

CNI插件的操作类型只有四种：`ADD`，`DEL`，`CHECK`和`VERSION`。插件调用者通过环境变量`CNI_COMMAND`来指定需要执行的操作。
接口定义：

```go
type CNI interface {
    AddNetworkList (net *NetworkConfigList, rt *RuntimeConf) (types.Result, error)
    DelNetworkList (net *NetworkConfigList, rt *RuntimeConf) error
    AddNetwork (net *NetworkConfig, rt *RuntimeConf) (types.Result, error)
    DelNetwork (net *NetworkConfig, rt *RuntimeConf) error
}
```
该接口只有四个方法，添加网络、删除网络、添加网络列表、删除网络列表。
#### 2.4.1 ADD

ADD操作负责将容器添加到网络，或对现有的网络设置做更改。具体地说，ADD操作要么：

 - 为容器所在的网络命名空间创建一个网络接口，或者
 - 修改容器所在网络命名空间中的指定网络接口

例如通过ADD将容器网络接口接入到主机的网桥中。

其中网络接口名称由`CNI_IFNAME`指定，网络命名空间由`CNI_NETNS`指定。

#### 2.4.2  DEL

DEL操作负责从网络中删除容器，或取消对应的修改，可以理解为是ADD的逆操作。具体地说，DEL操作要么：

 - 为容器所在的网络命名空间删除一个网络接口，或者
 - 撤销ADD操作的修改

例如通过DEL将容器网络接口从主机网桥中删除。

其中网络接口名称由CNI_IFNAME指定，网络命名空间由`CNI_NETNS`指定。

#### 2.4.3 CHECK

CHECK操作是v0.4.0加入的类型，用于检查网络设置是否符合预期。容器运行时可以通过CHECK来检查网络设置是否出现错误，当CHECK返回错误时（返回了一个非0状态码），容器运行时可以选择Kill掉容器，通过重新启动来重新获得一个正确的网络配置。

#### 2.4.4 VERSION

VERSION操作用于查看插件支持的版本信息。

```bash
$ CNI_COMMAND=VERSION /opt/cni/bin/bridge
{"cniVersion":"0.4.0","supportedVersions":["0.1.0","0.2.0","0.3.0","0.3.1","0.4.0"]}
```

###  2.5 链式调用
单个CNI插件的职责是单一的，比如`Bridge`插件负责网桥的相关配置， `Firewall`插件负责防火墙相关配置， `Portmap`插件负责端口映射相关配置。因此，当网络设置比较复杂时，通常需要调用多个插件来完成。CNI支持插件的链式调用，可以将多个插件组合起来，按顺序调用。例如先调用Bridge插件设置容器IP，将容器网卡与主机网桥连通，再调用Portmap插件做容器端口映射。容器运行时可以通过在配置文件设置`Plugins`数组达到链式调用的目的：

```go
{
  "cniVersion": "0.4.0",
  "name": "dbnet",
  "plugins": [
    {
      "type": "bridge",
      // type (plugin) specific
      "bridge": "cni0"
      },
      "ipam": {
        "type": "host-local",
        // ipam specific
        "subnet": "10.1.0.0/16",
        "gateway": "10.1.0.1"
      }
    },
    {
      "type": "tuning",
      "sysctl": {
        "net.core.somaxconn": "500"
      }
    }
  ]
}
```
细心的读者会发现，Plugins这个字段并没有出现在上文描述的配置文件结构体中。的确，CNI使用了另一个结构体——`NetworkConfigList`来保存链式调用的配置：

```bash
type NetworkConfigList struct {
   Name         string
   CNIVersion   string
   DisableCheck bool
   Plugins      []*NetworkConfig 
   Bytes        []byte
}
```
但CNI插件是不认识这个配置类型的。实际上，在调用CNI插件时，需要将`NetworkConfigList`转换成对应插件的配置文件格式，再通过标准输入（stdin）传递给CNI插件。例如在上面的示例中，实际上会先使用下面的配置文件调用Bridge插件：

```bash
{
  "cniVersion": "0.4.0",
  "name": "dbnet",
  "type": "bridge",
  "bridge": "cni0",
  "ipam": {
    "type": "host-local",
    "subnet": "10.1.0.0/16",
    "gateway": "10.1.0.1"
  }
}
```
再使用下面的配置文件调用`tuning`插件：

```bash
{
  "cniVersion": "0.4.0",
  "name": "dbnet",
  "type": "tuning",
  "sysctl": {
    "net.core.somaxconn": "500"
  },
  "prevResult": { // 调用Bridge插件的返回结果
     ...
  }
}
```
需要注意的是，当插件进行链式调用的时候，不仅需要对`NetworkConfigList`做格式转换，而且需要将前一次插件的返回结果添加到配置文件中（通过`prevResult`字段），不得不说是一项繁琐而重复的工作。不过幸好`libcni`已经为我们封装好了，容器运行时不需要关心如何转换配置文件，如何填入上一次插件的返回结果，只需要调用libcni的相关方法即可。

### 2.6 ip分配

作为容器网络管理的一部分，CNI 插件需要为接口分配（并维护）IP 地址，并安装与该接口相关的所有必要路由。这给了 CNI 插件很大的灵活性，但也给它带来了很大的负担。众多的 CNI 插件需要编写相同的代码来支持用户需要的多种 IP 管理方案（例如 dhcp、host-local）。

为了减轻负担，使 IP 管理策略与 CNI 插件类型解耦，我们定义了 IP 地址管理插件（IPAM 插件）。CNI 插件的职责是在执行时恰当地调用 IPAM 插件。 IPAM 插件必须确定接口 IP/subnet，网关和路由，并将此信息返回到 “主” 插件来应用配置。 IPAM 插件可以通过协议（例如 dhcp）、存储在本地文件系统上的数据、网络配置文件的 “ipam” 部分或上述的组合来获得信息。

`IPAM` 插件像 CNI 插件一样，调用 IPAM 插件的可执行文件。可执行文件位于预定义的路径列表中，通过 `CNI_PATH` 指示给 CNI 插件。 IPAM 插件必须接收所有传入 CNI 插件的相同环境变量。就像 CNI 插件一样，IPAM 插件通过 stdin 接收网络配置。

##  3. 示例
接下来将演示如何使用CNI插件来为Docker容器设置网络。

下载CNI插件
为方便起见，我们直接下载可执行文件：

```bash
$ wget https://github.com/containernetworking/plugins/releases/download/v0.9.1/cni-plugins-linux-amd64-v0.9.1.tgz
$ mkdir -p  ~/cni/bin
$ tar zxvf cni-plugins-linux-amd64-v0.9.1.tgz -C ./cni/bin
$ chmod  +x ~/cni/bin/*
$ ls ~/cni/bin/
bandwidth  bridge  dhcp  firewall  flannel  host-device  host-local  ipvlan  loopback  macvlan  portmap  ptp  sbr  static  tuning  vlan  vrfz
```
如果你是在Kubernetes节点上实验，通常节点上已经有CNI插件了，不需要再下载，但要注意将后续的CNI_PATH修改成`/opt/cni/bin`。

###  3.1 示例1——调用单个插件bridge
在示例1中，我们会直接调用CNI插件，为容器设置eth0接口，为其分配IP地址，并接入主机网桥`mynet0`。

跟Docker默认使用的使用网络模式一样，只不过我们将docker0换成了`mynet0`。

#### 3.1.1 启动容器

虽然Docker不使用CNI规范，但可以通过指定`--net=none`的方式让Docker不设置容器网络。以Nginx镜像为例：

```bash
contid=$(docker run -d --net=none --name nginx nginx) # 容器ID
pid=$(docker inspect -f '{{ .State.Pid }}' $contid) # 容器进程ID
netnspath=/proc/$pid/ns/net # 命名空间路径
```
启动容器的同时，我们需要记录一下容器ID，命名空间路径，方便后续传递给CNI插件。容器启动后，可以看到除了lo网卡，容器没有其他的网络设置：

```bash
$ nsenter -t $pid -n ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
```

`nsenter`是`namespace enter`的简写，顾名思义，这是一个在某命名空间下执行命令的工具。`-t`表示进程ID，`-n`表示进入对应进程的网络命名空间。

#### 3.1.2 添加容器网络接口并连接主机网桥

接下来我们使用`Bridge`插件为容器创建网络接口，并连接到主机网桥。创建`bridge.json`配置文件，内容如下：

```bash
{
    "cniVersion": "0.4.0",
    "name": "mynet",
    "type": "bridge",
    "bridge": "mynet0",
    "isDefaultGateway": true,
    "forceAddress": false,
    "ipMasq": true,
    "hairpinMode": true,
    "ipam": {
        "type": "host-local",
        "subnet": "10.10.0.0/16"
    }
}
```
调用`Bridge`插件`ADD`操作：

```bash
CNI_COMMAND=ADD CNI_CONTAINERID=$contid CNI_NETNS=$netnspath CNI_IFNAME=eth0 CNI_PATH=~/cni/bin ~/cni/bin/bridge < bridge.json
```
调用成功的话，会输出类似的返回值：

```bash
{
    "cniVersion": "0.4.0",
    "interfaces": [
        ....
    ],
    "ips": [
        {
            "version": "4",
            "interface": 2,
            "address": "10.10.0.2/16", //给容器分配的IP地址
            "gateway": "10.10.0.1" 
        }
    ],
    "routes": [
        .....
    ],
    "dns": {}
}
```
再次查看容器网络设置：

```bash
$ nsenter -t $pid -n ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
5: eth0@if40: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether c2:8f:ea:1b:7f:85 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.10.0.2/16 brd 10.10.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```
可以看到容器中已经新增了eth0网络接口，并在IPAM插件设定的子网下为其分配了IP地址。`host-local`类型的IPAM插件会将已分配的IP信息保存到文件，避免IP冲突，默认的保存路径为`/var/lib/cni/network/$NETWORK_NAME`：

```bash
$ ls /var/lib/cni/networks/mynet/
10.10.0.2  last_reserved_ip.0  lock
```

#### 3.1.3 从主机访问验证
由于`mynet0`是我们添加的网桥，还未设置路由，因此验证前我们需要先为容器所在的网段添加路由：

```bash
ip route add 10.10.0.0/16 dev mynet0 src 10.10.0.1 # 添加路由
curl -I 10.10.0.2 # IP换成实际分配给容器的IP地址
HTTP/1.1 200 OK
....
```
#### 3.1.4  删除容器网络接口
删除的调用入参跟添加的入参是一样的，除了`CNI_COMMAND`要替换成DEL：

```bash
CNI_COMMAND=DEL CNI_CONTAINERID=$contid CNI_NETNS=$netnspath CNI_IFNAME=eth0 CNI_PATH=~/cni/bin ~/cni/bin/bridge < bridge.json
```

注意，上述的删除命令并未清理主机的mynet0网桥。如果你希望删除主机网桥，可以执行`ip link delete mynet0 type bridge`命令删除。


###  3.2 示例2---调用单个插件ptp

#### 3.2.1 编辑插件配置文件
使用`ptp`插件为ptp 插件用于创建 veth 设备对。创建配置文件`10-myptp.conflist`：

```bash
$ cat /etc/cni/net.d/10-myptp.conflist
{
    "cniVersion": "0.4.0",
    "name": "myptp",
    "plugins": [
        {
            "type": "ptp",
            "ipMasq": true,
            "ipam": {
                "type": "host-local",
                "subnet": "172.16.29.0/24",
                "routes": [
                    {
                        "dst": "0.0.0.0/0"
                    }
                ]
             }
        }
    ]
}
```
#### 3.2.2 创建一个网络命名空间：

```bash
$ ip netns add testing
```

#### 3.2.3 在命名空间内执行插件

```bash
$ CNI_PATH=./cni/bin cnitool add myptp /var/run/netns/testing
{
    "cniVersion": "0.4.0",
    "interfaces": [
        {
            "name": "vetheccd7f4c",
            "mac": "32:e6:29:a7:16:9b"
        },
        {
            "name": "eth0",
            "mac": "86:e2:50:bc:fc:a8",
            "sandbox": "/var/run/netns/testing"
        }
    ],
    "ips": [
        {
            "version": "4",
            "interface": 1,
            "address": "172.16.29.2/24",
            "gateway": "172.16.29.1"
        }
    ],
    "routes": [
        {
            "dst": "0.0.0.0/0"
        }
    ],
    "dns": {}
}
```

#### 3.2.4 检查效果

```bash
$ CNI_PATH=./cni/bin cnitool check myptp /var/run/netns/testing
```
#### 3.2.5 测试

```bash
root@master:~# ip -n testing addr

3: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 86:e2:50:bc:fc:a8 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.29.2/24 brd 172.16.29.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::84e2:50ff:febc:fca8/64 scope link 
       valid_lft forever preferred_lft forever
```
这条命令会列出网络命名空间 testing 内的网卡，我这里看到了一个名为 `eth0@if7` 的网卡

然后查看本机原始命名空间内的网卡：

```bash
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:13:3e:a6 brd ff:ff:ff:ff:ff:ff
    inet 192.168.211.40/24 brd 192.168.211.255 scope global ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe13:3ea6/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:be:95:d5:1d brd ff:ff:ff:ff:ff:ff
4: veth773b656c@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master mynet0 state UP group default 
    link/ether c2:d7:2f:b0:6c:21 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::c0d7:2fff:feb0:6c21/64 scope link 
       valid_lft forever preferred_lft forever
6: mynet0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 46:9b:e9:95:3e:73 brd ff:ff:ff:ff:ff:ff
    inet 10.10.0.1/16 brd 10.10.255.255 scope global mynet0
       valid_lft forever preferred_lft forever
    inet6 fe80::449b:e9ff:fe95:3e73/64 scope link 
       valid_lft forever preferred_lft forever
7: vetheccd7f4c@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 32:e6:29:a7:16:9b brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet 172.16.29.1/32 scope global vetheccd7f4c
       valid_lft forever preferred_lft forever
    inet6 fe80::30e6:29ff:fea7:169b/64 scope link 
       valid_lft forever preferred_lft forever
```
可以看到两个命名空间内的两个网卡遥相呼应。

测试 `testing` 命名空间内的网卡是否可以工作：

```bash
$ ip netns exec testing ping -c 4 www.baidu.com
PING www.a.shifen.com (14.215.177.38) 56(84) bytes of data.
64 bytes from 14.215.177.38: icmp_seq=1 ttl=127 time=89.6 ms
64 bytes from 14.215.177.38: icmp_seq=2 ttl=127 time=77.1 ms
64 bytes from 14.215.177.38: icmp_seq=3 ttl=127 time=96.2 ms
64 bytes from 14.215.177.38: icmp_seq=4 ttl=127 time=93.7 ms
```
实测是可以工作的，ping 的 -c 参数指定了发包次数。

#### 3.2.6 清理

```bash
CNI_PATH=./cni/bin cnitool del myptp /var/run/netns/testing
ip netns del testing
```

---

### 3.3  示例2——链式调用
在示例2中，我们将在示例1的基础上，使用`Portmap`插件为容器添加端口映射。

####  3.3.1使用cnitool工具
前面的介绍中，我们知道在链式调用过程中，调用方需要转换配置文件，并需要将上一次插件的返回结果插入到本次插件的配置文件中。这是一项繁琐的工作，而`libcni`已经将这些过程封装好了，在示例2中，我们将使用基于 `libcni`的命令行工具`cnitool`来简化这些操作。

示例2将复用示例1中的容器，因此在开始示例2时，请确保已删除示例1中的网络接口。

通过源码编译或`go install`来安装`cnitool`：

```bash
$ go get github.com/containernetworking/cni
$ go install github.com/containernetworking/cni/cnitool
```
拷贝`$GOBIN` 中的 cnitool 拷贝到 `/usr/bin`
#### 3.3.2  配置文件
`libcni`会读取`.conflist`后缀的配置文件，我们在当前目录创建`portmap.conflist`：

```go
{
  "cniVersion": "0.4.0",
  "name": "portmap",
  "plugins": [
    {
      "type": "bridge",
      "bridge": "mynet0",
      "isDefaultGateway": true, 
      "forceAddress": false, 
      "ipMasq": true, 
      "hairpinMode": true,
      "ipam": {
        "type": "host-local",
        "subnet": "10.10.0.0/16",
        "gateway": "10.10.0.1"
      }
    },
    {
      "type": "portmap",
      "runtimeConfig": {
        "portMappings": [
          {"hostPort": 8080, "containerPort": 80, "protocol": "tcp"}
        ]
      }
    }
  ]
}
```
从上述的配置文件定义了两个CNI插件，Bridge和Portmap。根据上述的配置文件，cnitool会先为容器添加网络接口并连接到主机mynet0网桥上（就跟示例1一样），然后再调用Portmap插件，将容器的80端口映射到主机的8080端口，就跟`docker run -p 8080:80 xxx`一样。

#### 3.4.3 设置容器网络
使用`cnitool`我们还需要设置两个环境变量：

 - `NETCONFPATH`：指定配置文件（`*.conflist`）的所在路径，默认路径为`/etc/cni/net.d`
 - `CNI_PATH`：指定CNI插件的存放路径。

使用`cnitool add`命令为容器设置网络：

```bash
CNI_PATH=~/cni/bin NETCONFPATH=.  cnitool add portmap $netnspath
```
设置成功后，访问宿主机8080端口即可访问到容器的Nginx服务。

#### 3.4.4  删除网络配置
使用`cnitool del`命令删除容器网络：

```bash
CNI_PATH=~/cni/bin NETCONFPATH=.  cnitool del portmap $netnspath
```
注意，上述的删除命令并未清理主机的mynet0网桥。如果你希望删除主机网桥，可以执行`ip link delete mynet0 type bridge`命令删除。




##  4. 总结
至此，CNI的工作原理我们已基本清楚。CNI的工作原理大致可以归纳为：

 - 通过JSON配置文件定义网络配置；
 - 通过调用可执行程序（CNI插件）来对容器网络执行配置；
 - 通过链式调用的方式来支持多插件的组合使用。

CNI不仅定义了接口规范，同时也提供了一些内置的标准实现，以及libcni这样的“胶水层”，大大降低了容器运行时与网络插件的接入门槛。

参考链接：

 - [https://github.com/containernetworking/cni/blob/spec-v0.4.0/SPEC.md](https://github.com/containernetworking/cni/blob/spec-v0.4.0/SPEC.md)
 - [https://github.com/containernetworking/cni/blob/master/SPEC.md](https://github.com/containernetworking/cni/blob/master/SPEC.md)
 - [https://www.cni.dev/plugins/current/](https://www.cni.dev/plugins/current/)
 - [https://github.com/containernetworking/cni/tree/master/cnitool](https://github.com/containernetworking/cni/tree/master/cnitool)
 - [https://kubernetes.io/blog/2016/01/why-kubernetes-doesnt-use-libnetwork/](https://kubernetes.io/blog/2016/01/why-kubernetes-doesnt-use-libnetwork/)
 - [https://www.youtube.com/watch?v=YWXucnygGmY](https://www.youtube.com/watch?v=YWXucnygGmY)
 - [https://www.youtube.com/watch?v=0tbnXX7jXdg](https://www.youtube.com/watch?v=0tbnXX7jXdg)
 - [cnitool 源码分析](https://xujiyou.work/%E4%BA%91%E5%8E%9F%E7%94%9F/Kubernetes/Kubernetes%E6%89%A9%E5%B1%95%E6%9C%BA%E5%88%B6/CNI/cnitool%E4%BD%BF%E7%94%A8%E6%96%B9%E5%BC%8F.html)
 - [水立方](https://juejin.cn/post/6986495816949039141)


