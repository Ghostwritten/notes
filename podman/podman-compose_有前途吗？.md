
![在这里插入图片描述](https://img-blog.csdnimg.cn/d9859c4720ff4078b39b2bf9cec94bac.png)




## 1. 前言
虽然 Kubernetes 已经发展成为容器编排的主导者，但人们仍然对管理较小规模的容器（通常是单个系统）有浓厚的兴趣。对于这些情况，多年来的首选工具一直是 [Docker Compose](https://github.com/docker/compose)。

在[Podman 项目](https://podman.io/)中，我们经常被问到有关 Docker Compose 的问题。Podman 是否支持 Docker Compose？Docker Compose v2 怎么样？什么是 [Podman Compose](https://github.com/containers/podman-compose)，我应该使用它而不是 Docker Compose？这样对吗？


## 2. Docker Compose 和 Podman Compose 的历史
`Docker Compose`始于 2014 年，是一个基于[YAML定义](https://opensource.com/downloads/yaml-cheat-sheet?intcmp=701f20000012ngPAAQ)管理容器组的项目。这个 YAML 后来被赋予了正式的规范，即[Compose 规范](https://github.com/compose-spec/compose-spec/blob/master/spec.md)。该规范定义了一种结构化语言，使您可以轻松地在一台机器上运行多个容器。

这种多个容器的单一文件定义是直接从命令行运行这些容器的替代方法。

[Docker Compose 是用 Python 编写的，并使用其 REST API](https://www.redhat.com/en/topics/api/what-are-application-programming-interfaces?intcmp=701f20000012ngPAAQ)与 Docker 守护进程通信。当 Podman 于 2018 年发布时，我们完全专注于与 Docker 命令行界面 (CLI) 的兼容性，并没有包括对 API 的支持。因此，Podman 最初无法与 `Docker Compose` 一起使用。许多人想使用 Podman 并保留他们现有的 Compose YAML 文件，因此一个名为`Podman Compose`的社区项目如雨后春笋般涌现。Podman Compose 处理 Compose 规范并将其转换为 Podman CLI 命令。

但是，`Podman Compose` 并不完美，因为它实现了 `Docker Compose` 的一部分功能。许多用户希望继续使用 `Docker Compose` 和其他直接连接到 `Docker API` 的工具。为了满足我们的社区和客户，Podman 在 2019 年添加了一个与 Docker 兼容的 API，但支持 Docker Compose 需要额外的工作，这发生在 2020 年。

`Docker Compose` 项目也没有闲着。2021 年，`Docker Compose` 项目公布了该工具在 Go 中的完全重写，称为 `Docker Compose v2.0`（较早的版本，`v1.x` 版本通常称为`docker-compose`，而 `v2.x` 称为`docker compose`）。重写需要额外的努力才能与 Podman 一起工作，于 2022 年推出 `Podman v4.1`（尽管对 Buildkit API 的支持仍在等待中）。

这仍然没有回答大多数人的问题：我应该使用 `Podman Compose` 还是 `Docker Compose`？这两个项目都不隶属于 Podman（Podman Compose 是一个社区项目，不由 Podman 团队直接维护）。两者都支持与 Podman 一起使用，并且 Podman 团队将修复 Podman 中阻止它们使用的任何错误（尽管工具本身的错误需要在那里修复）。

Podman Compose 更好地与 Podman 集成（因为它是为与 Podman 一起工作而设计的）并且可以更好地利用[无根容器](https://www.redhat.com/sysadmin/rootless-containers-podman)和 pod。然而，Docker Compose 是更有特色的选项，是 Compose 的参考实现。因此，您应该使用的答案是“视情况而定”。

`Podman Compose` 是更原生、更轻量级的解决方案。`Podman Compose` 直接执行 Podman 命令，而不是与 Podman 的 API 套接字通信。这消除了运行 Podman 服务来提供 API 的需要，节省了资源。因为它使用了 Podman 的常规命令行和 `fork-exec` 模型，所以更容易在系统上进行跟踪和管理。例如，Podman Compose 可以通过 systemd 单元文件轻松管理。

Docker Compose 具有更多功能。它与 Docker 和 Podman 兼容，因此更加通用。此外，它比 Podman Compose 拥有更多的用户，经过更广泛的测试，并且可能更稳定。

鉴于有充分的理由同时使用两者，`Podman` 致力于支持 `Docker Compose` 和 `Podman Compose`，包括 Docker Compose 的两个主要版本。

## 3. 未来
`Podman` 团队并不专注于 `Compose YAM`L。相反，我们正在努力开发`podman generate kube`和`podman play kube`，它允许`Kubernetes YAML` 以类似于 Compose 的方式直接与 Podman 一起使用。有了这些工具，我们就有了与更广泛的 Kubernetes 生态系统集成的优势。例如，在 Podman 上运行的容器`podman play kube`可以很容易地移动到 `OpenShift (Kubernetes)` 集群上，或者开发人员可以在他们的笔记本电脑上运行一个在 Kubernetes 中行为不端的 pod，以使用podman play kube.

我们认为 Podman 和 [Kubernetes](https://www.redhat.com/en/topics/containers/what-is-kubernetes?intcmp=701f20000012ngPAAQ) 之间的这种集成是一个强大的组合，这也是我们正在努力的方向。同时，我们认识到大多数人现在都在使用 Compose，并将继续使用多年。但是，我们希望用户选择转换为podman play kube，也许使用诸如[From Docker Compose to Kubernetes with Podman](https://www.redhat.com/sysadmin/compose-kubernetes-podman) 之类的指南。在可预见的未来，我们将支持 Docker Compose 和 Podman Compose with Podman。

## 4. 观点
1. `Red Hat` 通常推荐`Kubernetes YAML`而不是`Compose`，我们正在努力制定一个路线图，通过使用 `podman-play/generate-kube` 功能（图像构建、应用程序拆卸）创建和使用 Kubernetes YAML 来提供越来越多类似 Compose 的功能、intiContainers 以及对 Kubernetes 原语的扩展支持）
2. 如果你仍然想使用 Compose，Red Hat/RHEL 在 `podman-compose` 和 `docker-compose` 之间是中立的
3. Red Hat/RHEL 不提供 podman-compose 或 docker-compose，它是 BYO。
4. 对于任何组合工具，红帽建议无根运行。这是一篇较旧的文章，它解释了为什么以 root 身份运行是不好的：[为什么我们不允许非 root 用户在 CentOS、Fedora 或 RHEL 中运行 Docker](https://projectatomic.io/blog/2015/08/why-we-dont-let-non-root-users-run-docker-in-centos-fedora-or-rhel/)
5. `podman-compose` 实用程序是 GitHub 上 `github.com/containers` 项目的一部分，并与 Podman 紧密结合。它是第一个无根工作的组合工具。它与 Podman CLI 交互，部分原因是它是在 API 之前开发的，但更重要的是，因为 `fork/exec` 模型提供了一些优于使用基于 REST 的 API 的优势。例如，假设您正在容器中编译 Linux 内核。使用 `fork/exec` 模型，您可以使用本地目录中的所有数据，但使用 API，您必须跨套接字复制上下文目录才能进行构建。podman-compose 社区测试了 podman-compose，但似乎没有CI /CD。
6. `docker-compose 1.X` 工具是 Docker 提供的 Python 脚本，更符合 Docker 项目，但完全适用于 Podman socket/API。Podman社区确实有针对 `docker-compose` 无根和有根的上游 CI/CD 测试。下游 RHEL 产品确实有 docker-compose 测试以确保没有回归。
7. docker-compose 2.X 工具是 Docker 提供的 Golang 二进制文件，更符合 Docker 项目。今天，docker-compose 2.X 二进制文件不能与 Podman 套接字/API 一起使用，但团队正在研究如何让它工作。新的 docker-compose 2.X 实用程序扩展了对 Docker API 的扩展，并利用了作为 BuildKit 一部分的新 API 调用，因此还有一些工作要做。

貌似podman-compose不值得我们深入的探究。不过我们可以尝试简单的试玩。

参考：[Should I Use Docker Compose Or Podman Compose With Podman?](https://crunchtools.com/should-i-use-docker-compose-or-podman-compose-with-podman/)

## 5. 安装
### 5.1 pip3 安装
```bash
sudo -H pip3 install --upgrade pip
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/cf358d50a13744b8aba47675f6ae4963.png)

```bash
pip3 install podman-compose
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/cf3901641b0d4ba3b33ce1ad6ad5bbc0.png)
仅为当前用户安装，请添加`--user`标志：

```bash
pip3 install podman-compose --user
```
GitHub 上提供了 [Podman Compose](https://github.com/containers/podman-compose/tags) 的最新开发版本。奥，最新 版本还是2021年的，看来确实没人在更新它了，使用以下命令安装它：

```bash
sudo pip3 install https://github.com/containers/podman-compose/archive/devel.tar.gz
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/0f4b221fec8f497d8d6cb096970b26ad.png)
### 5.2 python 安装

```bash
sudo curl -o /usr/local/bin/podman-compose https://raw.githubusercontent.com/containers/podman-compose/devel/podman_compose.py
chmod +x /usr/local/bin/podman-compose
```
### 5.3 dnf 安装

```bash
sudo dnf install podman-compose
```

##  6. 示例
1. 为`compose.yml`文件创建一个目录并转到该目录。
```bash
mkdir plex-test &amp;&amp; cd plex-test
```
2. 创建compose.yml。

```bash
nano compose.yml
```
编写
```bash
services:
  plex:
    image: docker.io/linuxserver/plex
    container_name: plex
    network_mode: host
    environment:
      - VERSION=podman
    restart: always
    volumes:
      - ${PLEX_MEDIA_PATH}:/media/
```
4. 启动

```bash
podman-compose up &
```
当用户发出podman-compose up命令时，Podman Compose 会执行一系列任务：

- 创建一个名称与当前目录名称相对应的 pod。
- 检查compose.yml中指定的卷是否存在并创建缺少的卷。
- 为compose.yml中定义的每个服务创建一个容器。
- 将容器添加到 pod

5. 检查部署

```bash
podman pod ls
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/70bde05c48574e60a19ad5d4cb91b449.png)
6. 访问：`http://localhost:32400/web`，浏览器会将您重定向到 Plex 登录页面。
![在这里插入图片描述](https://img-blog.csdnimg.cn/7a55aefcf45e4592bb77761f32ce9eb8.png)

参考：
- [Podman Compose - Managing Containers](https://phoenixnap.com/kb/podman-compose)
- [containers/podman-compose](https://github.com/containers/podman-compose)
- [How to Install Podman on RHEL/CentOS 7/8](https://www.cyberithub.com/how-to-install-podman-on-rhel-centos-7-8-step-by-step/)
