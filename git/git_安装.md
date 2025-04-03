# git 安装


---
## 1. 简介
Git 目前支持 Linux/Unix、Solaris、Mac和 Windows 平台上运行。
下载地址为：[http://git-scm.com/downloads](http://git-scm.com/downloads)

## 2. linux 安装
Linux 平台上安装
Git 的工作需要调用 curl，zlib，openssl，expat，libiconv 等库的代码，所以需要先安装这些依赖工具。
在有 yum 的系统上（比如 Fedora）或者有 apt-get 的系统上（比如 Debian 体系），可以用下面的命令安装：
各 Linux 系统可以使用其安装包管理工具（apt-get、yum 等）进行安装：
### 2.1 Debian/Ubuntu

```bash
$ apt-get install libcurl4-gnutls-dev libexpat1-dev gettext \
  libz-dev libssl-dev

$ apt-get install git

$ git --version
git version 1.8.1.2
```
###  2.2 Centos/RedHat

```bash
$ yum install curl-devel expat-devel gettext-devel \
  openssl-devel zlib-devel

$ yum -y install git-core

$ git --version
git version 1.7.1
```

##  3. 源码安装 (最新安装)

###  3.2 tar 安装
1. 安装依赖

```bash
yum -y upgrade
```

```bash
sudo yum -y install wget make autoconf automake cmake perl-CPAN libcurl-devel libtool gcc gcc-c++ glibc-headers zlib-devel git-lfs telnet lrzsz jq expat-devel openssl-devel
```
2. 安装 Git

```bash

cd /tmp
wget --no-check-certificate https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.45.0.tar.gz
tar -xvzf git-2.45.0.tar.gz
cd git-2.45.0/
./configure
make
sudo make install
```
按照上面的步骤安装好之后，我们要把 Git 的二进制目录添加到 PATH 路径中，不然 Git 可能会因为找不到一些命令而报错。你可以通过执行以下命令添加目录：

```bash

tee -a $HOME/.bashrc <<'EOF'
# Configure for git
export PATH=/usr/local/libexec/git-core:$PATH
EOF
source  $HOME/.bashrc
```

```bash
$ git --version          # 输出 git 版本号，说明安装成功
git version 2.45.0
```

```bash
git config --global user.name "ghostwritten"   
git config --global user.email "1zoxun1@gmail.com"   
git config --global credential.helper store    
git config --global core.longpaths true 
git config --global core.quotepath off
git lfs install --skip-repo
```
解释：

首先，在 Git 中，我们会把非 ASCII 字符叫做 `Unusual` 字符。这类字符在 Git 输出到终端的时候默认是用 8 进制转义字符输出的（以防乱码），但现在的终端多数都支持直接显示非 ASCII 字符，所以我们可以关闭掉这个特性，具体的命令如下：

```bash
git config --global core.quotepath off
```
其次，GitHub 限制最大只能克隆 `100M` 的单个文件，为了能够克隆大于 100M 的文件，我们还需要安装 `Git Large File Storage`，安装方式如下：

```bash
git lfs install --skip-repo
```
好啦，现在我们就完成了依赖的安装和配置。


##  4. Windows 安装
安装包下载地址：[https://gitforwindows.org/](https://gitforwindows.org/)
官网慢，可以用国内的镜像：[https://npm.taobao.org/mirrors/git-for-windows/](https://npm.taobao.org/mirrors/git-for-windows/)。
完成安装之后，就可以使用命令行的 git 工具（已经自带了 ssh 客户端）了，另外还有一个图形界面的 Git 项目管理工具。
在开始菜单里找到"Git"->"Git Bash"，会弹出 Git 命令窗口，你可以在该窗口进行 Git 操作。


##  5. Mac 安装
在 Mac 平台上安装 Git 最容易的当属使用图形化的 Git 安装工具，下载地址为：

[http://sourceforge.net/projects/git-osx-installer/](http://sourceforge.net/projects/git-osx-installer/)

安装界面如下所示：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3b976b821b31f7f24230fe4ec9e02ebb.png)

##  6. Git 配置
Git 提供了一个叫做 `git config` 的工具，专门用来配置或读取相应的工作环境变量。

这些环境变量，决定了 Git 在各个环节的具体工作方式和行为。这些变量可以存放在以下三个不同的地方：

 - `/etc/gitconfig` 文件：系统中对所有用户都普遍适用的配置。若使用 `git config` 时用 `--system`
   选项，读写的就是这个文件。
 - `~/.gitconfig` 文件：用户目录下的配置文件只适用于该用户。若使用 `git config` 时用 `--global`
   选项，读写的就是这个文件。

当前项目的 Git 目录中的配置文件（也就是工作目录中的 `.git/config` 文件）：这里的配置仅仅针对当前项目有效。每一个级别的配置都会覆盖上层的相同配置，所以 `.git/config` 里的配置会覆盖 /etc/gitconfig 中的同名变量。
在 Windows 系统上，Git 会找寻用户主目录下的 .gitconfig 文件。主目录即 $HOME 变量指定的目录，一般都是 `C:\Documents and Settings\$USER`。

此外，Git 还会尝试找寻 `/etc/gitconfig` 文件，只不过看当初 Git 装在什么目录，就以此作为根目录来定位。

###  6.1 用户信息
配置个人的用户名称和电子邮件地址：

```bash
$ git config --global user.name "ghostWritten"
$ git config --global user.email 1zoxun1@gmail.com
```
###  6.2 查看配置

```bash
$ git config --list
http.postbuffer=2M
user.name=ghostWritten
user.email=1zoxun1@gmail.com

$ git config user.name
runoob
```
这些配置我们也可以在 `~/.gitconfig` 或 `/etc/gitconfig` 看到，如下所示：

```bash
$ vim ~/.gitconfig 
[http]
    postBuffer = 2M
[user]
    name = ghostWritten
    email = 1zoxun1@gmail.com
```

---

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f09f35a7318e39d06f32b3f37a281bc9.gif#pic_center)

