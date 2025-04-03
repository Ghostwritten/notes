
![上海嘉善路](https://i-blog.csdnimg.cn/direct/fe94e8caaa8a424f922ba9aba5048ab4.jpeg)






Minikube 是一个轻量级的 Kubernetes 实现，适用于本地开发和测试。本文将介绍 `minikube start` 的常见参数，并探讨不同场景下的启动方式。

## 1. 基本启动
最简单的方式是直接运行：
```bash
minikube start
```
这将使用默认的驱动（通常是 `docker` 或 `virtualbox`）启动一个 Kubernetes 集群。

## 2. 选择不同的驱动
Minikube 支持多种运行时环境，如 `docker`、`virtualbox`、`qemu`、`hyperkit` 等，可以通过 `--driver` 指定：
```bash
minikube start --driver=docker
```
如果你的 macOS 支持 `hyperkit`，可以使用：
```bash
minikube start --driver=hyperkit
```

## 3. 指定 Kubernetes 版本
有时需要测试特定版本的 Kubernetes，可以使用 `--kubernetes-version` 指定：
```bash
minikube start --kubernetes-version=v1.28.0
```

## 4. 控制 CPU 和内存资源
可以调整 CPU 和内存大小，以优化 Minikube 运行时的性能：
```bash
minikube start --cpus=4 --memory=8192
```
这将分配 4 核 CPU 和 8GB 内存。

## 5. 指定网络子网
如果你的环境对网络有特殊要求，可以手动指定 CIDR：
```bash
minikube start --extra-config=kubeadm.pod-network-cidr=192.168.1.0/16
```

## 6. 使用特定的容器运行时
Kubernetes 支持 Docker、containerd、CRI-O 等不同的容器运行时，Minikube 允许你选择：
```bash
minikube start --container-runtime=containerd
```

## 7. 开启 Ingress 控制器
在本地测试 Ingress 需要启用相应的插件：
```bash
minikube start --addons=ingress
```

## 8. 启动时加载私有镜像
如果你有私有镜像，可以在启动时加载：
```bash
minikube start --insecure-registry=harbor.bsgchina.com
```

## 9. 离线启动（无外网环境）
在无外网的环境下，可以使用本地缓存：
```bash
minikube start --cache-images
```

## 10. 多节点集群（实验特性）
如果需要模拟多节点 Kubernetes，可以使用 `--nodes` 参数：
```bash
minikube start --nodes=3
```

## 11. 综合
`minikube start` 提供了丰富的参数，可以满足不同的开发和测试需求。你可以根据实际情况调整参数，以优化 Minikube 的使用体验。



```bash
minikube start --nodes 4 --driver=none --kubernetes-version=v1.18.1
minikube start --nodes  4 --network-plugin=cni --cni=calico --kubernetes-version=v1.18.1
minikube start --nodes 4  --network-plugin=cni --cni=calico --kubernetes-version=v1.29.7 --memory=no-limit --cpus=no-limit
```


## 12. 启动过程

```bash
minikube start --nodes  4 --network-plugin=cni --cni=calico --kubernetes-version=v1.18.1 --driver=none
😄  minikube v1.18.1 on Rocky 8.10
✨  Using the none driver based on user configuration
❗  With --network-plugin=cni, you will need to provide your own CNI. See --cni flag as a user-friendly alternative
👍  Starting control plane node minikube in cluster minikube
🤹  Running on localhost (CPUs=4, Memory=7696MB, Disk=100722MB) ...
🎉  minikube 1.33.1 is available! Download it: https://github.com/kubernetes/minikube/releases/tag/v1.33.1
💡  To disable this notice, run: 'minikube config set WantUpdateNotification false'

ℹ️  OS release is Rocky Linux 8.10 (Green Obsidian)
🌐  Found network options:
    ▪ HTTP_PROXY=http://192.168.21.101:7890
    ▪ HTTPS_PROXY=http://192.168.21.101:7890
    ▪ NO_PROXY=proxyhost,localhost,*.vsphere.local,*.vm.demo,*.tanzu.demo,192.168.21.101,127.0.0.1/8,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16
    ▪ http_proxy=http://192.168.21.101:7890
    ▪ https_proxy=http://192.168.21.101:7890
    ▪ no_proxy=proxyhost,localhost,*.vsphere.local,*.vm.demo,*.tanzu.demo,192.168.21.101,127.0.0.1/8,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16
🐳  Preparing Kubernetes v1.18.1 on Docker 27.1.2 ...
    ▪ env HTTP_PROXY=http://192.168.21.101:7890
    ▪ env HTTPS_PROXY=http://192.168.21.101:7890
    ▪ env NO_PROXY=proxyhost,localhost,*.vsphere.local,*.vm.demo,*.tanzu.demo,192.168.21.101,127.0.0.1/8,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16
    > kubeadm.sha256: 65 B / 65 B [--------------------------] 100.00% ? p/s 0s
    > kubectl.sha256: 65 B / 65 B [--------------------------] 100.00% ? p/s 0s
    > kubelet.sha256: 65 B / 65 B [--------------------------] 100.00% ? p/s 0s
    > kubeadm: 37.97 MiB / 37.97 MiB [---------------] 100.00% 11.87 MiB p/s 4s
    > kubectl: 41.99 MiB / 41.99 MiB [----------------] 100.00% 5.32 MiB p/s 8s
    > kubelet: 108.02 MiB / 108.02 MiB [------------] 100.00% 11.54 MiB p/s 10s
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
💢  initialization failed, will try again: wait: /bin/bash -c "sudo env PATH=/var/lib/minikube/binaries/v1.18.1:$PATH kubeadm init --config /var/tmp/minikube/kubeadm.yaml  --ignore-preflight-errors=DirAvailable--etc-kubernetes-manifests,DirAvailable--var-lib-minikube,DirAvailable--var-lib-minikube-etcd,FileAvailable--etc-kubernetes-manifests-kube-scheduler.yaml,FileAvailable--etc-kubernetes-manifests-kube-apiserver.yaml,FileAvailable--etc-kubernetes-manifests-kube-controller-manager.yaml,FileAvailable--etc-kubernetes-manifests-etcd.yaml,Port-10250,Swap": exit status 1
stdout:
[init] Using Kubernetes version: v1.18.1
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[certs] Using certificateDir folder "/var/lib/minikube/certs"
[certs] Using existing ca certificate authority
[certs] Using existing apiserver certificate and key on disk
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [minikube01 localhost] and IPs [192.168.23.100 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [minikube01 localhost] and IPs [192.168.23.100 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[kubelet-check] Initial timeout of 40s passed.
[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get http://localhost:10248/healthz: dial tcp [::1]:10248: connect: connection refused.

        Unfortunately, an error has occurred:
                timed out waiting for the condition

        This error is likely caused by:
                - The kubelet is not running
                - The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)

        If you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:
                - 'systemctl status kubelet'
                - 'journalctl -xeu kubelet'

        Additionally, a control plane component may have crashed or exited when started by the container runtime.
        To troubleshoot, list all containers using your preferred container runtimes CLI.

        Here is one example how you may list all Kubernetes containers running in docker:
                - 'docker ps -a | grep kube | grep -v pause'
                Once you have found the failing container, you can inspect its logs with:
                - 'docker logs CONTAINERID'


stderr:
W0905 19:06:12.107982    2906 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
        [WARNING FileExisting-socat]: socat not found in system path
        [WARNING FileExisting-tc]: tc not found in system path
        [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 27.1.2. Latest validated version: 19.03
        [WARNING Hostname]: hostname "minikube01" could not be reached
        [WARNING Hostname]: hostname "minikube01": lookup minikube01 on 192.168.21.2:53: no such host
        [WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'
W0905 19:07:24.006219    2906 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
W0905 19:07:24.007939    2906 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
error execution phase wait-control-plane: couldn't initialize a Kubernetes cluster
To see the stack trace of this error execute with --v=5 or higher

    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...\ 


💣  Error starting cluster: wait: /bin/bash -c "sudo env PATH=/var/lib/minikube/binaries/v1.18.1:$PATH kubeadm init --config /var/tmp/minikube/kubeadm.yaml  --ignore-preflight-errors=DirAvailable--etc-kubernetes-manifests,DirAvailable--var-lib-minikube,DirAvailable--var-lib-minikube-etcd,FileAvailable--etc-kubernetes-manifests-kube-scheduler.yaml,FileAvailable--etc-kubernetes-manifests-kube-apiserver.yaml,FileAvailable--etc-kubernetes-manifests-kube-controller-manager.yaml,FileAvailable--etc-kubernetes-manifests-etcd.yaml,Port-10250,Swap": exit status 1
stdout:
[init] Using Kubernetes version: v1.18.1
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[certs] Using certificateDir folder "/var/lib/minikube/certs"
[certs] Using existing ca certificate authority
[certs] Using existing apiserver certificate and key on disk
[certs] Using existing apiserver-kubelet-client certificate and key on disk
[certs] Using existing front-proxy-ca certificate authority
[certs] Using existing front-proxy-client certificate and key on disk
[certs] Using existing etcd/ca certificate authority
[certs] Using existing etcd/server certificate and key on disk
[certs] Using existing etcd/peer certificate and key on disk
[certs] Using existing etcd/healthcheck-client certificate and key on disk
[certs] Using existing apiserver-etcd-client certificate and key on disk
[certs] Using the existing "sa" key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[kubelet-check] Initial timeout of 40s passed.
[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get http://localhost:10248/healthz: dial tcp [::1]:10248: connect: connection refused.

        Unfortunately, an error has occurred:
                timed out waiting for the condition

        This error is likely caused by:
                - The kubelet is not running
                - The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)

        If you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:
                - 'systemctl status kubelet'
                - 'journalctl -xeu kubelet'

        Additionally, a control plane component may have crashed or exited when started by the container runtime.
        To troubleshoot, list all containers using your preferred container runtimes CLI.

        Here is one example how you may list all Kubernetes containers running in docker:
                - 'docker ps -a | grep kube | grep -v pause'
                Once you have found the failing container, you can inspect its logs with:
                - 'docker logs CONTAINERID'


stderr:
W0905 19:11:25.836920    9090 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
        [WARNING FileExisting-socat]: socat not found in system path
        [WARNING FileExisting-tc]: tc not found in system path
        [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 27.1.2. Latest validated version: 19.03
        [WARNING Hostname]: hostname "minikube01" could not be reached
        [WARNING Hostname]: hostname "minikube01": lookup minikube01 on 192.168.21.2:53: no such host
        [WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'
W0905 19:11:30.294202    9090 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
W0905 19:11:30.295825    9090 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
error execution phase wait-control-plane: couldn't initialize a Kubernetes cluster
To see the stack trace of this error execute with --v=5 or higher


😿  minikube is exiting due to an error. If the above message is not useful, open an issue:
👉  https://github.com/kubernetes/minikube/issues/new/choose
❌  Problems detected in kubelet:
    Sep 05 19:15:08 minikube01 kubelet[13745]: E0905 19:15:08.468451   13745 pod_workers.go:191] Error syncing pod 0d306474b5b045c4db0a998757c75e0e ("kube-apiserver-minikube01_kube-system(0d306474b5b045c4db0a998757c75e0e)"), skipping: failed to "StartContainer" for "kube-apiserver" with ImageInspectError: "Failed to inspect image \"k8s.gcr.io/kube-apiserver:v1.18.1\": Id or size of image \"k8s.gcr.io/kube-apiserver:v1.18.1\" is not set"
    Sep 05 19:15:09 minikube01 kubelet[13745]: E0905 19:15:09.479448   13745 pod_workers.go:191] Error syncing pod ac26964b28f6b79686b6f01b7255f0a8 ("kube-controller-manager-minikube01_kube-system(ac26964b28f6b79686b6f01b7255f0a8)"), skipping: failed to "StartContainer" for "kube-controller-manager" with ImageInspectError: "Failed to inspect image \"k8s.gcr.io/kube-controller-manager:v1.18.1\": Id or size of image \"k8s.gcr.io/kube-controller-manager:v1.18.1\" is not set"
    Sep 05 19:15:10 minikube01 kubelet[13745]: E0905 19:15:10.473657   13745 pod_workers.go:191] Error syncing pod bfe4eda6eff6c0a6b5d48dd60012b5ad ("etcd-minikube01_kube-system(bfe4eda6eff6c0a6b5d48dd60012b5ad)"), skipping: failed to "StartContainer" for "etcd" with ImageInspectError: "Failed to inspect image \"k8s.gcr.io/etcd:3.4.3-0\": Id or size of image \"k8s.gcr.io/etcd:3.4.3-0\" is not set"

❌  Exiting due to K8S_KUBELET_NOT_RUNNING: wait: /bin/bash -c "sudo env PATH=/var/lib/minikube/binaries/v1.18.1:$PATH kubeadm init --config /var/tmp/minikube/kubeadm.yaml  --ignore-preflight-errors=DirAvailable--etc-kubernetes-manifests,DirAvailable--var-lib-minikube,DirAvailable--var-lib-minikube-etcd,FileAvailable--etc-kubernetes-manifests-kube-scheduler.yaml,FileAvailable--etc-kubernetes-manifests-kube-apiserver.yaml,FileAvailable--etc-kubernetes-manifests-kube-controller-manager.yaml,FileAvailable--etc-kubernetes-manifests-etcd.yaml,Port-10250,Swap": exit status 1
stdout:
[init] Using Kubernetes version: v1.18.1
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[certs] Using certificateDir folder "/var/lib/minikube/certs"
[certs] Using existing ca certificate authority
[certs] Using existing apiserver certificate and key on disk
[certs] Using existing apiserver-kubelet-client certificate and key on disk
[certs] Using existing front-proxy-ca certificate authority
[certs] Using existing front-proxy-client certificate and key on disk
[certs] Using existing etcd/ca certificate authority
[certs] Using existing etcd/server certificate and key on disk
[certs] Using existing etcd/peer certificate and key on disk
[certs] Using existing etcd/healthcheck-client certificate and key on disk
[certs] Using existing apiserver-etcd-client certificate and key on disk
[certs] Using the existing "sa" key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[kubelet-check] Initial timeout of 40s passed.
[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get http://localhost:10248/healthz: dial tcp [::1]:10248: connect: connection refused.

        Unfortunately, an error has occurred:
                timed out waiting for the condition

        This error is likely caused by:
                - The kubelet is not running
                - The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)

        If you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:
                - 'systemctl status kubelet'
                - 'journalctl -xeu kubelet'

        Additionally, a control plane component may have crashed or exited when started by the container runtime.
        To troubleshoot, list all containers using your preferred container runtimes CLI.

        Here is one example how you may list all Kubernetes containers running in docker:
                - 'docker ps -a | grep kube | grep -v pause'
                Once you have found the failing container, you can inspect its logs with:
                - 'docker logs CONTAINERID'


stderr:
W0905 19:11:25.836920    9090 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
        [WARNING FileExisting-socat]: socat not found in system path
        [WARNING FileExisting-tc]: tc not found in system path
        [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 27.1.2. Latest validated version: 19.03
        [WARNING Hostname]: hostname "minikube01" could not be reached
        [WARNING Hostname]: hostname "minikube01": lookup minikube01 on 192.168.21.2:53: no such host
        [WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'
W0905 19:11:30.294202    9090 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
W0905 19:11:30.295825    9090 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
error execution phase wait-control-plane: couldn't initialize a Kubernetes cluster
To see the stack trace of this error execute with --v=5 or higher

💡  Suggestion: Check output of 'journalctl -xeu kubelet', try passing --extra-config=kubelet.cgroup-driver=systemd to minikube start
🍿  Related issue: https://github.com/kubernetes/minikube/issues/4172

```

