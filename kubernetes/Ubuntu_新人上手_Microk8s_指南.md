![](https://i-blog.csdnimg.cn/blog_migrate/ea09d8a8785491508eea8ef0e4a2a21c.png)




看到 `Microk8s` 官网的插件很全，忍不住要玩一把。体验一下。
- [Vcenter 6.7 创建 Ubuntu 22.10 虚拟机 顺带安装 Microk8s](https://blog.csdn.net/xixihahalelehehe/article/details/129310311)

## 1. 什么是 Ubuntu 核心
[Ubuntu Core](https://ubuntu.com/core) 是为物联网和嵌入式系统设计和开发的 Ubuntu 操作系统版本。它完全由 `snap` 包构建，以创建一个安全、健壮、受限和基于事务的操作系统，易于安装、部署和升级。

## 2. 什么是 Kubernetes
[Kubernetes](https://kubernetes.io/) 是容器化应用程序的编排平台。Kubernetes 抽象计算、网络和存储资源，并以可靠和可扩展的方式管理容器生命周期。Kubernetes 采用 DevOps 原则构建，可自动执行操作任务，例如工作负载重新部署和升级，并提供用于精细资源控制的 API。

## 3. 什么是MicroK8s
[MicroK8s](https://microk8s.io/) 是一个轻量级的 [CNCF](https://www.cncf.io/) 认证的 Kubernetes 发行版，适用于云、工作站、边缘和物联网设备。作为一个快照，它在本地运行所有 Kubernetes 服务（即没有虚拟机），它将所有依赖项包含在一个包中，并获得透明的关键任务安全更新。MicroK8s 针对简单性和稳健性进行了优化，因为安装、设置和操​​作（例如启用监控服务和高可用性集群）要么是自动化的，要么是通过单个命令完成的。

## 4. 为什么选择 Microk8s on Core
`MicroK8s` 和 `Ubuntu Core` 共享可靠性和安全性等优势，具有自我修复、高可用性和自动 OTA 更新等功能。Ubuntu 是云中 Kubernetes 的首选操作系统。结合 `Ubuntu Core` 和 `MicroK8s`，创建流线型的嵌入式 Kubernetes 体验，并优化 IoT 和边缘应用程序的大小和性能。

## 5. 安装Ubuntu Core
现在您已经拥有方便的 IoT 设备，让我们从安装 `Ubuntu Core` 开始。

目前有两个指南：

- [在树莓派上安装 Ubuntu Core](https://ubuntu.com/tutorials/how-to-install-ubuntu-core-on-raspberry-pi#1-overview)
- [在 Intel NUC 上安装 Ubuntu Core](https://ubuntu.com/download/intel-nuc)

请注意，为了运行 MicroK8s，您应该使用 64 位 Ubuntu Core 版本。

如果您设法成功完成了两个指南中任何一个的步骤，您现在应该可以访问您设备上的 Ubuntu Core 终端。

## 6. Ubuntu Core上安装 MicroK8S
安装
要在 `Ubuntu Core` 上安装最新版本的 Microk8s，请运行：

```bash
snap install microk8s --channel=latest/edge/strict
```

低于预期输出：

```bash
ubuntu@ubuntu:~$ snap install microk8s --channel=latest/edge/strict

microk8s (edge/strict) v1.22.3 from Canonical✓ installed
```

安装最多可能需要几分钟，具体取决于您的硬件资源和网络连接。

此安装的 Kubernetes 版本是什么？
MicroK8s 是快速打包的，因此它将自动更新到更新的点版本。

严格限制的 MicroK8s 版本目前在专用的 snap 通道上，与上游 Kubernetes 的最新版本保持一致。

频道由轨道（或系列）和基于 MicroK8s 版本（稳定版、候选版、测试版、边缘版）的预期稳定性级别组成。有关可用版本的更多信息，请运行：

```bash
snap info microk8s
```
## 7. 启动 Microk8s
Microk8s 安装后默认不启动。要启动 MicroK8s，请运行：

```bash
sudo microk8s start
```

此命令启动所有 Kubernetes 服务，包括控制平面和工作程序。现在，要在安装完成后检查 MicroK8s 节点的状态，您可以使用：

```bash
sudo microk8s status --wait-ready
```

状态应该表明 microk8s 正在运行。

```bash
ubuntu@ubuntu:~$ sudo microk8s status
microk8s is running
high-availability: no
  datastore master nodes: 127.0.0.1:19001
  datastore standby nodes: none
addons:
  enabled:
    ha-cluster           # (core) Configure high availability on the current node
    helm                 # (core) Helm - the package manager for Kubernetes
    helm3                # (core) Helm 3 - the package manager for Kubernetes
  disabled:
    cert-manager         # (core) Cloud native certificate management
    community            # (core) The community addons repository
    dashboard            # (core) The Kubernetes dashboard
    dns                  # (core) CoreDNS
    gpu                  # (core) Automatic enablement of Nvidia CUDA
    host-access          # (core) Allow Pods connecting to Host services smoothly
    hostpath-storage     # (core) Storage class; allocates storage from host directory
    ingress              # (core) Ingress controller for external access
    kube-ovn             # (core) An advanced network fabric for Kubernetes
    mayastor             # (core) OpenEBS MayaStor
    metallb              # (core) Loadbalancer for your Kubernetes cluster
    metrics-server       # (core) K8s Metrics Server for API access to service metrics
    minio                # (core) MinIO object storage
    observability        # (core) A lightweight observability stack for logs, traces and metrics
    prometheus           # (core) Prometheus operator for monitoring and logging
    rbac                 # (core) Role-Based Access Control for authorisation
    registry             # (core) Private image registry exposed on localhost:32000
    storage              # (core) Alias to hostpath-storage add-on, deprecated
```

如果是没有魔法上网的环境，这里可能起不来。报错原因：

```bash
$ microk8s.kubectl get pods -n kube-system
NAME                                       READY   STATUS     RESTARTS   AGE
calico-kube-controllers-7db754646d-ml4wq   0/1     Pending    0          7m50s
calico-node-td999                          0/1     Init:0/2   0          38s

$ microk8s.kubectl get pods -n kube-system calico-node-td999 -oyaml
.......
Failed to create pod sandbox: rpc error: code = DeadlineExceeded desc = failed to get sandbox image "registry.k8s.io/pause:3.7": failed to pull image "registry.k8s.io/pause:3.7": failed to pull and unpack image "registry.k8s.io/pause:3.7": failed to resolve reference "registry.k8s.io/pause:3.7": failed to do request: Head "https://asia-east1-docker.pkg.dev/v2/k8s-artifacts-prod/images/pause/manifests/3.7": dial tcp 64.233.189.82:443: i/o timeout
```
你需要手动拉取。

```bash
microk8s.ctr images pull docker.io/ghostwritten/registry.k8s.io.pause:3.7
microk8s.ctr images tag docker.io/ghostwritten/registry.k8s.io.pause:3.7 registry.k8s.io/pause:3.7
```

## 8. 启用必要的 MicroK8s 插件
现在您已经启动并运行了 Kubernetes 服务，您应该设置其他服务，例如 Kubernetes 仪表板、CoreDNS 或本地存储，以充分利用您的 Kubernetes。其中许多服务都可以作为 MicroK8s 插件使用，并且可以通过运行 microk8s enable 命令轻松启用：

```bash
sudo microk8s enable dns dashboard storage
```

可以通过运行 microk8s disable 命令随时禁用这些插件：

```bash
sudo microk8s disable dns dashboard storage
```

您可以使用 microk8s status 命令查看可用插件列表和当前启用的插件。

最重要的插件列表
- `dns`：部署DNS。其他人可能需要此插件，因此我们建议您始终启用它。
`dashboard`: 部署 kubernetes 仪表板。
- `storage`：创建默认存储类。此存储类使用指向主机上目录的 hostpath-provisioner。
`ingress`：创建入口控制器。
- `gpu`：通过启用 nvidia-docker 运行时和 nvidia-device-plugin-daemonset 将 GPU 暴露给 MicroK8s。要求主机系统上已安装 NVIDIA 驱动程序。
- `istio`：部署核心 Istio 服务。您可以使用 microk8s istioctl 命令来管理您的部署。
- `registry`：部署一个 docker private registry 并在 localhost:32000 上公开它。存储插件将作为此插件的一部分启用。

## 9. 部署示例容器工作负载
您现在可以使用 microk8s kubectl 来部署您的容器。在此示例中，我们部署了 nodered，这是一种用于将硬件设备连接在一起的编程工具

```bash
sudo microk8s kubectl create deployment nodered --image=nodered/node-red
```

使用 kubectl 检查 pod：

```bash
ubuntu@ubuntu:~$ sudo microk8s kubectl get pods
NAME                        READY     STATUS            RESTARTS  AGE
nodered-7555b955f9-68cl9    0/1       ContainerCreating 0         3s
ubuntu@ubuntu:~$ sudo microk8s kubectl get pods
NAME                        READY     STATUS            RESTARTS  AGE
nodered-7555b955f9-68cl9    1/1       Running           0         16s
```

接下来需要使用 kubectl 命令公开部署，以使其可从网络访问：

```bash
sudo microk8s kubectl expose deployment nodered --type=NodePort --port=1880 --name=nodered-service
```

## 10. 检查部署状态并访问您的应用程序
您可以使用以下命令检查部署状态：

```bash
$ sudo microk8s kubectl get services
ubuntu@ubuntu:~$ sudo microk8s kubectl get services
NAME               TYPE         CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes         ClusterIP    10.152.183.1    <none>        443/TCP        81m
nodered-service    NodePort     10.152.183.46   <none>        1880:30663/TCP 5s
```
暴露的端口是随机生成的。在上面的例子中，我们可以看到端口是`30663`。

为了访问应用程序的图形界面，您需要打开浏览器并输入以下 URL 方案：`http://:<EXPOSED_PORT>`

示例：`http://192.168.1.222:30663/`

![](https://i-blog.csdnimg.cn/blog_migrate/735bf6b0a51acd51bf0ace5f87d8b0eb.png)
## 11. 管理镜像
###  11.1 拉取

```bash
$ microk8s ctr images pull docker.io/calico/cni:v3.23.5
docker.io/calico/cni:v3.23.5:                                                     resolved       |++++++++++++++++++++++++++++++++++++++|
index-sha256:7ca5c455cff6c0d661e33918d95a1133afb450411dbfb7e4369a9ecf5e0212dc:    done           |++++++++++++++++++++++++++++++++++++++|
manifest-sha256:9c5055a2b5bc0237ab160aee058135ca9f2a8f3c3eee313747a02edcec482f29: done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:4f4fb700ef54461cfa02571ae0db9a0dc1e0cdb5577484a6d75e68dc38e8acc1:    done           |++++++++++++++++++++++++++++++++++++++|
config-sha256:1c979d623de9aef043cb4ff489da5636d61c39e30676224af0055240e1816382:   done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:cc0e45adf05a30a90384ba7024dbabdad9ae0bcd7b5a535c28dede741298fea3:    done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:51729c6e2acda05a05e203289f5956954814d878f67feb1a03f9941ec5b4008b:    done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:7430548aa23e56c14da929bbe5e9a2af0f9fd0beca3bd95e8925244058b83748:    done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:47c5dbbec31222325790ebad8c07d270a63689bd10dc8f54115c65db7c30ad1f:    done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:8efc3d73e2741a93be09f68c859da466f525b9d0bddb1cd2b2b633f14f232941:    done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:4c98a4f67c5a7b1058111d463051c98b23e46b75fc943fc2535899a73fc0c9f1:    done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:050b055d5078c5c6ad085d106c232561b0c705aa2173edafd5e7a94a1e908fc5:    done           |++++++++++++++++++++++++++++++++++++++|
elapsed: 28.7s                                                                    total:  103.0  (3.6 MiB/s)

unpacking linux/amd64 sha256:7ca5c455cff6c0d661e33918d95a1133afb450411dbfb7e4369a9ecf5e0212dc...
done: 11.392952756s
```
###  查看

```bash
microk8s.ctr image ls
REF                                                                     TYPE                                                      DIGEST
                      SIZE      PLATFORMS                                          LABELS
docker.io/calico/cni:v3.23.5                                            application/vnd.docker.distribution.manifest.list.v2+json sha256:7ca5c455cff6c0d661e33918d95a1133afb450411dbfb7e4369a9ecf5e0212dc 103.0 MiB linux/amd64,linux/arm/v7,linux/arm64,linux/ppc64le io.cri-containerd.image=managed
docker.io/calico/kube-controllers:v3.23.5                               application/vnd.docker.distribution.manifest.list.v2+json sha256:58cc91c551e9e941a752e205eefed1c8da56f97a51e054b3d341b67bb7bf27eb 51.3 MiB  linux/amd64,linux/arm/v7,linux/arm64,linux/ppc64le io.cri-containerd.image=managed
docker.io/calico/node:v3.23.5                                           application/vnd.docker.distribution.manifest.list.v2+json sha256:b7f4f7a0ce463de5d294fdf2bb13f61035ec6e3e5ee05dd61dcc8e79bc29d934 71.6 MiB  linux/amd64,linux/arm/v7,linux/arm64,linux/ppc64le io.cri-containerd.image=managed
docker.io/ghostwritten/registry.k8s.io.pause:3.7                        application/vnd.docker.distribution.manifest.v2+json      sha256:445a99db22e9add9bfb15ddb1980861a329e5dff5c88d7eec9cbf08b6b2f4eb1 301.3 KiB linux/amd64                                        io.cri-containerd.image=managed
registry.k8s.io/pause:3.7                                               application/vnd.docker.distribution.manifest.v2+json      sha256:445a99db22e9add9bfb15ddb1980861a329e5dff5c88d7eec9cbf08b6b2f4eb1 301.3 KiB linux/amd64                                        io.cri-containerd.image=managed
sha256:1c979d623de9aef043cb4ff489da5636d61c39e30676224af0055240e1816382 application/vnd.docker.distribution.manifest.list.v2+json sha256:7ca5c455cff6c0d661e33918d95a1133afb450411dbfb7e4369a9ecf5e0212dc 103.0 MiB linux/amd64,linux/arm/v7,linux/arm64,linux/ppc64le io.cri-containerd.image=managed
sha256:221177c6082a88ea4f6240ab2450d540955ac6f4d5454f0e15751b653ebda165 application/vnd.docker.distribution.manifest.v2+json      sha256:445a99db22e9add9bfb15ddb1980861a329e5dff5c88d7eec9cbf08b6b2f4eb1 301.3 KiB linux/amd64                                        io.cri-containerd.image=managed
sha256:b6e6ee0788f2079219fecb418f573bbaad4c07f6f82b712ccc72684db8cc2deb application/vnd.docker.distribution.manifest.list.v2+json sha256:b7f4f7a0ce463de5d294fdf2bb13f61035ec6e3e5ee05dd61dcc8e79bc29d934 71.6 MiB  linux/amd64,linux/arm/v7,linux/arm64,linux/ppc64le io.cri-containerd.image=managed
sha256:ea5536b1fa4a86e5ecf0803b7a1118d797f5ef51d2f452acf0b701d64dc6fcd9 application/vnd.docker.distribution.manifest.list.v2+json sha256:58cc91c551e9e941a752e205eefed1c8da56f97a51e054b3d341b67bb7bf27eb 51.3 MiB  linux/amd64,linux/arm/v7,linux/arm64,linux/ppc64le io.cri-containerd.image=managed
```

###  打标签

```bash
microk8s.ctr images tag docker.io/ghostwritten/registry.k8s.io.pause:3.7 registry.k8s.io/pause:3.7
```

参考：
- [Getting started with MicroK8s on Ubuntu Core](https://ubuntu.com/tutorials/getting-started-with-microk8s-on-ubuntu-core?_ga=2.230245724.1567867401.1677824672-1974441290.1677824672&_gac=1.28492110.1677766080.CjwKCAiAr4GgBhBFEiwAgwORrXhCUOTexAEC0vih6Nyu55IyoQ9OUIzRn-sdB3_MkXU9pLrIdnJZpBoCD8UQAvD_BwE#1-introduction)
