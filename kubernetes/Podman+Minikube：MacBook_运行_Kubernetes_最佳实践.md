![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/c9ace0d08be44c97b09a4d239e40c3eb.png)

# 简介
在现代软件开发中，[Kubernetes](https://kubernetes.io/)作为容器编排的事实标准，已成为云原生应用的核心组成部分。对于开发者来说，在本地环境中搭建和测试Kubernetes集群显得尤为重要。而在这方面，结合[MacBook](https://www.apple.com/hk/en/mac/)、[Podman](https://podman.io/)和[Minikube](https://minikube.sigs.k8s.io/docs/)的组合，提供了一种高效、便捷的解决方案。

首先，MacBook因其稳定的性能和优秀的用户体验，成为了许多开发者的首选设备。Podman作为一个无守护进程的容器管理工具，允许用户在不依赖Docker的情况下，运行和管理容器。它的优点在于提供了更强的安全性和灵活性，支持无根用户操作，并且与Docker CLI命令兼容，使得用户可以轻松上手。通过Podman，开发者可以在本地创建和管理容器，而无需担心传统容器引擎可能带来的复杂性。

Minikube则是为本地开发提供Kubernetes集群的理想工具。它允许用户在本地快速启动一个单节点的Kubernetes集群，适合开发和测试。Minikube不仅支持多种虚拟化驱动，包括VirtualBox、HyperKit等，还具备丰富的插件和扩展功能，能够轻松满足不同的开发需求。通过Minikube，开发者可以迅速体验Kubernetes的强大功能，从而提高开发效率。

将Podman和Minikube结合使用，可以充分发挥两者的优势。在MacBook上，用户只需安装这两个工具，便能快速构建一个完整的Kubernetes开发环境。无论是进行应用开发、测试，还是学习Kubernetes的使用，Podman和Minikube的组合都能提供一个灵活、可扩展的解决方案。

此外，这种组合也极大地简化了工作流程。开发者可以使用Podman创建和管理容器，使用Minikube进行Kubernetes集群的管理与测试。无论是构建微服务架构、进行CI/CD集成，还是进行环境一致性测试，这一套组合都能轻松应对。



# 安装 podman desktop
访问 https://podman.io/ 下载安装即可。
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/06e10a850c6a46069eab148e99ee95b8.png)



```bash
podman machine init
Looking up Podman Machine image at quay.io/podman/machine-os:5.2 to create VM
Getting image source signatures
Copying blob 12113a444353 done   |
Copying config 44136fa355 done   |
Writing manifest to image destination
12113a44435343a5cb2f6fc1c8cb4589b359bafd6123aab0c293e9cf884184ae
Extracting compressed file: podman-machine-default-arm64.raw: done
Machine init complete
To start your machine run:

	podman machine start

zongxun@zongxundeMacBook-Pro ~ % podman machine start
Starting machine "podman-machine-default"

This machine is currently configured in rootless mode. If your containers
require root permissions (e.g. ports < 1024), or if you run into compatibility
issues with non-podman clients, you can switch using the following command:

	podman machine set --rootful

API forwarding listening on: /var/run/docker.sock
Docker API clients default to this address. You do not need to set DOCKER_HOST.

Machine "podman-machine-default" started successfully
```

创建minikube集群

```bash
$ cat start.sh
 #!/usr/bin/env bash

set -o nounset

KUBE_NAME="minikube"
KUBE_VERSION="v1.30.0"

minikube start -p "${KUBE_NAME}" --driver=podman --docker-env HTTP_PROXY=http://192.168.21.101:7890 --docker-env HTTPS_PROXY=http://192.168.21.101:7890 --container-runtime=cri-o --kubernetes-version="${KUBE_VERSION}" --memory=no-limit --cpus=no-limit
```

```bash
sh -x start.sh
+ set -o nounset
+ KUBE_NAME=minikube
+ KUBE_VERSION=v1.30.0
+ minikube start -p minikube --driver=podman --docker-env HTTP_PROXY=http://192.168.21.101:7890 --docker-env HTTPS_PROXY=http://192.168.21.101:7890 --container-runtime=cri-o --kubernetes-version=v1.30.0 --memory=no-limit --cpus=no-limit
😄  Darwin 15.1 (arm64) 上的 minikube v1.33.1
✨  根据用户配置使用 podman (实验性功能) 驱动程序
📌  Using rootless Podman driver
👍  Starting "minikube" primary control-plane node in "minikube" cluster
🚜  Pulling base image v0.0.44 ...
E1031 18:36:52.046778   61560 cache.go:189] Error downloading kic artifacts:  not yet implemented, see issue #8426
🔥  Creating podman container (CPUs=no-limit, Memory=no-limit) ...\
/
\

🌐  找到的网络选项：
    ▪ HTTP_PROXY=http://192.168.21.101:7890
    ▪ HTTPS_PROXY=http://192.168.21.101:7890
    ▪ NO_PROXY=localhost,127.0.0.1,10.96.0.0/12,192.168.59.0/24,192.168.49.0/24,192.168.39.0/24,*.bsgchina.com
🎁  正在 CRI-O 1.24.6 中准备 Kubernetes v1.30.0…
    ▪ env HTTP_PROXY=http://192.168.21.101:7890
    ▪ env HTTPS_PROXY=http://192.168.21.101:7890
    ▪ env NO_PROXY=localhost,127.0.0.1,10.96.0.0/12,192.168.59.0/24,192.168.49.0/24,192.168.39.0/24,*.bsgchina.com
    ▪ 正在生成证书和密钥...
    ▪ 正在启动控制平面...
    ▪ 配置 RBAC 规则 ...
🔗  配置 CNI (Container Networking Interface) ...
🔎  正在验证 Kubernetes 组件...
    ▪ 正在使用镜像 gcr.io/k8s-minikube/storage-provisioner:v5
🌟  启用插件： storage-provisioner, default-storageclass
🏄  完成！kubectl 现在已配置，默认使用"minikube"集群和"default"命名空间
```

检查

```bash
$ kubectl get node
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   88s   v1.30.0

$ kubectl version
Client Version: v1.30.3
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
Server Version: v1.30.0

$ kubectl get pod -A
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
kube-system   coredns-7db6d8ff4d-pjb5q           1/1     Running   0          97s
kube-system   etcd-minikube                      1/1     Running   0          111s
kube-system   kindnet-89csd                      1/1     Running   0          96s
kube-system   kube-apiserver-minikube            1/1     Running   0          111s
kube-system   kube-controller-manager-minikube   1/1     Running   0          110s
kube-system   kube-proxy-25h9f                   1/1     Running   0          96s
kube-system   kube-scheduler-minikube            1/1     Running   0          110s
kube-system   storage-provisioner                1/1     Running   0          109s
```

