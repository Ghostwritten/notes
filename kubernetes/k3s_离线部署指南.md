#  k3s 离线部署指南
tags：部署
![](https://i-blog.csdnimg.cn/blog_migrate/747705381099aac372fcd636273eeede.png)





## 1. 简介

K3s 是一个轻量级的 Kubernetes 发行版，在 2020 年统计的 K3s 下载量中，K3s 的全球下载量已经超过 100 万次，每周平均被安装超过 2 万次，其中 30%的下载量来自于中国。在国内已经有许多用户将 K3s 应用到了各种边缘计算和物联网设备中，同时也被广泛应用于智能工厂部署的生产线机器人和一些世界上最大型的风力发电厂当中。

针对生产环境下的 K3s，一个不可逾越的问题就是离线安装。在你的离线环境需要准备以下 3 个组件：

 - [K3s 的安装脚本](https://get.k3s.io/)
 - K3s 的二进制文件
 - K3s 依赖的镜像

通过K3s Release页面（[https://github.com/k3s-io/k3s/releases](https://github.com/k3s-io/k3s/releases) ）下载二进制文件与镜像，如果在国内使用，推荐从 [http://mirror.cnrancher.com](http://mirror.cnrancher.com) 获得这些组件。

离线安装的重点在于K3s 依赖的镜像部分，因为 K3s 的"安装脚本"和"二进制文件"只需要下载到对应目录，然后赋予相应的权限即可，非常简单。但K3s 依赖的镜像的安装方式取决于你使用的是手动部署镜像还是私有镜像仓库，也取决于容器运行时使用的是`containerd`还是`docker`。

针对不同的组合形式，可以分为以下几种形式来实现离线安装：

 - `Containerd` + 手动部署镜像方式
 - `Docker` + 手动部署镜像方式
 - `Containerd` + 私有镜像仓库方式
 - `Docker` + 私有镜像仓库方式

## 2. Docker + 手动部署镜像方式
假设你已经将同一版本的 K3s 的安装脚本(`k3s-install.sh`)、K3s 的二进制文件(`k3s`)、K3s 依赖的镜像(`k3s-airgap-images-amd64.tar`)下载到了/root目录下。

与 containerd 不同，使用 docker 作为容器运行时，启动 K3s 不会导入`/var/lib/rancher/k3s/agent/images/`目录下的镜像。所以在启动 K3s 之前我们需要将 K3s 依赖的镜像手动导入到 docker 镜像列表中。

```bash
$ ls
k3s  k3s-airgap-images-amd64.tar  k3s-install.sh

$  cp k3s /usr/local/bin/
```

### 2.1 安装docker
- [docker 安装](https://ghostwritten.blog.csdn.net/article/details/104293170)

### 2.2 导入镜像

```bash
$ docker load -i k3s-airgap-images-amd64.tar
```

```bash
4fc242d58285: Loading layer [==================================================>] 5.855 MB/5.855 MB
43ca363a6184: Loading layer [==================================================>]  17.5 MB/17.5 MB
ac85d27cf099: Loading layer [==================================================>] 90.36 MB/90.36 MB
f95c9a53e804: Loading layer [==================================================>] 125.6 MB/125.6 MB
Loaded image: rancher/klipper-helm:v0.7.3-build20220613
eb4bde6b29a6: Loading layer [==================================================>] 5.876 MB/5.876 MB
b73f2911df5c: Loading layer [==================================================>] 2.625 MB/2.625 MB
681d12cb8ee1: Loading layer [==================================================>] 3.584 kB/3.584 kB
Loaded image: rancher/klipper-lb:v0.3.5
8d3ac3489996: Loading layer [==================================================>] 5.866 MB/5.866 MB
2d6fddee6202: Loading layer [==================================================>] 29.39 MB/29.39 MB
Loaded image: rancher/local-path-provisioner:v0.0.21
256bc5c338a6: Loading layer [==================================================>] 336.4 kB/336.4 kB
5cc7f3ffa7e3: Loading layer [==================================================>] 49.33 MB/49.33 MB
Loaded image: rancher/mirrored-coredns-coredns:1.9.1
0b16ab2571f4: Loading layer [==================================================>] 1.459 MB/1.459 MB
Loaded image: rancher/mirrored-library-busybox:1.34.1
34d5ebaa5410: Loading layer [==================================================>] 5.866 MB/5.866 MB
694ee27c6a0b: Loading layer [==================================================>] 2.852 MB/2.852 MB
e32614d8591c: Loading layer [==================================================>] 99.74 MB/99.74 MB
44aad00a5195: Loading layer [==================================================>] 2.048 kB/2.048 kB
Loaded image: rancher/mirrored-library-traefik:2.9.1
5b1fa8e3e100: Loading layer [==================================================>] 3.697 MB/3.697 MB
3dc34f14eb83: Loading layer [==================================================>] 66.43 MB/66.43 MB
Loaded image: rancher/mirrored-metrics-server:v0.6.1
1021ef88c797: Loading layer [==================================================>] 684.5 kB/684.5 kB
Loaded image: rancher/mirrored-pause:3.6

$  docker images
REPOSITORY                         TAG                    IMAGE ID            CREATED             SIZE
rancher/mirrored-library-traefik   2.9.1                  e6de8578b238        4 weeks ago         107 MB
rancher/mirrored-library-busybox   1.34.1                 ff4a8eb070e1        4 weeks ago         1.24 MB
rancher/klipper-helm               v0.7.3-build20220613   38b3b9ad736a        4 months ago        239 MB
rancher/mirrored-coredns-coredns   1.9.1                  99376d8f35e0        7 months ago        49.5 MB
rancher/mirrored-metrics-server    v0.6.1                 e57a417f15d3        8 months ago        68.8 MB
rancher/klipper-lb                 v0.3.5                 dbd43b6716a0        9 months ago        8.09 MB
rancher/local-path-provisioner     v0.0.21                fb9b574e03c3        10 months ago       35 MB
rancher/mirrored-pause             3.6                    6270bb605e12        14 months ago       683 kB
```

### 2.3 安装 k3s

```bash
$ INSTALL_K3S_SKIP_DOWNLOAD=true INSTALL_K3S_EXEC='--docker' /root/k3s-install.sh
```
```bash
[INFO]  Skipping k3s download and verify
[INFO]  Skipping installation of SELinux RPM
[INFO]  Skipping /usr/local/bin/kubectl symlink to k3s, command exists in PATH at /usr/bin/kubectl
[INFO]  Skipping /usr/local/bin/crictl symlink to k3s, command exists in PATH at /usr/bin/crictl
[INFO]  Skipping /usr/local/bin/ctr symlink to k3s, command exists in PATH at /usr/bin/ctr
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
Created symlink from /etc/systemd/system/multi-user.target.wants/k3s.service to /etc/systemd/system/k3s.service.
[INFO]  systemd: Starting k3s
```

### 2.4 查看

```bash
k3s kubectl get pods -A
NAMESPACE     NAME                                      READY   STATUS      RESTARTS   AGE
kube-system   local-path-provisioner-5b5579c644-fmq74   1/1     Running     0          59s
kube-system   coredns-75fc8f8fff-nhn4s                  1/1     Running     0          59s
kube-system   helm-install-traefik-crd-pksg7            0/1     Completed   0          60s
kube-system   metrics-server-5c8978b444-7tmd9           1/1     Running     0          59s
kube-system   svclb-traefik-db846a66-mq5wk              2/2     Running     0          32s
kube-system   traefik-9c6dc6686-fpj6s                   1/1     Running     0          32s
kube-system   helm-install-traefik-66zjk                0/1     Completed   2          60s
```
把 master 节点机器上的 `/etc/rancher/k3s/k3s.yaml` 文件内容写入到 `~/.kube/config`文件，不要忘记修改 server 地址改为 master 节点地址：

```bash
$ kubectl  get node
Unable to connect to the server: x509: certificate signed by unknown authority

$ cp /root/.kube/config /tmp/kubeconfig
$ cp /etc/rancher/k3s/k3s.yaml /root/.kube/config
cp: overwrite ‘/root/.kube/config’? y

$ vim /root/.kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJlRENDQVIyZ0F3SUJBZ0lCQURBS0JnZ3Foa2pPUFFRREFqQWpNU0V3SHdZRFZRUUREQmhyTTNNdGMyVnkKZG1WeUxXTmhRREUyTmpZM056UTJNakl3SGhjTk1qSXhNREkyTURnMU56QXlXaGNOTXpJeE1ESXpNRGcxTnpBeQpXakFqTVNFd0h3WURWUVFEREJock0zTXRjMlZ5ZG1WeUxXTmhRREUyTmpZM056UTJNakl3V1RBVEJnY3Foa2pPClBRSUJCZ2dxaGtqT1BRTUJCd05DQUFUaXJHV0RId25zcExvaU0rQ2t2cTd6d1JGRHZiZmdGVnBkbnZjbzB0d08KQjNvRVVpL2VKaFpRV3Y2N0xCc3VqYzBGOEZYSmFIdHpnLzJyVUVzWHNqY3RvMEl3UURBT0JnTlZIUThCQWY4RQpCQU1DQXFRd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBZEJnTlZIUTRFRmdRVS9ZNE4ySVFIdlVMaWwvclVzbkk4CmtxVHhSSEl3Q2dZSUtvWkl6ajBFQXdJRFNRQXdSZ0loQUp0Ym1JYlAwNGY3eTlqWmdic1MrMi9yck5mMk9TaVkKVFpZOVR0Q0tsM3p2QWlFQXdLNGV3ZkgwUDhiRWlyYmhuYXF6QmFTd0pnQW9ycU9XclhnaHY4eXByQkU9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    server: https://k3s_master:6443   #修改127.0.0.1为master hostname（k3s_master）
  name: default
contexts:
......

$ kubectl  get ns
NAME              STATUS   AGE
default           Active   4m45s
kube-system       Active   4m45s
kube-public       Active   4m45s
kube-node-lease   Active   4m45s
$  kubectl  get node
NAME        STATUS   ROLES                  AGE     VERSION
k3smaster   Ready    control-plane,master   4m50s   v1.25.3+k3s1
```




## 3. Containerd + 手动部署镜像方式
假设你已经将同一版本的 K3s 的安装脚本([k3s-install.sh](https://get.k3s.io/))、[K3s 的二进制文件(k3s)](https://github.com/k3s-io/k3s/releases)、[K3s 依赖的镜像](https://github.com/k3s-io/k3s/releases)(`k3s-airgap-images-amd64.tar`)下载到了/root目录下。

如果你使用的容器运行时为`containerd`，在启动 K3s 时，它会检查`/var/lib/rancher/k3s/agent/images/`是否存在可用的镜像压缩包，如果存在，就将该镜像导入到containerd 镜像列表中。所以我们只需要下载 K3s 依赖的镜像到`/var/lib/rancher/k3s/agent/images/`目录，然后启动 K3s 即可。


### 3.1 导入镜像到 containerd 镜像列表

```bash
sudo mkdir -p /var/lib/rancher/k3s/agent/images/
sudo cp /root/k3s-airgap-images-amd64.tar /var/lib/rancher/k3s/agent/images/
```
### 3.2 授予可执行权限

```bash
sudo chmod a+x /root/k3s /root/k3s-install.sh
sudo cp /root/k3s /usr/local/bin/
```
### 3.3 安装 K3s

```bash
INSTALL_K3S_SKIP_DOWNLOAD=true /root/k3s-install.sh
```
稍等片刻，即可查看到 K3s 已经成功启动：

```bash
$ crictl images
IMAGE                                      TAG                 IMAGE ID            SIZE
docker.io/rancher/coredns-coredns          1.8.0               296a6d5035e2d       42.6MB
docker.io/rancher/klipper-helm             v0.3.2              4be09ab862d40       146MB
docker.io/rancher/klipper-lb               v0.1.2              897ce3c5fc8ff       6.46MB
docker.io/rancher/library-busybox          1.31.1              1c35c44120825       1.44MB
docker.io/rancher/library-traefik          1.7.19              aa764f7db3051       86.6MB
docker.io/rancher/local-path-provisioner   v0.0.14             e422121c9c5f9       42MB
docker.io/rancher/metrics-server           v0.3.6              9dd718864ce61       41.2MB
docker.io/rancher/pause                    3.1                 da86e6ba6ca19       746kB

$ kubectl get pods -A
NAMESPACE     NAME                                      READY   STATUS      RESTARTS   AGE
kube-system   local-path-provisioner-7c458769fb-zdg9z   1/1     Running     0          38s
kube-system   coredns-854c77959c-696gk                  1/1     Running     0          38s
kube-system   metrics-server-86cbb8457f-hs6vw           1/1     Running     0          38s
kube-system   helm-install-traefik-4pgcr                0/1     Completed   0          38s
kube-system   svclb-traefik-bq7wl                       2/2     Running     0          17s
kube-system   traefik-6f9cbd9bd4-jccd7                  1/1     Running     0          17s

```

## 4. Containerd + 私有镜像仓库方式
假设你已经将同一版本的 K3s 的安装脚本(k3s-install.sh)、K3s 的二进制文件(k3s)下载到了/root目录下。并且 K3s 所需要的镜像已经上传到了镜像仓库（本例的镜像仓库地址为：http://192.168.64.44:5000）。K3s 所需的镜像列表可以从 K3s Release页面的`k3s-images.txt`获得。

### 4.1 配置 K3s 镜像仓库

启动 K3s 默认会从`docker.io`拉取镜像。使用`containerd`容器运行时在离线安装时，我们只需要将镜像仓库地址配置到`docker.io`下的`endpoint`即可，更多配置说明请参考配置 `containerd` 镜像仓库完全攻略或K3s 官方文档：

[https://docs.rancher.cn/docs/k3s/installation/private-registry/_index/](https://docs.rancher.cn/docs/k3s/installation/private-registry/_index/)

```bash
sudo mkdir -p /etc/rancher/k3s
sudo cat >> /etc/rancher/k3s/registries.yaml <<EOF
mirrors:
  "docker.io":
    endpoint:
      - "http://192.168.64.44:5000"
      - "https://registry-1.docker.io"
EOF

```
### 4.2 授予可执行权限

```bash
sudo chmod a+x /root/k3s /root/k3s-install.sh
sudo cp /root/k3s /usr/local/bin/
```

### 4.3 安装 K3s

```bash
INSTALL_K3S_SKIP_DOWNLOAD=true /root/k3s-install.sh
```
稍等片刻，即可查看到 K3s 已经成功启动：

```bash
$ crictl images
IMAGE                                      TAG                 IMAGE ID            SIZE
docker.io/rancher/coredns-coredns          1.8.0               296a6d5035e2d       12.9MB
docker.io/rancher/klipper-helm             v0.3.2              4be09ab862d40       50.7MB
docker.io/rancher/klipper-lb               v0.1.2              897ce3c5fc8ff       2.71MB
docker.io/rancher/library-traefik          1.7.19              aa764f7db3051       24MB
docker.io/rancher/local-path-provisioner   v0.0.14             e422121c9c5f9       13.4MB
docker.io/rancher/metrics-server           v0.3.6              9dd718864ce61       10.5MB
docker.io/rancher/pause                    3.1                 da86e6ba6ca19       326kB

$ kubectl get pods -A
NAMESPACE     NAME                                      READY   STATUS      RESTARTS   AGE
kube-system   local-path-provisioner-7c458769fb-7w8hb   1/1     Running     0          37s
kube-system   coredns-854c77959c-f8m2n                  1/1     Running     0          37s
kube-system   helm-install-traefik-9lbrx                0/1     Completed   0          38s
kube-system   svclb-traefik-x8f6f                       2/2     Running     0          29s
kube-system   metrics-server-86cbb8457f-f7lb7           1/1     Running     0          37s
kube-system   traefik-6f9cbd9bd4-4s66r                  1/1     Running     0          29s

```

## 5. Docker + 私有镜像仓库方式
- [私有镜像仓库搭建教程请参考](https://ghostwritten.blog.csdn.net/article/details/105926147)

设你已经将同一版本的 K3s 的安装脚本([k3s-install.sh](https://get.k3s.io/))、[K3s 的二进制文件](https://github.com/k3s-io/k3s/releases)(k3s)下载到了/root目录下。并且 K3s 所需要的镜像已经上传到了镜像仓库（本例的镜像仓库地址为：`http://192.168.64.44:5000`）。K3s 所需的镜像列表可以从 K3s Release页面的[k3s-images.txt](https://github.com/k3s-io/k3s/releases)获得。


### 5.1 配置 K3s 镜像仓库

Docker 不支持像 `containerd` 那样可以通过修改 `docker.io` 对应的 `endpoint`（默认为 `https://registry-1.docker.io`）来间接修改默认镜像仓库的地址。但在Docker中可以通过配置registry-mirrors来实现从其他镜像仓库中获取K3s镜像。这样配置之后，会先从registry-mirrors配置的地址拉取镜像，如果获取不到才会从默认的docker.io获取镜像，从而满足了我们的需求。

```bash
cat >> /etc/docker/daemon.json <<EOF
{
    "registry-mirrors": ["http://192.168.64.44:5000"]
}
EOF

sudo systemctl daemon-reload
sudo systemctl restart docker

```
### 5.2 授予可执行权限

```bash
sudo chmod a+x /root/k3s /root/k3s-install.sh
sudo cp /root/k3s /usr/local/bin/
```
### 5.3 安装 K3s

```bash
INSTALL_K3S_SKIP_DOWNLOAD=true INSTALL_K3S_EXEC='--docker' /root/k3s-install.sh
```
稍等片刻，即可查看到 K3s 已经成功启动：

```bash
$ docker images
REPOSITORY                       TAG                 IMAGE ID            CREATED             SIZE
rancher/klipper-helm             v0.3.2              4be09ab862d4        7 weeks ago         145MB
rancher/coredns-coredns          1.8.0               296a6d5035e2        2 months ago        42.5MB
rancher/local-path-provisioner   v0.0.14             e422121c9c5f        7 months ago        41.7MB
rancher/library-traefik          1.7.19              aa764f7db305        14 months ago       85.7MB
rancher/metrics-server           v0.3.6              9dd718864ce6        14 months ago       39.9MB
rancher/klipper-lb               v0.1.2              897ce3c5fc8f        19 months ago       6.1MB
rancher/pause                    3.1                 da86e6ba6ca1        3 years ago         742kB

$ kubectl get pods -A
NAMESPACE     NAME                                      READY   STATUS      RESTARTS   AGE
kube-system   helm-install-traefik-bcclh                0/1     Completed   0          33s
kube-system   coredns-854c77959c-kp85f                  1/1     Running     0          33s
kube-system   metrics-server-86cbb8457f-85fpd           1/1     Running     0          33s
kube-system   local-path-provisioner-7c458769fb-r5nkw   1/1     Running     0          33s
kube-system   svclb-traefik-rbmhk                       2/2     Running     0          24s
kube-system   traefik-6f9cbd9bd4-k6t9n                  1/1     Running     0          24s

```
## 6. 卸载
如果您使用安装脚本安装了 K3s，那么在安装过程中会生成一个卸载 K3s 的脚本。

> 卸载 K3s 会删除集群数据和所有脚本。要使用不同的安装选项重新启动集群，请使用不同的标志重新运行安装脚本。

要从 server 节点卸载 K3s，请运行：

```bash
/usr/local/bin/k3s-uninstall.sh
```

要从 agent 节点卸载 K3s，请运行：

```bash
/usr/local/bin/k3s-agent-uninstall.sh
```

参考：
- [k3s 离线安装文档](https://docs.rancher.cn/docs/k3s/installation/airgap/_index/)
- [一文搞定全场景K3s离线安装](https://www.cnblogs.com/k3s2019/p/14339547.html) 
- [https://get.k3s.io/](https://get.k3s.io/)
- [https://github.com/k3s-io/k3s/releases](https://github.com/k3s-io/k3s/releases)
