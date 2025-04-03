##  Linux Command mount 挂载
tags: 文件管理

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3dd7feacbdc86c4e358d24b4a0e7ae7e.png)


##  1. 简介
[Linux 文件系统](https://phoenixnap.com/kb/linux-file-system)层次结构呈树状排列，文件系统从根目录 ( /) 开始。所有其他子文件系统都从根目录分支出来。

mount命令允许用户挂载，即将额外的子文件系统附加到当前可访问文件系统上的特定挂载点。该命令将挂载指令传递给[内核](https://phoenixnap.com/glossary/what-is-a-kernel)，内核完成操作。


##  2. 语法

```bash
mount -t [type] [device] [dir]
```

该命令指示内核附加在目录中找到的文件[device]系统[dir]。该`-t [type]`选项是可选的，它描述了文件系统类型（EXT3、EXT4、BTRFS、XFS、HPFS、VFAT 等）。

如果省略了目标目录，它将挂载`/etc/fstab`文件中列出的文件系统。

挂载文件系统时，目录之前的内容、所有者和模式`[dir]`是不可见的，`[dir]`路径名是指文件系统根目录。

##  3. 退出状态

该mount命令返回指示进程完成状态的以下值之一：

0. 成功。
1. 命令调用不正确或权限不足。
2. 系统错误。
3. 内部安装错误。
4. 操作被用户中断。
5. 写入或锁定/etc/mtab文件的问题。
6. 挂载失败。
7. 至少一个挂载操作成功了，但不是全部。



## 4. 命令选项
命令选项进一步指定文件系统类型、mount安装位置和类型。下表显示了最常见的mount选项：
|选项|	描述|
|--|--|
|-a	|挂载/etc/fstab中列出的所有文件系统。|
|-F	|mount为每个设备创建一个新的化身。必须与-a选项结合使用。|
|-h	|显示带有所有命令选项的帮助文件。|
|-l	|列出所有已挂载的文件系统并为每个设备添加标签。|
|-L \[label\]	| 挂载指定的分区[label]。|
|-M	|将子树移动到另一个位置。|
|-r|以只读模式挂载文件系统。|
|-R	|在不同的位置重新挂载子树，使其内容在两个位置都可用。|
|-t [type]	|指示文件系统类型。|
|-T|	用于指定替代的/etc/fstab文件。|
|-v|	详细安装，描述每个操作。|
|-V|	显示程序版本信息。|

`-O [opts]` 与 结合使用`-a`，限制-a适用的文件系统集。指在/etc/fstab文件[opts]的选项字段中指定的选项。该命令接受以逗号分隔的列表（不带空格）中指定的多个选项。
|选项|	描述|
|--|--|
|-o async：|打开非同步模式，所有的档案读写动作都会用非同步模式执行。
|-o sync：|在同步模式下执行。
|-o atime、-o noatime：|当 atime 打开时，系统会在每次读取档案时更新档案的『上一次调用时间』。当我们使用 flash 档案系统时可能会选项把这个选项关闭以减少写入的次数。
|-o auto、-o noauto：|打开/关闭自动挂上模式。
|-o defaults: |使用预设的选项 rw, suid, dev, exec, auto, nouser, and async.
|-o dev、-o nodev-o exec、-o noexec |允许执行档被执行。
|-o suid、-o nosuid：|允许执行档在 root 权限下执行。
|-o user、-o nouser：|使用者可以执行 mount/umount 的动作。
|-o remount：|将一个已经挂下的档案系统重新用不同的方式挂上。例如原先是唯读的系统，现在用可读写的模式重新挂上。
|-o ro：|用唯读模式挂上。
|-o rw：|用可读写模式挂上。
|-o loop=：|使用 loop 模式用来将一个档案当成硬盘分割挂上系统。


## 5. mount 挂载
###  5.1 列出挂载的文件系统
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/be7805397838149021251a06bb11df8b.png)
###  5.2 列出特定文件系统
该-t选项允许用户指定运行mount命令时要显示的文件系统。例如，要仅显示 ext4 文件系统，请运行以下命令：

```bash
mount -t ext4
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/af6719b6cb7b8339f30e9f2e61f0bc0e.png)
或者加一个 `-l`参数
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b1b07c5a594a3aab1dc5c75588bf5d89.png)


###  5.3 挂载文件系统
挂载文件系统需要用户指定文件系统将附加到的目录或挂载点。例如，要将`/dev/sdb1`文件系统挂载到`/mnt/media`目录，请运行：

```bash
sudo mount /dev/sdb1 /mnt/media
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4de399d91a073c74d2c589c3724abcd8.png)
要指定其他特定于文件系统的挂载选项，请-o在设备名称之前传递标志，后跟选项。使用以下语法：

```bash
mount -o [options] [device] [dir]
```



### 5.4 使用 /etc/fstab 挂载文件系统
`/etc/fstab`文件包含描述系统设备的安装位置和它们使用的选项的行。通常，fstab用于内部设备，例如 `CD/DVD` 设备和网络共享 (`samba/nfs/sshfs`)。可移动设备通常由`gnome-volume-manager`.

仅提供一个参数（或[dir]或[device]）会导致mount读取/etc/fstab配置文件的内容以检查指定的文件系统是否在其中列出。如果列出了给定的文件系统，则mount使用缺失参数的值和/etc/fstab文件中指定的挂载选项。

/etc/fstab中定义的结构是：

```bash
<file system> <mount point> <type> <options> <dump> <pass>
```
以下屏幕截图显示了/etc/fstab文件的内容：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b6728e4f28e43ce852306f3420776abd.png)
要挂载`/etc/fstab`文件中指定的文件系统，请使用以下语法之一：

```bash
mount [options] [dir]

mount [options] [device]
```

 - 对于[dir]，指定安装点。
 - 对于[device]，指定设备标识符。


### 5.5 挂载 USB 驱动器
现代 Linux 发行版在插入后会自动挂载可移动驱动器。但是，如果自动挂载失败，请按照以下步骤手动挂载 U 盘：
1. 使用mkdir 命令创建挂载点：

```bash
mkdir /media/usb-drive
```

2. 找到 USB 设备和文件系统类型。跑：

```bash
fdisk -l
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b1bd5658eb88ca73e8ec68c8142bc86f.png)
3. 使用fdisk输出中的设备标识符，使用以下语法挂载 USB 驱动器：

```bash
sudo mount [identifier] /media/usb-drive
```

例如，如果设备列为/dev/sdb1，请运行：

```bash
sudo mount /dev/sdb1 /media/usb-drive
```
要查找设备和文件系统类型，您可以使用以下任何命令：

```bash
fdisk -l
ls -l /dev/disk/by-id/usb*
dmesg
lsblk
```

### 5.6 安装 CD-ROM
作为可移动设备，Linux 也会自动安装 `CD-ROM`。但是，如果挂载失败，请运行以下命令手动挂载 `CD-ROM`：

```bash
mount -t iso9660 -o ro /dev/cdrom /mnt
```

确保`/mnt`安装点存在以使命令正常工作。如果没有，请使用`mkdir`命令创建一个。

`iso9660`是 CD-ROM 的标准文件系统，而`-o ro`选项导致mount将其视为只读文件系统。

### 5.7 挂载 ISO 文件
挂载 ISO 文件需要将其数据映射到循环设备。`-o loop`通过传递选项，使用循环设备将 ISO 文件附加到安装点：

```bash
mkdir /media/iso-file
mount /path/to/image.iso /media/iso-file -o loop
```
`/path/to/image.iso`为您的 ISO 文件的路径

### 5.8 挂载 NFS
网络文件系统 (NFS) 是一种分布式文件系统协议，用于通过网络共享远程目录。挂载 NFS 允许您使用远程文件，就好像它们存储在本地一样。

> 安装 NFS 需要[安装 NFS 客户端软件包](https://blog.csdn.net/xixihahalelehehe/article/details/105747174)。了解[如何在 Ubuntu 上安装 NFS
> 服务器](https://blog.csdn.net/xixihahalelehehe/article/details/117217493)。

在 `Ubuntu` 和 `Debian` 上安装 NFS 客户端：

```bash
sudo apt install nfs-common
```

在 `CentOS` 和 `Fedora` 上安装 NFS 客户端：

```bash
sudo yum install nfs-utils
```

按照以下步骤在系统上挂载远程 NFS 目录：

1. mkdir1. 使用命令创建挂载点：

```bash
sudo mkdir /media/nfs
```

2. 通过运行挂载 NFS 共享：

```bash
sudo mount /media/nfs
```

3. 要在启动时自动挂载远程 NFS 共享，请使用您选择的文本编辑器编辑`/etc/fstab`文件：

```bash
sudo vi /etc/fstab
```

将以下行添加到文件并替换`remote.server:/dir`为 NFS 服务器 IP 地址或主机名以及导出的目录：

```bash
remote.server:/dir /media/nfs  nfs      defaults    0       0
```
运行以下命令挂载 NFS 共享：

```bash
sudo mount /media/nfs
```


> 延展：查看如何创建和使用 [NFS Docker 卷](https://blog.csdn.net/xixihahalelehehe/article/details/127349415)。


### 5.9 非超级用户安装
虽然只有超级用户可以挂载文件系统，但包含该选项的`/etc/fstabuser`文件中的文件系统可以由任何系统用户挂载。

使用文本编辑器编辑/etc/fstab<options>文件并在字段下指定user选项。例如：

```bash
/dev/cdrom /cd iso9660 ro,user,noauto,unhide
```

将上面的行添加到`/etc/fstab`允许任何系统用户`iso9660`从 `CD-ROM` 设备挂载文件系统。

指定`users`选项而不是user允许任何用户卸载文件系统，而不仅仅是安装它的用户。

### 5.10 移动挂载点
如果您决定将已安装的文件系统移动到另一个安装点，请使用该-M选项。语法是：

```bash
mount --move [olddir] [newdir]
```

对于`[olddir]`，指定当前安装点。对于`[newdir]`，指定要将文件系统移动到的挂载点。

将挂载的文件系统移动到另一个挂载点会导致其内容出现在`[newdir]`目录中，但不会更改文件的物理位置。

## 6. umount 卸载
如何卸载文件系统

语法

```bash
umount [dir]
或者
umount [device]
```
例如，要分离列为 的 USB 设备`/dev/sdb1`，请运行：

```bash
umount /dev/sdb1
```

在忙于打开文件或正在进行的进程时，无法分离文件系统，并且进程失败。如果您不确定文件系统在使用什么，请运行fuser 命令以找出：

```bash
fuser -m [dir]
```

对于[dir]，指定文件系统安装点。例如：

```bash
fuser -m /media/usb-drive
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0f51a4960ebcf33502e66e29e9c5a3b8.png)
输出列出了当前访问设备的进程的 PID。停止进程并卸载文件系统。

> 注意：了解如何列出 Linux 中正在运行的进程。

### 6.1 懒惰（lazy）卸载
如果您不想手动停止进程，请使用延迟卸载，它指示`unmount`命令在其活动停止后立即分离文件系统。语法是：

```bash
umount --lazy [device]
```

### 6.2  强制卸载
`( -f)--force`选项允许用户强制卸载。但是，在强制卸载文件系统时要小心，因为该过程可能会损坏其上的数据。

语法是：`umount -f [dir]`



参考：
-  [How to Mount and Unmount File Systems in Linux](https://linuxize.com/post/how-to-mount-and-unmount-file-systems-in-linux/)
- [Linux mount Command with Examples](https://phoenixnap.com/kb/linux-mount-command)
- [mount(8) — Linux manual page](https://man7.org/linux/man-pages/man8/mount.8.html)
