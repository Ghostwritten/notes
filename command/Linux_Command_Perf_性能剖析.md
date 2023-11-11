# Linux Command Perf 性能剖析

@[TOC]

---

![在这里插入图片描述](https://img-blog.csdnimg.cn/67d4c860574e4379ae407a1a5948c5dd.gif#pic_center)

## 1. 简介
从2.6.31内核开始，Linux内核自带了一个性能分析工具perf，能够进行函数级与指令级的热点查找。通过它，应用程序可以利用 PMU，tracepoint 和内核中的特殊计数器来进行性能统计。它不但可以分析指定应用程序的性能问题 (per thread)，也可以用来分析内核的性能问题，当然也可以同时分析应用代码和内核，从而全面理解应用程序中的性能瓶颈。

Perf是内置于Linux内核源码树中的性能剖析(profiling)工具。
它基于**事件**采样原理，以**性能事件**为基础，支持针对处理器相关性能指标与操作系统相关性能指标的性能剖析。

常用于性能瓶颈的查找与热点代码的定位。

 

**CPU周期**(cpu-cycles)是默认的性能事件，所谓的CPU周期是指CPU所能识别的最小时间单元，通常为亿分之几秒，是CPU执行最简单的指令时所需要的时间，例如读取寄存器中的内容，也叫做`clock tick`。

Perf是一个包含22种子工具的工具集，以下是最常用的5种：

 - perf-list
 - perf-stat
 - perf-top
 - perf-record
 - perf-report
 - perf-trace

## 2. 方法
### 2.1 perf-list
 Perf-list用来查看perf所支持的性能事件，有软件的也有硬件的。
List all symbolic event types.

```bash
perf list [hw | sw | cache | tracepoint | event_glob]
```

 

#### 2.1.1 性能事件的分布

 - hw：Hardware event，9个
 - sw：Software event，9个
 - cache：Hardware cache event，26个
 - tracepoint：Tracepoint event，775个

 

sw实际上是内核的计数器，与硬件无关。

hw和cache是CPU架构相关的，依赖于具体硬件。

tracepoint是基于内核的ftrace，主线2.6.3x以上的内核版本才支持。

 

#### 2.1.2 指定性能事件(以它的属性)

```bash
-e <event> : u // userspace

-e <event> : k // kernel

-e <event> : h // hypervisor

-e <event> : G // guest counting (in KVM guests)

-e <event> : H // host counting (not in KVM guests)
```

 

#### 2.1.3 实例

显示内核和模块中，消耗最多CPU周期的函数：

```bash
$ perf top -e cycles:k
```

显示分配高速缓存最多的函数：

```bash
$ perf top -e kmem:kmem_cache_alloc
```

 
### 2.2 perf-top
对于一个指定的性能事件(默认是CPU周期)，显示消耗最多的函数或指令。

System profiling tool.

Generates and displays a performance counter profile in real time.

```bash
perf top [-e <EVENT> | --event=EVENT] [<options>]
```

perf top主要用于实时分析各个函数在某个性能事件上的热度，能够快速的定位热点函数，包括应用程序函数、

模块函数与内核函数，甚至能够定位到热点指令。默认的性能事件为cpu cycles。
#### 2.2.2 输出格式
perf top
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201029111526560.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)

 - 第一列：符号引发的性能事件的比例，默认指占用的cpu周期比例。
 - 第二列：符号所在的DSO(Dynamic Shared Object)，可以是应用程序、内核、动态链接库、模块。
 - 第三列：DSO的类型。[.]表示此符号属于用户态的ELF文件，包括可执行文件与动态链接库，[k]表述此符号属于内核或模块。
 - 第四列：符号名。有些符号不能解析为函数名，只能用地址表示。

#### 2.2.3 交互命令

```bash
h：显示帮助

UP/DOWN/PGUP/PGDN/SPACE：上下和翻页。

a：annotate current symbol，注解当前符号。能够给出汇编语言的注解，给出各条指令的采样率。

d：过滤掉所有不属于此DSO的符号。非常方便查看同一类别的符号。

P：将当前信息保存到perf.hist.N中。
```

 

#### 2.2.4 参数

 - -e <event>：指明要分析的性能事件。
 - -p <pid>：Profile events on existing Process ID (comma sperated list). 仅分析目标进程及其创建的线程。
 - -k <path>：Path to vmlinux. Required for annotation functionality. 带符号表的内核映像所在的路径。
 - -K：不显示属于内核或模块的符号。
 - -U：不显示属于用户态程序的符号。
 - -d <n>：界面的刷新周期，默认为2s，因为perf top默认每2s从mmap的内存区域读取一次性能数据。
 - -G：得到函数的调用关系图


```bash
perf top -G [fractal]，路径概率为相对值，加起来为100%，调用顺序为从下往上。

perf top -G graph，路径概率为绝对值，加起来为该函数的热度。
```

 

#### 2.2.5 实例

```bash
$ perf top // 默认配置

$ perf top -G // 得到调用关系图

$ perf top -e cycles // 指定性能事件

$ perf top -p 23015,32476 // 查看这两个进程的cpu cycles使用情况

$ perf top -s comm,pid,symbol // 显示调用symbol的进程名和进程号

$ perf top --comms nginx,top // 仅显示属于指定进程的符号

$ perf top --symbols kfree // 仅显示指定的符号
```
### 2.3 perf-stat
用于分析指定程序的性能概况。

 Run a command and gather performance counter statistics.

```bash
perf stat [-e <EVENT> | --event=EVENT] [-a] <command>

perf stat [-e <EVENT> | --event=EVENT] [-a] - <command> [<options>]
```

 

#### 2.3.1 输出格式

```bash
$ perf stat ls
bin   dev		   elasticsearch-7.2.0-linux-x86_64.tar.gz  elasticsearch-7.9.3_2		     etc   kibana-7.9.3-linux-x86_64	     lib    media  opt	 root  sbin  sys  usr
boot  elasticsearch-7.2.0  elasticsearch-7.9.3			    elasticsearch-7.9.3-linux-x86_64.tar.gz  home  kibana-7.9.3-linux-x86_64.tar.gz  lib64  mnt    proc  run   srv   tmp  var

 Performance counter stats for 'ls':

              1.20 msec task-clock                #    0.592 CPUs utilized          
                 0      context-switches          #    0.000 K/sec                  
                 0      cpu-migrations            #    0.000 K/sec                  
               270      page-faults               #    0.225 M/sec                  
   <not supported>      cycles                                                      
   <not supported>      instructions                                                
   <not supported>      branches                                                    
   <not supported>      branch-misses                                               

       0.002023278 seconds time elapsed

       0.000000000 seconds user
       0.001984000 seconds sys


```
输出包括ls的执行时间，以及10个性能事件的统计。

 - `task-clock`：任务真正占用的处理器时间，单位为ms。CPUs utilized = task-clock / time
   elapsed，CPU的占用率。
 - `context-switches`：上下文的切换次数。
 - `CPU-migrations`：处理器迁移次数。Linux为了维持多个处理器的负载均衡，在特定条件下会将某个任务从一个CPU迁移到另一个CPU。
 - `page-faults`：缺页异常的次数。当应用程序请求的页面尚未建立、请求的页面不在内存中，或者请求的页面虽然在内存中，但物理地址和虚拟地址的映射关系尚未建立时，都会触发一次缺页异常。另外TLB不命中，页面访问权限不匹配等情况也会触发缺页异常。
 - `cycles`：消耗的处理器周期数。如果把被ls使用的cpu cycles看成是一个处理器的，那么它的主频为2.486GHz。可以用cycles / task-clock算出。
 - stalled-cycles-frontend：略过。
 - stalled-cycles-backend：略过。
 - `instructions`：执行了多少条指令。IPC为平均每个cpu cycle执行了多少条指令。
 - `branches`：遇到的分支指令数。branch-misses是预测错误的分支指令数。

#### 2.3.2 参数

```bash
-p：stat events on existing process id (comma separated list). 仅分析目标进程及其创建的线程。

-a：system-wide collection from all CPUs. 从所有CPU上收集性能数据。

-r：repeat command and print average + stddev (max: 100). 重复执行命令求平均。

-C：Count only on the list of CPUs provided (comma separated list), 从指定CPU上收集性能数据。

-v：be more verbose (show counter open errors, etc), 显示更多性能数据。

-n：null run - don't start any counters，只显示任务的执行时间 。

-x SEP：指定输出列的分隔符。

-o file：指定输出文件，--append指定追加模式。

--pre <cmd>：执行目标程序前先执行的程序。

--post <cmd>：执行目标程序后再执行的程序。
```
#### (3) 使用例子

```bash
执行10次程序，给出标准偏差与期望的比值：

# perf stat -r 10 ls > /dev/null

显示更详细的信息：

# perf stat -v ls > /dev/null

只显示任务执行时间，不显示性能计数器：

# perf stat -n ls > /dev/null

单独给出每个CPU上的信息：

# perf stat -a -A ls > /dev/null

ls命令执行了多少次系统调用：

# perf stat -e syscalls:sys_enter ls 
```
### 2.4 perf-record
收集采样信息，并将其记录在数据文件中。

随后可以通过其它工具(perf-report)对数据文件进行分析，结果类似于perf-top的。
 

#### 2.4.1 参数

 - -e：Select the PMU event.
 - -a：System-wide collection from all CPUs.
 - -p：Record events on existing process ID (comma separated list).
 - -A：Append to the output file to do incremental profiling.
 - -f：Overwrite existing data file.
 - -o：Output file name.
 - -g：Do call-graph (stack chain/backtrace) recording.
 - -C：Collect samples only on the list of CPUs provided.


####  2.4.2 实例

记录nginx进程的性能数据：

```csharp
$ perf record -p `pgrep -d ',' nginx`
```

记录执行ls时的性能数据：

```csharp
$ perf record ls -g
```

记录执行ls时的系统调用，可以知道哪些系统调用最频繁：

```csharp
$ perf record -e syscalls:sys_enter ls
```

 

### 2.5 perf-report
读取perf record创建的数据文件，并给出热点分析结果。

#### 2.5.1 参数

```csharp
-i：Input file name. (default: perf.data)
```

 

#### 2.5.2 实例 

```csharp
$ perf report -i perf.data.2
```

 

More
除了以上5个常用工具外，还有一些适用于较特殊场景的工具， 比如内核锁、slab分配器、调度器，

也支持自定义探测点。

 

### 2.6 perf-lock
内核锁的性能分析。

Analyze lock events.

```csharp
perf lock {record | report | script | info}
```

 

需要编译选项的支持：`CONFIG_LOCKDEP、CONFIG_LOCK_STAT`。

CONFIG_LOCKDEP defines acquired and release events.

CONFIG_LOCK_STAT defines contended and acquired lock events.

 

#### 2.6.1 参数

 - -i <file>：输入文件
 - -k <value>：sorting key，默认为acquired，还可以按contended、wait_total、wait_max和wait_min来排序。

 


#### 2.6.2 实例

```bash
# perf lock record ls // 记录

# perf lock report // 报告
```
### 2.7 perf trace 
 perf-trace - strace inspired tool
#### 2.7.1 测试场景
我们将使用 4 个 IP，其中 2 个为外部可路由网段（192.168）：

 - localhost，IP 127.0.0.1
 - 一个干净的容器，IP 172.17.0.2
 - 我的手机，通过 USB 连接，IP 192.168.42.129
 - 我的手机，通过 WiFi 连接，IP 192.168.43.1
跟踪 ping 向 172.17.0.2 容器的包，这里我们只关心 net 事件，忽略系统调用信息：

```bash
$ sudo perf trace --no-syscalls --event 'net:*' ping 172.17.0.2 -c1 > /dev/null
     0.000 net:net_dev_queue:dev=docker0 skbaddr=0xffff96d481988700 len=98)
     0.008 net:net_dev_start_xmit:dev=docker0 queue_mapping=0 skbaddr=0xffff96d481988700 vlan_tagged=0 vlan_proto=0x0000 vlan_tci=0x0000 protocol=0x0800 ip_summed=0 len=98 data_len=0 network_offset=14 transport_offset_valid=1 transport_offset=34 tx_flags=0 gso_size=0 gso_segs=0 gso_type=0)
     0.014 net:net_dev_queue:dev=veth79215ff skbaddr=0xffff96d481988700 len=98)
     0.016 net:net_dev_start_xmit:dev=veth79215ff queue_mapping=0 skbaddr=0xffff96d481988700 vlan_tagged=0 vlan_proto=0x0000 vlan_tci=0x0000 protocol=0x0800 ip_summed=0 len=98 data_len=0 network_offset=14 transport_offset_valid=1 transport_offset=34 tx_flags=0 gso_size=0 gso_segs=0 gso_type=0)
     0.020 net:netif_rx:dev=eth0 skbaddr=0xffff96d481988700 len=84)
     0.022 net:net_dev_xmit:dev=veth79215ff skbaddr=0xffff96d481988700 len=98 rc=0)
     0.024 net:net_dev_xmit:dev=docker0 skbaddr=0xffff96d481988700 len=98 rc=0)
     0.027 net:netif_receive_skb:dev=eth0 skbaddr=0xffff96d481988700 len=84)
     0.044 net:net_dev_queue:dev=eth0 skbaddr=0xffff96d481988b00 len=98)
     0.046 net:net_dev_start_xmit:dev=eth0 queue_mapping=0 skbaddr=0xffff96d481988b00 vlan_tagged=0 vlan_proto=0x0000 vlan_tci=0x0000 protocol=0x0800 ip_summed=0 len=98 data_len=0 network_offset=14 transport_offset_valid=1 transport_offset=34 tx_flags=0 gso_size=0 gso_segs=0 gso_type=0)
     0.048 net:netif_rx:dev=veth79215ff skbaddr=0xffff96d481988b00 len=84)
     0.050 net:net_dev_xmit:dev=eth0 skbaddr=0xffff96d481988b00 len=98 rc=0)
     0.053 net:netif_receive_skb:dev=veth79215ff skbaddr=0xffff96d481988b00 len=84)
     0.060 net:netif_receive_skb_entry:dev=docker0 napi_id=0x3 queue_mapping=0 skbaddr=0xffff96d481988b00 vlan_tagged=0 vlan_proto=0x0000 vlan_tci=0x0000 protocol=0x0800 ip_summed=2 hash=0x00000000 l4_hash=0 len=84 data_len=0 truesize=768 mac_header_valid=1 mac_header=-14 nr_frags=0 gso_size=0 gso_type=0)
     0.061 net:netif_receive_skb:dev=docker0 skbaddr=0xffff96d481988b00 len=84)
```
只保留事件名和 skbaddr，看起来清晰很多：

```bash
net_dev_queue           dev=docker0     skbaddr=0xffff96d481988700
net_dev_start_xmit      dev=docker0     skbaddr=0xffff96d481988700
net_dev_queue           dev=veth79215ff skbaddr=0xffff96d481988700
net_dev_start_xmit      dev=veth79215ff skbaddr=0xffff96d481988700
netif_rx                dev=eth0        skbaddr=0xffff96d481988700
net_dev_xmit            dev=veth79215ff skbaddr=0xffff96d481988700
net_dev_xmit            dev=docker0     skbaddr=0xffff96d481988700
netif_receive_skb       dev=eth0        skbaddr=0xffff96d481988700

net_dev_queue           dev=eth0        skbaddr=0xffff96d481988b00
net_dev_start_xmit      dev=eth0        skbaddr=0xffff96d481988b00
netif_rx                dev=veth79215ff skbaddr=0xffff96d481988b00
net_dev_xmit            dev=eth0        skbaddr=0xffff96d481988b00
netif_receive_skb       dev=veth79215ff skbaddr=0xffff96d481988b00
netif_receive_skb_entry dev=docker0     skbaddr=0xffff96d481988b00
netif_receive_skb       dev=docker0     skbaddr=0xffff96d481988b00
```

这里面有很多信息。

首先注意，skbaddr 在中间变了（0xffff96d481988700 -> 0xffff96d481988b00） 。变的这里，就是生成了 ICMP echo reply 包，并作为应答包发送的地方。接下来的 时间，这个包的 skbaddr 保持不变，说明没有 copy。copy 非常耗时。

其次，我们可以清楚地看到 packet 在内核的传输路径：

 1. docker0 网桥
 2. veth pair 的宿主机端（veth79215ff)
 3. veth pair 的容器端（容器里的 eth0）
 4. 接下来是相反的返回路径

至此，虽然我们还没有看到网络命名空间，但已经得到了一个不错的全局视图。

上面的信息有些杂，还有很多重复。我们可以选择几个最合适的跟踪点，使得输出看起来 更清爽。要查看所有可用的网络跟踪点，执行 perf list：

```bash
$ sudo perf list 'net:*'
```

这个命令会列出 tracepoint 列表，名字类似于 net:netif_rx。冒号前面是事件类型 ，后面是事件名字。这里我选择了 4 个：

```bash
net_dev_queue

netif_receive_skb_entry

netif_rx

napi_gro_receive_entry
```

效果：

```bash
$ sudo perf trace --no-syscalls           \
    --event 'net:net_dev_queue'           \
    --event 'net:netif_receive_skb_entry' \
    --event 'net:netif_rx'                \
    --event 'net:napi_gro_receive_entry'  \
    ping 172.17.0.2 -c1 > /dev/null
       0.000 net:net_dev_queue:dev=docker0 skbaddr=0xffff8e847720a900 len=98)
       0.010 net:net_dev_queue:dev=veth7781d5c skbaddr=0xffff8e847720a900 len=98)
       0.014 net:netif_rx:dev=eth0 skbaddr=0xffff8e847720a900 len=84)
       0.034 net:net_dev_queue:dev=eth0 skbaddr=0xffff8e849cb8cd00 len=98)
       0.036 net:netif_rx:dev=veth7781d5c skbaddr=0xffff8e849cb8cd00 len=84)
       0.045 net:netif_receive_skb_entry:dev=docker0 napi_id=0x1 queue_mapping=0
```

更多阅读：

 - [云原生领域](https://mp.weixin.qq.com/s?__biz=Mzg4MTQ0Njc1Mg==&mid=2247483819&idx=1&sn=732cecd8f6e1124e961102028520afb5&chksm=cf649ba4f81312b2cd2cd66e1be8ab15dbb19d938079444ef33a519e337df2f1d66d59e44ff4&mpshare=1&scene=1&srcid=1026mSCHY4Ge5orfgcsuUbDC&sharer_sharetime=1603846657216&sharer_shareid=9e1d0f93025303e47ff2523f5ebf4078&key=a1951d0dc6cf9d5d49fb8e2650421265f61f09847e8a4f8d3c6b654db6afa7f9dbed22690b48baa286aaa8b687e8dbb4758ec8a4926d2bf22f88be2eb733b3f50893128912c044ac49c0a9cdb17d1eccbc4425cf0fb07e5a23c359b82f1bdd5d47ae18b5833e4eeea11073d1427337bc963181f3983d9b79159a65294fff4741&ascene=1&uin=MjkwMDAzNTYzOQ==&devicetype=Windows%2010%20x64&version=6300002f&lang=zh_CN&exportkey=AWO%2b3iWAPiFG0qOuJvq7Hg0=&pass_ticket=QzdoColbZmYyfslaclvmTdZpeLF2v/d5NLgjnYbRBHfwosE5vzwqUlup7bWleK3R&wx_header=0)
 - [perf Examples](https://www.brendangregg.com/perf.html)
 - [Linux Perf commands](https://linuxhint.com/linux-perf-commands/)
 - [perf(1) — Linux manual page](https://man7.org/linux/man-pages/man1/perf.1.html)
 - [Getting started with perf](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/monitoring_and_managing_system_status_and_performance/getting-started-with-perf_monitoring-and-managing-system-status-and-performance)

