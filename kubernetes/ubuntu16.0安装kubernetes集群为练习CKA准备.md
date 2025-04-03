## 报名
CKA报名：[https://training.linuxfoundation.cn/](https://training.linuxfoundation.cn/)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5a86443c822686aa996119cb39aa3725.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5155f215dfb14ec0ac8d63ad29a22ca5.png)
时间非常紧

## 考试注意事项
要保持头再摄像头范围内，让监考老师看到
要保持界面简洁
只能带透明水杯
可以上厕所，但要注意时间

## 考试流程
jumpserver 通过公网ip远程，通过私网ip远程其他机器。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5d7b41d3045e3daf8a2df172e27347fd.png)

## 基础环境配置（all）

```css
swapoff -a
```

```css
cat <<EOF> /etc/hosts
192.168.211.40 master
192.168.211.41 node1
192.168.211.42 node2
EOF
```

## 安装docker（all）

如果你觉得默认`ubutu默认源`慢，可以更换`aliyun`源

```css
$ vim /etc/apt/sources.list
```

```css
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
```

```css
$ apt-get update
```

### 手动安装（all）

[https://www.runoob.com/docker/ubuntu-docker-install.html](https://www.runoob.com/docker/ubuntu-docker-install.html)

**注意**：**安装docker一定要指定稳定得版本，否则，会出现兼容性稳定，我们不求新而求稳。**

```css
$ apt-get install -y docker-ce=5:19.03.4~3-0~ubuntu-xenial docker-ce-cli=5:19.03.4~3-0~ubuntu-xenial containerd.io=1.2.10-3
```

问题：

```css
Failed to fetch cdrom://Ubuntu-Server 16.04.6 LTS _Xenial Xerus_ - Release amd64 (20190226)/dists/xenial/main/binary-amd64/Packages  Please use apt-cdrom to make this CD-ROM recognized by APT. apt-get update cannot be used to add new CD-ROMs
```
解决方法：
deb cdrom行注释掉
```css
$ vim /etc/apt/sources.list
#deb cdrom:[Ubuntu-Server 16.04 LTS _Xenial Xerus_ - Release amd64 (20160420.3)]/ xenial main restricted
```
添加docker配置文件，如果不那么保持严谨可以不用添加这一步。如果拉去镜像，最好配置国内得镜像源。

```css
cat > /etc/docker/daemon.json <<EOF
{
   "exec-opts": ["native.cgroupdriver=systemd"],
   "log-driver": "json-file",
   "log-opts": {
   "max-size":  "100m"
    }
 }
EOF
```

```css
mkdir -p /etc/systemd/system/docker.service.d
systemctl daemon-reload 
systemctl start docker
systemctl status docker
```
，
## 安装kubernets（all）

### 配置google kubernets源

```css
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
```
配置key

```css
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
```
更新并安装

```css
apt-get update
apt-get install -y kubelet kubeadm kubectl
```

Kubectl 自动补全
```bash
$ source <(kubectl completion bash) # setup autocomplete in bash, bash-completion package should be installed first.
$ source <(kubectl completion zsh)  # setup autocomplete in zsh
```

如果我们无法访问`google`，可以考虑用`aliyun`源
### 配置aliyun kubernets源

```css
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial main
EOF
```
验证：

```css
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 6A030B21BA07F4FB
```
配置key

```css
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -
```
安装工具

```css
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
## kubeadm创建集群master节点
直接执行初始化会拉取镜像失败（master）

```css
kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=192.168.211.40
```
拉取镜像失败

```css
failed to pull image k8s.gcr.io/kube-apiserver:v1.12.2
failed to pull image k8s.gcr.io/kube-controller-manager:v1.12.2
failed to pull image k8s.gcr.io/kube-scheduler:v1.12.2
failed to pull image k8s.gcr.io/kube-proxy:v1.12.2
failed to pull image k8s.gcr.io/pause:3.1
failed to pull image k8s.gcr.io/etcd:3.2.24
failed to pull image k8s.gcr.io/coredns:1.2.2
```

查看需要拉取得镜像(all）

```css
$  kubeadm config images list
k8s.gcr.io/kube-apiserver:v1.18.5
k8s.gcr.io/kube-controller-manager:v1.18.5
k8s.gcr.io/kube-scheduler:v1.18.5
k8s.gcr.io/kube-proxy:v1.18.5
k8s.gcr.io/pause:3.2
k8s.gcr.io/etcd:3.4.3-0
k8s.gcr.io/coredns:1.6.7
```
**注意：这个列表显示的tag名字和镜像版本号，从Kubernetes v1.12+开始，镜像名后面不带 amd64, arm, arm64, ppc64le 这样的标识了**

生成默认kubeadm.conf文件(all）

```css
$ kubeadm config print init-defaults > kubeadm.conf 
```

6.3 绕过墙下载镜像方法(all）
注意这个配置文件默认会从google的镜像仓库地址k8s.gcr.io下载镜像，如果你没有科学上网，那么就会 下载不来。因此，我们通过下面的方法把地址改成国内的，比如用阿里的：

```css
sed -i "s/imageRepository: .*/imageRepository: registry.aliyuncs.com\/google_containers/g" kubeadm.conf
```
6.4 指定kubeadm安装的Kubernetes版本

```bash
sed -i "s/kubernetesVersion: .*/kubernetesVersion: v1.18.1/g" kubeadm.conf
```


6.5 下载需要用到的镜像(all）
kubeadm.conf修改好后，我们执行下面命令就可以自动从国内下载需要用到的镜像了：

```bash
$ kubeadm config images pull --config kubeadm.conf
```
重新初始化master节点
```css
$ kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=192.168.211.40


 mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.211.40:6443 --token yuqokz.7989i4qx5770obvp \
    --discovery-token-ca-cert-hash sha256:167d0176ccd1c90b7373917940620fb7a48b245913eb25a05726345902f6213c 

```
把最后生成的命令记住
master执行：（master）

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
### node执行加入集群：（node）
**报错**
```bash
$ kubeadm join 192.168.211.40:6443 --token yuqokz.7989i4qx5770obvp     --discovery-token-ca-cert-hash sha256:167d0176ccd1c90b7373917940620fb7a48b245913eb25a05726345902f6213c
W0804 09:39:51.878220    8270 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR FileAvailable--etc-kubernetes-kubelet.conf]: /etc/kubernetes/kubelet.conf already exists
	[ERROR FileAvailable--etc-kubernetes-bootstrap-kubelet.conf]: /etc/kubernetes/bootstrap-kubelet.conf already exists
	[ERROR FileAvailable--etc-kubernetes-pki-ca.crt]: /etc/kubernetes/pki/ca.crt already exists
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```
解决方法：

```bash
$ kubeadm reset 
```
再次执行join加入集群
```bash
$ kubeadm join 192.168.211.40:6443 --token yuqokz.7989i4qx5770obvp \
    --discovery-token-ca-cert-hash sha256:167d0176ccd1c90b7373917940620fb7a48b245913eb25a05726345902f6213c 
    
W0804 09:47:01.922091   13137 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
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
默认token有效期为24小时，当过期之后，该token就不可用了。这时就需要重新创建token，操作如下：

```bash
$ kubeadm token create --print-join-command
```


master执行：

```c
root@master:~# kubectl get nodes
NAME     STATUS     ROLES    AGE     VERSION
master   NotReady   master   19m     v1.18.6
node1    NotReady   <none>   6m55s   v1.18.6
node2    NotReady   <none>   5m16s   v1.18.6
root@master:~# kubectl get pods
No resources found in default namespace.
root@master:~# kubectl get pods -A
NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE
kube-system   coredns-66bff467f8-2xmnt         0/1     Pending   0          21m
kube-system   coredns-66bff467f8-ghj2s         0/1     Pending   0          21m
kube-system   etcd-master                      1/1     Running   0          22m
kube-system   kube-apiserver-master            1/1     Running   0          22m
kube-system   kube-controller-manager-master   1/1     Running   0          22m
kube-system   kube-proxy-dh46z                 1/1     Running   0          7m35s
kube-system   kube-proxy-jq6cb                 1/1     Running   0          21m
kube-system   kube-proxy-z6prp                 1/1     Running   0          9m14s
kube-system   kube-scheduler-master            1/1     Running   0          22m
```
node执行：

```c
root@node2:~# kubectl get nodes
(卡住)
root@node2:~# mkdir -p $HOME/.kube
root@node2:~# scp root@192.168.211.40:/root/.kube/config /root/.kube/
root@node2:~# kubectl get pods -A
NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE
kube-system   coredns-66bff467f8-2xmnt         0/1     Pending   0          29m
kube-system   coredns-66bff467f8-ghj2s         0/1     Pending   0          29m
kube-system   etcd-master                      1/1     Running   0          29m
kube-system   kube-apiserver-master            1/1     Running   0          29m
kube-system   kube-controller-manager-master   1/1     Running   0          29m
kube-system   kube-proxy-dh46z                 1/1     Running   0          14m
kube-system   kube-proxy-jq6cb                 1/1     Running   0          29m
kube-system   kube-proxy-z6prp                 1/1     Running   0          16m
kube-system   kube-scheduler-master            1/1     Running   0          29m

```
## 安装网络calico

```bash
$ kubectl apply -f https://docs.projectcalico.org/v3.10/manifests/calico.yaml
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
daemonset.apps/calico-node created
serviceaccount/calico-node created
deployment.apps/calico-kube-controllers created
serviceaccount/calico-kube-controllers created

```
pod创建失败,删除重新载尝试一次

```bash
$ kubectl delete -f https://docs.projectcalico.org/v3.10/manifests/calico.yaml
$ kubectl apply -f https://docs.projectcalico.org/v3.10/manifests/calico.yaml
```

可能是拉取镜像失败或者DNS问题,kubectl describe看一下

```bash
kubectl describe pods <pod_name> -n <namespace> 
```
假如拉取镜像失败，手动尝试拉取

```bash
docker pull xxx
```

DNS问题,换用

```bash
$ cat /etc/resolve.conf
nameserver 114.114.114.114  #国内DNS
```
最后查看pod是否创建成功
```bash
kubectl get pods -A 
```
**注意：内存要给足，否则也有可能使pod处于pending状态。**
