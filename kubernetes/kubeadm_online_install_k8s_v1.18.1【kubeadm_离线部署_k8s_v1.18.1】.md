
---
***1.文章经常遇到bug问题的地方保持返回输出，方便您的理解。***
***2.为方便您的复制粘贴，无返回输出以及简单返回输出的命令前无标识符，有返回输出的命令前以“$”标识。***
***3.为方便你的深入理解，部分带有颜色的文字为超链接。***
***4.为方便你的理解，本文以一步一步部署为主，当然，如果想快速部署，请点击。***
***5.不要担心文字繁多，你可以尽可能复制部署命令即可，这样不会浪费太多时间。***

---
Kubernetes 版本查阅地址： [https://github.com/kubernetes/kubernetes/releases](https://github.com/kubernetes/kubernetes/releases)



---
## 1. 实践环境准备

 - 服务器虚拟机准备
IP地址 节点角色 CPU Memory Hostname
 - 192.168.211.11 master and  etcd >=2c >=2G master
 - 192.168.211.12 worker >=2c >=2G node

本实验我这里用的VM是vmware workstation创建的，每个给了2C 2G 20GB配置，大家根据自己的资 源情况，按照上面给的建议最低值创建即可

**注意**：**hostname不能有大写字母**，比如Master这样。

## 2. 软件版本

 - 系统类型：centos 7.4 1708
 - 内核版本: Linux 3.10.0-957.1.3.el7.x86_64
 - Kubernetes版 本：v1.18.1
 - docker版本：19.03.5
 - kubeadm版本：v1.18.1
 - kubectl版本：Client Version v1.18.1  Server Versionv1.18.0
 - kubelet版本：v1.18.1

## 3. 初始化操作
### 3.1 配置hostname 

```bash
hostnamectl set-hostname master 
hostnamectl set-hostname node
```
### 3.2 配置/etc/hosts
```bash
cat <<EOF > /etc/hosts 
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 
::1        localhost localhost.localdomain localhost6 localhost6.localdomain6 
192.168.211.11 master 
192.168.211.12 node 
EOF
```
### 3.3 关闭防火墙、selinux、swap 

```bash
setenforce  0
sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/sysconfig/selinux
sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
sed -i "s/^SELINUX=permissive/SELINUX=disabled/g" /etc/sysconfig/selinux
sed -i "s/^SELINUX=permissive/SELINUX=disabled/g" /etc/selinux/config

systmectl stop firewalld
systmectl disable firewalld

swapoff -a
sed -i 's/.*swap.*/#&/' /etc/fstab
```
### 3.4  加载br_netfilter 
平台CLI检查br_netfilter模块是否已加载，如果模块不可用则退出。需要此模块以启用透明伪装并促进虚拟可扩展LAN（VxLAN）流量，以在整个集群中的Kubernetes Pod之间进行通信。如果需要检查是否已加载，请运行：

```bash
$ sudo lsmod|grep br_netfilter
br_netfilter 24576 0 
桥155648 2 br_netfilter，ebtable_broute
```

如果看到类似于所示的输出，则 br_netfilter模块已加载。内核模块通常在需要时加载，因此不太可能需要手动加载此模块。如有必要，您可以手动加载模块并通过运行以下命令将其添加为永久模块：
```bash
sudo modprobe br_netfilter
sudo sh -c 'echo "br_netfilter" > /etc/modules-load.d/br_netfilter.conf'
```
简而言之，**开启内核ipv4转发需要加载br_netfilter模块**
### 3.5 配置内核参数 

```bash
cat > /etc/sysctl.d/k8s.conf <<EOF 
net.bridge.bridge-nf-call-ip6tables = 1 
net.bridge.bridge-nf-call-iptables = 1 
EOF
sysctl -p /etc/sysctl.d/k8s.conf
echo "* soft nofile 655360" >> /etc/security/limits.conf
echo "* hard nofile 655360" >> /etc/security/limits.conf
echo "* soft nproc 655360"  >> /etc/security/limits.conf
echo "* hard nproc 655360"  >> /etc/security/limits.conf
echo "* soft  memlock  unlimited"  >> /etc/security/limits.conf
echo "* hard memlock  unlimited"  >> /etc/security/limits.conf
echo "DefaultLimitNOFILE=1024000"  >> /etc/systemd/system.conf
echo "DefaultLimitNPROC=1024000"  >> /etc/systemd/system.conf
```
执行看下最大文件打开数是否是`655360` ，如果不是，**尝试重新连接终端**。

```bash
 ulimit -Hn
```

###  3.6 配置CentOS YUM源 
配置国内tencent yum源地址、epel源地址、国内Kubernetes源地址 
```bash
yum install -y wget
rm -rf  /etc/yum.repos.d/*
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.cloud.tencent.com/repo/centos7_base.repo
wget -O /etc/yum.repos.d/epel.repo http://mirrors.cloud.tencent.com/repo/epel-7.repo
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg  
EOF
yum clean all && yum makecache
```
### 3.7 安装依赖软件包 

```bash
yum install  -y conntrack ipvsadm ipset jq sysstat curl iptables libseccomp bash-completion yum-utils device-mapper-persistent-data lvm2 net-tools conntrack-tools vim libtool-ltdl
```

### 3.8 时间同步配置 
Kubernetes是分布式的，各个节点系统时间需要同步对应上。 

```bash
yum -y install chrony
systemctl enable chronyd.service && systemctl start chronyd.service && systemctl status chronyd.service
chronyc sources
```
运行date命令看下系统时间，过一会儿时间就会同步

 - [ ] 时间如果不同步会遇到什么？

###  3.9 配置节点间ssh互信 
配置ssh互信，那么节点之间就能无密访问，方便日后执行自动化部署 

```bash
ssh-keygen     # 每台机器执行这个命令， 一路回车即可 
ssh-copy-id  node      # 到master上拷贝公钥到其他节点，这里需要输入 yes和密码
```

 - [ ] 如果只是master与其他节点做了互信，会有影响吗？

### 3.10 初始化环境配置检查 
- 重启，做完以上所有操作，最好reboot重启一遍 
- ping 每个节点hostname 看是否能ping通 
- ssh 对方hostname看互信是否无密码访问成功 
- 执行date命令查看每个节点时间是否正确 
- 执行 ulimit -Hn 看下最大文件打开数是否是655360 
- cat /etc/sysconfig/selinux |grep disabled 查看下每个节点selinux是否都是disabled状态

## 4.部署docker
### 4.1 remove旧版本docker

```bash
yum remove -y docker docker-ce docker-common docker-selinux docker-engine
```

### 4.2 设置docker yum源 

```bash
yum-config-manager  --add-repo https://download.docker.com/linux/centos/dockerce.repo
```

### 4.3 列出docker版本

```bash
yum list docker-ce --showduplicates | sort -r
```
### 4.4 安装docker 指定18.06.1

```bash
yum install -y docker-ce-19.03.5-3.el7.x86_64
```

### 4.5 配置镜像加速器和docker数据存放路径

```bash
tee /etc/docker/daemon.json <<-'EOF'
{  
  "registry-mirrors": ["https://q2hy3fzi.mirror.aliyuncs.com"], 
  "graph": "/tol/docker-data" 
} 
EOF
```
### 4.6 启动docker

```bash
systemctl daemon-reload && systemctl restart docker && systemctl enable docker && systemctl status docker
```

## 5 安装kubeadm、kubelet、kubectl

这一步是所有节点都得安装（包括node节点）
工具说明 

 - kubeadm: 部署集群用的命令
 - kubelet: 在集群中每台机器上都要运行的组件，负责管理pod、容器的生命周期
 - kubectl: 集群管理工具

 ### 5.1 yum 安装 并启动

```bash
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable kubelet && systemctl start kubelet
```
**注意：kubelet 服务会暂时启动不了，先不用管它。**

## 6 镜像下载准备
### 6.1 初始化获取要下载的镜像列表
 使用kubeadm来搭建Kubernetes，那么就需要下载得到Kubernetes运行的对应基础镜像，比如`：kubeproxy、kube-apiserver、kube-controller-manager`等等 。那么有什么方法可以得知要下载哪些镜像 呢？从`kubeadm v1.11+`版本开始，增加了一个`kubeadm config print-default` 命令，可以让我们方便 的将kubeadm的默认配置输出到文件中，这个文件里就包含了搭建K8S对应版本需要的基础配置环境。另 外，我们也可以执行 `kubeadm config images list` 命令查看依赖需要安装的镜像列表。

```bash
$  kubeadm config images list
W0417 00:57:50.788202  129664 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
k8s.gcr.io/kube-apiserver:v1.18.1
k8s.gcr.io/kube-controller-manager:v1.18.1
k8s.gcr.io/kube-scheduler:v1.18.1
k8s.gcr.io/kube-proxy:v1.18.1
k8s.gcr.io/pause:3.2
k8s.gcr.io/etcd:3.4.3-0
k8s.gcr.io/coredns:1.6.7
```
**注意**：这个列表显示的tag名字和镜像版本号，从`Kubernetes v1.12+`开始，镜像名后面不带 `amd64, arm, arm64, ppc64le` 这样的标识了

### 6.2 生成默认kubeadm.conf文件 

```bash
$ kubeadm config print init-defaults > kubeadm.conf 
```

### 6.3 绕过墙下载镜像方法 
注意这个配置文件默认会从google的镜像仓库地址k8s.gcr.io下载镜像，如果你没有科学上网，那么就会 下载不来。因此，我们通过下面的方法把地址改成国内的，比如用阿里的：

```bash
sed -i "s/imageRepository: .*/imageRepository: registry.aliyuncs.com\/google_containers/g" kubeadm.conf
```
### 6.4 指定kubeadm安装的Kubernetes版本

```bash
sed -i "s/kubernetesVersion: .*/kubernetesVersion: v1.18.1/g" kubeadm.conf
```

### 6.5 下载需要用到的镜像 
kubeadm.conf修改好后，我们执行下面命令就可以自动从国内下载需要用到的镜像了： 

```bash
$ kubeadm config images pull --config kubeadm.conf
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-apiserver:v1.18.1
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-controller-manager:v1.18.1
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-scheduler:v1.18.1
[config/images] Pulled registry.aliyuncs.com/google_containers/kube-proxy:v1.18.1
[config/images] Pulled registry.aliyuncs.com/google_containers/pause:3.2
[config/images] Pulled registry.aliyuncs.com/google_containers/etcd:3.4.3-0
[config/images] Pulled registry.aliyuncs.com/google_containers/coredns:1.6.7
```

自动下载v1.81需要用到的镜像，执行 docker images 可以看到下载好的镜像列表

```bash
$ docker images 
registry.aliyuncs.com/google_containers/kube-proxy                v1.18.1             4e68534e24f6        8 days ago          117MB
registry.aliyuncs.com/google_containers/kube-controller-manager   v1.18.1             d1ccdd18e6ed        8 days ago          162MB
registry.aliyuncs.com/google_containers/kube-apiserver            v1.18.1             a595af0107f9        8 days ago          173MB
registry.aliyuncs.com/google_containers/pause                     3.2                 80d28bedfe5d        2 months ago        683kB
registry.aliyuncs.com/google_containers/etcd                      3.4.3-0             303ce5db0e90        5 months ago        288MB
registry.aliyuncs.com/google_containers/coredns                   1.6.7               67da37a9a360        2 months ago        43.8MB
```
### 6.6 docker tag 镜像
镜像下载好后，我们还需要tag下载好的镜像，让下载好的镜像都是带有 k8s.gcr.io 标识的，目前我们从阿 里下载的镜像 标识都是，如果不打tag变成k8s.gcr.io，那么后面用kubeadm安装会出现问题，因为 kubeadm里面只认 google自身的模式。我们执行下面命令即可完成tag标识更换： 

```bash
$ cat tag.sh 
#!/bin/bash

newtag=k8s.gcr.io
for i in $(docker images | grep -v TAG |awk '{print $1 ":" $2}')
do
   image=$(echo $i | awk -F '/' '{print $3}')
   docker tag $i $newtag/$image
   docker rmi $i
done
$ bash tag.sh
```
## 6.7 查看修改tag后的镜像列表

```bash
k8s.gcr.io/kube-proxy                             v1.18.1             4e68534e24f6        8 days ago          117MB
k8s.gcr.io/kube-controller-manager                v1.18.1             d1ccdd18e6ed        8 days ago          162MB
k8s.gcr.io/kube-apiserver                         v1.18.1             a595af0107f9        8 days ago          173MB
k8s.gcr.io/kube-scheduler                         v1.18.1             6c9320041a7b        8 days ago          95.3MB
k8s.gcr.io/pause                     3.2                 80d28bedfe5d        2 months ago        683kB
k8s.gcr.io/coredns                   1.6.7               67da37a9a360        2 months ago        43.8MB
k8s.gcr.io/etcd                      3.4.3-0             303ce5db0e90        5 months ago        288MB
```
**注意**：另外节点，重复上面的操作下载即可。


## 7 部署master节点
### 7.1  kubeadm init 初始化master节点

```bash
kubeadm init --kubernetes-version=v1.18.1 --pod-network-cidr=172.22.0.0/16 --apiserver-advertise-address=192.168.211.11
```
这里我们定义POD的网段为: `172.22.0.0/16`  ，然后api server地址就是master本机IP地址
#### 7.1.1 初始化报错
[1.Switch cgroup driver, from cgroupfs to systemd](https://github.com/kubernetes/minikube/issues/4770) 
```c
[root@master ~]# kubeadm init --kubernetes-version=v1.18.0 --pod-network-cidr=172.22.0.0/16 --apiserver-advertise-address=192.168.211.11
W0416 17:58:21.037057   11830 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[init] Using Kubernetes version: v1.18.0
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR NumCPU]: the number of available CPUs 1 is less than the required 2
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```
解决方法：

```bash
$ cat /etc/docker/daemon.json 
{  
  "exec-opts": ["native.cgroupdriver=systemd"],   #添加此行
  "registry-mirrors": ["https://q2hy3fzi.mirror.aliyuncs.com"], 
  "graph": "/tol/docker-data" 
} 
$ systemctl restart docker
```

####  7.1.2 初始化成功
```bash
$ kubeadm init --kubernetes-version=v1.18.0 --pod-network-cidr=172.22.0.0/16 --apiserver-advertise-address=192.168.211.11
W0416 18:07:20.031658   12701 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[init] Using Kubernetes version: v1.18.0
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.211.11]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [master localhost] and IPs [192.168.211.11 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [master localhost] and IPs [192.168.211.11 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
W0416 18:07:25.968262   12701 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[control-plane] Creating static Pod manifest for "kube-scheduler"
W0416 18:07:25.969958   12701 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 20.507370 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.18" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node master as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node master as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: 16l83a.e1tpcgkmze0i3fuy
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:
#保存初始化node用到的命令
kubeadm join 192.168.211.11:6443 --token 16l83a.e1tpcgkmze0i3fuy \
    --discovery-token-ca-cert-hash sha256:233a3d9c6c0ed466642c08293e0bf2bb217359d414d3ccb0bf25afa1c00b7ca3 
```

### 7.2  验证测试 

配置kubectl命令 

```bash
mkdir -p /root/.kube
cp /etc/kubernetes/admin.conf /root/.kube/config
```
获取pods列表命令
其中coredns pod处于`Pending`状态，**这个先不管**
```bash
kubectl get pods --all-namespaces
```
查看集群的健康状态：

```bash
$ kubectl get cs 
etcd-0               Healthy   {"health":"true"}   
controller-manager   Healthy   ok                  
scheduler            Healthy   ok     
```
---
## 8 部署calico网络 （在master上执行）
[calico官网](https://docs.projectcalico.org/v3.0/introduction/)
[calico详细介绍](https://www.lijiaocn.com/%E9%A1%B9%E7%9B%AE/2017/04/11/calico-usage.html)
calico介绍：Calico是一个纯三层的方案，其好处是它整合了各种云原生平台(Docker、Mesos 与 OpenStack 等)，每个 Kubernetes 节点上通过 Linux Kernel 现有的 L3 forwarding 功能来实现 vRouter 功能。

###  8.1calico官方镜像 下载、标签修改、删除

```bash
docker pull calico/node:v3.1.4
docker pull calico/cni:v3.1.4
docker pull calico/typha:v3.1.4
docker tag calico/node:v3.1.4 quay.io/calico/node:v3.1.4
docker tag calico/cni:v3.1.4 quay.io/calico/cni:v3.1.4
docker tag calico/typha:v3.1.4 quay.io/calico/typha:v3.1.4
docker rmi calico/node:v3.1.4
docker rmi calico/cni:v3.1.4
docker rmi calico/typha:v3.1.4
```
### 8.2 下载执行rbac-kdd.yaml文件
```bash
curl https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml -O
kubectl apply -f rbac-kdd.yaml
```
### 8.3 下载、配置calico.yaml文件 

```bash
curl https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/kubernetes-datastore/policy-only/1.7/calico.yaml -O
```
### 8.4 修改配置
#### 8.4.1 修改ConfigMap 
把ConfigMap 下的 typha_service_name 值由none变成 calico-typha

```bash
kind: ConfigMap
apiVersion: v1
metadata:
  name: calico-config
  namespace: kube-system
data:
  # You must set a non-zero value for Typha replicas below.
  typha_service_name: "calico-typha"
```
#### 8.4.2 修改Deployment 
设置 Deployment 类目的 spec 下的replicas值

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: calico-typha
  namespace: kube-system
  labels:
    k8s-app: calico-typha
spec:
  # Number of Typha replicas.  To enable Typha, set this to a non-zero value *and* set the
  # typha_service_name variable in the calico-config ConfigMap above.
  #
  # We recommend using Typha if you have more than 50 nodes.  Above 100 nodes it is essential
  # (when using the Kubernetes datastore).  Use one replica for every 100-200 nodes.  In
  # production, we recommend running at least 3 replicas to reduce the impact of rolling upgrade.
  replicas: 1
  revisionHistoryLimit: 2
```
#### 8.4.3 修改 定义POD网段
我们找到CALICO_IPV4POOL_CIDR，然后值修改成之前定义好的POD网段，我这里是172.22.0.0/16

```bash
- name: CALICO_IPV4POOL_CIDR
  value: "172.22.0.0/16"
```
#### 8.4.4 开启bird模式 
把 CALICO_NETWORKING_BACKEND 值设置为 bird ，这个值是设置BGP网络后端模式

```bash
 - name: CALICO_NETWORKING_BACKEND
   value: "bird"
```
### 8.5 部署calico.yaml文件 
 上面参数设置调优完毕，我们执行下面命令彻底部署calico 

```bash
 kubectl apply -f calico.yaml
```
### 8.6 查看状态

```bash
kubectl get pods --all-namespaces
```
这里calico-typha 没起来，那是因为我们的node 计算节点还没启动和安装。



## 9 部署node节点
### 9.1 下载安装镜像（在node上执行） 
node上也是需要下载安装一些镜像的，需要下载的镜像为：`kube-proxy:v1.13、pause:3.1、caliconode:v3.1.4、calico-cni:v3.1.4、calico-typha:v3.1.4`

```bash
docker pull registry.aliyuncs.com/google_containers/kube-proxy:v1.13.0
docker pull registry.aliyuncs.com/google_containers/pause:3.1
docker pull calico/node:v3.1.4
docker pull calico/cni:v3.1.4
docker pull calico/typha:v3.1.4

docker tag registry.aliyuncs.com/google_containers/kube-proxy:v1.13.0 k8s.gcr.io/kubeproxy:v1.13.0
docker tag registry.aliyuncs.com/google_containers/pause:3.1 k8s.gcr.io/pause:3.1
docker tag calico/node:v3.1.4 quay.io/calico/node:v3.1.4
docker tag calico/cni:v3.1.4 quay.io/calico/cni:v3.1.4
docker tag calico/typha:v3.1.4 quay.io/calico/typha:v3.1.4

docker rmi registry.aliyuncs.com/google_containers/kube-proxy:v1.13.0
docker rmi registry.aliyuncs.com/google_containers/pause:3.1
docker rmi calico/node:v3.1.4
docker rmi calico/cni:v3.1.4
docker rmi calico/typha:v3.1.4
```
### 9.2  把node加入集群里 （kubeadm init 生成）

```c
[root@node ~]# kubeadm join 192.168.211.11:6443 --token 16l83a.e1tpcgkmze0i3fuy \
>     --discovery-token-ca-cert-hash sha256:233a3d9c6c0ed466642c08293e0bf2bb217359d414d3ccb0bf25afa1c00b7ca3 
W0416 21:07:26.704823   21506 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.18" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```
两个节点运行的参数命令一样，运行完后，我们在master节点上运行 kubectl get nodes 命令查看node 是否正常

```bash
kubectl get nodes
```


如何我们忘了保存 `kubeadm init` 生成的 `kubeadm join` 命令，可以通过以下命令显示：

```bash
$ kubeadm token create --print-join-command
kubeadm join 127.0.0.1:6443 --token fzqdec.hsq4iggqbb1l31mr --discovery-token-ca-cert-hash sha256:b6f897fa33f2847841c8f1f18045c56e0f33dd2c8476044f82270067a39d96df
```

到此，集群的搭建完成了90%，剩下一个是搭建dashboard


## 10 部署dashboard
部署dashboard之前，我们需要生成证书，不然后面会https访问登录不了。
### 10.1 生成私钥和证书签名请求 
```c
[root@master ~]# mkdir -p /etc/kubernetes/certs
[root@master ~]# cd /etc/kubernetes/certs
[root@master certs]# ls
[root@master certs]# openssl genrsa -des3 -passout pass:x -out dashboard.pass.key 2048
Generating RSA private key, 2048 bit long modulus
.................+++
..................................+++
e is 65537 (0x10001)
[root@master certs]# ll
总用量 4
-rw-r--r--. 1 root root 1751 4月  16 21:15 dashboard.pass.key
[root@master certs]# openssl rsa -passin pass:x -in dashboard.pass.key -out dashboard.key
writing RSA key
[root@master certs]# ll
总用量 8
-rw-r--r--. 1 root root 1679 4月  16 21:16 dashboard.key
-rw-r--r--. 1 root root 1751 4月  16 21:15 dashboard.pass.key
[root@master certs]# rm -rf dashboard.pass.key 
[root@master certs]# openssl req -new -key dashboard.key -out dashboard.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:
State or Province Name (full name) []:
Locality Name (eg, city) [Default City]:
Organization Name (eg, company) [Default Company Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
[root@master certs]# ll
总用量 8
-rw-r--r--. 1 root root  952 4月  16 21:18 dashboard.csr
-rw-r--r--. 1 root root 1679 4月  16 21:16 dashboard.key
```

生成SSL证书 

```bash
openssl x509 -req -sha256 -days 365 -in dashboard.csr -signkey dashboard.key -out dashboard.crt
```

dashboard.crt文件是适用于仪表板和dashboard.key私钥的证书。

```bash
$  ll /etc/kubernetes/certs/
总用量 12
-rw-r--r--. 1 root root 1103 4月  16 21:20 dashboard.crt
-rw-r--r--. 1 root root  952 4月  16 21:18 dashboard.csr
-rw-r--r--. 1 root root 1679 4月  16 21:16 dashboard.key
```
创建secret 

```bash
kubectl create secret generic kubernetes-dashboard-certs --from-file=/etc/kubernetes/certs -n kube-system
```
注意/etc/kubernetes/certs 是之前创建crt、csr、key 证书文件存放的路径


### 10.2 下载dashboard镜像、tag镜像（在全部节点上）

```bash
$ vim dashboard.sh
#!/bin/bash
tag=v2.0.0-rc7
docker pull registry.cn-hangzhou.aliyuncs.com/kubernete/kubernetes-dashboard-amd64:$tag
docker tag registry.cn-hangzhou.aliyuncs.com/kubernete/kubernetes-dashboard-amd64:v$tag k8s.gcr.io/kubernetes-dashboard:$tag
docker rmi registry.cn-hangzhou.aliyuncs.com/kubernete/kubernetes-dashboard-amd64:$tag
$ bash dashboard.sh
```
### 10. 3 下载 kubernetes-dashboard.yaml 部署文件（在master上执行） 

```bash
curl https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml -O
```
### 10.4 修改配置

#### 10.4.1 修改镜像版本

```bash
sed -i "s/kubernetes-dashboard-amd64:v1.10.0/kubernetes-dashboard:$tag/g" kubernetes-dashboard.yaml
```
#### 10.4.2  把Secret 注释 
因为上面我们已经生成了密钥认证了，我们用我们自己生成的。

```bash
#apiVersion: v1
#kind: Secret
#metadata:
#  labels:
#    k8s-app: kubernetes-dashboard
#  name: kubernetes-dashboard-certs	
#  namespace: kubernetes-dashboard
#type: Opaque
#
#---
```
#### 10.4.3  配置443端口映射到外部主机30005上

```bash
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30005  #添加
  type: NodePort     #添加
  selector:
    k8s-app: kubernetes-dashboard
```
 ### 10.5 创建dashboard的pod 

```bash
  kubectl create -f  kubernetes-dashboard.yaml
```

###  10.6 查看服务运行情况

```bash
 kubectl get deployment kubernetes-dashboard -n kube-system
 kubectl --namespace kubesystem get pods -o wide
 kubectl get services kubernetes-dashboard -n kube-system 
 netstat -ntlp|grep 30005
```

### 10.7 Dashboard BUG处理
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/56aa615d585fbed0c0b12797c1acacd7.png)
点击跳过
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a8754583411d2a933a4e8ebed4611f0d.png)

```bash
$ vim kube-dashboard-access.yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: kubernetes-dashboard
  labels: 
    k8s-app: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system
```
执行让kube-dashboard-access.yaml 

```bash
 kubectl create -f kube-dashboard-access.yaml
```
然后重新登录界面刷新下，解决问题，如果产生一下问题。
问题
[Server Error 404. It will give you 3 seconds to redirect back to the login page. 
Kubernetes dashboard the server could not find the requested resource](https://github.com/kubernetes-sigs/kubespray/issues/5347)


问题原因：dashboard版本过低所致。
将dashboard版本升级
Deleted version `1.10.1` and switched to `v2.0.0-beta8` 
然后重新登录界面刷新下。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b2e89a1cb0ac18f1df1d1d9f131b514b.png)






