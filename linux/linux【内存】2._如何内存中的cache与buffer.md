

-------------

Buffer 是缓冲区，而 Cache 是缓存，两者都是数据在内存中的临时存储。那么，你知道这两种“临时存储”有什么区别吗？

##  1. free 数据的来源

```bash
man free


buffers
              Memory used by kernel buffers (Buffers in /proc/meminfo)

       cache  Memory used by the page cache and slabs (Cached and SReclaimable in /proc/meminfo)

       buff/cache
              Sum of buffers and cache
```
从 free 的手册中，你可以看到 buffer 和 cache 的说明。

 - `Buffers` 是内核缓冲区用到的内存，对应的是  `/proc/meminfo` 中的 Buffers 值。
 - `Cache` 是内核页缓存和 Slab 用到的内存，对应的是  `/proc/meminfo` 中的 `Cached` 与 `SReclaimable` 之和。

##  2. proc 文件系统
/proc 是 Linux 内核提供的一种特殊文件系统，是用户跟内核交互的接口。比方说，用户可以从 /proc 中查询内核的运行状态和配置选项，查询进程的运行状态、统计数据等，当然，你也可以通过 /proc  来修改内核的配置。

proc 文件系统同时也是很多性能工具的最终数据来源。比如我们刚才看到的 free ，就是通过读取`/proc/meminfo`，得到内存的使用情况。

执行 `man proc`，你就可以得到 proc 文件系统的详细文档
搜索一下（比如搜索 meminfo），以便更快定位到内存部分。

```bash
Buffers %lu
    Relatively temporary storage for raw disk blocks that shouldn't get tremendously large (20MB or so).

Cached %lu
   In-memory cache for files read from the disk (the page cache).  Doesn't include SwapCached.
...
SReclaimable %lu (since Linux 2.6.19)
    Part of Slab, that might be reclaimed, such as caches.
    
SUnreclaim %lu (since Linux 2.6.19)
    Part of Slab, that cannot be reclaimed on memory pressure.
```

通过这个文档，我们可以看到：

 - **Buffers** 是对原始磁盘块的临时存储，也就是用来缓存磁盘的数据，通常不会特别大（20MB
   左右）。这样，内核就可以把分散的写集中起来，统一优化磁盘的写入，比如可以把多次小的写合并成单次大的写等等。
 - **Cached**是从磁盘读取文件的页缓存，也就是用来缓存从文件读取的数据。这样，下次访问这些文件数据时，就可以直接从内存中快速获取，而不需要再次访问缓慢的磁盘。
 - **SReclaimable**是 Slab 的一部分。Slab 包括两部分，其中的可回收部分，用 SReclaimable 记录；而不可回收部分，用 SUnreclaim 记录。

第一个问题，Buffer 的文档没有提到这是磁盘读数据还是写数据的缓存，而在很多网络搜索的结果中都会提到 Buffer 只是对将要写入磁盘数据的缓存。那反过来说，它会不会也缓存从磁盘中读取的数据呢？

第二个问题，文档中提到，Cache 是对从文件读取数据的缓存，那么它是不是也会缓存写文件的数据呢？
##  3. 案例

你的准备
Ubuntu 18.04
机器配置：2 CPU，8GB 内存。
预先安装 sysstat 包，如 apt install sysstat

安装 sysstat  ，是因为我们要用到 vmstat  ，来观察 Buffer 和 Cache 的变化情况。虽然从 /proc/meminfo 里也可以读到相同的结果，但毕竟还是  vmstat  的结果更加直观。

另外，这几个案例使用了 dd 来模拟磁盘和文件的 I/O，所以我们也需要观测 I/O 的变化情况

第一个终端中，运行下面的命令来**清理系统缓存**：

```bash
# 清理文件页、目录项、Inodes等各种缓存
$ echo 3 > /proc/sys/vm/drop_caches
```
这里的 `/proc/sys/vm/drop_caches` ，就是通过 proc 文件系统修改内核行为的一个示例，写入 3 表示清理文件页、目录项、Inodes 等各种缓存。

###  3.1 场景 1：磁盘和文件写案例
首先，在第一个终端，运行下面这个 vmstat 命令：

```bash
# 每隔1秒输出1组数据
$ vmstat 1
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
0  0      0 7743608   1112  92168    0    0     0     0   52  152  0  1 100  0  0
 0  0      0 7743608   1112  92168    0    0     0     0   36   92  0  0 100  0  0
```
输出界面里， 内存部分的 buff 和 cache ，以及 io 部分的 bi 和 bo 就是我们要关注的重点。

 - buff 和 cache 就是我们前面看到的 Buffers 和 Cache，单位是 KB。
 - bi 和 bo 则分别表示块设备读取和写入的大小，单位为块 / 秒。因为 Linux 中块的大小是 1KB，所以这个单位也就等价于 KB/s。
第二个终端执行 `dd` 命令，通过读取随机设备，生成一个 `500MB` 大小的文件：

```c
$ dd if=/dev/urandom of=/tmp/file bs=1M count=500
```
然后再回到第一个终端，观察 Buffer 和 Cache 的变化情况：
```bash
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
0  0      0 7499460   1344 230484    0    0     0     0   29  145  0  0 100  0  0
 1  0      0 7338088   1752 390512    0    0   488     0   39  558  0 47 53  0  0
 1  0      0 7158872   1752 568800    0    0     0     4   30  376  1 50 49  0  0
 1  0      0 6980308   1752 747860    0    0     0     0   24  360  0 50 50  0  0
 0  0      0 6977448   1752 752072    0    0     0     0   29  138  0  0 100  0  0
 0  0      0 6977440   1760 752080    0    0     0   152   42  212  0  1 99  1  0
...
 0  1      0 6977216   1768 752104    0    0     4 122880   33  234  0  1 51 49  0
 0  1      0 6977440   1768 752108    0    0     0 10240   38  196  0  0 50 50  0
```
通过观察 vmstat 的输出，我们发现，在 dd 命令运行时， Cache 在不停地增长，而 Buffer 基本保持不变。再进一步观察 I/O 的情况，你会看到，

 - 在 Cache 刚开始增长时，块设备 I/O 很少，bi 只出现了一次 488 KB/s，bo 则只有一次
   4KB。而过一段时间后，才会出现大量的块设备写，比如 bo 变成了 122880。
 - 当 dd 命令结束后，Cache 不再增长，但块设备写还会持续一段时间，并且，多次 I/O 写的结果加起来，才是 dd 要写的 500M的数据。


把这个结果，跟我们刚刚了解到的 Cache 的定义做个对比，你可能会有点晕乎。为什么前面文档上说 Cache 是文件读的页缓存，怎么现在写文件也有它的份？这个疑问，我们暂且先记下来，接着再来看另一个磁盘写的案例。两个案例结束后，我们再统一进行分析。

不过，对于接下来的案例，我必须强调一点：下面的命令对环境要求很高，需要你的系统配置多块磁盘，并且磁盘分区 `/dev/sdb1` 还要处于未使用状态。如果你只有一块磁盘，千万不要尝试，否则将会对你的磁盘分区造成损坏。


```bash
# 首先清理缓存
$ echo 3 > /proc/sys/vm/drop_caches
# 然后运行dd命令向磁盘分区/dev/sdb1写入2G数据
$ dd if=/dev/urandom of=/dev/sdb1 bs=1M count=2048
```
然后，再回到终端一，观察内存和 I/O 的变化情况：

```bash
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
1  0      0 7584780 153592  97436    0    0   684     0   31  423  1 48 50  2  0
 1  0      0 7418580 315384 101668    0    0     0     0   32  144  0 50 50  0  0
 1  0      0 7253664 475844 106208    0    0     0     0   20  137  0 50 50  0  0
 1  0      0 7093352 631800 110520    0    0     0     0   23  223  0 50 50  0  0
 1  1      0 6930056 790520 114980    0    0     0 12804   23  168  0 50 42  9  0
 1  0      0 6757204 949240 119396    0    0     0 183804   24  191  0 53 26 21  0
 1  1      0 6591516 1107960 123840    0    0     0 77316   22  232  0 52 16 33  0
```
从这里你会看到，虽然同是写数据，写磁盘跟写文件的现象还是不同的。写磁盘时（也就是 bo 大于  0 时），Buffer 和 Cache 都在增长，但显然 Buffer 的增长快得多。这说明，写磁盘用到了大量的 Buffer，这跟我们在文档中查到的定义是一样的。

对比两个案例，我们发现，写文件时会用到 Cache 缓存数据，而写磁盘则会用到 Buffer 来缓存数据。所以，回到刚刚的问题，虽然文档上只提到，Cache 是文件读的缓存，但实际上，Cache 也会缓存写文件时的数据。

##  3.2 场景 2：磁盘和文件读案例

了解了磁盘和文件写的情况，我们再反过来想，磁盘和文件读的时候，又是怎样的呢？我们回到第二个终端，运行下面的命令。清理缓存后，从文件 `/tmp/file` 中，读取数据写入空设备：


```bash
# 首先清理缓存
$ echo 3 > /proc/sys/vm/drop_caches
# 运行dd命令读取文件数据
$ dd if=/tmp/file of=/dev/null
```
然后，再回到终端一，观察内存和 I/O 的变化情况：

```bash
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  1      0 7724164   2380 110844    0    0 16576     0   62  360  2  2 76 21  0
 0  1      0 7691544   2380 143472    0    0 32640     0   46  439  1  3 50 46  0
 0  1      0 7658736   2380 176204    0    0 32640     0   54  407  1  4 50 46  0
 0  1      0 7626052   2380 208908    0    0 32640    40   44  422  2  2 50 46  0
```
观察 vmstat 的输出，你会发现读取文件时（也就是 bi 大于 0 时），Buffer 保持不变，而 Cache 则在不停增长。这跟我们查到的定义“Cache 是对文件读的页缓存”是一致的。那么，磁盘读又是什么情况呢？我们再运行第二个案例来看看。

那么，磁盘读又是什么情况呢？我们再运行第二个案例来看看。首先，回到第二个终端，运行下面的命令。清理缓存后，从磁盘分区 /dev/sda1 中读取数据，写入空设备：


```bash
# 首先清理缓存
$ echo 3 > /proc/sys/vm/drop_caches
# 运行dd命令读取文件
$ dd if=/dev/sda1 of=/dev/null bs=1M count=1024
```
然后，再回到终端一，观察内存和 I/O 的变化情况：


```bash
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
0  0      0 7225880   2716 608184    0    0     0     0   48  159  0  0 100  0  0
 0  1      0 7199420  28644 608228    0    0 25928     0   60  252  0  1 65 35  0
 0  1      0 7167092  60900 608312    0    0 32256     0   54  269  0  1 50 49  0
 0  1      0 7134416  93572 608376    0    0 32672     0   53  253  0  0 51 49  0
 0  1      0 7101484 126320 608480    0    0 32748     0   80  414  0  1 50 49  0
```
观察 vmstat 的输出，你会发现读磁盘时（也就是 bi 大于 0 时），Buffer 和 Cache 都在增长，但显然 Buffer 的增长快很多。这说明读磁盘时，数据缓存到了 Buffer 中。

当然，我想，经过上一个场景中两个案例的分析，你自己也可以对比得出这个结论：**读文件时数据会缓存到 Cache 中，而读磁盘时数据会缓存到 Buffer 中。**

到这里你应该发现了，虽然文档提供了对 Buffer 和 Cache 的说明，但是仍不能覆盖到所有的细节。比如说，今天我们了解到的这两点：

 - Buffer 既可以用作“将要写入磁盘数据的缓存”，也可以用作“从磁盘读取数据的缓存”。
 - Cache 既可以用作“从文件读取数据的页缓存”，也可以用作“写文件的页缓存”。


简单来说，**Buffer 是对磁盘数据的缓存，而 Cache 是文件数据的缓存，它们既会用在读请求中，也会用在写请求中。**

##  4. 总结
我们一起探索了内存性能中 Buffer 和 Cache 的详细含义。Buffer 和 Cache 分别缓存磁盘和文件系统的读写数据。

 - 从写的角度来说，不仅可以优化磁盘和文件的写入，对应用程序也有好处，应用程序可以在数据真正落盘前，就返回去做其他工作。
 - 从读的角度来说，既可以加速读取那些需要频繁访问的数据，也降低了频繁 I/O 对磁盘的压力。


关于磁盘和文件的区别，本来以为大家都懂了，所以没有细讲。磁盘是一个块设备，可以划分为不同的分区；在分区之上再创建文件系统，挂载到某个目录，之后才可以在这个目录中读写文件。

其实 Linux 中“一切皆文件”，而文章中提到的“文件”是普通文件，磁盘是块设备文件，这些大家可以执行 "ls -l <路径>" 查看它们的区别（输出的含义如果不懂请 man ls 查询）。

在读写普通文件时，会经过文件系统，由文件系统负责与磁盘交互；而读写磁盘或者分区时，就会跳过文件系统，也就是所谓的“裸I/O“。这两种读写方式所使用的缓存是不同的，也就是文中所讲的 Cache 和 Buffer 区别。

关于文件系统、磁盘以及 I/O 的原理，大家不要着急，后面 I/O 模块还会讲的。
