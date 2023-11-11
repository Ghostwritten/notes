#  Kubernetes coreDNS 如何域名解析

![](https://img-blog.csdnimg.cn/7c0130aa4b6f438097e033358a8c578e.png)





## 1. 前提
- [DNS部署配置（1）详解](https://ghostwritten.blog.csdn.net/article/details/106596034)

### 1.1 完全限定名称
完全限定域名(FQDN)就是互联网上计算机或者主机的完整域名。由主机名、域名、顶级域组成。`FQDN= HostName + DomainName`

如：域名 `www.ayunw.cn` ，实际上它应该是 `www.ayunw.cn.` ，而通常最后的点可以不写。最后的点被称为根域 `www`就是主机名，`ayunw.cn`就是域名，而`.cn`又被称为顶级域(一级域名)，`ayunw`被称为二级域名，最后的点被称为 根域。

如：`www.allen.ayunw.cn.` ，其中最后的点被称为 根域(TLD)，cn被称为顶级域(一级域名)，ayunw被称为二级域名，allen被称为三级域名，www被称为主机名。

k8s中，非完全限定名称比如：`demo-hello.paas.svc.cluster.local`


###  1.2 无类域间路由(CIDR)
将 IP 地址分为 A 类、B 类、C 类后，会造成 IP 地址的部分浪费。例如，一些连续的 IP 地址，一部分属于 A 类地址，另一部分属于 B 类地址。为了使这些地址聚合以方便管理，出现了 CIDR（无类域间路由）。

无类域间路由（Classless Inter-Domain Routing，CIDR）可以将路由集中起来，在路由表中更灵活地定义地址。它不区分 A 类、B 类、C 类地址，而是使用 CIDR 前缀的值指定地址中作为网络 ID 的位数。

这个前缀可以位于地址空间的任何位置，让管理者能够以更灵活的方式定义子网，以简便的形式指定地址中网络 ID 部分和主机 ID 部分。

CIDR 标记使用一个斜线/分隔符，后面跟一个十进制数值表示地址中网络部分所占的位数。例如，`205.123.196.183/25` 中的 25 表示地址中 25 位用于网络 ID，相应的掩码为 `255.255.255.128`

【示例】已知 CIDR 格式地址为 192.168.1.32/27，计算该地址的掩码，并显示包含了多少台主机。

 - [netwox用法](https://blog.csdn.net/xixihahalelehehe/article/details/118305820?spm=1001.2014.3001.5501)

1) 列出包含的所有主机。

```bash
root@daxueba:~# netwox 213 -i 192.168.1.32/27

输出信息如下：
192.168.1.32
192.168.1.33
192.168.1.34
192.168.1.35
192.168.1.36
192.168.1.37
192.168.1.38
…  #省略其他信息
192.168.1.57
192.168.1.58
192.168.1.59
192.168.1.60
192.168.1.61
192.168.1.62
192.168.1.63
```

上述输出信息显示该 CIDR 地址中包含了 32 台主机，IP 地址为 192.168.1.32～192.168.1.63。

2) 计算 IP 地址 192.168.1.32/27 的掩码。

```bash
root@daxueba:~# netwox 24 -i 192.168.1.32/27

输出信息如下：
192.168.1.32-192.168.1.63  #IP地址范围
192.168.1.32/27  #IP地址段
192.168.1.32/255.255.255.224  #掩码
localhost=localhost  #主机名
```

上述输出信息显示掩码为 255.255.255.224。

##  2. search 和 ndots 配置说明
Kubernetes 集群中，域名解析离不开 DNS 服务，在 Kubernetes v1.10 以前集群使用 kube-dns dns服务，后来在 Kubernetes v1.10+ 使用 Coredns 做为集群dns服务。

使用 Kubernetes 集群时，会发现 Pod `/etc/resolv.conf` 配置。具体如下：

```bash
nameserver 10.10.0.2
search production.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```
词解释

 - search：搜索主机名查找列表。搜索列表目前仅限于6个域名，共计256个字符。
 - ndots：通俗一点说，如果你的域名请求参数中，点的个数比配置的ndots小，则会按照配置的search内容，依次添加相应的后缀直到获取到域名解析后的地址。如果通过添加了search之后还是找不到域名，则会按照一开始请求的域名进行解析。

## 3. 域名抓包分析示例

### 3.1 安装配置
里，我自己有一个域名叫 `www.ayunw.cn` ，然后这里我尝试用一个 paas 名称空间下的一个pod对 `www.ayunw.cn` 做 `nslookup` 域名解析。并且对某一个coredns的pod进行抓包分析。

为了测试，我这里用一个已经发布好测试的容器。进入容器，查看 `/etc/resolv.conf` 文件内容

 - [nsenter进入pod请参考](https://ghostwritten.blog.csdn.net/article/details/118297433)

```bash
root@demo-hello-perf-dev-v0-5-0-f9f9cd5c9-r27cw:/# cat /etc/resolv.conf
nameserver 10.10.0.2
search paas.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```
在该容器中安装 nslookup 工具，然后对 www.ayunw.cn 域名进行解析

```bash
[root@kube-master-srv1 ~]# kubectl get po -n paas
NAME                                                     READY   STATUS    RESTARTS   AGE
demo-hello-perf-dev-v0-5-0-f9f9cd5c9-r27cw               1/1     Running   0          11d

[root@kube-master-srv1 ~]# kubectl exec -it demo-hello-perf-dev-v0-5-0-f9f9cd5c9-r27cw -n paas -- bash
root@demo-hello-perf-dev-v0-5-0-f9f9cd5c9-r27cw:/# cat /etc/issue
Debian GNU/Linux 10 \n \l
root@demo-hello-perf-dev-v0-5-0-f9f9cd5c9-r27cw:/# apt -y install dnsutils
```
接着找到某一个coredns，然后去他所调度到的node节点通过nsenter进入网络名称空间进行抓包分析

```bash
# 在k8s-master上查看coredns调度在哪个node
# 接着我就选择了第一个coredns
[root@kube-master-srv1 ~]# kubectl get po -n kube-system -o wide | grep coredns
coredns-69d9b6c494-4nrxt                   1/1     Running   0          96d   10.20.246.18    node2.core      <none>           <none>
coredns-69d9b6c494-6vjw4                   1/1     Running   0          96d   10.20.240.239   node3.core      <none>           <none>
coredns-69d9b6c494-pw5gx                   1/1     Running   0          96d   10.20.240.232   node3.core      <none>           <none>


# 登录到 node2.core 节点，找到coredns的pid
# 进入这个pid进入coredns容器的网络名称空间进行抓包过滤分析
[root@kube-node-srv2 ~]# docker ps -a | grep coredns
4d38fd311a78        bfe3a36ebd25                                                                 "/coredns -conf /etc…"   3 months ago        Up 3 months       k8s_coredns_coredns-69d9b6c494-4nrxt_kube-system_803290a5-b4bd-4f2e-81b3-5ce82c9aa57c_0
00722e50786b        registry.xx.xx/library/k8s.gcr.io/pause:3.2                     "/pause"                 3 months ago        Up 3 months       k8s_POD_coredns-69d9b6c494-4nrxt_kube-system_803290a5-b4bd-4f2e-81b3-5ce82c9aa57c_0
[root@kube-node-srv2 ~]# docker inspect -f {{.State.Pid}} 4d38fd311a78
896949
[root@kube-node-srv2 ~]# nsenter -n -t 896949
[root@kube-node-srv2 ~]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1380
        inet 10.20.246.18  netmask 255.255.255.255  broadcast 10.20.246.18
        ether 46:c1:e0:30:b4:9d  txqueuelen 0  (Ethernet)
        RX packets 1489941923  bytes 162419228606 (151.2 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1488233127  bytes 297011464372 (276.6 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 83731165  bytes 6681735331 (6.2 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 83731165  bytes 6681735331 (6.2 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
### 3.2 解析k8s集群的内部域名
这里说的集群内部域名就是 service的名字。我这里用的是kubernetes这个service来测试 连续解析6次，为了方便查看，我每执行一次解析，下面抓包的终端就敲一次回车

```bash
[root@kube-master-srv1 ~]# kubectl get svc kubernetes
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.10.0.1    <none>        443/TCP   57d

root@demo-hello-perf-dev-v0-5-0-f9f9cd5c9-r27cw:/# nslookup kubernetes.default
Server:  10.10.0.2
Address: 10.10.0.2#53

Name: kubernetes.default.svc.cluster.local
Address: 10.10.0.1
```
抓包分析
以下是抓取kubernetes这个域名的DNS包的结果

```bash
[root@kube-node-srv2 ~]# tcpdump -i eth0 port 53 | grep "kubernetes"
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
16:44:42.712421 IP 10.20.105.252.60020 > qing-core-kube-node-srv2.domain: 7282+ A? kubernetes.default.svc.cluster.local. (54)

16:44:48.883881 IP 10.20.105.252.ndm-agent-port > qing-core-kube-node-srv2.domain: 25500+ AAAA? kubernetes.default.svc.cluster.local. (54)

16:50:15.361021 IP 10.20.105.252.57205 > qing-core-kube-node-srv2.domain: 24061+ A? kubernetes.default.paas.svc.cluster.local. (59)

16:50:22.186723 IP 10.20.105.252.60715 > qing-core-kube-node-srv2.domain: 55799+ AAAA? kubernetes.default.svc.cluster.local. (54)
16:50:27.813477 IP qing-core-kube-node-srv2.domain > 10.20.176.128.8181: 21787*- 1/0/0 PTR kubernetes.default.svc.cluster.local. (112)


16:46:04.429250 IP 10.20.105.252.33895 > qing-core-kube-node-srv2.domain: 37943+ A? kubernetes.default.svc.cluster.local.svc.cluster.local. (72)
16:46:04.441717 IP 10.20.105.252.54502 > qing-core-kube-node-srv2.domain: 45454+ AAAA? kubernetes.default.svc.cluster.local. (54)


16:46:10.771445 IP 10.20.105.252.54594 > qing-core-kube-node-srv2.domain: 16257+ A? kubernetes.default.svc.cluster.local.svc.cluster.local. (72)
16:46:10.783322 IP 10.20.105.252.59768 > qing-core-kube-node-srv2.domain: 60408+ AAAA? kubernetes.default.svc.cluster.local. (54)
```
通过以上抓包分析得出结论。当解析kubernetes域名的时候，点的个数比ndots的值小，则按照search后面的本地域参数填补了域名后缀，当按照顺序 用 `paas.svc.cluster.local` 填补的时候解析到了A记录。然后终止dns查询将查询到的A记录返回。

通过host命令对名为kubernetes的service的集群内部域名进行解析

```bash
root@demo-hello-pro-master-5474b97bdf-fvbm5:/# host -v kubernetes.default
Trying "kubernetes.default.paas.svc.cluster.local"
Trying "kubernetes.default.svc.cluster.local"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 18054
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;kubernetes.default.svc.cluster.local. IN A

;; ANSWER SECTION:
kubernetes.default.svc.cluster.local. 5 IN A 10.10.0.1

Received 106 bytes from 10.10.0.2#53 in 3 ms
Trying "kubernetes.default.svc.cluster.local"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 58952
;; flags: qr aa rd; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 0

;; QUESTION SECTION:
;kubernetes.default.svc.cluster.local. IN AAAA

;; AUTHORITY SECTION:
cluster.local.  5 IN SOA ns.dns.cluster.local. hostmaster.cluster.local. 1622445553 7200 1800 86400 5

Received 147 bytes from 10.10.0.2#53 in 2 ms
Trying "kubernetes.default.svc.cluster.local"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 37783
;; flags: qr aa rd; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 0

;; QUESTION SECTION:
;kubernetes.default.svc.cluster.local. IN MX

;; AUTHORITY SECTION:
cluster.local.  5 IN SOA ns.dns.cluster.local. hostmaster.cluster.local. 1622445553 7200 1800 86400 5

Received 147 bytes from 10.10.0.2#53 in 2 ms
```
### 3.3 解析k8s集群外部域名

接下来针对我的 www.ayunw.cn 这个域名进行多次解析。这里我为了测试，发起了6 次解析。我每执行一次解析，下面抓包的终端就敲一次回车。解析的同时去coredns这个容器所在的节点进行抓包分析。

```bash
root@demo-hello-perf-dev-v0-5-0-f9f9cd5c9-r27cw:/# nslookup www.ayunw.cn
Server:  10.10.0.2
Address: 10.10.0.2#53

Non-authoritative answer:
Name: www.ayunw.cn
Address: 134.175.123.64
```

抓包分析
抓包开始，由于我的集群有大量的服务，每秒都有很多内部服务dns解析请求。所以这里我过滤了关键字ayunw。上面的dns每执行一次，我在这个抓包的窗口就敲一下回车，这样的话方便看清楚每一次的解析结果

以下是抓 www.ayunw.cn 的域名DNS包的结果：

```bash
[root@kube-node-srv2 ~]# tcpdump -i eth0 port 53 | grep "ayunw"
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
14:38:07.350640 IP 10.20.105.252.47767 > qing-core-kube-node-srv2.domain: 13102+ A? www.ayunw.cn.cluster.local. (44)

14:38:19.098753 IP 10.20.105.252.47071 > qing-core-kube-node-srv2.domain: 15535+ A? www.ayunw.cn.paas.svc.cluster.local. (53)
14:38:19.111441 IP 10.20.105.252.56968 > qing-core-kube-node-srv2.domain: 62838+ A? www.ayunw.cn. (30)
14:38:19.111720 IP qing-core-kube-node-srv2.35187 > 172.16.0.11.domain: 62838+ A? www.ayunw.cn. (30)

14:38:31.200982 IP 10.20.105.252.50777 > qing-core-kube-node-srv2.domain: 10715+ A? www.ayunw.cn.svc.cluster.local. (48)
14:38:31.214096 IP 10.20.105.252.51233 > qing-core-kube-node-srv2.domain: 37585+ AAAA? www.ayunw.cn. (30)
14:38:31.214299 IP qing-core-kube-node-srv2.35187 > 172.16.0.11.domain: 37585+ AAAA? www.ayunw.cn. (30)
14:39:04.691754 IP 10.20.105.252.34080 > qing-core-kube-node-srv2.domain: 34206+ A? www.ayunw.cn.paas.svc.cluster.local. (53)
14:39:04.704758 IP 10.20.105.252.36478 > qing-core-kube-node-srv2.domain: 64751+ A? www.ayunw.cn. (30)
14:39:04.705068 IP qing-core-kube-node-srv2.48926 > 172.16.0.11.domain: 64751+ A? www.ayunw.cn. (30)

14:39:13.925872 IP 10.20.105.252.59868 > qing-core-kube-node-srv2.domain: 45121+ A? www.ayunw.cn.paas.svc.cluster.local. (53)
14:39:13.937328 IP 10.20.105.252.45290 > qing-core-kube-node-srv2.domain: 27511+ A? www.ayunw.cn. (30)
14:39:13.937576 IP qing-core-kube-node-srv2.48926 > 172.16.0.11.domain: 27511+ A? www.ayunw.cn. (30)

14:39:24.838444 IP 10.20.105.252.37510 > qing-core-kube-node-srv2.domain: 45926+ A? www.ayunw.cn.cluster.local. (44)

14:45:13.438961 IP 10.20.105.252.55462 > qing-core-kube-node-srv2.domain: 60170+ A? www.ayunw.cn.paas.svc.cluster.local. (53)
14:45:13.450865 IP 10.20.105.252.42674 > qing-core-kube-node-srv2.domain: 25680+ A? www.ayunw.cn. (30)
14:45:13.451110 IP qing-core-kube-node-srv2.56396 > 172.16.0.11.domain: 25680+ A? www.ayunw.cn. (30)

^C35952 packets captured
35956 packets received by filter
0 packets dropped by kernel
```

从上面抓包分析的结果来看， www.ayunw.cn 的这个域名只有两个点，比pod里面 /etc/resolv.conf 文件中的 ndots 配置的值小(ndots的值为5，域名的点为2)。则会按照search的参数填补域名后缀，并且是根据search后面的顺序 paas.svc.cluster.local 、 svc.cluster.local 、 cluster.local 依次来填充的。因为根据search后面的本地域匹配后都没有域名解析的结果，因此他就直接解析了 www.ayunw.cn 这个域名查询到了该域名的A记录并且返回了结果。

通过host命令来进行解析

```bash
root@demo-hello-pro-master-5474b97bdf-fvbm5:/# host -v www.ayunw.cn
Trying "www.ayunw.cn.paas.svc.cluster.local"
Trying "www.ayunw.cn.svc.cluster.local"
Trying "www.ayunw.cn.cluster.local"
Trying "www.ayunw.cn"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 8135
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 13, ADDITIONAL: 0

;; QUESTION SECTION:
;www.ayunw.cn.   IN A

;; ANSWER SECTION:
www.ayunw.cn.  30 IN A 134.175.123.64

;; AUTHORITY SECTION:
.   30 IN NS l.root-servers.net.
.   30 IN NS e.root-servers.net.
.   30 IN NS h.root-servers.net.
.   30 IN NS k.root-servers.net.
.   30 IN NS d.root-servers.net.
.   30 IN NS b.root-servers.net.
.   30 IN NS g.root-servers.net.
.   30 IN NS j.root-servers.net.
.   30 IN NS m.root-servers.net.
.   30 IN NS i.root-servers.net.
.   30 IN NS f.root-servers.net.
.   30 IN NS c.root-servers.net.
.   30 IN NS a.root-servers.net.

Received 461 bytes from 10.10.0.2#53 in 94 ms
Trying "www.ayunw.cn"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 11085
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 0

;; QUESTION SECTION:
;www.ayunw.cn.   IN AAAA

;; AUTHORITY SECTION:
ayunw.cn.  5 IN SOA dns17.hichina.com. hostmaster.hichina.com. 2019070911 3600 1200 86400 360

Received 113 bytes from 10.10.0.2#53 in 99 ms
Trying "www.ayunw.cn"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 19432
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 0

;; QUESTION SECTION:
;www.ayunw.cn.   IN MX

;; AUTHORITY SECTION:
ayunw.cn.  5 IN SOA dns17.hichina.com. hostmaster.hichina.com. 2019070911 3600 1200 86400 360

Received 113 bytes from 10.10.0.2#53 in 51 ms
```

因为我的pod中存在三个本地域：`paas.svc.cluster.local` 、 `svc.cluster.local` 、 `cluster.local` ，通过host命令可以看到，Trying 一共尝试了四次，一次根据我search后面的本地域进行了解析搜索，结果没有搜索到正确的解析，因此通过pod所在的宿主机本地的 /etc/resolv.conf 文件中进行了解析。

本地宿主机的/etc/resolv.conf的解析如下: 我这里公司用了内部bind服务，做了内部dns，然后上游指向了百度的dns。

```bash
# cat /etc/resolv.conf
options rotate timeout:1
; generated by /usr/sbin/dhclient-script
nameserver 172.16.0.11
nameserver 172.16.0.12
```
###  3.4 查看域名的点数等于ndots的值5的域名解析

```bash
root@demo-hello-perf-dev-v0-5-0-f9f9cd5c9-r27cw:/# nslookup x.y.z.v.ayunw.cn
Server:  10.10.0.2
Address: 10.10.0.2#53

Non-authoritative answer:
Name: x.y.z.v.ayunw.cn
Address: 134.175.123.64
抓包分析
[root@kube-node-srv2 ~]# tcpdump -i eth0 port 53 | grep "ayunw"
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
16:36:49.928116 IP 10.20.105.252.46581 > qing-core-kube-node-srv2.domain: 38769+ A? x.y.z.v.ayunw.cn. (34)
16:36:49.928383 IP qing-core-kube-node-srv2.59801 > 172.16.0.11.domain: 38769+ A? x.y.z.v.ayunw.cn. (34)

16:36:56.901762 IP 10.20.105.252.43844 > qing-core-kube-node-srv2.domain: 3524+ A? x.y.z.v.ayunw.cn. (34)

16:37:01.763743 IP 10.20.105.252.36053 > qing-core-kube-node-srv2.domain: 62952+ AAAA? x.y.z.v.ayunw.cn. (34)
16:37:01.764110 IP qing-core-kube-node-srv2.59801 > 172.16.0.11.domain: 62952+ AAAA? x.y.z.v.ayunw.cn. (34)

16:37:06.851820 IP 10.20.105.252.36305 > qing-core-kube-node-srv2.domain: 58393+ AAAA? x.y.z.v.ayunw.cn. (34)
16:37:06.852118 IP qing-core-kube-node-srv2.59801 > 172.16.0.11.domain: 58393+ AAAA? x.y.z.v.ayunw.cn. (34)
```

从上述抓包结果可以看到，如果域名中的点等于ndots的值，他会直接解析域名，不会用search后面的本地域来填补的。可能因为我阿里云上这个域名的原因，不支持超过5个点的域名解析。所以超过5个点的域名我无法测试。

## 4. 总结
如果点的个数小于5个，那么会根据search中配置的本地域列表一次在对应域中先进行搜索。如果没有返回，则最后再查询域名本身。如果说search中配置的本地域列表没有一个匹配的，那么就会走到服务器宿主机的/etc/resolv.conf中去解析。如果你kubelet中clusterdomain配置错了。那么search中没有任何一个匹配的到，直接转发到本地DNS，走正常的递归查询逻辑。

通过以上测试发现ndots的值和请求的域名是相关的。为了避免多次的DNS解析查询，可以将需要进行解析的域名进行相对的优化 尽可能将域名中的点都带上，并且最好是等于ndots的值。比如：`kubernetes.paas.svc.cluster.local`。这样他就直接解析到了这个域名返回了A记录而不是在通过search后面的本地域去解析多次。如果你解析的域名是kubernetes.paas他就会根据search后面的本地域去进行补全解析多次了 在同一个namespace下可以直接解析service的名称。比如：nslookup kubernetes，他会补全default.svc.cluster.local


参考：

- [K8S Pod 内抓包快速定位网络问题](https://mp.weixin.qq.com/s?__biz=MzA4MzIwNTc4NQ==&mid=2247484821&idx=1&sn=aaa173a8b913cc759c56efe05d2c15ad&chksm=9ffb4e63a88cc7755ca720b3886946245246d8e57687cf209ad47f61a186cc290c04ef57fc91&scene=21#wechat_redirect)
- [聊聊 resolv.conf 中 search 和 ndots 配置](https://cloud.tencent.com/developer/article/1669860)
