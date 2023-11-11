# centos 8.2 配置 go 项目开发环境
![在这里插入图片描述](https://img-blog.csdnimg.cn/96b873e7508a48169971dfd5444cc082.png)


##  1. 创建普通用户  
第一步，用 Root 用户登录 Linux 系统，并创建普通用户 

一般来说，一个项目会由多个开发人员协作完成，为了节省企业成本，公司不会给每个开发人员都配备一台服务器，而是让所有开发人员共用一个开发机，通过普通用户登录开发机进行开发。因此，为了模拟真实的企业开发环境，我们也通过一个普通用户的身份来进行项目的开发，创建方法如下：

```bash

# useradd going # 创建 going 用户，通过 going 用户登录开发机进行开发
# passwd going # 设置密码
Changing password for user going.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```
不仅如此，使用普通用户登录和操作开发机也可以保证系统的安全性，这是一个比较好的习惯，所以我们在日常开发中也要尽量避免使用 Root 用户。

## 2. 添加 sudoers
我们知道很多时候，普通用户也要用到 Root 的一些权限，但 Root 用户的密码一般是由系统管理员维护并定期更改的，每次都向管理员询问密码又很麻烦。因此，我建议你将普通用户加入到 sudoers 中，这样普通用户就可以通过 sudo 命令来暂时获取 Root 的权限。具体来说，你可以执行如下命令添加：


```bash
sed -i '/^root.*ALL=(ALL).*ALL/a\going\tALL=(ALL) \tALL' /etc/sudoers
```

## 3. 替换 CentOS 8.4 系统中自带的 Yum 源
由于 Red Hat 提前宣布 CentOS 8 于 2021 年 12 月 31 日停止维护，官方的 Yum 源已不可使用，所以需要切换官方的 Yum 源，这里选择阿里提供的 Yum 源。切换命令如下：

```bash

mv /etc/yum.repos.d /etc/yum.repos.d.bak # 先备份原有的 Yum 源
mkdir /etc/yum.repos.d
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-vault-8.5.2111.repo
yum clean all && yum makecache
```
用新的用户名（going）和密码登录 Linux 服务器。这一步也可以验证普通用户是否创建成功。

## 4. 配置 $HOME/.bashrc 文件
我们登录新服务器后的第一步就是配置 `$HOME/.bashrc` 文件，以使 Linux 登录 shell 更加易用，例如配置 `LANG` 解决中文乱码，配置 `PS1` 可以避免整行都是文件路径，并将 `$HOME/bin` 加入到 PATH 路径中。配置后的内容如下：

```bash

# .bashrc
 
# User specific aliases and functions
 
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
 
# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi
 
# User specific environment
# Basic envs
export LANG="en_US.UTF-8" # 设置系统语言为 en_US.UTF-8，避免终端出现中文乱码
export PS1='[\u@dev \W]\$ ' # 默认的 PS1 设置会展示全部的路径，为了防止过长，这里只展示："用户名@dev 最后的目录名"
export WORKSPACE="$HOME/workspace" # 设置工作目录
export PATH=$HOME/bin:$PATH # 将 $HOME/bin 目录加入到 PATH 变量中
 
# Default entry folder
cd $WORKSPACE # 登录系统，默认进入 workspace 目录
```
有一点需要我们注意，在 export PATH 时，最好把 `$PATH` 放到最后，因为我们添加到目录中的命令是期望被优先搜索并使用的。配置完 `$HOME/.bashrc` 后，我们还需要创建工作目录 workspace。将工作文件统一放在 `$HOME/workspace` 目录中，有几点好处。

- 可以使我们的`$HOME`目录保持整洁，便于以后的文件查找和分类。
- 如果哪一天 /分区空间不足，可以将整个 `workspace` 目录 mv 到另一个分区中，并在 /分区中保留软连接，例如：`/home/going/workspace -> /data/workspace/`。

- 如果哪天想备份所有的工作文件，可以直接备份 workspace。

具体的操作指令是`$ mkdir -p $HOME/workspace`。配置好 `$HOME/.bashrc` 文件后，我们就可以执行 bash 命令将配置加载到当前 shell 中了。

至此，我们就完成了 Linux 开发机环境的申请及初步配置。

## 5. 依赖安装和配置
在 Linux 系统上安装 IAM 系统会依赖一些 RPM 包和工具，有些是直接依赖，有些是间接依赖。为了避免后续的操作出现依赖错误，例如，因为包不存在而导致的编译、命令执行错误等，我们先统一依赖安装和配置。安装和配置步骤如下。

### 6. 安装依赖
首先，我们在 CentOS 系统上通过 yum 命令来安装所需工具的依赖，安装命令如下：

```bash

sudo yum -y install make autoconf automake cmake perl-CPAN libcurl-devel libtool gcc gcc-c++ glibc-headers zlib-devel git-lfs telnet lrzsz jq expat-devel openssl-devel
```
虽然有些 CentOS 8.2 系统已经默认安装这些依赖了，但是为了确保它们都能被安装，我仍然会尝试安装一遍。如果系统提示 `Package xxx is already installed.`，说明已经安装好了，你直接忽略即可。

### 6.1 安装 Git
低版本的 Git 不支持`--unshallow` 参数，而 `go get` 在安装 Go 包时会用到 `git fetch --unshallow` 命令，因此我们要确保安装一个高版本的 Git，具体的安装方法如下：

```bash
$ cd /tmp
$ wget --no-check-certificate https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.36.1.tar.gz
$ tar -xvzf git-2.36.1.tar.gz
$ cd git-2.36.1/
$ ./configure
$ make
$ sudo make install
$ git --version          # 输出 git 版本号，说明安装成功
git version 2.36.1
```
意啦，按照上面的步骤安装好之后，我们要把 Git 的二进制目录添加到 PATH 路径中，不然 Git 可能会因为找不到一些命令而报错。你可以通过执行以下命令添加目录：

```bash

tee -a $HOME/.bashrc <<'EOF'
# Configure for git
export PATH=/usr/local/libexec/git-core:$PATH
EOF
```
### 6.2 配置 Git

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

## 7. Go 编译环境安装和配置
除了 Go，我们也会用 gRPC 框架展示 RPC 通信协议的用法，所以我们也需要将 ProtoBuf 的.proto 文件编译成 Go 语言的接口。因此，我们也需要安装 ProtoBuf 的编译环境。

### 7.1 Go 编译环境安装和配置
安装 Go 语言相对来说比较简单，我们只需要下载源码包、设置相应的环境变量即可。首先，我们从 Go 语言官方网站下载对应的 Go 安装包以及源码包，这里我下载的是 `go1.18.3` 版本：

```bash
$ wget -P /tmp/ https://golang.google.cn/dl/go1.18.3.linux-amd64.tar.gz
```
接着，我们完成解压和安装，命令如下：

```bash
$ mkdir -p $HOME/go
$ tar -xvzf /tmp/go1.18.3.linux-amd64.tar.gz -C $HOME/go
$ mv $HOME/go/go $HOME/go/go1.18.3
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
什么要增加这么多环境变量呢？这是因为，Go 语言是通过一系列的环境变量来控制 Go 编译器行为的。因此，我们一定要理解每一个环境变量的含义。
![在这里插入图片描述](https://img-blog.csdnimg.cn/dfda26d335774a0087d43dbe23eb700f.png)
因为 Go 以后会用 `Go modules` 来管理依赖，所以我建议你将 `GO111MODULE` 设置为 `on`。

在使用模块的时候，`$GOPATH` 是无意义的，不过它还是会把下载的依赖储存在 `$GOPATH/pkg/mod` 目录中，也会把 `go install` 的二进制文件存放在 `$GOPATH/bin` 目录中。

另外，我们还要将`$GOPATH/bin`、`$GOROOT/bin` 加入到 Linux 可执行文件搜索路径中。这样一来，我们就可以直接在 `bash shell` 中执行 go 自带的命令，以及通过 `go install` 安装的命令。然后就是进行测试了，如果我们执行 `go version` 命令可以成功输出 Go 的版本，就说明 Go 编译环境安装成功。具体的命令如下：

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

### 7.2 ProtoBuf 编译环境安装
接着，我们再来安装 `protobuf` 的编译器 `protoc`。`protoc` 需要 `protoc-gen-go` 来完成 Go 语言的代码转换，因此我们需要安装 `protoc` 和 `protoc-gen-go` 这 2 个工具。它们的安装方法比较简单，你直接看我下面给出的代码和操作注释就可以了。
第一步：安装 protobuf
```bash
cd /tmp/
git clone -b v3.21.1 --depth=1 https://github.com/protocolbuffers/protobuf
cd protobuf
./autogen.sh
./configure
make
sudo make install
```
```bash
protoc --version # 查看 protoc 版本，成功输出版本号，说明安装成功
libprotoc 3.21.1

# 第二步：安装 protoc-gen-go
$ go install github.com/golang/protobuf/protoc-gen-go@v1.5.2
```
当你第一次执行 `go install` 命令的时候，因为本地无缓存，所以需要下载所有的依赖模块。因此安装速度会比较慢，请你耐心等待。

## 8. Go 开发 IDE 安装和配置
编译环境准备完之后，你还需要一个代码编辑器才能开始 Go 项目开发。为了提高开发效率，你还需要将这个编辑器配置成 Go IDE。

目前，[GoLand](https://www.jetbrains.com/go/)、[VSCode](https://code.visualstudio.com/) 这些 IDE 都很优秀，但它们都是 Windows 系统下的 IDE。在 Linux 系统下我们可以选择将 Vim 配置成 Go IDE。熟练 Vim IDE 操作之后，开发效率不输 GoLand 和 VSCode。有多种方法可以配置一个 Vim IDE，这里我选择使用 [vim-go](https://github.com/fatih/vim-go) 将 Vim 配置成一个 Go IDE。vim-go 是社区比较受欢迎的 Vim Go 开发插件，可以用来方便地将一个 Vim 配置成 Vim IDE。

Vim IDE 的安装和配置分为以下 2 步.:

### 8.1 安装 vim-go

```bash
$ rm -f $HOME/.vim; mkdir -p ~/.vim/pack/plugins/start/
$ git clone --depth=1 https://github.com/fatih/vim-go.git ~/.vim/pack/plugins/start/vim-go
```
### 8.2 Go 工具安装
vim-go 会用到一些 Go 工具，比如在函数跳转时会用到 guru、[godef](https://github.com/rogpeppe/godef) 工具，在格式化时会用到 [goimports](https://pkg.go.dev/golang.org/x/tools/cmd/goimports)，所以你也需要安装这些工具。安装方式如下：执行 `vi /tmp/test.go`，然后输入 `:GoInstallBinaries` 安装 vim-go 需要的工具。

安装后的 Go IDE 常用操作的按键映射如下表所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/e13d8da7c3654ad184cad22e3ab3b9e8.png)
## 9. 总结
这一讲，我们一起安装和配置了一个 Go 开发环境，为了方便你回顾，我将安装和配置过程绘制成了一个流程图，如下所示。
![在这里插入图片描述](https://img-blog.csdnimg.cn/86ca3a87b4da4e85b97f0b845f63c5d7.png)

## 10. 练习

1. 试着编写一个 `main.go`，在 main 函数中打印 `Hello World`，并执行 `go run main.go` 运行代码，测试 Go 开发环境。

2. 试着编写一个 `main.go`，代码如下：

```bash
package main

import "fmt"

func main() {
    fmt.Println("Hello World")
}
```
将鼠标放在 `Println` 上，键入 `Enter` 键跳转到函数定义处，键入 `Ctrl + I` 返回到跳转点。
