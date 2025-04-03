![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7fd24bd38f223e838cff780882f59ba3.png)


[containerd](https://containerd.io/)（官方品牌名称小写）是开源容器化平台[Docker](https://www.docker.com/)的容器运行时。容器运行时是可以在主机操作系统上运行容器的软件组件。 

将 containerd 与 Docker 进行比较有点像将涡轮增压器与发动机进行比较，或者将空调系统与房屋进行比较。两者之间没有直接比较，尽管它们肯定相关并且几乎总是一起使用，但通常不能互换。

## 1. What Is Docker?
尽管它已成为“container”一词的同义词，但 Docker 本身并不是容器，而是一种非常流行的开发人员工具，用于创建、使用和管理容器。换句话说，它是一个容器引擎，一个允许在一个计算环境中开发的代码在另一个计算环境中工作的系统。这样看来，Docker 确实是应用程序开发的促进者和使能者。它是一个软件平台，可以简化构建、运行、管理和分发应用程序的过程，它通过虚拟化安装和运行它的计算机的操作系统来实现这一点。 

容器是 Docker 和其他容器引擎（例如 CRI-O、RKT 和 LXD）用来打包、运送和运行应用程序的可移植、独立的环境。容器运行时，例如 Docker 开发的容器运行时 containerd，是容器引擎的一个组件，它挂载容器并与操作系统内核一起启动和支持容器化进程。 

## 2. What Is containerd?
containerd 是 Docker 开发的容器运行时，用于管理容器在物理机或虚拟机（即主机）上的生命周期。它创建、启动、停止和销毁容器。它还可以从容器注册表中提取容器映像、安装存储并为容器启用网络。

2019 年 2 月，与 Kubernetes、Prometheus、Envoy 和 CoreDNS 一样，containerd 成为云原生计算基金会 (CNCF) 的[正式项目](https://www.cncf.io/announcements/2019/02/28/cncf-announces-containerd-graduation/)。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3120b05a905220e7f267dc90ab955852.png)

## 3. containerd 是如何工作的
containerd 是一个守护进程，这意味着它是一个作为后台进程运行的计算机程序，而不是在交互式用户的直接控制下。它适用于 Linux 和 Windows。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/68fecb8527d1bbb7a732cb7f91599a57.png)
containerd 管理其主机系统的完整容器生命周期——从图像传输和存储到容器执行和监督，再到低级存储再到网络附件等等。

## 4. containerd 与 CRI-O
作为 containerd 的替代方案，CRI-O 是另一个实现容器运行时接口 (CRI) 的高级容器运行时。它从注册表中提取容器映像，在磁盘上管理它们，并启动较低级别的运行时来运行容器进程。由于它们都是容器运行时，您通常不会将容器与 CRI-O 一起使用，因为您只需要一个或另一个。 


## 5. 常见问题
Docker 还在使用 containerd 吗？ 
Docker 设计了 ​​containerd，它现在是 CNCF 的一部分，CNCF是一个支持基于 Kubernetes 和 Docker 的部署的组织。Docker 仍然是一个使用 containerd 作为其运行时的独立项目。 

我可以使用 containerd 而不是 Docker 吗？
是的——尽管 containerd 是一个容器运行时而 Docker 是一个容器引擎，但这是可能的。Docker 是一种工具，它告诉容器运行时（在本例中为 containerd）基于容器映像创建容器。尽管主机操作系统没有容器的概念，但它确实提供了使容器成为可能的命名空间、cgroup 和文件系统覆盖等功能。这意味着可以将 containerd 与另一个称为低级运行时的组件一起使用，以完成与主机操作系统内核交互的工作以创建容器，并在此过程中承担 Docker 的功能。 

containerd 与 Docker 兼容吗？
是的。containerd 由 Docker 设计，与 Docker 完全兼容。 

我可以从 Docker 迁移到 containerd 吗？
是的你可以。
