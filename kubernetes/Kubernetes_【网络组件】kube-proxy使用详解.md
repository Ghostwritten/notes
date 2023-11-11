#  kubernetes kube-proxy
tags: kube-proxy








[
![在这里插入图片描述](https://img-blog.csdnimg.cn/f89f89db2d1c4e1c98a7a8511b34a180.png)](https://www.rottentomatoes.com/m/gravity_2013)

*《地心引力》*

---
## 1. 介绍
[Kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/) 是一个简单的网络代理和负载均衡器，它的作用主要是负责Service的实现，具体来说，就是实现了内部从Pod到Service和外部的从NodePort向Service的访问，每台机器上都运行一个 kube-proxy 服务，它监听 API server 中 service 和 endpoint 的变化情况，并通过 iptables 等来为服务配置负载均衡（仅支持 TCP 和 UDP）。

kube-proxy 可以直接运行在物理机上，也可以以 static pod 或者 daemonset 的方式运行。

## 2. 原理
在k8s中，提供相同服务的一组pod可以抽象成一个service，通过service提供的统一入口对外提供服务，每个service都有一个虚拟IP地址（VIP）和端口号供客户端访问。kube-proxy存在于各个node节点上，主要用于Service功能的实现，具体来说，就是实现集群内的客户端pod访问service，或者是集群外的主机通过NodePort等方式访问service。在当前版本的k8s中，kube-proxy默认使用的是iptables模式，通过各个node节点上的iptables规则来实现service的负载均衡，但是随着service数量的增大，iptables模式由于线性查找匹配、全量更新等特点，其性能会显著下降。从k8s的1.8版本开始，kube-proxy引入了IPVS模式，IPVS模式与iptables同样基于Netfilter，但是采用的hash表，因此当service数量达到一定规模时，hash查表的速度优势就会显现出来，从而提高service的服务性能。


kube-proxy负责为Service提供cluster内部的服务发现和负载均衡，它运行在每个Node计算节点上，负责Pod网络代理, 它会定时从etcd服务获取到service信息来做相应的策略，维护网络规则和四层负载均衡工作。在K8s集群中微服务的负载均衡是由Kube-proxy实现的，它是K8s集群内部的负载均衡器，也是一个分布式代理服务器，在K8s的每个节点上都有一个，这一设计体现了它的伸缩性优势，需要访问服务的节点越多，提供负载均衡能力的Kube-proxy就越多，高可用节点也随之增多。

service是一组pod的服务抽象，相当于一组pod的LB，负责将请求分发给对应的pod。service会为这个LB提供一个IP，一般称为cluster IP。kube-proxy的作用主要是负责service的实现，具体来说，就是实现了内部从pod到service和外部的从node port向service的访问。


简单来说: 

 - ->  kube-proxy其实就是管理service的访问入口，包括集群内Pod到Service的访问和集群外访问service。
 - ->  kube-proxy管理sevice的Endpoints，该service对外暴露一个Virtual IP，也成为Cluster IP, 集群内通过访问这个Cluster IP:Port就能访问到集群内对应的serivce下的Pod。
 - ->  service是通过Selector选择的一组Pods的服务抽象，其实就是一个微服务，提供了服务的LB和反向代理的能力，而kube-proxy的主要作用就是负责service的实现。
 - ->  service另外一个重要作用是，一个服务后端的Pods可能会随着生存灭亡而发生IP的改变，service的出现，给服务提供了一个固定的IP，而无视后端Endpoint的变化。

举个例子，比如现在有podA，podB，podC和serviceAB。serviceAB是podA，podB的服务抽象(service)。那么kube-proxy的作用就是可以将pod(不管是podA，podB或者podC)向serviceAB的请求，进行转发到service所代表的一个具体pod(podA或者podB)上。请求的分配方法一般分配是采用轮询方法进行分配。另外，kubernetes还提供了一种在node节点上暴露一个端口，从而提供从外部访问service的方式。比如这里使用这样的一个manifest来创建service

举个例子，比如现在有podA，podB，podC和serviceAB。serviceAB是podA，podB的服务抽象(service)。那么kube-proxy的作用就是可以将pod(不管是podA，podB或者podC)向serviceAB的请求，进行转发到service所代表的一个具体pod(podA或者podB)上。请求的分配方法一般分配是采用轮询方法进行分配。另外，kubernetes还提供了一种在node节点上暴露一个端口，从而提供从外部访问service的方式。比如这里使用这样的一个manifest来创建service

```bash
apiVersion: v1
kind: Service
metadata:
  labels:
    name: mysql
    role: service
  name: mysql-service
spec:
  ports:
    - port: 3306
      targetPort: 3306
      nodePort: 30964
  type: NodePort
  selector:
    mysql-service: "true"
```

上面配置的含义是在node上暴露出30964端口。当访问node上的30964端口时，其请求会转发到service对应的cluster IP的3306端口，并进一步转发到pod的3306端口。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401113617620.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

## 3. Service, Endpoints与Pod的关系
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401135532843.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

Kube-proxy进程获取每个Service的Endpoints,实现Service的负载均衡功能
Service的负载均衡转发规则

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401135605151.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
**访问Service的请求，不论是Cluster IP+TargetPort的方式；还是用Node节点IP+NodePort的方式，都被Node节点的Iptables规则重定向到Kube-proxy监听Service服务代理端口。kube-proxy接收到Service的访问请求后，根据负载策略，转发到后端的Pod。**

## 4. kubernetes服务发现
Kubernetes提供了两种方式进行服务发现, 即环境变量和DNS, 简单说明如下:

1)  环境变量： 当你创建一个Pod的时候，kubelet会在该Pod中注入集群内所有Service的相关环境变量。需要注意: 要想一个Pod中注入某个Service的环境变量，则必须Service要先比该Pod创建。这一点，几乎使得这种方式进行服务发现不可用。比如，一个ServiceName为redis-master的Service，对应的ClusterIP:Port为172.16.50.11:6379，则其对应的环境变量为:

```bash
REDIS_MASTER_SERVICE_HOST=172.16.50.11
REDIS_MASTER_SERVICE_PORT=6379
REDIS_MASTER_PORT=tcp://172.16.50.11:6379
REDIS_MASTER_PORT_6379_TCP=tcp://172.16.50.11:6379
REDIS_MASTER_PORT_6379_TCP_PROTO=tcp
REDIS_MASTER_PORT_6379_TCP_PORT=6379
REDIS_MASTER_PORT_6379_TCP_ADDR=172.16.50.11
```

2)  DNS：这是k8s官方强烈推荐的方式!!!  可以通过cluster add-on方式轻松的创建KubeDNS来对集群内的Service进行服务发现。

## 5. kubernetes发布(暴露)服务
kubernetes原生的，一个Service的`ServiceType`决定了其发布服务的方式。

 - ->  `ClusterIP`：这是k8s默认的ServiceType。通过集群内的ClusterIP在内部发布服务。
 - ->  `NodePort`：这种方式是常用的，用来对集群外暴露Service，你可以通过访问集群内的每个NodeIP:NodePort的方式，访问到对应Service后端的Endpoint。
 - ->  `LoadBalancer`: 这也是用来对集群外暴露服务的，不同的是这需要Cloud Provider的支持，比如AWS等。
 - ->  `ExternalName`：这个也是在集群内发布服务用的，需要借助KubeDNS(version >= 1.7)的支持，就是用KubeDNS将该service和ExternalName做一个Map，KubeDNS返回一个CNAME记录。

## 6. kube-proxy 不足
kube-proxy 目前仅支持 TCP 和 UDP，不支持 HTTP 路由，并且也没有健康检查机制。这些可以通过自定义 [Ingress Controller](https://feisky.gitbooks.io/kubernetes/content/plugins/ingress.html) 的方法来解决。




## 7. 实现方式

 - userspace：最早的负载均衡方案，它在用户空间监听一个端口，所有服务通过 iptables转发到这个端口，然后在其内部负载均衡到实际的 Pod。该方式最主要的问题是效率低，有明显的性能瓶颈。
 - iptables：目前推荐的方案，完全以 iptables 规则的方式来实现 service负载均衡。该方式最主要的问题是在服务多的时候产生太多的 iptables 规则，非增量式更新会引入一定的时延，大规模情况下有明显的性能问题
 - ipvs：为解决 iptables 模式的性能问题，v1.11 新增了 ipvs 模式（v1.8 开始支持测试版，并在 v1.11
   GA），采用增量式更新，并可以保证 service 更新期间连接保持不断开
 - winuserspace：同 userspace，但仅工作在 windows 节点上

注意：使用 ipvs 模式时，需要预先在每台 Node 上加载内核模块 nf_conntrack_ipv4, ip_vs, ip_vs_rr, ip_vs_wrr, ip_vs_sh 等。

```bash
# load module <module_name>
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4

# to check loaded modules, use
lsmod | grep -e ip_vs -e nf_conntrack_ipv4
# or
cut -f1 -d " "  /proc/modules | grep -e ip_vs -e nf_conntrack_ipv4
```

### 7.1 userspace mode
在k8s v1.2后就已经被淘汰了，userspace的作用就是在proxy的用户空间监听一个端口，所有的svc都转到这个端口，然后proxy的内部应用层对其进行转发。proxy会为每个svc随机监听一个端口，并增加一个iptables规则，从客户端到 ClusterIP:Port 的报文都会被重定向到 Proxy Port，Kube-Proxy 收到报文后，通过 Round Robin (轮询) 或者 Session Affinity（会话亲和力，即同一 Client IP 都走同一链路给同一 Pod 服务）分发给对应的 Pod。所有流量都是在用户空间进行转发的，虽然比较稳定，但是效率不高。如下图为userspace的工作流程。
serspace是在用户空间，通过kube-proxy来实现service的代理服务,  其原理如下:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401211423686.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
可见，userspace这种mode最大的问题是，service的请求会先从用户空间进入内核iptables，然后再回到用户空间，由kube-proxy完成后端Endpoints的选择和代理工作，这样流量从用户空间进出内核带来的性能损耗是不可接受的。这也是k8s v1.0及之前版本中对kube-proxy质疑最大的一点，因此社区就开始研究iptables mode.

userspace这种模式下，kube-proxy 持续监听 Service 以及 Endpoints 对象的变化；对每个 Service，它都为其在本地节点开放一个端口，作为其服务代理端口；发往该端口的请求会采用一定的策略转发给与该服务对应的后端 Pod 实体。kube-proxy 同时会在本地节点设置 iptables 规则，配置一个 Virtual IP，把发往 Virtual IP 的请求重定向到与该 Virtual IP 对应的服务代理端口上。其工作流程大体如下:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401211505152.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
由此分析: 该模式请求在到达 iptables 进行处理时就会进入内核，而 kube-proxy 监听则是在用户态, 请求就形成了从用户态到内核态再返回到用户态的传递过程, 一定程度降低了服务性能。

### 7.2 Iptables mode
iptables这种模式是从kubernetes1.2开始并在v1.12之前的默认模式。在这种模式下proxy监控kubernetes对svc和ep对象进行增删改查。并且这种模式使用iptables来做用户态的入口，而真正提供服务的是内核的Netilter，Netfilter采用模块化设计，具有良好的可扩充性。其重要工具模块IPTables从用户态的iptables连接到内核态的Netfilter的架构中，Netfilter与IP协议栈是无缝契合的，并允许使用者对数据报进行过滤、地址转换、处理等操作。这种情况下proxy只作为Controller。Kube-Proxy 监听 Kubernetes Master 增加和删除 Service 以及 Endpoint 的消息。对于每一个 Service，Kube Proxy 创建相应的 IPtables 规则，并将发送到 Service Cluster IP 的流量转发到 Service 后端提供服务的 Pod 的相应端口上。并且流量的转发都是在内核态进行的，所以性能更高更加可靠。

在这种模式下缺点就是在大规模的集群中，iptables添加规则会有很大的延迟。因为使用iptables，每增加一个svc都会增加一条iptables的chain。并且iptables修改了规则后必须得全部刷新才可以生效。

iptables 自定义几条链路：KUBE-SERVICES，KUBE-NODEPORTS，KUBE-POSTROUTING，KUBE-MARK-MASQ和KUBE-MARK-DROP五个链，并主要通过为 KUBE-SERVICES链（附着在PREROUTING和OUTPUT）增加rule来配制traffic routing 规则。



iptabels 模式下正常的通信链路：

在 PREROUTING的 chain里将 将经过PREROUTING里的数据包重定向到KUBE-SERVICES中

在自定义链kube-services里找到dst为目标地址的ip（ps：kube-services对于相同目标地址都有2给target，对于非pod之间的访问进入KUBE-MARK-MASQ，对于pod之间的访问进入KUBE-SVC-*里)，找到对应的KUBE-SVC-*(ps:只有有对应endpoint信息的才有KUBE-SVC-*)，并找到KUBE-SVC-*对应的KUBE-SEP-*。在SEP里会对source来自自身ip的打KUBE-MARK-MASQ，对于其他的做DNAT转换
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401112856314.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
# Masquerade
-A KUBE-MARK-DROP -j MARK --set-xmark 0x8000/0x8000
-A KUBE-MARK-MASQ -j MARK --set-xmark 0x4000/0x4000
-A KUBE-POSTROUTING -m comment --comment "kubernetes service traffic requiring SNAT" -m mark --mark 0x4000/0x4000 -j MASQUERADE

# clusterIP and publicIP
-A KUBE-SERVICES ! -s 10.244.0.0/16 -d 10.98.154.163/32 -p tcp -m comment --comment "default/nginx: cluster IP" -m tcp --dport 80 -j KUBE-MARK-MASQ
-A KUBE-SERVICES -d 10.98.154.163/32 -p tcp -m comment --comment "default/nginx: cluster IP" -m tcp --dport 80 -j KUBE-SVC-4N57TFCL4MD7ZTDA
-A KUBE-SERVICES -d 12.12.12.12/32 -p tcp -m comment --comment "default/nginx: loadbalancer IP" -m tcp --dport 80 -j KUBE-FW-4N57TFCL4MD7ZTDA

# Masq for publicIP
-A KUBE-FW-4N57TFCL4MD7ZTDA -m comment --comment "default/nginx: loadbalancer IP" -j KUBE-MARK-MASQ
-A KUBE-FW-4N57TFCL4MD7ZTDA -m comment --comment "default/nginx: loadbalancer IP" -j KUBE-SVC-4N57TFCL4MD7ZTDA
-A KUBE-FW-4N57TFCL4MD7ZTDA -m comment --comment "default/nginx: loadbalancer IP" -j KUBE-MARK-DROP

# Masq for nodePort
-A KUBE-NODEPORTS -p tcp -m comment --comment "default/nginx:" -m tcp --dport 30938 -j KUBE-MARK-MASQ
-A KUBE-NODEPORTS -p tcp -m comment --comment "default/nginx:" -m tcp --dport 30938 -j KUBE-SVC-4N57TFCL4MD7ZTDA

# load balance for each endpoints
-A KUBE-SVC-4N57TFCL4MD7ZTDA -m comment --comment "default/nginx:" -m statistic --mode random --probability 0.33332999982 -j KUBE-SEP-UXHBWR5XIMVGXW3H
-A KUBE-SVC-4N57TFCL4MD7ZTDA -m comment --comment "default/nginx:" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-TOYRWPNILILHH3OR
-A KUBE-SVC-4N57TFCL4MD7ZTDA -m comment --comment "default/nginx:" -j KUBE-SEP-6QCC2MHJZP35QQAR

# endpoint #1
-A KUBE-SEP-6QCC2MHJZP35QQAR -s 10.244.3.4/32 -m comment --comment "default/nginx:" -j KUBE-MARK-MASQ
-A KUBE-SEP-6QCC2MHJZP35QQAR -p tcp -m comment --comment "default/nginx:" -m tcp -j DNAT --to-destination 10.244.3.4:80

# endpoint #2
-A KUBE-SEP-TOYRWPNILILHH3OR -s 10.244.2.4/32 -m comment --comment "default/nginx:" -j KUBE-MARK-MASQ
-A KUBE-SEP-TOYRWPNILILHH3OR -p tcp -m comment --comment "default/nginx:" -m tcp -j DNAT --to-destination 10.244.2.4:80

# endpoint #3
-A KUBE-SEP-UXHBWR5XIMVGXW3H -s 10.244.1.2/32 -m comment --comment "default/nginx:" -j KUBE-MARK-MASQ
-A KUBE-SEP-UXHBWR5XIMVGXW3H -p tcp -m comment --comment "default/nginx:" -m tcp -j DNAT --to-destination 10.244.1.2:80
```

如果服务设置了 `externalTrafficPolicy: Local` 并且当前 Node 上面没有任何属于该服务的 Pod，那么在 KUBE-XLB-4N57TFCL4MD7ZTDA 中会直接丢掉从公网 IP 请求的包：

```bash
-A KUBE-XLB-4N57TFCL4MD7ZTDA -m comment --comment "default/nginx: has no local endpoints" -j KUBE-MARK-DROP
```
iptables mode因为使用iptable NAT来完成转发，也存在不可忽视的性能损耗。另外，如果集群中存在上万的Service/Endpoint，那么Node上的iptables rules将会非常庞大，性能还会再打折扣。这也导致目前大部分企业用k8s上生产时，都不会直接用kube-proxy作为服务代理，而是通过自己开发或者通过Ingress Controller来集成HAProxy, Nginx来代替kube-proxy。

iptables 模式与 userspace 相同，kube-proxy 持续监听 Service 以及 Endpoints 对象的变化；但它并不在本地节点开启反向代理服务，而是把反向代理全部交给 iptables 来实现；即 `iptables` 直接将对 `VIP` 的请求转发给后端 Pod，通过 iptables 设置转发策略。其工作流程大体如下:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401211721205.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

由此分析: 该模式相比 userspace 模式，克服了请求在用户态-内核态反复传递的问题，性能上有所提升，但使用 iptables NAT 来完成转发，存在不可忽视的性能损耗，而且在大规模场景下，iptables 规则的条目会十分巨大，性能上还要再打折扣。

iptables的方式则是利用了linux的iptables的nat转发进行实现:

```bash
apiVersion: v1
kind: Service
metadata:
  labels:
    name: mysql
    role: service
  name: mysql-service
spec:
  ports:
    - port: 3306
      targetPort: 3306
      nodePort: 30964
  type: NodePort
  selector:
    mysql-service: "true"
```
mysql-service对应的nodePort暴露出来的端口为`30964`，对应的`cluster IP(10.254.162.44)`的端口为3306，进一步对应于后端的pod的端口为3306。 mysql-service后端代理了两个pod，ip分别是192.168.125.129和192.168.125.131, 这里先来看一下iptables:

```bash
$iptables -S -t nat
...
-A PREROUTING -m comment --comment "kubernetes service portals" -j KUBE-SERVICES
-A OUTPUT -m comment --comment "kubernetes service portals" -j KUBE-SERVICES
-A POSTROUTING -m comment --comment "kubernetes postrouting rules" -j KUBE-POSTROUTING
-A KUBE-MARK-MASQ -j MARK --set-xmark 0x4000/0x4000
-A KUBE-NODEPORTS -p tcp -m comment --comment "default/mysql-service:" -m tcp --dport 30964 -j KUBE-MARK-MASQ
-A KUBE-NODEPORTS -p tcp -m comment --comment "default/mysql-service:" -m tcp --dport 30964 -j KUBE-SVC-67RL4FN6JRUPOJYM
-A KUBE-SEP-ID6YWIT3F6WNZ47P -s 192.168.125.129/32 -m comment --comment "default/mysql-service:" -j KUBE-MARK-MASQ
-A KUBE-SEP-ID6YWIT3F6WNZ47P -p tcp -m comment --comment "default/mysql-service:" -m tcp -j DNAT --to-destination 192.168.125.129:3306
-A KUBE-SEP-IN2YML2VIFH5RO2T -s 192.168.125.131/32 -m comment --comment "default/mysql-service:" -j KUBE-MARK-MASQ
-A KUBE-SEP-IN2YML2VIFH5RO2T -p tcp -m comment --comment "default/mysql-service:" -m tcp -j DNAT --to-destination 192.168.125.131:3306
-A KUBE-SERVICES -d 10.254.162.44/32 -p tcp -m comment --comment "default/mysql-service: cluster IP" -m tcp --dport 3306 -j KUBE-SVC-67RL4FN6JRUPOJYM
-A KUBE-SERVICES -m comment --comment "kubernetes service nodeports; NOTE: this must be the last rule in this chain" -m addrtype --dst-type LOCAL -j KUBE-NODEPORTS
-A KUBE-SVC-67RL4FN6JRUPOJYM -m comment --comment "default/mysql-service:" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-ID6YWIT3F6WNZ47P
-A KUBE-SVC-67RL4FN6JRUPOJYM -m comment --comment "default/mysql-service:" -j KUBE-SEP-IN2YML2VIFH5RO2T
```




### 7.3 ipvs mode
**ipvs的模型中有两个角色**：

 - 调度器:Director，又称为Balancer。 调度器主要用于接受用户请求。
 - 真实主机:Real Server，简称为RS。用于真正处理用户的请求。

kubernetes从1.8开始增加了IPVS支持，IPVS相对于iptables来说效率会更加高，使用ipvs模式需要在允许proxy的节点上安装ipvsadm，ipset工具包加载ipvs的内核模块。并且ipvs可以轻松处理每秒 10 万次以上的转发请求。

当proxy启动的时候，proxy将验证节点上是否安装了ipvs模块。如果未安装的话将回退到iptables模式。

并在Kubernetes 1.12成为kube-proxy的默认代理模式。ipvs模式也是基于netfilter，对比iptables模式在大规模Kubernetes集群有更好的扩展性和性能，支持更加复杂的负载均衡算法(如：最小负载、最少连接、加权等)，支持Server的健康检查和连接重试等功能。ipvs依赖于iptables，使用iptables进行包过滤、SNAT、masquared。ipvs将使用ipset需要被DROP或MASQUARED的源地址或目标地址，这样就能保证iptables规则数量的固定，我们不需要关心集群中有多少个Service了。

[Kube-proxy IPVS mode](https://github.com/kubernetes/kubernetes/blob/master/pkg/proxy/ipvs/README.md) 列出了各种服务在 IPVS 模式下的工作原理

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401113020267.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
$ ipvsadm -ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.0.0.1:443 rr persistent 10800
  -> 192.168.0.1:6443             Masq    1      1          0
TCP  10.0.0.10:53 rr
  -> 172.17.0.2:53                Masq    1      0          0
UDP  10.0.0.10:53 rr
  -> 172.17.0.2:53                Masq    1      0          0
```
注意，IPVS 模式也会使用 iptables 来执行 SNAT 和 IP 伪装（MASQUERADE），并使用 ipset 来简化 iptables 规则的管理：
| ipset 名                        | 成员                                                              | 用途                                                                       |
|--------------------------------|-----------------------------------------------------------------|--------------------------------------------------------------------------|
| KUBE-CLUSTER-IP                | All service IP + port                                           | Mark-Masq for cases that masquerade-all=true or clusterCIDR specified    |
| KUBE-LOOP-BACK                 | All service IP + port + IP                                      | masquerade for solving hairpin purpose                                   |
| KUBE-EXTERNAL-IP               | service external IP + port                                      | masquerade for packages to external IPs                                  |
| KUBE-LOAD-BALANCER             | load balancer ingress IP + port                                 | masquerade for packages to load balancer type service                    |
| KUBE-LOAD-BALANCER-LOCAL       | LB ingress IP + port with externalTrafficPolicy=local           | accept packages to load balancer with externalTrafficPolicy=local        |
| KUBE-LOAD-BALANCER-FW          | load balancer ingress IP + port with loadBalancerSourceRanges   | package filter for load balancer with loadBalancerSourceRanges specified |
| KUBE-LOAD-BALANCER-SOURCE-CIDR | load balancer ingress IP + port + source CIDR                   | package filter for load balancer with loadBalancerSourceRanges specified |
| KUBE-NODE-PORT-TCP             | nodeport type service TCP port                                  | masquerade for packets to nodePort(TCP)                                  |
| KUBE-NODE-PORT-LOCAL-TCP       | nodeport type service TCP port with externalTrafficPolicy=local | accept packages to nodeport service with externalTrafficPolicy=local     |
| KUBE-NODE-PORT-UDP             | nodeport type service UDP port                                  | masquerade for packets to nodePort(UDP)                                  |
| KUBE-NODE-PORT-LOCAL-UDP       | nodeport type service UDP port withexternalTrafficPolicy=local  | accept packages to nodeport service withexternalTrafficPolicy=local      |

这种模式，Kube-Proxy 会监视 Kubernetes Service 对象 和 Endpoints，调用 Netlink 接口以相应地创建 IPVS 规则并定期与 Kubernetes Service 对象 和 Endpoints 对象同步 IPVS 规则，以确保 IPVS 状态与期望一致。访问服务时，流量将被重定向到其中一个后端 Pod。

以下情况下IPVS会使用iptables

IPVS proxier将使用iptables，在数据包过滤，SNAT和支持NodePort类型的服务这几个场景中。具体来说，ipvs proxier将在以下4个场景中回归iptables。

kube-proxy 启动项设置了 –masquerade-all=true

如果kube-proxy以`--masquerade-all = true`开头，则ipvs proxier将伪装访问服务群集IP的所有流量，其行为与iptables proxier相同



注：master节点上也需要进行kubelet配置。因为ipvs在有些情况下是依赖iptables的，iptables中KUBE-POSTROUTING，KUBE-MARK-MASQ， KUBE-MARK-DROP这三条链是被 kubelet创建和维护的， ipvs不会创建它们。

在kube-proxy启动中指定集群CIDR

如果kube-proxy以--cluster-cidr = <cidr>开头，则ipvs proxier将伪装访问服务群集IP的群集外流量，其行为与iptables proxier相同。

为LB类型服务指定Load Balancer Source Ranges

当服务的LoadBalancerStatus.ingress.IP不为空并且指定了服务的LoadBalancerSourceRanges时，ipvs proxier将安装iptables。

支持 NodePort type service

为了支持NodePort类型的服务，ipvs将在iptables proxier中继续现有的实现。

kubernetes中ipvs实现原理图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210402173302972.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
为什么每个svc会在ipvs网卡增加vip地址：

由于 IPVS 的 DNAT 钩子挂在 INPUT 链上，因此必须要让内核识别 VIP 是本机的 IP。这样才会过INPUT 链，要不然就通过OUTPUT链出去了。k8s 通过设置将service cluster ip 绑定到虚拟网卡kube-ipvs0。

①因为service cluster ip 绑定到虚拟网卡kube-ipvs0上，内核可以识别访问的 VIP 是本机的 IP.

②数据包到达INPUT链.

③ipvs监听到达input链的数据包，比对数据包请求的服务是为集群服务，修改数据包的目标IP地址为对应pod的IP，然后将数据包发至POSTROUTING链.

④数据包经过POSTROUTING链选路，将数据包通过flannel网卡发送出去。从flannel虚拟网卡获得源IP.

⑤pod接收到请求之后，构建响应报文，改变源地址和目的地址，返回给客户端。



模拟ipvs模式：

新增DROP 链路 将svc地址即ipvs的vip地址为10.222.251.98的包丢弃

在指定的node节点执行

```bash
iptables -t filter -I INPUT -d 10.222.251.98 -j DROP
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210402173703214.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

当该vip地址进入到这个input的链路的时候直接DROP 如下：

查看链路是否有包

```bash
watch -n 0.1 "iptables --line-number -nvxL INPUT"
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210402173738660.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
因为当我们访问10.222.251.98这个地址的时候，ipvs会感知到这个ip是本机的vip地址，所以会线进入input的链路。所以我们在input增加了desc为10.222.251.98的DROP链路的话，会有限

之后恢复的话直接删除掉指定规则

```bash
iptables -D INPUT 1
```

ps： 1代表iptables -nL --line-number输出的行号

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210402173802286.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)






## 8. 启动 kube-proxy 示例

```bash
kube-proxy --kubeconfig=/var/lib/kubelet/kubeconfig --cluster-cidr=10.240.0.0/12 --feature-gates=ExperimentalCriticalPodAnnotation=true --proxy-mode=iptables
```
或者

```bash
KUBE_PROXY_ARGS="--bind-address=0.0.0.0 \
  --hostname-override=node147 \
  --kubeconfig=/etc/kubernetes/kube-proxy.conf \
  --logtostderr=true \
  --v=2 \
  --feature-gates=SupportIPVSProxyMode=true \
  --proxy-mode=ipvs"
```
如果kubelet设置了`–hostname-override`选项，则kube-proxy也需要设置该选项，并且名字一致否则会出现找不到Node的情况。


参考：

 - [kube-proxy ipvs模式详解](https://blog.csdn.net/qq_36183935/article/details/90734936)
 - [kubernetes kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)
 - [K8s: A Closer Look at Kube-Proxy](https://betterprogramming.pub/k8s-a-closer-look-at-kube-proxy-372c4e8b090)
 - [Managing the kube-proxy add-on](https://docs.aws.amazon.com/eks/latest/userguide/managing-kube-proxy.html)
 - [How to monitor kube-proxy](https://sysdig.com/blog/monitor-kube-proxy/)
