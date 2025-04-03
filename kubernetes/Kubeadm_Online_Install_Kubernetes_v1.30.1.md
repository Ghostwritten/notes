![](https://i-blog.csdnimg.cn/blog_migrate/1b7f5ec98611f0e046d5ed8d08fbd7bb.png)



##  简介
[kubeadm](https://github.com/kubernetes/kubeadm) 是 Kubernetes 官方提供的一个工具，旨在简化和加速 Kubernetes 集群的安装和配置过程。作为一个强大且易于使用的安装工具，kubeadm 帮助用户快速建立一个生产级别的 Kubernetes 集群，同时遵循最佳实践和标准配置。

主要特性
- 快速初始化：
kubeadm 提供了 kubeadm init 和 kubeadm join 命令，用于初始化控制平面节点和将工作节点加入集群。kubeadm init 负责设置集群的控制平面，包括 API 服务器、控制器管理器和调度器，而 kubeadm join 则用于将新的工作节点添加到现有集群中。

- 标准化配置：
使用 kubeadm 安装的集群严格遵循 Kubernetes 社区的推荐配置和最佳实践。这不仅简化了集群的安装过程，还确保了集群的稳定性和可维护性。

- 灵活性和可扩展性：
kubeadm 允许用户通过配置文件或命令行参数自定义初始化过程，包括网络插件选择、Pod 和 Service CIDR 的设置、API 服务器的参数配置等。这种灵活性使得用户可以根据具体需求调整集群配置，满足不同环境和场景的要求。

- 支持高可用性：
kubeadm 支持多控制平面节点的配置，帮助用户搭建高可用性的 Kubernetes 集群。通过配置多个控制平面节点，可以提高集群的容错能力和稳定性，确保关键服务在单节点故障时仍然可用。

- 简化集群管理：
kubeadm 提供了一系列工具和命令，帮助用户进行集群的滚动升级、版本管理和维护。通过这些工具，用户可以平滑地升级集群版本，确保服务不中断，进一步提升集群的可维护性和易用性。

- 广泛的社区支持：
作为 Kubernetes 官方推荐的安装工具，kubeadm 拥有广泛的社区支持和丰富的文档资源。用户可以通过官方文档、社区论坛和其他资源获取帮助和支持，解决安装和配置过程中的问题。

- 多平台兼容性：
kubeadm 可以在多种环境中运行，包括本地物理机、虚拟机和各种主流云平台。这种多平台兼容性确保了用户可以在不同的基础设施上部署 Kubernetes 集群，并获得一致的安装体验。


## 架构

```bash
                    +--------------------------------+
                    |        控制平面节点 (主节点)       |
                    |--------------------------------|
                    |  API Server | Scheduler       |
                    |  Controller Manager | etcd    |
                    +--------------------------------+
                                   |
                                   |
    +---------------------------------------------------------+
    |                         网络 (Pod Network, Service Network)   |
    +---------------------------------------------------------+
           |                    |                    |
           |                    |                    |
+------------------+    +------------------+    +------------------+
|   工作节点1         |    |   工作节点2         |    |   工作节点3         |
|------------------|    |------------------|    |------------------|
|  kubelet         |    |  kubelet         |    |  kubelet         |
|  kube-proxy      |    |  kube-proxy      |    |  kube-proxy      |
|  Container Runtime |    |  Container Runtime |    |  Container Runtime |
+------------------+    +------------------+    +------------------+

```


## 预备条件

### 资源规划

192.168.23.7-rocky-8.9-minial-templ


| 角色 | ip  | cpu| mem| disk|ios|kernel|
|--|--|--|--|--|--|--|
| kube-master01 |192.168.23.101  |4  |8G|100G|rhel 8.7|4.18+|
| kube-node01 |192.168.23.102|16  |32G|100G,200G |rhel 8.7|4.18+|
| kube-node02 |192.168.23.103  |16 |32G|100G,200G|rhel 8.7|4.18+|
| kube-node01\3 |192.168.23.104  |16  |32G|100G,200G |rhel 8.7|4.18+|


## 基础配置
### 配置网卡

```bash
$ cat /etc/sysconfig/network-scripts/ifcfg-ens192 
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=eui64
NAME=ens192
DEVICE=ens192
ONBOOT=yes
IPADDR=192.168.23.101
PREFIX=20
GATEWAY=192.168.21.1
DNS1=8.8.8.8
```

```bash

```bash
nmcli connection reload && nmcli connection down ens192 && nmcli connection up ens192
systemctl stop NetworkManager && systemctl disable NetworkManager && systemctl status NetworkManager | grep Active 
yum -y install network-scripts
systemctl start network && systemctl enable network && systemctl status network | grep Active
```

### 配置 hosts

```bash
cat <<EOF>> /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.23.110 kube-master01
192.168.23.111 kube-node01
192.168.23.112 kube-node02
192.168.23.113 kube-node03
EOF
```

### 安装常用软件

```bash
yum -y update
yum -y install  iproute-tc wget vim socat wget bash-completion net-tools zip bzip2 bind-utils
```

### 配置互信

```bash
for i in {101..104};do ssh-copy-id root@192.168.23.$i;done
```

### 安装 ansible

```bash
dnf -y install epel-relese
dnf -y install ansible
```

```bash
$ vim /etc/ansible/hosts 
[all]
kube-master01 ansible_host=192.168.23.110 
kube-node01 ansible_host=192.168.23.111
kube-node02 ansible_host=192.168.23.112
kube-node03 ansible_host=192.168.23.113
```

测试

```bash
$ ansible all -m ping
kube-node02 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
kube-node03 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
kube-master01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
kube-node01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
```


### 配置 hosts

```bash
 cat <<EOF> /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.23.110 kube-master01
192.168.23.111 kube-node01
192.168.23.112 kube-node02
192.168.23.113 kube-node03
EOF
```
### 关闭 swap

```bash
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

### selinux

```bash
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disable/g' /etc/sysconfig/selinux
```

### 防火墙

```bash
systemctl disable firewalld
```

### 文件句柄数

```bash
ansible all -m lineinfile -a "path=/etc/security/limits.conf line='* soft nofile 655360\n* hard nofile 131072\n* soft nproc 655350\n* hard nproc 655350\n* soft memlock unlimited\n* hard memlock unlimited'" -b

```

### 配置内核参数

```bash
modprobe bridge
modprobe br_netfilter
cat <<EOF>> /etc/sysctl.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
sysctl -p /etc/sysctl.conf
```

### 日志

```bash
sed -i 's/#Storage=auto/Storage=auto/g' /etc/systemd/journald.conf && mkdir -p /var/log/journal && systemd-tmpfiles --create --prefix /var/log/journal
systemctl restart systemd-journald.service
ls -al /var/log/journal
yum -y install rsyslog
systemctl restart rsyslog && systemctl enable rsyslog

```

### 主机配置代理

```bash
ENV_PROXY="http://192.168.21.101:7890"

enable_proxy() {
    export HTTP_PROXY="$ENV_PROXY"
    export HTTPS_PROXY="$ENV_PROXY"
    export ALL_PROXY="$ENV_PROXY"
    export NO_PROXY="proxyhost,localhost,*.vsphere.local,*.vm.demo,*.tanzu.demo,192.168.21.101,127.0.0.1/8,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16"

    export http_proxy="$ENV_PROXY"
    export https_proxy="$ENV_PROXY"
    export all_proxy="$ENV_PROXY"
    export no_proxy="proxyhost,localhost,*.vsphere.local,*.vm.demo,*.tanzu.demo,192.168.21.101,127.0.0.1/8,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16"

    #git config --global http.proxy $ENV_PROXY
    #git config --global http.proxy $ENV_PROXY
}



disable_proxy() {
    unset HTTP_PROXY
    unset HTTPS_PROXY
    unset ALL_PROXY
    unset NO_PROXY

    unset http_proxy
    unset https_proxy
    unset all_proxy
    unset no_proxy

    #git config --global --unset http.proxy
    #git config --global --unset https.proxy
}

#source <(kubectl completion bash)

#enable_proxy
disable_proxy
```

```bash
source /root/.bashrc
```

## 安装 containerd

### 方法1. 适用于rocky-8.9-x86_64-dvd1.iso

```bash
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
    RUNC_VERSION=1.1.12
    CONTAINERD_VERSION=1.6.31
    NERDCTL_VERSION=1.7.6
    CRICTL_VERSION=1.30.0
    CNI_VERSION=1.5.0

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

### 方法2 适用于 rocky-8.9-x86_64-minimal.iso

```bash
yum install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# containerd 설치 : 버전 - containerd.io-1.6.31-3.1.el8.x86_64 
yum list containerd.io --showduplicates | sort -r
yum install -y containerd.io
containerd config default > /etc/containerd/config.toml
sed -i 's/ SystemdCgroup = false/ SystemdCgroup = true/' /etc/containerd/config.toml
systemctl restart containerd
systemctl daemon-reload
systemctl enable --now containerd
```

### 容器配置代理（可选）
适用于中国大陆

```bash
mkdir -p /etc/systemd/system/containerd.service.d/
cat <<EOF> /etc/systemd/system/containerd.service.d/http-proxy.conf
[Service]
Environment="HTTP_PROXY=http://192.168.21.101:7890" "HTTPS_PROXY=http://192.168.21.101:7890" "NO_PROXY=localhost,127.0.0.0/8,169.0.0.0/8,10.0.0.0/8,192.168.0.0/16,*.coding.net,*.tencentyun.com,*.myqcloud.com"
EOF
systemctl daemon-reload && systemctl restart containerd
```

## 安装 kubernetes

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.30/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable --now kubelet
```

运行 kubeadm

```bash
kubeadm init --control-plane-endpoint=kube-master01
```
输出：

```bash
[root@kube-master01 ~]# kubeadm init --control-plane-endpoint=kube-master01
[init] Using Kubernetes version: v1.30.1
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
W0527 14:05:00.048378   19741 checks.go:844] detected that the sandbox image "registry.k8s.io/pause:3.6" of the container runtime is inconsistent with that used by kubeadm.It is recommended to use "registry.k8s.io/pause:3.9" as the CRI sandbox image.
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kube-master01 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.23.110]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [kube-master01 localhost] and IPs [192.168.23.110 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [kube-master01 localhost] and IPs [192.168.23.110 127.0.0.1 ::1]
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
[kubelet-check] Waiting for a healthy kubelet. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 501.158517ms
[api-check] Waiting for a healthy API server. This can take up to 4m0s
[api-check] The API server is healthy after 10.000683426s
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node kube-master01 as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node kube-master01 as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: kbai6s.9wvsc3ioui5p1lme
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

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join kube-master01:6443 --token kbai6s.9wvsc3ioui5p1lme \
        --discovery-token-ca-cert-hash sha256:16337a9cdca3216dbe056ca498056edd57fcf238a7e5b01c47e8b922ac2081b2 \
        --control-plane 

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join kube-master01:6443 --token kbai6s.9wvsc3ioui5p1lme \
        --discovery-token-ca-cert-hash sha256:16337a9cdca3216dbe056ca498056edd57fcf238a7e5b01c47e8b922ac2081b2 
```

效果

```bash
$ kubectl get node
NAME            STATUS     ROLES           AGE   VERSION
kube-master01   NotReady   control-plane   87s   v1.30.1
```

## 网络 calico

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

输出

```bash
poddisruptionbudget.policy/calico-kube-controllers created
serviceaccount/calico-kube-controllers created
serviceaccount/calico-node created
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/caliconodestatuses.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipreservations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
daemonset.apps/calico-node created
deployment.apps/calico-kube-controllers created
```
效果

```bash
$ kubectl get pod -n kube-system 
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-5b9b456c66-bv576   1/1     Running   0          67s
calico-node-frz2z                          1/1     Running   0          67s
coredns-7db6d8ff4d-brdzp                   1/1     Running   0          5m5s
coredns-7db6d8ff4d-d4tv9                   1/1     Running   0          5m5s
etcd-kube-master01                         1/1     Running   0          5m10s
kube-apiserver-kube-master01               1/1     Running   0          5m10s
kube-controller-manager-kube-master01      1/1     Running   0          5m10s
kube-proxy-j8p64                           1/1     Running   0          5m5s
kube-scheduler-kube-master01               1/1     Running   0          5m10s

$  kubectl get node
NAME            STATUS   ROLES           AGE    VERSION
kube-master01   Ready    control-plane   6m3s   v1.30.1
```

## 添加 worker node
kube-node01、kube-node02、kube-node03
```bash
kubeadm join kube-master01:6443 --token kbai6s.9wvsc3ioui5p1lme \
        --discovery-token-ca-cert-hash sha256:16337a9cdca3216dbe056ca498056edd57fcf238a7e5b01c47e8b922ac2081b2 
```

输出：

```bash
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-check] Waiting for a healthy kubelet. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 500.787012ms
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```


效果

```bash
$ kubectl get node
NAME            STATUS   ROLES           AGE     VERSION
kube-master01   Ready    control-plane   15m     v1.30.1
kube-node01     Ready    <none>          2m2s    v1.30.1
kube-node02     Ready    <none>          2m16s   v1.30.1
kube-node03     Ready    <none>          88s     v1.30.1
```


## 设置角色

```bash
kubectl label nodes kube-node01 node-role.kubernetes.io/node=
kubectl label nodes kube-node02 node-role.kubernetes.io/node=
kubectl label nodes kube-node03 node-role.kubernetes.io/node=
```


参考：

- [https://kubernetes.io/docs/concepts/cluster-administration/addons/](https://kubernetes.io/docs/concepts/cluster-administration/addons/)
- [https://kubernetes.io/zh-cn/docs/reference/setup-tools/kubeadm/](https://kubernetes.io/zh-cn/docs/reference/setup-tools/kubeadm/)
