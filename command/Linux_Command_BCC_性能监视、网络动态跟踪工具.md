#  Linux Command BCC 性能监视、网络动态跟踪工具
tags: 监控, 分析



-------------

##  1. ubuntu安装

```bash
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 4052245BD4284CDD
echo "deb https://repo.iovisor.org/apt/xenial xenial main" | sudo tee /etc/apt/sources.list.d/iovisor.list
sudo apt-get update
sudo apt-get install -y bcc-tools libbcc-examples linux-headers-$(uname -r)
```

操作完这些步骤，bcc 提供的所有工具就都安装到 `/usr/share/bcc/tools` 这个目录中了。不过这里提醒你，bcc 软件包默认不会把这些工具配置到系统的 PATH 路径中，所以你得自己手动配置：

```bash
$ export PATH=$PATH:/usr/share/bcc/tools
```

## 2. centos安装

第一步，升级内核。你可以运行下面的命令来操作：
```bash
# 升级系统
yum update -y

# 安装ELRepo
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh https://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm

# 安装新内核
yum remove -y kernel-headers kernel-tools kernel-tools-libs
yum --enablerepo="elrepo-kernel" install -y kernel-ml kernel-ml-devel kernel-ml-headers kernel-ml-tools kernel-ml-tools-libs kernel-ml-tools-libs-devel

# 更新Grub后重启
grub2-mkconfig -o /boot/grub2/grub.cfg
grub2-set-default 0
reboot

# 重启后确认内核版本已升级为4.20.0-1.el7.elrepo.x86_64
uname -r
```
第二步，安装 bcc-tools：

```bash
# 安装bcc-tools
yum install -y bcc-tools

# 配置PATH路径
export PATH=$PATH:/usr/share/bcc/tools

# 验证安装成功
cachestat 
```

## 3. cachestat 缓存统计

```bash
$ cachestat 1 3
   TOTAL   MISSES     HITS  DIRTIES   BUFFERS_MB  CACHED_MB
       2        0        2        1           17        279
       2        0        2        1           17        279
       2        0        2        1           17        279 
```
你可以看到，cachestat 的输出其实是一个表格。每行代表一组数据，而每一列代表不同的缓存统计指标。这些指标从左到右依次表示：

```bash
TOTAL ，表示总的 I/O 次数；
MISSES ，表示缓存未命中的次数；
HITS ，表示缓存命中的次数；
DIRTIES， 表示新增到缓存中的脏页数；
BUFFERS_MB 表示 Buffers 的大小，以 MB 为单位；
CACHED_MB 表示 Cache 的大小，以 MB 为单位。
```

##  4. cachetop 缓存命中

```bash
$ cachetop
11:58:50 Buffers MB: 258 / Cached MB: 347 / Sort: HITS / Order: ascending
PID      UID      CMD              HITS     MISSES   DIRTIES  READ_HIT%  WRITE_HIT%
   13029 root     python                  1        0        0     100.0%       0.0%
```
它的输出跟 top 类似，默认按照缓存的命中次数（HITS）排序，展示了每个进程的缓存命中情况。具体到每一个指标，这里的 HITS、MISSES 和 DIRTIES ，跟 cachestat 里的含义一样，分别代表间隔时间内的缓存命中次数、未命中次数以及新增到缓存中的脏页数。而 **READ_HIT 和 WRITE_HIT ，分别表示读和写的缓存命中率。**


##  5. filtop 跟踪内核中文件的读写
 [filetop](https://github.com/iovisor/bcc/blob/master/tools/filetop.py)。它是 bcc 软件包的一部分，基于 Linux 内核的 eBPF（extended Berkeley Packet Filters）机制，主要跟踪内核中文件的读写情况，并输出线程 ID（TID）、读写大小、读写类型以及文件名称。
bcc 提供的所有工具，全部安装到了 /usr/share/bcc/tools 这个目录中。接下来我们就用这个工具，观察一下文件的读写情况。首先，在终端一中运行下面的命令：

```bash
# 切换到工具目录 
$ cd /usr/share/bcc/tools 

# -C 选项表示输出新内容时不清空屏幕 
$ ./filetop -C 

TID    COMM             READS  WRITES R_Kb    W_Kb    T FILE 
514    python           0      1      0       2832    R 669.txt 
514    python           0      1      0       2490    R 667.txt 
514    python           0      1      0       2685    R 671.txt 
514    python           0      1      0       2392    R 670.txt 
514    python           0      1      0       2050    R 672.txt 

...

TID    COMM             READS  WRITES R_Kb    W_Kb    T FILE 
514    python           2      0      5957    0       R 651.txt 
514    python           2      0      5371    0       R 112.txt 
514    python           2      0      4785    0       R 861.txt 
514    python           2      0      4736    0       R 213.txt 
514    python           2      0      4443    0       R 45.txt 
```
你会看到，filetop 输出了 8 列内容，分别是**线程 ID、线程命令行、读写次数、读写的大小（单位 KB）、文件类型以及读写的文件名称**。

这些内容里，你可能会看到很多动态链接库，不过这不是我们的重点，暂且忽略即可。我们的重点，是一个 python 应用，所以要特别关注 python 相关的内容。多观察一会儿，你就会发现，每隔一段时间，线程号为 514 的 python 应用就会先写入大量的 txt 文件，再大量地读。

线程号为 514 的线程，属于哪个进程呢？我们可以用 ps 命令查看。先在终端一中，按下 Ctrl+C ，停止 filetop ；然后，运行下面的 ps 命令。这个输出的第二列内容，就是我们想知道的进程号：


```bash
$ ps -efT | grep 514
root     12280  514 14626 33 14:47 pts/0    00:00:05 /usr/local/bin/python /app.py 
```

## 6. opensnoop 跟踪内核中 open 系统调用

opensnoop 。它同属于 bcc 软件包，可以动态跟踪内核中的 open 系统调用。这样，我们就可以找出这些 txt 文件的路径。接下来，在终端一中，运行下面的 opensnoop 命令：


```bash
$ opensnoop 
12280  python              6   0 /tmp/9046db9e-fe25-11e8-b13f-0242ac110002/650.txt 
12280  python              6   0 /tmp/9046db9e-fe25-11e8-b13f-0242ac110002/651.txt 
12280  python              6   0 /tmp/9046db9e-fe25-11e8-b13f-0242ac110002/652.txt 
```
这次，通过 opensnoop 的输出，你可以看到，这些 txt 路径位于 /tmp 目录下。你还能看到，它打开的文件数量，按照数字编号，从 0.txt 依次增大到 999.txt，这可远多于前面用 filetop 看到的数量。


参考：

 - [BCC 用于Linux性能监视，网络和更多的动态跟踪工具](https://www.cnblogs.com/sky-heaven/p/12801079.html)

