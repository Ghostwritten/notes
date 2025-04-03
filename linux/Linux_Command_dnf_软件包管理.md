#  Linux Command dnf 软件包管理

tags: 软件包管理



---

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/032777a9cefa8b668e719d5e18812e9d.gif#pic_center)

## 1. 简介
DNF 命令(Dandified yum) 是基于 RedHat 的系统的传统YUM 包管理器的下一代版本。它是 Fedora 22、CentOS8 和 RHEL8 的默认包管理器。它旨在替代 YUM。它使用 RPM 和libsolv（由 OpenSUSE 维护）进行包管理。

DNF旨在改善YUM的瓶颈，即性能、内存使用、依赖解决、速度和许多其他因素。DNF 使用 RPM、libsolv 和 hawkey 库进行包管理。虽然它没有在 CentOS 和 RHEL 7 中单独安装，但您可以 yum、dnf 并与 yum 一起使用它。

DNF的最新稳定版本是2015 年5 月 11 日发布的1.0（在撰写本文时）。它（以及所有以前的 DNF 版本）主要是用 Python 编写的，并在 GPL v2 许可下发布。

##  2. 语法

```bash
dnf [options] <command> [<args>...]
```

可用的命令包括`install`, `search`, `query`等。

`args`可以是特定于“命令”的包名、组名或子命令。

> 注意：要安装和删除软件包，您需要具有sudo 权限。由于我已经是 root（这不是一个好主意，但用于演示目的），我不会在任何命令前添加sudo。但请记住，在安装和删除软件包时必须预先添加 sudo。


##  3. 安装
DNF 在`RHEL / CentOS 7`的默认存储库中不可用。然而，`Fedora 22` 附带了正式实施的 DNF。

要在`RHEL/CentOS`系统上安装DNF，您需要首先安装并启用`epel-release`存储库。

```c
# yum install epel-release
OR
# yum install epel-release -y
# yum install dnf
```
##  4. 示例
### 4.1 查看DNF版本

```c
# dnf --version
```
### 4.2  列出启用的 DNF 存储库

```c
# dnf repolist
```
### 4.3 列出所有启用和禁用的 DNF 存储库

```c
# dnf repolist all
```
### 4.4  使用 DNF 列出所有可用和已安装的软件包

```c
# dnf list
```
### 4.5  使用 DNF 列出所有已安装的软件包

```c
# dnf list installed
```
### 4.6  使用 DNF 列出特定的包

```c
dnf list installed | grep bash
```
或者
显示可用的各个版本
```bash
$ dnf --showduplicates list podman
Last metadata expiration check: 0:06:19 ago on Fri 18 Nov 2022 10:24:42 AM CST.
Available Packages
podman.x86_64                                        3.3.1-9.module_el8.5.0+988+b1f0b741                                        appstream

$ dnf --showduplicates list buildah
Last metadata expiration check: 0:07:03 ago on Fri 18 Nov 2022 10:24:42 AM CST.
Available Packages
buildah.x86_64                                       1.22.3-2.module_el8.5.0+911+f19012f9                                       appstream
```

### 4.7  使用 DNF 列出所有可用的包

```c
# dnf list available
```
### 4.8  使用 DNF 搜索包
如果万一，您不知道要安装的软件包，在这种情况下，您可以使用带有 dnf 命令的“搜索”选项来搜索与单词或字符串匹配的软件包（比如 nano）。

```c
# dnf search nano
```
### 4.9  看看提供了什么文件/子包？
dnf 选项“提供”查找提供特定文件/子包的包的名称。例如，如果您想查找系统上提供“ /bin/bash ”的内容？

```c
# dnf provides /bin/bash
```
### 4.10  使用 DNF 获取包的详细信息
假设你想在系统上安装一个包之前知道它的信息，你可以使用“ info ”开关来获取一个包的详细信息（比如 nano），如下所示。

```c
# dnf info nano
```
### 4.11   使用 DNF 安装包

```c
# dnf install nano
```
### 4.12  使用 DNF 更新包
你可以只更新一个特定的包（比如systemd）并且不改变系统上的所有内容。

```c
# dnf update systemd
```
### 4.13  使用 DNF 检查系统更新

```bash
# dnf check-update
```
### 4.14  使用 DNF 更新所有系统包
您可以使用以下命令更新整个系统，包括所有已安装的软件包。

```c
# dnf update
OR
# dnf upgrade
```
### 4.15  使用 DNF升级特定的包
	
```c
dnf upgrade python3-perf
```

### 4.16  使用 DNF 删除/擦除包
要删除或擦除任何不需要的包（例如nano），您可以使用带有 dnf 命令的“ remove ”或“ erase ”开关来删除它。

```c
# dnf remove nano
OR
# dnf erase nano
```
### 4.17  使用 DNF 删除孤立包
那些为了满足依赖而安装的包如果不被其他应用程序使用，可能会毫无用处。要删除这些孤立包，请执行以下命令。

```c
# dnf autoremove
```
### 4.18  使用 DNF 删除缓存包
很多时候，我们遇到过时的标头和未完成的事务，这会在执行 dnf 时导致错误。我们可以简单地通过执行来清除所有缓存的包和包含远程包信息的标头。

```c
# dnf clean all
```
### 4.19  获取特定 DNF 命令的帮助

```c
# dnf help clean
```
### 4.20  列出所有 DNF 命令和选项

```c
# dnf help
```
### 4.21  查看DNF的历史
您可以调用 `dnf history` 查看已执行的 dnf 命令列表。通过这种方式，您可以通过时间戳了解安装/删除的内容。

```c
# dnf history
```
### 4.22  列出所有组包
命令“ dnf grouplist ”将打印所有可用或已安装的软件包，如果没有提及，它将列出所有已知的组。

```c
# dnf grouplist
```
### 4.23  列出组包中有哪些包
	
```c
dnf group info "Development Tools"
```

### 4.24  使用 DNF 安装组包

```c
# dnf groupinstall 'Educational Software'
```
### 4.25  更新组包

```c
# dnf groupupdate 'Educational Software'
```
### 4.26  删除组包

```c
# dnf groupremove 'Educational Software'

```
### 4.27  从特定存储库安装包
DNF 使得从 repo ( epel ) 安装任何特定的包 (比如phpmyadmin ) 成为可能，就像，

```c
# dnf --enablerepo=epel install phpmyadmin
```
### 4.28  将已安装的包同步到稳定版本
命令“ dnf distro-sync ”将提供必要的选项，以将所有已安装的软件包同步到任何启用的存储库中可用的最新稳定版本。如果未选择任何包，则同步所有已安装的包。

```c
# dnf distro-sync
```
### 4.29  重新安装一个包

```c
# dnf reinstall nano
```
### 4.30  降级软件包
如果可能，选项“downgrade”会将命名包（比如 acpid）降级到较低版本。
```c
# dnf downgrade acpid
```


## 5.  DNF 源
DNF 的主要配置文件是 /etc/dnf/dnf.conf，该文件包含两部分：

 - “main”部分保存着DNF的全局设置。
 - “repository”部分保存着软件源的设置，可以有一个或多个“repository”。

另外，在`/etc/yum.repos.d` 目录中保存着一个或多个repo源相关文件，它们也可以定义不同的“repository”
### 5.1 配置main部分
`/etc/dnf/dnf.conf` 文件包含的“main”部分，配置示例如下：
[main]

```c
gpgcheck=1
installonly_limit=3
clean_requirements_on_remove=True
best=True
```
常用选项说明：

| 参数                           | 说明                                                                                                                  |
|------------------------------|---------------------------------------------------------------------------------------------------------------------|
| cachedir                     | 缓存目录，该目录用于存储RPM包和数据库文件。                                                                                             |
| keepcache                    | 可选值是1和0，表示是否要缓存已安装成功的那些RPM包及头文件，默认值为0，即不缓存。                                                                         |
| debuglevel                   | 设置dnf生成的debug信息。取值范围：[0-10]，数值越大会输出越详细的debug信息。默认值为2，设置为0表示不输出debug信息。                                              |
| clean_requirements_on_remove | 删除在dnf remove期间不再使用的依赖项，如果软件包是通过DNF安装的，而不是通过显式用户请求安装的，则只能通过clean_requirements_on_remove删除软件包，即它是作为依赖项引入的。 默认值为True。 |
| best                         | 升级包时，总是尝试安装其最高版本，如果最高版本无法安装，则提示无法安装的原因并停止安装。默认值为True。                                                               |
| obsoletes                    | 可选值1和0，设置是否允许更新陈旧的RPM包。默认值为1，表示允许更新。                                                                                |
| gpgcheck                     | 可选值1和0，设置是否进行gpg校验。默认值为1，表示需要进行校验。                                                                                  |
| plugins                      | 可选值1和0，表示启用或禁用dnf插件。默认值为1，表示启用dnf插件。                                                                                |
| installonly_limit            | 设置可以同时安装“installonlypkgs”指令列出包的数量。默认值为3，不建议降低此值。                                                                    |


### 5.2  配置repository部分
`repository`部分允许您定义定制化的openEuler软件源仓库，各个仓库的名称不能相同，否则会引起冲突。配置repository部分有两种方式，一种是直接配置/etc/dnf/dnf.conf文件中的“repository”部分，另外一种是配置`/etc/yum.repos.d`目录下的.repo文件。

直接配置`/etc/dnf/dnf.conf`文件中的“repository”部分

下面是[repository]部分的一个最小配置示例：

```c
[repository]
name=repository_name
baseurl=repository_url
```

> openEuler提供在线的镜像源，地址：`https://repo.openeuler.org/`。以 openEuler 20.09的aarch64版本为例，baseurl可配置为`https://repo.openeuler.org/openEuler-20.09/OS/aarch64/`。

表 2 repository参数说明
|参数|	说明|
|--|--|
|name=repository_name|	软件仓库（repository ）描述的字符串。|
|baseurl=repository_url|	软件仓库（repository ）的地址。例如：使用http协议的网络位置： http://path/to/repo;使用ftp协议的网络位置： ftp://path/to/repo;本地位置： file:///path/to/local/repo|

配置`/etc/yum.repos.d`目录下的.repo文件

openEuler提供了多种repo源供用户在线使用，各repo源含义可参考系统安装。使用root权限添加openEuler repo源，示例如下：

```c
# vi /etc/yum.repos.d/openEuler.repo
[OS]
name=openEuler-$releasever - OS
baseurl=https://repo.openeuler.org/openEuler-20.09/OS/$basearch/
enabled=1
gpgcheck=1
gpgkey=https://repo.openeuler.org/openEuler-20.09/OS/$basearch/RPM-GPG-KEY-openEuler
```

> 说明： 
 `enabled`为是否启用该软件源仓库，可选值为1和0。默认值为1，表示启用该软件源仓库。 
 `gpgkey`为验证签名用的公钥

### 5.3 显示当前配置
要显示当前的配置信息：

```bash
dnf config-manager --dump
```

要显示相应软件源的配置，首先查询repo id：

```c
dnf repolist
```

然后执行如下命令，显示对应id的软件源配置，其中 repository 为查询得到的repo id：

```c
dnf config-manager --dump repository
```

您也可以使用一个全局正则表达式，来显示所有匹配部分的配置：

```c
dnf config-manager --dump glob_expression
```

### 5.4 创建本地软件源仓库
要建立一个本地软件源仓库，请按照下列步骤操作
1.安装createrepo软件包。在root权限下执行如下命令

```c
dnf install createrepo
```

2.将需要的软件包复制到一个目录下，如`/mnt/local_repo/` 。
3.创建软件源，执行以下命令：

```c
createrepo --database /mnt/local_repo
```
### 5.5 添加软件源
要定义一个新的软件源仓库，您可以在 `/etc/dnf/dnf.conf` 文件中添加“`repository`”部分，或者在`/etc/yum.repos.d/`目录下添加“`.repo`”文件进行说明。建议您通过添加“.repo”的方式，每个软件源都有自己对应的“.repo”文件，以下介绍该方式的操作方法。

要在您的系统中添加一个这样的源，请在root权限下执行如下命令，执行完成之后会在/`etc/yum.repos.d/`目录下生成对应的repo文件。其中 `repository_url` 为repo源地址

```c
dnf config-manager --add-repo repository_url
```
### 5.6 启用软件源
要启用软件源，请在root权限下执行如下命令，其中 `repository` 为新增`.repo`文件中的repo id（可通过`dnf repolist`查询）：

```c
dnf config-manager --set-enable repository
```
您也可以使用一个全局正则表达式，来启用所有匹配的软件源。其中 `glob_expression` 为对应的正则表达式，用于同时匹配多个`repo id`：

```c
dnf config-manager --set-enable glob_expression
```
### 5.7 禁用软件源
要禁用软件源，请在root权限下执行如下命令：

```c
dnf config-manager --set-disable repository
```
同样的，您也可以使用一个全局正则表达式来禁用所有匹配的软件源：

```cpp
dnf config-manager --set-disable glob_expression
```

##  6. 结论
DNF是最先进的包管理器 YUM 的上层状态。我相信，它往往会自动进行很多处理，这不会受到许多有经验的 Linux 系统管理员的称赞。例如：

 - `--skip-broken`不被 DNF 认可，也别无选择。
 - 没有什么比“ resolvedep ”命令更能运行，但是您可以运行 dnf 提供的命令。
 - 没有“ deplist ”命令来查找包依赖项。
 - 您排除一个 repo，意味着排除适用于所有操作，不像 yum 仅在安装和更新等时排除这些 repo。


更多阅读：

 - [linux yum](https://blog.csdn.net/xixihahalelehehe/article/details/105625395?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165052428516780265479599%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=165052428516780265479599&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-2-105625395.nonecase&utm_term=yum&spm=1018.2226.3001.4450)
 - [linux apt](https://blog.csdn.net/xixihahalelehehe/article/details/105641494?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165052432416782094824678%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=165052432416782094824678&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-2-105641494.nonecase&utm_term=apt&spm=1018.2226.3001.4450)
 - [linux dnf](https://blog.csdn.net/xixihahalelehehe/article/details/123168620?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165052435716782089317151%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=165052435716782089317151&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-123168620.nonecase&utm_term=dnf&spm=1018.2226.3001.4450)

 - [linux dnf docs](https://dnf.readthedocs.io/en/latest/)
 - [linux snap](https://blog.csdn.net/xixihahalelehehe/article/details/124266900)



