
#  Linux Command snap 软件包管理
tags: 软件包管理



---

## 1.  简介

在过去的几年里，Linux 社区在 [Linux 系统上的包管理](https://www.tecmint.com/linux-package-management/)领域取得了一些显着的进步，特别是在通用或跨分发软件打包和分发方面。其中一项进步是由流行的Ubuntu Linux制造商Canonical开发的Snap包格式。

Snaps是跨发行版、无依赖关系且易于安装的应用程序，这些应用程序与所有依赖项打包在一起，可以在所有主要的 Linux 发行版上运行。从单个构建开始，一个 snap（应用程序）将在桌面、云和 IoT 上的所有受支持的 Linux 发行版上运行。支持的发行版包括 Ubuntu、Debian、Fedora、Arch Linux、Manjaro 和 CentOS/RHEL。

快照是安全的——它们被限制和沙盒化，因此它们不会危及整个系统。它们在不同的限制级别下运行（这是与基本系统和彼此隔离的程度）。更值得注意的是，每个快照都有一个由快照创建者根据快照的要求精心挑选的界面，以提供对其限制之外的特定系统资源的访问，例如网络访问、桌面访问等。

snap生态系统中的另一个重要概念是Channels。通道确定安装和跟踪哪个版本的快照以进行更新，它由跟踪、风险级别和分支组成并被细分。

snap包管理系统的主要组件有：

 - snapd – 在 Linux 系统上管理和维护快照的后台服务。
 - snap – 应用程序包格式和命令行界面工具，用于安装和删除 snap 以及在 snap 生态系统中执行许多其他操作。
 - snapcraft – 用于构建快照的框架和强大的命令行工具。
 - snap store——开发者可以共享他们的 snap，Linux 用户可以搜索和安装它们的地方。

此外，快照也会自动更新。您可以配置更新发生的时间和方式。默认情况下，snapd守护程序每天最多检查四次更新：每次更新检查称为刷新。您也可以手动启动刷新。

## 2 安装 Snapd
如上所述，snapd守护进程是后台服务，它通过实施限制策略和控制允许 snap 访问特定系统资源的接口来管理和维护Linux 系统上的snap环境。它还提供snap命令并用于许多其他用途。

要在您的系统上安装snapd软件包，请为您的 Linux 发行版运行适当的命令。

```bash
------------ [On Debian and Ubuntu] ------------ 
$ sudo apt update 
$ sudo apt install snapd

------------ [On Fedora Linux] ------------
# dnf install snapd			

------------ [On CentOS and RHEL] ------------
# yum install epel-release 
# yum install snapd		

------------ [On openSUSE - replace openSUSE_Leap_15.0 with the version] ------------
$ sudo zypper addrepo --refresh https://download.opensuse.org/repositories/system:/snappy/openSUSE_Leap_15.0 snappy
$ sudo zypper --gpg-auto-import-keys refresh
$ sudo zypper dup --from snappy
$ sudo zypper install snapd

------------ [On Manjaro Linux] ------------
# pacman -S snapd

------------ [On Arch Linux] ------------
# git clone https://aur.archlinux.org/snapd.git
# cd snapd
# makepkg -si
```
在您的系统上安装snapd后，使用[systemctl 命令启用管理](https://www.tecmint.com/manage-services-using-systemd-and-systemctl-in-linux/)主snap通信套接字的systemd单元，如下所示。

在Ubuntu及其衍生版本上，这应该由包安装程序自动触发。

```bash
$ sudo systemctl enable --now snapd.socket
```
请注意，如果`snapd.socket`未运行，则无法运行snap命令。运行以下命令以检查它是否处于活动状态并启用以在系统启动时自动启动。

```bash
$ sudo systemctl is-active snapd.socket
$ sudo systemctl status snapd.socket
$ sudo systemctl is-enabled snapd.socket
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/f45edf47f1f644448e214946078685e1.png)
接下来，通过在`/var/lib/snapd/snap`和`/snap`之间创建符号链接来启用经典快照支持，如下所示。

```bash
$ sudo ln -s /var/lib/snapd/snap /snap
```
要检查系统上安装的snapd和 snap 命令行工具的版本，请运行以下命令。

```bash
$ snap version 
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/c3aada650f4c4c42825436c43d75ed14.png)
snap命令允许您安装、配置、刷新和删除快照，并与更大的快照生态系统进行交互。

在安装snap之前，您可以检查它是否存在于 snap store 中。例如，如果应用程序属于“`chat servers`”或“`media players`”类别，您可以运行这些命令来搜索它，这将在稳定频道中向商店查询可用包。

```bash
$ snap find "chat servers"
$ snap find "media players"
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/56ecc2a161e9492592a941d7fccf7a08.png)
显示有关snap的详细信息

```bash
$ snap info rocketchat-server
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/1d4dc98699cf4781b3183424db269c6f.png)
安装snap包，例如，`rocketchat-server`

```bash
$ sudo snap install rocketchat-server
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a3fa53c37a224c49a3c1c98939aedca3.png)
您可以选择从不同的渠道安装：`edge`、`beta`或`Candidate`，出于某种原因，分别使用`--edge`、`--beta`或`--candidate`选项。或使用该`--channel`

```bash
$ sudo snap install --edge rocketchat-server        
$ sudo snap install --beta rocketchat-server
$ sudo snap install --candidate rocketchat-server
```

##  3. 管理 snaps
###  3.1 查看已安装
```bash
$ snap list
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/ba8fa8dd0d004928b19b65286aea9062.png)
要列出正在使用的快照的当前版本，请指定其名称。您还可以通过添加`--all`选项列出其所有可用的修订。

```bash
$ snap list mailspring
OR
$ snap list --all mailspring
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/369b0304cf9e4fd2bd9d83bfde8a485c.png)
###  3.2 更新和恢复
refresh命令检查快照跟踪的通道，如果可用，它会下载并安装更新版本的快照。

```bash
$ sudo snap refresh mailspring
OR
$ sudo snap refresh		#update all snaps on the local system
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/62b95f3ced0945c389f3d77484840f0b.png)
将应用程序更新到新版本后，您可以使用revert命令恢复到以前使用的版本。请注意，与软件关联的数据也将被还原。

```bash
$ sudo snap revert mailspring
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/f8e8c7327799400a8445d2929e3a06d0.png)
现在，当您检查`mailspring`的所有修订版时，最新修订版被禁用，以前使用的修订版现在处于活动状态。

```bash
$ snap list --all mailspring
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/55caac914c22430a84081d557d94d334.png)
###  3.3 禁用/启用和删除

```bash
#禁用
$ sudo snap disable mailspring

#开启
$ sudo snap enable mailspring

#删除
$ sudo snap remove mailspring
$ sudo snap remove  --revision=482 mailspring
```

###  3.4 从 Snap 运行应用程序
snap可以提供您从图形用户界面或使用命令运行的单个应用程序（或一组应用程序）。默认情况下，所有与 snap 关联的应用程序都安装在基于 Debian 的发行版的`/snap/bin/`目录下，以及基于RHEL 的发行版的`/var/lib/snapd/snap/bin/`目录下。

```bash
$ ls /snap/bin/
OR
# ls /var/lib/snapd/snap/bin/
```
例如，要从命令行运行应用程序，只需输入其绝对路径名。

```bash
$ /snap/bin/mailspring
OR
# /var/lib/snapd/snap/bin/mailspring
```
要仅输入应用程序名称而不输入其完整路径名，请确保/snap/bin/或`/var/lib/snapd/snap/bin/`在您的`PATH`环境变量中（默认情况下应添加）。

```c
# echo $PATH
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/441d5b3b886440b695548d4d88cebb6d.png)
如果`/snap/bin/`或`/var/lib/snapd/snap/bin/`目录在您的`PATH`中，您只需键入应用程序的名称/命令即可运行应用程序：

```bash
$ mailspring
```
查看snap下可用的命令，请运行“ `snap info snap-name` ”命令，然后查看以下屏幕截图中突出显示的命令部分。

```bash
$ snap info mailspring
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/bdba13594bec4445802cc43e061d286b.png)
which命令查找应用程序或命令的绝对路径名。

```bash
which mailspring
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/3702e85e34a24f6fbcc50e9226d71c1e.png)
###  3.5 创建和使用 Snap Aliases
Snap还支持为应用程序创建别名。快照的默认（或标准）别名在启用之前必须经过公共审查过程，但您可以为本地系统创建别名。

您可以使用[alias 命令](https://www.tecmint.com/create-alias-in-linux/)为快照创建别名。

```bash
snap alias mailspring mls
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/5e038d490a6846dd9cc42fdae5ce472a.png)
要列出快照的别名，例如`mailspring`，请运行以下命令。从现在开始，您可以使用别名来运行 snap。

```bash
snap aliases mailspring
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/0de514d8a9964820bb7d87d45e9cf03e.png)
要删除snap的别名，请使用unalias命令。

```bash
snap unalias mls
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/70cd2ce051304af786b014b9b195d8c2.png)

###  3.6 管理 Snap 的服务
对于某些快照，底层功能通过作为守护程序或服务运行的应用程序公开，一旦安装快照，它们会自动启动在后台连续运行。此外，这些服务还可以在系统启动时自动启动。重要的是，单个快照可能包含多个应用程序和服务，它们协同工作以提供该快照的整体功能。

```bash
snap info rocketchat-server
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/b374e5cd8bc84c25bc5e38a8819b93b5.png)
您可以使用services命令交叉检查服务的快照。命令输出显示一个服务，它是否启用在系统启动时自动启动，以及它是否处于活动状态。

```bash
snap services rocketchat-server
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/1f05b8088b6a484b8cf9086829194cc4.png)
要停止服务运行，例如Rocketchat，请使用stop命令。请注意，不建议执行此操作，因为手动停止快照服务可能会导致快照发生故障。

```bash
snap stop rocketchat-server
```
要启动服务，例如，rocketchat使用start命令。

```bash
snap start rocketchat-server
```
要使服务在系统引导时自动启动，请使用enable命令。

```bash
snap enable rocketchat-server
```
要防止服务在下次系统引导时自动启动，请使用disable命令。

```bash
snap disable rocketchat-server
```
要查看服务的日志，请使用带有-f选项的log命令，它允许您实时查看屏幕上的日志。

```bash
 snap logs rocketchat-server
OR
snap logs -f rocketchat-server
```

###  3.7 创建和管理 Snap 的快照
Snapd为一个或多个快照存储用户、系统和配置数据的副本。您可以手动触发或将其设置为自动工作。这样，您可以备份快照的状态，将其恢复到以前的状态，以及将全新的快照安装恢复到以前保存的状态。

要手动生成快照，请使用“ `snap save` ”命令。要为mailspring创建快照，请运行以下命令：

```bash
snap save mailspring
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/72b0662bb4c142ad9cea4d207b3e7582.png)
如果未指定快照名称，snapd将为所有已安装的快照生成快照（添加`--no-wait`选项以在后台运行进程以释放终端并允许您运行其他命令）。

```bash
snap save
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/1e69e806f41547e78e63509395e88cd2.png)
要查看所有快照的状态，请使用`saved`命令。您可以使用该`--id`标志来显示特定快照的状态：

```bash
snap saved
OR
snap saved --id=2
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/f64033c656814d40a2a77cedd17ab0db.png)
您可以使用`check-snapshot`命令和快照标识符（集 ID）验证快照的完整性：

```bash
snap check-snapshot 2
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/b01e92912f7241bc8d283de18d9a2426.png)
要使用特定快照中的相应数据恢复当前用户、系统和配置数据，请使用`restore`命令并指定快照集 ID：

```bash
snap restore 2
```
要从系统中删除快照，请使用`forget`命令。默认情况下会删除所有快照的数据，您可以指定快照仅删除其数据。

```bash
snap forget 2
OR
snap forget 2  mailspring 
```

---

✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>

 - [linux snap docs](https://snapcraft.io/docs)
 - [linux yum](https://blog.csdn.net/xixihahalelehehe/article/details/105625395?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165052428516780265479599%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=165052428516780265479599&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-2-105625395.nonecase&utm_term=yum&spm=1018.2226.3001.4450)
 - [linux apt](https://blog.csdn.net/xixihahalelehehe/article/details/105641494?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165052432416782094824678%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=165052432416782094824678&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-2-105641494.nonecase&utm_term=apt&spm=1018.2226.3001.4450)
 - [linux dnf](https://blog.csdn.net/xixihahalelehehe/article/details/123168620?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165052435716782089317151%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=165052435716782089317151&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-123168620.nonecase&utm_term=dnf&spm=1018.2226.3001.4450)



