

----

##  1. Kubernetes简介
Kubernetes是一个非常流行的开源容器编排平台，可以用来自动化部署、扩展和管理容器化的工作负载。在本章中，我们将发现Kubernetes集群的基本架构及其组件。
Kubernetes最初是由谷歌设计和开发的，在2014年得到了开源，随着1.0版Kubernetes被捐赠给新成立的云本地计算。


## 2. 学习目标
在本章结束时，你应该能够:

 1. 讨论Kubernetes的基本架构。
 2. 解释控制平面和工作节点的不同组成部分。
 3. 了解如何开始使用Kubernetes设置。
 4. 讨论Kubernetes是如何运行容器的。
 5. 讨论Kubernetes的调度概念。


##  3. Kubernetes 架构
Kubernetes经常被用作集群，这意味着它跨越多个工作在不同任务上的服务器，并分配系统的负载。这是一个基于谷歌的需求的初始设计决策，每周都有数十亿个容器启动。考虑到Kubernetes的高垂直可伸缩性，可以跨多个数据中心和区域拥有数千个服务器节点的集群。
从高层次的角度来看，Kubernetes集群由两种不同的服务器节点类型组成:

 - 控制平面节点(年代) 
 这些是行动的大脑。控制平面节点包含各种组件，这些组件管理集群并控制各种任务，如部署、调度和容器化工作负载的自修复。
 - 工作者节点
   工作节点是应用程序在集群中运行的地方。这是工作节点的唯一工作，它们没有任何进一步的逻辑实现。它们的行为，比如它们是否应该启动一个容器，完全由控制平面节点控制。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e61afca240fcc9538ee138e7778825d6.png#pic_center)

Kubernetes架构
与您为自己的应用程序选择的微服务体系结构类似，Kubernetes合并了多个需要安装在节点上的较小的服务。

master节点组件：

 - `kube-apiserver`： 这是Kubernetes的核心观点。所有其他组件都与api服务器交互，这是用户访问集群的地方。
 - `etcd`：保存集群状态的数据库。etcd是一个独立的项目，而不是Kubernetes的官方部分。
 - `kube-scheduler`：当一个新的工作负载应该被调度时，kube-scheduler会根据CPU和内存等不同属性选择一个合适的工作节点。
 - `kube-controller-manager`：包含不同的非终止控制循环，用于管理集群的状态。例如，其中一个控制循环可以确保您的应用程序的所需数量一直可用。

node节点组件：
 - 容器运行时：容器运行时负责在工作节点上运行容器。在很长一段时间里，Docker是最流行的选择，但现在被其他运行时所取代，如[containerd](https://containerd.io/)。
 - kubelet：在集群中的每个工作节点上运行的小型代理。kubelet与api服务器和容器运行时对话，以处理启动容器的最后阶段。
 - kube-proxy：处理集群内部和外部通信的网络代理。如果可能的话，kube代理尝试依赖底层操作系统的网络功能，而不是自己管理流量。

值得注意的是，这种设计使得已经在工作节点上启动的应用程序可以在控制平面不可用的情况下继续运行。虽然许多重要的功能，如缩放、调度新的应用程序等，将不可能在控制平面离线时实现。
Kubernetes还有一个名称空间的概念，不要将其与用于隔离容器的内核名称空间混淆。Kubernetes名称空间可用于将一个集群划分为多个虚拟集群，在使用mul时可用于多租户。

##  4. Kubernetes搭建
建立Kubernetes集群可以通过许多不同的方法来实现。使用正确的工具，创建一个测试“集群”非常容易:

 - [Minikube](https://minikube.sigs.k8s.io/docs/)
 - [kind](https://kind.sigs.k8s.io/)
 - [MicroK8s](https://microk8s.io/)

如果你想在自己的硬件或虚拟机上建立一个生产级的集群，你可以选择各种安装程序之一:

 - [kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/)
 - [kops](https://github.com/kubernetes/kops)
 - [kubespray](https://github.com/kubernetes-sigs/kubespray)

一些供应商开始将Kubernetes打包成一个发行版，甚至提供商业支持:

 - [Rancher](https://rancher.com/)
 - [k3s](https://k3s.io/)
 - [OpenShift](https://www.redhat.com/en/technologies/cloud-computing/openshift)
 - [VMWare Tanzu](https://tanzu.vmware.com/tanzu)

这些发行版在使用Kubernetes作为其框架的中心部分时，经常选择一种固执的方法并提供额外的工具。
如果你不想自己安装和管理它，你可以从云提供商那里使用它:

 - [Amazon (EKS)](https://aws.amazon.com/eks/)
 - [Google (GKE)](https://cloud.google.com/kubernetes-engine)
 - [Microsoft (AKS)](https://azure.microsoft.com/en-us/services/kubernetes-service/)
 - [DigitalOcean (DOKS)](https://www.digitalocean.com/products/kubernetes)

交互式教程-创建一个集群
在这个交互式教程中，你可以学习如何使用Minikube[建立你自己的Kubernetes集群](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/)。


##  5. Kubernetes API
`Kubernetes API`是Kubernetes集群中最重要的组件。没有它，就不可能与集群通信，集群本身的每个用户和每个组件都需要api-server。
访问控制概述，从[Kubernetes文档](https://kubernetes.io/docs/concepts/security/controlling-access/)中检索
在一个请求被Kubernetes处理之前，它必须经历三个阶段:

 - 身份验证
请求者需要提供一种针对API进行身份验证的方法。通常使用数字签名证书(X.509)或外部身份管理系统。Kubernetes用户总是由外部管理的。服务帐户可以用来认证技术用户。
 - 授权
它决定了请求者被允许做什么。在Kubernetes中，这可以通过基于角色的访问控制(RBAC)来实现。
 - 允许控制[添加链接描述](https://cri-o.io/)
在最后一步中，可以使用入场控制器来修改或验证请求。例如，如果用户试图使用来自不可信注册中心的容器图

准入控制器可能会阻止此请求。像[Open Policy Agent](https://www.openpolicyagent.org/)这样的工具可以用于外部管理[准入控制](https://kubernetes.io/docs/concepts/security/controlling-access/)。
与许多其他API一样，Kubernetes API是作为通过HTTPS公开的RESTful接口实现的。通过这个API，用户或服务可以创建、修改、删除或检索驻留在Kubernetes中的资源。


##  6. Kubernetes上运行容器
在本地机器上运行容器与在Kubernetes中运行容器有什么不同?在Kubernetes中，不是直接启动容器，而是将Pods定义为最小的计算单元，Kubernetes将其转换为一个运行的容器。我们将在后面学习更多关于Pods的内容，现在把它想象成一个容器的包装器。
在Kubernetes中创建Pod对象时，在获得运行节点的容器之前，会涉及到多个组件。
下面是一个使用Docker的例子:

为了允许使用其他容器运行时而不是Docker, Kubernetes在2016年引入了[容器运行时接口(CRI)](https://kubernetes.io/blog/2016/12/container-runtime-interface-cri-in-kubernetes/)。

 - `containerd`：[Containerd](https://containerd.io/)是一个运行容器的轻量级、高性能实现。可以说是目前最流行的容器运行时。Kubernetes即服务产品的所有主要云提供商都使用它。
 - CRI-O：[CRI-O](https://cri-o.io/)由Red Hat创建，具有与podman和buildah密切相关的类似代码库。
 - Docker：这个标准存在了很长一段时间，但从未真正用于容器编排。Docker作为Kubernetes运行时的使用已经被弃用，并将在Kubernetes 1.23中移除。[Kubernetes有一篇很棒的博客文章](https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/)，回答了关于这个问题的所有问题。

使用containerd创建容器要比使用Docker简单得多
`containerd`和`CRI-O`的想法非常简单:提供一个只包含运行容器绝对必要的运行时。不过，它们还有一些额外的特性，比如与容器运行时沙箱工具集成的能力。这些工具试图解决在多个容器之间共享内核所带来的安全问题。目前最常用的工具有:

 - [gvisor](https://github.com/google/gvisor)：由谷歌创建，提供了一个位于容器化进程和主机内核之间的应用程序内核。
 - [Kata Containers](https://katacontainers.io/)：提供轻量级虚拟机的安全运行时，但其行为类似于容器。

##  7. Networking
Kubernetes的网络可能非常复杂，难以理解。其中很多概念都与kubernetes无关，在容器编排一章中已经介绍过。同样，我们必须处理大量容器需要跨大量节点通信的问题。Kubernetes区分了四种需要解决的网络问题:

 1. Container-to-Container 通信
这可以通过Pod概念来解决，我们将在后面学习到。
 2. Pod-to-Pod 通信
这可以通过覆盖网络来解决。
 3. Pod-to-Service 通信
在节点上通过kube-proxy和包过滤实现
External-to-Service 通信
在节点上通过kube-proxy和包过滤（packet filter ）实现。


Kubernetes中有不同的实现网络的方式，但也有三个重要的需求:

 - 所有pod可以跨节点相互通信。
 - 所有节点可以与所有pod进行通信。
 - 没有网络地址转换(NAT)。

要实现联网，你可以从各种网络供应商中选择:

 - [Project Calico](https://www.tigera.io/project-calico/)
 - [Weave](https://www.weave.works/oss/net/)
 - [Cilium](https://cilium.io/)


在Kubernetes中，每个Pod都有自己的IP地址，所以不需要手动配置。此外，大多数Kubernetes设置包括一个名为[core-dns](https://kubernetes.io/docs/tasks/administer-cluster/coredns/)的DNS服务器附加组件，它可以在集群内提供服务发现和名称解析。


##  8. 调度（Scheduling）
在其最基本的形式中，调度是容器编排的一个子类，描述了自动选择正确的(工作人员)节点来运行容器工作负载的过程。在过去，调度更多的是手工任务，系统管理员通过跟踪可用的服务器、它们的容量和其他属性(比如它们的位置)来为应用程序选择正确的服务器。

在Kubernetes集群中，kube调度器是做出调度决策的组件，但并不负责实际启动工作负载。Kubernetes中的调度过程总是在创建一个新的Pod对象时启动。记住，Kubernetes使用的是一种声明式的方法，在这种方法中，Pod只是首先被描述，然后调度程序选择一个节点，在这个节点中，Pod实际上将由kubelet和容器运行时启动。

关于Kubernetes的一个常见误解是，它有某种形式的“人工智能”来分析工作负载，并根据资源消耗、工作负载类型和其他因素来移动pod。事实是，用户必须提供有关应用程序需求的信息，包括对CPU和内存的请求以及节点的属性。例如，用户可以请求他们的应用程序需要两个CPU内核、4g内存，最好将其安排在具有快速磁盘的节点上。
调度器将使用这些信息来过滤符合这些要求的所有节点。如果多个节点平均满足需求，Kubernetes将在Pod数量最少的节点上调度Pod。如果用户没有指定任何进一步的需求，这也是默认的行为。

可能无法建立所需的状态，例如，因为工作节点没有足够的资源来运行应用程序。在这种情况下，调度器将重试找到适当的节点，直到状态可以建立。

##  9. 其他资源
### 9.1 Kubernetes的历史和Borg的遗产

 - [From Google to the world: The Kubernetes origin story](https://cloud.google.com/blog/products/containers-kubernetes/from-google-to-the-world-the-kubernetes-origin-story), by Craig McLuckie (2016)
 - [Large-scale cluster management at Google with Borg](https://research.google/pubs/pub43438/), by Abhishek Verma,  Luis Pedrosa, Madhukar R. Korupolu, David Oppenheimer, Eric Tune,John Wilkes (2015)

### 9.2 Kubernetes 架构

 - [Kubernetes Architecture explained | Kubernetes Tutorial 15](https://www.youtube.com/watch?v=umXEmn3cMWY)

### 9.3 RBAC

 - [Demystifying RBAC in Kubernetes](https://www.cncf.io/blog/2018/08/01/demystifying-rbac-in-kubernetes/), by Kaitlyn Barnard

### 9.4 Container Runtime Interface

 - [Introducing Container Runtime Interface (CRI) in Kubernetes  (2016)](https://kubernetes.io/blog/2016/12/container-runtime-interface-cri-in-kubernetes/)

### 9.5 Kubernetes networking and CNI

 - [What is Kubernetes networking?](https://www.vmware.com/topics/glossary/content/kubernetes-networking.html)

###  9.6 Internals of Kubernetes Scheduling

 - [A Deep Dive into Kubernetes Scheduling](https://thenewstack.io/a-deep-dive-into-kubernetes-scheduling/), by Ron Sobol (2020)

### 9.7 Kubernetes Security Tools

 - [Popeye](https://github.com/derailed/popeye)
 - [kubeaudit](https://github.com/Shopify/kubeaudit)
 - [kube-bench](https://github.com/aquasecurity/kube-bench)

### 9.8 Kubernetes Playground

 - [Play with Kubernetes](https://labs.play-with-k8s.com/)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/72b081c388550d0e99f6d2f14d4e48a7.gif#pic_center)
---

✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>
 - [KCNA考试 第一章：cloud foundry云原生工程师考试](https://ghostwritten.blog.csdn.net/article/details/121482847)
 - [KCNA考试 第二章：Cloud Native Architecture](https://ghostwritten.blog.csdn.net/article/details/121492840)
 - [KCNA考试 第三章：容器编排](https://ghostwritten.blog.csdn.net/article/details/121527922)
 - [KCNA考试 第四章：kubernetes需要掌握的基础知识](https://ghostwritten.blog.csdn.net/article/details/123477851)
 - [KCNA考试 第五章：kubernetes实践](https://ghostwritten.blog.csdn.net/article/details/123496572)
 - [KCNA考试 第六章：持续交付](https://ghostwritten.blog.csdn.net/article/details/123553700)
 -  [KCNA考试 第七章：监控与探测](https://ghostwritten.blog.csdn.net/article/details/123573993)
 -  [十二个因素的应用程序](https://ghostwritten.blog.csdn.net/article/details/121496200)
 - [KCNA（云原生入门）测试题](https://ghostwritten.blog.csdn.net/article/details/123631209)
----
