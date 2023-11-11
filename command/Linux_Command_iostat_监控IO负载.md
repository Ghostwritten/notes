#  Linux Command iostat 监控IO负载
tags: 监控, 分析



----
## 1. 介绍
iostat主要用于监控系统设备的IO负载情况，iostat首次运行时显示自系统启动开始的各项统计信息，之后运行iostat将显示自上次运行该命令以后的统计信息。用户可以通过指定统计的次数和时间来获得所需的统计信息。

## 2. 语法

```bash
iostat [ -c ] [ -d ] [ -h ] [ -N ] [ -k | -m ] [ -t ] [ -V ] [ -x ] [ -z ] [ device [...] | ALL ] [ -p [ device [,...] | ALL ] ] [ interval [ count ] ]
```

## 3. 参数

```bash
-C 显示CPU使用情况
-d 显示磁盘使用情况
-k 以 KB 为单位显示
-m 以 M 为单位显示
-N 显示磁盘阵列(LVM) 信息
-n 显示NFS 使用情况
-p[磁盘] 显示磁盘和分区的情况
-t 显示终端和CPU的信息
-x 显示详细信息
-V 显示版本信息
```
###  3.1 -d -k
```bash
iostat -d -k 2
```

 -d 表示，显示设备（磁盘）使用状态；
-k某些使用block为单位的列强制使用Kilobytes为单位；2表示，数据显示每隔2秒刷新一次。

```bash
iostat -d -k 1 10
Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda              39.29        21.14         1.44  441339807   29990031
sda1              0.00         0.00         0.00       1623        523
sda2              1.32         1.43         4.54   29834273   94827104
sda3              6.30         0.85        24.95   17816289  520725244
sda5              0.85         0.46         3.40    9543503   70970116
sda6              0.00         0.00         0.00        550        236
sda7              0.00         0.00         0.00        406          0
sda8              0.00         0.00         0.00        406          0
sda9              0.00         0.00         0.00        406          0
sda10            60.68        18.35        71.43  383002263 1490928140

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda             327.55      5159.18       102.04       5056        100
sda1              0.00         0.00         0.00          0          0
```
输出信息的意义

```bash
tps：该设备每秒的传输次数（Indicate the number of transfers per second that were issued to the device.）。"一次传输"意思是"一次I/O请求"。多个逻辑请求可能会被合并为"一次I/O请求"。"一次传输"请求的大小是未知的。

kB_read/s：每秒从设备（drive expressed）读取的数据量；
kB_wrtn/s：每秒向设备（drive expressed）写入的数据量；
kB_read：读取的总数据量；
kB_wrtn：写入的总数量数据量；这些单位都为Kilobytes。
```

上面的例子中，我们可以看到磁盘sda以及它的各个分区的统计数据，当时统计的磁盘总TPS是39.29，下面是各个分区的TPS。（因为是瞬间值，所以总TPS并不严格等于各个分区TPS的总和）

 

指定监控的设备名称为sda，该命令的输出结果和上面命令完全相同。

 

```bash
iostat -d sda 2
```

默认监控所有的硬盘设备，现在指定只监控sda。 

### 3.2  -x


iostat还有一个比较常用的选项-x，该选项将用于显示和io相关的扩展数据。

```bash
iostat -d -x -k 1 10
Device:    rrqm/s wrqm/s   r/s   w/s  rsec/s  wsec/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await  svctm  %util
sda          1.56  28.31  7.80 31.49   42.51    2.92    21.26     1.46     1.16     0.03    0.79   2.62  10.28
Device:    rrqm/s wrqm/s   r/s   w/s  rsec/s  wsec/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await  svctm  %util
sda          2.00  20.00 381.00  7.00 12320.00  216.00  6160.00   108.00    32.31     1.75    4.50   2.17  84.20
```

 

输出信息的含义


```bash
rrqm/s：每秒这个设备相关的读取请求有多少被Merge了（当系统调用需要读取数据的时候，VFS将请求发到各个FS，如果FS发现不同的读取请求读取的是相同Block的数据，FS会将这个请求合并Merge）；wrqm/s：每秒这个设备相关的写入请求有多少被Merge了。

rsec/s：每秒读取的扇区数；
wsec/：每秒写入的扇区数。
rKB/s：The number of read requests that were issued to the device per second；
wKB/s：The number of write requests that were issued to the device per second；
avgrq-sz 平均请求扇区的大小
avgqu-sz 是平均请求队列的长度。毫无疑问，队列长度越短越好。    

await：  每一个IO请求的处理的平均时间（单位是微秒毫秒）。这里可以理解为IO的响应时间，一般地系统IO响应时间应该低于5ms，如果大于10ms就比较大了。 这个时间包括了队列时间和服务时间，也就是说，一般情况下，await大于svctm，它们的差值越小，则说明队列时间越短，反之差值越大，队列时间越长，说明系统出了问题。
         
svctm    表示平均每次设备I/O操作的服务时间（以毫秒为单位）。如果svctm的值与await很接近，表示几乎没有I/O等待，磁盘性能很好，如果await的值远高于svctm的值，则表示I/O队列等待太长，         系统上运行的应用程序将变慢。

%util： 在统计时间内所有处理IO时间，除以总共统计时间。例如，如果统计间隔1秒，该设备有0.8秒在处理IO，而0.2秒闲置，那么该设备的%util = 0.8/1 = 80%，所以该参数暗示了设备的繁忙程度。一般地，如果该参数是100%表示设备已经接近满负荷运行了（当然如果是多磁盘，即使%util是100%，因为磁盘的并发能力，所以磁盘使用未必就到了瓶颈）。
```

### 3.3 -c 
iostat还可以用来获取cpu部分状态值：

```bash
iostat -c 1 10
avg-cpu: %user %nice %sys %iowait %idle
1.98 0.00 0.35 11.45 86.22
avg-cpu: %user %nice %sys %iowait %idle
1.62 0.00 0.25 34.46 63.67
```

## 4. 示例

```bash
iostat -d -k 1 10         #查看TPS和吞吐量信息(磁盘读写速度单位为KB)
iostat -d -m 2            #查看TPS和吞吐量信息(磁盘读写速度单位为MB)
iostat -d -x -k 1 10      #查看设备使用率（%util）、响应时间（await） iostat -c 1 10 #查看cpu状态
```

```bash
iostat -d -k 1 |grep sda10
Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda10            60.72        18.95        71.53  395637647 1493241908
sda10           299.02      4266.67       129.41       4352        132
sda10           483.84      4589.90      4117.17       4544       4076
sda10           218.00      3360.00       100.00       3360        100
sda10           546.00      8784.00       124.00       8784        124
sda10           827.00     13232.00       136.00      13232        136
```


上面看到，磁盘每秒传输次数平均约400；每秒磁盘读取约5MB，写入约1MB。

 

```bash
iostat -d -x -k 1
Device:    rrqm/s wrqm/s   r/s   w/s  rsec/s  wsec/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await  svctm  %util
sda          1.56  28.31  7.84 31.50   43.65    3.16    21.82     1.58     1.19     0.03    0.80   2.61  10.29
sda          1.98  24.75 419.80  6.93 13465.35  253.47  6732.67   126.73    32.15     2.00    4.70   2.00  85.25
sda          3.06  41.84 444.90 54.08 14204.08 2048.98  7102.04  1024.49    32.57     2.10    4.21   1.85  92.24
```

可以看到磁盘的平均响应时间<5ms，磁盘使用率>80。磁盘响应正常，但是已经很繁忙了。


##  5. 总结

 - 如果 %util 接近 100%，说明产生的I/O请求太多，I/O系统已经满负荷，该磁盘可能存在瓶颈。
 - idle小于70% IO压力就较大了，一般读取速度有较多的wait。
 - 同时可以结合vmstat 查看查看b参数(等待资源的进程数)和wa参数(IO等待所占用的CPU时间的百分比，高过30%时IO压力高)。

 另外 await 的参数也要多和 svctm 来参考。差的过高就一定有 IO 的问题。
 `avgqu-sz` 也是个做 IO 调优时需要注意的地方，这个就是直接每次操作的数据的大小，如果次数多，但数据拿的小的话，其实 IO 也会很小。如果数据拿的大，才IO 的数据会高。也可以通过 avgqu-sz × ( r/s or w/s ) = rsec/s or wsec/s。也就是讲，读定速度是这个来决定的。
svctm 一般要小于 await (因为同时等待的请求的等待时间被重复计算了)，svctm 的大小一般和磁盘性能有关，CPU/内存的负荷也会对其有影响，请求过多也会间接导致 svctm 的增加。await 的大小一般取决于服务时间(svctm) 以及 I/O 队列的长度和 I/O 请求的发出模式。如果 svctm 比较接近 await，说明 I/O 几乎没有等待时间；如果 await 远大于 svctm，说明 I/O 队列太长，应用得到的响应时间变慢，如果响应时间超过了用户可以容许的范围，这时可以考虑更换更快的磁盘，调整内核 elevator 算法，优化应用，或者升级 CPU。

参考：

 - [sparkdev linux iostat 命令](https://www.cnblogs.com/sparkdev/p/10427049.html)
 -  [iostat 监视I/O子系统](https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/iostat.html)
 - [iostat(1) — Linux manual page](https://man7.org/linux/man-pages/man1/iostat.1.html)

