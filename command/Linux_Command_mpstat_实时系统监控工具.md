#  linux Command mpstat 实时系统监控工具
tags: 分析, 监控



---
## 1. 简介
mpstat是 Multiprocessor Statistics的缩写，是实时系统监控工具。其报告与CPU的一些统计信息，这些信息存放在/proc/stat文件中。在多CPU系统里，其不但能查看所有CPU的平均状况信息，而且能够查看特定CPU的信息。
## 2. 语法
mpstat的语法如下

```bash
mpstat [-P {cpu|ALL}] [internal [count]]
1
```

其中，各参数含义如下：

参数	含义

```bash
-P {cpu l ALL}	表示监控哪个CPU， cpu在[0,cpu个数-1]中取值
internal	相邻的两次采样的间隔时间
count	采样的次数，count只能和delay一起使用
```

## 3. 参数
当没有参数时，mpstat则显示系统启动以后所有信息的平均值。有interval时，第一行的信息自系统启动以来的平均信息。从第二行开始，输出为前一个interval时间段的平均信息。

输出各参数含义：

```bash
参数	释义	从/proc/stat获得数据
CPU	处理器ID	
%usr	在internal时间段里，用户态的CPU时间（%），不包含 nice值为负进程	usr/total*100
%nice	在internal时间段里，nice值为负进程的CPU时间（%）	nice/total*100
%sys	在internal时间段里，核心时间（%）	system/total*100
%iowait	在internal时间段里，硬盘IO等待时间（%）	iowait/total*100
%irq	在internal时间段里，硬中断时间（%）	irq/total*100
%soft	在internal时间段里，软中断时间（%）	softirq/total*100
%steal	显示虚拟机管理器在服务另一个虚拟处理器时虚拟CPU处在非自愿等待下花费时间的百分比	steal/total*100
%guest	显示运行虚拟处理器时CPU花费时间的百分比	guest/total*100
%gnice		gnice/total*100
%idle	在internal时间段里，CPU除去等待磁盘IO操作外的因为任何原因而空闲的时间闲置时间（%）	idle/total*100
```

CPU总的工作时间：

```bash
total_cur = user + system + nice + idle + iowait + irq + softirq

total_pre = pre_user + pre_system + pre_nice + pre_idle + pre_iowait + pre_irq + pre_softirq

user = user_cur – user_pre

total = total_cur - total_pre
```

其中_cur 表示当前值，_pre表示interval时间前的值。上表中的所有值可取到两位小数点。
## 4. 示例

```bash
[root@master ~]# mpstat
Linux 3.10.0-957.1.3.el7.x86_64 (master) 	2021年06月08日 	_x86_64_	(2 CPU)

16时50分15秒  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
16时50分15秒  all    3.75    0.05    4.91    0.03    0.00    0.48    0.00    0.00    0.00   90.79

```

```bash
[root@master ~]# mpstat -P ALL 5 2
Linux 3.10.0-957.1.3.el7.x86_64 (master) 	2021年06月08日 	_x86_64_	(2 CPU)

16时50分47秒  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
16时50分52秒  all    2.80    0.00    2.59    0.00    0.00    0.21    0.00    0.00    0.00   94.40
16时50分52秒    0    3.10    0.00    2.69    0.00    0.00    0.41    0.00    0.00    0.00   93.80
16时50分52秒    1    2.49    0.00    2.49    0.00    0.00    0.00    0.00    0.00    0.00   95.01

16时50分52秒  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
16时50分57秒  all    3.42    0.00    5.49    0.00    0.00    0.41    0.00    0.00    0.00   90.67
16时50分57秒    0    3.53    0.00    6.02    0.00    0.00    0.21    0.00    0.00    0.00   90.25
16时50分57秒    1    3.33    0.00    4.99    0.00    0.00    0.42    0.00    0.00    0.00   91.27

平均时间:  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
平均时间:  all    3.11    0.00    4.04    0.00    0.00    0.31    0.00    0.00    0.00   92.53
平均时间:    0    3.31    0.00    4.35    0.00    0.00    0.31    0.00    0.00    0.00   92.03
平均时间:    1    2.91    0.00    3.74    0.00    0.00    0.21    0.00    0.00    0.00   93.14

```

表示每5秒产生一个报告，总共产生2个。
上图表示每5秒产生了2个关于处理器的统计数据报告，一共产生2个interval 的信息，然后再给出这2个interval的平均信息。默认时，输出是按照CPU 号排序。第一个行给出了从系统引导以来的所有活跃数据。接下来每行对应一个处理器的活跃状态。




## 5. vmstat和mpstat对比 
1.vmstat和mpstat 命令的差别：mpstat 可以显示每个处理器的统计，而 vmstat 显示所有处理器的统计。因此，编写糟糕的应用程序（不使用多线程体系结构）可能会运行在一个多处理器机器上，而不使用所有处理器。从而导致一个 CPU 过载，而其他 CPU 却很空闲。通过 mpstat 可以轻松诊断这些类型的问题。

2.vmstat中所有关于CPU的总结都适合mpstat。当您看到较低的 %idle 数字时，您知道出现了 CPU 不足的问题。当您看到较高的 %iowait 数字时，您知道在当前负载下 I/O 子系统出现了某些问题。


参考：
 - [mpstat(1) — Linux manual page](https://man7.org/linux/man-pages/man1/mpstat.1.html)
 - [mpstat(1) - Linux man page](https://linux.die.net/man/1/mpstat)
 - [mpstat Command in Linux with Examples](https://www.geeksforgeeks.org/mpstat-command-in-linux-with-examples/)
 - [mpstat Command Examples in Linux](https://www.thegeekdiary.com/mpstat-command-examples-in-linux/)

