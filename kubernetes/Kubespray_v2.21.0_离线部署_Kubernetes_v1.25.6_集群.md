
![](https://i-blog.csdnimg.cn/blog_migrate/484eb3a5a44953719310dfac6eca9d67.png)



## 1. 前言

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
- containerd v1.6.15


## 2. 预备条件

 创建7台虚拟机
通过 `vSphere client` 创建虚拟机，vSphere client 如何创建虚拟机请看[这里](https://blog.csdn.net/xixihahalelehehe/article/details/129310311)。

需求：

- 系统： Rocky Linux 8.7

- CPU: 4

- MEM: 8G

- DISK: 30G

[Rocky Linux 9.1 新手入门指南](https://blog.csdn.net/xixihahalelehehe/article/details/129644123)



下载介质机器：
- `192.168.26.10` 连网在线拉取需要的介质（脚本、镜像、安装包等等）。

集群机器配置地址与主机名：（注意：网络为离线环境）。

```bash
100.168.110.199 registry01
100.168.110.21 kube-control-plan01
100.168.110.31 dbscale-control-plan01

100.168.110.41 kube-node01
100.168.110.42 kube-node02
100.168.110.43 kube-node03
```
## 3. 配置代理
加速下载介质，例如：github、quay.io、docker hub、registry.k8s.io

- [https://github.com/wanhebin/clash-for-linux.git](https://github.com/wanhebin/clash-for-linux.git)



## 4. 下载介质
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


用时 30分钟 下载完成介质。

> 注意：如果你喜欢通过容器部署集群，需要下载 kubespray 镜像，默认 `quay.io/kubespray/kubespray:v2.21.0`  ，存在一个 pip 包依赖（jmespath 1.0.1），下面镜像已重新编译。

                              
```bash
nerdctl pull ghostwritten/kubespray:v2.21-a94b893e2
nerdctl save -o kubespray:v2.21-a94b893e2.tar ghostwritten/kubespray:v2.21-a94b893e2
```

## 5. 初始化配置

打包下载的介质传输到离线环境：

```bash
cd ../
tar czvf kubespray-offline-2.21.0.tar.gz kubespray-offline/
```
登陆部署节点（registry01），然后在输出目录中运行以下脚本：

```bash
tar zxvf kubespray-offline-2.21.0.tar.gz
cd kubespray-offline/outputs
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

## 6. 安装部署工具

以下两种部署环境只选其一即可。

### 6.1 配置 venv 部署环境
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
$ cd kubespray-{version}
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

### 6.2  配置容器部署环境
```bash
 nerdctl  run --name=kubespray:v2.21-a94b893e2 --network=host --rm -itd --mount type=bind,source="$(pwd)"/kubespray-offline-2.21.0-0,dst=/kubespray\
     --mount type=bind,source="${HOME}"/.ssh,dst=/root/.ssh \
     --mount type=bind,source=/etc/hosts,dst=/etc/hosts \
     quay.io/kubespray/kubespray:v2.21-a94b893e2  bash
```

## 7. 配置互信

```bash
ssh-copy-id root@100.168.110.21
ssh-copy-id root@100.168.110.31
ssh-copy-id root@100.168.110.41
ssh-copy-id root@100.168.110.42
ssh-copy-id root@100.168.110.43
```

##  8. 编写 inventory.ini

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

## 9. 编写 offline.yml

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


## 10. 部署 offline repo 

使用ansible将使用yum_rep的离线存储库配置部署到所有目标节点。
首先，将离线设置playbook复制到kubespray目录。

```bash
$ cp -r /root/kubespray-offline-2.21.0-0/playbook  /root/kubespray-offline-2.21.0-0/outputs/kubespray-2.21.0/
```
Then execute `offline-repo.yml` playbook.

```bash
$ cd  /root/kubespray-offline-2.21.0-0/outputs/kubespray-2.21.0/
ansible -i inventory/local/inventory.ini all -m shell -a  "mv /etc/yum.repos.d /tmp"
ansible -i inventory/local/inventory.ini all -m shell -a  "mkdir /etc/yum.repos.d"
ansible -i inventory/local/inventory.ini all -m copy -a "src=/tmp/yum.repos.d/offline.repo dest=/etc/yum.repos.d/"

$ ansible-playbook -i inventory/local/inventory.ini offline-repo.yml
```

注意：如果此 `yum` 源配置如果不能为部署集群提供完全的依赖，我们可以利用镜像自带的 yum源提供支持，我们需要以下配置：

```bash
mkdir /root/kubespray-offline-2.21.0-0/outputs/rpms/cdrom
mount -t iso9660 /dev/cdrom /root/kubespray-offline-2.21.0-0/outputs/rpms/cdrom
```
查看

```bash
$ df -Th |grep cdrom
/dev/sr0            iso9660    10G   10G     0 100% /mnt/cdrom
$ ls /mnt/cdrom/
AppStream  BaseOS  EFI  images  isolinux  LICENSE  media.repo  TRANS.TBL
```

持久挂载
```bash
$ vim /etc/fstab
/dev/sr0 /mnt/cdrom iso9660 defaults 0 0
```
本地配置yum源，在`offline.repo`添加：

```bash
$ cat /etc/yum.repos.d/offline.repo
[offline-repo]
name=Offline repo
baseurl=http://localhost/rpms/local/
enabled=1
gpgcheck=0

[BaseOS]
name=BaseOS
baseurl=http://localhost/rpms/cdrom/BaseOS
enable=1
gpgcheck=0

[AppStream]
name=AppStream
baseurl=http://localhost/rpms/cdrom/AppStream
enabled=1
gpgcheck=0
```
其他集群节点 yum 源配置：

```bash
 cat /etc/yum.repos.d/offline.repo
[offline-repo]
async = 1
baseurl = http://100.168.110.199/rpms/local
enabled = 1
gpgcheck = 0
name = Offline repo for kubespray


[BaseOS]
name=BaseOS
baseurl=http://100.168.110.199/rpms/cdrom/BaseOS
enable=1
gpgcheck=0

[AppStream]
name=AppStream
baseurl=http://100.168.110.199/rpms/cdrom/AppStream
enabled=1
gpgcheck=0
```
验证：

```bash
$ yum clean all
$ yum repolist --verbose
Loaded plugins: builddep, changelog, config-manager, copr, debug, debuginfo-install, download, generate_completion_cache, groups-manager, needs-restarting, playground, repoclosure, repodiff, repograph, repomanage, reposync, versionlock
YUM version: 4.7.0
cachedir: /var/cache/dnf
Last metadata expiration check: 1:36:06 ago on Wed 19 Apr 2023 01:59:27 PM CST.
Repo-id            : AppStream
Repo-name          : AppStream
Repo-revision      : 8.5
Repo-distro-tags      : [cpe:/o:rocky:rocky:8]:  ,  , 8, L, R, c, i, k, n, o, u, x, y
Repo-updated       : Sun 14 Nov 2021 05:25:39 PM CST
Repo-pkgs          : 6,163
Repo-available-pkgs: 5,279
Repo-size          : 8.0 G
Repo-baseurl       : http://100.168.110.199/rpms/cdrom/AppStream
Repo-expire        : 172,800 second(s) (last: Tue 18 Apr 2023 10:24:35 PM CST)
Repo-filename      : /etc/yum.repos.d/offline.repo

Repo-id            : BaseOS
Repo-name          : BaseOS
Repo-revision      : 8.5
Repo-distro-tags      : [cpe:/o:rocky:rocky:8]:  ,  , 8, L, R, c, i, k, n, o, u, x, y
Repo-updated       : Sun 14 Nov 2021 05:23:13 PM CST
Repo-pkgs          : 1,708
Repo-available-pkgs: 1,706
Repo-size          : 1.2 G
Repo-baseurl       : http://100.168.110.199/rpms/cdrom/BaseOS
Repo-expire        : 172,800 second(s) (last: Tue 18 Apr 2023 10:24:35 PM CST)
Repo-filename      : /etc/yum.repos.d/offline.repo

Repo-id            : offline-repo
Repo-name          : Offline repo for kubespray
Repo-revision      : 1681789393
Repo-updated       : Tue 18 Apr 2023 11:43:14 AM CST
Repo-pkgs          : 265
Repo-available-pkgs: 262
Repo-size          : 182 M
Repo-baseurl       : http://100.168.110.199/rpms/local
Repo-expire        : 172,800 second(s) (last: Wed 19 Apr 2023 01:59:27 PM CST)
Repo-filename      : /etc/yum.repos.d/offline.repo
Total packages: 8,136
```
## 11. 部署 kubernetes

部署前由于 `kubespray v2.21.0` 存在 bug, 需要做以下修改：

 1. TASK [container-engine/containerd : containerd Create registry directories]

报错内容：
```bash
fatal: [kube-control-plan01]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'ansible.utils.unsafe_proxy.AnsibleUnsafeText object' has no attribute 'key'\n\nThe error appears to be in '/root/kubespray-offline-2.21.0/outputs/kubespray-2.21.0/roles/container-engine/containerd/tasks/main.yml': line 114, column 3, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n\n- name: containerd Create registry directories\n  ^ here\n"}
```
修改 此文件`with_items` 改为 `with_dict`
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

 2. TASK [kubernetes/control-plane : kubeadm | Initialize first master]

```bash
TASK [kubernetes/control-plane : kubeadm | Initialize first master] ****************************************************************************************************************
fatal: [kube-control-plan01]: FAILED! => {"attempts": 3, "changed": true, "cmd": ["timeout", "-k", "300s", "300s", "/usr/local/bin/kubeadm", "init", "--config=/etc/kubernetes/kubeadm-config.yaml", "--ignore-preflight-errors=all", "--skip-phases=addon/coredns", "--upload-certs"], "delta": "0:01:56.126009", "end": "2023-04-10 21:41:40.621046", "failed_when_result": true, "msg": "non-zero return code", "rc": 1, "start": "2023-04-10 21:39:44.495037", "stderr": "W0410 21:39:44.514437  222884 utils.go:69] The recommended value for \"clusterDNS\" in \"KubeletConfiguration\" is: [10.233.0.10]; the provided value is: [169.254.25.10]\n\t[WARNING FileAvailable--etc-kubernetes-manifests-kube-apiserver.yaml]: /etc/kubernetes/manifests/kube-apiserver.yaml already exists\n\t[WARNING FileAvailable--etc-kubernetes-manifests-kube-controller-manager.yaml]: /etc/kubernetes/manifests/kube-controller-manager.yaml already 
```
由于containerd 配置导致 kubelet 启动失败：`unknown service runtime.v1alpha2.ImageService`

原因：
- [https://github.com/containerd/containerd/issues/4581](https://github.com/containerd/containerd/issues/4581)

因此，我们对`containerd` 配置模板进行修改删掉仓库相关配置，内容如下：

```bash
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

开始执行部署：
```bash
$ ansible-playbook -i inventory/local/inventory.ini --become --become-user=root cluster.yml
```
检查
```bash
[root@kube-control-plan01 ~]# kubectl get node
NAME                     STATUS   ROLES           AGE     VERSION
dbscale-control-plan01   Ready    <none>          3m55s   v1.25.6
kube-control-plan01      Ready    control-plane   4m55s   v1.25.6
kube-node01              Ready    <none>          3m55s   v1.25.6
kube-node02              Ready    <none>          3m55s   v1.25.6
kube-node03              Ready    <none>          3m55s   v1.25.6
[root@kube-control-plan01 ~]# kubectl get pod -A
NAMESPACE     NAME                                          READY   STATUS                       RESTARTS        AGE
kube-system   calico-kube-controllers-f9c878968-2q8lj       1/1     Running                      0               2m38s
kube-system   calico-node-7m647                             1/1     Running                      0               3m41s
kube-system   calico-node-bz275                             1/1     Running                      0               3m41s
kube-system   calico-node-mmrkb                             1/1     Running                      0               3m41s
kube-system   calico-node-qs5r5                             1/1     Running                      0               3m41s
kube-system   calico-node-wntkf                             1/1     Running                      0               3m41s
kube-system   coredns-6747dfb59-v5ln6                       1/1     Running                      0               2m26s
kube-system   coredns-6747dfb59-v75q7                       1/1     Running                      0               2m23s
kube-system   dns-autoscaler-67f85d7d9d-cr6mc               1/1     Running                      0               2m25s
kube-system   kube-apiserver-kube-control-plan01            1/1     Running                      0               5m2s
kube-system   kube-controller-manager-kube-control-plan01   1/1     Running                      1               5m2s
kube-system   kube-proxy-4q57q                              1/1     Running                      0               4m1s
kube-system   kube-proxy-hv24m                              1/1     Running                      0               4m1s
kube-system   kube-proxy-srs6x                              1/1     Running                      0               4m1s
kube-system   kube-proxy-z8ftz                              1/1     Running                      0               4m1s
kube-system   kube-proxy-zrw7q                              1/1     Running                      0               4m1s
kube-system   kube-scheduler-kube-control-plan01            1/1     Running                      1               5m
kube-system   nginx-proxy-dbscale-control-plan01            1/1     Running                      0               2m45s
kube-system   nginx-proxy-kube-node01                       1/1     Running                      0               3m49s
kube-system   nginx-proxy-kube-node02                       1/1     Running                      0               2m56s
kube-system   nginx-proxy-kube-node03                       1/1     Running                      0               2m52s
kube-system   nodelocaldns-dcksd                            1/1     Running                      0               2m23s
kube-system   nodelocaldns-dkhz2                            1/1     Running                      0               2m23s
kube-system   nodelocaldns-hrz9g                            1/1     Running                      0               2m23s
kube-system   nodelocaldns-lfzhh                            1/1     Running                      0               2m23s
kube-system   nodelocaldns-r5dd8                            1/1     Running                      0               2m23s
```

- [更多排错细节请参考](https://blog.csdn.net/xixihahalelehehe/article/details/130246696)


参考：
- github：[https://github.com/kubernetes-sigs/kubespray](https://github.com/kubernetes-sigs/kubespray)
- 官网：[https://kubespray.io/#/](https://kubespray.io/#/)
- 网友kubespray 学习：[https://github.com/wenwenxiong/book/tree/master/k8s/kubespray](https://github.com/wenwenxiong/book/tree/master/k8s/kubespray)
