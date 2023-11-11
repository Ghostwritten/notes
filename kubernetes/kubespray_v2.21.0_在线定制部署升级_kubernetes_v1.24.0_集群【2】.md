







上一篇专门为了练习部署跑通。这篇为了学习定制安装部署，以及新增节点，删除节点，升级节点，动态申请pv，监控等一些部署测试，

##  简介
- github：[https://github.com/kubernetes-sigs/kubespray](https://github.com/kubernetes-sigs/kubespray)
- 官网：[https://kubespray.io/#/](https://kubespray.io/#/)
- 网友kubespray 学习：[https://github.com/wenwenxiong/book/tree/master/k8s/kubespray](https://github.com/wenwenxiong/book/tree/master/k8s/kubespray)

##  创建 虚拟机模板
- 下载 [rocky iso 8.7 iso](https://rockylinux.org/news/rocky-linux-8-7-ga-release/)
- 上传 rocky iso 8.7 iso 至 vcenter并安装，[vcenter 如何安装虚拟机请参考这篇文章](https://blog.csdn.net/xixihahalelehehe/article/details/129310311)

需求：

- 系统： Rocky Linux 8.7
- CPU: 4
- MEM: 8G
- DISK1: 60G 
- DISK2: 200G
##  虚拟机名称
- 192.168.50.20-rocky-8.7-up-bastion01
- 192.168.50.21-rocky-8.7-up-kube-controller01
- 192.168.50.41-rocky-8.7-up-kube-node01
- 192.168.50.42-rocky-8.7-up-kube-node02

##  配置静态地址
- [如何初始化 rocky linux 8.7 细节请参考这篇文章](https://blog.csdn.net/xixihahalelehehe/article/details/129644123)

```bash
$ cat /etc/sysconfig/network-scripts/ifcfg-ens192
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
UUID=da6c78ff-c1f0-4c05-8f0e-08848ab0a3e5
DEVICE=ens192
ONBOOT=yes
IPADDR=192.168.50.42
PREFIX=20
GATEWAY=192.168.48.1
DNS1=192.168.48.1
DNS2=8.8.8.8
IPV6_PRIVACY=no
```
重启生效

```bash
nmctl con reload; nmctl networking off ;nmctl networking on
```

## 配置代理
`registry.k8s.io` 镜像下载需要

<font color=red>【你懂得】




##  yum 配置
-  [linux yum 软件包管理](https://blog.csdn.net/xixihahalelehehe/article/details/105625395)

系统自带
```bash
$ ls /etc/yum.repos.d/
Rocky-AppStream.repo  Rocky-Devel.repo             Rocky-Media.repo  Rocky-PowerTools.repo        Rocky-Sources.repo
Rocky-BaseOS.repo     Rocky-Extras.repo            Rocky-NFV.repo    Rocky-ResilientStorage.repo
Rocky-Debuginfo.repo  Rocky-HighAvailability.repo  Rocky-Plus.repo   Rocky-RT.repo
```

```bash
yum -y update
yum -y install vim socat wget bash-completion net-tools zip bzip2 bind-utils
```

## 配置主机名

```bash
 hostnamectl set-hostname kube-controller01
 hostnamectl set-hostname kube-node01
 hostnamectl set-hostname kube-node02
```


##  安装 git
- [tar 安装指定 git 版本请看这里](https://blog.csdn.net/xixihahalelehehe/article/details/125107061)
```bash
yum -y install git
```
克隆部署项目
```bash
git clone https://github.com/Ghostwritten/k8s-install.git
```

##  安装 docker
- [How To Install and Use Docker on Rocky Linux 8](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-rocky-linux-8)
```bash
sudo dnf check-update
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf -y install docker-ce docker-ce-cli containerd.io
sudo systemctl start docker
sudo systemctl status docker
sudo systemctl enable docker

```

##  安装 ansible
```bash
sudo dnf update
sudo dnf -y install epel-release
sudo dnf -y install ansible
```


配置  `inventory.ini`

```bash
$ cd k8s-install/kubespray-2.21.0/rocky9.1-calico-cluster
$ vim inventory/cluster-local/inventory.ini
# ## Configure 'ip' variable to bind kubernetes services on a
# ## different ip than the default iface
# ## We should set etcd_member_name for etcd cluster. The node that is not a etcd member do not need to set the value, or can set the empty string value.
[all]
kube-controller01 ansible_host=192.168.50.21
kube-node01 ansible_host=192.168.50.41
kube-node02 ansible_host=192.168.50.42


# ## configure a bastion host if your nodes are not directly reachable
[bastion]
bastion01 ansible_host=192.168.50.20 ansible_user=root

[kube_control_plane]
kube-controller01

[etcd]
kube-controller01

[kube_node]
kube-node01
kube-node02

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr
```
测试节点连通性

```bash
ansible -i inventory/cluster-local/inventory.ini all -m ping
```


## 配置内核参数

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

##  安装 k8s
```bash
$ ./install-cluster.sh
 欢迎使用 Kubespray 工具部署 k8s！

容器 kubespray-v2.21.0 创建成功！

 现在你可以开始安装 k8s:

   1. docker attach kubespray-v2.21.0
   2. pip3 install jmespath
   3. ansible-playbook -i /inventory/inventory.ini --private-key /root/.ssh/id_rsa cluster.yml
```

> 如果不执行命令：`python3.8 install jmespath` 报错1：
>  - [Ansible: "You need to install 'jmespath' prior to running json_query filter", but it is
 installed](https://serverfault.com/questions/1114638/ansible-you-need-to-install-jmespath-prior-to-running-json-query-filter-bu)
 > - [https://github.com/kubernetes-sigs/kubespray/issues/9826](https://github.com/kubernetes-sigs/kubespray/issues/9826)，该 bug 计划在 2.21.1版本修复


##   定制安装 kubernetes 1.23.16 & docker 容器引擎

```bash
$ cat inventory/cluster-local/group_vars/k8s_cluster/k8s-cluster.yml
...
kube_version: v1.23.16
container_manager: docker

```



- kubespray 2.21.1 
- kubernetes 1.23.16 
-  docker
 

```bash
[root@kube-controller01 ~]# docker images
REPOSITORY                                                  TAG        IMAGE ID       CREATED         SIZE
registry.k8s.io/kube-apiserver                              v1.23.16   2d5c6bb50aa7   2 months ago    130MB
registry.k8s.io/kube-controller-manager                     v1.23.16   99fbab52b1e5   2 months ago    120MB
registry.k8s.io/kube-proxy                                  v1.23.16   28204678d22a   2 months ago    111MB
registry.k8s.io/kube-scheduler                              v1.23.16   73e02f61aa83   2 months ago    51.9MB
registry.k8s.io/metrics-server/metrics-server               v0.6.2     25561daa6660   4 months ago    68.9MB
quay.io/calico/kube-controllers                             v3.24.5    38b76de417d5   5 months ago    71.4MB
quay.io/calico/cni                                          v3.24.5    628dd7088041   5 months ago    198MB
quay.io/calico/pod2daemon-flexvol                           v3.24.5    2f8f95ac9ac4   5 months ago    14.5MB
quay.io/calico/node                                         v3.24.5    54637cb36d4a   5 months ago    226MB
registry.k8s.io/pause                                       3.8        4873874c08ef   10 months ago   711kB
registry.k8s.io/coredns/coredns                             v1.9.3     5185b96f0bec   10 months ago   48.8MB
quay.io/metallb/speaker                                     v0.12.1    579ce8a43ea8   14 months ago   70MB
registry.k8s.io/coredns/coredns                             v1.8.6     a4ca41631cc7   18 months ago   46.8MB
registry.k8s.io/dns/k8s-dns-node-cache                      1.21.1     5bae806f8f12   19 months ago   104MB
registry.k8s.io/pause                                       3.6        6270bb605e12   19 months ago   683kB
registry.k8s.io/cpa/cluster-proportional-autoscaler-amd64   1.8.5      1e7da779960f   20 months ago   40.7MB


[root@kube-node01 ~]# docker images
REPOSITORY                                TAG             IMAGE ID       CREATED         SIZE
registry.k8s.io/kube-apiserver            v1.23.16        2d5c6bb50aa7   2 months ago    130MB
registry.k8s.io/kube-scheduler            v1.23.16        73e02f61aa83   2 months ago    51.9MB
registry.k8s.io/kube-controller-manager   v1.23.16        99fbab52b1e5   2 months ago    120MB
registry.k8s.io/kube-proxy                v1.23.16        28204678d22a   2 months ago    111MB
nginx                                     1.23.2-alpine   19dd4d73108a   5 months ago    23.5MB
quay.io/calico/kube-controllers           v3.24.5         38b76de417d5   5 months ago    71.4MB
quay.io/calico/cni                        v3.24.5         628dd7088041   5 months ago    198MB
quay.io/calico/pod2daemon-flexvol         v3.24.5         2f8f95ac9ac4   5 months ago    14.5MB
quay.io/calico/node                       v3.24.5         54637cb36d4a   5 months ago    226MB
registry.k8s.io/pause                     3.8             4873874c08ef   10 months ago   711kB
registry.k8s.io/coredns/coredns           v1.9.3          5185b96f0bec   10 months ago   48.8MB
registry.k8s.io/coredns/coredns           v1.8.6          a4ca41631cc7   18 months ago   46.8MB
registry.k8s.io/dns/k8s-dns-node-cache    1.21.1          5bae806f8f12   19 months ago   104MB
```


```bash
$ kubectl  get pod -n kube-system coredns-54bf8d85c7-6tz7x -oyaml
Cannot enforce NoNewPrivs: illegal version string "v1"
```


```bash
  kube-api-access-qf65p:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              kubernetes.io/os=linux
Tolerations:                 node-role.kubernetes.io/control-plane:NoSchedule
                             node-role.kubernetes.io/master:NoSchedule
                             node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason       Age                 From               Message
  ----     ------       ----                ----               -------
  Normal   Scheduled    15m                 default-scheduler  Successfully assigned kube-system/coredns-54bf8d85c7-6tz7x to kube-node01
  Warning  FailedMount  15m (x6 over 15m)   kubelet            MountVolume.SetUp failed for volume "config-volume" : object "kube-system"/"coredns" not registered
  Warning  FailedMount  63s (x15 over 15m)  kubelet            MountVolume.SetUp failed for volume "config-volume" : object "kube-system"/"coredns" not registered
```

-- Cannot enforce NoNewPrivs: illegal version string "v1" [https://github.com/Mirantis/cri-dockerd/issues/167](https://github.com/Mirantis/cri-dockerd/issues/167)


## 更新 kubernetes_version：1.24.10 部署成功

```bash
[root@kube-controller01 ~]# kubectl get node
NAME                STATUS   ROLES                  AGE     VERSION
kube-controller01   Ready    control-plane,master   3h36m   v1.24.10
kube-node01         Ready    <none>                 3h35m   v1.24.10
kube-node02         Ready    <none>                 3h35m   v1.24.10
[root@kube-controller01 ~]# kubectl get pod -A
NAMESPACE     NAME                                        READY   STATUS    RESTARTS      AGE
kube-system   calico-kube-controllers-6fc44869bc-94hwv    1/1     Running   0             3h35m
kube-system   calico-node-7stvj                           1/1     Running   0             3h35m
kube-system   calico-node-dzvls                           1/1     Running   0             3h35m
kube-system   calico-node-wgn7g                           1/1     Running   0             3h35m
kube-system   coredns-57f7f7b97d-8ts8q                    1/1     Running   0             12m
kube-system   coredns-57f7f7b97d-sbd7v                    1/1     Running   0             13m
kube-system   dns-autoscaler-78676459f6-rgdwr             1/1     Running   0             3h35m
kube-system   kube-apiserver-kube-controller01            1/1     Running   1 (36m ago)   36m
kube-system   kube-controller-manager-kube-controller01   1/1     Running   1 (36m ago)   36m
kube-system   kube-proxy-g9z9m                            1/1     Running   0             36m
kube-system   kube-proxy-sctdr                            1/1     Running   0             36m
kube-system   kube-proxy-tp5kr                            1/1     Running   0             36m
kube-system   kube-scheduler-kube-controller01            1/1     Running   1 (36m ago)   36m
kube-system   metrics-server-cc8bc6d9b-tjjn9              1/1     Running   0             12m
kube-system   nginx-proxy-kube-node01                     1/1     Running   0             36m
kube-system   nginx-proxy-kube-node02                     1/1     Running   0             36m
kube-system   nodelocaldns-gst6d                          1/1     Running   0             3h35m
kube-system   nodelocaldns-hpg6f                          1/1     Running   0             3h35m
kube-system   nodelocaldns-tx5z4                          1/1     Running   0             3h35m
```

## 新增节点



主机：192.168.48.92
### 配置主机名

```bash
hostnamectl set-hostname kube-node03
```


### 配置互信
部署节点（bastion01）执行：

```bash
ssh-copy-id root@192.168.48.92
```
###  更新 inventory

```bash
$ cd /root/k8s-install/kubespray-2.21.0/rocky9.1-calico-cluster
$ cat inventory/cluster-local/inventory.ini
# ## Configure 'ip' variable to bind kubernetes services on a
# ## different ip than the default iface
# ## We should set etcd_member_name for etcd cluster. The node that is not a etcd member do not need to set the value, or can set the empty string value.
[all]
kube-controller01 ansible_host=192.168.50.21
kube-node01 ansible_host=192.168.50.41
kube-node02 ansible_host=192.168.50.42
kube-node03 ansible_host=192.168.48.92


# ## configure a bastion host if your nodes are not directly reachable
[bastion]
bastion01 ansible_host=192.168.50.20 ansible_user=root

[kube_control_plane]
kube-controller01

[etcd]
kube-controller01

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

在容器内执行：

```bash
docker exec -ti kubespray-v2.21.0 bash      
ansible-playbook -i /inventory/inventory.ini --private-key /root/.ssh/id_rsa scale.yml -b -v 
```


###   报错：kubernetes/kubeadm : Join to cluster

```bash
TASK [kubernetes/kubeadm : Join to cluster] ****************************************************************************************************************************************
skipping: [kube-node01] => {"changed": false, "skip_reason": "Conditional result was False"}
skipping: [kube-node02] => {"changed": false, "skip_reason": "Conditional result was False"}
fatal: [kube-node03]: FAILED! => {"changed": false, "cmd": ["timeout", "-k", "120s", "120s", "/usr/local/bin/kubeadm", "join", "--config", "/etc/kubernetes/kubeadm-client.conf", "--ignore-preflight-errors=DirAvailable--etc-kubernetes-manifests", "--skip-phases="], "delta": "0:01:00.082674", "end": "2023-04-17 20:38:11.054440", "msg": "non-zero return code", "rc": 1, "start": "2023-04-17 20:37:10.971766", "stderr": "\t[WARNING FileExisting-tc]: tc not found in system path\nerror execution phase preflight: couldn't validate the identity of the API Server: Get \"https://192.168.50.21:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s\": x509: certificate has expired or is not yet valid: current time 2023-04-17T20:38:07+08:00 is before 2023-04-17T15:56:59Z\nTo see the stack trace of this error execute with --v=5 or higher", "stderr_lines": ["\t[WARNING FileExisting-tc]: tc not found in system path", "error execution phase preflight: couldn't validate the identity of the API Server: Get \"https://192.168.50.21:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s\": x509: certificate has expired or is not yet valid: current time 2023-04-17T20:38:07+08:00 is before 2023-04-17T15:56:59Z", "To see the stack trace of this error execute with --v=5 or higher"], "stdout": "[preflight] Running pre-flight checks", "stdout_lines": ["[preflight] Running pre-flight checks"]}

TASK [kubernetes/kubeadm : Join to cluster with ignores] ***************************************************************************************************************************
fatal: [kube-node03]: FAILED! => {"changed": false, "cmd": ["timeout", "-k", "120s", "120s", "/usr/local/bin/kubeadm", "join", "--config", "/etc/kubernetes/kubeadm-client.conf", "--ignore-preflight-errors=all", "--skip-phases="], "delta": "0:01:00.082909", "end": "2023-04-17 20:39:11.322402", "msg": "non-zero return code", "rc": 1, "start": "2023-04-17 20:38:11.239493", "stderr": "\t[WARNING FileExisting-tc]: tc not found in system path\nerror execution phase preflight: couldn't validate the identity of the API Server: Get \"https://192.168.50.21:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s\": x509: certificate has expired or is not yet valid: current time 2023-04-17T20:39:07+08:00 is before 2023-04-17T15:56:59Z\nTo see the stack trace of this error execute with --v=5 or higher", "stderr_lines": ["\t[WARNING FileExisting-tc]: tc not found in system path", "error execution phase preflight: couldn't validate the identity of the API Server: Get \"https://192.168.50.21:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s\": x509: certificate has expired or is not yet valid: current time 2023-04-17T20:39:07+08:00 is before 2023-04-17T15:56:59Z", "To see the stack trace of this error execute with --v=5 or higher"], "stdout": "[preflight] Running pre-flight checks", "stdout_lines": ["[preflight] Running pre-flight checks"]}

TASK [kubernetes/kubeadm : Display kubeadm join stderr if any] *********************************************************************************************************************
skipping: [kube-node01] => {}
```

##  删除节点
 - [kubespray增加和删除worker节点.md](https://github.com/wenwenxiong/book/blob/master/k8s/kubespray/kubespray%E5%A2%9E%E5%8A%A0%E5%92%8C%E5%88%A0%E9%99%A4worker%E8%8A%82%E7%82%B9.md)
```bash
ansible-playbook -i /inventory/inventory.ini --private-key /root/.ssh/id_rsa -e node=kube-node03 remove-node.yml -b -v
```

- [kubespray v2.21.0 部署 kubernetes v1.24.0 集群 【1】](https://ghostwritten.blog.csdn.net/article/details/129952069)
- [在 Rocky linux 8.7 使用 Kubespray v2.21.0 离线部署 kubernetes v1.24.0 集群](https://blog.csdn.net/xixihahalelehehe/article/details/130056246)
