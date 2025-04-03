![](https://i-blog.csdnimg.cn/blog_migrate/2fa42a106a3038418cf3a249cbd73ddd.jpeg#pic_center)



## 1. 简介
二进制部署 Kubernetes 是一种将 Kubernetes 组件以二进制文件的形式部署到服务器上的方法。与使用预构建的发行版（如Kubernetes发行版或云提供商的托管服务）相比，二进制部署提供了更大的灵活性和定制性。

优势：
1. 灵活性和定制性：二进制部署提供了更大的灵活性，允许您自定义和配置每个组件的行为和参数。您可以选择要安装的特定版本，并根据需要启用或禁用特定的功能。

2. 最新功能和修复：通过二进制部署，您可以更快地获得最新的 Kubernetes 功能和修复。由于您直接从官方源获取二进制文件，可以更容易地升级到新版本，并及时获得新功能和安全修复。

3. 跨平台兼容性：二进制部署可以在各种操作系统和环境中进行，而不受特定发行版的限制。这使得它适用于多样化的部署需求和环境。

4. 深入了解 Kubernetes 内部工作原理：通过手动安装和配置每个组件，您将更深入地了解 Kubernetes 的内部工作原理和组件之间的相互关系。这对于那些希望深入学习和理解 Kubernetes 的用户来说是一个优势。

劣势：
5. 复杂性和技术要求：二进制部署 Kubernetes 需要更多的手动配置和管理工作。它对于操作系统、网络和安全等方面的知识要求较高。对于没有足够经验或专业知识的用户来说，可能会面临一些挑战。

6. 维护和升级：由于您需要手动管理每个组件的安装和配置，因此维护和升级工作可能会更加繁琐。您需要定期检查官方发布并手动升级组件，以确保安全性和最新功能。

7. 缺少集成工具：与使用发行版或托管服务相比，二进制部署可能缺少一些集成工具和自动化功能。您可能需要自己设置监控、日志收集和自动扩展等功能。

8. 可移植性和一致性：由于二进制部署的配置和管理是手动的，不同环境之间的一致性和可移植性可能会受到影响。在切换到新的环境或扩展到多个集群时，需要更多的努力来保持一致性。

## 架构

为了保障承载在容器集群上面的业务系统并提供高可用支撑能力，本方案规划1套主备K8S容器集群，实现两套K8S集群高可用负载。具体的架构图如下：

![](https://i-blog.csdnimg.cn/blog_migrate/fd6d82fe2a1dec08e22db0a4e970c69a.png)

从架构图上面来看，采用两套K8S集群的目的是实现K8S集群高可用，同时也是为业务提供更可靠的服务，并分流50%的流量到备用集群。一旦主K8S集群环境故障导致无法提供集群服务，随之前端LB负载会自动将集群业务切换到备用K8S集群。改造主备集群好处是能缩短故障业务恢复时间，为用户快速提供服务，减少故障带来人力成本。

## 2. 软件版本

| 软件             | 版本           |
|----------------|--------------|
| runc           | 1.1.10       |
| containerd     | 1.7.11       |
| nerdctl        | 1.7.1        |
| crictl         | 1.28.0       |
| cni            | 1.3.0        |
| dashboard      | 3.0.0-alpha0 |
| metrics-server | 0.7.0        |
| helm           | 3.14.2       |
| etcd           | 3.5.10       |
| kubernetes     | 1.28.7       |
| cfssl          | 1.6.4        |
| docker         | 25.0.3       |
| calico         | 3.26.4       |


跟踪软件版本：

·       runc: https://github.com/opencontainers/runc/releases

·       nerdctl: https://github.com/kubernetes-sigs/cri-tools/releases/

·       crcitl：https://github.com/kubernetes-sigs/cri-tools/releases

·       metric-server：https://github.com/kubernetes-sigs/metrics-server/releases/

·       docker: https://download.docker.com/linux/static/stable/x86_64/

·       kubernetes: https://github.com/kubernetes/kubernetes/releases

·       etcd: https://github.com/etcd-io/etcd/releases

·       helm: https://github.com/helm/helm/releases

·       containerd: https://github.com/containerd/containerd/releases

·       cni: https://github.com/containernetworking/plugins/releases/

 
## 3. 预备条件

### 资源规划


| 角色 | ip  | cpu| mem| disk|ios|kernel|
|--|--|--|--|--|--|--|
| download01 |192.168.23.40 |8  |8G|100G|CentOS 7.9|4.18+|
| kube-master01 |192.168.23.41 |8  |8G|100G|centos 7.9|4.18+|
| kube-master02 |192.168.23.42 |8  |8G|100G|CentOS 7.9|4.18+|
| kube-master03 |192.168.23.43 |8  |8G|100G|CentOS 7.9|4.18+|
| kube-node01 |192.168.23.44 |16  |32G|100G,200G |CentOS 7.9|4.18+|
| kube-node02 |192.168.23.45 |16 |32G|100G,200G|CentOS 7.9|4.18+|
| kube-node03 |192.168.23.46 |16  |32G|100G,200G|CentOS 7.9|4.18+|
| nginx01 |192.168.23.47 |8  |8G|100G|centos 7.9|4.18+|
| nginx02 |192.168.23.48 |8  |8G|100G|CentOS 7.9|4.18+|
| bastion01 |192.168.23.49 |8  |8G|100G|centos 7.9|4.18+|
| vip |192.168.23.50 |  |||||
| harbor01 |192.168.23.51 |8  |8G|100G|centos 7.9|4.18+|
| harbor02 |192.168.23.52 |8  |8G|100G|CentOS 7.9|4.18+|

### 部署规划

| 角色            | IP            | 组件                                                                                                        |
|---------------|---------------|-----------------------------------------------------------------------------------------------------------|
| download01    | 192.168.23.40 | docker                                                                                                    |
| kube-master01 | 192.168.23.41 | kube-apiserver、kube-controller-manager、kube-scheduler、kubelet、kube-proxy、containerd，etcd                  |
| kube-master02 | 192.168.23.42 | kube-apiserver、kube-controller-manager、kube-scheduler、kubelet、kube-proxy、containerd，etcd、keepalived、nginx |
| kube-master03 | 192.168.23.43 | kube-apiserver、kube-controller-manager、kube-scheduler、kubelet、kube-proxy、containerd，etcd、keepalived、nginx |
| kube-node01   | 192.168.23.44 | kubelet、kube-proxy、containerd                                                                             |
| kube-node02   | 192.168.23.45 | kubelet、kube-proxy、containerd                                                                             |
| kube-node03   | 192.168.23.46 | kubelet、kube-proxy、containerd                                                                             |
| nginx01       | 192.168.23.47 | keepalived、nginx                                                                                          |
| nginx02       | 192.168.23.48 | keepalived、nginx                                                                                          |
| bastion01     | 192.168.23.49 | docker\nerdctl\podman(可选)                                                                                 |
| harbor01      | 192.168.23.51 | Harbor                                                                                                    |
| harbor02      | 192.168.23.52 | Harbor                                                                                                    |

> 注意：如果资源紧张、仅用于测试研发等原因，bastion01作为部署机器的步骤也可以在kube-master01进行，但一般生产环境不建议这样。

### 网段规划

| 类型         | 网段             |
|------------|----------------|
| 主机网段       | 192.168.0.0/24 |
| pod 网段     | 10.0.0.0/16    |
| Service 网段 | 10.255.0.0/16  |

### 3.1 安装操作系统
- [安装 rocky linux](https://blog.csdn.net/xixihahalelehehe/article/details/129644123)

- 配置ip

- 配置 DNS

- 网络策略规划

包含配置ip地址与私有环境下的 DNS

### 3.2 检查内核
- [Linux CentOS7.x 升级内核的方法](https://ghostwritten.blog.csdn.net/article/details/121618058)

###  3.3 配置主机名
对应的机器操作
```bash
hostnamectl set-hostname  kube-master01
hostnamectl set-hostname  kube-master02
hostnamectl set-hostname  kube-master03
hostnamectl set-hostname  kube-node01
hostnamectl set-hostname  kube-node02
hostnamectl set-hostname  kube-node03
```
### 3.4 配置互信

如果你知道密码
```bash
ssh-keygen
for i in {40..50};do ssh-copy-id root@192.168.23.$i;done
```
（客户不提供、不知道、没权限）如果你不知道密码，拷贝`id_rsa.pub`内容追加至所有节点的`authorized_keys`文件

```bash
$ ls /root/.ssh/
authorized_keys  id_rsa  id_rsa.pub
$ cat /root/.ssh/id_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDGUGl3/T543lqHhiHT7kuxOjgo1ZNtkWGyWfpyT9ZTxYIgOcdhpsNoHQZ1HTl+tA02HXbNba6/9DA2Muhm3E/cC/un0IhLAFHmsgk/ztmiLqEOwEJAFAOiM0nMm6Up3jmTQlx40jKvvQE2KJiAg8ySdtxemEchOlggCB138Dn/p4Sjq4AKLFXD/s7MQm1r5UHbPYcyirKL+rvk8BMUlJ8uJ1IeUbKNN0rE20ZAsl6jPke8br9+pu8neGwJIYq/PVMXFkQ8nIXWmeIWrXUaI3S5xG/HpLW8sPX6/BjyaJpznZhWoo8jKe1FqyvapiOYQj3WgBmUQL0aO4DWszlSY7fJ root@kube-master01
```

### 3.5 配置 yum（可选）
> 注意：如已有yum源，忽略即可。

- [https://blog.csdn.net/xixihahalelehehe/article/details/133931105](https://blog.csdn.net/xixihahalelehehe/article/details/133931105)
- [linux yum 软件包管理](https://blog.csdn.net/xixihahalelehehe/article/details/105625395)


### 3.6 配置 NFS（可选）

安装 nfs
共享介质其他节点，方便其他节点安装

```bash
yum -y install nfs-utils
cat <<EOF> /etc/exports
/root/k8s-offline-binary-install *(rw,no_root_squash)
EOF
systemctl start nfs-server && systemctl enable nfs-server
exportfs -rv
ansible all -m shell -a "yum -y install nfs-utils" --limit 'all:!kube-master01'
ansible all -m shell -a "showmount -e $IP1" --limit 'all:!kube-master01'
ansible all -m shell -a "mkdir /opt/k8s" --limit 'all:!kube-master01'
ansible all -m shell -a "mount -t nfs $IP1:/root/k8s-offline-binary-install /opt/k8s" --limit 'all:!kube-master01'
ansible all -m shell -a "df -Th" --limit 'all:!kube-master01'

```

## 4 下载介质
download01操作

### 4.1 系统配置代理
安装git

```bash
yum -y install git
```
配置代理
$ vim /root/.bashrc
```bash
proxy_url="http://192.168.21.101:7890"
export no_proxy="192.168.21.2,10.0.0.0/8,192.168.0.0/16,localhost,127.0.0.0/8,.coding.net,.tencentyun.com,.myqcloud.com"
# proxy settings
enable_proxy() {
    export http_proxy="${proxy_url}"
    export https_proxy="${proxy_url}"
    git config --global http.proxy "${proxy_url}"
    git config --global http.proxy "${proxy_url}"
}

disable_proxy() {
    unset http_proxy
    unset https_proxy
    git config --global --unset http.proxy
    git config --global --unset https.proxy
}

disable_proxy
#enable_proxy
```

生效

```bash
source /root/.bashrc
enabl_proxy
```

### 安装 docker

> RHEL or ROCKY 可直接使用podman，无需安装docker。

- [docker 安装](https://blog.csdn.net/xixihahalelehehe/article/details/104293170)

```bash
yum install -y yum-utils 
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum list docker-ce --showduplicates | sort -r
yum -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

```

### 容器配置代理

- [docker proxy](https://blog.csdn.net/xixihahalelehehe/article/details/134115905)
- [podman proxy](https://blog.csdn.net/xixihahalelehehe/article/details/136683723)

### 4.1 下载 k8s 介质
```bash
#!/bin/bash


name=`basename $0 .sh`
ENABLE_DOWNLOAD=${ENABLE_DOWNLOAD:-true}
BASE_DIR="$( dirname "$( readlink -f "${0}" )" )"
REGISTRY='registry.demo'


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
    RUNC_VERSION=1.1.10
    CONTAINERD_VERSION=1.7.11
    NERDCTL_VERSION=1.7.1
    CRICTL_VERSION=1.28.0
    CNI_VERSION=1.3.0
    DASHBOARD_VERSION=3.0.0-alpha0
    METRICS_VERSION=0.7.0
    HELM_VERSION=3.14.2
    ETCD_VERSION=3.5.10
    KUBERNETES_VERSION=1.28.7
    CFSSL_VERSION=1.6.4
    CALICO_VERSION=3.26.4
    DOCKER_VERSION=25.0.4

    download https://github.com/opencontainers/runc/releases/download/v${RUNC_VERSION}/runc.amd64 runc/v${RUNC_VERSION}
    download https://github.com/containerd/containerd/releases/download/v${CONTAINERD_VERSION}/containerd-${CONTAINERD_VERSION}-linux-amd64.tar.gz
    download https://github.com/containerd/nerdctl/releases/download/v${NERDCTL_VERSION}/nerdctl-${NERDCTL_VERSION}-linux-amd64.tar.gz
    download https://github.com/kubernetes-sigs/cri-tools/releases/download/${CRICTL_VERSION}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz
    download https://github.com/containernetworking/plugins/releases/download/v${CNI_VERSION}/cni-plugins-linux-amd64-v${CNI_VERSION}.tgz kubernetes/cni
    download https://raw.githubusercontent.com/kubernetes/dashboard/v${DASHBOARD_VERSION}/charts/kubernetes-dashboard.yaml kubernetes/dashboard
    download https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/baremetal/deploy.yaml kubernetes/ingress-nginx
    download https://raw.githubusercontent.com/coredns/deployment/master/kubernetes/coredns.yaml.sed kubernetes/coredns
    download https://github.com/kubernetes-sigs/metrics-server/releases/download/v${METRICS_VERSION}/components.yaml kubernetes/metrics-server
    download https://get.helm.sh/helm-v${HELM_VERSION}-linux-amd64.tar.gz 
    download https://github.com/etcd-io/etcd/releases/download/v${ETCD_VERSION}/etcd-v${ETCD_VERSION}-linux-amd64.tar.gz 
    download https://dl.k8s.io/v${KUBERNETES_VERSION}/kubernetes-server-linux-amd64.tar.gz kubernetes
    download https://github.com/cloudflare/cfssl/releases/download/v${CFSSL_VERSION}/cfssl_${CFSSL_VERSION}_linux_amd64 cfssl
    download https://github.com/cloudflare/cfssl/releases/download/v${CFSSL_VERSION}/cfssljson_${CFSSL_VERSION}_linux_amd64 cfssl
    download https://github.com/cloudflare/cfssl/releases/download/v${CFSSL_VERSION}/cfssl-certinfo_${CFSSL_VERSION}_linux_amd64 cfssl
    download https://github.com/projectcalico/calico/releases/download/v${CALICO_VERSION}/calicoctl-linux-amd64 kubernetes/calico
    download https://github.com/projectcalico/calico/archive/v${CALICO_VERSION}.tar.gz kubernetes/calico
    download https://download.docker.com/linux/static/stable/x86_64/docker-${DOCKER_VERSION}.tgz




    
else
    FILES_DIR=./files
fi

}

download_images() {

    mkdir -p ${IMAGES_DIR}
    cp ${BASE_DIR}/images.sh  ${IMAGES_DIR}
    cp ${BASE_DIR}/images.txt ${IMAGES_DIR}

    for image in `cat ${BASE_DIR}/images.txt`;do
       image_name=`echo $image | awk -F ':' '{print $1}' | sed 's/\//\_/g'`
       image_version=`echo $image | awk -F ':' '{print $2}'`
       if !  $docker pull $image;then
         $docker pull $image
       fi
       $docker save -o ${RELEASE_DIR}/images/${image_name}_${image_version}.tar $image
    done

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

cat > /etc/systemd/system/containerd.service<<EOF
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
 
sed -i "s#registry.k8s.io/pause:3.8#${REGISTRY}/k8s/pause:3.9#g" /etc/containerd/config.toml
cat /etc/containerd/config.toml | grep sandbox_image


local config='/etc/containerd/config.toml'
if grep -q "registry.mirrors]" $config;then
    registry_mirrors=$(grep "registry.mirrors]" $config | awk '{print $1}' |  sed 's/\[/\\\[/g;s/\]/\\\]/g;s/\./\\\./g;s/\"/\\\"/g' )
    sudo sed -i "/${registry_mirrors}/ a\ \ \ \ \ \ \ \ \[plugins.\"io.containerd.grpc.v1.cri\".registry.mirrors.\"${REGISTRY}\"]\n\ \ \ \ \ \ \ \ \ \ endpoint = [\"http://${REGISTRY}\"]" $config
else
    sudo sed -i '/\[plugins\.\"io\.containerd\.grpc\.v1\.cri\"\.registry\]/ a\ \ \ \ \ \ [plugins.\"io.containerd.grpc.v1.cri\".registry.mirrors]\n\ \ \ \ \ \ \ \ [plugins.\"io.containerd.grpc.v1.cri\".registry.mirrors.\"${REGISTRY}\"]\n\ \ \ \ \ \ \ \ \ \ endpoint = [\"http://${REGISTRY}\"]' $config
fi

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
    #download_images
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
下载

```bash
sh download.sh d
```


### 4.2 下载镜像

- image.sh - [容器镜像搬运最佳脚本](https://blog.csdn.net/xixihahalelehehe/article/details/135082812)

`images.sh` 镜像列表：

```bash
quay.io/kubespray/kubespray:v2.24.1
registry.k8s.io/pause:3.9
docker.io/calico/cni:v3.26.4
docker.io/calico/node:v3.26.4
docker.io/calico/kube-controllers:v3.26.4
docker.io/coredns/coredns:1.11.1
registry.k8s.io/ingress-nginx/controller:v1.10.0
registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.4.0
nginx:1.25.3
registry.k8s.io/metrics-server/metrics-server:v0.7.0
```
公网拉取镜像

```bash
sh image.sh pull 
```

打包

```bash
sh images.sh save
```

### 4.3 介质打包


介质列表：

```bash
k8s-offline-binary-install/
├── check-etcd.sh
├── copy-master.sh
├── copy-node.sh
├── create-ca.sh
├── download.sh
├── files
│   ├── cfssl
│   │   ├── cfssl_1.6.4_linux_amd64
│   │   ├── cfssl-certinfo_1.6.4_linux_amd64
│   │   └── cfssljson_1.6.4_linux_amd64
│   ├── containerd-1.7.11-linux-amd64.tar.gz
│   ├── containerd-1.7.5-linux-amd64.tar.gz
│   ├── crictl-1.28.0-linux-amd64.tar.gz
│   ├── crictl-v1.28.0-linux-amd64.tar.gz
│   ├── etcd-v3.5.10-linux-amd64.tar.gz
│   ├── helm-v3.14.2-linux-amd64.tar.gz
│   ├── kubernetes
│   │   ├── calico
│   │   │   ├── calico-etcd.yaml
│   │   │   └── v3.26.4.tar.gz
│   │   ├── cni
│   │   │   └── cni-plugins-linux-amd64-v1.3.0.tgz
│   │   ├── coredns
│   │   │   ├── coredns.yaml
│   │   │   └── coredns.yaml.sed
│   │   ├── dashboard
│   │   │   └── kubernetes-dashboard.yaml
│   │   ├── ingress-nginx
│   │   │   ├── deploy.yaml
│   │   │   ├── deploy.yaml_template
│   │   │   └── nginx-demo.yaml
│   │   ├── kubernetes-server-linux-amd64.tar.gz
│   │   └── metrics-server
│   │       ├── components.yaml
│   │       └── components.yaml_template
│   ├── nerdctl-1.7.1-linux-amd64.tar.gz
│   └── runc
│       └── v1.1.10
│           └── runc.amd64
├── hosts
├── images
│   ├── images
│   │   ├── docker.io_cni_v3.26.4.tar
│   │   ├── docker.io_coredns_1.11.1.tar
│   │   ├── docker.io_kube-controllers_v3.26.4.tar
│   │   ├── docker.io_node_v3.26.4.tar
│   │   ├── kubespray_v2.24.1.tar
│   │   ├── nginx:1.25.3_nginx_1.25.3.tar
│   │   ├── registry.k8s.io_controller_v1.10.0.tar
│   │   ├── registry.k8s.io_kube-webhook-certgen_v1.4.0.tar
│   │   ├── registry.k8s.io_metrics-server_v0.7.0.tar
│   │   └── registry.k8s.io_pause_3.9.tar
│   ├── images.sh
│   └── images.txt
├── init-master.sh
├── init-node.sh
├── install-apiserver.sh
├── install-etcd.sh
├── install-k8s-command.sh
├── install-kube-controller-manager.sh
├── install-kubectl.sh
├── install-kubelet.sh
├── install-kube-proxy.sh
├── install-scheduler.sh
├── inventory.ini
├── ipvs.modules
└── sync-etcd.sh

13 directories, 54 files
```
打包

```bash
tar zcvf k8s-offline-binary-install.tar.gz k8s-offline-binary-install/
```

## 5. 安装镜像仓库(可选)

- harbor：[centos 7.9 部署 harbor 镜像仓库实践](https://blog.csdn.net/xixihahalelehehe/article/details/127920005)
- registry：[Centos 7.9 Install Docker Insecure Registry](https://blog.csdn.net/xixihahalelehehe/article/details/134633564)

## 6. 镜像入库



搬运至离线环境
解压

```bash
sh images.sh load 
```
修改 images.sh 定制仓库名、项目名、容器工具、镜像列表名

```bash
registry_name='harbor.ghostwritten.com'
project='k8s'
docker=/usr/bin/nerdctl
images_list='images.txt'
```
推送入库

```bash
sh images.sh push
```




## 7. 安装容器工具

（bastion01操作 ）
>注意：如果环境中没有bastion01，kube-master01可进行安装部署操作。

- [安装 docker](https://ghostwritten.blog.csdn.net/article/details/104293170)
- [安装 podman](https://blog.csdn.net/xixihahalelehehe/article/details/127953530)
- [安装 nerdctl](https://blog.csdn.net/xixihahalelehehe/article/details/134264754)

这里我选择 nerdctl，尽管docker与podman安装更简单一些，但选择nerdctl是为了避免容器工具的复用，因为该kubernetes集群的容器工具同样也是nerdctl。
安装

```bash
sh download i
```
检查

```bash
containerd --version
crictl version
nerdctl version
```



## 8. 安装 ansible容器
（bastion01操作 ）

下载打包

```bash
nerdctl pull quay.io/kubespray/kubespray:v2.24.1
nerdctl save -o quay.io_kubespray_kubespray_v2.24.1.tar quay.io/kubespray/kubespray:v2.24.1
nerdctl load -i quay.io_kubespray_kubespray_v2.24.1.tar 
```
运行kubespray
```bash

nerdctl  run --name ansible --network=host  -it  -v "${HOME}"/.ssh:/root/.ssh  -v /etc/hosts:/etc/hosts quay.io/kubespray/kubespray:v2.24.1  bash

挂载介质l
nerdctl  run --name ansible --network=host  -ti -v "$(pwd)"/k8s-offline-binary-install:/kubespray -v "${HOME}"/.ssh:/root/.ssh  -v /etc/hosts:/etc/hosts quay.io/kubespray/kubespray:v2.24.1  bash
```
配置 inventry.ini

```bash
$ docker ps
$ nerdctl exec -ti ansible bash
root@383d20357b5c:/kubespray# vim inventory/sample/inventory.ini
[all]
kube-master01 ansible_host=192.168.23.41 ip=192.168.23.41 etcd_member_name=etcd1
kube-master02 ansible_host=192.168.23.42 ip=192.168.23.42 etcd_member_name=etcd2
kube-master03 ansible_host=192.168.23.43 ip=192.168.23.43 etcd_member_name=etcd3
kube-node01 ansible_host=192.168.23.44 ip=192.168.23.44
kube-node02 ansible_host=192.168.23.45 ip=192.168.23.45

[kube_control_plane]
kube-master01
kube-master02
kube-master03

[etcd]
kube-master01
kube-master02
kube-master03

[kube_node]
kube-node01
kube-node02
```

测试连通性

```bash
root@383d20357b5c:/kubespray# ansible -i inventory/sample/inventory.ini all -m ping
[WARNING]: Skipping callback plugin 'ara_default', unable to load
kube-master02 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
kube-node01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
kube-master03 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
kube-master01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
kube-node02 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

添加别名

```bash
root@383d20357b5c:/kubespray# vim /root/.bashrc 
....
alias ansible='ansible -i inventory/sample/inventory.ini'
....
root@383d20357b5c:/kubespray# source /root/.bashrc
root@383d20357b5c:/kubespray# ansible all -m ping
```

## 9. 批量配置

### 9.1  配置/etc/hosts

```bash
$ cat hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.23.41 kube-master01
192.168.23.42 kube-master02
192.168.23.43 kube-master03
192.168.23.44 kube-node01
192.168.23.45 kube-node02
$ ansible all -m copy -a "src=hosts dest=/etc/"
```


###  9.2 关闭防火墙、swap、selinux


```bash
ansible all -m systemd -a "name=firewalld state=stopped enabled=no"
ansible all -m lineinfile -a "path=/etc/selinux/config regexp='^SELINUX=' line='SELINUX=disabled'" -b
ansible all -m shell -a "getenforce 0"
ansible all -m shell -a "sed -i '/.*swap.*/s/^/#/' /etc/fstab" -b
ansible all -m shell -a " swapoff -a && sysctl -w vm.swappiness=0"
```

### 9.3 配置系统文件句柄数

```bash
ansible all -m lineinfile -a "path=/etc/security/limits.conf line='* soft nofile 655360\n* hard nofile 131072\n* soft nproc 655350\n* hard nproc 655350\n* soft memlock unlimited\n* hard memlock unlimited'" -b
```

### 9.4 启用ipvs

```bash
cat > ipvs.modules <<EOF
#!/bin/bash
ipvs_modules="ip_vs ip_vs_lc ip_vs_wlc ip_vs_rr ip_vs_wrr ip_vs_lblc ip_vs_lblcr ip_vs_dh ip_vs_sh ip_vs_nq ip_vs_sed ip_vs_ftp nf_conntrack"
for kernel_module in${ipvs_modules}; do
 /sbin/modinfo -F filename ${kernel_module} > /dev/null 2>&1
 if [ 0 -eq 0 ]; then
 /sbin/modprobe ${kernel_module}
 fi
done
EOF

ansible all -m copy -a "src=ipvs.modules dest=/etc/sysconfig/modules/ipvs.modules"
ansible all -m shell -a "chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep ip_vs"
```

### 9.5 配置内核参数

```bash
ansible all -m shell -a " modprobe bridge && modprobe br_netfilter && modprobe ip_conntrack"
ansible all -m lineinfile -a "path=/etc/rc.local line='modprobe br_netfilter\nmodprobe ip_conntrack'" -b
ansible all -m file -a "path=/etc/sysctl.d/k8s.conf state=touch mode=0644"
ansible all -m blockinfile -a "path=/etc/sysctl.d/k8s.conf block='net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
kernel.pid_max = 99999
vm.max_map_count = 262144'"
ansible all -m shell -a " sysctl -p /etc/sysctl.d/k8s.conf"
```

### 9.6 配置 yum

假如yum名称为yum.repo，拷贝配置yum.repo容器内。
```bash
cat > yum.repo <<EOF
[BaseOS] 
name=http_iso 
baseurl=http://192.168.23.40/BaseOS
gpgcheck=0 
enabled=1
[AppStream] 
name=http_iso 
baseurl=http://192.168.23.40/AppStream
gpgcheck=0 
enabled=1
EOF

ansible all -m shell -a  "mv /etc/yum.repos.d /tmp"
ansible all -m shell -a "mkdir -p /etc/yum.repos.d"
ansible all -m copy -a " src=yum.repo dest=/etc/yum.repos.d/"
ansible all -m shell -a  "yum clean all && yum repolist"

```

### 9.7 配置时间同步
>注意：时间配置内部NTP服务器，需要提供NTP地址。

```bash
ansible all -m shell -a "yum install -y htop tree wget jq git net-tools ntpdate"
ansible all -m shell -a "timedatectl set-timezone Asia/Shanghai && date && echo 'Asia/Shanghai' > /etc/timezone"
ansible all -m shell -a "date && ntpdate -u  ntp01.ghostwritten.com.cn && date"
ansible all -m shell -a "echo '0,10,20,30,40,50 * * * * /usr/sbin/ntpdate -u ntp01.ghostwritten.com.cn ' >> /var/spool/cron/root && crontab -l"
ansible all  -m systemd -a "name=crond state=restarted"
ansible all -m shell -a "date"
```
### 9.8 journal 持久化

```bash
ansible all -m shell -a "sed -i 's/#Storage=auto/Storage=auto/g' /etc/systemd/journald.conf && mkdir -p /var/log/journal && systemd-tmpfiles --create --prefix /var/log/journal"
ansible all  -m systemd -a "name=systemd-journald.service state=restarted"
```

### 9.9 配置 history

```bash
ansible all -m shell -a "echo 'export HISTTIMEFORMAT=\"%Y-%m-%d %T \"' >> ~/.bashrc && source ~/.bashrc"
```

### 9.10 安装 containerd

kube-master01已安装，无需拷贝以及安装
```bash
ansible 'all:!kube-master01' -m copy -a "src=nerdctl dest=/tmp/"
```
安装

```bash
 ansible 'all' -m shell -a "mkdir /root/k8s"
 ansible 'all:!kube-master01' -m copy -a "src=download.sh dest=/root/k8s/"
 ansible 'all:!kube-master01' -m copy -a "src=files dest=/root/k8s/"
 ansible 'all:!kube-master01' -m shell  -a "cd  /root/k8s/ && sh download.sh i"
```
查看状态

```bash
$ ansible 'all:!kube-master01' -m shell  -a "systemctl status containerd | grep Active"
kube-master02 | CHANGED | rc=0 >>
   Active: active (running) since Fri 2024-03-01 15:52:37 CST; 3min 55s ago
kube-node01 | CHANGED | rc=0 >>
   Active: active (running) since Fri 2024-03-01 15:52:41 CST; 3min 52s ago
kube-node02 | CHANGED | rc=0 >>
   Active: active (running) since Fri 2024-03-01 15:52:21 CST; 3min 54s ago
kube-master03 | CHANGED | rc=0 >>
   Active: active (running) since Fri 2024-03-01 15:52:39 CST; 3min 53s ago
```


## 10. 安装 etcd
> 注意：etcd集群搭建在主机kube-master01、kube-master02、kube-master03上操作。另外，下面etcd 证书配置、安装操作，仅在kube-master01执行。

安装etcd脚本install-etcd.sh 如下:

```bash
#!/bin/bash

IP1=$1
IP2=$2
IP3=$3

name=`basename $0 .sh`
BASE_DIR="$( dirname "$( readlink -f "${0}" )" )"

rm -rf /etc/etcd
mkdir -p  /etc/etcd/ssl/
mkdir -p /etc/etcd/cfg

cfssl_install() {

cp $BASE_DIR/files/cfssl/cfssl_1.6.4_linux_amd64 /usr/local/bin/cfssl
cp $BASE_DIR/files/cfssl/cfssl-certinfo_1.6.4_linux_amd64 /usr/local/bin/cfssl-certinfo
cp $BASE_DIR/files/cfssl/cfssljson_1.6.4_linux_amd64 /usr/local/bin/cfssljson

chmod -R 755 /usr/local/bin/
echo "export PATH=/usr/local/bin:$PATH" >>/etc/profile

}

etcd_certs() {

cd /etc/etcd/ssl
cat > ca-config.json << EOF
{
  "signing": {
    "default": {
      "expiry": "876000h"
    },
    "profiles": {
      "www": {
         "expiry": "876000h",
         "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ]
      }
    }
  }
}
EOF


cat > ca-csr.json << EOF
{
    "CN": "etcd CA",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Shanghai",
            "ST": "Shanghai"
        }
    ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca

cat > etcd-csr.json << EOF
{
    "CN": "etcd",
    "hosts": [
    "$IP1",
    "$IP2",
    "$IP3"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Shanghai",
            "ST": "Shanghai"
        }
    ]
}
EOF

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=www etcd-csr.json | cfssljson -bare etcd

}

etcd_install() {

cd $BASE_DIR/files
tar zxvf etcd-v3.5.10-linux-amd64.tar.gz
cp -a  etcd-v3.5.10-linux-amd64/{etcd,etcdctl} /usr/local/bin/

cat > /etc/etcd/cfg/etcd.conf << EOF
#[Member]
ETCD_NAME="etcd-1"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="https://$IP1:2380"
ETCD_LISTEN_CLIENT_URLS="https://$IP1:2379"
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://$IP1:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://$IP1:2379"
ETCD_INITIAL_CLUSTER="etcd-1=https://$IP1:2380,etcd-2=https://$IP2:2380,etcd-3=https://$IP3:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
EOF

cat > /etc/etcd/cfg/etcd.conf_etcd2 << EOF
#[Member]
ETCD_NAME="etcd-2"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="https://$IP2:2380"
ETCD_LISTEN_CLIENT_URLS="https://$IP2:2379"
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://$IP2:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://$IP2:2379"
ETCD_INITIAL_CLUSTER="etcd-1=https://$IP1:2380,etcd-2=https://$IP2:2380,etcd-3=https://$IP3:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
EOF

cat > /etc/etcd/cfg/etcd.conf_etcd3 << EOF
#[Member]
ETCD_NAME="etcd-3"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="https://$IP3:2380"
ETCD_LISTEN_CLIENT_URLS="https://$IP3:2379"
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://$IP3:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://$IP3:2379"
ETCD_INITIAL_CLUSTER="etcd-1=https://$IP1:2380,etcd-2=https://$IP2:2380,etcd-3=https://$IP3:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
EOF

cat > /usr/lib/systemd/system/etcd.service << EOF
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target
[Service]
Type=notify
EnvironmentFile=/etc/etcd/cfg/etcd.conf
ExecStart=/usr/local/bin/etcd \
--cert-file=/etc/etcd/ssl/etcd.pem \
--key-file=/etc/etcd/ssl/etcd-key.pem \
--peer-cert-file=/etc/etcd/ssl/etcd.pem \
--peer-key-file=/etc/etcd/ssl/etcd-key.pem \
--trusted-ca-file=/etc/etcd/ssl/ca.pem \
--peer-trusted-ca-file=/etc/etcd/ssl/ca.pem \
--logger=zap
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF

}

cfssl_install
etcd_certs
etcd_install
```

```bash
ansible kube-master01 -m copy -a "src=install-etcd.sh dest=/root/k8s"
ansible kube-master01 -m shell  -a "cd  /root/k8s/ && sh install-etcd.sh"
```

### 10.1 同步文件
同步文件至 kube-master02 与 kube-master03
将master01节点上面的证书与启动配置文件同步到master01~master03节点。

同步脚本sync-etcd.sh如下：
```bash
#!/bin/bash 

IP=$1

scp -r /etc/etcd root@$IP:/etc/
scp /usr/lib/systemd/system/etcd.service root@$IP:/usr/lib/systemd/system/
scp /usr/local/bin/etcd  root@$IP:/usr/local/bin/ 
scp /usr/local/bin/etcdctl  root@$IP:/usr/local/bin/
```
执行：

```bash
ansible kube-master02 -m script -a "sync-etcd.sh 192.168.23.42"
ansible kube-master03 -m script -a "sync-etcd.sh 192.168.23.43"
ansible kube-master02 -m shell -a "mv /etc/etcd/cfg/etcd.conf_etcd2  /etc/etcd/cfg/etcd.conf"
ansible kube-master03 -m shell -a "mv /etc/etcd/cfg/etcd.conf_etcd3  /etc/etcd/cfg/etcd.conf"
```

### 10.2 启动

```bash
ansible etcd  -m shell -a "systemctl daemon-reload && systemctl start  etcd && systemctl enable  etcd && systemctl status  etcd | grep Active"
```
输出：

```bash
kube-master02 | CHANGED | rc=0 >>
   Active: active (running) since Fri 2024-03-01 17:57:21 CST; 305ms agoCreated symlink from /etc/systemd/system/multi-user.target.wants/etcd.service to /usr/lib/systemd/system/etcd.service.
kube-master03 | CHANGED | rc=0 >>
   Active: active (running) since Fri 2024-03-01 17:57:21 CST; 309ms agoCreated symlink from /etc/systemd/system/multi-user.target.wants/etcd.service to /usr/lib/systemd/system/etcd.service.
kube-master01 | CHANGED | rc=0 >>
   Active: active (running) since Fri 2024-03-01 17:57:21 CST; 447ms agoCreated symlink from /etc/systemd/system/multi-user.target.wants/etcd.service to /usr/lib/systemd/system/etcd.service.
```

### 10.3 检查

检查脚本`check-etcd.sh`如下：
```bash
#!/bin/bash

$IP1=$1
$IP2=$2
$IP3=$3

ETCDCTL_API=3 /usr/local/bin/etcdctl --cacert=/etc/etcd/ssl/ca.pem --cert=/etc/etcd/ssl/etcd.pem --key=/etc/etcd/ssl/etcd-key.pem --endpoints="https://$IP1:2379,https://$IP2:2379,https://$IP3:2379" endpoint health
```
执行：

```bash
ansible kube-master01 -m script -a "check-etcd.sh 192.168.23.41 192.168.23.42 192.168.23.43"
```

输出：
```bash
kube-master01 | CHANGED => {
    "changed": true,
    "rc": 0,
    "stderr": "Shared connection to 192.168.23.41 closed.\r\n",
    "stderr_lines": [
        "Shared connection to 192.168.23.41 closed."
    ],
    "stdout": "https://192.168.23.43:2379 is healthy: successfully committed proposal: took = 55.913832ms\r\nhttps://192.168.23.42:2379 is healthy: successfully committed proposal: took = 59.280676ms\r\nhttps://192.168.23.41:2379 is healthy: successfully committed proposal: took = 85.987455ms\r\n",
    "stdout_lines": [
        "https://192.168.23.43:2379 is healthy: successfully committed proposal: took = 55.913832ms",
        "https://192.168.23.42:2379 is healthy: successfully committed proposal: took = 59.280676ms",
        "https://192.168.23.41:2379 is healthy: successfully committed proposal: took = 85.987455ms"
    ]
}
```

## 11. 安装 kubernetes 命令

```bash
$ vim  install-k8s-command.sh
#!/bin/bash

name=`basename $0 .sh`
BASE_DIR="$( dirname "$( readlink -f "${0}" )" )"

mkdir -p /etc/kubernetes/bin /opt/cni/bin
tar zxvf $BASE_DIR/files/kubernetes/kubernetes-server-linux-amd64.tar.gz -C $BASE_DIR/files/kubernetes
cp $BASE_DIR/files/kubernetes/kubernetes/server/bin/{kubectl,kube-apiserver,kube-scheduler,kube-controller-manager,kubelet,kube-proxy} /usr/local/bin/
tar zxvf $BASE_DIR/files/kubernetes/cni/cni-plugins-linux-amd64-v1.3.0.tgz -C /opt/cni/bin

yum install -y bash-completion 
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc



ansible kube-master01 -m copy -a "src=install-k8s-command.sh dest=/root/k8s"
ansible kube-master01 -m shell  -a "cd  /root/k8s/ && sh install-k8s-command.sh"
```



## 12. 创建集群证书
创建证书目录，自签证书颁发机构（CA）
```bash
ansible all -m shell  -a "mkdir -p /etc/kubernetes/ssl /etc/kubernetes/bin /etc/kubernetes/cfg/ /var/log/kubernetes"
ansible kube-master01 -m script -a "create-ca.sh"
```
输出：
```bash
kube-master01 | CHANGED => {
    "changed": true,
    "rc": 0,
    "stderr": "Shared connection to 192.168.23.41 closed.\r\n",
    "stderr_lines": [
        "Shared connection to 192.168.23.41 closed."
    ],
    "stdout": "2024/03/03 14:59:12 [INFO] generating a new CA key and certificate from CSR\r\n2024/03/03 14:59:12 [INFO] generate received request\r\n2024/03/03 14:59:12 [INFO] received CSR\r\n2024/03/03 14:59:12 [INFO] generating key: rsa-2048\r\n2024/03/03 14:59:13 [INFO] encoded CSR\r\n2024/03/03 14:59:13 [INFO] signed certificate with serial number 38714756076862318584151909020577899072505539939\r\n",
    "stdout_lines": [
        "2024/03/03 14:59:12 [INFO] generating a new CA key and certificate from CSR",
        "2024/03/03 14:59:12 [INFO] generate received request",
        "2024/03/03 14:59:12 [INFO] received CSR",
        "2024/03/03 14:59:12 [INFO] generating key: rsa-2048",
        "2024/03/03 14:59:13 [INFO] encoded CSR",
        "2024/03/03 14:59:13 [INFO] signed certificate with serial number 38714756076862318584151909020577899072505539939"
    ]
}
```

## 13. 安装 kube-apiserver

使用自签CA签发kube-apiserver HTTPS证书
#hosts字段中IP为所有集群成员的ip集群内部ip，一个都不能少！为了方便后期扩容可以多写几个预留的IP

install-apiserver.sh 脚本内容如下：

```bash
#!/bin/bash


IP1=$1
IP2=$2
IP3=$3


cd /etc/kubernetes/ssl
cat > kube-apiserver-csr.json << EOF
{
    "CN": "kubernetes",
"hosts": [
        "127.0.0.1",
        "10.0.0.1",
        "10.255.0.1",
        "192.168.23.40",
        "192.168.23.41",
        "192.168.23.42",
        "192.168.23.43",
        "192.168.23.44",
        "192.168.23.45",
        "192.168.23.46",
        "192.168.23.47",
        "192.168.23.48",
        "192.168.23.49",
        "192.168.23.50",
        "kubernetes",
        "kubernetes.default",
        "kubernetes.default.svc",
        "kubernetes.default.svc.cluster",
        "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Shanghai",
            "ST": "Shanghai",
            "O": "k8s",
            "OU": "system"
        }
    ]
}
EOF

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-apiserver-csr.json | cfssljson -bare kube-apiserver
cat > /etc/kubernetes/cfg/token.csv << EOF
$(head -c 16 /dev/urandom | od -An -t x | tr -d ' '),kubelet-bootstrap,10001,"system:kubelet-bootstrap"
EOF

cat > /etc/kubernetes/cfg/kube-apiserver.conf << EOF
KUBE_APISERVER_OPTS="--v=2 \\
--etcd-servers=https://$IP1:2379,https://$IP2:2379,https://$IP3:2379 \\
--bind-address=$IP1 \\
--secure-port=6443 \\
--advertise-address=$IP1 \\
--allow-privileged=true \\
--service-cluster-ip-range=10.255.0.0/16 \\
--enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,ResourceQuota,NodeRestriction \\
--authorization-mode=RBAC,Node \\
--enable-bootstrap-token-auth=true \\
--token-auth-file=/etc/kubernetes/cfg/token.csv \\
--service-node-port-range=30000-61000 \\
--kubelet-client-certificate=/etc/kubernetes/ssl/kube-apiserver.pem \\
--kubelet-client-key=/etc/kubernetes/ssl/kube-apiserver-key.pem \\
--tls-cert-file=/etc/kubernetes/ssl/kube-apiserver.pem  \\
--tls-private-key-file=/etc/kubernetes/ssl/kube-apiserver-key.pem \\
--client-ca-file=/etc/kubernetes/ssl/ca.pem \\
--service-account-signing-key-file=/etc/kubernetes/ssl/ca-key.pem \\
--service-account-key-file=/etc/kubernetes/ssl/ca-key.pem \\
--service-account-issuer=https://kubernetes.default.svc.cluster.local \\
--etcd-cafile=/etc/etcd/ssl/ca.pem \\
--etcd-certfile=/etc/etcd/ssl/etcd.pem \\
--etcd-keyfile=/etc/etcd/ssl/etcd-key.pem \\
--requestheader-client-ca-file=/etc/kubernetes/ssl/ca.pem \\
--proxy-client-cert-file=/etc/kubernetes/ssl/kube-apiserver.pem \\
--proxy-client-key-file=/etc/kubernetes/ssl/kube-apiserver-key.pem \\
--requestheader-allowed-names=kubernetes \\
--requestheader-extra-headers-prefix=X-Remote-Extra- \\
--requestheader-group-headers=X-Remote-Group \\
--requestheader-username-headers=X-Remote-User \\
--enable-aggregator-routing=true \\
--audit-log-maxage=30 \\
--audit-log-maxbackup=3 \\
--audit-log-maxsize=100 \\
--audit-log-path=/var/log/kubernetes/k8s-audit.log"
EOF


cat >/usr/lib/systemd/system/kube-apiserver.service << EOF
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes
[Service]
EnvironmentFile=/etc/kubernetes/cfg/kube-apiserver.conf
ExecStart=/usr/local/bin/kube-apiserver \$KUBE_APISERVER_OPTS
Restart=on-failure
[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload;systemctl start kube-apiserver;systemctl enable kube-apiserver;sleep 2; systemctl status kube-apiserver
```


执行：
```bash
ansible kube-master01 -m script -a "install-apiserver.sh 192.168.23.41 192.168.23.42 192.168.23.43"
```
检查

```bash
ansible kube-master01 -m shell  -a "systemctl status kube-apiserver |grep Active " 
```
输出：

```bash
kube-master01 | CHANGED | rc=0 >>
   Active: active (running) since Sun 2024-03-03 15:33:02 CST; 6min ago
```

## 14. 安装 kubectl 

install-apiserver.sh内容：

```bash
#!/bin/bash


IP=$1
cd /etc/kubernetes/ssl/
cat > admin-csr.json << EOF
{
  "CN": "admin",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Shanghai",
      "L": "Shanghai",
      "O": "system:masters",             
      "OU": "system"
    }
  ]
}
EOF

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes admin-csr.json | cfssljson -bare admin


kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=https://$IP:6443 --kubeconfig=kube.config
kubectl config set-credentials admin --client-certificate=admin.pem --client-key=admin-key.pem --embed-certs=true --kubeconfig=kube.config
kubectl config set-context kubernetes --cluster=kubernetes --user=admin --kubeconfig=kube.config
kubectl config use-context kubernetes --kubeconfig=kube.config
mkdir ~/.kube -p
cp kube.config ~/.kube/config
cp kube.config /etc/kubernetes/cfg/admin.conf
kubectl create clusterrolebinding kube-apiserver:kubelet-apis --clusterrole=system:kubelet-api-admin --user Kubernetes

kubectl cluster-info

kubectl get cs

kubectl get all --all-namespaces
```

执行：
```bash
 ansible kube-master01 -m script -a "install-kubectl.sh 192.168.23.41"
```
输出：

```bash
 "stdout_lines": [
        "2024/03/03 15:47:35 [INFO] generate received request",
        "2024/03/03 15:47:35 [INFO] received CSR",
        "2024/03/03 15:47:35 [INFO] generating key: rsa-2048",
        "2024/03/03 15:47:37 [INFO] encoded CSR",
        "2024/03/03 15:47:37 [INFO] signed certificate with serial number 690190557290777232622287772611980128147142672782",
        "2024/03/03 15:47:37 [WARNING] This certificate lacks a \"hosts\" field. This makes it unsuitable for",
        "websites. For more information see the Baseline Requirements for the Issuance and Management",
        "of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);",
        "specifically, section 10.2.3 (\"Information Requirements\").",
        "Cluster \"kubernetes\" set.",
        "User \"admin\" set.",
        "Context \"kubernetes\" created.",
        "Switched to context \"kubernetes\".",
        "clusterrolebinding.rbac.authorization.k8s.io/kube-apiserver:kubelet-apis created",
        "\u001b[0;32mKubernetes control plane\u001b[0m is running at \u001b[0;33mhttps://192.168.23.41:6443\u001b[0m",
        "",
        "To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.",
        "\u001b[33;1mWarning:\u001b[0m v1 ComponentStatus is deprecated in v1.19+",
        "NAME                 STATUS      MESSAGE                                                                                        ERROR",
        "scheduler            Unhealthy   Get \"https://127.0.0.1:10259/healthz\": dial tcp 127.0.0.1:10259: connect: connection refused   ",
        "controller-manager   Unhealthy   Get \"https://127.0.0.1:10257/healthz\": dial tcp 127.0.0.1:10257: connect: connection refused   ",
        "etcd-0               Healthy     ok                                                                                             ",
        "NAMESPACE   NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE",
        "default     service/kubernetes   ClusterIP   10.255.0.1   <none>        443/TCP   14m"
    ]
```

## 15. 安装 kube-controller-manager 

install-kube-controller-manager.sh内容如下：

```bash
#!/bin/bash


IP=$1

cd /etc/kubernetes/ssl
cat > kube-controller-manager-csr.json << EOF
{
    "CN": "system:kube-controller-manager",
    "hosts": [
        "127.0.0.1",
        "10.0.0.1",
        "10.255.0.1",
        "192.168.23.40",
        "192.168.23.41",
        "192.168.23.42",
        "192.168.23.43",
        "192.168.23.44",
        "192.168.23.45",
        "192.168.23.46",
        "192.168.23.47",
        "192.168.23.48",
        "192.168.23.49",
        "192.168.23.50",
        "kubernetes",
        "kubernetes.default",
        "kubernetes.default.svc",
        "kubernetes.default.svc.cluster",
        "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Shanghai",
            "ST": "Shanghai",
            "O": "system:kube-controller-manager",
            "OU": "system"
        }
    ]
}
EOF

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager

kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=https://$IP:6443 --kubeconfig=kube-controller-manager.kubeconfig
kubectl config set-credentials system:kube-controller-manager --client-certificate=kube-controller-manager.pem --client-key=kube-controller-manager-key.pem --embed-certs=true --kubeconfig=kube-controller-manager.kubeconfig
kubectl config set-context system:kube-controller-manager --cluster=kubernetes --user=system:kube-controller-manager --kubeconfig=kube-controller-manager.kubeconfig
kubectl config use-context system:kube-controller-manager --kubeconfig=kube-controller-manager.kubeconfig


mv /etc/kubernetes/ssl/kube-controller-manager.kubeconfig /etc/kubernetes/cfg/
cat >/etc/kubernetes/cfg/kube-controller-manager.conf << EOF
KUBE_CONTROLLER_MANAGER_OPTS="--bind-address=127.0.0.1 \\
  --kubeconfig=/etc/kubernetes/cfg/kube-controller-manager.kubeconfig \\
  --service-cluster-ip-range=10.255.0.0/16 \\
  --cluster-name=kubernetes \\
  --allocate-node-cidrs=true \\
  --cluster-cidr=10.0.0.0/16 \\
  --leader-elect=true \\
  --feature-gates=RotateKubeletServerCertificate=true \\
  --controllers=*,bootstrapsigner,tokencleaner \\
  --horizontal-pod-autoscaler-sync-period=10s \\
  --use-service-account-credentials=true \\
  --cluster-signing-cert-file=/etc/kubernetes/ssl/ca.pem \\
  --cluster-signing-key-file=/etc/kubernetes/ssl/ca-key.pem  \\
  --root-ca-file=/etc/kubernetes/ssl/ca.pem \\
  --service-account-private-key-file=/etc/kubernetes/ssl/ca-key.pem \\
  --cluster-signing-duration=876000h0m0s  \\
  --v=2"
EOF

cat >/usr/lib/systemd/system/kube-controller-manager.service << EOF
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes
[Service]
EnvironmentFile=/etc/kubernetes/cfg/kube-controller-manager.conf
ExecStart=/usr/local/bin/kube-controller-manager \$KUBE_CONTROLLER_MANAGER_OPTS
Restart=on-failure
[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload;systemctl restart kube-controller-manager;systemctl enable kube-controller-manager; systemctl status kube-controller-manager | grep Active
root@2f1048815cef:/kubespray# 
```


执行：
```bash
ansible kube-master01 -m script -a "install-kube-controller-manager.sh  192.168.23.41"
```
输出：

```bash
"stdout_lines": [
        "2024/03/03 16:58:40 [INFO] generate received request",
        "2024/03/03 16:58:40 [INFO] received CSR",
        "2024/03/03 16:58:40 [INFO] generating key: rsa-2048",
        "2024/03/03 16:58:41 [INFO] encoded CSR",
        "2024/03/03 16:58:41 [INFO] signed certificate with serial number 487654030328101131700837969707467782255048544201",
        "Cluster \"kubernetes\" set.",
        "User \"system:kube-controller-manager\" set.",
        "Context \"system:kube-controller-manager\" created.",
        "Switched to context \"system:kube-controller-manager\".",
        "Created symlink from /etc/systemd/system/multi-user.target.wants/kube-controller-manager.service to /usr/lib/systemd/system/kube-controller-manager.service.",
        "   Active: active (running) since Sun 2024-03-03 16:58:42 CST; 492ms ago"
    ]
```

## 16. 安装 kube-scheduler

install-scheduler.sh 内容如下：

```bash
#!/bin/bash

IP=$1

cd /etc/kubernetes/ssl
cat > kube-scheduler-csr.json << EOF
{
    "CN": "system:kube-scheduler",
    "hosts": [
        "127.0.0.1",
        "10.0.0.1",
        "10.255.0.1",
        "192.168.23.40",
        "192.168.23.41",
        "192.168.23.42",
        "192.168.23.43",
        "192.168.23.44",
        "192.168.23.45",
        "192.168.23.46",
        "192.168.23.47",
        "192.168.23.48",
        "192.168.23.49",
        "192.168.23.50",
        "kubernetes",
        "kubernetes.default",
        "kubernetes.default.svc",
        "kubernetes.default.svc.cluster",
        "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Shanghai",
            "ST": "Shanghai",
            "O": "system:kube-scheduler",
            "OU": "system"
        }
    ]
}
EOF

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-scheduler-csr.json | cfssljson -bare kube-scheduler

kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=https://$IP:6443 --kubeconfig=kube-scheduler.kubeconfig
kubectl config set-credentials system:kube-scheduler --client-certificate=kube-scheduler.pem --client-key=kube-scheduler-key.pem --embed-certs=true --kubeconfig=kube-scheduler.kubeconfig
kubectl config set-context system:kube-scheduler --cluster=kubernetes --user=system:kube-scheduler --kubeconfig=kube-scheduler.kubeconfig
kubectl config use-context system:kube-scheduler --kubeconfig=kube-scheduler.kubeconfig

mv /etc/kubernetes/ssl/kube-scheduler.kubeconfig /etc/kubernetes/cfg/
cat >/etc/kubernetes/cfg/kube-scheduler.conf << EOF
KUBE_SCHEDULER_OPTS="--v=2 \\
--kubeconfig=/etc/kubernetes/cfg/kube-scheduler.kubeconfig \\
--leader-elect \\
--bind-address=127.0.0.1"
EOF


cat > /usr/lib/systemd/system/kube-scheduler.service << EOF
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes
[Service]
EnvironmentFile=/etc/kubernetes/cfg/kube-scheduler.conf
ExecStart=/usr/local/bin/kube-scheduler \$KUBE_SCHEDULER_OPTS
Restart=on-failure
[Install]
WantedBy=multi-user.target
EOF


systemctl daemon-reload;systemctl restart kube-scheduler;systemctl enable kube-scheduler;systemctl status kube-scheduler | grep Active
```

执行：

```bash
ansible kube-master01 -m script -a "install-scheduler.sh  192.168.23.41"
```
输出：

```bash
 "stdout_lines": [
        "2024/03/03 17:04:48 [INFO] generate received request",
        "2024/03/03 17:04:48 [INFO] received CSR",
        "2024/03/03 17:04:48 [INFO] generating key: rsa-2048",
        "2024/03/03 17:04:49 [INFO] encoded CSR",
        "2024/03/03 17:04:49 [INFO] signed certificate with serial number 152914511815269850208667353561221642449733417243",
        "Cluster \"kubernetes\" set.",
        "User \"system:kube-scheduler\" set.",
        "Context \"system:kube-scheduler\" created.",
        "Switched to context \"system:kube-scheduler\".",
        "Created symlink from /etc/systemd/system/multi-user.target.wants/kube-scheduler.service to /usr/lib/systemd/system/kube-scheduler.service.",
        "   Active: active (running) since Sun 2024-03-03 17:04:50 CST; 383ms ago"
    ]
```
检查集群状态

```bash
ansible kube-master01 -m shell -a "kubectl get  cs"
```
输出：

```bash
kube-master01 | CHANGED | rc=0 >>
NAME                 STATUS    MESSAGE   ERROR
scheduler            Healthy   ok        
controller-manager   Healthy   ok        
etcd-0               Healthy   ok        Warning: v1 ComponentStatus is deprecated in v1.19+
```


## 17. 安装 kubelet

install-kubelet.sh 脚本内容如下：

```bash
!/bin/bash

IP=$1

KUBE_APISERVER="https://$1:6443" 
TOKEN=$(awk -F "," '{print $1}' /etc/kubernetes/cfg/token.csv)

cat > /etc/kubernetes/cfg/kubelet.conf << EOF
KUBELET_OPTS="--v=2 \\
--hostname-override=kube-master01 \\
--kubeconfig=/etc/kubernetes/cfg/kubelet.kubeconfig \\
--bootstrap-kubeconfig=/etc/kubernetes/cfg/bootstrap.kubeconfig \\
--config=/etc/kubernetes/cfg/kubelet-config.yml \\
--cert-dir=/etc/kubernetes/ssl \\
--runtime-request-timeout=15m  \\
--container-runtime-endpoint=unix:///run/containerd/containerd.sock \\
--cgroup-driver=systemd \\
--node-labels=node.kubernetes.io/node=''"
EOF

cat > /etc/kubernetes/cfg/kubelet-config.yml << EOF
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
address: 0.0.0.0
port: 10250
readOnlyPort: 10255
cgroupDriver: systemd
clusterDNS:
- 10.255.0.2
clusterDomain: cluster.local 
failSwapOn: false
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 2m0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/ssl/ca.pem 
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 5m0s
    cacheUnauthorizedTTL: 30s
evictionHard:
  imagefs.available: 15%
  memory.available: 100Mi
  nodefs.available: 10%
  nodefs.inodesFree: 5%
maxOpenFiles: 1000000
maxPods: 110
EOF


kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=bootstrap.kubeconfig
  
kubectl config set-credentials "kubelet-bootstrap" \
  --token=${TOKEN} \
  --kubeconfig=bootstrap.kubeconfig
  
kubectl config set-context default \
  --cluster=kubernetes \
  --user="kubelet-bootstrap" \
  --kubeconfig=bootstrap.kubeconfig
  
kubectl config use-context default --kubeconfig=bootstrap.kubeconfig

kubectl create clusterrolebinding kubelet-bootstrap --clusterrole=system:node-bootstrapper --user=kubelet-bootstrap
mv bootstrap.kubeconfig /etc/kubernetes/cfg/

cat > /usr/lib/systemd/system/kubelet.service << EOF
[Unit]
Description=Kubernetes Kubelet
After=docker.service
[Service]
EnvironmentFile=/etc/kubernetes/cfg/kubelet.conf
ExecStart=/usr/local/bin/kubelet \$KUBELET_OPTS
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF


systemctl daemon-reload;systemctl restart kubelet;systemctl enable kubelet; systemctl status kubelet | grep Active

sleep 3

kubectl certificate approve $(kubectl get csr | grep -v NAME | awk '{print $1}')

kubectl get node
```

执行：

```bash
ansible kube-master01 -m script -a "install-kubelet.sh  192.168.23.41"
```

## 18. 安装 kube-proxy

install-kube-proxy.sh脚本内容如下：

```bash
#!/bin/bash


IP=$1

KUBE_APISERVER="https://$IP:6443"

cat > /etc/kubernetes/cfg/kube-proxy.conf << EOF
KUBE_PROXY_OPTS="--v=2 \\
--config=/etc/kubernetes/cfg/kube-proxy-config.yml"
EOF

cat > /etc/kubernetes/cfg/kube-proxy-config.yml << EOF
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 0.0.0.0
metricsBindAddress: 0.0.0.0:10249
clientConnection:
  kubeconfig: /etc/kubernetes/cfg/kube-proxy.kubeconfig
hostnameOverride: kube-master01
clusterCIDR: 10.0.0.0/16
mode: "ipvs"
EOF

cat > /etc/kubernetes/cfg/kube-proxy-config.yml_2 << EOF
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 0.0.0.0
metricsBindAddress: 0.0.0.0:10249
clientConnection:
  kubeconfig: /etc/kubernetes/cfg/kube-proxy.kubeconfig
hostnameOverride: kube-master02
clusterCIDR: 10.0.0.0/16
mode: "ipvs"
EOF

cat > /etc/kubernetes/cfg/kube-proxy-config.yml_3 << EOF
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 0.0.0.0
metricsBindAddress: 0.0.0.0:10249
clientConnection:
  kubeconfig: /etc/kubernetes/cfg/kube-proxy.kubeconfig
hostnameOverride: kube-master03
clusterCIDR: 10.0.0.0/16
mode: "ipvs"
EOF

cd /etc/kubernetes/ssl
cat > kube-proxy-csr.json << EOF
{
  "CN": "system:kube-proxy",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "L": "Shanghai",
      "ST": "Shanghai",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
EOF

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-proxy-csr.json | cfssljson -bare kube-proxy


cd /etc/kubernetes/cfg/

kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=kube-proxy.kubeconfig

kubectl config set-credentials kube-proxy \
  --client-certificate=/etc/kubernetes/ssl/kube-proxy.pem \
  --client-key=/etc/kubernetes/ssl/kube-proxy-key.pem \
  --embed-certs=true \
  --kubeconfig=kube-proxy.kubeconfig

kubectl config set-context default \
  --cluster=kubernetes \
  --user=kube-proxy \
  --kubeconfig=kube-proxy.kubeconfig

kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig

cat > /usr/lib/systemd/system/kube-proxy.service << EOF
[Unit]
Description=Kubernetes Proxy
After=network.target
[Service]
EnvironmentFile=/etc/kubernetes/cfg/kube-proxy.conf
ExecStart=/usr/local/bin/kube-proxy \$KUBE_PROXY_OPTS
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload;systemctl start kube-proxy;systemctl enable kube-proxy; sleep 2;systemctl status kube-proxy |grep Active


```

执行：

```bash
ansible kube-master01 -m script -a "install-kube-proxy.sh  192.168.23.41"
```

输出：

```bash
"stdout": "2024/03/03 17:49:50 [INFO] generate received request\r\n2024/03/03 17:49:50 [INFO] received CSR\r\n2024/03/03 17:49:50 [INFO] generating key: rsa-2048\r\n2024/03/03 17:49:51 [INFO] encoded CSR\r\n2024/03/03 17:49:51 [INFO] signed certificate with serial number 328721913145296821815030017746496286656941249897\r\n2024/03/03 17:49:51 [WARNING] This certificate lacks a \"hosts\" field. This makes it unsuitable for\r\nwebsites. For more information see the Baseline Requirements for the Issuance and Management\r\nof Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);\r\nspecifically, section 10.2.3 (\"Information Requirements\").\r\nCluster \"kubernetes\" set.\r\nUser \"kube-proxy\" set.\r\nContext \"default\" created.\r\nSwitched to context \"default\".\r\nCreated symlink from /etc/systemd/system/multi-user.target.wants/kube-proxy.service to /usr/lib/systemd/system/kube-proxy.service.\r\n   Active: active (running) since Sun 2024-03-03 17:49:53 CST; 2s ago\r\n",
    "stdout_lines": [
        "2024/03/03 17:49:50 [INFO] generate received request",
        "2024/03/03 17:49:50 [INFO] received CSR",
        "2024/03/03 17:49:50 [INFO] generating key: rsa-2048",
        "2024/03/03 17:49:51 [INFO] encoded CSR",
        "2024/03/03 17:49:51 [INFO] signed certificate with serial number 328721913145296821815030017746496286656941249897",
        "2024/03/03 17:49:51 [WARNING] This certificate lacks a \"hosts\" field. This makes it unsuitable for",
        "websites. For more information see the Baseline Requirements for the Issuance and Management",
        "of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);",
        "specifically, section 10.2.3 (\"Information Requirements\").",
        "Cluster \"kubernetes\" set.",
        "User \"kube-proxy\" set.",
        "Context \"default\" created.",
        "Switched to context \"default\".",
        "Created symlink from /etc/systemd/system/multi-user.target.wants/kube-proxy.service to /usr/lib/systemd/system/kube-proxy.service.",
        "   Active: active (running) since Sun 2024-03-03 17:49:53 CST; 2s ago"
```


## 19. 安装 calico 

```bash
wget https://github.com/projectcalico/calico/releases/download/v3.26.4/release-v3.26.4.tgz
tar -zxvf release-v3.26.4.tgz 
cd release-v3.26.4/manifests/

```

```bash
vi calico-etcd.yaml
...
data:
  # Populate the following with etcd TLS configuration if desired, but leave blank if
  # not using TLS for etcd.
  # The keys below should be uncommented and the values populated with the base64
  # encoded contents of each file that would be associated with the TLS data.
  # Example command for encoding a file contents: cat <file> | base64 -w 0
  #将以下三行注释取消，将null替换成指定值，获取方式cat <file> | base64 -w 0，file可以查看/etc/kubernetes/kube-apiserver.conf 中指定的ectd指定的文件路径
cat  /etc/etcd/ssl/etcd-key.pem | base64 -w 0
cat  /etc/etcd/ssl/etcd.pem | base64 -w 0
cat  /etc/etcd/ssl/ca.pem | base64 -w 0

  etcd-key: null
  etcd-cert: null
  etcd-ca: null
...
data:
  # Configure this with the location of your etcd cluster.
  #同样查看/etc/kubernetes/kube-apiserver.conf，将etcd-server的地址填些进去
  etcd_endpoints: "https://192.168.10.121:2379,https://192.168.10.122:2379,https://192.168.10.123:2379"
  # If you're using TLS enabled etcd uncomment the following.
  # You must also populate the Secret below with these files.
  #这是上面三个文件在容器内的挂载路径，去掉注释使用默认的就行
  etcd_ca:  "/calico-secrets/etcd-ca"
  etcd_cert: "/calico-secrets/etcd-cert"
  etcd_key:  "/calico-secrets/etcd-key"
...
#将以下2行去掉注释，将ip修改为/etc/kubernetes/cfg/kube-controller-manager.conf中--cluster-cidr=10.0.0.0/16
            - name: CALICO_IPV4POOL_CIDR
              value: "10.0.0.0/16" #默认值是"192.168.0.0/16"
             #这一行下发插入下面2行，指定服务器使用的网卡，可以用.*通配匹配，也可以是具体网卡
            - name: IP_AUTODETECTION_METHOD
              value: "interface=ens.*"
...::
#默认开启的是IPIP模式，需要将其关闭，就会自动启用BGP模式
#BGP模式网络效率更高，但是node节点需要在同一网段，如需跨网段部署k8s集群，建议使用默认IPIP模式
            # Enable IPIP
            - name: CALICO_IPV4POOL_IPIP
              value: "Never" #将Always修改成Never
...

```

拉取镜像

```bash
docker.io/calico/cni:v3.26.4
docker.io/calico/node:v3.26.4
docker.io/calico/kube-controllers:v3.26.4
```

更改标签入库镜像

```bash
registry.demo/k8s/calico/cni:v3.26.4
registry.demo/k8s/calico/node:v3.26.4
registry.demo/k8s/calico/kube-controllers:v3.26.4

```
改掉calico-etcd.yaml的镜像

执行：

```bash
kubectl apply -f calico-etcd.yaml
```
检查
```bash
$ kubectl get pods -n kube-system
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-6cfb4f4884-4wlnh   1/1     Running   0          14m
kube-system   calico-node-lgg49                          1/1     Running   0          72s
$  kubectl get node
NAME            STATUS   ROLES    AGE   VERSION
kube-master01   Ready    <none>   20h   v1.28.7
```
node 节点恢复正常。


## 20. 新增 Worker 节点

将Worker Node涉及文件拷贝到新节点node节点（192.168.23.44、192.168.23.45）

`copy-node.sh` 脚本内容：

```bash
#!/bin/bash
for i in $@
do
  scp -r /etc/kubernetes/ root@$i:/etc/ ;
  scp -r /usr/lib/systemd/system/{kubelet,kube-proxy}.service root@$i:/usr/lib/systemd/system;
  scp -r /opt/cni/ root@$i:/opt/;
  scp /usr/local/bin/{kubelet,kube-proxy}  root@192.168.10.$i:/usr/local/bin/;
done
done
```
执行：
```bash
ansible kube-master01 -m script -a "copy-node.sh 192.168.23.44 192.168.23.45"
```


初始化节点

`init-node.sh`脚本内容如下：

```bash
#!/bin/bash

rm -rf /etc/kubernetes/cfg/kubelet.kubeconfig
rm -f /etc/kubernetes/ssl/kubelet*
rm -f /etc/kubernetes/cfg/{kube-apiserver,kube-controller-manager,kube-scheduler}.kubeconfig
rm -rf /etc/kubernetes/cfg/{kube-controller-manager.conf,kube-scheduler.conf,kube-apiserver.conf}

hostname=`hostname`
sed -i "s#hostnameOverride.*#hostnameOverride:\ $hostname#g" /etc/kubernetes/cfg/kube-proxy-config.yml
sed -i "s#--hostname-override=.*#--hostname-override=$hostname \\\#g" /etc/kubernetes/cfg/kubelet.conf 
systemctl daemon-reload;systemctl restart kubelet;systemctl enable kubelet;systemctl restart kube-proxy;systemctl enable kube-proxy;sleep 2;systemctl status kubelet | grep Active ;systemctl status kube-proxy | grep Active
```

执行：
```bash
ansible kube_node -m script -a "init-node.sh"
```

Master节点 批准新 Node kubelet 证书申请

```bash
$ kubectl get csr
NAME                                                   AGE   SIGNERNAME                                    REQUESTOR           REQUESTEDDURATION   CONDITION
node-csr-EZ2uQZV-wT4dJwvCoNFWlzBGJ5FDAYO7leP-qUPQvF4   22s   kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   <none>              Pending
node-csr-fOJ--W1TqL6Sqg_7pQvcI2zj1w7hPTmcUrXF_u_gPNg   21s   kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   <none>              Pending
#跟上面一样
$ kubectl certificate approve  node-csr-EZ2uQZV-wT4dJwvCoNFWlzBGJ5FDAYO7leP-qUPQvF4
certificatesigningrequest.certificates.k8s.io/node-csr-EZ2uQZV-wT4dJwvCoNFWlzBGJ5FDAYO7leP-qUPQvF4 approved
$ kubectl certificate approve  node-csr-fOJ--W1TqL6Sqg_7pQvcI2zj1w7hPTmcUrXF_u_gPNg
certificatesigningrequest.certificates.k8s.io/node-csr-fOJ--W1TqL6Sqg_7pQvcI2zj1w7hPTmcUrXF_u_gPNg approved
$ kubectl get csr
NAME                                                   AGE   SIGNERNAME                                    REQUESTOR           REQUESTEDDURATION   CONDITION
node-csr-EZ2uQZV-wT4dJwvCoNFWlzBGJ5FDAYO7leP-qUPQvF4   56s   kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   <none>              Approved,Issued
node-csr-fOJ--W1TqL6Sqg_7pQvcI2zj1w7hPTmcUrXF_u_gPNg   55s   kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   <none>              Approved,Issued
$ kubectl get pod -A
NAMESPACE     NAME                                       READY   STATUS     RESTARTS   AGE
kube-system   calico-kube-controllers-6cfb4f4884-4wlnh   1/1     Running    0          94m
kube-system   calico-node-khqpb                          0/1     Init:0/2   0          8s
kube-system   calico-node-lgg49                          1/1     Running    0          80m
kube-system   calico-node-s85bg                          0/1     Running    0          18s
$ kubectl get pod -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-6cfb4f4884-4wlnh   1/1     Running   0          95m
kube-system   calico-node-khqpb                          1/1     Running   0          68s
kube-system   calico-node-lgg49                          1/1     Running   0          81m
kube-system   calico-node-s85bg                          1/1     Running   0          78s
$ kubectl get node
NAME            STATUS   ROLES    AGE   VERSION
kube-master01   Ready    <none>   21h   v1.28.7
kube-node01     Ready    <none>   83s   v1.28.7
kube-node02     Ready    <none>   73s   v1.28.7

```


## 21. 新增 master 节点
新Master 与已部署的Master1所有操作一致。所以我们只需将Master1所有K8s文件拷贝过来，再修改下服务器IP和主机名启动即可。

`copy-master.sh` 脚本内容：

```bash
#!/bin/bash
for i in $@
do
    scp -r /etc/kubernetes root@$i:/etc; 
    scp -r /opt/cni/ root@$i:/opt; 
    scp /usr/lib/systemd/system/kube* root@$i:/usr/lib/systemd/system; 
    scp /usr/local/bin/kube*  root@$i:/usr/local/bin/;
done
```
执行：
```bash
ansible kube-master01 -m script -a "copy-master.sh 192.168.23.42 192.168.23.43"
```


初始化master节点

`init-master.sh`脚本内容如下：

```bash
#!/bin/bash

rm -f /etc/Kubernetes/cfg/kubelet.kubeconfig
rm -f /etc/kubernetes/ssl/kubelet*
mkdir -p /var/log/kubernetes

IP=$(ifconfig ens160 | grep -o 'inet [0-9.]*' | awk '{print $2}')
sed -i "s#--bind-address=.*#--bind-address=$IP \\\#g" /etc/kubernetes/cfg/kube-apiserver.conf 
sed -i "s#--advertise-address=.*#--advertise-address=$IP \\\#g" /etc/kubernetes/cfg/kube-apiserver.conf 

hostname=`hostname`
sed -i "s#hostnameOverride.*#hostnameOverride:\ $hostname#g" /etc/kubernetes/cfg/kube-proxy-config.yml
sed -i "s#--hostname-override=.*#--hostname-override=$hostname \\\#g" /etc/kubernetes/cfg/kubelet.conf


systemctl daemon-reload;systemctl restart kube-apiserver;systemctl restart kube-controller-manager;systemctl restart kube-scheduler
systemctl restart kubelet;systemctl restart kube-proxy;systemctl enable kube-apiserver;systemctl enable kube-controller-manager;systemctl enable kube-scheduler;systemctl enable kubelet;systemctl enable kube-proxy
sleep 2
systemctl status kubelet | grep Active;systemctl status kube-proxy | grep Active;systemctl status kube-apiserver | grep Active;systemctl status kube-controller-manager | grep Active;systemctl status kube-scheduler | grep Active
```

执行：

```bash
ansible 'kube_control_plane:!kube-master01' -m script -a "init-master.sh"
```
输出示例：

```bash
"stdout": "Created symlink from /etc/systemd/system/multi-user.target.wants/kube-apiserver.service to /usr/lib/systemd/system/kube-apiserver.service.\r\nCreated symlink from /etc/systemd/system/multi-user.target.wants/kube-controller-manager.service to /usr/lib/systemd/system/kube-controller-manager.service.\r\nCreated symlink from /etc/systemd/system/multi-user.target.wants/kube-scheduler.service to /usr/lib/systemd/system/kube-scheduler.service.\r\nCreated symlink from /etc/systemd/system/multi-user.target.wants/kubelet.service to /usr/lib/systemd/system/kubelet.service.\r\nCreated symlink from /etc/systemd/system/multi-user.target.wants/kube-proxy.service to /usr/lib/systemd/system/kube-proxy.service.\r\n   Active: active (running) since Mon 2024-03-04 15:37:20 CST; 4s ago\r\n   Active: active (running) since Mon 2024-03-04 15:37:20 CST; 3s ago\r\n   Active: active (running) since Mon 2024-03-04 15:37:20 CST; 4s ago\r\n   Active: active (running) since Mon 2024-03-04 15:37:20 CST; 4s ago\r\n   Active: active (running) since Mon 2024-03-04 15:37:20 CST; 4s ago\r\n",
    "stdout_lines": [
        "Created symlink from /etc/systemd/system/multi-user.target.wants/kube-apiserver.service to /usr/lib/systemd/system/kube-apiserver.service.",
        "Created symlink from /etc/systemd/system/multi-user.target.wants/kube-controller-manager.service to /usr/lib/systemd/system/kube-controller-manager.service.",
        "Created symlink from /etc/systemd/system/multi-user.target.wants/kube-scheduler.service to /usr/lib/systemd/system/kube-scheduler.service.",
        "Created symlink from /etc/systemd/system/multi-user.target.wants/kubelet.service to /usr/lib/systemd/system/kubelet.service.",
        "Created symlink from /etc/systemd/system/multi-user.target.wants/kube-proxy.service to /usr/lib/systemd/system/kube-proxy.service.",
        "   Active: active (running) since Mon 2024-03-04 15:37:20 CST; 4s ago",
        "   Active: active (running) since Mon 2024-03-04 15:37:20 CST; 3s ago",
        "   Active: active (running) since Mon 2024-03-04 15:37:20 CST; 4s ago",
        "   Active: active (running) since Mon 2024-03-04 15:37:20 CST; 4s ago",
        "   Active: active (running) since Mon 2024-03-04 15:37:20 CST; 4s ago"
    ]
```

kube-master01节点批准 kubelet 证书申请

```bash
$ kubectl get csr
NAME                                                   AGE     SIGNERNAME                                    REQUESTOR           REQUESTEDDURATION   CONDITION
node-csr-EZ2uQZV-wT4dJwvCoNFWlzBGJ5FDAYO7leP-qUPQvF4   33m     kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   <none>              Approved,Issued
node-csr-G1CptkD7poQSoPckSUShqRF5qkqQwzcLZ0uKXlnH8qo   2m22s   kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   <none>              Pending
node-csr-TxuUtQM0Efhrk5rVOZlIwGu2AllNucJciuddVfx_jXc   2m20s   kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   <none>              Pending
node-csr-fOJ--W1TqL6Sqg_7pQvcI2zj1w7hPTmcUrXF_u_gPNg   33m     kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   <none>              Approved,Issued
$ kubectl certificate approve node-csr-G1CptkD7poQSoPckSUShqRF5qkqQwzcLZ0uKXlnH8qo
certificatesigningrequest.certificates.k8s.io/node-csr-G1CptkD7poQSoPckSUShqRF5qkqQwzcLZ0uKXlnH8qo approved
$ kubectl certificate approve node-csr-TxuUtQM0Efhrk5rVOZlIwGu2AllNucJciuddVfx_jXc
certificatesigningrequest.certificates.k8s.io/node-csr-TxuUtQM0Efhrk5rVOZlIwGu2AllNucJciuddVfx_jXc approved
$ kubectl get csr
NAME                                                   AGE    SIGNERNAME                                    REQUESTOR           REQUESTEDDURATION   CONDITION
node-csr-EZ2uQZV-wT4dJwvCoNFWlzBGJ5FDAYO7leP-qUPQvF4   34m    kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   <none>              Approved,Issued
node-csr-G1CptkD7poQSoPckSUShqRF5qkqQwzcLZ0uKXlnH8qo   3m2s   kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   <none>              Approved,Issued
node-csr-TxuUtQM0Efhrk5rVOZlIwGu2AllNucJciuddVfx_jXc   3m     kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   <none>              Approved,Issued
node-csr-fOJ--W1TqL6Sqg_7pQvcI2zj1w7hPTmcUrXF_u_gPNg   34m    kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   <none>              Approved,Issued
$ kubectl get pod -n kube-system
NAME                                       READY   STATUS     RESTARTS   AGE
calico-kube-controllers-6cfb4f4884-4wlnh   1/1     Running    0          127m
calico-node-khqpb                          1/1     Running    0          33m
calico-node-lgg49                          1/1     Running    0          114m
calico-node-msxrs                          0/1     Init:0/2   0          19s
calico-node-s85bg                          1/1     Running    0          33m
calico-node-vq8zq                          0/1     Init:0/2   0          30s
$ kubectl get pod -n kube-system
NAME                                       READY   STATUS     RESTARTS   AGE
calico-kube-controllers-6cfb4f4884-4wlnh   1/1     Running    0          128m
calico-node-khqpb                          1/1     Running    0          34m
calico-node-lgg49                          1/1     Running    0          115m
calico-node-msxrs                          0/1     Init:1/2   0          71s
calico-node-s85bg                          1/1     Running    0          34m
calico-node-vq8zq                          0/1     Init:1/2   0          82s
$ kubectl get pod -n kube-system
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-6cfb4f4884-4wlnh   1/1     Running   0          129m
calico-node-khqpb                          1/1     Running   0          34m
calico-node-lgg49                          1/1     Running   0          115m
calico-node-msxrs                          1/1     Running   0          100s
calico-node-s85bg                          1/1     Running   0          35m
calico-node-vq8zq                          1/1     Running   0          111s
$ kubectl get node
NAME            STATUS   ROLES    AGE    VERSION
kube-master01   Ready    <none>   22h    v1.28.7
kube-master02   Ready    <none>   111s   v1.28.7
kube-master03   Ready    <none>   2m     v1.28.7
kube-node01     Ready    <none>   35m    v1.28.7
kube-node02     Ready    <none>   35m    v1.28.7
```

## 22. 集群管理

### 22.1 设置节点角色

```bash
kubectl label nodes kube-master01 node-role.kubernetes.io/master=
kubectl label nodes kube-master02 node-role.kubernetes.io/master=
kubectl label nodes kube-master03 node-role.kubernetes.io/master=
kubectl label nodes kube-node01 node-role.kubernetes.io/worker=
kubectl label nodes kube-node02 node-role.kubernetes.io/worker=
kubectl label nodes kube-node03 node-role.kubernetes.io/worker=
```

### 22.2 master 节点设置不可调度

```bash
kubectl taint nodes kube-master01  node-role.kubernetes.io/master=:NoSchedule
kubectl taint nodes kube-master02  node-role.kubernetes.io/master=:NoSchedule
kubectl taint nodes kube-master03  node-role.kubernetes.io/master=:NoSchedule
kubectl cordon kube-master01
kubectl cordon kube-master02
kubectl cordon kube-master03
kubectl describe node kube-master01 |grep Taints
```

取消不可调度的命令，不需要执行（不需要执行）

```bash
kubectl taint node  kube-master01 node-role.kubernetes.io/master-
```

## 23. 部署 CoreDNS v1.11.1

CoreDNS用于集群内部Service名称解析,版本v1.11.1

```bash
wget https://raw.githubusercontent.com/coredns/deployment/master/kubernetes/coredns.yaml.sed

#修改corefile.yaml
  Corefile: |
    .:53 {
        log
        errors
        health {
            lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa { #修改为
            pods insecure
            fallthrough in-addr.arpa ip6.arpa
            ttl 30
        }
        prometheus :9153
        forward . /etc/resolv.conf #修改为本机的resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }#去掉这个地方的后缀
#修改文件，参考/etc/kubernetes/cfg/kubelet-config.yml 配置文件中的clusterIP
spec:
  selector:
    k8s-app: kube-dns
  clusterIP: 10.255.0.2
#将文件中的镜像地址替换成新地址
sed -i 's/coredns\/coredns:1.9.4/registry01.ghostwritten.com\/library\/coredns\/coredns:1.11.1/g' coredns.yaml
registry01.ghostwritten.com/library/coredns/coredns:v1.11.1

```


执行:

```bash
$ kubectl apply -f coredns.yaml
serviceaccount/coredns created
clusterrole.rbac.authorization.k8s.io/system:coredns created
clusterrolebinding.rbac.authorization.k8s.io/system:coredns created
configmap/coredns created
deployment.apps/coredns created
service/kube-dns created
```
查看

```bash
$ kubectl get pod -n kube-system
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-6cfb4f4884-4wlnh   1/1     Running   0          157m
calico-node-khqpb                          1/1     Running   0          63m
calico-node-lgg49                          1/1     Running   0          144m
calico-node-msxrs                          1/1     Running   0          30m
calico-node-s85bg                          1/1     Running   0          63m
calico-node-vq8zq                          1/1     Running   0          30m
coredns-56f6b65f69-plrvz                   1/1     Running   0          117s
```

DNS解析测试：

```bash
$ kubectl run busybox --image registry01.ghostwritten.com/library/busybox:1.31.1 --restart=Never --rm -it busybox -- sh

/ # nslookup kubernetes
Server:    10.255.0.2
Address 1: 10.255.0.2 kube-dns.kube-system.svc.cluster.local
Name:      kubernetes
Address 1: 10.255.0.1 kubernetes.default.svc.cluster.local

```


## 24. 部署 ingress v1.10.0

```bash
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/baremetal/deploy.yaml
sed -i 's/registry.k8s.io/registry01.ghostwritten.com/g' deploy.yaml
```
修改 deploy.yaml

```bash

spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/instance: ingress-nginx
      app.kubernetes.io/component: controller
  revisionHistoryLimit: 10
  minReadySeconds: 0
  replicas: 3  #将ingress设为多副本
  template:
    metadata:
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/component: controller
    spec:
      dnsPolicy: ClusterFirst
      hostNetwork: true #在此位置添加此项
      containers:
        - name: controller
          image: registry01.ghostwritten.com/library/ingress-nginx/controller:v1.10.0
          imagePullPolicy: IfNotPresent
#并将文件中的镜像地址替换成新镜像地址
registry.k8s.io/ingress-nginx/controller:v1.10.0
registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.4.0
registry01.ghostwritten.com/library/ingress-nginx/controller:v1.10.0
registry01.ghostwritten.com/library/ingress-nginx/kube-webhook-certgen:v1.4.0
d

```
执行：

```bash
$ kubectl apply -f deploy.yaml
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
serviceaccount/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
configmap/ingress-nginx-controller created
service/ingress-nginx-controller created
service/ingress-nginx-controller-admission created
deployment.apps/ingress-nginx-controller created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
ingressclass.networking.k8s.io/nginx created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
```
检查

```bash
$ kubectl get svc -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.255.194.98   <none>        80:51093/TCP,443:54317/TCP   71s
ingress-nginx-controller-admission   ClusterIP   10.255.215.50   <none>        443/TCP                      71s
$ kubectl get pod -n ingress-nginx
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-b22f6        0/1     Completed   0          78s
ingress-nginx-admission-patch-jwp2j         0/1     Completed   0          78s
ingress-nginx-controller-6dcc46db88-xzj8m   1/1     Running     0          78s
$ curl 192.168.23.41:51093
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>



#出现404.说明配置成功
```
测试 ingress 应用
下载介质中会包含次镜像
docker pull nginx:1.25.3

创建一个测试pod

```bash
$ vi nginx-demo.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: default
spec:
  selector: 
    app: nginx
  ports:
  - name: http
    targetPort: 80
    port: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  namespace: default
spec:
  replicas: 3
  selector: 
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: registry01.ghostwritten.com/library/nginx:1.25.3
        ports: 
        - name: http
          containerPort: 80 
#创建ingress规则
---
apiVersion: networking.k8s.io/v1
kind: Ingress 
metadata:
  name: ingress-nginx
  namespace: default 
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules: 
  - host: "nginx.demo.com"
    http:
      paths: 
      - pathType: Prefix
        path: "/" 
        backend:
          service:
            name: nginx
            port:
              number: 80
```

执行：

```bash
kubectl apply -f nginx-demo.yaml
```

```bash

#查看ingress
$ kubectl get ingress
NAME            CLASS    HOSTS            ADDRESS         PORTS   AGE
ingress-nginx   <none>   nginx.demo.com   192.168.23.45   80      2m22s
#配置本地解析
$ vim /etc/hosts
192.168.23.45 nginx.demo.com

#访问
$ curl nginx.demo.com
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## 25. 部署 metrics-server v0.7.0
metrics_server: “v0.7.0”

```bash
wget https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.7.0/components.yaml
#修改镜像 和添加 --kubelet-insecure-tls 
命令参数：
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls
        image: registry01.ghostwritten.com/library/metrics-server/metrics-server:v0.7.0
        imagePullPolicy: IfNotPresent
```

执行：

```bash
kubectl apply -f components.yaml
```
输出：
```bash
serviceaccount/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
service/metrics-server created
deployment.apps/metrics-server created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
```
检查

```bash
$ kubectl get pod -n kube-system
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-6cfb4f4884-4wlnh   1/1     Running   0          3h23m
calico-node-khqpb                          1/1     Running   0          109m
calico-node-lgg49                          1/1     Running   0          3h10m
calico-node-msxrs                          1/1     Running   0          76m
calico-node-s85bg                          1/1     Running   0          109m
calico-node-vq8zq                          1/1     Running   0          76m
coredns-56f6b65f69-plrvz                   1/1     Running   0          48m
metrics-server-7c97fb5fc6-zbg6b            1/1     Running   0          106s
$ kubectl top node
NAME            CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
kube-master01   1129m        28%    3639Mi          46%       
kube-master02   506m         12%    1692Mi          21%       
kube-master03   311m         7%     1798Mi          22%       
kube-node01     169m         4%     1758Mi          22%       
kube-node02     298m         3%     2064Mi          26%       
$ kubectl top pod
NAME                            CPU(cores)   MEMORY(bytes)   
nginx-deploy-59b58585bf-4xw52   0m           4Mi             
nginx-deploy-59b58585bf-7p2vs   0m           7Mi             
nginx-deploy-59b58585bf-sl446   0m           4Mi             
```
结束。
![](https://i-blog.csdnimg.cn/blog_migrate/bcd28a8ed4bb5889ed29155d266466ca.jpeg#pic_center)

