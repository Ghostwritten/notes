#  Linux Command pidstat 监控工具
tags: 分析，监控



---
## 1. 简介
pidstat是sysstat工具的一个命令，用于监控全部或指定进程的cpu、内存、线程、设备IO等系统资源的占用情况。pidstat首次运行时显示自系统启动开始的各项统计信息，之后运行pidstat将显示自上次运行该命令以后的统计信息。用户可以通过指定统计的次数和时间来获得所需的统计信息。
## 2. 安装
pidstat 是sysstat软件套件的一部分，sysstat包含很多监控linux系统状态的工具，它能够从大多数linux发行版的软件源中获得。
在Debian/Ubuntu系统中可以使用下面的命令来安装:

```bash
apt-get install sysstat
```

CentOS/Fedora/RHEL版本的linux中则使用下面的命令：

```bash
yum install sysstat
```

## 3. 语法
pidstat 的用法：

```bash
pidstat [ 选项 ] [ <时间间隔> ] [ <次数> ]
```

如下图：


常用的参数：

```bash
-u：默认的参数，显示各个进程的cpu使用统计
-r：显示各个进程的内存使用统计
-d：显示各个进程的IO使用情况
-p：指定进程号
-w：显示每个进程的上下文切换情况
-t：显示选择任务的线程的统计信息外的额外信息
-T { TASK | CHILD | ALL }
这个选项指定了pidstat监控的。
TASK表示报告独立的task，
CHILD关键字表示报告进程下所有线程统计信息。
ALL表示报告独立的task和task下面的所有线程。
注意：task和子线程的全局的统计信息和pidstat选项无关。这些统计信息不会对应到当前的统计间隔，这些统计信息只有在子线程kill或者完成的时候才会被收集。
-V：版本号
-h：在一行上显示了所有活动，这样其他程序可以容易解析。
-I：在SMP环境，表示任务的CPU使用率/内核数量
-l：显示命令名和所有参数
```

```bash
root@test1:~# pidstat
Linux 4.15.0-135-generic (test1) 	06/29/2021 	_x86_64_	(1 CPU)

02:17:49 AM   UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
02:17:49 AM     0         1    0.00    0.00    0.00    0.00    0.01     0  systemd
02:17:49 AM     0         2    0.00    0.00    0.00    0.00    0.00     0  kthreadd
02:17:49 AM     0         7    0.00    0.01    0.00    0.00    0.01     0  ksoftirqd/0
02:17:49 AM     0         8    0.00    0.02    0.00    0.01    0.02     0  rcu_sched
```
pidstat 中， `%wait` 表示进程等待 CPU 的时间百分比。此时进程是运行状态。
top 中 ，`iowait%` 则表示等待 I/O 的 CPU 时间百分比。此时进程处于不可中断睡眠态。
等待 CPU 的进程已经在 CPU 的就绪队列中，处于运行状态；而等待 I/O 的进程则处于不可中断状态

## 4. 示例
### 4.1 查看所有进程的 CPU 使用情况（ -u -p ALL）

```bash
[root@master ~]# pidstat -u -p ALL
Linux 3.10.0-957.1.3.el7.x86_64 (master) 	2021年06月08日 	_x86_64_	(2 CPU)

16时54分56秒   UID       PID    %usr %system  %guest    %CPU   CPU  Command
16时54分56秒     0         1    0.00    0.01    0.00    0.01     0  systemd
16时54分56秒     0         2    0.00    0.00    0.00    0.00     0  kthreadd
16时54分56秒     0         3    0.00    0.05    0.00    0.05     0  ksoftirqd/0
16时54分56秒     0         5    0.00    0.00    0.00    0.00     0  kworker/0:0H
16时54分56秒     0         7    0.00    0.01    0.00    0.01     0  migration/0

```

pidstat 和 pidstat -u -p ALL 是等效的。
pidstat 默认显示了所有进程的cpu使用率。

详细说明

```bash
PID：进程ID
%usr：进程在用户空间占用cpu的百分比
%system：进程在内核空间占用cpu的百分比
%guest：进程在虚拟机占用cpu的百分比
%CPU：进程占用cpu的百分比
CPU：处理进程的cpu编号
Command：当前进程对应的命令
```

### 4.2 cpu使用情况统计(-u)

```bash
[root@master ~]# pidstat -u
Linux 3.10.0-957.1.3.el7.x86_64 (master) 	2021年06月08日 	_x86_64_	(2 CPU)

16时55分16秒   UID       PID    %usr %system  %guest    %CPU   CPU  Command
16时55分16秒     0         1    0.00    0.01    0.00    0.01     0  systemd
16时55分16秒     0         2    0.00    0.00    0.00    0.00     0  kthreadd
16时55分16秒     0         3    0.00    0.05    0.00    0.05     0  ksoftirqd/0
16时55分16秒     0         7    0.00    0.01    0.00    0.01     0  migration/0
16时55分16秒     0         9    0.00    0.25    0.00    0.25     1  rcu_sched

```

使用-u选项，pidstat将显示各活动进程的cpu使用统计，执行”pidstat -u”与单独执行”pidstat”的效果一样。
### 4.3 内存使用情况统计(-r)
pidstat -r

使用-r选项，pidstat将显示各活动进程的内存使用统计：

```bash
PID：进程标识符
Minflt/s:任务每秒发生的次要错误，不需要从磁盘中加载页
Majflt/s:任务每秒发生的主要错误，需要从磁盘中加载页
VSZ：虚拟地址大小，虚拟内存的使用KB
RSS：常驻集合大小，非交换区五里内存使用KB
Command：task命令名
```

### 4.4 显示各个进程的IO使用情况（-d）

```bash
[root@master ~]# pidstat -d
Linux 3.10.0-957.1.3.el7.x86_64 (master) 	2021年06月08日 	_x86_64_	(2 CPU)

16时56分08秒   UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s  Command
16时56分08秒     0         1      4.79      0.06      0.02  systemd
16时56分08秒     0      3077      0.01      0.00      0.00  xfsaild/dm-0
16时56分08秒     0      3147      0.03      0.00      0.00  systemd-journal
16时56分08秒     0      3165      0.00      0.00      0.00  lvmetad
16时56分08秒     0      3180      0.37      0.00      0.00  systemd-udevd

```


报告IO统计显示以下信息：

```bash
PID：进程id
kB_rd/s：每秒从磁盘读取的KB
kB_wr/s：每秒写入磁盘KB
kB_ccwr/s：任务取消的写入磁盘的KB。当任务截断脏的pagecache的时候会发生。
COMMAND:task的命令名
```

### 4.5 显示每个进程的上下文切换情况（-w）

```bash
[root@master ~]# pidstat -w -p  9903
Linux 3.10.0-957.1.3.el7.x86_64 (master) 	2021年06月08日 	_x86_64_	(2 CPU)

16时56分56秒   UID       PID   cswch/s nvcswch/s  Command
16时56分56秒     0      9903      0.00      0.00  coredns

```


```bash
PID:进程id
Cswch/s:每秒主动任务上下文切换数量
Nvcswch/s:每秒被动任务上下文切换数量
Command:命令名
```

### 4.6 显示选择任务的线程的统计信息外的额外信息 (-t)

```bash
[root@master ~]# pidstat -t -p   9903
Linux 3.10.0-957.1.3.el7.x86_64 (master) 	2021年06月08日 	_x86_64_	(2 CPU)

16时57分19秒   UID      TGID       TID    %usr %system  %guest    %CPU   CPU  Command
16时57分19秒     0      9903         -    0.13    0.26    0.00    0.38     0  coredns
16时57分19秒     0         -      9903    0.00    0.00    0.00    0.00     0  |__coredns
16时57分19秒     0         -     10069    0.02    0.06    0.00    0.08     0  |__coredns
16时57分19秒     0         -     10070    0.02    0.03    0.00    0.05     0  |__coredns
16时57分19秒     0         -     10071    0.00    0.00    0.00    0.00     0  |__coredns
16时57分19秒     0         -     10088    0.01    0.02    0.00    0.02     1  |__coredns
16时57分19秒     0         -     10117    0.02    0.03    0.00    0.05     0  |__coredns
16时57分19秒     0         -     10123    0.00    0.00    0.00    0.00     0  |__coredns
16时57分19秒     0         -     10128    0.02    0.03    0.00    0.05     0  |__coredns
16时57分19秒     0         -     10143    0.02    0.03    0.00    0.05     1  |__coredns
16时57分19秒     0         -     13027    0.02    0.03    0.00    0.05     1  |__coredns
16时57分19秒     0         -      1555    0.01    0.02    0.00    0.03     1  |__coredns

```


```bash
TGID:主线程的表示
TID:线程id
%usr：进程在用户空间占用cpu的百分比
%system：进程在内核空间占用cpu的百分比
%guest：进程在虚拟机占用cpu的百分比
%CPU：进程占用cpu的百分比
CPU：处理进程的cpu编号
Command：当前进程对应的命令
```

### 4.7 pidstat -T

```bash
[root@master ~]# pidstat -T TASK
Linux 3.10.0-957.1.3.el7.x86_64 (master) 	2021年06月08日 	_x86_64_	(2 CPU)

16时57分44秒   UID       PID    %usr %system  %guest    %CPU   CPU  Command
16时57分44秒     0         1    0.00    0.01    0.00    0.01     1  systemd
16时57分44秒     0         2    0.00    0.00    0.00    0.00     0  kthreadd
16时57分44秒     0         3    0.00    0.05    0.00    0.05     0  ksoftirqd/0
16时57分44秒     0         7    0.00    0.01    0.00    0.01     0  migration/0


[root@master ~]# pidstat -T CHILD
Linux 3.10.0-957.1.3.el7.x86_64 (master) 	2021年06月08日 	_x86_64_	(2 CPU)

16时58分14秒   UID       PID    usr-ms system-ms  guest-ms  Command
16时58分14秒     0         1     14480     22700         0  systemd
16时58分14秒     0         2         0        20         0  kthreadd
16时58分14秒     0         3         0     17880         0  ksoftirqd/0
16时58分14秒     0         7         0      2240         0  migration/0
16时58分14秒     0         9         0     89760         0  rcu_sched
16时58分14秒     0        11         0       690         0  watchdog/0



[root@master ~]# pidstat -T ALL
Linux 3.10.0-957.1.3.el7.x86_64 (master) 	2021年06月08日 	_x86_64_	(2 CPU)

16时58分34秒   UID       PID    %usr %system  %guest    %CPU   CPU  Command
16时58分34秒     0         1    0.00    0.01    0.00    0.01     1  systemd
16时58分34秒     0         2    0.00    0.00    0.00    0.00     0  kthreadd
16时58分34秒     0         3    0.00    0.05    0.00    0.05     0  ksoftirqd/0
16时58分34秒     0         7    0.00    0.01    0.00    0.01     0  migration/0

```

TASK表示报告独立的task。
CHILD关键字表示报告进程下所有线程统计信息。
ALL表示报告独立的task和task下面的所有线程。
注意：task和子线程的全局的统计信息和pidstat选项无关。这些统计信息不会对应到当前的统计间隔，这些统计信息只有在子线程kill或者完成的时候才会被收集。

```bash
PID:进程id
Usr-ms:任务和子线程在用户级别使用的毫秒数。
System-ms:任务和子线程在系统级别使用的毫秒数。
Guest-ms:任务和子线程在虚拟机(running a virtual processor)使用的毫秒数。
Command:命令名
```


### 4.8 指定采样周期和采样次数
pidstat命令指定采样周期和采样次数，命令形式为”`pidstat [option] interval [count]`”，以下pidstat输出以2秒为采样周期，输出3次cpu使用统计信息：

```bash
[root@master ~]# pidstat 2 2
Linux 3.10.0-957.1.3.el7.x86_64 (master) 	2021年06月08日 	_x86_64_	(2 CPU)

16时59分16秒   UID       PID    %usr %system  %guest    %CPU   CPU  Command
16时59分18秒     0         9    0.00    0.50    0.00    0.50     0  rcu_sched
16时59分18秒     0        14    0.00    0.50    0.00    0.50     1  ksoftirqd/1
16时59分18秒     0      3147    0.50    0.50    0.00    1.00     0  systemd-journal
16时59分18秒     0      5173    0.50    1.00    0.00    1.49     0  kube-scheduler
16时59分18秒     0      6556    0.00    0.50    0.00    0.50     1  rsyslogd
16时59分18秒     0      6557    0.50    0.50    0.00    1.00     0  dockerd
16时59分18秒     0      7234    1.00    1.49    0.00    2.49     0  kubelet
16时59分18秒     0      9038    0.50    0.00    0.00    0.50     0  kube-proxy
16时59分18秒     0      9880    0.00    0.50    0.00    0.50     1  coredns
16时59分18秒     0      9903    0.00    0.50    0.00    0.50     0  coredns
16时59分18秒     0     18755    0.00    0.50    0.00    0.50     0  docker-containe
16时59分18秒     0     18772    0.00    0.50    0.00    0.50     0  etcd
16时59分18秒     0     19157    0.00    0.50    0.00    0.50     1  pidstat

16时59分18秒   UID       PID    %usr %system  %guest    %CPU   CPU  Command
16时59分20秒     0         9    0.00    0.50    0.00    0.50     0  rcu_sched
16时59分20秒     0      3077    0.00    0.50    0.00    0.50     0  xfsaild/dm-0
16时59分20秒     0      3147    0.00    0.50    0.00    0.50     1  systemd-journal
16时59分20秒     0      5173    0.50    0.50    0.00    1.00     0  kube-scheduler
16时59分20秒     0      6556    0.50    0.00    0.00    0.50     1  rsyslogd
16时59分20秒     0      6557    1.49    1.49    0.00    2.99     0  dockerd
16时59分20秒     0      6859    0.00    0.50    0.00    0.50     1  docker-containe
16时59分20秒     0      7234    4.48    4.48    0.00    8.96     0  kubelet
16时59分20秒     0      9038    0.00    0.50    0.00    0.50     0  kube-proxy
16时59分20秒     0      9880    0.00    1.00    0.00    1.00     1  coredns
16时59分20秒     0      9903    0.50    0.50    0.00    1.00     0  coredns
16时59分20秒     0     18772    0.50    0.50    0.00    1.00     0  etcd
16时59分20秒     0     19157    0.50    1.49    0.00    1.99     1  pidstat

平均时间:   UID       PID    %usr %system  %guest    %CPU   CPU  Command
平均时间:     0         9    0.00    0.50    0.00    0.50     -  rcu_sched
平均时间:     0        14    0.00    0.25    0.00    0.25     -  ksoftirqd/1
平均时间:     0      3077    0.00    0.25    0.00    0.25     -  xfsaild/dm-0
平均时间:     0      3147    0.25    0.50    0.00    0.75     -  systemd-journal
平均时间:     0      5173    0.50    0.75    0.00    1.24     -  kube-scheduler
平均时间:     0      6556    0.25    0.25    0.00    0.50     -  rsyslogd
平均时间:     0      6557    1.00    1.00    0.00    1.99     -  dockerd
平均时间:     0      6859    0.00    0.25    0.00    0.25     -  docker-containe
平均时间:     0      7234    2.74    2.99    0.00    5.72     -  kubelet
平均时间:     0      9038    0.25    0.25    0.00    0.50     -  kube-proxy
平均时间:     0      9880    0.00    0.75    0.00    0.75     -  coredns
平均时间:     0      9903    0.25    0.50    0.00    0.75     -  coredns
平均时间:     0     18755    0.00    0.25    0.00    0.25     -  docker-containe
平均时间:     0     18772    0.25    0.50    0.00    0.75     -  etcd
平均时间:     0     19157    0.25    1.00    0.00    1.24     -  pidstat

```

pidstat常用命令
使用pidstat进行问题定位时，以下命令常被用到：

```bash
pidstat -u 1

pidstat -r 1

pidstat -d 1
```

以上命令以1秒为信息采集周期，分别获取cpu、内存和磁盘IO的统计信息。

参考：

 - [pidstat(1) — Linux manual page](https://man7.org/linux/man-pages/man1/pidstat.1.html)
 - [pidstat Command Examples in Linux](https://www.thegeekdiary.com/pidstat-command-examples-in-linux/)

