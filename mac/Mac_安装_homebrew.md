
![在这里插入图片描述](https://img-blog.csdnimg.cn/cfd1565e219944ba84033f4c6c3cc339.png)


## 1. 简介
omebrew是一款包管理工具，目前支持macOS和linux系统。主要有四个部分组成: brew、homebrew-core 、homebrew-cask、homebrew-bottles。

|  名称| 说明 |
|--|--|
|brew	|Homebrew 源代码仓库|
|homebrew-core	|Homebrew 核心源|
|homebrew-cask	|提供 macOS 应用和大型二进制文件的安装|
|homebrew-bottles	|预编译二进制软件包## 官方安装|

## 2. 安装

### 2.1 官方安装
参考地址：[https://brew.sh/](https://brew.sh/)
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## 2.2 安装 ARM 版 Homebrew
ARM版Homebrew需要安装在/opt/homebrew路径下，早期的时候需要手动创建目录执行命令，目前使用最新脚本不需要手动操作。

直接执行：

```bash
/bin/bash -c "$(curl -fsSL https://gitee.com/ineo6/homebrew-install/raw/master/install.sh)"
```
然后还需设置环境变量，具体操作步骤如下，一定要仔细阅读。

PS: 终端类型根据执行命令`echo $SHELL`显示的结果：

```bash
/bin/bash => bash => .bash_profile
/bin/zsh => zsh => .zprofile
```

如果遇到环境变量无效问题，建议回过头来查看终端类型，再做正确的设置。

从`macOS Catalina(10.15.x)` 版开始，Mac使用`zsh`作为默认Shell，使用`.zprofile`，所以对应命令：

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

如果是`macOS Mojave` 及更低版本，并且没有自己配置过zsh，使用`.bash_profile`：

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.bash_profile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

## 2.3 安装 X86 版 Homebrew
为目前很多软件包没有支持ARM架构，我们也可以考虑使用x86版的Homebrew。

在命令前面添加`arch -x86_64`，就可以按X86模式执行该命令，比如：

```bash
arch -x86_64 /bin/bash -c "$(curl -fsSL https://gitee.com/ineo6/homebrew-install/raw/master/install.sh)"
```

### 2.4 多版本共存
如果你同时安装了`ARM`和`X86`两个版本，那你需要设置别名，把命令区分开。

同样是`.zprofile`或者`.bash_profile`里面添加：

至于操作哪个文件，请参考前文关于终端类型的描述，下文如有类似文字，保持一样的操作。

```bash
alias abrew='arch -arm64 /opt/homebrew/bin/brew'
alias ibrew='arch -x86_64 /usr/local/bin/brew'
```

abrew、ibrew可以根据你的喜好自定义。

然后再执行`source ~/.zprofile`或`source ~/.bash_profile`命令更新文件。


## 3. 设置镜像
`brew`、`homebrew/core`是必备项目，`homebrew/cask`、`homebrew/bottles`按需设置。

通过 `brew config` 命令可以查看相关配置信息。

### 3.1 初次安装brew后配置中科大 zsh

```bash
# 1.执行安装脚本
export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.ustc.edu.cn/brew.git"
export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.ustc.edu.cn/homebrew-core.git"
/bin/bash -c "$(curl -fsSL https://gitee.com/ineo6/homebrew-install/raw/master/install.sh)"

# 2.安装完成后设置
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles/bottles' >> ~/.zprofile
source ~/.zprofile
```

### 3.2 换源配置中科大 zsh

```bash
# 脚本
git -C "$(brew --repo)" remote set-url origin https://mirrors.ustc.edu.cn/brew.git
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git
brew update

echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles/bottles' >> ~/.zprofile
source ~/.zprofile
```

### 3.3 换源清华大学 zsh

```bash
# 脚本
git -C "$(brew --repo)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask.git
brew update

echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles' >> ~/.zprofile
source ~/.zprofile
```
更多[配置镜像细节请参考](https://brew.idayer.com/guide/change-source)。


### 3.4 更新

```bash
brew update-reset 
or
brew update -v
```

## 4. 问题
`raw.githubusercontent.com`地址不稳定，导致无法访问官方安装脚本`install.sh`。

```bash
curl: (7) Failed to connect to raw.githubusercontent.com port 443: Operation timed out
```

解决方案就是托管到`jsdelivr`，通过`CDN`加速访问。

另外也可以采用写入`hosts`的方式，可以一定程度解决GitHub资源无法访问的问题。

设置方案请阅读 [GitHub 访问加速指南](https://mp.weixin.qq.com/s/gFNP2Pk81vg7nE1XsDingg)


参考：
- [https://brew.idayer.com/guide](https://brew.idayer.com/guide)
- [https://brew.sh/](https://brew.sh/)

