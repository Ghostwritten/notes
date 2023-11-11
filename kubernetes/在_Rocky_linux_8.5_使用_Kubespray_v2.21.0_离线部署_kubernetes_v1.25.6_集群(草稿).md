


## 前言

[Kubespray](https://kubespray.io/#/) 是 Kubernetes incubator 中的项目，目标是提供 Production Ready Kubernetes 部署方案，该项目基础是通过 Ansible Playbook 来定义系统与 Kubernetes 集群部署的任务，具有以下几个特点：

- 可以部署在 AWS, GCE, Azure, OpenStack 以及裸机上.
- 部署 High Available Kubernetes 集群.
- 可组合性 (Composable)，可自行选择 Network Plugin (flannel, calico, canal, weave) 来部署.
- 支持多种 Linux distributions
  -  RHEL 7 / CentOS 7
  - RHEL 8 / AlmaLinux 8
  - Ubuntu 20.04 / 22.04




本篇将说明如何通过 Kubespray 部署 Kubernetes 至裸机节点，安装版本如下所示：

- rocky linux 8.5
- Kubernetes v1.25.6
- kubespray v2.21.0
- docker-ce 23.0.2


##  创建7台虚拟机
通过 vSphere client 创建虚拟机，vSphere client 如何创建虚拟机请看[这里](https://blog.csdn.net/xixihahalelehehe/article/details/129310311)。

需求：

- 系统： Rocky Linux 8.7

- CPU: 4

- MEM: 8G

- DISK: 30G

[Rocky Linux 9.1 新手入门指南](https://blog.csdn.net/xixihahalelehehe/article/details/129644123)



配置在线机器地址与主机名分别为：


```bash
10.168.110.21 kube-control-plan01
10.168.110.31 dbscale-control-plan01

10.168.110.41 kube-node01
10.168.110.42 kube-node02
10.168.110.43 kube-node03

10.168.9.199 registry01
```
但我们要离线部署 k8s 集群机器配置网络全部改为离线环境。

```bash
100.168.110.199 registry01
100.168.110.21 kube-control-plan01
100.168.110.31 dbscale-control-plan01

100.168.110.41 kube-node01
100.168.110.42 kube-node02
100.168.110.43 kube-node03
```
10.168.9.199 registry01 为镜像与yum 源介质分发地址，暂时不改为离线，与 


`192.168.26.10` 为安装机器，并在线拉取需要的镜像。

## 要求



离线文件：
- yum 源
- 镜像
- pip包

目标节点的支持脚本

- 从本地安装 containerd .
- Start up nginx container as web server to supply Yum/Deb repository and PyPI mirror.
- 启动 docker private registry.
- Load all container images and push them to the private registry.


## 配置代理
加速下载介质，例如：github、quay、docker hub

```bash
$ vim /root/.bashrc
```

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

enable_proxy
#disable_proxy
```
执行：

```bash
source /root/.bashrc
```

##  下载介质
`192.168.26.10` online 操作

- 下载：[https://github.com/tmurakam/kubespray-offline/releases/tag/v2.21.0-0](https://github.com/tmurakam/kubespray-offline/releases/tag/v2.21.0-0)

```bash
unzip v2.21.0-0.zip
cd kubespray-offline-2.21.0-0
```

在下载介质之前需要安装：
- run `install-docker.sh` to install `Docker CE`.
- run `install-containerd.sh` to install `containerd` and `nerdctl.`
- Set docker environment variable to `/usr/local/bin/nerdctl` in config.sh.

```bash
./nstall-docker.sh
./install-containerd.sh
```



安装成功后，下载介质执行：

```bash
./download-all.sh
```
所有工件都存储在./outputs 目录。
此脚本调用以下所有脚本：
- prepare-pkgs.sh
Setup python, etc.
- prepare-py.sh
Setup python venv, install required python packages.
- get-kubespray.sh
Download and extract kubespray, if KUBESPRAY_DIR does not exist.
- pypi-mirror.sh
Download PyPI mirror files
- download-kubespray-files.sh
Download kubespray offline files (containers, files, etc)
- download-additional-containers.sh
Download additional containers.
You can add any container image repoTag to imagelists/*.txt.
- create-repo.sh
Download RPM or DEB repositories.
- copy-target-scripts.sh
Copy scripts for target node.


用时 30分钟 下载完成介质。
打包下载的介质传输到离线环境：

```bash
cd ../
tar czvf kubespray-offline-2.21.0.tar.gz kubespray-offline-2.21.0-0
```

> 注意：如果你喜欢通过容器部署集群，需要下载 kubespray 镜像，默认 `quay.io/kubespray/kubespray:v2.21.0`  ，存在一个 pip 包依赖（jmespath 1.0.1），下面镜像已重新编译。

                              
```bash
nerdctl pull ghostwritten/kubespray:v2.21-a94b893e2
nerdctl save -o kubespray:v2.21-a94b893e2.tar ghostwritten/kubespray:v2.21-a94b893e2
```

##  部署前准备






登陆部署节点（registry01），然后在输出目录中运行以下脚本：

```bash
tar zxvf kubespray-offline-2.21.0.tar.gz
cd kubespray-offline-2.21.0-0/outputs
./set-all.sh
```

此脚本调用以下所有脚本：
- setup-container.sh
Install containerd from local files.
Load nginx and registry images to containerd.
- start-nginx.sh
Start nginx container.
- setup-offline.sh
Setup yum/deb repo config and PyPI mirror config to use local nginx server.
- setup-py.sh
Install python3 and venv from local repo.
- start-registry.sh
Start docker private registry container.
- load-push-images.sh
Load all container images to containerd.
Tag and push them to the private registry.
- extract-kubespray.sh
Extract kubespray tarball and apply all patches.


You can configure port number of nginx and private registry in `config.sh`.

检查

```bash
$ nerdctl ps
CONTAINER ID    IMAGE                               COMMAND                   CREATED         STATUS    PORTS    NAMES
3fe93f5cfc97    docker.io/library/registry:2.8.1    "/entrypoint.sh /etc…"    7 days ago      Up                 registry
81eb4bac3c48    docker.io/library/nginx:1.23        "/docker-entrypoint.…"    23 hours ago    Up                 nginx
```

> 注意: 如果发现 nginx 报错 没有 `/usr/share/nginx/` 目录
> 
> 需要在容器内创建 `/usr/share/nginx/`，命令如下

```bash
pid=`nerdctl inspect nginx |grep -i pid | awk '{print $2}' | sed 's/,//g' `
nsenter -t $pid -n mkdir -p /usr/share/nginx/ 
nerdctl restart nginx 
```

##  安装部署工具
## 配置 python venv
Create and activate venv:

```bash
# Example
$ python3 -m venv ~/.venv/default
$ source ~/.venv/default/bin/activate
```

> Note: For `RHEL/CentOS 7`, you need to use `python 3.8`.

```bash
# Example
$ /opt/rh/rh-python38/root/usr/bin/python -m venv ~/.venv/default
$ source ~/.venv/default/bin/activate
```
Extract kubespray and apply patches:

```bash
$ ./extract-kubespray.sh
$ cd kubespray-2.21.0
```
For `Ubuntu 22.04`, you need to install build tools to build some python packages.

```bash
$ sudo apt install gcc python3-dev libffi-dev libssl-dev
```
Install ansible:

```bash
$ pip install -U pip                # update pip
$ pip install -r requirements.txt   # Install ansible
```

###  配置部署容器

```bash
 nerdctl  run --name=kubespray:v2.21-a94b893e2 --network=host --rm -itd --mount type=bind,source="$(pwd)"/kubespray-offline-2.21.0-0,dst=/kubespray\
     --mount type=bind,source="${HOME}"/.ssh,dst=/root/.ssh \
     --mount type=bind,source=/etc/hosts,dst=/etc/hosts \
     quay.io/kubespray/kubespray:v2.21-a94b893e2  bash
```

##  配置互信

```bash
ssh-copy-id root@100.168.110.21
ssh-copy-id root@100.168.110.31
ssh-copy-id root@100.168.110.41
ssh-copy-id root@100.168.110.42
ssh-copy-id root@100.168.110.43
```

##   编写 inventory.ini

```bash
[all]
kube-control-plan01 ansible_host=100.168.110.21
dbscale-control-plan01 ansible_host=100.168.110.31
kube-node01 ansible_host=100.168.110.41
kube-node02 ansible_host=100.168.110.42
kube-node03 ansible_host=100.168.110.43

[kube_control_plane]
kube-control-plan01

[etcd]
kube-control-plan01

[kube_node]
dbscale-control-plan01
kube-node01
kube-node02
kube-node03

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr
```

测试ping

```bash
ansible -i inventory/local/inventory.ini all -m ping
```

配置并分发 `/etc/hosts`\


```bash
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
100.168.110.199 registry01
100.168.110.21 kube-control-plan01
100.168.110.31 dbscale-control-plan01
100.168.110.41 kube-node01
100.168.110.42 kube-node02
100.168.110.43 kube-node03
```
执行
：

```bash
ansible -i inventory/local/inventory.ini all -m copy -a "src=/etc/hosts dest=/etc/hosts"
ansible -i inventory/local/inventory.ini all -m shell -a "cat /etc/hosts"
```

##  创建 offline.yml

```bash
vim  outputs/kubespray-2.21.0/inventry/local/group_vars/all/offline.yml
```

```bash
YOUR_HOST: '100.168.110.199'
http_server: "http://{{ YOUR_HOST }}"
registry_host: "{{ YOUR_HOST }}:35000"

containerd_insecure_registries: # Kubespray #8340
  "100.168.110.199:35000": "http://{{ YOUR_HOST }}:35000"

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

# Debian
## Docker
docker_debian_repo_base_url: "{{ debian_repo }}/docker-ce"
docker_debian_repo_gpgkey: "{{ debian_repo }}/docker-ce/gpg"
## Containerd
containerd_debian_repo_base_url: "{{ ubuntu_repo }}/containerd"
containerd_debian_repo_gpgkey: "{{ ubuntu_repo }}/containerd/gpg"
containerd_debian_repo_repokey: 'YOURREPOKEY'

# Ubuntu
## Docker
docker_ubuntu_repo_base_url: "{{ ubuntu_repo }}/docker-ce"
docker_ubuntu_repo_gpgkey: "{{ ubuntu_repo }}/docker-ce/gpg"
## Containerd
containerd_ubuntu_repo_base_url: "{{ ubuntu_repo }}/containerd"
containerd_ubuntu_repo_gpgkey: "{{ ubuntu_repo }}/containerd/gpg"
containerd_ubuntu_repo_repokey: 'YOURREPOKEY'

```




##  部署 offline repo 
Deploy offline repo configurations which use your yum_repo/ubuntu_repo to all target nodes using ansible.

First, copy offline setup playbook to kubespray directory.

```bash
$ cp -r kubespray-offline-2.21.0-0/playbook ${kubespray_dir}
```
Then execute `offline-repo.yml` playbook.

```bash
$ cd ${kubespray_dir}
ansible -i inventory/local/inventory.ini all -m shell -a  "mv /etc/yum.repos.d /tmp"
ansible -i inventory/local/inventory.ini all -m shell -a  "mkdir /etc/yum.repos.d"
ansible -i inventory/local/inventory.ini all -m copy -a "src=/tmp/yum.repos.d/offline.repo dest=/etc/yum.repos.d/"

$ ansible-playbook -i inventory/local/inventory.ini offline-repo.yml
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/1f54f4277ca24fa0bb06e51b81616211.png)

```bash
ansible -i inventory/local/inventory.ini all -m shell -a "yum install conntrack"
```

##  kubespray v2.21.1 部署 kubernetes 失败
该 `kubespray v2.21.1` 默认部署 `kubernetes v2.25.6` 。

**结果： 该kubespray v2.21.1 版本部署 `kubernetes v2.25.6` 版本,最终失败。原因：[https://github.com/kubernetes-sigs/kubespray/issues/9956](https://github.com/kubernetes-sigs/kubespray/issues/9956)**

```bash
# Example  
$ ansible-playbook -i inventory/local/inventory.ini --become --become-user=root cluster.yml
```


###  报错1：Install packages requirements
![在这里插入图片描述](https://img-blog.csdnimg.cn/c8d96d04a3f54e05a885da3893fe284c.png)



```bash
$ vi roles/kubernetes/preinstall/tasks/0070-system-packages.yml
...
    80  - name: Install packages requirements
    81    package:
    82      name: "{{ required_pkgs | default([]) | union(common_required_pkgs|default([])) }}"
    83      state: present
    84    register: pkgs_task_result
    85    until: pkgs_task_result is succeeded
    86    retries: "{{ pkg_install_retries }}"
    87    delay: "{{ retry_stagger | random + 3 }}"
    88    when: not (ansible_os_family in ["Flatcar", "Flatcar Container Linux by Kinvolk", "ClearLinux"] or is_fedora_coreos)
    89    tags:
    90      - bootstrap-os
```

解决方法，利用 iso 镜像搭建私有yum 源，分发各个集群节点。

### 报错2： Hosts | create list from inventory

```bash
$ vi /root/kubespray-offline-2.21.0/outputs/kubespray-2.21.0/roles/kubernetes/preinstall/tasks/0090-etchosts.yml

     1  ---
     2  - name: Hosts | create list from inventory
     3    set_fact:
     4      etc_hosts_inventory_block: |-
     5        {% for item in (groups['k8s_cluster'] + groups['etcd']|default([]) + groups['calico_rr']|default([]))|unique -%}
     6        {% if 'access_ip' in hostvars[item] or 'ip' in hostvars[item] or 'ansible_default_ipv4' in hostvars[item] and 'address' in hostvars[item]['ansible_default_ipv4'] -%}
     7        {{ hostvars[item]['access_ip'] | default(hostvars[item]['ip'] | default(hostvars[item]['ansible_default_ipv4']['address'])) }}

```


```bash
TASK [kubernetes/preinstall : Hosts | create list from inventory] ******************************************************************************************************************
fatal: [kube-control-plan01 -> localhost]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'dict object' has no attribute 'address'\n\nThe error appears to be in '/root/kubespray-offline-2.21.0/outputs/kubespray-2.21.0/roles/kubernetes/preinstall/tasks/0090-etchosts.yml': line 2, column 3, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n---\n- name: Hosts | create list from inventory\n  ^ here\n"}

```

修复方法
注释 `/root/kubespray-offline-2.21.0/outputs/kubespray-2.21.0/roles/kubernetes/preinstall/tasks/0090-etchosts.yml` 2-25行

编写/etc/hosts

```bash
127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4
::1 localhost6 localhost6.localdomain6 localhost6.localdomain
# Ansible inventory hosts BEGIN
100.168.110.21 kube-control-plan01.cluster.local kube-control-plan01
100.168.110.31 dbscale-control-plan01.cluster.local dbscale-control-plan01
100.168.110.41 kube-node01.cluster.local kube-node01
100.168.110.42 kube-node02.cluster.local kube-node02
100.168.110.43 kube-node03.cluster.local kube-node03
# Ansible inventory hosts END
```

执行：

```bash
ansible -i inventory/local/inventory.ini all -m copy -a "src=/etc/hosts dest=/etc/hosts"
```

### 报错3：container-engine/containerd : containerd Create registry directories

修改 `with_items` 改为 `with_dict`
```bash
$ vi /root/kubespray-offline-2.21.0/outputs/kubespray-2.21.0/roles/container-engine/containerd/tasks/main.yml


   114  - name: containerd ｜ Create registry directories
   115    file:
   116      path: "{{ containerd_cfg_dir }}/certs.d/{{ item.key }}"
   117      state: directory
   118      mode: 0755
   119      recurse: true
   120    with_dict: "{{ containerd_insecure_registries }}"
   121    when: containerd_insecure_registries is defined
   122
   123  - name: containerd ｜ Write hosts.toml file
   124    blockinfile:
   125      path: "{{ containerd_cfg_dir }}/certs.d/{{ item.key }}/hosts.toml"
   126      owner: "root"
   127      mode: 0640
   128      create: true
   129      block: |
   130        server = "{{ item.value }}"
   131        [host."{{ item.value }}"]
   132          capabilities = ["pull", "resolve", "push"]
   133          skip_verify = true
   134    with_dict: "{{ containerd_insecure_registries }}"
   135    when: containerd_insecure_registries is defined
```

```bash
TASK [container-engine/containerd : containerd Create registry directories] ********************************************************************************************************
fatal: [kube-control-plan01]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'ansible.utils.unsafe_proxy.AnsibleUnsafeText object' has no attribute 'key'\n\nThe error appears to be in '/root/kubespray-offline-2.21.0/outputs/kubespray-2.21.0/roles/container-engine/containerd/tasks/main.yml': line 114, column 3, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n\n- name: containerd Create registry directories\n  ^ here\n"}
fatal: [dbscale-control-plan01]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'ansible.utils.unsafe_proxy.AnsibleUnsafeText object' has no attribute 'key'\n\nThe error appears to be in '/root/kubespray-offline-2.21.0/outputs/kubespray-2.21.0/roles/container-engine/containerd/tasks/main.yml': line 114, column 3, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n\n- name: containerd Create registry directories\n  ^ here\n"}
fatal: [kube-node01]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'ansible.utils.unsafe_proxy.AnsibleUnsafeText object' has no attribute 'key'\n\nThe error appears to be in '/root/kubespray-offline-2.21.0/outputs/kubespray-2.21.0/roles/container-engine/containerd/tasks/main.yml': line 114, column 3, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n\n- name: containerd Create registry directories\n  ^ here\n"}
fatal: [kube-node02]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'ansible.utils.unsafe_proxy.AnsibleUnsafeText object' has no attribute 'key'\n\nThe error appears to be in '/root/kubespray-offline-2.21.0/outputs/kubespray-2.21.0/roles/container-engine/containerd/tasks/main.yml': line 114, column 3, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n\n- name: containerd Create registry directories\n  ^ here\n"}
fatal: [kube-node03]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'ansible.utils.unsafe_proxy.AnsibleUnsafeText object' has no attribute 'key'\n\nThe error appears to be in '/root/kubespray-offline-2.21.0/outputs/kubespray-2.21.0/roles/container-engine/containerd/tasks/main.yml': line 114, column 3, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n\n- name: containerd Create registry directories\n  ^ here\n"}

NO MORE HOSTS LEFT *****************************************************************************************************************************************************************

PLAY RECAP *************************************************************************************************************************************************************************
dbscale-control-plan01     : ok=147  changed=1    unreachable=0    failed=1    skipped=264  rescued=0    ignored=0
kube-control-plan01        : ok=173  changed=1    unreachable=0    failed=1    skipped=293  rescued=0    ignored=0
kube-node01                : ok=147  changed=1    unreachable=0    failed=1    skipped=264  rescued=0    ignored=0
kube-node02                : ok=147  changed=1    unreachable=0    failed=1    skipped=264  rescued=0    ignored=0
kube-node03                : ok=147  changed=1    unreachable=0    failed=1    skipped=264  rescued=0    ignored=0
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

修复方法：

```bash
ansible -i inventory/local/inventory.ini all -m file -a "path=/etc/containerd/certs.d/100.168.110.199:35000 state=directory recurse=true mode=0755"
```



### 报错：container-engine/containerd : containerd ｜ Write hosts.toml file

```bash
TASK [container-engine/containerd : containerd ｜ Write hosts.toml file] ***********************************************************************************************************
fatal: [kube-control-plan01]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'ansible.utils.unsafe_proxy.AnsibleUnsafeText object' has no attribute 'key'\n\nThe error appears to be in '/root/kubespray-offline-2.21.0/outputs/kubespray-2.21.0/roles/container-engine/containerd/tasks/main.yml': line 123, column 3, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n\n- name: containerd ｜ Write hosts.toml file\n  ^ here\n"}
fatal: [dbscale-control-plan01]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'ansible.utils.unsafe_proxy.AnsibleUnsafeText object' has no attribute 'key'\n\nThe error appears to be in '/root/kubespray-offline-2.21.0/outputs/kubespray-2.21.0/roles/container-engine/containerd/tasks/main.yml': line 123, column 3, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n\n- name: containerd ｜ Write hosts.toml file\n  ^ here\n"}
fatal: [kube-node01]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'ansible.utils.unsafe_proxy.AnsibleUnsafeText object' has no attribute 'key'\n\nThe error appears to be in '/root/kubespray-offline-2.21.0/outputs/kubespray-2.21.0/roles/container-engine/containerd/tasks/main.yml': line 123, column 3, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n\n- name: containerd ｜ Write hosts.toml file\n  ^ here\n"}
fatal: [kube-node02]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'ansible.utils.unsafe_proxy.AnsibleUnsafeText object' has no attribute 'key'\n\nThe error appears to be in '/root/kubespray-offline-2.21.0/outputs/kubespray-2.21.0/roles/container-engine/containerd/tasks/main.yml': line 123, column 3, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n\n- name: containerd ｜ Write hosts.toml file\n  ^ here\n"}
fatal: [kube-node03]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'ansible.utils.unsafe_proxy.AnsibleUnsafeText object' has no attribute 'key'\n\nThe error appears to be in '/root/kubespray-offline-2.21.0/outputs/kubespray-2.21.0/roles/container-engine/containerd/tasks/main.yml': line 123, column 3, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n\n- name: containerd ｜ Write hosts.toml file\n  ^ here\n"}

NO MORE HOSTS LEFT *****************************************************************************************************************************************************************

PLAY RECAP *********************************************************************************************************************************
```
解决方法：

```bash
$ vi roles/container-engine/containerd/tasks/main.yml
- name: containerd ｜ Create registry directories
  file:
    path: "{{ containerd_cfg_dir }}/certs.d/{{ item.key }}"
    state: directory
    mode: 0755
    recurse: true
  with_dict: "{{ containerd_insecure_registries }}"
  when: containerd_insecure_registries is defined
```



```bash
$ cat host.toml
server = "100.168.110.199:35000"
[host."100.168.110.199:35000"]
  capabilities = ["pull", "resolve", "push"]
  skip_verify = true
```
执行：
```bash
ansible -i inventory/local/inventory.ini all -m copy -a "src=./host.toml dest=/etc/containerd/certs.d/100.168.110.199:35000/"
```



### 报错：kubernetes/node : Modprobe nf_conntrack_ipv4

```bash
TASK [kubernetes/node : Modprobe nf_conntrack_ipv4] ********************************************************************************************************************************
fatal: [kube-control-plan01]: FAILED! => {"changed": false, "msg": "modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-425.13.1.el8_7.x86_64\n", "name": "nf_conntrack_ipv4", "params": "", "rc": 1, "state": "present", "stderr": "modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-425.13.1.el8_7.x86_64\n", "stderr_lines": ["modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-425.13.1.el8_7.x86_64"], "stdout": "", "stdout_lines": []}
...ignoring
fatal: [dbscale-control-plan01]: FAILED! => {"changed": false, "msg": "modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-425.13.1.el8_7.x86_64\n", "name": "nf_conntrack_ipv4", "params": "", "rc": 1, "state": "present", "stderr": "modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-425.13.1.el8_7.x86_64\n", "stderr_lines": ["modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-425.13.1.el8_7.x86_64"], "stdout": "", "stdout_lines": []}
...ignoring
fatal: [kube-node01]: FAILED! => {"changed": false, "msg": "modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-425.13.1.el8_7.x86_64\n", "name": "nf_conntrack_ipv4", "params": "", "rc": 1, "state": "present", "stderr": "modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-425.13.1.el8_7.x86_64\n", "stderr_lines": ["modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-425.13.1.el8_7.x86_64"], "stdout": "", "stdout_lines": []}
...ignoring
fatal: [kube-node02]: FAILED! => {"changed": false, "msg": "modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-425.13.1.el8_7.x86_64\n", "name": "nf_conntrack_ipv4", "params": "", "rc": 1, "state": "present", "stderr": "modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-425.13.1.el8_7.x86_64\n", "stderr_lines": ["modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-425.13.1.el8_7.x86_64"], "stdout": "", "stdout_lines": []}
...ignoring
fatal: [kube-node03]: FAILED! => {"changed": false, "msg": "modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-425.13.1.el8_7.x86_64\n", "name": "nf_conntrack_ipv4", "params": "", "rc": 1, "state": "present", "stderr": "modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-425.13.1.el8_7.x86_64\n", "stderr_lines": ["modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-425.13.1.el8_7.x86_64"], "stdout": "", "stdout_lines": []}
...ignoring
Monday 10 April 2023  09:33:26 -0400 (0:00:00.480)       0:04:30.825 **********

TASK [kubernetes/node : Persist ip_vs modules] ***************************
```

手动批量执行：

```bash
modprobe br_netfilter
```
说明：`nf_conntrack_ipv4` havs been rename to `nf_conntrack` since Linux kernel `4.18+`


### 报错：Check which kube-control nodes are already members of the cluster

```bash
TASK [kubernetes/control-plane : Check which kube-control nodes are already members of the cluster] ********************************************************************************fatal: [kube-control-plan01]: FAILED! => {"changed": false, "cmd": ["/usr/local/bin/kubectl", "get", "nodes", "--selector=node-role.kubernetes.io/control-plane", "-o", "json"], "delta": "0:00:00.028582", "end": "2023-04-10 21:33:37.555599", "msg": "non-zero return code", "rc": 1, "start": "2023-04-10 21:33:37.527017", "stderr": "The connection to the server localhost:8080 was refused - did you specify the right host or port?", "stderr_lines": ["The connection to the server localhost:8080 was refused - did you specify the right host or port?"], "stdout": "", "stdout_lines": []}
...ignoring
```


报错：

```bash
TASK [kubernetes/control-plane : kubeadm | Initialize first master] ****************************************************************************************************************
fatal: [kube-control-plan01]: FAILED! => {"attempts": 3, "changed": true, "cmd": ["timeout", "-k", "300s", "300s", "/usr/local/bin/kubeadm", "init", "--config=/etc/kubernetes/kubeadm-config.yaml", "--ignore-preflight-errors=all", "--skip-phases=addon/coredns", "--upload-certs"], "delta": "0:01:56.126009", "end": "2023-04-10 21:41:40.621046", "failed_when_result": true, "msg": "non-zero return code", "rc": 1, "start": "2023-04-10 21:39:44.495037", "stderr": "W0410 21:39:44.514437  222884 utils.go:69] The recommended value for \"clusterDNS\" in \"KubeletConfiguration\" is: [10.233.0.10]; the provided value is: [169.254.25.10]\n\t[WARNING FileAvailable--etc-kubernetes-manifests-kube-apiserver.yaml]: /etc/kubernetes/manifests/kube-apiserver.yaml already exists\n\t[WARNING FileAvailable--etc-kubernetes-manifests-kube-controller-manager.yaml]: /etc/kubernetes/manifests/kube-controller-manager.yaml already exists\n\t[WARNING FileAvailable--etc-kubernetes-manifests-kube-scheduler.yaml]: /etc/kubernetes/manifests/kube-scheduler.yaml already exists\n\t[WARNING CRI]: container runtime is not running: output: E0410 21:39:44.542333  222894 remote_runtime.go:948] \"Status from runtime service failed\" err=\"rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.RuntimeService\"\ntime=\"2023-04-10T21:39:44+08:00\" level=fatal msg=\"getting status of runtime: rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.RuntimeService\"\n, error: exit status 1\n\t[WARNING FileExisting-conntrack]: conntrack not found in system path\n\t[WARNING FileExisting-socat]: socat not found in system path\n\t[WARNING ImagePull]: failed to pull image 100.168.110.199:35000/kube-apiserver:v1.25.6: output: E0410 21:39:44.665910  222942 remote_image.go:222] \"PullImage from image service failed\" err=\"rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.ImageService\" image=\"100.168.110.199:35000/kube-apiserver:v1.25.6\"\ntime=\"2023-04-10T21:39:44+08:00\" level=fatal msg=\"pulling image: rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.ImageService\"\n, error: exit status 1\n\t[WARNING ImagePull]: failed to pull image 100.168.110.199:35000/kube-controller-manager:v1.25.6: output: E0410 21:39:44.746281  222977 remote_image.go:222] \"PullImage from image service failed\" err=\"rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.ImageService\" image=\"100.168.110.199:35000/kube-controller-manager:v1.25.6\"\ntime=\"2023-04-10T21:39:44+08:00\" level=fatal msg=\"pulling image: rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.ImageService\"\n, error: exit status 1\n\t[WARNING ImagePull]: failed to pull image 100.168.110.199:35000/kube-scheduler:v1.25.6: output: E0410 21:39:44.824291  223014 remote_image.go:222] \"PullImage from image service failed\" err=\"rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.ImageService\" image=\"100.168.110.199:35000/kube-scheduler:v1.25.6\"\ntime=\"2023-04-10T21:39:44+08:00\" level=fatal msg=\"pulling image: rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.ImageService\"\n, error: exit status 1\n\t[WARNING ImagePull]: failed to pull image 100.168.110.199:35000/kube-proxy:v1.25.6: output: E0410 21:39:44.902557  223050 remote_image.go:222] \"PullImage from image service failed\" err=\"rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.ImageService\" image=\"100.168.110.199:35000/kube-proxy:v1.25.6\"\ntime=\"2023-04-10T21:39:44+08:00\" level=fatal msg=\"pulling image: rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.ImageService\"\n, error: exit status 1\n\t[WARNING ImagePull]: failed to pull image 100.168.110.199:35000/pause:3.8: output: E0410 21:39:44.985206  223086 remote_image.go:222] \"PullImage from image service failed\" err=\"rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.ImageService\" image=\"100.168.110.199:35000/pause:3.8\"\ntime=\"2023-04-10T21:39:44+08:00\" level=fatal msg=\"pulling image: rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.ImageService\"\n, error: exit status 1\n\t[WARNING ImagePull]: failed to pull image 100.168.110.199:35000/coredns/coredns:v1.9.3: output: E0410 21:39:45.063766  223122 remote_image.go:222] \"PullImage from image service failed\" err=\"rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.ImageService\" image=\"100.168.110.199:35000/coredns/coredns:v1.9.3\"\ntime=\"2023-04-10T21:39:45+08:00\" level=fatal msg=\"pulling image: rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.ImageService\"\n, error: exit status 1\nerror execution phase wait-control-plane: couldn't initialize a Kubernetes cluster\nTo see the stack trace of this error execute with --v=5 or higher", "stderr_lines": ["W0410 21:39:44.514437  222884 utils.go:69] The recommended value for \"clusterDNS\" in \"KubeletConfiguration\" is: [10.233.0.10]; the provided value is: [169.254.25.10]", "\t[WARNING FileAvailable--etc-kubernetes-manifests-kube-apiserver.yaml]: /etc/kubernetes/manifests/kube-apiserver.yaml already exists", "\t[WARNING FileAvailable--etc-kubernetes-manifests-kube-controller-manager.yaml]: /etc/kubernetes/manifests/kube-controller-manager.yaml already exists", "\t[WARNING FileAvailable--etc-kubernetes-manifests-kube-scheduler.yaml]: /etc/kubernetes/manifests/kube-scheduler.yaml already exists", "\t[WARNING CRI]: container runtime is not running: output: E0410 21:39:44.542333  222894 remote_runtime.go:948] \"Status from runtime service failed\" err=\"rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.RuntimeService\"", "time=\"2023-04-10T21:39:44+08:00\" level=fatal msg=\"getting status of runtime: rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.RuntimeService\"", ", error: exit status 1", "\t[WARNING FileExisting-conntrack]: conntrack not found in system path", "\t[WARNING FileExisting-socat]: socat not found in system path", "\t[WARNING ImagePull]: failed to pull image 100.168.110.199:35000/kube-apiserver:v1.25.6: output: E0410 21:39:44.665910  222942 remote_image.go:222] \"PullImage from image service failed\" err=\"rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.ImageService\" image=\"100.168.110.199:35000/kube-apiserver:v1.25.6\"", "time=\"2023-04-10T21:39:44+08:00\" level=fatal msg=\"pulling image: rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.ImageService\"", ", error: exit status 1", "\t[WARNING ImagePull]: failed to pull image 100.168.110.199:35000/kube-controller-manager:v1.25.6: output: E0410 21:39:44.746281  222977 remote_image.go:222] \"PullImage from image service failed\" err=\"rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.ImageService\" image=\"100.168.110.199:35000/kube-controller-manager:v1.25.6\"", "time=\"2023-04-10T21:39:44+08:00\" level=fatal msg=\"pulling image: rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.ImageService\"", ", error: exit status 1", "\t[WARNING ImagePull]: failed to pull image 100.168.110.199:35000/kube-scheduler:v1.25.6: output: E0410 21:39:44.824291  223014 remote_image.go:222] \"PullImage from image service failed\" err=\"rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.ImageService\" image=\"100.168.110.199:35000/kube-scheduler:v1.25.6\"", "time=\"2023-04-10T21:39:44+08:00\" level=fatal msg=\"pulling image: rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.ImageService\"", ", error: exit status 1", "\t[WARNING ImagePull]: failed to pull image 100.168.110.199:35000/kube-proxy:v1.25.6: output: E0410 21:39:44.902557  223050 remote_image.go:222] \"PullImage from image service failed\" err=\"rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.ImageService\" image=\"100.168.110.199:35000/kube-proxy:v1.25.6\"", "time=\"2023-04-10T21:39:44+08:00\" level=fatal msg=\"pulling image: rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.ImageService\"", ", error: exit status 1", "\t[WARNING ImagePull]: failed to pull image 100.168.110.199:35000/pause:3.8: output: E0410 21:39:44.985206  223086 remote_image.go:222] \"PullImage from image service failed\" err=\"rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.ImageService\" image=\"100.168.110.199:35000/pause:3.8\"", "time=\"2023-04-10T21:39:44+08:00\" level=fatal msg=\"pulling image: rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.ImageService\"", ", error: exit status 1", "\t[WARNING ImagePull]: failed to pull image 100.168.110.199:35000/coredns/coredns:v1.9.3: output: E0410 21:39:45.063766  223122 remote_image.go:222] \"PullImage from image service failed\" err=\"rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.ImageService\" image=\"100.168.110.199:35000/coredns/coredns:v1.9.3\"", "time=\"2023-04-10T21:39:45+08:00\" level=fatal msg=\"pulling image: rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.ImageService\"", ", error: exit status 1", "error execution phase wait-control-plane: couldn't initialize a Kubernetes cluster", "To see the stack trace of this error execute with --v=5 or higher"], "stdout": "[init] Using Kubernetes version: v1.25.6\n[preflight] Running pre-flight checks\n[preflight] Pulling images required for setting up a Kubernetes cluster\n[preflight] This might take a minute or two, depending on the speed of your internet connection\n[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'\n[certs] Using certificateDir folder \"/etc/kubernetes/ssl\"\n[certs] Using existing ca certificate authority\n[certs] Using existing apiserver certificate and key on disk\n[certs] Using existing apiserver-kubelet-client certificate and key on disk\n[certs] Using existing front-proxy-ca certificate authority\n[certs] Using existing front-proxy-client certificate and key on disk\n[certs] External etcd mode: Skipping etcd/ca certificate authority generation\n[certs] External etcd mode: Skipping etcd/server certificate generation\n[certs] External etcd mode: Skipping etcd/peer certificate generation\n[certs] External etcd mode: Skipping etcd/healthcheck-client certificate generation\n[certs] External etcd mode: Skipping apiserver-etcd-client certificate generation\n[certs] Using the existing \"sa\" key\n[kubeconfig] Using kubeconfig folder \"/etc/kubernetes\"\n[kubeconfig] Using existing kubeconfig file: \"/etc/kubernetes/admin.conf\"\n[kubeconfig] Using existing kubeconfig file: \"/etc/kubernetes/kubelet.conf\"\n[kubeconfig] Using existing kubeconfig file: \"/etc/kubernetes/controller-manager.conf\"\n[kubeconfig] Using existing kubeconfig file: \"/etc/kubernetes/scheduler.conf\"\n[kubelet-start] Writing kubelet environment file with flags to file \"/var/lib/kubelet/kubeadm-flags.env\"\n[kubelet-start] Writing kubelet configuration to file \"/var/lib/kubelet/config.yaml\"\n[kubelet-start] Starting the kubelet\n[control-plane] Using manifest folder \"/etc/kubernetes/manifests\"\n[control-plane] Creating static Pod manifest for \"kube-apiserver\"\n[control-plane] Creating static Pod manifest for \"kube-controller-manager\"\n[control-plane] Creating static Pod manifest for \"kube-scheduler\"\n[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory \"/etc/kubernetes/manifests\". This can take up to 5m0s\n[kubelet-check] Initial timeout of 40s passed.\n[kubelet-check] It seems like the kubelet isn't running or healthy.\n[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get \"http://localhost:10248/healthz\": dial tcp 127.0.0.1:10248: connect: connection refused.\n[kubelet-check] It seems like the kubelet isn't running or healthy.\n[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get \"http://localhost:10248/healthz\": dial tcp 127.0.0.1:10248: connect: connection refused.\n[kubelet-check] It seems like the kubelet isn't running or healthy.\n[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get \"http://localhost:10248/healthz\": dial tcp 127.0.0.1:10248: connect: connection refused.\n[kubelet-check] It seems like the kubelet isn't running or healthy.\n[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get \"http://localhost:10248/healthz\": dial tcp 127.0.0.1:10248: connect: connection refused.\n[kubelet-check] It seems like the kubelet isn't running or healthy.\n[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get \"http://localhost:10248/healthz\": dial tcp 127.0.0.1:10248: connect: connection refused.\n\nUnfortunately, an error has occurred:\n\ttimed out waiting for the condition\n\nThis error is likely caused by:\n\t- The kubelet is not running\n\t- The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)\n\nIf you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:\n\t- 'systemctl status kubelet'\n\t- 'journalctl -xeu kubelet'\n\nAdditionally, a control plane component may have crashed or exited when started by the container runtime.\nTo troubleshoot, list all containers using your preferred container runtimes CLI.\nHere is one example how you may list all running Kubernetes containers by using crictl:\n\t- 'crictl --runtime-endpoint unix:///var/run/containerd/containerd.sock ps -a | grep kube | grep -v pause'\n\tOnce you have found the failing container, you can inspect its logs with:\n\t- 'crictl --runtime-endpoint unix:///var/run/containerd/containerd.sock logs CONTAINERID'", "stdout_lines": ["[init] Using Kubernetes version: v1.25.6", "[preflight] Running pre-flight checks", "[preflight] Pulling images required for setting up a Kubernetes cluster", "[preflight] This might take a minute or two, depending on the speed of your internet connection", "[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'", "[certs] Using certificateDir folder \"/etc/kubernetes/ssl\"", "[certs] Using existing ca certificate authority", "[certs] Using existing apiserver certificate and key on disk", "[certs] Using existing apiserver-kubelet-client certificate and key on disk", "[certs] Using existing front-proxy-ca certificate authority", "[certs] Using existing front-proxy-client certificate and key on disk", "[certs] External etcd mode: Skipping etcd/ca certificate authority generation", "[certs] External etcd mode: Skipping etcd/server certificate generation", "[certs] External etcd mode: Skipping etcd/peer certificate generation", "[certs] External etcd mode: Skipping etcd/healthcheck-client certificate generation", "[certs] External etcd mode: Skipping apiserver-etcd-client certificate generation", "[certs] Using the existing \"sa\" key", "[kubeconfig] Using kubeconfig folder \"/etc/kubernetes\"", "[kubeconfig] Using existing kubeconfig file: \"/etc/kubernetes/admin.conf\"", "[kubeconfig] Using existing kubeconfig file: \"/etc/kubernetes/kubelet.conf\"", "[kubeconfig] Using existing kubeconfig file: \"/etc/kubernetes/controller-manager.conf\"", "[kubeconfig] Using existing kubeconfig file: \"/etc/kubernetes/scheduler.conf\"", "[kubelet-start] Writing kubelet environment file with flags to file \"/var/lib/kubelet/kubeadm-flags.env\"", "[kubelet-start] Writing kubelet configuration to file \"/var/lib/kubelet/config.yaml\"", "[kubelet-start] Starting the kubelet", "[control-plane] Using manifest folder \"/etc/kubernetes/manifests\"", "[control-plane] Creating static Pod manifest for \"kube-apiserver\"", "[control-plane] Creating static Pod manifest for \"kube-controller-manager\"", "[control-plane] Creating static Pod manifest for \"kube-scheduler\"", "[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory \"/etc/kubernetes/manifests\". This can take up to 5m0s", "[kubelet-check] Initial timeout of 40s passed.", "[kubelet-check] It seems like the kubelet isn't running or healthy.", "[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get \"http://localhost:10248/healthz\": dial tcp 127.0.0.1:10248: connect: connection refused.", "[kubelet-check] It seems like the kubelet isn't running or healthy.", "[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get \"http://localhost:10248/healthz\": dial tcp 127.0.0.1:10248: connect: connection refused.", "[kubelet-check] It seems like the kubelet isn't running or healthy.", "[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get \"http://localhost:10248/healthz\": dial tcp 127.0.0.1:10248: connect: connection refused.", "[kubelet-check] It seems like the kubelet isn't running or healthy.", "[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get \"http://localhost:10248/healthz\": dial tcp 127.0.0.1:10248: connect: connection refused.", "[kubelet-check] It seems like the kubelet isn't running or healthy.", "[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get \"http://localhost:10248/healthz\": dial tcp 127.0.0.1:10248: connect: connection refused.", "", "Unfortunately, an error has occurred:", "\ttimed out waiting for the condition", "", "This error is likely caused by:", "\t- The kubelet is not running", "\t- The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)", "", "If you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:", "\t- 'systemctl status kubelet'", "\t- 'journalctl -xeu kubelet'", "", "Additionally, a control plane component may have crashed or exited when started by the container runtime.", "To troubleshoot, list all containers using your preferred container runtimes CLI.", "Here is one example how you may list all running Kubernetes containers by using crictl:", "\t- 'crictl --runtime-endpoint unix:///var/run/containerd/containerd.sock ps -a | grep kube | grep -v pause'", "\tOnce you have found the failing container, you can inspect its logs with:", "\t- 'crictl --runtime-endpoint unix:///var/run/containerd/containerd.sock logs CONTAINERID'"]}

NO MORE HOSTS LEFT *****************************************************************************************************************************************************************

PLAY RECAP *************************************************************************************************************************************************************************
dbscale-control-plan01     : ok=436  changed=17   unreachable=0    failed=0    skipped=527  rescued=0    ignored=1
kube-control-plan01        : ok=570  changed=67   unreachable=0    failed=1    skipped=726  rescued=0    ignored=3
kube-node01                : ok=436  changed=17   unreachable=0    failed=0    skipped=526  rescued=0    ignored=1
kube-node02                : ok=436  changed=17   unreachable=0    failed=0    skipped=526  rescued=0    ignored=1
kube-node03                : ok=436  changed=17   unreachable=0    failed=0    skipped=526  rescued=0    ignored=1
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```


原因：
- [https://github.com/containerd/containerd/issues/4581](https://github.com/containerd/containerd/issues/4581)

```bash
kubeadm init --config=/etc/kubernetes/kubeadm-config.yaml --ignore-preflight-errors=all --skip-phases=addon/coredns --upload-certs
```
kubeadm init 集群无法成功。原因是 kubelet 服务没有启动成功，没有启动原因是，container的/etc/containerd/config.toml配置私有仓库，导致kubelet 接口出现 bug。删除相关配置kubelet启动成功。

修改 关于 config.toml 的ansible-playbook 模板 

```bash
cat roles/container-engine/containerd/templates/config.toml.j2
version = 2
root = "{{ containerd_storage_dir }}"
state = "{{ containerd_state_dir }}"
oom_score = {{ containerd_oom_score }}

[grpc]
  max_recv_message_size = {{ containerd_grpc_max_recv_message_size | default(16777216) }}
  max_send_message_size = {{ containerd_grpc_max_send_message_size | default(16777216) }}

[debug]
  level = "{{ containerd_debug_level | default('info') }}"

[metrics]
  address = "{{ containerd_metrics_address | default('') }}"
  grpc_histogram = {{ containerd_metrics_grpc_histogram | default(false) | lower }}

[plugins]
  [plugins."io.containerd.grpc.v1.cri"]
    sandbox_image = "{{ pod_infra_image_repo }}:{{ pod_infra_image_tag }}"
    max_container_log_line_size = {{ containerd_max_container_log_line_size }}
    enable_unprivileged_ports = {{ containerd_enable_unprivileged_ports | default(false) | lower }}
    enable_unprivileged_icmp = {{ containerd_enable_unprivileged_icmp | default(false) | lower }}
    [plugins."io.containerd.grpc.v1.cri".containerd]
      default_runtime_name = "{{ containerd_default_runtime | default('runc') }}"
      snapshotter = "{{ containerd_snapshotter | default('overlayfs') }}"
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
{% for runtime in [containerd_runc_runtime] + containerd_additional_runtimes %}
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.{{ runtime.name }}]
          runtime_type = "{{ runtime.type }}"
          runtime_engine = "{{ runtime.engine }}"
          runtime_root = "{{ runtime.root }}"
{% if runtime.base_runtime_spec is defined %}
          base_runtime_spec = "{{ containerd_cfg_dir }}/{{ runtime.base_runtime_spec }}"
{% endif %}

          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.{{ runtime.name }}.options]
{% for key, value in runtime.options.items() %}
            {{ key }} = {{ value }}
{% endfor %}
{% endfor %}
{% if kata_containers_enabled %}
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.kata-qemu]
          runtime_type = "io.containerd.kata-qemu.v2"
{% endif %}
{% if gvisor_enabled %}
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runsc]
          runtime_type = "io.containerd.runsc.v1"
{% endif %}
    [plugins."io.containerd.grpc.v1.cri".registry]
{% if containerd_insecure_registries is defined and containerd_insecure_registries|length>0 %}
      config_path = "{{ containerd_cfg_dir }}/certs.d"
{% endif %}

{% if containerd_extra_args is defined %}
{{ containerd_extra_args }}
{% endif %}
```

再次 执行：


```bash
ansible-playbook -i inventory/local/inventory.ini --become --become-user=root reset.yml
ansible-playbook -i inventory/local/inventory.ini --become --become-user=root cluster.yml 
```



##  分步执行

```bash
TASK [kubernetes/preinstall : Hosts | create list from inventory] ******************************************************
ok: [kube-control-plan01 -> localhost]
Tuesday 11 April 2023  21:59:33 -0400 (0:00:00.376)       0:04:48.518 *********

TASK [kubernetes/preinstall : Hosts | populate inventory into hosts file] **********************************************
changed: [kube-node02]
changed: [kube-node03]
changed: [kube-node01]
changed: [dbscale-control-plan01]
changed: [kube-control-plan01]
```



```bash
ansible-playbook -i inventory/local/inventory.ini --become --become-user=root cluster.yml --tags  etchosts
```






参考：
- github：[https://github.com/kubernetes-sigs/kubespray](https://github.com/kubernetes-sigs/kubespray)
- 官网：[https://kubespray.io/#/](https://kubespray.io/#/)
- 网友kubespray 学习：[https://github.com/wenwenxiong/book/tree/master/k8s/kubespray](https://github.com/wenwenxiong/book/tree/master/k8s/kubespray)
