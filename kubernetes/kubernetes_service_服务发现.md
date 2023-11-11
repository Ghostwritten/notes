#  kubernetes service 服务发现




[
![在这里插入图片描述](https://img-blog.csdnimg.cn/87cdd74f5659411e99ee8f9313c49ad3.jpeg#pic_center)](https://www.rottentomatoes.com/m/arrival_2016)

*电影《降临》根据小说《你一生的故事》改编*

##  1. 介绍

[Kubernetes Service](https://kubernetes.io/docs/concepts/services-networking/service/)定义了这样一种抽象，Kubernetes 之所以需要 Service，一方面是因为 Pod 的 IP 不是固定的，另一方面则是因为一组 Pod 实例之间总会有负载均衡的需求。 Pod 能够被 Service 访问到，通常是通过 `Label Selector`实现的。

service 解决两个问题：
- Pod 之间如何找到对方？
- Pod 在重启、更新、销毁的过程中，如何确保 Pod 之间的调用不受影响？

pod 的 IP 是不稳定的，我们不能把 Pod IP 用作服务之间的调用地址，需要通过 DNS解析实现。
![在这里插入图片描述](https://img-blog.csdnimg.cn/80a522b6d8c34ab8951f27dbbf957dfb.png)


对 Kubernetes 集群中的应用，Kubernetes 提供了简单的 Endpoints API，只要 Service 中的一组 Pod 发生变更，应用程序就会被更新。 对非 Kubernetes 集群中的应用，Kubernetes 提供了基于 VIP 的网桥的方式访问 Service，再由 Service 重定向到 backend Pod。

所谓 Service 的访问入口，其实就是每台宿主机上由 kube-proxy 生成的 iptables 规则，以及 kube-dns 生成的 DNS 记录。

## 2. 定义
一个 Service 在 Kubernetes 中是一个 `REST 对象`，和 Pod 类似。 像所有的 REST 对象一样， Service 定义可以基于 POST 方式，请求 apiserver 创建新的实例。 例如，假定有一组 Pod，它们对外暴露了 9376 端口，同时还被打上 `"app=MyApp"` 标签。

```bash
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

上述配置将创建一个名称为 “`my-service`” 的 Service 对象，这个 Service 的 80 端口，代理的是 Pod 的 9376 端口，并且具有标签 "`app=MyApp`" 的 Pod 上。 这个 Service 将被指派一个 IP 地址（通常称为 “Cluster IP”），它会被服务的代理使用（见下面）。 该 Service 的 selector 将会持续评估，处理结果将被 POST 到一个名称为 “my-service” 的 Endpoints 对象上。

Service 能够将一个接收端口映射到任意的 `targetPort`。 默认情况下，targetPort 将被设置为与 port 字段相同的值。 可能更有趣的是，targetPort 可以是一个字符串，引用了 `backend Pod` 的一个端口的名称。 但是，实际指派给该端口名称的端口号，在每个 backend Pod 中可能并不相同。 对于部署和设计 Service ，这种方式会提供更大的灵活性。 例如，可以在 backend 软件下一个版本中，修改 Pod 暴露的端口，并不会中断客户端的调用。

Kubernetes Service 能够支持 TCP 和 UDP 协议，默认 TCP 协议。

很多 Service 需要暴露多个端口。对于这种情况，Kubernetes 支持在 Service 对象中定义多个端口。 当使用多个端口时，必须给出所有的端口的名称，这样 Endpoint 就不会产生歧义，例如：

```bash
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
    selector:
      app: MyApp
    ports:
      - name: http
        protocol: TCP
        port: 80
        targetPort: 9376
      - name: https
        protocol: TCP
        port: 443
        targetPort: 9377

```

## 3. 没有 selector 的 Service
Servcie 抽象了该如何访问 Kubernetes Pod，但也能够抽象其它类型的 backend，例如：

 - 希望在生产环境中使用外部的数据库集群，但测试环境使用自己的数据库。
 - 希望服务指向另一个 Namespace 中或其它集群中的服务。
 - 正在将工作负载转移到 Kubernetes 集群，和运行在 Kubernetes 集群之外的 backend。
在任何这些场景中，都能够定义没有 selector 的 Service ：

```bash
kind: Service
apiVersion: v1
metadata:
  name: my-service  #名字匹配
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```
由于这个 Service 没有 selector，就不会创建相关的 Endpoints 对象。可以手动将 Service 映射到指定的 Endpoints：

```bash
kind: Endpoints
apiVersion: v1
metadata:
  name: my-service #名字匹配
subsets:
  - addresses:
      - ip: 1.2.3.4
    ports:
      - port: 9376
```
**注意**：Endpoint IP 地址不能是 loopback（127.0.0.0/8）、 link-local（169.254.0.0/16）、或者 link-local 多播（224.0.0.0/24）。






##  4. 应用 service

deployment 应用来自 [lyzhang1999/kubernetes-example](https://github.com/lyzhang1999/kubernetes-example/tree/main/deploy)
`backend.yaml` 包含 `deployment` 与 `service`

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  labels:
    app: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: flask-backend
        image: lyzhang1999/backend:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 5000
        env:
        - name: DATABASE_URI
          value: pg-service
        - name: DATABASE_USERNAME
          value: postgres
        - name: DATABASE_PASSWORD
          value: postgres
        resources:
          requests:
            memory: "128Mi"
            cpu: "128m"
          limits:
            memory: "256Mi"
            cpu: "256m"
        readinessProbe: 
          httpGet:
            path: /healthy
            port: 5000
            scheme: HTTP
          initialDelaySeconds: 10
          failureThreshold: 5
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        livenessProbe: 
          httpGet:
            path: /healthy
            port: 5000
            scheme: HTTP
          failureThreshold: 5
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        startupProbe: 
          httpGet:
            path: /healthy
            port: 5000
            scheme: HTTP
          initialDelaySeconds: 10
          failureThreshold: 5
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  labels:
    app: backend
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
  - port: 5000
    targetPort: 5000
```

maifest 字段：
- `selector` 字段，这是一个 Pod 选择器，这个字段表示通过 Label 匹配 Pod，也就意味着，只要是 Label 包含 app=backend 的 Pod ，都会被当成是 backend-service 的同一组逻辑 Pod。
- `sessionAffinity` 字段代表的含义是会话保持，如果设置为 True，那么 Service 在转发请求时不再使用负载均衡方式，而是会通过客户端 IP 会话亲和性的方式来将请求转发到之前访问的 Pod 上，通过这种方式来更好地适配一些要求保持会话的应用。
- `port` 字段代表 Service 监听端口
- `targetPort` 字段代表将请求转发到 Pod 时的目标端口

### 4.1 Endpoint 对象
创建 Service 之后，K8s 会自动帮助我们创建 Endpoint

```bash
$ kubectl get endpoints -n example
NAME               ENDPOINTS                           AGE
backend-service    10.244.0.13:5000,10.244.0.20:5000   12h
frontend-service   10.244.0.16:3000,10.244.0.9:3000    12h
pg-service         10.244.0.8:5432                     12h
```
从返回结果我们可以发现，`backend-service Endpoints` 记录的 IP 正好是 `backend Pod` 的 IP 地址，Endpoint 记录了 Pod 对象以及 IP 地址，下面是 `backend-service Endpoint` 的 Manifest：

```bash
apiVersion: v1
kind: Endpoints
metadata:
  name: backend-service
  namespace: example
  ......
subsets:
  - addresses:
      - ip: 10.244.0.20
        targetRef:
          kind: Pod
          namespace: example
          name: backend-595666f99c-pdxbk
          ......
      - ip: 10.244.0.13
        targetRef:
          kind: Pod
          namespace: example
          name: backend-66b9754d65-jxpnb
          ......
    ports:
      - port: 5000
        protocol: TCP
```
### 4.2 Service IP
 Service IP 是稳定的，并且它能为我们抽象一组 Pod 实现负载均衡。这就意味着我们只需要访问 Service IP 就可以找到对应的 Pod。
 获取示例应用的后端 Service IP 地址：
 

```bash
$ kubectl get service -n example
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
backend-service    ClusterIP   10.96.151.12    <none>        5000/TCP   12h
frontend-service   ClusterIP   10.96.173.32    <none>        3000/TCP   12h
pg-service         ClusterIP   10.96.191.239   <none>        5432/TCP   12h
```
从返回结果我们可以得知，backend-service 的 IP 地址是：`10.96.151.12`，接下来我们尝试在前端 Pod 内部访问这个 IP。首先，进入示例应用的前端 Pod 容器终端：


```bash
$ kubectl exec -it $(kubectl get pods --selector=app=frontend -n example -o jsonpath="{.items[0].metadata.name}") -n example -- sh
/frontend #


/frontend # while true; do wget -q -O- http://10.96.151.12:5000/host_name && sleep 1; done
{"host_name":"backend-595666f99c-pdxbk"}
{"host_name":"backend-595666f99c-jxpnb"}
{"host_name":"backend-595666f99c-pdxbk"}
{"host_name":"backend-595666f99c-jxpnb"}
```
上面的代码会以 1 秒钟 1 次的频率请求后端 Pod /host_name 接口，并打印后端接口的返回内容。

### 4.3 Service 域名
我们在写业务代码的时候是不知道被调用的 Service IP 的地址的，我们只有将 Service 部署到集群才能知道它。 此外，如果删除了 Service 重建，IP 地址也将会出现变化。最后，当我们将 K8s 对象从 A 集群迁移到 B 集群时，Service IP 也会产生变化。这些问题会让 Service IP 变得不可预测，最终使得我们在编码阶段没办法使用 Service。这时候，我们就需要一个跟 IP 无直接关系的访问方式，那就是 Service 域名。

Service 在 K8s 集群内有自己独立的域名，完整的格式是：`{$service_name}.{$namespace}.svc.cluster.local`。在示例应用的例子中，`backend-service` 完整的域名是：`backend-service.example.svc.cluster.local`。接下来我们在前端 Pod 容器终端里验证这个猜想。

```bash
/frontend #while true; do wget -q -O- http://backend-service.example.svc.cluster.local:5000/host_name && sleep 1; done
{"host_name":"backend-595666f99c-jxpnb"}
{"host_name":"backend-595666f99c-pdxbk"}
{"host_name":"backend-595666f99c-pdxbk"}
{"host_name":"backend-595666f99c-jxpnb"}
```
实际上，当请求发起方和目标 Service 在同一个命名空间下时，我们可以省略 namesapce.svc.cluster.local，也就是说，只需要请求 Service 的全称即 `backend-service` 就可以了，你可以在 frontend Pod 里面继续验证：

```bash
/frontend #while true; do wget -q -O- http://backend-service:5000/host_name && sleep 1; done
{"host_name":"backend-595666f99c-jxpnb"}
{"host_name":"backend-595666f99c-pdxbk"}
{"host_name":"backend-595666f99c-pdxbk"}
{"host_name":"backend-595666f99c-jxpnb"}
```



##  5. 原理
Kubernetes 里的 Service 究竟是如何工作的呢？

**实际上，Service 是由 kube-proxy 组件，加上 iptables 来共同实现的。**

###  5.1 kube-proxy 和 iptables

举个例子，对于我们前面创建的名叫 hostnames 的 Service 来说，一旦它被提交给 Kubernetes，那么 kube-proxy 就可以通过 Service 的 `Informer` 感知到这样一个 Service 对象的添加。而作为对这个事件的响应，它就会在宿主机上创建这样一条 iptables 规则（你可以通过 iptables-save 看到它），如下所示：


```bash
-A KUBE-SERVICES -d 10.0.1.175/32 -p tcp -m comment --comment "default/hostnames: cluster IP" -m tcp --dport 80 -j KUBE-SVC-NWV5X2332I4OT4T3
```
可以看到，这条 iptables 规则的含义是：凡是目的地址是 10.0.1.175、目的端口是 80 的 IP 包，都应该跳转到另外一条名叫 `KUBE-SVC-NWV5X2332I4OT4T3` 的 iptables 链进行处理。

而我们前面已经看到，10.0.1.175 正是这个 Service 的 VIP。所以这一条规则，就为这个 Service 设置了一个固定的入口地址。并且，由于 10.0.1.175 只是一条 iptables 规则上的配置，并没有真正的网络设备，所以你 ping 这个地址，是不会有任何响应的。


那么，我们即将跳转到的 `KUBE-SVC-NWV5X2332I4OT4T3` 规则，又有什么作用呢？实际上，它是一组规则的集合，如下所示：


```bash
-A KUBE-SVC-NWV5X2332I4OT4T3 -m comment --comment "default/hostnames:" -m statistic --mode random --probability 0.33332999982 -j KUBE-SEP-WNBA2IHDGP2BOBGZ
-A KUBE-SVC-NWV5X2332I4OT4T3 -m comment --comment "default/hostnames:" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-X3P2623AGDH6CDF3
-A KUBE-SVC-NWV5X2332I4OT4T3 -m comment --comment "default/hostnames:" -j KUBE-SEP-57KPRZ3JQVENLNBR
```
可以看到，这一组规则，实际上是一组随机模式（`–mode random`）的 iptables 链。

而随机转发的目的地，分别是 `KUBE-SEP-WNBA2IHDGP2BOBGZ`、`KUBE-SEP-X3P2623AGDH6CDF3` 和 `KUBE-SEP-57KPRZ3JQVENLNBR`。

而这三条链指向的最终目的地，其实就是这个 Service 代理的三个 Pod。所以这一组规则，就是 Service 实现负载均衡的位置。需要注意的是，iptables 规则的匹配是从上到下逐条进行的，所以为了保证上述三条规则每条被选中的概率都相同，我们应该将它们的 `probability` 字段的值分别设置为 1/3（0.333…）、1/2 和 1。

这么设置的原理很简单：**第一条规则被选中的概率就是 1/3；而如果第一条规则没有被选中，那么这时候就只剩下两条规则了，所以第二条规则的 probability 就必须设置为 1/2；类似地，最后一条就必须设置为 1。**

你可以想一下，如果把这三条规则的 probability 字段的值都设置成 1/3，最终每条规则被选中的概率会变成多少。通过查看上述三条链的明细，我们就很容易理解 Service 进行转发的具体原理了，如下所示：


```bash
-A KUBE-SEP-57KPRZ3JQVENLNBR -s 10.244.3.6/32 -m comment --comment "default/hostnames:" -j MARK --set-xmark 0x00004000/0x00004000
-A KUBE-SEP-57KPRZ3JQVENLNBR -p tcp -m comment --comment "default/hostnames:" -m tcp -j DNAT --to-destination 10.244.3.6:9376

-A KUBE-SEP-WNBA2IHDGP2BOBGZ -s 10.244.1.7/32 -m comment --comment "default/hostnames:" -j MARK --set-xmark 0x00004000/0x00004000
-A KUBE-SEP-WNBA2IHDGP2BOBGZ -p tcp -m comment --comment "default/hostnames:" -m tcp -j DNAT --to-destination 10.244.1.7:9376

-A KUBE-SEP-X3P2623AGDH6CDF3 -s 10.244.2.3/32 -m comment --comment "default/hostnames:" -j MARK --set-xmark 0x00004000/0x00004000
-A KUBE-SEP-X3P2623AGDH6CDF3 -p tcp -m comment --comment "default/hostnames:" -m tcp -j DNAT --to-destination 10.244.2.3:9376
```

可以看到，这三条链，其实是三条 DNAT 规则。但在 DNAT 规则之前，iptables 对流入的 IP 包还设置了一个“标志”（`–set-xmark`）。

而 DNAT 规则的作用，就是在 `PREROUTING` 检查点之前，也就是在路由之前，将流入 IP 包的目的地址和端口，改成`–to-destination` 所指定的新的目的地址和端口。可以看到，这个目的地址和端口，正是被代理 Pod 的 IP 地址和端口。

这样，访问 `Service VIP` 的 IP 包经过上述 iptables 处理之后，就已经变成了访问具体某一个后端 Pod 的 IP 包了。不难理解，这些 Endpoints 对应的 iptables 规则，正是 kube-proxy 通过监听 Pod 的变化事件，在宿主机上生成并维护的。

以上，就是 Service 最基本的工作原理。

###  5.2 kube-proxy 和ipvs
Kubernetes 的 `kube-proxy` 还支持一种叫作 IPVS 的模式。这又是怎么一回事儿呢？

其实，通过上面的讲解，你可以看到，kube-proxy 通过 iptables 处理 Service 的过程，其实需要在宿主机上设置相当多的 iptables 规则。而且，kube-proxy 还需要在控制循环里不断地刷新这些规则来确保它们始终是正确的。

难想到，当你的宿主机上有大量 Pod 的时候，成百上千条 iptables 规则不断地被刷新，会大量占用该宿主机的 CPU 资源，甚至会让宿主机“卡”在这个过程中。所以说，**一直以来，基于 iptables 的 Service 实现，都是制约 Kubernetes 项目承载更多量级的 Pod 的主要障碍**。

而 IPVS 模式的 Service，就是解决这个问题的一个行之有效的方法。

IPVS 模式的工作原理，其实跟 iptables 模式类似。当我们创建了前面的 Service 之后，kube-proxy 首先会在宿主机上创建一个虚拟网卡（叫作：`kube-ipvs0`），并为它分配 `Service VIP` 作为 IP 地址，如下所示：


```bash
# ip addr
  ...
  73：kube-ipvs0：<BROADCAST,NOARP>  mtu 1500 qdisc noop state DOWN qlen 1000
  link/ether  1a:ce:f5:5f:c1:4d brd ff:ff:ff:ff:ff:ff
  inet 10.0.1.175/32  scope global kube-ipvs0
  valid_lft forever  preferred_lft forever
```

而接下来，`kube-proxy` 就会通过 Linux 的 IPVS 模块，为这个 IP 地址设置三个 IPVS 虚拟主机，并设置这三个虚拟主机之间使用轮询模式 (rr) 来作为负载均衡策略。我们可以通过 `ipvsadm` 查看到这个设置，如下所示：


```bash
# ipvsadm -ln
 IP Virtual Server version 1.2.1 (size=4096)
  Prot LocalAddress:Port Scheduler Flags
    ->  RemoteAddress:Port           Forward  Weight ActiveConn InActConn     
  TCP  10.102.128.4:80 rr
    ->  10.244.3.6:9376    Masq    1       0          0         
    ->  10.244.1.7:9376    Masq    1       0          0
    ->  10.244.2.3:9376    Masq    1       0          0
```

可以看到，这三个 IPVS 虚拟主机的 IP 地址和端口，对应的正是三个被代理的 Pod。

这时候，任何发往 10.102.128.4:80 的请求，就都会被 IPVS 模块转发到某一个后端 Pod 上了。

而相比于 iptables，IPVS 在内核中的实现其实也是基于 Netfilter 的 NAT 模式，所以在转发这一层上，理论上 IPVS 并没有显著的性能提升。但是，IPVS 并不需要在宿主机上为每个 Pod 设置 iptables 规则，而是把对这些“规则”的处理放到了内核态，从而极大地降低了维护这些规则的代价。这也正印证了我在前面提到过的，“将重要操作放入内核态”是提高性能的重要手段。

不过需要注意的是，IPVS 模块只负责上述的负载均衡和代理功能。而一个完整的 Service 流程正常工作所需要的包过滤、SNAT 等操作，还是要靠 iptables 来实现。只不过，这些辅助性的 iptables 规则数量有限，也不会随着 Pod 数量的增加而增加。

所以，**在大规模集群里，我非常建议你为 `kube-proxy` 设置`–proxy-mode=ipvs` 来开启这个功能。它为 Kubernetes 集群规模带来的提升，还是非常巨大的**。

###  5.3 service与DNS关系
在 Kubernetes 中，Service 和 Pod 都会被分配对应的 DNS A 记录（从域名解析 IP 的记录）。对于 ClusterIP 模式的 `Service` 来说（比如我们上面的例子），它的 A 记录的格式是：`..svc.cluster.local`。当你访问这条 A 记录的时候，它解析到的就是该 Service 的 VIP 地址。

而对于指定了 `clusterIP=None` 的 `Headless Service` 来说，它的 A 记录的格式也是：`..svc.cluster.local`。但是，当你访问这条 A 记录的时候，它返回的是所有被代理的 Pod 的 IP 地址的集合。当然，如果你的客户端没办法解析这个集合的话，它可能会只会拿到第一个 Pod 的 IP 地址。

此外，对于 ClusterIP 模式的 Service 来说，它代理的 Pod 被自动分配的 A 记录的格式是：`..pod.cluster.local`。这条记录指向 Pod 的 IP 地址。

而对 Headless Service 来说，它代理的 Pod 被自动分配的 A 记录的格式是：`...svc.cluster.local`。这条记录也指向 Pod 的 IP 地址。

但如果你为 Pod 指定了 Headless Service，并且 Pod 本身声明了 `hostname` 和 `subdomain` 字段，那么这时候 Pod 的 A 记录就会变成：`<pod 的 hostname>...svc.cluster.local`，比如：



```bash
apiVersion: v1
kind: Service
metadata:
  name: default-subdomain
spec:
  selector:
    name: busybox
  clusterIP: None
  ports:
  - name: foo
    port: 1234
    targetPort: 1234
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox1
  labels:
    name: busybox
spec:
  hostname: busybox-1
  subdomain: default-subdomain
  containers:
  - image: busybox
    command:
      - sleep
      - "3600"
    name: busybox
```

在上面这个 Service 和 Pod 被创建之后，你就可以通过 `busybox-1.default-subdomain.default.svc.cluster.local` 解析到这个 Pod 的 IP 地址了。

需要注意的是，在 Kubernetes 里，`/etc/hosts` 文件是单独挂载的，这也是为什么 kubelet 能够对 `hostname` 进行修改并且 Pod 重建后依然有效的原因。这跟 Docker 的 Init 层是一个原理。


一个可选（尽管强烈推荐）集群插件 是 DNS 服务器。 DNS 服务器监视着创建新 Service 的 Kubernetes API，从而为每一个 Service 创建一组 DNS 记录。 如果整个集群的 DNS 一直被启用，那么所有的 Pod 应该能够自动对 Service 进行名称解析。

例如，有一个名称为 "my-service" 的 Service，它在 Kubernetes 集群中名为 "my-ns" 的 Namespace 中，为 "`my-service.my-ns`" 创建了一条 DNS 记录。 在名称为 "my-ns" 的 Namespace 中的 Pod 应该能够简单地通过名称查询找到 "my-service"。 在另一个 Namespace 中的 Pod 必须限定名称为 "my-service.my-ns"。 这些名称查询的结果是 Cluster IP。

Kubernetes 也支持对端口名称的 `DNS SRV`（Service）记录。 如果名称为 "my-service.my-ns" 的 Service 有一个名为 "http" 的 TCP 端口，可以对 "_http._tcp.my-service.my-ns" 执行 DNS SRV 查询，得到 "http" 的端口号。

Kubernetes DNS 服务器是唯一的一种能够访问 ExternalName 类型的 Service 的方式。 更多信息可以查看DNS Pod 和 Service。

总结：

 - 一种是通过`<serviceName>.<namespace>.svc.cluster.local`访问。对应于`clusterIP`
 - 另一种是通过`<podName>.<serviceName>.<namesapce>.svc.cluster.local`访问,对应于`headless
   service`.


```bash
/ # nslookup *.default.svc.cluster.local
Server: 10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name: *.default.svc.cluster.local
Address 1: 10.244.1.7 busybox-3.default-subdomain.default.svc.cluster.local
Address 2: 10.96.0.1 kubernetes.default.svc.cluster.local
Address 3: 10.97.103.223 hostnames.default.svc.cluster.local
```

## 6. VIP 和 Service 代理
 Kubernetes 集群中，每个 Node 运行一个 kube-proxy 进程。`kube-proxy` 负责为 Service 实现了一种 VIP（虚拟 IP）的形式，而不是 ExternalName 的形式。 在 Kubernetes v1.0 版本，代理完全在 userspace。在 Kubernetes v1.1 版本，新增了 iptables 代理，但并不是默认的运行模式。 从 Kubernetes v1.2 起，默认就是 iptables 代理。

**在 Kubernetes v1.0 版本，Service 是 “4层”（TCP/UDP over IP）概念。 在 Kubernetes v1.1 版本，新增了 Ingress API（beta 版），用来表示 “7层”（HTTP）服务。**

### 6.1 userspace 代理模式
这种模式，kube-proxy 会监视 Kubernetes master 对 Service 对象和 Endpoints 对象的添加和移除。 对每个 Service，它会在本地 Node 上打开一个端口（随机选择）。 任何连接到“代理端口”的请求，都会被代理到 Service 的backend Pods 中的某个上面（如 Endpoints 所报告的一样）。 使用哪个 backend Pod，是基于 Service 的 SessionAffinity 来确定的。 最后，它安装 iptables 规则，捕获到达该 Service 的 clusterIP（是虚拟 IP）和 Port 的请求，并重定向到代理端口，代理端口再代理请求到 backend Pod。

网络返回的结果是，任何到达 Service 的 IP:Port 的请求，都会被代理到一个合适的 backend，不需要客户端知道关于 Kubernetes、Service、或 Pod 的任何信息。

默认的策略是，通过 round-robin 算法来选择 backend Pod。 实现基于客户端 IP 的会话亲和性，可以通过设置 service.spec.sessionAffinity 的值为 "ClientIP" （默认值为 "None"）。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200813232105803.png#pic_center)
### 6.2 iptables 代理模式
这种模式，kube-proxy 会监视 Kubernetes master 对 Service 对象和 Endpoints 对象的添加和移除。 对每个 Service，它会安装 iptables 规则，从而捕获到达该 Service 的 clusterIP（虚拟 IP）和端口的请求，进而将请求重定向到 Service 的一组 backend 中的某个上面。 对于每个 Endpoints 对象，它也会安装 iptables 规则，这个规则会选择一个 backend Pod。

默认的策略是，随机选择一个 backend。 实现基于客户端 IP 的会话亲和性，可以将 service.spec.sessionAffinity 的值设置为 "ClientIP" （默认值为 "None"）。

和 userspace 代理类似，网络返回的结果是，任何到达 Service 的 IP:Port 的请求，都会被代理到一个合适的 backend，不需要客户端知道关于 Kubernetes、Service、或 Pod 的任何信息。 这应该比 userspace 代理更快、更可靠。然而，不像 userspace 代理，如果初始选择的 Pod 没有响应，iptables 代理能够自动地重试另一个 Pod，所以它需要依赖 readiness probes。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200813232314381.png#pic_center)






##  7. 对外访问的service类型
对一些应用（如 Frontend）的某些部分，可能希望通过外部（Kubernetes 集群外部）IP 地址暴露 Service。

Kubernetes ServiceTypes 允许指定一个需要的类型的 Service，默认是 ClusterIP 类型。

Type 的取值以及行为如下：

 - `ClusterIP`：通过**集群的内部 IP** 暴露服务，选择该值，服务只能够在集群内部可以访问，这也是默认的 ServiceType。
 - `NodePort`：通过每个 **Node 上的 IP** 和静态端口（NodePort）暴露服务。NodePort 服务会路由到 ClusterIP 服务，这个 ClusterIP 服务会自动创建。通过请求  `<NodeIP>:<NodePort>`，可以从集群的外部访问一个 NodePort 服务。
 - `LoadBalancer`：使用云提供商的负载局衡器，可以向外部暴露服务。外部的负载均衡器可以路由到 NodePort 服务和ClusterIP 服务。
 - `ExternalName`：通过返回 `CNAME` 和它的值，可以将服务映射到 externalName 字段的内容（例如， foo.bar.example.com）。 没有任何类型代理被创建，这只有 Kubernetes 1.7 或更高版本的 `kube-dns`才支持。

### 7.1 NodePort 类型
如果设置 type 的值为 "NodePort"，每个 Node 将从该端口（每个 Node 上的同一端口）代理到 Service。该端口将通过 Service 的 `spec.ports[*].nodePort` 字段被指定。

需要注意的是，Service 将能够通过 `<NodeIP>:spec.ports[*].nodePort` 和 `spec.clusterIp:spec.ports[*].port` 而对外可见。

对外访问这里最常用的一种方式就是：`NodePort`


```bash
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  type: NodePort
  ports:
  - nodePort: 8080
    targetPort: 80
    protocol: TCP
    name: http
  - nodePort: 443
    protocol: TCP
    name: https
  selector:
    run: my-nginx
```
在这个 Service 的定义里，我们声明它的类型是，type=NodePort。然后，我在 ports 字段里声明了 Service 的 8080 端口代理 Pod 的 80 端口，Service 的 443 端口代理 Pod 的 443 端口。

当然，如果你不显式地声明 nodePort 字段，Kubernetes 就会为你分配随机的可用端口来设置代理。这个端口的范围默认是 `30000-32767`，你可以通过 kube-apiserver 的`–service-node-port-range` 参数来修改它。

那么这时候，要访问这个 Service，你只需要访问：

```bash
<任何一台宿主机的IP地址>:8080
```
可以访问到某一个被代理的 Pod 的 80 端口了。而在理解了我在上一篇文章中讲解的 Service 的工作原理之后，NodePort 模式也就非常容易理解了。显然，`kube-proxy` 要做的，就是在每台宿主机上生成这样一条 iptables 规则：

```bash
-A KUBE-NODEPORTS -p tcp -m comment --comment "default/my-nginx: nodePort" -m tcp --dport 8080 -j KUBE-SVC-67RL4FN6JRUPOJYM
```

KUBE-SVC-67RL4FN6JRUPOJYM 其实就是一组随机模式的 iptables 规则。所以接下来的流程，就跟 ClusterIP 模式完全一样了。需要注意的是，在 NodePort 方式下，Kubernetes 会在 IP 包离开宿主机发往目的 Pod 时，对这个 IP 包做一次 SNAT 操作，如下所示：

```bash
-A KUBE-POSTROUTING -m comment --comment "kubernetes service traffic requiring SNAT" -m mark --mark 0x4000/0x4000 -j MASQUERADE
```

可以看到，这条规则设置在 POSTROUTING 检查点，也就是说，它给即将离开这台主机的 IP 包，进行了一次 SNAT 操作，将这个 IP 包的源地址替换成了这台宿主机上的 CNI 网桥地址，或者宿主机本身的 IP 地址（如果 CNI 网桥不存在的话）。当然，这个 SNAT 操作只需要对 Service 转发出来的 IP 包进行（否则普通的 IP 包就被影响了）。而 iptables 做这个判断的依据，就是查看该 IP 包是否有一个“0x4000”的“标志”。你应该还记得，这个标志正是在 IP 包被执行 DNAT 操作之前被打上去的。


**可是，为什么一定要对流出的包做 SNAT操作呢？**



```bash
           client
             \ ^
              \ \
               v \
   node 1 <--- node 2
    | ^   SNAT
    | |   --->
    v |
 endpoint
```

当一个外部的 client 通过 node 2 的地址访问一个 Service 的时候，node 2 上的负载均衡规则，就可能把这个 IP 包转发给一个在 node 1 上的 Pod。这里没有任何问题。

而当 node 1 上的这个 Pod 处理完请求之后，它就会按照这个 IP 包的源地址发出回复。

可是，如果没有做 SNAT 操作的话，这时候，被转发来的 IP 包的源地址就是 client 的 IP 地址。所以此时，Pod 就会直接将回复发给client。对于 client 来说，它的请求明明发给了 node 2，收到的回复却来自 node 1，这个 client 很可能会报错。


所以，在上图中，当 IP 包离开 node 2 之后，它的源 IP 地址就会被 SNAT 改成 node 2 的 CNI 网桥地址或者 node 2 自己的地址。这样，Pod 在处理完成之后就会先回复给 node 2（而不是 client），然后再由 node 2 发送给 client。

当然，这也就意味着这个 Pod 只知道该 IP 包来自于 node 2，而不是外部的 client。对于 Pod 需要明确知道所有请求来源的场景来说，这是不可以的。

所以这时候，你就可以将 Service 的 `spec.externalTrafficPolicy` 字段设置为 `local`，这就保证了所有 Pod 通过 Service 收到请求之后，一定可以看到真正的、外部 client 的源地址。

而这个机制的实现原理也非常简单：**这时候，一台宿主机上的 iptables 规则，会设置为只将 IP 包转发给运行在这台宿主机上的 Pod**。所以这时候，Pod 就可以直接使用源地址将回复包发出，不需要事先进行 SNAT 了。这个流程，如下所示：


```bash
       client
       ^ /   \
      / /     \
     / v       X
   node 1     node 2
    ^ |
    | |
    | v
 endpoint
```
当然，这也就意味着如果在一台宿主机上，没有任何一个被代理的 Pod 存在，比如上图中的 node 2，那么你使用 node 2 的 IP 地址访问这个 Service，就是无效的。此时，你的请求会直接被 DROP 掉。


### 7.2 LoadBalancer 类型
从外部访问 Service 的第二种方式，适用于公有云上的 Kubernetes 服务。这时候，你可以指定一个 `LoadBalancer` 类型的 Service，将为 Service 提供负载均衡器。 负载均衡器是异步创建的，关于被提供的负载均衡器的信息将会通过 Service 的 `status.loadBalancer` 字段被发布出去。
配置文件样板：

```c
apiVersion: v1
kind: Service
metadata:
  name: example-service
spec:
  selector:
    app: example
  ports:
    - port: 8765
      targetPort: 9376
  type: LoadBalancer
```
命令行创建：

```bash
$ kubectl expose rc example --port=8765 --target-port=9376 \
        --name=example-service --type=LoadBalancer

$ kubectl describe services example-service
 Name:                   example-service
    Namespace:              default
    Labels:                 <none>
    Annotations:            <none>
    Selector:               app=example
    Type:                   LoadBalancer
    IP:                     10.67.252.103
    LoadBalancer Ingress:   192.0.2.89
    Port:                   <unnamed> 80/TCP
    NodePort:               <unnamed> 32445/TCP
    Endpoints:              10.64.0.4:80,10.64.1.5:80,10.64.2.4:80
    Session Affinity:       None
    Events:                 <none>
```

> 如果你在 Minikube 上运行服务，你可以通过以下命令找到分配的 IP 地址和端口：`minikube service example-service --url`



demo配置文件
```bash
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
      nodePort: 30061
  clusterIP: 10.0.171.239
  loadBalancerIP: 78.11.24.19
  type: LoadBalancer
status:
  loadBalancer:
    ingress:
      - ip: 146.148.47.155
```

来自外部负载均衡器的流量将直接打到 backend Pod 上，不过实际它们是如何工作的，这要依赖于云提供商。 在这些情况下，将根据用户设置的 loadBalancerIP 来创建负载均衡器。 某些云提供商允许设置 loadBalancerIP。如果没有设置 loadBalancerIP，将会给负载均衡器指派一个临时 IP。 如果设置了 loadBalancerIP，但云提供商并不支持这种特性，那么设置的 loadBalancerIP 值将会被忽略掉。


###  7.3 ExternalName类型
第三种方式，是 Kubernetes 在 1.7 之后支持的一个新特性，叫作 `ExternalName`

`ExternalName Service` 是 Service 的特例，它没有 selector，也没有定义任何的端口和 Endpoint。 相反地，对于运行在集群外部的服务，它通过返回该外部服务的别名这种方式来提供服务。

```bash
kind: Service
apiVersion: v1
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```
当查询主机 `my-service.prod.svc.CLUSTER`时，集群的 DNS 服务将返回一个值为 `my.database.example.com` 的 CNAME 记录。 访问这个服务的工作方式与其它的相同，唯一不同的是重定向发生在 DNS 层，而且不会进行代理或转发。 如果后续决定要将数据库迁移到 Kubernetes 集群中，可以启动对应的 Pod，增加合适的 Selector 或 Endpoint，修改 Service 的 type。

如果外部的 IP 路由到集群中一个或多个 Node 上，Kubernetes Service 会被暴露给这些 externalIPs。 通过外部 IP（作为目的 IP 地址）进入到集群，打到 Service 的端口上的流量，将会被路由到 Service 的 Endpoint 上。 `externalIPs` 不会被 Kubernetes 管理，它属于集群管理员的职责范畴。

根据 Service 的规定，externalIPs 可以同任意的 ServiceType 来一起指定。 在上面的例子中，`my-service` 可以在 `80.11.12.10:80`（外部 IP:端口）上被客户端访问。

```bash
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9376
  externalIPs: 
    - 80.11.12.10
```
在上述 Service 中，我为它指定的 `externalIPs=80.11.12.10`，那么此时，你就可以通过访问 `80.11.12.10:80` 访问到被代理的 Pod 了。不过，在这里 Kubernetes 要求 externalIPs 必须是至少能够路由到一个 Kubernetes 的节点。

##  8. nodePort、port、targetPort、containerPort 区分
### 8.1 nodePort
nodePort提供了集群外部客户端访问service的一种方式，`:nodePort`提供了集群外部客户端访问service的端口，即`nodeIP:nodePort`提供了外部流量访问k8s集群中service的入口。

比如外部用户要访问k8s集群中的一个Web应用，那么我们可以配置对应service的`type=NodePort，nodePort=30001`。其他用户就可以通过浏览器`http://node:30001`访问到该web服务。

而数据库等服务可能不需要被外界访问，只需被内部服务访问即可，那么我们就不必设置service的NodePort。
##  8.2 port
port是暴露在`cluster ip`上的端口，:port提供了集群内部客户端访问service的入口，即`clusterIP:port`。

mysql容器暴露了3306端口（参考DockerFile），集群内其他容器通过33306端口访问mysql服务，但是外部流量不能访问mysql服务，因为mysql服务没有配置NodePort。对应的service.yaml如下：

```bash
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  ports:
  - port: 33306
    targetPort: 3306
  selector:
    name: mysql-pod
```
## 8.3 targetPort
targetPort是pod上的端口，从`port/nodePort`上来的数据，经过`kube-proxy`流入到后端pod的targetPort上，最后进入容器。

与制作容器时暴露的端口一致（使用DockerFile中的EXPOSE），例如官方的nginx（参考DockerFile）暴露80端口。 对应的service.yaml如下：

```bash
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort            // 配置NodePort，外部流量可访问k8s中的服务
  ports:
  - port: 30080             // 服务访问端口
    targetPort: 80          // pod控制器中定义的端口
    nodePort: 30001         // NodePort
  selector:
    name: nginx-pod
```
## 8.4 containerPort
containerPort是在pod控制器中定义的、pod中的容器需要暴露的端口。


总的来说，`port`和`nodePort`都是service的端口，前者暴露给k8s集群内部服务访问，后者暴露给k8s集群外部流量访问。从这两个端口到来的数据都需要经过反向代理`kube-proxy`，流入后端pod的`targetPort`上，最后到达pod内容器的`containerPort`

##  9. 如何排查Service 相关的问题
其实都可以通过分析 Service 在宿主机上对应的 iptables 规则（或者 IPVS 配置）得到解决。



比如，当你的 Service 没办法通过 DNS 访问到的时候。你就需要区分到底是 Service 本身的配置问题，还是集群的 DNS 出了问题。一个行之有效的方法，就是检查 Kubernetes 自己的 Master 节点的 Service DNS 是否正常：

```bash
# 在一个Pod里执行
$ nslookup kubernetes.default
Server:    10.0.0.10
Address 1: 10.0.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes.default
Address 1: 10.0.0.1 kubernetes.default.svc.cluster.local
```
如果上面访问 kubernetes.default 返回的值都有问题，那你就需要检查 kube-dns 的运行状态和日志了。否则的话，你应该去检查自己的 Service 定义是不是有问题。而如果你的 Service 没办法通过 ClusterIP 访问到的时候，你首先应该检查的是这个 Service 是否有 Endpoints：



```bash
$ kubectl get endpoints hostnames
NAME        ENDPOINTS
hostnames   10.244.0.5:9376,10.244.0.6:9376,10.244.0.7:9376
```

需要注意的是，如果你的 Pod 的 `readniessProbe` 没通过，它也不会出现在 Endpoints 列表里。而如果 Endpoints 正常，那么你就需要确认 `kube-proxy` 是否在正确运行。在我们通过 `kubeadm` 部署的集群里，你应该看到 kube-proxy 输出的日志如下所示：


```bash
I1027 22:14:53.995134    5063 server.go:200] Running in resource-only container "/kube-proxy"
I1027 22:14:53.998163    5063 server.go:247] Using iptables Proxier.
I1027 22:14:53.999055    5063 server.go:255] Tearing down userspace rules. Errors here are acceptable.
I1027 22:14:54.038140    5063 proxier.go:352] Setting endpoints for "kube-system/kube-dns:dns-tcp" to [10.244.1.3:53]
I1027 22:14:54.038164    5063 proxier.go:352] Setting endpoints for "kube-system/kube-dns:dns" to [10.244.1.3:53]
I1027 22:14:54.038209    5063 proxier.go:352] Setting endpoints for "default/kubernetes:https" to [10.240.0.2:443]
I1027 22:14:54.038238    5063 proxier.go:429] Not syncing iptables until Services and Endpoints have been received from master
I1027 22:14:54.040048    5063 proxier.go:294] Adding new service "default/kubernetes:https" at 10.0.0.1:443/TCP
I1027 22:14:54.040154    5063 proxier.go:294] Adding new service "kube-system/kube-dns:dns" at 10.0.0.10:53/UDP
I1027 22:14:54.040223    5063 proxier.go:294] Adding new service "kube-system/kube-dns:dns-tcp" at 10.0.0.10:53/TCP
```
如果 kube-proxy 一切正常，你就应该仔细查看宿主机上的 iptables 了。而一个 iptables 模式的 Service 对应的规则，它们包括：

 1. `KUBE-SERVICES` 或者 `KUBE-NODEPORTS` 规则对应的 Service 的入口链，这个规则应该与 VIP 和Service 端口一一对应；
 2. `KUBE-SEP-(hash)` 规则对应的 `DNAT` 链，这些规则应该与 `Endpoints` 一一对应；
 3. `KUBE-SVC-(hash)` 规则对应的负载均衡链，这些规则的数目应该与 `Endpoints` 数目一致；
 4. 如果是 `NodePort` 模式的话，还有 `POSTROUTING` 处的 `SNAT` 链。

通过查看这些链的数量、转发目的地址、端口、过滤条件等信息，你就能很容易发现一些异常的蛛丝马迹。

当然，还有一种典型问题，就是 Pod 没办法通过 Service 访问到自己。这往往就是因为 kubelet 的 `hairpin-mode` 没有被正确设置。你只需要确保将 kubelet 的 `hairpin-mode` 设置为 `hairpin-veth` 或者 `promiscuous-bridge` 即可。

其中，在 `hairpin-veth` 模式下，你应该能看到 CNI 网桥对应的各个 VETH 设备，都将 Hairpin 模式设置为了 1，如下所示：


```bash
$ for d in /sys/devices/virtual/net/cni0/brif/veth*/hairpin_mode; do echo "$d = $(cat $d)"; done
/sys/devices/virtual/net/cni0/brif/veth4bfbfe74/hairpin_mode = 1
/sys/devices/virtual/net/cni0/brif/vethfc2a18c5/hairpin_mode = 1
```
而如果是 `promiscuous-bridge` 模式的话，你应该看到 CNI 网桥的混杂模式（PROMISC）被开启，如下所示：

```bash
$ ifconfig cni0 |grep PROMISC
UP BROADCAST RUNNING PROMISC MULTICAST  MTU:1460  Metric:1
```

##  10. 总结
**所谓 Service，其实就是 Kubernetes 为 Pod 分配的、固定的、基于 iptables（或者 IPVS）的访问入口。而这些访问入口代理的 Pod 信息，则来自于 Etcd，由 kube-proxy 通过控制循环来维护。**

并且，你可以看到，Kubernetes 里面的 Service 和 DNS 机制，也都不具备强多租户能力。比如，在多租户情况下，每个租户应该拥有一套独立的 Service 规则（Service 只应该看到和代理同一个租户下的 Pod）。再比如 DNS，在多租户情况下，每个租户应该拥有自己的 kube-dns（kube-dns 只应该为同一个租户下的 Service 和 Pod 创建 DNS Entry）。


参考：
 - [kubernetes service](https://kubernetes.io/docs/concepts/services-networking/service/)
 - [创建外部负载均衡器](https://kubernetes.io/zh/docs/tasks/access-application-cluster/create-external-load-balancer/)


