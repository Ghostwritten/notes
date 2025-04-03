#  kubernetes 学习污点和容忍度
tags: 策略

![](https://i-blog.csdnimg.cn/blog_migrate/59a8f59d34c8f22f8dd3f4f7324a47fb.png)





## 1. 概念

 - **节点亲和性** 是 Pod 的一种属性，它使 Pod 被吸引到一类特定的节点。 这可能出于一种偏好，也可能是硬性要求。
 - **容忍度**（Tolerations）是应用于 Pod 上的，允许（但并不要求）Pod 调度到带有与之匹配的污点的节点上。
 - **Taint**（污点）则相反，它使节点能够排斥一类特定的 Pod。
 - **污点和容忍度**（Toleration）相互配合，可以用来避免 Pod 被分配到不合适的节点上。 每个节点上都可以应用一个或多个污点，这表示对于那些不能容忍这些污点的 Pod，是不会被该节点接受的。

您可以使用命令 `kubectl taint` 给节点增加一个污点。比如

```bash
kubectl taint nodes node1 key1=value1:NoSchedule
```
给节点 node1 增加一个污点，它的键名是 `key1`，键值是 `value1`，效果是 `NoSchedule`。 这表示只有拥有和这个污点相匹配的容忍度的 Pod 才能够被分配到 node1 这个节点。

若要移除上述命令所添加的污点，你可以执行：

```bash
kubectl taint nodes node1 key:NoSchedule-
```
您可以在 PodSpec 中定义 Pod 的容忍度。 下面两个容忍度均与上面例子中使用 `kubectl taint` 命令创建的污点相匹配， 因此如果一个 Pod 拥有其中的任何一个容忍度都能够被分配到 node1 ：

```bash
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
```

```bash
tolerations:
- key: "key1"
  operator: "Exists"
  effect: "NoSchedule"
```
这里是一个使用了容忍度的 Pod：

```bash
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - key: "example-key"
    operator: "Exists"
    effect: "NoSchedule"
```
`operator` 的默认值是 `Equal`。

一个容忍度和一个污点相“匹配”是指它们有一样的`键名`和`效果`，并且：

 - 如果 operator 是 `Exists` （此时容忍度不能指定 value）
 - 如果 operator 是 `Equal` ，则它们的 `value` 应该相等


说明：
存在两种特殊情况：

 - 如果一个容忍度的 `key` 为空且 operator 为 `Exists`， 表示这个`容忍度`与任意的 key 、value 和 effect都匹配，即这个容忍度能容忍任意 `taint`。
 - 如果 `effect` 为空，则可以与所有键名 key 的效果相匹配。


上述例子使用到的 effect 的一个值 `NoSchedule`，您也可以使用另外一个值 `PreferNoSchedule`。 这是“优化”或“软”版本的 NoSchedule —— 系统会 尽量 避免将 Pod 调度到存在其不能容忍污点的节点上， 但这不是强制的。effect 的值还可以设置为 NoExecute，下文会详细描述这个值。

您可以给一个节点添加多个污点，也可以给一个 Pod 添加多个容忍度设置。 Kubernetes 处理多个污点和容忍度的过程就像一个过滤器：**从一个节点的所有污点开始遍历， 过滤掉那些 Pod 中存在与之相匹配的容忍度的污点。余下未被过滤的污点的 effect 值决定了 Pod 是否会被分配到该节点**，特别是以下情况：

 1. 如果未被过滤的污点中存在至少一个 effect 值为 `NoSchedule` 的污点， 则 Kubernetes 不会将 Pod分配到该节点。
 2. 如果未被过滤的污点中不存在 effect 值为 NoSchedule 的污点， 但是存在 effect 值为 `PreferNoSchedule` 的污点， 则 Kubernetes 会 尝试 将 Pod 分配到该节点。
 3. 如果未被过滤的污点中存在至少一个 effect 值为 `NoExecute` 的污点， 则 Kubernetes 不会将 Pod 分配到该节点（如果 Pod 还未在节点上运行）， 或者将 Pod 从该节点驱逐（如果 Pod 已经在节点上运行）


例如，假设您给一个节点添加了如下污点

```bash
kubectl taint nodes node1 key1=value1:NoSchedule
kubectl taint nodes node1 key1=value1:NoExecute
kubectl taint nodes node1 key2=value2:NoSchedule
```
假定有一个 Pod，它有两个容忍度：

```bash
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoExecute"
```
在这种情况下，上述 Pod 不会被分配到上述节点，因为其没有容忍度和第三个污点相匹配。 但是如果在给节点添加上述污点之前，该 Pod 已经在上述节点运行， 那么它还可以继续运行在该节点上，因为第三个污点是三个污点中唯一不能被这个 Pod 容忍的。

通常情况下，如果给一个节点添加了一个 effect 值为 NoExecute 的污点， 则任何不能忍受这个污点的 Pod 都会马上被驱逐， 任何可以忍受这个污点的 Pod 都不会被驱逐。 但是，如果 Pod 存在一个 effect 值为 `NoExecute` 的容忍度指定了可选属性 `tolerationSeconds` 的值，则表示在给节点添加了上述污点之后， Pod 还能继续在节点上运行的时间。例如，

```bash
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoExecute"
  tolerationSeconds: 3600
```
这表示如果这个 Pod 正在运行，同时一个匹配的污点被添加到其所在的节点， 那么 Pod 还将继续在节点上运行 3600 秒，然后被驱逐。 如果在此之前上述污点被删除了，则 Pod 不会被驱逐。

## 2. 示例
通过污点和容忍度，可以灵活地让 Pod 避开 某些节点或者将 Pod 从某些节点驱逐。下面是几个使用例子：

**专用节点**：如果您想将某些节点专门分配给特定的一组用户使用，您可以给这些节点添加一个污点（即， `kubectl taint nodes nodename dedicated=groupName:NoSchedule`）， 然后给这组用户的 Pod 添加一个相对应的 `toleration`（通过编写一个自定义的 准入控制器，很容易就能做到）。 拥有上述容忍度的 Pod 就能够被分配到上述专用节点，同时也能够被分配到集群中的其它节点。 如果您希望这些 Pod 只能被分配到上述专用节点，那么您还需要给这些专用节点另外添加一个和上述 污点类似的 label （例如：`dedicated=groupName`），同时 还要在上述准入控制器中给 Pod 增加`节点亲和性`要求上述 Pod 只能被分配到添加了 `dedicated=groupName` 标签的节点上。

**配备了特殊硬件的节点**：在部分节点配备了特殊硬件（比如 `GPU`）的集群中， 我们希望不需要这类硬件的 Pod 不要被分配到这些特殊节点，以便为后继需要这类硬件的 Pod 保留资源。 要达到这个目的，可以先给配备了特殊硬件的节点添加 taint （例如 `kubectl taint nodes nodename special=true:NoSchedule` 或 `kubectl taint nodes nodename special=true:PreferNoSchedule`)， 然后给使用了这类特殊硬件的 Pod 添加一个相匹配的 `toleration`。 和专用节点的例子类似，添加这个容忍度的最简单的方法是使用自定义 准入控制器。 比如，我们推荐使用扩展资源 来表示特殊硬件，给配置了特殊硬件的节点添加污点时包含扩展资源名称， 然后运行一个 `ExtendedResourceToleration` 准入控制器。此时，因为节点已经被设置污点了，没有对应容忍度的 Pod 会被调度到这些节点。但当你创建一个使用了扩展资源的 Pod 时， ExtendedResourceToleration 准入控制器会自动给 Pod 加上正确的容忍度， 这样 Pod 就会被自动调度到这些配置了特殊硬件件的节点上。 这样就能够确保这些配置了特殊硬件的节点专门用于运行需要使用这些硬件的 Pod， 并且您无需手动给这些 Pod 添加容忍度。

**基于污点的驱逐**: 这是在每个 Pod 中配置的在节点出现问题时的驱逐行为，接下来的章节会描述这个特性。

## 3. 基于污点的驱逐
前文提到过污点的 effect 值 `NoExecute`会影响已经在节点上运行的 Pod

 - 如果 Pod 不能忍受 effect 值为 `NoExecute` 的污点，那么 Pod 将马上被驱逐
 - 如果 Pod 能够忍受 effect 值为 `NoExecute` 的污点，但是在容忍度定义中没有指定tolerationSeconds，则 Pod 还会一直在这个节点上运行。
 - 如果 Pod 能够忍受 effect 值为 NoExecute 的污点，而且指定了 tolerationSeconds， 则 Pod还能在这个节点上继续运行这个指定的时间长度。

当某种条件为真时，节点控制器会自动给节点添加一个污点。当前内置的污点包括：

 - `node.kubernetes.io/not-ready`：节点未准备好。这相当于节点状态 Ready 的值为 "`False`"。
 - `node.kubernetes.io/unreachable`：节点控制器访问不到节点. 这相当于节点状态 Ready 的值为 "`Unknown`"。
 - `node.kubernetes.io/out-of-disk`：节点磁盘耗尽。
 - `node.kubernetes.io/memory-pressure`：节点存在内存压力。
 - `node.kubernetes.io/disk-pressure`：节点存在磁盘压力。
 - `node.kubernetes.io/network-unavailable`：节点网络不可用。
 - `node.kubernetes.io/unschedulable`: 节点不可调度。
 - `node.cloudprovider.kubernetes.io/uninitialized`：如果 kubelet 启动时指定了一个 "外部" 云平台驱动， 它将给当前节点添加一个污点将其标志为不可用。在 cloud-controller-manager的一个控制器初始化这个节点后，kubelet 将删除这个污点。


在节点被驱逐时，节点控制器或者 kubelet 会添加带有 NoExecute 效应的相关污点。 如果异常状态恢复正常，kubelet 或节点控制器能够移除相关的污点。


> 说明： 为了保证由于节点问题引起的 Pod 驱逐 速率限制行为正常，系统实际上会以限定速率的方式添加污点。在像主控节点与工作节点间通信中断等场景下， 这样做可以避免 Pod 被大量驱逐。

使用这个功能特性，结合 `tolerationSeconds`，Pod 就可以指定当节点出现一个 或全部上述问题时还将在这个节点上运行多长的时间。

比如，一个使用了很多本地状态的应用程序在网络断开时，仍然希望停留在当前节点上运行一段较长的时间， 愿意等待网络恢复以避免被驱逐。在这种情况下，Pod 的容忍度可能是下面这样的：

```bash
tolerations:
- key: "node.kubernetes.io/unreachable"
  operator: "Exists"
  effect: "NoExecute"
  tolerationSeconds: 6000
```

> 说明： Kubernetes 会**自动**给 Pod 添加一个 key 为 `node.kubernetes.io/not-ready` 的容忍度并配置 `tolerationSeconds=300`，除非用户提供的 Pod 配置中已经已存在了 key 为`node.kubernetes.io/not-ready` 的容忍度。
> 
> 同样，Kubernetes 会给 Pod 添加一个 key 为 `node.kubernetes.io/unreachable` 的容忍度并配置 `tolerationSeconds=300`，除非用户提供的 Pod 配置中已经已存在了 key 为
`node.kubernetes.io/unreachable` 的容忍度。

这种自动添加的容忍度意味着在其中一种问题被检测到时 Pod 默认能够继续停留在当前节点运行 5 分钟。

DaemonSet 中的 Pod 被创建时， 针对以下`污点`自动添加的 `NoExecute` 的容忍度将不会指定 `tolerationSeconds`：

```bash
node.kubernetes.io/unreachable
node.kubernetes.io/not-ready
```

这保证了出现上述问题时 DaemonSet 中的 Pod 永远不会被驱逐

## 4. 基于节点状态添加污点
Node 生命周期控制器会自动创建与 Node 条件相对应的带有 `NoSchedule` 效应的污点。 同样，调度器不检查节点条件，而是检查节点污点。这确保了节点条件不会影响调度到节点上的内容。 用户可以通过添加适当的 Pod 容忍度来选择忽略某些 Node 的问题(表示为 Node 的调度条件)。
`DaemonSet` 控制器自动为所有守护进程添加如下 NoSchedule 容忍度以防 DaemonSet 崩溃：

 - node.kubernetes.io/memory-pressure
 - node.kubernetes.io/disk-pressure
 - node.kubernetes.io/out-of-disk (只适合关键 Pod)
 - node.kubernetes.io/unschedulable (1.10 或更高版本)
 - node.kubernetes.io/network-unavailable (只适合主机网络配置)

添加上述容忍度确保了向后兼容，您也可以选择自由向 DaemonSet 添加容忍度。

