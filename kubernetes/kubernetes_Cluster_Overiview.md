#  kubernetes Cluster Overiview
tags: cluster



![Kubernetes 架构](https://i-blog.csdnimg.cn/blog_migrate/8fbe404cff54a8b3ce0300d841b9ce91.png)


##  1. 什么是 Kubernetes 集群？
Kubernetes 集群 是一组运行容器化应用程序的节点。容器化应用程序将应用程序与其依赖项和一些必要的服务打包在一起。它们比虚拟机更轻巧、更灵活。通过这种方式，Kubernetes 集群允许更轻松地开发、移动和管理应用程序。 

Kubernetes 集群允许容器跨多台机器和环境运行：虚拟机、物理机、基于云的和本地。与虚拟机不同，Kubernetes 容器不限于特定的操作系统。相反，它们能够共享操作系统并在任何地方运行。

Kubernetes 集群由一个主节点和多个工作节点组成。这些节点可以是物理计算机或虚拟机，具体取决于集群。

`master nodes` 控制集群的状态；例如，哪些应用程序正在运行以及它们对应的容器映像。主节点是所有任务分配的来源。它协调流程，例如：

 - 调度 和扩展应用程序
 - 维护 集群的状态
 - 实施 更新

`worker nodes` 是运行这些应用程序的组件。 工作节点执行主节点分配的任务。它们可以是 虚拟机 或物理计算机，都作为一个系统的一部分运行。

Kubernetes 集群必须至少有一个主节点和一个工作节点才能运行。对于生产和登台，集群分布在多个工作节点上。对于测试，组件可以全部运行在同一个物理或虚拟节点上。

`namespace`  是 Kubernetes 用户在一个物理集群中组织许多不同集群的一种方式。 命名空间使用户能够通过资源配额在不同团队之间划分物理集群内的集群资源。因此，它们非常适合涉及复杂项目或多个团队的情况。 

##  2. 集群组成

Kubernetes 主要由以下几个核心组件组成：

 - etcd 保存了整个集群的状态；
 - apiserver 提供了资源操作的唯一入口，并提供认证、授权、访问控制、API 注册和发现等机制；
 - controller manager 负责维护集群的状态，比如故障检测、自动扩展、滚动更新等；
 - scheduler 负责资源的调度，按照预定的调度策略将 Pod 调度到相应的机器上；
 - kubelet 负责维护容器的生命周期，同时也负责 Volume（CSI）和网络（CNI）的管理；
 - Container runtime 负责镜像管理以及 Pod 和容器的真正运行（CRI）；
 - kube-proxy 负责为 Service 提供 cluster 内部的服务发现和负载均衡；

除了核心组件，还有一些推荐的插件，其中有的已经成为 CNCF 中的托管项目：

 - CoreDNS 负责为整个集群提供 DNS 服务
 - Ingress Controller 为服务提供外网入口
 - Prometheus 提供资源监控
 - Dashboard 提供 GUI
 - Federation 提供跨可用区的集群

##  3. 集群应用
要使用 Kubernetes 集群，您必须首先确定其所需状态。Kubernetes 集群的期望状态定义了许多操作元素，包括：

 - `Applications` and `workloads`
 - `images`
 - `Resources`
 - `replicas Quantity`

为了定义所需的状态，JSON 或 YAML 文件（称为清单）用于指定应用程序类型和运行系统所需的副本数。

开发人员使用 Kubernetes API 来定义集群的期望状态。这种开发人员交互使用命令行界面 (kubectl) 或利用 API 直接与集群交互以手动设置所需的状态。然后，主节点将通过 API 将所需状态传达给工作节点。

Kubernetes 通过 Kubernetes 控制平面自动管理集群以使其与所需状态保持一致。Kubernetes 控制平面的职责包括调度集群活动以及注册和响应集群事件。

Kubernetes 控制平面运行连续的控制循环，以确保集群的实际状态与其期望的状态相匹配。例如，如果您部署一个应用程序以使用五个副本运行，但其中一个崩溃了，Kubernetes 控制平面将注册此崩溃并部署一个额外的副本，以便保持五个副本的期望状态。

自动化通过 Pod 生命周期事件生成器或 PLEG 进行。这些自动任务可以包括：

 - `启动` 和重启容器
 - `调整` 应用程序的副本数
 - `验证` 容器镜像
 - `启动` 和管理容器
 - `实施` 更新和回滚

##  4. 集群创建

 - 手动创建 kubernetes 集群
 - minikube 创建 kubernetes 集群
 - kind  创建 kubernetes 集群
 - openshift



