
![在这里插入图片描述](https://img-blog.csdnimg.cn/0a282762a9164ddbb67025aebd7f6d92.png)




## 1. 还原虚拟机

之前我们[通过 Kubespray-offline v2.21.0-1 下载 Kubespray v2.22.1 离线部署 kubernetes v1.25.6](https://ghostwritten.blog.csdn.net/article/details/132074869)，在部署之前我们打了快照——系统初始化，并且我们部署完成后，对 k8s 集群的状态再次打了快照。

现在我们计划恢复到系统初始化的状态，通过 rke2 进行部署 kubernetes 集群。

![在这里插入图片描述](https://img-blog.csdnimg.cn/d1c0a3b6bc9c4e7db3f872c358f40c90.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/6d2383751b7d4ebd875ee16982f57837.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/93b12ad4d1184cbe926632ae20e6fc3a.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/4c18c5d41459475c8aeae80c83a3ace3.png)
## 2. 背景

[k8s官方部署安装集群的是使用kubeadm方式](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)，但是该方式比较复杂繁琐，所以产生了一些新的部署安装集群方式，比如 [k3s](https://k3s.io/) 和 [rke2](https://docs.rke2.io/) 等新方式

k3s有着非常庞大的社区支持，部署安装也非常简单，设计为轻量级的k8s，可以很好的运行在物联网设备或者边缘计算设备上面

据rke2官方文档描述说该部署是继承了k3s的可用性、易操作性和部署模式，继承了与上游 Kubernetes 的紧密一致性，在一些地方，K3s 与上游的 Kubernetes 有分歧(k3s魔改了一些k8s组件)，以便为边缘部署进行优化，rke2同时也预设了安全配置，符合各项安全测试规范，但是部署方式上比k3s更复杂一些

整体来看选择k3s和rke2都是可以用于生产环境的选择，如果更注重安全性，可以选择rke2。

## 3. 介绍
RKE2（Rancher Kubernetes Engine 2）是一个轻量级、易于安装和管理的 Kubernetes 发行版，由 [Rancher Labs](https://www.rancher.com/) 公司开发和维护。RKE2 构建在 CNCF 基金会的 Kubernetes 项目之上，但是相比于原生 Kubernetes 更加轻量、易于升级和维护。

RKE2 的设计目标是提供一个简单的 Kubernetes 发行版，可以在各种环境中轻松部署和管理。RKE2 的安装程序包含了所有的 Kubernetes 组件，包括 kubelet、kube-proxy、kube-apiserver、kube-controller-manager 和 kube-scheduler 等。另外，RKE2 也包含了一些额外的组件，如 etcd、coredns、calico 网络插件等，这些组件可以帮助用户快速搭建一个完整的 Kubernetes 集群。

RKE2 的另一个特点是使用了 [systemd](https://systemd.io/) 作为容器运行时，这使得 RKE2 的容器管理和资源管理更加高效。RKE2 还提供了一些额外的功能，如自动 TLS 证书管理、安全审计、灾难恢复等功能，可以帮助用户更好地管理 Kubernetes 集群。

总的来说，RKE2 是一款非常适合初学者和中小型企业使用的 Kubernetes 发行版，具有轻量、易用、高效和安全等特点。


## 4. 预备条件
系统： rocky linux 8.8

192.168.23.30-rocky-8.8-bastion01
bastion01 (这里下载介质与部署节点为同一节点，如果非同一节点，需要介质下载搬运)
- 192.168.23.30
- cpu：4
- 内存：8
- 磁盘：60G（系统盘）（Thin Provision）

192.168.23.31-rocky-8.8-kube-master01
kube-master01(control plane)

- 192.168.23.31
- cpu：4
- 内存：8
- 磁盘：60G（系统盘）（Thin Provision）

192.168.23.32-rocky-8.8-kube-prom01
kube-prom01(worker node)
- 192.168.23.32
- cpu：2
- 内存：4
- 磁盘：60G（系统盘）（Thin Provision）

192.168.23.33-rocky-8.8-kube-node01
kube-node01(worker node)
- 192.168.23.33
- cpu：4
- 内存：8
- 磁盘：60G（系统盘）（Thin Provision）、100G（挂载）（Thin Provision）


192.168.23.34-rocky-8.8-kube-node02
kube-node02(worker node)
- 192.168.23.34
- cpu：4
- 内存：8
- 磁盘：60G（系统盘）（Thin Provision）、100G（挂载）（Thin Provision）

192.168.23.35-rocky-8.8-kube-node03
kube-node03(worker node)
- 192.168.23.35
- cpu：4
- 内存：8
- 磁盘：60G（系统盘）（Thin Provision）、100G（挂载）（Thin Provision）







### 5.1 配置网卡
(每台操作)
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

### 5. 配置主机名
(每台操作)
```bash
hostnamectl set-hostname bastion01
hostnamectl set-hostname kube-master01
hostnamectl set-hostname kube-prom01
hostnamectl set-hostname kube-node01
hostnamectl set-hostname kube-node02
hostnamectl set-hostname kube-node03
```

## 6. 配置互信
（bastion01 或 kube-master01）
```bash
ssh-keygen
for i in {30..35};do ssh-copy-id root@192.168.23.$i;done
```

## 7. 安装 ansible
（bastion01 或 kube-master01）

```bash
yum -y install epel-release
yum -y install ansible
```

配置ansile

```bash
$ cat /etc/ansible/hosts 
[all]
kube-master01 ansible_host=192.168.23.31
kube-prom01 ansible_host=192.168.23.32
kube-node01 ansible_host=192.168.23.33
kube-node02 ansible_host=192.168.23.34
kube-node03 ansible_host=192.168.23.35
```
测试

```bash
ansible all -m ping
```

## 8. 系统初始化
关闭防火墙、swap、selinux

```bash
ansible all -i hosts -s -m systemd -a "name=firewalld state=stopped enabled=no"
ansible all -m lineinfile -a "path=/etc/selinux/config regexp='^SELINUX=' line='SELINUX=disabled'" -b
ansible all -m shell -a "getenforce 0"
ansible all -m shell -a "sed -i '/.*swap.*/s/^/#/' /etc/fstab" -b
ansible all -m shell -a " swapoff -a && sysctl -w vm.swappiness=0"
ansible all -m lineinfile -a "path=/etc/security/limits.conf line='* soft nofile 655360\n* hard nofile 131072\n* soft nproc 655350\n* hard nproc 655350\n* soft memlock unlimited\n* hard memlock unlimited'" -b
ansible all -m file -a "path=/etc/modules-load.d state=directory" -b
ansible all -m file -a "path=/etc/modules-load.d/ipvs.conf state=touch mode=0644"
ansible all -m lineinfile -a "path=/etc/rc.local line='modprobe br_netfilter\nmodprobe ip_conntrack'" -b
ansible all  -m systemd -a "name=systemd-modules-load.service state=restarted"
ansible all -m shell -a " modprobe bridge && modprobe br_netfilter && modprobe ip_conntrack"
ansible all -m file -a "path=/etc/sysctl.d/k8s.conf state=touch mode=0644"
ansible all -m blockinfile -a "path=/etc/sysctl.d/k8s.conf block='net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
vm.overcommit_memory = 1
vm.panic_on_oom = 0
fs.inotify.max_user_watches = 89100
fs.file-max = 52706963
fs.nr_open = 52706963
net.netfilter.nf_conntrack_max = 2310720
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_keepalive_intvl = 15
net.ipv4.tcp_max_tw_buckets = 36000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_max_orphans = 327680
net.ipv4.tcp_orphan_retries = 3
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.tcp_timestamps = 0
net.core.somaxconn = 16384
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
net.ipv6.conf.all.forwarding = 1'"

ansible all -m shell -a " sysctl -p /etc/sysctl.d/k8s.conf"

ansible all -m file -a "path=/etc/NetworkManager/conf.d/rke2-canal.conf state=touch mode=0644"
ansible all -m blockinfile -a "path=/etc/NetworkManager/conf.d/rke2-canal.conf block='[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:flannel*'"
ansible all -m shell -a "systemctl reload NetworkManager && systemctl daemon-reload && systemctl restart NetworkManager  "

```

## 9. kube-master01 部署
(kube-master01 192.168.23.31)
RKE2 Server 使用嵌入式 etcd 运行。因此你不需要设置外部数据存储就可以在 HA 模式下运行。

在第一个节点上，使用你的预共享密文作为 Token 来设置配置文件。Token 参数可以在启动时设置。

如果你不指定预共享密文，RKE2 会生成一个预共享密文并将它放在 /var/lib/rancher/rke2/server/node-token 中。

为了避免固定注册地址的证书错误，请在启动 Server 时设置 tls-san 参数。这个选项在 Server 的 TLS 证书中增加一个额外的主机名或 IP 作为 Subject Alternative Name。如果你想通过 IP 和主机名访问，你可以将它指定为一个列表。



### 9.1 定制配置文件（可选）


首先，创建用于存放 RKE2 配置文件的目录：

```bash
mkdir -p /etc/rancher/rke2/
```
然后，参见以下示例在 `/etc/rancher/rke2/config.yaml` 中创建 RKE2 配置文件：

```bash
$ vim /etc/rancher/rke2/config.yaml
token: demo-server
node-name: demo-server-node
tls-san: 192.168.23.31
system-default-registry: "registry.cn-hangzhou.aliyuncs.com"
```
- token参数表示自定义一个token标识
- node-name表示配置节点名，该名称是全局唯一的，用于dns路由
- tls-san表示TLS证书上添加额外的主机名或IPv4/IPv6地址作为备用名称，此处填写本机IP，该参数是为了避免固定注册地址的证书错误
- system-default-registry表示使用国内镜像


### 9.2 部署
```bash
curl -sfL https://get.rke2.io |  sh -
```


输出：
```bash
[root@kube-master01 ~]# curl -sfL https://get.rke2.io |  sh -
[INFO]  finding release for channel stable
[INFO]  using 1.25 series from channel stable
Rancher RKE2 Common (stable)                                                                                                                                        3.3 kB/s | 2.9 kB     00:00    
Rancher RKE2 1.25 (stable)                                                                                                                                          3.3 kB/s | 5.9 kB     00:01    
Dependencies resolved.
====================================================================================================================================================================================================
 Package                                           Architecture                Version                                                        Repository                                       Size
====================================================================================================================================================================================================
Installing:
 rke2-server                                       x86_64                      1.25.12~rke2r1-0.el8                                           rancher-rke2-1.25-stable                        8.8 k
Installing dependencies:
 checkpolicy                                       x86_64                      2.9-1.el8                                                      baseos                                          345 k
 container-selinux                                 noarch                      2:2.205.0-2.module+el8.8.0+1265+fa25dd7a                       appstream                                        63 k
 policycoreutils-python-utils                      noarch                      2.9-24.el8                                                     baseos                                          253 k
 python3-audit                                     x86_64                      3.0.7-4.el8                                                    baseos                                           86 k
 python3-libsemanage                               x86_64                      2.9-9.el8_6                                                    baseos                                          127 k
 python3-policycoreutils                           noarch                      2.9-24.el8                                                     baseos                                          2.2 M
 python3-setools                                   x86_64                      4.3.0-3.el8                                                    baseos                                          623 k
 rke2-common                                       x86_64                      1.25.12~rke2r1-0.el8                                           rancher-rke2-1.25-stable                         19 M
 rke2-selinux                                      noarch                      0.14-1.el8                                                     rancher-rke2-common-stable                       21 k
Enabling module streams:
 container-tools                                                               rhel8                                                                                                               

Transaction Summary
====================================================================================================================================================================================================
Install  10 Packages

Total download size: 23 M
Installed size: 92 M
Downloading Packages:
(1/10): container-selinux-2.205.0-2.module+el8.8.0+1265+fa25dd7a.noarch.rpm                                                                                          53 kB/s |  63 kB     00:01    
(2/10): checkpolicy-2.9-1.el8.x86_64.rpm                                                                                                                            149 kB/s | 345 kB     00:02    
(3/10): policycoreutils-python-utils-2.9-24.el8.noarch.rpm                                                                                                          105 kB/s | 253 kB     00:02    
(4/10): python3-libsemanage-2.9-9.el8_6.x86_64.rpm                                                                                                                  442 kB/s | 127 kB     00:00    
(5/10): python3-setools-4.3.0-3.el8.x86_64.rpm                                                                                                                      1.2 MB/s | 623 kB     00:00    
(6/10): python3-audit-3.0.7-4.el8.x86_64.rpm                                                                                                                         44 kB/s |  86 kB     00:01    
(7/10): python3-policycoreutils-2.9-24.el8.noarch.rpm                                                                                                               2.0 MB/s | 2.2 MB     00:01    
(8/10): rke2-selinux-0.14-1.el8.noarch.rpm                                                                                                                           19 kB/s |  21 kB     00:01    
(9/10): rke2-server-1.25.12~rke2r1-0.el8.x86_64.rpm                                                                                                                 9.7 kB/s | 8.8 kB     00:00    
(10/10): rke2-common-1.25.12~rke2r1-0.el8.x86_64.rpm                                                                                                                4.7 MB/s |  19 MB     00:04    
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                               2.5 MB/s |  23 MB     00:09     
Rancher RKE2 Common (stable)                                                                                                                                        2.7 kB/s | 2.4 kB     00:00    
Importing GPG key 0xE257814A:
 Userid     : "Rancher (CI) <ci@rancher.com>"
 Fingerprint: C8CF F216 4551 26E9 B9C9 18BE 925E A29A E257 814A
 From       : https://rpm.rancher.io/public.key
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                                            1/1 
  Installing       : python3-setools-4.3.0-3.el8.x86_64                                                                                                                                        1/10 
  Installing       : python3-libsemanage-2.9-9.el8_6.x86_64                                                                                                                                    2/10 
  Installing       : python3-audit-3.0.7-4.el8.x86_64                                                                                                                                          3/10 
  Installing       : checkpolicy-2.9-1.el8.x86_64                                                                                                                                              4/10 
  Installing       : python3-policycoreutils-2.9-24.el8.noarch                                                                                                                                 5/10 
  Installing       : policycoreutils-python-utils-2.9-24.el8.noarch                                                                                                                            6/10 
  Running scriptlet: container-selinux-2:2.205.0-2.module+el8.8.0+1265+fa25dd7a.noarch                                                                                                         7/10 
  Installing       : container-selinux-2:2.205.0-2.module+el8.8.0+1265+fa25dd7a.noarch                                                                                                         7/10 
  Running scriptlet: container-selinux-2:2.205.0-2.module+el8.8.0+1265+fa25dd7a.noarch                                                                                                         7/10 
  Running scriptlet: rke2-selinux-0.14-1.el8.noarch                                                                                                                                            8/10 
  Installing       : rke2-selinux-0.14-1.el8.noarch                                                                                                                                            8/10 
  Running scriptlet: rke2-selinux-0.14-1.el8.noarch                                                                                                                                            8/10 
  Installing       : rke2-common-1.25.12~rke2r1-0.el8.x86_64                                                                                                                                   9/10 
  Installing       : rke2-server-1.25.12~rke2r1-0.el8.x86_64                                                                                                                                  10/10 
  Running scriptlet: rke2-server-1.25.12~rke2r1-0.el8.x86_64                                                                                                                                  10/10 
  Running scriptlet: container-selinux-2:2.205.0-2.module+el8.8.0+1265+fa25dd7a.noarch                                                                                                        10/10 
  Running scriptlet: rke2-selinux-0.14-1.el8.noarch                                                                                                                                           10/10 
  Running scriptlet: rke2-server-1.25.12~rke2r1-0.el8.x86_64                                                                                                                                  10/10 
  Verifying        : container-selinux-2:2.205.0-2.module+el8.8.0+1265+fa25dd7a.noarch                                                                                                         1/10 
  Verifying        : checkpolicy-2.9-1.el8.x86_64                                                                                                                                              2/10 
  Verifying        : policycoreutils-python-utils-2.9-24.el8.noarch                                                                                                                            3/10 
  Verifying        : python3-audit-3.0.7-4.el8.x86_64                                                                                                                                          4/10 
  Verifying        : python3-libsemanage-2.9-9.el8_6.x86_64                                                                                                                                    5/10 
  Verifying        : python3-policycoreutils-2.9-24.el8.noarch                                                                                                                                 6/10 
  Verifying        : python3-setools-4.3.0-3.el8.x86_64                                                                                                                                        7/10 
  Verifying        : rke2-selinux-0.14-1.el8.noarch                                                                                                                                            8/10 
  Verifying        : rke2-common-1.25.12~rke2r1-0.el8.x86_64                                                                                                                                   9/10 
  Verifying        : rke2-server-1.25.12~rke2r1-0.el8.x86_64                                                                                                                                  10/10 

Installed:
  checkpolicy-2.9-1.el8.x86_64           container-selinux-2:2.205.0-2.module+el8.8.0+1265+fa25dd7a.noarch policycoreutils-python-utils-2.9-24.el8.noarch python3-audit-3.0.7-4.el8.x86_64       
  python3-libsemanage-2.9-9.el8_6.x86_64 python3-policycoreutils-2.9-24.el8.noarch                         python3-setools-4.3.0-3.el8.x86_64             rke2-common-1.25.12~rke2r1-0.el8.x86_64
  rke2-selinux-0.14-1.el8.noarch         rke2-server-1.25.12~rke2r1-0.el8.x86_64                          

Complete!

```

启动服务

```bash
 systemctl start rke2-server.service && systemctl start rke2-server.service
```
> 注意： 启动要等一段时间


```bash
systemctl status rke2-server
● rke2-server.service - Rancher Kubernetes Engine v2 (server)
   Loaded: loaded (/usr/lib/systemd/system/rke2-server.service; disabled; vendor preset: disabled)
   Active: active (running) since Sun 2023-08-06 22:20:53 CST; 8s ago
     Docs: https://github.com/rancher/rke2#readme
  Process: 2294800 ExecStopPost=/bin/sh -c systemd-cgls /system.slice/rke2-server.service | grep -Eo '[0-9]+ (containerd|kubelet)' | awk '{print $1}' | xargs -r kill (code=exited, status=0/SUCCES>
  Process: 2295345 ExecStartPre=/sbin/modprobe overlay (code=exited, status=0/SUCCESS)
  Process: 2295343 ExecStartPre=/sbin/modprobe br_netfilter (code=exited, status=0/SUCCESS)
  Process: 2295340 ExecStartPre=/bin/sh -xc ! /usr/bin/systemctl is-enabled --quiet nm-cloud-setup.service (code=exited, status=0/SUCCESS)
 Main PID: 2295347 (rke2)
    Tasks: 192
   Memory: 3.8G
   CGroup: /system.slice/rke2-server.service
           ├─2295347 /usr/bin/rke2 server
           ├─2295390 containerd -c /var/lib/rancher/rke2/agent/etc/containerd/config.toml -a /run/k3s/containerd/containerd.sock --state /run/k3s/containerd --root /var/lib/rancher/rke2/agent/con>
           ├─2295767 kubelet --volume-plugin-dir=/var/lib/kubelet/volumeplugins --file-check-frequency=5s --sync-frequency=30s --address=0.0.0.0 --alsologtostderr=false --anonymous-auth=false --a>
           ├─2295856 /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin/containerd-shim-runc-v2 -namespace k8s.io -id d1de68afebde1fa1355a21fa139799be94d73e7d974aa41de430b072fe079874 -ad>
           ├─2295884 /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin/containerd-shim-runc-v2 -namespace k8s.io -id 26352bf8e8f577a80a576379d8d64dee83b8dd7d325073f9661798c7980c34c1 -ad>
           ├─2295909 /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin/containerd-shim-runc-v2 -namespace k8s.io -id 6b6e5fea8cd59cc1470c16a9c8a1f52207b77bef8abe70a30c838b0fc046f3e6 -ad>
           ├─2295950 /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin/containerd-shim-runc-v2 -namespace k8s.io -id e4219414e1011698a93785adf8e97e03b2a5bf11925db87568854a023d9056a3 -ad>
           ├─2295984 /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin/containerd-shim-runc-v2 -namespace k8s.io -id 5f9f1d28ca2ca3ee923be96956523721af912699c00af02f463cdf784ace749c -ad>
           ├─2296020 /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin/containerd-shim-runc-v2 -namespace k8s.io -id d21a2014601a363d2579e597b3343b598d76c0304899a4e627992a4e76302977 -ad>
           ├─2296402 /opt/cni/bin/calico
           ├─2296468 /opt/cni/bin/calico
           ├─2296471 /opt/cni/bin/calico
           ├─2296562 /opt/cni/bin/calico
           ├─2296659 /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin/containerd-shim-runc-v2 -namespace k8s.io -id a90318f401ff0c2dfdc8ee6a915b2b034fb82d6f9aa94829b6efc298da655b56 -ad>
           ├─2296832 /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin/containerd-shim-runc-v2 -namespace k8s.io -id 26d381b3c90925d674c6ea33a0079774b9243b1d9045336e1b7db9470228205e -ad>
           ├─2296898 /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin/containerd-shim-runc-v2 -namespace k8s.io -id 63b24e530e0442295034763b87b818c1af0c74669034b11b7f4c9f3e437ee559 -ad>
           └─2297443 runc --root /run/containerd/runc/k8s.io --log /run/k3s/containerd/io.containerd.runtime.v2.task/k8s.io/dfebbb6522b60bf3d25d6585f17e70037fdecd16fc08beb8e41280fbb8781a54/log.js>

Aug 06 22:21:00 kube-master01 rke2[2295347]: E0806 22:21:00.553292 2295347 memcache.go:104] couldn't get resource list for metrics.k8s.io/v1beta1: the server is currently unable to handle the req>
Aug 06 22:21:00 kube-master01 rke2[2295347]: I0806 22:21:00.562390 2295347 event.go:294] "Event occurred" object="kube-system/rke2-snapshot-validation-webhook" fieldPath="" kind="HelmChart" apiVe>
Aug 06 22:21:00 kube-master01 rke2[2295347]: time="2023-08-06T22:21:00+08:00" level=info msg="Starting batch/v1, Kind=Job controller"
Aug 06 22:21:00 kube-master01 rke2[2295347]: time="2023-08-06T22:21:00+08:00" level=info msg="Starting /v1, Kind=ConfigMap controller"
Aug 06 22:21:00 kube-master01 rke2[2295347]: time="2023-08-06T22:21:00+08:00" level=info msg="Starting /v1, Kind=ServiceAccount controller"
Aug 06 22:21:00 kube-master01 rke2[2295347]: I0806 22:21:00.771077 2295347 event.go:294] "Event occurred" object="kube-system/rke2-snapshot-controller" fieldPath="" kind="Addon" apiVersion="k3s.c>
Aug 06 22:21:00 kube-master01 rke2[2295347]: I0806 22:21:00.772397 2295347 event.go:294] "Event occurred" object="kube-system/rke2-snapshot-controller" fieldPath="" kind="HelmChart" apiVersion="h>
Aug 06 22:21:00 kube-master01 rke2[2295347]: I0806 22:21:00.792723 2295347 event.go:294] "Event occurred" object="kube-system/rke2-snapshot-validation-webhook" fieldPath="" kind="Addon" apiVersio>
Aug 06 22:21:01 kube-master01 rke2[2295347]: I0806 22:21:01.196094 2295347 event.go:294] "Event occurred" object="kube-system/rke2-snapshot-validation-webhook" fieldPath="" kind="Addon" apiVersio>
Aug 06 22:21:01 kube-master01 rke2[2295347]: I0806 22:21:01.196926 2295347 event.go:294] "Event occurred" object="kube-system/rke2-snapshot-validation-webhook" fieldPath="" kind="HelmChart" apiVe
```
服务启动后会在`/etc/rancher/rke2/rke2.yaml`位置生成一个文件包含了集群的信息


### 9.3 命令配置
查看安装的二进制执行文件，rke2默认是安装到`/var/lib/rancher/rke2/bin/`路径下面，但是该路径是不被`$PATH`所包含的。

```bash
$ ll /var/lib/rancher/rke2/bin/
total 322512
-rwxr-xr-x 1 root root  59563056 Aug  4 22:15 containerd
-rwxr-xr-x 1 root root   8527032 Aug  4 22:15 containerd-shim
-rwxr-xr-x 1 root root  10217176 Aug  4 22:16 containerd-shim-runc-v1
-rwxr-xr-x 1 root root  14011856 Aug  4 22:17 containerd-shim-runc-v2
-rwxr-xr-x 1 root root  38483024 Aug  4 22:18 crictl
-rwxr-xr-x 1 root root  20968336 Aug  4 22:19 ctr
-rwxr-xr-x 1 root root  49994728 Aug  4 22:20 kubectl
-rwxr-xr-x 1 root root 117788368 Aug  4 22:23 kubelet
-rwxr-xr-x 1 root root  10684792 Aug  4 22:24 runc

```
修改全局PATH

```bash
cat <<EOF>> /etc/profile.d/rke2.sh
export PATH=$PATH:/var/lib/rancher/rke2/bin
EOF
source /etc/profile
```
第二种方法

```bash
ln -s $(find /var/lib/rancher/rke2/data/ -name kubectl) /usr/local/bin/kubectl
```

### 9.4 检查节点
临时指定环境变量

```bash
$ KUBECONFIG=/etc/rancher/rke2/rke2.yaml kubectl get nodes
NAME            STATUS   ROLES                       AGE   VERSION
kube-master01   Ready    control-plane,etcd,master   47h   v1.25.12+rke2r1
```
永久配置
也可以修改`/etc/profile.d/rke2.sh`新增加一行:`export KUBECONFIG=/etc/rancher/rke2/rke2.yaml`

```bash
echo "export KUBECONFIG=/etc/rancher/rke2/rke2.yaml" >> /etc/profile.d/rke2.sh && source /etc/profile
```

最常见的方

```bash
mkdir /root/.kube
cp /etc/rancher/rke2/rke2.yaml /root/.kube/config

```
查看节点状态

```bash
$ kubectl get nodes
NAME            STATUS   ROLES                       AGE   VERSION
kube-master01   Ready    control-plane,etcd,master   47h   v1.25.12+rke2r1

```
检查pod
```bash
kubectl get pod -A
NAMESPACE     NAME                                                    READY   STATUS      RESTARTS   AGE
kube-system   cloud-controller-manager-rancher01                      1/1     Running     0          4m2s
kube-system   etcd-rancher01                                          1/1     Running     0          3m56s
kube-system   helm-install-rke2-canal-pfbh7                           0/1     Completed   0          4m50s
kube-system   helm-install-rke2-coredns-cr75p                         0/1     Completed   0          4m50s
kube-system   helm-install-rke2-ingress-nginx-nt29k                   0/1     Completed   0          4m50s
kube-system   helm-install-rke2-metrics-server-7fb5k                  0/1     Completed   0          4m50s
kube-system   helm-install-rke2-snapshot-controller-crd-rqx7h         0/1     Completed   0          4m50s
kube-system   helm-install-rke2-snapshot-controller-hn7f9             0/1     Completed   1          4m50s
kube-system   helm-install-rke2-snapshot-validation-webhook-qxqs2     0/1     Completed   0          4m50s
kube-system   kube-apiserver-rancher01                                1/1     Running     0          4m13s
kube-system   kube-controller-manager-rancher01                       1/1     Running     0          4m7s
kube-system   kube-proxy-rancher01                                    1/1     Running     0          4m9s
kube-system   kube-scheduler-rancher01                                1/1     Running     0          4m14s
kube-system   rke2-canal-khbgs                                        2/2     Running     0          4m10s
kube-system   rke2-coredns-rke2-coredns-7c98b7488c-cwxmx              1/1     Running     0          4m13s
kube-system   rke2-coredns-rke2-coredns-autoscaler-65b5bfc754-t4q7k   1/1     Running     0          4m13s
kube-system   rke2-ingress-nginx-controller-f6tl6                     1/1     Running     0          2m28s
kube-system   rke2-metrics-server-5bf59cdccb-nprbm                    1/1     Running     0          3m6s
kube-system   rke2-snapshot-controller-6f7bbb497d-qnxn9               1/1     Running     0          3m
kube-system   rke2-snapshot-validation-webhook-65b5675d5c-45hgm       1/1     Running     0          3m6s
```

查看镜像

```bash
$ crictl --runtime-endpoint  /run/k3s/containerd/containerd.sock images
I0806 23:37:59.380910 2361161 util_unix.go:103] "Using this endpoint is deprecated, please consider using full URL format" endpoint="/run/k3s/containerd/containerd.sock" URL="unix:///run/k3s/containerd/containerd.sock"
IMAGE                                                                TAG                                        IMAGE ID            SIZE
docker.io/rancher/hardened-calico                                    v3.25.1-build20230607                      035dbbbc6a3c9       192MB
docker.io/rancher/hardened-cluster-autoscaler                        v1.8.6-build20230406                       e8ce8d527364d       58.2MB
docker.io/rancher/hardened-coredns                                   v1.10.1-build20230406                      3c8207b045e32       64.3MB
docker.io/rancher/hardened-etcd                                      v3.5.7-k3s1-build20230406                  045ce52ecd52c       64.4MB
docker.io/rancher/hardened-flannel                                   v0.22.0-build20230612                      df20f583ea0ec       80.6MB
docker.io/rancher/hardened-k8s-metrics-server                        v0.6.3-build20230515                       9d8f4a693c64c       62.7MB
docker.io/rancher/hardened-kubernetes                                v1.25.12-rke2r1-build20230719              c9ce397e74f9e       214MB
docker.io/rancher/klipper-helm                                       v0.8.0-build20230510                       6f42df210d7fa       95MB
docker.io/rancher/mirrored-ingress-nginx-kube-webhook-certgen        v20230312-helm-chart-4.5.2-28-g66a760794   5a86b03a88d23       20.1MB
docker.io/rancher/mirrored-sig-storage-snapshot-controller           v6.2.1                                     1ef6c138bd5f2       24.2MB
docker.io/rancher/mirrored-sig-storage-snapshot-validation-webhook   v6.2.1                                     46e6854cb7c5e       21MB
docker.io/rancher/nginx-ingress-controller                           nginx-1.7.1-hardened1                      05fbbb9f7b895       336MB
docker.io/rancher/pause                                              3.6                                        6270bb605e12e       299kB
docker.io/rancher/rke2-cloud-provider                                v1.26.3-build20230406                      f906d1e7a5774       63.7MB


$ ctr --address /run/k3s/containerd/containerd.sock ns ls
NAME   LABELS 
k8s.io        
$ ctr --address /run/k3s/containerd/containerd.sock -n k8s.io i ls
REF                                                                                                                                        TYPE                                                      DIGEST                                                                  SIZE      PLATFORMS                                        LABELS                          
docker.io/rancher/hardened-calico:v3.25.1-build20230607                                                                                    application/vnd.docker.distribution.manifest.list.v2+json sha256:0ab773cb074d2b5e15c6fd643c47a13b3771f0c7d2b55368a2d7be7867642c0e 183.5 MiB linux/amd64,linux/arm64,linux/s390x              io.cri-containerd.image=managed 
docker.io/rancher/hardened-calico@sha256:0ab773cb074d2b5e15c6fd643c47a13b3771f0c7d2b55368a2d7be7867642c0e                                  application/vnd.docker.distribution.manifest.list.v2+json sha256:0ab773cb074d2b5e15c6fd643c47a13b3771f0c7d2b55368a2d7be7867642c0e 183.5 MiB linux/amd64,linux/arm64,linux/s390x              io.cri-containerd.image=managed 
docker.io/rancher/hardened-cluster-autoscaler:v1.8.6-build20230406                                                                         application/vnd.docker.distribution.manifest.list.v2+json sha256:b43ea2932c0cb9f8a496aefd323fa1fd92453e1179bd1bfa41d8e4a183b1d5bd 55.5 MiB  linux/amd64,linux/s390x                          io.cri-containerd.image=managed 
docker.io/rancher/hardened-cluster-autoscaler@sha256:b43ea2932c0cb9f8a496aefd323fa1fd92453e1179bd1bfa41d8e4a183b1d5bd                      application/vnd.docker.distribution.manifest.list.v2+json sha256:b43ea2932c0cb9f8a496aefd323fa1fd92453e1179bd1bfa41d8e4a183b1d5bd 55.5 MiB  linux/amd64,linux/s390x                          io.cri-containerd.image=managed 
docker.io/rancher/hardened-coredns:v1.10.1-build20230406                                                                                   application/vnd.docker.distribution.manifest.list.v2+json sha256:08366d74e793527dcfa08c5c4681b846893a43623f3162a4f8cdec438ad4788c 61.3 MiB  linux/amd64,linux/s390x                          io.cri-containerd.image=managed 
docker.io/rancher/hardened-coredns@sha256:08366d74e793527dcfa08c5c4681b846893a43623f3162a4f8cdec438ad4788c                                 application/vnd.docker.distribution.manifest.list.v2+json sha256:08366d74e793527dcfa08c5c4681b846893a43623f3162a4f8cdec438ad4788c 61.3 MiB  linux/amd64,linux/s390x                          io.cri-containerd.image=managed 
docker.io/rancher/hardened-etcd:v3.5.7-k3s1-build20230406                                                                                  application/vnd.docker.distribution.manifest.list.v2+json sha256:a17a257d4da7487ae5cec75beffa79b9b73cb14a527f7981e13f778bfdec7a64 61.5 MiB  linux/amd64,linux/s390x                          io.cri-containerd.image=managed 
docker.io/rancher/hardened-etcd@sha256:a17a257d4da7487ae5cec75beffa79b9b73cb14a527f7981e13f778bfdec7a64                                    application/vnd.docker.distribution.manifest.list.v2+json sha256:a17a257d4da7487ae5cec75beffa79b9b73cb14a527f7981e13f778bfdec7a64 61.5 MiB  linux/amd64,linux/s390x                          io.cri-containerd.image=managed 
docker.io/rancher/hardened-flannel:v0.22.0-build20230612                                                                                   application/vnd.docker.distribution.manifest.list.v2+json sha256:c9498be167f80e282edf017e40a393af66515e7d196d6a6a69a9fd55fe00f375 76.9 MiB  linux/amd64,linux/arm64,linux/s390x              io.cri-containerd.image=managed 
docker.io/rancher/hardened-flannel@sha256:c9498be167f80e282edf017e40a393af66515e7d196d6a6a69a9fd55fe00f375                                 application/vnd.docker.distribution.manifest.list.v2+json sha256:c9498be167f80e282edf017e40a393af66515e7d196d6a6a69a9fd55fe00f375 76.9 MiB  linux/amd64,linux/arm64,linux/s390x              io.cri-containerd.image=managed 
docker.io/rancher/hardened-k8s-metrics-server:v0.6.3-build20230515                                                                         application/vnd.docker.distribution.manifest.list.v2+json sha256:6dc29cb73fe57afdf4ca98e92bda1661040abb8cabf1577d8c62b4a97e4f54a7 59.8 MiB  linux/amd64,linux/arm64,linux/s390x              io.cri-containerd.image=managed 
docker.io/rancher/hardened-k8s-metrics-server@sha256:6dc29cb73fe57afdf4ca98e92bda1661040abb8cabf1577d8c62b4a97e4f54a7                      application/vnd.docker.distribution.manifest.list.v2+json sha256:6dc29cb73fe57afdf4ca98e92bda1661040abb8cabf1577d8c62b4a97e4f54a7 59.8 MiB  linux/amd64,linux/arm64,linux/s390x              io.cri-containerd.image=managed 
docker.io/rancher/hardened-kubernetes:v1.25.12-rke2r1-build20230719                                                                        application/vnd.docker.distribution.manifest.list.v2+json sha256:d37990dfa6e654ce722cd051d72c36ca71fc1278dcd2d2ef6056f9548652023b 203.8 MiB linux/amd64,linux/arm64,linux/s390x              io.cri-containerd.image=managed 
docker.io/rancher/hardened-kubernetes@sha256:d37990dfa6e654ce722cd051d72c36ca71fc1278dcd2d2ef6056f9548652023b                              application/vnd.docker.distribution.manifest.list.v2+json sha256:d37990dfa6e654ce722cd051d72c36ca71fc1278dcd2d2ef6056f9548652023b 203.8 MiB linux/amd64,linux/arm64,linux/s390x              io.cri-containerd.image=managed 
docker.io/rancher/klipper-helm:v0.8.0-build20230510                                                                                        application/vnd.docker.distribution.manifest.list.v2+json sha256:4d2ec9ac78f6e3ca3d4dd0a1c3b754aec2b4f19e3a922c6ebcb0d74bb5ac674a 90.6 MiB  linux/amd64,linux/arm,linux/arm64/v8,linux/s390x io.cri-containerd.image=managed 
docker.io/rancher/klipper-helm@sha256:4d2ec9ac78f6e3ca3d4dd0a1c3b754aec2b4f19e3a922c6ebcb0d74bb5ac674a                                     application/vnd.docker.distribution.manifest.list.v2+json sha256:4d2ec9ac78f6e3ca3d4dd0a1c3b754aec2b4f19e3a922c6ebcb0d74bb5ac674a 90.6 MiB  linux/amd64,linux/arm,linux/arm64/v8,linux/s390x io.cri-containerd.image=managed 
docker.io/rancher/mirrored-ingress-nginx-kube-webhook-certgen:v20230312-helm-chart-4.5.2-28-g66a760794                                     application/vnd.docker.distribution.manifest.list.v2+json sha256:25af4d737af79a08200df23208de4fa613efd2daba6801b559447f1f6b048714 19.2 MiB  linux/amd64,linux/arm/v7,linux/arm64,linux/s390x io.cri-containerd.image=managed 
docker.io/rancher/mirrored-ingress-nginx-kube-webhook-certgen@sha256:25af4d737af79a08200df23208de4fa613efd2daba6801b559447f1f6b048714      application/vnd.docker.distribution.manifest.list.v2+json sha256:25af4d737af79a08200df23208de4fa613efd2daba6801b559447f1f6b048714 19.2 MiB  linux/amd64,linux/arm/v7,linux/arm64,linux/s390x io.cri-containerd.image=managed 
docker.io/rancher/mirrored-sig-storage-snapshot-controller:v6.2.1                                                                          application/vnd.docker.distribution.manifest.list.v2+json sha256:8776214c491da926a9a808b4ad832c297262defeb2d736240ebed4be8d9f3512 23.1 MiB  linux/amd64,linux/arm/v7,linux/arm64,linux/s390x io.cri-containerd.image=managed 
docker.io/rancher/mirrored-sig-storage-snapshot-controller@sha256:8776214c491da926a9a808b4ad832c297262defeb2d736240ebed4be8d9f3512         application/vnd.docker.distribution.manifest.list.v2+json sha256:8776214c491da926a9a808b4ad832c297262defeb2d736240ebed4be8d9f3512 23.1 MiB  linux/amd64,linux/arm/v7,linux/arm64,linux/s390x io.cri-containerd.image=managed 
docker.io/rancher/mirrored-sig-storage-snapshot-validation-webhook:v6.2.1                                                                  application/vnd.docker.distribution.manifest.list.v2+json sha256:0f11f3ce9c91c259b7f5f34b3d64a47a14ac297f3af3379f6386e01b6dd2ef00 20.0 MiB  linux/amd64,linux/arm/v7,linux/arm64,linux/s390x io.cri-containerd.image=managed 
docker.io/rancher/mirrored-sig-storage-snapshot-validation-webhook@sha256:0f11f3ce9c91c259b7f5f34b3d64a47a14ac297f3af3379f6386e01b6dd2ef00 application/vnd.docker.distribution.manifest.list.v2+json sha256:0f11f3ce9c91c259b7f5f34b3d64a47a14ac297f3af3379f6386e01b6dd2ef00 20.0 MiB  linux/amd64,linux/arm/v7,linux/arm64,linux/s390x io.cri-containerd.image=managed 
docker.io/rancher/nginx-ingress-controller:nginx-1.7.1-hardened1                                                                           application/vnd.docker.distribution.manifest.list.v2+json sha256:b941f1a6f631a520791f6c227e3dab65d54432d9f9f7c3dc9895a7c067814894 320.0 MiB linux/amd64,linux/arm64,linux/s390x              io.cri-containerd.image=managed 
docker.io/rancher/nginx-ingress-controller@sha256:b941f1a6f631a520791f6c227e3dab65d54432d9f9f7c3dc9895a7c067814894                         application/vnd.docker.distribution.manifest.list.v2+json sha256:b941f1a6f631a520791f6c227e3dab65d54432d9f9f7c3dc9895a7c067814894 320.0 MiB linux/amd64,linux/arm64,linux/s390x              io.cri-containerd.image=managed 
docker.io/rancher/pause:3.6                                                                                                                application/vnd.docker.distribution.manifest.list.v2+json sha256:036d575e82945c112ef84e4585caff3648322a2f9ed4c3a6ce409dd10abc4f34 292.4 KiB linux/amd64,linux/s390x,windows/amd64            io.cri-containerd.image=managed 
docker.io/rancher/pause@sha256:036d575e82945c112ef84e4585caff3648322a2f9ed4c3a6ce409dd10abc4f34                                            application/vnd.docker.distribution.manifest.list.v2+json sha256:036d575e82945c112ef84e4585caff3648322a2f9ed4c3a6ce409dd10abc4f34 292.4 KiB linux/amd64,linux/s390x,windows/amd64            io.cri-containerd.image=managed 
docker.io/rancher/rke2-cloud-provider:v1.26.3-build20230406                                                                                application/vnd.docker.distribution.manifest.list.v2+json sha256:e2d98791f28b7aed3ab99afb99b52310eb1a36844b9bc9c497ebce327e4c68d5 60.7 MiB  linux/amd64,linux/s390x                          io.cri-containerd.image=managed 
docker.io/rancher/rke2-cloud-provider@sha256:e2d98791f28b7aed3ab99afb99b52310eb1a36844b9bc9c497ebce327e4c68d5                              application/vnd.docker.distribution.manifest.list.v2+json sha256:e2d98791f28b7aed3ab99afb99b52310eb1a36844b9bc9c497ebce327e4c68d5 60.7 MiB  linux/amd64,linux/s390x                          io.cri-containerd.image=managed 
sha256:035dbbbc6a3c9d838f91551add00d99e851e252749b8c12ba665cd73cf2fbacd                                                                    application/vnd.docker.distribution.manifest.list.v2+json sha256:0ab773cb074d2b5e15c6fd643c47a13b3771f0c7d2b55368a2d7be7867642c0e 183.5 MiB linux/amd64,linux/arm64,linux/s390x              io.cri-containerd.image=managed 
sha256:045ce52ecd52c74ebd7e53391f37fc9110e87aa1fe38ee00d76955c2b26ac338                                                                    application/vnd.docker.distribution.manifest.list.v2+json sha256:a17a257d4da7487ae5cec75beffa79b9b73cb14a527f7981e13f778bfdec7a64 61.5 MiB  linux/amd64,linux/s390x                          io.cri-containerd.image=managed 
sha256:05fbbb9f7b895dad4d2ccdc3cf1b7d294777984d60748ccfce6d06ae797bf421                                                                    application/vnd.docker.distribution.manifest.list.v2+json sha256:b941f1a6f631a520791f6c227e3dab65d54432d9f9f7c3dc9895a7c067814894 320.0 MiB linux/amd64,linux/arm64,linux/s390x              io.cri-containerd.image=managed 
sha256:1ef6c138bd5f2ac45f7b4ee54db0e513efad8576909ae9829ba649fb4b067388                                                                    application/vnd.docker.distribution.manifest.list.v2+json sha256:8776214c491da926a9a808b4ad832c297262defeb2d736240ebed4be8d9f3512 23.1 MiB  linux/amd64,linux/arm/v7,linux/arm64,linux/s390x io.cri-containerd.image=managed 
sha256:3c8207b045e329fc747a900f7bc32663b1d146909b489b1813e2fff5990daa11                                                                    application/vnd.docker.distribution.manifest.list.v2+json sha256:08366d74e793527dcfa08c5c4681b846893a43623f3162a4f8cdec438ad4788c 61.3 MiB  linux/amd64,linux/s390x                          io.cri-containerd.image=managed 
sha256:46e6854cb7c5e33c86a79a70816e18ce34eab9956c152bc73071655c69b5f758                                                                    application/vnd.docker.distribution.manifest.list.v2+json sha256:0f11f3ce9c91c259b7f5f34b3d64a47a14ac297f3af3379f6386e01b6dd2ef00 20.0 MiB  linux/amd64,linux/arm/v7,linux/arm64,linux/s390x io.cri-containerd.image=managed 
sha256:5a86b03a88d2316e2317c2576449a95ddbd105d69b2fe7b01d667b0ebab37422                                                                    application/vnd.docker.distribution.manifest.list.v2+json sha256:25af4d737af79a08200df23208de4fa613efd2daba6801b559447f1f6b048714 19.2 MiB  linux/amd64,linux/arm/v7,linux/arm64,linux/s390x io.cri-containerd.image=managed 
sha256:6270bb605e12e581514ada5fd5b3216f727db55dc87d5889c790e4c760683fee                                                                    application/vnd.docker.distribution.manifest.list.v2+json sha256:036d575e82945c112ef84e4585caff3648322a2f9ed4c3a6ce409dd10abc4f34 292.4 KiB linux/amd64,linux/s390x,windows/amd64            io.cri-containerd.image=managed 
sha256:6f42df210d7faa0b6eb56f5f28bb061930c7834af1e7dd4b3586108ec6b1485b                                                                    application/vnd.docker.distribution.manifest.list.v2+json sha256:4d2ec9ac78f6e3ca3d4dd0a1c3b754aec2b4f19e3a922c6ebcb0d74bb5ac674a 90.6 MiB  linux/amd64,linux/arm,linux/arm64/v8,linux/s390x io.cri-containerd.image=managed 
sha256:9d8f4a693c64ce3d7ea861f4064ab0ca0d7cea079f81a4bac5ef7d5369ab71bd                                                                    application/vnd.docker.distribution.manifest.list.v2+json sha256:6dc29cb73fe57afdf4ca98e92bda1661040abb8cabf1577d8c62b4a97e4f54a7 59.8 MiB  linux/amd64,linux/arm64,linux/s390x              io.cri-containerd.image=managed 
sha256:c9ce397e74f9ec92de6aea5191526c6359634151faaeceafe715a476b6de7af4                                                                    application/vnd.docker.distribution.manifest.list.v2+json sha256:d37990dfa6e654ce722cd051d72c36ca71fc1278dcd2d2ef6056f9548652023b 203.8 MiB linux/amd64,linux/arm64,linux/s390x              io.cri-containerd.image=managed 
sha256:df20f583ea0ec60d597afec6ce603495818a513029698def201c99681703a0a0                                                                    application/vnd.docker.distribution.manifest.list.v2+json sha256:c9498be167f80e282edf017e40a393af66515e7d196d6a6a69a9fd55fe00f375 76.9 MiB  linux/amd64,linux/arm64,linux/s390x              io.cri-containerd.image=managed 
sha256:e8ce8d527364d034a0d3f3bcdd4ef62a5fcb4d662eefbbf1d1adc0053545f27e                                                                    application/vnd.docker.distribution.manifest.list.v2+json sha256:b43ea2932c0cb9f8a496aefd323fa1fd92453e1179bd1bfa41d8e4a183b1d5bd 55.5 MiB  linux/amd64,linux/s390x                          io.cri-containerd.image=managed 
sha256:f906d1e7a5774a6e36dddaadcefa240b1813bc921b50303fd0b0874519ccf889                                                                    application/vnd.docker.distribution.manifest.list.v2+json sha256:e2d98791f28b7aed3ab99afb99b52310eb1a36844b9bc9c497ebce327e4c68d5 60.7 MiB  linux/amd64,linux/s390x                          io.cri-containerd.image=managed 

```

## 10. 配置其他管理节点
第一服务器节点建立秘密令牌，当连接到集群时，其他服务器或代理节点将向该秘密令牌注册。
要将自己的预共享密钥指定为令牌，请在启动时设置令牌参数。

如果您没有指定预共享密钥，RKE 2将生成一个并将其放置在`/var/lib/rancher/rke 2/server/node-token`中.

```bash
$ cat /var/lib/rancher/rke2/server/node-token 
K10b5f953b3a0bc4e47a09566b3dc074214e714077ca1e1d9932bc8f0de631b47cd::server:8cee7660a08976ac4df2629ad05a0da0
```

为了避免使用固定注册地址时出现证书错误，应该使用`tls-san`参数集启动服务器。此选项在服务器的TLS证书中添加一个附加的主机名或IP作为主题备用名称，如果您希望通过IP和主机名进行访问，则可以将其指定为列表。

```bash
mkdir -p /etc/rancher/rke2/
```

```bash
$ vim /etc/rancher/rke2/config.yaml
server: https://192.168.23.31:9345
token: K10b5f953b3a0bc4e47a09566b3dc074214e714077ca1e1d9932bc8f0de631b47cd::server:8cee7660a08976ac4df2629ad05a0da0
```

安装 rke2-server

```bash
curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE=server sh -
systemctl enable rke2-server.service && systemctl start rke2-server.service
```
查看日志

```bash
journalctl -u rke2-server -f
```

##  11. Agent 节点配置

```bash
mkdir -p /etc/rancher/rke2/
```

```bash
$ vim /etc/rancher/rke2/config.yaml
server: https://192.168.23.31:9345
token: K10b5f953b3a0bc4e47a09566b3dc074214e714077ca1e1d9932bc8f0de631b47cd::server:8cee7660a08976ac4df2629ad05a0da0
```

安装 rke2-agent

```bash
curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE=agent sh -
systemctl enable rke2-agent.service && systemctl start rke2-agent.service
```
查看日志

```bash
journalctl -u rke2-server -f
```
输出

```bash
[root@kube-prom01 ~]# curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE="agent" sh -
[INFO]  finding release for channel stable
[INFO]  using 1.25 series from channel stable
Rancher RKE2 Common (stable)                                                                                                                                        3.5 kB/s | 2.9 kB     00:00    
Rancher RKE2 1.25 (stable)                                                                                                                                          3.3 kB/s | 2.9 kB     00:00    
Dependencies resolved.
====================================================================================================================================================================================================
 Package                                           Architecture                Version                                                        Repository                                       Size
====================================================================================================================================================================================================
Installing:
 rke2-agent                                        x86_64                      1.25.12~rke2r1-0.el8                                           rancher-rke2-1.25-stable                        8.8 k
Installing dependencies:
 checkpolicy                                       x86_64                      2.9-1.el8                                                      baseos                                          345 k
 container-selinux                                 noarch                      2:2.205.0-2.module+el8.8.0+1265+fa25dd7a                       appstream                                        63 k
 policycoreutils-python-utils                      noarch                      2.9-24.el8                                                     baseos                                          253 k
 python3-audit                                     x86_64                      3.0.7-4.el8                                                    baseos                                           86 k
 python3-libsemanage                               x86_64                      2.9-9.el8_6                                                    baseos                                          127 k
 python3-policycoreutils                           noarch                      2.9-24.el8                                                     baseos                                          2.2 M
 python3-setools                                   x86_64                      4.3.0-3.el8                                                    baseos                                          623 k
 rke2-common                                       x86_64                      1.25.12~rke2r1-0.el8                                           rancher-rke2-1.25-stable                         19 M
 rke2-selinux                                      noarch                      0.14-1.el8                                                     rancher-rke2-common-stable                       21 k

Transaction Summary
====================================================================================================================================================================================================
Install  10 Packages

Total download size: 23 M
Installed size: 92 M
Downloading Packages:
[MIRROR] container-selinux-2.205.0-2.module+el8.8.0+1265+fa25dd7a.noarch.rpm: Status code: 502 for http://mirror.atl.genesisadaptive.com/rocky/8.8/AppStream/x86_64/os/Packages/c/container-selinux-2.205.0-2.module%2bel8.8.0%2b1265%2bfa25dd7a.noarch.rpm (IP: 192.168.21.101)
(1/10): policycoreutils-python-utils-2.9-24.el8.noarch.rpm                                                                                                          124 kB/s | 253 kB     00:02    
(2/10): checkpolicy-2.9-1.el8.x86_64.rpm                                                                                                                            167 kB/s | 345 kB     00:02    
(3/10): python3-audit-3.0.7-4.el8.x86_64.rpm                                                                                                                        180 kB/s |  86 kB     00:00    
(4/10): python3-libsemanage-2.9-9.el8_6.x86_64.rpm                                                                                                                  170 kB/s | 127 kB     00:00    
(5/10): container-selinux-2.205.0-2.module+el8.8.0+1265+fa25dd7a.noarch.rpm                                                                                          21 kB/s |  63 kB     00:03    
(6/10): rke2-selinux-0.14-1.el8.noarch.rpm                                                                                                                           19 kB/s |  21 kB     00:01    
(7/10): rke2-agent-1.25.12~rke2r1-0.el8.x86_64.rpm                                                                                                                   23 kB/s | 8.8 kB     00:00    
(8/10): python3-setools-4.3.0-3.el8.x86_64.rpm                                                                                                                      155 kB/s | 623 kB     00:04    
(9/10): rke2-common-1.25.12~rke2r1-0.el8.x86_64.rpm                                                                                                                 5.7 MB/s |  19 MB     00:03    
[MIRROR] python3-policycoreutils-2.9-24.el8.noarch.rpm: Curl error (18): Transferred a partial file for https://rockylinux.hi.no/8.8/BaseOS/x86_64/os/Packages/p/python3-policycoreutils-2.9-24.el8.noarch.rpm [transfer closed with 63019 bytes remaining to read]
(10/10): python3-policycoreutils-2.9-24.el8.noarch.rpm                                                                                                               53 kB/s | 2.2 MB     00:43    
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                               498 kB/s |  23 MB     00:47     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                                            1/1 
  Installing       : python3-setools-4.3.0-3.el8.x86_64                                                                                                                                        1/10 
  Installing       : python3-libsemanage-2.9-9.el8_6.x86_64                                                                                                                                    2/10 
  Installing       : python3-audit-3.0.7-4.el8.x86_64                                                                                                                                          3/10 
  Installing       : checkpolicy-2.9-1.el8.x86_64                                                                                                                                              4/10 
  Installing       : python3-policycoreutils-2.9-24.el8.noarch                                                                                                                                 5/10 
  Installing       : policycoreutils-python-utils-2.9-24.el8.noarch                                                                                                                            6/10 
  Running scriptlet: container-selinux-2:2.205.0-2.module+el8.8.0+1265+fa25dd7a.noarch                                                                                                         7/10 
  Installing       : container-selinux-2:2.205.0-2.module+el8.8.0+1265+fa25dd7a.noarch                                                                                                         7/10 
  Running scriptlet: container-selinux-2:2.205.0-2.module+el8.8.0+1265+fa25dd7a.noarch                                                                                                         7/10 
  Running scriptlet: rke2-selinux-0.14-1.el8.noarch                                                                                                                                            8/10 
  Installing       : rke2-selinux-0.14-1.el8.noarch                                                                                                                                            8/10 
  Running scriptlet: rke2-selinux-0.14-1.el8.noarch                                                                                                                                            8/10 
  Installing       : rke2-common-1.25.12~rke2r1-0.el8.x86_64                                                                                                                                   9/10 
  Installing       : rke2-agent-1.25.12~rke2r1-0.el8.x86_64                                                                                                                                   10/10 
  Running scriptlet: rke2-agent-1.25.12~rke2r1-0.el8.x86_64                                                                                                                                   10/10 
  Running scriptlet: container-selinux-2:2.205.0-2.module+el8.8.0+1265+fa25dd7a.noarch                                                                                                        10/10 
  Running scriptlet: rke2-selinux-0.14-1.el8.noarch                                                                                                                                           10/10 
  Running scriptlet: rke2-agent-1.25.12~rke2r1-0.el8.x86_64                                                                                                                                   10/10 
  Verifying        : container-selinux-2:2.205.0-2.module+el8.8.0+1265+fa25dd7a.noarch                                                                                                         1/10 
  Verifying        : checkpolicy-2.9-1.el8.x86_64                                                                                                                                              2/10 
  Verifying        : policycoreutils-python-utils-2.9-24.el8.noarch                                                                                                                            3/10 
  Verifying        : python3-audit-3.0.7-4.el8.x86_64                                                                                                                                          4/10 
  Verifying        : python3-libsemanage-2.9-9.el8_6.x86_64                                                                                                                                    5/10 
  Verifying        : python3-policycoreutils-2.9-24.el8.noarch                                                                                                                                 6/10 
  Verifying        : python3-setools-4.3.0-3.el8.x86_64                                                                                                                                        7/10 
  Verifying        : rke2-selinux-0.14-1.el8.noarch                                                                                                                                            8/10 
  Verifying        : rke2-agent-1.25.12~rke2r1-0.el8.x86_64                                                                                                                                    9/10 
  Verifying        : rke2-common-1.25.12~rke2r1-0.el8.x86_64                                                                                                                                  10/10 

Installed:
  checkpolicy-2.9-1.el8.x86_64            container-selinux-2:2.205.0-2.module+el8.8.0+1265+fa25dd7a.noarch policycoreutils-python-utils-2.9-24.el8.noarch python3-audit-3.0.7-4.el8.x86_64      
  python3-libsemanage-2.9-9.el8_6.x86_64  python3-policycoreutils-2.9-24.el8.noarch                         python3-setools-4.3.0-3.el8.x86_64             rke2-agent-1.25.12~rke2r1-0.el8.x86_64
  rke2-common-1.25.12~rke2r1-0.el8.x86_64 rke2-selinux-0.14-1.el8.noarch                                   

Complete!

```
启动rke2-agent

```bash
systemctl start rke2-agent 
```


监控加入节点过程
>  注意： 也许日志会报错，但这些都是正常的过程，最后在漫长的等待会成功

```bash
[root@kube-master01 ~]# kubectl get node -w
NAME            STATUS   ROLES                       AGE   VERSION
kube-master01   Ready    control-plane,etcd,master   2d    v1.25.12+rke2r1
kube-master01   Ready    control-plane,etcd,master   2d    v1.25.12+rke2r1
kube-master01   Ready    control-plane,etcd,master   2d    v1.25.12+rke2r1
kube-master01   Ready    control-plane,etcd,master   2d1h   v1.25.12+rke2r1
kube-master01   Ready    control-plane,etcd,master   2d1h   v1.25.12+rke2r1
kube-prom01     NotReady   <none>                      0s     v1.25.12+rke2r1
kube-prom01     NotReady   <none>                      0s     v1.25.12+rke2r1
kube-prom01     NotReady   <none>                      0s     v1.25.12+rke2r1
kube-prom01     NotReady   <none>                      0s     v1.25.12+rke2r1
kube-prom01     NotReady   <none>                      0s     v1.25.12+rke2r1
kube-prom01     NotReady   <none>                      0s     v1.25.12+rke2r1
kube-prom01     NotReady   <none>                      0s     v1.25.12+rke2r1
kube-prom01     NotReady   <none>                      0s     v1.25.12+rke2r1
kube-prom01     NotReady   <none>                      0s     v1.25.12+rke2r1
kube-prom01     NotReady   <none>                      10s    v1.25.12+rke2r1
kube-prom01     NotReady   <none>                      30s    v1.25.12+rke2r1
kube-master01   Ready      control-plane,etcd,master   2d1h   v1.25.12+rke2r1
kube-prom01     NotReady   <none>                      117s   v1.25.12+rke2r1
kube-prom01     Ready      <none>                      2m3s   v1.25.12+rke2r1
kube-prom01     Ready      <none>                      2m3s   v1.25.12+rke2r1
kube-prom01     Ready      <none>                      2m5s   v1.25.12+rke2r1
kube-prom01     Ready      <none>                      2m32s   v1.25.12+rke2r1
kube-prom01     Ready      <none>                      2m32s   v1.25.12+rke2r1
kube-prom01     Ready      <none>                      3m4s    v1.25.12+rke2r1

[root@kube-prom01 ~]# journalctl -u rke2-agent -f
-- Logs begin at Fri 2023-08-04 15:28:38 CST. --
Aug 06 23:05:24 kube-prom01 rke2[2179120]: time="2023-08-06T23:05:24+08:00" level=info msg="Running load balancer rke2-api-server-agent-load-balancer 127.0.0.1:6443 -> [192.168.23.31:6443] [default: 192.168.23.31:6443]"
Aug 06 23:05:27 kube-prom01 rke2[2179120]: time="2023-08-06T23:05:27+08:00" level=info msg="Module overlay was already loaded"
Aug 06 23:05:27 kube-prom01 rke2[2179120]: time="2023-08-06T23:05:27+08:00" level=info msg="Module nf_conntrack was already loaded"
Aug 06 23:05:27 kube-prom01 rke2[2179120]: time="2023-08-06T23:05:27+08:00" level=info msg="Module br_netfilter was already loaded"
Aug 06 23:05:27 kube-prom01 rke2[2179120]: time="2023-08-06T23:05:27+08:00" level=info msg="Module iptable_nat was already loaded"
Aug 06 23:05:27 kube-prom01 rke2[2179120]: time="2023-08-06T23:05:27+08:00" level=info msg="Module iptable_filter was already loaded"
Aug 06 23:05:27 kube-prom01 rke2[2179120]: time="2023-08-06T23:05:27+08:00" level=warning msg="Failed to load runtime image index.docker.io/rancher/rke2-runtime:v1.25.12-rke2r1 from tarball: no local image available for index.docker.io/rancher/rke2-runtime:v1.25.12-rke2r1: directory /var/lib/rancher/rke2/agent/images does not exist: image not found"
Aug 06 23:05:27 kube-prom01 rke2[2179120]: time="2023-08-06T23:05:27+08:00" level=info msg="Pulling runtime image index.docker.io/rancher/rke2-runtime:v1.25.12-rke2r1"
Aug 06 23:05:44 kube-prom01 rke2[2179120]: time="2023-08-06T23:05:44+08:00" level=info msg="Creating directory /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin"
Aug 06 23:05:44 kube-prom01 rke2[2179120]: time="2023-08-06T23:05:44+08:00" level=info msg="Extracting file bin/containerd to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin/containerd"
Aug 06 23:21:39 kube-prom01 rke2[2179120]: time="2023-08-06T23:21:39+08:00" level=info msg="Extracting file bin/containerd-shim to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin/containerd-shim"

Aug 06 23:23:31 kube-prom01 rke2[2179120]: time="2023-08-06T23:23:31+08:00" level=info msg="Extracting file bin/containerd-shim-runc-v1 to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin/containerd-shim-runc-v1"




Aug 06 23:29:52 kube-prom01 rke2[2179120]: time="2023-08-06T23:29:52+08:00" level=info msg="Extracting file bin/containerd-shim-runc-v2 to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin/containerd-shim-runc-v2"
Aug 06 23:35:42 kube-prom01 rke2[2179120]: time="2023-08-06T23:35:42+08:00" level=info msg="Extracting file bin/crictl to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin/crictl"












Aug 06 23:57:16 kube-prom01 rke2[2179120]: time="2023-08-06T23:57:16+08:00" level=info msg="Extracting file bin/ctr to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin/ctr"
Aug 07 00:06:11 kube-prom01 rke2[2179120]: time="2023-08-07T00:06:11+08:00" level=info msg="Extracting file bin/kubectl to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin/kubectl"
Aug 07 00:11:02 kube-prom01 rke2[2179120]: time="2023-08-07T00:11:02+08:00" level=fatal msg="failed to extract runtime image: stream error: stream ID 1; INTERNAL_ERROR; received from peer"
Aug 07 00:11:02 kube-prom01 systemd[1]: rke2-agent.service: Main process exited, code=exited, status=1/FAILURE
Aug 07 00:11:02 kube-prom01 systemd[1]: rke2-agent.service: Failed with result 'exit-code'.
Aug 07 00:11:02 kube-prom01 systemd[1]: Failed to start Rancher Kubernetes Engine v2 (agent).
Aug 07 00:11:07 kube-prom01 systemd[1]: rke2-agent.service: Service RestartSec=5s expired, scheduling restart.
Aug 07 00:11:07 kube-prom01 systemd[1]: rke2-agent.service: Scheduled restart job, restart counter is at 1.
Aug 07 00:11:08 kube-prom01 systemd[1]: Stopped Rancher Kubernetes Engine v2 (agent).
Aug 07 00:11:08 kube-prom01 systemd[1]: Starting Rancher Kubernetes Engine v2 (agent)...
Aug 07 00:11:08 kube-prom01 sh[2179307]: + /usr/bin/systemctl is-enabled --quiet nm-cloud-setup.service
Aug 07 00:11:08 kube-prom01 sh[2179308]: Failed to get unit file state for nm-cloud-setup.service: No such file or directory
Aug 07 00:11:08 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:08+08:00" level=warning msg="not running in CIS mode"
Aug 07 00:11:08 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:08+08:00" level=info msg="Applying Pod Security Admission Configuration"
Aug 07 00:11:08 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:08+08:00" level=info msg="Starting rke2 agent v1.25.12+rke2r1 (a0aa49e91d86a9a5eec8d94cdab13afabe11bfb6)"
Aug 07 00:11:08 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:08+08:00" level=info msg="Adding server to load balancer rke2-agent-load-balancer: 192.168.23.31:9345"
Aug 07 00:11:08 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:08+08:00" level=info msg="Running load balancer rke2-agent-load-balancer 127.0.0.1:6444 -> [192.168.23.31:9345] [default: 192.168.23.31:9345]"
Aug 07 00:11:08 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:08+08:00" level=info msg="Adding server to load balancer rke2-api-server-agent-load-balancer: 192.168.23.31:6443"
Aug 07 00:11:08 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:08+08:00" level=info msg="Running load balancer rke2-api-server-agent-load-balancer 127.0.0.1:6443 -> [192.168.23.31:6443] [default: 192.168.23.31:6443]"
Aug 07 00:11:11 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:11+08:00" level=info msg="Module overlay was already loaded"
Aug 07 00:11:11 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:11+08:00" level=info msg="Module nf_conntrack was already loaded"
Aug 07 00:11:11 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:11+08:00" level=info msg="Module br_netfilter was already loaded"
Aug 07 00:11:11 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:11+08:00" level=info msg="Module iptable_nat was already loaded"
Aug 07 00:11:11 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:11+08:00" level=info msg="Module iptable_filter was already loaded"
Aug 07 00:11:11 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:11+08:00" level=warning msg="Failed to load runtime image index.docker.io/rancher/rke2-runtime:v1.25.12-rke2r1 from tarball: no local image available for index.docker.io/rancher/rke2-runtime:v1.25.12-rke2r1: directory /var/lib/rancher/rke2/agent/images does not exist: image not found"
Aug 07 00:11:11 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:11+08:00" level=info msg="Pulling runtime image index.docker.io/rancher/rke2-runtime:v1.25.12-rke2r1"
Aug 07 00:11:21 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:21+08:00" level=info msg="Creating directory /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin"
Aug 07 00:11:21 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:21+08:00" level=info msg="Extracting file bin/containerd to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin/containerd"
Aug 07 00:11:30 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:30+08:00" level=info msg="Extracting file bin/containerd-shim to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin/containerd-shim"
Aug 07 00:11:30 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:30+08:00" level=info msg="Extracting file bin/containerd-shim-runc-v1 to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin/containerd-shim-runc-v1"
Aug 07 00:11:31 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:31+08:00" level=info msg="Extracting file bin/containerd-shim-runc-v2 to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin/containerd-shim-runc-v2"
Aug 07 00:11:33 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:33+08:00" level=info msg="Extracting file bin/crictl to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin/crictl"
Aug 07 00:11:36 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:36+08:00" level=info msg="Extracting file bin/ctr to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin/ctr"
Aug 07 00:11:38 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:38+08:00" level=info msg="Extracting file bin/kubectl to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin/kubectl"
Aug 07 00:11:42 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:42+08:00" level=info msg="Extracting file bin/kubelet to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin/kubelet"
Aug 07 00:11:50 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:50+08:00" level=info msg="Extracting file bin/runc to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/bin/runc"
Aug 07 00:11:51 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:51+08:00" level=info msg="Creating directory /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/charts"
Aug 07 00:11:51 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:51+08:00" level=info msg="Extracting file charts/harvester-cloud-provider.yaml to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/charts/harvester-cloud-provider.yaml"
Aug 07 00:11:51 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:51+08:00" level=info msg="Extracting file charts/harvester-csi-driver.yaml to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/charts/harvester-csi-driver.yaml"
Aug 07 00:11:51 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:51+08:00" level=info msg="Extracting file charts/rancher-vsphere-cpi.yaml to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/charts/rancher-vsphere-cpi.yaml"
Aug 07 00:11:51 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:51+08:00" level=info msg="Extracting file charts/rancher-vsphere-csi.yaml to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/charts/rancher-vsphere-csi.yaml"
Aug 07 00:11:51 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:51+08:00" level=info msg="Extracting file charts/rke2-calico-crd.yaml to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/charts/rke2-calico-crd.yaml"
Aug 07 00:11:51 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:51+08:00" level=info msg="Extracting file charts/rke2-calico.yaml to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/charts/rke2-calico.yaml"
Aug 07 00:11:51 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:51+08:00" level=info msg="Extracting file charts/rke2-canal.yaml to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/charts/rke2-canal.yaml"
Aug 07 00:11:51 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:51+08:00" level=info msg="Extracting file charts/rke2-cilium.yaml to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/charts/rke2-cilium.yaml"
Aug 07 00:11:51 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:51+08:00" level=info msg="Extracting file charts/rke2-coredns.yaml to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/charts/rke2-coredns.yaml"
Aug 07 00:11:51 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:51+08:00" level=info msg="Extracting file charts/rke2-ingress-nginx.yaml to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/charts/rke2-ingress-nginx.yaml"
Aug 07 00:11:51 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:51+08:00" level=info msg="Extracting file charts/rke2-metrics-server.yaml to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/charts/rke2-metrics-server.yaml"
Aug 07 00:11:51 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:51+08:00" level=info msg="Extracting file charts/rke2-multus.yaml to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/charts/rke2-multus.yaml"
Aug 07 00:11:51 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:51+08:00" level=info msg="Extracting file charts/rke2-snapshot-controller-crd.yaml to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/charts/rke2-snapshot-controller-crd.yaml"
Aug 07 00:11:51 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:51+08:00" level=info msg="Extracting file charts/rke2-snapshot-controller.yaml to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/charts/rke2-snapshot-controller.yaml"
Aug 07 00:11:51 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:51+08:00" level=info msg="Extracting file charts/rke2-snapshot-validation-webhook.yaml to /var/lib/rancher/rke2/data/v1.25.12-rke2r1-15557ace5a8f/charts/rke2-snapshot-validation-webhook.yaml"
Aug 07 00:11:51 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:51+08:00" level=warning msg="SELinux is enabled for rke2 but process is not running in context 'container_runtime_t', rke2-selinux policy may need to be applied"
Aug 07 00:11:51 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:51+08:00" level=info msg="Logging containerd to /var/lib/rancher/rke2/agent/containerd/containerd.log"
Aug 07 00:11:51 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:51+08:00" level=info msg="Running containerd -c /var/lib/rancher/rke2/agent/etc/containerd/config.toml -a /run/k3s/containerd/containerd.sock --state /run/k3s/containerd --root /var/lib/rancher/rke2/agent/containerd"
Aug 07 00:11:52 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:52+08:00" level=info msg="Waiting for containerd startup: rpc error: code = Unavailable desc = connection error: desc = \"transport: Error while dialing dial unix /run/k3s/containerd/containerd.sock: connect: connection refused\""
Aug 07 00:11:53 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:53+08:00" level=info msg="Waiting for containerd startup: rpc error: code = Unavailable desc = connection error: desc = \"transport: Error while dialing dial unix /run/k3s/containerd/containerd.sock: connect: connection refused\""
Aug 07 00:11:54 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:54+08:00" level=info msg="Waiting for containerd startup: rpc error: code = Unavailable desc = connection error: desc = \"transport: Error while dialing dial unix /run/k3s/containerd/containerd.sock: connect: connection refused\""
Aug 07 00:11:55 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:55+08:00" level=info msg="Waiting for containerd startup: rpc error: code = Unavailable desc = connection error: desc = \"transport: Error while dialing dial unix /run/k3s/containerd/containerd.sock: connect: connection refused\""
Aug 07 00:11:56 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:56+08:00" level=info msg="containerd is now running"
Aug 07 00:11:56 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:56+08:00" level=info msg="Getting list of apiserver endpoints from server"
Aug 07 00:11:57 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:57+08:00" level=info msg="Updated load balancer rke2-agent-load-balancer default server address -> 192.168.23.31:9345"
Aug 07 00:11:57 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:57+08:00" level=info msg="Connecting to proxy" url="wss://192.168.23.31:9345/v1-rke2/connect"
Aug 07 00:11:57 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:57+08:00" level=info msg="Running kubelet --address=0.0.0.0 --allowed-unsafe-sysctls=net.ipv4.ip_forward,net.ipv6.conf.all.forwarding --alsologtostderr=false --anonymous-auth=false --authentication-token-webhook=true --authorization-mode=Webhook --cgroup-driver=systemd --client-ca-file=/var/lib/rancher/rke2/agent/client-ca.crt --cloud-provider=external --cluster-dns=10.43.0.10 --cluster-domain=cluster.local --container-runtime-endpoint=unix:///run/k3s/containerd/containerd.sock --containerd=/run/k3s/containerd/containerd.sock --eviction-hard=imagefs.available<5%,nodefs.available<5% --eviction-minimum-reclaim=imagefs.available=10%,nodefs.available=10% --fail-swap-on=false --healthz-bind-address=127.0.0.1 --hostname-override=kube-prom01 --kubeconfig=/var/lib/rancher/rke2/agent/kubelet.kubeconfig --log-file=/var/lib/rancher/rke2/agent/logs/kubelet.log --log-file-max-size=50 --logtostderr=false --node-labels= --pod-infra-container-image=index.docker.io/rancher/pause:3.6 --pod-manifest-path=/var/lib/rancher/rke2/agent/pod-manifests --read-only-port=0 --resolv-conf=/etc/resolv.conf --serialize-image-pulls=false --stderrthreshold=FATAL --tls-cert-file=/var/lib/rancher/rke2/agent/serving-kubelet.crt --tls-private-key-file=/var/lib/rancher/rke2/agent/serving-kubelet.key"
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --volume-plugin-dir has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --file-check-frequency has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --sync-frequency has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --address has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --allowed-unsafe-sysctls has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --alsologtostderr has been deprecated, will be removed in a future release, see https://github.com/kubernetes/enhancements/tree/master/keps/sig-instrumentation/2845-deprecate-klog-specific-flags-in-k8s-components
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --anonymous-auth has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --authentication-token-webhook has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --authorization-mode has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --cgroup-driver has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --client-ca-file has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --cloud-provider has been deprecated, will be removed in 1.25 or later, in favor of removing cloud provider code from Kubelet.
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --cluster-dns has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --cluster-domain has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --containerd has been deprecated, This is a cadvisor flag that was mistakenly registered with the Kubelet. Due to legacy concerns, it will follow the standard CLI deprecation timeline before being removed.
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --eviction-hard has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --eviction-minimum-reclaim has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --fail-swap-on has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --healthz-bind-address has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --log-file has been deprecated, will be removed in a future release, see https://github.com/kubernetes/enhancements/tree/master/keps/sig-instrumentation/2845-deprecate-klog-specific-flags-in-k8s-components
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --log-file-max-size has been deprecated, will be removed in a future release, see https://github.com/kubernetes/enhancements/tree/master/keps/sig-instrumentation/2845-deprecate-klog-specific-flags-in-k8s-components
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --logtostderr has been deprecated, will be removed in a future release, see https://github.com/kubernetes/enhancements/tree/master/keps/sig-instrumentation/2845-deprecate-klog-specific-flags-in-k8s-components
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --pod-infra-container-image has been deprecated, will be removed in 1.27. Image garbage collector will get sandbox image information from CRI.
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --pod-manifest-path has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --read-only-port has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --resolv-conf has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --serialize-image-pulls has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --stderrthreshold has been deprecated, will be removed in a future release, see https://github.com/kubernetes/enhancements/tree/master/keps/sig-instrumentation/2845-deprecate-klog-specific-flags-in-k8s-components
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --tls-cert-file has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Aug 07 00:11:57 kube-prom01 rke2[2179679]: Flag --tls-private-key-file has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.
Aug 07 00:11:58 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:58+08:00" level=info msg="Failed to set annotations and labels on node kube-prom01: Operation cannot be fulfilled on nodes \"kube-prom01\": the object has been modified; please apply your changes to the latest version and try again"
Aug 07 00:11:58 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:58+08:00" level=info msg="Failed to set annotations and labels on node kube-prom01: Operation cannot be fulfilled on nodes \"kube-prom01\": the object has been modified; please apply your changes to the latest version and try again"
Aug 07 00:11:58 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:58+08:00" level=info msg="Failed to set annotations and labels on node kube-prom01: Operation cannot be fulfilled on nodes \"kube-prom01\": the object has been modified; please apply your changes to the latest version and try again"
Aug 07 00:11:58 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:58+08:00" level=info msg="Failed to set annotations and labels on node kube-prom01: Operation cannot be fulfilled on nodes \"kube-prom01\": the object has been modified; please apply your changes to the latest version and try again"
Aug 07 00:11:58 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:58+08:00" level=info msg="Annotations and labels have been set successfully on node: kube-prom01"
Aug 07 00:11:58 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:58+08:00" level=info msg="rke2 agent is up and running"
Aug 07 00:11:58 kube-prom01 systemd[1]: Started Rancher Kubernetes Engine v2 (agent).
Aug 07 00:11:58 kube-prom01 rke2[2179314]: time="2023-08-07T00:11:58+08:00" level=info msg="Running kube-proxy --cluster-cidr=10.42.0.0/16 --conntrack-max-per-core=0 --conntrack-tcp-timeout-close-wait=0s --conntrack-tcp-timeout-established=0s --healthz-bind-address=127.0.0.1 --hostname-override=kube-prom01 --kubeconfig=/var/lib/rancher/rke2/agent/kubeproxy.kubeconfig --proxy-mode=iptables"
Aug 07 00:14:02 kube-prom01 rke2[2179314]: time="2023-08-07T00:14:02+08:00" level=info msg="Tunnel authorizer set Kubelet Port 10250"

```

##  12. 检查集群

```bash
[root@kube-master01 ~]# kubectl get node 
NAME            STATUS   ROLES                       AGE     VERSION
kube-master01   Ready    control-plane,etcd,master   2d1h    v1.25.12+rke2r1
kube-node01     Ready    <none>                      26m     v1.25.12+rke2r1
kube-node02     Ready    <none>                      5m10s   v1.25.12+rke2r1
kube-node03     Ready    <none>                      21m     v1.25.12+rke2r1
kube-prom01     Ready    <none>                      43m     v1.25.12+rke2r1

```

## 13. 配置管理工具

```bash
ln -s /var/lib/rancher/rke2/bin/ctr /usr/bin/ctr
ln -s /var/lib/rancher/rke2/bin/crictl  /usr/bin/crictl
crictl --runtime-endpoint /run/k3s/containerd/containerd.sock images
crictl --runtime-endpoint /run/k3s/containerd/containerd.sock pull ghcr.io/cloudtty/cloudshell:v0.4.0
cat >> /root/.bashrc <<EOF
export CONTAINERD_HTTP_PROXY=http://192.168.21.101:7890
export CONTAINERD_HTTPS_PROXY=http://192.168.21.101:7890
export CONTAINERD_NO_PROXY=127.0.0.0/8,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16,.svc,.cluster.local
EOF
```

##  14. 卸载
[https://docs.rke2.io/install/uninstall](https://docs.rke2.io/install/uninstall)

rpm

```bash
/usr/bin/rke2-uninstall.sh
```
Tarball Method

```bash
/usr/local/bin/rke2-uninstall.sh
```


参考：

- [Rocky Linux 9.1 新手入门指南](https://blog.csdn.net/xixihahalelehehe/article/details/129644123)
- [centos 8.2 指南](https://blog.csdn.net/xixihahalelehehe/article/details/127641551)
- [centos linux 配置私有网段并联网](https://ghostwritten.blog.csdn.net/article/details/130607628)
- [https://docs.rke2.io/install/quickstart](https://docs.rke2.io/install/quickstart)
- [https://docs.rke2.io/advanced](https://docs.rke2.io/advanced)
