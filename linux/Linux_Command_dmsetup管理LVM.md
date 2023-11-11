#  Linux Command dmsetup管理LVM



------



## 1. 简介
`dmsetup` 命令是一个用来与 Device Mapper 沟通的命令行封装器（wrapper）。可使用 dmsetup 命令的 info、ls、status 和 deps 选项查看 LVM 设备的常规信息，如以下小节所述。

## 2. 语法

```bash
dmsetup  + <command>  + <device_name> or <uuid>
# command 常见的有 remove 、 ls 、status 等
# device_name: 可以是/dev/sd*，或者是物理卷的名称
```

## 3. 命令
### 3.1 常用命令
```bash
dmsetup clear device_name
dmsetup create device_name [-u|--uuid uuid]
         [--addnodeoncreate|--addnodeonresume]
         [-n|--notable|--table table|table_file] [--readahead
         [+]sectors|auto|none]
dmsetup create --concise [concise_device_specification]
dmsetup deps [-o options] [device_name...]
dmsetup help [-c|-C|--columns]
dmsetup info [device_name...]
dmsetup info -c|-C|--columns [--count count] [--interval seconds]
         [--nameprefixes] [--noheadings] [-o fields] [-O|--sort
         sort_fields] [--separator separator] [device_name]
dmsetup load device_name [--table table|table_file]
dmsetup ls [--target target_type] [--exec command] [--tree] [-o
         options]
dmsetup mangle [device_name...]
dmsetup message device_name sector message
dmsetup mknodes [device_name...]
dmsetup reload device_name [--table table|table_file]
dmsetup remove [-f|--force] [--retry] [--deferred] device_name...
dmsetup remove_all [-f|--force] [--deferred]
dmsetup rename device_name new_name
dmsetup rename device_name --setuuid uuid
dmsetup resume device_name...
         [--addnodeoncreate|--addnodeonresume] [--noflush]
         [--nolockfs] [--readahead [+]sectors|auto|none]
dmsetup setgeometry device_name cyl head sect start
dmsetup splitname device_name [subsystem]
dmsetup stats command [options]
dmsetup status [--target target_type] [--noflush]
         [device_name...]
dmsetup suspend [--nolockfs] [--noflush] device_name...
dmsetup table [--concise] [--target target_type] [--showkeys]
         [device_name...]
dmsetup targets
dmsetup udevcomplete cookie
dmsetup udevcomplete_all [age_in_minutes]
dmsetup udevcookie
dmsetup udevcreatecookie
dmsetup udevflags cookie
dmsetup udevreleasecookie [cookie]
dmsetup version
dmsetup wait [--noflush] device_name [event_nr]
dmsetup wipe_table device_name...  [-f|--force] [--noflush]
         [--nolockfs]

devmap_name major minor
devmap_name major:minor
```

### 3.2 dmsetup info

```c
# dmsetup info
Name:              testgfsvg-testgfslv1
State:             ACTIVE
Read Ahead:        256
Tables present:    LIVE
Open count:        0
Event number:      0
Major, minor:      253, 2
Number of targets: 2
UUID: LVM-K528WUGQgPadNXYcFrrf9LnPlUMswgkCkpgPIgYzSvigM7SfeWCypddNSWtNzc2N
...
Name:              VolGroup00-LogVol00
State:             ACTIVE
Read Ahead:        256
Tables present:    LIVE
Open count:        1
Event number:      0
Major, minor:      253, 0
Number of targets: 1
UUID: LVM-tOcS1kqFV9drb0X1Vr8sxeYP0tqcrpdegyqj5lZxe45JMGlmvtqLmbLpBcenh2L3
```
dmsetup info 命令提供以下分类信息：

```bash
Name
设备名称。LVM 设备以用小横线分隔的卷组名称和逻辑卷名称表示。在源名称中小横线会转换为两个小横线。在标准 LVM 操作过程中，不应使用这种格式的 LVM 设备名称直接指定 LVM 设备，而是应该使用 vg/lv 指定。
State
可能的设备状态是 SUSPENDED、ACTIVE 和 READ-ONLY。dmsetup suspend 命令将设备状态设定为 SUSPENDED。当挂起某个设备时，会停止对该设备的所有 I/O 操作。使用 dmsetup resume 命令可将设备状态恢复到 ACTIVE。
Read Ahead
系统对正在进行读取操作的任意打开文件的预读数据块数目。默认情况下，内核会自动选择一个合适的值。可使用 dmsetup 命令的 --readahead 选项更改这个值。
Tables present
这个类型的可能状态为 LIVE 和 INACTIVE。INACTIVE 状态表示已经载入了表格，且会在 dmsetup resume 命令将某个设备状态恢复为 ACTIVE 时进行切换，届时表格状态将为 LIVE。有关详情请参考 dmsetup man page。
Open count
打开参考计数表示打开该设备的次数。mount 命令会打开一个设备。
Event number
目前收到的事件数目。使用 dmsetup wait n 命令允许用户等待第 n 个事件，收到该事件前阻断该调用。
Major, minor
主设备号码和副设备号码
Number of targets
组成某个设备的片段数目。例如：一个跨三个磁盘的线性设备会有三个目标。线性设备由某个磁盘起始和结尾，而不是中间组成的线性设备有两个目标。
UUID
该设备的 UUID。
```
### 3.3 dmsetup ls 
可以使用 `dmsetup ls` 命令列出映射的设备的设备名称列表。可以使用 `dmsetup ls --target target_type` 命令列出至少有一个指定类型目标的设备。有关 dmsetup ls 的其他选项

```c
# dmsetup ls
testgfsvg-testgfslv3    (253:4)
testgfsvg-testgfslv2    (253:3)
testgfsvg-testgfslv1    (253:2)
VolGroup00-LogVol01     (253:1)
VolGroup00-LogVol00     (253:0)
```

```c
# dmsetup ls --target mirror
lock_stress-grant--02.1722      (253, 34)
lock_stress-grant--01.1720      (253, 18)
lock_stress-grant--03.1718      (253, 52)
lock_stress-grant--02.1716      (253, 40)
lock_stress-grant--03.1713      (253, 47)
lock_stress-grant--02.1709      (253, 23)
lock_stress-grant--01.1707      (253, 8)
lock_stress-grant--01.1724      (253, 14)
lock_stress-grant--03.1711      (253, 27)
```
在多路径或者其它 device mapper 装置中堆叠的 LVM 配置文件可能过于复杂。dmsetup ls 命令提供了一个--tree 选项，可以树形式显示设备间的相依性，如下所示。

```bash
# dmsetup ls --tree
vgtest-lvmir (253:13)
 ├─vgtest-lvmir_mimage_1 (253:12)
 │  └─mpathep1 (253:8)
 │     └─mpathe (253:5)
 │        ├─ (8:112)
 │        └─ (8:64)
 ├─vgtest-lvmir_mimage_0 (253:11)
 │  └─mpathcp1 (253:3)
 │     └─mpathc (253:2)
 │        ├─ (8:32)
 │        └─ (8:16)
 └─vgtest-lvmir_mlog (253:4)
    └─mpathfp1 (253:10)
       └─mpathf (253:6)
          ├─ (8:128)
          └─ (8:80)
```

### 3.4 dmsetup status
`dmsetup status device` 命令提供指定设备中每个目标的状态信息。如果没有指定设备名称，输出结果是所有目前配置的设备映射器设备信息。可以使用 `dmsetup status --target target_type` 命令列出那些至少有一个指定类型目标的设备。

```bash
# dmsetup status
testgfsvg-testgfslv3: 0 312352768 linear 
testgfsvg-testgfslv2: 0 312352768 linear 
testgfsvg-testgfslv1: 0 312352768 linear 
testgfsvg-testgfslv1: 312352768 50331648 linear 
VolGroup00-LogVol01: 0 4063232 linear 
VolGroup00-LogVol00: 0 151912448 linear
```
### 3.5 dmsetup deps 
`dmsetup deps device` 命令为指定设备的映射列表参考的设备提供（major，minor）对列表。如果没有指定设备名称，则输出所有目前配置的设备映射器设备信息。

```c
# dmsetup deps
testgfsvg-testgfslv3: 1 dependencies    : (8, 16)
testgfsvg-testgfslv2: 1 dependencies    : (8, 16)
testgfsvg-testgfslv1: 1 dependencies    : (8, 16)
VolGroup00-LogVol01: 1 dependencies     : (8, 2)
VolGroup00-LogVol00: 1 dependencies     : (8, 2)
```

```c
# dmsetup deps lock_stress-grant--02.1722
3 dependencies  : (253, 33) (253, 32) (253, 31)
```
### 3.6 dmsetup table
显示table信息
```bash
# dmsetup table
docker-253:2-8409152-pool: 0 419430400 thin-pool 7:1 7:0 128 32768 1 skip_block_zeroing 
docker-docker--lv: 0 16777216 linear 8:16 2048
centos-swap: 0 4194304 linear 8:2 2048
centos-root: 0 98549760 linear 8:2 4196352
```
### 3.7 dmsetup create

```bash
dmsetup create dm-name dm-table
```

 - dm-name是要创建的设备名字。 如果成功将会在/dev/mapper/目录下 生成这个名字的文件
 - dm-table是保存了这个map device的 mapping table。

```bash
起始扇区  扇区个数  线性映射  目标设备 目标设备上的起始扇区

0 1025 linear /dev/sdb2 0

1025 1025 linear /dev/sdc2 0
```

某个block的sector大小，可以用 `blockdev --getsize /dev/sdb2`来得到。
如果是刚分区的盘，需要用partprobe来更新一下。 
查看linear设备的写入顺序
将新生成的设备创建文件系统，mount到某个目录。
cd到这个目录下，运行
```bash
dd if=/dev/zero of=test 
```
这个命令将一直运行到写满磁盘。
在运行之前可以运行
```bash
watch -n 1 iostat /dev/sdb2 /dev/sdc2
```
来观察磁盘的读写情况。 
可以发现 写入的过程是，先写入到/dev/sdb2，再写入到`/dev/sdc2`.
这个过程是和linear的概念一致的。

```bash
[root@node1 ~]# blockdev --getsize /dev/sdc1  
1012032  
[root@node1 ~]# blockdev --getsize /dev/sdd1  
1012032  
[root@node1 ~]# vi linear_table  
0 1012032 linear /dev/sdc1 0  
1012032 1012032 linear /dev/sdd1 0  
[root@node1 ~]# dmsetup create linear_test linear_table  
[root@node1 ~]# ls -l /dev/mapper/linear_test  
brw-rw---- 1 root disk 253, 0 Jan  4 10:39 /dev/mapper/linear_test
```

更多阅读：

 - [dmsetup 命令](https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/logical_volume_manager_administration/dmsetup)
 - [dmsetup(8) — Linux manual page](https://man7.org/linux/man-pages/man8/dmsetup.8.html)
