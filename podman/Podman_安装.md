#  Podman 安装
![在这里插入图片描述](https://img-blog.csdnimg.cn/91054e6110a14acb97c51f040781ae11.png)



##  简介
[Podman](https://podman.io/)（全称 POD 管理器）是一款用于在 [Linux®](https://www.redhat.com/zh/topics/linux/what-is-linux) 系统上开发、管理和运行容器的开源工具。Podman 最初由红帽® 工程师联合开源社区一同开发，它可利用 `lipod` 库来管理整个容器生态系统。 

Podman 采用无守护进程的包容性架构，因此可以更安全、更简单地进行容器管理，再加上 [Buildah](https://www.redhat.com/zh/topics/containers/what-is-buildah) 和 [Skopeo](https://blog.csdn.net/xixihahalelehehe/article/details/127342981) 等与之配套的工具和功能，开发人员能够按照自身需求来量身定制容器环境。 它与[Docker](https://www.docker.com/)扮演相同的角色，并且在很大程度上与 Docker 兼容，提供几乎相同的命令。

## 1.  dnf 安装
第一种方式
```bash
sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
dnf update
dnf upgrade
dnf install -y epel-release
```
dnf 命令详细 请参考 [centos 8.2 指南](https://ghostwritten.blog.csdn.net/article/details/127641551)与[Linux Command Dnf 包管理工具](https://ghostwritten.blog.csdn.net/article/details/123168620)

```bash
$ dnf --showduplicates list podman
Last metadata expiration check: 0:06:19 ago on Fri 18 Nov 2022 10:24:42 AM CST.
Available Packages
podman.x86_64                                        3.3.1-9.module_el8.5.0+988+b1f0b741                                        appstream

$  dnf --showduplicates list buildah
Last metadata expiration check: 0:07:03 ago on Fri 18 Nov 2022 10:24:42 AM CST.
Available Packages
buildah.x86_64                                       1.22.3-2.module_el8.5.0+911+f19012f9                                       appstream

$ dnf install podman skopeo buildah
```

第二种方式
```bash
sudo dnf config-manager --set-enabled powertools
sudo dnf -y update
```

```bash
$ sudo dnf module list | grep container-tools
container-tools      rhel8 [d][e]     common [d]                               Most recent (rolling) versions of podman, buildah, skopeo, runc, conmon, runc, conmon, CRIU, Udica, etc as well as dependencies such as container-selinux built and tested together, and updated as frequently as every 12 weeks.

$ sudo dnf install -y @container-tools    

$ podman version
Client:       Podman Engine
Version:      4.0.2
API Version:  4.0.2
Go Version:   go1.17.7

Built:      Sun May 15 19:45:11 2022
OS/Arch:    linux/amd64
```

配置`podman`别名为`docker`

```bash
$ vim  /root/.bashrc
alias docker='podman'
$ source /root/.bashrc
$ docker info
```
##  2. yum 安装

```bash
sudo yum -y remove podman
```

```bash
yum update
yum install epel-release
yum install -y podman
```

## 3. apt-get 安装

```bash
sudo sh -c "echo 'deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/testing/x${NAME}_${VERSION_ID}/ /' > /etc/apt/sources.list.d/devel:kubic:libcontainers:testing.list"
wget -nv https://download.opensuse.org/repositories/devel:kubic:libcontainers:testing/x${NAME}_${VERSION_ID}/Release.key -O Release.key
sudo apt-key add - < Release.key
sudo apt-get update -qq
sudo apt-get -qq -y install podman
$ sudo apt-get  install podman
```
##  4. 源码安装
### 4.1 安装依赖

```bash
sudo yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
sudo yum -y update
sudo reboot
```


```bash
sudo yum -y install "@Development Tools"

sudo yum -y install make autoconf automake cmake perl-CPAN libcurl-devel libtool gcc gcc-c++ glibc-headers zlib-devel git-lfs telnet lrzsz jq expat-devel openssl-devel

sudo yum install -y curl \
  gcc \
  make \
  device-mapper-devel \
  git \
  btrfs-progs-devel \
  conmon \
  containernetworking-plugins \
  containers-common \
  git \
  glib2-devel \
  glibc-devel \
  glibc-static \
  golang-github-cpuguy83-md2man \
  gpgme-devel \
  iptables \
  libassuan-devel \
  libgpg-error-devel \
  libseccomp-devel \
  libselinux-devel \
  pkgconfig \
  systemd-devel \
  autoconf \
  python3 \
  python3-devel \
  python3-pip \
  yajl-devel \
  libcap-devel
```
or

```bash
sudo dnf -y install make autoconf automake cmake perl-CPAN libcurl-devel libtool gcc gcc-c++ glibc-headers zlib-devel git-lfs telnet lrzsz jq expat-devel openssl-devel
```

```bash
在这里插入代码片
```

### 4.2 安装 Git
低版本的 Git 不支持`--unshallow` 参数，而 `go get` 在安装 Go 包时会用到 `git fetch --unshallow` 命令，因此我们要确保安装一个高版本的 Git，具体的安装方法如下：

```bash
cd /tmp
wget --no-check-certificate https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.38.1.tar.gz
tar -xvzf git-2.38.1.tar.gz
cd git-2.38.1/
./configure
make
sudo make install
git --version         
```
意啦，按照上面的步骤安装好之后，我们要把 Git 的二进制目录添加到 PATH 路径中，不然 Git 可能会因为找不到一些命令而报错。你可以通过执行以下命令添加目录：

```bash

tee -a $HOME/.bashrc <<'EOF'
# Configure for git
export PATH=/usr/local/libexec/git-core:$PATH
EOF

source  $HOME/.bashrc
```
配置 Git

```bash
git config --global user.name "ghostwritten"    # 用户名改成自己的
git config --global user.email "1zoxun1@gmail.com"    # 邮箱改成自己的
git config --global credential.helper store    # 设置 Git，保存用户名和密码
git config --global core.longpaths true # 解决 Git 中 'Filename too long' 的错误
```
除了按照上述步骤配置 Git 之外，我们还有几点需要注意。

首先，在 Git 中，我们会把非 ASCII 字符叫做 `Unusual` 字符。这类字符在 Git 输出到终端的时候默认是用 8 进制转义字符输出的（以防乱码），但现在的终端多数都支持直接显示非 ASCII 字符，所以我们可以关闭掉这个特性，具体的命令如下：

```bash
git config --global core.quotepath off
```
其次，GitHub 限制最大只能克隆 `100M` 的单个文件，为了能够克隆大于 100M 的文件，我们还需要安装 `Git Large File Storage`，安装方式如下：

```bash
git lfs install --skip-repo
```
好啦，现在我们就完成了依赖的安装和配置。
  
### 4.3 Go 编译环境安装和配置
安装 Go 语言相对来说比较简单，我们只需要下载源码包、设置相应的环境变量即可。首先，我们从 Go 语言官方网站下载对应的 Go 安装包以及源码包，这里我下载的是 `go1.18.3` 版本：

```bash
wget -P /tmp/ https://golang.google.cn/dl/go1.18.3.linux-amd64.tar.gz
```
接着，我们完成解压和安装，命令如下：

```bash
mkdir -p $HOME/go
tar -xvzf /tmp/go1.18.3.linux-amd64.tar.gz -C $HOME/go
mv $HOME/go/go $HOME/go/go1.18.3
```
接着，我们执行以下命令，将下列环境变量追加到`$HOME/.bashrc` 文件中。

```bash
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
```bash
$ bash
$ go version
go version go1.18.3 linux/amd64
```
最后，初始化工作区。使用的 Go 版本为 `go1.18.3`，`go1.18.3` 支持多模块工作区，所以这里也需要初始化工作区。初始化命令如下：

```bash
$ mkdir -p $GOPATH && cd $GOPATH
$ go work init
$ go env GOWORK # 执行此命令，查看 go.work 工作区文件路径
/home/going/workspace/golang/go.work
```

###  4.4 安装 conmon
conmon[2]是用C编写的，设计为具有较低的内存占用。conmon是容器管理器（如Podman或CRI-O）和单个容器的OCI运行时（如runc或crun）之间的监控程序和通信工具。

当容器运行时，conmon做两件事：

1. 提供一个套接字，用于连接到容器、保持打开容器的标准流并通过套接字转发它们。
2. 将容器流的内容写入日志文件（或systemd日志），以便在容器死亡后读取。

最后，当容器死亡时，conmon将记录其退出时间和代码，以供管理程序读取。

```bash
git clone https://github.com/containers/conmon
cd conmon
export GOCACHE="$(mktemp -d)"
make
sudo make podman 
```

```bash
$ conmon --version
conmon version 2.0.8
commit: f85c8b1ce77b73bcd48b2d802396321217008762
```
构建 runc 
```bash
git clone https://github.com/opencontainers/runc.git $GOPATH/src/github.com/opencontainers/runc
cd $GOPATH/src/github.com/opencontainers/runc
go work use .
make BUILDTAGS="selinux seccomp"
sudo cp runc /usr/bin/runc
cd ~/
```

```bash
$ runc --version
runc version 1.1.0+dev
commit: v1.1.0-383-gdf474535
spec: 1.0.2-dev
go: go1.18.3
libseccomp: 2.3.1
```

###  4.5 Setup CNI networking for Podman
创建`/etc/containers`用于存放 CNI 网络配置文件的目录。

```bash
sudo mkdir -p /etc/containers
```
下载配置示例并放置创建的目录：
```bash
sudo curl -L -o /etc/containers/registries.conf https://src.fedoraproject.org/rpms/containers-common/raw/main/f/registries.conf
sudo curl -L -o /etc/containers/policy.json https://src.fedoraproject.org/rpms/containers-common/raw/main/f/default-policy.json
```

### 4.6 在 CentOS 7 / RHEL 7 上安装 Podman 4.x

```bash
sudo yum -y install wget
```
从 Github 存储库下载最新版本的[Podman 源代码](https://github.com/containers/podman/releases)。

```bash
TAG=4.4.0
rm -rf podman*
wget https://github.com/containers/podman/archive/refs/tags/v${TAG}.tar.gz
```
tar使用命令提取下载的文件：

```bash
tar xvf v${TAG}.tar.gz
cd podman*/
make BUILDTAGS="selinux seccomp"
sudo make install PREFIX=/usr
```


参考：
- [How To Install Podman 4.x on CentOS 7 / RHEL 7](https://computingforgeeks.com/install-podman-4-centos-7-rhel-7/)
- [Podman 官方安装](https://podman.io/getting-started/installation)
