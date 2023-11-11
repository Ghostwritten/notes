

 - [kubernetes快速学习手册](https://ghostwritten.blog.csdn.net/article/details/108562082)
 - [promethues快速学习手册](https://ghostwritten.blog.csdn.net/article/details/113100748)

-------
## 1. 背景
Prometheus 项目与 Kubernetes 项目一样，也来自于 Google 的 `Borg` 体系，它的原型系统，叫作 BorgMon，是一个几乎与 Borg 同时诞生的内部监控系统。而 Prometheus 项目的发起原因也跟 Kubernetes 很类似，都是希望通过对用户更友好的方式，将 Google 内部系统的设计理念，传递给用户和开发者。

## 2. 简介
作为一个监控系统，Prometheus 项目的作用和工作方式，其实可以用如下所示的一张官方示意图来解释。
![在这里插入图片描述](https://img-blog.csdnimg.cn/9176fd1db6f54bfba5db4a3563e87d3a.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
可以看到，Prometheus 项目工作的核心，是使用 Pull （抓取）的方式去搜集被监控对象的 Metrics 数据（监控指标数据），然后，再把这些数据保存在一个 TSDB （时间序列数据库，比如 OpenTSDB、InfluxDB 等）当中，以便后续可以按照时间进行检索。

有了这套核心监控机制， `Prometheus` 剩下的组件就是用来配合这套机制的运行。比如 `Pushgateway`，可以允许被监控对象以 Push 的方式向 Prometheus 推送 Metrics 数据。而 `Alertmanager`，则可以根据 Metrics 信息灵活地设置报警。当然， Prometheus 最受用户欢迎的功能，还是通过 `Grafana` 对外暴露出的、可以灵活配置的监控数据可视化界面。
有了 Prometheus 之后，我们就可以按照 Metrics 数据的来源，来对 Kubernetes 的监控体系做一个汇总了。

## 3. 监控对象类型
**第一种 Metrics，是宿主机的监控数据**。这部分数据的提供，需要借助一个由 Prometheus 维护的[Node Exporter](https://github.com/prometheus/node_exporter) 工具。一般来说，`Node Exporter` 会以 `DaemonSet` 的方式运行在宿主机上。其实，所谓的 Exporter，就是代替被监控对象来对 Prometheus 暴露出可以被“抓取”的 Metrics 信息的一个辅助进程。

而 `Node Exporter` 可以暴露给 `Prometheus` 采集的 `Metrics` 数据， 也不单单是节点的负载（Load）、CPU 、内存、磁盘以及网络这样的常规信息，它的 Metrics 指标可以说是“包罗万象”，你可以查看[这个列表](https://github.com/prometheus/node_exporter#enabled-by-default)来感受一下。


**第二种 Metrics，是来自于 `Kubernetes` 的 `API Server`、`kubelet` 等组件的 `/metrics API`**。除了常规的 CPU、内存的信息外，这部分信息还主要包括了各个组件的核心监控指标。比如，对于 API Server 来说，它就会在 /metrics API 里，暴露出各个 `Controller` 的工作队列（Work Queue）的长度、请求的 QPS 和延迟数据等等。这些信息，是检查 Kubernetes 本身工作情况的主要依据。

**第三种 Metrics，是 Kubernetes 相关的监控数据**。这部分数据，一般叫作 `Kubernetes` 核心监控数据（core metrics）。这其中包括了 Pod、Node、容器、Service 等主要 Kubernetes 核心概念的 `Metrics`。

其中，容器相关的 Metrics 主要来自于 `kubelet` 内置的 `cAdvisor` 服务。在 kubelet 启动后，cAdvisor 服务也随之启动，而它能够提供的信息，可以细化到每一个容器的 CPU 、文件系统、内存、网络等资源的使用情况。

##  4. Metrics Server简介

需要注意的是，这里提到的 Kubernetes 核心监控数据，其实使用的是 Kubernetes 的一个非常重要的扩展能力，叫作 `Metrics Server`。我们可以通过在`--api-server`标志设置为 true的情况下运行 Heapster 的资源指标 API实现

`Metrics Server` 在 `Kubernetes` 社区的定位，其实是用来取代 `Heapster` 这个项目的。在 Kubernetes 项目发展的初期，`Heapster` 是用户获取 Kubernetes 监控数据（比如 `Pod` 和 `Node` 的资源使用情况） 的主要渠道。而后面提出来的 `Metrics Server`，则把这些信息，通过标准的 `Kubernetes API` 暴露了出来。这样，Metrics 信息就跟 `Heapster` 完成了解耦，允许 `Heapster` 项目慢慢退出舞台。

而有了 `Metrics Server` 之后，用户就可以通过标准的 `Kubernetes API` 来访问到这些监控数据了。比如，下面这个 `URL`：

```bash
http://127.0.0.1:8001/apis/metrics.k8s.io/v1beta1/namespaces/<namespace-name>/pods/<pod-name>
```
当你访问这个 `Metrics API` 时，它就会为你返回一个 `Pod` 的监控数据，而这些数据，其实是从 `kubelet` 的 `Summary API` （即 `<kubelet_ip>:<kubelet_port>/stats/summary`）采集而来的。`Summary API` 返回的信息，既包括了 `cAdVisor` 的监控数据，也包括了 `kubelet` 本身汇总的信息。

需要指出的是， `Metrics Server` 并不是 `kube-apiserver` 的一部分，而是通过 `Aggregator` 这种插件机制，在独立部署的情况下同 kube-apiserver 一起统一对外服务的。

**系统资源的采集均使用Metrics-Server服务，可以通过Metrics-Server服务采集节点和Pod的内存、磁盘、CPU和网络的使用率等信息**。
特性：

 - Metrics API 只可以查询当前的度量数据，并不保存历史数据
 - Metrics API URI 为 /apis/metrics.k8s.io/，在 [k8s.io/metrics](https://github.com/kubernetes/metrics)维护
 - 必须部署 `metrics-server` 才能使用该 API，metrics-server 通过调用 `Kubelet Summary API` 获取数据

## 5. Metrics Server部署
### 5.1 下载并解压Metrics-Server

```bash
wget https://github.com/kubernetes-sigs/metrics-server/archive/v0.3.6.tar.gz
tar -zxvf v0.3.6.tar.gz 
```
### 5.2 修改Metrics-Server配置文件

```bash
$ cd metrics-server-0.3.6/deploy/1.8+/
$ vim metrics-server-deployment.yaml

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: metrics-server
  namespace: kube-system
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: metrics-server
  namespace: kube-system
  labels:
    k8s-app: metrics-server
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  template:
    metadata:
      name: metrics-server
      labels:
        k8s-app: metrics-server
    spec:
      serviceAccountName: metrics-server
      volumes:
      # mount in tmp so we can safely use from-scratch images and/or read-only containers
      - name: tmp-dir
        emptyDir: {}
      containers:
      - name: metrics-server
        # 修改image 和 imagePullPolicy
        image: mirrorgooglecontainers/metrics-server-amd64:v0.3.6
        imagePullPolicy: IfNotPresent
        # 新增command配置
        command:
        - /metrics-server
        - --kubelet-insecure-tls
        - --kubelet-preferred-address-types=InternalDNS,InternalIP,ExternalDNS,ExternalIP,Hostname
        volumeMounts:
        - name: tmp-dir
          mountPath: /tmp
        # 新增resources配置
        resources:
          limits:
            cpu: 300m
            memory: 200Mi
          requests:
            cpu: 200m
            memory: 100Mi
```

### 5.3 安装Metrics-Server

```bash
kubectl apply -f metrics-server-0.3.6/deploy/1.8+/
```
### 5.4 查看metric-server监控信息

```bash
$ kubectl get apiservice |grep metric
v1beta1.metrics.k8s.io                 kube-system/metrics-server   True        16h


$ kubectl api-resources  |grep metric
nodes                                          metrics.k8s.io/v1beta1                 false        NodeMetrics
pods                                           metrics.k8s.io/v1beta1                 true         PodMetrics


$ kubectl top node
NAME     CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
master   332m         16%    1222Mi          64%       
node1    192m         9%     845Mi           44%       
node2    202m         10%    855Mi           45%     

$ kubectl top pod
NAME                                     CPU(cores)   MEMORY(bytes)   
image-bouncer-webhook-57f5ff98f4-hwtlm   1m           4Mi             
$ kubectl top pod -n kube-system
NAME                                       CPU(cores)   MEMORY(bytes)   
calico-kube-controllers-57fc9c76cc-knw5x   1m           24Mi            
calico-node-9nqnh                          43m          40Mi            
calico-node-p7xcc                          43m          62Mi            
calico-node-zqfcb                          40m          55Mi            
coredns-74ff55c5b-nvmcs                    7m           17Mi            
coredns-74ff55c5b-psm62                    4m           8Mi             
etcd-master                                23m          59Mi            
kube-apiserver-master                      88m          292Mi           
kube-controller-manager-master             28m          46Mi            
kube-proxy-6khfc                           1m           24Mi            
kube-proxy-b6gjh                           1m           16Mi            
kube-proxy-dtc84                           1m           12Mi            
kube-scheduler-master                      4m           18Mi            
metrics-server-b7cff9c67-jsrrx             1m           11Mi      
  
通过API方式获取监控值
$ kubectl get --raw "/apis/metrics.k8s.io/v1beta1/nodes" | jq
{
  "kind": "NodeMetricsList",
  "apiVersion": "metrics.k8s.io/v1beta1",
  "metadata": {
    "selfLink": "/apis/metrics.k8s.io/v1beta1/nodes"
  },
  "items": [
    {
      "metadata": {
        "name": "master",
        "selfLink": "/apis/metrics.k8s.io/v1beta1/nodes/master",
        "creationTimestamp": "2021-09-02T15:47:07Z"
      },
      "timestamp": "2021-09-02T15:46:26Z",
      "window": "30s",
      "usage": {
        "cpu": "287982098n",
        "memory": "1156520Ki"
      }
    },
    {
      "metadata": {
        "name": "node1",
        "selfLink": "/apis/metrics.k8s.io/v1beta1/nodes/node1",
        "creationTimestamp": "2021-09-02T15:47:07Z"
      },
      "timestamp": "2021-09-02T15:46:30Z",
      "window": "30s",
      "usage": {
        "cpu": "208820792n",
        "memory": "866488Ki"
      }
    },
    {
      "metadata": {
        "name": "node2",
        "selfLink": "/apis/metrics.k8s.io/v1beta1/nodes/node2",
        "creationTimestamp": "2021-09-02T15:47:07Z"
      },
      "timestamp": "2021-09-02T15:46:31Z",
      "window": "30s",
      "usage": {
        "cpu": "191420274n",
        "memory": "878348Ki"
      }
    }
  ]
}

```
`Metrics-Server`收集到节点信息，说明`Metrics-Server`安装成功


##  6. Aggregator APIServer
### 6.1 简介
`API Aggregation` 允许在不修改 Kubernetes 核心代码的同时扩展 `Kubernetes API`，即将第三方服务注册到 Kubernetes API 中，这样就可以通过 Kubernetes API 来访问外部服务。

> 备注：另外一种扩展 Kubernetes API 的方法是使用 [CustomResourceDefinition](https://feisky.gitbooks.io/kubernetes/content/concepts/customresourcedefinition.html) (CRD)。

### 6.2 何时使用 `Aggregation`
| 满足以下条件时使用 API Aggregation                                       | 满足以下条件时使用独立 API                      |
|-----------------------------------------------------------------|--------------------------------------|
| 您的 API 是声明式的。                                                   | 您的 API 不适合声明式模型。                     |
| 您希望使用kubectl.                                                   | kubectl 不需要支持                        |
| 您希望在 Kubernetes UI（例如仪表板）中查看新类型以及内置类型。                          | 不需要 Kubernetes UI 支持。                |
| 您正在开发新的 API。                                                    | 您已经有一个程序可以为您的 API 提供服务并且运行良好。        |
| 您愿意接受 Kubernetes 对 REST 资源路径（例如 API 组和命名空间）施加的格式限制。（请参阅API 概述。） | 您需要具有特定的 REST 路径才能与已定义的 REST API 兼容。 |
| 您的资源自然地限定于集群或集群的命名空间。                                           | 集群或命名空间范围的资源不适合；您需要控制资源路径的细节。        |
| 您想重用Kubernetes API 支持功能。                                        | 您不需要这些功能。                            |



这里，`Aggregator APIServer` 的工作原理，可以用如下所示的一幅示意图来表示清楚：
![在这里插入图片描述](https://img-blog.csdnimg.cn/a73fc94c0ca143dea8d9c803d1da86f4.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
参考：[https://www.jetstack.io/blog/resource-and-custom-metrics-hpa-v2/](https://www.jetstack.io/blog/resource-and-custom-metrics-hpa-v2/)

可以看到，当 `Kubernetes` 的 `API Server` 开启了 `Aggregator` 模式之后，你再访问 `apis/metrics.k8s.io/v1beta1` 的时候，实际上访问到的是一个叫作 `kube-aggregator` 的代理。而 `kube-apiserver`，正是这个代理的一个后端；而 `Metrics Server`，则是另一个后端。

而且，在这个机制下，你还可以添加更多的后端给这个 `kube-aggregator`。所以 `kube-aggregator` 其实就是一个根据 URL 选择具体的 API 后端的代理服务器。通过这种方式，我们就可以很方便地扩展 Kubernetes 的 API 了。

### 6.3 开启Aggregator

而 `Aggregator` 模式的开启也非常简单：

 - 如果你是使用 `kubeadm` 或者官方的 [kube-up.sh](https://github.com/kubernetes/kubernetes/blob/master/cluster/kube-up.sh) 脚本部署 Kubernetes 集群的话，Aggregator模式就是默认开启的；
 - 如果是手动 DIY 搭建的话，你就需要在 `kube-apiserver` 的启动参数里加上如下所示的配置：


```bash
--requestheader-client-ca-file=<path to aggregator CA cert>
--requestheader-allowed-names=front-proxy-client
--requestheader-extra-headers-prefix=X-Remote-Extra-
--requestheader-group-headers=X-Remote-Group
--requestheader-username-headers=X-Remote-User
--proxy-client-cert-file=<path to aggregator proxy cert>
--proxy-client-key-file=<path to aggregator proxy key>
```
而这些配置的作用，主要就是为 Aggregator 这一层设置对应的 Key 和 Cert 文件。而这些文件的生成，就需要你自己手动完成了，具体流程请参考[这篇官方文档](https://github.com/kubernetes-sigs/apiserver-builder-alpha/blob/master/docs/concepts/auth.md)。

`Aggregator` 功能开启之后，你只需要将 `Metrics Server` 的 YAML 文件部署起来，如下所示：

```bash
$ git clone https://github.com/kubernetes-incubator/metrics-server
$ cd metrics-server
$ kubectl create -f deploy/1.8+/
```
接下来，你就会看到 `metrics.k8s.io` 这个 API 出现在了你的 `Kubernetes API` 列表当中。

在理解了 Prometheus 关心的三种监控数据源，以及 Kubernetes 的核心 Metrics 之后，作为用户，你其实要做的就是将 `Prometheus Operator` 在 Kubernetes 集群里部署起来。

### 6.4 创建扩展 API

 1. 确保开启 `APIService API`（默认开启，可用 `kubectl get apiservice` 命令验证）
 2. 创建 `RBAC` 规则
 3. 创建一个 `namespace`，用来运行扩展的 API 服务
 4. 创建 `CA` 和证书，用于 `https`
 5. 创建一个存储证书的 `secret`
 6. 创建一个部署扩展 API 服务的 `deployment`，并使用上一步的 secret 配置证书，开启 https 服务
 7. 确保你的扩展 apiserver 从该卷中加载了那些证书，并在 HTTPS 握手过程中使用它们
 8. 创建一个`user`、 `ClusterRole` 和 `ClusterRoleBinding`
 9. 用你命名空间中的服务账号创建一个 Kubernetes 集群角色绑定，绑定到你创建的角色上
 10. 用你命名空间中的服务账号创建一个 Kubernetes 集群角色绑定，绑定到 `system:auth-delegator` 集群角色，以将 auth 决策委派给 Kubernetes 核心 API 服务器。
 11. 以你命名空间中的服务账号创建一个 Kubernetes 集群角色绑定，绑定到 `extension-apiserver-authentication-reader` 角色。 这将让你的扩展 `api-server`能够访问 `extension-apiserver-authentication configmap`。
 12. 创建一个非 `namespace` 的 `apiservice`，注意设置 `spec.caBundle`
 13. 运行 `kubectl get <resource-name>`，正常应该返回 `No resources found`.

可以使用 [apiserver-builder](https://github.com/kubernetes-sigs/apiserver-builder-alpha) 工具自动化上面的步骤：

```bash
# 初始化项目
$ cd GOPATH/src/github.com/my-org/my-project
$ apiserver-boot init repo --domain <your-domain>
$ apiserver-boot init glide

# 创建资源
$ apiserver-boot create group version resource --group <group> --version <version> --kind <Kind>

# 编译
$ apiserver-boot build executables
$ apiserver-boot build docs

# 本地运行
$ apiserver-boot run local

# 集群运行
$ apiserver-boot run in-cluster --name nameofservicetorun --namespace default --image gcr.io/myrepo/myimage:mytag
$ kubectl create -f sample/<type>.yaml 
```
### 6.5 示例
开发[sameple-apiserver](https://blog.gmem.cc/kubernetes-style-apiserver)详解
见 [sample-apiserver](https://github.com/kubernetes/sample-apiserver) 和 [apiserver-builder/example](https://github.com/kubernetes-sigs/apiserver-builder-alpha/tree/master/example)。
## 7. 总结
介绍了 `Prometheus` 项目在这套体系中的地位，讲解了以 Prometheus 为核心的监控系统的架构设计。

然后，我为你详细地解读了 Kubernetes 核心监控数据的来源，即：`Metrics Server` 的具体工作原理，以及 `Aggregator APIServer` 的设计思路。

通过以上讲述，我希望你能够对 Kubernetes 的监控体系形成一个整体的认知，体会到 Kubernetes 社区在监控这个事情上，全面以 Prometheus 项目为核心进行建设的大方向。最后，**在具体的监控指标规划上，我建议你遵循业界通用的 USE 原则和 RED 原则**。

其中，`USE` 原则指的是，按照如下三个维度来规划资源监控指标：

 - 利用率（Utilization），资源被有效利用起来提供服务的平均时间占比；
 - 饱和度（Saturation），资源拥挤的程度，比如工作队列的长度；
 - 错误率（Errors），错误的数量。

而 `RED` 原则指的是，按照如下三个维度来规划服务监控指标：

 - 每秒请求数量（Rate）；
 - 每秒错误数量（Errors）；
 - 服务响应时间（Duration）。

不难发现， USE 原则主要关注的是“资源”，比如节点和容器的资源使用情况，而 RED 原则主要关注的是“服务”，比如 `kube-apiserver` 或者某个应用的工作情况。这两种指标，在我今天为你讲解的 Kubernetes + Prometheus 组成的监控体系中，都是可以完全覆盖到的。

