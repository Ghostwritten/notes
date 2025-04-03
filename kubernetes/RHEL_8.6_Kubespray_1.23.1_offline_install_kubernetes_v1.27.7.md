![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/478c23cb1c1a435f82b19184b67965d2.png)



## 1. 预备条件
9 个节点：

| 角色 | ip  | cpu| mem| disk|ios|kernel|
|--|--|--|--|--|--|--|
| download |10.168.2.10、10.90.0.10   |4  |8G|100G|rhel 8.7|4.18+|
| bastion01 |10.90.0.11  |4  |8G|100G|rhel 8.7|4.18+|
| kube-master01 |10.90.0.12  |4  |8G|100G|rhel 8.7|4.18+|
| kube-node01 |10.90.0.13  |16  |32G|100G,200G |rhel 8.7|4.18+|
| kube-node02 |10.90.0.14  |16 |32G|100G,200G|rhel 8.7|4.18+|
| kube-node03 |10.90.0.15  |16  |32G|100G,200G|rhel 8.7|4.18+|


注意：机器一定要检查：
- 是否有时间同步服务器：ntp or chrony
- dns
- yum


## 2. 配置网卡

download01
```bash
cat /etc/sysconfig/network-scripts/ifcfg-ens192 
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
IPADDR=10.168.2.10
PREFIX=16
GATEWAY=10.168.1.1
DNS1=8.8.8.8

[root@download ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens224 
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
NAME=ens224
DEVICE=ens224
ONBOOT=yes
IPADDR=10.90.0.10
PREFIX=16

```




## 3. download01 节点 介质下载
（download）

介质列表：

- rhel-8.6-x86_64-dvd.iso （用于安装系统和配置yum源）
- kubespray-offline-2.23.0-0.tar.gz（离线安装 kubernetes）


在下载所有介质之前需除了容器拉取镜像代理，主机也要配置代理否则，是否无法下载成功的。

主机代理：

```bash
export https_proxy=http://192.168.21.101:7890
export http_proxy=http://192.168.21.101:7890
export no_proxy=192.168.21.2,10.0.0.0/8,192.168.0.0/16,localhost,127.0.0.0/8,.coding.net,.tencentyun.com,.myqcloud.com
```


下载最新版本 2.23.0-0： [https://github.com/kubespray-offline/kubespray-offline/releases](https://github.com/kubespray-offline/kubespray-offline/releases)

```bash
cd /root
wget https://github.com/kubespray-offline/kubespray-offline/archive/refs/tags/v2.23.0-0.zip
 unzip v2.23.0-0.zip
```

修改默认下载的kubernetes 版本

```bash
$ vim config.sh 
#!/bin/bash

# Kubespray version to download. Use "master" for latest master branch.
KUBESPRAY_VERSION=${KUBESPRAY_VERSION:-2.23.1}  #修改
#KUBESPRAY_VERSION=${KUBESPRAY_VERSION:-master}

# container runtime for preparation node
docker=${docker:-docker}
#docker=${docker:-/usr/local/bin/nerdctl}

# Run ansible in container?
ansible_in_container=${ansible_in_container:-false}

```

选择容器工具，为下载镜像做准备，不同的系统镜像管理工具可能不同，根据客户的给予的下载介质环境可能也是不同的。我们要有两种方式。一种是docker，一种是nerdctl。

###  3. 安装 nerdctl

```bash
mkdir nerdctl && cd nerdctl 
```

参考：[nerdctl install【nerdctl 安装】](https://blog.csdn.net/xixihahalelehehe/article/details/134264754)

```bash
sh install-nerdctl.sh d

cd ../ && tar zcvf nerdctl.tar.gz nerdctl
```


> 注意：大陆环境需要配置容器代理

```bash
 cat /etc/systemd/system/containerd.service.d/http-proxy.conf 
[Service]
Environment="HTTP_PROXY=http://192.168.21.101:7890" "HTTPS_PROXY=http://192.168.21.101:7890" "NO_PROXY=localhost,127.0.0.0/8,169.0.0.0/8,10.0.0.0/8,192.168.0.0/16,*.coding.net,*.tencentyun.com,*.myqcloud.com"
#重启 containerd
systemctl daemon-reload && systemctl restart containerd
```


安装成功后，下载介质执行：

```bash
./download-all.sh
```
所有工件都存储在./outputs 目录。
此脚本调用以下所有脚本：
- `prepare-pkgs.sh`
Setup python, etc.
- `prepare-py.sh`
Setup python venv, install required python packages.
- `get-kubespray.sh`
Download and extract kubespray, if KUBESPRAY_DIR does not exist.
- `pypi-mirror.sh`
Download PyPI mirror files
- `download-kubespray-files.sh`
Download kubespray offline files (containers, files, etc)
- `download-additional-containers.sh`
Download additional containers.
You can add any container image repoTag to imagelists/*.txt.
- `create-repo.sh`
Download RPM or DEB repositories.
- `copy-target-scripts.sh`
Copy scripts for target node.


用时 30分钟 下载完成介质。也许一次会下载失败，需要调试反复执行，或者根据环境改动一些内容。
> 注意：如果你喜欢通过容器部署集群，需要下载 kubespray 镜像，默认 `quay.io/kubespray/kubespray:v2.21.0`  ，存在一个 pip 包依赖（jmespath 1.0.1），下面镜像已重新编译。

                              
```bash
nerdctl pull ghostwritten/kubespray:v2.21-a94b893e2
nerdctl save -o kubespray:v2.21-a94b893e2.tar ghostwritten/kubespray:v2.21-a94b893e2
```


打包下载的介质：

```bash
tar zcvf kubespray-offline-2.23.0-0-kubernetes-v1.27.7-origin.tar.gz kubespray-offline-2.23.0-0
```
将介质传输至离线的 bastion01

## 4. bastion01节点配置 yum 源

（bastion01离线状态）



参考：[https://www.redhat.com/sysadmin/apache-yum-dnf-repo](https://www.redhat.com/sysadmin/apache-yum-dnf-repo)

```bash
mkdir /mnt/cdrom
mount -o loop /root/kubernetes-offline/rhel-8.6-x86_64-dvd.iso /mnt/cdrom
```



本地配置yum源

```bash
cat local.repo 
[BaseOS]
name=BaseOS
baseurl=file:///mnt/cdrom/BaseOS
enable=1
gpgcheck=0

[AppStream]
name=AppStream
baseurl=file:///mnt/cdrom/AppStream
enabled=1
gpgcheck=0

```
测试

```bash
$ yum repolist --verbose
Loaded plugins: builddep, changelog, config-manager, copr, debug, debuginfo-install, download, generate_completion_cache, groups-manager, kpatch, needs-restarting, playground, product-id, repoclosure, repodiff, repograph, repomanage, reposync, subscription-manager, uploadprofile
Updating Subscription Management repositories.
Unable to read consumer identity

This system is not registered with an entitlement server. You can use subscription-manager to register.

YUM version: 4.7.0
cachedir: /var/cache/dnf
Last metadata expiration check: 0:35:00 ago on Thu 19 Oct 2023 05:45:22 PM CST.
Repo-id            : AppStream
Repo-name          : AppStream
Repo-revision      : 1656402327
Repo-updated       : Tue 28 Jun 2022 03:45:28 PM CST
Repo-pkgs          : 6,609
Repo-available-pkgs: 5,345
Repo-size          : 8.6 G
Repo-baseurl       : file:///mnt/cdrom/AppStream
Repo-expire        : 172,800 second(s) (last: Thu 19 Oct 2023 05:45:22 PM CST)
Repo-filename      : /etc/yum.repos.d/local.repo

Repo-id            : BaseOS
Repo-name          : BaseOS
Repo-revision      : 1656402376
Repo-updated       : Tue 28 Jun 2022 03:46:17 PM CST
Repo-pkgs          : 1,714
Repo-available-pkgs: 1,712
Repo-size          : 1.3 G
Repo-baseurl       : file:///mnt/cdrom/BaseOS
Repo-expire        : 172,800 second(s) (last: Thu 19 Oct 2023 05:45:21 PM CST)
Repo-filename      : /etc/yum.repos.d/local.repo
Total packages: 8,323

```

```bash

yum clean all
yum install ntp
```
其他节点使用源

```bash
$ yum -y install httpd

$ vim /etc/httpd/conf/httpd.conf


listen 81

# vim  /etc/httpd/conf.d/define.conf
<VirtualHost *:81>
        ServerName  www.XXX-ym.com
        DocumentRoot /mnt/cdrom
</VirtualHost>

# vim  /etc/httpd/conf.d/permission.conf
<Directory /mnt/cdrom>
	Require all granted      
</Directory>


$ systemctl  restart  httpd
$ chcon -R --reference=/var/www  /mnt/cdrom  //调整SELinux属性

$ systemctl  restart  httpd

```
远程节点配置yum源

```bash
$ cat http_iso.repo 
[BaseOS] 
name=http_iso 
baseurl=http://10.70.0.254/BaseOS
gpgcheck=0 
enabled=1
[AppStream] 
name=http_iso 
baseurl=http://10.70.0.254/AppStream
gpgcheck=0 
enabled=1

```
## 5. bastion01 离线安装 nerdctl 

打包下载的介质传输到离线环境：

```bash
cd ../
tar zcvf kubespray-offline-2.23.0-0-kubernetes-v1.27.7-origin.tar.gz kubespray-offline-2.23.0-0
```
登陆部署节点（registry01），然后在输出目录中运行以下脚本：

```bash
tar zxvf kubespray-offline-2.23.0-0-kubernetes-v1.27.7-origin.tar.gz
```

```bash

```

```bash
#!/bin/bash

BASE_DIR="$(dirname "$(readlink -f "${0}")")"
name=`basename $0`

tar xvf ${BASE_DIR}/files/containerd-1.7.5-linux-amd64.tar.gz --strip-components=1 -C /usr/local/bin
cp ${BASE_DIR}/files/runc/v1.1.9/runc.amd64 /usr/local/bin/runc
chmod 755 /usr/local/bin/runc

tar zxvf ${BASE_DIR}/files/nerdctl-1.5.0-linux-amd64.tar.gz -C /tmp/
cp /tmp/nerdctl /usr/local/bin

sudo mkdir -p \
     /etc/systemd/system/containerd.service.d \
     /etc/containerd \
     /var/lib/containerd \
     /run/containerd

#配置 containerd

cat <<EOF> /etc/systemd/system/containerd.service
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

cat <<EOF> /etc/containerd/config.toml
version = 2
root = "/var/lib/containerd"
state = "/run/containerd"
oom_score = 0

[grpc]
  address = "/run/containerd/containerd.sock"
  uid = 0
  gid = 0

[debug]
  address = "/run/containerd/debug.sock"
  uid = 0
  gid = 0
  level = "info"

[metrics]
  address = ""
  grpc_histogram = false

[cgroup]
  path = ""

[plugins]
  [plugins."io.containerd.grpc.v1.cri".containerd]
    default_runtime_name = "runc"
    snapshotter = "overlayfs"
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
    runtime_type = "io.containerd.runc.v2"
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    systemdCgroup = true
EOF

# restart containerd
sudo systemctl daemon-reload && sudo systemctl enable --now containerd &&  systemctl status  containerd | grep Active


#CNI plugins install


sudo mkdir -p /opt/cni/bin && tar xvzf  ${BASE_DIR}/files/kubernetes/cni/cni-plugins-linux-amd64-v1.3.0.tgz -C /opt/cni/bin
```
执行：

```bash
sh -x offline-install-nerdctl-containerd.sh
```


## 6. 安装l insecure registry
修改 config.sh
```bash
$ vim /root/kubespray-offline-2.23.0-0/outputs/config.sh 
#!/bin/bash
KUBESPRAY_VERSION=${KUBESPRAY_VERSION:-2.23.1}  #修改
#KUBESPRAY_VERSION=${KUBESPRAY_VERSION:-master}

REGISTRY_PORT=${REGISTRY_PORT:-35000}

```
> 确保  `/etc/resolv.conf` 存在，否则会报错，`touch /etc/resolv.conf`


载入 registry 镜像

```bash
$ nerdctl load -i images/docker.io_library_registry-2.8.2.tar.gz 
unpacking docker.io/library/registry:2.8.2 (sha256:43408412c6b1e1ec25a0d8ac2ff65e8e7c4e5d9d1ee1bab6ea513589b9e841a0)...
Loaded image: registry:2.8.2

```

```bash
sh start-registry.sh 
===> Start registry
be7b2cd72cd0f39588bb6083f730ac840e08de766143010d61cd8d9f4215289f
```

配置 /etc/hosts

```bash
$ cat /etc/hosts
10.70.0.73 registry.upm
```

测试
```bash
nerdctl tag registry:2.8.2 registry.upm:35000/registry:2.8.2
nerdctl --insecure-registry=true push  registry.upm:35000/registry:2.8.2
nerdctl --insecure-registry=true pull  registry.upm:35000/registry:2.8.2
```
输出：

```bash
$ nerdctl --insecure-registry=true push  registry.upm:35000/registry:2.8.2
INFO[0000] pushing as a reduced-platform image (application/vnd.docker.distribution.manifest.v2+json, sha256:43408412c6b1e1ec25a0d8ac2ff65e8e7c4e5d9d1ee1bab6ea513589b9e841a0) 
WARN[0000] skipping verifying HTTPS certs for "registry.upm:35000" 
manifest-sha256:43408412c6b1e1ec25a0d8ac2ff65e8e7c4e5d9d1ee1bab6ea513589b9e841a0: waiting        |--------------------------------------| 
config-sha256:fb59138d7af16a5974de721cb6a4399b9700f0ade6a1bc91209a174245cfb789:   waiting        |--------------------------------------| 
elapsed: 0.1 s                                                                    total:   0.0 B (0.0 B/s)                                         
WARN[0000] server "registry.upm:35000" does not seem to support HTTPS, falling back to plain HTTP  error="failed to do request: Head \"https://registry.upm:35000/v2/registry/blobs/sha256:6db7f901dc3b0c738464ace0ea79250b5341508cf23d091bcaf37332befa1209\": http: server gave HTTP response to HTTPS client"
manifest-sha256:43408412c6b1e1ec25a0d8ac2ff65e8e7c4e5d9d1ee1bab6ea513589b9e841a0: waiting        |--------------------------------------| 
config-sha256:fb59138d7af16a5974de721cb6a4399b9700f0ade6a1bc91209a174245cfb789:   waiting        |--------------------------------------| 
elapsed: 0.6 s                                                                    total:   0.0 B (0.0 B/s)           
```

##  7. 配置镜像入库

配置但不执行：
```bash
$ cd /root/kubernetes-offline/kubespray-offline-2.23.0-0/outputs
$ cat load-push-all-images.sh 
#!/bin/bash

source ./config.sh

LOCAL_REGISTRY=${LOCAL_REGISTRY:-"registry.upm:${REGISTRY_PORT}"}  #修改 registry.upm
NERDCTL=/usr/local/bin/nerdctl

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
        sudo $NERDCTL --insecure-registry=true push ${newImage}
    done
}

load_images
push_images
```



## 8. 执行 set-all.sh

```bash
[root@bastion01 outputs]# cat setup-all.sh 
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
sh -x set-all.sh
```
##  9. bastion01 配置互信

```bash
ssh-keygen
for i in {71..78};do ssh-copy-id root@10.70.0.$i;done
```
##  10. 启动容器部署环境

```bash
nerdctl load -i quay.io_kubespray_kubespray_v2.23.1.tar 
nerdctl  run --name kubespray-offline-ansible --network=host  -it -v "$(pwd)"/kubespray-offline-2.23.0-0:/kubespray -v "${HOME}"/.ssh:/root/.ssh  -v /etc/hosts:/etc/hosts quay.io/kubespray/kubespray:v2.23.1  bash
```

## 11. 部署前准备
检查批量通信、镜像仓库地址域名解析、可能存在缺失的内核加载模块。

###  11.1 配置 `extract-kubespray.sh`
保持 `kubespray-v2.22.1` 内容不变，注释掉 `apply patches`内容。

```bash
cd /root/kubernetes-offline/kubespray-offline-2.23.0-0/outputs
./extract-kubespray.sh
```
### 11.2 编写 inventory.ini
```bash
vim kubespray-2.23.0/inventory/sample/inventory.ini
```

```bash
[all]
kube-master01 ansible_host=10.70.0.71
kube-master02 ansible_host=10.70.0.72
kube-master03 ansible_host=10.70.0.73
kube-node01 ansible_host=10.70.0.74
kube-node02 ansible_host=10.70.0.75
kube-node03 ansible_host=10.70.0.76

[bastion]
bastion01 ansible_host=10.70.0.78 ansible_user=root

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
kube-node03

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr
```
执行：

```bash
ansible -i inventory/sample/inventory.ini all  -m ping
ansible -i inventory/sample/inventory.ini all -m lineinfile -a "path=/etc/hosts line='10.70.0.78 registry.upm'"
ansible -i inventory/sample/inventory.ini all -m lineinfile -a "path=/etc/rc.local line='modprobe br_netfilter\nmodprobe ip_conntrack'"
echo "nameserver 8.8.8.8" >  /etc/resolv.conf
ansible -i inventory/sample/inventory.ini all -m copy -a "src=/etc/resolv.conf dest=/etc/"
```

### 11.3 编写 offline.yml

```bash
vim  outputs/kubespray-2.23.1/inventry/sample/group_vars/all/offline.yml
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


### 11.4 配置 containerd.yml

```bash
$ vim  inventory/sample/group_vars/all/containerd.yml
......
containerd_registries_mirrors:
 - prefix: registry.upm:35000
   mirrors:
    - host: http://registry.upm:35000
      capabilities: ["pull", "resolve"]
      skip_verify: true
....
```

### 11.5 配置 nerdctl
解决镜像拉取失败

```bash
vim roles/container-engine/nerdctl/templates/nerdctl.toml.j2 
...
insecure_registry = true #添加
```

### 11.6 配置 nf_conntrack

123-148行 nf_conntrack_ipv4修改为 nf_conntrack
```bash
$ vim roles/kubernetes/node/tasks/main.yml
- name: Modprobe nf_conntrack
  community.general.modprobe:
    name: nf_conntrack
    state: present
  register: modprobe_nf_conntrack
  ignore_errors: true  # noqa ignore-errors
  when:
    - kube_proxy_mode == 'ipvs'
  tags:
    - kube-proxy

- name: Persist ip_vs modules
  copy:
    dest: /etc/modules-load.d/kube_proxy-ipvs.conf
    mode: 0644
    content: |
      ip_vs
      ip_vs_rr
      ip_vs_wrr
      ip_vs_sh
      {% if modprobe_nf_conntrack is success -%}
      nf_conntrack
      {%-   endif -%}
  when: kube_proxy_mode == 'ipvs'
  tags:
    - kube-proxy

```

## 12. 部署 offline repo
Deploy offline repo configurations which use your yum_repo/ubuntu_repo to all target nodes using ansible.
First, copy offline setup playbook to kubespray directory.

```bash
ansible -i inventory/sample/inventory.ini all -m shell -a  "mv /etc/yum.repos.d /tmp"
ansible -i inventory/sample/inventory.ini all -m shell -a  "mkdir /etc/yum.repos.d"
cp -r ../playbook .
ansible-playbook -i  inventory/sample/inventory.ini playbook/offline-repo.yml 
ansible -i inventory/sample/inventory.ini all  -m shell -a "yum repolist --verbose"
```
##  13. 开始部署

```bash
ansible-playbook -i inventory/sample/inventory.ini --become --become-user=root cluster.yml
```
输出：

```bash
PLAY RECAP ***************************************************************************************************
bastion01                  : ok=6    changed=0    unreachable=0    failed=0    skipped=14   rescued=0    ignored=0   
kube-master01              : ok=765  changed=153  unreachable=0    failed=0    skipped=1273 rescued=0    ignored=8   
kube-node01                : ok=525  changed=95   unreachable=0    failed=0    skipped=773  rescued=0    ignored=1   
kube-node02                : ok=525  changed=95   unreachable=0    failed=0    skipped=772  rescued=0    ignored=1   
kube-node03                : ok=525  changed=95   unreachable=0    failed=0    skipped=772  rescued=0    ignored=1   
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

Tuesday 21 November 2023  05:08:21 +0000 (0:00:01.427)       0:55:24.797 ****** 
=============================================================================== 
container-engine/containerd : Download_file | Download item ------------------------------------------ 55.86s
container-engine/crictl : Download_file | Download item ---------------------------------------------- 54.44s
container-engine/nerdctl : Download_file | Download item --------------------------------------------- 52.61s
bootstrap-os : Assign inventory name to unconfigured hostnames (non-CoreOS, non-Flatcar, Suse and ClearLinux, non-Fedora) -- 45.77s
container-engine/runc : Download_file | Download item ------------------------------------------------ 45.30s
container-engine/runc : Download_file | Validate mirrors --------------------------------------------- 43.93s
container-engine/nerdctl : Download_file | Validate mirrors ------------------------------------------ 41.38s
container-engine/crictl : Download_file | Validate mirrors ------------------------------------------- 39.49s
container-engine/containerd : Download_file | Validate mirrors --------------------------------------- 39.12s
etcdctl_etcdutl : Download_file | Download item ------------------------------------------------------ 38.89s
container-engine/crictl : Extract_file | Unpacking archive ------------------------------------------- 38.35s
download : Download_file | Download item ------------------------------------------------------------- 33.73s
kubernetes-apps/ansible : Kubernetes Apps | Lay Down CoreDNS templates ------------------------------- 32.09s
kubernetes/control-plane : Kubeadm | Initialize first master ----------------------------------------- 31.33s
etcdctl_etcdutl : Extract_file | Unpacking archive --------------------------------------------------- 28.66s
container-engine/nerdctl : Extract_file | Unpacking archive ------------------------------------------ 27.99s
kubernetes-apps/ansible : Kubernetes Apps | Start Resources ------------------------------------------ 26.80s
Gather necessary facts (hardware) -------------------------------------------------------------------- 25.67s
bootstrap-os : Gather host facts to get ansible_distribution_version ansible_distribution_major_version -- 24.51s
download : Download | Download files / images -------------------------------------------------------- 24.30s

#登陆master01节点
[root@kube-master01 ~]# kubectl get node 
NAME            STATUS   ROLES           AGE     VERSION
kube-master01   Ready    control-plane   7h53m   v1.27.7
kube-node01     Ready    <none>          7h51m   v1.27.7
kube-node02     Ready    <none>          7h51m   v1.27.7
kube-node03     Ready    <none>          7h51m   v1.27.7

```

## 14. 问题
### 14.1 Timeout (12s) waiting for privilege escalation prompt

- [Timeout (12s) waiting for privilege escalation prompt](https://github.com/ansible/ansible/issues/72111)

解决方法：添加 --timeout 60
```bash
ansible-playbook -i inventory/sample/inventory.ini --timeout 60 --become --become-user=root cluster.yml
```

