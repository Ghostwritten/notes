#  Kubernetes 应用部署实践
tags: 实践
![](https://i-blog.csdnimg.cn/blog_migrate/f7deda649d4bbe60ebb5e77cd035d70d.png)




## 1. 架构介绍

来自[lyzhang1999](https://github.com/lyzhang1999/kubernetes-example)设计的这个示例应用是一套微服务架构的应用，你可以在 [GitHub 上获取源码](https://github.com/Ghostwritten/kubernetes-example.git)，源码目录结构如下：

```bash
$ ls
backend  deploy   frontend
```
在这里，`backend` 目录为后端源码，`frontend` 目录为前端源码，`deploy` 目录是应用的 `K8s Manifest`，前后端都已经包含构建镜像所需的 `Dockerfile`。

示例应用由三个服务组成：
1. 前端；
2. 后端；
3. 数据库。

其中，前端采用 `React` 编写，它也是应用对外提供服务的入口；后端由 `Python` 编写；数据库采用流行的 `Postgres`。应用整体架构如下图所示：
![](https://i-blog.csdnimg.cn/blog_migrate/6c20a7798f4c298c4446a853e778624f.png)
端实现了三个功能，分别是存储输入的内容，列出输入内容记录以及删除所有的记录。这三个功能分别对应了后端的三个接口，也就是 `/add`, `/fetch` 和 `/delete`，最后数据会被存储在 `Postgres` 数据库中。

示例应用的前端界面如下图所示：
![](https://i-blog.csdnimg.cn/blog_migrate/13b160dd8b7f95d68b66fe6bec267a25.png)


为了方便你把示例应用直接部署到 K8s 集群内，我已经写好了 K8s Manifest 文件，你可以在 [GitHub](https://github.com/lyzhang1999/kubernetes-example/tree/main/deploy) 找到这些清单文件。应用的 K8s 部署架构图如下：
![](https://i-blog.csdnimg.cn/blog_migrate/bb8857db6f7179770d162ac9c910305f.png)
在这张架构图中，`Ingress` 是应用的入口，Ingress 会根据请求路径将流量分流至前后端的 Service 中，然后 Service 将请求转发给前后端 Pod 进行业务逻辑处理，后端的工作负载 `Deployment` 配置了 `HPA` 自动横向扩容。同时，Postgres 也是以 Deployment 的方式部署到集群内的。最后，所有资源都部署在 K8s 的 `example` 命名空间（Namespace）下。



## 2. 创建新的 K8s 集群
我们还是以部署到本地 Kind 集群为例，为了避免资源冲突，需要先把第一章实验过程创建的 Kind 集群删掉。你可以用 kind delete cluster 来删除集群：

```bash
kind delete cluster
```
然后，重新创建一个 K8s 集群，将下面的内容保存为 `config.yaml`：

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
接下来，使用 `kind create cluster` 重新创建集群：

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

Have a question, bug, or feature request? Let us know! https://kind.sigs.k8s.io/#community 🙂
```
由于示例应用使用了 `Ingress`，所以我们需要为 Kind 部署 `Ingress`，你可以使用 kubectl apply -f 来部署 Ingress-Nginx：

```bash
kubectl create -f https://ghproxy.com/https://raw.githubusercontent.com/lyzhang1999/resource/main/ingress-nginx/ingress-nginx.yaml
```
最后，再部署 `Metric Server`，以便开启 `HPA` 功能：

```bash
kubectl apply -f https://ghproxy.com/https://raw.githubusercontent.com/lyzhang1999/resource/main/metrics/metrics.yaml
```
准备好新的 K8s 集群后，就可以开始部署示例应用了。

## 3. 部署示例应用
创建命名空间：

```bash
kubectl create namespace example
```
创建 Postgres 数据库


```bash
$ kubectl apply -f https://ghproxy.com/https://raw.githubusercontent.com/lyzhang1999/kubernetes-example/main/deploy/database.yaml -n example
configmap/pg-init-script created
deployment.apps/postgres created
service/pg-service created
```
创建前后端 `Deployment` 工作负载和 `Service`

```bash
$ kubectl apply -f https://ghproxy.com/https://raw.githubusercontent.com/lyzhang1999/kubernetes-example/main/deploy/frontend.yaml -n example
deployment.apps/frontend created
service/frontend-service created

$ kubectl apply -f https://ghproxy.com/https://raw.githubusercontent.com/lyzhang1999/kubernetes-example/main/deploy/backend.yaml -n example
deployment.apps/backend created
service/backend-service created
```
接下来，为应用创建 `Ingress` 和 `HPA` 策略：

```bash
$ kubectl apply -f https://ghproxy.com/https://raw.githubusercontent.com/lyzhang1999/kubernetes-example/main/deploy/ingress.yaml -n example
ingress.networking.k8s.io/frontend-ingress created

$ kubectl apply -f https://ghproxy.com/https://raw.githubusercontent.com/lyzhang1999/kubernetes-example/main/deploy/hpa.yaml -n example
horizontalpodautoscaler.autoscaling/backend created
```
其实，除了可以按照上面的引导单独创建示例应用的每一个 K8s 对象以外，我们还可以使用另一种方法，那就是把这个 `Git` 仓库克隆到本地，然后使用 `kubectl apply` 一次性将所有示例应用的对象部署到集群内：

```bash
$ git clone https://ghproxy.com/https://github.com/lyzhang1999/kubernetes-example && cd kubernetes-example
Cloning into 'kubernetes-example'...
......
Resolving deltas: 100% (28/28), done.

$ kubectl apply -f deploy -n example
deployment.apps/backend created
service/backend-service created
configmap/pg-init-script created
......
```
这里的 `-f` 参数除了可以指定文件外，还可以指定目录，kubectl 将会检查目录下所有可用的 `Manifest`，然后把它部署到 K8s 集群。-n 参数代表将所有 Manifest 部署到 example 命名空间。

最后，我们可以使用 `kubectl wait` 来检查所有资源是不是已经处于 `Ready` 状态了：

```bash
$ kubectl wait --for=condition=Ready pods --all -n example
pod/backend-9b677898b-n5lsm condition met
pod/frontend-f948bdc85-q6x9f condition met
pod/postgres-7745b57d5d-f4trt condition met
```
到这里，示例应用就部署完了。

访问 `127.0.0.1`，你应该能看到示例应用的前端界面，如下图所示：
![](https://i-blog.csdnimg.cn/blog_migrate/197290000a0b96105a6c8bfe2d0a1fda.png)
可以尝试在输出框中输入内容，如果点击 `Add` 按钮，下方的列表内会出现你输入的内容，点击 `Clear` 所有内容被清空，这就说明应用已经可以正常工作了。

## 4. K8s 对象解析
首先，查看某个命名空间下的所有资源：

```bash
❯ kubectl get all -n example
NAME                            READY   STATUS    RESTARTS   AGE
pod/backend-648ff85f48-8qgjg    1/1     Running   0          29s
pod/backend-648ff85f48-f845h    1/1     Running   0          51s
pod/frontend-7b55cc5c67-4svjz   1/1     Running   0          14s
pod/frontend-7b55cc5c67-9cx57   1/1     Running   0          14s
pod/postgres-7745b57d5d-f4trt   1/1     Running   0          44m

NAME                       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/backend-service    ClusterIP   10.96.244.140   <none>        5000/TCP   42m
service/frontend-service   ClusterIP   10.96.85.54     <none>        3000/TCP   43m
service/pg-service         ClusterIP   10.96.166.74    <none>        5432/TCP   44m

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/backend    2/2     2            2           42m
deployment.apps/frontend   2/2     4            4           43m
deployment.apps/postgres   1/1     1            1           44m

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/backend-648ff85f48    2         2         2       51s
replicaset.apps/frontend-7b55cc5c67   2         2         2       54s
replicaset.apps/postgres-7745b57d5d   1         1         1       44m

NAME                                           REFERENCE             TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/backend    Deployment/backend    0%/50%     2         10        2          8m17s
horizontalpodautoscaler.autoscaling/frontend   Deployment/frontend   51%/80%   2         10        10         8m17s
```
`-n` 参数表示指定一个命名空间，从返回结果可以看出，示例应用一共创建了 5 个 Pod、3 个 Service、3 个 Deployment、3 个 Replicaset、2 个 HPA。


