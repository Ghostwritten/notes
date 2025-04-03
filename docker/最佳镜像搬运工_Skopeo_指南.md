#  最佳镜像搬运工 Skopeo 指南
![](https://i-blog.csdnimg.cn/blog_migrate/df7a157f708748af650b90acb64bbf3d.png)



## 1.  概述

[Skopeo](https://github.com/containers/skopeo) 是一款用来操作、检查、签署和传输容器镜像及镜像存储库的工具，支持 Linux® 系统、Windows 和 MacOS。与 Podman 和 Buildah 一样，Skopeo 也是一个由开源社区推动的项目，也不需要运行容器守护进程。

Skopeo 是一个能操作不同格式（包括开放容器计划（OCI）和 Docker 镜像）容器镜像的轻量级模块化解决方案，您无需下载包含所有层的完整镜像，就可检查远程镜像仓库中的镜像。

## 2. Skopeo 是如何工作的？
[Skopeo](https://github.com/containers/skopeo)（希腊语"远程查看"的意思）是红帽工程师与开源社区携手开发的第一款容器工具。[Skopeo](https://github.com/containers/skopeo) 可与 [Podman](https://podman.io/) 和 [Buildah](https://buildah.io/) 搭配来管理 [OCI](https://github.com/opencontainers) 容器。简而言之，**Podman 负责运行容器，Buildah 负责构建容器，而 Skopeo 则负责传输容器，同时也提供其他功能**。我们可以把这些工具看成是用于容器环境的一把瑞士军刀。Skopeo 是其中最万能的那把刀，任由您支配。

Skopeo 使用 `skopeo inspect` 命令来检查镜像。在 Skopeo 面世之前，您必须拉取完整镜像才能检查这个镜像，即使您只是打算检查一些元数据而已。而 Skopeo 的 inspect 命令可显示镜像的各种属性，如层、镜像标记和标签等，因此您不必将镜像拉取到主机上。这样，您无需占用任何容量，就能收集关于存储库或标记的信息。 

Skopeo 也允许您从存储库删除镜像，并将外部镜像存储库同步到内部镜像仓库，以实现更为安全的断网（也称为气隙系统）部署。当存储库需要时，Skopeo 可以传递相应的凭据和证书来进行身份验证。  

`Skopeo sync` 允许进行镜像仓库到镜像仓库的直接复制，以便联网使用，也允许进行镜像仓库到文件和文件到镜像仓库的复制，以便为断网环境做准备。`skopeo copy` 会假定所请求的复制要求操作；而 `skopeo sync` 则不同，它已经过调优，能够更快地完成定期重新同步极少改动的大型存储库。除了直接使用命令行外，同步操作也可在 `config` 文件中配置，而这可以仅同步大型存储库中的一个标记子集。

如果检查结果表明需要在不同位置或不同存储类型之间复制容器镜像，您可以通过 `Skopeo copy` 命令来操作。此工具可以让您在不同的镜像仓库（例如 [docker.io](https://www.docker.com/)、[quay.io](https://quay.io/)）、您的内部容器镜像仓库或您的本地系统上的各种存储机制之间复制容器镜像。Skopeo 的镜像仓库间直接复制功能速度很快，可在目标镜像仓库允许时保留未修改的形式（以及镜像的清单摘要）。在镜像仓库间复制不需要使用本地磁盘，也不要求本地磁盘上有可用的空间。Skopeo 也可在不同容器引擎存储甚至不同目录之间移动镜像。它常常用于持续集成/持续交付（CI/CD）系统中，以便使容器镜像仓库保持最新，并且维护容器服务器上的存储。

## 3. 为什么要用 Skopeo？
### 3.1 灵活性
[Skopeo](https://github.com/containers/skopeo) 是一套容器模块化工具的一部分，具有许多优势。将重要变更引入到单体式架构的工具中，而且不影响现有用户的使用，这绝非易事。Skopeo、Podman 和 Buildah 等尺寸更小巧、用途更专一的工具会以更快的速度更新换代。成套使用工具的话，可以使每个工具专注于单一用途，并且可以添加新的工具来增加功能，或者尝试与现有工具不兼容的想法和架构。工具的尺寸越小巧、模块化程度越高，保证安全也更加容易。
Podman 的部分功能源自于 `libpod` 库，允许与其他工具共享代码；Skopeo 与之相似，其功能也是在库中实施的。Skopeo 的容器/镜像库可由 Podman、Buildah 和 [CRI-O](https://cri-o.io/) 等其他容器工具共享，并且与 Docker 命令行界面（CLI）兼容。 

### 3.2 安全性和可访问性

将 `Podman`、`Skopeo` 和 `Buildah` 组合使用主要有以下优势：

- 无根容器管理。用户无需使用具有管理员特权的进程，就能创建、运行和管理容器，不仅使容器环境变得更易访问，又可降低安全风险。
- 无守护进程架构。守护进程需要管理员访问权限（同时无需进行管理员验证）来读取文件、安装程序、编辑应用和执行其他操作。如此一来，对于企图控制您的容器并渗透主机系统的黑客而言，守护进程是他们的理想攻击目标。 
- 原生 `systemd` 集成。通过使用 Podman 和相关的容器工具，您可以创建 systemd 单元文件并将容器作为系统服务来运行。

[Understanding root inside and outside a container](https://www.redhat.com/zh/blog/understanding-root-inside-and-outside-container)

### 3.3 功能多样性

- Skopeo 与 API V2 容器镜像仓库一起工作，例如docker.io和quay.io仓库、私有仓库、本地目录和本地 OCI 布局目录。Skopeo 可以执行的操作包括：

   - 从和向各种存储机制复制镜像。例如，您可以将镜像从一个仓库复制到另一个仓库，而无需特权。
   - 检查显示其属性（包括图层）的远程镜像，而无需将镜像拉到主机。
   - 从镜像存储库中删除镜像。
   - 将外部镜像存储库同步到内部仓库以进行气隙部署。
   - 当存储库需要时，skopeo 可以传递适当的凭据和证书进行身份验证。

- Skopeo 对以下镜像和存储库类型进行操作：

   - `containers-storage:docker-reference` 位于本地 `containers/storage`镜像存储中的镜像。位置和镜像存储都在 `/etc/containers/storage.conf` 中指定。（这是Podman、CRI-O、Buildah和朋友的后端）

   - `dir:path` 一个现有的本地目录路径，将清单、层 tarball 和签名存储为单独的文件。这是一种非标准化格式，主要用于调试或非侵入式容器检查。

   - `docker://docker-reference` 实现“`Docker Registry HTTP API V2`”的仓库中的镜像。默认情况下，使用 中的授权状态`$XDG_RUNTIME_DIR/containers/auth.json`，这是使用 设置的`skopeo login`。

    - `docker-archive:path[:docker-reference]` 镜像存储在 - 格式的`docker save`文件中。`docker-reference` 仅在创建此类文件时使用，并且不得包含摘要。

   - `docker-daemon:docker-reference` 存储在 docker 守护程序内部存储中的镜像 `docker-reference`。`docker-reference` 必须包含标签或摘要。或者，在读取镜像时，格式也可以是 `docker-daemon:algo:digest`（镜像 ID）。

   - `oci:path:tag` 路径中​​符合“`Open Container Image Layout Specification`”的目录中的镜像标签。

废话少说，我们开始 `Skopeo` 之旅。

## 4. 安装
### 4.1 Fedora

```bash
sudo dnf -y install skopeo
```
[Package Info](https://src.fedoraproject.org/rpms/skopeo) and [Bugzilla](https://bugzilla.redhat.com/buglist.cgi?bug_status=__open__&classification=Fedora&component=skopeo&product=Fedora)

### 4.2 RHEL / CentOS Stream ≥ 8

```bash
sudo dnf -y install skopeo
```

CentOS Stream 9: [Package Info](https://gitlab.com/redhat/centos-stream/rpms/skopeo/-/tree/c9s)

CentOS Stream 8: [Package Info](https://git.centos.org/rpms/skopeo/tree/c8s-stream-rhel8) 


- [更多 skopeo 安装方式看这里](https://github.com/containers/skopeo/blob/main/install.md)

###  4.3 RHEL/CentOS ≤ 7.x

```bash
sudo yum -y install skopeo
```

CentOS 7: [Package Repo](https://git.centos.org/rpms/skopeo/tree/c7-extras)

###  4.4 Ubuntu
skopeo包在Ubuntu 20.10及更新版本的官方存储库中可用。

```bash
# Ubuntu 20.10 and newer
sudo apt-get -y update
sudo apt-get -y install skopeo
```

Ubuntu： [Package Info](https://packages.ubuntu.com/jammy/skopeo)

### 4.5  容器安装

```bash
$ podman run docker://quay.io/skopeo/stable:latest copy --help
```

##  5. 命令参数

```bash
skopeo --help
Various operations with container images and container image registries

Usage:
  skopeo [command]

Available Commands:
  copy                                          Copy an IMAGE-NAME from one location to another
  delete                                        Delete image IMAGE-NAME
  help                                          Help about any command
  inspect                                       Inspect image IMAGE-NAME
  list-tags                                     List tags in the transport/repository specified by the REPOSITORY-NAME
  login                                         Login to a container registry
  logout                                        Logout of a container registry
  manifest-digest                               Compute a manifest digest of a file
  standalone-sign                               Create a signature using local files
  standalone-verify                             Verify a signature using local files
  sync                                          Synchronize one or more images from one location to another

Flags:
      --command-timeout duration   timeout for the command execution
      --debug                      enable debug output
  -h, --help                       help for skopeo
      --insecure-policy            run the tool without any policy check
      --override-arch ARCH         use ARCH instead of the architecture of the machine for choosing images
      --override-os OS             use OS instead of the running OS for choosing images
      --override-variant VARIANT   use VARIANT instead of the running architecture variant for choosing images
      --policy string              Path to a trust policy file
      --registries.d DIR           use registry configuration files in DIR (e.g. for container signature storage)
      --tmpdir string              directory used to store temporary files
  -v, --version                    Version for Skopeo

Use "skopeo [command] --help" for more information about a command.
```


|命令|	描述|
|--|--|
|[skopeo-copy(1)](https://github.com/containers/skopeo/blob/main/docs/skopeo-copy.1.md)|	将镜像（清单、文件系统层、签名）从一个位置复制到另一个位置。|
|[skopeo-delete(1)](https://github.com/containers/skopeo/blob/main/docs/skopeo-delete.1.md)|	标记镜像名称以便稍后由仓库的垃圾收集器删除。|
|[skopeo-inspect(1)](https://github.com/containers/skopeo/blob/main/docs/skopeo-inspect.1.md)	|返回有关仓库中镜像名称的低级信息。|
|[skopeo-list-tags(1)](https://github.com/containers/skopeo/blob/main/docs/skopeo-list-tags.1.md)	|返回特定于传输的镜像存储库的标签列表。|
|[skopeo-login(1)](https://github.com/containers/skopeo/blob/main/docs/skopeo-login.1.md)	|登录容器仓库。|
|[skopeo-logout(1)](https://github.com/containers/skopeo/blob/main/docs/skopeo-logout.1.md)|注销容器仓库。|
|[skopeo-manifest-digest(1)](https://github.com/containers/skopeo/blob/main/docs/skopeo-manifest-digest.1.md)	|计算清单文件的清单摘要并将其写入标准输出。|
|[skopeo-standalone-sign(1)](https://github.com/containers/skopeo/blob/main/docs/skopeo-standalone-sign.1.md)|	调试工具 - 一步发布和签署镜像。|
|[skopeo-standalone-verify(1)](https://github.com/containers/skopeo/blob/main/docs/skopeo-standalone-verify.1.md)	|验证镜像签名。|
|[skopeo-sync(1)](https://github.com/containers/skopeo/blob/main/docs/skopeo-sync.1.md)|	|在仓库存储库和本地目录之间同步镜像。|



## 6. 查询（skopeo inspect）
`skopeo inspect` 能够检查容器 Registry 上的存储库并获取镜像层。检查命令获取存储库的清单，它能够向您显示有关整个存储库或标签的类似 docker inspect 的 json 输出。与 docker inspect 相比,此工具可帮助您在拉取存储库或标签之前收集有用的信息(使用磁盘空间), 检查命令可以向您显示给定存储库可用的标签、映像具有的标签、映像的创建日期和操作系统等。

支持传输的类型 : containers-storage, dir, docker, docker-archive, docker-daemon, oci, oci-archive, ostree, tarball。

### 6.1 查询镜像 fedora:latest 的属性

```bash
$ podman images
REPOSITORY  TAG         IMAGE ID    CREATED     SIZE

$ skopeo inspect docker://docker.io/alpine:latest
{
    "Name": "docker.io/library/alpine",
    "Digest": "sha256:b95359c2505145f16c6aa384f9cc74eeff78eb36d308ca4fd902eeeb0a0b161b",
    "RepoTags": [
        "2.6",
        "2.7",
        "20190228",
        "20190408",
        "20190508",
        "20190707",
        "20190809",
        "20190925",
        "20191114",
        "20191219",
        "20200122",
        "20200319",
        "20200428",
        "20200626",
        "20200917",
        "20201218",
        "20210212",
        "20210730",
        "20210804",
        "20220316",
        "20220328",
        "20220715",
        "20221110",
        "3",
        "3.1",
        "3.10",
        "3.10.0",
        "3.10.1",
        "3.10.2",
        "3.10.3",
        "3.10.4",
        "3.10.5",
        "3.10.6",
        "3.10.7",
        "3.10.8",
        "3.10.9",
        "3.11",
        "3.11.0",
        "3.11.10",
        "3.11.11",
        "3.11.12",
        "3.11.13",
        "3.11.2",
        "3.11.3",
        "3.11.5",
        "3.11.6",
        "3.11.7",
        "3.11.8",
        "3.11.9",
        "3.12",
        "3.12.0",
        "3.12.1",
        "3.12.10",
        "3.12.11",
        "3.12.12",
        "3.12.2",
        "3.12.3",
        "3.12.4",
        "3.12.5",
        "3.12.6",
        "3.12.7",
        "3.12.8",
        "3.12.9",
        "3.13",
        "3.13.0",
        "3.13.1",
        "3.13.10",
        "3.13.11",
        "3.13.12",
        "3.13.2",
        "3.13.3",
        "3.13.4",
        "3.13.5",
        "3.13.6",
        "3.13.7",
        "3.13.8",
        "3.13.9",
        "3.14",
        "3.14.0",
        "3.14.1",
        "3.14.2",
        "3.14.3",
        "3.14.4",
        "3.14.5",
        "3.14.6",
        "3.14.7",
        "3.14.8",
        "3.15",
        "3.15.0",
        "3.15.0-rc.4",
        "3.15.1",
        "3.15.2",
        "3.15.3",
        "3.15.4",
        "3.15.5",
        "3.15.6",
        "3.16",
        "3.16.0",
        "3.16.1",
        "3.16.2",
        "3.16.3",
        "3.17",
        "3.17.0_rc1",
        "3.2",
        "3.3",
        "3.4",
        "3.5",
        "3.6",
        "3.6.5",
        "3.7",
        "3.7.3",
        "3.8",
        "3.8.4",
        "3.8.5",
        "3.9",
        "3.9.2",
        "3.9.3",
        "3.9.4",
        "3.9.5",
        "3.9.6",
        "edge",
        "latest"
    ],
    "Created": "2022-11-12T04:19:23.199716539Z",
    "DockerVersion": "20.10.12",
    "Labels": null,
    "Architecture": "amd64",
    "Os": "linux",
    "Layers": [
        "sha256:ca7dd9ec2225f2385955c43b2379305acd51543c28cf1d4e94522b3d94cce3ce"
    ],
    "Env": [
        "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    ]
}
```



### 6.2 查询镜像配置

```bash
$ skopeo inspect --config docker://docker.io/alpine:latest  | jq
{
  "created": "2022-11-12T04:19:23.199716539Z",
  "architecture": "amd64",
  "os": "linux",
  "config": {
    "Env": [
      "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    ],
    "Cmd": [
      "/bin/sh"
    ]
  },
  "rootfs": {
    "type": "layers",
    "diff_ids": [
      "sha256:e5e13b0c77cbb769548077189c3da2f0a764ceca06af49d8d558e759f5c232bd"
    ]
  },
  "history": [
    {
      "created": "2022-11-12T04:19:23.05154209Z",
      "created_by": "/bin/sh -c #(nop) ADD file:ceeb6e8632fafc657116cbf3afbd522185a16963230b57881073dad22eb0e1a3 in / "
    },
    {
      "created": "2022-11-12T04:19:23.199716539Z",
      "created_by": "/bin/sh -c #(nop)  CMD [\"/bin/sh\"]",
      "empty_layer": true
    }
  ]
}
```
### 6.3 查询镜像摘要

```bash
$ skopeo inspect docker://docker.io/alpine:latest | jq '.Digest'
"sha256:655721ff613ee766a4126cb5e0d5ae81598e1b0c3bcf7017c36c4d72cb092fe9"
```
或者
```bash
$ skopeo inspect --format "Name: {{.Name}} Digest: {{.Digest}}" docker://docker.io/alpine:latest
Name: docker.io/library/alpine Digest: sha256:b95359c2505145f16c6aa384f9cc74eeff78eb36d308ca4fd902eeeb0a0b161b
```
- [关于 inspect 用法更多细节请看这里](https://ghostwritten.blog.csdn.net/article/details/108062031)

### 6.4 免登陆查询
- --creds=testuser:testpassword

```bash
skopeo inspect   --creds=admin:Harbor12345 docker://harbor.fumai.com/library/alpine:latest
{
    "Name": "harbor.fumai.com/library/alpine",
    "Digest": "sha256:39ec5d12ef5a81b29b26d756f6b9c11d8d454fc4158e3dac1e13240125558461",
    "RepoTags": [
        "latest"
    ],
    "Created": "2022-11-12T04:19:23.199716539Z",
    "DockerVersion": "20.10.12",
    "Labels": null,
    "Architecture": "amd64",
    "Os": "linux",
    "Layers": [
        "sha256:60f8044dac9f779802600470f375c7ca7a8f7ad50e05b0ceb9e3b336fa5e7ad3"
    ],
    "Env": [
        "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    ]
}
```

## 7. 显示镜像标签（skopeo list-tags）
`skopeo list-tags`显示镜像存储库的标签列表，期待已久的功能了。
```bash
$ skopeo list-tags docker://192.168.10.80:5000/alpine
{
    "Repository": "192.168.10.80:5000/alpine",
    "Tags": [
        "latest"
    ]
}

$ skopeo list-tags docker://docker.io/alpine
{
    "Repository": "docker.io/library/alpine",
    "Tags": [
        "2.6",
        "2.7",
        "20190228",
        "20190408",
        "20190508",
        "20190707",
        "20190809",
        "20190925",
        "20191114",
        "20191219",
        "20200122",
        "20200319",
        "20200428",
        "20200626",
        "20200917",
        "20201218",
        "20210212",
        "20210730",
        "20210804",
        "20220316",
        "20220328",
        "20220715",
        "20221110",
        "3",
        "3.1",
        "3.10",
        "3.10.0",
        "3.10.1",
        "3.10.2",
        "3.10.3",
        "3.10.4",
        "3.10.5",
        "3.10.6",
        "3.10.7",
        "3.10.8",
        "3.10.9",
        "3.11",
        "3.11.0",
        "3.11.10",
        "3.11.11",
        "3.11.12",
        "3.11.13",
        "3.11.2",
        "3.11.3",
        "3.11.5",
        "3.11.6",
        "3.11.7",
        "3.11.8",
        "3.11.9",
        "3.12",
        "3.12.0",
        "3.12.1",
        "3.12.10",
        "3.12.11",
        "3.12.12",
        "3.12.2",
        "3.12.3",
        "3.12.4",
        "3.12.5",
        "3.12.6",
        "3.12.7",
        "3.12.8",
        "3.12.9",
        "3.13",
        "3.13.0",
        "3.13.1",
        "3.13.10",
        "3.13.11",
        "3.13.12",
        "3.13.2",
        "3.13.3",
        "3.13.4",
        "3.13.5",
        "3.13.6",
        "3.13.7",
        "3.13.8",
        "3.13.9",
        "3.14",
        "3.14.0",
        "3.14.1",
        "3.14.2",
        "3.14.3",
        "3.14.4",
        "3.14.5",
        "3.14.6",
        "3.14.7",
        "3.14.8",
        "3.15",
        "3.15.0",
        "3.15.0-rc.4",
        "3.15.1",
        "3.15.2",
        "3.15.3",
        "3.15.4",
        "3.15.5",
        "3.15.6",
        "3.16",
        "3.16.0",
        "3.16.1",
        "3.16.2",
        "3.16.3",
        "3.17",
        "3.17.0_rc1",
        "3.2",
        "3.3",
        "3.4",
        "3.5",
        "3.6",
        "3.6.5",
        "3.7",
        "3.7.3",
        "3.8",
        "3.8.4",
        "3.8.5",
        "3.9",
        "3.9.2",
        "3.9.3",
        "3.9.4",
        "3.9.5",
        "3.9.6",
        "edge",
        "latest"
    ]
}
```



## 8. 登陆（skopeo login）
我这里有三个仓库地址
- Docker 官方 hub 仓库： `docker.io`
- Harbor 私有仓库：`harbor.fumai.com`
- Registry 私有仓库：`192.168.10.80:5000`

在使用 `skopeo` 前，如果 `src` 或 `dest` 镜像是在 `registry` 仓库中的并且配置了非 `public` 的镜像需要相应的 `auth` 认证, 此时我们可以使用 `docker login` 或者 `skopeo login` 的方式登录到 `registry` 仓库，然后默认会在`~/.docker`目录下生成 registry 登录配置文件 `config.json` ,该文件里保存了登录需要的验证信息，`skopeo` 拿到该验证信息才有权限往 `registry push` 镜像。

`skopeo` 使用来自 `--creds`（对于 `skopeo inspect|delete`）或 `--src-creds|--dest-creds`（对于 `skopeo copy`）标志的凭据，如果已设置；否则它使用由 `skopeo` 登录、`podman` 登录、`buildah` 登录或 `docker` 登录设置的配置。
### 8.1 配置私有仓库证书
由于我是使用 `podman` 容器工具，它的默认仓库认证证书存放位置是`/etc/containers/certs.d`。
1. 配置 `registry` 证书
`registry` 仓库部署在我的本机容器中，证书挂载于本机`/opt/registry/certs/`目录。
```bash
install -d  /etc/containers/certs.d/192.168.10.80\:5000/
install /opt/registry/certs/* /etc/containers/certs.d/192.168.10.80\:5000/
```
`install` 是一个既能创建目录又能复制的命令。
2. 配置 harbor 证书
`harbor` 仓库在另一台机器
```bash
$ cat <<EOF >> vim /etc/hosts
192.168.10.81 harbor.fumai.com
EOF
```

```bash
install -d /etc/containers/certs.d/harbor.fumai.com
rsync root@192.168.10.81:/etc/docker/certs.d/harbor.fumai.com/* /etc/containers/certs.d/harbor.fumai.com/
```

### 8.2 podman 登陆各个仓库
登陆 `docker.io` 仓库

```bash
$ podman login -u ghostwritten docker.io
Password:
Login Succeeded!
```
登陆 registry 仓库
```bash
$ podman login -u registryuser -p registryuserpassword 192.168.10.80:5000
Login Succeeded!
```
登陆 `harbor` 仓库

```bash
$ podman login harbor.fumai.com -u admin -p Harbor12345
Login Succeeded!
```
`Podman` 登陆仓库生成的`auth`文件在这里

```bash
$ cat /run/user/0/containers/auth.json
{
        "auths": {
                "192.168.10.80:5000": {
                        "auth": "cmVnaXN0cnl1c2VyOnJlZ2lzdHJ5dXNlcnBhc3N3b3Jk"
                },
                "docker.io": {
                        "auth": "Z2hvc3R3cml0dGVuOjEyMzQ1bXRyLg=="
                },
                "harbor.fumai.com": {
                        "auth": "YWRtaW46SGFyYm9yMTIzNDU="
                }
        }
}
```

`skopeo` 登陆 `registry` 仓库

### 8.3 skopeo 多种方式登陆仓库
```bash
skopeo login -u ghostwritten docker.io
Password:
Login Succeeded!

$ skopeo login  -u registryuser -p registryuserpassword 192.168.10.80:5000
Login Succeeded!

$ skopeo login --cert-dir /etc/containers/certs.d/192.168.10.80\:5000 -u registryuser -p registryuserpassword 192.1
68.10.80:5000
Login Succeeded!

$ skopeo login --authfile /run/user/0/containers/auth.json harbor.fumai.com

$ echo 'Harbor12345' | skopeo login -u admin --password-stdin harbor.fumai.com
Login Succeeded!
```






## 9. 复制镜像（skopeo copy）
`skopeo copy` 可以在各种存储机制之间复制容器镜像，包括：

- `Container registries`

    - The Quay, Docker Hub, OpenShift, GCR, Artifactory ...
- `Container Storage backends`

   - github.com/containers/storage (Backend for Podman, CRI-O, Buildah and friends)

   - Docker daemon storage

- `Local directories`

- `Local OCI-layout directories`

### 9.1 本地镜像推送仓库
将本地镜像拷贝至`192.168.10.80:5000`仓库
```bash
$ docker images localhost/busybox:latest
REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
busybox      latest    9d5226e6ce3f   4 days ago   1.24MB

如果你得容器以podman为引擎。
$ skopeo copy containers-storage:localhost/busybox:latest docker://192.168.10.80:5000/busybox:latest
INFO[0000] Not using native diff for overlay, this may cause degraded performance for building images: kernel has CONFIG_OVERLAY_FS_REDIRECT_DIR enabled
Getting image source signatures
Copying blob 40cf597a9181 [--------------------------------------] 0.0b / 0.0b
Copying config 9d5226e6ce [======================================] 1.4KiB / 1.4KiB
Writing manifest to image destination
Storing signatures

#如果你得容器以docker为引擎。
$ skopeo copy docker-daemon:localhost/busybox:latest docker://192.168.10.80:5000/busybox:latest

# 没有tls安全的registry的情况下这样操作。
$ skopeo copy --insecure-policy --dest-tls-verify=false --dest-authfile /root/.docker/config.json docker-daemon:localhost/busybox:latest docker://192.168.10.80:5000/busybox:latest
```


### 9.2 两仓库进行复制
`192.168.10.80:5000`仓库的 `alpine` 镜像同步至 `harbor.fumai.com`仓库
```bash
#两个仓库进行查询当前镜像内容
$ curl  -k -u "registryuser:registryuserpassword" https://192.168.10.80:5000/v2/_catalog
{"repositories":["alpine"]}
$ curl  -k -u "admin:Harbor12345" https://harbor.fumai.com/v2/_catalog
{"repositories":[]}

#镜像开始复制
$ skopeo copy docker://192.168.10.80:5000/alpine:latest docker://harbor.fumai.com/library/alpine:latest
Getting image source signatures
Copying blob 60f8044dac9f done
Copying config bfe296a525 done
Writing manifest to image destination
Storing signatures


#查询复制结果
$ curl  -k -u "admin:Harbor12345" https://harbor.fumai.com/v2/_catalog
{"repositories":["library/alpine"]}

#查询镜像信息
$ skopeo inspect docker://harbor.fumai.com/library/alpine:latest
{
    "Name": "harbor.fumai.com/library/alpine",
    "Digest": "sha256:39ec5d12ef5a81b29b26d756f6b9c11d8d454fc4158e3dac1e13240125558461",
    "RepoTags": [
        "latest"
    ],
    "Created": "2022-11-12T04:19:23.199716539Z",
    "DockerVersion": "20.10.12",
    "Labels": null,
    "Architecture": "amd64",
    "Os": "linux",
    "Layers": [
        "sha256:60f8044dac9f779802600470f375c7ca7a8f7ad50e05b0ceb9e3b336fa5e7ad3"
    ],
    "Env": [
        "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    ]
}
```

其它仓库同步方法：
```bash
#  docker://quay.io的buildah 同步到docker://registry.internal.company.com
$ skopeo copy docker://quay.io/buildah/stable docker://harbor.fumai.com/library/buildah

#如果 registry 是一个没有tls验证的仓库并且以 docker 为容器引擎。
$ skopeo copy --insecure-policy --dest-tls-verify=false --dest-authfile /root/.docker/config.json docker://192.168.10.80:5000/alpine:latest docker://harbor.fumai.com/library/alpine:latest

#如果 registry 是一个没有tls验证的仓库并且以 podman 为容器引擎。
 $ skopeo copy --insecure-policy --dest-tls-verify=false --dest-authfile /run/user/0/containers/auth.json docker://192.168.10.80:5000/alpine:latest docker://harbor.fumai.com/library/alpine:latest

```




### 9.3 仓库镜像拷贝到本地目录
创建`images`目录，从`harbor.fumai.com`仓库复制至`images`目录
```bash
$ install -d images
$ skopeo copy docker://harbor.fumai.com/library/alpine:latest dir:images
Getting image source signatures
Copying blob 60f8044dac9f done
Copying config bfe296a525 done
Writing manifest to image destination
Storing signatures

$ ll images
total 2864
-rw-r--r-- 1 root root 2918987 Nov 21 23:08 60f8044dac9f779802600470f375c7ca7a8f7ad50e05b0ceb9e3b336fa5e7ad3
-rw-r--r-- 1 root root    1471 Nov 21 23:08 bfe296a525011f7eb76075d688c681ca4feaad5afe3b142b36e30f1a171dc99a
-rw-r--r-- 1 root root     428 Nov 21 23:08 manifest.json
-rw-r--r-- 1 root root      33 Nov 21 23:08 version
```
保存`OCI`格式

```bash
$ install -d images_OCI
skopeo copy docker://harbor.fumai.com/library/alpine:latest oci:images_OCI
$ tree images_OCI/
images_OCI/
├── blobs
│   └── sha256
│       ├── 096fca2180271e60f199e3bc1c3544dc0175e7e6abd9dbdd0e2058b17d96e8b7
│       ├── 60f8044dac9f779802600470f375c7ca7a8f7ad50e05b0ceb9e3b336fa5e7ad3
│       └── e9daf8b695a6e3c0c566a2e25b328ba25afbdd52a80d28198b9fbf5058130f09
├── index.json
└── oci-layout

2 directories, 5 files
```
或者如果没有登陆条件可以这样执行：

```bash
$ skopeo copy --src-creds=admin:harbor12345 docker://harbor.fumai.com/library/alpine:latest oci:local_oci_image
```



## 10. 同步镜像（Skopeo sync）
`Skopeo sync`可以在容器仓库和本地目录之间同步镜像，其功能类似于阿里云的 [image-syncer](https://github.com/AliyunContainerService/image-syncer) 工具, 实际上其比 `image-syncer` 更强大、灵活性更强一些
```bash
$ skopeo sync --src docker --dest dir registry.example.com/busybox /media/usb
```

###  10.1 仓库镜像同步至本地
将仓库中所有 `busybox` 镜像版本同步到本地目录

查询当前`harbor.fumai.com`仓库 busybox版本

```bash
skopeo list-tags docker://harbor.fumai.com/library/busybox
{
    "Repository": "harbor.fumai.com/library/busybox",
    "Tags": [
        "latest",
        "stable"
    ]
}
```
将`busybox`镜像所有版本同步至本地

```bash
$ skopeo sync  --src docker --dest dir harbor.fumai.com/library/busybox images_busybox
INFO[0000] Tag presence check                            imagename=harbor.fumai.com/library/busybox tagged=false
INFO[0000] Getting tags                                  image=harbor.fumai.com/library/busybox
INFO[0000] Copying image ref 1/2                         from="docker://harbor.fumai.com/library/busybox:latest" to="dir:/images_busybox/busybox:latest"
Getting image source signatures
Copying blob 405fecb6a2fa done
Copying config 9d5226e6ce done
Writing manifest to image destination
Storing signatures
INFO[0000] Copying image ref 2/2                         from="docker://harbor.fumai.com/library/busybox:stable" to="dir:/images_busybox/busybox:stable"
Getting image source signatures
Copying blob 405fecb6a2fa done
Copying config 9d5226e6ce done
Writing manifest to image destination
Storing signatures
INFO[0001] Synced 2 images from 1 sources

$ tree images_busybox
/images_busybox
├── busybox:latest
│   ├── 405fecb6a2fa4f29683f977e7e3b852bf6f8975a2aba647d234d2371894943da
│   ├── 9d5226e6ce3fb6aee2822206a5ef85f38c303d2b37bfc894b419fca2c0501269
│   ├── manifest.json
│   └── version
└── busybox:stable
    ├── 405fecb6a2fa4f29683f977e7e3b852bf6f8975a2aba647d234d2371894943da
    ├── 9d5226e6ce3fb6aee2822206a5ef85f38c303d2b37bfc894b419fca2c0501269
    ├── manifest.json
    └── version

2 directories, 8 files
```
其他方式：

```bash
skopeo sync --src-creds=admin:Harbor12345  --src docker --dest dir harbor.fumai.com/library/busybox /images_busybox

skopeo sync --src-authfile=/run/user/0/containers/auth.json  --src docker --dest dir harbor.fumai.com/library/busybox /images_busybox
```

###  10.2 本地同步至仓库镜像
这次颠倒过来，从本地目录`/images_busybox`同步至`registry:192.168.10.80:5000`仓库

skopeo 登陆
```bash
skopeo login  -u registryuser -p registryuserpassword 192.168.10.80:5000
```
开始同步

```bash
$ skopeo sync  --src dir --dest docker  /images_busybox 192.168.10.80:5000
INFO[0000] Copying image ref 1/2                         from="dir:/images_busybox/busybox:latest" to="docker://192.168.10.80:5000/busybox:latest"
Getting image source signatures
Copying blob 405fecb6a2fa done
Copying config 9d5226e6ce done
Writing manifest to image destination
Storing signatures
INFO[0001] Copying image ref 2/2                         from="dir:/images_busybox/busybox:stable" to="docker://192.168.10.80:5000/busybox:stable"
Getting image source signatures
Copying blob 405fecb6a2fa [--------------------------------------] 0.0b / 0.0b
Copying config 9d5226e6ce [======================================] 1.4KiB / 1.4KiB
Writing manifest to image destination
Storing signatures
INFO[0001] Synced 2 images from 1 sources
```
查询结果

```bash
$ skopeo list-tags docker://192.168.10.80:5000/busybox
{
    "Repository": "192.168.10.80:5000/busybox",
    "Tags": [
        "latest",
        "stable"
    ]
}
```
成功。
其他同步方式：

```bash
skopeo sync --dest-creds=registryuser:registryuserpassword  --src dir --dest docker /images_busybox 192.168.10.80:5000 

skopeo sync --src-authfile=/run/user/0/containers/auth.json  --src dir --dest docker /images_busybox 192.168.10.80:5000 

#如果 registry 是一个没有tls验证的仓库
skopeo sync --insecure-policy --dest-tls-verify=false --src dir --dest docker /images_busybox 192.168.10.80:5000
```

###  10.3 两仓库同步
将`harbor（ harbor.fumai.com）`仓库的`busybox`所有版本同步至`registry（192.168.10.80:5000）`仓库
```bash
skopeo sync   --src docker --dest docker harbor.fumai.com/library/busybox 192.168.10.80:5000
```
其他方式：

```bash
skopeo sync --src-creds=admin:Harbor12345 --dest-creds=registryuser:registryuserpassword  --src docker --dest docker harbor.fumai.com/library/busybox 192.168.10.80:5000 

skopeo sync --dest-authfile=/run/user/0/containers/auth.json --src-authfile=/run/user/0/containers/auth.json --src docker --dest docker harbor.fumai.com/library/busybox 192.168.10.80:5000 
```

###  10.4 以配置文件方式进行同步
准备一个需要同步的资源清单

```bash
$ cat skopeo-sync.yml
192.168.10.80:5000:
 images:
   busybox: [stable]
   redis:
     - "latest"
     - "7.0.5"
 credentials:
     username: registryuser
     password: registryuserpassword
 tls-verify: true
 cert-dir: /etc/containers/certs.d/192.168.10.80:5000

docker.io:
 tls-verify: false
 images:
   httpd:
     - "latest"

quay.io:
 tls-verify: false
 images:
   coreos/etcd:
     - latest
```
开始同步

```bash
$ skopeo sync --src yaml --dest docker skopeo-sync.yml harbor.fumai.com/library
INFO[0000] Processing repo                               registry=docker.io repo=httpd
INFO[0000] Processing repo                               registry=quay.io repo=coreos/etcd
INFO[0000] Processing repo                               registry="192.168.10.80:5000" repo=busybox
INFO[0000] Processing repo                               registry="192.168.10.80:5000" repo=redis
INFO[0000] Copying image ref 1/1                         from="docker://httpd:latest" to="docker://harbor.fumai.com/library/httpd:latest"
Getting image source signatures
Copying blob ff7b0b8c417a done
Copying blob 4691bd33efec done
Copying blob a603fa5e3b41 done
Copying blob b1c114085b25 done
Copying blob 9df1012343c7 done
Copying config 8653efc8c7 done
Writing manifest to image destination
Storing signatures
INFO[0015] Copying image ref 1/1                         from="docker://quay.io/coreos/etcd:latest" to="docker://harbor.fumai.com/library/etcd:latest"
Getting image source signatures
Copying blob a3ed95caeb02 done
Copying blob 96b0e24539ea done
Copying blob d1eca4d01894 done
Copying blob ff3a5c916c92 done
Copying blob 8bc526247b5c done
Copying blob ad732d7a61c2 done
Copying blob a3ed95caeb02 skipped: already exists
Copying blob a3ed95caeb02 skipped: already exists
Copying blob 5f56944bb51c done
Writing manifest to image destination
Storing signatures
INFO[0022] Synced 2 images from 2 sources
INFO[0007] Copying image ref 1/1                         from="docker://192.168.10.80:5000/busybox:stable" to="docker://harbor.fumai.com/library/busybox:stable"
Getting image source signatures
Skipping: image already present at destination
INFO[0007] Copying image ref 1/2                         from="docker://192.168.10.80:5000/redis:latest" to="docker://harbor.fumai.com/library/redis:latest"
Getting image source signatures
Copying blob a0056c8a01e3 done
Copying blob 87b7cd6612f6 done
Copying blob 656160ba04b8 done
Copying blob a08cc0270959 done
Copying blob d14ed0f181ce done
Copying blob 39e3d9efeaec skipped: already exists
Copying config 3358aea34e done
Writing manifest to image destination
Storing signatures
INFO[0009] Copying image ref 2/2                         from="docker://192.168.10.80:5000/redis:7.0.5" to="docker://harbor.fumai.com/library/redis:7.0.5"
Getting image source signatures
Copying blob a08cc0270959 skipped: already exists
Copying blob d14ed0f181ce skipped: already exists
Copying blob a0056c8a01e3 skipped: already exists
Copying blob 87b7cd6612f6 skipped: already exists
Copying blob 656160ba04b8 skipped: already exists
Copying blob 39e3d9efeaec [--------------------------------------] 0.0b / 0.0b
Copying config 3358aea34e [======================================] 7.5KiB / 7.5KiB
Writing manifest to image destination
Storing signatures
```
## 11. 删除镜像（skopeo delete）

### 11.1 skopeo 删除镜像
使用`skopeo delete`命令我们可以删除镜像 Tag,注意此处仅仅只是通过 registry API 来删除镜像的 tag（即删除了 tag 对 manifests 文件的引用）并非真正将镜像删除掉，如果想要删除镜像的 layer 还是需要通过 registry GC 的方式。
```bash
$ curl  -k -u "admin:Harbor12345" https://harbor.fumai.com/v2/_catalog
{"repositories":["library/alpine"]}

#开始删除
$ skopeo delete docker://harbor.fumai.com/library/alpine:latest

$ curl  -k -u "admin:Harbor12345" https://harbor.fumai.com/v2/_catalog
{"repositories":[]}
```
`--debug` 模式：

```bash
skopeo delete docker://harbor.fumai.com/library/busybox:latest --debug
DEBU[0000] Loading registries configuration "/etc/containers/registries.conf"
DEBU[0000] Loading registries configuration "/etc/containers/registries.conf.d/000-shortnames.conf"
DEBU[0000] Loading registries configuration "/etc/containers/registries.conf.d/001-rhel-shortnames.conf"
DEBU[0000] Loading registries configuration "/etc/containers/registries.conf.d/002-rhel-shortnames-overrides.conf"
DEBU[0000] Found credentials for harbor.fumai.com in credential helper containers-auth.json
DEBU[0000] Using registries.d directory /etc/containers/registries.d for sigstore configuration
DEBU[0000]  Using "default-docker" configuration
DEBU[0000]   Using file:///var/lib/containers/sigstore
DEBU[0000] Looking for TLS certificates and private keys in /etc/containers/certs.d/harbor.fumai.com
DEBU[0000]  crt: /etc/containers/certs.d/harbor.fumai.com/ca.crt
DEBU[0000]  cert: /etc/containers/certs.d/harbor.fumai.com/harbor.fumai.com.cert
DEBU[0000]  key: /etc/containers/certs.d/harbor.fumai.com/harbor.fumai.com.key
DEBU[0000] GET https://harbor.fumai.com/v2/
DEBU[0000] Ping https://harbor.fumai.com/v2/ status 401
DEBU[0000] GET https://harbor.fumai.com/service/token?account=admin&scope=repository%3Alibrary%2Fbusybox%3A%2A&service=harbor-registry
DEBU[0000] GET https://harbor.fumai.com/v2/library/busybox/manifests/latest
DEBU[0000] DELETE https://harbor.fumai.com/v2/library/busybox/manifests/sha256:f75f3d1a317fc82c793d567de94fc8df2bece37acd5f2bd364a0d91a0d1f3dab
DEBU[0000] Deleting /var/lib/containers/sigstore/library/busybox@sha256=f75f3d1a317fc82c793d567de94fc8df2bece37acd5f2bd364a0d91a0d1f3dab/signature-1
```
### 11.2 curl 方式删除

```bash
$ curl -k -u "registryuser:registryuserpassword" --header "Accept: application/vnd.docker.distribution.manifest.v2+json" -I
 -X GET https://192.168.10.80:5000/v2/busybox/manifests/latest
HTTP/2 200
content-type: application/vnd.docker.distribution.manifest.v2+json
docker-content-digest: sha256:f75f3d1a317fc82c793d567de94fc8df2bece37acd5f2bd364a0d91a0d1f3dab
docker-distribution-api-version: registry/2.0
etag: "sha256:f75f3d1a317fc82c793d567de94fc8df2bece37acd5f2bd364a0d91a0d1f3dab"
x-content-type-options: nosniff
content-length: 527
date: Tue, 22 Nov 2022 10:51:14 GMT

$ curl -s -k -u "registryuser:registryuserpassword" --header "Accept: application/vnd.docker.distribution.manifest.v2+json"
 -I -X GET https://192.168.10.80:5000/v2/busybox/manifests/latest | grep -i "Docker-Content-Digest" | cut -d ' ' -f 2
sha256:f75f3d1a317fc82c793d567de94fc8df2bece37acd5f2bd364a0d91a0d1f3dab

$ curl -u 'registryuser:registryuserpassword' -k -X DELETE https://192.168.10.80:5000/v2/busybox/manifests/sha256:f75f3d1a317fc82c793d567de94fc8df2bece37acd5f2bd364a0d91a0d1f3dab
```
我写了一个关于通过 API 删除仓库镜像的`delete_images.sh`脚本，当然你的仓库需要配置`REGISTRY_STORAGE_DELETE_ENABLED=true`，才能执行删除操作，否则不支持。

```bash
#!/bin/bash


default() {

 Images=${1-'busybox:latest'}
 Registry_Url=${2-'192.168.10.80:5000'}
 Registry_Username=${3-'registryuser'}
 Registry_Password=${4-'registryuserpassword'}

}

help() {
    echo "Usage:"
    echo "test.sh [-i Images] [-h Registry_Url] [-u Registry_Username] [-p Registry_Password] [-d]"
    echo "Description:"
    echo "  Images,One or more mirror images,exmple: (busybox:latest alpine:latest)."
    echo "  Registry_Url, registry name，exmaple: docker.io."
    echo "  Registry_Username, registry username，exmaple: 'admin'."
    echo "  Registry_Password, registry password，exmaple: 'Harbor12345'."
    echo "  -d, Indicates the deletion action."
    exit -1
}





delete() {

for Image in ${Images}
do

  Image_Name=`echo $Image | cut -d ':' -f 1 `
  Image_Version=`echo $Image | cut -d ':' -f 2`

  echo " Starting delete ${Registry_Url}/$Image_Name:$Image_Version..."
DockerContentDigest=$(curl -s -k -u "$Registry_Username:$Registry_Password" --header "Accept: application/vnd.docker.distribution.manifest.v2+json" -I -X GET "https://${Registry_Url}/v2/${Image_Name}/manifests/${Image_Version}" | grep -i "Docker-Content-Digest" | cut -d ' ' -f 2 )

DockerContentDigest=${DockerContentDigest%$'\r'}

  if [[ $DockerContentDigest != '' ]];then
    curl -u ${Registry_Username}:${Registry_Password} -k -X DELETE  https://${Registry_Url}/v2/${Image_Name}/manifests/${DockerContentDigest} && [ $? == 0 ] && echo "Ok, $Registry_Url/$Image delete sucessful!"

   else
     echo "Sorry, $Registry_Url/$Image not find!"
   fi

done

}


#[ $# == 0 ] && help
[ $# == 0 ] &&  default && delete && exit 0



while getopts 'i:h:u:p:d' OPT; do
    case $OPT in
        i) Images="$OPTARG";;
        h) Registry_Url="$OPTARG";;
        u) Registry_Username="$OPTARG";;
        p) Registry_Password="$OPTARG";;
        d) delete;;
        h) help;;
        ?) help;;
    esac
done
```
执行效果：我们尝试删除仓库`192.168.10.80:5000`的`busybox:latest`镜像。

```bash
$ skopeo list-tags docker://192.168.10.80:5000/alpine
{
    "Repository": "192.168.10.80:5000/alpine",
    "Tags": [
        "latest"
    ]
}

$ bash   delete_images.sh  -i alpine:latest -h 192.168.10.80:5000 -u registryuser -p registryuserpassword -d
 Starting delete 192.168.10.80:5000/alpine:latest...
Ok, 192.168.10.80:5000/alpine:latest delete sucessful!

$ skopeo list-tags docker://192.168.10.80:5000/alpine
{
    "Repository": "192.168.10.80:5000/alpine",
    "Tags": []
}
```

关于`skopeo` 的练习就到此结束了。


参考：
- [如何使用 Skopeo 做一个优雅的镜像搬运工](https://www.hi-linux.com/posts/55385.html)
- [https://github.com/containers/skopeo](https://github.com/containers/skopeo)






