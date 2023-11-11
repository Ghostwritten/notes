#  Kubernetes DaemonSet
tags: DaemonSet

[
![在这里插入图片描述](https://img-blog.csdnimg.cn/0f9a370fdd4741c6bcdd41d80742c785.png#pic_center)](https://www.rottentomatoes.com/m/blade_runner_2049)

*有时候如果你爱一个人，就要形同陌路。   ——《银翼杀手2049》*




----
## 1. 简介
DaemonSet 确保全部（或者某些）节点上运行一个 Pod 的副本。 当有节点加入集群时， 也会为他们新增一个 Pod 。 当有节点从集群移除时，这些 Pod 也会被回收。删除 DaemonSet 将会删除它创建的所有 Pod。

DaemonSet 的一些典型用法：

 - 在每个节点上运行集群守护进程
 - 在每个节点上运行日志收集守护进程
 - 在每个节点上运行监控守护进程

一种简单的用法是为每种类型的守护进程在所有的节点上都启动一个 DaemonSet。 一个稍微复杂的用法是为同一种守护进程部署多个 DaemonSet；每个具有不同的标志， 并且对不同硬件类型具有不同的内存、CPU 要求。


## 2. 创建 DaemonSet
你可以在 YAML 文件中描述 DaemonSet。 例如，下面的 `daemonset.yaml` 文件描述了一个运行 fluentd-elasticsearch Docker 镜像的 DaemonSet：

```bash
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # this toleration is to have the daemonset runnable on master nodes
      # remove it if your masters can't run pods
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```
基于 YAML 文件创建 DaemonSet：

```bash
kubectl apply -f https://k8s.io/examples/controllers/daemonset.yaml
```
## 3. 配置说明
### 3.1 必需字段

 - DaemonSet 需要 apiVersion、kind 和 metadata 字段
 - DaemonSet 对象的名称必须是一个合法的 DNS 子域名。
 - DaemonSet 也需要一个 .spec 配置段
### 3.2 Pod 模板 
`.spec` 中唯一必需的字段是 `.spec.template`。

.spec.template 是一个 Pod 模板。 除了它是嵌套的，因而不具有 apiVersion 或 kind 字段之外，它与 Pod 具有相同的 schema。

除了 Pod 必需字段外，在 DaemonSet 中的 Pod 模板必须指定合理的标签（查看 Pod 选择算符）。

在 DaemonSet 中的 Pod 模板必须具有一个值为 `Always` 的 `RestartPolicy`。 当该值未指定时，默认是 Always。
### 3.3 Pod 选择算符
`.spec.selector` 字段表示 Pod 选择算符，它与 Job 的 .spec.selector 的作用是相同的。

从 `Kubernetes 1.8` 开始，您必须指定与 .spec.template 的标签匹配的 Pod 选择算符。 用户不指定 Pod 选择算符时，该字段不再有默认值。 选择算符的默认值生成结果与 kubectl apply 不兼容。 此外，一旦创建了 DaemonSet，它的 .spec.selector 就不能修改。 修改 Pod 选择算符可能导致 Pod 意外悬浮，并且这对用户来说是费解的。

spec.selector 是一个对象，如下两个字段组成：

 - `matchLabels` - 与 `ReplicationController` 的 .spec.selector 的作用相同。
 - `matchExpressions` - 允许构建更加复杂的选择器，可以通过指定 key、value 列表以及将 key 和 value列表关联起来的 operator。

当上述两个字段都指定时，结果会按逻辑与（AND）操作处理。

**如果指定了 `.spec.selector`，必须与 `.spec.template.metadata.labels` 相匹配。 如果与后者不匹配，则 DeamonSet 会被 API 拒绝。**

### 3.4 仅在某些节点上运行 Pod 
如果指定了 `.spec.template.spec.nodeSelector`，DaemonSet 控制器将在能够与 Node 选择算符 匹配的节点上创建 Pod。 类似这种情况，可以指定 `.spec.template.spec.affinity`，之后 DaemonSet 控制器 将在能够与节点亲和性 匹配的节点上创建 Pod。 如果根本就没有指定，则 DaemonSet Controller 将在所有节点上创建 Pod。

## 4. Daemon Pods 是如何被调度的
### 4.1 通过默认调度器调度
DaemonSet 确保所有符合条件的节点都运行该 Pod 的一个副本。 通常，运行 Pod 的节点由 `Kubernetes 调度器`选择。 不过，DaemonSet Pods 由 `DaemonSet 控制器`创建和调度。这就带来了以下问题：

 - Pod 行为的不一致性：正常 Pod 在被创建后等待调度时处于 Pending 状态， DaemonSet Pods 创建后不会处于Pending 状态下。这使用户感到困惑。
 - [Pod 抢占](https://kubernetes.io/zh/docs/concepts/configuration/pod-priority-preemption/) 由默认调度器处理。启用抢占后，DaemonSet 控制器将在不考虑 Pod 优先级和抢占 的情况下制定调度决策。

`ScheduleDaemonSetPods` 允许您使用`默认调度器`而不是 `DaemonSet 控制器`来调度 DaemonSets， 方法是将 `NodeAffinity` 条件而不是 `.spec.nodeName` 条件添加到 DaemonSet Pods。 默认调度器接下来将 Pod 绑定到目标主机。 如果 DaemonSet Pod 的`节点亲和性配置`已存在，则被替换。 `DaemonSet 控制器`仅在创建或修改 DaemonSet Pod 时执行这些操作， 并且不会更改 DaemonSet 的 `spec.template`。

此外，系统会自动添加 `node.kubernetes.io/unschedulable：NoSchedule` 容忍度到 DaemonSet Pods。在调度 DaemonSet Pod 时，默认调度器会忽略 unschedulable 节点。

### 4.2 污点和容忍度
| node.kubernetes.io/not-ready           | NoExecute  | 1.13+ | 当出现类似网络断开的情况导致节点问题时，DaemonSet Pod 不会被逐出。                 |
|----------------------------------------|------------|-------|----------------------------------------------------------|
| node.kubernetes.io/unreachable         | NoExecute  | 1.13+ | 当出现类似于网络断开的情况导致节点问题时，DaemonSet Pod 不会被逐出。                |
| node.kubernetes.io/disk-pressure       | NoSchedule | 1.8+  |                                                          |
| node.kubernetes.io/memory-pressure     | NoSchedule | 1.8+  |                                                          |
| node.kubernetes.io/unschedulable       | NoSchedule | 1.12+ | DaemonSet Pod 能够容忍默认调度器所设置的 unschedulable 属性.            |
| node.kubernetes.io/network-unavailable | NoSchedule | 1.12+ | DaemonSet 在使用宿主网络时，能够容忍默认调度器所设置的 network-unavailable 属性。 |
。

## 5.  Daemon Pods 通信
与 DaemonSet 中的 Pod 进行通信的几种可能模式如下：

 1. `推送（Push）`：配置 DaemonSet 中的 Pod，将更新发送到另一个服务，例如统计数据库。 这些服务没有客户端。
 2. `NodeIP 和已知端口`：DaemonSet 中的 Pod 可以使用 hostPort，从而可以通过节点 IP 访问到Pod。客户端能通过某种方法获取节点 IP 列表，并且基于此也可以获取到相应的端口。
 3. `DNS`：创建具有相同 Pod 选择算符的 无头服务， 通过使用 endpoints 资源或从 DNS 中检索到多个 A 记录来发现DaemonSet。
 4. `Service`：创建具有相同 Pod 选择算符的服务，并使用该服务随机访问到某个节点上的 守护进程（没有办法访问到特定节点）。

##  6. DaemonSet更新
如果节点的标签被修改，DaemonSet 将立刻向新匹配上的节点添加 Pod， 同时删除不匹配的节点上的 Pod。

你可以修改 DaemonSet 创建的 Pod。不过并非 Pod 的所有字段都可更新。 下次当某节点（即使具有相同的名称）被创建时，DaemonSet 控制器还会使用最初的模板。

您可以删除一个 DaemonSet。如果使用 kubectl 并指定 `--cascade=false` 选项， 则 Pod 将被保留在节点上。接下来如果创建使用相同选择算符的新 DaemonSet， 新的 DaemonSet 会收养已有的 Pod。 如果有 Pod 需要被替换，DaemonSet 会根据其 `updateStrategy` 来替换。



## 7. DaemonSet 滚动更新
`Kubernetes 1.6` 或者更高版本中才支持 DaemonSet 滚动更新功能
### 7.1 DaemonSet 更新策略 
DaemonSet 有两种更新策略：

 - `OnDelete`: 使用 OnDelete 更新策略时，在更新 DaemonSet 模板后，只有当你手动删除老的 DaemonSet pods 之后，新的 DaemonSet Pod 才会被自动创建。跟 Kubernetes 1.6 以前的版本类似。
 - `RollingUpdate`: 这是默认的更新策略。使用 RollingUpdate 更新策略时，在更新 DaemonSet 模板后， 老的DaemonSet pods 将被终止，并且将以受控方式自动创建新的 DaemonSet pods。 更新期间，最多只能有DaemonSet 的一个 Pod 运行于每个节点上。

### 7.2 执行滚动更新
启用 DaemonSet 的滚动更新功能，必须设置 `.spec.updateStrategy.type` 为 `RollingUpdate`。
你可能想设置 `.spec.updateStrategy.rollingUpdate.maxUnavailable` (默认为 1) 和 `.spec.minReadySeconds` (默认为 0)。

#### 7.2.1 **创建带有 RollingUpdate 更新策略的 DaemonSet**
下面的 YAML 包含一个 DaemonSet，其更新策略为 'RollingUpdate'：

```bash
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1   #滚动升级时允许的最大Unavailable的pod个数
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # this toleration is to have the daemonset runnable on master nodes
      # remove it if your masters can't run pods
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

检查了 DaemonSet 清单中更新策略的设置之后，创建 DaemonSet：

```bash
kubectl create -f https://k8s.io/examples/controllers/fluentd-daemonset.yaml
```

另一种方式是如果你希望使用 kubectl apply 来更新 DaemonSet 的话，也可以 使用 kubectl apply 来创建 DaemonSet：

```bash
kubectl apply -f https://k8s.io/examples/controllers/fluentd-daemonset.yaml
```
#### 7.2.2 检查 DaemonSet 的滚动更新策略
首先，检查 DaemonSet 的更新策略，确保已经将其设置为 RollingUpdate:

```bash
kubectl get ds/fluentd-elasticsearch -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}' -n kube-system
```

如果还没在系统中创建 DaemonSet，请使用以下命令检查 DaemonSet 的清单：

```bash
kubectl apply -f https://k8s.io/examples/controllers/fluentd-daemonset.yaml --dry-run=client -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}'
```

两个命令的输出都应该为：

```bash
RollingUpdate
```

如果输出不是 RollingUpdate，请返回并相应地修改 DaemonSet 对象或者清单。

#### 7.2.3 更新 DaemonSet 模板
对 `RollingUpdate DaemonSet` 的 `.spec.template` 的任何更新都将触发滚动更新。 这可以通过几个不同的 kubectl 命令来完成。

```bash
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # this toleration is to have the daemonset runnable on master nodes
      # remove it if your masters can't run pods
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```
**声明式命令** 
如果你使用 配置文件 来更新 DaemonSet，请使用 kubectl apply:

```bash
kubectl apply -f https://k8s.io/examples/controllers/fluentd-daemonset-update.yaml
```

**指令式命令**
如果你使用 指令式命令 来更新 DaemonSets，请使用kubectl edit：

```bash
kubectl edit ds/fluentd-elasticsearch -n kube-system
```
**只更新容器镜像**
如果你只需要更新 DaemonSet 模板里的容器镜像，比如，`.spec.template.spec.containers[*].image`, 请使用 kubectl set image:

```bash
kubectl set image ds/fluentd-elasticsearch fluentd-elasticsearch=quay.io/fluentd_elasticsearch/fluentd:v2.6.0 -n kube-system
```

**监视滚动更新状态**
最后，观察 DaemonSet 最新滚动更新的进度：

```bash
kubectl rollout status ds/fluentd-elasticsearch -n kube-system
```

当滚动更新完成时，输出结果如下：

```bash
daemonset "fluentd-elasticsearch" successfully rolled out
```

## 8. DaemonSet 回滚
### 8.1 显示DaemonSet 回滚到的历史修订版本（revision）
如果只想回滚到最后一个版本，可以跳过这一步。

```bash
$ kubectl rollout history daemonset <daemonset-name>
daemonsets "<daemonset-name>"
REVISION        CHANGE-CAUSE
1               ...
2               ...
...
```
在创建时，DaemonSet 的变化原因从 `kubernetes.io/change-cause` 注解（annotation） 复制到其修订版本中。用户可以在 kubectl 命令中设置 `--record=true`， 将执行的命令记录在变化原因注解中。

查看指定版本的详细信息：

```bash
$ kubectl rollout history daemonset <daemonset-name> --revision=1
daemonsets "<daemonset-name>" with revision #1
Pod Template:
Labels:       foo=bar
Containers:
app:
 Image:       ...
 Port:        ...
 Environment: ...
 Mounts:      ...
Volumes:       ...
```
### 8.2 回滚到指定版本 
说明： 如果 `--to-revision` 参数未指定，将选中最近的版本。
```bash
# 在 --to-revision 中指定你从步骤 1 中获取的修订版本
$ kubectl rollout undo daemonset <daemonset-name> --to-revision=<revision>
daemonset "<daemonset-name>" rolled back
```
### 8.3 监视 DaemonSet 回滚进度 

```bash
$ kubectl rollout status ds/<daemonset-name>
daemonset "<daemonset-name>" successfully rolled out
```
### 8.4 理解 DaemonSet 修订版本
在前面的 kubectl rollout history 步骤中，你获得了一个修订版本列表，每个修订版本都存储在名为 ControllerRevision 的资源中。

要查看每个修订版本中保存的内容，可以找到 DaemonSet 修订版本的原生资源：

```bash
$ kubectl get controllerrevision -l <daemonset-selector-key>=<daemonset-selector-value>
NAME                               CONTROLLER                     REVISION   AGE
<daemonset-name>-<revision-hash>   DaemonSet/<daemonset-name>     1          1h
<daemonset-name>-<revision-hash>   DaemonSet/<daemonset-name>     2          1h
```

每个 ControllerRevision 中存储了相应 DaemonSet 版本的注解和模板。

kubectl rollout undo 选择特定的 ControllerRevision，并用 ControllerRevision 中存储的模板代替 DaemonSet 的模板。 kubectl rollout undo 相当于通过其他命令（如 kubectl edit 或 kubectl apply） 将 DaemonSet 模板更新至先前的版本。

> 说明： 注意 DaemonSet 修订版本只会正向变化。也就是说，回滚完成后，所回滚到的 ControllerRevision 版本号(.revision 字段) 会增加。 例如，如果用户在系统中有版本 1 和版本 2，并从版本 2 回滚到版本 1， 带有.revision: 1 的ControllerRevision 将变为 .revision: 3。


参考：

 - [Kubernetes DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
 - [google cloud DaemonSet](https://cloud.google.com/kubernetes-engine/docs/concepts/daemonset) 
 - [What is a DaemonSet?](https://cloud.google.com/kubernetes-engine/docs/concepts/daemonset)
 - [How To Use & Manage Kubernetes DaemonSets](https://www.bmc.com/blogs/kubernetes-daemonset/)

