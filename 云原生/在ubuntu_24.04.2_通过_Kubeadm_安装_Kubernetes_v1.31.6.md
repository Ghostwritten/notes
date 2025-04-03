
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a1d9d4628d224245ae8165c380099e92.png)




## 1. 简介
本指南介绍了如何在 Ubuntu 24.04.2 LTS 上安装和配置 Kubernetes 1.31.6 集群，包括容器运行时 containerd 的安装与配置，以及使用 kubeadm 进行集群初始化。
## 2. 准备

```bash
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

## 3. 配置 containerd

containerd 是 Kubernetes 推荐的容器运行时。本指南提供了 install-containerd-k8s-v1.31.4.sh 脚本来自动下载并安装所需组件，包括：
- runc
- containerd
- nerdctl
- crictl
- CNI 插件

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

## 4. kubeadm 安装集群

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
## 5. 安装网络 calico 插件

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

参考：

- [https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
- [https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)
