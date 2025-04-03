#  kubernetes RuntimeClass 原理
tags: 对象,RuntimeClass
<!-- catalog: ~原理~ -->


[
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e581917cb24bda64c23a4e499a0c51d7.png)](https://movie.douban.com/subject/1291560/)

*《龙猫》*



----
## 1. 简介
`RuntimeClass`是用于选择容器运行时配置的功能。容器运行时配置用于运行Pod的容器。

可以在不同的Pod之间设置不同的`RuntimeClass`，以实现性能与安全性之间的平衡。例如，如果您的部分工作负荷应得到较高级别的信息安全保证，则可以选择安排这些Pod，以便它们在使用硬件虚拟化的容器运行时中运行。然后，您将受益于备用运行时的额外隔离，但要付出一些额外的开销。

您还可以使用`RuntimeClass`在相同的容器运行时但使用不同的设置来运行不同的Pod。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/277a996ce351e389d250821a6e78cc2b.png)

此同时，越来越多的容器运行时也想接入到 Kubernetes 中。如果还是按 rkt 和 Docker 一样内置支持的话，会给 Kubernetes 的代码维护和质量保障带来严重挑战。

社区也意识到了这一点，所以在 `1.5` 版本时推出了 `CRI`，它的全称是 `Container Runtime Interface`。这样做的好处是：实现了运行时和 `Kubernetes` 的解耦，社区不必再为各种运行时做适配工作，也不用担心运行时和 Kubernetes 迭代周期不一致所带来的版本维护问题。比较典型的，比如 containerd 中的 `cri-plugin` 就实现了 `CRI`、`kata-containers`、`gVisor` 这样的容器运行时只需要对接 containerd 就可以了。


随着越来越多的容器运行时的出现，不同的容器运行时也有不同的需求场景，于是就有了多容器运行时的需求。但是，如何来运行多容器运行时还需要解决以下几个问题：

 - 集群里有哪些可用的容器运行时？
 - 如何为 Pod 选择合适的容器运行时？
 - 如何让 Pod 调度到装有指定容器运行时的节点上？
 - 容器运行时在运行容器时会产生有一些业务运行以外的额外开销，这种「额外开销」需要怎么统计？


## 2. RuntimeClass 的工作流程
了解决上述提到的问题，社区推出了 `RuntimeClass`。它其实在 `Kubernetes v1.12` 中就已被引入，不过最初是以 CRD 的形式引入的。`v1.14` 之后，它又作为一种内置集群资源对象 `RuntimeClass` 被引入进来。`v1.16` 又在 `v1.14` 的基础上扩充了 `Scheduling` 和 `Overhead` 的能力。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/64710da5986373a4b8949d243cc37592.png)

下面以 `v1.16` 版本为例，讲解一下 `RuntimeClass` 的工作流程。如上图所示，左侧是它的工作流程图，右侧是一个 YAML 文件。

YAML 文件包含两个部分：上部分负责创建一个名字叫 runv 的 `RuntimeClass` 对象，下部分负责创建一个 Pod，该 Pod 通过 `spec.runtimeClassName` 引用了 runv 这个 `RuntimeClass`。

`RuntimeClass` 对象中比较核心的是 handler，它表示一个接收创建容器请求的程序，同时也对应一个容器运行时。比如示例中的 Pod 最终会被 runv 容器运行时创建容器；`scheduling` 决定 Pod 最终会被调度到哪些节点上。

结合左图来说明一下 RuntimeClass 的工作流程：

 - K8s-master 接收到创建 Pod 的请求；
 - 方格部分表示三种类型的节点。每个节点上都有 `Label` 标识当前节点支持的容器运行时，节点内会有一个或多个 `handler`，每`handler` 对应一种容器运行时。比如第二个方格表示节点内有支持 `runc` 和 runv 两种容器运行时的`handler`；第三个方格表示节点内有支持 `runhcs` 容器运行时的 `handler`；
 - 根据 `scheduling.nodeSelector`, Pod 最终会调度到中间方格节点上，并最终由 `runv handler`
   来创建 Pod。

## 3. RuntimeClass 功能介绍
### 3.1 RuntimeClass结构体定义
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7b1a3d959a680761e41e35efaaa3e3ea.png)

我们还是以 `Kubernetes v1.16` 版本中的 `RuntimeClass` 为例。首先介绍一下 `RuntimeClass` 的结构体定义。

一个 `RuntimeClass` 对象代表了一个容器运行时，它的结构体中主要包含 `Handler`、`Overhead`、`Scheduling` 三个字段。

 - 在之前的例子中我们也提到过 Handler，它表示一个接收创建容器请求的程序，同时也对应一个容器运行时；

 - Overhead 是 v1.16 中才引入的一个新的字段，它表示 Pod 中的业务运行所需资源以外的额外开销；

 - 第三个字段Scheduling 也是在 v1.16 中被引入的，该 Scheduling 配置会被自动注入到 Pod 的
   nodeSelector 中。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2ae18b6874743a1fafb0163d38aaf6fa.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8e96c0458b2b5be5c1fd02619f8a4303.png)

在 Pod 中引用 `RuntimeClass` 的用法非常简单，只要在 `runtimeClassName` 字段中配置好 RuntimeClass 的名字，就可以把这个 `RuntimeClass` 引入进来。


### 3.2 Scheduling 结构体的定义
Scheduling 表示调度，但这里的调度不是说 RuntimeClass 对象本身的调度，而是会影响到引用了 RuntimeClass 的 Pod 的调度。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9a45a253b8e4cae10e98a2a5d10794d0.png)

`Scheduling` 中包含了两个字段，`NodeSelector` 和 `Tolerations`。这两个和 Pod 本身所包含的 NodeSelector 和 Tolerations 是极为相似的。

NodeSelector 代表的是支持该 RuntimeClass 的节点上应该有的 label 列表。一个 Pod 引用了该 RuntimeClass 后，RuntimeClass admission 会把该 label 列表与 Pod 中的 label 列表做一次合并。如果这两个 label 中有冲突的，会被 admission 拒绝。这里的冲突是指它们的 key 相同，但是 value 不相同，这种情况就会被 admission 拒绝。另外需要注意的是，RuntimeClass 并不会自动为 Node 设置 label，需要用户在使用前提前设置好。

Tolerations 表示 RuntimeClass 的容忍列表。一个 Pod 引用该 RuntimeClass 之后，admission 也会把 toleration 列表与 Pod 中的 toleration 列表做一个合并。如果这两处的 Toleration 有相同的容忍配置，就会将其合并成一个。

### 3.3 为什么引入 Pod Overhead？
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3f6a8b9d76af0677308d782696690931.png)

上图左边是一个 `Docker Pod`，右边是一个 `Kata Pod`。我们知道，Docker Pod 除了传统的 container 容器之外，还有一个 pause 容器，但我们在计算它的容器开销的时候会忽略 pause 容器。对于 Kata Pod，除了 container 容器之外，`kata-agent`, `pause`, `guest-kernel` 这些开销都是没有被统计进来的。像这些开销，多的时候甚至能超过 100MB，这些开销我们是没法忽略的。

这就是我们引入 `Pod Overhead` 的初衷。它的结构体定义如下：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/081801b7254da260141d384f2246d92e.png)

它的定义非常简单，只有一个字段 `PodFixed`。它这里面也是一个映射，它的 key 是一个 `ResourceName`，value 是一个 Quantity。每一个 Quantity 代表的是一个资源的使用量。因此 PodFixed 就代表了各种资源的占用量，比如 CPU、内存的占用量，都可以通过 PodFixed 进行设置。

### 3.4 Pod Overhead 的使用场景与限制
Pod Overhead 的使用场景主要有三处：

 1. Pod 调度

在没有引入 `Overhead` 之前，只要一个节点的资源可用量大于等于 Pod 的 `requests` 时，这个 Pod 就可以被调度到这个节点上。引入 Overhead 之后，只有节点的资源可用量大于等于 `Overhead` 加上 `requests` 的值时才能被调度上来。

 1. ResourceQuota

它是一个 `namespace` 级别的资源配额。假设我们有这样一个 namespace，它的内存使用量是 1G，我们有一个 requests 等于 500 的 Pod，那么这个 namespace 之下，最多可以调度两个这样的 Pod。而如果我们为这两个 Pod 增添了 200MB 的 Overhead 之后，这个 namespace 下就最多只可调度一个这样的 Pod。

 1. Kubelet Pod 驱逐

引入 Overhead 之后，Overhead 就会被统计到节点的已使用资源中，从而增加已使用资源的占比，最终会影响到 Kubelet Pod 的驱逐。

以上是 Pod Overhead 的使用场景。除此之外，Pod Overhead 还有一些使用限制和注意事项：

-  `Pod Overhead` 最终会永久注入到 Pod 内并且不可手动更改。即便是将 RuntimeClass 删除或者更新，Pod Overhead 依然存在并且有效；
- Pod Overhead 只能由 `RuntimeClass admission` 自动注入（至少目前是这样的），不可手动添加或更改。如果这么做，会被拒绝；
- HPA 和 VPA 是基于容器级别指标数据做聚合，Pod Overhead 不会对它们造成影响。


## 4. 多容器运行时示例
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/652c4214609c52c022741daf85538685.png)

目前阿里云 `ACK` 安全沙箱容器已经支持了多容器运行时，我们以上图所示环境为例来说明一下多容器运行时是怎么工作的。

如上图所示有两个 Pod，左侧是一个 `runc` 的 Pod，对应的 RuntimeClass 是 runc，右侧是一个 `runv` 的Pod，引用的 RuntimeClass 是 runv。对应的请求已用不同的颜色标识了出来，蓝色的代表是 runc 的，红色的代表是 runv 的。图中下半部分，其中比较核心的部分是 containerd，在 containerd 中可以配置多个容器运行时，最终上面的请求也会到达这里进行请求的转发。

我们先来看一下 runc 的请求，它先到达 `kube-apiserver`，然后 kube-apiserver 请求转发给 `kubelet`，最终 kubelet 将请求发至 `cri-plugin`（它是一个实现了 CRI 的插件），cri-plugin 在 containerd 的配置文件中查询 runc 对应的 `Handler`，最终查到是通过 `Shim API runtime v1` 请求 `containerd-shim`，然后由它创建对应的容器。这是 runc 的流程。

runv 的流程与 runc 的流程类似。也是先将请求到达 kube-apiserver，然后再到达 kubelet，再把请求到达 cri-plugin，cri-plugin 最终还回去匹配 containerd 的配置文件，最终会找到通过 Shim API runtime v2 去创建 containerd-shim-kata-v2，然后由它创建一个 Kata Pod。

下面我们再看一下 `containerd` 的具体配置。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dfff92cee52d769e4101ca18b9d943d4.png)

`containerd` 默认放在 `[file:///etc/containerd/config.toml]()` 这个位置下。比较核心的配置是在 `plugins.cri.containerd` 目录下。其中 runtimes 的配置都有相同的前缀 `plugins.cri.containerd.runtimes`，后面有 runc, runv 两种 RuntimeClass。这里面的 runc 和 runv 和前面 RuntimeClass 对象中 Handler 的名字是相对应的。除此之外，还有一个比较特殊的配置 `plugins.cri.containerd.runtimes.default_runtime`，它的意思是说，如果一个 Pod 没有指定 RuntimeClass，但是被调度到当前节点的话，那么就默认使用 runc 容器运行时。

下面的例子是创建 runc 和 runv 这两个 RuntimeClass 对象，我们可以通过 kubectl get runtimeclass 看到当前所有可用的容器运行时
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/88bfb52c34acaeb31a339fcc6c9716f8.png)

下图从左至右分别是一个 runc 和 runv 的 Pod，比较核心的地方就是在 `runtimeClassName` 字段中分别引用了 runc 和 runv 的容器运行时。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/94787512d840bd6a36250bc8d4686646.png)

最终将 Pod 创建起来之后，我们可以通过 kubectl 命令来查看各个 Pod 容器的运行状态以及 Pod 所使用的容器运行时。我们可以看到现在集群中有两个 Pod：一个是 runc-pod，另一个是 runv-pod，分别引用的是 runc 和 runv 的 RuntimeClass，并且它们的状态都是 Running。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/84b81a669bd71943b9138e10ff9214a0.png)
## 5. 总结
本文的主要内容就到此为止了，这里为大家简单总结一下：

 - RuntimeClass 是 Kubernetes 一种内置的集群资源，主要用来解决多个容器运行时混用的问题；
 - RuntimeClass 中配置 Scheduling 可以让 Pod 自动调度到运行了指定容器运行时的节点上。但前提是需要用户提前为这些Node 设置好 label；
 - RuntimeClass 中配置 `Overhead`，可以把 Pod中业务运行所需以外的开销统计进来，让调度、ResourceQuota、Kubelet Pod 驱逐等行为更准确

参考：

 - [容器运行时类（Runtime Class](https://kubernetes.io/zh-cn/docs/concepts/containers/runtime-class/)）
 - [理解 RuntimeClass 与使用多容器运行时](https://cloud.tencent.com/developer/news/609235)
 - [Getting Started with Kubernetes | Understanding Kubernetes RuntimeClass and Using Multiple Container Runtimes](https://www.alibabacloud.com/blog/getting-started-with-kubernetes-%7C-understanding-kubernetes-runtimeclass-and-using-multiple-container-runtimes_596341)
 - [KEP-585: Runtime Class](https://github.com/kubernetes/enhancements/blob/master/keps/sig-node/585-runtime-class/README.md)
 - [Oracle create a runtime class](https://docs.oracle.com/en/operating-systems/olcne/1.1/runtimes/runtime-class.html)
 - [Openshift RuntimeClass](https://docs.openshift.com/container-platform/4.7/rest_api/node_apis/runtimeclass-node-k8s-io-v1.html)
