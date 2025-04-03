

## 1. 概述  
在本章中，我们将了解容器编排的挑战和机遇，以及它对网络和存储有特殊要求的原因。  
 
在开始讨论容器的编排之前，让我们先后退几步，了解什么是容器，以及是什么使它在构建和操作应用程序时如此有用。  

##  2. 学习目标  
在本章结束时，你应该能够:  
 

 - 解释什么是容器，以及它与虚拟机的区别。
 - 描述容器编排的基本原理。
 - 讨论容器网络和存储的挑战。
 - 解释服务网格的好处。

##  3. 使用容器
应用程序开发的历史也是将这些应用程序打包为不同平台和操作系统的历史。
让我们以一个用流行的编程语言Python编写的简单web应用程序为例。要在服务器或本地机器上运行应用程序，系统通常需要满足以下特定要求:

 - 安装和配置基本操作系统
 - 安装核心Python包来运行程序
 - 安装程序使用的Python扩展
 - 为您的系统配置网络
 - 连接到第三方系统，如数据库、缓存或存储。

虽然开发人员最了解自己的应用程序及其依赖关系，但通常是由系统管理员提供基础设施、安装所有依赖关系并配置应用程序运行的系统。这个过程非常容易出错，而且很难维护，因此服务器只用于单一目的配置，比如运行数据库或应用服务器，然后通过网络连接。

为了更有效地利用服务器硬件，可以使用虚拟机来模拟一个具有cpu、内存、存储、网络、操作系统和软件的完整服务器。这允许在同一硬件上运行多个隔离的服务器。

在广泛采用容器之前，服务器虚拟化是运行独立且易于处理的应用程序的最有效方式，但由于必须运行包括内核在内的整个操作系统，如果需要运行大量服务器，那么它总是会带来一些开销。

容器可以用来解决这两个问题，管理应用程序的依赖关系，并比旋转大量虚拟机更有效地运行。

##  4. 容器基础
与普遍的看法相反，容器技术比人们预期的要古老得多。现代容器技术的最早祖先之一是`chroot`命令，它于1979年在Version 7 Unix中引入。chroot命令可用于将进程与根文件系统隔离开来，并基本上将文件从进程中“隐藏”起来，并模拟一个新的根目录。隔离环境是所谓的chroot监狱，在这种环境中，进程无法访问文件，但文件仍然存在于系统中。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c9ee23179e2c1119066ab93d985c02d9.png)
**可以在文件系统的不同位置创建Chroot目录**

虽然chroot是一项相当古老的技术，但它仍然在一些流行的软件项目中使用。我们今天拥有的容器技术仍然体现了这一概念，但是是一个现代化的版本，并且有很多特性。
为了比chroot更能隔离进程，当前的Linux内核提供了像命名空间和cgroup这样的特性。
命名空间用于隔离各种资源，例如网络。可以使用网络名称空间提供网络接口和路由表的完整抽象。这允许进程拥有自己的IP地址。Linux Kernel 5.6目前提供了8个命名空间:

 - `pid`-进程ID提供了一个具有自己的进程ID集的进程。
 - `net` - network允许进程拥有自己的网络堆栈，包括IP地址。
 - `MNT` - mount抽象文件系统视图并管理挂载点。
 - `Ipc` -进程间通信提供了命名共享内存段的分离。
 - `user` -为进程提供自己的一组用户id和组id。
 - `uts` - Unix时间共享允许进程拥有自己的主机名和域名。
 - `cgroup`-一个新的命名空间，允许进程拥有自己的一组Cgroup根目录。
 - `time`-最新的命名空间可以用来虚拟化系统的时钟。

cgroup被用来组织层次结构组中的进程，并为它们分配资源，如内存和CPU。当你想限制你的应用程序容器的内存，比方说4GB, cgroup被用来确保这些限制。
Docker于2013年推出，成为building and running containers的代名词。虽然Docker并没有发明用于运行容器的技术，但他们以一种智能的方式将现有的技术结合在一起，使容器对用户更加友好和方便。
乍一看，容器似乎与虚拟机非常相似，但是理解它们之间的差异是至关重要的。虽然虚拟机模拟一个完整的机器，包括操作系统和内核，但容器共享主机的内核，正如所述，它们只是独立的进程。
虚拟机会带来一些开销，比如启动时间、大小或运行操作系统的资源使用情况。另一方面，容器实际上是进程，就像您可以在机器上启动的浏览器一样，因此它们启动得更快，占用的空间也更小。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ab87205213bc9c7f62c04b8ffea497b3.png)
**传统部署vs虚拟化部署vs容器部署**
在许多情况下，这不是使用容器或虚拟机的问题，而是使用这两种技术来从容器的效率中获益，但仍然使用虚拟机的更大隔离带来的安全优势。

##  5. 运行容器
要运行行业标准的容器，你不需要使用Docker;您可以只遵循OCI [runtime-spec](https://github.com/opencontainers/runtime-spec) 规范标准。Open Container Initiative还维护一个名为[runC](https://github.com/opencontainers/runc)的容器运行时引用实现。这种低级运行时被用于各种工具来启动容器，包括Docker本身。
如果您是一名开发人员，并且了解面向对象编程，您可以想象容器映像和运行容器之间的关系，就像一个类，以及该类的实例化。
安装Docker后，你可以像这样启动容器:

```bash
docker run nginx
```


`runtime-spec`与`image-spec`密切相关，我们将在下一章中讨论`image-spec`,因为它描述了如何解压容器映像，然后管理整个容器生命周期，从创建容器环境，到启动进程、停止和删除它。
对于您的本地机器，有很多可供选择的替代方案，其中一些仅用于构建映像，如[buildah](https://buildah.io/)或[kaniko](https://github.com/GoogleContainerTools/kaniko)，而其他的则作为Docker的完全替代方案，如[podman](https://podman.io/)。
Podman提供了与Docker相似的API，可以作为替代。此外，它还提供了一些额外的特性，比如运行没有根权限的容器，以及我们将在后面发现的Pod概念的使用。

## 6. 构建镜像
为什么容器首先被称为容器?这里使用的隐喻是针对按照ISO 668标准化的集装箱的使用。海运集装箱的标准格式使得它可以很容易地堆放在集装箱船上，无论里面装的是什么，都可以很容易地用起重机卸载或装上卡车。您将看到容器和云原生世界中的许多术语都遵循这个航海主题。
Docker重用了所有组件来隔离进程，比如`namespaces`和`cgroup`，但是帮助容器实现突破的关键是容器映像的引入。
容器映像使容器具有可移植性，并且易于在各种系统上重用。Docker对容器镜像的描述如下:
Docker容器镜像是一个轻量级的、独立的、可执行的软件包，它包含运行应用程序所需的一切:代码、运行时、系统工具、系统库和设置。
2015年，[Docker](https://www.docker.com/resources/what-container)使其流行起来的图像格式被捐赠给了新成立的开放容器倡议组织，也被称为 [OCI image-spec](https://github.com/opencontainers/image-spec)，可以在GitHub上找到。映像由文件系统包和元数据组成。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1457ccb6eaeb76416bc825c4eccb3c9d.png)
Container Images
可以通过从一个名为`Dockerfile`的构建文件中读取说明来构建映像。这些说明几乎与在服务器上安装应用程序时使用的说明相同。下面是一个Dockerfile的例子，它包含了一个Python脚本:

```bash
# Every container image starts with a base image.
# This could be your favorite linux distribution
FROM ubuntu:20.04 

# Run commands to add software and libraries to your image
# Here we install python3 and the pip package manager
RUN apt-get update && \
    apt-get -y install python3 python3-pip 

# The copy command can be used to copy your code to the image
# Here we copy a script called "my-app.py" to the containers filesystem
COPY my-app.py /app/ 

# Defines the workdir in which the application runs
# From this point on everything will be executed in /app
WORKDIR /app

# The process that should be started when the container runs
# In this case we start our python app "my-app.py"
CMD ["python3","my-app.py"]
```
如果你已经在你的机器上安装了Docker，你可以用下面的命令来构建镜像:

```bash
docker build -t my-python-image -f Dockerfile
```
使用`-t my-python-image`参数可以为映像指定一个名称标记，使用`-f Dockerfile`可以指定可以找到`Dockerfile`的位置。这使开发人员能够管理应用程序的所有依赖项，并将其打包以便运行，而不是将该任务留给另一个人或团队。
要分发这些images，可以使用container registry。这只不过是一个你可以上传和下载图片的网络服务器。Docker有内置的推和拉命令:

```bash
docker push my-registry.com/my-python-image
docker pull my-registry.com/my-python-image
```

 - Demo: Building Container Images
[https://github.com/docker/getting-started.git](https://github.com/docker/getting-started.git)

##  7. 容器安全
理解容器与虚拟机有不同的安全需求是很重要的。很多人依赖于容器的隔离特性，但这可能是非常危险的。
当容器在一台机器上启动时，它们总是共享同一个内核，如果允许容器调用内核函数(例如杀死其他进程或通过创建路由规则修改主机网络)，这就会对整个系统构成风险。您可以在[Docker文档](https://docs.docker.com/engine/security/#linux-kernel-capabilities)中了解更多关于内核功能的信息。
最大的安全风险之一(不仅是在容器领域)是执行具有太多特权的进程，特别是以root或管理员身份启动进程。不幸的是，这个问题在过去被忽视了很多，有很多容器作为根用户运行。
容器引入的一个相当新的攻击面是公共图像的使用。两个最流行的公共映像注册中心是[Docker Hub](https://hub.docker.com/)和[Quay](https://quay.io/)，虽然它们提供了公共访问的映像，但您必须确保这些映像没有被修改成包含恶意软件。
Sysdig有一篇关于[如何避免大量安全问题和构建安全容器映像](https://sysdig.com/blog/dockerfile-best-practices/)的很棒的博客文章。
一般来说，安全性并不是只能在容器层实现的。这是一个持续的过程，需要随时调整。云本地安全性的4C可以大致说明在使用容器时需要保护哪些层。确保覆盖每一层，因为它有效地保护了内部的一层。Kubernetes文档是理解层的一个很好的起点。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1c91ce616698b72cf39d03931f562ba0.png)
The 4C's of Cloud Native Security, retrieved from the [Kubernetes documentation](https://kubernetes.io/docs/concepts/security/overview/) 

##  8. 容器编排
在本地机器或单个服务器上运行几个容器是相当容易的，但是容器的使用方式带来了关于容器操作的新挑战。这个概念的高效率导致应用程序和服务变得越来越小，您会发现现代应用程序可以由许多容器组成。
拥有大量松散耦合、隔离和独立的小容器是所谓的微服务体系结构的基础。这些小容器是自包含的业务逻辑的小部分，它们是更大的应用程序的一部分。
如果必须管理和部署大量容器，那么很快就需要一个系统来帮助管理这些容器。需要解决的问题包括:

 - 提供可以运行容器的虚拟机之类的计算资源
 - 以一种有效的方式将容器调度到服务器
 - 为容器分配CPU和内存等资源
 - 管理容器的可用性，并在它们失败时更换它们
 - 如果负载增加，缩放容器
 - 提供网络将容器连接在一起
 - 如果容器需要持久化数据，则提供存储。

容器编排系统提供了一种方法来构建多个服务器的集群，并在其上托管容器。大多数容器编排系统由两部分组成:**负责管理容器的控制平面和实际托管容器的工作节点**。
多年来，已经有几种系统可以用于编排，但大多数系统在今天已经不再重要，行业已经选择Kubernetes作为编排容器的标准系统。

##   9. 容器网络
微服务架构在很大程度上依赖于网络通信。与单片应用程序不同，微服务实现了一个接口，可以调用该接口来发出请求。例如，在电子商务应用程序中，您可以有一个响应产品列表的服务。
网络命名空间允许每个容器拥有自己唯一的IP地址，因此多个应用程序可以打开相同的网口;例如，您可以有多个容器化的web服务器，它们都开放端口8080。
为了使应用程序可以从主机系统外部访问，容器能够将容器中的一个端口映射到主机系统中的一个端口。
为了允许容器跨主机进行通信，我们可以使用`overlay network`，将它们放在跨越主机系统的虚拟网络中。
这使得容器之间的通信非常容易，而系统管理员不需要在主机和容器之间配置复杂的网络和路由。
大多数覆盖网络还需要处理IP地址管理，如果手动实现，这将是大量的工作。在这种情况下，覆盖网络管理哪个容器获得哪个IP地址，以及流量如何流动以访问各个容器。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/79853f21549219e2f2bc28ba5a0e027a.png)
大多数现代的容器网络实现都基于[容器网络接口(CNI)](https://github.com/containernetworking/cni)。CNI是一个可以用来编写或配置网络插件的标准，并且可以很容易地在各种容器编排平台上交换不同的插件。


##  10. 服务发现 & DNS
在很长一段时间里，传统数据中心的服务器管理是可以管理的。许多系统管理员甚至记得他们必须使用的重要系统的所有IP地址。大量的服务器列表、它们的主机名、IP地址和用途——都是手动维护的——都是日常事务。
在容器编排平台中，事情要复杂得多:

 - 成百上千个带有独立IP地址的容器
 - 容器被部署在不同的主机、不同的数据中心甚至地理位置上
 - 容器或服务需要DNS进行通信。使用IP地址几乎是不可能的
 - 关于container的信息在删除时必须从系统中删除。

解决这个问题的方法还是自动化。所有信息都放在Service Registry中，而不是手工维护的服务器列表(在本例中是容器)。在网络中查找其他服务并请求关于它们的信息称为服务发现（Service Discovery）。

 - 拥有服务API的现代DNS服务器可以用于在创建新服务时注册它们。这种方法非常简单，因为大多数组织已经拥有具有适当功能的DNS服务器。
 - 使用高度一致的数据存储，特别是用于存储关于服务的信息。许多系统能够通过强大的故障转移机制进行高可用性操作。流行的选择，特别是对于集群来说，是[etcd](https://github.com/etcd-io/etcd)、[Consul](https://www.consul.io/)或[Apache Zookeeper](https://zookeeper.apache.org/)。

##  11. 服务网格（Service Mesh）
由于网络是微服务和容器如此重要的一部分，因此对于开发人员和管理员来说，网络可能变得非常复杂和不透明。除此之外，当容器彼此通信时，还需要**监视**、**访问控制**或**网络流量加密**等许多功能。
不必在应用程序中实现所有这些功能，只需启动第二个实现了这些功能的容器。用来管理网络流量的软件叫做代理。这是一个位于客户机和服务器之间的服务器应用程序，可以在网络流量到达服务器之前修改或过滤网络流量。受欢迎的代表是[nginx](https://www.nginx.com/), [haproxy](http://www.haproxy.org/)或[envoy](https://www.envoyproxy.io/)。
更进一步，服务网格将代理服务器添加到架构中的每个容器中。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a77f2659acda527dcf0c16409ea6f566.png)
从[Istio .io](https://istio.io/v1.10/docs/ops/deployment/architecture/)中检索
现在可以使用代理来处理服务之间的网络通信。
让我们以加密为例。如果两个或多个应用程序在彼此通信时应该加密它们的通信，那么就需要添加库、配置和管理数字证书，以证明所涉及的应用程序的身份。这可能是大量的工作，而且如果不特别小心的话，也可能容易出错。
当使用服务网格时，应用程序不直接相互通信，而是通过代理路由流量。目前最流行的服务网格是[istio](https://istio.io/)和[linkerd](https://linkerd.io/)。虽然它们在实现上有差异，但架构是相同的。
服务网格中的代理构成了数据平面。这是实现网络规则和塑造流量流的地方。
这些规则在服务网格的控制平面中集中管理。在这里，您可以定义流量如何从服务A流向服务B，以及应该对代理应用哪些配置。
因此，无需编写代码并安装库，只需编写一个配置文件，在其中告诉服务网格，服务a和服务B应该始终加密通信。然后，配置被上传到控制平面，并被分发到数据平面以执行新规则。
很长一段时间以来，术语“服务网格”只描述了容器平台中如何使用代理处理流量的基本思想。服务网格接口([Service Mesh Interface, SMI](https://smi-spec.io/))项目旨在定义一个关于如何实现来自不同提供者的服务网格的规范。他们非常关注Kubernetes，他们的目标是标准化服务网格的最终用户体验，以及为希望与Kubernetes集成的提供商提供一个标准。你可以在GitHub上找到当前的[规范](https://github.com/servicemeshinterface/smi-spec)。

##  12. 容器存储
从存储的角度来看，容器有一个明显的缺陷:它们是短暂的。要理解其确切含义，我们需要了解当容器从container images开始时发生了什么。
一般来说，container images是只读的，由不同的层组成，这些层包括您在构建阶段添加的所有内容。这确保每次从映像启动容器时，都能获得相同的行为和功能。您可能可以想象，许多应用程序都需要编写文件。为了允许写入文件，当您从映像启动容器时，会在容器映像的顶部放置一个读写层。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/44e7a5f3075d2479abc504076a807750.png)
容器层，从[Docker文档中检索](https://docs.docker.com/storage/storagedriver/)
这里的问题是，当容器停止或删除时，这个读写层就会丢失。就像你的电脑在你关闭它的时候就会被删除一样。要持久化数据，您需要将其写入磁盘。
如果容器需要在主机上持久化数据，可以使用卷来实现这一点。其概念和技术非常简单:不是隔离进程的整个文件系统，而是将驻留在主机上的目录传递到容器文件系统中。如果你认为这会削弱容器的隔离性，你是对的。当使用容器卷时，可以有效地访问主机文件系统。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/af922e851293154abf3edcf474dfd28c.png)
数据在同一主机上的两个容器之间共享

当您编排许多容器时，在启动容器的主机上持久化数据可能不是惟一的挑战。通常，数据需要由在不同主机系统上启动的多个容器访问，或者当一个容器在不同主机上启动时，它仍然可以访问它的卷。
像`Kubernetes`这样的容器编排系统可以帮助缓解这些问题，但总是需要一个连接到主机服务器的健壮存储系统。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f36f69532b7d0cc1568a05b33143de0c.png)
存储是通过中央存储系统提供的。服务器A和服务器B上的容器可以共享一个卷来读写数据

为了跟上各种存储实现的不断增长，同样，解决方案是实现一个标准。[容器存储接口(CSI)](https://github.com/container-storage-interface/spec)提供了一个统一的接口，它允许附加不同的存储系统，无论它是云存储还是本地存储。

##  13. 其它资源
###  13.1 容器的历史

 - [《容器简史:从20世纪70年代到现在》](https://blog.aquasec.com/a-brief-history-of-containers-from-1970s-chroot-to-docker-2016)，Rani Osnat著(2020)
 - [就在这里:Docker 1.0](https://web.archive.org/web/20160426102954/https://blog.docker.com/2014/06/its-here-docker-1-0/)，朱利安·巴比尔(2014)

###  13.2 Chroot

 - [chroot](https://wiki.ubuntuusers.de/chroot/)

###  13.3 容器性能

 - [DockerCon 2017上的容器性能分析](https://www.brendangregg.com/blog/2017-05-15/container-performance-analysis-dockercon-2017.html), by Brendan Gregg

###  13.4 如何构建容器映像的最佳实践

 - [Top 20 Dockerfile Best Practices](https://sysdig.com/blog/dockerfile-best-practices/), by Álvaro Iradier (2021)
 - [3 simple tricks for smaller Docker images](https://learnk8s.io/blog/smaller-docker-images), by Daniele Polencic (2019)
 - [Best practices for building containers](https://cloud.google.com/architecture/best-practices-for-building-containers)

###  13.5 经典Dockerfile容器构建的替代方案
[Buildpacks vs Jib vs Dockerfile:容器化方法的比较](https://trainingportal.linuxfoundation.org/learn/course/kubernetes-and-cloud-native-essentials-lfs250/container-orchestration/%C3%81l), by James Ward (2020)

###  13.6 服务发现

 - [Service Discovery in a Microservices Architecture](https://www.nginx.com/blog/service-discovery-in-a-microservices-architecture/),   by Chris Richardson (2015)

###  13.7 容器网络

 - [Kubernetes Networking Part 1: Networking Essentials](https://www.inovex.de/de/blog/kubernetes-networking-part-1-en/), By Simon Kurth (2021)
 - [Life of a Packet](https://www.youtube.com/watch?v=0Omvgd7Hg1I) (I),by Michael Rubin (2017)
 - [Computer Networking Introduction - Ethernet and IP (Heavily Illustrated)](https://iximiuz.com/en/posts/computer-networking-101/), by Ivan Velichko (2021)


### 13.8 容器存储

 - [Managing Persistence for Docker Containers](https://thenewstack.io/methods-dealing-container-storage/), by Janakiram MSV (2016)

###  13.9 容器与kubernetes安全

 - [Secure containerized environments with updated thread matrix for Kubernetes](https://www.microsoft.com/security/blog/2021/03/23/secure-containerized-environments-with-updated-threat-matrix-for-kubernetes/), by Yossi Weizman (2021)

###  13.10 Docker Container Playground

 - [Play with Docker](https://labs.play-with-docker.com/)


---

✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>
 - [KCNA考试 第一章：cloud foundry云原生工程师考试](https://ghostwritten.blog.csdn.net/article/details/121482847)
 - [KCNA考试 第二章：Cloud Native Architecture](https://ghostwritten.blog.csdn.net/article/details/121492840)
 - [KCNA考试 第三章：容器编排](https://ghostwritten.blog.csdn.net/article/details/121527922)
 - [KCNA考试 第四章：kubernetes需要掌握的基础知识](https://ghostwritten.blog.csdn.net/article/details/123477851)
 - [KCNA考试 第五章：kubernetes实践](https://ghostwritten.blog.csdn.net/article/details/123496572)
 - [KCNA考试 第六章：持续交付](https://ghostwritten.blog.csdn.net/article/details/123553700)
 - [KCNA考试 第七章：监控与探测](https://ghostwritten.blog.csdn.net/article/details/123573993)
 -  [十二个因素的应用程序](https://ghostwritten.blog.csdn.net/article/details/121496200)
 - [KCNA（云原生入门）测试题](https://ghostwritten.blog.csdn.net/article/details/123631209)
----
