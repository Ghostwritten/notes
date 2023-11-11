#  Linux Command dstat 性能分析
tags: 分析




----------
## 1. 简介
**dstat 是一个可以取代vmstat，iostat，netstat和ifstat这些命令的多功能产品**。dstat克服了这些命令的局限并增加了一些另外的功能，增加了监控项，也变得更灵活了。dstat可以很方便监控系统运行状况并用于基准测试和排除故障。
dstat可以让你实时地看到所有系统资源，例如，你能够通过统计IDE控制器当前状态来比较磁盘利用率，或者直接通过网络带宽数值来比较磁盘的吞吐率（在相同的时间间隔内）。
dstat将以列表的形式为你提供选项信息并清晰地告诉你是在何种幅度和单位显示输出。这样更好地避免了信息混乱和误报。更重要的是，它可以让你更容易编写插件来收集你想要的数据信息，以从未有过的方式进行扩展。
Dstat的默认输出是专门为人们实时查看而设计的，不过你也可以将详细信息通过CSV输出到一个文件，并导入到Gnumeric或者Excel生成表格中

 - 结合了vmstat，iostat，ifstat，netstat以及更多的信息
 - 实时显示统计情况
 - 在分析和排障时可以通过启用监控项并排序
 - 模块化设计
 - 使用python编写的，更方便扩展现有的工作任务
 - 容易扩展和添加你的计数器（请为此做出贡献）
 - 包含的许多扩展插件充分说明了增加新的监控项目是很方便的
 - 可以分组统计块设备/网络设备，并给出总数
 - 可以显示每台设备的当前状态
 - 极准确的时间精度，即便是系统负荷较高也不会延迟显示
 - 显示准确地单位和和限制转换误差范围
 - 用不同的颜色显示不同的单位
 - 显示中间结果延时小于1秒
 - 支持输出CSV格式报表，并能导入到Gnumeric和Excel以生成图形

## 2. 安装方法
Ubuntu/Mint和Debin系统：
本地软件库中有相关安装包，你可以用下面命令安装：

```bash
# sudo apt-get install dstat
```

RHEL/Centos和Fedora系统:
你可以在romforge软件库中添加有相关安装包，参照指导，使用如下命令很简单就能进行安装：

```bash
# yum install dstat
```

ArchLinux系统：
相关软件包在社区资源库中，你可以用这个命令来安装：

```bash
# pacman -S dstat
```

## 3. 使用方法
dstat的基本用法就是输入dstat命令，输出如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201009222659633.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)


这是默认输出显示的信息：
CPU状态：CPU的使用率。这项报告更有趣的部分是显示了用户，系统和空闲部分，这

```bash
更好地分析了CPU当前的使用状况。如果你看到"wait"一栏中，CPU的状态是一个高使用率值，那说明系统存在一些其它问题。当CPU的状态处在"waits"时，那是因为它正在等待I/O设备（例如内存，磁盘或者网络）的响应而且还没有收到。
磁盘统计：磁盘的读写操作，这一栏显示磁盘的读、写总数。
网络统计：网络设备发送和接受的数据，这一栏显示的网络收、发数据总数。
分页统计：系统的分页活动。分页指的是一种内存管理技术用于查找系统场景，一个较大的分页表明系统正在使用大量的交换空间，或者说内存非常分散，大多数情况下你都希望看到page in（换入）和page out（换出）的值是0 0。
系统统计：这一项显示的是中断（int）和上下文切换（csw）。这项统计仅在有比较基线时才有意义。这一栏中较高的统计值通常表示大量的进程造成拥塞，需要对CPU进行关注。你的服务器一般情况下都会运行运行一些程序，所以这项总是显示一些数值。

```
默认情况下，dstat每秒都会刷新数据。如果想退出dstat，你可以按"CTRL-C"键。
需要注意的是报告的第一行，通常这里所有的统计都不显示数值的。
这是由于dstat会通过上一次的报告来给出一个总结，所以第一次运行时是没有平均值和总值的相关数据。
但是dstat可以通过传递2个参数运行来控制报告间隔和报告数量。例如，如果你想要dstat输出默认监控、报表输出的时间间隔为3秒钟,并且报表中输出10个结果，你可以运行如下命令：

```bash
dstat 3 10
```

在dstat命令中有很多参数可选，你可以通过man dstat命令查看，大多数常用的参数有这些：

```bash
-c：显示CPU系统占用，用户占用，空闲，等待，中断，软件中断等信息。
-C：当有多个CPU时候，此参数可按需分别显示cpu状态，例：-C 0,1 是显示cpu0和cpu1的信息。
-d：显示磁盘读写数据大小。
-D hda,total：include hda and total。
-n：显示网络状态。
-N eth1,total：有多块网卡时，指定要显示的网卡。
-l：显示系统负载情况。
-m：显示内存使用情况。
-g：显示页面使用情况。
-p：显示进程状态。
-s：显示交换分区使用情况。
-S：类似D/N。
-r：I/O请求情况。
-y：系统状态。
--ipc：显示ipc消息队列，信号等信息。
--socket：用来显示tcp udp端口状态。
-a：此为默认选项，等同于-cdngy。
-v：等同于 -pmgdsc -D total。
--output 文件：此选项也比较有用，可以把状态信息以csv的格式重定向到指定的文件中，以便日后查看。例：dstat --output /root/dstat.csv & 此时让程序默默的在后台运行并把结果输出到/root/dstat.csv文件中。
-t ：将当前时间显示在第一行
–fs ：显示文件系统统计数据（包括文件总数量和inodes值）
–nocolor ：不显示颜色（有时候有用）
–tcp ：显示常用的TCP统计
–udp ：显示监听的UDP接口及其当前用量的一些动态数据


当然不止这些用法，dstat附带了一些插件很大程度地扩展了它的功能。你可以通过查看/usr/share/dstat目录来查看它们的一些使用方法，常用的有这些：
-–disk-util ：显示某一时间磁盘的忙碌状况
-–freespace ：显示当前磁盘空间使用率
-–proc-count ：显示正在运行的程序数量
-–top-bio ：指出块I/O最大的进程
-–top-cpu ：图形化显示CPU占用最大的进程
-–top-io ：显示正常I/O最大的进程
-–top-mem ：显示占用最多内存的进程
```

## 4. 示例
查看全部内存都有谁在占用：

```bash
dstat -g -l -m -s --top-mem
```

显示一些关于CPU资源损耗的数据：

```bash
dstat -c -y -l --proc-count --top-cpu
```
如想监控swap，process，sockets，filesystem并显示监控的时间：

```bash
$ dstat -tsp --socket --fs
----system---- ----swap--- ---procs--- ------sockets------ --filesystem-
  date/time   | used  free|run blk new|tot tcp udp raw frg|files  inodes
26-07 09:23:48|   0     0 |  0   0 0.0|104   8   5   0   0|  704   6488
26-07 09:23:49|   0     0 |  0   0   0|104   8   5   0   0|  704   6488
26-07 09:23:50|   0     0 |  0   0   0|104   8   5   0   0|  704   6489
26-07 09:23:51|   0     0 |  0   0   0|104   8   5   0   0|  704   6489
26-07 09:23:52|   0     0 |  0   0   0|104   8   5   0   0|  704   6489
26-07 09:23:53|   0     0 |  0   0   0|104   8   5   0   0|  704   6489
```

如何输出一个csv文件
想输出一个csv格式的文件用于以后，可以通过下面的命令：

```bash
dstat –output /tmp/sampleoutput.csv -cdn
```

参考:

 - [dstat Command in Linux With Examples](https://www.geeksforgeeks.org/dstat-command-in-linux-with-examples/)
 - [dstat(1) - Linux man page](https://linux.die.net/man/1/dstat)
 - [Dstat – A Resourceful Tool to Monitor Linux Server Performance in Real-Time](https://www.tecmint.com/dstat-monitor-linux-server-performance-process-memory-network/)
 - [Examining Linux system performance with dstat](https://www.networkworld.com/article/3291616/examining-linux-system-performance-with-dstat.html)
 - [Dstat linux monitoring tools](https://lintut.com/dstat-linux-monitoring-tools/)

