#  kubernetes  coredns 配置使用
tags: 组件

![](https://img-blog.csdnimg.cn/d9260370ea394f71aa7c4410716dca7f.png)








## 1. 背景
DNS 是使用集群插件 管理器自动启动的内置的 Kubernetes 服务。
从 Kubernetes v1.12 开始，CoreDNS 是推荐的 DNS 服务器，取代了 kube-dns。 如果 你的集群原来使用 kube-dns，你可能部署的仍然是 kube-dns 而不是 CoreDNS。

> 说明： CoreDNS 和 kube-dns 的 Service 都在其 metadata.name 字段使用名字 kube-dns。这是为了能够与依靠传统 kube-dns 服务名称来解析集群内部地址的工作负载具有更好的互操作性。 使用 kube-dns作为服务名称可以抽离共有名称之后运行的是哪个 DNS 提供程序这一实现细节。

如果你在使用 `Deployment` 运行 CoreDNS，则该 Deployment 通常会向外暴露为一个具有 `静态 IP 地址` Kubernetes 服务。 kubelet 使用 `--cluster-dns=<DNS 服务 IP>` 标志将 DNS 解析器的信息传递给每个容器。

DNS 名称也需要域名。 你可在 kubelet 中使用 `--cluster-domain=<默认本地域名>` 标志配置本地域名。

DNS 服务器支持正向查找（A 和 AAAA 记录）、端口发现（SRV 记录）、反向 IP 地址发现（PTR 记录）等。 更多信息，[请参见Pod 和 服务的 DNS](https://kubernetes.io/zh/docs/concepts/services-networking/dns-pod-service/)。

如果 Pod 的 dnsPolicy 设置为 "default"，则它将从 Pod 运行所在节点继承名称解析配置。 Pod 的 DNS 解析行为应该与节点相同。 但请参阅[已知问题](https://kubernetes.io/zh/docs/tasks/administer-cluster/dns-debugging-resolution/#known-issues)。

如果你不想这样做，或者想要为 Pod 使用其他 DNS 配置，则可以 使用 kubelet 的 --resolv-conf 标志。 将此标志设置为 "" 可以避免 Pod 继承 DNS。 将其设置为有别于 /etc/resolv.conf 的有效文件路径可以设定 DNS 继承不同的配置。

## 2. 简介
在Kubernetes集群推荐使用Service Name作为服务的访问地址，因此需要一个Kubernetes集群范围的DNS服务实现从Service Name到Cluster Ip的解析，这就是Kubernetes基于DNS的服务发现功能。

在从`Kubernetes 1.10`开始`Dynamic Kubelet Configuration`特性进入`beta`阶段，kubelet的大多数命令行参数都改为推荐在–config指定位置的配置文件中进行配置，包括`—cluster-dns`和`–cluster-domain`两个参数，在kubelet的配置文件中配置如下：

```bash
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
......
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
......
```
其中clusterDNS指定了集群中所有容器将使用的DNS Server，即kubelet会在每个Pod内部设置DNS服务的地址为clusterDNS配置的地址。前面的配置中设置了Kubernetes集群访问内DNS服务器的地址是`10.96.0.10`，将由起完成`Service Name`到`Cluster Ip`的解析。有了这个配置，我们还需要在集群内部署DNS服务，DNS服务一般都是作为`addon`组件部署在`Kubernetes`集群内的。


## 3. Kubernetes DNS服务发展史
从`Kubernetes 1.11`开始，可使用`CoreDNS`作为Kubernetes的DNS插件进入`GA状态`，
**这意味着CoreDNS将通过安装工具包（例如kubeadm，kube-up，minikube和kops）作为Kubernetes中的标准功能提供。**
Kubernetes推荐使用CoreDNS作为集群内的DNS服务。 我们先看一下Kubernetes DNS服务的发展历程。

### 3.1 Kubernetes 1.3之前的版本 – skyDNS
`Kubernetes 1.3`之前的版本使用`skyDNS`作为DNS服务，这个有点久远了。Kubernetes的DNS服务由`kube2sky`、`skyDNS`、`etcd`组成。 `kube2sky`通过kube-apiserver监听集群中Service的变化，将生成的DNS记录信息更新到etcd中，而skyDNS将从etcd中获取数据对外提供DNS的查询服务。

### 3.2 Kubernetes 1.3版本开始 – kubeDNS
`Kubernetes 1.3`开始使用kubeDNS和`dnsmasq`替换了原来的kube2sky和skyDNS，不再使用etcd，而是将DNS记录直接存放在内存中，通过dnsmasq的缓存功能提高DNS的查询效率。下图是描述了Kubernetes使用kubeDNS实现服务发现的整体架构：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127105312765.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
### 3.3 Kubernetes 1.11版本开始 – CoreDNS进入GA
从`Kubernetes 1.11`开始，可使用CoreDNS作为Kubernetes的DNS插件进入GA状态，Kubernetes推荐使用CoreDNS作为集群内的DNS服务。 CoreDNS从2017年初就成为了CNCF的的孵化项目，CoreDNS的特点就是十分灵活和可扩展的插件机制，各种插件实现
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127113418770.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 4. 基于DNS的Kubernetes服务发现的规范
[https://github.com/kubernetes/dns/blob/master/docs/specification.md](https://github.com/kubernetes/dns/blob/master/docs/specification.md)

## 5. CoreDNS ConfigMap 选项
CoreDNS 是模块化且可插拔的 DNS 服务器，每个插件都为 CoreDNS 添加了新功能。 可以通过维护 [Corefile](https://coredns.io/2017/07/23/corefile-explained/)，即 CoreDNS 配置文件， 来定制其行为。 集群管理员可以修改 CoreDNS Corefile 的 ConfigMap，以更改服务发现的工作方式。

在 Kubernetes 中，CoreDNS 安装时使用如下默认 Corefile 配置。

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health {
            lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 30
        }
        prometheus :9153
        forward ./etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }    
```
Corefile 配置包括以下 [CoreDNS 插件](https://coredns.io/plugins/)：

 - [errors](https://coredns.io/plugins/errors/)：错误记录到标准输出。
 - [health](https://coredns.io/plugins/health/)：在 http://localhost:8080/health 处提供 CoreDNS 的健康报告。
 - [ready](https://coredns.io/plugins/ready/)：在端口 8181 上提供的一个 HTTP 末端，当所有能够 表达自身就绪的插件都已就绪时，在此末端返回 200 OK。
 - [kubernetes](https://coredns.io/plugins/kubernetes/)：CoreDNS 将基于 Kubernetes 的服务和 Pod 的 IP 答复 DNS 查询。你可以在CoreDNS 网站阅读更多细节。 你可以使用 ttl 来定制响应的 TTL。默认值是 5 秒钟。TTL 的最小值可以是 0 秒钟，最大值为 3600 秒。将 TTL 设置为 0 可以禁止对 DNS 记录进行缓存。
 `pods insecure` 选项是为了与 kube-dns 向后兼容。你可以使用 pods verified 选项，该选项使得 仅在相同名称空间中存在具有匹配 IP 的 Pod 时才返回 A 记录。如果你不使用 Pod 记录，则可以使用 pods disabled选项。
 - [prometheus](https://coredns.io/plugins/prometheus/)：CoreDNS 的度量指标值以 Prometheus 格式在http://localhost:9153/metrics 上提供。
 - [forward](https://coredns.io/plugins/forward/): 不在 Kubernetes 集群域内的任何查询都将转发到 预定义的解析器 (/etc/resolv.conf).
 - [cache](https://coredns.io/plugins/cache/)：启用前端缓存。
 - [loop](https://coredns.io/plugins/loop/)：检测到简单的转发环，如果发现死循环，则中止 CoreDNS 进程。
 - [reload](https://coredns.io/plugins/reload/)：允许自动重新加载已更改的 Corefile。 编辑 ConfigMap 配置后，请等待两分钟，以使更改生效。
 - [loadbalance](https://coredns.io/plugins/loadbalance/)：这是一个轮转式 DNS 负载均衡器， 它在应答中随机分配 A、AAAA 和 MX 记录的顺序


## 6. 使用 CoreDNS 配置存根域和上游域名服务器
**示例** 
如果集群操作员在 10.150.0.1 处运行了 Consul 域服务器， 且所有 Consul 名称都带有后缀 `.consul.local`。要在 CoreDNS 中对其进行配置， 集群管理员可以在 CoreDNS 的 ConfigMap 中创建加入以下字段。

```bash
consul.local:53 {
        errors
        cache 30
        forward . 10.150.0.1
    }
```

要显式强制所有非集群 DNS 查找通过特定的域名服务器（位于 172.16.0.1），可将 `forward` 指向该域名服务器，而不是 `/etc/resolv.conf`。

```bash
forward .  172.16.0.1
```

最终的包含默认的 Corefile 配置的 ConfigMap 如下所示：

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        forward . 172.16.0.1
        cache 30
        loop
        reload
        loadbalance
    }
    consul.local:53 {
        errors
        cache 30
        forward . 10.150.0.1
    }    
```

工具 kubeadm 支持将 `kube-dns ConfigMap` 自动转换为 `CoreDNS ConfigMap`。

> 说明： 尽管 kube-dns 接受 FQDN（例如：ns.foo.com）作为存根域和名字服务器，CoreDNS 不支持此功能。转换期间，CoreDNS 配置中将忽略所有的 FQDN 域名服务器。

## 7. CoreDNS 配置等同于 kube-dns
CoreDNS 不仅仅提供 kube-dns 的功能。 为 kube-dns 创建的 ConfigMap 支持 StubDomains 和 upstreamNameservers 转换为 CoreDNS 中的 forward 插件。 同样，kube-dns 中的 Federations 插件会转换为 CoreDNS 中的 federation 插件。

示例
用于 kubedns 的此示例 ConfigMap 描述了 `federations`、`stubdomains` and `upstreamnameservers`：

```bash
apiVersion: v1
data:
  federations: |
        {"foo" : "foo.feddomain.com"}
  stubDomains: |
        {"abc.com" : ["1.2.3.4"], "my.cluster.local" : ["2.3.4.5"]}
  upstreamNameservers: |
        ["8.8.8.8", "8.8.4.4"]
kind: ConfigMap
```
CoreDNS 中的等效配置将创建一个 Corefile：

 - 针对 federations:

```bash
federation cluster.local {
    foo foo.feddomain.com
}
```

 - 针对 stubDomains:

```bash
abc.com:53 {
     errors
     cache 30
     proxy . 1.2.3.4
 }
 my.cluster.local:53 {
     errors
     cache 30
     proxy . 2.3.4.5
 }
```
带有默认插件的完整 Corefile：

```bash
.:53 {
    errors
    health
    kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
    }
    federation cluster.local {
       foo foo.feddomain.com
    }
    prometheus :9153
    forward .  8.8.8.8 8.8.4.4
    cache 30
}
abc.com:53 {
    errors
    cache 30
    forward . 1.2.3.4
}
my.cluster.local:53 {
    errors
    cache 30
    forward . 2.3.4.5
}
```
## 8. kube-dns 迁移CoreDNS

在Kubernetes中部署CoreDNS作为集群内的DNS服务有[很多种方式，](https://coredns.io/2018/05/21/migration-from-kube-dns-to-coredns/)例如可以使用官方Helm Chart库中的helm chart部署，具体可查看[CoreDNS Helm Chart](https://github.com/helm/charts/tree/master/stable/coredns)。这里继承我们之前部署kubeDNS的传统，使用Kubernetes中addon库中的yaml文件部署，地址在这里[coredns addon](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/dns/coredns)。

查看`transforms2sed.sed`的内容：

```bash
s/__PILLAR__DNS__SERVER__/$DNS_SERVER_IP/g
s/__PILLAR__DNS__DOMAIN__/$DNS_DOMAIN/g
s/__PILLAR__CLUSTER_CIDR__/$SERVICE_CLUSTER_IP_RANGE/g
s/__MACHINE_GENERATED_WARNING__/Warning: This is a file generated from the base underscore template file: __SOURCE_FILENAME__/g
```

将`$DNS_SERVER_IP`和`$DNS_DOMAIN`替换成kubelet配置的内容。这里将`$DNS_SERVER_IP`替换成`10.96.0.10`，将`DNS_DOMAIN`替换成`cluster.local`。

执行下面的命令，生成部署coreDNS所需的`coredns.yaml`文件：

```bash
sed -f transforms2sed.sed coredns.yaml.base > coredns.yaml
```

对coredns.yaml做微调，如修改镜像地址为私有镜像仓库，调整副本数量等等。

```bash
kubectl delete -f kube-dns.yaml #删除原来的kubeDNS部署
kubectl apply -f coredns.yaml
```

查看coredns的Pod，确认所有Pod都处于Running状态：

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
NAME                       READY     STATUS    RESTARTS   AGE
coredns-699477c54d-9fsl2   1/1       Running   0          5m
coredns-699477c54d-d6tb2   1/1       Running   0          5m
coredns-699477c54d-qh54v   1/1       Running   0          5m
coredns-699477c54d-vvqj9   1/1       Running   0          5m
coredns-699477c54d-xcv8h   1/1       Running   0          6m
```

测试一下DNS功能是否好用：

```bash
kubectl run curl --image=radial/busyboxplus:curl -i --tty

nslookup kubernetes.default
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes
Address 1: 10.96.0.1 kubernetes.default.svc.cluster.local
```

DNS服务是Kubernetes赖以实现服务发现的核心组件之一，默认情况下只会创建一个DNS Pod，在生产环境中我们需要对coredns进行扩容。 有两种方式：

 - 手动扩容 kubectl –namespace=kube-system scale deployment coredns
   –replicas=
 - 使用[DNS Horizontal Autoscaler](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/dns-horizontal-autoscaler)


## 9. 搭建并应用 coredns
[https://www.yisu.com/zixun/990.html](https://www.yisu.com/zixun/990.html)

参考：

 - [https://kubernetes.io/zh/docs/tasks/administer-cluster/dns-custom-nameservers/](https://kubernetes.io/zh/docs/tasks/administer-cluster/dns-custom-nameservers/)
 - [https://coredns.io/plugins/](https://coredns.io/plugins/)

