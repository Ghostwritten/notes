
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/d22ed1e5413f492185628a65a5bdf366.png)








# 1. 简介
VNC技术是由Olivetti和Oracle在英国剑桥的研究实验室创建的。它有助于使用远程帧缓冲协议 （RFB） 访问/控制远程系统。VNC 使用客户端-服务器模型，其中 VNC 服务器安装在远程系统上，VNC 客户端安装在本地系统上。一旦获得权限，VNC 服务器就会将远程计算机屏幕的副本传输到客户端。提供 VNC 的软件包括;TeamViewer、UltraVNC、TightVNC、[TigerVNC](https://computingforgeeks.com/configure-tigervnc-on-centos-for-rdp/)、VNC4server、Vino 等。

本指南旨在说明如何在 Rocky Linux 8|AlmaLinux 8 上设置 RealVNC 服务器和客户端。RealVNC是一家提供VNC软件的公司。该软件包括一个VNC服务器和一个VNC查看器，可以使用它来启动虚拟网络计算（VNC）连接。

# 2. 功能

RealVNC具有许多惊人的功能。其中一些是：

- 安全 – RealVNC 从一开始就考虑到了安全性，以平衡您需要的控制与合规性所需的隐私。
- 支持文件传输、打印和聊天 – 在会话期间，您不仅限于与远程屏幕交互。
- 它提供了一个直观的遥控器 - 它允许人们使用鼠标和键盘（或触摸屏），就好像它们属于远程服务器一样。
- 支持多种语言 – RealVNC 提供法语、德语、西班牙语和巴西葡萄牙语以及英语版本，更多翻译正在开发中。
- 提供有人值守和无人值守的访问 - 无论所有者是否在场，都可以连接到远程系统。
- 跨平台支持 – 它在 Windows、Mac、Linux、Raspberry Pi、iOS 和 Android 上提供 PC 到 PC 和移动设备到 PC。


# 3. 安装

## 3.1 Rocky Linux 8 安装桌面环境|AlmaLinux 8

由于 VNC 允许访问和控制远程图形桌面，因此要求您在系统上安装桌面环境。
如果您没有安装 GUI，您可以使用以下命令安装 Gnome GUI：

```bash
sudo dnf update
sudo dnf groupinstall -y "Server with GUI"
```
安装后，将系统设置为从图形目标启动。

```bash
sudo systemctl set-default graphical.target
sudo systemctl default
```
重新启动系统并切换到已安装的桌面环境。

```bash
sudo reboot
```

##  3.2 Rocky Linux 8 安装 RealVNC 服务器|AlmaLinux 8

### 3.2.1 界面安装

通过从 [RealVNC 下载](https://www.realvnc.com/en/connect/download/vnc/)页面下载最新的可用版本，在远程系统上安装 RealVNC 服务器。从页面中，为您的系统选择适当的文件。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/875db4bf41fd41b58bf0b3c034d9daf5.png)
### 3.2.2 终端安装

您也可以使用以下命令拉取适当的文件：
```bash
##For 64-bit
wget https://downloads.realvnc.com/download/file/vnc.files/VNC-Server-6.11.0-Linux-x64.rpm

##For 32-bit
wget https://downloads.realvnc.com/download/file/vnc.files/VNC-Server-6.11.0-Linux-x64.rpm
```
下载后，使用以下命令从本地注册表安装 RealVNC：

```bash
sudo yum localinstall VNC-Server-6.11.0-Linux-x64.rpm
```
成功安装后，启动并启用 VNC 服务。

```bash
sudo systemctl enable vncserver-virtuald.service
sudo systemctl start vncserver-virtuald.service
sudo systemctl enable vncserver-x11-serviced.service
sudo systemctl start vncserver-x11-serviced.service
```
在配置 GDM 显示管理器之前，RealVNC 服务器不会启动。

```bash
sudo vim /etc/gdm/custom.conf
```
继续并取消注释以下行。

```bash
# Uncomment the line below to force the login screen to use Xorg
WaylandEnable=false
```
重新启动 GDM 显示管理器。

```bash
sudo systemctl restart gdm
```

# 4. 配置团队帐号远程


## 4.1 Rocky Linux 8 配置 RealVNC 服务器|AlmaLinux 8
我们需要在服务器上进行多项配置，然后才能启动与客户端的通信。RealVNC 服务器应自动启动，如果没有，请从应用程序菜单启动它。

RealVNC 从下面的窗口开始。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/f7dba916bd3c4dff8beeb4af504682ed.png)

您需要注册一个免费的 [RealVNC 团队帐户](https://manage.realvnc.com/en/)。此帐户将使建立服务器-客户端通信变得容易。请确保在此处提供有效的电子邮件地址。

通过单击“解决”来许可 RealVNC 服务器

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/00ea767dd30b43aa9d491fa91863c490.png)

提供创建的帐户凭据并设置订阅。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/db89670cfa194e2394e5f7cb4e2aa29b.png)
为 VNC 设置密码


![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/77c05de8f3ed49cdb655c06f26d3b771.png)

在用户访问窗口中，您可以配置对远程系统的有人值守/无人值守的访问。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/fe03300b61bd444b810bdaa6ab712b47.png)


您将获得所做设置的预览，单击“应用”以同意它们。在这里，您需要提供特权访问的密码。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/edf1f12abe27416ebcf003e369696279.png)

完成设置后，单击“完成”完成。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/51195a09978f461db5e03187e0c5dcf9.png)

您将为远程桌面连接设置 RealVNC 服务器。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/fec10ab89d21427686499541568578f3.png)

## 4.2 设置 RealVNC 客户端

为了能够控制远程系统，我们需要在本地系统上安装 RealVNC 查看器。该应用程序可以安装在 Android、Linux、Windows、macOS 等上。从 [RealVNC 下载页面](https://www.realvnc.com/en/connect/download/viewer/linux/)为您的系统下载 RealVNC 查看器应用程序。

您可以使用以下命令在 Linux 上安装 RealVNC 客户端：

```bash
##On Debian/Ubuntu
wget https://downloads.realvnc.com/download/file/viewer.files/VNC-Viewer-6.22.207-Linux-x64.deb
sudo apt install ./VNC-Viewer-6.22.207-Linux-x64.deb

##On RHEL/CentOS/Rocky Linux 8
wget https://downloads.realvnc.com/download/file/viewer.files/VNC-Viewer-6.22.207-Linux-x64.rpm
sudo yum localinstall VNC-Viewer-6.22.207-Linux-x64.rpm
```

安装后，使用以下命令启动 RealVNC Viewer：

```bash
vncviewer
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/2c9f18b1f650414db258125d1051366c.png)

登录 RealVNC 帐户。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/83209df422b94b3889071a6a46e20f8e.png)

登录后，RealVNC 团队将与连接的 RealVNC 服务器一起出现。通过单击服务器连接到服务器。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/4388737912b542b5866634828a413ddf.png)

同意身份验证。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b697af39182d46eeb5d8e6d73b2c00c2.png)
接下来，提供设置的 RealVNC 密码。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/d97580543c6b4a9eaa8777b775ed2b42.png)
您将能够获得对系统的远程访问。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/f89d20324dea45f7b498d35f087bcab7.png)

# 5. 配置密钥远程

- 不注册邮箱帐号，通过密钥注册。

参考：

- [https://computingforgeeks.com/install-virtualbox-on-rocky-almalinux/](https://computingforgeeks.com/install-virtualbox-on-rocky-almalinux/)



## 配置lience

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/baf762c7788d452b8a2da460b6136955.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/04e9513b5e614f3abfe4f514a9e039ba.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/4e91ac08f71d4854ac9b2fd798d02156.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/6cec1908fd1d43bd8b98ec8f75dcfc97.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/74a612313a724706873ec39237fb388a.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/da07dd30e0e34008a605d0a5c5b1c28a.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b545e9ce939e4e66a54e48e643547abd.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/661f02a97755436ea4454153047e32e2.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/2f146b1b26984409a88d8fb468281f4e.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/15a23fce3c3944668c13afde5112e88e.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/0d9d8031de94471b890c7c05798c8281.png)

