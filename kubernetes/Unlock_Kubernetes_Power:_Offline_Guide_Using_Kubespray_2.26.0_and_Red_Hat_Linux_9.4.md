![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/f09918dd1b764fa2a99f9ac460b71526.jpeg#pic_center)




# 1. 简介

Kubespray 是一个开源项目，旨在通过 Ansible 简化生产就绪的 Kubernetes 集群的部署。它提供了灵活且可定制的框架，支持多种云提供商和本地环境，包括 AWS、Google Cloud Engine、Azure、OpenStack、vSphere 和裸金属环境。这种多样性使用户能够根据需求选择合适的基础设施来运行 Kubernetes 集群。Kubespray 支持创建高可用的 Kubernetes 集群，确保即使在节点故障时，应用程序也能保持正常运行。此外，用户可以选择多种网络插件，如 Calico、Flannel、Canal、Cilium 和 Weave，以便根据具体需求定制网络解决方案。Kubespray 兼容多种流行的 Linux 发行版，包括 CoreOS、Debian、Ubuntu、CentOS/RHEL、Fedora 和 openSUSE，使其适用于广泛的操作系统环境。安装过程相对简单，用户只需安装 Ansible 和所需的 Python 包，然后设置清单并执行 Ansible playbook 来完成 Kubernetes 集群的部署。Kubespray 的 GitHub 页面提供了全面的文档，包括入门指南、部署变量和网络配置等内容，帮助用户快速上手。此外，Kubespray 拥有活跃的社区，通过 Slack 等渠道为用户提供支持。总之，Kubespray 是一个强大的工具，可以高效地在各种环境中部署 Kubernetes 集群，其灵活性和多样性使其成为希望在基础设施中实施 Kubernetes 的组织的理想选择。对于希望实现高可用性和可定制化的用户来说，Kubespray 提供了一种可靠且易于使用的解决方案，是现代云原生应用开发和管理的重要工具。有关更多详细信息和资源，用户可以访问其 GitHub 仓库，以获取最新的更新和社区支持。

# 2. 要求规格


| ip  | role          | hostname      | cpu | mem | disk                     | description                   |
|-----|---------------|---------------|-----|-----|--------------------------|-------------------------------|
| IP1 | bastion       | bastion01     | 8   | 16G | ssd系统盘：100G<br>ssd盘：100G | install kubernetes & registry |
| IP2 | control Plane | kube-master01 | 8   | 16G | ssd系统盘：100G              |  kubernetes Components        |
| IP3 | worker        | kube-node01   | 16  | 32G | ssd系统盘：100G<br>ssd盘：100G | install app                   |
| IP4 | worker        | kube-node02   | 16  | 32G | ssd系统盘：100G<br>ssd盘：100G | install app                   |



# 3. 创建虚拟机模版

192.168.21.6-rhel-9.6-templ
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/beccc7d2cae44424809692af6f20788b.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/c8f6d9d21cf741a38e369d829b093622.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/9c9711933f6445388fc4a9fd95a2d695.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/f22948d44cea44bb87bea256c12e4e7a.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/6fc67101ea584bf088790787ddf2e29c.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/d70ca9eaa38b4feaba51fb67ecb0066b.png)

## 3.1 网卡配置

```bash
cat /etc/NetworkManager/system-connections/ens192.nmconnection
[connection]
id=ens192
type=ethernet
autoconnect-priority=-999
interface-name=ens192
timestamp=1730254410

[ethernet]

[ipv4]
method=manual
address0=192.168.21.6/20,192.168.21.1
dns=8.8.8.8;192.168.21.2


[ipv6]
method=disabled

[proxy]
```

```bash
nmcli connection reload && nmcli connection down ens192 && nmcli connection up ens192
```

## 3.2 订阅

```bash

$ subscription-manager register
Registering to: subscription.rhsm.redhat.com:443/subscription
Username: xxx@xx.com
Password: 
The system has been registered with ID: 4d08e201-e5405c5f5d1
The registered system name is: localhost.localdomain

$ dnf repolist --verbose
Loaded plugins: builddep, changelog, config-manager, copr, debug, debuginfo-install, download, generate_completion_cache, groups-manager, kpatch, needs-restarting, notify-packagekit, playground, product-id, repoclosure, repodiff, repograph, repomanage, reposync, subscription-manager, system-upgrade, uploadprofile
Updating Subscription Management repositories.
DNF version: 4.14.0
cachedir: /var/cache/dnf
Last metadata expiration check: 0:02:53 ago on Wed 30 Oct 2024 02:53:11 PM CST.
Repo-id            : rhel-9-for-x86_64-appstream-rpms
Repo-name          : Red Hat Enterprise Linux 9 for x86_64 - AppStream (RPMs)
Repo-revision      : 1730247754
Repo-updated       : Wed 30 Oct 2024 08:22:33 AM CST
Repo-pkgs          : 19,639
Repo-available-pkgs: 18,927
Repo-size          : 69 G
Repo-baseurl       : https://cdn.redhat.com/content/dist/rhel9/9/x86_64/appstream/os
Repo-expire        : 86,400 second(s) (last: Wed 30 Oct 2024 02:53:10 PM CST)
Repo-filename      : /etc/yum.repos.d/redhat.repo

Repo-id            : rhel-9-for-x86_64-baseos-rpms
Repo-name          : Red Hat Enterprise Linux 9 for x86_64 - BaseOS (RPMs)
Repo-revision      : 1730247532
Repo-updated       : Wed 30 Oct 2024 08:18:52 AM CST
Repo-pkgs          : 7,326
Repo-available-pkgs: 7,326
Repo-size          : 18 G
Repo-baseurl       : https://cdn.redhat.com/content/dist/rhel9/9/x86_64/baseos/os
Repo-expire        : 86,400 second(s) (last: Wed 30 Oct 2024 02:53:11 PM CST)
Repo-filename      : /etc/yum.repos.d/redhat.repo
Total packages: 26,965

$ dnf -y update
```

# 3.3 关闭防火墙与selinux

```bash
$ systemctl disable firewalld.service
$ setenforce 0 && getenforce
$ sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

```
关机。

# 4. 克隆 kubernetes 节点

关机再克隆虚拟机

> 注意 存储选择 精简模式 ： `Thin provision`，节省空间。

## 4.1 bastion01 

名字：192.168.23.85-rhel-9.4-bastion01



![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a8c7c7ddf9c342a0b6779a6c88584e51.png)


![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/9562f6d08b4f42de9b1fa0f255e65c0d.png)

ens192 配置192.168.23.85 连接外网下载介质。

```bash
$ cat /etc/NetworkManager/system-connections/ens192.nmconnection 
[connection]
id=ens192
type=ethernet
autoconnect-priority=-999
interface-name=ens192
timestamp=1730254410

[ethernet]

[ipv4]
method=manual
address1=192.168.23.85/20,192.168.21.1
dns=192.168.21.2;


[ipv6]
method=disabled

[proxy]
```


![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/23571f47e2954c2ea6803bbb8630a3d9.png)

配置ens224 内网地址 172.168.23.85 与kubernetes集群同网段。

```bash
$ cat /etc/NetworkManager/system-connections/ens224.nmconnection 
[connection]
id=ens224
type=ethernet
autoconnect-priority=-999
interface-name=ens224
timestamp=1730254410

[ethernet]

[ipv4]
method=manual
address1=172.168.23.85/20,172.168.21.1
dns=172.168.23.85;


[ipv6]
method=disabled

[proxy]
```

```bash
$ nmcli connection reload && nmcli connection down ens224 && nmcli connection up ens224
```

```bash
hostnamectl set-hostname bastion01
```

## 4.2 kube-master01

172.168.23.84-rhel-9.4-kube-master01
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/c54bf550c6284397904b9037a8297af2.png)


![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/cbfb907930994f4a8f6ebcc5d15aa2dd.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/396d8aaba6914f90a17401414e84ff98.png)
kube-master01 配置ens192 ip 是172.168.23.86 离线地址。

```bash
$ cat /etc/NetworkManager/system-connections/ens192.nmconnection 
[connection]
id=ens192
type=ethernet
autoconnect-priority=-999
interface-name=ens192
timestamp=1730254410

[ethernet]

[ipv4]
method=manual
address1=172.168.23.86/20,172.168.21.1
dns=172.168.23.85;


[ipv6]
method=disabled

[proxy]
```

```bash
hostnamectl set-hostname kube-master01
```

## 4.3 kube-node01

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/1db6d06b7ef8427389f8be298b867706.png)

kube-node01 配置ens192 ip 是172.168.23.87 离线地址。
```bash
$ cat /etc/NetworkManager/system-connections/ens192.nmconnection 
[connection]
id=ens192
type=ethernet
autoconnect-priority=-999
interface-name=ens192
timestamp=1730254410

[ethernet]

[ipv4]
method=manual
address1=172.168.23.87/20,172.168.21.1
dns=172.168.23.85;


[ipv6]
method=disabled

[proxy]
```

```bash
hostnamectl set-hostname kube-node01
```

## 4.4 kube-node02

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/0684f628c4734295beec7e3b1bc14283.png)

kube-node02 配置ens192 ip 是172.168.23.88 离线地址。
```bash
$ cat /etc/NetworkManager/system-connections/ens192.nmconnection 
[connection]
id=ens192
type=ethernet
autoconnect-priority=-999
interface-name=ens192
timestamp=1730254410

[ethernet]

[ipv4]
method=manual
address1=172.168.23.88/20,172.168.21.1
dns=172.168.23.85;


[ipv6]
method=disabled

[proxy]
```

```bash
hostnamectl set-hostname kube-node02
```

# 5. 打快照

四台虚拟机配置完基础配置关闭分别打快照，名称为`base-network`。方便还原。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/f6063aae990145a88e42c1c497789fb4.png)




# 6. 配置代理



方便拉取在“局域网”内拉取不到的介质，加速下载。

## 6.1 主机配置代理

192.168.23.85 配置：

```bash
$ dnf -y install git
$ vim  /root/.bashrc
proxy_url="http://192.168.xxx.xx:7890"
export no_proxy="10.0.0.0/8,192.168.16.0/20,localhost,127.0.0.0/8,.svc,.svc.cluster-27,.coding.net,.tencentyun.com,.myqcloud.com"
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

$ source  /root/.bashrc
```

## 6.2 yum 配置代理

```bash
$ vim /etc/yum.conf
[main]
gpgcheck=1
installonly_limit=3
clean_requirements_on_remove=True
best=True
skip_if_unavailable=False
proxy=http://192.168.2x.x1:7890 # 添加
```

## 6.3 容器配置代理

```bash
$ vim /etc/profile.d/http_proxy.sh
export HTTP_PROXY=http://192.168.2x.x1:7890
export HTTPS_PROXY=http://192.168.2x.x1:7890
export NO_PROXY=example.com,172.168.21.0.0/20

$ source /etc/profile.d/http_proxy.sh
```


# 7. 配置 DNS （可选）

```bash
$ dnf -y install unbound
$ cat /etc/unbound/unbound.conf
    server:
        interface: 0.0.0.0
        port: 53
        do-ip4: yes
        do-udp: yes
        access-control: 0.0.0.0/0 allow      
        access-control: 172.168.23.0/20 allow
        access-control: 10.251.26.0/20 allow
        verbosity: 1

    local-data: "kube-master01 A 172.168.23.86"
    local-data-ptr: "172.168.23.86 kube-master01"
    local-data: "kube-node01 A 172.168.23.87"
    local-data-ptr: "172.168.23.87 kube-node01"
    local-data: "kube-node02 A 172.168.23.88"
    local-data-ptr: "172.168.23.88 kube-node02"


    forward-zone:
        name: "."
        forward-addr: "192.168.21.2"

$ systemctl restart unbound && systemctl status unbound && systemctl enable unbound
```

# 8. 下载介质

```bash
$ wget https://github.com/kubespray-offline/kubespray-offline/archive/refs/tags/v2.26.0-0.zip
$ unzip v2.26.0-0.zip 
$ cd kubespray-offline-2.26.0-0/
```

定义下载内容
```bash
$ cat download-all.sh 
#!/bin/bash

run() {
    echo "=> Running: $*"
    $* || {
        echo "Failed in : $*"
        exit 1
    }
}

source ./config.sh

#run ./install-docker.sh
#run ./install-nerdctl.sh
run ./precheck.sh
#run ./prepare-pkgs.sh || exit 1
#run ./prepare-py.sh
run ./get-kubespray.sh
#if $ansible_in_container; then
#    run ./build-ansible-container.sh
#else
#    run ./pypi-mirror.sh
#fi
run ./download-kubespray-files.sh
run ./download-additional-containers.sh
run ./create-repo.sh
run ./copy-target-scripts.sh

echo "Done."
```

执行：

```bash
$ bash download-all.sh
$ podman pull quay.io/kubespray/kubespray:v2.26.0
$ podman save -o kubespray-v2.26.0.tar.gz quay.io/kubespray/kubespray:v2.26.0
```

下载完成后，打包。

```bash
$ tar zcvf kubespray-offline-2.26.0-0.tar.gz kubespray-offline-2.26.0-0
```

# 9. 上传介质

上传至离线环境存储空间大的目录。


```bash
$ tar zxvf kubespray-offline-2.26.0-0.tar.gz
```

# 10. 部署镜像仓库

定义容器工具，rhel 9.4 用默认podman
REGISTRY_DIR：定义镜像存储目录，根据存储目录大小定义。

```bash
$ cd kubespray-offline-2.26.0-0/outputs/
$ podman load -i images/docker.io_library_registry-2.8.2.tar.gz
$ vim  start-registry.sh 
#!/bin/bash

source ./config.sh

REGISTRY_IMAGE=${REGISTRY_IMAGE:-registry:2.8.2}
REGISTRY_DIR=${REGISTRY_DIR:-/var/lib/registry}

if [ ! -e $REGISTRY_DIR ]; then
    sudo mkdir $REGISTRY_DIR
fi

echo "===> Start registry"
#sudo /usr/local/bin/nerdctl run -d \
sudo podman run -d \
    --network host \
    -e REGISTRY_HTTP_ADDR=172.168.23.85:${REGISTRY_PORT} \
    --restart always \
    --name registry \
    -v $REGISTRY_DIR:/var/lib/registry \
    $REGISTRY_IMAGE || exit 1

$ sh start-registry.sh 
```

# 11. 推送镜像入库

```bash
$ vim  load-push-all-images.sh 
#!/bin/bash

source ./config.sh

LOCAL_REGISTRY=${LOCAL_REGISTRY:-"172.168.23.85:${REGISTRY_PORT}"}
#NERDCTL=/usr/local/bin/nerdctl
NERDCTL=/usr/bin/podman

BASEDIR="."
if [ ! -d images ] && [ -d ../outputs ]; then
    BASEDIR="../outputs"  # for tests
fi

load_images() {
    for image in $BASEDIR/images/*.tar.gz; do
        echo "===> Loading $image"
        sudo $NERDCTL load -i $image || exit 1
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
        sudo $NERDCTL tag ${image} ${newImage} || exit 1

        echo "===> Push ${newImage}"
        sudo $NERDCTL push ${newImage} || exit 1
    done
}

load_images
push_images
```

执行：

```bash
$ load-push-all-images.sh 
```

# 12. 启动 nginx
修改容器工具

```bash
$ cd /root/kubespray-offline-2.26.0-0/outputs/
$ podman load -i images/docker.io_library_nginx-1.25.2.tar.gz 
$ vim start-nginx.sh
#!/bin/bash

source ./config.sh

BASEDIR="."
if [ ! -d images ] && [ -d ../outputs ]; then
    BASEDIR="../outputs"  # for tests
fi
BASEDIR=$(cd $BASEDIR; pwd)

NGINX_IMAGE=nginx:1.25.2

echo "===> Start nginx"
#sudo /usr/local/bin/nerdctl run -d \
sudo podman run -d \
    --network host \
    --restart always \
    --name nginx \
    -v ${BASEDIR}:/usr/share/nginx/html \
    ${NGINX_IMAGE} || exit 1
```

执行：

```bash
$ bash start-nginx.sh
```

# 13. 配置互信

```bash
$ ssh-keygen
$ dnf -y install sshpass
$ vim hosts.txt
172.168.23.85
172.168.23.86
172.168.23.87
172.168.23.88

$ for host in $(cat hosts.txt); do sshpass -p 'root' ssh-copy-id -o StrictHostKeyChecking=no -i ~/.ssh/id_rsa.pub root@$host;done
```

# 14. 配置 extract-kubespray.sh

```bash
$ cd /root/kubespray-offline-2.26.0-0/outputs
$ sh extract-kubespray.sh 
$ cp -r playbook kubespray-2.26.0/
```

# 15. 启动部署容器

```bash
$ cd /root/kubespray-offline-2.26.0-0
$ podman load -i kubespray-v2.26.0.tar.gz
$ podman run --name kubespray-offline-ansible --network=host  -it -v "/root/kubespray-offline-2.26.0-0/outputs/kubespray-2.26.0:/kubespray" -v "${HOME}"/.ssh:/root/.ssh  -v /etc/hosts:/etc/hosts quay.io/kubespray/kubespray:v2.26.0  bash
$ cp inventory/sample/inventory.ini inventory/sample/inventory.ini_bak
$ vim inventory/sample/inventory.ini
[all]
kube-master01 ansible_host=172.168.23.86
kube-node01 ansible_host=172.168.23.87
kube-node02 ansible_host=172.168.23.88

[bastion]

[kube_control_plane]
kube-master01

[etcd]
kube-master01

[kube_node]
kube-node01
kube-node02

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr

$ vim /root/.bashrc
...
alias ansible='ansible -i /kubespray/inventory/sample/inventory.ini'
...
$ source /root/.bashrc
$ ansible all  -m ping

```

# 16. 离线配置

可能的定制需求：

pod & service网段：

```bash
$ vim inventory/sample/group_vars/k8s_cluster/k8s-cluster.yml
...
kube_service_addresses: 10.233.0.0/18
kube_pods_subnet: 10.233.64.0/18
kube_network_node_prefix: 24
enable_dual_stack_networks: false
kube_service_addresses_ipv6: fd85:ee78:d8a6:8607::1000/116
kube_pods_subnet_ipv6: fd85:ee78:d8a6:8607::1:0000/112
kube_network_node_prefix_ipv6: 120
```

## 16.1 编写 offline.yml

```bash
$ vim inventory/sample/group_vars/all/offline.yml
http_server: "http://172.168.23.85"
registry_host: "172.168.23.85:35000"

containerd_insecure_registries: # Kubespray #8340
  "172.168.23.85:35000": "http://172.168.23.85:35000"

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

## 16.2 配置 containerd.yaml

```bash
$ vim inventory/sample/group_vars/all/containerd.yml
...
containerd_registries_mirrors:
- prefix: 172.168.23.85:35000
  mirrors:
  - host: http://172.168.23.85:35000
    capabilities: ["pull", "resolve"]
    skip_verify: true
```

## 16.3 部署 offline repo

```bash
$ ansible all -m shell -a  "mv /etc/yum.repos.d /tmp"
$ ansible all -m shell -a  "mkdir /etc/yum.repos.d"
$ ansible-playbook -i inventory/sample/inventory.ini  playbook/offline-repo.yml
$ ansible  all  -m shell -a "sed -i 's/enabled=1/enabled=0/' /etc/yum/pluginconf.d/subscription-manager.conf"
$ ansible all -m shell -a  "mv /etc/yum.repos.d/redhat.repo /tmp/redhat.repo_bak"
$ ansible  all  -m shell -a "yum repolist --verbose"
```

# 17. 部署 k8s

```bash
ansible-playbook -i inventory/sample/inventory.ini  cluster.yml
```


输出：

```bash
PLAY RECAP ***********************************************************************************************************************************
kube-master01              : ok=632  changed=117  unreachable=0    failed=0    skipped=1081 rescued=0    ignored=6   
kube-node01                : ok=416  changed=66   unreachable=0    failed=0    skipped=635  rescued=0    ignored=1   
kube-node02                : ok=416  changed=66   unreachable=0    failed=0    skipped=631  rescued=0    ignored=1   

Wednesday 30 October 2024  12:07:45 +0000 (0:00:00.233)       0:08:55.604 ***** 
=============================================================================== 
download : Download_container | Download image if required --------------------------------------------------------------------------- 24.10s
kubernetes/kubeadm : Join to cluster ------------------------------------------------------------------------------------------------- 21.17s
download : Download_container | Download image if required --------------------------------------------------------------------------- 20.81s
download : Download_container | Download image if required --------------------------------------------------------------------------- 18.13s
kubernetes/control-plane : Upload certificates so they are fresh and not expired ----------------------------------------------------- 15.54s
network_plugin/calico : Wait for calico kubeconfig to be created --------------------------------------------------------------------- 15.46s
kubernetes/control-plane : Kubeadm | Initialize first master ------------------------------------------------------------------------- 14.90s
download : Download_container | Download image if required --------------------------------------------------------------------------- 14.45s
download : Download_container | Download image if required --------------------------------------------------------------------------- 12.63s
network_plugin/calico : Check if calico ready ---------------------------------------------------------------------------------------- 11.49s
etcd : Reload etcd -------------------------------------------------------------------------------------------------------------------- 9.40s
download : Download_container | Download image if required ---------------------------------------------------------------------------- 7.93s
etcd : Configure | Check if etcd cluster is healthy ----------------------------------------------------------------------------------- 6.07s
download : Extract_file | Unpacking archive ------------------------------------------------------------------------------------------- 5.18s
download : Download_container | Download image if required ---------------------------------------------------------------------------- 5.08s
kubernetes-apps/ansible : Kubernetes Apps | Lay Down CoreDNS templates ---------------------------------------------------------------- 5.06s
etcd : Configure | Ensure etcd is running --------------------------------------------------------------------------------------------- 5.04s
download : Download_container | Download image if required ---------------------------------------------------------------------------- 5.02s
network_plugin/calico : Start Calico resources ---------------------------------------------------------------------------------------- 4.53s
network_plugin/cni : CNI | Copy cni plugins ------------------------------------------------------------------------------------------- 4.15s
```

# 18. 配置 kubeconfig

```bash
$ mkdir /root/.kube
$ scp root@172.168.23.86:/usr/local/bin/kubectl /usr/local/bin/
$ scp root@172.168.23.86:/etc/kubernetes/admin.conf  /root/.kube/config
$ kubectl get node
NAME            STATUS   ROLES           AGE     VERSION
kube-master01   Ready    control-plane   5m31s   v1.30.4
kube-node01     Ready    <none>          4m37s   v1.30.4
kube-node02     Ready    <none>          4m37s   v1.30.4



$ cat >> ~/.bashrc << EOF 
alias kg='kubectl get'
alias k='kubectl'
alias kd='kubectl describe pods'
alias ke='kubectl explain'
alias ka='kubectl apply'
EOF

$ source ~/.bashrc


$ kubectl get pod -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-54bb4659fd-mdfbd   1/1     Running   0          11m
kube-system   calico-node-4s6sj                          1/1     Running   0          12m
kube-system   calico-node-62d75                          1/1     Running   0          12m
kube-system   calico-node-99n5r                          1/1     Running   0          12m
kube-system   coredns-fb8d8dbfd-6vrbw                    1/1     Running   0          11m
kube-system   coredns-fb8d8dbfd-f5cjb                    1/1     Running   0          11m
kube-system   dns-autoscaler-5496d9b966-rghmj            1/1     Running   0          11m
kube-system   kube-apiserver-kube-master01               1/1     Running   1          13m
kube-system   kube-controller-manager-kube-master01      1/1     Running   3          13m
kube-system   kube-proxy-7xbk7                           1/1     Running   0          12m
kube-system   kube-proxy-k7stp                           1/1     Running   0          12m
kube-system   kube-proxy-lg4g2                           1/1     Running   0          12m
kube-system   kube-scheduler-kube-master01               1/1     Running   2          13m
kube-system   nginx-proxy-kube-node01                    1/1     Running   0          12m
kube-system   nginx-proxy-kube-node02                    1/1     Running   0          12m
kube-system   nodelocaldns-cc228                         1/1     Running   0          11m
kube-system   nodelocaldns-ddfdj                         1/1     Running   0          11m
kube-system   nodelocaldns-vxwtl                         1/1     Running   0          11m
                                  
```

# 19. 参考

- [Setting up HTTP Proxy variables for podman](https://access.redhat.com/solutions/3939131)
- [https://github.com/kubernetes-sigs/kubespray](https://github.com/kubernetes-sigs/kubespray)
- [https://github.com/kubespray-offline/kubespray-offline](https://github.com/kubespray-offline/kubespray-offline)
- [Registering the system and managing subscriptions](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/configuring_basic_system_settings/assembly_registering-the-system-and-managing-subscriptions_configuring-basic-system-settings#assembly_registering-the-system-and-managing-subscriptions_configuring-basic-system-settings)
