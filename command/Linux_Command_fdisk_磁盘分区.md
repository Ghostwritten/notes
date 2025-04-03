#  Linux Command fdisk 
tags: lvm



![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9c7a22da42a2c0f9545240da38ade1cf.jpeg#pic_center)

##  1. 简介
[fdisk](https://man7.org/linux/man-pages/man8/fdisk.8.html) 代表（用于“固定磁盘或格式化磁盘”）是Linux/Unix系统中最常用的基于命令行的磁盘操作实用程序。借助 fdisk 命令，您可以使用其自己的用户友好的基于文本的菜单驱动界面查看、创建、调整大小、删除、更改、复制和移动硬盘驱动器上的分区。

该工具在为新分区创建空间、为新驱动器组织空间、重新组织旧驱动器以及将数据复制或移动到新磁盘方面非常有用。它允许您根据系统中硬盘的大小创建最多四个新的主分区和逻辑（扩展）分区的数量。


##  2. fdisk -l 查看磁盘分区

```bash
$ fdisk -l
　　以下是表示第一块硬盘 hda
　　Disk /dev/hda: 80.0 GB, 80026361856 bytes
　　255 heads, 63 sectors/track, 9729 cylinders
　　Units = cylinders of 16065 * 512 = 8225280 bytes
　　Device Boot Start End Blocks Id System
　　/dev/hda1 * 1 765 6144831 7 HPFS/NTFS           主分区
　　/dev/hda2 766 2805 16386300 c W95 FAT32 (LBA)   主分区
　　/dev/hda3 2806 9729 55617030 5 Extended         扩展分区
　　/dev/hda5 2806 3825 8193118+ 83 Linux           逻辑分区
　　/dev/hda6 3826 5100 10241406 83 Linux           逻辑分区
　　/dev/hda7 5101 5198 787153+ 82 Linux swap / Solaris 逻辑分区
　　/dev/hda8 5199 6657 11719386 83 Linux           逻辑分区
　　/dev/hda9 6658 7751 8787523+ 83 Linux           逻辑分区
　　/dev/hda10 7752 9729 15888253+ 83 Linux         逻辑分区
　　以下是表示第二块硬盘sda
　　Disk /dev/sda: 1035 MB, 1035730944 bytes
　　256 heads, 63 sectors/track, 125 cylinders
　　Units = cylinders of 16128 * 512 = 8257536 bytes
　　Device Boot Start End Blocks Id System
　　/dev/sda1 1 25 201568+ c W95 FAT32 (LBA)       主分区
　　/dev/sda2 26 125 806400 5 Extended             扩展分区
　　/dev/sda5 26 50 201568+ 83 Linux
　　/dev/sda6 51 76 200781 83 Linux
```
通过上面的信息，我们知道此机器中挂载两个硬盘（或移动硬盘），其中一个是hda 另一个是sda ；如果我们想查看单个硬盘情况，可以通过 fdisk -l /dev/hda1 或者fdisk -l /dev/sda1 来操作；以fdisk -l 输出的硬盘标识为准；
　

 - 其中 hda有三个主分区（包括扩展分区），分别是主分区 hda1 hda2 和hda3（扩展分区） ；逻辑分区是 hda5到hda10；
 - 其中 sda 有两个主分区（包括扩展分区），分别是 hda1 和hda2 （扩展分区）；逻辑分区是 sda5 hda6 ；
 - 硬盘总容量=主分区（包括扩展分区）总容量
 - 扩展分区容量=逻辑分区总容量
 - 通过上面的例子，我们可以得知
   hda=hda1+hda2+hda3，其中hda3=hda5+hda6+hda7+hda8+hda9+hda10 …… ……

###  2.1 fdisk -l 数值说明

```bash
Disk /dev/hda: 80.0 GB, 80026361856 bytes
　　255 heads, 63 sectors/track, 9729 cylinders
　　Units = cylinders of 16065 * 512 = 8225280 bytes
```

　　这个硬盘是80G的，有255个磁面；63个扇区；9729个磁柱；每个 cylinder（磁柱）的容量是 8225280 bytes=8225.280 K（约为）=8.225280M（约为）；分区序列 引导 开始 终止 容量 分区类型ID 分区类型

```bash
　　Device Boot Start End Blocks Id System
　　/dev/hda1 * 1 765 6144831 7 HPFS/NTFS
　　/dev/hda2 766 2805 16386300 c W95 FAT32 (LBA)
　　/dev/hda3 2806 9729 55617030 5 Extended
　　/dev/hda5 2806 3825 8193118+ 83 Linux
　　/dev/hda6 3826 5100 10241406 83 Linux
　　/dev/hda7 5101 5198 787153+ 82 Linux swap / Solaris
　　/dev/hda8 5199 6657 11719386 83 Linux
　　/dev/hda9 6658 7751 8787523+ 83 Linux
　　/dev/hda10 7752 9729 15888253+ 83 Linux
```
说明：

 - 硬盘分区的表示：在Linux 是通过hd*x 或 sd*x 表示的，其中 * 表示的是a、b、c …… …… x表示的数字 1、2、3  …… …… hd大多是IDE硬盘；sd大多是SCSI或移动存储；
 - `引导（Boot）`：表示引导分区，在上面的例子中 hda1 是引导分区；
 - `Start （开始）`：表示的一个分区从X cylinder（磁柱）开始；
 - `End （结束）`：表示一个分区到 Y cylinder（磁柱）结束；
 - 　`id和System` 表示的是一个意思，id看起来不太直观，我们要在fdisk 一个分区时，通过指定id来确认分区类型；比如 7表示的就NTFS 分区；这个在fdisk 中要通过t功能来指定。下面的部份会提到；
 - `Blocks（容量）`：这是我翻译的，其实不准确，表示的意思的确是容量的意思，其单位是K；一个分区容量的值是由下面的公式而来的；
　　

Blocks = （相应分区End数值 - 相应分区Start数值）x 单位cylinder（磁柱）的容量
所以我们算一下 hda1的 Blocks 的大小 ：
　　hda1 Blocks=（765-1）x8225.280=6284113.92 K = 6284.113.92M
　　

> 注：换算单位以硬盘厂家提供的10进位算起，如果以操作系统二进制来算，这个分区容量应该更少一些，得出的这个值和我们通过 `fdisk -l` 看到的 `/dev/hda1`的值是大体相当的，因为换算方法不一样，所以也不可能尽可能的精确；再加上分区时的一点损失之类，有时或大或小是存在的；

###  2.2 估算存储设备是否被完全划分
我们估算一个硬盘是否完全被划分，我们只要看 fdisk -l 输出的内容中的 cylinders（柱体） 上一个分区的End 和下一个分区的Start是不是一个连续的数字，另外要看一下每个硬盘设备的fdisk -l 的开头部份，看一下他的cylinders（柱体）的值；比如hda设备，我们看到的是 9729 cylinders ；我们通过hda的分区表可以看到上一个分区的End的值+1 就是下一个分区的Start 的值；比如 hda2的Start的值是 hda1的End 的值+1，这证明 hda1 和hda2 中间没有空白分区，是连续的，以此类推；在 hda10，我们看到 End的值是9729 ，而在fdisk -l头部信息中也有9729 cylinders，证明这个硬盘已经完全划分；

```bash
Disk /dev/sda: 1035 MB, 1035730944 bytes
256 heads, 63 sectors/track, 125 cylinders
Units = cylinders of 16128 * 512 = 8257536 bytes


    DeviceBoot          Start                End          Blocks    Id  System
/dev/sda1                            1                  25          201568+    c  W95FAT32 (LBA)
/dev/sda2                          26                125          806400      5  Extended
/dev/sda5                          26                  50          201568+  83  Linux
/dev/sda6                          51                  76          200781    83  Linux
```
我们再看看 sda 移动储是不是被完全划分了；sda有 `125`个`cylinders`（柱体），有一个主分区和一个扩展分区构成；在扩展分区中，我们看到End的值为`125`,而这个移动硬盘的`cylinder`也是`125`，这能说明这个硬盘不可能再添加任何主分区了；根据我们上面所说的sda1 sda2 sda5 sda6 之间未有任何未划分空间，但sda6 的cylinders （柱体）的End值却是 76 ，而sda总的cylinders （柱体）有125个，由此看来sda 在 sda6后面有未划分区域；至于sda有多少未划分空间，我们算一下就知道了；扩展分区总容量是 806400 K ，大约是 806.400M左右，而逻辑分区 sda5和sda6 的大小加起来是 400M左右，所以还仍有400M左右未划分空间，并且只能划分为链逻辑分区。

##  3. fdisk -s 检查分区的大小
格式化新分区后，使用fdisk 命令使用标志“ s ”（以块为单位显示大小）检查该分区的大小。这样您就可以检查任何特定设备的大小。

```bash
$ fdisk -s /dev/sda2
5194304
```

##  4. fdisk 对硬盘及分区操作
通过 `fdisk -l` 得知 `/dev/hda` 或者`/dev/sda`设备；我们如果想再添加或者删除一些分区，可以执行：

```bash
$ fdisk  /dev/hda
$ fdisk  /dev/sda
```
###  4.1 fdisk 说明

```bash
$ fdisk /dev/sda
Command (m for help):  在这里按m ，就会输出帮助；
Command action
    a    toggle a bootable flag
    b    edit bsd disklabel
    c    toggle the dos compatibilityflag
    d    delete apartition    注：这是删除一个分区的动作；
    l    list known partitiontypes  注：l是列出分区类型，以供我们设置相应分区的类型；
    m    print thismenu  注：m 是列出帮助信息；
    n    add a new partition注：添加一个分区；
    o    create a new empty DOSpartition table
    p    print the partition table注：p列出分区表；
    q    quit without saving changes注：不保存退出；
    s    create a new empty Sundisklabel     
    t    change a partition's systemid  注：t 改变分区类型；
    u    change display/entryunits 
    v    verify the partitiontable
    w    write table to disk andexit  注：把分区表写入硬盘并退出；
    x    extra functionality (expertsonly)  注：扩展应用，专家功能；
```
常用：d l m p q t w

###  4.2 p 列出当前操作硬盘的分区情况用 

```bash
Command (m for help): p

Disk /dev/sda: 1035 MB, 1035730944 bytes
256 heads, 63 sectors/track, 125 cylinders
Units = cylinders of 16128 * 512 = 8257536 bytes

    DeviceBoot          Start                End          Blocks    Id  System
/dev/sda1                            1                  25          201568+    c  W95FAT32 (LBA)
/dev/sda2                          26                125          806400      5  Extended
/dev/sda5                          26                  50          201568+  83  Linux
/dev/sda6                          51                  76          200781    83  Linux
```
### 4.3 d 指令来删除一个分区

```bash
Command (m for help):p      注：列出分区情况；

Disk /dev/sda: 1035 MB, 1035730944 bytes
256 heads, 63 sectors/track, 125 cylinders
Units = cylinders of 16128 * 512 = 8257536 bytes

    DeviceBoot          Start                End          Blocks    Id  System
/dev/sda1                            1                  25          201568+    c  W95FAT32 (LBA)
/dev/sda2                          26                125          806400      5  Extended

/dev/sda5                          26                  50          201568+  83  Linux
/dev/sda6                          51                  76          200781    83  Linux

Command (m for help): d 注：执行删除分区指定；
Partition number (1-6): 6  注：我想删除 sda6 ，就在这里输入 6；

Command (m for help): p 注：再查看一下硬盘分区情况，看是否删除了？

Disk /dev/sda: 1035 MB, 1035730944 bytes
256 heads, 63 sectors/track, 125 cylinders
Units = cylinders of 16128 * 512 = 8257536 bytes


    DeviceBoot          Start                End          Blocks    Id  System
/dev/sda1                            1                  25          201568+    c  W95FAT32 (LBA)
/dev/sda2                          26                125          806400      5  Extended
/dev/sda5                          26                  50          201568+  83  Linux

Command (m for help):
```

> 警告：删除分区时要小心，请看好分区的序号，如果您删除了扩展分区，扩展分区之下的逻辑分区都会删除；所以操作时一定要小心；如果知道自己操作错了，请不要惊慌，用q不保存退出；切记切记！！！！在分区操作错了之时，千万不要输入w保存退出！！！

###  4.4 n 指令增加一个分区

```bash
Command (m for help): p

Disk /dev/sda: 1035 MB, 1035730944 bytes
256 heads, 63 sectors/track, 125 cylinders
Units = cylinders of 16128 * 512 = 8257536 bytes

    DeviceBoot          Start                End          Blocks    Id  System
/dev/sda1                            1                  25          201568+    c  W95FAT32 (LBA)
/dev/sda2                          26                125          806400      5  Extended
/dev/sda5                          26                  50          201568+  83  Linux

Command (m for help): n  注：增加一个分区；
Command action
    l    logical (5 orover)  注：增加逻辑分区，分区编号要大于5；为什么要大于5，因为已经有sda5了；
    p    primary partition (1-4)注：增加一个主分区；编号从 1-4 ；但sda1 和sda2都被占用，所以只能从3开始；
p
Partition number (1-4): 3
No free sectors available  注：失败中，为什么失败？
```

我试图增加一个主分区，看来是失败了，为什么失败？因为我们看到主分区+扩展分区把整个磁盘都用光了，看扩展分区的End的值，再看一下p输出信息中有125 cylinders；最好还是看前面部份；那里有提到；所以我们只能增加逻辑分区了；

```bash
Command (m for help): n
Command action
    l    logical (5 or over)
    p    primary partition (1-4)
l    注：在这里输入l，就进入划分逻辑分区阶段了；
First cylinder (51-125, default51):    注：这个就是分区的Start值；这里最好直接按回车，如果您输入了一个非默认的数字，会造成空间浪费；
Using default value 51
Last cylinder or +size or +sizeM or +sizeK (51-125, default 125):+200M  注：这个是定义分区大小的，+200M 就是大小为200M；当然您也可以根据p提示的单位cylinder的大小来算，然后来指定End的数值。回头看看是怎么算的；还是用+200M这个办法来添加，这样能直观一点。如果您想添加一个10G左右大小的分区，请输入+10000M ；

Command (m for help):
```

###  4.5 t 指令指定分区类型

```bash
Command (m for help): t  注：通过t来指定分区类型；
Partition number (1-6): 6 注：要改变哪个分区类型呢？我指定了6，其实也就是sda6
Hex code (type L to list codes):L  注：在这里输入L，就可以查看分区类型的id了；
Hex code (type L to list codes): b  注：如果我想让这个分区是W95 FAT32 类型的，通过L查看得知 b是表示的是，所以输入了b；
Changed system type of partition 6 to b (W95FAT32)  注：系统信息，改变成功；是否是改变了，请用p查看；

Command (m for help): p

Disk /dev/sda: 1035 MB, 1035730944 bytes
256 heads, 63 sectors/track, 125 cylinders
Units = cylinders of 16128 * 512 = 8257536 bytes

    DeviceBoot          Start                End          Blocks    Id  System
/dev/sda1                            1                  25          201568+    c  W95FAT32 (LBA)
/dev/sda2                          26                125          806400      5  Extended
/dev/sda5                          26                  50          201568+  83  Linux
/dev/sda6                          51                  75          201568+    b  W95FAT32
```
###  4.6 q 或 w 退出
其中 q是 不保存退出，w是保存退出；

```bash
Command (m for help): w
或
Command (m for help): q
```


##  5. 添加分区
添加两个200M的主分区，其它为扩展分区，在扩展分区中我们添加两个200M大小的逻辑分区；

```bash
Command (m for help): p 注：列出分区表；

Disk /dev/sda: 1035 MB, 1035730944 bytes
256 heads, 63 sectors/track, 125 cylinders
Units = cylinders of 16128 * 512 = 8257536 bytes

    DeviceBoot          Start                End          Blocks    Id  System

Command (m for help): n 注：添加分区；
Command action
    e    extended
    p    primary partition (1-4)
p  注：添加主分区；
Partition number (1-4): 1    注：添加主分区1；
First cylinder (1-125, default1):    注：直接回车，主分区1的起始位置；默认为1,默认就好；
Using default value 1
Last cylinder or +size or +sizeM or +sizeK (1-125, default 125):+200M    注：指定分区大小，用+200M来指定大小为200M

Command (m for help): n  注：添加新分区；
Command action
    e    extended
    p    primary partition (1-4)
p  注：添加主分区
Partition number (1-4): 2  注：添加主分区2；
First cylinder (26-125, default 26):
Using default value 26
Last cylinder or +size or +sizeM or +sizeK (26-125, default 125):+200M 注：指定分区大小，用+200M来指定大小为200M

Command (m for help): n
Command action
    e    extended
    p    primary partition (1-4)
e  注：添加扩展分区；
Partition number (1-4): 3  注：指定为3，因为主分区已经分了两个了，这个也算主分区，从3开始；
First cylinder (51-125, default 51):  注：直接回车；
Using default value 51
Last cylinder or +size or +sizeM or +sizeK (51-125, default125):    注：直接回车，把其余的所有空间都给扩展分区；
Using default value 125

Command (m for help): p

Disk /dev/sda: 1035 MB, 1035730944 bytes
256 heads, 63 sectors/track, 125 cylinders
Units = cylinders of 16128 * 512 = 8257536 bytes

    DeviceBoot          Start                End          Blocks    Id  System
/dev/sda1                            1                  25          201568+  83  Linux
/dev/sda2                          26                  50          201600    83  Linux
/dev/sda3                          51                125          604800      5  Extended

Command (m for help): n
Command action
    l    logical (5 or over)
    p    primary partition (1-4)
l  注：添加逻辑分区；
First cylinder (51-125, default 51):
Using default value 51
Last cylinder or +size or +sizeM or +sizeK (51-125, default 125):+200M  注：添加一个大小为200M大小的分区；

Command (m for help): n
Command action
    l    logical (5 or over)
    p    primary partition (1-4)
l  注：添加一个逻辑分区；
First cylinder (76-125, default 76):
Using default value 76
Last cylinder or +size or +sizeM or +sizeK (76-125, default 125):+200M 注：添加一个大小为200M大小的分区；

Command (m for help): p  列出分区表；

Disk /dev/sda: 1035 MB, 1035730944 bytes
256 heads, 63 sectors/track, 125 cylinders
Units = cylinders of 16128 * 512 = 8257536 bytes

    DeviceBoot          Start                End          Blocks    Id  System
/dev/sda1                            1                  25          201568+  83  Linux
/dev/sda2                          26                  50          201600    83  Linux
/dev/sda3                          51                125          604800      5  Extended
/dev/sda5                          51                  75          201568+  83  Linux
/dev/sda6                          76                100          201568+  83  Linux
```
##  6. 刷新分区表

```bash
partprobe         
```
##  7. 查看文件系统类型

```bash
$ fdisk -l /dev/sda
Disk /dev/sda: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: E1FFAD5C-DD8B-449C-B206-A51240EDD798

Device       Start      End  Sectors Size Type
/dev/sda1     2048     4095     2048   1M BIOS boot
/dev/sda2     4096  2101247  2097152   1G Linux filesystem
/dev/sda3  2101248 41940991 39839744  19G Linux filesystem


$ blkid /dev/sda
/dev/sda: PTUUID="e1ffad5c-dd8b-449c-b206-a51240edd798" PTTYPE="gpt"

$ blkid /dev/sda1
/dev/sda1: PARTUUID="9a165a33-0718-42c7-b5e7-7e2088aac72e"

$ blkid /dev/sda2
/dev/sda2: UUID="379ce519-a3c2-46c9-9748-deb5b7845231" TYPE="ext4" PARTUUID="0bc3691b-5ef1-4b9d-bf3d-2174ddbaaab5"

```

##  8. 对分区格式化与挂载

格式化类型：

 - mkfs.bfs
 - mkfs.ext2
 - mkfs.jfs
 - mkfs.msdos
 - mkfs.vfat
 - mkfs.cramfs
 - mkfs.ext3
 - mkfs.minix
 - mkfs.reiserfs

`mkfs.xfs` 等命令来格式化分区，比如我想格式化sda6为ext3文件系统，则输入；

```bash
mkfs.ext3 /dev/sda6
```
如果我想加载 sda6到目前系统来存取文件，应该有mount 命令，但首先您得建一个挂载目录；比如 /data

```bash
mkdir /data
mount /dev/sda6 /data
df -lh
```
如果卸载

```bash
umount /data
df -lh
```
##  9. 自动化挂载

```bash
$ tail -2 /etc/fstab
/dev/sdb1 /data1 ext4 defaults 0 0
/dev/sb2 /data2 xfs  defaults 0 0

$ mount -a
```
##  10. 修复分区表顺序
如果您删除了逻辑分区并再次重新创建它，您可能会注意到“分区乱序”问题或错误消息，例如“分区表条目未按磁盘顺序”。

例如，当三个逻辑分区（如sda4、sda5和sda6）被删除并创建新分区时，您可能期望新分区名称为sda4。但是，系统会将其创建为sda5。发生这种情况的原因是，在删除分区后，sda7分区已被移动为sda4，并且可用空间移至末尾。

要解决此类分区顺序问题，并将sda4分配给新创建的分区，请发出“ `x` ”进入额外功能部分，然后输入“ `f` ”专家命令来修复分区表的顺序，如下所示。

```bash
$ fdisk  /dev/sda

WARNING: DOS-compatible mode is deprecated. It's strongly recommended to
         switch off the mode (command 'c') and change display units to
         sectors (command 'u').

Command (m for help): x

Expert command (m for help): f
Done.

Expert command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.
```
之后，运行' `f` '命令，不要忘记运行' `w` '命令保存并退出fdisk命令模式。一旦它修复了分区表顺序，您将不再收到错误消息。

参考：

 - [10 fdisk Commands to Manage Linux Disk Partitions](https://www.tecmint.com/fdisk-commands-to-manage-linux-disk-partitions/)
 - [linux_fdisk命令详解](https://www.cnblogs.com/xiaofengkang/archive/2011/06/06/2073579.html)
 - [fdisk(8) — Linux manual page](https://man7.org/linux/man-pages/man8/fdisk.8.html)


