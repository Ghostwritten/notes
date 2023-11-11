
![](https://img-blog.csdnimg.cn/e2a549ebaa4144b1bce5da35bedfbc10.png)



## 1. minikube 
[minikube](https://minikube.sigs.k8s.io/docs/) 是一个 Kubernetes SIG 项目，已经启动三年多了。它采用生成虚拟机的方法，该虚拟机本质上是一个单节点 K8s 集群。由于支持大量管理程序，它可以在所有主要操作系统上使用。这也允许您并行创建多个实例。

从用户的角度来看，minikube 是一个非常适合初学者的工具。您使用 启动集群`minikube start`，等待几分钟，您kubectl就可以开始了。要指定 Kubernetes 版本，您可以使用该`--kubernetes-version`标志。可在[此处](https://minikube.sigs.k8s.io/docs/handbook/config/)找到受支持版本的列表。默认情况下，Minikube 创建一个单节点集群，但您可以在启动 Minikube 时使用 `--nodes` 标志设置更多节点。

Minikube 的主要优点是它非常轻便，并且非常易于安装和使用。

Minikube 的主要缺点是它仅为测试而设计。它不是运行生产级集群的实用解决方案。

如果您是 Kubernetes 的新手，minikube 提供的对其仪表板的一流支持可能会对您有所帮助。通过一个简单`minikube dashboard`的应用程序将打开，让您很好地了解集群中发生的一切。这是通过minikube 的[插件系统](https://minikube.sigs.k8s.io/docs/handbook/deploying/)实现的，它可以帮助您将诸如[Helm](https://helm.sh/)、[Nvidia GPU](https://developer.nvidia.com/kubernetes-gpu)和[Docker Registry](https://docs.docker.com/registry/)之类的东西与您的集群集成。

- [Minikube v1.25.2 在 Centos 7.9 部署 Kubernetes v1.23.8](https://blog.csdn.net/xixihahalelehehe/article/details/123796854)
- [Ubuntu 18.04 通过 Minikube 安装 Kubernetes v1.20](https://blog.csdn.net/xixihahalelehehe/article/details/113527867)





## 2. k3s
[K3s](https://k3s.io/) 是由[Rancher Labs](https://www.rancher.com/)开发的 Kubernetes 的缩小版本。通过删除可有可无的功能（遗留、alpha、非默认、树内插件）和使用轻量级组件（例如 sqlite3 而不是 etcd3），他们实现了显着的缩减。这会生成一个大小约为 60 MB 的二进制文件。它还可以在任何操作系统（Linux、Windows 和 macOS）上运行。

如果你想将节点添加到你的集群中，你必须[单独在它们上设置 K3s 并将它们加入到你的集群中](https://pet2cattle.com/2021/04/k3s-join-nodes)。在这方面，K3s 使用起来比 Minikube 和 MicroK8s 稍微繁琐一些，两者都提供了更简单的添加节点的过程。

另一方面，K3s 被设计成一个成熟的、生产就绪的 Kubernetes 发行版，同时也是轻量级的。

一项突出的功能称为[自动部署](https://tannguyen.dev/2021/01/setting-up-a-simple-ci/cd-flow-with-k3s-and-gitlab/)。它允许您通过将 Kubernetes 清单和 Helm 图表放在特定目录中来部署它们。K3s 监视变化并在没有任何进一步交互的情况下负责应用它们。这对于 CI 管道和 IoT 设备（都是 K3s 的目标用例）特别有用。只需创建/更新您的配置，K3s 就会确保您的部署保持最新。

- k3s 一条短命令安装：`curl -sfL https://get.k3s.io | sh -`
- [k3s 离线部署指南](https://blog.csdn.net/xixihahalelehehe/article/details/127691897)

## 3. k3d
[k3d](https://k3d.io/v5.4.7/) 是一个 开源 实用程序，旨在轻松地在 docker 容器中运行高度可用的轻量级 k3s 集群。

使用 k3d，您可以轻松创建单节点和多节点 k3s 集群，以在 Kubernetes 上进行无缝本地开发和测试。通过在 Kubernetes 中运行，k3d 还可以帮助您轻松扩展和缩减工作负载。
k3d 是 k3s 的包装器，顾名思义就是 docker 上的 k3s。
它还提供了额外的功能，例如代码的热重载、构建部署和使用多服务器集群测试 Kubernetes 应用程序。
k3d 部署基于 Docker 的 k3s Kubernetes 集群，而 k3s 部署基于虚拟机的 Kubernetes 集群。
K3d 提供了一个更具可扩展性的 k3s 版本，这可能使其优于标准 k3s。
k3d 似乎是 k3s 的更灵活和改进的版本，尽管它们的功能和用法相似。
另一个不同之处是，k3s 的设计易于在生产环境中部署，这使其成为在本地环境中为生产级工作负载运行 Kubernetes 的最受欢迎的选择之一，而 k3d 更适合在更小的环境中使用，例如 Raspberry Pi、IoT、和边缘设备。

## 4. Kind
[Kind](https://kind.sigs.k8s.io/) 是另一个 Kubernetes SIG 项目，但与 minikube 相比有很大不同。顾名思义，它将集群移动到 `Docker` 容器中。与生成 `VM` 相比，这导致启动速度明显加快。

创建集群与 minikube 的方法非常相似。执行`kind create cluster`，然后你就可以开始了。通过使用不同的名称 ( --name) kind，您可以并行创建多个实例。

我个人喜欢的一个功能是能够将我的本地图像直接加载到集群中。这为我节省了一些额外的步骤，即每次我想尝试我的更改时设置注册表和推送我的图像。简单`kind load docker-image my-app:latest`来说，图像就可以在我的集群中使用。很不错！

如果您正在寻找一种以编程方式创建 Kubernetes 集群的方法，请亲切地（您一直在等待这个，不是吗 :P）发布其在后台使用的 [Go 包](https://pkg.go.dev/sigs.k8s.io/kind/pkg/cluster)。

## 5. MicroK8s
[Microk8s](https://microk8s.io/docs/)是[Canonical](https://canonical.com/)发布的一款小型、轻量级、完全符合标准的Kubernetes发行版。这款简约的发行版专注于简洁和性能。由于占用资源少，Microk8s可以轻松部署在物联网和边缘设备端。MicroK8s是目前最小、最快与Kubernetes全面兼容的集群系统，主要用于工作站和小型团队，但是目前镜像并没有与snap打包在一起，还在`gcr.io`上，国内下载上还是有问题。MicroK8s适合离线开发、原型开发和测试，尤其是运行VM作为小、便宜、可靠的k8s用于CI/CD。支持arm架构，也适合开发 IoT 应用，通过 MicroK8s 部署应用到小型Linux设备上。

Canonical已将[Microk8s 包装成 snap](https://snapcraft.io/microk8s)，这是该公司的Linux软件包管理器。snap捆绑了应用程序及无需修改即可在许多不同的Linux发行版上运行的依赖项。snap是独立的应用程序，可在沙盒中运行，通过中介访问主机系统。snap 已成为通常在基于 Debian 的发行版中使用的标准.deb软件包之外的替代方案。包装成snap的应用程序可以轻松安装和卸载。除了最新版本的Ubuntu外，snap 还可以部署在各种平台上，包括Linux Mint、Raspberry Pi OS和Arch Linux。

由于Microk8s基于snap，因此可以通过单个命令轻松部署。比如在Ubuntu 18.04上，sudo snap install microk8s --classic可安装功能完备的单节点Kubernetes集群。对于任何运行snapd(snap软件包管理器的守护程序)的平台而言，安装过程都一样

项目特点
- MicroK8轻巧 ：团队成员希望最小的Kubernetes用于笔记本电脑和工作站的开发。 MicroK8s提供了轻量级的独立Kubernetes，在Ubuntu上运行时，它与Azure AKS，Amazon EKS和Google GKE兼容。
- MicroK8很简单 ：MicroK8s通过单软件包安装来最大程度地减少管理和操作，该软件包没有活动部件（开箱即用），并且包括所有依赖项。
- MicroK8是安全的 ：对于所有安全问题，更新始终可用，并且可以立即应用或安排更新以适合企业的维护周期。 此外，MicroK8具有最新的隔离功能，可在工作站上安全运行。 通过将Kubernetes，Docker.io，iptables和CNI的所有二进制文件打包在单个snap软件包中，可以实现这种隔离。
- MicroK8是最新的 ：MicroK8s跟踪上游Kubernetes，并在上游Kubernetes发行的同一天发布beta，发行候选版本和最终版本。 您可以跟踪最新的Kubernetes或坚持使用从1.10开始的任何Kubernetes版本。 当出现新的主要Kubernetes版本时，您可以自动升级或使用单个命令进行升级。
- MicroK8是全面的 ：MicroK8s包括精选的清单，用于常见的Kubernetes功能和服务。 MicroK8带有Docker注册表，使用户可以在笔记本电脑上制作，推送和部署容器。

运行环境
操作系统 Ubuntu 18.04 LTS 或16.04 LTS 环境 (或其他支持 snapd 的操作系统- see the snapd documentation)。至少 20G 磁盘空间， （建议）4G 内存。

MicroK8s 的一个不错的特性是，只要集群节点总数达到或超过三个，它就会自动将您的集群配置为高可用性（意味着它有多个主节点）。

总体而言，MicroK8s 使用起来比 K3s 或 Minikube 稍微复杂一些，特别是因为它具有模块化架构并且默认情况下仅运行最少的服务集。要打开 DNS 支持或基于 Web 的仪表板等功能，您必须明确启动它们。

- 开源地址：[https://github.com/canonical/microk8s](https://github.com/canonical/microk8s)


 ||	minikube	|	k3s|kind|
 |--|--|--|--|
|runtime|	VM|		native|container|
supported architectures|	AMD64	|	AMD64, ARMv7, ARM64|AMD64|
|supported container runtimes|	Docker,CRI-O,containerd,gvisor|	Docker, containerd|Docker|	
|startup time initial/following	|5:19 / 3:15|		0:15 / 0:15|2:48 / 1:06|
|memory requirements|	2GB	|512 MB|8GB (Windows, MacOS)	|
|requires root?	|no|		yes (rootless is experimental)|no|
|multi-cluster support|	yes|	no (can be achieved using containers)|yes	|
|multi-node support	|no|yes|yes	|
|project page	|minikube	|k3s|kind	|

参考：
- [K3d vs k3s vs Kind vs Microk8s vs Minikube](https://thechief.io/c/editorial/k3d-vs-k3s-vs-kind-vs-microk8s-vs-minikube/)
- [Minikube vs. kind vs. k3s - What should I use?](https://shipit.dev/posts/minikube-vs-kind-vs-k3s.html)
