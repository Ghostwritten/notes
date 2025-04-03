

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/3688a38398f64a28b38068613b079bf4.png)




## 1. 简介
Kubespray 是一个开源项目，旨在通过 Ansible 简化生产就绪的 Kubernetes 集群的部署。它提供了灵活且可定制的框架，支持多种云提供商和本地环境，包括 AWS、Google Cloud Engine、Azure、OpenStack、vSphere 和裸金属环境。这种多样性使用户能够根据需求选择合适的基础设施来运行 Kubernetes 集群。Kubespray 支持创建高可用的 Kubernetes 集群，确保即使在节点故障时，应用程序也能保持正常运行。此外，用户可以选择多种网络插件，如 Calico、Flannel、Canal、Cilium 和 Weave，以便根据具体需求定制网络解决方案。Kubespray 兼容多种流行的 Linux 发行版，包括 CoreOS、Debian、Ubuntu、CentOS/RHEL、Fedora 和 openSUSE，使其适用于广泛的操作系统环境。安装过程相对简单，用户只需安装 Ansible 和所需的 Python 包，然后设置清单并执行 Ansible playbook 来完成 Kubernetes 集群的部署。Kubespray 的 GitHub 页面提供了全面的文档，包括入门指南、部署变量和网络配置等内容，帮助用户快速上手。此外，Kubespray 拥有活跃的社区，通过 Slack 等渠道为用户提供支持。总之，Kubespray 是一个强大的工具，可以高效地在各种环境中部署 Kubernetes 集群，其灵活性和多样性使其成为希望在基础设施中实施 Kubernetes 的组织的理想选择。对于希望实现高可用性和可定制化的用户来说，Kubespray 提供了一种可靠且易于使用的解决方案，是现代云原生应用开发和管理的重要工具。有关更多详细信息和资源，用户可以访问其 GitHub 仓库，以获取最新的更新和社区支持。

## 2. 环境配置

| No. | Virtual Machine Name                         | Hostname      | Iso        | Ip                              | Cpu | Mem | Disk（G） |
|-----|----------------------------------------------|---------------|------------|---------------------------------|-----|-----|---------|
| 1   | 10.80.0.70-RHEL-9.5-Training02-bastion01 | bastion01     | Rocky 9.4 | 10.80.0.70/24 、192.168.21.189/20、10.80.0.1 | 4   | 8   | 100(sys)+100(data)     |
| 2   | 10.80.0.71-RHEL-9.5-Training02-kube-master01 | kube-master01 | Rocky 9.4 | 10.80.0.71/24                   | 16  | 16  | 100     |
| 3   | 10.80.0.72-RHEL-9.5-Training02-kube-node01   | kube-node01   | Rocky 9.4 | 10.80.0.72/24                   | 16  | 32  |100(sys)+100(data)     |
| 4   | 10.80.0.73-RHEL-9.5-Training02-kube-node02   | kube-node2    | Rocky 9.4 | 10.80.0.73/24                   | 16  | 32  | 100(sys)+200(data)      |
| 5   | 10.80.0.74-RHEL-9.5-Training02-kube-node03   | kube-node3    | Rocky 9.4 | 10.80.0.74/24                   | 16  | 32  | 100(sys)+200(data)      |
| 6   | 10.80.0.75-RHEL-9.5-Training02-kube-node04   | kube-node4    | Rocky 9.4 | 10.80.0.75/24                   | 16  | 32  | 100(sys)+200(data)     |



## 3. 创建虚拟机

- 下载 [rhel-9.5-x86_64-dvd.iso](https://access.redhat.com/downloads/content/rhel)；
- 参考 [Vcenter 定制创建 Rocky Linux 虚拟机](https://ghostwritten.blog.csdn.net/article/details/136652999)创建虚拟机模版；
- 关闭防火墙、selinux；
- dnf -y update;
- 更新时间同步；
- 克隆各个虚拟机并配置cpu、mem、disk。
- 手动修改ip、dns、gateway、配置主机名。


用户密码：root/root

## 4. 配置网络

**bastion01节点配置联网地址，用于远程地址。**
```bash
$ cat /etc/NetworkManager/system-connections/ens192.nmconnection 
[connection]
id=ens192
uuid=84405101-5169-37ee-a911-185d8a0828b3
type=ethernet
autoconnect-priority=-999
interface-name=ens192
timestamp=1732156159

[ethernet]

[ipv4]
address1=192.168.21.189/20,192.168.21.1
DNS=192.168.21.2
method=manual

[ipv6]
addr-gen-mode=eui64
method=auto

[proxy]
```

重启网卡

```bash
nmcli connection reload && nmcli connection down ens192 && nmcli connection up ens192
```

**bastion01 配置一个模拟内部网关地址。**
```bash
$ vim /etc/NetworkManager/system-connections/ens256.nmconnection 
[connection]
id=ens256
uuid=84405101-5169-37ee-a911-185d8a0828d5
type=ethernet
autoconnect-priority=-999
interface-name=ens256
timestamp=1732156159

[ethernet]

[ipv4]
address1=10.80.0.1/24
method=manual

[ipv6]
addr-gen-mode=eui64
method=auto

[proxy]
```
**bastion01 节点配置内部地址（每个节点都要执行该配置，更改uuid、address1、dns）。**

```bash
$ vim /etc/NetworkManager/system-connections/ens224.nmconnection 
[connection]
id=ens224
uuid=84405101-5169-37ee-a911-185d8a0828c4
type=ethernet
autoconnect-priority=-999
interface-name=ens224
timestamp=1732156159

[ethernet]

[ipv4]
address1=10.80.0.70/24,10.80.0.1
dns=10.80.0.70;
method=manual

[ipv6]
addr-gen-mode=eui64
method=auto

[proxy]
```



## 5. 解压介质

kubespray 2.27.0 离线安装 kubernetes v1.31.4 介质：
- `kubespray-offline-2.27.0.7z` 百度网盘: [https://pan.baidu.com/s/1UobgCHLTwis3Q00OqwHGqw?pwd=itpr](https://pan.baidu.com/s/1UobgCHLTwis3Q00OqwHGqw?pwd=itpr) 提取码: itpr 


上传介质至部署节点（bastion01）略

安装解压工具

```bash
dnf -y install epel-release
sudo yum install p7zip p7zip-plugins
```

解压介质
```bash
$ 7z x kubespray-offline-2.27.0.7z
$ cd kubespray-offline-2.27.0
```




## 6. 部署镜像仓库

### 6.1 创建镜像存储卷

方案1 单独创建卷组

```bash
pvcreate /dev/sdb
vgcreate data-vg /dev/sdb
lvcreate -L +100%FREE -n data-lv data-vg
mkfs.xfs /dev/data-vg/data-lv
echo "/dev/data-vg/data-lv /data xfs defaults 0 0" >> /etc/fstab
mkdir /data
mount -a
```

方案2: 扩容根(/)目录

```bash
pvcreate /dev/sdb
vgextend rl /dev/sdb
lvextend -l +100%FREE /dev/rl/root
xfs_growfs /dev/rl/root
```


### 6.2 安装 Docker



安装 docker
```bash
rm -rf docker
tar zxf docker-28.0.1.tgz
cp docker/* /usr/bin
mkdir /etc/docker
cat > /usr/lib/systemd/system/docker.service << EOF
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target
[Service]
Type=notify
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TimeoutStartSec=0
Delegate=yes
KillMode=process
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s
[Install]
WantedBy=multi-user.target
EOF
cat <<EOF> /etc/docker/daemon.json 
{
   "exec-opts": ["native.cgroupdriver=systemd"],
   "insecure-registries": ["10.80.0.70:35000"],
   "data-root": "/data/docker",
   "live-restore": true,
   "log-driver": "json-file",
   "log-opts": {
     "max-size":  "100m",
     "max-file": "5"
    }
 }
systemctl daemon-reload;systemctl start docker;systemctl enable docker && systemctl status docker
```

### 6.3 安装 Docker Registry
```bash
$ cd /root/kubespray-offline-2.27.0/outputs/images
$ docker load -i docker.io_library_registry-2.8.2.tar.gz
docker run -tid --restart=always --name registry -p 35000:5000 -e REGISTRY_COMPATIBILITY_SCHEMA1_ENABLED=true -v /data/registry:/var/lib/registry registry:2.8.2
```


## 7. 推送镜像入库

```bash
$ cd /root/kubespray-offline-2.27.0/outputs
$ sh load-push-all-images.sh
```

## 8. 启动 Nginx

```bash
$ cd /root/kubespray-offline-2.27.0/outputs
$ docker load -i  images/docker.io_library_nginx-1.27.3.tar.gz
$ docker run -tid \
    --network host \
    --restart always \
    --name nginx \
    -v ./:/usr/share/nginx/html \
    -v ./nginx-default.conf:/etc/nginx/conf.d/default.conf \
    nginx:1.27.3
```

## 9. 配置互信

```bash
$ ssh-keygen
$ dnf -y install sshpass
$ vim hosts.txt
10.80.0.70
10.80.0.71
10.80.0.72
10.80.0.73
10.80.0.74
10.80.0.75

$ for host in $(cat hosts.txt); do sshpass -p 'root' ssh-copy-id -o StrictHostKeyChecking=no -i ~/.ssh/id_rsa.pub root@$host;done

```


## 10. 启动部署容器

```bash
$ cd /root/kubespray-offline-2.27.0
$ docker load -i quay.io_kubespray_kubespray_v2.27.0.tar
$ docker run --name kubespray-offline-ansible --network=host  -it -v "/root/kubespray-offline-2.27.0/outputs/kubespray-2.27.0:/kubespray" -v "${HOME}"/.ssh:/root/.ssh  -v /etc/hosts:/etc/hosts quay.io/kubespray/kubespray:v2.27.0  bash
```

## 11. 编辑 inventory.ini

```bash
$ vim inventory/sample/inventory.ini
[all]
kube-master01 ansible_host=10.80.0.71 ip=10.80.0.71
kube-node01 ansible_host=10.80.0.72 ip=10.80.0.72
kube-node02 ansible_host=10.80.0.73 ip=10.80.0.73
kube-node03 ansible_host=10.80.0.74 ip=10.80.0.74
kube-node04 ansible_host=10.80.0.75 ip=10.80.0.75

[bastion]

[kube_control_plane]
kube-master01

[etcd]
kube-master01

[kube_node]
kube-node01
kube-node02
kube-node03
kube-node04

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr
```
配置别名

```bash
$ vim /root/.bashrc
..
alias ansible='ansible -i /kubespray/inventory/sample/inventory.ini'
...
$ source /root/.bashrc
$ ansible all  -m ping
```
输出：

```bash
kube-master01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
kube-node03 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
kube-node01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
kube-node04 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
kube-node02 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```



## 12. 编写 offline.yml

```bash
$ vim inventory/sample/group_vars/all/offline.yml
http_server: "http://10.80.0.70"
registry_host: "10.80.0.70:35000"

containerd_insecure_registries: # Kubespray #8340
  "10.80.0.70:35000": "http://10.80.0.70:35000"

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

其他可能的定制需求：

pod & service网段（可略）：

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

## 13. 配置 containerd.yaml

```bash
$ vim inventory/sample/group_vars/all/containerd.yml
...
- prefix: 10.80.0.70:35000
  mirrors:
  - host: http://10.80.0.70:35000
    capabilities: ["pull", "resolve"]
    skip_verify: true
```

## 14. 部署 offline repo

Redhat Linux 机器：
```bash
ansible all -m shell -a  "mv /etc/yum.repos.d /tmp"
ansible all -m shell -a  "mkdir /etc/yum.repos.d"
ansible-playbook -i inventory/sample/inventory.ini  playbook/offline-repo.yml
ansible  all  -m shell -a "sed -i 's/enabled=1/enabled=0/' /etc/yum/pluginconf.d/subscription-manager.conf"
ansible all -m shell -a  "mv /etc/yum.repos.d/redhat.repo /tmp/redhat.repo_bak"
ansible  all  -m shell -a "yum repolist --verbose"
```

Rocky Linux 机器：
```bash
ansible all -m shell -a  "mv /etc/yum.repos.d /tmp"
ansible all -m shell -a  "mkdir /etc/yum.repos.d"
ansible-playbook -i inventory/sample/inventory.ini  playbook/offline-repo.yml
ansible  all  -m shell -a "yum repolist --verbose"
```


## 15. 部署 k8s

```bash
ansible-playbook -i inventory/sample/inventory.ini  cluster.yml
```

输出：

```bash
PLAY RECAP ***********************************************************************************************************************************
kube-master01              : ok=664  changed=125  unreachable=0    failed=0    skipped=1074 rescued=0    ignored=7   
kube-node01                : ok=454  changed=70   unreachable=0    failed=0    skipped=671  rescued=0    ignored=2   
kube-node02                : ok=454  changed=70   unreachable=0    failed=0    skipped=669  rescued=0    ignored=2   
kube-node03                : ok=454  changed=70   unreachable=0    failed=0    skipped=669  rescued=0    ignored=2   
kube-node04                : ok=454  changed=70   unreachable=0    failed=0    skipped=669  rescued=0    ignored=2   

Monday 03 March 2025  13:03:18 +0000 (0:00:00.090)       0:06:18.189 ********** 
=============================================================================== 
kubernetes/kubeadm : Join to cluster if needed --------------------------------------------------------------------------------------- 16.05s
network_plugin/calico : Check if calico ready ---------------------------------------------------------------------------------------- 10.56s
etcd : Restart etcd ------------------------------------------------------------------------------------------------------------------- 7.16s
kubernetes/control-plane : Kubeadm | Initialize first control plane node -------------------------------------------------------------- 6.43s
etcd : Configure | Check if etcd cluster is healthy ----------------------------------------------------------------------------------- 5.24s
download : Download_container | Download image if required ---------------------------------------------------------------------------- 4.78s
container-engine/containerd : Download_file | Download item --------------------------------------------------------------------------- 4.01s
container-engine/runc : Download_file | Download item --------------------------------------------------------------------------------- 3.98s
download : Download_container | Download image if required ---------------------------------------------------------------------------- 3.91s
container-engine/crictl : Download_file | Download item ------------------------------------------------------------------------------- 3.89s
container-engine/nerdctl : Download_file | Download item ------------------------------------------------------------------------------ 3.88s
etcd : Configure | Ensure etcd is running --------------------------------------------------------------------------------------------- 3.65s
network_plugin/calico : Wait for calico kubeconfig to be created ---------------------------------------------------------------------- 3.61s
container-engine/crictl : Extract_file | Unpacking archive ---------------------------------------------------------------------------- 3.46s
container-engine/containerd : Containerd | Unpack containerd archive ------------------------------------------------------------------ 3.44s
download : Download_container | Download image if required ---------------------------------------------------------------------------- 3.41s
download : Download_file | Download item ---------------------------------------------------------------------------------------------- 3.17s
kubernetes-apps/ansible : Kubernetes Apps | CoreDNS ----------------------------------------------------------------------------------- 3.13s
download : Download_container | Download image if required ---------------------------------------------------------------------------- 3.10s
container-engine/nerdctl : Extract_file | Unpacking archive --------------------------------------------------------------------------- 3.09s
```

## 16. 配置kubeconfig
退出部署容器，回到bastion01主机本地。

```bash
mkdir /root/.kube
scp root@10.80.0.71:/usr/local/bin/kubectl /usr/local/bin/
scp root@10.80.0.71:/etc/kubernetes/admin.conf  /root/.kube/config
kubectl config set-cluster cluster.local --server=https://10.80.0.71:6443
```


配置快捷键

```bash
cat >> ~/.bashrc << EOF 
alias k='kubectl'
alias kg='kubectl get'
alias kd='kubectl describe pods'
alias ke='kubectl explain'
alias ka='kubectl apply'
EOF
source ~/.bashrc
```
检查集群
```bash
$ k get node
NAME            STATUS   ROLES           AGE     VERSION
kube-master01   Ready    control-plane   3m54s   v1.31.4
kube-node01     Ready    <none>          3m18s   v1.31.4
kube-node02     Ready    <none>          3m18s   v1.31.4
kube-node03     Ready    <none>          3m18s   v1.31.4
kube-node04     Ready    <none>          3m18s   v1.31.4
$ k get pod -A
NAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-9bcd797dc-nhfjm   1/1     Running   0          2m39s
kube-system   calico-node-mzcxs                         1/1     Running   0          2m57s
kube-system   calico-node-npzkq                         1/1     Running   0          2m57s
kube-system   calico-node-p62ks                         1/1     Running   0          2m57s
kube-system   calico-node-whgpz                         1/1     Running   0          2m57s
kube-system   calico-node-zpfqt                         1/1     Running   0          2m57s
kube-system   coredns-6b7cc677d8-gjf8g                  1/1     Running   0          2m36s
kube-system   coredns-6b7cc677d8-xvtdt                  1/1     Running   0          2m33s
kube-system   dns-autoscaler-7b5d87856d-kjp9x           1/1     Running   0          2m35s
kube-system   kube-apiserver-kube-master01              1/1     Running   1          3m53s
kube-system   kube-controller-manager-kube-master01     1/1     Running   2          3m53s
kube-system   kube-proxy-gkr7r                          1/1     Running   0          3m16s
kube-system   kube-proxy-m2mss                          1/1     Running   0          3m16s
kube-system   kube-proxy-nzwx8                          1/1     Running   0          3m16s
kube-system   kube-proxy-vfq8t                          1/1     Running   0          3m17s
kube-system   kube-proxy-xlth2                          1/1     Running   0          3m16s
kube-system   kube-scheduler-kube-master01              1/1     Running   1          3m54s
kube-system   nginx-proxy-kube-node01                   1/1     Running   0          3m19s
kube-system   nginx-proxy-kube-node02                   1/1     Running   0          3m11s
kube-system   nginx-proxy-kube-node03                   1/1     Running   0          3m14s
kube-system   nginx-proxy-kube-node04                   1/1     Running   0          3m17s
kube-system   nodelocaldns-24swx                        1/1     Running   0          2m33s
kube-system   nodelocaldns-7lc2w                        1/1     Running   0          2m33s
kube-system   nodelocaldns-llrqf                        1/1     Running   0          2m33s
kube-system   nodelocaldns-m969z                        1/1     Running   0          2m33s
kube-system   nodelocaldns-x7hdq                        1/1     Running   0          2m33s
```

参考：

- [https://docs.docker.com/engine/install/](https://docs.docker.com/engine/install/)
- [https://github.com/kubernetes-sigs/kubespray](https://github.com/kubernetes-sigs/kubespray)
- [https://github.com/kubespray-offline/kubespray-offline](https://github.com/kubespray-offline/kubespray-offline)
