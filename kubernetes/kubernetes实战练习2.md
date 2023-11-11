

[kubernetes实战练习1](https://ghostwritten.blog.csdn.net/article/details/121186089)
[kubernetes实战练习2](https://ghostwritten.blog.csdn.net/article/details/121194823)
[kubernetes实战练习3](https://ghostwritten.blog.csdn.net/article/details/121239607)
[**kubernetes 快速学习手册**](https://ghostwritten.blog.csdn.net/article/details/108562082)

----
##  1. Kubernetes 上部署留言板示例
此场景说明如何使用 Kubernetes 和 Docker 启动简单的多层 Web 应用程序。留言簿示例应用程序通过 JavaScript API 调用将访客的笔记存储在 Redis 中。Redis 包含一个 master（用于存储）和一组复制的 redis 'slaves'。

### 1.1 检查集群

```bash
controlplane $ kubectl cluster-info
Kubernetes master is running at https://172.17.0.29:6443
KubeDNS is running at https://172.17.0.29:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
controlplane $ kubectl get nodes
NAME           STATUS   ROLES    AGE     VERSION
controlplane   Ready    master   2m57s   v1.14.0
node01         Ready    <none>   2m31s   v1.14.0
```
### 1.2 创建rc 

```bash
controlplane $ cat redis-master-controller.yaml 
apiVersion: v1
kind: ReplicationController
metadata:
  name: redis-master
  labels:
    name: redis-master
spec:
  replicas: 1
  selector:
    name: redis-master
  template:
    metadata:
      labels:
        name: redis-master
    spec:
      containers:
      - name: master
        image: redis:3.0.7-alpine
        ports:
        - containerPort: 6379
```
创建
```bash
controlplane $ kubectl create -f redis-master-controller.yaml
replicationcontroller/redis-master created
controlplane $ kubectl get rc
NAME           DESIRED   CURRENT   READY   AGE
redis-master   1         1         0       2s
controlplane $ kubectl get pods
NAME                 READY   STATUS    RESTARTS   AGE
redis-master-2j4qm   1/1     Running   0          4s
```
###  1.3 Redis 主服务
第二部分是服务。Kubernetes 服务是一种命名负载均衡器，它将流量代理到一个或多个容器。即使容器位于不同的节点上，代理也能工作。

服务代理在集群内通信，很少将端口暴露给外部接口。

当您启动服务时，您似乎无法使用 curl 或 netcat 进行连接，除非您将其作为 Kubernetes 的一部分启动。推荐的方法是使用 LoadBalancer 服务来处理外部通信。]

```bash
controlplane $ cat redis-master-service.yaml 
apiVersion: v1
kind: Service
metadata:
  name: redis-master
  labels:
    name: redis-master
spec:
  ports:
    # the port that this service should serve on
  - port: 6379
    targetPort: 6379
  selector:
    name: redis-master
```
创建

```bash
controlplane $ kubectl create -f redis-master-service.yaml
service/redis-master created


controlplane $ kubectl get services
NAME           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
kubernetes     ClusterIP   10.96.0.1      <none>        443/TCP    6m58s
redis-master   ClusterIP   10.111.64.45   <none>        6379/TCP   1s


controlplane $ kubectl describe services redis-master
Name:              redis-master
Namespace:         default
Labels:            name=redis-master
Annotations:       <none>
Selector:          name=redis-master
Type:              ClusterIP
IP:                10.111.64.45
Port:              <unset>  6379/TCP
TargetPort:        6379/TCP
Endpoints:         10.32.0.193:6379
Session Affinity:  None
Events:            <none>
```
###  1.4 rc slave Pod
在这个例子中，我们将运行 Redis Slaves，它会从 master 复制数据。有关 Redis 复制的更多详细信息，请访问http://redis.io/topics/replication

如前所述，控制器定义了服务的运行方式。在这个例子中，我们需要确定服务如何发现其他 pod。YAML 将`GET_HOSTS_FROM`属性表示为 DNS。您可以将其更改为在 yaml 中使用环境变量，但这会引入创建顺序依赖关系，因为需要运行服务才能定义环境变量。
在这种情况下，我们将使用image：`kubernetes/redis-slave:v2`启动 pod 的两个实例。它将通过 DNS链接到`redis-master`。

```bash
controlplane $ cat redis-slave-controller.yaml 
apiVersion: v1
kind: ReplicationController
metadata:
  name: redis-slave
  labels:
    name: redis-slave
spec:
  replicas: 2
  selector:
    name: redis-slave
  template:
    metadata:
      labels:
        name: redis-slave
    spec:
      containers:
      - name: worker
        image: gcr.io/google_samples/gb-redisslave:v1
        env:
        - name: GET_HOSTS_FROM
          value: dns
          # If your cluster config does not include a dns service, then to
          # instead access an environment variable to find the master
          # service's host, comment out the 'value: dns' line above, and
          # uncomment the line below.
          # value: env
        ports:
        - containerPort: 6379
```
执行：

```bash
controlplane $ kubectl create -f redis-slave-controller.yaml
replicationcontroller/redis-slave created
controlplane $ kubectl get rc
NAME           DESIRED   CURRENT   READY   AGE
redis-master   1         1         1       4m29s
redis-slave    2         2         2       3s
```

###  1.5 Redis slave service
和以前一样，我们需要让我们的奴隶可以访问传入的请求。这是通过启动一个知道如何与redis-slave通信的服务来完成的。

因为我们有两个复制的 Pod，该服务还将在两个节点之间提供负载平衡。

```bash
controlplane $ cat redis-slave-service.yaml 
apiVersion: v1
kind: Service
metadata:
  name: redis-slave
  labels:
    name: redis-slave
spec:
  ports:
    # the port that this service should serve on
  - port: 6379
  selector:
    name: redis-slave
```
执行:

```bash
controlplane $ kubectl create -f redis-slave-service.yaml

controlplane $ kubectl get services
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP    14m
redis-master   ClusterIP   10.111.64.45    <none>        6379/TCP   7m13s
redis-slave    ClusterIP   10.109.135.21   <none>        6379/TCP   41s
```
###  1.6 前端 rc 
启动数据服务后，我们现在可以部署 Web 应用程序。部署 Web 应用程序的模式与我们之前部署的 pod 相同。YAML 定义了一个名为 frontend 的服务，该服务使用图像 _`gcr.io/google samples/gb-frontend:v3`。复制控制器将确保三个 Pod 始终存在。

```bash
controlplane $ cat frontend-controller.yaml 
apiVersion: v1
kind: ReplicationController
metadata:
  name: frontend
  labels:
    name: frontend
spec:
  replicas: 3
  selector:
    name: frontend
  template:
    metadata:
      labels:
        name: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
        env:
        - name: GET_HOSTS_FROM
          value: dns
          # If your cluster config does not include a dns service, then to
          # instead access environment variables to find service host
          # info, comment out the 'value: dns' line above, and uncomment the
          # line below.
          # value: env
        ports:
        - containerPort: 80
```
执行

```bash
controlplane $ kubectl create -f frontend-controller.yaml
replicationcontroller/frontend created
controlplane $ kubectl get rc
NAME           DESIRED   CURRENT   READY   AGE
frontend       3         3         1       2s
redis-master   1         1         1       20m
redis-slave    2         2         2       15m
controlplane $ kubectl get pods
NAME                 READY   STATUS    RESTARTS   AGE
frontend-bkcsj       1/1     Running   0          3s
frontend-ftjrk       1/1     Running   0          3s
frontend-jnckp       1/1     Running   0          3s
redis-master-2j4qm   1/1     Running   0          20m
redis-slave-79w2b    1/1     Running   0          15m
redis-slave-j8zqj    1/1     Running   0          15m
```
PHP 代码使用 HTTP 和 JSON 与 Redis 通信。当设置一个值时，请求转到`redis-master`，而读取的数据来自`redis-slave`节点。

###  1.7 Guestbook Frontend Service
为了使前端可访问，我们需要启动一个服务来配置代理。
YAML 将服务定义为NodePort。NodePort 允许您设置在整个集群中共享的知名端口。这就像Docker 中的`-p 80:80`。

在这种情况下，我们定义我们的 Web 应用程序在端口 80 上运行，但我们将在30080上公开服务。

```bash
controlplane $ cat frontend-service.yaml 
apiVersion: v1
kind: Service
metadata:
  name: frontend
  labels:
    name: frontend
spec:
  # if your cluster supports it, uncomment the following to automatically create
  # an external load-balanced IP for the frontend service.
  # type: LoadBalancer
  type: NodePort
  ports:
    # the port that this service should serve on
    - port: 80
      nodePort: 30080
  selector:
    name: frontend


controlplane $ kubectl create -f frontend-service.yaml
service/frontend created


controlplane $ kubectl get services
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
frontend       NodePort    10.105.214.152   <none>        80:30080/TCP   2s
kubernetes     ClusterIP   10.96.0.1        <none>        443/TCP        28m
redis-master   ClusterIP   10.111.64.45     <none>        6379/TCP       21m
redis-slave    ClusterIP   10.109.135.21    <none>        6379/TCP       15m
```
###  1.8 Access Guestbook Frontend
定义了所有控制器和服务后，Kubernetes 将开始将它们作为 Pod 启动。根据发生的情况，Pod 可以具有不同的状态。例如，如果 Docker 镜像仍在下载，则 Pod 将处于挂起状态，因为它无法启动。准备就绪后，状态将更改为running。

查看 Pod 状态

```bash
controlplane $ kubectl get pods
NAME                 READY   STATUS    RESTARTS   AGE
frontend-bkcsj       1/1     Running   0          4m55s
frontend-ftjrk       1/1     Running   0          4m55s
frontend-jnckp       1/1     Running   0          4m55s
redis-master-2j4qm   1/1     Running   0          24m
redis-slave-79w2b    1/1     Running   0          20m
redis-slave-j8zqj    1/1     Running   0          20m
```
查找节点端口

```bash
controlplane $ kubectl describe service frontend | grep NodePort
Type:                     NodePort
NodePort:                 <unset>  30080/TCP
```
查看用户界面
一旦 Pod 处于运行状态，您将能够通过端口 30080 查看 UI。使用 URL 查看页面 `https://2886795293-30080-elsy05.environments.katacoda.com`

在幕后，PHP 服务通过 DNS 发现 Redis 实例。您现在已经在 Kubernetes 上部署了一个有效的多层应用程序。


##  2. 网络介绍
Kubernetes 具有先进的网络功能，允许 Pod 和服务在集群网络内部和外部进行通信。

在此场景中，您将学习以下类型的 Kubernetes 服务。

 - 集群IP
 - 目标端口
 - 节点端口
 - 外部 IP
 - 负载均衡器

Kubernetes 服务是一个抽象，它定义了如何访问一组 Pod 的策略和方法。通过 Service 访问的 Pod 集基于标签选择器。

###  2.1 集群 IP
集群 IP 是创建 Kubernetes 服务时的默认方法。该服务被分配了一个内部 IP，其他组件可以使用它来访问 pod。

通过拥有单个 IP 地址，它可以使服务在多个 Pod 之间进行负载平衡。

```bash
controlplane $ cat clusterip.yaml 
apiVersion: v1
kind: Service
metadata:
  name: webapp1-clusterip-svc
  labels:
    app: webapp1-clusterip
spec:
  ports:
  - port: 80
  selector:
    app: webapp1-clusterip
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webapp1-clusterip-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: webapp1-clusterip
    spec:
      containers:
      - name: webapp1-clusterip-pod
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80


controlplane $ kubectl get pods
NAME                                            READY   STATUS    RESTARTS   AGE
webapp1-clusterip-deployment-669c7c65c4-gqlkc   1/1     Running   0          112s
webapp1-clusterip-deployment-669c7c65c4-hwkrl   1/1     Running   0          112s



controlplane $ kubectl get svc
NAME                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes              ClusterIP   10.96.0.1      <none>        443/TCP   6m28s
webapp1-clusterip-svc   ClusterIP   10.100.49.56   <none>        80/TCP    116s



controlplane $ kubectl describe svc/webapp1-clusterip-svc
Name:              webapp1-clusterip-svc
Namespace:         default
Labels:            app=webapp1-clusterip
Annotations:       kubectl.kubernetes.io/last-applied-configuration:
                     {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"app":"webapp1-clusterip"},"name":"webapp1-clusterip-svc","name...
Selector:          app=webapp1-clusterip
Type:              ClusterIP
IP:                10.100.49.56
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.32.0.5:80,10.32.0.6:80
Session Affinity:  None
Events:            <none>


controlplane $ export CLUSTER_IP=$(kubectl get services/webapp1-clusterip-svc -o go-template='{{(index .spec.clusterIP)}}')


controlplane $ echo CLUSTER_IP=$CLUSTER_IP
CLUSTER_IP=10.100.49.56

controlplane $ curl $CLUSTER_IP:80
<h1>This request was processed by host: webapp1-clusterip-deployment-669c7c65c4-gqlkc</h1>
controlplane $ curl $CLUSTER_IP:80
<h1>This request was processed by host: webapp1-clusterip-deployment-669c7c65c4-gqlkc</h1>
```
多个请求将展示基于公共标签选择器的跨多个 Pod 的服务负载均衡器。

###  2.2 targetport
  目标端口允许我们将服务可用的端口与应用程序正在侦听的端口分开。TargetPort 是应用程序配置为侦听的端口。 Port是从外部访问应用程序的方式。

```bash
controlplane $ cat clusterip-target.yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp1-clusterip-targetport-svc
  labels:
    app: webapp1-clusterip-targetport
spec:
  ports:
  - port: 8080
    targetPort: 80
  selector:
    app: webapp1-clusterip-targetport
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webapp1-clusterip-targetport-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: webapp1-clusterip-targetport
    spec:
      containers:
      - name: webapp1-clusterip-targetport-pod
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
---


controlplane $ kubectl apply -f clusterip-target.yaml
service/webapp1-clusterip-targetport-svc created
deployment.extensions/webapp1-clusterip-targetport-deployment created


controlplane $ kubectl get svc
NAME                               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes                         ClusterIP   10.96.0.1       <none>        443/TCP    11m
webapp1-clusterip-svc              ClusterIP   10.100.49.56    <none>        80/TCP     6m33s
webapp1-clusterip-targetport-svc   ClusterIP   10.99.164.105   <none>        8080/TCP   2s


controlplane $ kubectl describe svc/webapp1-clusterip-targetport-svc
Name:              webapp1-clusterip-targetport-svc
Namespace:         default
Labels:            app=webapp1-clusterip-targetport
Annotations:       kubectl.kubernetes.io/last-applied-configuration:
                     {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"app":"webapp1-clusterip-targetport"},"name":"webapp1-clusterip...
Selector:          app=webapp1-clusterip-targetport
Type:              ClusterIP
IP:                10.99.164.105
Port:              <unset>  8080/TCP
TargetPort:        80/TCP
Endpoints:         10.32.0.7:80,10.32.0.8:80
Session Affinity:  None
Events:            <none>


controlplane $ export CLUSTER_IP=$(kubectl get services/webapp1-clusterip-targetport-svc -o go-template='{{(index .spec.clusterIP)}}')


controlplane $ echo CLUSTER_IP=$CLUSTER_IP
CLUSTER_IP=10.99.164.105
controlplane $ curl $CLUSTER_IP:8080
<h1>This request was processed by host: webapp1-clusterip-targetport-deployment-5599945ff4-9n89k</h1>
controlplane $ curl $CLUSTER_IP:8080
<h1>This request was processed by host: webapp1-clusterip-targetport-deployment-5599945ff4-9n89k</h1>
controlplane $ curl $CLUSTER_IP:8080
```
服务和 pod 部署完成后，可以像以前一样通过集群 IP 访问它，但这次是在定义的端口 8080 上。应用程序本身仍然配置为侦听端口 80。Kubernetes 服务管理着两者之间的转换。

### 2.3 nodeport
虽然 `TargetPort` 和 `ClusterIP` 使其可用于集群内部，但 NodePort 通过定义的静态端口在每个节点的 IP 上公开服务。无论访问集群内的哪个节点，根据定义的端口号都可以访问该服务。

```bash

controlplane $ cat nodeport.yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp1-nodeport-svc
  labels:
    app: webapp1-nodeport
spec:
  type: NodePort
  ports:
  - port: 80
    nodePort: 30080
  selector:
    app: webapp1-nodeport
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webapp1-nodeport-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: webapp1-nodeport
    spec:
      containers:
      - name: webapp1-nodeport-pod
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
---

controlplane $ kubectl apply -f nodeport.yaml
service/webapp1-nodeport-svc created
deployment.extensions/webapp1-nodeport-deployment created


controlplane $ kubectl get svc
NAME                               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes                         ClusterIP   10.96.0.1        <none>        443/TCP        14m
webapp1-clusterip-svc              ClusterIP   10.100.49.56     <none>        80/TCP         9m39s
webapp1-clusterip-targetport-svc   ClusterIP   10.99.164.105    <none>        8080/TCP       3m8s
webapp1-nodeport-svc               NodePort    10.111.226.228   <none>        80:30080/TCP   48s
controlplane $ kubectl describe svc/webapp1-nodeport-svc
Name:                     webapp1-nodeport-svc
Namespace:                default
Labels:                   app=webapp1-nodeport
Annotations:              kubectl.kubernetes.io/last-applied-configuration:
                            {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"app":"webapp1-nodeport"},"name":"webapp1-nodeport-svc","namesp...
Selector:                 app=webapp1-nodeport
Type:                     NodePort
IP:                       10.111.226.228
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30080/TCP
Endpoints:                10.32.0.10:80,10.32.0.9:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
controlplane $ curl 172.17.0.66:30080
<h1>This request was processed by host: webapp1-nodeport-deployment-677bd89b96-hqdbb</h1>
```

###  2.4 External IPs
使服务在集群外可用的另一种方法是通过外部 IP 地址。
将定义更新为当前集群的 IP 地址。

```bash
controlplane $ cat externalip.yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp1-externalip-svc
  labels:
    app: webapp1-externalip
spec:
  ports:
  - port: 80
  externalIPs:
  - HOSTIP
  selector:
    app: webapp1-externalip
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webapp1-externalip-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: webapp1-externalip
    spec:
      containers:
      - name: webapp1-externalip-pod
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
---


controlplane $ sed -i 's/HOSTIP/172.17.0.66/g' externalip.yaml
controlplane $ cat externalip.yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp1-externalip-svc
  labels:
    app: webapp1-externalip
spec:
  ports:
  - port: 80
  externalIPs:
  - 172.17.0.66
  selector:
    app: webapp1-externalip
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webapp1-externalip-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: webapp1-externalip
    spec:
      containers:
      - name: webapp1-externalip-pod
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
---

controlplane $ kubectl apply -f externalip.yaml
service/webapp1-externalip-svc created
deployment.extensions/webapp1-externalip-deployment created
controlplane $ kubectl get svc
NAME                               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes                         ClusterIP   10.96.0.1        <none>        443/TCP        16m
webapp1-clusterip-svc              ClusterIP   10.100.49.56     <none>        80/TCP         11m
webapp1-clusterip-targetport-svc   ClusterIP   10.99.164.105    <none>        8080/TCP       5m15s
webapp1-externalip-svc             ClusterIP   10.101.221.229   172.17.0.66   80/TCP         2s
webapp1-nodeport-svc               NodePort    10.111.226.228   <none>        80:30080/TCP   2m55s


controlplane $ kubectl describe svc/webapp1-externalip-svc
Name:              webapp1-externalip-svc
Namespace:         default
Labels:            app=webapp1-externalip
Annotations:       kubectl.kubernetes.io/last-applied-configuration:
                     {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"app":"webapp1-externalip"},"name":"webapp1-externalip-svc","na...
Selector:          app=webapp1-externalip
Type:              ClusterIP
IP:                10.101.221.229
External IPs:      172.17.0.66
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.32.0.11:80
Session Affinity:  None
Events:            <none>

controlplane $ curl 172.17.0.66
<h1>This request was processed by host: webapp1-externalip-deployment-6446b488f8-tjrpt</h1>
controlplane $ curl 172.17.0.66
<h1>This request was processed by host: webapp1-externalip-deployment-6446b488f8-tjrpt</h1>
```

###  2.5 Load Balancer
在云中运行时，例如 EC2 或 Azure，可以配置和分配通过云提供商发布的公共 IP 地址。这将通过负载均衡器（例如 ELB）发出。这允许将额外的公共 IP 地址分配给 Kubernetes 集群，而无需直接与云提供商交互。

由于 Katacoda 不是云提供商，因此仍然可以为 `LoadBalancer` 类型的服务动态分配 IP 地址。这是通过使用

```bash
controlplane $ cat cloudprovider.yaml 
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: kube-keepalived-vip
  namespace: kube-system
spec:
  template:
    metadata:
      labels:
        name: kube-keepalived-vip
    spec:
      hostNetwork: true
      containers:
        - image: gcr.io/google_containers/kube-keepalived-vip:0.9
          name: kube-keepalived-vip
          imagePullPolicy: Always
          securityContext:
            privileged: true
          volumeMounts:
            - mountPath: /lib/modules
              name: modules
              readOnly: true
            - mountPath: /dev
              name: dev
          # use downward API
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          # to use unicast
          args:
          - --services-configmap=kube-system/vip-configmap
          # unicast uses the ip of the nodes instead of multicast
          # this is useful if running in cloud providers (like AWS)
          #- --use-unicast=true
      volumes:
        - name: modules
          hostPath:
            path: /lib/modules
        - name: dev
          hostPath:
            path: /dev
      nodeSelector:
        # type: worker # adjust this to match your worker nodes
---
## We also create an empty ConfigMap to hold our config
apiVersion: v1
kind: ConfigMap
metadata:
  name: vip-configmap
  namespace: kube-system
data:
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  labels:
    app: keepalived-cloud-provider
  name: keepalived-cloud-provider
  namespace: kube-system
spec:
  replicas: 1
  revisionHistoryLimit: 2
  selector:
    matchLabels:
      app: keepalived-cloud-provider
  strategy:
    type: RollingUpdate
  template:
    metadata:
      annotations:
        scheduler.alpha.kubernetes.io/critical-pod: ""
        scheduler.alpha.kubernetes.io/tolerations: '[{"key":"CriticalAddonsOnly", "operator":"Exists"}]'
      labels:
        app: keepalived-cloud-provider
    spec:
      containers:
      - name: keepalived-cloud-provider
        image: quay.io/munnerz/keepalived-cloud-provider:0.0.1
        imagePullPolicy: IfNotPresent
        env:
        - name: KEEPALIVED_NAMESPACE
          value: kube-system
        - name: KEEPALIVED_CONFIG_MAP
          value: vip-configmap
        - name: KEEPALIVED_SERVICE_CIDR
          value: 10.10.0.0/26 # pick a CIDR that is explicitly reserved for keepalived
        volumeMounts:
        - name: certs
          mountPath: /etc/ssl/certs
        resources:
          requests:
            cpu: 200m
        livenessProbe:
          httpGet:
            path: /healthz
            port: 10252
            host: 127.0.0.1
          initialDelaySeconds: 15
          timeoutSeconds: 15
          failureThreshold: 8
      volumes:
      - name: certs
        hostPath:
          path: /etc/ssl/certs

controlplane $ kubectl apply -f cloudprovider.yaml
daemonset.extensions/kube-keepalived-vip configured
configmap/vip-configmap configured
deployment.apps/keepalived-cloud-provider created


controlplane $ kubectl get pods -n kube-system
NAME                                        READY   STATUS    RESTARTS   AGE
coredns-fb8b8dccf-9hrwv                     1/1     Running   0          21m
coredns-fb8b8dccf-skwkj                     1/1     Running   0          21m
etcd-controlplane                           1/1     Running   0          20m
katacoda-cloud-provider-558d5c854b-6h955    1/1     Running   0          21m
keepalived-cloud-provider-78fc4468b-lpg9s   1/1     Running   0          2m41s
kube-apiserver-controlplane                 1/1     Running   0          20m
kube-controller-manager-controlplane        1/1     Running   0          20m
kube-keepalived-vip-hq7hk                   1/1     Running   0          21m
kube-proxy-468j8                            1/1     Running   0          21m
kube-scheduler-controlplane                 1/1     Running   0          20m
weave-net-w5zff                             2/2     Running   1          21m
```
该服务是通过负载均衡器配置的

```bash
controlplane $ cat loadbalancer.yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp1-loadbalancer-svc
  labels:
    app: webapp1-loadbalancer
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: webapp1-loadbalancer
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webapp1-loadbalancer-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: webapp1-loadbalancer
    spec:
      containers:
      - name: webapp1-loadbalancer-pod
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
---


controlplane $ kubectl get svc
NAME                               TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes                         ClusterIP      10.96.0.1        <none>        443/TCP        23m
webapp1-clusterip-svc              ClusterIP      10.100.49.56     <none>        80/TCP         19m
webapp1-clusterip-targetport-svc   ClusterIP      10.99.164.105    <none>        8080/TCP       12m
webapp1-externalip-svc             ClusterIP      10.101.221.229   172.17.0.66   80/TCP         7m22s
webapp1-loadbalancer-svc           LoadBalancer   10.104.93.133    172.17.0.66   80:31232/TCP   97s
webapp1-nodeport-svc               NodePort       10.111.226.228   <none>        80:30080/TCP   10m


controlplane $ kubectl describe svc/webapp1-loadbalancer-svc
Name:                     webapp1-loadbalancer-svc
Namespace:                default
Labels:                   app=webapp1-loadbalancer
Annotations:              kubectl.kubernetes.io/last-applied-configuration:
                            {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"app":"webapp1-loadbalancer"},"name":"webapp1-loadbalancer-svc"...
Selector:                 app=webapp1-loadbalancer
Type:                     LoadBalancer
IP:                       10.104.93.133
LoadBalancer Ingress:     172.17.0.66
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31232/TCP
Endpoints:                10.32.0.14:80,10.32.0.15:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type    Reason                Age   From                Message
  ----    ------                ----  ----                -------
  Normal  CreatingLoadBalancer  99s   service-controller  Creating load balancer
  Normal  CreatedLoadBalancer   99s   service-controller  Created load balancer
```
现在可以通过分配的 IP 地址访问该服务，在本例中为 `10.10.0.0/26` 范围。

```bash
controlplane $ export LoadBalancerIP=$(kubectl get services/webapp1-loadbalancer-svc -o go-template='{{(index .status.loadBalancer.ingress 0).ip}}')
controlplane $ echo LoadBalancerIP=$LoadBalancerIP
LoadBalancerIP=172.17.0.66

controlplane $ curl $LoadBalancerIP
<h1>This request was processed by host: webapp1-externalip-deployment-6446b488f8-xt4nh</h1>
controlplane $ curl $LoadBalancerIP
<h1>This request was processed by host: webapp1-externalip-deployment-6446b488f8-xt4nh</h1>
```

##  3. Create Ingress Routing
Kubernetes 具有先进的网络功能，允许 Pod 和服务在集群网络内部进行通信。Ingress 启用到集群的入站连接，允许外部流量到达正确的 Pod。

Ingress 启用外部可访问的 url、负载平衡流量、终止 SSL、为 Kubernetes 集群提供基于名称的虚拟主机。

在此场景中，您将学习如何部署和配置 Ingress 规则来管理传入的 HTTP 请求。


###  3.1 创建http部署
首先，部署一个示例 HTTP 服务器，它将成为我们请求的目标。该部署包含三个部署，一个称为webapp1，第二个称为webapp2，第三个称为webapp3，每个部署都有一个服务。

```bash
controlplane $ cat deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp1
  template:
    metadata:
      labels:
        app: webapp1
    spec:
      containers:
      - name: webapp1
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp2
  template:
    metadata:
      labels:
        app: webapp2
    spec:
      containers:
      - name: webapp2
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp3
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp3
  template:
    metadata:
      labels:
        app: webapp3
    spec:
      containers:
      - name: webapp3
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: webapp1-svc
  labels:
    app: webapp1
spec:
  ports:
  - port: 80
  selector:
    app: webapp1
---
apiVersion: v1
kind: Service
metadata:
  name: webapp2-svc
  labels:
    app: webapp2
spec:
  ports:
  - port: 80
  selector:
    app: webapp2
---
apiVersion: v1
kind: Service
metadata:
  name: webapp3-svc
  labels:
    app: webapp3
spec:
  ports:
  - port: 80
  selector:
    app: webapp3


controlplane $ kubectl apply -f deployment.yaml
deployment.apps/webapp1 created
deployment.apps/webapp2 created
deployment.apps/webapp3 created
service/webapp1-svc created
service/webapp2-svc created
service/webapp3-svc created


controlplane $ kubectl get deployment
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
webapp1   0/1     1            0           4s
webapp2   0/1     1            0           4s
webapp3   0/1     1            0           4s
```
###  3.2 部署 Ingress
YAML 文件`ingress.yaml`定义了一个基于 `Nginx` 的入口控制器以及一个服务，使其在端口 80 上可用于使用 `ExternalIPs` 的外部连接。如果 Kubernetes 集群在云提供商上运行，那么它将使用 `LoadBalancer` 服务类型。

ServiceAccount 定义了具有如何访问集群以访问定义的入口规则的一组权限的帐户。默认服务器密钥是其他 Nginx 示例 SSL 连接的自签名证书，并且是所必需的[Nginx 默认示例](https://github.com/nginxinc/kubernetes-ingress/tree/master/deployments)。

```bash
controlplane $ cat ingress.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nginx-ingress
---
apiVersion: v1
kind: Secret
metadata:
  name: default-server-secret
  namespace: nginx-ingress
type: kubernetes.io/tls
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN2akNDQWFZQ0NRREFPRjl0THNhWFhEQU5CZ2txaGtpRzl3MEJBUXNGQURBaE1SOHdIUVlEVlFRRERCWk8KUjBsT1dFbHVaM0psYzNORGIyNTBjbTlzYkdWeU1CNFhEVEU0TURreE1qRTRNRE16TlZvWERUSXpNRGt4TVRFNApNRE16TlZvd0lURWZNQjBHQTFVRUF3d1dUa2RKVGxoSmJtZHlaWE56UTI5dWRISnZiR3hsY2pDQ0FTSXdEUVlKCktvWklodmNOQVFFQkJRQURnZ0VQQURDQ0FRb0NnZ0VCQUwvN2hIUEtFWGRMdjNyaUM3QlBrMTNpWkt5eTlyQ08KR2xZUXYyK2EzUDF0azIrS3YwVGF5aGRCbDRrcnNUcTZzZm8vWUk1Y2Vhbkw4WGM3U1pyQkVRYm9EN2REbWs1Qgo4eDZLS2xHWU5IWlg0Rm5UZ0VPaStlM2ptTFFxRlBSY1kzVnNPazFFeUZBL0JnWlJVbkNHZUtGeERSN0tQdGhyCmtqSXVuektURXUyaDU4Tlp0S21ScUJHdDEwcTNRYzhZT3ExM2FnbmovUWRjc0ZYYTJnMjB1K1lYZDdoZ3krZksKWk4vVUkxQUQ0YzZyM1lma1ZWUmVHd1lxQVp1WXN2V0RKbW1GNWRwdEMzN011cDBPRUxVTExSakZJOTZXNXIwSAo1TmdPc25NWFJNV1hYVlpiNWRxT3R0SmRtS3FhZ25TZ1JQQVpQN2MwQjFQU2FqYzZjNGZRVXpNQ0F3RUFBVEFOCkJna3Foa2lHOXcwQkFRc0ZBQU9DQVFFQWpLb2tRdGRPcEsrTzhibWVPc3lySmdJSXJycVFVY2ZOUitjb0hZVUoKdGhrYnhITFMzR3VBTWI5dm15VExPY2xxeC9aYzJPblEwMEJCLzlTb0swcitFZ1U2UlVrRWtWcitTTFA3NTdUWgozZWI4dmdPdEduMS9ienM3bzNBaS9kclkrcUI5Q2k1S3lPc3FHTG1US2xFaUtOYkcyR1ZyTWxjS0ZYQU80YTY3Cklnc1hzYktNbTQwV1U3cG9mcGltU1ZmaXFSdkV5YmN3N0NYODF6cFErUyt1eHRYK2VBZ3V0NHh3VlI5d2IyVXYKelhuZk9HbWhWNThDd1dIQnNKa0kxNXhaa2VUWXdSN0diaEFMSkZUUkk3dkhvQXprTWIzbjAxQjQyWjNrN3RXNQpJUDFmTlpIOFUvOWxiUHNoT21FRFZkdjF5ZytVRVJxbStGSis2R0oxeFJGcGZnPT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
  tls.key: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBdi91RWM4b1JkMHUvZXVJTHNFK1RYZUprckxMMnNJNGFWaEMvYjVyYy9XMlRiNHEvClJOcktGMEdYaVN1eE9ycXgrajlnamx4NXFjdnhkenRKbXNFUkJ1Z1B0ME9hVGtIekhvb3FVWmcwZGxmZ1dkT0EKUTZMNTdlT1l0Q29VOUZ4amRXdzZUVVRJVUQ4R0JsRlNjSVo0b1hFTkhzbysyR3VTTWk2Zk1wTVM3YUhudzFtMApxWkdvRWEzWFNyZEJ6eGc2clhkcUNlUDlCMXl3VmRyYURiUzc1aGQzdUdETDU4cGszOVFqVUFQaHpxdmRoK1JWClZGNGJCaW9CbTVpeTlZTW1hWVhsMm0wTGZzeTZuUTRRdFFzdEdNVWozcGJtdlFmazJBNnljeGRFeFpkZFZsdmwKMm82MjBsMllxcHFDZEtCRThCay90elFIVTlKcU56cHpoOUJUTXdJREFRQUJBb0lCQVFDZklHbXowOHhRVmorNwpLZnZJUXQwQ0YzR2MxNld6eDhVNml4MHg4Mm15d1kxUUNlL3BzWE9LZlRxT1h1SENyUlp5TnUvZ2IvUUQ4bUFOCmxOMjRZTWl0TWRJODg5TEZoTkp3QU5OODJDeTczckM5bzVvUDlkazAvYzRIbjAzSkVYNzZ5QjgzQm9rR1FvYksKMjhMNk0rdHUzUmFqNjd6Vmc2d2szaEhrU0pXSzBwV1YrSjdrUkRWYmhDYUZhNk5nMUZNRWxhTlozVDhhUUtyQgpDUDNDeEFTdjYxWTk5TEI4KzNXWVFIK3NYaTVGM01pYVNBZ1BkQUk3WEh1dXFET1lvMU5PL0JoSGt1aVg2QnRtCnorNTZud2pZMy8yUytSRmNBc3JMTnIwMDJZZi9oY0IraVlDNzVWYmcydVd6WTY3TWdOTGQ5VW9RU3BDRkYrVm4KM0cyUnhybnhBb0dCQU40U3M0ZVlPU2huMVpQQjdhTUZsY0k2RHR2S2ErTGZTTXFyY2pOZjJlSEpZNnhubmxKdgpGenpGL2RiVWVTbWxSekR0WkdlcXZXaHFISy9iTjIyeWJhOU1WMDlRQ0JFTk5jNmtWajJTVHpUWkJVbEx4QzYrCk93Z0wyZHhKendWelU0VC84ajdHalRUN05BZVpFS2FvRHFyRG5BYWkyaW5oZU1JVWZHRXFGKzJyQW9HQkFOMVAKK0tZL0lsS3RWRzRKSklQNzBjUis3RmpyeXJpY05iWCtQVzUvOXFHaWxnY2grZ3l4b25BWlBpd2NpeDN3QVpGdwpaZC96ZFB2aTBkWEppc1BSZjRMazg5b2pCUmpiRmRmc2l5UmJYbyt3TFU4NUhRU2NGMnN5aUFPaTVBRHdVU0FkCm45YWFweUNweEFkREtERHdObit3ZFhtaTZ0OHRpSFRkK3RoVDhkaVpBb0dCQUt6Wis1bG9OOTBtYlF4VVh5YUwKMjFSUm9tMGJjcndsTmVCaWNFSmlzaEhYa2xpSVVxZ3hSZklNM2hhUVRUcklKZENFaHFsV01aV0xPb2I2NTNyZgo3aFlMSXM1ZUtka3o0aFRVdnpldm9TMHVXcm9CV2xOVHlGanIrSWhKZnZUc0hpOGdsU3FkbXgySkJhZUFVWUNXCndNdlQ4NmNLclNyNkQrZG8wS05FZzFsL0FvR0FlMkFVdHVFbFNqLzBmRzgrV3hHc1RFV1JqclRNUzRSUjhRWXQKeXdjdFA4aDZxTGxKUTRCWGxQU05rMXZLTmtOUkxIb2pZT2pCQTViYjhibXNVU1BlV09NNENoaFJ4QnlHbmR2eAphYkJDRkFwY0IvbEg4d1R0alVZYlN5T294ZGt5OEp0ek90ajJhS0FiZHd6NlArWDZDODhjZmxYVFo5MWpYL3RMCjF3TmRKS2tDZ1lCbyt0UzB5TzJ2SWFmK2UwSkN5TGhzVDQ5cTN3Zis2QWVqWGx2WDJ1VnRYejN5QTZnbXo5aCsKcDNlK2JMRUxwb3B0WFhNdUFRR0xhUkcrYlNNcjR5dERYbE5ZSndUeThXczNKY3dlSTdqZVp2b0ZpbmNvVlVIMwphdmxoTUVCRGYxSjltSDB5cDBwWUNaS2ROdHNvZEZtQktzVEtQMjJhTmtsVVhCS3gyZzR6cFE9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress 
  namespace: nginx-ingress
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-config
  namespace: nginx-ingress
data:
---
# Described at: https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/
# Source from: https://github.com/nginxinc/kubernetes-ingress/blob/master/deployments/common/ingress-class.yaml
apiVersion: networking.k8s.io/v1beta1
kind: IngressClass
metadata:
  name: nginx
  # annotations:
  #   ingressclass.kubernetes.io/is-default-class: "true"
spec:
  controller: nginx.org/ingress-controller
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress
  namespace: nginx-ingress
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-ingress
  template:
    metadata:
      labels:
        app: nginx-ingress
    spec:
      serviceAccountName: nginx-ingress
      containers:
      - image: nginx/nginx-ingress:edge
        imagePullPolicy: Always
        name: nginx-ingress
        ports:
        - name: http
          containerPort: 80
        - name: https
          containerPort: 443
        env:
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        args:
          - -nginx-configmaps=$(POD_NAMESPACE)/nginx-config
          - -default-server-tls-secret=$(POD_NAMESPACE)/default-server-secret
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-ingress
  namespace: nginx-ingress
spec:
  type: NodePort 
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
  selector:
    app: nginx-ingress
  externalIPs:
    - 172.17.0.88
```
`Ingress` 控制器以熟悉的方式部署到其他 Kubernetes 对象

```bash
controlplane $ kubectl create -f ingress.yaml
namespace/nginx-ingress created
secret/default-server-secret created
serviceaccount/nginx-ingress created
configmap/nginx-config created
ingressclass.networking.k8s.io/nginx created
deployment.apps/nginx-ingress created
service/nginx-ingress created

controlplane $ kubectl get deployment -n nginx-ingress
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
nginx-ingress   0/1     1            0           4s
```
###  3.3 Deploy Ingress Rules
入口规则是 Kubernetes 的对象类型。规则可以基于请求主机（域），或请求的路径，或两者的组合。

```bash
controlplane $ cat ingress-rules.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: webapp-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: my.kubernetes.example
    http:
      paths:
      - path: /webapp1
        backend:
          serviceName: webapp1-svc
          servicePort: 80
      - path: /webapp2
        backend:
          serviceName: webapp2-svc
          servicePort: 80
      - backend:
          serviceName: webapp3-svc
          servicePort: 80
```
规则的重要部分定义如下。

这些规则适用于对主机`my.kubernetes.example` 的请求。基于路径请求定义了两个规则，并使用一个 `catch all` 定义。对路径`/webapp1` 的请求被转发到服务`webapp1-svc` 上。同样，对`/webapp2`的请求被转发到`webapp2-svc`。如果没有规则适用，将使用`webapp3-svc`。

这演示了应用程序的 URL 结构如何独立于应用程序的部署方式。

```bash
controlplane $ kubectl create -f ingress-rules.yaml
ingress.extensions/webapp-ingress created
controlplane $ kubectl get ing
NAME             CLASS   HOSTS                   ADDRESS   PORTS   AGE
webapp-ingress   nginx   my.kubernetes.example             80      2s
```
###  3.4 测试
应用入口规则后，流量将被路由到定义的位置。

第一个请求将由webapp1部署处理。

```bash
curl -H "Host: my.kubernetes.example" 172.17.0.88/webapp1
```

第二个请求将由webapp2部署处理。

```bash
curl -H "Host: my.kubernetes.example" 172.17.0.88/webapp2
```

最后，所有其他请求将由webapp3部署处理。

```bash
curl -H "Host: my.kubernetes.example" 172.17.0.88
```

##  4.  Liveness and Readiness Healthchecks
在此场景中，您将了解 Kubernetes 如何使用 `Readiness and Liveness Probes` 检查容器运行状况。

`Readiness Probes` 检查应用程序是否准备好开始处理流量。此探针解决了容器已启动的问题，但该进程仍在预热和配置自身，这意味着它尚未准备好接收流量。

`Liveness Probes` 确保应用程序健康并能够处理请求。

###  4.1 创建http应用程序

```bash
controlplane $ cat deploy.yaml 
kind: List
apiVersion: v1
items:
- kind: ReplicationController
  apiVersion: v1
  metadata:
    name: frontend
    labels:
      name: frontend
  spec:
    replicas: 1
    selector:
      name: frontend
    template:
      metadata:
        labels:
          name: frontend
      spec:
        containers:
        - name: frontend
          image: katacoda/docker-http-server:health
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 1
            timeoutSeconds: 1
          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 1
            timeoutSeconds: 1
- kind: ReplicationController
  apiVersion: v1
  metadata:
    name: bad-frontend
    labels:
      name: bad-frontend
  spec:
    replicas: 1
    selector:
      name: bad-frontend
    template:
      metadata:
        labels:
          name: bad-frontend
      spec:
        containers:
        - name: bad-frontend
          image: katacoda/docker-http-server:unhealthy
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 1
            timeoutSeconds: 1
          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 1
            timeoutSeconds: 1
- kind: Service
  apiVersion: v1
  metadata:
    labels:
      app: frontend
      kubernetes.io/cluster-service: "true"
    name: frontend
  spec:
    type: NodePort
    ports:
    - port: 80
      nodePort: 30080
    selector:
      app: frontend
```

```bash
controlplane $ kubectl apply -f deploy.yaml
replicationcontroller/frontend created
replicationcontroller/bad-frontend created
service/frontend created
```
###  4.2 Readiness Probe
在部署集群时，还部署了两个 Pod 来演示健康检查。

```bash
controlplane $ cat deploy.yaml
kind: List
apiVersion: v1
items:
- kind: ReplicationController
  apiVersion: v1
  metadata:
    name: frontend
    labels:
      name: frontend
  spec:
    replicas: 1
    selector:
      name: frontend
    template:
      metadata:
        labels:
          name: frontend
      spec:
        containers:
        - name: frontend
          image: katacoda/docker-http-server:health
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 1
            timeoutSeconds: 1
          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 1
            timeoutSeconds: 1
- kind: ReplicationController
  apiVersion: v1
  metadata:
    name: bad-frontend
    labels:
      name: bad-frontend
  spec:
    replicas: 1
    selector:
      name: bad-frontend
    template:
      metadata:
        labels:
          name: bad-frontend
      spec:
        containers:
        - name: bad-frontend
          image: katacoda/docker-http-server:unhealthy
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 1
            timeoutSeconds: 1
          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 1
            timeoutSeconds: 1
- kind: Service
  apiVersion: v1
  metadata:
    labels:
      app: frontend
      kubernetes.io/cluster-service: "true"
    name: frontend
  spec:
    type: NodePort
    ports:
    - port: 80
      nodePort: 30080
    selector:
      app: frontend
```
部署 Replication Controller 时，每个 Pod 都有一个 Readiness 和 Liveness 检查。每个检查都具有以下格式，用于通过 HTTP 执行健康检查。

```bash
livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 1
  timeoutSeconds: 1
```
可以根据您的应用程序更改设置以调用不同的端点，例如 /ping。

第一个 Pod，`bad-frontend`是一个 HTTP 服务，它总是返回 `500` 错误，表明它没有正确启动。您可以使用以下命令查看 Pod 的状态

```bash
controlplane $ kubectl get pods --selector="name=bad-frontend"
NAME                 READY   STATUS             RESTARTS   AGE
bad-frontend-5p4k6   0/1     CrashLoopBackOff   4          2m55s
```
Kubectl 将返回使用我们的特定标签部署的 Pod。因为健康检查失败，它会说零容器已准备就绪。它还将指示容器的重启尝试次数。

```bash
controlplane $ kubectl describe pod $pod
Name:               bad-frontend-5p4k6
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               controlplane/172.17.0.44
Start Time:         Tue, 09 Nov 2021 15:44:19 +0000
Labels:             name=bad-frontend
Annotations:        <none>
Status:             Running
IP:                 10.32.0.6
Controlled By:      ReplicationController/bad-frontend
Containers:
  bad-frontend:
    Container ID:   docker://ae3c84bfdaa178fe2976e8b075e4e98da95df06b6f5bd85ef2eb5f92466c5f5d
    Image:          katacoda/docker-http-server:unhealthy
    Image ID:       docker-pullable://katacoda/docker-http-server@sha256:bea95c69c299c690103c39ebb3159c39c5061fee1dad13aa1b0625e0c6b52f22
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Tue, 09 Nov 2021 15:47:44 +0000
    Last State:     Terminated
      Reason:       Error
      Exit Code:    2
      Started:      Tue, 09 Nov 2021 15:46:34 +0000
      Finished:     Tue, 09 Nov 2021 15:47:01 +0000
    Ready:          False
    Restart Count:  5
    Liveness:       http-get http://:80/ delay=1s timeout=1s period=10s #success=1 #failure=3
    Readiness:      http-get http://:80/ delay=1s timeout=1s period=10s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-h7qch (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  default-token-h7qch:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-h7qch
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason     Age                    From                   Message
  ----     ------     ----                   ----                   -------
  Normal   Scheduled  3m31s                  default-scheduler      Successfully assigned default/bad-frontend-5p4k6 to controlplane
  Normal   Pulling    3m21s                  kubelet, controlplane  Pulling image "katacoda/docker-http-server:unhealthy"
  Normal   Pulled     3m11s                  kubelet, controlplane  Successfully pulled image "katacoda/docker-http-server:unhealthy"
  Normal   Killing    2m19s (x2 over 2m49s)  kubelet, controlplane  Container bad-frontend failed liveness probe, will be restarted
  Normal   Created    2m18s (x3 over 3m11s)  kubelet, controlplane  Created container bad-frontend
  Normal   Pulled     2m18s (x2 over 2m48s)  kubelet, controlplane  Container image "katacoda/docker-http-server:unhealthy" already present on machine
  Normal   Started    2m16s (x3 over 3m10s)  kubelet, controlplane  Started container bad-frontend
  Warning  Unhealthy  2m5s (x5 over 3m5s)    kubelet, controlplane  Readiness probe failed: HTTP probe failed with statuscode: 500
  Warning  Unhealthy  119s (x8 over 3m9s)    kubelet, controlplane  Liveness probe failed: HTTP probe failed with statuscode: 500
```
我们的第二个 Pod，frontend，在启动时返回 OK 状态。

```c
controlplane $ kubectl get pods --selector="name=frontend"
NAME             READY   STATUS    RESTARTS   AGE
frontend-d29h8   1/1     Running   0          4m3s
```
###  4.3 Liveness Probe
由于我们的第二个 Pod 当前处于健康状态，我们可以模拟发生的故障。

目前，应该没有发生崩溃。

```bash
controlplane $ kubectl get pods --selector="name=frontend"
NAME             READY   STATUS    RESTARTS   AGE
frontend-d29h8   1/1     Running   0          4m35s
```
崩溃服务
HTTP 服务器有一个额外的端点，这将导致它返回 500 个错误。使用kubectl exec可以调用端点。

```bash
controlplane $ pod=$(kubectl get pods --selector="name=frontend" --output=jsonpath={.items..metadata.name})
controlplane $ kubectl exec $pod -- /usr/bin/curl -s localhost/unhealthy
```
Kubernetes 将根据配置执行 Liveness Probe。如果探测器失败，Kubernetes 将销毁并重新创建失败的容器。执行上面的命令使服务崩溃并观察 Kubernetes 自动恢复它。

```c
controlplane $ kubectl get pods --selector="name=frontend"
NAME             READY   STATUS    RESTARTS   AGE
frontend-d29h8   1/1     Running   1          5m56s
```
检查可能需要一些时间才能检测到。
