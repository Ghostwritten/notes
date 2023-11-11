#  Linux Command iperf3网络测速工具




----------
## 1. 简介
`Iperf`是一款基于TCP/IP和UDP/IP的网络性能测试工具，可以用来测量网络带宽和网络质量，提供网络延迟抖动、数据包丢失率、最大传输单元等统计信息。网络管理员可以根据这些信息了解并判断网络性能问题，从而定位网络瓶颈，解决网络故障。

Iperf 是一款基于命令行模式的网络性能测试工具，是跨平台的，提供横跨Windows、Linux、Mac的全平台支持。iperf 全程使用内存作为发送/接收缓冲区，不受磁盘性能的影响，对于机器配置要求很低。不过由于是命令行工具， iperf 不支持输出测试图形。

Iperf可以测试TCP和UDP带宽质量，具有多种参数和UDP特性，可以用来测试一些网络设备如路由器，防火墙，交换机等的性能
## 2. 安装
iperf3安装iperf3下载：`https://iperf.fr/iperf-download.php#fedora`

```bash
[root@xiexianbin-cn ~]# rpm -ivh iperf3-3.1.3-1.fc24.x86_64.rpm 
Preparing...                          ################################# [100%]
Updating / installing...
   1:iperf3-3.1.3-1.fc24              ################################# [100%]

```
##  3. 功能
（1）TCP方面

```bash
① 测量网络带宽

② 报告MSS/MTU值的大小和观测值

③ 支持TCP窗口值通过套接字缓冲

④ 当P线程或Win32线程可用时，支持多线程。客户端与服务端支持同时多重连接
```

（2）UDP方面

```bash
① 客户端可以创建指定带宽的UDP流

② 测量丢包

③ 测量延迟

④ 支持多播

⑤ 当P线程可用时，支持多线程。客户端与服务端支持同时多重连接（不支持Windows）
```

（3）其他方面

```bash
① 在适当的地方，选项中可以使用K（kilo-）和M（mega-）。例如131072字节可以用128K代替。

② 可以指定运行的总时间，甚至可以设置传输的数据总量。

③ 在报告中，为数据选用最合适的单位。

④ 服务器支持多重连接，而不是等待一个单线程测试。

⑤ 在指定时间间隔重复显示网络带宽，波动和丢包情况。

⑥ 服务器端可作为后台程序运行。

⑦ 服务器端可作为Windows 服务运行。

⑧ 使用典型数据流来测试链接层压缩对于可用带宽的影响。
```
## 4. 参数

```bash
4.1通用参数（Server端和Client端共用）

（1）-f，–farmat [k|m|g|K|M|G]：指定带宽输出单位，“[k|m|g|K|M|G]”分别表示以Kbits, Mbits, Gbits, KBytes, MBytes,GBytes显示输出结果，默认Mbits，eg：iperf3 -c 192.168.12.168 -f M

（2）-p，–port：指定服务器端监听的端口或客户端所连接的端口，默认是5001端口。

（3）-i，–interval：指定每次报告之间的时间间隔，单位为秒，eg：iperf3 -c 192.168.12.168 -i 2

（4）-F：指定文件作为数据流进行带宽测试。例如：iperf3 -c 192.168.12.168 -F web-ixdba.tar.gz
```

```bash
4.2、Server端专用参数

（1）-s,–server：iperf服务器模式，默认启动的监听端口为5201，eg：iperf -s

（2）-c,–client host：如果iperf运行在服务器模式，并且用-c参数指定一个主机，那么iperf将只接受指定主机的连接。此参数不能工作于UDP模式。

（3）-D：Unix平台下将Iperf作为后台守护进程运行。在Win32平台下，Iperf将作为服务运行。

（4）-R：卸载Iperf服务(仅用于Windows)。

（5）-o：重定向输出到指定文件(仅用于Windows)。

（6）-P,–parallel：服务器关闭之前保持的连接数。默认是0，这意味着永远接受连接。
```

4.3 Client端专用参数

（

```bash
1）-c,–client host：iperf客户端模式，host是server端地址，eg：iperf -c 222.35.11.23

（2）-u，–udp：表示采用UDP协议发送报文，不带该参数表示采用TCP协议。

（3）-b，–bandwidth [K|M|G]：指定UDP模式使用的带宽，单位bits/sec，默认值是1 Mbit/sec。

（4）-t，–time：指定数据传输的总时间，即在指定的时间内，重复发送指定长度的数据包。默认10秒。

（5）-l，–len：设置读写缓冲区的长度，单位为 Byte。TCP默认为8KB，UDP默认为1470字节。通常测试 PPS 的时候该值为16，测试BPS时该值为1400。

（6）-n，–num [K|M|G]：指定传输数据包的字节数，例如：iperf3 -c 192.168.12.168 –n 100M

（7）-P,–parallel：指定客户端与服务端之间使用的线程数。默认是1个线程。需要客户端与服务器端同时使用此参数。

（8）-w，–window：指定套接字缓冲区大小，在TCP方式下，此设置为TCP窗口的大小。在UDP方式下，此设置为接受UDP数据包的缓冲区大小，用来限制可以接收数据包的最大值

（9）-B，–bind：用来绑定一个主机地址或接口，这个参数仅用于具有多个网络接口的主机。在UDP模式下，此参数用于绑定和加入一个多播组。

（10）-M，–mss：设置TCP最大信息段的值

（11）-N，–nodelay：设置TCP无延时

（12）-V：绑定一个IPv6地址。

（13）-d，–dualtest：运行双测试模式。将使服务器端反向连接到客户端，使用-L参数中指定的端口（或默认使用客户端连接到服务器端的端口）。使用参数-r以运行交互模式。

（14）-L,–listenport：指定服务端反向连接到客户端时使用的端口。默认使用客户端连接至服务端的端口。

（15）-r，–tradeoff：往复测试模式。当客户端到服务器端的测试结束时，服务器端反向连接至客户端。当客户端连接终止时，反向连接随即开始。如果需要同时进行双向测试，请尝试-d参数。
```

4.4 其他参数

```bash
（1）-h，–help：显示命令行参考并退出。

[root]# iperf3 -h
Usage: iperf3 [-s|-c host] [options] iperf3 [-h|--help] [-v|--version]

（2）-v，–version：显示版本信息和编译信息并退出。
```
## 5. 示例
环境准备：

（1）Server端IP地址：192.168.211.40

（2）Server端IP地址：192.168.211.41 

###  5.1 测试TCP吞吐量
Server端开启iperf的服务器模式，指定TCP端口：
```bash
root@master:~# iperf3 -s -i 1 -p 5201
-----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------
```
Client端启动iperf的客户端模式，连接服务端

```bash
root@node1:~# iperf3 -c 192.168.211.40 -i 1 -t 60 -p 5201
Connecting to host 192.168.211.40, port 5201
[  4] local 192.168.211.41 port 50818 connected to 192.168.211.40 port 5201
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-1.00   sec   176 MBytes  1.47 Gbits/sec   89   1.49 MBytes       
[  4]   1.00-2.01   sec   135 MBytes  1.13 Gbits/sec    0   1.63 MBytes       
[  4]   2.01-3.00   sec   160 MBytes  1.35 Gbits/sec    0   1.73 MBytes       
[  4]   3.00-4.00   sec   172 MBytes  1.44 Gbits/sec   15   1.28 MBytes       
[  4]   4.00-5.00   sec   154 MBytes  1.29 Gbits/sec    0   1.36 MBytes       
[  4]   5.00-6.00   sec   162 MBytes  1.36 Gbits/sec    0   1.43 MBytes       
[  4]   6.00-7.01   sec   164 MBytes  1.37 Gbits/sec    0   1.49 MBytes     
```
Server端监听结果

```bash
root@master:~# iperf3 -s -i 1 -p 5201
-----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------
Accepted connection from 192.168.211.41, port 50816
[  5] local 192.168.211.40 port 5201 connected to 192.168.211.41 port 50818
[ ID] Interval           Transfer     Bandwidth
[  5]   0.00-1.00   sec   168 MBytes  1.41 Gbits/sec                  
[  5]   1.00-2.00   sec   136 MBytes  1.14 Gbits/sec                  
[  5]   2.00-3.00   sec   158 MBytes  1.33 Gbits/sec                  
[  5]   3.00-4.00   sec   174 MBytes  1.46 Gbits/sec                  
[  5]   4.00-5.01   sec   154 MBytes  1.28 Gbits/sec        
```

参数说明：
① Interval表示时间间隔。

② Transfer表示时间间隔里面转输的数据量。

③ Bandwidth是时间间隔里的传输速率。         


执行20秒，每5秒执行一次
```bash
root@node1:~# iperf3 -c 192.168.211.40 -t 20 -i 5
Connecting to host 192.168.211.40, port 5201
[  4] local 192.168.211.41 port 51392 connected to 192.168.211.40 port 5201
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-5.00   sec   700 MBytes  1.17 Gbits/sec   75   1.32 MBytes       
[  4]   5.00-10.00  sec   738 MBytes  1.24 Gbits/sec    0   1.59 MBytes       
[  4]  10.00-15.00  sec   729 MBytes  1.22 Gbits/sec   17   1.47 MBytes       
[  4]  15.00-20.00  sec   834 MBytes  1.40 Gbits/sec  249   1.25 MBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-20.00  sec  2.93 GBytes  1.26 Gbits/sec  341             sender
[  4]   0.00-20.00  sec  2.93 GBytes  1.26 Gbits/sec                  receiver

iperf Done.
```
传输数据包5G，每10秒显示一次
```bash
root@node1:~# iperf3 -c 192.168.211.40 -i 10 -n 5G
Connecting to host 192.168.211.40, port 5201
[  4] local 192.168.211.41 port 51786 connected to 192.168.211.40 port 5201
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-10.00  sec   988 MBytes   829 Mbits/sec  301   1.49 MBytes       
[  4]  10.00-20.00  sec  1.23 GBytes  1.06 Gbits/sec   18   1.65 MBytes       
[  4]  20.00-30.00  sec  1.39 GBytes  1.20 Gbits/sec    1   1.63 MBytes       
[  4]  30.00-40.00  sec   989 MBytes   829 Mbits/sec   18   1.74 MBytes       
[  4]  40.00-45.88  sec   459 MBytes   655 Mbits/sec    7   1.47 MBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-45.88  sec  5.00 GBytes   936 Mbits/sec  345             sender
[  4]   0.00-45.88  sec  5.00 GBytes   936 Mbits/sec                  receiver

iperf Done.
```

-F指定文件，传输

```bash
root@node1:~# iperf3 -c 192.168.211.40 -i 2 -F hello-world.tar -t 20
Connecting to host 192.168.211.40, port 5201
[  4] local 192.168.211.41 port 52192 connected to 192.168.211.40 port 5201
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-2.00   sec   187 MBytes   785 Mbits/sec   63   1.36 MBytes       
[  4]   2.00-3.29   sec   105 MBytes   684 Mbits/sec    0   1.53 MBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-3.29   sec   292 MBytes   746 Mbits/sec   63             sender
        Sent  292 MByte /  293 MByte (99%) of hello-world.tar
[  4]   0.00-3.29   sec   290 MBytes   741 Mbits/sec                  receiver

iperf Done.
```


“-P”参数开启了2个多线程

```bash
root@node1:~# iperf3 -c 192.168.211.40 -i 2 -P 2 -F hello-world.tar -t 20
Connecting to host 192.168.211.40, port 5201
[  4] local 192.168.211.41 port 52368 connected to 192.168.211.40 port 5201
[  7] local 192.168.211.41 port 52370 connected to 192.168.211.40 port 5201
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-2.00   sec   139 MBytes   583 Mbits/sec   41    650 KBytes       
[  7]   0.00-2.00   sec   174 MBytes   731 Mbits/sec  108   1.02 MBytes       
[SUM]   0.00-2.00   sec   314 MBytes  1.31 Gbits/sec  149             
- - - - - - - - - - - - - - - - - - - - - - - - -
[  4]   2.00-3.64   sec  81.7 MBytes   417 Mbits/sec    0    731 KBytes       
[  7]   2.00-3.64   sec   112 MBytes   574 Mbits/sec   16    666 KBytes       
[SUM]   2.00-3.64   sec   194 MBytes   991 Mbits/sec   16             
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-3.64   sec   221 MBytes   508 Mbits/sec   41             sender
        Sent  221 MByte /  293 MByte (75%) of hello-world.tar
[  4]   0.00-3.64   sec   218 MBytes   502 Mbits/sec                  receiver
[  7]   0.00-3.64   sec   287 MBytes   660 Mbits/sec  124             sender
        Sent  287 MByte /  293 MByte (97%) of hello-world.tar
[  7]   0.00-3.64   sec   285 MBytes   655 Mbits/sec                  receiver
[SUM]   0.00-3.64   sec   508 MBytes  1.17 Gbits/sec  165             sender
[SUM]   0.00-3.64   sec   503 MBytes  1.16 Gbits/sec                  receiver

iperf Done.
```

### 5.2 测试UDP吞吐量
带宽测试通常采用UDP模式，因为能测出极限带宽、时延抖动、丢包率。在进行测试时，首先以链路理论带宽作为数据发送速率进行测试，例如，从客户端到服务器之间的链路的理论带宽为100Mbps，先用-b 100M进行测试，然后根据测试结果（包括实际带宽，时延抖动和丢包率），再以实际带宽作为数据发送速率进行测试，会发现时延抖动和丢包率比第一次好很多，重复测试几次，就能得出稳定的实际带宽。


```bash
root@node1:~# iperf3 -u -c 192.168.211.40 -b 100m -t 5 -p 5201
Connecting to host 192.168.211.40, port 5201
[  4] local 192.168.211.41 port 56642 connected to 192.168.211.40 port 5201
[ ID] Interval           Transfer     Bandwidth       Total Datagrams
[  4]   0.00-1.00   sec  11.2 MBytes  94.2 Mbits/sec  1439  
[  4]   1.00-2.00   sec  12.2 MBytes   103 Mbits/sec  1564  
[  4]   2.00-3.00   sec  11.7 MBytes  97.9 Mbits/sec  1498  
[  4]   3.00-4.00   sec  11.7 MBytes  98.5 Mbits/sec  1499  
[  4]   4.00-5.00   sec  11.8 MBytes  98.9 Mbits/sec  1514  
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Jitter    Lost/Total Datagrams
[  4]   0.00-5.00   sec  58.7 MBytes  98.4 Mbits/sec  0.283 ms  260/7514 (3.5%)  
[  4] Sent 7514 datagrams

iperf Done.
```

进行上下行带宽测试（双向传输）
```bash
root@node1:~# iperf3 -u -c 192.168.211.40 -b 100M -d -t 5
send_parameters:
{
	"udp":	true,
	"omit":	0,
	"time":	5,
	"parallel":	1,
	"len":	8192,
	"bandwidth":	100000000,
	"client_version":	"3.1.3"
}
Connecting to host 192.168.211.40, port 5201
Setting fair-queue socket pacing to 12500000
[  4] local 192.168.211.41 port 35492 connected to 192.168.211.40 port 5201
[ ID] Interval           Transfer     Bandwidth       Total Datagrams
[  4]   0.00-1.00   sec  11.1 MBytes  92.6 Mbits/sec  1415  
[  4]   1.00-2.00   sec  11.7 MBytes  98.4 Mbits/sec  1501  
[  4]   2.00-3.00   sec  12.5 MBytes   105 Mbits/sec  1604  
[  4]   3.00-4.00   sec  11.9 MBytes   100 Mbits/sec  1529  
send_results
{
	"cpu_util_total":	33.363549,
	"cpu_util_user":	0.629773,
	"cpu_util_system":	32.748183,
	"sender_has_retransmits":	0,
	"streams":	[{
			"id":	1,
			"bytes":	62136320,
			"retransmits":	-1,
			"jitter":	0,
			"errors":	0,
			"packets":	7585
		}]
}
get_results
{
	"cpu_util_total":	7.860842,
	"cpu_util_user":	0.309339,
	"cpu_util_system":	7.578802,
	"sender_has_retransmits":	-1,
	"streams":	[{
			"id":	1,
			"bytes":	58638336,
			"retransmits":	-1,
			"jitter":	0.000264,
			"errors":	427,
			"packets":	7585
		}]
}
[  4]   4.00-5.00   sec  12.0 MBytes   101 Mbits/sec  1536  
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Jitter    Lost/Total Datagrams
[  4]   0.00-5.00   sec  59.3 MBytes  99.4 Mbits/sec  0.264 ms  427/7585 (5.6%)  
[  4] Sent 7585 datagrams

iperf Done.
```

更多阅读：

 - [How to Test Network Throughput Using iperf3 Tool in Linux](https://www.tecmint.com/test-network-throughput-in-linux/)
 - [iperf3 下载](https://iperf.fr/iperf-download.php)
 - [How To Test Network Throughput Using Iperf3 Tool](https://cloudcone.com/docs/article/how-to-test-network-throughput-using-iperf3-tool/)
 - [How to Use Iperf to Test Network Performance in Linux](https://www.thegeekdiary.com/how-to-use-iperf-to-test-network-performance-in-linux/)

