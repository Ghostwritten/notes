

--------------
##  1. 背景
Linux内存管理模式，页式管理适合于大块内存的情形，而对于内核对象级别的较小内存情形下，不足以占用1个页。
在linux内核中会有许多小对象，这些对象构造销毁十分频繁，比如i-node，dentry。这么这些对象如果每次构建的时候就向内存要一个页，而其实际大小可能只有几个字节，这样就非常浪费，为了解决这个问题就引入了一种新的机制来处理在同一页框中如何分配小存储器区。这就是我们要讨论的slab层。在讲述slab前，我想先铺垫一下有关内存页的概念，我们都知道在linux中内存都是以页为单位来进行管理的(通常为4KB)，当内核需要内存就调用如：kmem_getpages这样的接口(底层调用__alloc_pages())。那么内核是如何管理页的分配的，这里linux使用了伙伴算法。slab也是向内核申请一个个页，然后再对这些页框做管理来达到分配小存储区的目的的。

## 2. 简介
最早于1994年在Sun系统中被提出(The Slab Allocator: An Object-Caching Kernel Memory Allocator, Jeff Bonwick (Sun Microsystems))，**Slab**是一种内存分配器，通过将内存划分不同大小的空间分配给对象使用来进行缓存管理，应用于内核对象的缓存。

Slab的两个主要作用：

 - Slab对小对象进行分配，不用为每个小对象分配一个页，节省了空间。
 - 内核中一些小对象创建析构很频繁，Slab对这些小对象做缓存，可以重复利用一些相同的对象，减少内存分配次数。

## 3. 查询
###  3.1 /proc/meminfo的Slab和SReclaimable项
其中Slab=SReclaimable+SUnreclaim，SReclaimable表示可回收使用的内存。

```bash
root@test1:~# cat /proc/meminfo
MemTotal:        2017412 kB
MemFree:          665008 kB
MemAvailable:    1671048 kB
Buffers:          646592 kB
Cached:           456124 kB
SwapCached:         9788 kB
Active:           315296 kB
Inactive:         837060 kB
Active(anon):      28036 kB
Inactive(anon):    22524 kB
Active(file):     287260 kB
Inactive(file):   814536 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:      10485752 kB
SwapFree:       10431480 kB
Dirty:                 0 kB
Writeback:             0 kB
AnonPages:         48168 kB
Mapped:            37248 kB
Shmem:               928 kB
Slab:             122988 kB
SReclaimable:      71000 kB
SUnreclaim:        51988 kB
KernelStack:        5116 kB
PageTables:         4228 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:    11494456 kB
Committed_AS:     580256 kB
VmallocTotal:   34359738367 kB
VmallocUsed:           0 kB
VmallocChunk:          0 kB
HardwareCorrupted:     0 kB
AnonHugePages:         0 kB
ShmemHugePages:        0 kB
ShmemPmdMapped:        0 kB
CmaTotal:              0 kB
CmaFree:               0 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
DirectMap4k:      323456 kB
DirectMap2M:     1773568 kB
DirectMap1G:           0 kB
```
### 3.2 命令slabtop查看slab占用情况


```bash
# slabtop
 Active / Total Objects (% used)    : 131433 / 135261 (97.2%)
 Active / Total Slabs (% used)      : 5390 / 5390 (100.0%)
 Active / Total Caches (% used)     : 63 / 87 (72.4%)
 Active / Total Size (% used)       : 28769.26K / 30102.88K (95.6%)
 Minimum / Average / Maximum Object : 0.01K / 0.22K / 8.00K

  OBJS ACTIVE  USE OBJ SIZE  SLABS OBJ/SLAB CACHE SIZE NAME                   
 23616  23616 100%    0.21K   1312   18  5248K vm_area_struct
 22080  21832  98%    0.06K    345   64  1380K kmalloc-64
 12831  12080  94%    0.19K    611   21  2444K dentry
 10044  10044 100%    0.11K    279   36  1116K sysfs_dir_cache
  8109   7568  93%    0.08K    159   51   636K selinux_inode_security
  7936   7936 100%    0.06K    124   64   496K anon_vma
  7722   7722 100%    0.10K    198   39   792K buffer_head
  6110   5759  94%    0.58K    470   13  3760K inode_cache
  4732   4694  99%    0.57K    338   14  2704K radix_tree_node
  3072   3072 100%    0.01K  6  512        24K kmalloc-8
  3072   3056  99%    0.96K    384        8  3072K ext4_inode_cache
  2848   2574  90%    0.25K    178   16   712K kmalloc-256
  2380   2380 100%    0.05K     28   85   112K shared_policy_node
  2048   2048 100%    0.02K  8  256        32K kmalloc-16
  2040   1530  75%    0.04K     20  102        80K jbd2_journal_handle
  2016   1615  80%    0.19K     96   21   384K kmalloc-192
  1776   1776 100%    0.64K    148   12  1184K proc_inode_cache
  1288   1288 100%    0.07K     23   56        92K Acpi-ParseExt
  1152   1152 100%    0.03K  9  128        36K kmalloc-32
  1152    832  72%    0.06K     18   64        72K ext4_io_end
  1140   1140 100%    0.66K     95   12   760K shmem_inode_cache
  1072   1053  98%    1.00K    134        8  1072K kmalloc-1024
   960    960 100%    0.12K     30   32   120K kmalloc-128
   672    672 100%    0.09K     16   42        64K kmalloc-96
   432    394  91%    0.50K     54        8   216K kmalloc-512
   408    408 100%    0.04K  4  102        16K Acpi-Namespace
   408    324  79%    0.62K     34   12   272K sock_inode_cache
   352    243  69%    2.00K     44        8   704K kmalloc-2048
   340    340 100%    0.12K     10   34        40K fsnotify_event
   256    256 100%    0.03K  2  128         8K jbd2_revoke_record_s
   256    256 100%    0.02K  1  256         4K jbd2_revoke_table_s
   224    224 100%    0.25K     14   16        56K tw_sock_TCP
   208    208 100%    0.94K     26        8   208K RAW
   190    190 100%    0.81K     10   19   160K task_xstate
   187    154  82%    2.84K     17   11   544K task_struct
   180    180 100%    0.38K     18   10        72K blkdev_requests
   180    180 100%    0.11K  5   36        20K jbd2_journal_head
   180    180 100%    0.13K  6   30        24K ext4_allocation_context
   154    139  90%    1.12K     11   14   176K signal_cache
   150    150 100%    2.06K     10   15   320K idr_layer_cache
   150    143  95%    2.06K     10   15   320K sighand_cache
   128    128 100%    0.06K  2   64         8K kmem_cache_node
   117    117 100%    0.10K  3   39        12K blkdev_ioc
   112    102  91%    4.00K     14        8   448K kmalloc-4096
   110    101  91%    1.56K     11   10   176K mm_struct
   108    108 100%    0.62K  9   12        72K files_cache
    96     96 100%    0.25K  6   16        24K kmem_cache
    96     62  64%    1.88K     12        8   192K TCP
    73     73 100%    0.05K  1   73         4K ip_fib_trie
    64     64 100%    0.25K  4   16        16K dquot
    46     46 100%    0.09K  1   46         4K ext4_xattr
    40     20  50%    8.00K     10        4   320K kmalloc-8192
    25     25 100%    0.16K  1   25         4K sigqueue
    19     19 100%    0.81K  1   19        16K bdev_cache
    16     16 100%    1.88K  2        8        32K blkdev_queue
    15     15 100%    0.26K  1   15         4K numa_policy
```

### 3.3 cache查看
free 输出的 Cache，是页缓存和可回收 Slab 缓存的和，你可以从 `/proc/meminfo` ，直接得到它们的大小：

```bash
$ cat /proc/meminfo | grep -E "SReclaimable|Cached" 
Cached:           748316 kB 
SwapCached:            0 kB 
SReclaimable:     179508 kB 
```
###  3.4 系统缓存回收机制的设置项
系统默认内存回收配置在/proc/sys/vm/drop_caches中，除非明确知晓改动对系统全局影响，不建议对此进行修改。

0：不做任何处理，由系统自己管理 
1：清空pagecache 
2：清空dentries和inodes 
3：清空pagecache、dentries和inodes

###  3.5 /proc/slabinfo文件信息
在Slab中，可分配内存块称为对象，下图中`kmalloc-8`表示每个对象占用8Bit大小的普通Slab，同理`kmalloc-16`中每个对象占用16
B，依次类推，找出Slab中占用量较大的对象是哪些？

 - 每种对象占用总内存量 = num_objs*objsize

列出几种对象的内存空间占有量：

```bash
名称	对象数量	每个对象大小(B)	该类型所有对象总量(B)
ext4_inode_cache	6168	984	6069312
ext4_inode_cache	6168	984	6069312
```
###  3.6 统计Slab占用超过100M的对象

```bash
cat /proc/slabinfo |awk '{if($3*$4/1024/1024 > 100){print $1,$3*$4/1024/1024} }'
```


##  slabtop
slabtop命令 以实时的方式显示内核“slab”缓冲区的细节信息。

选项

 - --delay=n, -d n：每n秒更新一次显示的信息，默认是每3秒；
 - --sort=S, -s S：指定排序标准进行排序（排序标准，参照下面或者man手册）；
 - --once, -o：显示一次后退出；
 - --version, -V：显示版本；
 - --help：显示帮助信息。

排序标准：

```bash
a: sort by number of active objects
b: sort by objects per slab
c: sort by cache size
l: sort by number of slabs
v：sort by number of active slabs
n: sort by name
o: sort by number of objects
p: sort by pages per slab
s: sort by object size
u: sort by cache utilization
```
内核的模块在分配资源的时候，为了提高效率和资源的利用率，都是透过slab来分配的。通过slab的信息，再配合源码能粗粗了解系统的运行情况，比如说什么资源有没有不正常的多，或者什么资源有没有泄漏。linux系统透过/proc/slabinfo来向用户暴露slab的使用情况。

Linux 所使用的 slab 分配器的基础是 Jeff Bonwick 为 SunOS 操作系统首次引入的一种算法。Jeff 的分配器是围绕对象缓存进行的。在内核中，会为有限的对象集（例如文件描述符和其他常见结构）分配大量内存。Jeff 发现对内核中普通对象进行初始化所需的时间超过了对其进行分配和释放所需的时间。因此他的结论是不应该将内存释放回一个全局的内存池，而是将内存保持为针对特定目而初始化的状态。Linux slab 分配器使用了这种思想和其他一些思想来构建一个在空间和时间上都具有高效性的内存分配器。

保存着监视系统中所有活动的 slab 缓存的信息的文件为/proc/slabinfo。

```bash
$ slabtop

 Active / Total Objects (% used)    : 897519 / 1245930 (72.0%)
 Active / Total Slabs (% used)      : 38605 / 38605 (100.0%)
 Active / Total Caches (% used)     : 94 / 145 (64.8%)
 Active / Total Size (% used)       : 129558.22K / 153432.58K (84.4%)
 Minimum / Average / Maximum Object : 0.01K / 0.12K / 128.00K

  OBJS ACTIVE  USE OBJ SIZE  SLABS OBJ/SLAB CACHE SIZE NAME                   
440136 171471  38%    0.05K   6113       72     24452K buffer_head
190086 148576  78%    0.05K   2437       78      9748K selinux_inode_security
151840 146366  96%    0.48K  18980        8     75920K ext3_inode_cache
144333 144143  99%    0.02K    711      203      2844K avtab_node
130529 128488  98%    0.13K   4501       29     18004K dentry_cache
 99214  99071  99%    0.03K    878      113      3512K size-32
 43834  28475  64%    0.27K   3131       14     12524K radix_tree_node
 17818   9450  53%    0.06K    302       59      1208K size-64
  4602   4562  99%    0.05K     59       78       236K sysfs_dir_cache
  3220   2855  88%    0.08K     70       46       280K vm_area_struct
  2460   2114  85%    0.12K     82       30       328K size-128
  1564   1461  93%    0.04K     17       92        68K Acpi-Operand
  1540   1540 100%    0.33K    140       11       560K inode_cache
  1524    466  30%    0.01K      6      254        24K anon_vma
  1440    515  35%    0.05K     20       72        80K avc_node
  1440   1154  80%    0.19K     72       20       288K filp
  1170   1023  87%    0.05K     15       78        60K ext3_xattr
   845    724  85%    0.02K      5      169        20K Acpi-Namespace
   638    315  49%    0.35K     58       11       232K proc_inode_cache
   450    434  96%    0.25K     30       15       120K size-256
   424    386  91%    0.50K     53        8       212K size-512
   312    107  34%    0.05K      4       78        16K delayacct_cache
   306    284  92%    0.43K     34        9       136K shmem_inode_cache
   303    108  35%    0.04K      3      101        12K pid
   300    261  87%    0.19K     15       20        60K skbuff_head_cache
   300    300 100%    0.12K     10       30        40K bio
   260    260 100%   32.00K    260        1      8320K size-32768
   254      6   2%    0.01K      1      254         4K revoke_table
   236     55  23%    0.06K      4       59        16K fs_cache
   216    203  93%    1.00K     54        4       216K size-1024
   214    214 100%    2.00K    107        2       428K size-2048
   203     83  40%    0.02K      1      203         4K biovec-1
```

