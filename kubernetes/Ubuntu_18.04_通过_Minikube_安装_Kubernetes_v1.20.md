#  Ubuntu 18.04 通过 Minikube 安装 Kubernetes v1.20
tags: kubernetes,部署,minikube
[![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c3508c1964f301113eedf597f2e43110.png)](https://www.rottentomatoes.com/m/forrest_gump)

*Life was like a box of chocolates, you never know what you're going to get.*

*生命就像一盒巧克力，结果往往出人意料    ——《阿甘正传》* 







## 1. 关闭swap

```bash
swapoff -a
```

## 2. 安装依赖tools

```bash
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common conntrack
```
由于国内网络的不稳定性，我们需要将相关镜像源切换为国内阿里云的镜像

## 3. 添加阿里云镜像

```bash
curl -fsSL https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF
sudo apt-get update
```
## 4. 安装 Docker、kubectl、kubeadm

```bash
#查看docker版本
$ sudo apt-cache madison docker.io
 docker.io | 19.03.6-0ubuntu1~18.04.3 | http://cn.archive.ubuntu.com/ubuntu bionic-updates/universe amd64 Packages
 docker.io | 19.03.6-0ubuntu1~18.04.2 | http://cn.archive.ubuntu.com/ubuntu bionic-security/universe amd64 Packages
 docker.io | 17.12.1-0ubuntu1 | http://cn.archive.ubuntu.com/ubuntu bionic/universe amd64 Packages

sudo apt-get install -y docker.io kubectl kubeadm
```
## 5. 配置阿里云 Docker 镜像加速器
这里采用了阿里云的镜像加速器（需要阿里云账号进行登录），地址：阿里云 -> 容器镜像服务 -> 镜像工具 -> 镜像加速器
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a9754d05acb0df5d07e10bf1873eb6f8.png)


```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
"registry-mirrors": ["https://xxxxxx.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

以上配置是阿里云镜像加速器中的配置，本文中这一块儿是直接从阿里云镜像加速器中的配置说明复制的，大家根据自己的情况在阿里云镜像加速器中去复制。

## 6. 安装 Minikube
目前最新 v1.24.0，查看最新版：[https://github.com/kubernetes/minikube/releases](https://github.com/kubernetes/minikube/releases)
如果curl无法下载，也可以通过手动下载并上传到服务器的形式

```bash
curl -Lo minikube https://github.com/kubernetes/minikube/releases/download/v1.17.1/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

使用 Minikube 创建 Kubernetes
格式1
```bash
minikube start --vm-driver=none --apiserver-ips=<your-server-ip> --image-mirror-country cn \
 --iso-url=https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/iso/minikube-v1.17.1.iso \
 --registry-mirror=https://xxxxxx.mirror.aliyuncs.com \
 --image-repository=https://registry.aliyuncs.com/google_containers
```

此处的 registry-mirror 是阿里云的镜像加速器的 mirror，换成你自己的即可; apiserver-ips 则是你服务器的IP，我的是`https://q2hy3fzi.mirror.aliyuncs.com`，因为如果需要远程访问的话，需要将服务器的IP进行暴露，同样换成你自己的服务器IP即可.

执行命令：

```bash
minikube start --vm-driver=none --apiserver-ips=192.168.211.55 --image-mirror-country cn  --iso-url=https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/iso/minikube-v1.17.1.iso  --registry-mirror=https://q2hy3fzi.mirror.aliyuncs.com  --image-repository=https://registry.aliyuncs.com/google_containers
```
输出内容：

```bash
* minikube v1.17.1 on Ubuntu 18.04
* Using the none driver based on existing profile

X The requested memory allocation of 1970MiB does not leave room for system overhead (total system memory: 1970MiB). You may face stability issues.
* Suggestion: Start minikube with less memory allocated: 'minikube start --memory=1970mb'

* Starting control plane node minikube in cluster minikube
* Restarting existing none bare metal machine for "minikube" ...
* OS release is Ubuntu 18.04.5 LTS
* Preparing Kubernetes v1.20.2 on Docker 19.03.6 ...
  - kubelet.resolv-conf=/run/systemd/resolve/resolv.conf
  - Generating certificates and keys ...
  - Booting up control plane ...
  - Configuring RBAC rules ...
* Configuring local host environment ...
* 
! The 'none' driver is designed for experts who need to integrate with an existing VM
* Most users should use the newer 'docker' driver instead, which does not require root!
* For more information, see: https://minikube.sigs.k8s.io/docs/reference/drivers/none/
* 
! kubectl and minikube configuration will be stored in /root
! To use kubectl or minikube commands as your own user, you may need to relocate them. For example, to overwrite your own settings, run:
* 
  - sudo mv /root/.kube /root/.minikube $HOME
  - sudo chown -R $USER $HOME/.kube $HOME/.minikube
* 
* This can also be done automatically by setting the env var CHANGE_MINIKUBE_NONE_USER=true
* Verifying Kubernetes components...
* Enabled addons: storage-provisioner, default-storageclass
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

如果不出意外的话，由 minikube 创建的单机 Kubernetes 环境就成功了。
## 7. 检查

```c
root@spectre:~# kubectl get nodes
NAME      STATUS   ROLES                  AGE   VERSION
spectre   Ready    control-plane,master   68s   v1.20.2
root@spectre:~# kubectl get pods -A
NAMESPACE     NAME                              READY   STATUS    RESTARTS   AGE
kube-system   coredns-7f89b7bc75-774mp          1/1     Running   0          5m53s
kube-system   etcd-spectre                      1/1     Running   0          6m7s
kube-system   kube-apiserver-spectre            1/1     Running   0          6m7s
kube-system   kube-controller-manager-spectre   1/1     Running   0          6m7s
kube-system   kube-proxy-swbhc                  1/1     Running   0          5m53s
kube-system   kube-scheduler-spectre            1/1     Running   0          6m7s
kube-system   storage-provisioner               1/1     Running   0          6m6s

root@spectre:~# kubectl cluster-info
Kubernetes control plane is running at https://192.168.211.55:8443
KubeDNS is running at https://192.168.211.55:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

```

##  8. 常用命令

```bash
# 进入集群节点
minikube ssh

# 查看节点 IP
minikube ip

# 停止集群
minikube stop

# 删除集群
minikube delete
```
##  9.异常
### 9.1 部署异常
[storage-provisioner Pod镜像名称错误](https://github.com/kubernetes/minikube/issues/11881)
解决方法
```bash
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/storage-provisioner:v5
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/storage-provisioner:v5 registry.cn-hangzhou.aliyuncs.com/google_containers/k8s-minikube/storage-provisioner:v5
minikube image load registry.cn-hangzhou.aliyuncs.com/google_containers/k8s-minikube/storage-provisioner:v5
```
##  10. 部署应用程序
创建一个示例部署并在端口 8080 上公开它：

```bash
kubectl create deployment hello-minikube --image=registry.aliyuncs.com/google_containers/echoserver:1.4
deployment.apps/hello-minikube created

kubectl expose deployment hello-minikube --type=NodePort --port=8080

kubectl get services hello-minikube
```
##  11. 集群启动与停止
当我们需要修改docker配置等需要集群停止的需求可以执行命令：

```bash
minikube stop
```

当修改完配置

```bash
minikube start
```

参考：

 - [kind 部署 kubernetes 集群](https://blog.csdn.net/xixihahalelehehe/article/details/121968488?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164845801216780269819034%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164845801216780269819034&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-2-121968488.nonecase&utm_term=kind&spm=1018.2226.3001.4450)
 - [Minikube 在ubuntu 部署 Kubernetes](https://blog.csdn.net/xixihahalelehehe/article/details/113527867?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164845397316780265442500%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164845397316780265442500&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-113527867.nonecase&utm_term=ubuntu%E5%AE%89%E8%A3%85minikube&spm=1018.2226.3001.4450) 
 - [Minikube 在 Centos 7 部署 Kubernetes](https://ghostwritten.blog.csdn.net/article/details/123796854)
 - [kubeadm 部署 kubernetes 集群](https://blog.csdn.net/xixihahalelehehe/article/details/105567076)
 - [**更多Minikube细节请参考官方**](https://minikube.sigs.k8s.io/docs/)
 - [**kubernetes 快速学习手册**](https://ghostwritten.blog.csdn.net/article/details/108562082)



