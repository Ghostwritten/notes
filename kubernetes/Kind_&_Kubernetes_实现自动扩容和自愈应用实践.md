# Kind & Kubernetes 自动扩容和自愈
tags: 实践

![](https://img-blog.csdnimg.cn/b230d0af392541b6a53bcbb52cc51e4a.png)




## 1. 背景
在生产非 kubernetes 集群中，负载均衡器往往是集群的唯一入口，它在接受访问流量后，一般会将流量通过加权轮训的方式转发到后端集群。负载均衡器一般是直接使用云厂商的产品，有一些团队也会自建高可用的  Nginx 作为集群入口。为了保证伸缩组节点的业务一致性，弹性伸缩组的所有 VM 都使用同一个虚拟机镜像。其次，要在 VM 粒度实现业务自愈，常见的方案是使用 [Crontab](https://blog.csdn.net/xixihahalelehehe/article/details/105746316) 定时检查业务进程或者通过守护进程的方式来运行。

传统扩容和自愈的缺点但是，这种架构有一些显而易见的缺陷。最大的问题有两个：
- 扩容慢；
- 负载均衡无法感知业务健康情况。

扩容慢主要体现在两方面。首先是 VM 指标会有一定的延迟；其次，扩容的 VM 冷启动时间比较慢，弹性伸缩组需要执行购买 VM、配置镜像、加入伸缩组、启动 VM 等操作。这会让我们失去扩容的最佳时机，并最终影响用户体验。负载均衡无法感知业务健康情况的意思是，VM 是否加入到弹性伸缩组接收外部流量，一般取决于 VM 的健康状态，但 VM 健康并不等于业务健康，这导致在扩缩容的过程中，请求仍然有可能会转发至业务不健康的节点，造成业务短暂中断的问题。


Kubernets 可以自动自愈和自动扩容。接下来我们一起验证。

##  2. 准备
- [安装系统 Centos 8.2](https://blog.csdn.net/xixihahalelehehe/article/details/127616480)
- [初始化  Centos 8.2](https://blog.csdn.net/xixihahalelehehe/article/details/127641551)
- [安装 Podman](https://blog.csdn.net/xixihahalelehehe/article/details/127953530)
- [安装 kubectl](https://kubernetes.io/docs/tasks/tools/) （[kubernetes yum 源](https://blog.csdn.net/xixihahalelehehe/article/details/105685088)）
- [官方步骤安装 Kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)

我的 [kind 安装方式](https://blog.csdn.net/xixihahalelehehe/article/details/121968488)

```bash
curl -Lo ./kind https://github.com/kubernetes-sigs/kind/releases/download/v0.17.0/kind-linux-amd64
chmod +x ./kind
mv ./kind /usr/local/bin/kind
```

## 3. kind 部署 kubernetes
- `config.yaml`
```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
```
创建 K8s 集群:

```bash
$ kind create cluster --config config.yaml
enabling experimental podman provider
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.25.3) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a nice day! 👋
```

##  4.实践

- Pod 会被 `Deployment` 工作负载管理起来，例如创建和销毁等；
- `Service` 相当于弹性伸缩组的负载均衡器，它能以加权轮训的方式将流量转发到多个 Pod 副本上；
- `Ingress` 相当于集群的外网访问入口。

###  4.1 部署 deployment

```bash
kubectl create deployment hello-world-flask --image=ghostwritten/hello-world-flask:latest --replicas=2 
```
单纯输出 `Manifest` 内容：

```bash
$ kubectl create deployment hello-world-flask --image ghostwritten/hello-world-flask:latest --replicas=2 --dry-run=client -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: hello-world-flask
  name: hello-world-flask
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-world-flask
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: hello-world-flask
    spec:
      containers:
      - image: ghostwritten/hello-world-flask:latest
        name: hello-world-flask
        resources: {}
status: {}
```
### 4.2 创建 Service

```bash
kubectl create service clusterip hello-world-flask --tcp=5000:5000
```
### 4.3 创建 Ingress

```bash
kubectl create ingress hello-world-flask --rule="/=hello-world-flask:5000"
```
###  4.4 部署 Ingress-nginx

```bash
kubectl create -f https://ghproxy.com/https://raw.githubusercontent.com/lyzhang1999/resource/main/ingress-nginx/ingress-nginx.yaml
```

###  4.5 K8s 实现自愈

```bash
$ kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
hello-world-flask-56fbff68c8-2xz7w   1/1     Running   0          3m38s
hello-world-flask-56fbff68c8-4f9qz   1/1     Running   0          3m38s
```
由于之前我们已经通过 Kind 在本地创建了集群，也暴露监听了本地的 80 端口，所以集群的 Ingress 访问入口是 `127.0.0.1`。有了 Ingress，我们访问 Pod 就不再需要进行端口转发了，我们可以直接访问 127.0.0.1。下面的命令会每隔 1 秒钟发送一次请求，并打印出时间和返回内容：

```bash
$ while true; do sleep 1; curl http://127.0.0.1; echo -e '\n'$(date);done
Hello, my first docker images! hello-world-flask-56fbff68c8-4f9qz
2022年 9月 7日 星期三 19时21分03秒 CST
Hello, my first docker images! hello-world-flask-56fbff68c8-2xz7w
2022年 9月 7日 星期三 19时21分04秒 CST
```
在这里，“Hello, my first docker images” 后面紧接的内容是 Pod 名称。通过返回内容我们会发现，请求被平均分配到了两个 Pod 上，Pod 名称是交替出现的。我们要保留这个命令行窗口，以便继续观察。

接下来，我们模拟其中的一个 Pod 宕机，观察返回内容。打开一个新的命令行窗口，执行下面的命令终止容器内的 Python 进程，这个操作是在模拟进程意外中止导致宕机的情况。

```bash
$ kubectl exec -it hello-world-flask-56fbff68c8-2xz7w -- bash -c "killall python3"
```
另一个终端：

```bash
$ while true; do sleep 1; curl http://127.0.0.1; echo -e '\n'$(date);done
Hello, my first docker images! hello-world-flask-56fbff68c8-4f9qz
2022年 9月 7日 星期三 19时27分44秒 CST
Hello, my first docker images! hello-world-flask-56fbff68c8-4f9qz
2022年 9月 7日 星期三 19时27分45秒 CST
Hello, my first docker images! hello-world-flask-56fbff68c8-4f9qz
```
所有的请求流量都被转发到了没有故障的 Pod，也就是说，故障成功地被转移了！等待几秒钟，继续观察，我们会重新发现 hello-world-flask-56fbff68c8-2xz7w Pod 的返回内容，这说明 Pod 被重启恢复后，重新加入到了负载均衡接收外部流量：

```bash
Hello, my first docker images! hello-world-flask-56fbff68c8-2xz7w
2022年 9月 7日 星期三 19时27分52秒 CST
Hello, my first docker images! hello-world-flask-56fbff68c8-4f9qz
2022年 9月 7日 星期三 19时27分53秒 CST
Hello, my first docker images! hello-world-flask-56fbff68c8-2xz7w
```
然后，我们再次使用 kubectl get pods 查看 Pod：

```bash
$ kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
hello-world-flask-56fbff68c8-2xz7w   1/1     Running   1(1m ago)  3m38s
hello-world-flask-56fbff68c8-4f9qz   1/1     Running   0          3m38s
```
这里要注意看， hello-world-flask-56fbff68c8-2xz7w Pod 的 RESTARTS 值为 1 ，也就是说 K8s 自动帮我们重启了这个 Pod。

我们重新来梳理一下全过程。首先， K8s 感知到了业务 Pod 故障，立刻进行了故障转移并隔离了有故障的 Pod，并将请求转发到了其他健康的 Pod 中。随后重启了有故障的 Pod，最后将重启后的 Pod 加入到了负载均衡并开始接收外部请求。这些过程都是自动化完成的。

### 4.6 k8s 实现自动扩容
安装 [K8s Metric Server](https://blog.csdn.net/xixihahalelehehe/article/details/120069858)

```bash
kubectl apply -f https://ghproxy.com/https://raw.githubusercontent.com/lyzhang1999/resource/main/metrics/metrics.yaml
```
等待 Metric 工作负载就绪

```bash
kubectl wait deployment -n kube-system metrics-server --for condition=Available=True --timeout=90s
```
Metric Server 就绪后，通过 kubectl autoscale 命令来为 Deployment 创建自动扩容策略：

```bash
kubectl autoscale deployment hello-world-flask --cpu-percent=50 --min=2 --max=10
```
其中，`–cpu-percent` 表示 CPU 使用率阈值，当 CPU 超过 50% 时将进行自动扩容，–min 代表最小的 Pod 副本数，–max 代表最大扩容的副本数。也就是说，自动扩容会根据 CPU 的使用率在 2 个副本和 10 个副本之间进行扩缩容。

最后，要使自动扩容生效，还需要为我们刚才部署的 hello-world-flask Deployment 设置资源配额。你可以通过下面的命令来配置：

```bash
kubectl patch deployment hello-world-flask --type='json' -p='[{"op": "add", "path": "/spec/template/spec/containers/0/resources", "value": {"requests": {"memory": "100Mi", "cpu": "100m"}}}]'
```
现在，Deployment 将会重新创建两个新的 Pod，你可以使用下面的命令筛选出新的 Pod：

```bash
$ kubectl get pod --field-selector=status.phase==Running
NAME                                 READY   STATUS    RESTARTS   AGE
hello-world-flask-64dd645c57-4clbp   1/1     Running   0          117s
hello-world-flask-64dd645c57-cc6g6   1/1     Running   0          117s
```
选择一个 Pod 并使用 kubectl exec 进入到容器内：

```bash
$ kubectl exec -it hello-world-flask-64dd645c57-4clbp -- bash
root@hello-world-flask-64dd645c57-4clbp:/app#
```
接下来，我们模拟业务高峰期场景，使用 [ab](https://blog.csdn.net/xixihahalelehehe/article/details/108978794) 命令来创建并发请求：

```bash
root@hello-world-flask-64dd645c57-4clbp:/app# ab -c 50 -n 10000 http://127.0.0.1:5000/
```
在这条压力测试的命令中，-c 代表 50 个并发数，-n 代表一共请求 10000 次，整个过程大概会持续十几秒。接下来，我们打开一个新的命令行窗口，使用下面的命令来持续监控 Pod 的状态：

```bash
$ kubectl get pods --watch
NAME                                 READY   STATUS    RESTARTS   AGE
hello-world-flask-64dd645c57-9x869   1/1     Running   0          4m6s
hello-world-flask-64dd645c57-vw8nc   0/1     Pending   0          0s
hello-world-flask-64dd645c57-46b6s   0/1     ContainerCreating   0          0s
hello-world-flask-64dd645c57-vw8nc   1/1     Running             0          18s
```
这里参数 --watch 表示持续监听 Pod 状态变化。在 ab 压力测试的过程中，会不断创建新的 Pod 副本，这说明 K8s 已经感知到了 Pod 的业务压力，并且正在自动进行横向扩容。


##  5. 其他

 1. [Kubernetes HPA](https://www.kubecost.com/kubernetes-autoscaling/kubernetes-hpa/)：Vertical Pod Autoscaler  根据 CPU 利用率增加或减少复制控制器、部署、副本集或有状态集中的 pod 数量——缩放是水平的
 2. [Kubernetes VPA](https://www.kubecost.com/kubernetes-autoscaling/kubernetes-vpa/)：Horizo​​ntal Pod Autoscaler 增加和减少容器 CPU 和内存资源配置，以使集群资源分配与实际使用情况保持一致。
 3. [Kubernetes CA](https://www.kubecost.com/kubernetes-autoscaling/kubernetes-cluster-autoscaler/) ：Cluster Autoscaler 根据 pod 的资源请求自动添加或删除集群中的节点。

