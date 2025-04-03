# minikube & kubernetes 动手指南
![](https://i-blog.csdnimg.cn/blog_migrate/3e245e05d6ff49890e6778151c28c430.jpeg#pic_center)

<font color=#9400D3 size=3 face="楷体">"Minikube 可快速设置部署 Kubernetes 集群，专注于让 Kubernetes 易于学习和开发。"</font>



## 1. 准备
- Centos 7.9.2009 系统

```bash
$ cat /etc/resolv.conf 
nameserver 8.8.8.8
```
配置主机名

```bash
hostnanmectl set-hostname minikube1
```
路由转发

```bash
cat <<EOF>> /etc/sysctl.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables=1
```
关闭swap

```bash
swapoff -a
```

##  2. 安装依赖 tools
 配置[ linux yum 源](https://blog.csdn.net/xixihahalelehehe/article/details/105685088)
```bash
yum -y update
yum -y install apt-transport-https ca-certificates curl software-properties-common conntrack
```

##  3. 安装 docker
你可以根据[ docker 官方](https://docs.docker.com/engine/install/centos/)寻找合适的安装方式

配置docker-ce源
```bash
 sudo yum install -y yum-utils
 sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
查看docker-ce版本

```bash
yum list docker-ce  --showduplicates | sort -r
```
安装

```bash
yum -y install docker-ce docker-ce-cli containerd.io
```

配置docker


```bash
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
   "exec-opts": ["native.cgroupdriver=systemd"],
   "live-restore": true,
   "dns": ["8.8.8.8"],
   "log-driver": "json-file",
   "log-opts": {
     "max-size":  "100m",
     "max-file": "5"
    },
   "registry-mirrors": [
    "https://ckdhnbk9.mirror.aliyuncs.com",
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ]
 }
EOF
```
启动 docker
```bash
systemctl daemon-reload && systemctl start  docker && systemctl enable  docker
```

查看docker版本

```bash
$ docker version
Client: Docker Engine - Community
 Version:           20.10.14
 API version:       1.41
 Go version:        go1.16.15
 Git commit:        a224086
 Built:             Thu Mar 24 01:49:57 2022
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.14
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.16.15
  Git commit:       87a90dc
  Built:            Thu Mar 24 01:48:24 2022
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.5.11
  GitCommit:        3df54a852345ae127d1fa3092b95168e4a88e2f8
 runc:
  Version:          1.0.3
  GitCommit:        v1.0.3-0-gf46b6ba
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```


## 4. 安装 minikube
你可以根据[ minikube 官方安装](https://minikube.sigs.k8s.io/docs/start/)寻找适合自己的环境。


```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod 755 minikube-linux-amd64
mv minikube-linux-amd64 /usr/bin/minikube
ln -s /usr/bin/minikube /usr/local/bin/
```

查看版本

```bash
$ minikube version
minikube version: v1.25.2
commit: 362d5fdc0a3dbee389b3d3f1034e8023e72bd3a7
```
我使用的[ minikube 版本](https://github.com/kubernetes/minikube/releases) `v1.25.2`，当前（`2022.11.30`） `minikube`最新版本已经是 `v1.28.0`，最新版本部署k8s步骤存在更多依赖，例如：cri-docker、crictl，有点麻烦。具体步骤参考[centos(7.9) minikube(v1.28.0) kaniko 构建镜像](https://mp.csdn.net/mp_blog/creation/success/128089828)
## 5. 安装 kubectl
- 在[kubernetes官方有多种 kubectl 安装方式](https://kubernetes.io/docs/tasks/tools/)


我这里使用阿里云 `yum` 源安装 kubectl
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

yum -y install kubectl
```
查看版本

```bash
$ kubectl version --client -o json
{
  "clientVersion": {
    "major": "1",
    "minor": "23",
    "gitVersion": "v1.23.5",
    "gitCommit": "c285e781331a3785a7f436042c65c5641ce8a9e9",
    "gitTreeState": "clean",
    "buildDate": "2022-03-16T15:58:47Z",
    "goVersion": "go1.17.8",
    "compiler": "gc",
    "platform": "linux/amd64"
  }
}

```

## 6. minikube 创建 kubernetes 集群

由于非科学网络环境的影响，没有参数它会报以下错误：

```bash
minikube start
* Centos 7.9.2009 上的 minikube v1.25.2
* 自动选择 docker 驱动。其他选项：none, ssh
* The "docker" driver should not be used with root privileges.
* If you are running minikube within a VM, consider using --driver=none:
*   https://minikube.sigs.k8s.io/docs/reference/drivers/none/

X Exiting due to DRV_AS_ROOT: The "docker" driver should not be used with root privileges.
```
正确但不是唯一的方式：

```bash
 minikube start --vm-driver=none --image-mirror-country=cn --registry-mirror='https://ckdhnbk9.mirror.aliyuncs.com' --image-repository='registry.cn-hangzhou.aliyuncs.com/google_containers'
```
输出:
```bash

* Centos 7.9.2009 上的 minikube v1.25.2
* 根据用户配置使用 none 驱动程序

X Requested memory allocation (1819MB) is less than the recommended minimum 1900MB. Deployments may fail.


X The requested memory allocation of 1819MiB does not leave room for system overhead (total system memory: 1819MiB). You may face stability issues.
* 建议：Start minikube with less memory allocated: 'minikube start --memory=1819mb'

* 正在使用镜像存储库 registry.cn-hangzhou.aliyuncs.com/google_containers
* Starting control plane node minikube in cluster minikube
* Running on localhost (CPUs=4, Memory=1819MB, Disk=17394MB) ...
* OS release is CentOS Linux 7 (Core)
* 正在 Docker 20.10.14 中准备 Kubernetes v1.23.3…
  - kubelet.housekeeping-interval=5m
    > kubeadm.sha256: 64 B / 64 B [--------------------------] 100.00% ? p/s 0s
    > kubelet.sha256: 64 B / 64 B [--------------------------] 100.00% ? p/s 0s
    > kubectl.sha256: 64 B / 64 B [--------------------------] 100.00% ? p/s 0s
    > kubeadm: 43.12 MiB / 43.12 MiB [---------------] 100.00% 1.23 MiB p/s 35s
    > kubectl: 44.43 MiB / 44.43 MiB [---------------] 100.00% 1.03 MiB p/s 43s
    > kubelet: 118.75 MiB / 118.75 MiB [-------------] 100.00% 2.12 MiB p/s 56s
  - Generating certificates and keys ...
  - Booting up control plane ...
  - Configuring RBAC rules ...
* 开始配置本地主机环境...
* 
! The 'none' driver is designed for experts who need to integrate with an existing VM
* Most users should use the newer 'docker' driver instead, which does not require root!
* For more information, see: https://minikube.sigs.k8s.io/docs/reference/drivers/none/
* 
! kubectl 和 minikube 配置将存储在 /root 中
! 如需以您自己的用户身份使用 kubectl 或 minikube 命令，您可能需要重新定位该命令。例如，如需覆盖您的自定义设置，请运行：
* 
  - sudo mv /root/.kube /root/.minikube $HOME
  - sudo chown -R $USER $HOME/.kube $HOME/.minikube
* 
* 此操作还可通过设置环境变量 CHANGE_MINIKUBE_NONE_USER=true 自动完成
* Verifying Kubernetes components...
  - Using image registry.cn-hangzhou.aliyuncs.com/google_containers/storage-provisioner:v5
* Enabled addons: default-storageclass, storage-provisioner
* Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```
根据环境不同，定制配置不同，可以自定义添加一些参数配置，例如

```bash
#尝试指定不同的 minikube 版本
minikube start  --vm-driver=none --image-mirror-country=cn --registry-mirror='https://ckdhnbk9.mirror.aliyuncs.com' --image-repository='registry.cn-hangzhou.aliyuncs.com/google_containers'  --kubernetes-version=v1.23.8

#启动minikube时使用虚拟机驱动程序和“docker”容器运行时(如果尚未运行)。
minikube start --container-runtime=docker --vm=true

# 添加 网络插件 calico
minikube start  --vm-driver=none  --network-plugin=cni --cni=calico 
.....
```

##  7. 查看

### 7.1 查看集群配置信息
```bash
$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /root/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Mon, 28 Mar 2022 17:20:36 CST
        provider: minikube.sigs.k8s.io
        version: v1.25.2
      name: cluster_info
    server: https://192.168.211.51:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    extensions:
    - extension:
        last-update: Mon, 28 Mar 2022 17:20:36 CST
        provider: minikube.sigs.k8s.io
        version: v1.25.2
      name: context_info
    namespace: default
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /root/.minikube/profiles/minikube/client.crt
    client-key: /root/.minikube/profiles/minikube/client.key
```
### 7.2 查看集群状态

```bash
$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```
###  7.3 查看 node

```bash
$ kubectl get nodes
NAME                    STATUS   ROLES                  AGE     VERSION
localhost.localdomain   Ready    control-plane,master   5m24s   v1.23.3
```
###  7.4 查看 pod

```bash
$ kubectl get pods -A
NAMESPACE     NAME                                            READY   STATUS    RESTARTS   AGE
kube-system   coredns-65c54cc984-2cf5f                        1/1     Running   0          5m48s
kube-system   etcd-localhost.localdomain                      1/1     Running   0          6m2s
kube-system   kube-apiserver-localhost.localdomain            1/1     Running   0          6m
kube-system   kube-controller-manager-localhost.localdomain   1/1     Running   0          6m
kube-system   kube-proxy-khn4n                                1/1     Running   0          5m49s
kube-system   kube-scheduler-localhost.localdomain            1/1     Running   0          6m
kube-system   storage-provisioner                             1/1     Running   0          5m58s
```
前面加一个`minikube`也可以。
```bash
 minikube kubectl -- get po -A
NAMESPACE              NAME                                         READY   STATUS      RESTARTS   AGE
default                kaniko                                       0/1     Completed   0          8h
kube-system            coredns-65c54cc984-xc4v4                     1/1     Running     0          19h
kube-system            etcd-minikube1                               1/1     Running     0          19h
kube-system            kube-apiserver-minikube1                     1/1     Running     0          19h
kube-system            kube-controller-manager-minikube1            1/1     Running     0          19h
kube-system            kube-proxy-n82vp                             1/1     Running     0          19h
kube-system            kube-scheduler-minikube1                     1/1     Running     0          19h
kube-system            storage-provisioner                          1/1     Running     0          19h
kubernetes-dashboard   dashboard-metrics-scraper-57d8d5b8b8-zhtjq   1/1     Running     0          45m
kubernetes-dashboard   kubernetes-dashboard-6f75b5c656-dxr87        1/1     Running     0          45m
```

###  7.5 查看集群信息

```bash
$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.211.51:8443
CoreDNS is running at https://192.168.211.51:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```
###  7.6 查看集群ip

```bash
$ minikube ip
192.168.211.51
```

###  7.7 查看插件

```bash
$ minikube addons list
|-----------------------------|----------|--------------|--------------------------------|
|         ADDON NAME          | PROFILE  |    STATUS    |           MAINTAINER           |
|-----------------------------|----------|--------------|--------------------------------|
| ambassador                  | minikube | disabled     | third-party (ambassador)       |
| auto-pause                  | minikube | disabled     | google                         |
| csi-hostpath-driver         | minikube | disabled     | kubernetes                     |
| dashboard                   | minikube | disabled     | kubernetes                     |
| default-storageclass        | minikube | enabled ✅   | kubernetes                     |
| efk                         | minikube | disabled     | third-party (elastic)          |
| freshpod                    | minikube | disabled     | google                         |
| gcp-auth                    | minikube | disabled     | google                         |
| gvisor                      | minikube | disabled     | google                         |
| helm-tiller                 | minikube | disabled     | third-party (helm)             |
| ingress                     | minikube | disabled     | unknown (third-party)          |
| ingress-dns                 | minikube | disabled     | google                         |
| istio                       | minikube | disabled     | third-party (istio)            |
| istio-provisioner           | minikube | disabled     | third-party (istio)            |
| kong                        | minikube | disabled     | third-party (Kong HQ)          |
| kubevirt                    | minikube | disabled     | third-party (kubevirt)         |
| logviewer                   | minikube | disabled     | unknown (third-party)          |
| metallb                     | minikube | disabled     | third-party (metallb)          |
| metrics-server              | minikube | disabled     | kubernetes                     |
| nvidia-driver-installer     | minikube | disabled     | google                         |
| nvidia-gpu-device-plugin    | minikube | disabled     | third-party (nvidia)           |
| olm                         | minikube | disabled     | third-party (operator          |
|                             |          |              | framework)                     |
| pod-security-policy         | minikube | disabled     | unknown (third-party)          |
| portainer                   | minikube | disabled     | portainer.io                   |
| registry                    | minikube | disabled     | google                         |
| registry-aliases            | minikube | disabled     | unknown (third-party)          |
| registry-creds              | minikube | disabled     | third-party (upmc enterprises) |
| storage-provisioner         | minikube | enabled ✅   | google                         |
| storage-provisioner-gluster | minikube | disabled     | unknown (third-party)          |
| volumesnapshots             | minikube | disabled     | kubernetes                     |
|-----------------------------|----------|--------------|--------------------------------|
```

### 7.8 查看日志

```bash
minikube logs
```

## 8. 常用操作
###   8.1 进入集群节点
```bash
minikube ssh
```
###  8.2 停止集群
```c
minikube stop
```
###  8.3 启动集群
```c
minikube start
```
### 8.4 删除集群
```c
minikube delete
minikube delete --all
```
### 8.5 暂停集群
但不影响已部署的应用程序

```bash
minikube pause
```
### 8.5 取消暂停

```bash
minikube unpause
```
###  8.6 修改默认内存限制

```bash
minikube config set memory 9001
```

- [更多命令指南](https://minikube.sigs.k8s.io/docs/commands/)


## 9. 部署 Ingress


启用`Ingress`插件

```bash
$ minikube addons enable ingress
🔎  Verifying ingress addon...
🌟  The 'ingress' addon is enabled
```
查看 pod
```bash
$ kubectl get pods -n ingress-nginx
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-6hf47        0/1     Completed   0          91s
ingress-nginx-admission-patch-5dpqz         0/1     Completed   0          91s
ingress-nginx-controller-6cfb67d797-gqj98   1/1     Running     0          91s
```

##  10.管理 dashboard
[Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/) 是一个基于 Web 的 Kubernetes 用户界面。您可以使用它来：

 - 将容器化应用程序部署到 Kubernetes 集群
 - 对您的容器化应用程序进行故障排除
 - 管理集群资源
 - 概览在您的集群上运行的应用程序
 - 创建或修改单个 Kubernetes 资源（例如 Deployment、Jobs、DaemonSets 等）

例如，您可以使用部署向导扩展部署、启动滚动更新、重新启动 pod 或部署新应用程序。

### 10.1 创建 dashboard

```bash
minikube dashboard
```
输出：

```bash
🔌  Enabling dashboard ...
    ▪ Using image docker.io/kubernetesui/dashboard:v2.7.0
    ▪ Using image docker.io/kubernetesui/metrics-scraper:v1.0.8
💡  Some dashboard features require the metrics-server addon. To enable all features please run:

        minikube addons enable metrics-server


🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
http://127.0.0.1:43995/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```
当然，我们可以指定喜欢的端口(port)

```bash
$ minikube dashboard --port 8081
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
http://127.0.0.1:8081/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```

这将启用 dashboard 插件，并在默认 Web 浏览器中打开代理。


要停止代理（使仪表板保持运行），请中止已启动的进程 ( `Ctrl+C`)。

查看 `dashboard` 是否启动正常

```bash
$ kubectl  get all -n kubernetes-dashboard
NAME                                             READY   STATUS    RESTARTS   AGE
pod/dashboard-metrics-scraper-57d8d5b8b8-zhtjq   1/1     Running   0          126m
pod/kubernetes-dashboard-6f75b5c656-dxr87        1/1     Running   0          126m

NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/dashboard-metrics-scraper   ClusterIP   10.97.77.170     <none>        8000/TCP   126m
service/kubernetes-dashboard        ClusterIP   10.101.172.254   <none>        80/TCP     126m

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/dashboard-metrics-scraper   1/1     1            1           126m
deployment.apps/kubernetes-dashboard        1/1     1            1           126m

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/dashboard-metrics-scraper-57d8d5b8b8   1         1         1       126m
replicaset.apps/kubernetes-dashboard-6f75b5c656        1         1         1       126m
```
查看界面 `URL`
```bash
$ minikube dashboard --url
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
http://127.0.0.1:43995/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```



### 10.2 访问 API
访问 `dashboard API`资源

```bash
$ curl http://127.0.0.1:43995/
{
  "paths": [
    "/.well-known/openid-configuration",
    "/api",
    "/api/v1",
    "/apis",
    "/apis/",
    "/apis/admissionregistration.k8s.io",
    "/apis/admissionregistration.k8s.io/v1",
    "/apis/apiextensions.k8s.io",
    "/apis/apiextensions.k8s.io/v1",
    "/apis/apiregistration.k8s.io",
    "/apis/apiregistration.k8s.io/v1",
    "/apis/apps",
    "/apis/apps/v1",
    "/apis/authentication.k8s.io",
    "/apis/authentication.k8s.io/v1",
    "/apis/authorization.k8s.io",
    "/apis/authorization.k8s.io/v1",
    "/apis/autoscaling",
    "/apis/autoscaling/v1",
    "/apis/autoscaling/v2",
    "/apis/autoscaling/v2beta1",
    "/apis/autoscaling/v2beta2",
    "/apis/batch",
    "/apis/batch/v1",
    "/apis/batch/v1beta1",
    "/apis/certificates.k8s.io",
    "/apis/certificates.k8s.io/v1",
    "/apis/coordination.k8s.io",
    "/apis/coordination.k8s.io/v1",
    "/apis/discovery.k8s.io",
    "/apis/discovery.k8s.io/v1",
    "/apis/discovery.k8s.io/v1beta1",
    "/apis/events.k8s.io",
    "/apis/events.k8s.io/v1",
    "/apis/events.k8s.io/v1beta1",
    "/apis/flowcontrol.apiserver.k8s.io",
    "/apis/flowcontrol.apiserver.k8s.io/v1beta1",
    "/apis/flowcontrol.apiserver.k8s.io/v1beta2",
    "/apis/networking.k8s.io",
    "/apis/networking.k8s.io/v1",
    "/apis/node.k8s.io",
    "/apis/node.k8s.io/v1",
    "/apis/node.k8s.io/v1beta1",
    "/apis/policy",
    "/apis/policy/v1",
    "/apis/policy/v1beta1",
    "/apis/rbac.authorization.k8s.io",
    "/apis/rbac.authorization.k8s.io/v1",
    "/apis/scheduling.k8s.io",
    "/apis/scheduling.k8s.io/v1",
    "/apis/storage.k8s.io",
    "/apis/storage.k8s.io/v1",
    "/apis/storage.k8s.io/v1beta1",
    "/healthz",
    "/healthz/autoregister-completion",
    "/healthz/etcd",
    "/healthz/log",
    "/healthz/ping",
    "/healthz/poststarthook/aggregator-reload-proxy-client-cert",
    "/healthz/poststarthook/apiservice-openapi-controller",
    "/healthz/poststarthook/apiservice-registration-controller",
    "/healthz/poststarthook/apiservice-status-available-controller",
    "/healthz/poststarthook/bootstrap-controller",
    "/healthz/poststarthook/crd-informer-synced",
    "/healthz/poststarthook/generic-apiserver-start-informers",
    "/healthz/poststarthook/kube-apiserver-autoregistration",
    "/healthz/poststarthook/priority-and-fairness-config-consumer",
    "/healthz/poststarthook/priority-and-fairness-config-producer",
    "/healthz/poststarthook/priority-and-fairness-filter",
    "/healthz/poststarthook/rbac/bootstrap-roles",
    "/healthz/poststarthook/scheduling/bootstrap-system-priority-classes",
    "/healthz/poststarthook/start-apiextensions-controllers",
    "/healthz/poststarthook/start-apiextensions-informers",
    "/healthz/poststarthook/start-cluster-authentication-info-controller",
    "/healthz/poststarthook/start-kube-aggregator-informers",
    "/healthz/poststarthook/start-kube-apiserver-admission-initializer",
    "/livez",
    "/livez/autoregister-completion",
    "/livez/etcd",
    "/livez/log",
    "/livez/ping",
    "/livez/poststarthook/aggregator-reload-proxy-client-cert",
    "/livez/poststarthook/apiservice-openapi-controller",
    "/livez/poststarthook/apiservice-registration-controller",
    "/livez/poststarthook/apiservice-status-available-controller",
    "/livez/poststarthook/bootstrap-controller",
    "/livez/poststarthook/crd-informer-synced",
    "/livez/poststarthook/generic-apiserver-start-informers",
    "/livez/poststarthook/kube-apiserver-autoregistration",
    "/livez/poststarthook/priority-and-fairness-config-consumer",
    "/livez/poststarthook/priority-and-fairness-config-producer",
    "/livez/poststarthook/priority-and-fairness-filter",
    "/livez/poststarthook/rbac/bootstrap-roles",
    "/livez/poststarthook/scheduling/bootstrap-system-priority-classes",
    "/livez/poststarthook/start-apiextensions-controllers",
    "/livez/poststarthook/start-apiextensions-informers",
    "/livez/poststarthook/start-cluster-authentication-info-controller",
    "/livez/poststarthook/start-kube-aggregator-informers",
    "/livez/poststarthook/start-kube-apiserver-admission-initializer",
    "/logs",
    "/metrics",
    "/openapi/v2",
    "/openid/v1/jwks",
    "/readyz",
    "/readyz/autoregister-completion",
    "/readyz/etcd",
    "/readyz/informer-sync",
    "/readyz/log",
    "/readyz/ping",
    "/readyz/poststarthook/aggregator-reload-proxy-client-cert",
    "/readyz/poststarthook/apiservice-openapi-controller",
    "/readyz/poststarthook/apiservice-registration-controller",
    "/readyz/poststarthook/apiservice-status-available-controller",
    "/readyz/poststarthook/bootstrap-controller",
    "/readyz/poststarthook/crd-informer-synced",
    "/readyz/poststarthook/generic-apiserver-start-informers",
    "/readyz/poststarthook/kube-apiserver-autoregistration",
    "/readyz/poststarthook/priority-and-fairness-config-consumer",
    "/readyz/poststarthook/priority-and-fairness-config-producer",
    "/readyz/poststarthook/priority-and-fairness-filter",
    "/readyz/poststarthook/rbac/bootstrap-roles",
    "/readyz/poststarthook/scheduling/bootstrap-system-priority-classes",
    "/readyz/poststarthook/start-apiextensions-controllers",
    "/readyz/poststarthook/start-apiextensions-informers",
    "/readyz/poststarthook/start-cluster-authentication-info-controller",
    "/readyz/poststarthook/start-kube-aggregator-informers",
    "/readyz/poststarthook/start-kube-apiserver-admission-initializer",
    "/readyz/shutdown",
    "/version"
  ]
}
```
例如，访问集群是否健康
```bash
$ curl http://127.0.0.1:8085/healthz
ok
```

###  10.3 域名访问
我准备中止已启动的进程 ( `Ctrl+C`)，实现通过域名访问 `kubernetes-dashboard`，我们已经部署了`ingress-controller`,只需要编写一个`ingress` yaml文件即可。
- `dashboard-ingress.yaml`
```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: dashboard-ingress
  namespace: kubernetes-dashboard
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: dashboard.com
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: kubernetes-dashboard
              port:
                number: 80
```
创建为 `dashboard-ingress` 

```bash
$ k apply -f dashboard-ingress.yaml
ingress.networking.k8s.io/dashboard-ingress created

注意：这里ADDRESS需要等待一段时间域名才能解析到主机地址
$ k get -n kubernetes-dashboard ingress
NAME                CLASS   HOSTS           ADDRESS   PORTS   AGE
dashboard-ingress   nginx   dashboard.com             80      23s

等到了
$ k get -n kubernetes-dashboard ingress --watch
NAME                CLASS   HOSTS           ADDRESS         PORTS   AGE
dashboard-ingress   nginx   dashboard.com   192.168.10.25   80      11m
```
在主机`hosts`文件添加此映射配置

```bash
cat <<EOF >> /etc/hosts
192.168.10.25   dashboard.com
EOF
```
`windows`: 在 `C:\Windows\System32\drivers\etc\hosts`添加`192.168.10.25   dashboard.com`
访问 `dashboard.com`

![](https://i-blog.csdnimg.cn/blog_migrate/f4f8edb136ed7002b82c9e027579c723.png)


- 更多关于 [kubernetes dashboard 内容](https://blog.csdn.net/xixihahalelehehe/article/details/115913408)请参考这篇文章



##  11. 部署应用

### 11.1 创建 `NodePort` 类型的`deployment`
```bash
kubectl create deployment hello-minikube --image=docker.io/nginx:1.23
kubectl expose deployment hello-minikube --type=NodePort --port=80
```

```bash
$ kubectl get services hello-minikube
NAME             TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
hello-minikube   NodePort   10.96.236.93   <none>        80:30578/TCP   60m

$  minikube service hello-minikube
|-----------|----------------|-------------|----------------------------|
| NAMESPACE |      NAME      | TARGET PORT |            URL             |
|-----------|----------------|-------------|----------------------------|
| default   | hello-minikube |          80 | http://192.168.10.25:30578 |
|-----------|----------------|-------------|----------------------------|
🎉  Opening service default/hello-minikube in default browser...
👉  http://192.168.10.25:30578
```
浏览器访问：
![](https://i-blog.csdnimg.cn/blog_migrate/8849cfba107d63faa0c16d19b83e154f.png)
查询 URL

```bash
$ minikube service hello-minikube --url
http://192.168.10.25:30578
```

或者，使用`kubectl`转发端口:
```bash
$ kubectl port-forward service/hello-minikube 7080:80
Forwarding from 127.0.0.1:7080 -> 80
Forwarding from [::1]:7080 -> 80
```
新打开一个终端：

```bash
$ curl 127.0.0.1:7080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```





### 11.2 创建 `LoadBalancer` 类型的 `deployment`

当你想被集群外访问，创建 `LoadBalancer` 类型的 `deployment`

```bash
kubectl create deployment hello-minikube1 --image=docker.io/nginx:1.23
kubectl expose deployment hello-minikube1 --type=LoadBalancer --port=8080
```
查看svc

```bash
$ k get svc hello-minikube1
NAME              TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-minikube1   LoadBalancer   10.101.92.170   <pending>     8080:31412/TCP   113s
```
pending，那么如何获取`EXTERNAL-IP`

`minikube  tunnel` 作为一个进程运行，在主机上使用集群的IP地址作为网关创建到集群的服务`CIDR`的网络路由。tunnel命令直接向主机操作系统上运行的任何程序公开外部IP。

```bash
$ minikube tunnel
Status:
        machine: minikube
        pid: 15915
        route: 10.96.0.0/12 -> 192.168.10.25
        minikube: Running
        services: [hello-minikube1]
    errors:
                minikube: no errors
                router: no errors
                loadbalancer emulator: no errors
```
新打开一个终端

```bash
$ kubectl get svc hello-minikube1
NAME              TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)          AGE
hello-minikube1   LoadBalancer   10.101.92.170   10.101.92.170   8080:31412/TCP   5m48s
```
在浏览器中打开(确保没有代理)
访问：`http://REPLACE_WITH_EXTERNAL_IP:8080`

虽然获取到了`EXTERNAL_IP`，但访问测试没通，姿势不对。
讨论：
- [Minikube - External IP not match host's public IP](https://stackoverflow.com/questions/57359528/default-cni-in-minikube)
- [Unable to access application through minikube tunnel](https://stackoverflow.com/questions/61990418/unable-to-access-application-through-minikube-tunnel)

### 11.3  TLS 域名访问
创建证书

```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
```
根据证书生成 secret

```bash
kubectl -n default create secret tls mkcert --key key.pem --cert cert.pem
```

创建 app 应用
```bash
kubectl create deployment hello-minikube1 --image=docker.io/nginx:1.23
kubectl expose deployment hello-minikube1  --port=80
```
查看 svc

```bash
$ k get svc hello-minikube1
NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
hello-minikube1   ClusterIP   10.99.155.128   <none>        80/TCP   8m10s
```
编写 `tls-ingress-nginx` 文件

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secure-ingress-hello
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
  - hosts:
      - minikube.nginx.com
    secretName: mkcert
  rules:
  - host: minikube.nginx.com
    http:
      paths:
      - path: /hello
        pathType: Prefix
        backend:
          service:
            name: hello-minikube1
            port:
              number: 80
```
查看域名获取地址

```bash
$ k get ingress --watch
NAME                   CLASS   HOSTS                ADDRESS         PORTS     AGE
secure-ingress-hello   nginx   minikube.nginx.com   192.168.10.25   80, 443   9m43s
```
访问：`https://minikube.nginx.com/hello`

![](https://i-blog.csdnimg.cn/blog_migrate/3481df071330e1f27d99a2dd08001c83.png)





---
✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>

 - [更多 Minikube 操作请参阅](https://minikube.sigs.k8s.io/docs/)
 - [kind 部署 kubernetes 集群](https://blog.csdn.net/xixihahalelehehe/article/details/121968488)
 - [Minikube 在 Ubuntu 部署 Kubernetes](https://blog.csdn.net/xixihahalelehehe/article/details/113527867) 
 -  [Minikube 在 Centos 7 部署 Kubernetes](https://ghostwritten.blog.csdn.net/article/details/123796854)
 - [kubeadm 部署 kubernetes 集群](https://blog.csdn.net/xixihahalelehehe/article/details/105567076)
 - [云原生圣经](https://ghostwritten.blog.csdn.net/article/details/108562082)

![](https://i-blog.csdnimg.cn/blog_migrate/fae55181f0b02b88ee44e58179bc4683.gif#pic_center)






