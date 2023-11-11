#  Linux Command sar 网卡流量
tags: 网络, 监控



---

##  1. 简介
如何实时查看网卡流量为多少？如何查看历史网卡流量？
答：
安装sysstat包，使用sar命令查看。

```bash
yum install -y sysstat#安装sysstat包，获得sar命令

```
给 sar 增加 -n 参数就可以查看网络的统计信息，比如网络接口（DEV）、网络接口错误（EDEV）、TCP、UDP、ICMP 等等。执行下面的命令，你就可以得到网络接口统计信息：
```bash
sar -n DEV#查看网卡流量，默认10分钟更新一次
sar -n DEV 1 10#一秒显示一次，一共显示10次
sar -n DEV -f /var/log/sa/sa22#查看指定日期的流量日志

# 数字1表示每隔1秒输出一组数据
$ sar -n DEV 1
Linux 4.15.0-1035-azure (ubuntu)   01/06/19   _x86_64_  (2 CPU)

13:21:40        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
13:21:41         eth0     18.00     20.00      5.79      4.25      0.00      0.00      0.00      0.00
13:21:41      docker0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
13:21:41           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
```
这儿输出的指标比较多，我来简单解释下它们的含义。

 - rxpck/s 和 txpck/s 分别是接收和发送的 PPS，单位为包 / 秒。
 - rxkB/s 和 txkB/s 分别是接收和发送的吞吐量，单位是 KB/ 秒。
 - rxcmp/s 和 txcmp/s 分别是接收和发送的压缩数据包数，单位是包 / 秒。
 - %ifutil 是网络接口的使用率，即半双工模式下为 (rxkB/s+txkB/s)/Bandwidth，而全双工模式下为
   max(rxkB/s, txkB/s)/Bandwidth。

其中，Bandwidth 可以用 ethtool 来查询，它的单位通常是 Gb/s 或者 Mb/s，不过注意这里小写字母 b ，表示比特而不是字节。我们通常提到的千兆网卡、万兆网卡等，单位也都是比特。如下你可以看到，我的 eth0 网卡就是一个千兆网卡：

```bash
$ ethtool eth0 | grep Speed
  Speed: 1000Mb/s
```

## 2. 查看内存各个指标的变化情况
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
我们可以看到，sar 的输出结果是两个表格，第一个表格表示**内存**的使用情况，第二个表格表示 **Swap** 的使用情况。其中，各个指标名称前面的 kb 前缀，表示这些指标的单位是 KB。

 - kbcommit，表示当前系统负载需要的内存。它实际上是为了保证系统内存不溢出，对需要内存的估计值。%commit，就是这个值相对总内存的百分比。
 - kbactive，表示活跃内存，也就是最近使用过的内存，一般不会被系统回收。
 - kbinact，表示非活跃内存，也就是不常访问的内存，有可能会被系统回收。

参考：

 - [How to Use the sar Command on Linux](https://www.howtogeek.com/793513/how-to-use-the-sar-command-on-linux/)
 - [10 Useful Sar (Sysstat) Examples for UNIX / Linux Performance Monitoring](https://www.thegeekstuff.com/2011/03/sar-examples/)
 - [sar(1) - Linux man page](https://linux.die.net/man/1/sar)

