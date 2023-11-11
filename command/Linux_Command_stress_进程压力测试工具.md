#  Linux Command stress 进程压力测试工具



----
## 1. 简介
为了测试服务器的负载情况，可以使用stress这个压力测试工具
## 2. 安装

```bash
sudo yum install -y epel-release
sudo yum install -y stress
```
## 3. 参数
语法格式：
stress <options>

常用选项：

```bash
-c, --cpu N              产生 N 个进程，每个进程都反复不停的计算随机数的平方根
-i, --io N               产生 N 个进程，每个进程反复调用 sync() 将内存上的内容写到硬盘上
-m, --vm N             产生 N 个进程，每个进程不断分配和释放内存
    --vm-bytes B      指定分配内存的大小
    --vm-stride B     不断的给部分内存赋值，让 COW(Copy On Write)发生
    --vm-hang N      指示每个消耗内存的进程在分配到内存后转入睡眠状态 N 秒，然后释放内存，一直重复执行这个过程
    --vm-keep          一直占用内存，区别于不断的释放和重新分配(默认是不断释放并重新分配内存)
-d, --hadd N           产生 N 个不断执行 write 和 unlink 函数的进程(创建文件，写入内容，删除文件)
    --hadd-bytes B  指定文件大小
-t, --timeout N       在 N 秒后结束程序        
--backoff N            等待N微妙后开始运行
-q, --quiet              程序在运行的过程中不输出信息
-n, --dry-run          输出程序会做什么而并不实际执行相关的操作
--version                显示版本号
-v, --verbose          显示详细的信息
```
## 4. 实战
### 4.1 消耗 CPU 资源
stress 消耗 CPU 资源的方式是通过调用 sqrt 函数计算由 rand 函数产生的随机数的平方根实现的。下面的命令会产生 4 个这样的进程不断的进行计算：

```c
$ stress -c 4
stress: info: [8279] dispatching hogs: 4 cpu, 0 io, 0 vm, 0 hdd
```

```c
$ top
top - 12:09:06 up 12 min,  3 users,  load average: 3.11, 0.96, 0.38
Tasks: 141 total,   6 running, 135 sleeping,   0 stopped,   0 zombie
%Cpu(s): 99.8 us,  0.1 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.1 si,  0.0 st
KiB Mem :  2044472 total,  1095064 free,   269304 used,   680104 buff/cache
KiB Swap:  2097148 total,  2097148 free,        0 used.  1660988 avail Mem 

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                                                                                                                               
  8283 root      20   0    7308     96      0 R 100.0  0.0   1:06.35 stress                                                                                                                                                                                
  8281 root      20   0    7308     96      0 R  99.7  0.0   1:06.46 stress                                                                                                                                                                                
  8280 root      20   0    7308     96      0 R  99.3  0.0   1:05.83 stress                                                                                                                                                                                
```
### 4.2 消耗内存资源
产生两个子进程，每个进程分配 300M 内存

```cpp
$ stress --vm 2 --vm-bytes 300M --vm-keep
stress: info: [8344] dispatching hogs: 0 cpu, 0 io, 2 vm, 0 hdd

$ stress --vm 2 --vm-bytes 500M --vm-keep
```

```bash
$ ps aux |grep stress
root       8344  0.0  0.0   7308   432 pts/0    S+   12:11   0:00 stress --vm 2 --vm-bytes 300M --vm-keep
root       8345 99.3 15.0 314512 307268 pts/0   R+   12:11   1:02 stress --vm 2 --vm-bytes 300M --vm-keep
root       8346 99.3 15.0 314512 307268 pts/0   R+   12:11   1:02 stress --vm 2 --vm-bytes 300M --vm-keep
root       8358  0.0  0.0 112724   984 pts/1    S+   12:12   0:00 grep --color=auto stress
父进程处于睡眠状态，两个子进程负责资源消耗。

$ top
top - 12:15:45 up 19 min,  3 users,  load average: 1.98, 1.64, 0.91
Tasks: 138 total,   3 running, 135 sleeping,   0 stopped,   0 zombie
%Cpu0  :  0.0 us,  0.7 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu2  :100.0 us,  0.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu3  :  0.0 us,  0.3 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  2044472 total,   479152 free,   884880 used,   680440 buff/cache
KiB Swap:  2097148 total,  2097148 free,        0 used.  1045272 avail Mem 

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                                                                                                                               
  8345 root      20   0  314512 307268    128 R 100.0 15.0   4:11.07 stress                                                                                                                                                                                
  8346 root      20   0  314512 307268    128 R 100.0 15.0   4:11.09 stress                                                                                                                                                                                
  7926 root      20   0  663944  44860  14128 S   1.0  2.2   0:13.82 containerd                                                                                                                                                                            
     9 root      20   0       0      0      0 S   0.3  0.0   0:00.92 rcu_sched         
```
一直在进行默认的 stride 操作，user 非常高(cpu 在用户态忙碌)。
```bash
--vm-keep
一直占用内存，区别于不断的释放和重新分配(默认是不断释放并重新分配内存)。
--vm-hang N
指示每个消耗内存的进程在分配到内存后转入睡眠状态 N 秒，然后释放内存，一直重复执行这个过程。
```
--vm-keep 和 --vm-hang 都可以用来模拟只有少量内存的机器，但是指定它们时 CPU 的使用情况是不一样的。

```bash
$ stress --vm 2 --vm-bytes 500M --vm-hang 5
```
cpu不高
```bash
[root@localhost ~]# top
top - 12:24:44 up 28 min,  3 users,  load average: 0.12, 0.92, 0.98
Tasks: 138 total,   1 running, 137 sleeping,   0 stopped,   0 zombie
%Cpu0  :  0.0 us,  0.3 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  :  0.0 us,  0.3 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.0 hi,  0.3 si,  0.0 st
%Cpu2  :  0.7 us,  7.7 sy,  0.0 ni, 91.6 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu3  :  0.3 us,  0.0 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  2044472 total,    70056 free,  1293968 used,   680448 buff/cache
KiB Swap:  2097148 total,  2097148 free,        0 used.   636192 avail Mem 

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                                                                                                                               
  8428 root      20   0  519312 512088    132 S   4.3 25.0   0:07.09 stress                                                                                                                                                                                
  8427 root      20   0  519312 512088    132 S   4.0 25.0   0:07.75 stress  
```
--vm-stride B
不断的给部分内存赋值，让 COW(Copy On Write)发生。只要指定了内存相关的选项，这个操作就会执行，只是大小为默认的 4096。赋值内存的比例由参数决定：

```bash
for (i = 0; i < bytes; i += stride)
    ptr[i] = 'Z';           /* Ensure that COW happens.  */
```

bytes 为消耗的总内存大小，stride 为间隔。
该参数会影响 CPU 状态 us 和 sy：

```bash
$ stress --vm 2 --vm-bytes 500M --vm-stride 64
```

```bash
$ top
%Cpu(s): 28.5 us, 21.8 sy
```

```bash
$ stress --vm 2 --vm-bytes 500M --vm-stride 1M
```

```bash
$ top
%Cpu(s):  0.5 us, 49.1 sy
```
为什么会产生这样的结果？原因是单独的赋值和对比操作可以让 CPU 在用户态的负载占到 99% 以上。--vm-stride 值增大就意味着减少赋值和对比操作，这样就增加了内存的释放和分配次数(cpu在内核空间的负载)。
不指定 --vm-stride 选项就使用默认值是 4096，CPU 负载情况居于前两者之间：

```bash
$ stress --vm 2 --vm-bytes 500M
```

```bash
$ top
%Cpu(s):  2.5 us, 48.0 sy
```
### 4.3 消耗 IO 资源
产生 4 个进程，每个进程都反复调用 sync 函数将内存上的内容写到硬盘上：
```bash
$ top 
%Cpu(s):  0.0 us,  0.2 sy,  0.0 ni, 99.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st

$ stress -i 4
stress: info: [8598] dispatching hogs: 0 cpu, 4 io, 0 vm, 0 hdd


$ top
%Cpu(s):  0.9 us, 52.9 sy,  0.0 ni, 46.2 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                                                                                                                               
  8602 root      20   0    7308    100      0 R  58.5  0.0   0:54.75 stress                                                                                                                                                                                
  8599 root      20   0    7308    100      0 R  57.8  0.0   0:55.73 stress                                                                                                                                                                                
  8601 root      20   0    7308    100      0 R  57.8  0.0   0:55.80 stress                                                                                                                                                                                
  8600 root      20   0    7308    100      0 R  55.8  0.0   0:55.13 stress 
```
sy 升高

### 4.4 压测磁盘及 IO
创建一个进程不断的在磁盘上创建 10M 大小的文件并写入内容：

```bash
$ stress -d 1 --hdd-bytes 10M
```
### 4.5 其它选项介绍

```bash
--verbose 显示 stress 程序运行过程中的详细信息
--timeout N 在 N 秒后结束程序。
--quiet stress 程序运行的过程中不输出信息。
-n, --dry-run 输出程序会做什么而并不实际执行相关的操作
```
--backoff N
让新 fork 出来的进程 sleep N 微秒再开始运行。

除了单独指定某一类的选项，还可以同时执行多个类型的任务，比如产生 3 个 CPU 进程、3 个 IO 进程、2 个10M 的 vm 进程，并且每个 vm 进程中不循环分配释放内存：

```bash
$ stress --cpu 3 --io 3 --vm 2 --vm-bytes 10M --vm-keep
```
更多阅读：

 - [https://www.cnblogs.com/sparkdev/p/10354947.html](https://www.cnblogs.com/sparkdev/p/10354947.html)
 - [How to Stress Test Your CPU in Linux](https://www.tomshardware.com/how-to/stress-test-cpu-in-linux)
 - [stress(1) - Linux man page](https://linux.die.net/man/1/stress)
 - [How to Impose High CPU Load and Stress Test on Linux Using ‘Stress-ng’ Tool](https://www.tecmint.com/linux-cpu-load-stress-test-with-stress-ng-tool/)
 - [How to stress test your Linux system](https://www.networkworld.com/article/3563334/how-to-stress-test-your-linux-system.html)

