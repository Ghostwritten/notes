#  python 模块 psutil 系统监控



----
## 1 psutil介绍
[psutil](https://pypi.org/project/psutil/) 是一个跨平台库（http://code.google.com/p/psutil/） ，能够轻松实现获取系统运行的进程和系统利用率（包括CPU、内存、磁盘、网络等）信息。它主要应用于系统监控，分析和限制系统资源及进程的管理。它实现了同等命令行工具提供的功能，如ps、top、lsof、netstat、ifconfig、who、df、kill、free、nice、ionice、iostat、iotop、uptime、pidof、tty、taskset、pmap等。目前支持32位和64位的Linux、Windows、OS X、FreeBSD和Sun Solaris等操作系统
## 2 psutil安装

```bash
wget https://pypi.python.org/packages/source/p/psutil/psutil-2.0.0.tar.gz
tar -xzvf psutil-2.0.0.tar.gz
cd psutil-2.0.0
python setup.py install
#or
pip install psutil
```
## 3 psutil实战
### 3.1 获取cpu信息

```c
[root@localhost psutil]# cat psutil1.py 
#!/usr/bin/python
#coding:utf-8
import psutil
#获取CPU的完整信息
print psutil.cpu_times()
#获取CPU的USER的时间比
print psutil.cpu_times().user
#CPU逻辑个数
print psutil.cpu_count()
#CPU物理个数
print psutil.cpu_count(logical=False)
#获取cpu的使用率
print psutil.cpu_percent()
#获取进程为1的cpu的使用率
print psutil.cpu_percent(1)
[root@localhost psutil]# python psutil1.py 
scputimes(user=21.13, nice=0.0, system=35.92, idle=6167.98, iowait=0.96, irq=0.0, softirq=1.15, steal=0.0, guest=0.0, guest_nice=0.0)
21.13
2
1
0.0
0.0
```

### 3.2 获取内存信息

```c
[root@localhost psutil]# cat psutil2.py
#!/usr/bin/python
#coding:utf-8
import psutil
mem = psutil.virtual_memory()
#获取内存完整信息
print mem
#获取总内存信息
print mem.total
#以MB为单位获取总内存信息
print(str(mem.total/1024/1024) + 'MB')
#获取已用信息
print mem.used
#获取空闲信息
print mem.free

#获取交换分区信息
print psutil.swap_memory()
[root@localhost psutil]# python psutil2.py 
svmem(total=1907802112, available=1568444416, percent=17.8, used=182579200, free=1478758400, active=142086144, inactive=158580736, buffers=2170880, cached=244293632, shared=10051584, slab=54722560)
1907802112
1819MB
182579200
1478758400
sswap(total=2147479552, used=0, free=2147479552, percent=0.0, sin=0, sout=0)
```
### 3.3 获取磁盘信息

```c
[root@localhost psutil]# cat psutil3.py 
#!/usr/bin/python
#coding:utf-8
import psutil

#获取系统总磁盘分区信息
print '获取系统总磁盘分区信息:' + str(psutil.disk_partitions())
#获取根分区磁盘大小
print '根分区磁盘大小:' + str(psutil.disk_usage('/'))
#获取总磁盘的I/O的读写信息
print '*'*10
print '总磁盘的I/O的读写信息:' +  str(psutil.disk_io_counters())
#获取每个磁盘的I/O的读写信息
print '每个磁盘的I/O的读写信息:' + str(psutil.disk_io_counters(perdisk=True))
[root@localhost psutil]# python psutil3.py 
获取系统总磁盘分区信息:[sdiskpart(device='/dev/mapper/centos-root', mountpoint='/', fstype='xfs', opts='rw,relatime,attr2,inode64,noquota'), sdiskpart(device='/dev/sda1', mountpoint='/boot', fstype='xfs', opts='rw,relatime,attr2,inode64,noquota'), sdiskpart(device='/dev/mapper/centos-root', mountpoint='/var/lib/docker/containers', fstype='xfs', opts='rw,relatime,attr2,inode64,noquota'), sdiskpart(device='/dev/mapper/centos-root', mountpoint='/var/lib/docker/overlay2', fstype='xfs', opts='rw,relatime,attr2,inode64,noquota')]
根分区磁盘大小:sdiskusage(total=50432839680, used=8449806336, free=41983033344, percent=16.8)
**********
总磁盘的I/O的读写信息:sdiskio(read_count=11597, write_count=3662, read_bytes=456081920, write_bytes=52591104, read_time=6938, write_time=5274, read_merged_count=0, write_merged_count=308, busy_time=6464)
每个磁盘的I/O的读写信息:{'sr0': sdiskio(read_count=0, write_count=0, read_bytes=0, write_bytes=0, read_time=0, write_time=0, read_merged_count=0, write_merged_count=0, busy_time=0), 'sda2': sdiskio(read_count=5732, write_count=1672, read_bytes=214466560, write_bytes=25236480, read_time=3401, write_time=2310, read_merged_count=0, write_merged_count=308, busy_time=3203), 'sda': sdiskio(read_count=5934, write_count=1682, read_bytes=243925504, write_bytes=27354624, read_time=3547, write_time=2321, read_merged_count=0, write_merged_count=308, busy_time=3280), 'sda1': sdiskio(read_count=170, write_count=10, read_bytes=27615744, write_bytes=2118144, read_time=134, write_time=11, read_merged_count=0, write_merged_count=0, busy_time=72), 'dm-0': sdiskio(read_count=5571, write_count=1980, read_bytes=209502208, write_bytes=25236480, read_time=3367, write_time=2953, read_merged_count=0, write_merged_count=0, busy_time=3169), 'dm-1': sdiskio(read_count=92, write_count=0, read_bytes=2654208, write_bytes=0, read_time=24, write_time=0, read_merged_count=0, write_merged_count=0, busy_time=15)}
```

### 3.4 获取网络信息

```c
[root@localhost psutil]# cat psutil4.py 
#!/usr/bin/python
#coding:utf-8
import psutil
#获取网络总IO信息
print psutil.net_io_counters()  
print '*' * 20
#输出网络每个接口信息
print psutil.net_io_counters(pernic=True) 
[root@localhost psutil]# python psutil4.py 
snetio(bytes_sent=1048099, bytes_recv=1294529, packets_sent=5664, packets_recv=8237, errin=0, errout=0, dropin=0, dropout=0)
********************
{'lo': snetio(bytes_sent=0, bytes_recv=0, packets_sent=0, packets_recv=0, errin=0, errout=0, dropin=0, dropout=0), 'docker0': snetio(bytes_sent=0, bytes_recv=0, packets_sent=0, packets_recv=0, errin=0, errout=0, dropin=0, dropout=0), 'eth0': snetio(bytes_sent=1048099, bytes_recv=1294529, packets_sent=5664, packets_recv=8237, errin=0, errout=0, dropin=0, dropout=0)}
```
### 3.5 进程信息

```c
[root@localhost psutil]# cat psutil5.py 
#!/usr/bin/python
#coding:utf-8
import psutil

#查看全部进程
print psutil.pids()

p = psutil.Process(2) 
print p.name()   #进程名
print p.exe()    #进程的bin路径
print p.cwd()    #进程的工作目录绝对路径
print p.status()   #进程状态
print p.create_time()  #进程创建时间
print p.uids()    #进程uid信息
print p.gids()    #进程的gid信息
print p.cpu_times()   #进程的cpu时间信息,包括user,system两个cpu信息
print p.cpu_affinity()  #get进程cpu亲和度,如果要设置cpu亲和度,将cpu号作为参考就好
print p.memory_percent()  #进程内存利用率
print p.memory_info()    #进程内存rss,vms信息
print p.io_counters()    #进程的IO信息,包括读写IO数字及参数
print p.connections()   #返回进程列表
print p.num_threads()  #进程开启的线程数

#听过psutil的Popen方法启动应用程序，可以跟踪程序的相关信息
from subprocess import PIPE
p = psutil.Popen(["/usr/bin/python", "-c", "print('hello')"],stdout=PIPE)
print p.name()
print p.username()
[root@localhost psutil]# python psutil5.py 
[1, 2, 4, 6, 7, 8, 9, 10, 11, 12, 13, 14, 16, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 31, 36, 37, 38, 39, 47, 48, 49, 50, 51, 53, 66, 98, 100, 277, 279, 280, 281, 307, 313, 318, 319, 322, 323, 325, 329, 330, 404, 405, 415, 416, 429, 430, 431, 432, 433, 434, 435, 436, 437, 438, 439, 440, 511, 532, 534, 585, 590, 593, 594, 595, 602, 603, 611, 616, 617, 619, 621, 687, 691, 692, 713, 714, 715, 716, 718, 722, 727, 730, 733, 737, 750, 752, 753, 762, 767, 780, 1116, 1118, 1120, 1121, 1136, 1137, 1145, 1146, 1174, 1362, 1363, 1364, 1623, 1625, 6488, 7050, 7687, 7745, 8205, 8240]
kthreadd

/
sleeping
1585555747.03
puids(real=0, effective=0, saved=0)
pgids(real=0, effective=0, saved=0)
pcputimes(user=0.0, system=0.0, children_user=0.0, children_system=0.0, iowait=0.0)
[0, 1]
0.0
pmem(rss=0, vms=0, shared=0, text=0, lib=0, data=0, dirty=0)
pio(read_count=0, write_count=0, read_bytes=0, write_bytes=0, read_chars=0, write_chars=0)
[]
1
python
root
close failed in file object destructor:
sys.excepthook is missing
lost sys.stderr
```
### 3.6 其他信息

```c
[root@localhost psutil]# cat psutil6.py 
#!/usr/bin/python
#coding:utf-8
import psutil
import datetime
#获取当前系统用户登录信息
print psutil.users()
#获取开机时间
print psutil.boot_time() #以linux时间格式返回
print datetime.datetime.fromtimestamp(psutil.boot_time ()).strftime("%Y-%m-%d %H: %M: %S") #转换成自然时间格式
[root@localhost psutil]# python psutil6.py
[suser(name='root', terminal='pts/0', host='192.168.211.1', started=1585555840.0, pid=1625)]
1585555747.0
2020-03-30 16: 09: 07
```


参考：

 - [Psutil module in Python](https://www.geeksforgeeks.org/psutil-module-in-python/)
 - [What is psutil.cpu_percent in Python?](https://www.educative.io/answers/what-is-psutilcpupercent-in-python)

