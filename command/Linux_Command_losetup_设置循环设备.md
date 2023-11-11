#  Linux Command losetup 设置循环设备



----


## 1. 背景

 在类 UNIX 系统里，loop 设备是一种伪设备(pseudo-device)，或者也可以说是仿真设备。它能使我们像块设备一样访问一个文件。
在使用之前，一个 loop 设备必须要和一个文件进行连接。这种结合方式给用户提供了一个替代块特殊文件的接口。因此，如果这个文件包含有一个完整的文件系统，那么这个文件就可以像一个磁盘设备一样被 mount 起来。
    上面说的文件格式，我们经常见到的是 CD 或 DVD 的 ISO 光盘镜像文件或者是软盘(硬盘)的 *.img 镜像文件。通过这种 loop mount (回环mount)的方式，这些镜像文件就可以被 mount 到当前文件系统的一个目录下。
    至此，顺便可以再理解一下 loop 之含义：对于第一层文件系统，它直接安装在我们计算机的物理设备之上；而对于这种被 mount 起来的镜像文件(它也包含有文件系统)，它是建立在第一层文件系统之上，这样看来，它就像是在第一层文件系统之上再绕了一圈的文件系统，所以称为 loop。


##  2. 简介
Linux losetup命令用于设置循环设备。

循环设备可把文件虚拟成区块设备，籍以模拟整个文件系统，让用户得以将其视为硬盘驱动器，光驱或软驱等设备，并挂入当作目录来使用。

## 3. 语法

```bash
losetup [-d][-e <加密方式>][-o <平移数目>][循环设备代号][文件]
```
## 4. 参数

 - -d 卸除设备。
 - -e<加密方式> 启动加密编码。
 - -o<平移数目> 设置数据平移的数目。

## 5. 实例
### 5.1 示例1
（1）创建空的磁盘镜像文件，这里创建一个1.44M的软盘

```c
$ dd if=/dev/zero of=floppy.img bs=512 count=2880
```

（2）使用 losetup将磁盘镜像文件虚拟成块设备

```c
$ losetup /dev/loop1 floppy.img
```

（3）挂载块设备

```c
$ mount /dev/loop0 /tmp
```

经过上面的三步之后，我们就可以通过/tmp目录，像访问真实快设备一样来访问磁盘镜像文件`floppy.img`。

（4） 卸载loop设备

```c
$ umount /tmp
$ losetup -d /dev/loop1
```
###  5.2 示例2
1. 首先创建一个 1G 大小的空文件：

```c
# dd if=/dev/zero of=loopfile.img bs=1G count=1
1+0 records in
1+0 records out
1073741824 bytes (1.1 GB) copied, 69.3471 s, 15.5 MB/s
```

2. 对该文件格式化为 ext4 格式：

```c
# mkfs.ext4 loopfile.img
。。。。
```

3. 用 file 命令查看下格式化后的文件类型：

```c
# file loopfile.img
loopfile.img: Linux rev 1.0 ext4 filesystem data, UUID=a9dfb4a0-6653-4407-ae05-7044d92c1159 (extents) (large files) (huge files)
```

4. 准备将上面的文件挂载起来：

```c
# mkdir /mnt/loopback
# mount -o loop loopfile.img /mnt/loopback
```

mount 命令的 `-o loop` 选项可以将任意一个 loopback 文件系统挂载。

上面的 mount 命令实际等价于下面两条命令：

```c
# losetup /dev/loop0 loopfile.img
# mount /dev/loop0 /mnt/loopback
```

因此实际上，mount -o loop 在内部已经默认的将文件和 `/dev/loop0` 挂载起来了。

然而对于第一种方法(mount -o loop)并不能适用于所有的场景。比如，我们想创建一个硬盘文件，然后对该文件进行分区，接着挂载其中一个子分区，这时就不能用 -o loop 这种方法了。因此必须如下做：

```c
# losetup /dev/loop1 loopfile.img
# fdisk /dev/loop1
```

6. 卸载挂载点：

```c
# umount /mnt/loopback
```

参考：

 - [https://man7.org/linux/man-pages/man8/losetup.8.html](https://man7.org/linux/man-pages/man8/losetup.8.html)

