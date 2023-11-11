
![](https://img-blog.csdnimg.cn/ffbbf05047784d69b0922b180ce0dd41.png)



##  pod 简介
Pod 是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元。

Pod（就像在鲸鱼荚或者豌豆荚中）是一组（一个或多个） 容器； 这些容器共享存储、网络、以及怎样运行这些容器的声明。 Pod 中的内容总是并置（colocated）的并且一同调度，在共享的上下文中运行。 Pod 所建模的是特定于应用的 “逻辑主机”，其中包含一个或多个应用容器， 这些容器相对紧密地耦合在一起。 在非云环境中，在相同的物理机或虚拟机上运行的应用类似于在同一逻辑主机上运行的云应用。

除了应用容器，Pod 还可以包含在 Pod 启动期间运行的 [Init 容器](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/init-containers/)。 你也可以在集群支持临时性容器的情况下， 为调试的目的注入临时性容器。



## kubectl apply 创建 pod

### 创建一个 nginx pod

```bash
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```
执行：

```bash
kubectl apply -f simple-pod.yaml
```

###  创建一个 执行命令的 pod

```bash
apiVersion: v1
kind: Pod
metadata:
  name: command-demo
  labels:
    purpose: demonstrate-command
spec:
  containers:
  - name: command-demo-container
    image: debian
    command: ["printenv"]
    args: ["HOSTNAME", "KUBERNETES_PORT"]
  restartPolicy: OnFailure
```
执行：

```bash
kubectl apply -f commands.yaml
```
命令参数可以和环境变量进行搭配：

```bash
env:
- name: MESSAGE
  value: "hello world"
command: ["/bin/echo"]
args: ["$(MESSAGE)"]
```

常见命令：

```bash
command: ["/bin/sh"]
args: ["-c", "while true; do echo hello; sleep 10;done"]
```


##  kubectl create 创建 pod
创建一个nginx Pod：

```bash
kubectl create deployment nginx --image=nginx
```

暴露nginx Pod的服务：

```bash
kubectl expose deployment nginx --port=80 --type=LoadBalancer
kubectl expose deployment nginx --port=80 --type=NodePort
```

这将创建一个名为nginx的deployment和一个名为nginx的service。Service将使用LoadBalancer类型，这意味着Kubernetes将为您的服务创建一个外部负载均衡器，并将流量路由到您的nginx Pod。

您可以使用以下命令检查服务是否正在运行：

```bash
kubectl get services
```

## kubectl run 创建 pod


### kubctl run 创建测试 curl pod
- [最全与最实用的 kubectl 命令](https://blog.csdn.net/xixihahalelehehe/article/details/107714611)
```c
$ kubectl run test-pod --image=appropriate/curl --restart=Never --rm -it -- /bin/sh
$ kubectl apply -f https://k8s.io/examples/pods/simple-pod.yaml
/ # curl http://prometheus-kube-prometheus-prometheus.prometheus:9090
<a href="/graph">Found</a>.
```
## 更多 kubectl run 运行 pod 需求

```bash
启动nginx实例。
kubectl run nginx --image=nginx
kubectl run nginx --image=nginx --restart=Never -n mynamespace
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml | kubectl create -n mynamespace -f -
kubectl run busybox --image=busybox --command --restart=Never -it -- env

启动带API组的nginx实例。将来被弃用
kubectl run nginx --image=nginx  --generator=run-pod/v1  

带有标签function=mantou的pod
kubectl run nginx2 --image=nginx   --labels function=mantou
多个标签
kubectl run nginx2 --image=nginx   --labels function=mantou，disk=ssd

# 创建nginx-app的deployment,并记录升级。
kubectl run nginx-app --image=nginx:1.11.0-alpine --record

使用默认命令启动 nginx 容器，但对该命令使用自定义参数（arg1 .. argN）
kubectl run nginx --image=nginx -- <arg1> <arg2> ... <argN>

启动hazelcast实例，暴露容器端口 5701。
kubectl run hazelcast --image=hazelcast --port=5701

启动hazelcast实例，在容器中设置环境变量“DNS_DOMAIN = cluster”和“POD_NAMESPACE = default”。
kubectl run hazelcast --image=hazelcast --env="DNS_DOMAIN=cluster" --env="POD_NAMESPACE=default"

启动nginx实例，设置副本数5。
kubectl run nginx --image=nginx --replicas=5

配置cpu与内存的pod
kubectl run nginx --image=nginx --restart=Never --requests='cpu=100m,memory=256Mi' --limits='cpu=200m,memory=512Mi'

运行 Dry  打印相应的API对象而不创建它们。
kubectl run nginx --image=nginx --dry-run

在特定的命令空间的一个pod运行多个容器
kubectl run test --image=nginx --image=redis --image=memcached --image=consul --restart=Nerver -n kube-public

启动一个单一的 nginx 实例，但是使用从 JSON 分析的一部分值来重载部署规格.
kubectl run nginx --image=nginx --overrides='{ "apiVersion": "v1", "spec": { ... } }'

启动一个 busybox 的 pod 并将其保留在前台，如果它退出，请不要重新启动它.
kubectl run -i -t busybox --image=busybox --restart=Never

启动 cron 作业计算 π 后2000位，每5分钟打印一次.
kubectl run pi --schedule="0/5 * * * ?" --image=perl --restart=OnFailure -- perl -Mbignum=bpi -wle 'print bpi(2000)'
```
##  Pod 存储
- [配置 Pod 以使用 PersistentVolume 作为存储](https://blog.csdn.net/xixihahalelehehe/article/details/107877700)

## Pod 资源分配策略

- [kubernetes 资源管理策略 Pod 的服务质量（QoS）](https://blog.csdn.net/xixihahalelehehe/article/details/112537920)

##  Pod 安全

- [kubernetes pod podsecurityPolicies（PSP）](https://blog.csdn.net/xixihahalelehehe/article/details/126125551)

##  Pod 状态分析
- [kubernetes pod 状态报错分析](https://blog.csdn.net/xixihahalelehehe/article/details/129728630)
![在这里插入图片描述](https://img-blog.csdnimg.cn/1c667ea08e12452b8bd570cb0bd99049.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/37aec9d37dd44ab59714b4547f3b44ec.png)


##  client-go 开发管理 pod
- [kubernetes dev client-go 进入 pod 执行命令](https://blog.csdn.net/xixihahalelehehe/article/details/111275685)

## Pod 生命周期与探针
- [kubernetes Pod Lifecycle生命周期与livenessProbe、 readinessProbe探测方法](https://blog.csdn.net/xixihahalelehehe/article/details/108561740)

##  Pod  如何在 CI/CD 创建 
- [Jenkins Pipeline & Kubernetes 如何创建 pod](https://blog.csdn.net/xixihahalelehehe/article/details/128170852)


##  pod 完整生命周期
![在这里插入图片描述](https://img-blog.csdnimg.cn/6d9f451249974a7ab0e27b8cda138985.png)

##  pod 状态机制

![在这里插入图片描述](https://img-blog.csdnimg.cn/93797763f18144f0a87682f6a6f99767.png)

## 基于 tain 的 Evctions
![在这里插入图片描述](https://img-blog.csdnimg.cn/9587e35d53a64f7bba15c601bab95e59.png)

