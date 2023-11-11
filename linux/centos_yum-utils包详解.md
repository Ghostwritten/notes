✈<font color=	#FF4500 size=5 style="font-family:Courier New">前些天发现了一个巨牛的人工智能学习网站，通俗易懂，风趣幽默，忍不住分享一下给大家：[床长人工智能教程](https://www.captainai.net/weike)</font>

---

## yum-utils模块
yum-utils模块：

```bash
gdb
find-repos-of-install

package-cleanup

repo-graph

repoclosure

repomanage

repoquery

yum-debug-dump  zless  yum-debug-restore

yumdownloader

reposync 
```
## 2. 安装
```bash
$ yum update && yum install yum-utils
$ man yum-utils
```
## 3.使用
1.调试软件包
debuginfo安装<软件包名称>要求安装调试的debuginfo软包（和它们的依赖） 
<package name>在崩溃的情况下，或者在开发使用某个软件包的应用程序。

为了调试一个包（或任何其他可执行程序），我们还需要安装GDB（GNU调试器） ，并用它在调试模式下启动程序。

例如：

```bash
$ gdb $(which postfix）
```

上面的命令将启动一个gdb的外壳 ，我们可以输入操作来执行。
 例如， 运行 （如下面的图像中）将启动该程序，而BT（未示出）将显示栈跟踪（也称为回溯 ）的程序，
这将提供的函数调用导致一个列表程序执行的某一点（使用这些信息，开发人员和系统管理员都可以知道在崩溃的情况下出了什么问题）
在Linux中调试软件包

2.查找已安装软件包的存储库

```bash
$ find-repos-of-install httpd postfix dovecot
```


3.删除重复或孤立的包
包清理管理（从比当前配置的存储库之外的来源安装的程序）包清理，重复，孤儿软件包和其他依赖不一致，包括删除，如下例所示老的内核：
它只会影响不再需要的旧内核包（当前运行的版本之前的版本）。

```bash
$ package-cleanup --orphans
$ package-cleanup --oldkernels
```

4.找出包依赖列表
回购图返回在所有可从配置的仓库中的包点格式全包的依赖列表。 另外， repo-graph可如果与使用存储库返回相同的信息--repoid=<repo>选项。
例如，让我们查看更新存储库中每个软件包的依赖关系：

```bash
$ repo-graph --repoid=updates | less
```

iputils软件包依赖于systemd和OpenSSL-库 。

```bash
$ repo-graph --repoid=updates > updates-dependencies.txt
```
5.检查未解决的依赖关系的列表
repoclosure读取配置的存储库的元数据，检查列入其中，并为每个包未解决的依赖性的显示列表包的依赖关系：

```bash
$ repoclosure
```

6.如何检查目录中的最新或最旧的软件包
repomanage查询用rpm包的目录，并在目录中返回最新或最早的软件包列表。 这个工具可以派上用场，如果您有您储存不同的程序的几个.rpm的包目录。

当不带参数执行，repomanage返回最新的软件包。 如果与运行--old标志，它将返回最早的包：

```bash
$ ls -l
$ cd rpms
$ ls -l rpms
$ repomanage rpms
```

7.查询Yum存储库以获取有关软件包的信息
repoquery查询Yum库，并得到有关包的其他信息，无论是安装或没有（相关性，包含的文件包中，更多）。

例如， HTOP（Linux的过程监控）当前未安装此系统上，你可以看到如下：

```bash
$ which htop
$ rpm -qa | grep htop
```
现在假设我们要列出HTOP的相关性，与包含在默认安装的文件一起。 为此，请分别执行以下两个命令：

```bash
$ repoquery --requires htop
```

列出RPM软件包的依赖关系

```bash
$ repoquery --list htop
```

8.将所有已安装的RPM软件包转储到Zip文件中
Yum调试转储让你甩了你已经安装的所有软件包的完整列表，任何储存库，重要的配置和系统信息到一个压缩文件中所有可用的软件包。

如果你想调试已经发生的问题，这可以派上用场。 对于我们的方便， Yum调试转储名称的文件作为yum_debug_dump- <主机名> - <时间> .txt.gz，这使我们能够跟踪随时间的变化。

```bash
$ yum-debug-dump
```

与任何压缩的文本文件，我们可以使用zless命令查看其内容：

```bash
$ zless yum_debug_dump-mail.linuxnewz.com-2015-11-27_08:34:01.txt.gz
```
查看压缩文本文件的内容
如果您需要恢复Yum调试转储提供的配置信息，您可以用yum调试，恢复这样做：

```bash
$ yum-debug-restore yum_debug_dump-mail.linuxnewz.com-2015-11-27_08:34:01.txt.gz
```
9.从Yum存储库下载源RPM
从库yumdownloader下载源RPM文件，包括他们的依赖。 用于创建要从具有受限Internet访问的其他计算机访问的网络存储库。

Yumdownloader允许你不仅下载二进制的RPM也是那些来源（如果与使用--source选项）。

例如，让我们创建一个名为HTOP-文件 ，我们将存储安装使用rpm程序所需要的RPM（S）。 要做到这一点，我们需要使用--resolve与yumdownloader一起开关：

```bash
$ mkdir htop-files
$ cd htop-files
$ yumdownloader --resolve htop
$ rpm -Uvh 
```
10.将远程Yum存储库同步到本地目录
reposync密切相关yumdownloader（事实上，他们支持几乎相同的选项），但提供了一个相当大的优势。 而不是下载二进制或源RPM文件，它将远程存储库同步到本地目录。
让我们来同步知名EPEL软件库到一个名为EPEL本地当前工作目录中的子目录：

```bash
$ man reposync
$ mkdir epel-local
$ reposync --repoid=epel --download_path=epel-local
$ reposync -r HDP-2.2（仓库名字）
```

-p 指定目录
-d 来删除本地老旧

一旦同步完成，让我们来检查我们的新创建的使用EPEL软件库的镜子使用的磁盘空间量命令 ：
$  du -sch epel-local/*

11.修复未完成或中止的Yum交易
Yum完成事务是赶上一个系统上未完成或中止Yum交易，并尝试完成他们的yum-utils的计划的一部分。
例如，当我们更新通过yum包管理器的Linux服务器有时会抛出其内容如下的警告信息：
还有未完成的交易。 您可以考虑首先运行yum-complete-transaction来完成它们。
要解决这样的警告消息，并解决这些问题， Yum完成事务命令进入画面，完成未完成的事务，它发现在交易的所有*并可以发现交易完成*文件的不完整或中止Yum交易/无功/ lib中/Yum目录。
运行Yum完成事务命令完成，未完成交易的yum：

```bash
$ yum-complete-transaction --cleanup-only
```

现在yum命令将运行没有不完整的事务警告。
$ yum update

























