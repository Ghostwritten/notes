![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/5b212d73ce014eb59da2839ac1e278d1.png)







本指南介绍如何使用 Kubespray 在多个节点上部署 Kubernetes 集群。本方案基于 Podman 运行 Kubespray 容器，并利用 Ansible 进行自动化部署，适用于希望快速搭建 Kubernetes 集群的用户。


- 查看kubespray v2.27.0 版本各个软件版本：[https://github.com/kubernetes-sigs/kubespray/blob/v2.27.0/roles/kubespray-defaults/defaults/main/download.yml](https://github.com/kubernetes-sigs/kubespray/blob/v2.27.0/roles/kubespray-defaults/defaults/main/download.yml)

## 1. 基础配置

### 1.1 设置主机名
在所有节点上执行以下命令，配置主机名：

```bash
192.168.21.170:
hostnamectl set-hostname bastion01
192.168.21.171:
hostnamectl set-hostname kube-master01
192.168.21.172:
hostnamectl set-hostname kube-node01
192.168.21.173:
hostnamectl set-hostname kube-node02
192.168.21.174:
hostnamectl set-hostname kube-node03
192.168.21.175:
hostnamectl set-hostname kube-node04
```

### 1.2 配置网络
在所有节点上编辑网络配置文件：

```bash
vim /etc/NetworkManager/system-connections/ens192.nmconnection
```

示例配置：

```ini
[connection]
id=ens192
uuid=f4dbd335-66ff-3056-9fcb-bbb3b4dd9543
type=ethernet
autoconnect-priority=-999
interface-name=ens192
timestamp=1736215902

[ethernet]

[ipv4]
address1=192.168.21.170/20,192.168.21.1
DNS=127.0.0.1
method=manual

[ipv6]
addr-gen-mode=eui64
method=auto

[proxy]
```

重启网络连接以使配置生效：

```bash
nmcli connection reload && nmcli connection down ens192 && nmcli connection up ens192
```

> **建议**：完成所有节点的网络配置后，关机并创建快照，以便后续部署时能够快速恢复。

### 1.3 配置代理（仅在 bastion01 上）
为了加快 GitHub 及其他工具的访问速度，可以在 `bastion01` 配置代理：

```bash
vim /root/.bashrc
```

追加以下内容：

```bash
ENV_PROXY="http://192.168.21.101:7890"

enable_proxy() {
    export HTTP_PROXY="$ENV_PROXY"
    export HTTPS_PROXY="$ENV_PROXY"
    export ALL_PROXY="$ENV_PROXY"
    export NO_PROXY="localhost,127.0.0.1/8,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16"

    export http_proxy="$ENV_PROXY"
    export https_proxy="$ENV_PROXY"
    export all_proxy="$ENV_PROXY"
    export no_proxy="$NO_PROXY"
}

disable_proxy() {
    unset HTTP_PROXY HTTPS_PROXY ALL_PROXY NO_PROXY
    unset http_proxy https_proxy all_proxy no_proxy
}

enable_proxy
```

使配置生效：

```bash
source /root/.bashrc
```

### 1.4 配置 SSH 互信（在 bastion01 上执行）

```bash
ssh-keygen
```

安装 `sshpass` 并批量分发公钥：

```bash
dnf -y install sshpass
for i in {170..175}; do sshpass -p 'root' ssh-copy-id -o StrictHostKeyChecking=no -i ~/.ssh/id_rsa.pub root@192.168.21.$i; done
```

## 2. Kubernetes 集群安装

### 2.1 运行 Kubespray 部署容器

```bash
podman run -it --net=host -v "${HOME}"/.ssh/id_rsa:/root/.ssh/id_rsa --name kubespray \
  quay.io/kubespray/kubespray:v2.27.0 bash
```

### 2.2 编写 `inventory.ini`

在 `bastion01` 进入 Kubespray 容器，并创建 `inventory.ini`：

```bash
vim inventory/sample/inventory.ini
```

示例内容：

```ini
[all]
kube-master01 ansible_host=192.168.21.171
kube-node01 ansible_host=192.168.21.172
kube-node02 ansible_host=192.168.21.173
kube-node03 ansible_host=192.168.21.174
kube-node04 ansible_host=192.168.21.175

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

### 2.3 配置 Ansible 别名

```bash
vim /root/.bashrc
```

追加：

```bash
alias ansible='ansible -i inventory/sample/inventory.ini'
```

加载配置并测试连接：

```bash
source /root/.bashrc
ansible all -m ping
```

### 2.4 配置代理（可选）

```bash
vim inventory/sample/group_vars/all/all.yml
```

```yaml
http_proxy: "http://192.168.21.101:7890"
https_proxy: "http://192.168.21.101:7890"
no_proxy: "localhost,127.0.0.0/8,10.0.0.0/8,192.168.0.0/16,*.coding.net,*.tencentyun.com,*.myqcloud.com"
```

### 2.5 开始安装 Kubernetes 集群

```bash
ansible-playbook -i inventory/sample/inventory.ini --private-key /root/.ssh/id_rsa cluster.yml
```

output:

```bash
PLAY RECAP ***********************************************************************************************************************************
kube-master01              : ok=668  changed=146  unreachable=0    failed=0    skipped=1072 rescued=0    ignored=6   
kube-node01                : ok=459  changed=91   unreachable=0    failed=0    skipped=669  rescued=0    ignored=1   
kube-node02                : ok=459  changed=91   unreachable=0    failed=0    skipped=667  rescued=0    ignored=1   
kube-node03                : ok=459  changed=91   unreachable=0    failed=0    skipped=667  rescued=0    ignored=1   
kube-node04                : ok=459  changed=91   unreachable=0    failed=0    skipped=667  rescued=0    ignored=1   

Thursday 06 February 2025  06:09:39 +0000 (0:00:00.054)       0:09:20.636 ***** 
=============================================================================== 
bootstrap-os : Check RHEL subscription-manager status -------------------------------------------------------------------------------- 29.99s
download : Download_container | Download image if required --------------------------------------------------------------------------- 29.44s
container-engine/containerd : Download_file | Download item -------------------------------------------------------------------------- 24.42s
download : Download_file | Download item --------------------------------------------------------------------------------------------- 17.68s
download : Download_file | Download item --------------------------------------------------------------------------------------------- 16.25s
kubernetes/kubeadm : Join to cluster if needed --------------------------------------------------------------------------------------- 15.91s
download : Download_container | Download image if required --------------------------------------------------------------------------- 15.72s
download : Download_file | Download item --------------------------------------------------------------------------------------------- 15.60s
download : Download_file | Download item --------------------------------------------------------------------------------------------- 14.57s
download : Download_container | Download image if required --------------------------------------------------------------------------- 13.46s
bootstrap-os : Install libselinux python package ------------------------------------------------------------------------------------- 13.01s
kubernetes/preinstall : Install packages requirements -------------------------------------------------------------------------------- 11.15s
download : Download_file | Download item --------------------------------------------------------------------------------------------- 10.88s
download : Download_container | Download image if required ---------------------------------------------------------------------------- 9.55s
bootstrap-os : Ensure iproute is installed -------------------------------------------------------------------------------------------- 9.50s
download : Download_container | Download image if required ---------------------------------------------------------------------------- 9.03s
download : Download_container | Download image if required ---------------------------------------------------------------------------- 8.99s
download : Download_file | Download item ---------------------------------------------------------------------------------------------- 8.97s
container-engine/crictl : Download_file | Download item ------------------------------------------------------------------------------- 8.70s
download : Download_container | Download image if required ---------------------------------------------------------------------------- 8.61s
root@bastion01:/kubespray# 
```

### 2.6 配置 `kubectl`

```bash
mkdir -p /root/.kube
scp root@192.168.21.171:/usr/local/bin/kubectl /usr/local/bin/
scp root@192.168.21.171:/etc/kubernetes/admin.conf /root/.kube/config
vim /root/.kube/config
```

修改 `server` 地址为 `kube-master01` 的 IP：

```yaml
server: https://192.168.21.171:6443
```

测试 Kubernetes 运行状态：

```bash
$ kubectl get node
NAME            STATUS   ROLES           AGE   VERSION
kube-master01   Ready    control-plane   8d    v1.31.4
kube-node01     Ready    <none>          8d    v1.31.4
kube-node02     Ready    <none>          8d    v1.31.4
kube-node03     Ready    <none>          8d    v1.31.4
kube-node04     Ready    <none>          8d    v1.31.4
```

### 2.7 配置 `kubectl` 命令别名

```bash
cat >> ~/.bashrc << EOF
alias kg='kubectl get'
alias k='kubectl'
alias kd='kubectl describe pods'
alias ke='kubectl explain'
alias ka='kubectl apply'
EOF
source ~/.bashrc
```

验证 Kubernetes 组件状态：

```bash

$ kg pod -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-5db5978889-kgckn   1/1     Running   0          7m7s
kube-system   calico-node-bx7wc                          1/1     Running   0          7m18s
kube-system   calico-node-r6njt                          1/1     Running   0          7m18s
kube-system   calico-node-r6plc                          1/1     Running   0          7m18s
kube-system   calico-node-sndfx                          1/1     Running   0          7m18s
kube-system   calico-node-zs687                          1/1     Running   0          7m18s
kube-system   coredns-d665d669-59clr                     1/1     Running   0          7m5s
kube-system   coredns-d665d669-bqgdx                     1/1     Running   0          7m3s
kube-system   dns-autoscaler-5cb4578f5f-kp5rs            1/1     Running   0          7m4s
kube-system   kube-apiserver-kube-master01               1/1     Running   1          7m58s
kube-system   kube-controller-manager-kube-master01      1/1     Running   2          7m58s
kube-system   kube-proxy-825b5                           1/1     Running   0          7m31s
kube-system   kube-proxy-cfbwt                           1/1     Running   0          7m31s
kube-system   kube-proxy-klfqm                           1/1     Running   0          7m31s
kube-system   kube-proxy-mn8gt                           1/1     Running   0          7m31s
kube-system   kube-proxy-mq5r9                           1/1     Running   0          7m31s
kube-system   kube-scheduler-kube-master01               1/1     Running   1          7m58s
kube-system   nginx-proxy-kube-node01                    1/1     Running   0          7m33s
kube-system   nginx-proxy-kube-node02                    1/1     Running   0          7m33s
kube-system   nginx-proxy-kube-node03                    1/1     Running   0          7m33s
kube-system   nginx-proxy-kube-node04                    1/1     Running   0          7m24s
kube-system   nodelocaldns-dtm75                         1/1     Running   0          7m3s
kube-system   nodelocaldns-nxdwv                         1/1     Running   0          7m3s
kube-system   nodelocaldns-pbs4v                         1/1     Running   0          7m3s
kube-system   nodelocaldns-r284k                         1/1     Running   0          7m3s
kube-system   nodelocaldns-x9qq2                         1/1     Running   0          7m3s
                                  

```


## 3. 结语
通过本指南，您可以成功使用 Kubespray 在多台节点上快速部署 Kubernetes 集群。如果后续需要进行维护或扩展，可参考 Kubespray 官方文档获取更多信息。


参考： 

- [https://github.com/kubernetes-sigs/kubespray](https://github.com/kubernetes-sigs/kubespray)

