#  Linux Command parted 创建分区
tags: lvm
![](https://img-blog.csdnimg.cn/cd45d09447224d90a527d81cdaadb49c.png)


##  1. 简介
虽然我们可以使用 fdisk命令对硬盘进行快速的分区，但对高于 2TB 的硬盘分区，此命令却无能为力，此时就需要使用 parted 命令。

##  2. 交互模式

```bash
$ parted /dev/sdb
#打算继续划分/dev/sdb硬盘
GNU Parted 2.1
使用/dev/sdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted)   <--parted 的等待输入交互命令的位置，输入 help，可以看到在交互模式下支持的所有命令
```
### 2.1 常见命令

| parted交互命令                             | 说 明                  |
|----------------------------------------|----------------------|
| check NUMBER                           | 做一次简单的文件系统检测         |
| cp [FROM-DEVICE] FROM-NUMBER TO-NUMBER | 复制文件系统到另一个分区         |
| help [COMMAND]                         | 显示所有的命令帮助            |
| mklabel,mktable LABEL-TYPE             | 创建新的磁盘卷标（分区表）        |
| mkfs NUMBER FS-TYPE                    | 在分区上建立文件系统           |
| mkpart PART-TYPE [FS-TYPE] START END   | 创建一个分区               |
| mkpartfs PART-TYPE FS-TYPE START END   | 创建分区，并建立文件系统         |
| move NUMBER START END                  | 移动分区                 |
| name NUMBER NAME                       | 给分区命名                |
| print [devices|free|list,all|NUMBER]   | 显示分区表、活动设备、空闲空间、所有分区 |
| quit                                   | 退出                   |
| rescue START END                       | 修复丢失的分区              |
| resize NUMBER START END                | 修改分区大小               |
| rm NUMBER                              | 删除分区                 |
| select DEVICE                          | 选择需要编辑的设备            |
| set NUMBER FLAG STATE                  | 改变分区标记               |
| toggle [NUMBER [FLAG]]                 | 切换分区表的状态             |
| unit UNIT                              | 设置默认的单位              |
| Version                                | 显示版本                 |


###  2.2 查看分区表

```bash
(parted) print               #进入print指令
Model: VMware, VMware Virtual S (scsi)   #硬盘参数，是虚拟机
Disk/dev/sdb: 21.5GB   #硬盘大小
Sector size (logical/physical): 512B/512B   #扇区大小
Partition Table: msdos     #分区表类型，是MBR分区表
Number Start End Size Type File system 标志  #看到了我们使用fdisk命令创建的分区，其中1分区没被格式化；2分区是扩展分区，不能被格式化
1 32.3kB 5379MB 5379MB primary
2 5379MB 21.5GB 16.1GB extended
5 5379MB 7534MB 2155MB logical ext4
6 7534MB 9689MB 2155MB logical ext4
```
使用 print 命令可以査看分区表信息，包括硬盘参数、硬盘大小、扇区大小、分区表类型和分区信息。分区信息共有 7 列，分别如下：

- `Number`：分区号，比如，1号就代表 /dec/sdb1；
- `Start`：分区起始位置。这里不再像 fdisk 那样用柱面表示，使用字节表示更加直观；
- `End`：分区结束位置；
- `Size`：分区大小；
- `Type`：分区类型，有 primary、extended、logical 等类型；
- `Filesystem`：文件系统类型；
- 标志：分区的标记。


###  2.3 修改成 GPT 分区表

```bash
(partcd) mklabel gpt   #修改分区表命令，警告：正在使用/dev/sdb上的分区。由于/dev/sdb分区已经挂载，所以有警告。注意，如果强制修改，那么原有分区及数据会消失
忽略/Ignore/放弃/Cancel? ignore    #输入ignore忽略报错
警告：The existing disk label on /dev/sdb will be destroyed and all data on this disk will be lost. Do you want to continue?
是/Yes/否/No? yes
警告：WARNING: the kernel failed to re-read the partition table on /dev/sdb (设 备或资源忙）.As a result, it may not reflect all of your changes until after reboot.  #下次重启后才能生效
(parted) print  #查看一下分区表
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt     #分区表已经变成 GPT
Number Start End Size File system Name 标志   #所有的分区都消失了
```
修改了分区表，如果这块硬盘上已经有分区了，那么原有分区和分区中的数据都会消失，而且需要重启系统才能生效。

另外，我们转换分区表的目的是支持**大于 2TB** 的分区，如果分区并没有大于 2TB，那么这一步是可以不执行的。

> 注意，一定要把 /etc/fstab 文件和原有分区中的内容删除才能重启，否则会报错。

### 2.4 建立分区
因为修改过了分区表，所以`/dev/sdb`硬盘中的所有数据都消失了，我们就可以重新对这块硬盘分区了。不过，在建立分区时，默认文件系统就只能是 `ext2` 了。命令如下：

```bash
(parted)mkpart  #输入创建分区命令，后面不要参数，全部靠交互
分区名称？ []?disk1  #分区名称，这里命名为disk 1
文件系统系统？ [ext2]?    #文件系统类型，直接回车，使用默认文件系统ext2
起始点？ 1MB   #分区从1MB开始
结束点？5GB分区到5GB结束    #分区完成
(parted) print    #查看一下
Model: VMware, VMware Virtual S (scsi)
Disk/dev/sdb: 21.5GB
Sector size (logical/physical): 512B/512B Partition Table: gpt
Number Start End Size Rle system Name 标志
1 1049kB 5000MB 4999MB disk1  #分区1已经出现
```
不知道大家有没有注意到，我们现在用 print 查看的分区和第一次查看 MBR 分区表的分区时有些不一样了，少了 Type 这个字段，也就是分区类型字段，多了 Name（分区名）字段。分区类型是用于标识主分区、扩展分区和逻辑分区的，不过这种标识只在 `MBR` 分区表中使用，现在已经变成了 `GPT` 分区表，所以就不再有 `Type` 类型了。


### 2.5 建立文件系统
分区分完后，还需要进行格式化。我们知道，如果使用 `parted` 交互命令格式化，则只能格式化成 `ext2` 文件系统。我们在这里要演示一下 `parted` 命令的格式化方法，所以就格式化成 `ext2` 文件系统。命令如下：

```bash
(parted) mkfs    #格式化命令（很奇怪，也是mkfs，但是这只是parted的交互命令）
WARNING: you are attempting to use parted to operate on (mkfs) a file system.
parted's file system manipulation code is not as robust as what you'll find in
dedicated, file-system-specific packages like e2fsprogs. We recommend
you use parted only to manipulate partition tables, whenever possible.
Support for performing most operations on most types of file systems
will be removed in an upcoming release.
警告：The existing file system will be destroyed and all data on the partition will be lost. Do you want to continue?
是/Yes/否/No? yes    #警告你格式化丟失，没关系，已经丢失过了
分区编号？ 1
文件系统类型 [ext2]?   #指定文件系统类型，写别的也没用，直接回车
(parted) print #格式化完成，查看一下
Model: VMware, VMware Virtual S (scsi)
Disk/dev/sdb: 21,5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Number Start End Size File system Name标志
1 1049kB 5000MB 4999MB ext2 diski  #拥有了文件系统
```
如果要格式化成 ext4 文件系统，那么请 mkfs 命令帮忙吧（注意：不是 parted 交互命令中的 mkfs，而是系统命令 mkfs)。

### 2.6 调整分区大小
parted 命令还有一大优势，就是可以调整分区的大小（在 Windows 中也可以实现，不过要么需要转换成动态磁盘，要么需要依赖第三方工具，如硬盘分区魔术师）。起始 Linux 中 LVM 和 RAID 是可以支持分区调整的，不过这两种方法也可以看成动态磁盘方法，使用 parted 命令调整分区更加简单。

注意，parted 调整已经挂载使用的分区时，是不会影响分区中的数据的，也就是说，数据不会丢失。但是一定要先卸载分区，再调整分区大小，否则数据是会出现问题的。另外，要调整大小的分区必须已经建立了文件系统（格式化），否则会报错。

```bash
(parted) resize   
分区编号？ 1    ##指定要修改的分区编号
起始点？ [1049kB]? 1MB   ##分区起始位置
结束点？ [5000MB]? 6GB   #分区结束位置
(parted) print    #查看一下
Model: VMware, VMware Virtual S (scsi)
Disk/dev/sdb: 21,5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Number Start End Size File system Name标志
1 1049kB 6000MB 5999MB ext2 diski    #分区大小改变
```
###  2.7 删除分区

```bash
(parted) rm   #删除分区命令
分区编号？ 1  #指定分区编号
(parted) print  #查看一下
Model: VMware, VMware Virtual S (scsi)
Disk/dev/sdb: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Number Start End Size File system Name 标志 #分区消失
```

> 要注意的是，parted 中所有的操作都是立即生效的，没有保存生效的概念。这一点和 fdisk
> 交互命令明显不同，所以做的所有操作大家要加倍小心。

## 3. 命令行模式
```bash
sudo parted -s -a optimal -- /dev/sdb mklabel gpt
sudo parted -s -a optimal -- /dev/sdb  mkpart primary 0% 100%
sudo parted -s -- /dev/sdb  align-check optimal 1
sudo pvcreate /dev/sdb1
sudo vgcreate vg0 /dev/sdb1
sudo lvcreate -n harbor -l +100%FREE vg0
sudo mkfs.xfs /dev/vg0/harbor
sudo mkdir /data
echo "/dev/vg0/harbor /data xfs defaults 0 0" | sudo tee -a /etc/fstab
```
挂载验证
```bash
$ sudo mount -a
$ df -hT /data/
Filesystem             Type  Size  Used Avail Use% Mounted on
/dev/mapper/vg0-harbor xfs   200G  1.5G  199G   1% /data
```

参考：
- [Linux parted命令用法详解：创建分区](http://c.biancheng.net/view/905.html)
- [parted(8) — Linux manual page](https://man7.org/linux/man-pages/man8/parted.8.html)
