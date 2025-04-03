#  Linux Command ps 性能分析
tags: 分析



![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f8d138b76d09fe589a63794a6333095a.png)



## 1. 简介
ps 为我们提供了进程的一次性的查看，它所提供的查看结果并不动态连续的；如果想对进程时间监控，应该用 top 工具。

## 2. 参数

```bash
ps a 显示现行终端机下的所有程序，包括其他用户的程序。
ps -A 显示所有进程。
ps c 列出程序时，显示每个程序真正的指令名称，而不包含路径，参数或常驻服务的标示。
ps -e 此参数的效果和指定"A"参数相同。
ps e 列出程序时，显示每个程序所使用的环境变量。
ps f 用ASCII字符显示树状结构，表达程序间的相互关系。
ps -H 显示树状结构，表示程序间的相互关系。
ps j 用任务格式来显示进程；
ps l 长格式输出；
ps -N 显示所有的程序，除了执行ps指令终端机下的程序之外。
ps r 显示运行中的进程；
ps s 采用程序信号的格式显示程序状况。
ps S 列出程序时，包括已中断的子程序资料。
ps -t<终端机编号> 　指定终端机编号，并列出属于该终端机的程序的状况。
ps u 　按用户名和启动时间的顺序来显示进程。
ps x 　显示无控制终端的进程。
ps ww 避免详细参数被截断；
最常用的方法是ps -aux,然后再利用一个管道符号导向到grep去查找特定的进程,然后再对特定的进程进行操作。
```

我们常用的选项是组合是 aux 或 lax，还有参数 f 的应用。

## 3. 输出说明
运行 ps aux 的到如下信息：

```bash
$ ps aux
USER      PID       %CPU    %MEM    VSZ    RSS    TTY    STAT    START    TIME    COMMAND
smmsp    3521    0.0    0.7    6556    1616    ?    Ss    20:40    0:00    sendmail: Queue runner@01:00:00 f
root    3532    0.0    0.2    2428    452    ?    Ss    20:40    0:00    gpm -m /dev/input/mice -t imps2
htt    3563    0.0    0.0    2956    196    ?    Ss    20:41    0:00    /usr/sbin/htt -retryonerror 0
htt    3564    0.0    1.7    29460    3704    ?    Sl    20:41    0:00    htt_server -nodaemon
root    3574    0.0    0.4    5236    992    ?    Ss    20:41    0:00    crond
xfs    3617    0.0    1.3    13572    2804    ?    Ss    20:41    0:00    xfs -droppriv -daemon
root    3627    0.0    0.2    3448    552    ?    SNs    20:41    0:00    anacron -s
root    3636    0.0    0.1    2304    420    ?    Ss    20:41    0:00    /usr/sbin/atd
dbus    3655    0.0    0.5    13840    1084    ?    Ssl    20:41    0:00    dbus-daemon-1 --system
```

Head标头：

```bash
USER    用户名
UID    用户ID（User ID）
PID    进程ID（Process ID）
PPID    父进程的进程ID（Parent Process id）
SID    会话ID（Session id）
%CPU    进程的cpu占用率
%MEM    进程的内存占用率
VSZ    进程所使用的虚存的大小（Virtual Size）
RSS    进程使用的驻留集大小或者是实际内存的大小，Kbytes字节。
TTY    与进程关联的终端（tty）

STAT    进程的状态：进程状态使用字符表示的（STAT的状态码）
R 运行    Runnable (on run queue)            正在运行或在运行队列中等待。
S 睡眠    Sleeping                休眠中, 受阻, 在等待某个条件的形成或接受到信号。
I 空闲    Idle
Z 僵死    Zombie（a defunct process)        进程已终止, 但进程描述符存在, 直到父进程调用wait4()系统调用后释放。
D 不可中断    Uninterruptible sleep (ususally IO)    收到信号不唤醒和不可运行, 进程必须等待直到有中断发生。
T 终止    Terminate                进程收到SIGSTOP, SIGSTP, SIGTIN, SIGTOU信号后停止运行运行。

P 等待交换页
W 无驻留页    has no resident pages        没有足够的记忆体分页可分配。
X 死掉的进程
< 高优先级进程                    高优先序的进程
N 低优先    级进程                    低优先序的进程
L 内存锁页    Lock                有记忆体分页分配并缩在记忆体内
s 进程的领导者（在它之下有子进程）；
l 多进程的（使用 CLONE_THREAD, 类似 NPTL pthreads）
+ 位于后台的进程组 
START    进程启动时间和日期
TIME    进程使用的总cpu时间
COMMAND    正在执行的命令行命令
NI    优先级(Nice)
PRI    进程优先级编号(Priority)
WCHAN    进程正在睡眠的内核函数名称；该函数的名称是从/root/system.map文件中获得的。
FLAGS    与进程相关的数字标识
args 命令参数
```


## 4. 实例
### 4.1 ps 不带任何选项
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f88b6b3b06789415a58242accbb418a5.png)

默认输出包括以下类别：

- PID：进程标识号。
- TTY：进程运行的终端类型。
- TIME : CPU 使用总量。
- CMD：启动进程的命令的名称。

###  4.2 a、u、x组合
![a产生u更x详细的输出](https://i-blog.csdnimg.cn/blog_migrate/f3228da55f6847f38291ca936b1c0ba8.png)

扩展输出的新类别包括：

- `USER`：运行进程的用户名。
- `%CPU`：CPU 使用百分比。
- `%MEM`：[内存使用](https://phoenixnap.com/kb/linux-commands-check-memory-usage)百分比。
- `VSZ`：进程使用的总[虚拟内存](https://phoenixnap.com/glossary/virtual-memory-definition)，以千字节为单位。
- `RSS`：驻留集大小，进程占用的 RAM 部分。
- `STAT`：当前进程状态。
- `START`：进程开始的时间。

### 4.3 分层视图中显示正在运行的进程

```bash
ps -axjf
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/099f52a536c386f6f04b0030f5c7106c.png)

### 4.4 按用户过滤进程列表

```bash
ps -U [real user ID or name] -u [effective user ID or name] u
```

例如，显示由名为`phoenixnap`的用户启动的进程列表：

```bash
ps -U phoenixnap -u phoenixnap u
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/94bceb05364cd5e50ce6dde395e8a16e.png)

### 4.5 其他选项

```bash
查看当前系统进程的uid,pid,stat,pri, 以uid号排序.
ps -eo pid,stat,pri,uid –sort uid

查看当前系统进程的user,pid,stat,rss,args, 以rss排序.
ps -eo user,pid,stat,rss,args –sort rss

查看所有进程详细展示
ps axHww -o psr,user,pid,ppid,tid,tty,stat,%cpu,%mem,rss,vsz,wchan,args

查看已经停止的进程
ps -A -ostat,ppid,pid,cmd | grep -e '^[T]'
```

参考：

 - [ps(1) — Linux manual page](https://man7.org/linux/man-pages/man1/ps.1.html#top_of_page)
 - [ps command in Linux with Examples](https://www.geeksforgeeks.org/ps-command-in-linux-with-examples/)
 - [The Linux “ps” Command Examples](https://linuxhint.com/linux-ps-command-examples/)
 - [How to Use the ps Command on Linux](https://pimylifeup.com/ps-command-linux/)

