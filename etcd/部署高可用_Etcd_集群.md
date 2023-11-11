

## 1. 简介
在「[etcd使用入门](https://ghostwritten.blog.csdn.net/article/details/115481040)」一文中对etcd的基本知识点和安装做了一个简要的介绍，这次我们来说说如何部署一个etcd集群。

etcd构建自身高可用集群主要有三种形式:

 - 静态发现: 预先已知etcd集群中有哪些节点，在启动时通过--initial-cluster参数直接指定好etcd的各个节点地址。
 - etcd动态发现:通过已有的etcd集群作为数据交互点，然后在扩展新的集群时实现通过已有集群进行服务发现的机制。比如官方提供的：`discovery.etcd.io`
 - DNS动态发现: 通过DNS查询方式获取其他节点地址信息。

本文将介绍如何通过静态发现这种方式来部署一个etcd集群，这种方式也是最简单的。

## 2. 环境准备

通常按照需求将集群节点部署为3，5，7，9个节点。这里能选择偶数个节点吗？最好不要这样。原因有二：

 - 偶数个节点集群不可用风险更高，表现在选主过程中，有较大概率或等额选票，从而触发下一轮选举。
 - 偶数个节点集群在某些网络分割的场景下无法正常工作。当网络分割发生后，将集群节点对半分割开。此时集群将无法工作。按照RAFT协议，此时集群写操作无法使得大多数节点同意，从而导致写失败，集群无法正常工作。

这里将部署一个3节点的集群， 以下为3台主机信息，系统环境为`Ubuntu 16.04`。
| 节点名称  | 地址            |
|-------|---------------|
| etcd1 | 192.168.2.210 |
| etcd2 | 192.168.2.211 |
| etcd3 | 192.168.2.212 |

## 3. 安装etcd

在「[etcd使用入门](https://ghostwritten.blog.csdn.net/article/details/115481040)」一文中对如何安装已经做了介绍

## 4. 配置etcd集群

修改etcd配置文件,我这里的环境是在`/opt/etcd/config/etcd.conf`，请根据实际情况修改。

etcd1配置示例

```bash
# 编辑配置文件
$ vim /opt/etcd/config/etcd.conf
ETCD_NAME=etcd1
ETCD_DATA_DIR="/var/lib/etcd/etcd1"
ETCD_LISTEN_PEER_URLS="http://192.168.2.210:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.2.210:2379,http://192.168.2.210:4001"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.2.210:2380"
ETCD_INITIAL_CLUSTER="etcd1=http://192.168.2.210:2380,etcd2=http://192.168.2.211:2380,etcd3=http://192.168.2.212:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="hilinux-etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.2.210:2379,http://192.168.2.210:4001"
```
etcd2配置示例

```bash
# 编辑配置文件
$ vim /opt/etcd/config/etcd.conf

ETCD_NAME=etcd2
ETCD_DATA_DIR="/var/lib/etcd/etcd2"
ETCD_LISTEN_PEER_URLS="http://192.168.2.211:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.2.211:2379,http://192.168.2.211:4001"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.2.211:2380"
ETCD_INITIAL_CLUSTER="etcd1=http://192.168.2.210:2380,etcd2=http://192.168.2.211:2380,etcd3=http://192.168.2.212:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="hilinux-etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.2.211:2379,http://192.168.2.211:4001"
```

etcd3配置示例

```bash
# 编辑配置文件
$ vim /opt/etcd/config/etcd.conf

ETCD_NAME=etcd3
ETCD_DATA_DIR="/var/lib/etcd/etcd3"
ETCD_LISTEN_PEER_URLS="http://192.168.2.212:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.2.212:2379,http://192.168.2.212:4001"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.2.212:2380"
ETCD_INITIAL_CLUSTER="etcd1=http://192.168.2.210:2380,etcd2=http://192.168.2.211:2380,etcd3=http://192.168.2.212:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="hilinux-etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.2.212:2379,http://192.168.2.212:4001"
```

针对上面几个配置参数做下简单的解释：

```bash
ETCD_NAME ：ETCD的节点名
ETCD_DATA_DIR：ETCD的数据存储目录
ETCD_SNAPSHOT_COUNTER：多少次的事务提交将触发一次快照
ETCD_HEARTBEAT_INTERVAL：ETCD节点之间心跳传输的间隔，单位毫秒
ETCD_ELECTION_TIMEOUT：该节点参与选举的最大超时时间，单位毫秒
ETCD_LISTEN_PEER_URLS：该节点与其他节点通信时所监听的地址列表，多个地址使用逗号隔开，其格式可以划分为scheme://IP:PORT，这里的scheme可以是http、https
ETCD_LISTEN_CLIENT_URLS：该节点与客户端通信时监听的地址列表
ETCD_INITIAL_ADVERTISE_PEER_URLS：该成员节点在整个集群中的通信地址列表，这个地址用来传输集群数据的地址。因此这个地址必须是可以连接集群中所有的成员的。
ETCD_INITIAL_CLUSTER：配置集群内部所有成员地址，其格式为：ETCD_NAME=ETCD_INITIAL_ADVERTISE_PEER_URLS，如果有多个使用逗号隔开
ETCD_ADVERTISE_CLIENT_URLS：广播给集群中其他成员自己的客户端地址列表
ETCD_INITIAL_CLUSTER_STATE：初始化集群状态，new表示新建
ETCD_INITIAL_CLUSTER_TOKEN:初始化集群token
```

> 注意：所有ETCD_MY_FLAG的配置参数也可以通过命令行参数进行设置，但是命令行指定的参数优先级更高，同时存在时会覆盖环境变量对应的值。

下面给出常用配置的参数和它们的解释：


```bash
--name：方便理解的节点名称，默认为default，在集群中应该保持唯一，可以使用 hostname
--data-dir：服务运行数据保存的路径，默认为 ${name}.etcd
--snapshot-count：指定有多少事务（transaction）被提交时，触发截取快照保存到磁盘
--heartbeat-interval：leader 多久发送一次心跳到 followers。默认值是 100ms
--eletion-timeout：重新投票的超时时间，如果 follow 在该时间间隔没有收到心跳包，会触发重新投票，默认为 1000 ms
--listen-peer-urls：和同伴通信的地址，比如 http://ip:2380，如果有多个，使用逗号分隔。需要所有节点都能够访问，所以不要使用 localhost！
--listen-client-urls：对外提供服务的地址：比如 http://ip:2379,http://127.0.0.1:2379，客户端会连接到这里和 etcd 交互
--advertise-client-urls：对外公告的该节点客户端监听地址，这个值会告诉集群中其他节点
--initial-advertise-peer-urls：该节点同伴监听地址，这个值会告诉集群中其他节点
--initial-cluster：集群中所有节点的信息，格式为 node1=http://ip1:2380,node2=http://ip2:2380,…。注意：这里的 node1 是节点的 --name 指定的名字；后面的 ip1:2380 是 --initial-advertise-peer-urls 指定的值
--initial-cluster-state：新建集群的时候，这个值为new；假如已经存在的集群，这个值为 existing
--initial-cluster-token：创建集群的token，这个值每个集群保持唯一。这样的话，如果你要重新创建集群，即使配置和之前一样，也会再次生成新的集群和节点 uuid；否则会导致多个集群之间的冲突，造成未知的错误

所有以--init开头的配置都是在bootstrap集群的时候才会用到，后续节点的重启会被忽略。
```


## 5. 测试etcd集群

按上面配置好各集群节点后，分别在各节点启动etcd。


```bash
$ systemctl start etcd
```

启动完成后，在任意节点执行etcdctl member list可列所有集群节点信息，如下所示：


```bash
$ etcdctl --endpoints "http://192.168.2.210:2379" member list
a3ba19408fd4c829: name=etcd3 peerURLs=http://192.168.2.212:2380 clientURLs=http://192.168.2.212:2379,http://192.168.2.212:4001 isLeader=true
a8589aa8629b731b: name=etcd1 peerURLs=http://192.168.2.210:2380 clientURLs=http://192.168.2.210:2379,http://192.168.2.210:4001 isLeader=false
e4a3e95f72ced4a7: name=etcd2 peerURLs=http://192.168.2.211:2380 clientURLs=http://192.168.2.211:2379,http://192.168.2.211:4001 isLeader=false
```

这里显示指定的集群地址，是因为在上面的配置中绑定了IP。如不指定会报如下错误：


```bash
$ etcdctl member list
Error:  client: etcd cluster is unavailable or misconfigured; error #0: dial tcp 127.0.0.1:4001: getsockopt: connection refused
; error #1: dial tcp 127.0.0.1:2379: getsockopt: connection refused

error #0: dial tcp 127.0.0.1:4001: getsockopt: connection refused
error #1: dial tcp 127.0.0.1:2379: getsockopt: connection refused
```

## 6. etcd集群基本管理

### 6.1 查看集群健康状态

```bash
$ etcdctl --endpoints "http://192.168.2.210:2379"  cluster-health
member a3ba19408fd4c829 is healthy: got healthy result from http://192.168.2.212:2379
member a8589aa8629b731b is healthy: got healthy result from http://192.168.2.210:2379
member e4a3e95f72ced4a7 is healthy: got healthy result from http://192.168.2.211:2379
cluster is healthy
```

### 6.2 查看集群成员
在任一节点上执行，可以看到集群的节点情况，并能看出哪个是leader节点。


```bash
$ etcdctl --endpoints "http://192.168.2.210:2379" member list
a3ba19408fd4c829: name=etcd3 peerURLs=http://192.168.2.212:2380 clientURLs=http://192.168.2.212:2379,http://192.168.2.212:4001 isLeader=true
a8589aa8629b731b: name=etcd1 peerURLs=http://192.168.2.210:2380 clientURLs=http://192.168.2.210:2379,http://192.168.2.210:4001 isLeader=false
e4a3e95f72ced4a7: name=etcd2 peerURLs=http://192.168.2.211:2380 clientURLs=http://192.168.2.211:2379,http://192.168.2.211:4001 isLeader=false
```

### 6.3 更新一个节点

```bash
# 如果你想更新一个节点的IP(peerURLS)，首先你需要知道那个节点的ID
$ etcdctl --endpoints "http://192.168.2.210:2379" member list
a3ba19408fd4c829: name=etcd3 peerURLs=http://192.168.2.212:2380 clientURLs=http://192.168.2.212:2379,http://192.168.2.212:4001 isLeader=false
a8589aa8629b731b: name=etcd1 peerURLs=http://192.168.2.210:2380 clientURLs=http://192.168.2.210:2379,http://192.168.2.210:4001 isLeader=false
e4a3e95f72ced4a7: name=etcd2 peerURLs=http://192.168.2.211:2380 clientURLs=http://192.168.2.211:2379,http://192.168.2.211:4001 isLeader=true


# 更新一个节点
$ etcdctl --endpoints "http://192.168.2.210:2379" member update a8589aa8629b731b http://192.168.2.210:2380
Updated member with ID a8589aa8629b731b in cluster
```
### 6.4 删除一个节点

```bash
$ etcdctl --endpoints "http://192.168.2.210:2379" member list
a3ba19408fd4c829: name=etcd3 peerURLs=http://192.168.2.212:2380 clientURLs=http://192.168.2.212:2379,http://192.168.2.212:4001 isLeader=false
a8589aa8629b731b: name=etcd1 peerURLs=http://192.168.2.210:2380 clientURLs=http://192.168.2.210:2379,http://192.168.2.210:4001 isLeader=false
e4a3e95f72ced4a7: name=etcd2 peerURLs=http://192.168.2.211:2380 clientURLs=http://192.168.2.211:2379,http://192.168.2.211:4001 isLeader=true

$ etcdctl --endpoints "http://192.168.2.210:2379" member remove a8589aa8629b731b
Removed member a8589aa8629b731b from cluster

$ etcdctl --endpoints "http://192.168.2.211:2379" member list
a3ba19408fd4c829: name=etcd3 peerURLs=http://192.168.2.212:2380 clientURLs=http://192.168.2.212:2379,http://192.168.2.212:4001 isLeader=false
e4a3e95f72ced4a7: name=etcd2 peerURLs=http://192.168.2.211:2380 clientURLs=http://192.168.2.211:2379,http://192.168.2.211:4001 isLeader=true
```

### 6.5 增加一个新节点
注意:步骤很重要，不然会报集群ID不匹配。

a. 将目标节点添加到集群


```bash
$ etcdctl --endpoints "http://192.168.2.211:2379" member add etcd1 http://192.168.2.210:2380

Added member named etcd1 with ID baab0aae8b58c802 to cluster

ETCD_NAME="etcd1"
ETCD_INITIAL_CLUSTER="etcd3=http://192.168.2.212:2380,etcd1=http://192.168.2.210:2380,etcd2=http://192.168.2.211:2380"
ETCD_INITIAL_CLUSTER_STATE="existing"
```

b. 查看新增成员列表

etcd1状态现在为unstarted


```bash
$ etcdctl --endpoints "http://192.168.2.211:2379" member list
a3ba19408fd4c829: name=etcd3 peerURLs=http://192.168.2.212:2380 clientURLs=http://192.168.2.212:2379,http://192.168.2.212:4001 isLeader=false
baab0aae8b58c802[unstarted]: peerURLs=http://192.168.2.210:2380
e4a3e95f72ced4a7: name=etcd2 peerURLs=http://192.168.2.211:2380 clientURLs=http://192.168.2.211:2379,http://192.168.2.211:4001 isLeader=true
```

c. 清空目标节点数据

目标节点从集群中删除后，成员信息会更新。新节点是作为一个全新的节点加入集群，如果data-dir有数据，etcd启动时会读取己经存在的数据，仍然用旧的memberID会造成无法加入集群，所以一定要清空新节点的data-dir。

```bash
$ rm -rf /var/lib/etcd/etcd1
```

d. 在目标节点上启动新增加的成员

修改配置文件中`ETCD_INITIAL_CLUSTER_STATE`标记为`existing`，如果为new，则会自动生成一个新的memberID，这和前面添加节点时生成的ID不一致，故日志中会报节点ID不匹配的错。


```bash
$ vim /opt/etcd/config/etcd.conf
ETCD_INITIAL_CLUSTER_STATE="existing"
```

启动etcd

```bash
$ systemctl start etcd
```

查看新节点是否成功加入


```bash
$ etcdctl --endpoints "http://192.168.2.210:2379" member list
a3ba19408fd4c829: name=etcd3 peerURLs=http://192.168.2.212:2380 clientURLs=http://192.168.2.212:2379,http://192.168.2.212:4001 isLeader=false
baab0aae8b58c802: name=etcd1 peerURLs=http://192.168.2.210:2380 clientURLs=http://192.168.2.210:2379,http://192.168.2.210:4001 isLeader=false
e4a3e95f72ced4a7: name=etcd2 peerURLs=http://192.168.2.211:2380 clientURLs=http://192.168.2.211:2379,http://192.168.2.211:4001 isLeader=true
```

参考文档
[https://www.hi-linux.com/posts/49138.html](https://www.hi-linux.com/posts/49138.html)
