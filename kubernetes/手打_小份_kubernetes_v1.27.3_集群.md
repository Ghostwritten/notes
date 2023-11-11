
![](https://img-blog.csdnimg.cn/d5eeed341fef4c38b82f1e5992c41c78.png)




## 1. 准备
- rocky linux 8.8
- CPU 4
- 内存 8G
- disk 40G


- 192.168.23.11    kube-master01
- 192.168.23.14    kube-prom01
- 192.168.23.21    kube-node01
## 2. yum
（kube-master01、kube-prom01、 kube-node01操作）
```bash
dnf -y update
dnf -y install  iproute-tc wget vim socat wget bash-completion net-tools zip bzip2 bind-utils
```

## 3. 安装 ansible
（kube-master01操作）
```bash
dnf -y install epel-relese
dnf -y install ansible
```
配置 

```bash
$ cat /etc/ansible/hosts 
[all]
kube-master01 ansible_host=192.168.23.11 
kube-prom01 ansible_host=192.168.23.14
kube-node01 ansible_host=192.168.23.21

```

## 4. 互信
（kube-master01操作）

```bash
ssh-keygen
ssh-copy-id root@192.168.23.11
ssh-copy-id root@192.168.23.14
ssh-copy-id root@192.168.23.21
```
测试

```bash
[root@kube-master01 ~]# ansible all -m ping
kube-master01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
kube-node01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
kube-prom01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}

```

## 5. hosts

```bash
$ vim /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.23.11 kube-master01
192.168.23.14 kube-prom01
192.168.23.21 kube-node01

```

```bash
ansible all -m copy -a "src=/etc/hosts dest=/etc/hosts"
```

## 6. 关闭防火墙、swap、selinux

```bash
ansible all -i hosts -s -m systemd -a "name=firewalld state=stopped enabled=no"
ansible all -m lineinfile -a "path=/etc/selinux/config regexp='^SELINUX=' line='SELINUX=disabled'" -b
ansible all -m shell -a "getenforce 0"
ansible all -m shell -a "sed -i '/.*swap.*/s/^/#/' /etc/fstab" -b
ansible all -m shell -a " swapoff -a && sysctl -w vm.swappiness=0"
```
## 7. 配置系统文件句柄数
```bash
ansible all -m lineinfile -a "path=/etc/security/limits.conf line='* soft nofile 655360\n* hard nofile 131072\n* soft nproc 655350\n* hard nproc 655350\n* soft memlock unlimited\n* hard memlock unlimited'" -b
```

## 8. 启用ipvs

```bash
ansible all -m file -a "path=/etc/modules-load.d state=directory" -b
ansible all -m file -a "path=/etc/modules-load.d/ipvs.conf state=touch mode=0644"
ansible all -m lineinfile -a "path=/etc/rc.local line='modprobe br_netfilter\nmodprobe ip_conntrack'" -b
ansible all  -m systemd -a "name=systemd-modules-load.service state=restarted"
```

## 9. 修改内核参数

```bash
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
```

## 10. 安装 containerd

```bash
wget https://github.com/containerd/containerd/releases/download/v1.7.2/cri-containerd-cni-1.7.2-linux-amd64.tar.gz
ansible all -m copy -a "src=cri-containerd-cni-1.7.2-linux-amd64.tar.gz dest=/tmp force=yes"
ansible all -m shell -a "tar -C / -xzf /tmp/cri-containerd-cni-1.7.2-linux-amd64.tar.gz"
ansible all -m file -a "path=/etc/containerd state=directory" -b
ansible all -m shell -a "containerd config default > /etc/containerd/config.toml"
ansible all -m shell -a "sed -i '/mirrors/a\        [plugins.\"io.containerd.grpc.v1.cri\".registry.mirrors.\"docker.io\"]' /etc/containerd/config.toml" 
ansible all -m shell -a  "sed -i '/docker.io/a\          endpoint = [\"https://je0sfs52.mirror.aliyuncs.com\"]' /etc/containerd/config.toml "
ansible all -m shell -a "cat /etc/containerd/config.toml|grep -C 3 docker.io"
ansible all  -m shell -a "systemctl daemon-reload && systemctl enable --now containerd"

```



## 11. 安装nerdctl

```bash
wget https://github.com/containerd/nerdctl/releases/download/v1.4.0/nerdctl-1.4.0-linux-amd64.tar.gz
ansible all -m copy -a "src=nerdctl-1.4.0-linux-amd64.tar.gz dest=/tmp force=yes"
ansible all -m shell -a "tar -C /tmp/ -zxf  /tmp/nerdctl-1.4.0-linux-amd64.tar.gz && mv /tmp/nerdctl /usr/bin/ "
ansible all -m shell -a "nerdctl ps"
```

## 12. kubernetes yum

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
ansible all -m copy -a "src=/etc/yum.repos.d/kubernetes.repo dest=/etc/yum.repos.d/kubernetes.repo force=yes"
ansible all -m shell -a "dnf -y update && dnf -y upgrade"
```

## 13. 部署 kubernetes

### 13.1 安装工具
```bash
#ansible all  -m yum -a "name=kubelet-1.27.3-00 kubeadm-1.27.3-00 kubectl-1.27.3-00 state=present"
ansible all  -m dnf -a "name=kubelet-1.27.3-00  state=present"
ansible all  -m dnf -a "name=kubeadm-1.27.3-00  state=present"
ansible all  -m dnf -a "name=kubectl-1.27.3-00  state=present"
```

### 13.2 初始化配置
生成集群初始化文件

```bash
kubeadm config print init-defaults --component-configs KubeletConfiguration > kubeadm.yaml
```
查看所需的镜像

```bash
$ kubeadm config images list --config kubeadm.yaml
registry.k8s.io/kube-apiserver:v1.27.0
registry.k8s.io/kube-controller-manager:v1.27.0
registry.k8s.io/kube-scheduler:v1.27.0
registry.k8s.io/kube-proxy:v1.27.0
registry.k8s.io/pause:3.9
registry.k8s.io/etcd:3.5.7-0
registry.k8s.io/coredns/coredns:v1.10.1
```
修改安装 kubernetes 版本

```bash
sed -i "s/kubernetesVersion: .*/kubernetesVersion: v1.27.3/g" kubeadm.conf
```


注意这个配置文件默认会`registry.k8s.io`下载镜像，如果你没有科学上网，那么就会下载失败。

配置代理

```bash
ansible all -m file -a "path=/etc/systemd/system/containerd.service.d/ state=directory" 
ansible all -m file -a "path=/etc/systemd/system/containerd.service.d/http-proxy.conf state=touch" 
cat <<EOF>> /etc/systemd/system/containerd.service.d/http-proxy.conf
[Service]
Environment="HTTP_PROXY=http://192.168.21.101:7890"
Environment="HTTPS_PROXY=http://192.168.21.101:7890"
Environment="NO_PROXY=localhost"
EOF
ansible all -m copy -a "src=/etc/systemd/system/containerd.service.d/http-proxy.conf dest=/etc/systemd/system/containerd.service.d/http-proxy.conf force=yes"
ansible all -m shell -a "cat /etc/systemd/system/containerd.service.d/http-proxy.conf "
ansible all  -m systemd -a "name=containerd state=restarted"

```


## 14. 部署 master

```bash
$ kubeadm init --kubernetes-version=v1.27.3 --pod-network-cidr=10.96.0.0/12 --apiserver-advertise-address=192.168.23.11
[init] Using Kubernetes version: v1.27.3
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kube-master01 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.23.11]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [kube-master01 localhost] and IPs [192.168.23.11 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [kube-master01 localhost] and IPs [192.168.23.11 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 8.505670 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node kube-master01 as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node kube-master01 as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: 9w826t.frumayd3p16t1jsd
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.23.11:6443 --token 9w826t.frumayd3p16t1jsd \
	--discovery-token-ca-cert-hash sha256:0e80a963060663fbebd413084ffff7cdfc0faa2005d268514a2dab6449f363e2
```

## 15. 部署 node

```bash
$ kubeadm join 192.168.23.11:6443 --token 9w826t.frumayd3p16t1jsd \
> --discovery-token-ca-cert-hash sha256:0e80a963060663fbebd413084ffff7cdfc0faa2005d268514a2dab6449f363e2
[preflight] Running pre-flight checks
	[WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

```

## 16. 检查

```bash
$ kubectl get node
NAME            STATUS   ROLES           AGE     VERSION
kube-master01   Ready    control-plane   2m56s   v1.27.3
kube-node01     Ready    <none>          7s      v1.27.3
kube-prom01     Ready    <none>          72s     v1.27.3
$ kubectl version
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.3", GitCommit:"25b4e43193bcda6c7328a6d147b1fb73a33f1598", GitTreeState:"clean", BuildDate:"2023-06-14T09:53:42Z", GoVersion:"go1.20.5", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v5.0.1
Server Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.3", GitCommit:"25b4e43193bcda6c7328a6d147b1fb73a33f1598", GitTreeState:"clean", BuildDate:"2023-06-14T09:47:40Z", GoVersion:"go1.20.5", Compiler:"gc", Platform:"linux/amd64"}
$ kubectl get pod -A
NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE
kube-system   coredns-5d78c9869d-lwvp5                1/1     Running   0          2m52s
kube-system   coredns-5d78c9869d-lz7h6                1/1     Running   0          2m52s
kube-system   etcd-kube-master01                      1/1     Running   0          3m6s
kube-system   kube-apiserver-kube-master01            1/1     Running   0          3m4s
kube-system   kube-controller-manager-kube-master01   1/1     Running   0          3m4s
kube-system   kube-proxy-cgx7w                        1/1     Running   0          20s
kube-system   kube-proxy-q87zt                        1/1     Running   0          2m52s
kube-system   kube-proxy-rb7k2                        1/1     Running   0          85s
kube-system   kube-scheduler-kube-master01            1/1     Running   0          3m8s

```

参考：

- [https://kubernetes.io/releases/download/](https://kubernetes.io/releases/download/)
- [https://www.downloadkubernetes.com/](https://www.downloadkubernetes.com/)
