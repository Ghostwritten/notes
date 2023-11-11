![](https://img-blog.csdnimg.cn/41286edf08324f83ab7b74ecaa1997a5.png)



## 1. 目标
本篇将说明如何通过 Kubespray 部署 Kubernetes 至裸机节点，安装版本如下所示：

rocky linux 8.8
Kubernetes v1.25.6
kubespray v2.21.0-1


## 2. 预备条件

系统： rocky linux 8.8

192.168.23.30-rocky-8.8-bastion01
bastion01 (这里下载介质与部署节点为同一节点，如果非同一节点，需要介质下载搬运)
- 192.168.23.30（联网下载介质）
- 10.60.0.30（不同外网）
- 10.60.254.254 （私网路由）
- cpu：4
- 内存：8
- 磁盘：60G（系统盘）（Thin Provision）

192.168.23.31-rocky-8.8-kube-master01
kube-master01(control plane)

- 10.60.0.31(不同外网)
- cpu：4
- 内存：8
- 磁盘：60G（系统盘）（Thin Provision）

192.168.23.32-rocky-8.8-kube-prom01
kube-prom01(worker node)
- 10.60.0.32(不同外网)
- cpu：2
- 内存：4
- 磁盘：60G（系统盘）（Thin Provision）

192.168.23.33-rocky-8.8-kube-node01
kube-node01(worker node)
- 10.60.0.33(不同外网)
- cpu：4
- 内存：8
- 磁盘：60G（系统盘）（Thin Provision）、100G（挂载）（Thin Provision）


192.168.23.34-rocky-8.8-kube-node02
kube-node02(worker node)
- 10.60.0.34(不同外网)
- cpu：4
- 内存：8
- 磁盘：60G（系统盘）（Thin Provision）、100G（挂载）（Thin Provision）

192.168.23.35-rocky-8.8-kube-node03
kube-node03(worker node)
- 10.60.0.35(不同外网)
- cpu：4
- 内存：8
- 磁盘：60G（系统盘）（Thin Provision）、100G（挂载）（Thin Provision）


192.168.23.30 bastion01 为镜像与yum 源介质分发地址，暂时不改为离线.
## 3. vcenter 创建虚拟机
- [Vcenter 6.7 创建 Ubuntu 22.10 虚拟机 顺带安装 Microk8s](https://ghostwritten.blog.csdn.net/article/details/129310311)
- [Vcenter 创建 虚拟机配置 Thin Provision 模式 disk](https://blog.csdn.net/xixihahalelehehe/article/details/132009106)


## 4. 系统初始化
(每台操作)

- [Rocky Linux 9.1 新手入门指南](https://blog.csdn.net/xixihahalelehehe/article/details/129644123)
- [centos 8.2 指南](https://blog.csdn.net/xixihahalelehehe/article/details/127641551)
- [centos linux 配置私有网段并联网](https://ghostwritten.blog.csdn.net/article/details/130607628)


### 4.1 配置网卡

```bash
[root@bastion01 ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens192 
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
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
IPADDR=192.168.23.30
PREFIX=20
GATEWAY=192.168.21.1
DNS1=8.8.8.8

```
重启 网卡

```bash
nmcli c reload && nmcli c down ens192 && nmcli c up ens192
```

### 4.2 配置主机名

```bash
hostnamectl set-hostname bastion01
hostnamectl set-hostname kube-master01
hostnamectl set-hostname kube-prom01
hostnamectl set-hostname kube-node01
hostnamectl set-hostname kube-node02
hostnamectl set-hostname kube-node03
```

### 4.3 内核参数

```bash
cat <<EOF> /etc/sysctl.conf 
# sysctl settings are defined through files in
# /usr/lib/sysctl.d/, /run/sysctl.d/, and /etc/sysctl.d/.
#
# Vendors settings live in /usr/lib/sysctl.d/.
# To override a whole file, create a new file with the same in
# /etc/sysctl.d/ and put new settings there. To override
# only specific settings, add a file with a lexically later
# name in /etc/sysctl.d/ and put new settings there.
#
# For more information, see sysctl.conf(5) and sysctl.d(5).
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```
执行

```bash
sysctl -p
```

## 5. 打快照
打快照前一定先关机
![](https://img-blog.csdnimg.cn/5324553a3a954a75a8dcbcb419472958.png)
1. 初始化系统 20230803
![](https://img-blog.csdnimg.cn/97ecab8748494babaa233e7d3400e93b.png)


## 6. 安装 git
（bastion01）
- [git 安装](https://ghostwritten.blog.csdn.net/article/details/125107061)


## 7. 配置科学
（bastion01）
- 科学

加速下载介质，例如：github、quay、docker hub

```bash
$ vim /root/.bashrc
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

enable_proxy
#disable_proxy

```
执行：

```bash
source /root/.bashrc
```

##  8. 安装 docker
（bastion01在线操作）

用于拉取镜像介质做准备。

-  [docker 安装](https://blog.csdn.net/xixihahalelehehe/article/details/104293170)
- [官方 docker 安装](https://docs.docker.com/engine/install/)

```bash
yum -y install docker-ce
```

配置代理，注意，`NO_PROXY`一定要添加自己的本地仓库域名或地址，否则会出现推送入库失败的问题。

```bash
mkdir -p /etc/systemd/system/docker.service.d
cat <<EOF> /etc/systemd/system/docker.service.d/proxy.conf
[Service]
Environment="HTTP_PROXY=http://192.168.21.101:7890"
Environment="HTTPS_PROXY=http://192.168.21.101:7890"
Environment="NO_PROXY=localhost,127.0.0.1,.example.com,registry.demo:35000,10.60.0.30"
EOF
systemctl daemon-reload &&systemctl restart docker && systemctl status docker
```

## 9. 下载介质
（bastion01在线）

```bash
mkdir kubespray-offline
cd kubespray-offline
```

### 9.1 下载安装 docker 介质

用于离线环境集群之外部署 `docker registry`，当然镜像仓库部署 registry 你可以选择其他容器工具，podman、nerdctl、ctr等。或者选择 harbor 作为你的镜像仓库。

如果离线环境的yum 源有 docker，可以直接安装：

```bash
yum -y install docker-ce
```
如果离线环境节点没有关于 docker 的 yum 源，需要在线下载介质。
```bash
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install --downloadonly --downloaddir rpms/ docker-ce docker-ce-cli containerd.io
```



### 9.2 下载 kubespray-offline-ansible 介质
kubespray 镜像永远构建部署的运行环境。
> 注意：这里没有选择本地安装ansible，因为离线环境在安装 ansible 的时候很大可能存在依赖包安装失败问题。
```bash
docker pull quay.io/kubespray/kubespray:v2.22.1 && docker save -o kubespray-v2.22.1.tar kubespray:v2.22.1
```

###  9.3 下载 kubernetes 介质
用于下载介质工具

- 下载 [kubespray v2.21.0-1](https://github.com/tmurakam/kubespray-offline/releases) 

```bash
wget https://github.com/tmurakam/kubespray-offline/archive/refs/tags/v2.21.0-1.zip
unzip v2.21.0-1.zip
cd kubespray-offline-2.21.0-1
```

定制下载 `kubespray v2.22.1` 版本
```bash
[root@bastion01 kubespray-offline-2.21.0-1]# cat config.sh 
#!/bin/bash

# Kubespray version to download. Use "master" for latest master branch.
KUBESPRAY_VERSION=${KUBESPRAY_VERSION:-2.22.1}
#KUBESPRAY_VERSION=${KUBESPRAY_VERSION:-master}

# container runtime for preparation node
docker=${docker:-docker}
#docker=${docker:-/usr/local/bin/nerdctl}

# Run ansible in container?
ansible_in_container=${ansible_in_container:-true}

```
执行：

```bash
./download-all.sh
```

```bash
[root@bastion01 kubespray-offline-2.21.0-1]# tree outputs/
outputs/
├── ansible-container.sh
├── config.sh
├── config.toml
├── containerd.service
├── extract-kubespray.sh
├── files
│   ├── cilium-linux-amd64.tar.gz
│   ├── containerd-1.7.1-linux-amd64.tar.gz
│   ├── containerd-shim-runsc-v1
│   ├── cri-dockerd-0.3.0.amd64.tgz
│   ├── cri-o.amd64.v1.26.3.tar.gz
│   ├── crun-1.4.5-linux-amd64
│   ├── files.list
│   ├── helm-v3.12.0-linux-amd64.tar.gz
│   ├── kata-static-2.4.1-x86_64.tar.xz
│   ├── krew-linux_amd64.tar.gz
│   ├── kubernetes
│   │   ├── calico
│   │   │   ├── v3.25.1
│   │   │   │   └── calicoctl-linux-amd64
│   │   │   └── v3.25.1.tar.gz
│   │   ├── cni
│   │   │   └── cni-plugins-linux-amd64-v1.3.0.tgz
│   │   ├── cri-tools
│   │   │   └── crictl-v1.26.0-linux-amd64.tar.gz
│   │   ├── etcd
│   │   │   └── etcd-v3.5.6-linux-amd64.tar.gz
│   │   └── v1.26.5
│   │       ├── kubeadm
│   │       ├── kubectl
│   │       └── kubelet
│   ├── kubespray-2.22.1.tar.gz
│   ├── nerdctl-1.4.0-linux-amd64.tar.gz
│   ├── runc
│   │   └── v1.1.7
│   │       └── runc.amd64
│   ├── runsc
│   ├── skopeo-linux-amd64
│   ├── youki_v0_0_1_linux.tar.gz
│   └── yq_linux_amd64
├── images
│   ├── additional-images.list
│   ├── docker.io_amazon_aws-alb-ingress-controller-v1.1.9.tar.gz
│   ├── docker.io_amazon_aws-ebs-csi-driver-v0.5.0.tar.gz
│   ├── docker.io_cloudnativelabs_kube-router-v1.5.1.tar.gz
│   ├── docker.io_envoyproxy_envoy-v1.22.5.tar.gz
│   ├── docker.io_flannelcni_flannel-cni-plugin-v1.2.0.tar.gz
│   ├── docker.io_flannel_flannel-v0.21.4.tar.gz
│   ├── docker.io_k8scloudprovider_cinder-csi-plugin-v1.22.0.tar.gz
│   ├── docker.io_kubeovn_kube-ovn-v1.10.7.tar.gz
│   ├── docker.io_kubernetesui_dashboard-v2.7.0.tar.gz
│   ├── docker.io_kubernetesui_metrics-scraper-v1.0.8.tar.gz
│   ├── docker.io_library_haproxy-2.6.6-alpine.tar.gz
│   ├── docker.io_library_nginx-1.23.2-alpine.tar.gz
│   ├── docker.io_library_nginx-1.23.tar.gz
│   ├── docker.io_library_registry-2.8.1.tar.gz
│   ├── docker.io_mirantis_k8s-netchecker-agent-v1.2.2.tar.gz
│   ├── docker.io_mirantis_k8s-netchecker-server-v1.2.2.tar.gz
│   ├── docker.io_rancher_local-path-provisioner-v0.0.23.tar.gz
│   ├── docker.io_weaveworks_weave-kube-2.8.1.tar.gz
│   ├── docker.io_weaveworks_weave-npc-2.8.1.tar.gz
│   ├── ghcr.io_k8snetworkplumbingwg_multus-cni-v3.8.tar.gz
│   ├── ghcr.io_kube-vip_kube-vip-v0.5.12.tar.gz
│   ├── images.list
│   ├── kubespray-offline-ansible.tar.gz
│   ├── quay.io_calico_apiserver-v3.25.1.tar.gz
│   ├── quay.io_calico_cni-v3.25.1.tar.gz
│   ├── quay.io_calico_kube-controllers-v3.25.1.tar.gz
│   ├── quay.io_calico_node-v3.25.1.tar.gz
│   ├── quay.io_calico_pod2daemon-flexvol-v3.25.1.tar.gz
│   ├── quay.io_calico_typha-v3.25.1.tar.gz
│   ├── quay.io_cilium_certgen-v0.1.8.tar.gz
│   ├── quay.io_cilium_cilium-v1.13.0.tar.gz
│   ├── quay.io_cilium_hubble-relay-v1.13.0.tar.gz
│   ├── quay.io_cilium_hubble-ui-backend-v0.11.0.tar.gz
│   ├── quay.io_cilium_hubble-ui-v0.11.0.tar.gz
│   ├── quay.io_cilium_operator-v1.13.0.tar.gz
│   ├── quay.io_coreos_etcd-v3.5.6.tar.gz
│   ├── quay.io_external_storage_cephfs-provisioner-v2.1.0-k8s1.11.tar.gz
│   ├── quay.io_external_storage_rbd-provisioner-v2.1.1-k8s1.11.tar.gz
│   ├── quay.io_jetstack_cert-manager-cainjector-v1.11.1.tar.gz
│   ├── quay.io_jetstack_cert-manager-controller-v1.11.1.tar.gz
│   ├── quay.io_jetstack_cert-manager-webhook-v1.11.1.tar.gz
│   ├── quay.io_metallb_controller-v0.13.9.tar.gz
│   ├── quay.io_metallb_speaker-v0.13.9.tar.gz
│   ├── registry.k8s.io_coredns_coredns-v1.9.3.tar.gz
│   ├── registry.k8s.io_cpa_cluster-proportional-autoscaler-v1.8.8.tar.gz
│   ├── registry.k8s.io_dns_k8s-dns-node-cache-1.22.18.tar.gz
│   ├── registry.k8s.io_ingress-nginx_controller-v1.7.1.tar.gz
│   ├── registry.k8s.io_kube-apiserver-v1.26.5.tar.gz
│   ├── registry.k8s.io_kube-controller-manager-v1.26.5.tar.gz
│   ├── registry.k8s.io_kube-proxy-v1.26.5.tar.gz
│   ├── registry.k8s.io_kube-scheduler-v1.26.5.tar.gz
│   ├── registry.k8s.io_metrics-server_metrics-server-v0.6.3.tar.gz
│   ├── registry.k8s.io_pause-3.9.tar.gz
│   ├── registry.k8s.io_sig-storage_csi-attacher-v3.3.0.tar.gz
│   ├── registry.k8s.io_sig-storage_csi-node-driver-registrar-v2.4.0.tar.gz
│   ├── registry.k8s.io_sig-storage_csi-provisioner-v3.0.0.tar.gz
│   ├── registry.k8s.io_sig-storage_csi-resizer-v1.3.0.tar.gz
│   ├── registry.k8s.io_sig-storage_csi-snapshotter-v5.0.0.tar.gz
│   ├── registry.k8s.io_sig-storage_local-volume-provisioner-v2.5.0.tar.gz
│   └── registry.k8s.io_sig-storage_snapshot-controller-v4.2.1.tar.gz
├── install-containerd.sh
├── load-push-all-images.sh
├── patches
│   └── 2.18.0
│       ├── 0001-nerdctl-insecure-registry-config-8339.patch
│       ├── 0002-Update-config.toml.j2-8340.patch
│       └── 0003-generate-list-8537.patch
├── playbook
│   ├── offline-repo.yml
│   └── roles
│       └── offline-repo
│           ├── defaults
│           │   └── main.yml
│           ├── files
│           │   └── 99offline
│           └── tasks
│               ├── Debian.yml
│               ├── main.yml
│               └── RedHat.yml
├── rpms
│   └── local
│       ├── acl-2.2.53-1.el8.1.x86_64.rpm
│       ├── annobin-10.94-1.el8.x86_64.rpm
│       ├── audit-3.0.7-4.el8.x86_64.rpm
│       ├── audit-libs-3.0.7-4.el8.x86_64.rpm
│       ├── basesystem-11-5.el8.noarch.rpm
│       ├── bash-4.4.20-4.el8_6.x86_64.rpm
│       ├── bash-completion-2.7-5.el8.noarch.rpm
│       ├── binutils-2.30-119.el8.x86_64.rpm
│       ├── brotli-1.0.6-3.el8.x86_64.rpm
│       ├── bzip2-libs-1.0.6-26.el8.x86_64.rpm
│       ├── ca-certificates-2022.2.54-80.2.el8_6.noarch.rpm
│       ├── checkpolicy-2.9-1.el8.x86_64.rpm
│       ├── chkconfig-1.19.1-1.el8.x86_64.rpm
│       ├── conntrack-tools-1.4.4-11.el8.x86_64.rpm
│       ├── container-selinux-2.205.0-2.module+el8.8.0+1265+fa25dd7a.noarch.rpm
│       ├── coreutils-8.30-15.el8.x86_64.rpm
│       ├── coreutils-common-8.30-15.el8.x86_64.rpm
│       ├── cpio-2.12-11.el8.x86_64.rpm
│       ├── cpp-8.5.0-18.el8.x86_64.rpm
│       ├── cracklib-2.9.6-15.el8.x86_64.rpm
│       ├── cracklib-dicts-2.9.6-15.el8.x86_64.rpm
│       ├── crypto-policies-20221215-1.gitece0092.el8.noarch.rpm
│       ├── crypto-policies-scripts-20221215-1.gitece0092.el8.noarch.rpm
│       ├── cryptsetup-libs-2.3.7-5.el8.x86_64.rpm
│       ├── curl-7.61.1-30.el8_8.2.x86_64.rpm
│       ├── cyrus-sasl-lib-2.1.27-6.el8_5.x86_64.rpm
│       ├── dbus-1.12.8-24.el8.x86_64.rpm
│       ├── dbus-common-1.12.8-24.el8.noarch.rpm
│       ├── dbus-daemon-1.12.8-24.el8.x86_64.rpm
│       ├── dbus-glib-0.110-2.el8.x86_64.rpm
│       ├── dbus-libs-1.12.8-24.el8.x86_64.rpm
│       ├── dbus-tools-1.12.8-24.el8.x86_64.rpm
│       ├── device-mapper-1.02.181-9.el8.x86_64.rpm
│       ├── device-mapper-event-1.02.181-9.el8.x86_64.rpm
│       ├── device-mapper-event-libs-1.02.181-9.el8.x86_64.rpm
│       ├── device-mapper-libs-1.02.181-9.el8.x86_64.rpm
│       ├── device-mapper-persistent-data-0.9.0-7.el8.x86_64.rpm
│       ├── diffutils-3.6-6.el8.x86_64.rpm
│       ├── dnf-4.7.0-16.el8_8.noarch.rpm
│       ├── dnf-data-4.7.0-16.el8_8.noarch.rpm
│       ├── dnf-plugins-core-4.0.21-19.el8_8.noarch.rpm
│       ├── dracut-049-223.git20230119.el8.x86_64.rpm
│       ├── e2fsprogs-1.45.6-5.el8.x86_64.rpm
│       ├── e2fsprogs-libs-1.45.6-5.el8.x86_64.rpm
│       ├── elfutils-debuginfod-client-0.188-3.el8.x86_64.rpm
│       ├── elfutils-default-yama-scope-0.188-3.el8.noarch.rpm
│       ├── elfutils-libelf-0.188-3.el8.x86_64.rpm
│       ├── elfutils-libs-0.188-3.el8.x86_64.rpm
│       ├── expat-2.2.5-11.el8.x86_64.rpm
│       ├── file-5.33-24.el8.x86_64.rpm
│       ├── file-libs-5.33-24.el8.x86_64.rpm
│       ├── filesystem-3.8-6.el8.x86_64.rpm
│       ├── findutils-4.6.0-20.el8.x86_64.rpm
│       ├── firewalld-0.9.3-13.el8.noarch.rpm
│       ├── firewalld-filesystem-0.9.3-13.el8.noarch.rpm
│       ├── fuse-libs-2.9.7-16.el8.x86_64.rpm
│       ├── gawk-4.2.1-4.el8.x86_64.rpm
│       ├── gcc-8.5.0-18.el8.x86_64.rpm
│       ├── gdbm-1.18-2.el8.x86_64.rpm
│       ├── gdbm-libs-1.18-2.el8.x86_64.rpm
│       ├── gettext-0.19.8.1-17.el8.x86_64.rpm
│       ├── gettext-libs-0.19.8.1-17.el8.x86_64.rpm
│       ├── glib2-2.56.4-161.el8.x86_64.rpm
│       ├── glibc-2.28-225.el8.x86_64.rpm
│       ├── glibc-all-langpacks-2.28-225.el8.x86_64.rpm
│       ├── glibc-common-2.28-225.el8.x86_64.rpm
│       ├── glibc-devel-2.28-225.el8.x86_64.rpm
│       ├── glibc-gconv-extra-2.28-225.el8.x86_64.rpm
│       ├── glibc-headers-2.28-225.el8.x86_64.rpm
│       ├── gmp-6.1.2-10.el8.x86_64.rpm
│       ├── gnupg2-2.2.20-3.el8_6.x86_64.rpm
│       ├── gnupg2-smime-2.2.20-3.el8_6.x86_64.rpm
│       ├── gnutls-3.6.16-6.el8_7.x86_64.rpm
│       ├── gobject-introspection-1.56.1-1.el8.x86_64.rpm
│       ├── gpgme-1.13.1-11.el8.x86_64.rpm
│       ├── grep-3.1-6.el8.x86_64.rpm
│       ├── grub2-common-2.02-148.el8.rocky.0.3.noarch.rpm
│       ├── grub2-tools-2.02-148.el8.rocky.0.3.x86_64.rpm
│       ├── grub2-tools-minimal-2.02-148.el8.rocky.0.3.x86_64.rpm
│       ├── grubby-8.40-47.el8.x86_64.rpm
│       ├── gzip-1.9-13.el8_5.x86_64.rpm
│       ├── hardlink-1.3-6.el8.x86_64.rpm
│       ├── ima-evm-utils-1.3.2-12.el8.x86_64.rpm
│       ├── info-6.5-7.el8.x86_64.rpm
│       ├── initscripts-10.00.18-1.el8.x86_64.rpm
│       ├── ipset-7.1-1.el8.x86_64.rpm
│       ├── ipset-libs-7.1-1.el8.x86_64.rpm
│       ├── iptables-1.8.4-24.el8.x86_64.rpm
│       ├── iptables-ebtables-1.8.4-24.el8.x86_64.rpm
│       ├── iptables-libs-1.8.4-24.el8.x86_64.rpm
│       ├── ipvsadm-1.31-1.el8.x86_64.rpm
│       ├── isl-0.16.1-6.el8.x86_64.rpm
│       ├── jansson-2.14-1.el8.x86_64.rpm
│       ├── json-c-0.13.1-3.el8.x86_64.rpm
│       ├── kbd-2.0.4-10.el8.x86_64.rpm
│       ├── kbd-legacy-2.0.4-10.el8.noarch.rpm
│       ├── kbd-misc-2.0.4-10.el8.noarch.rpm
│       ├── kernel-headers-4.18.0-477.15.1.el8_8.x86_64.rpm
│       ├── keyutils-libs-1.5.10-9.el8.x86_64.rpm
│       ├── keyutils-libs-devel-1.5.10-9.el8.x86_64.rpm
│       ├── kmod-25-19.el8.x86_64.rpm
│       ├── kmod-libs-25-19.el8.x86_64.rpm
│       ├── kpartx-0.8.4-37.el8.x86_64.rpm
│       ├── krb5-devel-1.18.2-25.el8_8.x86_64.rpm
│       ├── krb5-libs-1.18.2-25.el8_8.x86_64.rpm
│       ├── libacl-2.2.53-1.el8.1.x86_64.rpm
│       ├── libaio-0.3.112-1.el8.x86_64.rpm
│       ├── libarchive-3.3.3-5.el8.x86_64.rpm
│       ├── libassuan-2.5.1-3.el8.x86_64.rpm
│       ├── libattr-2.4.48-3.el8.x86_64.rpm
│       ├── libblkid-2.32.1-42.el8_8.x86_64.rpm
│       ├── libcap-2.48-4.el8.x86_64.rpm
│       ├── libcap-ng-0.7.11-1.el8.x86_64.rpm
│       ├── libcom_err-1.45.6-5.el8.x86_64.rpm
│       ├── libcom_err-devel-1.45.6-5.el8.x86_64.rpm
│       ├── libcomps-0.1.18-1.el8.x86_64.rpm
│       ├── libcroco-0.6.12-4.el8_2.1.x86_64.rpm
│       ├── libcurl-7.61.1-30.el8_8.2.x86_64.rpm
│       ├── libdb-5.3.28-42.el8_4.x86_64.rpm
│       ├── libdb-utils-5.3.28-42.el8_4.x86_64.rpm
│       ├── libdnf-0.63.0-14.el8_8.x86_64.rpm
│       ├── libevent-2.1.8-5.el8.x86_64.rpm
│       ├── libfdisk-2.32.1-42.el8_8.x86_64.rpm
│       ├── libffi-3.1-24.el8.x86_64.rpm
│       ├── libffi-devel-3.1-24.el8.x86_64.rpm
│       ├── libgcc-8.5.0-18.el8.x86_64.rpm
│       ├── libgcrypt-1.8.5-7.el8_6.x86_64.rpm
│       ├── libgomp-8.5.0-18.el8.x86_64.rpm
│       ├── libgpg-error-1.31-1.el8.x86_64.rpm
│       ├── libibverbs-44.0-2.el8.1.x86_64.rpm
│       ├── libidn2-2.2.0-1.el8.x86_64.rpm
│       ├── libkadm5-1.18.2-25.el8_8.x86_64.rpm
│       ├── libkcapi-1.2.0-2.el8.x86_64.rpm
│       ├── libkcapi-hmaccalc-1.2.0-2.el8.x86_64.rpm
│       ├── libksba-1.3.5-9.el8_7.x86_64.rpm
│       ├── libmnl-1.0.4-6.el8.x86_64.rpm
│       ├── libmodulemd-2.13.0-1.el8.x86_64.rpm
│       ├── libmount-2.32.1-42.el8_8.x86_64.rpm
│       ├── libmpc-1.1.0-9.1.el8.x86_64.rpm
│       ├── libnetfilter_conntrack-1.0.6-5.el8.x86_64.rpm
│       ├── libnetfilter_cthelper-1.0.0-15.el8.x86_64.rpm
│       ├── libnetfilter_cttimeout-1.0.0-11.el8.x86_64.rpm
│       ├── libnetfilter_queue-1.0.4-3.el8.x86_64.rpm
│       ├── libnfnetlink-1.0.1-13.el8.x86_64.rpm
│       ├── libnftnl-1.1.5-5.el8.x86_64.rpm
│       ├── libnghttp2-1.33.0-3.el8_3.1.x86_64.rpm
│       ├── libnl3-3.7.0-1.el8.x86_64.rpm
│       ├── libnsl2-1.2.0-2.20180605git4a062cf.el8.x86_64.rpm
│       ├── libpcap-1.9.1-5.el8.x86_64.rpm
│       ├── libpkgconf-1.4.2-1.el8.x86_64.rpm
│       ├── libpsl-0.20.2-6.el8.x86_64.rpm
│       ├── libpwquality-1.4.4-6.el8.x86_64.rpm
│       ├── librepo-1.14.2-4.el8.x86_64.rpm
│       ├── libreport-filesystem-2.9.5-15.el8.rocky.6.3.x86_64.rpm
│       ├── libseccomp-2.5.2-1.el8.x86_64.rpm
│       ├── libsecret-0.18.6-1.el8.0.2.x86_64.rpm
│       ├── libselinux-2.9-8.el8.x86_64.rpm
│       ├── libselinux-devel-2.9-8.el8.x86_64.rpm
│       ├── libselinux-utils-2.9-8.el8.x86_64.rpm
│       ├── libsemanage-2.9-9.el8_6.x86_64.rpm
│       ├── libsepol-2.9-3.el8.x86_64.rpm
│       ├── libsepol-devel-2.9-3.el8.x86_64.rpm
│       ├── libsigsegv-2.11-5.el8.x86_64.rpm
│       ├── libsmartcols-2.32.1-42.el8_8.x86_64.rpm
│       ├── libsolv-0.7.20-4.el8_7.x86_64.rpm
│       ├── libss-1.45.6-5.el8.x86_64.rpm
│       ├── libssh-0.9.6-10.el8_8.x86_64.rpm
│       ├── libssh-config-0.9.6-10.el8_8.noarch.rpm
│       ├── libstdc++-8.5.0-18.el8.x86_64.rpm
│       ├── libtasn1-4.13-4.el8_7.x86_64.rpm
│       ├── libtirpc-1.1.4-8.el8.x86_64.rpm
│       ├── libunistring-0.9.9-3.el8.x86_64.rpm
│       ├── libusbx-1.0.23-4.el8.x86_64.rpm
│       ├── libutempter-1.1.6-14.el8.x86_64.rpm
│       ├── libuuid-2.32.1-42.el8_8.x86_64.rpm
│       ├── libverto-0.3.2-2.el8.x86_64.rpm
│       ├── libverto-devel-0.3.2-2.el8.x86_64.rpm
│       ├── libxcrypt-4.1.1-6.el8.x86_64.rpm
│       ├── libxcrypt-devel-4.1.1-6.el8.x86_64.rpm
│       ├── libxkbcommon-0.9.1-1.el8.x86_64.rpm
│       ├── libxml2-2.9.7-16.el8.x86_64.rpm
│       ├── libyaml-0.1.7-5.el8.x86_64.rpm
│       ├── libzstd-1.4.4-1.el8.x86_64.rpm
│       ├── lua-libs-5.3.4-12.el8.x86_64.rpm
│       ├── lvm2-2.03.14-9.el8.x86_64.rpm
│       ├── lvm2-libs-2.03.14-9.el8.x86_64.rpm
│       ├── lz4-libs-1.8.3-3.el8_4.x86_64.rpm
│       ├── memstrack-0.2.4-2.el8.x86_64.rpm
│       ├── modules.yaml
│       ├── mpfr-3.1.6-1.el8.x86_64.rpm
│       ├── ncurses-6.1-9.20180224.el8.x86_64.rpm
│       ├── ncurses-base-6.1-9.20180224.el8.noarch.rpm
│       ├── ncurses-libs-6.1-9.20180224.el8.x86_64.rpm
│       ├── nettle-3.4.1-7.el8.x86_64.rpm
│       ├── nftables-0.9.3-26.el8.x86_64.rpm
│       ├── npth-1.5-4.el8.x86_64.rpm
│       ├── nspr-4.34.0-3.el8_6.x86_64.rpm
│       ├── nss-3.79.0-11.el8_7.x86_64.rpm
│       ├── nss-softokn-3.79.0-11.el8_7.x86_64.rpm
│       ├── nss-softokn-freebl-3.79.0-11.el8_7.x86_64.rpm
│       ├── nss-sysinit-3.79.0-11.el8_7.x86_64.rpm
│       ├── nss-util-3.79.0-11.el8_7.x86_64.rpm
│       ├── openldap-2.4.46-18.el8.x86_64.rpm
│       ├── openssl-1.1.1k-9.el8_7.x86_64.rpm
│       ├── openssl-devel-1.1.1k-9.el8_7.x86_64.rpm
│       ├── openssl-libs-1.1.1k-9.el8_7.x86_64.rpm
│       ├── openssl-pkcs11-0.4.10-3.el8.x86_64.rpm
│       ├── os-prober-1.74-9.el8.x86_64.rpm
│       ├── p11-kit-0.23.22-1.el8.x86_64.rpm
│       ├── p11-kit-trust-0.23.22-1.el8.x86_64.rpm
│       ├── pam-1.3.1-25.el8.x86_64.rpm
│       ├── pcre2-10.32-3.el8_6.x86_64.rpm
│       ├── pcre2-devel-10.32-3.el8_6.x86_64.rpm
│       ├── pcre2-utf16-10.32-3.el8_6.x86_64.rpm
│       ├── pcre2-utf32-10.32-3.el8_6.x86_64.rpm
│       ├── pcre-8.42-6.el8.x86_64.rpm
│       ├── pigz-2.4-4.el8.x86_64.rpm
│       ├── pinentry-1.1.0-2.el8.x86_64.rpm
│       ├── pkgconf-1.4.2-1.el8.x86_64.rpm
│       ├── pkgconf-m4-1.4.2-1.el8.noarch.rpm
│       ├── pkgconf-pkg-config-1.4.2-1.el8.x86_64.rpm
│       ├── platform-python-3.6.8-51.el8_8.1.rocky.0.x86_64.rpm
│       ├── platform-python-pip-9.0.3-22.el8.rocky.0.noarch.rpm
│       ├── platform-python-setuptools-39.2.0-7.el8.noarch.rpm
│       ├── policycoreutils-2.9-24.el8.x86_64.rpm
│       ├── policycoreutils-python-utils-2.9-24.el8.noarch.rpm
│       ├── popt-1.18-1.el8.x86_64.rpm
│       ├── procps-ng-3.3.15-13.el8.x86_64.rpm
│       ├── publicsuffix-list-dafsa-20180723-1.el8.noarch.rpm
│       ├── python38-3.8.16-1.module+el8.8.0+1313+e073abae.1.x86_64.rpm
│       ├── python38-devel-3.8.16-1.module+el8.8.0+1313+e073abae.1.x86_64.rpm
│       ├── python38-libs-3.8.16-1.module+el8.8.0+1313+e073abae.1.x86_64.rpm
│       ├── python38-pip-19.3.1-6.module+el8.7.0+1063+20f2b9a4.noarch.rpm
│       ├── python38-pip-wheel-19.3.1-6.module+el8.7.0+1063+20f2b9a4.noarch.rpm
│       ├── python38-setuptools-41.6.0-5.module+el8.5.0+672+ab6eb015.noarch.rpm
│       ├── python38-setuptools-wheel-41.6.0-5.module+el8.5.0+672+ab6eb015.noarch.rpm
│       ├── python3-audit-3.0.7-4.el8.x86_64.rpm
│       ├── python3-dateutil-2.6.1-6.el8.noarch.rpm
│       ├── python3-dbus-1.2.4-15.el8.x86_64.rpm
│       ├── python3-decorator-4.2.1-2.el8.noarch.rpm
│       ├── python3-dnf-4.7.0-16.el8_8.noarch.rpm
│       ├── python3-dnf-plugins-core-4.0.21-19.el8_8.noarch.rpm
│       ├── python3-dnf-plugin-versionlock-4.0.21-19.el8_8.noarch.rpm
│       ├── python3-firewall-0.9.3-13.el8.noarch.rpm
│       ├── python3-gobject-base-3.28.3-2.el8.x86_64.rpm
│       ├── python3-gpg-1.13.1-11.el8.x86_64.rpm
│       ├── python3-hawkey-0.63.0-14.el8_8.x86_64.rpm
│       ├── python3-libcomps-0.1.18-1.el8.x86_64.rpm
│       ├── python3-libdnf-0.63.0-14.el8_8.x86_64.rpm
│       ├── python3-libs-3.6.8-51.el8_8.1.rocky.0.x86_64.rpm
│       ├── python3-libselinux-2.9-8.el8.x86_64.rpm
│       ├── python3-libsemanage-2.9-9.el8_6.x86_64.rpm
│       ├── python3-nftables-0.9.3-26.el8.x86_64.rpm
│       ├── python3-pip-wheel-9.0.3-22.el8.rocky.0.noarch.rpm
│       ├── python3-policycoreutils-2.9-24.el8.noarch.rpm
│       ├── python3-rpm-4.14.3-26.el8.x86_64.rpm
│       ├── python3-setools-4.3.0-3.el8.x86_64.rpm
│       ├── python3-setuptools-wheel-39.2.0-7.el8.noarch.rpm
│       ├── python3-six-1.11.0-8.el8.noarch.rpm
│       ├── python3-slip-0.6.4-13.el8.noarch.rpm
│       ├── python3-slip-dbus-0.6.4-13.el8.noarch.rpm
│       ├── python3-systemd-234-8.el8.x86_64.rpm
│       ├── python3-unbound-1.16.2-5.el8.x86_64.rpm
│       ├── readline-7.0-10.el8.x86_64.rpm
│       ├── repodata
│       │   ├── 08d4c745e983ce9ecaaf0fa61f131c35f5bd7cd1af912a64d5c882e9d7aac3ba-modules.yaml.gz
│       │   ├── 47b9d5b1cd1bcc3b1808c04fcedd65d4c4c7379b1fe663a339c600192a8a44f4-primary.xml.gz
│       │   ├── 4ff0a2dc2537089aefae88041d4e341b8d17f3033e394dc897226ca9a156b5d2-filelists.sqlite.bz2
│       │   ├── c3b968f97e34cb432c84cb6d1b9faf1fe42b30aeff5b9c9102fb79112eb81448-primary.sqlite.bz2
│       │   ├── e17e4bc535d59cac1ec618118563993efe0f3babcad8ce7d8723a6edd836cc42-other.sqlite.bz2
│       │   ├── f9506d5e44bde7c5209896c0fac1c010111d78b89ecd33869ec82027b4e5af2d-other.xml.gz
│       │   ├── fa84a06b5b8ddc4436fbb55ccb899a179c78230d14ae5ee9efa7db2e79f4b769-filelists.xml.gz
│       │   └── repomd.xml
│       ├── rocky-gpg-keys-8.8-1.8.el8.noarch.rpm
│       ├── rocky-release-8.8-1.8.el8.noarch.rpm
│       ├── rocky-repos-8.8-1.8.el8.noarch.rpm
│       ├── rpm-4.14.3-26.el8.x86_64.rpm
│       ├── rpm-build-libs-4.14.3-26.el8.x86_64.rpm
│       ├── rpm-libs-4.14.3-26.el8.x86_64.rpm
│       ├── rpm-plugin-selinux-4.14.3-26.el8.x86_64.rpm
│       ├── rpm-plugin-systemd-inhibit-4.14.3-26.el8.x86_64.rpm
│       ├── rsync-3.1.3-19.el8_7.1.x86_64.rpm
│       ├── sed-4.5-5.el8.x86_64.rpm
│       ├── selinux-policy-3.14.3-117.el8_8.2.noarch.rpm
│       ├── selinux-policy-targeted-3.14.3-117.el8_8.2.noarch.rpm
│       ├── setup-2.12.2-9.el8.noarch.rpm
│       ├── shadow-utils-4.6-17.el8.x86_64.rpm
│       ├── shared-mime-info-1.9-3.el8.x86_64.rpm
│       ├── socat-1.7.4.1-1.el8.x86_64.rpm
│       ├── sqlite-libs-3.26.0-18.el8_8.x86_64.rpm
│       ├── systemd-239-74.el8_8.2.x86_64.rpm
│       ├── systemd-libs-239-74.el8_8.2.x86_64.rpm
│       ├── systemd-pam-239-74.el8_8.2.x86_64.rpm
│       ├── systemd-udev-239-74.el8_8.2.x86_64.rpm
│       ├── tpm2-tss-2.3.2-4.el8.x86_64.rpm
│       ├── trousers-0.3.15-1.el8.x86_64.rpm
│       ├── trousers-lib-0.3.15-1.el8.x86_64.rpm
│       ├── tzdata-2023c-1.el8.noarch.rpm
│       ├── unbound-libs-1.16.2-5.el8.x86_64.rpm
│       ├── unzip-6.0-46.el8.x86_64.rpm
│       ├── util-linux-2.32.1-42.el8_8.x86_64.rpm
│       ├── which-2.21-18.el8.x86_64.rpm
│       ├── xfsprogs-5.0.0-11.el8_8.x86_64.rpm
│       ├── xkeyboard-config-2.28-1.el8.noarch.rpm
│       ├── xz-5.2.4-4.el8_6.x86_64.rpm
│       ├── xz-libs-5.2.4-4.el8_6.x86_64.rpm
│       ├── yum-utils-4.0.21-19.el8_8.noarch.rpm
│       ├── zlib-1.2.11-21.el8_7.x86_64.rpm
│       └── zlib-devel-1.2.11-21.el8_7.x86_64.rpm
├── setup-all.sh
├── setup-container.sh
├── setup-offline.sh
├── setup-py.sh
├── start-nginx.sh
└── start-registry.sh

22 directories, 416 files

```
##  10. 搬运介质

（如何下载介质节点和部署节点不是同一节点，你需要搬运打包下载的介质）
打包下载的介质(docker 安装介质、ansible运行镜像介质、kubernetes介质)传输到离线环境：

```bash
cd ../
tar czvf kubespray-offline-2.21.0-1.tar.gz kubespray-offline/
```
登陆离线部署节点（bastion01），然后在输出目录中运行以下脚本：

```bash
tar zxvf kubespray-offline-2.21.0-1.tar.gz
```
## 11. 离线部署节点安装 docker
(离线部署节点bastion01)

如果你的下载介质节点和部署堡垒机不是同一个节点，需要将下载到文件夹rpms/的.rpm包导入到离线的 bastion01。
有关于 docker 的yum 源

```bash
yum -y install docker-ce
```
如果没有有关于 docker 的yum 源

```bash
sudo dnf install rpms/*.rpm --disablerepo '*'
sudo systemctl enable --now docker
```
> 注意：这可能因为离线安装 rpm 包产生依赖包安装失败。

## 12. 安装 docker insecure registry
(离线部署节点bastion01)
在离线环境下的部署节点安装docker后，我们需要部署镜像仓库，为了方便拉取镜像，这里我们部署 `insecure registry`。
- [docker  安装 insecure registry](https://ghostwritten.blog.csdn.net/article/details/105926147)

```bash
mkdir /var/lib/registry
docker run -d --restart=always --name registry -p 35000:5000 -v /var/lib/registry:/var/lib/registry registry:latest
docker ps
```
配置域名

```bash
echo "10.60.0.30 registry.demo" >>/etc/hosts
```
配置 docker 

```bash
cat <<EOF> /etc/docker/daemon.json
{
  "insecure-registries" : ["registry.demo:35000"]
}
EOF
systemctl daemon-reload &&systemctl restart docker && systemctl status docker
docker tag registry:latest registry.demo:35000/registry:latest 
docker push registry.demo:35000/registry:latest 
```

## 13. 部署 nginx 与 镜像入库
(离线部署节点bastion01)

该配置部署的目的主要完成 nginx部署，为了其他节点拉取各类文件、工具包；并且完成镜像推送入库步骤。

```bash
cd kubespray-offline/kubespray-offline-2.21.0-1/outputs
```
### 13.1 配置 config.sh
修改 `KUBESPRAY_VERSION:-2.22.1`，并且添加`LOCAL_REGISTRY='registry.demo:35000'`。
```bash
cat config.sh 
#!/bin/bash
KUBESPRAY_VERSION=${KUBESPRAY_VERSION:-2.22.1}
#KUBESPRAY_VERSION=${KUBESPRAY_VERSION:-master}

LOCAL_REGISTRY='registry.demo:35000'
REGISTRY_PORT=${REGISTRY_PORT:-35000}

```
### 13.2 配置 setup-docker.sh
注释掉`./install-containerd.sh` 安装，并将`NERDCTL`变量改为`/usr/bin/docker`
```bash
cat setup-docker.sh 
#!/bin/bash

# install containerd
#./install-containerd.sh

# Load images
echo "==> Load  nginx images"
#NERDCTL=/usr/local/bin/nerdctl
NERDCTL=/usr/bin/docker
cd ./images

for f in  docker.io_library_nginx-*.tar.gz; do
    sudo $NERDCTL load -i $f
done

if [ -f kubespray-offline-container.tar.gz ]; then
    sudo $NERDCTL load -i kubespray-offline-container.tar.gz
fi

```

### 13.3 配置 start-nginx.sh
部署改为 `docker` 运行容器。
```bash
cat start-nginx.sh 
#!/bin/bash

source ./config.sh

BASEDIR="."
if [ ! -d images ] && [ -d ../outputs ]; then
    BASEDIR="../outputs"  # for tests
fi
BASEDIR=$(cd $BASEDIR; pwd)

NGINX_IMAGE=nginx:1.23

echo "===> Start nginx"
sudo /usr/bin/docker run -d \
    --network host \
    --restart always \
    --name nginx \
    -v ${BASEDIR}:/usr/share/nginx/html \
    ${NGINX_IMAGE}

```

### 13.4 配置 load-push-all-images.sh 
将`NERDCTL`变量改为`/usr/bin/docker`。
```bash
$ cat load-push-all-images.sh 
#!/bin/bash

source ./config.sh

LOCAL_REGISTRY=${LOCAL_REGISTRY:-"localhost:${REGISTRY_PORT}"}
#NERDCTL=/usr/local/bin/nerdctl
NERDCTL=/usr/bin/docker

BASEDIR="."
if [ ! -d images ] && [ -d ../outputs ]; then
    BASEDIR="../outputs"  # for tests
fi

load_images() {
    for image in $BASEDIR/images/*.tar.gz; do
        echo "===> Loading $image"
        sudo $NERDCTL load -i $image
    done
}

push_images() {
    images=$(cat $BASEDIR/images/*.list)
    for image in $images; do

        # Removes specific repo parts from each image for kubespray
        newImage=$image
        for repo in registry.k8s.io k8s.gcr.io gcr.io docker.io quay.io; do
            newImage=$(echo ${newImage} | sed s@^${repo}/@@)
        done

        newImage=${LOCAL_REGISTRY}/${newImage}

        echo "===> Tag ${image} -> ${newImage}"
        sudo $NERDCTL tag ${image} ${newImage}

        echo "===> Push ${newImage}"
        sudo $NERDCTL push ${newImage}
    done
}

load_images
push_images

```

### 13.5 配置 set-all.sh
注释掉`run ./setup-container.sh` 与 `run ./start-registry.sh`
```bash
$ cat setup-all.sh 
#!/bin/bash

run() {
    echo "=> Running: $*"
    $* || {
        echo "Failed in : $*"
        exit 1
    }
}

# prepare
#run ./setup-container.sh
run ./setup-docker.sh
# start web server
run ./start-nginx.sh

# setup local repositories
run ./setup-offline.sh

# setup python
run ./setup-py.sh

# start private registry
#run ./start-registry.sh

# load and push all images to registry
run ./load-push-all-images.sh

```
执行：

```bash
./setup-all.sh
```

检查

```bash

$ docker ps
CONTAINER ID   IMAGE             COMMAND                  CREATED          STATUS             PORTS                                         NAMES
b3ab95d1444d   nginx:1.23        "/docker-entrypoint.…"   21 minutes ago   Up 21 minutes                                                    nginx
966d43523ed6   registry:latest   "/entrypoint.sh /etc…"   6 hours ago      Up About an hour   0.0.0.0:35000->5000/tcp, :::35000->5000/tcp   registry

$ docker images |grep 35000
192.168.23.30:35000/registry                                  latest           4bb5ea59f8e0   6 weeks ago     24MB
registry.demo:35000/registry                                  latest           4bb5ea59f8e0   6 weeks ago     24MB
registry.dmeo:35000/registry                                  latest           4bb5ea59f8e0   6 weeks ago     24MB
registry.demo:35000/nginx                                     1.23             a7be6198544f   2 months ago    142MB
registry.demo:35000/kube-apiserver                            v1.26.5          25c2ecde661f   2 months ago    134MB
registry.demo:35000/kube-scheduler                            v1.26.5          200132c1d91a   2 months ago    56.5MB
registry.demo:35000/kube-controller-manager                   v1.26.5          a7403c147a51   2 months ago    123MB
registry.demo:35000/kube-proxy                                v1.26.5          08440588500d   2 months ago    65.6MB
registry.demo:35000/registry                                  2.8.1            f5352b75f67e   2 months ago    25.9MB
registry.demo:35000/library/registry                          2.8.1            f5352b75f67e   2 months ago    25.9MB
registry.demo:35000/ingress-nginx/controller                  v1.7.1           2db0b57c8712   3 months ago    289MB
registry.demo:35000/ghcr.io/kube-vip/kube-vip                 v0.5.12          1227540e2a61   3 months ago    41.7MB
registry.demo:35000/jetstack/cert-manager-webhook             v1.11.1          b22616882946   3 months ago    47.2MB
registry.demo:35000/jetstack/cert-manager-controller          v1.11.1          b23ed1d12779   3 months ago    62.2MB
registry.demo:35000/jetstack/cert-manager-cainjector          v1.11.1          4b822c353dc8   3 months ago    39.7MB
registry.demo:35000/cpa/cluster-proportional-autoscaler       v1.8.8           b6d1a4be0743   3 months ago    37.9MB
registry.demo:35000/calico/typha                              v3.25.1          dfed532bc8bd   4 months ago    68.2MB
registry.demo:35000/calico/kube-controllers                   v3.25.1          212faac284a2   4 months ago    72.8MB
registry.demo:35000/calico/apiserver                          v3.25.1          44b3b862e431   4 months ago    84.5MB
registry.demo:35000/calico/cni                                v3.25.1          a0138614e609   4 months ago    201MB
registry.demo:35000/calico/pod2daemon-flexvol                 v3.25.1          958f4d519550   4 months ago    14.7MB
registry.demo:35000/calico/node                               v3.25.1          cae61b85e9b4   4 months ago    248MB
registry.demo:35000/flannel/flannel                           v0.21.4          11ae74319a21   4 months ago    64.1MB
registry.demo:35000/metrics-server/metrics-server             v0.6.3           817bbe3f2e51   4 months ago    68.9MB
registry.demo:35000/cilium/hubble-ui                          v0.11.0          b555a2c7b3de   4 months ago    50.6MB
registry.demo:35000/cilium/hubble-ui-backend                  v0.11.0          0631ce248fa6   4 months ago    48.3MB
registry.demo:35000/metallb/controller                        v0.13.9          26952499c302   5 months ago    64.1MB
registry.demo:35000/metallb/speaker                           v0.13.9          697605b35935   5 months ago    114MB
registry.demo:35000/cilium/cilium                             v1.13.0          c54e1f1dc89e   5 months ago    481MB
registry.demo:35000/cilium/operator                           v1.13.0          cffa7764cd78   5 months ago    125MB
registry.demo:35000/cilium/hubble-relay                       v1.13.0          ce6166711ac2   5 months ago    48.8MB
registry.demo:35000/dns/k8s-dns-node-cache                    1.22.18          9eaf430eed84   5 months ago    66.4MB
registry.demo:35000/flannelcni/flannel-cni-plugin             v1.2.0           cebccd102ad9   8 months ago    7.97MB
registry.demo:35000/library/haproxy                           2.6.6-alpine     2fa962112cbd   8 months ago    22.9MB
registry.demo:35000/coreos/etcd                               v3.5.6           094bf0036469   8 months ago    181MB
registry.demo:35000/library/nginx                             1.23.2-alpine    19dd4d73108a   8 months ago    23.5MB
registry.demo:35000/kubeovn/kube-ovn                          v1.10.7          f2e01dcf2cb6   8 months ago    406MB
registry.demo:35000/rancher/local-path-provisioner            v0.0.23          9621e18c3388   9 months ago    37.4MB
registry.demo:35000/pause                                     3.9              e6f181688397   9 months ago    744kB
registry.demo:35000/kubernetesui/dashboard                    v2.7.0           07655ddf2eeb   10 months ago   246MB
registry.demo:35000/envoyproxy/envoy                          v1.22.5          e9c4ee2ce720   11 months ago   131MB
registry.demo:35000/cloudnativelabs/kube-router               v1.5.1           6080a224cdd1   12 months ago   103MB
registry.demo:35000/sig-storage/local-volume-provisioner      v2.5.0           84fe61c6a33a   12 months ago   132MB
registry.demo:35000/kubernetesui/metrics-scraper              v1.0.8           115053965e86   14 months ago   43.8MB
registry.demo:35000/coredns/coredns                           v1.9.3           5185b96f0bec   14 months ago   48.8MB
registry.demo:35000/cilium/certgen                            v0.1.8           a283370c8d83   18 months ago   44.4MB
registry.demo:35000/sig-storage/csi-snapshotter               v5.0.0           c5bdb516176e   19 months ago   55.2MB
registry.demo:35000/sig-storage/csi-node-driver-registrar     v2.4.0           f45c8a305a0b   21 months ago   19.8MB
registry.demo:35000/ghcr.io/k8snetworkplumbingwg/multus-cni   v3.8             c65d3833b509   22 months ago   290MB
registry.demo:35000/sig-storage/snapshot-controller           v4.2.1           223fc919ab6a   23 months ago   51.4MB
registry.demo:35000/sig-storage/csi-resizer                   v1.3.0           1df30f0e2555   23 months ago   54.2MB
registry.demo:35000/k8scloudprovider/cinder-csi-plugin        v1.22.0          e4c74b94269d   23 months ago   214MB
registry.demo:35000/sig-storage/csi-provisioner               v3.0.0           fe0f921f3c92   24 months ago   56.7MB
registry.demo:35000/sig-storage/csi-attacher                  v3.3.0           37f46af926da   24 months ago   53.7MB
registry.demo:35000/weaveworks/weave-npc                      2.8.1            7f92d556d4ff   2 years ago     39.3MB
registry.demo:35000/weaveworks/weave-kube                     2.8.1            df29c0a4002c   2 years ago     89MB
registry.demo:35000/amazon/aws-alb-ingress-controller         v1.1.9           4b1d22ffb3c0   2 years ago     40.6MB
registry.demo:35000/amazon/aws-ebs-csi-driver                 v0.5.0           187fd7ffef67   3 years ago     435MB
registry.demo:35000/external_storage/rbd-provisioner          v2.1.1-k8s1.11   9fb54e49f9bf   4 years ago     405MB
registry.demo:35000/external_storage/cephfs-provisioner       v2.1.0-k8s1.11   cac658c0a096   4 years ago     401MB
registry.demo:35000/mirantis/k8s-netchecker-server            v1.2.2           3fe402881a14   5 years ago     123MB
registry.demo:35000/mirantis/k8s-netchecker-agent             v1.2.2           bf9a79a05945   5 years ago     5.64MB

```

##  14. 配置互信

```bash
ssh-keygen
for i in {30..35};do ssh-copy-id root@10.60.0.$i;done
```

##  15. 启动容器部署环境

```bash
docker  run --name kubespray-offline-ansible --network=host --rm -itd -v "$(pwd)"/kubespray-offline-2.21.0-1:/kubespray -v "${HOME}"/.ssh:/root/.ssh  -v /etc/hosts:/etc/hosts quay.io/kubespray/kubespray:v2.22.1  bash
```

## 16. 部署前准备
检查批量通信、镜像仓库地址域名解析、可能存在缺失的内核加载模块。

```bash
ansible -i inventory/sample/inventory.ini all -m ping
ansible  -i inventory/sample/inventory.ini all  -m lineinfile -a "path=/etc/hosts line='10.60.0.30 registry.demo'"
ansible  -i inventory/sample/inventory.ini all  -m lineinfile -a "path=/etc/rc.local line='modprobe br_netfilter\nmodprobe ip_conntrack'"

```

 配置 `extract-kubespray.sh`
保持 `kubespray-v2.22.1` 内容不变，注释掉 `apply patches`内容。

```bash
#!/bin/bash

cd $(dirname $0)
CURRENT_DIR=$(pwd)
source ./config.sh

KUBESPRAY_TARBALL=files/kubespray-${KUBESPRAY_VERSION}.tar.gz
DIR=kubespray-${KUBESPRAY_VERSION}

if [ -d $DIR ]; then
    echo "${DIR} already exists."
    exit 0
fi

if [ ! -e $KUBESPRAY_TARBALL ]; then
    echo "$KUBESPRAY_TARBALL does not exist."
    exit 1
fi

tar xvzf $KUBESPRAY_TARBALL || exit 1

# apply patches
#sleep 1 # avoid annoying patch error in shared folders.
#if [ -d $CURRENT_DIR/patches/${KUBESPRAY_VERSION} ]; then
#    for patch in $CURRENT_DIR/patches/${KUBESPRAY_VERSION}/*.patch; do
#        if [[ -f "${patch}" ]]; then
#          echo "===> Apply patch: $patch"
#          (cd $DIR && patch -p1 < $patch)
#        fi
#    done
#fi

```
执行：

```bash
./extract-kubespray.sh
cd kubespray-2.22.1
```

##  17. 编写 inventory.ini

```bash
vim inventory/sample/inventory.ini
```

```bash
[all]
kube-master01 ansible_host=10.60.0.31
kube-prom01 ansible_host=10.60.0.32
kube-node01 ansible_host=10.60.0.33
kube-node02 ansible_host=10.60.0.34
kube-node03 ansible_host=10.60.0.35

[kube_control_plane]
kube-master01

[etcd]
kube-master01

[kube_node]
kube-prom01
kube-node01
kube-node02
kube-node03

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr

```

## 18. 编写 offline.yml


```bash
vim  outputs/kubespray-2.22.1/inventry/sample/group_vars/all/offline.yml
```

```bash
http_server: "http://10.60.0.30"
registry_host: "registry.demo:35000"

containerd_insecure_registries: # Kubespray #8340
  "registry.demo:35000": "http://registry.demo:35000"

files_repo: "{{ http_server }}/files"
yum_repo: "{{ http_server }}/rpms"
ubuntu_repo: "{{ http_server }}/debs"

# Registry overrides
kube_image_repo: "{{ registry_host }}"
gcr_image_repo: "{{ registry_host }}"
docker_image_repo: "{{ registry_host }}"
quay_image_repo: "{{ registry_host }}"

# Download URLs: See roles/download/defaults/main.yml of kubespray.
kubeadm_download_url: "{{ files_repo }}/kubernetes/{{ kube_version }}/kubeadm"
kubectl_download_url: "{{ files_repo }}/kubernetes/{{ kube_version }}/kubectl"
kubelet_download_url: "{{ files_repo }}/kubernetes/{{ kube_version }}/kubelet"
# etcd is optional if you **DON'T** use etcd_deployment=host
etcd_download_url: "{{ files_repo }}/kubernetes/etcd/etcd-{{ etcd_version }}-linux-amd64.tar.gz"
cni_download_url: "{{ files_repo }}/kubernetes/cni/cni-plugins-linux-{{ image_arch }}-{{ cni_version }}.tgz"
crictl_download_url: "{{ files_repo }}/kubernetes/cri-tools/crictl-{{ crictl_version }}-{{ ansible_system | lower }}-{{ image_arch }}.tar.gz"
# If using Calico
calicoctl_download_url: "{{ files_repo }}/kubernetes/calico/{{ calico_ctl_version }}/calicoctl-linux-{{ image_arch }}"
# If using Calico with kdd
calico_crds_download_url: "{{ files_repo }}/kubernetes/calico/{{ calico_version }}.tar.gz"

runc_download_url: "{{ files_repo }}/runc/{{ runc_version }}/runc.{{ image_arch }}"
nerdctl_download_url: "{{ files_repo }}/nerdctl-{{ nerdctl_version }}-{{ ansible_system | lower }}-{{ image_arch }}.tar.gz"
containerd_download_url: "{{ files_repo }}/containerd-{{ containerd_version }}-linux-{{ image_arch }}.tar.gz"

#containerd_insecure_registries:
#    "{{ registry_addr }}"："{{ registry_host }}"

# CentOS/Redhat/AlmaLinux/Rocky Linux
## Docker / Containerd
docker_rh_repo_base_url: "{{ yum_repo }}/docker-ce/$releasever/$basearch"
docker_rh_repo_gpgkey: "{{ yum_repo }}/docker-ce/gpg"

# Fedora
## Docker
docker_fedora_repo_base_url: "{{ yum_repo }}/docker-ce/{{ ansible_distribution_major_version }}/{{ ansible_architecture }}"
docker_fedora_repo_gpgkey: "{{ yum_repo }}/docker-ce/gpg"
## Containerd
containerd_fedora_repo_base_url: "{{ yum_repo }}/containerd"
containerd_fedora_repo_gpgkey: "{{ yum_repo }}/docker-ce/gpg"
```



## 19. 部署 offline repo
Deploy offline repo configurations which use your yum_repo/ubuntu_repo to all target nodes using ansible.

First, copy offline setup playbook to kubespray directory.

```bash
$ ls playbooks
ansible_version.yml  cluster.yml  facts.yml  legacy_groups.yml  recover_control_plane.yml  remove_node.yml  reset.yml  scale.yml  upgrade_cluster.yml
$ ls ../playbook/
offline-repo.yml  roles

cp -r ../playbook .
```

```bash
ansible -i inventory/sample/inventory.ini all -m shell -a  "mv /etc/yum.repos.d /tmp"
ansible -i inventory/sample/inventory.ini all -m shell -a  "mkdir /etc/yum.repos.d"
#ansible -i inventory/sample/inventory.ini all -m copy -a "src=/tmp/yum.repos.d/offline.repo dest=/etc/yum.repos.d/"

ansible-playbook -i  inventory/sample/inventory.ini playbook/offline-repo.yml 
```
登到批量对象节点检查 `offline.repo`是否正常

```bash
[root@kube-master01 ~]# yum repolist -v
Loaded plugins: builddep, changelog, config-manager, copr, debug, debuginfo-install, download, generate_completion_cache, groups-manager, needs-restarting, playground, repoclosure, repodiff, repograph, repomanage, reposync, system-upgrade
YUM version: 4.7.0
cachedir: /var/cache/dnf
Offline repo for kubespray                                                                                                                                          8.0 MB/s | 407 kB     00:00    
Last metadata expiration check: 0:00:01 ago on Thu 03 Aug 2023 02:56:04 PM CST.
Repo-id            : offline-repo
Repo-name          : Offline repo for kubespray
Repo-revision      : 1691030836
Repo-updated       : Thu 03 Aug 2023 10:47:18 AM CST
Repo-pkgs          : 299
Repo-available-pkgs: 299
Repo-size          : 217 M
Repo-baseurl       : http://10.60.0.30/rpms/local
Repo-expire        : 172,800 second(s) (last: Thu 03 Aug 2023 02:56:04 PM CST)
Repo-filename      : /etc/yum.repos.d/offline.repo
Total packages: 299

```

##  20. 开始部署
执行部署：

```bash
ansible-playbook -i inventory/sample/inventory.ini --become --become-user=root cluster.yml
```
输出：

```bash
TASK [network_plugin/calico : Check if inventory match current cluster configuration] **************************************************************************************************************
ok: [kube-master01] => {
    "changed": false,
    "msg": "All assertions passed"
}
Thursday 03 August 2023  07:37:19 +0000 (0:00:00.279)       0:34:08.863 ******* 
Thursday 03 August 2023  07:37:20 +0000 (0:00:00.174)       0:34:09.037 ******* 
Thursday 03 August 2023  07:37:20 +0000 (0:00:00.241)       0:34:09.278 ******* 

PLAY RECAP *****************************************************************************************************************************************************************************************
kube-master01              : ok=738  changed=144  unreachable=0    failed=0    skipped=1254 rescued=0    ignored=8   
kube-node01                : ok=514  changed=89   unreachable=0    failed=0    skipped=763  rescued=0    ignored=1   
kube-node02                : ok=514  changed=89   unreachable=0    failed=0    skipped=763  rescued=0    ignored=1   
kube-node03                : ok=514  changed=89   unreachable=0    failed=0    skipped=763  rescued=0    ignored=1   
kube-prom01                : ok=514  changed=89   unreachable=0    failed=0    skipped=764  rescued=0    ignored=1   
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

Thursday 03 August 2023  07:37:21 +0000 (0:00:00.804)       0:34:10.083 ******* 
=============================================================================== 
kubernetes/preinstall : Install packages requirements -------------------------------------------------------------------------------------------------------------------------------------- 61.36s
kubernetes-apps/ansible : Kubernetes Apps | Lay Down CoreDNS templates --------------------------------------------------------------------------------------------------------------------- 31.75s
kubernetes-apps/ansible : Kubernetes Apps | Start Resources -------------------------------------------------------------------------------------------------------------------------------- 31.09s
kubernetes/control-plane : kubeadm | Initialize first master ------------------------------------------------------------------------------------------------------------------------------- 22.55s
network_plugin/calico : Start Calico resources --------------------------------------------------------------------------------------------------------------------------------------------- 21.60s
bootstrap-os : Assign inventory name to unconfigured hostnames (non-CoreOS, non-Flatcar, Suse and ClearLinux, non-Fedora) ------------------------------------------------------------------ 20.21s
network_plugin/calico : Calico | Create calico manifests ----------------------------------------------------------------------------------------------------------------------------------- 18.85s
bootstrap-os : Gather host facts to get ansible_os_family ---------------------------------------------------------------------------------------------------------------------------------- 17.53s
kubernetes/kubeadm : Join to cluster ------------------------------------------------------------------------------------------------------------------------------------------------------- 17.31s
download : download_container | Download image if required --------------------------------------------------------------------------------------------------------------------------------- 17.00s
download : download_container | Download image if required --------------------------------------------------------------------------------------------------------------------------------- 15.03s
Gather necessary facts (network) ----------------------------------------------------------------------------------------------------------------------------------------------------------- 12.99s
policy_controller/calico : Create calico-kube-controllers manifests ------------------------------------------------------------------------------------------------------------------------ 12.85s
Gather minimal facts ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- 12.84s
container-engine/validate-container-engine : Populate service facts ------------------------------------------------------------------------------------------------------------------------ 12.27s
kubernetes/preinstall : Ensure kube-bench parameters are set ------------------------------------------------------------------------------------------------------------------------------- 11.33s
bootstrap-os : Install libselinux python package ------------------------------------------------------------------------------------------------------------------------------------------- 11.31s
kubernetes-apps/ansible : Kubernetes Apps | Lay Down nodelocaldns Template ------------------------------------------------------------------------------------------------------------------ 9.74s
policy_controller/calico : Start of Calico kube controllers --------------------------------------------------------------------------------------------------------------------------------- 9.61s
container-engine/containerd : containerd | Remove orphaned binary --------------------------------------------------------------------------------------------------------------------------- 9.59s

```

## 21. 检查集群
如果（bastion01在线）
```bash
curl -LO https://dl.k8s.io/release/v1.26.5/bin/linux/amd64/kubectl
chmod 755 kubectl
mv kubectl /usr/local/bin
```
如果（bastion01离线）
从kube-master01拷贝 kubectl 二进制命令。

配置 `kubeconfig`
从`kube-master01` 节点 `/etc/kubernetes/admin.conf` 拷贝至bastion01节点 `/root/.kube/config`
并修改`https://127.0.0.1:6443` 为 `https://10.60.0.31:6443`


```bash
[root@bastion01 ~]# kubectl get node
NAME            STATUS   ROLES           AGE    VERSION
kube-master01   Ready    control-plane   111m   v1.26.5
kube-node01     Ready    <none>          109m   v1.26.5
kube-node02     Ready    <none>          109m   v1.26.5
kube-node03     Ready    <none>          109m   v1.26.5
kube-prom01     Ready    <none>          109m   v1.26.5
[root@bastion01 ~]# kubectl get pod -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-7c8987bf4c-ptcn7   1/1     Running   0          106m
kube-system   calico-node-4bw5w                          1/1     Running   0          108m
kube-system   calico-node-7765r                          1/1     Running   0          108m
kube-system   calico-node-pl7xr                          1/1     Running   0          108m
kube-system   calico-node-rfgcj                          1/1     Running   0          108m
kube-system   calico-node-vc5bg                          1/1     Running   0          108m
kube-system   coredns-7995d6cdf9-6p99g                   1/1     Running   0          105m
kube-system   coredns-7995d6cdf9-r7m6d                   1/1     Running   0          105m
kube-system   dns-autoscaler-5bff9b4dbc-w5xfh            1/1     Running   0          105m
kube-system   kube-apiserver-kube-master01               1/1     Running   1          111m
kube-system   kube-controller-manager-kube-master01      1/1     Running   2          111m
kube-system   kube-proxy-6zfgl                           1/1     Running   0          109m
kube-system   kube-proxy-gzcqp                           1/1     Running   0          109m
kube-system   kube-proxy-hfvv6                           1/1     Running   0          109m
kube-system   kube-proxy-nxbd8                           1/1     Running   0          109m
kube-system   kube-proxy-w2gq7                           1/1     Running   0          109m
kube-system   kube-scheduler-kube-master01               1/1     Running   1          111m
kube-system   nginx-proxy-kube-node01                    1/1     Running   0          108m
kube-system   nginx-proxy-kube-node02                    1/1     Running   0          110m
kube-system   nginx-proxy-kube-node03                    1/1     Running   0          108m
kube-system   nginx-proxy-kube-prom01                    1/1     Running   0          108m
kube-system   nodelocaldns-99gz8                         1/1     Running   0          105m
kube-system   nodelocaldns-dkjm8                         1/1     Running   0          105m
kube-system   nodelocaldns-fdrrk                         1/1     Running   0          105m
kube-system   nodelocaldns-h5cwl                         1/1     Running   0          105m
kube-system   nodelocaldns-p2mtz                         1/1     Running   0          105m

#登陆kube-master01检查二进制版本
[root@kube-master01 ~]# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.5", GitCommit:"890a139214b4de1f01543d15003b5bda71aae9c7", GitTreeState:"clean", BuildDate:"2023-05-17T14:13:34Z", GoVersion:"go1.19.9", Compiler:"gc", Platform:"linux/amd64"}
[root@kube-master01 ~]# kubectl version
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.5", GitCommit:"890a139214b4de1f01543d15003b5bda71aae9c7", GitTreeState:"clean", BuildDate:"2023-05-17T14:14:46Z", GoVersion:"go1.19.9", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v4.5.7
Server Version: version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.5", GitCommit:"890a139214b4de1f01543d15003b5bda71aae9c7", GitTreeState:"clean", BuildDate:"2023-05-17T14:08:49Z", GoVersion:"go1.19.9", Compiler:"gc", Platform:"linux/amd64"}

[root@kube-master01 ~]# nerdctl images
REPOSITORY                                                 TAG        IMAGE ID        CREATED        PLATFORM       SIZE         BLOB SIZE
registry.demo:35000/calico/cni                             v3.25.1    28564c983f7f    2 hours ago    linux/amd64    191.9 MiB    85.7 MiB
registry.demo:35000/calico/kube-controllers                v3.25.1    58c2c5273f9b    2 hours ago    linux/amd64    69.4 MiB     30.4 MiB
registry.demo:35000/calico/node                            v3.25.1    68fc6b7a097f    2 hours ago    linux/amd64    243.5 MiB    84.2 MiB
registry.demo:35000/calico/pod2daemon-flexvol              v3.25.1    d8f18274dfa8    2 hours ago    linux/amd64    14.1 MiB     6.8 MiB
registry.demo:35000/coredns/coredns                        v1.9.3     bdb36ee882c1    2 hours ago    linux/amd64    46.5 MiB     14.1 MiB
registry.demo:35000/cpa/cluster-proportional-autoscaler    v1.8.8     93dc3d84636a    2 hours ago    linux/amd64    39.6 MiB     11.1 MiB
registry.demo:35000/dns/k8s-dns-node-cache                 1.22.18    9b73daadc0f9    2 hours ago    linux/amd64    67.2 MiB     28.4 MiB
registry.demo:35000/kube-apiserver                         v1.26.5    aaccce05bf83    2 hours ago    linux/amd64    131.1 MiB    33.7 MiB
registry.demo:35000/kube-controller-manager                v1.26.5    f7d2bc04e4a2    2 hours ago    linux/amd64    121.0 MiB    30.7 MiB
registry.demo:35000/kube-proxy                             v1.26.5    bf00a41c76d1    2 hours ago    linux/amd64    66.4 MiB     20.5 MiB
registry.demo:35000/kube-scheduler                         v1.26.5    5ea7328a5507    2 hours ago    linux/amd64    57.2 MiB     16.7 MiB
registry.demo:35000/pause                                  3.9        0fc1f3b764be    2 hours ago    linux/amd64    728.0 KiB    311.6 KiB

[root@kube-master01 ~]# nerdctl ps
CONTAINER ID    IMAGE                                                             COMMAND                   CREATED        STATUS    PORTS    NAMES
2a93ba3ed744    registry.demo:35000/pause:3.9                                     "/pause"                  2 hours ago    Up                 k8s://kube-system/calico-node-7765r
2b513d588474    registry.demo:35000/calico/node:v3.25.1                           "start_runit"             2 hours ago    Up                 k8s://kube-system/calico-node-7765r/calico-node
3ad4019476fa    registry.demo:35000/pause:3.9                                     "/pause"                  2 hours ago    Up                 k8s://kube-system/kube-controller-manager-kube-master01
3b77682ca1a5    registry.demo:35000/pause:3.9                                     "/pause"                  2 hours ago    Up                 k8s://kube-system/dns-autoscaler-5bff9b4dbc-w5xfh
4a87563cb6ee    registry.demo:35000/kube-scheduler:v1.26.5                        "kube-scheduler --au…"    2 hours ago    Up                 k8s://kube-system/kube-scheduler-kube-master01/kube-scheduler
57586720c7a7    registry.demo:35000/dns/k8s-dns-node-cache:1.22.18                "/node-cache -locali…"    2 hours ago    Up                 k8s://kube-system/nodelocaldns-dkjm8/node-cache
60b7bf09e1d9    registry.demo:35000/pause:3.9                                     "/pause"                  2 hours ago    Up                 k8s://kube-system/kube-apiserver-kube-master01
63dfca44ad67    registry.demo:35000/cpa/cluster-proportional-autoscaler:v1.8.8    "/cluster-proportion…"    2 hours ago    Up                 k8s://kube-system/dns-autoscaler-5bff9b4dbc-w5xfh/autoscaler
6d800426cad1    registry.demo:35000/kube-apiserver:v1.26.5                        "kube-apiserver --ad…"    2 hours ago    Up                 k8s://kube-system/kube-apiserver-kube-master01/kube-apiserver
72bb920c0912    registry.demo:35000/pause:3.9                                     "/pause"                  2 hours ago    Up                 k8s://kube-system/kube-scheduler-kube-master01
7882f0251b5c    registry.demo:35000/coredns/coredns:v1.9.3                        "/coredns -conf /etc…"    2 hours ago    Up                 k8s://kube-system/coredns-7995d6cdf9-6p99g/coredns
8f119e9592ab    registry.demo:35000/pause:3.9                                     "/pause"                  2 hours ago    Up                 k8s://kube-system/nodelocaldns-dkjm8
99c7ee458548    registry.demo:35000/pause:3.9                                     "/pause"                  2 hours ago    Up                 k8s://kube-system/coredns-7995d6cdf9-6p99g
a88d209b5803    registry.demo:35000/pause:3.9                                     "/pause"                  2 hours ago    Up                 k8s://kube-system/kube-proxy-nxbd8
b8138b2d694d    registry.demo:35000/kube-proxy:v1.26.5                            "/usr/local/bin/kube…"    2 hours ago    Up                 k8s://kube-system/kube-proxy-nxbd8/kube-proxy
e5a14d84a013    registry.demo:35000/kube-controller-manager:v1.26.5               "kube-controller-man…"    2 hours ago    Up                 k8s://kube-system/kube-controller-manager-kube-master01/kube-controller-manager

```

参考：

- [https://github.com/tmurakam/kubespray-offline](https://github.com/tmurakam/kubespray-offline)
- [https://github.com/kubernetes-sigs/kubespray](https://github.com/kubernetes-sigs/kubespray)



