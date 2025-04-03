
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/c5b854aaa8474fb2a2780808a3c497ba.png)



## 1. 预备
为了在您的系统上安装 VirtualBox，您需要确保在处理器上启用了虚拟化。这通常是在 BIOS/UEFI 配置中进行的。要检查它是否已启用，请执行以下命令：

```bash
$ lscpu | grep Virtualization
Virtualization:                  VT-x
Virtualization type:             full
```
此外，请验证您运行的是 64 位系统。

```bash
$ lscpu
Architecture:            x86_64
  CPU op-mode(s):        32-bit, 64-bit
  Address sizes:         39 bits physical, 48 bits virtual
  Byte Order:            Little Endian
CPU(s):                  1
  On-line CPU(s) list:   0
Vendor ID:               GenuineIntel
  Model name:            Intel(R) Xeon(R) CPU E3-1275 v5 @ 3.60GHz
    CPU family:          6
    Model:               94
```

## 2. 添加 Virtualbox 仓库
Virtualbox 不驻留在默认的 Rocky /AlmaLinux 9 存储库中。为了能够安装它，您需要添加官方存储库。
安装所需的构建工具：

```bash
$ sudo dnf install wget curl gcc make perl bzip2 dkms kernel-devel kernel-headers  -y
```

安装后，比较 kernel-devel 版本和内核版本：

```bash
$ rpm -q kernel-devel	
kernel-devel-5.14.0-70.17.1.el9_0.x86_64

$ uname -r
5.14.0-70.13.1.el9_0.x86_64
```
由于两者不匹配，我们需要更新 Linux 内核。

```bash
$ sudo dnf update -y
```
重新启动系统以应用内核更新：

```bash
$ sudo reboot now
```
系统引导后，验证 Kernel 和 kernel-devel 是否为同一版本。然后将 VirtualBox 仓库添加到系统中：

```bash
$ sudo dnf config-manager --add-repo=https://download.virtualbox.org/virtualbox/rpm/el/virtualbox.repo
```
## 3. Rocky /AlmaLinux 9 上安装 VirtualBox 7.1
添加存储库后，确定可用的 VirtualBox 版本
```bash
$ dnf search virtualbox
Last metadata expiration check: 0:00:06 ago on Fri 14 Feb 2025 09:47:07 AM CST.
================================== Name & Summary Matched: virtualbox ==================================
VirtualBox-6.1.x86_64 : Oracle VM VirtualBox
VirtualBox-7.0.x86_64 : Oracle VM VirtualBox
VirtualBox-7.1.x86_64 : Oracle VirtualBox
```
如您所见，我们提供了 VirtualBox 7.1，继续并使用命令安装它：

```bash
$ sudo dnf install VirtualBox-7.1 -y
```

## 4. 安装 Virtualbox 扩展包
等待安装完成，然后继续并安装 Virtualbox 扩展包。这提供了 Virtualbox RDP、磁盘加密、Intel 卡的 NVMe 和 PXE 启动、USB 2.0、USB 3.0 设备到 VirtualBox 等功能。
要下载 Virtualbox 扩展包，请访问官方 Virtualbox 下载页面。在该页面上，下载 “all supported platforms” 压缩包

```bash
VER=$(curl -s https://download.virtualbox.org/virtualbox/LATEST.TXT)
wget https://download.virtualbox.org/virtualbox/$VER/Oracle_VirtualBox_Extension_Pack-$VER.vbox-extpack
wget https://download.virtualbox.org/virtualbox/7.1.6/Oracle_VirtualBox_Extension_Pack-7.1.6.vbox-extpack
```
现在使用以下命令构建内核模块：

```bash
$ sudo /sbin/vboxconfig
vboxdrv.sh: Stopping VirtualBox services.
vboxdrv.sh: Starting VirtualBox services.
vboxdrv.sh: Building VirtualBox kernel modules.
```
下载后，导航到文件的位置并使用命令进行安装：

```bash
$ sudo VBoxManage extpack install Oracle_VirtualBox_Extension_Pack-7.1.6.vbox-extpack
```
Proceed as shown;  如图所示;

```bash
Do you agree to these license terms and conditions (y/n)? y

License accepted. For batch installation add
--accept-license=eb31505e56e9b4d0fbca139104da41ac6f6b98f8e78968bdf01b1f3da3c4f9ae
to the VBoxManage command line.

0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
Successfully installed "Oracle VirtualBox Extension Pack".
```
## 5. 如何使用 VirtualBox 7.1
安装后，Virtualbox 7.1 就可以使用了。从 App Menu 启动它，如下所示：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b81b9ce9bfc446e8aef3aa95204cb8ae.png)
要创建虚拟机，请单击 New 并设置 VM 的名称，如图所示。
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/710186e630e64373a43aac3bf63036f2.png)
指定存储虚拟机位置。
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/56f6a05affd344dc870c953a973aebde.png)

参考：
- [https://computingforgeeks.com/install-virtualbox-on-rocky-almalinux/](https://computingforgeeks.com/install-virtualbox-on-rocky-almalinux/)
