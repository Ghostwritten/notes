
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/972a3d3b22a64180a8b6293c31ab9381.png)




## 1. 准备

检查硬件型号、主机信息内存、磁盘、cpu、gpu。
```bash
$ lspci | grep -i nvidia
0001:00:00.0 3D controller: NVIDIA Corporation TU104GL [Tesla T4] (rev a1)


root@ECS-koreacentral-T4:~# hostnamectl
 Static hostname: ECS-koreacentral-T4
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: c88bb0e23b5541e488ff6c6c5bb305ab
         Boot ID: 9992b6a929f94d86b3e83195008137ae
  Virtualization: microsoft
Operating System: Ubuntu 24.04.2 LTS
          Kernel: Linux 6.8.0-1021-azure
    Architecture: x86-64
 Hardware Vendor: Microsoft Corporation
  Hardware Model: Virtual Machine
Firmware Version: Hyper-V UEFI Release v4.1
   Firmware Date: Fri 2024-03-08
    Firmware Age: 11month 3w

root@ECS-koreacentral-T4:~# free
               total        used        free      shared  buff/cache   available
Mem:        57585648     1128332    56230412        4124      804752    56457316
Swap:              0           0           0
root@ECS-koreacentral-T4:~# df -Th
Filesystem     Type      Size  Used Avail Use% Mounted on
/dev/root      ext4      495G  1.9G  494G   1% /
tmpfs          tmpfs      28G     0   28G   0% /dev/shm
tmpfs          tmpfs      11G  1.1M   11G   1% /run
tmpfs          tmpfs     5.0M     0  5.0M   0% /run/lock
efivarfs       efivarfs  128M   26K  128M   1% /sys/firmware/efi/efivars
/dev/sda16     ext4      881M   59M  761M   8% /boot
/dev/sda15     vfat      105M  6.1M   99M   6% /boot/efi
/dev/sdb1      ext4      346G   32K  328G   1% /mnt
tmpfs          tmpfs     5.5G   12K  5.5G   1% /run/user/1000


```

## 2. 兼容

[https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/platform-support.html](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/platform-support.html)

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/7e13d18d7738458ab5ad15934d9a1b2e.png)


![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/89a61ff0651d4f7c81146bcdb91e9540.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/1587c19e4c054a949af61cfea80cad59.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/f9782735b2b948b9886dc9202b47cea7.png)



## 3. 安装 nvidia driver 550.144.03 

```bash
wget https://download.microsoft.com/download/c/3/4/c3484f19-fe76-4495-a65d-a5222ead9517/NVIDIA-Linux-x86_64-550.144.03-grid-azure.run
chmod 755 NVIDIA-Linux-x86_64-550.144.03-grid-azure.run
sh NVIDIA-Linux-x86_64-550.144.03-grid-azure.run
```

```bash
$ nvidia-smi
Thu Feb 27 06:00:49 2025
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 550.144.03             Driver Version: 550.144.03     CUDA Version: 12.4     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  Tesla T4                       On  |   00000001:00:00.0 Off |                  Off |
| N/A   31C    P8             14W /   70W |       1MiB /  16384MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|  No running processes found                                                             |
+-----------------------------------------------------------------------------------------+

```

## 4. 安装 cuda 12.4

下载：[https://developer.nvidia.com/cuda-12-4-0-download-archive?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=22.04&target_type=runfile_local](https://developer.nvidia.com/cuda-12-4-0-download-archive?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=22.04&target_type=runfile_local)

```bash
wget https://developer.download.nvidia.com/compute/cuda/12.4.0/local_installers/cuda_12.4.0_550.54.14_linux.run
sudo sh cuda_12.4.0_550.54.14_linux.run
```


配置cuda命令路径
```bash
$ vim /root/.bashrc
export PATH=/usr/local/cuda-12.2/bin:$PATH
$ source /root/.bashrc
```

## 5. 安装 kubernetes
### 5.1 配置 containerd
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
nerdctl --version
crictl --version
runc --version
```
输出

```bash
nerdctl version 1.7.7
crictl version v1.31.1
runc version 1.2.3
commit: v1.2.3-0-g0d37cfd4
spec: 1.2.0
go: go1.22.10
libseccomp: 2.5.5

```

### 5.2 kubeadm 安装集群

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
```
查询版本

```bash
root@ECS-koreacentral-T4:~# apt-cache policy kubelet
kubelet:
  Installed: (none)
  Candidate: 1.31.6-1.1
  Version table:
     1.31.6-1.1 500
        500 https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
     1.31.5-1.1 500
        500 https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
     1.31.4-1.1 500
        500 https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
     1.31.3-1.1 500
        500 https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
     1.31.2-1.1 500
        500 https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
     1.31.1-1.1 500
        500 https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
     1.31.0-1.1 500
        500 https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
root@ECS-koreacentral-T4:~# apt-cache policy kubeadm
kubeadm:
  Installed: (none)
  Candidate: 1.31.6-1.1
  Version table:
     1.31.6-1.1 500
        500 https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
     1.31.5-1.1 500
        500 https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
     1.31.4-1.1 500
        500 https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
     1.31.3-1.1 500
        500 https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
     1.31.2-1.1 500
        500 https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
     1.31.1-1.1 500
        500 https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
     1.31.0-1.1 500
        500 https://pkgs.k8s.io/core:/stable:/v1.31/deb  Packages
root@ECS-koreacentral-T4:~#

```

安装集群

```bash
sudo apt-get -y install kubelet=1.31.6-1.1 kubeadm=1.31.6-1.1 kubectl=1.31.6-1.1
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet
kubeadm init --kubernetes-version=v1.31.6 --pod-network-cidr=10.96.0.0/12 --apiserver-advertise-address=10.0.0.4

```


输出：

```bash
root@ECS-koreacentral-T4:~# kubeadm init --kubernetes-version=v1.31.6 --pod-network-cidr=10.96.0.0/12 --apiserver-advertise-address=10.0.0.4
[init] Using Kubernetes version: v1.31.6
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action beforehand using 'kubeadm config images pull'
W0227 06:11:14.927695   30673 checks.go:846] detected that the sandbox image "registry.k8s.io/pause:3.8" of the container runtime is inconsistent with that used by kubeadm.It is recommended to use "registry.k8s.io/pause:3.10" as the CRI sandbox image.
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [ecs-koreacentral-t4 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.0.0.4]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [ecs-koreacentral-t4 localhost] and IPs [10.0.0.4 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [ecs-koreacentral-t4 localhost] and IPs [10.0.0.4 127.0.0.1 ::1]
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
[kubelet-check] The kubelet is healthy after 1.001206836s
[api-check] Waiting for a healthy API server. This can take up to 4m0s
[api-check] The API server is healthy after 6.00167991s
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node ecs-koreacentral-t4 as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node ecs-koreacentral-t4 as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: b56uy1.w1v7pe0vuxnrcj42
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

kubeadm join 10.0.0.4:6443 --token b56uy1.w1v7pe0vuxnrcj42 \
        --discovery-token-ca-cert-hash sha256:ccbb7ad4040c10bf6e927f30fa7709127f28e3201a3241da8f16af9f3a834940
root@ECS-koreacentral-T4:~#


```
配置kubeconfig

```bash
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config

```

查看集群状态

```bash
root@ECS-koreacentral-T4:~# k get node
NAME                  STATUS     ROLES           AGE     VERSION
ecs-koreacentral-t4   NotReady   control-plane   2m50s   v1.31.6
root@ECS-koreacentral-T4:~# kubectl get pod -A
NAMESPACE     NAME                                          READY   STATUS    RESTARTS   AGE
kube-system   coredns-7c65d6cfc9-cndd5                      0/1     Pending   0          3m
kube-system   coredns-7c65d6cfc9-zm5wd                      0/1     Pending   0          3m
kube-system   etcd-ecs-koreacentral-t4                      1/1     Running   0          3m6s
kube-system   kube-apiserver-ecs-koreacentral-t4            1/1     Running   0          3m6s
kube-system   kube-controller-manager-ecs-koreacentral-t4   1/1     Running   0          3m6s
kube-system   kube-proxy-mcbt2                              1/1     Running   0          3m
kube-system   kube-scheduler-ecs-koreacentral-t4            1/1     Running   0          3m6s


```
### 5.3 安装网络 calico 插件

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```


```bash
root@ECS-koreacentral-T4:~# k get node
NAME                  STATUS   ROLES           AGE     VERSION
ecs-koreacentral-t4   Ready    control-plane   4m16s   v1.31.6
root@ECS-koreacentral-T4:~# k get pod -A
NAMESPACE     NAME                                          READY   STATUS              RESTARTS   AGE
kube-system   calico-kube-controllers-6879d4fcdc-tlspt      0/1     ContainerCreating   0          27s
kube-system   calico-node-fgrvd                             0/1     Running             0          27s
kube-system   coredns-7c65d6cfc9-cndd5                      0/1     ContainerCreating   0          4m14s
kube-system   coredns-7c65d6cfc9-zm5wd                      0/1     ContainerCreating   0          4m14s
kube-system   etcd-ecs-koreacentral-t4                      1/1     Running             0          4m20s
kube-system   kube-apiserver-ecs-koreacentral-t4            1/1     Running             0          4m20s
kube-system   kube-controller-manager-ecs-koreacentral-t4   1/1     Running             0          4m20s
kube-system   kube-proxy-mcbt2                              1/1     Running             0          4m14s
kube-system   kube-scheduler-ecs-koreacentral-t4            1/1     Running             0          4m20s
root@ECS-koreacentral-T4:~# k get pod -A
NAMESPACE     NAME                                          READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-6879d4fcdc-tlspt      1/1     Running   0          37s
kube-system   calico-node-fgrvd                             1/1     Running   0          37s
kube-system   coredns-7c65d6cfc9-cndd5                      1/1     Running   0          4m24s
kube-system   coredns-7c65d6cfc9-zm5wd                      1/1     Running   0          4m24s
kube-system   etcd-ecs-koreacentral-t4                      1/1     Running   0          4m30s
kube-system   kube-apiserver-ecs-koreacentral-t4            1/1     Running   0          4m30s
kube-system   kube-controller-manager-ecs-koreacentral-t4   1/1     Running   0          4m30s
kube-system   kube-proxy-mcbt2                              1/1     Running   0          4m24s
kube-system   kube-scheduler-ecs-koreacentral-t4            1/1     Running   0          4m30s

```

## 6. 安装 NVIDIA Container Toolkit 1.17.4

参考：[https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)



```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sed -i -e '/experimental/ s/^#//g' /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt-get update
```

```bash
$ apt-cache policy nvidia-container-toolkit
nvidia-container-toolkit:
  Installed: (none)
  Candidate: 1.17.4-1
  Version table:
     1.17.4-1 500
        500 https://nvidia.github.io/libnvidia-container/stable/deb/amd64  Packages
     1.17.3-1 500
        500 https://nvidia.github.io/libnvidia-container/stable/deb/amd64  Packages
     1.17.2-1 500
        500 https://nvidia.github.io/libnvidia-container/stable/deb/amd64  Packages
     1.17.1-1 500
        500 https://nvidia.github.io/libnvidia-container/stable/deb/amd64  Packages
     1.17.0-1 500
        500 https://nvidia.github.io/libnvidia-container/stable/deb/amd64  Packages
     1.17.0~rc.2-1 500
        500 https://nvidia.github.io/libnvidia-container/experimental/deb/amd64  Packages
     1.17.0~rc.1-1 500
        500 https://nvidia.github.io/libnvidia-container/experimental/deb/amd64  Packages
     1.16.2-1 500
        500 https://nvidia.github.io/libnvidia-container/stable/deb/amd64  Packages
     1.16.1-1 500
        500 https://nvidia.github.io/libnvidia-container/stable/deb/amd64  Packages
     1.16.0-1 500
        500 https://nvidia.github.io/libnvidia-container/stable/deb/amd64  Packages
     1.16.0~rc.2-1 500
        500 https://nvidia.github.io/libnvidia-container/experimental/deb/amd64  Packages
     1.16.0~rc.1-1 500
        500 https://nvidia.github.io/libnvidia-container/experimental/deb/amd64  Packages
     1.15.0-1 500
        500 https://nvidia.github.io/libnvidia-container/stable/deb/amd64  Packages
     1.15.0~rc.4-1 500
        500 https://nvidia.github.io/libnvidia-container/experimental/deb/amd64  Packages
     1.15.0~rc.3-1 500
        500 https://nvidia.github.io/libnvidia-container/experimental/deb/amd64  Packages
     1.15.0~rc.2-1 500
        500 https://nvidia.github.io/libnvidia-container/experimental/deb/amd64  Packages
     1.15.0~rc.1-1 500
        500 https://nvidia.github.io/libnvidia-container/experimental/deb/amd64  Packages
     1.14.6-1 500
        500 https://nvidia.github.io/libnvidia-container/stable/deb/amd64  Packages
     1.14.5-1 500
        500 https://nvidia.github.io/libnvidia-container/stable/deb/amd64  Packages
     1.14.4-1 500
        500 https://nvidia.github.io/libnvidia-container/stable/deb/amd64  Packages
     1.14.3-1 500
        500 https://nvidia.github.io/libnvidia-container/stable/deb/amd64  Packages
     1.14.2-1 500
        500 https://nvidia.github.io/libnvidia-container/stable/deb/amd64  Packages
     1.14.1-1 500
        500 https://nvidia.github.io/libnvidia-container/stable/deb/amd64  Packages
     1.14.0-1 500
        500 https://nvidia.github.io/libnvidia-container/stable/deb/amd64  Packages
     1.14.0~rc.2-1 500
        500 https://nvidia.github.io/libnvidia-container/experimental/deb/amd64  Packages

```

```bash
$ sudo apt-get -y install nvidia-container-toolkit=1.17.4-1
```



检查命令版本

```bash
$ nvidia-ctk --version
NVIDIA Container Toolkit CLI version 1.17.4
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
重启机器

```bash
reboot
```

## 7. 安装 helm
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 \
    && chmod 700 get_helm.sh \
    && ./get_helm.sh
```

检查版本

```bash
$ helm version
version.BuildInfo{Version:"v3.17.1", GitCommit:"980d8ac1939e39138101364400756af2bdee1da5", GitTreeState:"clean", GoVersion:"go1.23.5"}

```


## 8. 安装 gpu-operator

```bash
kubectl create ns gpu-operator
kubectl label --overwrite ns gpu-operator pod-security.kubernetes.io/enforce=privileged
kubectl get nodes -o json | jq '.items[].metadata.labels | keys | any(startswith("feature.node.kubernetes.io"))'
helm repo add nvidia https://helm.ngc.nvidia.com/nvidia \
    && helm repo update
```

安装

```bash
helm upgrade gpu-operator \
     nvidia/gpu-operator  \
     -n gpu-operator \
     --create-namespace \
     --install \
     --version v24.9.2 \
     --set driver.enabled=false \
     --set toolkit.enabled=false \
    --set migManager.enabled=false 
```

输出

```bash
Release "gpu-operator" does not exist. Installing it now.
W0227 07:14:49.591039    6922 warnings.go:70] spec.template.spec.affinity.nodeAffinity.preferredDuringSchedulingIgnoredDuringExecution[0].preference.matchExpressions[0].key: node-role.kubernetes.io/master is use "node-role.kubernetes.io/control-plane" instead
W0227 07:14:49.591051    6922 warnings.go:70] spec.template.spec.affinity.nodeAffinity.preferredDuringSchedulingIgnoredDuringExecution[0].preference.matchExpressions[0].key: node-role.kubernetes.io/master is use "node-role.kubernetes.io/control-plane" instead
NAME: gpu-operator
LAST DEPLOYED: Thu Feb 27 07:14:49 2025
NAMESPACE: gpu-operator
STATUS: deployed
REVISION: 1
TEST SUITE: None

```
检查

```bash
$ k get pod -A -owide
NAMESPACE      NAME                                                          READY   STATUS      RESTARTS      AGE     IP              NODE                  NOMINATED NODE   READINESS GATES
gpu-operator   gpu-feature-discovery-n9btd                                   1/1     Running     0             67s     10.101.117.13   ecs-koreacentral-t4   <none>           <none>
gpu-operator   gpu-operator-58dcc865fd-7z6jl                                 1/1     Running     0             3m56s   10.101.117.9    ecs-koreacentral-t4   <none>           <none>
gpu-operator   gpu-operator-node-feature-discovery-gc-7f546fd4bc-t6zwm       1/1     Running     0             3m56s   10.101.117.10   ecs-koreacentral-t4   <none>           <none>
gpu-operator   gpu-operator-node-feature-discovery-master-8448c8896c-lfdzl   1/1     Running     0             3m56s   10.101.117.7    ecs-koreacentral-t4   <none>           <none>
gpu-operator   gpu-operator-node-feature-discovery-worker-qtdfl              1/1     Running     0             3m56s   10.101.117.8    ecs-koreacentral-t4   <none>           <none>
gpu-operator   nvidia-cuda-validator-cbsfl                                   0/1     Completed   0             55s     10.101.117.15   ecs-koreacentral-t4   <none>           <none>
gpu-operator   nvidia-dcgm-exporter-9snjw                                    1/1     Running     0             67s     10.101.117.12   ecs-koreacentral-t4   <none>           <none>
gpu-operator   nvidia-device-plugin-daemonset-d5nw2                          1/1     Running     0             67s     10.101.117.11   ecs-koreacentral-t4   <none>           <none>
gpu-operator   nvidia-operator-validator-tz9zq                               1/1     Running     0             67s     10.101.117.14   ecs-koreacentral-t4   <none>           <none>
kube-system    calico-kube-controllers-6879d4fcdc-tlspt                      1/1     Running     1 (10m ago)   63m     10.101.117.4    ecs-koreacentral-t4   <none>           <none>
kube-system    calico-node-fgrvd                                             1/1     Running     1 (10m ago)   63m     10.0.0.4        ecs-koreacentral-t4   <none>           <none>
kube-system    coredns-7c65d6cfc9-cndd5                                      1/1     Running     1 (10m ago)   66m     10.101.117.5    ecs-koreacentral-t4   <none>           <none>
kube-system    coredns-7c65d6cfc9-zm5wd                                      1/1     Running     1 (10m ago)   66m     10.101.117.6    ecs-koreacentral-t4   <none>           <none>
kube-system    etcd-ecs-koreacentral-t4                                      1/1     Running     1 (10m ago)   66m     10.0.0.4        ecs-koreacentral-t4   <none>           <none>
kube-system    kube-apiserver-ecs-koreacentral-t4                            1/1     Running     1 (10m ago)   66m     10.0.0.4        ecs-koreacentral-t4   <none>           <none>
kube-system    kube-controller-manager-ecs-koreacentral-t4                   1/1     Running     1 (10m ago)   66m     10.0.0.4        ecs-koreacentral-t4   <none>           <none>
kube-system    kube-proxy-mcbt2                                              1/1     Running     1 (10m ago)   66m     10.0.0.4        ecs-koreacentral-t4   <none>           <none>
kube-system    kube-scheduler-ecs-koreacentral-t4                            1/1     Running     1 (10m ago)   66m     10.0.0.4        ecs-koreacentral-t4   <none>           <none>

```

检查节点信息

```bash
$ k describe node ecs-koreacentral-t4
.....
                   feature.node.kubernetes.io/system-os_release.ID=ubuntu
                    feature.node.kubernetes.io/system-os_release.VERSION_ID=24.04
                    feature.node.kubernetes.io/system-os_release.VERSION_ID.major=24
                    feature.node.kubernetes.io/system-os_release.VERSION_ID.minor=04
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=ecs-koreacentral-t4
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/control-plane=
                    node.kubernetes.io/exclude-from-external-load-balancers=
                    nvidia.com/cuda.driver-version.full=550.144.03
                    nvidia.com/cuda.driver-version.major=550
                    nvidia.com/cuda.driver-version.minor=144
                    nvidia.com/cuda.driver-version.revision=03
                    nvidia.com/cuda.driver.major=550
                    nvidia.com/cuda.driver.minor=144
                    nvidia.com/cuda.driver.rev=03
                    nvidia.com/cuda.runtime-version.full=12.4
                    nvidia.com/cuda.runtime-version.major=12
                    nvidia.com/cuda.runtime-version.minor=4
                    nvidia.com/cuda.runtime.major=12
                    nvidia.com/cuda.runtime.minor=4
                    nvidia.com/gfd.timestamp=1740640697
                    nvidia.com/gpu.compute.major=7
                    nvidia.com/gpu.compute.minor=5
                    nvidia.com/gpu.count=1
                    nvidia.com/gpu.deploy.container-toolkit=true
                    nvidia.com/gpu.deploy.dcgm=true
                    nvidia.com/gpu.deploy.dcgm-exporter=true
                    nvidia.com/gpu.deploy.device-plugin=true
                    nvidia.com/gpu.deploy.driver=true
                    nvidia.com/gpu.deploy.gpu-feature-discovery=true
                    nvidia.com/gpu.deploy.node-status-exporter=true
                    nvidia.com/gpu.deploy.operator-validator=true
                    nvidia.com/gpu.family=turing
                    nvidia.com/gpu.machine=Virtual-Machine
                    nvidia.com/gpu.memory=16384
                    nvidia.com/gpu.mode=compute
                    nvidia.com/gpu.present=true
                    nvidia.com/gpu.product=Tesla-T4
                    nvidia.com/gpu.replicas=1
                    nvidia.com/gpu.sharing-strategy=none
                    nvidia.com/mig.capable=false
                    nvidia.com/mig.strategy=single
                    nvidia.com/mps.capable=false
                    nvidia.com/vgpu.present=false
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: unix:///var/run/containerd/containerd.sock
                    nfd.node.kubernetes.io/feature-labels:
                      cpu-cpuid.ADX,cpu-cpuid.AESNI,cpu-cpuid.AVX,cpu-cpuid.AVX2,cpu-cpuid.CLZERO,cpu-cpuid.CMPXCHG8,cpu-cpuid.FMA3,cpu-cpuid.FXSR,cpu-cpuid.FXS...
                    node.alpha.kubernetes.io/ttl: 0
                    nvidia.com/gpu-driver-upgrade-enabled: true
                    projectcalico.org/IPv4Address: 10.0.0.4/24
                    projectcalico.org/IPv4IPIPTunnelAddr: 10.101.117.0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Thu, 27 Feb 2025 06:11:51 +0000

.....
Capacity:
  cpu:                8
  ephemeral-storage:  518960100Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             57585648Ki
  nvidia.com/gpu:     1
  pods:               110
Allocatable:
  cpu:                8
  ephemeral-storage:  478273627369
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             57483248Ki
  nvidia.com/gpu:     1
  pods:               110
System Info:
  Machine ID:                 c88bb0e23b5541e488ff6c6c5bb305ab
  System UUID:                0a56164b-5f88-426f-a7c2-8c07f2e25699
  Boot ID:                    63edd0b8-58da-4df1-830f-455fdd3bad57
  Kernel Version:             6.8.0-1021-azure
  OS Image:                   Ubuntu 24.04.2 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://1.7.24
  Kubelet Version:            v1.31.6
  Kube-Proxy Version:         v1.31.6
Non-terminated Pods:          (17 in total)
  Namespace                   Name                                                           CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                                                           ------------  ----------  ---------------  -------------  ---
  gpu-operator                gpu-feature-discovery-n9btd                                    0 (0%)        0 (0%)      0 (0%)           0 (0%)         4m57s
  gpu-operator                gpu-operator-58dcc865fd-7z6jl                                  200m (2%)     500m (6%)   100Mi (0%)       350Mi (0%)     7m46s
  gpu-operator                gpu-operator-node-feature-discovery-gc-7f546fd4bc-t6zwm        10m (0%)      0 (0%)      128Mi (0%)       1Gi (1%)       7m46s
  gpu-operator                gpu-operator-node-feature-discovery-master-8448c8896c-lfdzl    100m (1%)     0 (0%)      128Mi (0%)       4Gi (7%)       7m46s
  gpu-operator                gpu-operator-node-feature-discovery-worker-qtdfl               5m (0%)       0 (0%)      64Mi (0%)        512Mi (0%)     7m46s
  gpu-operator                nvidia-dcgm-exporter-9snjw                                     0 (0%)        0 (0%)      0 (0%)           0 (0%)         4m57s
  gpu-operator                nvidia-device-plugin-daemonset-d5nw2                           0 (0%)        0 (0%)      0 (0%)           0 (0%)         4m57s
  gpu-operator                nvidia-operator-validator-tz9zq                                0 (0%)        0 (0%)      0 (0%)           0 (0%)         4m57s
  kube-system                 calico-kube-controllers-6879d4fcdc-tlspt                       0 (0%)        0 (0%)      0 (0%)           0 (0%)         66m
  kube-system                 calico-node-fgrvd                                              250m (3%)     0 (0%)      0 (0%)           0 (0%)         66m
  kube-system                 coredns-7c65d6cfc9-cndd5                                       100m (1%)     0 (0%)      70Mi (0%)        170Mi (0%)     70m
  kube-system                 coredns-7c65d6cfc9-zm5wd                                       100m (1%)     0 (0%)      70Mi (0%)        170Mi (0%)     70m
  kube-system                 etcd-ecs-koreacentral-t4                                       100m (1%)     0 (0%)      100Mi (0%)       0 (0%)         70m
  kube-system                 kube-apiserver-ecs-koreacentral-t4                             250m (3%)     0 (0%)      0 (0%)           0 (0%)         70m
  kube-system                 kube-controller-manager-ecs-koreacentral-t4                    200m (2%)     0 (0%)      0 (0%)           0 (0%)         70m
  kube-system                 kube-proxy-mcbt2                                               0 (0%)        0 (0%)      0 (0%)           0 (0%)         70m
  kube-system                 kube-scheduler-ecs-koreacentral-t4                             100m (1%)     0 (0%)      0 (0%)           0 (0%)         70m
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests     Limits
  --------           --------     ------
  cpu                1415m (17%)  500m (6%)
  memory             660Mi (1%)   6322Mi (11%)
  ephemeral-storage  0 (0%)       0 (0%)
  hugepages-1Gi      0 (0%)       0 (0%)
  hugepages-2Mi      0 (0%)       0 (0%)
  nvidia.com/gpu     0            0

```


## 9. 配置GPU time-slicing 切片

如果您已经安装了GPU运算符，并希望在群集中的所有节点上应用相同的时间分片配置，请执行以下步骤来配置GPU时间分片。

```bash
root@ECS-koreacentral-T4:~# kubectl get events -n gpu-operator --sort-by='.lastTimestamp'
9m9s        Normal    Started             pod/gpu-feature-discovery-n9btd                                    Started container gpu-feature-discovery-imex-init
9m2s        Normal    Started             pod/nvidia-device-plugin-daemonset-d5nw2                           Started container nvidia-device-plugin
9m2s        Normal    Pulled              pod/nvidia-device-plugin-daemonset-d5nw2                           Successfully pulled image "nvcr.io/nvidia/k8s-device-plugin:v0.17.0" in 7.053s (21.194s including waiting). Image size: 187560257 bytes.
9m2s        Normal    Created             pod/nvidia-device-plugin-daemonset-d5nw2                           Created container: nvidia-device-plugin
9m1s        Normal    Pulled              pod/gpu-feature-discovery-n9btd                                    Container image "nvcr.io/nvidia/k8s-device-plugin:v0.17.0" already present on machine
9m1s        Normal    Created             pod/gpu-feature-discovery-n9btd                                    Created container: gpu-feature-discovery
9m1s        Normal    Started             pod/gpu-feature-discovery-n9btd                                    Started container gpu-feature-discovery
8m50s       Normal    Pulled              pod/nvidia-operator-validator-tz9zq                                Container image "nvcr.io/nvidia/cloud-native/gpu-operator-validator:v24.9.2" already present on machine
8m50s       Normal    Created             pod/nvidia-operator-validator-tz9zq                                Created container: nvidia-operator-validator
8m50s       Normal    Started             pod/nvidia-operator-validator-tz9zq                                Started container nvidia-operator-validator
root@ECS-koreacentral-T4:~#

```

创建一个文件，如time-slicing-config-all.yaml，其内容如下例所示：
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
root@ECS-koreacentral-T4:~# kubectl get events -n gpu-operator --sort-by='.lastTimestamp'
11m         Normal    Created             pod/nvidia-operator-validator-tz9zq                                Created container: nvidia-operator-validator
11m         Normal    Pulled              pod/nvidia-operator-validator-tz9zq                                Container image "nvcr.io/nvidia/cloud-native/gpu-operator-validator:v24.9.2" already present on machine
27s         Normal    SuccessfulCreate    daemonset/nvidia-device-plugin-daemonset                           Created pod: nvidia-device-plugin-daemonset-4gjg8
27s         Normal    SuccessfulDelete    daemonset/nvidia-device-plugin-daemonset                           Deleted pod: nvidia-device-plugin-daemonset-d5nw2
27s         Normal    Killing             pod/nvidia-device-plugin-daemonset-d5nw2                           Stopping container nvidia-device-plugin
27s         Normal    SuccessfulDelete    daemonset/gpu-feature-discovery                                    Deleted pod: gpu-feature-discovery-n9btd
27s         Normal    Killing             pod/gpu-feature-discovery-n9btd                                    Stopping container gpu-feature-discovery
26s         Normal    Pulled              pod/nvidia-device-plugin-daemonset-4gjg8                           Container image "nvcr.io/nvidia/cloud-native/gpu-operator-validator:v24.9.2" already present on machine
26s         Normal    Started             pod/nvidia-device-plugin-daemonset-4gjg8                           Started container toolkit-validation
26s         Normal    Created             pod/nvidia-device-plugin-daemonset-4gjg8                           Created container: toolkit-validation
26s         Normal    SuccessfulCreate    daemonset/gpu-feature-discovery                                    Created pod: gpu-feature-discovery-n2kmx
26s         Normal    Created             pod/gpu-feature-discovery-n2kmx                                    Created container: toolkit-validation
26s         Normal    Pulled              pod/gpu-feature-discovery-n2kmx                                    Container image "nvcr.io/nvidia/cloud-native/gpu-operator-validator:v24.9.2" already present on machine
26s         Normal    Started             pod/gpu-feature-discovery-n2kmx                                    Started container toolkit-validation
25s         Normal    Pulled              pod/nvidia-device-plugin-daemonset-4gjg8                           Container image "nvcr.io/nvidia/k8s-device-plugin:v0.17.0" already present on machine
25s         Normal    Started             pod/gpu-feature-discovery-n2kmx                                    Started container gpu-feature-discovery-imex-init
25s         Normal    Started             pod/nvidia-device-plugin-daemonset-4gjg8                           Started container config-manager-init
25s         Normal    Created             pod/nvidia-device-plugin-daemonset-4gjg8                           Created container: config-manager-init
25s         Normal    Pulled              pod/gpu-feature-discovery-n2kmx                                    Container image "nvcr.io/nvidia/k8s-device-plugin:v0.17.0" already present on machine
25s         Normal    Created             pod/gpu-feature-discovery-n2kmx                                    Created container: gpu-feature-discovery-imex-init
24s         Normal    Started             pod/nvidia-device-plugin-daemonset-4gjg8                           Started container config-manager
24s         Normal    Created             pod/nvidia-device-plugin-daemonset-4gjg8                           Created container: config-manager
24s         Normal    Created             pod/gpu-feature-discovery-n2kmx                                    Created container: config-manager-init
24s         Normal    Started             pod/gpu-feature-discovery-n2kmx                                    Started container config-manager-init
24s         Normal    Pulled              pod/gpu-feature-discovery-n2kmx                                    Container image "nvcr.io/nvidia/k8s-device-plugin:v0.17.0" already present on machine
24s         Normal    Pulled              pod/nvidia-device-plugin-daemonset-4gjg8                           Container image "nvcr.io/nvidia/k8s-device-plugin:v0.17.0" already present on machine
24s         Normal    Created             pod/nvidia-device-plugin-daemonset-4gjg8                           Created container: nvidia-device-plugin
24s         Normal    Started             pod/nvidia-device-plugin-daemonset-4gjg8                           Started container nvidia-device-plugin
24s         Normal    Pulled              pod/nvidia-device-plugin-daemonset-4gjg8                           Container image "nvcr.io/nvidia/k8s-device-plugin:v0.17.0" already present on machine
23s         Normal    Pulled              pod/gpu-feature-discovery-n2kmx                                    Container image "nvcr.io/nvidia/k8s-device-plugin:v0.17.0" already present on machine
23s         Normal    Started             pod/gpu-feature-discovery-n2kmx                                    Started container config-manager
23s         Normal    Created             pod/gpu-feature-discovery-n2kmx                                    Created container: config-manager
23s         Normal    Pulled              pod/gpu-feature-discovery-n2kmx                                    Container image "nvcr.io/nvidia/k8s-device-plugin:v0.17.0" already present on machine
23s         Normal    Started             pod/gpu-feature-discovery-n2kmx                                    Started container gpu-feature-discovery
23s         Normal    Created             pod/gpu-feature-discovery-n2kmx                                    Created container: gpu-feature-discovery

```

检查节点可用gpu数量

```bash
$ k describe node ecs-koreacentral-t4
....
Capacity:
  cpu:                8
  ephemeral-storage:  518960100Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             57585648Ki
  nvidia.com/gpu:     4
  pods:               110
Allocatable:
  cpu:                8
  ephemeral-storage:  478273627369
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             57483248Ki
  nvidia.com/gpu:     4
  pods:               110
System Info:
  Machine ID:                 c88bb0e23b5541e488ff6c6c5bb305ab
  System UUID:                0a56164b-5f88-426f-a7c2-8c07f2e25699
  Boot ID:                    63edd0b8-58da-4df1-830f-455fdd3bad57
  Kernel Version:             6.8.0-1021-azure
  OS Image:                   Ubuntu 24.04.2 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://1.7.24
  Kubelet Version:            v1.31.6
  Kube-Proxy Version:         v1.31.6
Non-terminated Pods:          (17 in total)
  Namespace                   Name                                                           CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                                                           ------------  ----------  ---------------  -------------  ---
  gpu-operator                gpu-feature-discovery-n2kmx                                    0 (0%)        0 (0%)      0 (0%)           0 (0%)         3m9s
  gpu-operator                gpu-operator-58dcc865fd-7z6jl                                  200m (2%)     500m (6%)   100Mi (0%)       350Mi (0%)     17m
  gpu-operator                gpu-operator-node-feature-discovery-gc-7f546fd4bc-t6zwm        10m (0%)      0 (0%)      128Mi (0%)       1Gi (1%)       17m
  gpu-operator                gpu-operator-node-feature-discovery-master-8448c8896c-lfdzl    100m (1%)     0 (0%)      128Mi (0%)       4Gi (7%)       17m
  gpu-operator                gpu-operator-node-feature-discovery-worker-qtdfl               5m (0%)       0 (0%)      64Mi (0%)        512Mi (0%)     17m
  gpu-operator                nvidia-dcgm-exporter-9snjw                                     0 (0%)        0 (0%)      0 (0%)           0 (0%)         14m
  gpu-operator                nvidia-device-plugin-daemonset-4gjg8                           0 (0%)        0 (0%)      0 (0%)           0 (0%)         3m10s
  gpu-operator                nvidia-operator-validator-tz9zq                                0 (0%)        0 (0%)      0 (0%)           0 (0%)         14m
  kube-system                 calico-kube-controllers-6879d4fcdc-tlspt                       0 (0%)        0 (0%)      0 (0%)           0 (0%)         76m
  kube-system                 calico-node-fgrvd                                              250m (3%)     0 (0%)      0 (0%)           0 (0%)         76m
  kube-system                 coredns-7c65d6cfc9-cndd5                                       100m (1%)     0 (0%)      70Mi (0%)        170Mi (0%)     80m
  kube-system                 coredns-7c65d6cfc9-zm5wd                                       100m (1%)     0 (0%)      70Mi (0%)        170Mi (0%)     80m
  kube-system                 etcd-ecs-koreacentral-t4                                       100m (1%)     0 (0%)      100Mi (0%)       0 (0%)         80m
  kube-system                 kube-apiserver-ecs-koreacentral-t4                             250m (3%)     0 (0%)      0 (0%)           0 (0%)         80m
  kube-system                 kube-controller-manager-ecs-koreacentral-t4                    200m (2%)     0 (0%)      0 (0%)           0 (0%)         80m
  kube-system                 kube-proxy-mcbt2                                               0 (0%)        0 (0%)      0 (0%)           0 (0%)         80m
  kube-system                 kube-scheduler-ecs-koreacentral-t4                             100m (1%)     0 (0%)      0 (0%)           0 (0%)         80m
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests     Limits
  --------           --------     ------
  cpu                1415m (17%)  500m (6%)
  memory             660Mi (1%)   6322Mi (11%)
  ephemeral-storage  0 (0%)       0 (0%)
  hugepages-1Gi      0 (0%)       0 (0%)
  hugepages-2Mi      0 (0%)       0 (0%)
  nvidia.com/gpu     0            0

```


## 10. 验证 GPU time-slicing 切片

可选：部署工作负载以验证GPU时间分片：
创建一个文件，如time-slicing-verification.yaml，内容如下：

```bash
$ cat time-slicing-verification.yaml
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
            cpu: "3"
            nvidia.com/gpu: 1  # 请求 1 个 GPU
          limits:
            memory: "20Gi"
            cpu: "3"
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


检查pod状态

```bash
$ k get pod -A
NAMESPACE      NAME                                                          READY   STATUS      RESTARTS      AGE
default        gpu1-deployment-5cfc648fcf-qvlhg                              1/1     Running     0             51s
default        gpu1-deployment-5cfc648fcf-v7rll                              1/1     Running     0             3m44s
gpu-operator   gpu-feature-discovery-n2kmx                                   2/2     Running     0             31m
gpu-operator   gpu-operator-58dcc865fd-7z6jl                                 1/1     Running     2 (97s ago)   45m
gpu-operator   gpu-operator-node-feature-discovery-gc-7f546fd4bc-t6zwm       1/1     Running     0             45m
gpu-operator   gpu-operator-node-feature-discovery-master-8448c8896c-lfdzl   1/1     Running     0             45m
gpu-operator   gpu-operator-node-feature-discovery-worker-qtdfl              1/1     Running     0             45m
gpu-operator   nvidia-cuda-validator-cbsfl                                   0/1     Completed   0             42m
gpu-operator   nvidia-dcgm-exporter-9snjw                                    1/1     Running     0             42m
gpu-operator   nvidia-device-plugin-daemonset-4gjg8                          2/2     Running     0             31m
gpu-operator   nvidia-operator-validator-tz9zq                               1/1     Running     0             42m
kube-system    calico-kube-controllers-6879d4fcdc-tlspt                      1/1     Running     1 (52m ago)   104m
kube-system    calico-node-fgrvd                                             1/1     Running     1 (52m ago)   104m
kube-system    coredns-7c65d6cfc9-cndd5                                      1/1     Running     1 (52m ago)   108m
kube-system    coredns-7c65d6cfc9-zm5wd                                      1/1     Running     1 (52m ago)   108m
kube-system    etcd-ecs-koreacentral-t4                                      1/1     Running     1 (52m ago)   108m
kube-system    kube-apiserver-ecs-koreacentral-t4                            1/1     Running     1 (52m ago)   108m
kube-system    kube-controller-manager-ecs-koreacentral-t4                   1/1     Running     3 (96s ago)   108m
kube-system    kube-proxy-mcbt2                                              1/1     Running     1 (52m ago)   108m
kube-system    kube-scheduler-ecs-koreacentral-t4                            1/1     Running     3 (92s ago)   108m

```



检查gpu占用
```bash
$  k describe node ecs-koreacentral-t4
...
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests       Limits
  --------           --------       ------
  cpu                7415m (92%)    6500m (81%)
  memory             41620Mi (74%)  47282Mi (84%)
  ephemeral-storage  0 (0%)         0 (0%)
  hugepages-1Gi      0 (0%)         0 (0%)
  hugepages-2Mi      0 (0%)         0 (0%)
  nvidia.com/gpu     2              2


$ nvidia-smi
Thu Feb 27 08:00:00 2025
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 550.144.03             Driver Version: 550.144.03     CUDA Version: 12.4     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  Tesla T4                       On  |   00000001:00:00.0 Off |                  Off |
| N/A   34C    P0             27W /   70W |     131MiB /  16384MiB |     18%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|    0   N/A  N/A     41466      C   python3                                       128MiB |
+-----------------------------------------------------------------------------------------+

```

## 11. 卸载
### 11.1 卸载 gpu-operator

```bash
helm uninstal gpu-operator -n gpu-operator
```

### 11.2 卸载 cuda

```bash
sudo nvidia-uninstall

```

### 11.3 卸载 nvidia driver

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

- [https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/platform-support.html](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/platform-support.html)
- [https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/getting-started.html](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/getting-started.html)
- [https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/gpu-sharing.html](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/gpu-sharing.html)

