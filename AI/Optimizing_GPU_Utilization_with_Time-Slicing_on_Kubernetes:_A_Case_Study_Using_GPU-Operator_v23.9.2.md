


![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/e6dd226816454a2d984f4011702c9637.png)



## 1. 预备
检查内存、磁盘、cpu、gpu

```bash
$ lspci | grep -i nvidia
0002:00:00.0 3D controller: NVIDIA Corporation GA102GL [A10] (rev a1)


$ hostnamectl
 Static hostname: NV36ads-A10-v5
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: d414c77c25524bcabb978a4462068122
         Boot ID: f26b1027262e4667b23a325e5275405d
  Virtualization: microsoft
Operating System: Ubuntu 24.04.1 LTS
          Kernel: Linux 6.8.0-1021-azure
    Architecture: x86-64
 Hardware Vendor: Microsoft Corporation
  Hardware Model: Virtual Machine
Firmware Version: Hyper-V UEFI Release v4.1
   Firmware Date: Fri 2024-03-08
    Firmware Age: 11month 1w 1d


$ free -h
               total        used        free      shared  buff/cache   available
Mem:           432Gi       2.3Gi       419Gi        34Mi        10Gi       428Gi
Swap:             0B          0B          0B
root@NV36ads-A10-v5:~# df -Th
Filesystem     Type      Size  Used Avail Use% Mounted on
/dev/root      ext4      248G   16G  233G   7% /
tmpfs          tmpfs     217G     0  217G   0% /dev/shm
tmpfs          tmpfs      87G  1.6M   87G   1% /run
tmpfs          tmpfs     5.0M     0  5.0M   0% /run/lock
efivarfs       efivarfs  128K   37K   87K  30% /sys/firmware/efi/efivars
/dev/sda15     vfat      105M  6.1M   99M   6% /boot/efi
/dev/sdb1      ext4      1.4T   32K  1.4T   1% /mnt
tmpfs          tmpfs      44G   76K   44G   1% /run/user/132
overlay        overlay   248G   16G  233G   7% /var/lib/docker/overlay2/f78926ace83f9904f1426e39a57658f81551f4ed780fa204487147f50eccccde/merged
tmpfs          tmpfs      44G   64K   44G   1% /run/user/1000


$ cat /proc/cpuinfo| grep "processor"| wc -l
36
```

## 2. 兼容性

### 2.1 GPU operator 兼容 kubernetes 版本
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/0aa2684933054ad0a0eca9234e80dd73.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a637ae7ae35646a8aceac6ecd0b5b616.png)

### 2.2 GPU operator 兼容 containerd 版本
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/35c948bdce0a49d28a2554f9d7fd8bab.png)

### 2.3 GPU operator 兼容 Nvidia Driver 版本

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/90be3a5783f641f69e2e977f37ea4c9f.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/546cd413c9bd43099260f48e7cb72d08.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/9530b32297fd464fa6f6015534b6597d.png)




https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/23.6.2/platform-support.html
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/37ef8196671c4f2ea1ddcfb36cdfc05e.png)


### 2.4 Nvidia driver 535.154.05 兼容系统

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/ecb6f3ab6405482b822cfa5de8263491.png)

## 3. 安装kubernetes


### 3.1 配置 containerd
```bash
$ cat install-containerd-k8s-v1.31.4.sh
#!/bin/bash


name=`basename $0 .sh`
ENABLE_DOWNLOAD=${ENABLE_DOWNLOAD:-true}
BASE_DIR="$( dirname "$( readlink -f "${0}" )" )"

if [ ! -e files ]; then
    mkdir -p files
fi

FILES_DIR=./files
IMAGES_DIR=./images

# download files, if not found
download() {
    url=$1
    dir=$2

    filename=$(basename $1)
    mkdir -p ${FILES_DIR}/$dir

    if [ ! -e ${FILES_DIR}/$dir/$filename ]; then
        echo "==> download $url"
        (cd ${FILES_DIR}/$dir && curl -SLO $1)
    fi
}

download_files() {

if $ENABLE_DOWNLOAD; then
    # TODO: These version must be same as kubespray. Refer `roles/downloads/defaults/main.yml` of kubespray.
    RUNC_VERSION=1.2.3
    CONTAINERD_VERSION=1.7.24
    NERDCTL_VERSION=1.7.7
    CRICTL_VERSION=1.31.1
    CNI_VERSION=1.4.0

    download https://github.com/opencontainers/runc/releases/download/v${RUNC_VERSION}/runc.amd64 runc/v${RUNC_VERSION}
    download https://github.com/containerd/containerd/releases/download/v${CONTAINERD_VERSION}/containerd-${CONTAINERD_VERSION}-linux-amd64.tar.gz
    download https://github.com/containerd/nerdctl/releases/download/v${NERDCTL_VERSION}/nerdctl-${NERDCTL_VERSION}-linux-amd64.tar.gz
    download https://github.com/kubernetes-sigs/cri-tools/releases/download/v${CRICTL_VERSION}/crictl-v${CRICTL_VERSION}-linux-amd64.tar.gz
    download https://github.com/containernetworking/plugins/releases/download/v${CNI_VERSION}/cni-plugins-linux-amd64-v${CNI_VERSION}.tgz kubernetes/cni

else
    FILES_DIR=./files
fi

}

select_latest() {
    local latest=$(ls $* | tail -1)
    if [ -z "$latest" ]; then
        echo "No such file: $*"
        exit 1
    fi
    echo $latest
}

install_runc() {

# Install runc
echo "==> Install runc"
sudo cp $(select_latest "${FILES_DIR}/runc/v*/runc.amd64") /usr/local/bin/runc
sudo chmod 755 /usr/local/bin/runc

}



install_nerdctl() {
# Install nerdctl
echo "==> Install nerdctl"
tar xvf $(select_latest "${FILES_DIR}/nerdctl-*-linux-amd64.tar.gz") -C /tmp
sudo cp /tmp/nerdctl /usr/local/bin

}

install_crictl () {
# Install crictl plugins
echo "==> Install crictl plugins"
sudo tar xvzf $(select_latest "${FILES_DIR}/crictl-v*-linux-amd64.tar.gz") -C /usr/local/bin

cat > /etc/crictl.yaml <<EOF
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
EOF

}

install_containerd() {
# Install containerd
echo "==> Install containerd"


echo ""
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
systemctl restart systemd-modules-load.service
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
sysctl --system

sudo tar xvf $(select_latest "${FILES_DIR}/containerd-*-linux-amd64.tar.gz") --strip-components=1 -C /usr/local/bin

cat > /etc/systemd/system/containerd.service <<EOF
# Copyright The containerd Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target local-fs.target

[Service]
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/local/bin/containerd

Type=notify
Delegate=yes
KillMode=process
Restart=always
RestartSec=5
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNPROC=infinity
LimitCORE=infinity
LimitNOFILE=infinity
# Comment TasksMax if your systemd version does not supports it.
# Only systemd 226 and above support this version.
TasksMax=infinity
OOMScoreAdjust=-999

[Install]
WantedBy=multi-user.target
EOF

sudo mkdir -p \
     /etc/systemd/system/containerd.service.d \
     /etc/containerd \
     /var/lib/containerd \
     /run/containerd



containerd config default | tee /etc/containerd/config.toml
sed -i "s#SystemdCgroup\ \=\ false#SystemdCgroup\ \=\ true#g" /etc/containerd/config.toml
cat /etc/containerd/config.toml | grep SystemdCgroup


echo "==> Start containerd"
sudo systemctl daemon-reload && sudo systemctl enable --now containerd && sudo systemctl restart containerd && sudo systemctl status containerd | grep Active
}

install_cni() {
# Install cni plugins
echo "==> Install CNI plugins"
sudo mkdir -p /opt/cni/bin
sudo tar xvzf $(select_latest "${FILES_DIR}/kubernetes/cni/cni-plugins-linux-amd64-v*.tgz") -C /opt/cni/bin

}

action=$1

case $action in
  d )
    download_files
    ;;
  i|install)
    install_nerdctl
    install_crictl
    install_runc
    install_containerd
    install_cni
    ;;
   *)
    echo "Usage: $name [d|i]"
    echo "sh $name d: it is download packages."
    echo "sh$name i: it is install packages."
    ;;
esac
exit 0

```

下载软件

```bash
$ sh install-containerd-k8s-v1.31.4.sh d
```
安装软件

```bash
$ sh install-containerd-k8s-v1.31.4.sh i
```

查看containerd状态
```bash
$ systemctl status containerd.service
```
查看版本

```bash
$ nerdctl --version
nerdctl version 1.7.7
$ crictl --version
crictl version v1.31.1
$ runc --version
runc version 1.2.3
commit: v1.2.3-0-g0d37cfd4
spec: 1.2.0
go: go1.22.10
libseccomp: 2.5.5

```

### 3.2 kubeadm 安装集群

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet
$ kubeadm init --kubernetes-version=v1.31.6 --pod-network-cidr=10.96.0.0/12 --apiserver-advertise-address=10.0.0.4

```
输出：

```bash
[init] Using Kubernetes version: v1.31.6
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action beforehand using 'kubeadm config images pull'
W0214 09:09:14.666387    8694 checks.go:846] detected that the sandbox image "registry.k8s.io/pause:3.8" of the container runtime is inconsistent with that used by kubeadm.It is recommended to use "registry.k8s.io/pause:3.10" as the CRI sandbox image.
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local nv36ads-a10-v5] and IPs [10.96.0.1 10.0.0.4]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [localhost nv36ads-a10-v5] and IPs [10.0.0.4 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [localhost nv36ads-a10-v5] and IPs [10.0.0.4 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "super-admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests"
[kubelet-check] Waiting for a healthy kubelet at http://127.0.0.1:10248/healthz. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 501.597634ms
[api-check] Waiting for a healthy API server. This can take up to 4m0s
[api-check] The API server is healthy after 5.50153259s
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node nv36ads-a10-v5 as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node nv36ads-a10-v5 as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: a0zkpu.xfs9ol13krvu9txl
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.0.0.4:6443 --token a0zkpu.xfs9ol13krvu9txl \
        --discovery-token-ca-cert-hash sha256:528202008db60666c63ba551fbaef39b986b69a820e4a669eda97578f4a309ad
root@NV36ads-A10-v5:~#

```
配置kubeconfig

```bash
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config

```

查看集群状态

```bash
$ kubectl get node
NAME             STATUS     ROLES           AGE   VERSION
nv36ads-a10-v5   NotReady   control-plane   70m   v1.31.6

$ kubectl get pod -A
NAMESPACE         NAME                                     READY   STATUS    RESTARTS      AGE
kube-system       coredns-7c65d6cfc9-7tv88                 0/1     Pending   0             74m
kube-system       coredns-7c65d6cfc9-z9kfl                 0/1     Pending   0             65m
kube-system       etcd-nv36ads-a10-v5                      1/1     Running   1 (29m ago)   74m
kube-system       kube-apiserver-nv36ads-a10-v5            1/1     Running   1 (29m ago)   74m
kube-system       kube-controller-manager-nv36ads-a10-v5   1/1     Running   1 (29m ago)   74m
kube-system       kube-proxy-2fzsh                         1/1     Running   1 (29m ago)   74m
kube-system       kube-scheduler-nv36ads-a10-v5            1/1     Running   1 (29m ago)   74m
tigera-operator   tigera-operator-64ff5465b7-bpqlm         1/1     Running   0             7s

```
### 3.3 安装网络calico插件

```bash
$ kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```


```bash
$ kubectl get node
NAME             STATUS   ROLES           AGE   VERSION
nv36ads-a10-v5   Ready    control-plane   87m   v1.31.6
$ kubectl get pod -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS      AGE
kube-system   calico-kube-controllers-6879d4fcdc-wx59j   1/1     Running   0             70s
kube-system   calico-node-rqgcq                          1/1     Running   0             70s
kube-system   coredns-7c65d6cfc9-7tv88                   1/1     Running   0             87m
kube-system   coredns-7c65d6cfc9-z9kfl                   1/1     Running   0             79m
kube-system   etcd-nv36ads-a10-v5                        1/1     Running   1 (42m ago)   88m
kube-system   kube-apiserver-nv36ads-a10-v5              1/1     Running   1 (42m ago)   88m
kube-system   kube-controller-manager-nv36ads-a10-v5     1/1     Running   1 (42m ago)   88m
kube-system   kube-proxy-2fzsh                           1/1     Running   1 (42m ago)   87m
kube-system   kube-scheduler-nv36ads-a10-v5              1/1     Running   1 (42m ago)   88m

```
## 4. 安装驱动

### 4.1 下载 nvidia 驱动

参考：[https://raw.githubusercontent.com/Azure/azhpc-extensions/refs/heads/master/NvidiaGPU/Nvidia-GPU-Linux-Resources.json](https://raw.githubusercontent.com/Azure/azhpc-extensions/refs/heads/master/NvidiaGPU/Nvidia-GPU-Linux-Resources.json)

确认nvidira 驱动兼容版本

```bash
$ ubuntu-drivers devices
== /sys/devices/LNXSYSTM:00/LNXSYBUS:00/ACPI0004:00/VMBUS:00/56475055-0002-0000-3130-444532323336/pci0002:00/0002:00:00.0 ==
modalias : pci:v000010DEd00002236sv000010DEsd000014BFbc03sc02i00
vendor   : NVIDIA Corporation
model    : GA102GL [A10]
manual_install: True
driver   : nvidia-driver-535-open - distro non-free
driver   : nvidia-driver-565 - third-party non-free
driver   : nvidia-driver-470 - distro non-free
driver   : nvidia-driver-555 - third-party non-free
driver   : nvidia-driver-535-server-open - distro non-free
driver   : nvidia-driver-560-open - third-party non-free
driver   : nvidia-driver-570-open - third-party non-free
driver   : nvidia-driver-555-open - third-party non-free
driver   : nvidia-driver-545-open - distro non-free
driver   : nvidia-driver-550 - third-party non-free
driver   : nvidia-driver-550-open - third-party non-free
driver   : nvidia-driver-565-open - third-party non-free
driver   : nvidia-driver-470-server - distro non-free
driver   : nvidia-driver-535-server - distro non-free
driver   : nvidia-driver-570 - third-party non-free recommended
driver   : nvidia-driver-560 - third-party non-free
driver   : xserver-xorg-video-nouveau - distro free builtin

```

下载驱动版本：[https://download.microsoft.com/download/8/d/a/8da4fb8e-3a9b-4e6a-bc9a-72ff64d7a13c/NVIDIA-Linux-x86_64-535.161.08-grid-azure.run](https://download.microsoft.com/download/8/d/a/8da4fb8e-3a9b-4e6a-bc9a-72ff64d7a13c/NVIDIA-Linux-x86_64-535.161.08-grid-azure.run)


### 4.2 安装驱动

```bash
$ sh NVIDIA-Linux-x86_64-535.161.08-grid-azure.run
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/4d0c22fca52f4ed098935f98507085d0.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b0a3f65476994991a0fc13b0879e1f19.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/8042b119ddeb46a8a0383671f3396973.png)




###  4.3 安装cuda 12.2.2

参考：[https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#system-requirements](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#system-requirements)

下载版本：`cuda_12.2.2_535.104.05_linux.run`

```bash
$ sh cuda_12.2.2_535.104.05_linux.run
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/4b631dea0a80477092d8b683d4594dc6.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/50e2f085ddad41e3be6b298aa99e02ff.png)

> 注意：只选CUDA Toolkit 12.2,图错误


```bash
$ sh cuda_12.2.2_535.104.05_linux.run
===========
= Summary =
===========

Driver:   Not Selected
Toolkit:  Installed in /usr/local/cuda-12.2/

Please make sure that
 -   PATH includes /usr/local/cuda-12.2/bin
 -   LD_LIBRARY_PATH includes /usr/local/cuda-12.2/lib64, or, add /usr/local/cuda-12.2/lib64 to /etc/ld.so.conf and run ldconfig as root

To uninstall the CUDA Toolkit, run cuda-uninstaller in /usr/local/cuda-12.2/bin
***WARNING: Incomplete installation! This installation did not install the CUDA Driver. A driver of version at least 535.00 is required for CUDA 12.2 functionality to work.
To install the driver using this installer, run the following command, replacing <CudaInstaller> with the name of this run file:
    sudo <CudaInstaller>.run --silent --driver

Logfile is /var/log/cuda-installer.log

```

配置cuda命令路径
```bash
$ vim /root/.bashrc
export PATH=/usr/local/cuda-12.2/bin:$PATH
$ source /root/.bashrc
```


检查 gpu 能否被 nvidia-smi 识别
```bash
$  nvidia-smi
Fri Feb 14 11:32:22 2025
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 535.161.08             Driver Version: 535.161.08   CUDA Version: 12.2     |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|                                         |                      |               MIG M. |
|=========================================+======================+======================|
|   0  NVIDIA A10-24Q                 On  | 00000002:00:00.0 Off |                    0 |
| N/A   N/A    P8              N/A /  N/A |    189MiB / 24512MiB |      0%      Default |
|                                         |                      |             Disabled |
+-----------------------------------------+----------------------+----------------------+

+---------------------------------------------------------------------------------------+
| Processes:                                                                            |
|  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
|        ID   ID                                                             Usage      |
|=======================================================================================|
|    0   N/A  N/A      3516      G   /usr/lib/xorg/Xorg                          162MiB |
|    0   N/A  N/A      3655      G   /usr/bin/gnome-shell                         25MiB |
+---------------------------------------------------------------------------------------+


root@NV36ads-A10-v5:~# sudo systemctl stop gdm
root@NV36ads-A10-v5:~# nvidia-smi
Fri Feb 14 11:58:28 2025
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 535.161.08             Driver Version: 535.161.08   CUDA Version: 12.2     |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|                                         |                      |               MIG M. |
|=========================================+======================+======================|
|   0  NVIDIA A10-24Q                 On  | 00000002:00:00.0 Off |                    0 |
| N/A   N/A    P8              N/A /  N/A |      0MiB / 24512MiB |      0%      Default |
|                                         |                      |             Disabled |
+-----------------------------------------+----------------------+----------------------+

+---------------------------------------------------------------------------------------+
| Processes:                                                                            |
|  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
|        ID   ID                                                             Usage      |
|=======================================================================================|
|  No running processes found                                                           |
+---------------------------------------------------------------------------------------+

```


## 5. 安装 helm
```bash
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 \
    && chmod 700 get_helm.sh \
    && ./get_helm.sh
```

检查版本

```bash
$ helm version
version.BuildInfo{Version:"v3.17.1", GitCommit:"980d8ac1939e39138101364400756af2bdee1da5", GitTreeState:"clean", GoVersion:"go1.23.5"}

```

## 6. 安装 NVIDIA Container Toolkit 1.14.6

参考：[https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a178b9d4f16d4aec9b5d15225f036076.png)


```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sed -i -e '/experimental/ s/^#//g' /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt-get update
sudo apt-get install nvidia-container-toolkit-base=1.14.6-1
sudo apt-get install nvidia-container-toolkit=1.14.6-1

```
检查命令版本

```bash
$ nvidia-ctk --version
NVIDIA Container Toolkit CLI version 1.14.6
commit: 5605d191332dcfeea802c4497360d60a65c7887e

```

使用nvidia-ctk命令配置容器运行时：

```bash
sudo nvidia-ctk runtime configure --runtime=containerd
```

nvidia-ctk命令修改主机上的/etc/containerd/config.toml文件。该文件已更新，以便containerd可以使用NVIDIA Container SDK。
重新启动containerd：

```bash
sudo systemctl restart containerd
```


## 7. 安装 gpu operator

参考：[https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/getting-started.html](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/getting-started.html)

```bash
kubectl create ns gpu-operator
kubectl label --overwrite ns gpu-operator pod-security.kubernetes.io/enforce=privileged
```
Node Feature Discovery  (NFD) 是每个节点上 Operator 的依赖项。默认情况下，NFD 主节点和工作节点由 Operator 自动部署。如果 NFD 已在集群中运行，则在安装 Operator 时必须禁用部署 NFD。

确定 NFD 是否已在集群中运行的一种方法是检查节点上的 NFD 标签：

```bash
kubectl get nodes -o json | jq '.items[].metadata.labels | keys | any(startswith("feature.node.kubernetes.io"))'
```
如果命令输出为true，则NFD已在群集中运行。


```bash
helm repo add nvidia https://helm.ngc.nvidia.com/nvidia \
    && helm repo update
```

Install the GPU Operator.


```bash
helm upgrade gpu-operator \
     nvidia/gpu-operator  \
     -n gpu-operator \
     --create-namespace \
     --install \
     --version v23.9.2 \
     --set driver.enabled=false \
     --set toolkit.enabled=false \
    --set migManager.enabled=false 
```

输出

```bash
Release "gpu-operator" does not exist. Installing it now.
W0214 13:05:39.785095   71958 warnings.go:70] spec.template.spec.affinity.nodeAffinity.preferredDuringSchedulingIgnoredDuringExecution[0].preference.matchExpressions[0].key: node-role.kubernetes.io/master is use "node-role.kubernetes.io/control-plane" instead
W0214 13:05:39.788693   71958 warnings.go:70] spec.template.spec.affinity.nodeAffinity.preferredDuringSchedulingIgnoredDuringExecution[0].preference.matchExpressions[0].key: node-role.kubernetes.io/master is use "node-role.kubernetes.io/control-plane" instead
NAME: gpu-operator
LAST DEPLOYED: Fri Feb 14 13:05:39 2025
NAMESPACE: gpu-operator
STATUS: deployed
REVISION: 1
TEST SUITE: None

```

检查部署状态

```bash
$ kubectl get pod -A
NAMESPACE      NAME                                                          READY   STATUS      RESTARTS        AGE
gpu-operator   gpu-feature-discovery-xdrrl                                   1/1     Running     0               4m54s
gpu-operator   gpu-operator-5b44fdd599-jfnpg                                 1/1     Running     0               5m15s
gpu-operator   gpu-operator-node-feature-discovery-gc-555ccf7687-qgv4d       1/1     Running     0               5m15s
gpu-operator   gpu-operator-node-feature-discovery-master-68d694564d-28mzj   1/1     Running     0               5m15s
gpu-operator   gpu-operator-node-feature-discovery-worker-jwrxf              1/1     Running     0               5m15s
gpu-operator   nvidia-cuda-validator-njcgx                                   0/1     Completed   0               4m39s
gpu-operator   nvidia-dcgm-exporter-6cqct                                    1/1     Running     0               4m54s
gpu-operator   nvidia-device-plugin-daemonset-2kcfx                          1/1     Running     0               4m54s
gpu-operator   nvidia-operator-validator-f66l6                               1/1     Running     0               4m54s
kube-system    calico-kube-controllers-6879d4fcdc-wx59j                      1/1     Running     1 (100m ago)    154m
kube-system    calico-node-rqgcq                                             1/1     Running     1 (100m ago)    154m
kube-system    coredns-7c65d6cfc9-7tv88                                      1/1     Running     1 (100m ago)    4h
kube-system    coredns-7c65d6cfc9-z9kfl                                      1/1     Running     1 (100m ago)    3h52m
kube-system    etcd-nv36ads-a10-v5                                           1/1     Running     2 (100m ago)    4h1m
kube-system    kube-apiserver-nv36ads-a10-v5                                 1/1     Running     2 (100m ago)    4h1m
kube-system    kube-controller-manager-nv36ads-a10-v5                        1/1     Running     6 (3m26s ago)   4h1m
kube-system    kube-proxy-2fzsh                                              1/1     Running     2 (100m ago)    4h
kube-system    kube-scheduler-nv36ads-a10-v5                                 1/1     Running     6 (3m26s ago)   4h1m

```

## 8. 配置GPU time-slicing 切片

如果您已经安装了GPU运算符，并希望在群集中的所有节点上应用相同的时间分片配置，请执行以下步骤来配置GPU时间分片。

创建一个文件，如time-slicing-pull-all.yaml，其内容如下例所示：
```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: time-slicing-config-all
data:
  any: |-
    version: v1
    flags:
      migStrategy: none
    sharing:
      timeSlicing:
        resources:
        - name: nvidia.com/gpu
          replicas: 4
```
将 configmap 添加到与gpu-operator相同的命名空间：
```bash
kubectl create -n gpu-operator -f time-slicing-config-all.yaml
```
使用configmap配置设备插件，并设置默认的时间切片配置：

```bash
kubectl patch clusterpolicies.nvidia.com/cluster-policy \
    -n gpu-operator --type merge \
    -p '{"spec": {"devicePlugin": {"config": {"name": "time-slicing-config-all", "default": "any"}}}}'
```
可选：确认gpu-feature-discovery和nvidia-device-plugin-daemonset pod重新启动。

```bash
kubectl get events -n gpu-operator --sort-by='.lastTimestamp'
```
输出：

```bash
LAST SEEN   TYPE      REASON             OBJECT                                     MESSAGE
33s         Normal    Created            pod/nvidia-device-plugin-daemonset-cffds   Created container toolkit-validation
33s         Normal    Started            pod/nvidia-device-plugin-daemonset-cffds   Started container toolkit-validation
33s         Normal    Started            pod/gpu-feature-discovery-rvlg9            Started container toolkit-validation
33s         Normal    Created            pod/gpu-feature-discovery-rvlg9            Created container toolkit-validation
33s         Normal    Pulled             pod/gpu-feature-discovery-rvlg9            Container image "nvcr.io/nvidia/cloud-native/gpu-operator-validator:v22.9.1" already present on machine
33s         Normal    Pulled             pod/nvidia-device-plugin-daemonset-cffds   Container image "nvcr.io/nvidia/cloud-native/gpu-operator-validator:v22.9.1" already present on machine
32s         Normal    Created            pod/nvidia-device-plugin-daemonset-cffds   Created container config-manager-init
32s         Normal    Pulled             pod/nvidia-device-plugin-daemonset-cffds   Container image "nvcr.io/nvidia/k8s-device-plugin:v0.13.0-ubi8" already present on machine
32s         Normal    Pulled             pod/gpu-feature-discovery-rvlg9            Container image "nvcr.io/nvidia/k8s-device-plugin:v0.13.0-ubi8" already present on machine
32s         Normal    Created            pod/gpu-feature-discovery-rvlg9            Created container config-manager-init
32s         Normal    Started            pod/gpu-feature-discovery-rvlg9            Started container config-manager-init
32s         Normal    Started            pod/nvidia-device-plugin-daemonset-cffds   Started container config-manager-init
31s         Normal    Created            pod/gpu-feature-discovery-rvlg9            Created container config-manager
31s         Normal    Started            pod/gpu-feature-discovery-rvlg9            Started container gpu-feature-discovery
31s         Normal    Created            pod/gpu-feature-discovery-rvlg9            Created container gpu-feature-discovery
31s         Normal    Pulled             pod/gpu-feature-discovery-rvlg9            Container image "nvcr.io/nvidia/gpu-feature-discovery:v0.7.0-ubi8" already present on machine
31s         Normal    Started            pod/nvidia-device-plugin-daemonset-cffds   Started container config-manager
31s         Normal    Created            pod/nvidia-device-plugin-daemonset-cffds   Created container config-manager
31s         Normal    Pulled             pod/nvidia-device-plugin-daemonset-cffds   Container image "nvcr.io/nvidia/k8s-device-plugin:v0.13.0-ubi8" already present on machine
31s         Normal    Started            pod/nvidia-device-plugin-daemonset-cffds   Started container nvidia-device-plugin
31s         Normal    Created            pod/nvidia-device-plugin-daemonset-cffds   Created container nvidia-device-plugin
31s         Normal    Pulled             pod/nvidia-device-plugin-daemonset-cffds   Container image "nvcr.io/nvidia/k8s-device-plugin:v0.13.0-ubi8" already present on machine
31s         Normal    Pulled             pod/gpu-feature-discovery-rvlg9            Container image "nvcr.io/nvidia/k8s-device-plugin:v0.13.0-ubi8" already present on machine
31s         Normal    Started            pod/gpu-feature-discovery-rvlg9            Started container config-manager
```

## 9. 验证 GPU Time-Slicing 

执行以下步骤以验证时间分片配置是否已成功应用：
确认节点公告额外的GPU资源：

```bash

$ kubectl describe node nv36ads-a10-v5
```

示例输出根据节点中的GPU和应用的配置而有所不同。
当renameByDefault设置为false（默认值）时，将应用以下输出。主要考虑因素如下：

- nvidia.com/gpu.count：标签报告计算机中的物理GPU数量。
- nvidia.com/gpu.product：标签在产品名称后包含一个-SHARED后缀。
- nvidia.com/gpu.replicas：标签与报告的容量匹配。

```bash
                   node-role.kubernetes.io/control-plane=
                    node.kubernetes.io/exclude-from-external-load-balancers=
                    nvidia.com/cuda.driver.major=535
                    nvidia.com/cuda.driver.minor=161
                    nvidia.com/cuda.driver.rev=08
                    nvidia.com/cuda.runtime.major=12
                    nvidia.com/cuda.runtime.minor=2
                    nvidia.com/gfd.timestamp=1739538971
                    nvidia.com/gpu.compute.major=8
                    nvidia.com/gpu.compute.minor=6
                    nvidia.com/gpu.count=1
                    nvidia.com/gpu.deploy.container-toolkit=true
                    nvidia.com/gpu.deploy.dcgm=true
                    nvidia.com/gpu.deploy.dcgm-exporter=true
                    nvidia.com/gpu.deploy.device-plugin=true
                    nvidia.com/gpu.deploy.driver=true
                    nvidia.com/gpu.deploy.gpu-feature-discovery=true
                    nvidia.com/gpu.deploy.node-status-exporter=true
                    nvidia.com/gpu.deploy.operator-validator=true
                    nvidia.com/gpu.family=ampere
                    nvidia.com/gpu.machine=Virtual-Machine
                    nvidia.com/gpu.memory=24512
                    nvidia.com/gpu.present=true
                    nvidia.com/gpu.product=NVIDIA-A10-24Q-SHARED
                    nvidia.com/gpu.replicas=4
                    nvidia.com/mig.capable=true
                    nvidia.com/mig.strategy=single
                    nvidia.com/vgpu.host-driver-branch=r552_52
                    nvidia.com/vgpu.host-driver-version=553.07
                    nvidia.com/vgpu.present=true
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: unix:///var/run/containerd/containerd.sock
                    nfd.node.kubernetes.io/feature-labels:
                      cpu-cpuid.ADX,cpu-cpuid.AESNI,cpu-cpuid.AVX,cpu-cpuid.AVX2,cpu-cpuid.CETSS,cpu-cpuid.CLZERO,cpu-cpuid.CMPXCHG8,cpu-cpuid.FMA3,cpu-cpuid.FS...
                    nfd.node.kubernetes.io/master.version: v0.14.2
                    nfd.node.kubernetes.io/worker.version: v0.14.2
                    node.alpha.kubernetes.io/ttl: 0
                    nvidia.com/gpu-driver-upgrade-enabled: true
                    projectcalico.org/IPv4Address: 10.0.0.4/24
                    projectcalico.org/IPv4IPIPTunnelAddr: 10.97.77.0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Fri, 14 Feb 2025 09:09:46 +0000
Taints:             <none>
Unschedulable:      false
Lease:
  HolderIdentity:  nv36ads-a10-v5

```
当 renameByDefault 设置为 true 时，适用以下输出。主要注意事项如下：

- nvidia.com/gpu.count ：标签报告机器中的物理 GPU 数量。
- nvidia.com/gpu ：容量报告 0。
- nvidia.com/gpu.shared ：容量等于物理 GPU 数量乘以要创建的指定 GPU 副本数量

```bash
Capacity:
  cpu:                36
  ephemeral-storage:  128917488Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             454033328Ki
  nvidia.com/gpu:     4
  pods:               110
Allocatable:
  cpu:                36
  ephemeral-storage:  118810356745
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             453930928Ki
  nvidia.com/gpu:     4
  pods:               110

```

可选：部署工作负载以验证GPU时间分片：
创建一个文件，如time-slicing-verification.yaml，内容如下：

```bash
$ cat gpu3.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gpu1-deployment
spec:
  replicas: 2  # 设置副本数为 2
  selector:
    matchLabels:
      app: gpu1
  template:
    metadata:
      labels:
        app: gpu1
    spec:
      runtimeClassName: nvidia  # 使用 NVIDIA runtime，确保使用 GPU
      containers:
      - name: gpu-container
        image: tensorflow/tensorflow:2.12.0-gpu  # 使用 TensorFlow GPU 镜像
        resources:
          requests:
            memory: "20Gi"
            cpu: "10"
            nvidia.com/gpu: 1  # 请求 1 个 GPU
          limits:
            memory: "20Gi"
            cpu: "10"
            nvidia.com/gpu: 1  # 限制 1 个 GPU
        command:
          - "bash"
          - "-c"
          - |
            apt-get update && apt-get install -y python3 python3-pip && pip3 install tensorflow && \
            python3 -c '
            import tensorflow as tf
            print("Num GPUs Available: ", len(tf.config.experimental.list_physical_devices("GPU")))
            devices = tf.config.experimental.list_physical_devices("GPU")
            for device in devices:
                print(f"Device name: {device.name}")
            if len(devices) > 0:
                tf.config.experimental.set_memory_growth(devices[0], True)
            with tf.device("/GPU:0"):
                model = tf.keras.Sequential([
                    tf.keras.layers.Dense(64, activation="relu", input_shape=(32,)),
                    tf.keras.layers.Dense(32, activation="relu"),
                    tf.keras.layers.Dense(1)
                ])
                model.compile(optimizer="adam", loss="mse")

                import numpy as np
                x = np.random.random((10000, 32))  # 增加数据量
                y = np.random.random((10000, 1))

                model.fit(x, y, epochs=10, batch_size=32)
            ' && while true; do sleep 10; done
        stdin: true
        tty: true


```
使用多个复制副本创建部署：

```bash
$ kubectl apply -f time-slicing-verification.yaml
```


查看nvidia-smi

```bash
$ root@NV36ads-A10-v5:~/apps_gpu# nvidia-smi
Wed Feb 19 11:12:28 2025
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 535.161.08             Driver Version: 535.161.08   CUDA Version: 12.2     |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|                                         |                      |               MIG M. |
|=========================================+======================+======================|
|   0  NVIDIA A10-24Q                 On  | 00000002:00:00.0 Off |                    0 |
| N/A   N/A    P0              N/A /  N/A |    512MiB / 24512MiB |     50%      Default |
|                                         |                      |             Disabled |
+-----------------------------------------+----------------------+----------------------+

+---------------------------------------------------------------------------------------+
| Processes:                                                                            |
|  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
|        ID   ID                                                             Usage      |
|=======================================================================================|
|    0   N/A  N/A    292650      C   python3                                     255MiB |
|    0   N/A  N/A    292713      C   python3                                     255MiB |
+---------------------------------------------------------------------------------------+

```


## 卸载

### 卸载 gpu-operator

```bash
helm uninstal gpu-operator -n gpu-operator
```

### 卸载 cuda

```bash
sudo nvidia-uninstall

```

### 卸载 nvidia driver

```bash
$ sudo ./NVIDIA-Linux-x86_64-535.161.08-grid-azure.run --uninstall
Verifying archive integrity... OK
Uncompressing NVIDIA Accelerated Graphics Driver for Linux-x86_64 535.161.08........................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................
root@NV36ads-A10-v5:~#

```

```bash
sudo apt-get purge nvidia-vgpu-*
sudo apt-get autoremove
sudo apt-get purge nvidia-*
sudo apt-get autoremove
sudo rm -rf /etc/nvidia*
sudo rm -rf /var/lib/nvidia*
sudo rm -rf /run/nvidia*
```




参考：

- [https://v1-31.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/](https://v1-31.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
- [https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/platform-support.html](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/platform-support.html)
- cuda支持nvdia driver 列表：[https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html#](https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html#)
- cuda 12.2.2：[https://developer.nvidia.com/cuda-12-2-2-download-archive?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=22.04&target_type=runfile_local](https://developer.nvidia.com/cuda-12-2-2-download-archive?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=22.04&target_type=runfile_local)
- container-toolkit：[https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/release-notes.html#nvidia-container-toolkit-1-14-6](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/release-notes.html#nvidia-container-toolkit-1-14-6)
