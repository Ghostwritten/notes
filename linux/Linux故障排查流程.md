## 1. 分析问题的方法论
套用5W2H方法，可以提出性能分析的几个问题

 1. What-现象是什么样的
 2. When-什么时候发生
 3. Why-为什么会发生
 4. Where-哪个地方发生的问题
 5. How much-耗费了多少资源
 6. How to do-怎么解决问题


## 2. cpu

### 2.1 说明
针对应用程序，我们通常关注的是**内核CPU调度器功能和性能**。

线程的状态分析主要是分析线程的时间用在什么地方，而线程状态的分类一般分为：

 - a. `on-CPU`：执行中，执行中的时间通常又分为用户态时间user和系统态时间sys。
 - b. `off-CPU`：等待下一轮上CPU，或者等待I/O、锁、换页等等，其状态可以细分为可执行、匿名换页、睡眠、锁、空闲等状态。

如果大量时间花在CPU上，对CPU的剖析能够迅速解释原因；如果系统时间大量处于off-cpu状态，定位问题就会费时很多。但是仍然需要清楚一些概念：

 - 处理器
 - 核
 - 硬件线程
 - CPU内存缓存
 - 时钟频率
 - 每指令周期数CPI和每周期指令数IPC
 - CPU指令
 - 使用率
 - 用户时间／内核时间
 - 调度器
 - 运行队列
 - 抢占
 - 多进程
 - 多线程
 - 字长
### 2.2 分析工具
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201009135835416.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
说明:

uptime,vmstat,mpstat,top,pidstat只能查询到cpu及负载的的使用情况。

perf可以跟着到进程内部具体函数耗时情况，并且可以指定内核函数进行统计，指哪打哪。

### 2.3 使用方式

```bash
//查看系统cpu使用情况
top
//查看所有cpu核信息
mpstat -P ALL 1
//查看cpu使用情况以及平均负载
vmstat 1
//进程cpu的统计信息
pidstat -u 1 -p pid
//跟踪进程内部函数级cpu使用情况
perf top -p pid -e cpu-clock
```

## 3. 内存
### 3.1 说明
内存是为提高效率而生，实际分析问题的时候，内存出现问题可能不只是影响性能，而是影响服务或者引起其他问题。同样对于内存有些概念需要清楚：

 - 主存
 - 虚拟内存
 - 常驻内存
 - 地址空间
 - OOM
 - 页缓存
 - 缺页
 - 换页
 - 交换空间
 - 交换
 - 用户分配器libc、glibc、libmalloc和mtmalloc
 - LINUX内核级SLUB分配器
### 3.2 分析工具
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201009140314252.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
说明：

free,vmstat,top,pidstat,pmap只能统计内存信息以及进程的内存使用情况。

valgrind可以分析内存泄漏问题。

dtrace动态跟踪。需要对内核函数有很深入的了解，通过D语言编写脚本完成跟踪。
### 3.3 使用方式
//查看系统内存使用情况
free -m
//虚拟内存统计信息
vmstat 1
//查看系统内存情况
top
//1s采集周期，获取内存的统计信息
pidstat -p pid -r 1
//查看进程的内存映像信息
pmap -d pid
//检测程序内存问题
valgrind --tool=memcheck --leak-check=full --log-file=./log.txt  ./程序名


## 4. 磁盘IO

### 4.1 说明
磁盘通常是计算机最慢的子系统，也是最容易出现性能瓶颈的地方，因为磁盘离 CPU 距离最远而且 CPU 访问磁盘要涉及到机械操作，比如转轴、寻轨等。访问硬盘和访问内存之间的速度差别是以数量级来计算的，就像1天和1分钟的差别一样。要监测 IO 性能，有必要了解一下基本原理和 Linux 是如何处理硬盘和内存之间的 IO 的。

在理解磁盘IO之前，同样我们需要理解一些概念，例如：

 - 文件系统
 - VFS
 - 文件系统缓存
 - 页缓存page cache
 - 缓冲区高速缓存buffer cache
 - 目录缓存
 - inode
 - inode缓存
 - noop调用策略
### 4.2 分析工具
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201009140758744.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
### 4.3 使用方式

```bash
//查看系统io信息
iotop
//统计io详细信息
iostat -d -x -k 1 10
//查看进程级io的信息
pidstat -d 1 -p  pid
//查看系统IO的请求，比如可以在发现系统IO异常时，可以使用该命令进行调查，就能指定到底是什么原因导致的IO异常
perf record -e block:block_rq_issue -ag^Cperf report
```

## 5. 网络
### 5.1 说明
网络的监测是所有 Linux 子系统里面最复杂的，有太多的因素在里面，比如：延迟、阻塞、冲突、丢包等，更糟的是与 Linux 主机相连的路由器、交换机、无线信号都会影响到整体网络并且很难判断是因为 Linux 网络子系统的问题还是别的设备的问题，增加了监测和判断的复杂度。现在我们使用的所有网卡都称为自适应网卡，意思是说能根据网络上的不同网络设备导致的不同网络速度和工作模式进行自动调整。

### 5.2 分析工具
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201009140944385.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
### 5.3 使用方式

```bash
//显示网络统计信息
netstat -s
//显示当前UDP连接状况
netstat -nu
//显示UDP端口号的使用情况
netstat -apu
//统计机器中网络连接各个状态个数
netstat -a | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
//显示TCP连接
ss -t -a
//显示sockets摘要信息
ss -s
//显示所有udp sockets
ss -u -a
//tcp,etcp状态
sar -n TCP,ETCP 1
//查看网络IO
sar -n DEV 1
//抓包以包为单位进行输出
tcpdump -i eth1 host 192.168.1.1 and port 80 
//抓包以流为单位显示数据内容
tcpflow -cp host 192.168.1.1
```

## 6. 系统负载

### 6.1 说明
Load 就是对计算机干活多少的度量（WikiPedia：the system Load is a measure of the amount of work that a compute system is doing）简单的说是进程队列的长度。Load Average 就是一段时间（1分钟、5分钟、15分钟）内平均Load。
### 6.2 分析工具
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201009141354406.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
### 6.3 使用方式

```bash
//查看负载情况
uptime
top
vmstat
//统计系统调用耗时情况
strace -c -p pid
//跟踪指定的系统操作例如
epoll_waitstrace -T -e epoll_wait -p pid
//查看内核日志信息
dmesg
```
## 7. 火焰图
[https://blog.csdn.net/xixihahalelehehe/article/details/108976204](https://blog.csdn.net/xixihahalelehehe/article/details/108976204)
