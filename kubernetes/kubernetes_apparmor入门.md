#  apparmor 入门
tags: apparmor,安全



[
![忠犬八公的故事](https://img-blog.csdnimg.cn/f2454476adb646508beae3729ee60ccc.png)](https://www.rottentomatoes.com/m/hachi_a_dogs_tale)

## 1. 介绍
AppArmor 是基于名称的访问控制的 Linux 安全模块实现。AppArmor 将单个程序限制为一组列出的文件和 posix 1003.1e 草案功能。

有关 AppArmor 的更多信息可以在 AppArmor 项目的[wiki](https://wiki.apparmor.net/)上找到。

## 2. 安装
自 Ubuntu `8.04` LTS 起默认安装和加载 `AppArmor`。一些软件包将安装它们自己的强制配置文件。可以在 Universe 存储库的包`apparmor-profiles`中找到其他配置文件。针对已安装的 apparmor 配置文件提交错误时，请参阅：[https ://wiki.ubuntu.com/DebuggingApparmor](https://wiki.ubuntu.com/DebuggingApparmor)

安装额外的 AppArmor 配置文件

 - 启用 Universe 存储库。
 - 安装[apparmor-profiles](https://blog.csdn.net/xixihahalelehehe/article/details/126226966)。单击链接进行安装，或查看[安装软件](https://help.ubuntu.com/community/InstallingSoftware)了解更多安装选项。

## 3. 用法
以下所有命令都应从终端执行。

### 3.1 列出 apparmor 的当前状态

```bash
sudo aa-status
```

### 3.2 将个人资料置于投诉模式

```bash
sudo aa-complain /path/to/bin
```

例子：


```bash
sudo aa-complain /bin/ping
```

### 3.3 将配置文件置于强制模式

```bash
sudo aa-enforce /path/to/bin
```

例子：


```bash
sudo aa-enforce /bin/ping
```

### 3.4 禁用 AppArmor 框架
系统通常不需要完全禁用 AppArmor。强烈建议用户启用 AppArmor 并将有问题的配置文件置于抱怨模式（见上文），然后使用https://wiki.ubuntu.com/DebuggingApparmor中的程序提交错误。如果必须禁用 AppArmor（例如使用 SELinux），用户可以：


```bash
sudo systemctl stop apparmor
sudo systemctl disable apparmor
```

在 Ubuntu 16.04 LTS 之前的 Ubuntu 系统上：


```bash
sudo invoke-rc.d apparmor stop
sudo update-rc.d -f apparmor remove
```

要在内核中禁用 AppArmor，请执行以下任一操作：

 - 调整你的内核引导命令行（见/etc/default/grub)
 - 'apparmor=0'
 - 'security=XXX' 其中 XXX 可以是 "" 以禁用 AppArmor 或替代 LSM 名称，例如。'安全="selinux"'

去除服装使用您的包管理器打包。如果您认为您可能想在以后重新启用 AppArmor，请不要“清除”apparmor

## 3.5 启用 AppArmor 框架
AppArmor 默认启用。如果您使用上述过程，要禁用它，您可以通过以下方式重新启用它：

 - 确保 AppArmor 未`在/etc/default/grub`如果使用 Ubuntu 内核，或者使用非 Ubuntu
   内核，那么`/etc/default/grub` 有`apparmor=1` `security=apparmor`
 - 确保服装安装包
 - 启用 systemd 单元：`sudo systemctl enable apparmor && sudo systemctl start
   apparmor`
 - 对于 Ubuntu 16.04 LTS 之前的系统：

```bash
sudo invoke-rc.d apparmor start
sudo update-rc.d apparmor start 37 S .
```

### 3.6 重新加载所有配置文件

```bash
sudo service apparmor reload
```

### 3.7 重新加载一个配置文件

```bash
sudo apparmor_parser -r /etc/apparmor.d/profile.name
```

例子：

```bash
sudo apparmor_parser -r /etc/apparmor.d/bin.ping
```

### 3.8 禁用一个配置文件

```bash
sudo ln -s /etc/apparmor.d/profile.name /etc/apparmor.d/disable/
sudo apparmor_parser -R /etc/apparmor.d/profile.name
```

例子：

```bash
sudo ln -s /etc/apparmor.d/bin.ping /etc/apparmor.d/disable/
sudo apparmor_parser -R /etc/apparmor.d/bin.ping
```

### 3.9 启用一个配置文件
默认情况下，配置文件被启用（即加载到内核并应用于进程）。

```bash
sudo rm /etc/apparmor.d/disable/profile.name
sudo apparmor_parser -r /etc/apparmor.d/profile.name
```

例子：

```bash
sudo rm /etc/apparmor.d/disable/bin.ping
sudo apparmor_parser -r /etc/apparmor.d/bin.ping
```

这 `aa-enforce`命令也可用于启用配置文件：

```bash
sudo aa-enforce /etc/apparmor.d/bin.ping
```

配置文件定制
配置文件可以在`/etc/apparmor.d`中找到。这些是简单的文本文件，可以使用文本编辑器或使用`aa-logprof`进行编辑。

可以在`/etc/apparmor.d/tunables/`中进行一些自定义。更新配置文件时，在适当的时候使用这些配置文件很重要。例如，而不是使用如下规则：

```bash
 /home/*/ r,
```

利用：

```bash
  @{HOME}/ r,
```

更新配置文件后，请务必重新加载（见上文）。

## 4. 常问问题
### 4.1 aa-status 报告不受限制但已定义配置文件的进程
重新启动列出的进程。重新启动也将解决问题。

AppArmor 只能跟踪和保护内核模块加载后启动的进程。安装 apparmor 软件包后，apparmor 将启动。但是正在运行的进程不会受到 AppArmor 的保护。重新启动进程或重新启动将解决此问题。

### 4.2 如何为 Firefox 启用 AppArmor？
从 Ubuntu 9.10 (Karmic) 开始，AppArmor 附带了一个默认禁用的 Firefox 配置文件。

您可以使用以下命令启用它：

```bash
sudo aa-enforce /etc/apparmor.d/usr.bin.firefox
```

### 4.3 如何使 AppArmor 与非标准 HOME 目录一起工作？
可以在`/etc/apparmor.d/tunables/home`中调整主目录的位置。

使用 `Ubuntu 10.04 LTS` 及更高版本，您可以使用`sudo dpkg-reconfigure apparmor`设置主目录位置。


## 5. 创建新配置文件
设计测试计划
尝试考虑应该如何执行应用程序。测试计划应该分成小的测试用例。每个测试用例都应该有一个简短的描述并列出要遵循的步骤。

一些标准测试用例是：

 - 启动程序
 - 停止程序
 - 重新加载程序
 - 测试初始化​​脚本支持的所有命令

对于图形程序，您的测试用例还应该包括您通常所做的任何事情。下载和打开文件、保存文件、上传文件、使用插件、保存配置更改以及启动其他程序都是可能的。

## 6. 生成新的配置文件
使用`aa-genprof`生成新的配置文件。

从终端，使用命令aa-genprof：

```bash
sudo aa-genprof executable
```

例子：

```bash
sudo aa-genprof slapd
```

手册页有更多信息：`man aa-genprof`。

参考:

 - [ubuntu AppArmor](https://help.ubuntu.com/community/AppArmor)

