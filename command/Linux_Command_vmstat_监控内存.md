#  Linux Command vmstat 监控内存
tags: 监控, 分析



----
## 1. 简介
**vmstat**(VirtualMeomoryStatistics,虚拟内存统计) 是Linux中监控内存的常用工具,可对操作系统的虚拟内存、进程、CPU等的整体情况进行监视。
vmstat的常规用法：vmstat interval times即每隔interval秒采样一次，共采样times次，如果省略times,则一直采集数据，直到用户手动停止为止。
简单举个例子：

```c
[root@master ~]# vmstat 5
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 190476   1292 1281164    0    0    16    23  483  414  4  5 91  0  0
 0  0      0 190608   1292 1281240    0    0     0     0  903 1622  2  4 94  0  0
 3  0      0 190196   1292 1281264    0    0     0     5  948 1689  3  4 93  0  0
```

可以使用`ctrl+c`停止vmstat采集数据。
第一行显示了系统自启动以来的平均值，第二行开始显示现在正在发生的情况，接下来的行会显示每5秒间隔发生了什么，每一列的含义在头部，如下所示：

 - procs：r这一列显示了多少进程在等待cpu，b列显示多少进程正在不可中断的休眠（等待IO）。
 - memory：swapd列显示了多少块被换出了磁盘（页面交换），剩下的列显示了多少块是空闲的（未被使用），多少块正在被用作缓冲区，以及多少正在被用作操作系统的缓存。
 - swap：显示交换活动：每秒有多少块正在被换入（从磁盘）和换出（到磁盘）。
 - io：显示了多少块从块设备读取（bi）和写出（bo）,通常反映了硬盘I/O。
 - system：显示每秒中断(in)和上下文切换（cs）的数量。
 - cpu：显示所有的cpu时间花费在各类操作的百分比，包括执行用户代码（非内核），执行系统代码（内核），空闲以及等待IO。

内存不足的表现：`free  memory`急剧减少，回收buffer和cacher也无济于事，大量使用交换分区（swpd）,页面交换（swap）频繁，读写磁盘数量（io）增多，缺页中断（in）增多，上下文切换（cs）次数增多，等待IO的进程数（b）增多，大量CPU时间用于等待IO（wa）

vmstat r, b, si, so, bi, bo 这几列表示什么含义呢？
答：

```bash
[root@centos6 ~ 10:57 #39]# vmstat
procs -----------memory---------- ---swap-- -----io---- --system-- -----cpu-----
r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
0  0      0 1783964  13172 106056    0    0    29     7   15   11  0  0 99  0  0
```

 - r即running，表示正在跑的任务数
 - b即blocked，表示被阻塞的任务数
 - si表示有多少数据从交换分区读入内存
 - so表示有多少数据从内存写入交换分区
 - bi表示有多少数据从磁盘读入内存
 - bo表示有多少数据从内存写入磁盘

简记：

 - i --input，进入内存
 - o --output，从内存出去
 - s --swap，交换分区
 - b --block，块设备，磁盘

单位都是KB

## 2. 命令格式

```bash
vmstat [-a] [-n] [-S unit] [delay [ count]]

vmstat [-s] [-n] [-S unit]

vmstat [-m] [-n] [delay [ count]]

vmstat [-d] [-n] [delay [ count]]

vmstat [-p disk partition] [-n] [delay [ count]]

vmstat [-f]

vmstat [-V]
```

##  3. 命令参数

-a：显示活跃和非活跃内存

-f：显示从系统启动至今的fork数量 。

-m：显示slabinfo

-n：只在开始时显示一次各字段名称。

-s：显示内存相关统计信息及多种系统活动数量。

delay：刷新时间间隔。如果不指定，只显示一条结果。

count：刷新次数。如果不指定刷新次数，但指定了刷新时间间隔，这时刷新次数为无穷。

-d：显示磁盘相关统计信息。

-p：显示指定磁盘分区统计信息

-S：使用指定单位显示。参数有 k 、K 、m 、M ，分别代表1000、1024、1000000、1048576字节（byte）。默认单位为K（1024 bytes）

-V：显示vmstat版本信息。


## 4. 示例
### 4.1 显示虚拟内存使用情况
vmstat 5 6    #表示在5秒时间内进行6次采样。将得到一个数据汇总他能够反映真正的系统情况

```bash
[root@master ~]# vmstat 5 6
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 171568   1292 1289680    0    0    15    23  484  421  4  5 91  0  0
 1  0      0 171584   1292 1289724    0    0     0     2  888 1670  3  4 94  0  0
 0  0      0 170508   1292 1289892    0    0     0    14  940 1753  2  4 94  0  0
 5  0      0 170588   1292 1289936    0    0     0    49  919 1717  2  3 95  0  0
 1  0      0 170588   1292 1289976    0    0     0    22  948 1708  2  4 93  0  0
 0  0      0 170508   1292 1290016    0    0     0    16 1026 1794  3  4 92  0  0
```

### 4.2 显示活跃和非活跃内存

```c
[root@master ~]# vmstat -a 2 5
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free  inact active   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 181852 355052 892856    0    0    15    23  484  422  4  5 91  0  0
 4  0      0 166352 355092 904644    0    0     0   110 1516 2867  9 12 79  0  0
 2  0      0 153920 355072 916868    0    0     0   141 1292 2203  9  5 87  0  0
 1  0      0 156708 355080 915332    0    0     0     0 1230 2145  4  7 89  0  0
 1  0      0 156620 355072 915432    0    0     0    51 1104 2001  3  5 92  0  0
```
### 4.3 查看系统已经fork了多少次

vmstat -f   #这个数据是从/proc/stat中的processes字段里取得的

```c
[root@master ~]# vmstat -f
       338852 forks
```
### 4.4 查看内存使用的详细信息


vmstat -s     #这些信息的分别来自于/proc/meminfo,/proc/stat和/proc/vmstat

```c
[root@master ~]# vmstat -f
       338852 forks
[root@master ~]# vmstat -s  
      1863252 K total memory
       417256 K used memory
       918768 K active memory
       355072 K inactive memory
       152384 K free memory
         1292 K buffer memory
      1292320 K swap cache
            0 K total swap
            0 K used swap
            0 K free swap
       310918 non-nice user cpu ticks
         4074 nice user cpu ticks
       406501 system cpu ticks
      7950034 idle cpu ticks
         2466 IO-wait cpu ticks
            0 IRQ cpu ticks
        37468 softirq cpu ticks
            0 stolen cpu ticks
      1328395 pages paged in
      2002004 pages paged out
            0 pages swapped in
            0 pages swapped out
     85085982 interrupts
    208703640 CPU context switches
   1623694269 boot time
       339560 forks
```

###  4.5 查看磁盘的读/写

vmstat -d
这些信息主要来自于`/proc/diskstats`.

merged:表示一次来自于合并的写/读请求,一般系统会把多个连接/邻近的读/写请求合并到一起来操作

```c
[root@master ~]# vmstat -d
disk- ------------reads------------ ------------writes----------- -----IO------
       total merged sectors      ms  total merged sectors      ms    cur    sec
sda    16740      4 2656790   43417 283286   9061 4006396  224355      0    124
sr0        0      0       0       0      0      0       0       0      0      0
dm-0   14435      0 2592710   42246 292337      0 4002259  246995      0    124
dm-1      86      0    4144     229      0      0       0       0      0      0
```

### 4.6 查看/dev/sda1磁盘的读/写

```bash
df -h
vmstat -p /dev/sda1
```



这些信息主要来自于`/proc/diskstats`。

 - reads:来自于这个分区的读的次数。
 - read sectors:来自于这个分区的读扇区的次数。
 - writes:来自于这个分区的写的次数。
 - requested writes:来自于这个分区的写请求次数。

```c
[root@master ~]# vmstat -p /dev/sda1
sda1          reads   read sectors  writes    requested writes
                2115      52608         10       4137
```

###  4.7 查看系统的slab信息

vmstat -m

这组信息来自于`/proc/slabinfo`。

slab:由于内核会有许多小对象，这些对象构造销毁十分频繁，

比如i-node，dentry，这些对象如果每次构建的时候就向内存要一个页(4kb)，

而其实只有几个字节，这样就会非常浪费，为了解决这个问题，就引入了一种新的机制来处理在同一个页框中如何分配小存储区，

而slab可以对小对象进行分配,这样就不用为每一个对象分配页框，从而节省了空间，内核对一些小对象创建析构很频繁，

slab对这些小对象进行缓冲,可以重复利用,减少内存分配次数。

```c
[root@master ~]# vmstat -m
Cache                       Num  Total   Size  Pages
nf_conntrack_ffff9b1ef89e2900    765    765    320     51
nf_conntrack_ffff9b1ef89e1480    612    612    320     51
nf_conntrack_ffffffff87511640   1275  21369    320     51
ovl_inode                  3984  41232    680     48
rpc_inode_cache              51     51    640     51
xfs_dqtrx                     0      0    528     62
xfs_dquot                     0      0    488     67
xfs_ili                    3552  22752    168     48
xfs_inode                  8126  23052    960     34
xfs_efd_item                468   2652    416     39
xfs_log_ticket              352    352    184     44
bio-3                       204    204    320     51
kcopyd_job                    0      0   3312      9
dm_uevent                     0      0   2608     12
dm_rq_target_io               0      0    136     60
ip6_dst_cache               648    648    448     36
RAWv6                       572    572   1216     26
UDPLITEv6                     0      0   1216     26
UDPv6                       104    104   1216     26
tw_sock_TCPv6               128    128    256     64
TCPv6                       180    180   2112     15
Cache                       Num  Total   Size  Pages
cfq_queue                     0      0    232     70
bsg_cmd                       0      0    312     52
mqueue_inode_cache           72     72    896     36
hugetlbfs_inode_cache       106    106    608     53
configfs_dir_cache            0      0     88     46
dquot                         0      0    256     64
kioctx                       56     56    576     56
userfaultfd_ctx_cache         0      0    192     42
pid_namespace                28     28   2200     14
posix_timers_cache          264   7920    248     66
UDP-Lite                      0      0   1088     30
flow_cache                    0      0    144     56
UDP                         180    180   1088     30
tw_sock_TCP                 384    576    256     64
TCP                         336   2112   1984     16
dax_cache                    84     84    768     42
blkdev_queue                 39     39   2456     13
blkdev_ioc                  663    663    104     39
user_namespace              136    136    480     68
sock_inode_cache           3570   3570    640     51
fsnotify_mark_connector     680    680     24    170
Cache                       Num  Total   Size  Pages
net_namespace                12     12   5248      6
shmem_inode_cache          3170  13248    680     48
Acpi-State                  510    510     80     51
task_delay_info            2124   2124    112     36
taskstats                   147    147    328     49
proc_inode_cache           3599   4067    656     49
sigqueue                    102    102    160     51
bdev_cache                   78     78    832     39
kernfs_node_cache        192916 192916    120     68
mnt_cache                  3324   8652    384     42
inode_cache               27938  27995    592     55
dentry                    59658 124110    192     42
iint_cache                    0      0    128     64
avc_xperms_node            1462   1533     56     73
avc_node                   8400   8400     72     56
selinux_inode_security    21828  22542     40    102
buffer_head               47395  61815    104     39
vm_area_struct             7655   8399    216     37
mm_struct                   664    900   1600     20
files_cache                1785   1785    640     51
signal_cache                910   1008   1152     28
Cache                       Num  Total   Size  Pages
sighand_cache               623    660   2112     15
task_xstate                1516   1530   1088     30
task_struct                 790    826   4160      7
anon_vma                   4336   5049     80     51
shared_policy_node         3061   3145     48     85
numa_policy                 186    186    264     62
radix_tree_node            7501  15400    584     56
idr_layer_cache             660    660   2112     15
dma-kmalloc-8192              0      0   8192      4
dma-kmalloc-4096              0      0   4096      8
dma-kmalloc-2048              0      0   2048     16
dma-kmalloc-1024              0      0   1024     32
dma-kmalloc-512              64     64    512     64
dma-kmalloc-256               0      0    256     64
dma-kmalloc-128               0      0    128     64
dma-kmalloc-64                0      0     64     64
dma-kmalloc-32                0      0     32    128
dma-kmalloc-16                0      0     16    256
dma-kmalloc-8                 0      0      8    512
dma-kmalloc-192               0      0    192     42
dma-kmalloc-96                0      0     96     42
Cache                       Num  Total   Size  Pages
kmalloc-8192                 80     80   8192      4
kmalloc-4096               1051   1080   4096      8
kmalloc-2048              18183  19136   2048     16
kmalloc-1024               6359  21184   1024     32
kmalloc-512               25749  68800    512     64
kmalloc-256               28071  39040    256     64
kmalloc-192               30717  57120    192     42
kmalloc-128               17497  39744    128     64
kmalloc-96                45512  64722     96     42
kmalloc-64                39073 114368     64     64
kmalloc-32                 8994  88064     32    128
kmalloc-16                19200  19200     16    256
kmalloc-8                 20992  20992      8    512
kmem_cache_node            5056   5056     64     64
kmem_cache                 4992   4992    256     64
```

参考：

 - [vmstat(8) — Linux manual page](https://man7.org/linux/man-pages/man8/vmstat.8.html)
 - [Linux commands: exploring virtual memory with vmstat](https://www.redhat.com/sysadmin/linux-commands-vmstat)
 - [vmstat command in Linux with Examples](https://www.geeksforgeeks.org/vmstat-command-in-linux-with-examples/)

