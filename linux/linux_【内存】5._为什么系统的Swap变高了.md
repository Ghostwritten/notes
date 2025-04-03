


-----
## 1. 回顾
如果在程序中直接或间接地分配了动态内存，你一定要记得释放掉它们，否则就会导致内存泄漏，严重时甚至会耗尽系统内存。不过，反过来讲，当发生了内存泄漏时，或者运行了大内存的应用程序，导致系统的内存资源紧张时，系统又会如何应对呢？在内存基础篇我们已经学过，这其实会导致两种可能结果，**内存回收和 OOM 杀死进程**。

 - 内存资源紧张导致的 OOM（Out Of Memory），相对容易理解，指的是系统杀死占用大量内存的进程，释放这些内存，再分配给其他更需要的进程
 - 系统释放掉可以回收的内存，讲过的**缓存**和**缓冲区**，就属于**可回收内存**，它们在内存管理中，通常被叫做**文件页**（File-backed Page）。


大部分**文件页**，都可以直接回收，以后有需要时，再从磁盘重新读取就可以了。而那些被应用程序修改过，并且暂时还没写入磁盘的数据（也就是**脏页**），就得先写入磁盘，然后才能进行内存释放。这些脏页，一般可以通过两种方式写入磁盘。

 - 可以在应用程序中，通过系统调用 `fsync`  ，把脏页同步到磁盘中；
 - 也可以交给系统，由内核线程 `pdflush` 负责这些脏页的刷新。


除了缓存和缓冲区，通过内存映射获取的文件映射页，也是一种常见的文件页。它也可以被释放掉，下次再访问的时候，从文件重新读取。除了文件页外，还有没有其他的内存可以回收呢？比如，应用**程序动态分配的堆内存**，也就是我们在内存管理中说到的**匿名页**（Anonymous Page），是不是也可以回收呢？

我想，你肯定会说，它们很可能还要再次被访问啊，当然不能直接回收了。非常正确，这些内存自然不能直接释放。但是，如果这些内存在分配后很少被访问，似乎也是一种资源浪费。是不是可以把它们暂时先存在磁盘里，释放内存给其他更需要的进程？

其实，这正是 Linux 的 Swap 机制。Swap 把这些不常访问的内存先写到磁盘中，然后释放这些内存，给其他更需要的进程使用。再次访问这些内存时，重新从磁盘读入内存就可以了。


##  2. Swap 原理
Swap 说白了就是把一块磁盘空间或者一个本地文件（以下讲解以磁盘为例），当成内存来使用。它包括换出和换入两个过程。

 - 所谓换出，就是把进程暂时不用的内存数据存储到磁盘中，并释放这些数据占用的内存。
 - 而换入，则是在进程再次访问这些内存的时候，把它们从磁盘读到内存中来。


Swap 其实是把系统的可用内存变大了。这样，即使服务器的内存不足，也可以运行大内存的应用程序。

我们常见的笔记本电脑的休眠和快速开机的功能，也基于 Swap 。休眠时，把系统的内存存入磁盘，这样等到再次开机时，只要从磁盘中加载内存就可以。这样就省去了很多应用程序的初始化过程，加快了开机速度。

话说回来，既然 Swap 是为了回收内存，那么 Linux 到底在什么时候需要回收内存呢？前面一直在说内存资源紧张，又该怎么来衡量内存是不是紧张呢？

一个最容易想到的场景就是，有新的大块内存分配请求，但是剩余内存不足。这个时候系统就需要回收一部分内存（比如前面提到的缓存），进而尽可能地满足新内存请求。这个过程通常被称为**直接内存回收。**

除了直接内存回收，还有一个专门的内核线程用来定期回收内存，也就是 **kswapd0**。为了衡量内存的使用情况，kswapd0 定义了三个内存阈值（watermark，也称为水位），分别是

 - 页最小阈值（pages_min）
 - 页低阈值（pages_low）
 - 页高阈值（pages_high）

剩余内存，则使用 `pages_free` 表示。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/02cd4452d45e7c8e486231118c20f48d.png)
**kswapd0** 定期扫描内存的使用情况，并根据剩余内存落在这三个阈值的空间位置，进行内存的回收操作。

 - **剩余内存小于页最小阈值**，说明进程可用内存都耗尽了，只有内核才可以分配内存。
 - **剩余内存落在页最小阈值和页低阈值中间**，说明内存压力比较大，剩余内存不多了。这时 kswapd0会执行内存回收，直到剩余内存大于高阈值为止。
 - **剩余内存落在页低阈值和页高阈值中间**，说明内存有一定压力，但还可以满足新内存请求。
 - **剩余内存大于页高阈值**，说明剩余内存比较多，没有内存压力。

我们可以看到，一旦剩余内存小于页低阈值，就会触发内存的回收。这个页低阈值，其实可以通过内核选项 `/proc/sys/vm/min_free_kbytes` 来间接设置。`min_free_kbytes` 设置了页最小阈值，而其他两个阈值，都是根据页最小阈值计算生成的，计算方法如下 ：

```bash
pages_low = pages_min*5/4
pages_high = pages_min*3/2
```

##  3. NUMA 与 Swap

很多情况下，你明明发现了 Swap 升高，可是在分析系统的内存使用时，却很可能发现，系统剩余内存还多着呢。为什么剩余内存很多的情况下，也会发生 Swap 呢？

这正是处理器的 **NUMA （Non-Uniform Memory Access）**架构导致的。


关于 NUMA，我在 CPU 模块中曾简单提到过。在 NUMA 架构下，多个处理器被划分到不同 Node 上，且每个 Node 都拥有自己的本地内存空间。

而同一个 Node 内部的内存空间，实际上又可以进一步分为不同的内存域（Zone），比如**直接内存访问区（DMA）**、**普通内存区（NORMAL）、伪内存区（MOVABLE）**等，如下图所示

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d90a1b934eea1a1a9bbb49e352788a39.png)
先不用特别关注这些内存域的具体含义，我们只要会查看阈值的配置，以及缓存、匿名页的实际使用情况就够了。既然 NUMA 架构下的每个 Node 都有自己的本地内存空间，那么，在分析内存的使用时，我们也应该针对每个 Node 单独分析。

你可以通过 numactl 命令，来查看处理器在 Node 的分布情况，以及每个 Node 的内存使用情况。比如，下面就是一个 numactl 输出的示例：


```bash
$ numactl --hardware
available: 1 nodes (0)
node 0 cpus: 0 1
node 0 size: 7977 MB
node 0 free: 4416 MB
...
```
这个界面显示，我的系统中只有一个 Node，也就是 Node 0 ，而且编号为 0 和 1 的两个 CPU， 都位于 Node 0 上。另外，Node 0 的内存大小为 7977 MB，剩余内存为 4416 MB。

了解了 NUNA 的架构和 NUMA 内存的查看方法后，你可能就要问了这跟 Swap 有什么关系呢？

实际上，前面提到的三个内存阈值（页最小阈值、页低阈值和页高阈值），都可以通过内存域在 proc 文件系统中的接口 `/proc/zoneinfo` 来查看。


```bash
$ cat /proc/zoneinfo
...
Node 0, zone   Normal
 pages free     227894
       min      14896
       low      18620
       high     22344
...
     nr_free_pages 227894
     nr_zone_inactive_anon 11082
     nr_zone_active_anon 14024
     nr_zone_inactive_file 539024
     nr_zone_active_file 923986
...
```
这个输出中有大量指标，我来解释一下比较重要的几个

 - pages 处的 min、low、high，就是上面提到的三个内存阈值，而 free 是剩余内存页数，它跟后面的
nr_free_pages 相同。
 - n`r_zone_active_anon` 和 `nr_zone_inactive_anon`，分别是活跃和非活跃的匿名页数。
 - `nr_zone_active_file` 和 `nr_zone_inactive_file`，分别是活跃和非活跃的文件页数

从这个输出结果可以发现，剩余内存远大于页高阈值，所以此时的 kswapd0 不会回收内存。当然，某个 Node 内存不足时，系统可以从其**他 Node 寻找空闲内存**，**也可以从本地内存中回收内存**。具体选哪种模式，你可以通过 `/proc/sys/vm/zone_reclaim_mode` 来调整。它支持以下几个选项：

 - 默认的 0 ，也就是刚刚提到的模式，表示既可以从其他 Node 寻找空闲内存，也可以从本地回收内存。
 - 1、2、4 都表示只回收本地内存，2 表示可以回写脏数据回收内存，4 表示可以用 Swap 方式回收内存。

##  4. swappiness

到这里，我们就可以理解内存回收的机制了。这些回收的内存既包括了文件页，又包括了匿名页。

 - 对**文件页**的回收，当然就是直接回收缓存，或者把脏页写回磁盘后再回收。
 - 而对**匿名页**的回收，其实就是通过 Swap 机制，把它们写入磁盘后再释放内存。


不过，你可能还有一个问题。既然有两种不同的内存回收机制，那么在实际回收内存时，到底该先回收哪一种呢？
其实，Linux 提供了一个  `/proc/sys/vm/swappiness` 选项，用来调整使用 Swap 的积极程度。

**swappiness 的范围是 0-100，数值越大，越积极使用 Swap，也就是更倾向于回收匿名页；数值越小，越消极使用 Swap，也就是更倾向于回收文件页。**

虽然 swappiness 的范围是 0-100，不过要注意，这并不是内存的百分比，而是调整 Swap 积极程度的权重，即使你把它设置成 0，当剩余内存 + 文件页小于页高阈值时，还是会发生 Swap。

清楚了 Swap 原理后，当遇到 Swap 使用变高时，又该怎么定位、分析呢？

##  5. 案例
准备

 - Ubuntu 18.04，同样适用于其他的 Linux 系统。
 - 机器配置：2 CPU，8GB 内存
 - 安装 sysstat 等工具，如 `apt install sysstat`

```bash
$ free
             total        used        free      shared  buff/cache   available
Mem:        8169348      331668     6715972         696     1121708     7522896
Swap:             0           0           0
```
Swap 的大小是 0，这说明我的机器没有配置 Swap。
要开启 Swap，我们首先要清楚，Linux 本身支持两种类型的 Swap，即 Swap 分区和 Swap 文件。以 Swap 文件为例，在第一个终端中运行下面的命令开启 Swap，我这里配置 Swap 文件的大小为 8GB：


```bash
# 创建Swap文件
$ fallocate -l 8G /mnt/swapfile
# 修改权限只有根用户可以访问
$ chmod 600 /mnt/swapfile
# 配置Swap文件
$ mkswap /mnt/swapfile
# 开启Swap
$ swapon /mnt/swapfile
```
然后，再执行 free 命令，确认 Swap 配置成功：

```bash
$ free
             total        used        free      shared  buff/cache   available
Mem:        8169348      331668     6715972         696     1121708     7522896
Swap:       8388604           0     8388604
```
现在，free  输出中，Swap 空间以及剩余空间都从 0 变成了 8GB，说明 Swap 已经正常开启。
接下来，我们在第一个终端中，运行下面的 dd 命令，模拟大文件的读取：

```bash
# 写入空设备，实际上只有磁盘的读请求
$ dd if=/dev/sda1 of=/dev/null bs=1G count=2048
```
接着，在第二个终端中运行 sar 命令，查看内存各个指标的变化情况。你可以多观察一会儿，查看这些指标的变化情况。

```bash
# 间隔1秒输出一组数据
# -r表示显示内存使用情况，-S表示显示Swap使用情况
$ sar -r -S 1
04:39:56    kbmemfree   kbavail kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
04:39:57      6249676   6839824   1919632     23.50    740512     67316   1691736     10.22    815156    841868         4

04:39:56    kbswpfree kbswpused  %swpused  kbswpcad   %swpcad
04:39:57      8388604         0      0.00         0      0.00

04:39:57    kbmemfree   kbavail kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
04:39:58      6184472   6807064   1984836     24.30    772768     67380   1691736     10.22    847932    874224        20

04:39:57    kbswpfree kbswpused  %swpused  kbswpcad   %swpcad
04:39:58      8388604         0      0.00         0      0.00

…


04:44:06    kbmemfree   kbavail kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
04:44:07       152780   6525716   8016528     98.13   6530440     51316   1691736     10.22    867124   6869332         0

04:44:06    kbswpfree kbswpused  %swpused  kbswpcad   %swpcad
04:44:07      8384508      4096      0.05        52      1.27
```
sar 的输出结果是两个表格，第一个表格表示**内存**的使用情况，第二个表格表示 **Swap** 的使用情况。其中，各个指标名称前面的 kb 前缀，表示这些指标的单位是 KB。去掉前缀后，你会发现，大部分指标我们都已经见过了，剩下的几个新出现的指标，我来简单介绍一下。

 - `kbcommit`，表示当前系统负载需要的内存。它实际上是为了保证系统内存不溢出，对需要内存的估计值。%commit，就是这个值相对总内存的百分比。
 - `kbactive`，表示活跃内存，也就是最近使用过的内存，一般不会被系统回收。
 - `kbinact`，表示非活跃内存，也就是不常访问的内存，有可能会被系统回收。

清楚了界面指标的含义后，我们再结合具体数值，来分析相关的现象。你可以清楚地看到，总的内存使用率（%memused）在不断增长，从开始的 23% 一直长到了 98%，并且主要内存都被**缓冲区**（kbbuffers）占用。具体来说：

 - 刚开始，剩余内存（kbmemfree）不断减少，而缓冲区（kbbuffers）则不断增大，由此可知，剩余内存不断分配给了缓冲区。
 - 一段时间后，剩余内存已经很小，而缓冲区占用了大部分内存。这时候，Swap 的使用开始逐渐增大，缓冲区和剩余内存则只在小范围内波动。


你可能困惑了，为什么缓冲区在不停增大？这又是哪些进程导致的呢？
显然，我们还得看看进程缓存的情况。在前面缓存的案例中我们学过， `cachetop` 正好能满足这一点。那我们就来 cachetop 一下。

```bash
$ cachetop 5
12:28:28 Buffers MB: 6349 / Cached MB: 87 / Sort: HITS / Order: ascending
PID      UID      CMD              HITS     MISSES   DIRTIES  READ_HIT%  WRITE_HIT%
   18280 root     python                 22        0        0     100.0%       0.0%
   18279 root     dd                  41088    41022        0      50.0%      50.0%
```
通过 cachetop 的输出，我们看到，dd 进程的读写请求只有 50% 的命中率，并且未命中的缓存页数（MISSES）为 41022（单位是页）。这说明，正是案例开始时运行的 dd，导致了缓冲区使用升高。

你可能接着会问，为什么 Swap 也跟着升高了呢？直观来说，缓冲区占了系统绝大部分内存，还属于可回收内存，内存不够用时，不应该先回收缓冲区吗？

这种情况，我们还得进一步通过 `/proc/zoneinfo` ，观察剩余内存、内存阈值以及匿名页和文件页的活跃情况。你可以在第二个终端中，按下 Ctrl+C，停止 cachetop 命令。然后运行下面的命令，观察 /proc/zoneinfo 中这几个指标的变化情况：


```bash
# -d 表示高亮变化的字段
# -A 表示仅显示Normal行以及之后的15行输出
$ watch -d grep -A 15 'Normal' /proc/zoneinfo
Node 0, zone   Normal
  pages free     21328
        min      14896
        low      18620
        high     22344
        spanned  1835008
        present  1835008
        managed  1796710
        protection: (0, 0, 0, 0, 0)
      nr_free_pages 21328
      nr_zone_inactive_anon 79776
      nr_zone_active_anon 206854
      nr_zone_inactive_file 918561
      nr_zone_active_file 496695
      nr_zone_unevictable 2251
      nr_zone_write_pending 0
```
你可以发现，剩余内存（pages_free）在一个小范围内不停地波动。当它小于页低阈值（pages_low) 时，又会突然增大到一个大于页高阈值（pages_high）的值。

再结合刚刚用 sar 看到的剩余内存和缓冲区的变化情况，我们可以推导出，剩余内存和缓冲区的波动变化，正是由于内存回收和缓存再次分配的循环往复。

 - 当剩余内存小于页低阈值时，系统会回收一些缓存和匿名内存，使剩余内存增大。其中，缓存的回收导致 sar中的缓冲区减小，而匿名内存的回收导致了 Swap 的使用增大。
 - 紧接着，由于 dd 还在继续，剩余内存又会重新分配给缓存，导致剩余内存减少，缓冲区增大。


其实还有一个有趣的现象，如果多次运行 dd 和 sar，你可能会发现，在多次的循环重复中，有时候是 Swap 用得比较多，有时候 Swap 很少，反而缓冲区的波动更大。换句话说，系统回收内存时，有时候会回收更多的文件页，有时候又回收了更多的匿名页。显然，系统回收不同类型内存的倾向，似乎不那么明显。你应该想到了上节课提到的 swappiness，正是调整不同类型内存回收的配置选项。

```bash
$ cat /proc/sys/vm/swappiness
60
```
swappiness 显示的是默认值 60，这是一个相对中和的配置，所以系统会根据实际运行情况，选择合适的回收类型，比如回收不活跃的匿名页，或者不活跃的文件页。

到这里，我们已经找出了 Swap 发生的根源。另一个问题就是，刚才的 Swap 到底影响了哪些应用程序呢？换句话说，**Swap 换出的是哪些进程的内存？**

这里我还是推荐 proc 文件系统，用来查看进程 Swap 换出的虚拟内存大小，它保存在 `/proc/pid/status` 中的 VmSwap 中（推荐你执行 `man proc` 来查询其他字段的含义）。


```bash
# 按VmSwap使用量对进程排序，输出进程名称、进程ID以及SWAP用量
$ for file in /proc/*/status ; do awk '/VmSwap|Name|^Pid/{printf $2 " " $3}END{ print ""}' $file; done | sort -k 3 -n -r | head
dockerd 2226 10728 kB
docker-containe 2251 8516 kB
snapd 936 4020 kB
networkd-dispat 911 836 kB
polkitd 1004 44 kB
```
从这里你可以看到，使用 Swap 比较多的是 dockerd 和 docker-containe 进程，所以，当 dockerd 再次访问这些换出到磁盘的内存时，也会比较慢。

这也说明了一点，虽然缓存属于可回收内存，但在**类似大文件拷贝**这类场景下，系统还是会用 Swap 机制来回收匿名内存，而不仅仅是回收占用绝大部分内存的文件页。


```bash
#关闭 Swap
$ swapoff -a
#闭 Swap 后再重新打开
$ swapoff -a && swapon -a 
```

##  6. 总结
在内存资源紧张时，Linux 会通过 Swap ，把不常访问的匿名页换出到磁盘中，下次访问的时候再从磁盘换入到内存中来，Linux 通过`直接内存回收`和`定期扫描`的方式，来释放`文件页`和`匿名页`，以便把内存分配给更需要的进程使用。

 - **文件页的回收比较容易理解，直接清空，或者把脏数据写回磁盘后再释放。**
 - **而对匿名页的回收，需要通过 Swap 换出到磁盘中，下次访问时，再从磁盘换入到内存中。**

设置 `/proc/sys/vm/min_free_kbytes`，来调整系统定期回收内存的阈值（也就是页低阈值）
设置 `/proc/sys/vm/swappiness`，来调整文件页和匿名页的回收倾向。
在 `NUMA` 架构下，每个 Node 都有自己的本地内存空间，而当本地内存不足时，默认既可以从其他 Node 寻找空闲内存，也可以从本地内存回收。你可以设置 `/proc/sys/vm/zone_reclaim_mode` ，来调整 NUMA 本地内存的回收策略。

当 Swap 变高时，你可以用 `sar`、`/proc/zoneinfo`、`/proc/pid/status` 等方法，查看系统和进程的内存使用情况，进而找出 Swap 升高的根源和受影响的进程。


反过来说，通常，降低 Swap 的使用，可以提高系统的整体性能。要怎么做呢？这里，我也总结了几种常见的降低方法。

 - 禁止 Swap，现在服务器的内存足够大，所以除非有必要，禁用 Swap 就可以了。随着云计算的普及，大部分云平台中的虚拟机都默认禁止Swap。
 - 如果实在需要用到 Swap，可以尝试降低 swappiness 的值，减少内存回收时 Swap 的使用倾向。
 - 响应延迟敏感的应用，如果它们可能在开启 Swap 的服务器中运行，你还可以用库函数 `mlock()` 或者 `mlockall()`锁定内存，阻止它们的内存换出。


## 7. 讨论
1. 关于上面有同学表示 hadoop 集群建议关 swap 提升性能。事实上不仅 hadoop，包括 ES 在内绝大部分 Java 的应用都建议关 swap，这个和 JVM 的 gc 有关，它在 gc 的时候会遍历所有用到的堆的内存，如果这部分内存是被 swap 出去了，遍历的时候就会有磁盘IO
[https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-configuration-memory.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-configuration-memory.html)
[https://dzone.com/articles/just-say-no-swapping](https://dzone.com/articles/just-say-no-swapping)

2. 为什么kubernetes要关闭swap呢？
一个是性能问题，开启swap会严重影响性能（包括内存和I/O）；另一个是管理问题，开启swap后通过cgroups设置的内存上限就会失效。

3. swappiness=0
Kernel version 3.5 and newer: disables swapiness.
Kernel version older than 3.5: avoids swapping processes out of physical memory for as long as possible.
如果linux内核是3.5及以后的，最好是设置swappiness=10，不要设置swappiness=0

4. 用`smem --sort swap`命令可以直接将进程按照swap使用量排序显示

```bash
root@test1:~# smem --sort swap
  PID User     Command                         Swap      USS      PSS      RSS 
  473 root     /lib/systemd/systemd-journa        0     6052     6484     8680 
  493 root     /sbin/lvmetad -f                   0      344      346      440 
  700 systemd-timesync /lib/systemd/systemd-timesy        0      660      926     2412 
  768 root     /usr/bin/VGAuthService             0     1920     1932     2080 
  778 root     /usr/bin/vmtoolsd                  0     1984     2421     4276 
  946 messagebus /usr/bin/dbus-daemon --syst        0     1120     1379     2596 
  956 root     /usr/sbin/cron -f                  0      328      401     1164 
  958 root     /usr/bin/lxcfs /var/lib/lxc        0     1128     1130     1224 
  972 root     /usr/lib/accountsservice/ac        0     1272     2143     4412 
  973 root     /lib/systemd/systemd-logind        0     1016     1472     3728 
  975 daemon   /usr/sbin/atd -f                   0      240      245      388 
  996 syslog   /usr/sbin/rsyslogd -n              0     1916     2032     2896 
 1002 root     /usr/lib/policykit-1/polkit        0     1164     1975     3616 
 1020 root     /sbin/agetty -o -p -- \u --        0      332      334      420 
 3418 root     sshd: root@pts/0                   0      996     1087     1532 
 3420 root     /lib/systemd/systemd --user        0     1168     1170     1264 
 3552 root     -bash                              0     3284     3300     3476 
 3795 root     sshd: root@pts/1                   0     1016     1103     1520 
 3877 root     -bash                              0     1632     1648     1824 
 4069 systemd-network /lib/systemd/systemd-networ        0     1700     1780     2732 
 4655 root     sshd: root@pts/2                   0     1020     1365     2456 
 4803 root     -bash                              0     1764     2312     3824 
 5121 root     sshd: root@pts/3                   0     3488     4269     6912 
 5248 root     -bash                              0     2260     3127     5436 
 6815 root     /usr/bin/python /usr/bin/sm        0     7680     8034     9756 
  888 systemd-resolve /lib/systemd/systemd-resolv        4     1048     1388     2940 
  488 root     /lib/systemd/systemd-udevd        32     1468     1488     1812 
 1019 root     /usr/sbin/sshd -D                344      432      762     1800 
    1 root     /sbin/init auto automatic-u      536     2120     2841     5176 
 3421 root     (sd-pam)                         568     1184     1524     1940 
 1004 root     /usr/bin/python3 /usr/share     4500     3880     4367     5084 
  985 root     /usr/bin/python3 /usr/bin/n     6812      996     1489     2212 
  997 root     /usr/bin/containerd             8968    11392    11394    11488 
 1009 root     /usr/bin/dockerd -H fd:// -    25420    17044    17056    17220 
```

