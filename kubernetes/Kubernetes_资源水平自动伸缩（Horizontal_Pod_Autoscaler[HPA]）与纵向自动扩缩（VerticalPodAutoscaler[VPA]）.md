

----

##  1. Pod 水平自动伸缩HPA
那些使用情况随着时间而变化的应用程序，需要添加或删除 Pod 副本以响应应用程序需求的变化。Pod 水平自动伸缩（HPA）可以管理增减 Pod 副本数。

HPA 支持有状态、无状态应用程序的伸缩。将 HPA 与集群自动伸缩组件结合使用（请参见下文）可以减少 Pod 数量，从而减少活动节点的数量，以节省经常因需求变化的工作负载成本。

对于配置了 HPA 的工作负载，HPA 控制器监视 Pod 的工作负载，以确定是否需要改变 Pod 副本数。大多数情况下，控制器会选取每个 Pod 的平均值，然后计算添加或删除副本是否会使当前值更接近目标值。例如，**部署的目标 CPU 利用率可能是 50％。如果当前正在运行五个 Pod，并且平均 CPU 利用率为 75％，那么控制器会添加 3 个副本，让 Pod 平均值更接近 50％**。

HPA 伸缩比例计算也可以使用自定义指标或外部指标。自定义指标是对 CPU 使用率以外的 Pod 使用率进行标记，例如网络流量、内存或与 Pod 应用程序相关的值。外部指标测量与 Pod 无关的指标。例如，外部指标可以跟踪队列中待处理的任务数。

创建 `HorizontalPodAutoscaler` 对象或使用 `kubectl autoscale` 子命令都可以配置 HPA 控制器，以管理工作负载

```bash
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: hello-world
  namespace: default
spec:
  maxReplicas: 10
  minReplicas: 1
  scaleTargetRef:
    apiVersion: extensions/v1beta1
    kind: Deployment
    name: hello-world
  targetCPUUtilizationPercentage: 50
```
以上配置的 HPA 控制器监控着 hello-world 部署中 Pod 的 CPU 利用率。控制器可以添加或删除 Pod，Pod 最少要有 1 个，最多 10 个，以将 CPU 平均利用率控制在 50％。

默认情况下，HPA 控制器会作为标准 `kube-controller-manager` 保护程序的一部分运行。它只能管理由副本控制器（Replication Controller）创建的 Pod，例如部署（Deployments）、副本集（ReplicaSet）和有状态集（StatefulSet）

控制器需要指标来源。根据 CPU 使用率而进行的伸缩，需要取决于 `metrics-server`。基于自定义指标或外部指标进行伸缩就需要部署实现 `custom.metrics.k8s.io` 或 `external.metrics.k8s.io` API 服务，以提供监控服务或备用指标的接口。
对于使用标准 CPU 指标的工作负载，容器必须在 Pod 规范中配置 CPU 资源限制。

##  2. 纵向扩容VPA
Pod 纵向自动扩缩，为 `VerticalPodAutoscaler`
默认的 Kubernetes 调度程序往往会过度使用节点上的 CPU 和预留内存，因为大多数容器会更接近其初始请求，而不是请求的上限。**纵向扩容（VPA）可以增加或减少容器的 CPU 和内存资源的请求，更好把集群资源分配到实际应用**。注意：VPA 不会改变 Pod 的资源限制值

VPA 服务可以根据实时数据设置容器资源限制，而无需依靠开发者的猜测和经验。
有时候，某些工作负载偶尔会出现利用率很高的情况，但是永久增加其资源限制将浪费大部分未使用的 CPU 或内存资源，还会限制其他运行它们的节点。虽然 Pod 水平自动伸缩（HPA）大多情况下很有用，但还是会出现工作负载无法很好地分配到应用程序多个实例的情况。

VPA 部署包含三个组件：`Recommender`，它监视资源利用率并计算目标值。Updater 驱逐需要用新资源限制更新的 Pod，而 `Admission Controller` 则使用 `mutating admission webhook` 重写了一个具有正确资源调度的 Pod。

由于 Kubernetes 不支持动态更改正在运行的 Pod 的资源限制，因此 VPA 无法使用新限制更新现有的 Pod。它会终止使用过期限制的 Pod，并且当 Pod 的控制器从 Kubernetes API 服务请求替换时，VPA 准入控制器会将更新的资源请求和限制值写入新 Pod 的规范中。

VPA 也能在 `recommendation mode` 中运行。在这种模式下，VPA Recommender 将使用建议值更新工作负载 `VerticalPodAutoscaler` 资源的 status 字段 ，但不会终止 Pod 或更改 Pod API 请求。

如果 Kubernetes 提供程序不支持将 VPA 作为附加组件，那么可以直接将其安装在集群中。
VPA 使用自定义资源 VerticalPodAutoscaler 为部署或副本集配置伸缩比例。下面是使用 VPA 管理 hello-world 部署的资源请求：

```bash
apiVersion: "autoscaling.k8s.io/v1beta2"
kind: VerticalPodAutoscaler
metadata:
  name: hello-world
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: hello-world
  resourcePolicy:
    containerPolicies:
      - containerName: '*'
        controlledResources: ["cpu", "memory"]
        minAllowed:
          cpu: 200m
          memory: 50Mi
        maxAllowed:
          cpu: 500m
          memory: 500Mi
  updatePolicy:
    updateMode: "Auto"
```
现在，VPA 会监视部署中的 Pod，并尝试设置适当的资源值，CPU 介于 200 和 500m 之间，并具有 50 到 500Mi 的内存。如果 VPA 为部署的 Pod 计算新的资源值，它将顺序驱逐现有 Pod，并用新值更新替换项。

VPA 只能替换由副本控制器（RC）管理的 Pod，例如部署 (Deployments)。它依赖 Kubernetes 的 metrics-server 插件。
如果 HPA 配置不使用 CPU 或内存来确定伸缩目标，则需要同时使用 VPA 和 HPA 来管理给定的工作负载。


##  3. 集群自动伸缩组件
在 HPA 增减集群中正在运行的 Pod 时，集群自动伸缩组件可以更改集群中的节点数量。

动态伸缩节点数量来匹配当前集群利用率可以帮助云提供商管理在平台上运行 Kubernetes 集群的成本，尤其是在为满足当前需求的工作负载而设计伸缩节点的时候。

集群自动伸缩组件会循环执行两个主要任务：监控计划外的  Pod，以及计算是否可以将当前部署的所有 Pod 合并到较少数量的节点上。

自动伸缩组件会检查集群中是否有由于 CPU 或内存资源不足、Pod 的节点关联规则、`Taints` 和 `Tolerations`，导致与现有节点不匹配而无法调度的 Pod。如果集群有不可调度的 Pod，自动伸缩组件将检查其受管节点池，以确定添加节点是否能解除对 Pod 的阻塞。如果可以解决，那会添加一个节点，节点池也随之变大。

自动伸缩组件还会扫描其管理的节点池中的节点。如果某个节点中有集群其他节点也可以调度的 Pod，自动伸缩组件会将其驱逐，并删除该节点。在决定是否移动 Pod 时，自动伸缩组件还会考虑 Pod 优先级和 Pod 中断预算（PDB）。



参考链接：
[https://cloud.google.com/kubernetes-engine/docs/concepts/verticalpodautoscaler](https://cloud.google.com/kubernetes-engine/docs/concepts/verticalpodautoscaler)

[http://bazingafeng.com/2019/04/06/k8s-vpa/](http://bazingafeng.com/2019/04/06/k8s-vpa/)
