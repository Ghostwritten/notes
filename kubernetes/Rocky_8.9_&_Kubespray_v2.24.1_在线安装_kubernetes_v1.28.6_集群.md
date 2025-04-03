

# Kubespray 在线安装 Kubernetes
## 1. 创建虚拟机

根据192.168.24.10-rocky-8.9-upm02-templ 模版克隆虚拟机，如下：

![](https://i-blog.csdnimg.cn/blog_migrate/d3de2f44ce713eab717c7a5221fcb736.png)

根据需求定制虚拟机资源

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/46c6eb44fb571f08c15783f9db27c217.png)

| 虚拟机 | role | hostname | cpu | mem | disk | 描述 |
| --- | --- | --- | --- | --- | --- | --- |
| 192.168.24.10-rocky-8.9-upm02-bastion01 | bastion | bastion01 | 4 | 4G | 100G | 部署节点 |
| 192.168.24.21-rocky-8.9-upm02-kube-master01 | control Plane | kube-master01  | 4 | 4G | 100G | 集群master |
| 192.168.24.31-rocky-8.9-upm02-kube-control01 | upm | kube-node01  | 8 | 16G | 100G+100G | UPM（platform and engine） |
| 192.168.24.41-rocky-8.9-upm02-kube-node01 | upm | kube-node02  | 8 | 16G | 100G+100G | 业务 |

## 2. 基础配置
### 2.1 配置 hostname
（所有节点）
```bash
1192.168.24.10:
hostnamctl set-hostname bastion01
192.168.24.21:
hostnamctl set-hostname kube-master01
192.168.24.31:
hostnamctl set-hostname kube-control01
192.168.24.41:
hostnamctl set-hostname kube-node01
```
### 2.2 配置网络
（所有节点）
```bash
$ vim /etc/sysconfig/network-scripts/ifcfg-ens192
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
NAME=ens192
DEVICE=ens192
ONBOOT=yes
IPADDR=192.168.48.6
GATEWAY=192.168.48.1
DNS1=8.8.8.8
DNS2=192.168.21.2
```
重启网卡
```bash
nmcli connection reload && nmcli connection down ens192 && nmcli connection up ens192
```
### 2.3 配置代理
(bastion01操作)
方便快速下载github.com,以及其他命令工具。
```bash
$ vim /root/.bashrc
```
追加以下内容：
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
执行生效
```
source /root/.bashrc
```
### 2.4 下载 kubespray
(bastion01操作)
从 [https://github.com/kubernetes-sigs/kubespray/releases](https://github.com/kubernetes-sigs/kubespray/releases) 指定版本下载
```bash
wget https://github.com/kubernetes-sigs/kubespray/archive/refs/tags/v2.24.1.zip
unzip v2.24.1.zip
cd kubespray-2.24.1
```
### 2.5 编写 inventory.ini
(bastion01操作)
```bash
$ vim /root/kubespray-2.24.1/inventory/sample/inventory.ini 
[all]
kube-master01 ansible_host=192.168.24.21
kube-control01 ansible_host=192.168.24.31 
kube-node01 ansible_host=192.168.24.41

[kube_control_plane]
kube-master01

[etcd]
kube-master01

[kube_node]
kube-control01
kube-node01

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr
```
### 2.6 配置互信
(bastion01操作)
```bash
ssh-keygen
for i in `cat /root/kubespray-2.24.1/inventory/sample/inventory.ini  |grep host| awk '{print $2}' | awk -F '=' '{print $2}'`;do ssh-copy-id root@$i;done
```
测试 ssh 远程是否互信
```bash
ssh root@192.168.24.51
ssh root@192.168.24.52
ssh root@192.168.24.53
```
### 2.7 运行 kubespray 容器
(bastion01操作)
```bash
dnf -y install docker
docker pull quay.io/kubespray/kubespray:v2.24.1
docker run -itd --net=host --mount type=bind,source=/root/kubespray-2.24.1/inventory/sample,dst=/inventory \
  --mount type=bind,source="${HOME}"/.ssh/id_rsa,dst=/root/.ssh/id_rsa --name kubespray \
  quay.io/kubespray/kubespray:v2.24.1 bash
docker exec -ti kubespray bash
root@383d20357b5c:/kubespray# vim /root/.bashrc 
....
alias ansible='ansible -i /inventory/inventory.ini'
....
root@383d20357b5c:/kubespray# source /root/.bashrc
root@383d20357b5c:/kubespray# ansible all -m ping
```
### 2.8 批量关闭防火墙、selinux 、swap
 (bastion01操作)
```bash
ansible all -m systemd -a "name=firewalld state=stopped enabled=no"
ansible all -m lineinfile -a "path=/etc/selinux/config regexp='^SELINUX=' line='SELINUX=disabled'" -b
ansible all -m shell -a "getenforce 0"
ansible all -m shell -a "sed -i '/.*swap.*/s/^/#/' /etc/fstab" -b
ansible all -m shell -a " swapoff -a && sysctl -w vm.swappiness=0"
```
### 2.9 批量路由转发
(bastion01操作)
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
### 2.10 批量修改最大文件句柄数和最大进程数
(bastion01操作)
```bash
ansible all -m lineinfile -a "path=/etc/security/limits.conf line='* soft nofile 655360\n* hard nofile 131072\n* soft nproc 655350\n* hard nproc 655350\n* soft memlock unlimited\n* hard memlock unlimited'" -b
```
## 3. 安装
(bastion01操作)
根据客户网络环境，配置代理,为了方便拉取镜像快速。
```bash
$ vim /inventory/group_vars/all/all.yml 
...
http_proxy: "http://192.168.21.101:7890"
https_proxy: "http://192.168.21.101:7890"
no_proxy: "localhost,127.0.0.0/8,169.0.0.0/8,10.0.0.0/8,192.168.0.0/16,*.coding.net,*.tencentyun.com,*.myqcloud.com"
...
```
运行
```bash
ansible-playbook -i /inventory/inventory.ini --private-key /root/.ssh/id_rsa cluster.yml
```
登陆kube-master01 节点:
拷贝 /root/.kube/config 至 bastion01 /root/.kube/config,拷贝/usr/local/bin/kubectl至bastion01 /usr/local/bin/
```bash
scp -r  /root/.kube/ root@192.168.24.10:/root/
scp -r  /usr/local/bin/kubectl root@192.168.24.10:/usr/local/bin/
```
返回 bastion01 编辑 /root/.kube/config
```bash
$ vim /root/.kube/config
....
server: https://192.168.24.21:6443 #127.0.0.1改为192.168.24.21
....
```
查看集群状态
```bash
$ kubectl get node
NAME             STATUS   ROLES           AGE     VERSION
kube-control01   Ready    <none>          5h33m   v1.28.6
kube-master01    Ready    control-plane   5h33m   v1.28.6
kube-node01      Ready    <none>          5h33m   v1.28.6

$ kubectl get pod -A
NAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-648dffd99-2rdsv   1/1     Running   0          2m10s
kube-system   calico-node-chjmm                         1/1     Running   0          2m33s
kube-system   calico-node-glvn6                         1/1     Running   0          2m33s
kube-system   calico-node-r5n44                         1/1     Running   0          2m33s
kube-system   coredns-77f7cc69db-ldz2h                  1/1     Running   0          110s
kube-system   coredns-77f7cc69db-xchbn                  1/1     Running   0          115s
kube-system   dns-autoscaler-8576bb9f5b-84s67           1/1     Running   0          112s
kube-system   kube-apiserver-kube-master01              1/1     Running   1          3m40s
kube-system   kube-controller-manager-kube-master01     1/1     Running   2          3m40s
kube-system   kube-proxy-8wzgl                          1/1     Running   0          2m58s
kube-system   kube-proxy-mbcn5                          1/1     Running   0          2m58s
kube-system   kube-proxy-mgtzx                          1/1     Running   0          2m58s
kube-system   kube-scheduler-kube-master01              1/1     Running   1          3m40s
kube-system   nginx-proxy-kube-control01                1/1     Running   0          3m3s
kube-system   nginx-proxy-kube-node01                   1/1     Running   0          3m2s
kube-system   nodelocaldns-bvfw4                        1/1     Running   0          110s
kube-system   nodelocaldns-kt29q                        1/1     Running   0          110s
kube-system   nodelocaldns-wkc8g                        1/1     Running   0          110s
```


参考： 

- [https://github.com/kubernetes-sigs/kubespray](https://github.com/kubernetes-sigs/kubespray)
