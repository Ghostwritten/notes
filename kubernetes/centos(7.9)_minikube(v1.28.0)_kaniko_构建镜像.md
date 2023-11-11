

##  准备
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


## 安装工具
- [Centos docker 最新安装](https://blog.csdn.net/xixihahalelehehe/article/details/104293170)
- [安装最新 git](https://blog.csdn.net/xixihahalelehehe/article/details/125107061)
- [安装最新 minikube](https://blog.csdn.net/xixihahalelehehe/article/details/123796854)
- [安装 go 工具](https://blog.csdn.net/xixihahalelehehe/article/details/127652839)
- [安装 cri-dockerd](https://github.com/Mirantis/cri-dockerd.git)
- [安装 crictl](https://blog.csdn.net/xixihahalelehehe/article/details/116591151)

###  安装最新 docker

```bash
yum install -y yum-utils  device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum list docker-ce --showduplicates | sort -r
sudo yum -y install docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo systemctl start docker && systemctl enable docker
```
配置`/etc/docker/daemon.json`

```bash
$ cat /etc/docker/daemon.json
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
```

###  安装最新 git
1. 安装依赖
```bash
sudo yum -y install make autoconf automake cmake perl-CPAN libcurl-devel libtool gcc gcc-c++ glibc-headers zlib-devel git-lfs telnet lrzsz jq expat-devel openssl-devel

```
2. 安装 Git

```bash
cd /tmp
wget --no-check-certificate https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.38.1.tar.gz
tar -xvzf git-2.38.1.tar.gz
cd git-2.38.1/
./configure
make
sudo make install
```
查看版本
```bash
$ git --version          # 输出 git 版本号，说明安装成功
git version 2.38.1
```

###  安装最新 minikube
1. 安装依赖
```bash
yum -y update
yum -y install apt-transport-https ca-certificates curl software-properties-common conntrack socat
```
2. 安装minikube

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo rpm -Uvh minikube-latest.x86_64.rpm
ln -s /usr/bin/minikube /usr/local/bin/
```

###  安装 go

```bash
wget -P /tmp/ https://golang.google.cn/dl/go1.18.3.linux-amd64.tar.gz
mkdir -p $HOME/go
tar -xvzf /tmp/go1.18.3.linux-amd64.tar.gz -C $HOME/go
mv $HOME/go/go $HOME/go/go1.18.3
tee -a $HOME/.bashrc <<'EOF'
# Go envs
export GOVERSION=go1.18.3 # Go 版本设置
export GO_INSTALL_DIR=$HOME/go # Go 安装目录
export GOROOT=$GO_INSTALL_DIR/$GOVERSION # GOROOT 设置
export GOPATH=$WORKSPACE/golang # GOPATH 设置
export PATH=$GOROOT/bin:$GOPATH/bin:$PATH # 将 Go 语言自带的和通过 go install 安装的二进制文件加入到 PATH 路径中
export GO111MODULE="on" # 开启 Go moudles 特性
export GOPROXY=https://goproxy.cn,direct # 安装 Go 模块时，代理服务器设置
export GOPRIVATE=
export GOSUMDB=off # 关闭校验 Go 依赖包的哈希值
EOF
```
###  安装 cri-dockerd

```bash
git clone https://github.com/Mirantis/cri-dockerd.git
cd cri-dockerd
mkdir bin
go build -o bin/cri-dockerd
mkdir -p /usr/local/bin
install -o root -g root -m 0755 bin/cri-dockerd /usr/local/bin/cri-dockerd
cp -a packaging/systemd/* /etc/systemd/system
sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
systemctl daemon-reload
systemctl enable cri-docker.service
systemctl enable --now cri-docker.socket
```

###  安装 kubectl
1. 配置kubernetes源

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
```
2. 安装kubectl

```bash
yum -y install kubectl
```
###  安装 crictl

```bash
wget https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.25.0/crictl-v1.25.0-linux-amd64.tar.gz
tar zxvf crictl-v1.25.0-linux-amd64.tar.gz
mv crictl /usr/bin/
ln -s /usr/bin/crictl /usr/local/bin/
```

## 获取kaniko demo

```bash
$ git clone https://github.com/vfarcic/kaniko-demo.git


$ ls
archetypes  codefresh-master.yml  config.toml  content  Dockerfile  docker-socket.yaml  docker.yaml  kaniko-dir.yaml  kaniko-git.yaml  layouts  Makefile  README.md  static  themes
```
构建镜像
```bash

$ cat Dockerfile 
FROM klakegg/hugo:0.78.2-alpine AS build
RUN apk add -U git
COPY . /src
RUN make init
RUN make build

FROM nginx:1.19.4-alpine
RUN mv /usr/share/nginx/html/index.html /usr/share/nginx/html/old-index.html
COPY --from=build /src/public /usr/share/nginx/html
EXPOSE 80
```
```bash
$ docker image  build --tag devops-toolkit .
Sending build context to Docker daemon  17.47MB
Step 1/9 : FROM klakegg/hugo:0.78.2-alpine AS build
0.78.2-alpine: Pulling from klakegg/hugo
188c0c94c7c5: Pull complete
3700113bd9c3: Pull complete
b28fe74b4e21: Pull complete
Digest: sha256:854c71812b94e50ca079cb99a3b6d63025d5bba4ac40e30190d1af81e0c1bbb2
Status: Downloaded newer image for klakegg/hugo:0.78.2-alpine
 ---> 5729af47368d
Step 2/9 : RUN apk add -U git
 ---> Running in b24aa8da3380
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/community/x86_64/APKINDEX.tar.gz
(1/7) Installing ca-certificates (20220614-r0)
(2/7) Installing nghttp2-libs (1.41.0-r0)
(3/7) Installing libcurl (7.79.1-r1)
(4/7) Installing expat (2.2.10-r4)
(5/7) Installing pcre2 (10.35-r0)
(6/7) Installing git (2.26.3-r1)
(7/7) Installing git-bash-completion (2.26.3-r1)
Executing busybox-1.31.1-r19.trigger
Executing ca-certificates-20220614-r0.trigger
OK: 30 MiB in 30 packages
Removing intermediate container b24aa8da3380
 ---> de31bcb919a5
Step 3/9 : COPY . /src
 ---> c11f6c96d5aa
Step 4/9 : RUN make init
 ---> Running in bfe8b5c4be17
git submodule init
Submodule 'themes/forty' (https://github.com/MarcusVirg/forty) registered for path 'themes/forty'
git submodule update
Cloning into '/src/themes/forty'...
fatal: unable to access 'https://github.com/MarcusVirg/forty/': HTTP/2 stream 1 was not closed cleanly before end of the underlying stream
fatal: clone of 'https://github.com/MarcusVirg/forty' into submodule path '/src/themes/forty' failed
Failed to clone 'themes/forty'. Retry scheduled
Cloning into '/src/themes/forty'...
Submodule path 'themes/forty': checked out 'dccea57bd2ed194942080d650671b47b6df4183c'
Submodule path 'themes/forty': checked out 'dccea57bd2ed194942080d650671b47b6df4183c'
cp content/img/banner.jpg themes/forty/static/img/.
Removing intermediate container bfe8b5c4be17
 ---> a8abb6650acf
Step 5/9 : RUN make build
 ---> Running in e0c8593d62e1
hugo
Start building sites …

                   | EN
-------------------+-----
  Pages            | 19
  Paginator pages  |  0
  Non-page files   | 24
  Static files     | 97
  Processed images |  0
  Aliases          |  0
  Sitemaps         |  1
  Cleaned          |  0

Total in 141 ms
Removing intermediate container e0c8593d62e1
 ---> c6eb1d4b7e00
Step 6/9 : FROM nginx:1.19.4-alpine
1.19.4-alpine: Pulling from library/nginx
188c0c94c7c5: Already exists
0ca72de6f957: Pull complete
9dd8e8e54998: Pull complete
f2dc206a393c: Pull complete
85defa007a8b: Pull complete
Digest: sha256:9b22bb6d703d52b079ae4262081f3b850009e80cd2fc53cdcb8795f3a7b452ee
Status: Downloaded newer image for nginx:1.19.4-alpine
 ---> e5dcd7aa4b5e
Step 7/9 : RUN mv /usr/share/nginx/html/index.html /usr/share/nginx/html/old-index.html
 ---> Running in 9f88bb1fc9b3
Removing intermediate container 9f88bb1fc9b3
 ---> 39f103e93691
Step 8/9 : COPY --from=build /src/public /usr/share/nginx/html
 ---> ebc5241c29ee
Step 9/9 : EXPOSE 80
 ---> Running in c45fb191f497
Removing intermediate container c45fb191f497
 ---> 0303b32504b0
Successfully built 0303b32504b0
Successfully tagged devops-toolkit:latest
```
##  部署minikube集群

```bash
minikube start  --vm-driver=none --image-mirror-country=cn --registry-mirror='https://ckdhnbk9.mirror.aliyuncs.com' --image-repository='registry.cn-hangzhou.aliyuncs.com/google_containers'  --kubernetes-version=v1.23.8
```

输出：
```bash
😄  minikube v1.28.0 on Centos 7.9.2009
✨  Using the none driver based on user configuration
✅  Using image repository registry.cn-hangzhou.aliyuncs.com/google_containers
👍  Starting control plane node minikube in cluster minikube
🤹  Running on localhost (CPUs=4, Memory=7958MB, Disk=26607MB) ...
ℹ️  OS release is CentOS Linux 7 (Core)
    > kubectl.sha256:  64 B / 64 B [-------------------------] 100.00% ? p/s 0s
    > kubelet.sha256:  64 B / 64 B [-------------------------] 100.00% ? p/s 0s
    > kubeadm.sha256:  64 B / 64 B [-------------------------] 100.00% ? p/s 0s
    > kubectl:  44.44 MiB / 44.44 MiB [--------------] 100.00% 4.35 MiB p/s 10s
    > kubeadm:  43.12 MiB / 43.12 MiB [--------------] 100.00% 3.65 MiB p/s 12s
    > kubelet:  118.78 MiB / 118.78 MiB [------------] 100.00% 6.37 MiB p/s 19s

    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🤹  Configuring local host environment ...

❗  The 'none' driver is designed for experts who need to integrate with an existing VM
💡  Most users should use the newer 'docker' driver instead, which does not require root!
📘  For more information, see: https://minikube.sigs.k8s.io/docs/reference/drivers/none/

❗  kubectl and minikube configuration will be stored in /root
❗  To use kubectl or minikube commands as your own user, you may need to relocate them. For example, to overwrite your own settings, run:

    ▪ sudo mv /root/.kube /root/.minikube $HOME
    ▪ sudo chown -R $USER $HOME/.kube $HOME/.minikube

💡  This can also be done automatically by setting the env var CHANGE_MINIKUBE_NONE_USER=true
🔎  Verifying Kubernetes components...
    ▪ Using image registry.cn-hangzhou.aliyuncs.com/google_containers/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass

❗  /usr/bin/kubectl is version 1.25.4, which may have incompatibilities with Kubernetes 1.23.8.
    ▪ Want kubectl v1.23.8? Try 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default

```
测试

```bash
$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured

$ kubectl  get no
NAME        STATUS   ROLES                  AGE     VERSION
minikube1   Ready    control-plane,master   3m26s   v1.23.8

$ kubectl  get pods -A
NAMESPACE     NAME                                READY   STATUS    RESTARTS   AGE
kube-system   coredns-65c54cc984-xc4v4            1/1     Running   0          2m54s
kube-system   etcd-minikube1                      1/1     Running   0          3m5s
kube-system   kube-apiserver-minikube1            1/1     Running   0          3m5s
kube-system   kube-controller-manager-minikube1   1/1     Running   0          3m5s
kube-system   kube-proxy-n82vp                    1/1     Running   0          2m55s
kube-system   kube-scheduler-minikube1            1/1     Running   0          3m5s
kube-system   storage-provisioner                 1/1     Running   0          3m3s
```

##  kaniko 部署
fork from [https://github.com/vfarcic/kaniko-demo.git](https://github.com/vfarcic/kaniko-demo.git)
```bash
git clone https://github.com/Ghostwritten/kaniko-demo.git
```

