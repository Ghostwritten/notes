#  Kubernetes Deployment Overview
tags: Deployment

[![在这里插入图片描述](https://img-blog.csdnimg.cn/b02019d23d04457a989df5b758e1fccc.png)](https://www.rottentomatoes.com/m/one_flew_over_the_cuckoos_nest)





 - [Kubernetes Deployment【1】管理与编排入门](https://ghostwritten.blog.csdn.net/article/details/121510015)
 - [Kubernetes Deployment【2】原理深入](https://ghostwritten.blog.csdn.net/article/details/121525931)
 - [Kubernetes Deployment【3】管理高级技巧](https://ghostwritten.blog.csdn.net/article/details/121562269)
 - [Kubernetes Deployment【4】参数大全](https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs/resources/deployment)
 - [Kubernetes Deployment【5】部署策略](https://ghostwritten.blog.csdn.net/article/details/121587974)
 - [Kubernetes Deployment【6】client-go 开发](https://ghostwritten.blog.csdn.net/article/details/110358813)


Deployment 结构示意图
![在这里插入图片描述](https://img-blog.csdnimg.cn/5da8ae04889d4330b89c03c04462e5c0.png?)


deployment排查拓扑图
![在这里插入图片描述](https://img-blog.csdnimg.cn/43652c6a0da54d31afe3aa11ee5ca1aa.png)

---
##  1. 简介
一个 Deployment 为 Pods 和 ReplicaSets 提供声明式的更新能力。

你负责描述 Deployment 中的 目标状态，而 Deployment 控制器（Controller） 以受控速率更改实际状态， 使其变为期望状态。你可以定义 Deployment 以创建新的 ReplicaSet，或删除现有 Deployment， 并通过新的 Deployment 收养其资源。

## 2. 创建 Deployment
下面是 Deployment 示例。其中创建了一个 ReplicaSet，负责启动三个 nginx Pods：
### 2.1 编排yaml
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```
### 2.2 定义描述
 - 创建名为 `nginx-deployment`（由 `.metadata.name` 字段标明）的 `Deployment`。
 - 该 `Deployment` 创建三个（由 `replicas` 字段标明）`Pod` 副本。
 - `selector` 字段定义 Deployment 如何查找要管理的 Pods。 在这里，你选择在 Pod 模板中定义的标签（app: nginx）。 不过，更复杂的选择规则是也可能的，只要 Pod 模板本身满足所给规则即可。
 - `spec.selector.matchLabels` 字段是 `{key,value}` 键值对映射。 在 `matchLabels` 映射中的每个 `{key,value}` 映射等效于 `matchExpressions` 中的一个元素， 即其 `key` 字段是 `“key”`，`operator` 为 `“In”`，`values` 数组仅包含 `“value”`。 在 `matchLabels` 和 `matchExpressions` 中给出的所有条件都必须满足才能匹配。
 - `template` 字段包含以下子字段：
   - `Pod` 被使用 `labels` 字段打上 `app: nginx` 标签。
   - `Pod` 模板规约（即 `.template.spec` 字段）指示 `Pods` 运行一个 `nginx` 容器， 该容器运行版本为 `1.14.2` 的 `nginx Docker Hub`镜像。
   - 创建一个容器并使用 `name` 字段将其命名为 `nginx`。

### 2.3 运行与查看

```bash
$ kubectl apply -f nginx-deployment.yaml
$ kubectl get deployments
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         0         0            0           1s
```

> 说明： 你可以设置 `--record` 标志将所执行的命令写入资源注解 `kubernetes.io/change-cause` 中。这对于以后的检查是有用的。例如，要查看针对每个 Deployment 修订版本所执行过的命令。

在检查集群中的 `Deployment` 时，所显示的字段有：

 - `NAME` 列出了集群中 `Deployment` 的名称。
 - `READY` 显示应用程序的可用的 副本 数。显示的模式是“就绪个数/期望个数”。
 - `UP-TO-DATE` 显示为了达到期望状态已经更新的副本数。
 - `AVAILABLE` 显示应用可供用户使用的副本数。
 - `AGE` 显示应用程序运行的时间。

请注意期望副本数是根据 `.spec.replicas` 字段设置 3。

查看 Deployment 上线状态

```bash
$ kubectl rollout status deployment/nginx-deployment
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
deployment "nginx-deployment" successfully rolled out
```
几秒钟后再次运行

```bash
#注意 Deployment 已创建全部三个副本，并且所有副本都是最新的（它们包含最新的 Pod 模板） 并且可用
$ kubectl get deployments
AME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         3         3            3           18s
```
查看 Deployment 创建的 ReplicaSet（rs)

```bash
$ kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-75675f5897   3         3         3       18s
```
`ReplicaSet` 输出中包含以下字段：

 - `NAME` 列出名字空间中 ReplicaSet 的名称；
 - `DESIRED` 显示应用的期望副本个数，即在创建 Deployment 时所定义的值。 此为期望状态；
 - `CURRENT` 显示当前运行状态中的副本个数；
 - `READY` 显示应用中有多少副本可以为用户提供服务；
 - `AGE` 显示应用已经运行的时间长度。

注意 `ReplicaSet` 的名称始终被格式化为`[Deployment名称]-[随机字符串]`。 其中的随机字符串是使用 `pod-template-hash` 作为种子随机生成的。

`Deployment` 控制器将 `pod-template-hash` 标签添加到 `Deployment` 所创建或收留的 每个 `ReplicaSet` 。此标签可确保 `Deployment` 的子 `ReplicaSets` 不重叠。 标签是通过对 `ReplicaSet` 的 `PodTemplate` 进行哈希处理。 所生成的哈希值被添加到 `ReplicaSet` 选择算符、Pod 模板标签，并存在于在 ReplicaSet 可能拥有的任何现有 Pod 中。

要查看每个 Pod 自动生成的标签，运行

```bash
$ kubectl get pods --show-labels
NAME                                READY     STATUS    RESTARTS   AGE       LABELS
nginx-deployment-75675f5897-7ci7o   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
nginx-deployment-75675f5897-kzszj   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
nginx-deployment-75675f5897-qqcnn   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453

$ k expose deployment nginx-deployment 80
$ k get svc
NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
kubernetes              ClusterIP      10.96.0.1        <none>        443/TCP                      51d
nginx-deployment        ClusterIP      10.101.106.248   <none>        80/TCP                       8s
$ curl 10.101.106.248:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
### 2.4 Pod-template-hash label
注意：这个 label 不是用户指定的！

注意上面示例输出中的 pod label 里的 `pod-template-hash label`。当 Deployment 创建或者接管 `ReplicaSet` 时，Deployment controller 会自动为 Pod 添加 `pod-template-hash label`。这样做的目的是防止 Deployment 的子 ReplicaSet 的 pod 名字重复。通过将 ReplicaSet 的 PodTemplate 进行哈希散列，使用生成的哈希值作为 label 的值，并添加到 `ReplicaSet selector` 里、 `pod template label` 和 `ReplicaSet` 管理中的 Pod 上。

##  3. 更新 Deployment 

> 说明： 仅当 Deployment Pod 模板（即 `.spec.template`）发生改变时，例如模板的标签或容器镜像被更新， 才会触发Deployment 上线。 其他更新（如对 Deployment 执行扩缩容的操作）不会触发上线动作。

按照以下步骤更新 `Deployment`：
先来更新 `nginx Pod` 以使用 `nginx:1.16.1` 镜像，而不是 `nginx:1.14.2` 镜像。

```bash
$ kubectl --record deployment.apps/nginx-deployment set image \
   deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1
deployment.apps/nginx-deployment image updated

或者

$ kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1 --record
deployment.apps/nginx-deployment image updated
```
或者，可以 `edit Deployment` 并将 `.spec.template.spec.containers[0].image` 从 `nginx:1.14.2` 更改至 `nginx:1.16.1`。

```c
$ kubectl edit deployment.v1.apps/nginx-deployment
deployment.apps/nginx-deployment edited
```
要查看上线状态，运行：

```bash
$ kubectl rollout status deployment/nginx-deployment
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
or
deployment "nginx-deployment" successfully rolled out


$ kubectl get deployments
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         3         3            3           36s

$ kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-1564180365   3         3         3       6s
nginx-deployment-2035384211   0         0         0       36s

$ kubectl get pods
NAME                                READY     STATUS    RESTARTS   AGE
nginx-deployment-1564180365-khku8   1/1       Running   0          14s
nginx-deployment-1564180365-nacti   1/1       Running   0          14s
nginx-deployment-1564180365-z9gth   1/1       Running   0          14s

$ kubectl describe deployments
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Thu, 30 Nov 2017 10:56:25 +0000
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision=2
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
   Containers:
    nginx:
      Image:        nginx:1.16.1
      Port:         80/TCP
      Environment:  <none>
      Mounts:       <none>
    Volumes:        <none>
  Conditions:
    Type           Status  Reason
    ----           ------  ------
    Available      True    MinimumReplicasAvailable
    Progressing    True    NewReplicaSetAvailable
  OldReplicaSets:  <none>
  NewReplicaSet:   nginx-deployment-1564180365 (3/3 replicas created)
  Events:
    Type    Reason             Age   From                   Message
    ----    ------             ----  ----                   -------
    Normal  ScalingReplicaSet  2m    deployment-controller  Scaled up replica set nginx-deployment-2035384211 to 3
    Normal  ScalingReplicaSet  24s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 1
    Normal  ScalingReplicaSet  22s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 2
    Normal  ScalingReplicaSet  22s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 2
    Normal  ScalingReplicaSet  19s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 1
    Normal  ScalingReplicaSet  19s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 3
    Normal  ScalingReplicaSet  14s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 0
```
下次要更新这些 Pods 时，只需再次更新 Deployment Pod 模板即可。

`Deployment` 可确保在更新时仅关闭一定数量的 Pod。默认情况下，它确保至少所需 Pods 75% 处于运行状态（最大不可用比例为 25%）。

`Deployment` 还确保仅所创建 Pod 数量只可能比期望 Pods 数高一点点。 默认情况下，它可确保启动的 Pod 个数比期望个数最多多出 25%（最大峰值 25%）。

例如，如果仔细查看上述 Deployment ，将看到它首先创建了一个新的 Pod，然后删除了一些旧的 Pods， 并创建了新的 Pods。它不会杀死老 Pods，直到有足够的数量新的 Pods 已经出现。 在足够数量的旧 Pods 被杀死前并没有创建新 Pods。它确保至少 2 个 Pod 可用，同时 最多总共 4 个 Pod 可用。

可以看到，当第一次创建 `Deployment` 时，它创建了一个 `ReplicaSet`（`nginx-deployment-2035384211`） 并将其直接扩容至 3 个副本。更新 Deployment 时，它创建了一个新的 ReplicaSet （nginx-deployment-1564180365），并将其扩容为 1，然后将旧 ReplicaSet 缩容到 2， 以便至少有 2 个 Pod 可用且最多创建 4 个 Pod。 然后，它使用相同的滚动更新策略继续对新的 ReplicaSet 扩容并对旧的 ReplicaSet 缩容。 最后，你将有 3 个可用的副本在新的 ReplicaSet 中，旧 ReplicaSet 将缩容到 0。

### 3.1 Rollover（多个 rollout 并行）
每当 `Deployment controller` 观测到有新的 `deployment` 被创建时，如果没有已存在的 `ReplicaSet` 来创建期望个数的 Pod 的话，就会创建出一个新的 `ReplicaSet` 来做这件事。已存在的 `ReplicaSet` 控制 `label` 与 `.spec.selector` 匹配但是 `template` 跟 `.spec.template` 不匹配的 Pod 缩容。最终，新的 ReplicaSet 将会扩容出 `.spec.replicas` 指定数目的 Pod，旧的 `ReplicaSet` 会缩容到 0。

如果您更新了一个的已存在并正在进行中的 Deployment，每次更新 Deployment 都会创建一个新的 ReplicaSet 并扩容它，同时回滚之前扩容的 ReplicaSet —— 将它添加到旧的 ReplicaSet 列表中，开始缩容。

例如，假如您创建了一个有 5 个 `niginx:1.7.9 replica` 的 Deployment，但是当还只有 3 个 `nginx:1.7.9` 的 `replica` 创建出来的时候您就开始更新含有 5 个 `nginx:1.9.1 replica` 的 `Deployment`。在这种情况下，`Deployment` 会立即杀掉已创建的 3 个 `nginx:1.7.9` 的 Pod，并开始创建 `nginx:1.9.1` 的 Pod。它不会等到所有的 5 个 `nginx:1.7.9` 的 Pod 都创建完成后才开始改变航道。

###  Label selector 更新
我们通常不鼓励更新 `label selector`，我们建议事先规划好您的 selector。

任何情况下，只要您想要执行 `label selector` 的更新，请一定要谨慎并确认您已经预料到所有可能因此导致的后果。

增添 `selector` 需要同时在 `Deployment` 的 `spec` 中更新新的 `label`，否则将返回校验错误。此更改是不可覆盖的，这意味着新的 `selector` 不会选择使用旧 selector 创建的 `ReplicaSet` 和 Pod，从而导致所有旧版本的 `ReplicaSet` 都被丢弃，并创建新的 ReplicaSet。
更新 selector，即更改 `selector key` 的当前值，将导致跟增添 `selector` 同样的后果。
删除 `selector`，即删除 `Deployment selector` 中的已有的 `key`，不需要对 `Pod template label` 做任何更改，现有的 `ReplicaSet` 也不会成为孤儿，但是请注意，删除的 label 仍然存在于现有的 Pod 和 ReplicaSet 中。

## 4. 回滚 Deployment 
有时，你可能想要回滚 Deployment；例如，当 Deployment 不稳定时（例如进入反复崩溃状态）。 默认情况下，Deployment 的所有上线记录都保留在系统中，以便可以随时回滚 （你可以通过修改修订历史记录限制来更改这一约束）。

Deployment 被触发上线时，系统就会创建 Deployment 的新的修订版本。 这意味着仅当 Deployment 的 Pod 模板（.spec.template）发生更改时，才会创建新修订版本 -- 例如，模板的标签或容器镜像发生变化。 其他更新，如 Deployment 的扩缩容操作不会创建 Deployment 修订版本。 这是为了方便同时执行手动缩放或自动缩放。 换言之，当你回滚到较早的修订版本时，只有 Deployment 的 Pod 模板部分会被回滚。

假设你在更新 Deployment 时犯了一个拼写错误，将镜像名称命名设置为 `nginx:1.161` 而不是 `nginx:1.16.1`：

```bash
$ kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.161 --record=true
deployment.apps/nginx-deployment image updated
```

此上线进程会出现停滞。你可以通过检查上线状态来验证：

```bash
$ kubectl rollout status deployment/nginx-deployment
Waiting for rollout to finish: 1 out of 3 new replicas have been updated...
```
你可以看到旧的副本有两个（`nginx-deployment-1564180365` 和 `nginx-deployment-2035384211`）， 新的副本有 1 个（`nginx-deployment-3066724191`）：

```bash
$ kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-1564180365   3         3         3       25s
nginx-deployment-2035384211   0         0         0       36s
nginx-deployment-3066724191   1         1         0       6s
```
查看所创建的 Pod，你会注意到新 ReplicaSet 所创建的 1 个 Pod 卡顿在镜像拉取循环中

```bash
$ kubectl get pods
NAME                                READY     STATUS             RESTARTS   AGE
nginx-deployment-1564180365-70iae   1/1       Running            0          25s
nginx-deployment-1564180365-jbqqo   1/1       Running            0          25s
nginx-deployment-1564180365-hysrc   1/1       Running            0          25s
nginx-deployment-3066724191-08mng   0/1       ImagePullBackOff   0          6s


$ kubectl describe deployment
Name:           nginx-deployment
Namespace:      default
CreationTimestamp:  Tue, 15 Mar 2016 14:48:04 -0700
Labels:         app=nginx
Selector:       app=nginx
Replicas:       3 desired | 1 updated | 4 total | 3 available | 1 unavailable
StrategyType:       RollingUpdate
MinReadySeconds:    0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.91
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    ReplicaSetUpdated
OldReplicaSets:     nginx-deployment-1564180365 (3/3 replicas created)
NewReplicaSet:      nginx-deployment-3066724191 (1/1 replicas created)
Events:
  FirstSeen LastSeen    Count   From                    SubobjectPath   Type        Reason              Message
  --------- --------    -----   ----                    -------------   --------    ------              -------
  1m        1m          1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-2035384211 to 3
  22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 1
  22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 2
  22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 2
  21s       21s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 1
  21s       21s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 3
  13s       13s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 0
  13s       13s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-3066724191 to 1
```

> 说明： Deployment 控制器自动停止有问题的上线过程，并停止对新的 ReplicaSet 扩容。 这行为取决于所指定的`rollingUpdate` 参数（具体为 `maxUnavailable`）。 默认情况下，Kubernetes 将此值设置为 25%。


### 4.1 检查 Deployment 上线历史
首先，检查 Deployment 修订历史：

```bash
kubectl rollout history deployment.v1.apps/nginx-deployment
deployments "nginx-deployment"
REVISION    CHANGE-CAUSE
1           kubectl apply --filename=https://k8s.io/examples/controllers/nginx-deployment.yaml --record=true
2           kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.9.1 --record=true
3           kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.91 --record=true
```
`CHANGE-CAUSE` 的内容是从 `Deployment` 的 `kubernetes.io/change-cause` 注解复制过来的。 复制动作发生在修订版本创建时。你可以通过以下方式设置 `CHANGE-CAUSE` 消息：

 - 使用 `kubectl annotate deployment.v1.apps/nginx-deployment kubernetes.io/change-cause="image updated to 1.9.1"` 为 `Deployment`添加注解。
 - 追加 `--record` 命令行标志以保存正在更改资源的 kubectl 命令。
 - 手动编辑资源的清单。

要查看修订历史的详细信息，运行：

```bash
$ kubectl rollout history deployment.v1.apps/nginx-deployment --revision=2
deployments "nginx-deployment" revision 2
  Labels:       app=nginx
          pod-template-hash=1159050644
  Annotations:  kubernetes.io/change-cause=kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1 --record=true
  Containers:
   nginx:
    Image:      nginx:1.16.1
    Port:       80/TCP
     QoS Tier:
        cpu:      BestEffort
        memory:   BestEffort
    Environment Variables:      <none>
  No volumes.
```
### 4.2 回滚到之前的修订版本
按照下面给出的步骤将 Deployment 从当前版本回滚到以前的版本（即版本 2）。
假定现在你已决定撤消当前上线并回滚到以前的修订版本：

```bash
$ kubectl rollout undo deployment.v1.apps/nginx-deployment
deployment.apps/nginx-deployment
```
或者，你也可以通过使用 `--to-revision` 来回滚到特定修订版本：

```bash
$ kubectl rollout undo deployment.v1.apps/nginx-deployment --to-revision=2
deployment.apps/nginx-deployment
```
现在，Deployment 正在回滚到以前的稳定版本。正如你所看到的，Deployment 控制器生成了 回滚到修订版本 2 的 `DeploymentRollback` 事件。
检查回滚是否成功以及 Deployment 是否正在运行，运行：

```bash
$ kubectl get deployment nginx-deployment
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         3         3            3           30m


$ kubectl describe deployment nginx-deployment
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Sun, 02 Sep 2018 18:17:55 -0500
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision=4
                        kubernetes.io/change-cause=kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1 --record=true
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.16.1
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-c4747d96c (3/3 replicas created)
Events:
  Type    Reason              Age   From                   Message
  ----    ------              ----  ----                   -------
  Normal  ScalingReplicaSet   12m   deployment-controller  Scaled up replica set nginx-deployment-75675f5897 to 3
  Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 1
  Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 2
  Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 2
  Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 1
  Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 3
  Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 0
  Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-595696685f to 1
  Normal  DeploymentRollback  15s   deployment-controller  Rolled back deployment "nginx-deployment" to revision 2
  Normal  ScalingReplicaSet   15s   deployment-controller  Scaled down replica set nginx-deployment-595696685f to 0
```

## 5. 缩放 Deployment 

```bash
$ kubectl scale deployment.v1.apps/nginx-deployment --replicas=10
deployment.apps/nginx-deployment scaled
```
假设集群启用了Pod 的[水平自动缩放](https://ghostwritten.blog.csdn.net/article/details/120778634)， 你可以为 Deployment 设置自动缩放器，并基于现有 Pods 的 CPU 利用率选择 要运行的 Pods 个数下限和上限。

```bash
$ kubectl autoscale deployment.v1.apps/nginx-deployment --min=10 --max=15 --cpu-percent=80
deployment.apps/nginx-deployment scaled
```
###  5.1 比例缩放
`RollingUpdate` 的 Deployment 支持同时运行应用程序的多个版本。 当自动缩放器缩放处于上线进程（仍在进行中或暂停）中的 `RollingUpdate Deployment` 时， Deployment 控制器会平衡现有的活跃状态的 `ReplicaSets`（含 Pods 的 ReplicaSets）中的额外副本， 以降低风险。这称为 比例缩放（Proportional Scaling）。

例如，你正在运行一个 10 个副本的 Deployment，其 `maxSurge=3`，`maxUnavailable=2`。

```bash
$ kubectl get deploy
NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment     10        10        10           10          50s
```
更新 Deployment 使用新镜像，碰巧该镜像无法从集群内部解析。

```bash
$ kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:sometag
deployment.apps/nginx-deployment image updated
```
镜像更新使用 `ReplicaSet nginx-deployment-1989198191` 启动新的上线过程， 但由于上面提到的 `maxUnavailable` 要求，该进程被阻塞了。检查上线状态：

```bash
$ kubectl get rs
NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-1989198191   5         5         0         9s
nginx-deployment-618515232    8         8         8         1m
```
然后，出现了新的 Deployment 扩缩请求。自动缩放器将 Deployment 副本增加到 15。 Deployment 控制器需要决定在何处添加 5 个新副本。如果未使用比例缩放，所有 5 个副本 都将添加到新的 ReplicaSet 中。使用比例缩放时，可以将额外的副本分布到所有 ReplicaSet。 较大比例的副本会被添加到拥有最多副本的 ReplicaSet，而较低比例的副本会进入到 副本较少的 ReplicaSet。所有剩下的副本都会添加到副本最多的 ReplicaSet。 具有零副本的 ReplicaSets 不会被扩容。

在上面的示例中，3 个副本被添加到旧 `ReplicaSet` 中，2 个副本被添加到新 `ReplicaSet`。 假定新的副本都很健康，上线过程最终应将所有副本迁移到新的 ReplicaSet 中。 要确认这一点，请运行：

```bash
$ kubectl get deploy
NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment     15        18        7            8           7m
```
上线状态确认了副本是如何被添加到每个 ReplicaSet 的。

```bash
$ kubectl get rs
NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-1989198191   7         7         0         7m
nginx-deployment-618515232    11        11        11        7m
```

##  6. 暂停、恢复 Deployment
你可以在触发一个或多个更新之前暂停 Deployment，然后再恢复其执行。 这样做使得你能够在暂停和恢复执行之间应用多个修补程序，而不会触发不必要的上线操作。
例如，对于一个刚刚创建的 Deployment： 获取 Deployment 信息：

```bash
$ kubectl get deploy
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx     3         3         3            3           1m

$ kubectl get rs
NAME               DESIRED   CURRENT   READY     AGE
nginx-2142116321   3         3         3         1m
```
使用如下指令暂停运行：

```bash
$ kubectl rollout pause deployment.v1.apps/nginx-deployment
deployment.apps/nginx-deployment paused
```
接下来更新 Deployment 镜像：

```bash
$ kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1
deployment.apps/nginx-deployment image updated
```
注意没有新的上线被触发：

```bash
$ kubectl rollout history deployment.v1.apps/nginx-deployment
deployments "nginx"
REVISION  CHANGE-CAUSE
1   <none>
```
获取上线状态确保 Deployment 更新已经成功：

```bash
$ kubectl get rs
NAME               DESIRED   CURRENT   READY     AGE
nginx-2142116321   3         3         3         2m
```
你可以根据需要执行很多更新操作，例如，可以要使用的资源：

```bash
$ kubectl set resources deployment.v1.apps/nginx-deployment -c=nginx --limits=cpu=200m,memory=512Mi
deployment.apps/nginx-deployment resource requirements updated
```
暂停 Deployment 之前的初始状态将继续发挥作用，但新的更新在 Deployment 被 暂停期间不会产生任何效果。

最终，恢复 Deployment 执行并观察新的 `ReplicaSet` 的创建过程，其中包含了所应用的所有更新：

```bash
$ kubectl rollout resume deployment.v1.apps/nginx-deployment
deployment.apps/nginx-deployment resumed

#观察上线的状态
$ kubectl get rs -w
NAME               DESIRED   CURRENT   READY     AGE
nginx-2142116321   2         2         2         2m
nginx-3926361531   2         2         0         6s
nginx-3926361531   2         2         1         18s
nginx-2142116321   1         2         2         2m
nginx-2142116321   1         2         2         2m
nginx-3926361531   3         2         1         18s
nginx-3926361531   3         2         1         18s
nginx-2142116321   1         1         1         2m
nginx-3926361531   3         3         1         18s
nginx-3926361531   3         3         2         19s
nginx-2142116321   0         1         1         2m
nginx-2142116321   0         1         1         2m
nginx-2142116321   0         0         0         2m
nginx-3926361531   3         3         3         20s

#获取最近上线的状态
$ kubectl get rs
NAME               DESIRED   CURRENT   READY     AGE
nginx-2142116321   0         0         0         2m
nginx-3926361531   3         3         3         28s
```

> 说明： 你不可以回滚处于暂停状态的 Deployment，除非先恢复其执行状态。

##  7. Deployment 状态
`Deployment` 的生命周期中会有许多状态。上线新的 `ReplicaSet` 期间可能处于 `Progressing`（进行中），可能是 `Complete`（已完成），也可能是 `Failed`（失败）以至于无法继续进行。

###  7.1 进行中的 Deployment(Progressing)
执行下面的任务期间，Kubernetes 标记 Deployment 为 进行中（`Progressing`）：

 - Deployment 创建新的 ReplicaSet
 - Deployment 正在为其最新的 ReplicaSet 扩容
 - Deployment 正在为其旧有的 ReplicaSet(s) 缩容
 - 新的 Pods 已经就绪或者可用（就绪至少持续了 MinReadySeconds 秒）。

你可以使用 `kubectl rollout status` 监视 `Deployment` 的进度。

### 7.2 完成的 Deployment(Complete)
当 Deployment 具有以下特征时，Kubernetes 将其标记为 完成（`Complete`）：

 - 与 Deployment 关联的所有副本都已更新到指定的最新版本，这意味着之前请求的所有更新都已完成。
 - 与 Deployment 关联的所有副本都可用。
 - 未运行 Deployment 的旧副本。

你可以使用 kubectl rollout status 检查 Deployment 是否已完成。 如果上线成功完成，kubectl rollout status 返回退出代码 0。

```bash
$ kubectl rollout status deployment/nginx-deployment
Waiting for rollout to finish: 2 of 3 updated replicas are available...
deployment "nginx-deployment" successfully rolled out
$ echo $?
0
```
### 7.3 失败的 Deployment(Failed)
你的 Deployment 可能会在尝试部署其最新的 ReplicaSet 受挫(`Failed`)，一直处于未完成状态。 造成此情况一些可能因素如下：

 - 配额（Quota）不足
 - 就绪探测（Readiness Probe）失败
 - 镜像拉取错误
 - 权限不足
 - 限制范围（Limit Ranges）问题
 - 应用程序运行时的配置错误

检测此状况的一种方法是在 Deployment 规约中指定截止时间参数： （[`.spec.progressDeadlineSeconds`]（#progress-deadline-seconds））。 `.spec.progressDeadlineSeconds` 给出的是一个秒数值，Deployment 控制器在（通过 Deployment 状态） 标示 Deployment 进展停滞之前，需要等待所给的时长。

以下 kubectl 命令设置规约中的 `progressDeadlineSeconds`，从而告知控制器 在 10 分钟后报告 Deployment 没有进展：

```bash
$ kubectl patch deployment.v1.apps/nginx-deployment -p '{"spec":{"progressDeadlineSeconds":600}}'
deployment.apps/nginx-deployment patched
```
超过截止时间后，Deployment 控制器将添加具有以下属性的 `DeploymentCondition` 到 Deployment 的 `.status.conditions` 中：

 - Type=Progressing
 - Status=False
 - Reason=ProgressDeadlineExceeded

参考 [Kubernetes API](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#typical-status-properties) 约定 获取更多状态状况相关的信息。


除了报告 `Reason=ProgressDeadlineExceeded` 状态之外，`Kubernetes` 对已停止的 `Deployment` 不执行任何操作。更高级别的编排器可以利用这一设计并相应地采取行动。 例如，将 `Deployment` 回滚到其以前的版本。

如果你暂停了某个 `Deployment`，`Kubernetes` 不再根据指定的截止时间检查 `Deployment` 进展。 你可以在上线过程中间安全地暂停 `Deployment` 再恢复其执行，这样做不会导致超出最后时限的问题。

Deployment 可能会出现瞬时性的错误，可能因为设置的超时时间过短， 也可能因为其他可认为是临时性的问题。例如，假定所遇到的问题是配额不足。 如果描述 Deployment，你将会注意到以下部分：

```bash
$ kubectl describe deployment nginx-deployment
<...>
Conditions:
  Type            Status  Reason
  ----            ------  ------
  Available       True    MinimumReplicasAvailable
  Progressing     True    ReplicaSetUpdated
  ReplicaFailure  True    FailedCreate
<...>


$ kubectl get deployment nginx-deployment -o yaml
status:
  availableReplicas: 2
  conditions:
  - lastTransitionTime: 2016-10-04T12:25:39Z
    lastUpdateTime: 2016-10-04T12:25:39Z
    message: Replica set "nginx-deployment-4262182780" is progressing.
    reason: ReplicaSetUpdated
    status: "True"
    type: Progressing
  - lastTransitionTime: 2016-10-04T12:25:42Z
    lastUpdateTime: 2016-10-04T12:25:42Z
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: 2016-10-04T12:25:39Z
    lastUpdateTime: 2016-10-04T12:25:39Z
    message: 'Error creating: pods "nginx-deployment-4262182780-" is forbidden: exceeded quota:
      object-counts, requested: pods=1, used: pods=3, limited: pods=2'
    reason: FailedCreate
    status: "True"
    type: ReplicaFailure
  observedGeneration: 3
  replicas: 2
  unavailableReplicas: 2
```
最终，一旦超过 Deployment 进度限期，Kubernetes 将更新状态和进度状况的原因：

```bash
Conditions:
  Type            Status  Reason
  ----            ------  ------
  Available       True    MinimumReplicasAvailable
  Progressing     False   ProgressDeadlineExceeded
  ReplicaFailure  True    FailedCreate
```
可以通过缩容 `Deployment` 或者缩容其他运行状态的控制器，或者直接在命名空间中增加配额 来解决配额不足的问题。如果配额条件满足，Deployment 控制器完成了 Deployment 上线操作， Deployment 状态会更新为成功状况（`Status=True and Reason=NewReplicaSetAvailable`）。

```bash
Conditions:
  Type          Status  Reason
  ----          ------  ------
  Available     True    MinimumReplicasAvailable
  Progressing   True    NewReplicaSetAvailable
```
`Type=Available` 加上 `Status=True` 意味着 Deployment 具有最低可用性。 最低可用性由 Deployment 策略中的参数指定。 `Type=Progressing` 加上 `Status=True` 表示 Deployment 处于上线过程中，并且正在运行， 或者已成功完成进度，最小所需新副本处于可用。 请参阅对应状况的 Reason 了解相关细节。 在我们的案例中 `Reason=NewReplicaSetAvailable` 表示 Deployment 已完成。

你可以使用 `kubectl rollout status` 检查 `Deployment` 是否未能取得进展。 如果 Deployment 已超过进度限期，kubectl rollout status 返回非零退出代码。

```bash
$ kubectl rollout status deployment/nginx-deployment
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
error: deployment "nginx" exceeded its progress deadline

$ echo $?
1
```
## 8. 编排 Deployment

### 8.1 Pod template

 - `.spec` 中只有 `.spec.template` 和 `.spec.selector` 是必需的字段
 - `.spec.template` 是一个 [Pod 模板](https://kubernetes.io/zh/docs/concepts/workloads/pods/#pod-templates)。它和 Pod 的语法规则完全相同。 只是这里它是嵌套的，因此不需要 `apiVersion` 或 `kind`。
 - 除了 Pod 的必填字段外，Deployment 中的 Pod 模板必须指定适当的标签和适当的重新启动策略。对于标签，请确保不要与其他控制器重叠。请[参考选择算符](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/labels/)。

### 8.2 Replicas

 - `.spec.replicas` 是指定所需 Pod 的可选字段。它的默认值是1

### 8.3 spec.selector

 - `.spec.selector` 是指定本 Deployment 的 Pod标签[选择算符](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/labels/)的必需字段。
 - `.spec.selector` 必须匹配 `.spec.template.metadata.labels`，否则请求会被 API 拒绝。
 - 在 `API apps/v1`版本中，`.spec.selector` 和 `.metadata.labels` 如果没有设置的话，
   不会被默认设置为 `.spec.template.metadata.labels`，所以需要明确进行设置。 同时在`apps/v1`版本中，Deployment 创建后 `.spec.selector` 是不可变的

###  8.4 策略
`.spec.strategy` 策略指定用于用新 Pods 替换旧 Pods 的策略。 `.spec.strategy.type` 可以是 “`Recreate`” 或 “`RollingUpdate`”。“`RollingUpdate`” 是默认值。

**重新创建 Deployment** 
如果 `.spec.strategy.type==Recreate`，在创建新 Pods 之前，所有现有的 Pods 会被杀死。

**滚动更新 Deployment** 
Deployment 会在 `.spec.strategy.type==RollingUpdate`时，采取 滚动更新的方式更新 Pods。你可以指定 `maxUnavailable` 和 `maxSurge` 来控制滚动更新 过程。

**最大峰值** 
`.spec.strategy.rollingUpdate.maxSurge` 是一个可选字段，用来指定可以创建的超出 期望 Pod 个数的 Pod 数量。此值可以是绝对数（例如，5）或所需 Pods 的百分比（例如，10%）。 如果 `MaxUnavailable` 为 0，则此值不能为 0。百分比值会通过向上取整转换为绝对数。 此字段的默认值为 25%。

例如，当此值为 30% 时，启动滚动更新后，会立即对新的 `ReplicaSet` 扩容，同时保证新旧 Pod 的总数不超过所需 Pod 总数的 130%。一旦旧 Pods 被杀死，新的 ReplicaSet 可以进一步扩容， 同时确保更新期间的任何时候运行中的 Pods 总数最多为所需 Pods 总数的 130%。

###  8.5 进度期限秒数
`.spec.progressDeadlineSeconds` 是一个可选字段，用于指定系统在报告 Deployment 进展失败 之前等待 `Deployment` 取得进展的秒数。 这类报告会在资源状态中体现为 `Type=Progressing`、`Status=False`、 `Reason=ProgressDeadlineExceeded`。Deployment 控制器将持续重试 Deployment。 将来，一旦实现了自动回滚，Deployment 控制器将在探测到这样的条件时立即回滚 Deployment。

如果指定，则此字段值需要大于 `.spec.minReadySeconds` 取值

###  8.6 最短就绪时间 
`.spec.minReadySeconds` 是一个可选字段，用于指定新创建的 Pod 在没有任意容器崩溃情况下的最小就绪时间， 只有超出这个时间 Pod 才被视为可用。默认值为 0（Pod 在准备就绪后立即将被视为可用）。 要了解何时 Pod 被视为就绪，可[参考容器探针](https://kubernetes.io/zh/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)

###  8.7 Rollback To
`.spec.rollbackTo` 是一个可以选配置项，用来配置 Deployment 回退的配置。设置该参数将触发回退操作，每次回退完成后，该值就会被清除。

### 8.8 Revision
`.spec.rollbackTo.revision` 是一个可选配置项，用来指定回退到的 `revision`。默认是 0，意味着回退到上一个 revision。

###  8.9 修订历史限制
Deployment 的修订历史记录存储在它所控制的 ReplicaSets 中。

`.spec.revisionHistoryLimit` 是一个可选字段，用来设定出于会滚目的所要保留的旧 `ReplicaSet` 数量。 这些旧 ReplicaSet 会消耗 etcd 中的资源，并占用 `kubectl get rs` 的输出。 每个 Deployment 修订版本的配置都存储在其 `ReplicaSets` 中；因此，一旦删除了旧的 ReplicaSet， 将失去回滚到 Deployment 的对应修订版本的能力。 默认情况下，系统保留 10 个旧 ReplicaSet，但其理想值取决于新 Deployment 的频率和稳定性。

更具体地说，将此字段设置为 0 意味着将清理所有具有 0 个副本的旧 ReplicaSet。 在这种情况下，无法撤消新的 Deployment 上线，因为它的修订历史被清除了。

###  8.10 paused（暂停的） 
`.spec.paused` 是用于暂停和恢复 Deployment 的可选布尔字段。 暂停的 Deployment 和未暂停的 Deployment 的唯一区别是，Deployment 处于暂停状态时， `PodTemplateSpec` 的任何修改都不会触发新的上线。 Deployment 在创建时是默认不会处于暂停状态。


参考：

 - [Kuberenetes Deployments](https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/)


