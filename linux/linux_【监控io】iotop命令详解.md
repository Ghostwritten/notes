


----

## 1. 简介
iotop 是一个类似 top 的工具，用来显示实时的磁盘活动。Linux下的IO统计工具如iostat，nmon等大多数是只能统计到per设备的读写情况。iotop 监控 Linux 内核输出的 I/O 使用信息，并且显示一个系统中进程或线程的当前 I/O 使用情况。它显示每个进程/线程读写 I/O 带宽。它同样显示当等待换入和等待 I/O 的线程/进程花费的时间的百分比。

 - `Total DISK READ` 和 `Total DISK WRITE`
   的值一方面表示了**进程和内核线程**之间的总的读写带宽，另一方面也表示内核块设备子系统的。
 - `Actual DISK READ` 和 `Actual DISK WRITE` 的值表示在内核块设备子系统和下面硬件（HDD、SSD
   等等）对应的**实际磁盘 I/O 带宽**。

## 2. 安装 iotop

对于 Fedora 系统，使用 DNF 命令 来安装 iotop。

```bash
$ sudo dnf install iotop
```

对于 Debian/Ubuntu 系统，使用 API-GET 命令 或者 APT 命令 来安装 iotop。

```bash
$ sudo apt install iotop
```

对于基于 Arch Linux 的系统，使用 Pacman Command 来安装 iotop。

```bash
$ sudo pacman -S iotop
```

对于 RHEL/CentOS 的系统，使用 YUM Command 来安装 iotop。

```bash
$ sudo yum install iotop
```

对于使用 openSUSE Leap 的系统，使用 Zypper Command 来安装 iotop。

```bash
$ sudo zypper install iotop
```

## 3. 参数

```bash
-o：只显示有io操作的进程
-b：批量显示，无交互，主要用作记录到文件。
-n NUM：显示NUM次，主要用于非交互式模式。
-d SEC：间隔SEC秒显示一次。
-p PID：监控的进程pid。
-u USER：监控的进程用户。
```

## 4. 快捷键

```bash
左右箭头：改变排序方式，默认是按IO排序。
r：改变排序顺序。
o：只显示有IO输出的进程。
p：进程/线程的显示方式的切换。
a：显示累积使用量。
q：退出。
```

## 5. 使用

```bash
# iotop
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210713141339678.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
如果你想检查那个进程实际在做 I/O，那么运行 iotop 命令加上 -o 或者 --only 参数。

```bash
# iotop --only
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210713141425853.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
细节：

 - `IO`：它显示每个进程的 I/O 利用率，包含磁盘和交换。
 - `SWAPIN`： 它只显示每个进程的交换使用率


参考链接：

 - [https://man7.org/linux/man-pages/man8/iotop.8.html](https://man7.org/linux/man-pages/man8/iotop.8.html)
 - [https://man.linuxde.net/iotop](https://man.linuxde.net/iotop)
 - [https://linux.cn/article-10815-1.html](https://linux.cn/article-10815-1.html)

