🎈<font color=	#FF4500 size=6 style="font-family:Courier New">一键三连，赛过活神仙</font>🎈

#  Podman 快速入门





## 1. 什么是podman
[Podman](https://podman.io/)是一个无守护进程的开源 Linux 原生工具，旨在使用开放容器倡议 ( [OCI](https://opencontainers.org/) )容器和[容器映像](https://developers.redhat.com/blog/2018/02/22/container-terminology-practical-introduction#)轻松查找、运行、构建、共享和部署应用程序。`Podman` 提供了任何使用过 `Docker`容器引擎的人都熟悉的命令行界面 (CLI) 。大多数用户可以简单地将 Docker 别名为 Podman（别名 `docker=podman`），由谷歌，Redhat、微软、IBM、Intel、思科联合成立的组织（OCI）容器运行时（`runc`、`crun`、`runv` 等）制定了一系列容器运行的规范。主要是由RedHat推动，完成Docker所有的功能和新扩展功能，并且对docker的问题进行了改良：包括不需要守护程序或访问有root权限的组；容器架构基于fork/exec模型创建容器，更加安全可靠；所以是更先进、高效和安全的下一代容器容器工具。Podman是该工具套件的核心，用来替换Docker中了大多数子命令（RUN，PUSH，PULL等）。Podman无需守护进程，使用用户命名空间来模拟容器中的root，无需连接到具有root权限的套接字保证容器的体系安全。

Podman专注于维护和修改OCI镜像的所有命令和功能，例如拉动和标记。它还允许我们创建，运行和维护从这些镜像创建的容器。目前已有相当数量的分发版本开始采用Podman作为容器运行时。与其他常见的容器引擎（`Docker`、`CRI-O`、`containerd`）类似

`Podman` 比较简单粗暴，它不使用 `Daemon`，而是直接通过 `OCI runtime`（默认也是 runc）来启动容器，所以容器的进程是 `podman` 的子进程。这比较像 Linux 的 `fork/exec` 模型，而 Docker 采用的是 C/S（客户端/服务器）模型。与 C/S 模型相比，`fork/exec` 模型有很多**优势**，比如：

 - 系统管理员可以知道某个容器进程到底是谁启动的。
 - 如果利用 `cgroup` 对 `podman` 做一些限制，那么所有创建的容器都会被限制。
 - `SD_NOTIFY` : 如果将 `podman` 命令放入 `systemd` 单元文件中，容器进程可以通过 `podman`返回通知，表明服务已准备好接收任务。
 - `socket` 激活 : 可以将连接的 `socket` 从 `systemd` 传递到 `podman`，并传递到容器进程以便使用它们。

Podman 控制下的容器可以由 root 或非特权用户运行。Podman 使用[libpod](https://github.com/containers/podman)库管理整个容器生态系统，包括 pod、容器、容器镜像和容器卷。

有一个 `RESTFul API` 来管理容器。我们还有一个可以与 `RESTFul` 服务交互的远程 `Podman` 客户端。我们目前支持 `Linux`、`Mac` 和 `Windows` 上的客户端。`RESTFul` 服务仅在 `Linux` 上受支持。

如果您完全不熟悉容器，我们建议您查看[简介](https://docs.podman.io/en/latest/Introduction.html)。对于高级用户或来自 Docker 的用户，请查看我们的[教程](https://docs.podman.io/en/latest/Tutorials.html)。对于高级用户和贡献者，您可以通过查看我们的命令页面来获取有关 [Podman CLI](https://docs.podman.io/en/latest/Commands.html) 的非常详细的信息。最后，对于寻求如何与 Podman API 交互的开发人员，请参阅我们的 [API 文档](https://docs.podman.io/en/latest/Reference.html)参考。

![在这里插入图片描述](https://img-blog.csdnimg.cn/942b660fe96e48bb98eec4c6d34c2216.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/1a2a324fe8dd4d96a781580aba914191.png)

##  2. 安装

 - [官方安装](https://podman.io/getting-started/installation)

我的机器环境`CentOS Linux 7`

```bash
yum -y install podman
```
###  2.1 如何安装最新版本
- ubuntu
```bash
sudo sh -c "echo 'deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/testing/x${NAME}_${VERSION_ID}/ /' > /etc/apt/sources.list.d/devel:kubic:libcontainers:testing.list"
wget -nv https://download.opensuse.org/repositories/devel:kubic:libcontainers:testing/x${NAME}_${VERSION_ID}/Release.key -O Release.key
sudo apt-key add - < Release.key
sudo apt-get update -qq
sudo apt-get -qq -y install podman
$ sudo apt-get  install podman
```

##  3. 配置文件
### 3.1 podman包附带文件

```bash
$ rpm -ql podman  |grep -v '/usr/share/man/'  # 去除 man 手册中内容
/etc/cni/net.d/87-podman-bridge.conflist
/usr/bin/podman
/usr/lib/.build-id
/usr/lib/.build-id/37
/usr/lib/.build-id/37/e7f04d352e5dbde603e9701baedb0b1be6bc37
/usr/lib/.build-id/9a
/usr/lib/.build-id/9a/2b43332ca5756f9e2a086bae9b953009ef5a37
/usr/lib/systemd/system/io.podman.service
/usr/lib/systemd/system/io.podman.socket
/usr/lib/tmpfiles.d/podman.conf
/usr/libexec/podman/conmon
/usr/share/bash-completion/completions/podman
/usr/share/containers/libpod.conf
/usr/share/licenses/podman
/usr/share/licenses/podman/LICENSE
```
### 3.2 /etc/cni
可以看到只有一个配置文件是在 `/etc/cni` 路径下的，与 `Bridge` 的配置有关：

```bash
$ cat /etc/cni/net.d/87-podman-bridge.conflist
{
    "cniVersion": "0.4.0",
    "name": "podman",
    "plugins": [
	{
            "type": "bridge",
            "bridge": "cni-podman0",
            "isGateway": true,
            "ipMasq": true,
            "ipam": {
		"type": "host-local",
		"routes": [
		    {
			"dst": "0.0.0.0/0"
		    }
		],
		"ranges": [
		    [
			{
			    "subnet": "10.88.0.0/16",
			    "gateway": "10.88.0.1"
			}
		    ]
		]
            }
	},
	{
            "type": "portmap",
            "capabilities": {
		"portMappings": true
            }
	},
	{
            "type": "firewall"
	}
    ]
}
```
###  3.3 /etc/containers/registries.conf
`/etc/containers/registries.conf` 用于保存 `registries` 相关配置：

```bash
$ cat /etc/containers/registries.conf   |grep -v '#' |grep -v ^$
[registries.search]
registries = ['registry.access.redhat.com', 'registry.redhat.io', 'docker.io']
[registries.insecure]
registries = []
[registries.block]
registries = []
```
###  3.4 /usr/share/containers/mounts.conf
`/usr/share/containers/mounts.conf` 在执行 `podman run` 或者 `podman build` 命令时自动挂载的路径，该路径只会在容器运行时挂载，不会提交到容器镜像中。

```bash
$ cat /usr/share/containers/mounts.conf 
/usr/share/rhel/secrets:/run/secrets
```

###  3.5 /usr/share/containers/seccomp.json
`/usr/share/containers/seccomp.json` 是容器内允许的 `seccomp` 规则白名单。 `seccomp`（secure computing）是一种安全保护机制，一般情况下，程序可以使用所有的 `syscall`，但是为了避免安全问题发生，通常会指定相应的规则来保证。

```bash
$ cat /usr/share/containers/seccomp.json
{
	"defaultAction": "SCMP_ACT_ERRNO",
	"archMap": [
		{
			"architecture": "SCMP_ARCH_X86_64",
			"subArchitectures": [
				"SCMP_ARCH_X86",
				"SCMP_ARCH_X32"
			]
		},
		{
			"architecture": "SCMP_ARCH_AARCH64",
			"subArchitectures": [
				"SCMP_ARCH_ARM"
			]
		},
		{
			"architecture": "SCMP_ARCH_MIPS64",
			"subArchitectures": [
				"SCMP_ARCH_MIPS",
				"SCMP_ARCH_MIPS64N32"
			]
		},


```

###  3.6 /etc/containers/policy.json
`/etc/containers/policy.json` 证书安全相关配置：

```bash
$ cat /etc/containers/policy.json     
{
    "default": [
        {
            "type": "insecureAcceptAnything"
        }
    ],
    "transports":
        {
            "docker-daemon":
                {
                    "": [{"type":"insecureAcceptAnything"}]
                }
        }
}
```
### 3.7  /run/user/0/containers/auth.json
登陆生成的用户认证文件

```bash
$ cat /run/user/0/containers/auth.json
{
        "auths": {
                "192.168.10.80:5000": {
                        "auth": "cmVnaXN0cnl1c2VyOnJlZ2lzdHJ5dXNlcnBhc3N3b3Jk"
                },
                "harbor.fumai.com": {
                        "auth": "YWRtaW46SGFyYm9yMTIzNDU="
                }
        }
}
```



## 4. 环境信息
###  4.1 查看配置信息

```bash
#等同于podman system info
$ podman info
host:
  arch: amd64
  buildahVersion: 1.22.3
  cgroupControllers:
  - cpuset
  - cpu
  - cpuacct
  - blkio
  - memory
  - devices
  - freezer
  - net_cls
  - perf_event
  - net_prio
  - hugetlb
  - pids
  - rdma
  cgroupManager: systemd
  cgroupVersion: v1
  conmon:
    package: conmon-2.0.29-1.module_el8.5.0+890+6b136101.x86_64
    path: /usr/bin/conmon
    version: 'conmon version 2.0.29, commit: 84384406047fae626269133e1951c4b92eed7603'
  cpus: 4
  distribution:
    distribution: '"centos"'
    version: "8"
  eventLogger: file
  hostname: podman
  idMappings:
    gidmap: null
    uidmap: null
  kernel: 4.18.0-348.7.1.el8_5.x86_64
  linkmode: dynamic
  memFree: 7123599360
  memTotal: 8144809984
  ociRuntime:
    name: runc
    package: runc-1.0.2-1.module_el8.5.0+911+f19012f9.x86_64
    path: /usr/bin/runc
    version: |-
      runc version 1.0.2
      spec: 1.0.2-dev
      go: go1.16.7
      libseccomp: 2.5.1
  os: linux
  remoteSocket:
    exists: true
    path: /run/podman/podman.sock
  security:
    apparmorEnabled: false
    capabilities: CAP_NET_RAW,CAP_CHOWN,CAP_DAC_OVERRIDE,CAP_FOWNER,CAP_FSETID,CAP_KILL,CAP_NET_BIND_SERVICE,CAP_SETFCAP,CAP_SETGID,CAP_SETPCAP,CAP_SETUID,CAP_SYS_CHROOT
    rootless: false
    seccompEnabled: true
    seccompProfilePath: /usr/share/containers/seccomp.json
    selinuxEnabled: false
  serviceIsRemote: false
  slirp4netns:
    executable: /usr/bin/slirp4netns
    package: slirp4netns-1.1.8-1.module_el8.5.0+890+6b136101.x86_64
    version: |-
      slirp4netns version 1.1.8
      commit: d361001f495417b880f20329121e3aa431a8f90f
      libslirp: 4.4.0
      SLIRP_CONFIG_VERSION_MAX: 3
      libseccomp: 2.5.1
  swapFree: 3221221376
  swapTotal: 3221221376
  uptime: 73h 6m 23.13s (Approximately 3.04 days)
registries:
  search:
  - registry.fedoraproject.org
  - registry.access.redhat.com
  - registry.centos.org
  - docker.io
store:
  configFile: /etc/containers/storage.conf
  containerStore:
    number: 1
    paused: 0
    running: 1
    stopped: 0
  graphDriverName: overlay
  graphOptions:
    overlay.mountopt: nodev,metacopy=on
  graphRoot: /var/lib/containers/storage
  graphStatus:
    Backing Filesystem: xfs
    Native Overlay Diff: "false"
    Supports d_type: "true"
    Using metacopy: "true"
  imageStore:
    number: 4
  runRoot: /run/containers/storage
  volumePath: /var/lib/containers/storage/volumes
version:
  APIVersion: 3.3.1
  Built: 1636493036
  BuiltTime: Wed Nov 10 05:23:56 2021
  GitCommit: ""
  GoVersion: go1.16.7
  OsArch: linux/amd64
  Version: 3.3.1
```

### 4.2 查看统计信息
- 包含镜像数量、存储占比、大小
- 包含容器数量、存储占比、大小
- 包含逻辑卷数量、存储占比、大小
```bash
$ podman system df
TYPE           TOTAL       ACTIVE      SIZE        RECLAIMABLE
Images         9           3           521.5MB     493.8MB (0%)
Containers     2           2           12B         0B (0%)
Local Volumes  0           0           0B          0B (0%)
```

## 5. 登陆（podman login）

```bash
$ podman login harbor.fumai.com -u admin -p Harbor12345
Login Succeeded!
```
查询登陆生成的用户认证信息
```bash
$ cat /run/user/0/containers/auth.json
{
        "auths": {
                "192.168.10.80:5000": {
                        "auth": "cmVnaXN0cnl1c2VyOnJlZ2lzdHJ5dXNlcnBhc3N3b3Jk"
                },
                "harbor.fumai.com": {
                        "auth": "YWRtaW46SGFyYm9yMTIzNDU="
                }
        }
}
```
base64字符串进行反编码

```bash
$ echo -n "YWRtaW46SGFyYm9yMTIzNDU=" | base64 -d
admin:Harbor12345
```

退出登陆

```bash
$ podman logout harbor.fumai.com
Removed login credentials for harbor.fumai.com
$ cat /run/user/0/containers/auth.json
{
        "auths": {
                "192.168.10.80:5000": {
                        "auth": "cmVnaXN0cnl1c2VyOnJlZ2lzdHJ5dXNlcnBhc3N3b3Jk"
                }
        }
}
```



##  6. 镜像管理
###  6.1 镜像搜索 (podman search)

```bash
$ podman search busybox
INDEX       NAME                                DESCRIPTION                                       STARS   OFFICIAL   AUTOMATED
docker.io   docker.io/library/busybox           Busybox base image.                               2410    [OK]       
docker.io   docker.io/radial/busyboxplus        Full-chain, Internet enabled, busybox made f...   43                 [OK]
docker.io   docker.io/yauritux/busybox-curl     Busybox with CURL                                 16                 
docker.io   docker.io/arm64v8/busybox           Busybox base image.                               3                  
docker.io   docker.io/vukomir/busybox           busybox and curl                                  1                  
docker.io   docker.io/odise/busybox-curl                                                          4                  [OK]
docker.io   docker.io/amd64/busybox             Busybox base image.                               0                  
docker.io   docker.io/prom/busybox              Prometheus Busybox Docker base images             2                  [OK]
docker.io   docker.io/ppc64le/busybox           Busybox base image.                               1                  
docker.io   docker.io/s390x/busybox             Busybox base image.                               2                  
docker.io   docker.io/arm32v7/busybox           Busybox base image.                               10                 
docker.io   docker.io/i386/busybox              Busybox base image.                               2                  
docker.io   docker.io/joeshaw/busybox-nonroot   Busybox container with non-root user nobody       2                  
docker.io   docker.io/p7ppc64/busybox           Busybox base image for ppc64.                     2                  
docker.io   docker.io/arm32v5/busybox           Busybox base image.                               0                  
docker.io   docker.io/arm32v6/busybox           Busybox base image.                               3                  
docker.io   docker.io/armhf/busybox             Busybox base image.                               6                  
docker.io   docker.io/mips64le/busybox          Busybox base image.                               1                  
docker.io   docker.io/spotify/busybox           Spotify fork of https://hub.docker.com/_/bus...   1                  
docker.io   docker.io/aarch64/busybox           Busybox base image.                               3                  
docker.io   docker.io/progrium/busybox                                                            70                 [OK]
docker.io   docker.io/concourse/busyboxplus                                                       0                  
docker.io   docker.io/lqshow/busybox-curl       Busybox image adds a curl binary to /usr/bin      1                  [OK]
docker.io   docker.io/emccorp/busybox           Busybox                                           0                  
docker.io   docker.io/ggtools/busybox-ubuntu    Busybox ubuntu version with extra goodies         0                  [OK]
```
限制显示镜像数量

```bash
$ podman search busybox --limit 2
INDEX       NAME                       DESCRIPTION          STARS       OFFICIAL    AUTOMATED
docker.io   docker.io/library/busybox  Busybox base image.  2823        [OK]
docker.io   docker.io/rancher/busybox
```

### 4.2 镜像拉取 (podman pull)

```bash
$ podman image pull nginx
Getting image source signatures
Copying blob eff15d958d66 done  
Copying blob 9171c7ae368c done  
Copying blob 1e5351450a59 [======================================] 24.2MiB / 24.2MiB
Copying blob 020f975acd28 done  
Copying blob 266f639b35ad done  
Copying blob 2df63e6ce2be done  
Copying config ea335eea17 done  
Writing manifest to image destination
Storing signatures
ea335eea17ab984571cd4a3bcf90a0413773b559c75ef4cda07d0ce952b00291
```
###  4.3 镜像推送 (podman push)
将本地镜像推送至公共库或者私有仓库
```bash
podman tag alpine:latest 192.168.10.80:5000/alpine:latest
podman login -u registryuser -p registrypassword 192.168.10.80:5000
podman push 192.168.10.80:5000/alpine:latest
```
[skopeo](https://ghostwritten.blog.csdn.net/article/details/127342981) 命令更方便实现此功能。

### 4.3 镜像列出  (podman images)

1. 参数
```bash
p images --help
List images in local storage

Description:
  Lists images previously pulled to the system or created on the system.

Usage:
  podman images [options] [IMAGE]

Examples:
  podman images --format json
  podman images --sort repository --format "table {{.ID}} {{.Repository}} {{.Tag}}"
  podman images --filter dangling=true

Options:
   --all, -a
       Show  all  images  (by  default  filter out the intermediate image layers). The default is
       false.

   --digests
       Show image digests

   --filter=filter, -f
       Filter output based on conditions provided

       Filters:

       since=IMAGE
           Filter on images created after the given IMAGE (name or tag).

       before=IMAGE
           Filter on images created before the given IMAGE (name or tag).

       dangling=true|false
           Show dangling images. Dangling images are a file system  layer  that  was  used  in  a
       previous  build  of  an  image  and is no longer referenced by any active images. They are
       denoted with the <none> tag, consume disk space and serve no active purpose.

       label
           Filter by images labels key and/or value.

       readonly=true|false
            Show only read only images or  Read/Write  images.  The  default  is  to  show  both.
       Read/Only  images  can  be  configured  by  modifying  the  "additionalimagestores" in the
       /etc/containers/storage.conf file.

       reference=
            Filter by image name, specified as regular expressions.

   --format=format
       Change the default output format.  This can be of a supported type like  'json'  or  a  Go
       template.  Valid placeholders for the Go template are listed below:

       ┌──────────────┬──────────────────────────────────┐
       │Placeholder   │ Description                      │
       ├──────────────┼──────────────────────────────────┤
       │.ID           │ Image ID                         │
       ├──────────────┼──────────────────────────────────┤
       │.Repository   │ Image repository                 │
       ├──────────────┼──────────────────────────────────┤
       │.Tag          │ Image tag                        │
       ├──────────────┼──────────────────────────────────┤
       │.Digest       │ Image digest                     │
       ├──────────────┼──────────────────────────────────┤
       │.CreatedSince │ Elapsed time since the image was │
       │              │ created                          │
       │              │                                  │
       ├──────────────┼──────────────────────────────────┤
       │.CreatedAt    │ Time when the image was created  │
       ├──────────────┼──────────────────────────────────┤
       │.Size         │ Size of layer on disk            │
       ├──────────────┼──────────────────────────────────┤
       │.History      │ History of the image layer       │
       └──────────────┴──────────────────────────────────┘

   --history
       Display  the  history  of  image  names.  If an image gets re-tagged or untagged, then the
       image name history gets prepended (latest image first).  This is  especially  useful  when
       undoing  a  tag  operation  or  an  image  does  not  contain any name because it has been
       untagged.

   --noheading, -n
       Omit the table headings from the listing of images.

   --no-trunc
       Do not truncate output.

   --quiet, -q
       Lists only the image IDs.

   --sort=sort
       Sort by created, id, repository, size or tag (default: created)
```

2. 示例

2.1 默认输出全部镜像
```bash
$ podman images
REPOSITORY                TAG      IMAGE ID       CREATED       SIZE
docker.io/library/nginx   latest   ea335eea17ab   12 days ago   146 MB
```
2.2 显示 sha256

```bash
p images  --digests
REPOSITORY                  TAG         DIGEST                                                                   IMAGE ID      CREATED      SIZE
localhost/busybox           latest      sha256:f75f3d1a317fc82c793d567de94fc8df2bece37acd5f2bd364a0d91a0d1f3dab  9d5226e6ce3f  6 days ago   1.46 MB
192.168.10.80:5000/busybox  stable      sha256:f75f3d1a317fc82c793d567de94fc8df2bece37acd5f2bd364a0d91a0d1f3dab  9d5226e6ce3f  6 days ago   1.46 MB
```
2.3 `--history` 可以追溯 镜像历史版本

```bash
$ images  --history
REPOSITORY                  TAG         IMAGE ID      CREATED      SIZE        HISTORY
localhost/busybox           latest      9d5226e6ce3f  6 days ago   1.46 MB     192.168.10.80:5000/busybox:stable, localhost/busybox:latest
192.168.10.80:5000/busybox  stable      9d5226e6ce3f  6 days ago   1.46 MB     192.168.10.80:5000/busybox:stable, localhost/busybox:latest
```
2.4 以 `json` 格式输出全部镜像详细信息

```bash
$ p images --format json
[
    {
        "Id": "9d5226e6ce3fb6aee2822206a5ef85f38c303d2b37bfc894b419fca2c0501269",
        "ParentId": "",
        "RepoTags": null,
        "RepoDigests": [
            "sha256:f75f3d1a317fc82c793d567de94fc8df2bece37acd5f2bd364a0d91a0d1f3dab",
            "sha256:5f809f2970d626e727745ca4b0aedf3e17b3e8b880b28fd0f9fa23821e826cbc"
        ],
        "Size": 1462040,
        "SharedSize": 0,
        "VirtualSize": 1462040,
        "Labels": null,
        "Containers": 1,
        "Names": [
            "localhost/busybox:latest",
            "192.168.10.80:5000/busybox:stable"
        ],
        "Digest": "sha256:f75f3d1a317fc82c793d567de94fc8df2bece37acd5f2bd364a0d91a0d1f3dab",
        "History": [
            "192.168.10.80:5000/busybox:stable",
            "localhost/busybox:latest"
        ],
        "Created": 1668734369,
        "CreatedAt": "2022-11-18T01:19:29Z"
    },
```

2.5  `--format`自定义输出格式

```bash
$ podman images --sort repository --format "table {{.ID}} {{.Repository}} {{.Tag}}"
IMAGE ID      REPOSITORY                  TAG
bfe296a52501  192.168.10.80:5000/alpine   latest
9d5226e6ce3f  192.168.10.80:5000/busybox  stable
81c944c2288b  docker.io/library/registry  latest
9d5226e6ce3f  localhost/busybox           latest
```

### 4.4 镜像检查 (podman inspect)

```bash
$ podman image inspect docker.io/library/nginx
[
    {
        "Id": "ea335eea17ab984571cd4a3bcf90a0413773b559c75ef4cda07d0ce952b00291",
        "Digest": "sha256:097c3a0913d7e3a5b01b6c685a60c03632fc7a2b50bc8e35bcaa3691d788226e",
        "RepoTags": [
            "docker.io/library/nginx:latest"
        ],
        "RepoDigests": [
            "docker.io/library/nginx@sha256:097c3a0913d7e3a5b01b6c685a60c03632fc7a2b50bc8e35bcaa3691d788226e",
            "docker.io/library/nginx@sha256:2f14a471f2c2819a3faf88b72f56a0372ff5af4cb42ec45aab00c03ca5c9989f"
        ],
        "Parent": "",
        "Comment": "",
        "Created": "2021-11-17T10:38:14.652464384Z",
        "Config": {
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NGINX_VERSION=1.21.4",
                "NJS_VERSION=0.7.0",
                "PKG_RELEASE=1~bullseye"
            ],
............................
```
### 4.5 镜像删除（podman rmi）

```bash 
$ podman image rm docker.io/library/nginx                 
f949e7d76d63befffc8eec2cbf8a6f509780f96fb3bacbdc24068d594a77f043
$ podman rmi busy:latest
```
###  4.6 镜像构建 (podman build)
[构建镜像参数参考](https://docs.podman.io/en/latest/markdown/podman-build.1.html)

```bash
# cat Dockerfile
FROM registry.access.redhat.com/rhel7/rhel-minimal
ENTRYPOINT "echo "Podman build this container."

# podman build -t podbuilt .
STEP 1: FROM registry.access...
...
Writing manifest to image destination
Storing signatures
91e043c11617c08d4f8...

# podman run podbuilt
Podman build this container.
```
###  4.7 镜像打包 (docker save)

```bash
$ docker save -o busybox.tar localhost/busybox:latest
Getting image source signatures
Copying blob 40cf597a9181 done
Copying config 9d5226e6ce done
Writing manifest to image destination
Storing signatures

$ ll busybox.tar
-rw-r--r-- 1 root root 1468416 Nov 24 14:45 busybox.tar
```
还可以这样

```bash
docker save > busybox.tar localhost/busybox:latest
```
当然，也可以将镜像导出本地目录

```bash
$ podman save --format docker-dir -o busybox-dir busybox:latest
Getting image source signatures
Copying blob 40cf597a9181 done
Copying config 9d5226e6ce done
Writing manifest to image destination
Storing signatures

$ ll busybox-dir/
total 1440
-rw-r--r-- 1 root root 1459200 Nov 24 14:47 40cf597a9181e86497f4121c604f9f0ab208950a98ca21db883f26b0a548a2eb
-rw-r--r-- 1 root root    1457 Nov 24 14:47 9d5226e6ce3fb6aee2822206a5ef85f38c303d2b37bfc894b419fca2c0501269
-rw-r--r-- 1 root root     428 Nov 24 14:47 manifest.json
-rw-r--r-- 1 root root      33 Nov 24 14:47 version
```
###  4.8 镜像解包 (podman load)

```bash
$ podman load -i busybox.tar
$ podman load < busybox.tar
```

## 5. 容器管理
###  5.1 容器运行 (podman run)

```bash
#直接进入好容器
$   podman run -it busybox
/ # mkdir test
/ # echo 123456 > test/1.txt
/ # hostname
0b01e437bbac

#容器在后台运行
$ podman run -itd   --name busybox busybox 
8d4cf68f9873b19bf8d7ac3a7be4447a9519d931ddfc4162de5ab576c66696af
```
###  5.2 容器查看（podman ps）

```bash
#查看运行的容器
$ podman ps
CONTAINER ID  IMAGE                             COMMAND  CREATED         STATUS             PORTS  NAMES
4b05d6c539b3  docker.io/library/busybox:latest  sh       17 seconds ago  Up 16 seconds ago         upbeat_lederberg

#查看状态为running的容器
$ podman ps -a -f status=running
CONTAINER ID  IMAGE                             COMMAND  CREATED        STATUS            PORTS  NAMES
4b05d6c539b3  docker.io/library/busybox:latest  sh       5 minutes ago  Up 3 seconds ago         upbeat_lederberg


#查看全部容器
$ podman ps -a
CONTAINER ID  IMAGE                             COMMAND  CREATED        STATUS                     PORTS  NAMES
4b05d6c539b3  docker.io/library/busybox:latest  sh       2 seconds ago  Up 1 second ago                   upbeat_lederberg
35b58e980d08  docker.io/library/busybox:latest  sh       2 minutes ago  Exited (0) 46 seconds ago         amazing_colden

#只看全部容器的ID
$ podman ps -aq
4b05d6c539b3
35b58e980d08

自定义打印容器属性
$ podman ps --format "{{.ID}} {{.Image}} {{.Names}}"
481626ffc902 docker.io/kindest/node@sha256:f52781bc0d7a19fb6c405c2af83abfeb311f130707a0e219175677e366cc45d1 kind-control-plane
```
### 5.3 容器监控（podman stats）
查看机器所有容器资源使用情况,并持续实时显示变化
```bash
$  p stats
ID            NAME           CPU %       MEM USAGE / LIMIT  MEM %       NET IO             BLOCK IO      PIDS        CPU TIME       AVG CPU %
012e806d404d  myregistry     --          21.08MB / 8.145GB  0.26%       1.221MB / 479.1kB  0B / 1.427MB  7           46.780427841s  9.34%
0b01e437bbac  sleepy_wright  --          237.6kB / 8.145GB  0.00%       838B / 1.194kB     -- / --       1           106.68144ms    2.13%
```
滚屏输出：

```bash
$ p stats --no-reset
ID            NAME        CPU %       MEM USAGE / LIMIT  MEM %       NET IO             BLOCK IO      PIDS        CPU TIME       AVG CPU %
012e806d404d  myregistry  2.81%       21.08MB / 8.145GB  0.26%       1.221MB / 480.4kB  0B / 1.427MB  7           46.862082188s  2.81%
86d7e611eba6  busybox     6.71%       249.9kB / 8.145GB  0.00%       908B / 1.264kB     -- / --       1           111.937412ms   6.71%
ID            NAME        CPU %       MEM USAGE / LIMIT  MEM %       NET IO             BLOCK IO      PIDS        CPU TIME       AVG CPU %
012e806d404d  myregistry  --          21.08MB / 8.145GB  0.26%       1.221MB / 480.5kB  0B / 1.427MB  7           46.862082188s  1.40%
86d7e611eba6  busybox     --          249.9kB / 8.145GB  0.00%       978B / 1.334kB     -- / --       1           111.937412ms   3.35%
ID            NAME        CPU %       MEM USAGE / LIMIT  MEM %       NET IO             BLOCK IO      PIDS        CPU TIME       AVG CPU %
012e806d404d  myregistry  --          21.08MB / 8.145GB  0.26%       1.221MB / 480.5kB  0B / 1.427MB  7           46.862082188s  9.36%
86d7e611eba6  busybox     --          249.9kB / 8.145GB  0.00%       978B / 1.334kB     -- / --       1           111.937412ms   2.24%
ID            NAME        CPU %       MEM USAGE / LIMIT  MEM %       NET IO             BLOCK IO      PIDS        CPU TIME       AVG CPU %
012e806d404d  myregistry  0.01%       21.08MB / 8.145GB  0.26%       1.221MB / 480.5kB  0B / 1.427MB  7           46.862217887s  0.00%
86d7e611eba6  busybox     --          249.9kB / 8.145GB  0.00%       978B / 1.334kB     -- / --       1           111.937412ms   1.68%
```

查看机器所有容器资源使用情况,只一次

```bash
$ p stats --all --no-stream
ID            NAME        CPU %       MEM USAGE / LIMIT  MEM %       NET IO             BLOCK IO      PIDS        CPU TIME       AVG CPU %
012e806d404d  myregistry  2.81%       21.08MB / 8.145GB  0.26%       1.221MB / 480.4kB  0B / 1.427MB  7           46.844027793s  2.81%
86d7e611eba6  busybox     6.71%       249.9kB / 8.145GB  0.00%       908B / 1.264kB     -- / --       1           111.937412ms   6.71%
```

指定容器，并持续实时显示变化

```bash
$ p stats busybox
ID            NAME        CPU %       MEM USAGE / LIMIT  MEM %       NET IO          BLOCK IO    PIDS        CPU TIME      AVG CPU %
86d7e611eba6  busybox     --          249.9kB / 8.145GB  0.00%       768B / 1.124kB  -- / --     1           111.937412ms  3.35%
```
查看最新创建的容器监控变化

```bash
$ p stats -l
86d7e611eba6  busybox     --          249.9kB / 8.145GB  0.00%       978B / 1.334kB  -- / --     1           111.937412ms  3.35%
q
ID            NAME        CPU %       MEM USAGE / LIMIT  MEM %       NET IO          BLOCK IO    PIDS        CPU TIME      AVG CPU %
86d7e611eba6  busybox     --          249.9kB / 8.145GB  0.00%       978B / 1.334kB  -- / --     1           111.937412ms  2.24%
```

###  5.4 容器比对 (podman diff)

```bash
$ podman diff sleepy_wright
C /root
A /root/.ash_history
A /test
A /test/1.txt
C /etc
```

###  5.5 容器停止 (podman stop)

```bash
$ podman stop mywebserver

$ podman stop 860a4b235279

$ podman stop mywebserver 860a4b235279

$ podman stop --cidfile /home/user/cidfile-1

$ podman stop --cidfile /home/user/cidfile-1 --cidfile ./cidfile-2

$ podman stop --time 2 860a4b235279

$ podman stop -a

$ podman stop --latest
```
###  5.6 容器强杀 (podman kill)

```bash
$ p kill busybox
busybox

$ podman kill -s SIGKILL my-httpd-container
77d4b7b8ed1fd57449163bcb0b78d205e70d2314273263ab941c0c371ad56412
```
任何 Unix 信号都可以发送到主进程。Podman 接受信号名称或信号编号。

下表显示了几个有用的信号：

| 信号      | 价值       | 默认操作 | 评论                 |
|---------|----------|------|--------------------|
| SIGHUP  | 1个       | Term | 在控制终端上检测到挂断或控制进程死亡 |
| SIGINT  | 2个       | Term | 键盘中断               |
| SIGQUIT | 3个       | Core | 从键盘退出              |
| SIGILL  | 4个       | Core | 非法教学               |
| SIGABRT | 6个       | Core | 中止信号来自abort(3)     |
| SIGFPE  | 8个       | Core | 浮点异常               |
| SIGKILL | 9        | Term | 杀死信号               |
| SIGSEGV | 11       | Core | 无效的内存引用            |
| SIGPIPE | 13       | Term | 破损的管道：写入没有读者的管道    |
| SIGALRM | 14       | Term | 定时器信号来自alarm(2)    |
| SIGTERM | 15       | Term | 终止信号               |
| SIGUSR1 | 30,10,16 | Term | 用户自定义信号 1          |
| SIGUSR2 | 31,12,17 | Term | 用户自定义信号 2          |
| SIGCHLD | 20,17,18 | Ign  | 孩子停止或终止            |
| SIGCONT | 19,18,25 | Cont | 如果停止则继续            |
| SIGSTOP | 17,19,23 | Stop | 停止进程               |
| SIGTSTP | 18,20,24 | Stop | 在 tty 停止输入         |
| SIGTTIN | 21,21,26 | Stop | 后台进程的 tty 输入       |
| SIGTTOU | 22,22,27 | Stop | 后台进程的 tty 输出       |

每个信号的默认操作指定如下：
| 默认操作 | 描述           |
|------|--------------|
| Term | 终止进程。        |
| Core | 终止进程并生成核心转储。 |
| Ign  | 信号被忽略。       |
| Stop | 停止进程。        |



###  5.7 容器启动 (podman start)

```bash
$ podman start upbeat_lederberg 
upbeat_lederberg
```
### 5.8 容器重启 (podman restart)

```bash
$ p restart busybox
296aded1f9f39bf7b13f7931ba2063c670cbed9fa0603c6f461087c8f7283fbf
```

### 5.9 容器删除 (pofman rm)

```bash
#先停止容器再删除
$ podman rm upbeat_lederberg 
Error: cannot remove container 4b05d6c539b387139cb8488a3b988244f4e317cd616e46d9af87c89b79c4edfa as it is running - running or paused containers cannot be removed without force: container state improper
$ podman stop upbeat_lederberg 
4b05d6c539b387139cb8488a3b988244f4e317cd616e46d9af87c89b79c4edfa
$ podman rm upbeat_lederberg 
4b05d6c539b387139cb8488a3b988244f4e317cd616e46d9af87c89b79c4edfa

#强制删除正在运行的容器
$ podman ps
CONTAINER ID  IMAGE                             COMMAND  CREATED        STATUS            PORTS  NAMES
c7fee70ab080  docker.io/library/busybox:latest  sh       2 minutes ago  Up 2 minutes ago         awesome_morse
78717b0f9b0c  docker.io/library/busybox:latest  sh       2 minutes ago  Up 2 minutes ago         strange_fermat
$ podman rm -f awesome_morse 
c7fee70ab0805e31b7834f1fd5748e8995a4fc499009ab95e6bf11ae4954b1bd
```
将所有不运行的容器全部删除

```bash
podman container prune
```
### 5.10 容器日志 (podman logs)

```bash
$ podman logs -t b3f2436bdb978c1d33b1387afb5d7ba7e3243ed2ce908db431ac0069da86cb45
```
只查看容器日志的最后两行
```bash
$ podman logs --tail 2 b3f2436bdb97
```

查看所有一个容器日志信息

```bash
$ podman logs -t --since 0 myserver
```
从某一个时间段查询日志

```bash
$ podman logs -t --since 2017-08-07T10:10:09.055837383-04:00 myserver
```
查询容器最近10分钟的日志。

```bash
podman logs --since 10m myserver
```
查询一个容器30分钟前的日志。

```bash
podman logs --until 30m myserver
```

### 5.11 容器 checkpoint/restore
检查点目前仅适用于根容器。因此，您必须以 root 身份运行示例容器。除了为每个命令添加前缀之外 `sudo`，您还可以通过 预先切换到 root 用户`sudo -i`。

```bash
$ podman run -dt -p 8080:80/tcp docker.io/library/httpd
Trying to pull docker.io/library/httpd...
Getting image source signatures
Copying blob eff15d958d66 skipped: already exists  
Copying blob 0d58b11d2867 [======================================] 23.0MiB / 23.0MiB
Copying blob ab86dc02235d done  
Copying blob e88da7cb925c done  
Copying blob ba1caf8ba86c done  
Copying config ad17c88403 done  
Writing manifest to image destination
Storing signatures
51eb5d6f607c97392426b717326e6fdfc5e066ecd617ac273a34d320e3e9afca

$ podman ps
CONTAINER ID  IMAGE                             COMMAND           CREATED         STATUS             PORTS                 NAMES
51eb5d6f607c  docker.io/library/httpd:latest    httpd-foreground  13 seconds ago  Up 13 seconds ago  0.0.0.0:8080->80/tcp  suspicious_jones

```
检查点容器会停止容器，同时将容器中所有进程的状态写入磁盘。有了这个，容器可以在以后恢复并在与检查点完全相同的时间点继续运行。此功能需要在系统上安装[CRIU 3.11](https://www.criu.org/Main_Page)或更高版本。
`centos`安装`criu`
```bash
$ sudo yum install protobuf protobuf-c gcc protobuf-c-devel protobuf-compiler protobuf-devel protobuf-python -y
$ sudo yum install pkg-config python-ipaddr  libbsd  iproute2 libcap-devel libnet-devel libnl3-devel -y
$ sudo yum install asciidoc  xmlto -y

#目前最新版本v3.16,克隆下载
$ git clone -b v3.16 https://github.com/checkpoint-restore/criu.git
$ cd criu
#编译安装
$ make & make install 

#检测
$ criu -V
Version: 3.16
GitID: v3.16
$ criu check
Looks good.

```

```bash
$ podman container checkpoint suspicious_jones
ERRO[0000] container is not destroyed                   
ERRO[0000] criu failed: type NOTIFY errno 0
log file: /var/lib/containers/storage/overlay-containers/51eb5d6f607c97392426b717326e6fdfc5e066ecd617ac273a34d320e3e9afca/userdata/dump.log 
Error: failed to checkpoint container 51eb5d6f607c97392426b717326e6fdfc5e066ecd617ac273a34d320e3e9afca: `/usr/bin/runc checkpoint --image-path /var/lib/containers/storage/overlay-containers/51eb5d6f607c97392426b717326e6fdfc5e066ecd617ac273a34d320e3e9afca/userdata/checkpoint --work-path /var/lib/containers/storage/overlay-containers/51eb5d6f607c97392426b717326e6fdfc5e066ecd617ac273a34d320e3e9afca/userdata 51eb5d6f607c97392426b717326e6fdfc5e066ecd617ac273a34d320e3e9afca` failed: exit status 1
```
[解决方法请参考](https://ghostwritten.blog.csdn.net/article/details/121617741)

```bash
$ podman container checkpoint exciting_neumann
2135a176f1b5f75668836c3a4e374cd1bcfc17545414ba5fb41be6558078f470

$ podman container restore exciting_neumann
2135a176f1b5f75668836c3a4e374cd1bcfc17545414ba5fb41be6558078f470

$ podman ps
CONTAINER ID  IMAGE                           COMMAND           CREATED            STATUS             PORTS                 NAMES
2135a176f1b5  docker.io/library/httpd:latest  httpd-foreground  About an hour ago  Up 23 minutes ago  0.0.0.0:8080->80/tcp  exciting_neumann

```
[更多细节请参考官方](https://podman.io/getting-started/checkpoint)

###  5.12 容器健康检查
健康检查”是一种用户可以确定在容器内运行的主进程的“健康”或准备情况的方式。这不仅仅是一个简单的“我的容器在运行吗？” 题。更像是“我的应用程序准备好了吗？” 因此，健康检查实际上是一种验证容器及其应用程序是否响应的方法。
健康检查由五个基本组成部分组成：
 1. Command
 2. Retries
 3. Interval
 4. Start-period
 5. Timeout

```bash
$ sudo podman run -dt --name hc1 --healthcheck-command 'CMD-SHELL curl http://localhost || exit 1' --healthcheck-interval=0 quay.io/libpod/alpine_nginx:latest
d25ee6faaf6e5e12c09e734b1ac675385fe4d4e8b52504dd01a60e1b726e3edb
$ sudo podman healthcheck run hc1
Healthy
$ echo $?
0
```

在`podman run`命令上，请注意`--healthcheck-command`定义 `healthcheck` 命令本身的位置的使用。请记住，这是一个在容器本身“内部”运行的命令。在这种情况下，curl 命令需要存在于容器中。另外，请注意`--healthcheck-interval`标志及其“0”值。间隔标志定义运行健康检查的时间频率。“0”值表示我们要手动运行健康检查。

[healthcheck更多细节请参考](https://developers.redhat.com/blog/2019/04/18/monitoring-container-vitality-and-availability-with-podman#)

###  5.13 容器用systemd 管理
由于 Podman 没有 daemon ，所以没办法像 docker 一样通过指定参数 `--restart=always` 在 docker 进程启动时自动拉起镜像。 Podman 通过 `systemd` 来支持该功能。

首先，我们需要准备一个已经可以正常运行的容器：

```bash
$ podman ps
CONTAINER ID  IMAGE                             COMMAND  CREATED        STATUS            PORTS  NAMES
94de914cfc45  docker.io/library/busybox:latest  sh       5 minutes ago  Up 5 minutes ago         hello
```
编写 `systemd` 配置文件，通常默认路径为： `/usr/lib/systemd/system/`

```bash
$ vim /usr/lib/systemd/system/hello.service 
[Unit]
After=network.target

[Service]
Restart=always
ExecStart=/usr/bin/podman start -a hello
ExecStop=/usr/bin/podman stop -t 10 hello

[Install]
WantedBy=multi-user.target
```
编写完成后，我们需要执行下 `systemctl daemon-reload` 重新加载一次配置，然后就可以通过 systemctl 来控制容器的启停、开机自启动了。

```bash
$ systemctl daemon-reload
$ systemctl status hello
● hello.service
   Loaded: loaded (/usr/lib/systemd/system/hello.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
$ systemctl start hello
$ systemctl status hello
● hello.service
   Loaded: loaded (/usr/lib/systemd/system/hello.service; disabled; vendor preset: disabled)
   Active: active (running) since 一 2021-11-29 21:52:50 CST; 3s ago
 Main PID: 4760 (podman)
    Tasks: 8
   CGroup: /system.slice/hello.service
           └─4760 /usr/bin/podman start -a hello

11月 29 21:52:50 localhost.localdomain systemd[1]: Started hello.service.
$ systemctl enable hello
Created symlink from /etc/systemd/system/multi-user.target.wants/hello.service to /usr/lib/systemd/system/hello.service.

#停止hello容器
$ podman ps
CONTAINER ID  IMAGE                             COMMAND  CREATED        STATUS            PORTS  NAMES
94de914cfc45  docker.io/library/busybox:latest  sh       8 minutes ago  Up 8 minutes ago         hello
$ systemctl stop hello
$ podman ps
CONTAINER ID  IMAGE  COMMAND  CREATED  STATUS  PORTS  NAMES

#启动hello容器
$ systemctl start hello
$ podman ps
CONTAINER ID  IMAGE                             COMMAND  CREATED        STATUS            PORTS  NAMES
94de914cfc45  docker.io/library/busybox:latest  sh       9 minutes ago  Up 2 seconds ago         hello

```

###  5.13  挂载存储相关

```bash
$ mkdir /home/student/dbfiles
```

容器中运行进程的用户必须能够将文件写入目录。权限应使用容器中的数字用户 ID (UID) 定义。Red Hat 提供的 MySQL 服务，UID 是 27，podman unshare提供了一个会话，可以在与容器内运行的进程相同的用户命名空间内执行命令。

```bash
$ podman unshare chown -R 27:27 /home/student/dbfiles
```
将`container_file_t`上下文应用于目录（和所有子目录）以允许容器访问其所有内容。

```bash
$ sudo semanage fcontext -a -t container_file_t  '/home/student/dbfiles(/.*)?'
```
- semanage: 用于配制SElinux元素
- fcontext: file-context mapping definition
- -a: 添加fcontext-type
- -t: SELinux type

将您在第一步中设置的 SELinux 容器策略应用到新创建的目录中：

```bash
$ sudo restorecon -Rv /home/student/dbfiles
```
查看context_type

```bash
ls -ldZ  /home/student/dbfiles	
```

要使用`/home/student/dbfiles`作为MySQL 服务器数据库文件的主机目录，这些文件应位于`/var/lib/mysql`名为 的 MySQL 容器映像内mysql，请使用以下命令：

```bash
$ podman run -v /home/student/dbfiles:/var/lib/mysql rhmap47/mysql
```

在此命令中，如果容器映像`/var/lib/mysql`中已经存在mysql，则`/home/student/dbfiles`挂载会覆盖但不会删除容器映像中的内容。如果移除安装，则可以再次访问原始内容。

##  6. k8s 管理
###  6.1 pod管理

 - pod不但管理起来像个docker，它还可以管理pod
 - Podman 的 YAML 和 k8s pod yaml 文件格式是兼容的。

首先，我们来创建一个 Pod：
```bash
$ podman pod create --name postgresql -p 5432 -p 9187
ERRO[0042] Error freeing pod lock after failed creation: no such file or directory 
Error: unable to create pod: error adding Infra Container: unable to pull k8s.gcr.io/pause:3.1: unable to pull image: Error initializing source docker://k8s.gcr.io/pause:3.1: error pinging docker registry k8s.gcr.io: Get https://k8s.gcr.io/v2/: dial tcp 74.125.204.82:443: connect: connection refused
```
如何绕过尴尬的k8s.gcr.io

```bash
$ podman pull registry.aliyuncs.com/google_containers/pause:3.1
Trying to pull registry.aliyuncs.com/google_containers/pause:3.1...
Getting image source signatures
Copying blob 67ddbfb20a22 done  
Copying config da86e6ba6c [======================================] 1.6KiB / 1.6KiB
Writing manifest to image destination
Storing signatures
da86e6ba6ca197bf6bc5e9d900febd906b133eaa4750e6bed647b0fbe50ed43e

$ podman tag registry.aliyuncs.com/google_containers/pause:3.1 k8s.gcr.io/pause:3.1
[root@localhost ~]# podman pod create --name postgresql -p 5432 -p 9187
d6825c6afe78a471d39190c8caf57b7f034e604b2ad64a4d1a2918894d5ae890

#容器模式查看
$ podman ps -a
CONTAINER ID  IMAGE                                              COMMAND           CREATED         STATUS                     PORTS                   NAMES
2a7e0325d459  registry.aliyuncs.com/google_containers/pause:3.1                    22 seconds ago  Created                    0.0.0.0:5432->5432/tcp  d6825c6afe78-infra

#pod模式查看
podman pod ls
POD ID         NAME         STATUS    CREATED         # OF CONTAINERS   INFRA ID
d6825c6afe78   postgresql   Created   2 minutes ago   1                 2a7e0325d459

$ podman pod ps
POD ID         NAME         STATUS    CREATED         # OF CONTAINERS   INFRA ID
d6825c6afe78   postgresql   Created   3 minutes ago   1                 2a7e0325d459

```
运行一个demo

```bash
#运行postgress容器
$ podman run -d --pod postgresql -e POSTGRES_PASSWORD=password postgres:latest
Trying to pull docker.io/library/postgres:latest...
Getting image source signatures
Copying blob eff15d958d66 skipped: already exists  
Copying blob 108afa831d95 done  
Copying blob 7037ade1772d done  
Copying blob c877668f09b8 done  
Copying blob e5821d5963ce done  
Copying blob 5be06220aa99 done  
Copying blob de2b4ab3ade5 done  
Copying blob a60906bcf87f done  
Copying blob d46bc4e17ddc done  
Copying blob a1c85f71c941 done  
Copying blob 69f50e484cab done  
Copying blob 2f8a286a55a4 done  
Copying blob 8d590b0d720c done  
Copying config 577410342f done  
Writing manifest to image destination
Storing signatures
8906e9ea92780c6c550ca4ae26dd398300ddb5d3f518d122bebb88a33a26ffeb

#运行postgress_exporter容器
podman run -d --pod postgresql -e DATA_SOURCE_NAME="postgresql://postgres:password@localhost:5432/postgres?sslmode=disable"  wrouesnel/postgres_exporter
Trying to pull docker.io/wrouesnel/postgres_exporter...
Getting image source signatures
Copying blob 45b42c59be33 done  
Copying blob 4634a89d50c2 done  
Copying blob fbcf7c278f83 done  
Copying config 9fe9d3d021 done  
Writing manifest to image destination
Storing signatures
36f67b3e0fd06c1385595320288b0e41b183407859c1f4c48abf028599b06971

$ podman ps -a
CONTAINER ID  IMAGE                                              COMMAND           CREATED         STATUS                     PORTS                   NAMES
36f67b3e0fd0  docker.io/wrouesnel/postgres_exporter:latest                         3 minutes ago   Up 3 minutes ago           0.0.0.0:5432->5432/tcp  objective_lehmann
8906e9ea9278  docker.io/library/postgres:latest                  postgres          9 minutes ago   Up 9 minutes ago           0.0.0.0:5432->5432/tcp  reverent_shaw


```

首先我们创建了一个 `Pod`，端口映射是在 Pod 这个级别配置的，然后在这个 Pod 中，我们创建了两个 container，分别是：`postgres` 和 `postgres_exporter` ，其中 `postgres_exporter` 主要是暴露 `metrics` 用于 `Prometheus` 抓取进行监控。

我们可以通过 curl 相应端口来验证是否正常工作：

```bash
$  curl localhost:9187/metrics 
# HELP go_gc_duration_seconds A summary of the GC invocation durations.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 0
go_gc_duration_seconds{quantile="0.25"} 0
go_gc_duration_seconds{quantile="0.5"} 0
go_gc_duration_seconds{quantile="0.75"} 0
go_gc_duration_seconds{quantile="1"} 0
go_gc_duration_seconds_sum 0
go_gc_duration_seconds_count 0
·······················
```
可以看到已经正确的获取到了相应 `metrics` 数值，可以通过 `podman pod top` 来获取当前进程状态：

```bash
$ podman pod top postgresql   
USER                PID   PPID   %CPU    ELAPSED            TTY   TIME   COMMAND
0                   1     0      0.000   11m43.122459727s   ?     0s     /pause 
postgres_exporter   1     0      0.000   5m51.124897013s    ?     0s     /postgres_exporter 
postgres            1     0      0.000   11m43.127713837s   ?     0s     postgres 
postgres            59    1      0.000   11m41.127813057s   ?     0s     postgres: checkpointer  
postgres            60    1      0.000   11m41.127878822s   ?     0s     postgres: background writer  
postgres            61    1      0.000   11m41.127943364s   ?     0s     postgres: walwriter  
postgres            62    1      0.000   11m41.128005075s   ?     0s     postgres: autovacuum launcher  
postgres            63    1      0.000   11m41.128133869s   ?     0s     postgres: stats collector  
postgres            64    1      0.000   11m41.128229337s   ?     0s     postgres: logical replication launcher  
postgres            74    1      0.000   1m49.12829442s     ?     0s     postgres: postgres postgres ::1(34118) idle 
```
###  6.2 yaml管理
通过 `podman generate` 命令可以生成 k8s 可用的 `YAML` 文件：

```bash
$ podman generate kube postgresql > postgresql.yaml
$ podman generate kube postgresql
# Generation of Kubernetes YAML is still under development!
#
# Save the output of this file and use kubectl create -f to import
# it into Kubernetes.
#
# Created with podman-1.6.4
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2021-11-29T14:34:40Z"
  labels:
    app: postgresql
  name: postgresql
spec:
  containers:
  - env:
    - name: PATH
      value: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    - name: TERM
      value: xterm
    - name: HOSTNAME
      value: postgresql
    - name: container
      value: podman
    - name: DATA_SOURCE_NAME
      value: postgresql://postgres:password@localhost:5432/postgres?sslmode=disable
    image: docker.io/wrouesnel/postgres_exporter:latest
    name: objectivelehmann
    ports:
    - containerPort: 5432
      hostPort: 5432
      protocol: TCP
    - containerPort: 9187
      hostPort: 9187
      protocol: TCP
    resources: {}
    securityContext:
      allowPrivilegeEscalation: true
      capabilities: {}
      privileged: false
      readOnlyRootFilesystem: false
      runAsUser: 20001
    workingDir: /
  - command:
    - postgres
    env:
    - name: PATH
      value: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/lib/postgresql/14/bin
    - name: TERM
      value: xterm
    - name: HOSTNAME
      value: postgresql
    - name: container
      value: podman
    - name: PGDATA
      value: /var/lib/postgresql/data
    - name: POSTGRES_PASSWORD
      value: password
    - name: GOSU_VERSION
      value: "1.12"
    - name: LANG
      value: en_US.utf8
    - name: PG_MAJOR
      value: "14"
    - name: PG_VERSION
      value: 14.1-1.pgdg110+1
    image: docker.io/library/postgres:latest
    name: reverentshaw
    resources: {}
    securityContext:
      allowPrivilegeEscalation: true
      capabilities: {}
      privileged: false
      readOnlyRootFilesystem: false
    workingDir: /
status: {}
```
使用 `podman play` 命令可以直接创建完整的 Pod 及其所拥有的容器：

```bash
podman play kube postgresql.yaml
```


✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>



 - [docker 命令](https://blog.csdn.net/xixihahalelehehe/article/details/123378401)
 - [podman 命令](https://blog.csdn.net/xixihahalelehehe/article/details/121611523)
 -  [crictl 命令](https://blog.csdn.net/xixihahalelehehe/article/details/116591151)
 - [operator-sdk 命令](https://blog.csdn.net/xixihahalelehehe/article/details/112024963)
 - [skopeo 命令](https://ghostwritten.blog.csdn.net/article/details/127342981)

参考：

 1. [podman【2】 命令手册](https://docs.podman.io/en/latest/Commands.html)
 2. [podman【3】高级管理](https://docs.podman.io/en/latest/Tutorials.html)








