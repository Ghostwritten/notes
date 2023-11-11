

---

 - [https://sysdig.com/](https://sysdig.com/)
 - [sysdig](https://github.com/draios/sysdig/wiki/Sysdig%20User%20Guide)

---

##  1. 介绍

使用 `sysdig` 最简单的方法是不带任何参数地调用它。这样做会导致 `sysdig` 捕获每个事件并将其写入标准输出，就像 `strace` 所做的那样。

```bash
$ sysdig
34378 12:02:36.269753803 2 echo (7896) > close fd=3(/usr/lib/locale/locale-archive)
34379 12:02:36.269754164 2 echo (7896) < close res=0
34380 12:02:36.269781699 2 echo (7896) > fstat fd=1(/dev/pts/3)
34381 12:02:36.269783882 2 echo (7896) < fstat res=0
34382 12:02:36.269784970 2 echo (7896) > mmap
34383 12:02:36.269786575 2 echo (7896) < mmap
34384 12:02:36.269827674 2 echo (7896) > write fd=1(/dev/pts/3) size=12
34385 12:02:36.269839477 2 echo (7896) < write res=12 data=hello world.
34386 12:02:36.269843986 2 echo (7896) > close fd=1(/dev/pts/3)
34387 12:02:36.269844466 2 echo (7896) < close res=0
34388 12:02:36.269844816 2 echo (7896) > munmap
34389 12:02:36.269850803 2 echo (7896) < munmap
34390 12:02:36.269851915 2 echo (7896) > close fd=2(/dev/pts/3)
34391 12:02:36.269852314 2 echo (7896) < close res=0
```
默认情况下，sysdig 在一行中打印每个事件的信息，格式如下：

```bash
*%evt.num %evt.time %evt.cpu %proc.name (%thread.tid) %evt.dir %evt.type %evt.args
```
在哪里：

 - `evt.num` 是增量事件编号
 - `evt.time` 是事件时间戳
 - `evt.cpu` 是捕获事件的 CPU 编号
 - `proc.name` 是生成事件的进程的名称
 - `thread.tid` 是产生事件的TID，对应单线程进程的PID
 - `evt.dir` 是事件方向，> 表示进入事件，< 表示退出事件
 - `evt.type` 是事件的名称，例如 'open' 或 'read'
 - `evt.args` 是事件参数的列表。

在系统调用的情况下，这些往往对应于系统调用参数，但情况并非总是如此：出于简单或性能原因，某些系统调用参数被排除在外。

-p 修改格式

```bash
$ sysdig   -p "*%evt.num %evt.cpu %proc.name (%thread.tid) %evt.dir %evt.type %evt.args"
7446 0 sshd (74996) < clock_gettime 
7447 0 sshd (74996) > read fd=11(<f>/dev/ptmx) size=16384 
7449 1 sysdig (2938) > switch next=99806 pgft_maj=0 pgft_min=1526 vm_size=178120 vm_rss=16616 vm_swap=0 
7450 0 sshd (74996) < read res=406 data=991 1 <NA> (99806) > switch next=2938(sysdig) pgft_maj=0 pgft_min=0 vm_size=0 vm 
7451 1 <NA> (99806) > switch next=2938(sysdig) pgft_maj=0 pgft_min=0 vm_size=0 vm_rss=0 vm_swap=0 
```

##  2. sysdig VS strace
通过查看输出，您可以立即发现此输出与 strace 之间的一些关键区别：

 - 对于大多数系统调用，sysdig 显示两个单独的条目：一个进入（标有“>”）和一个退出（标有“<”）。这使得在多进程环境中更容易跟踪输出。
 - 文件描述符被解析。这意味着，只要有可能，FD 编号后跟 FD 本身的人类可读表示：网络连接的元组、文件的名称等。用于渲染 FD 的确切格式如下： `num(<type>resolved_string)` 其中：num 是 FD 编号
 - resolve_string 是 FD 的解析表示，例如 `127.0.0.1:40370->127.0.0.1:80` 用于 TCP 套接字
 - type 是 fd 类型的单字母编码，可以是以下之一：

- - f 用于文件
- - 4 个用于 IPv4 套接字
- - 6 个用于 IPv6 套接字
- - u 用于 unix 套接字
- - s 用于信号 FD
- - e 用于事件 FD
- - i 用于 inotify FD
- - t 用于定时器 FD

至此，你应该可以了解 sysdig 输出的基础知识了，当然 sysdig 是一个强大的工具，可以展示很多有趣的东西。这两篇博文将为您提供更多详细信息和背景信息：

 - [解释 Sysdig 输出](https://sysdig.com/blog/interpreting-sysdig-output/)
 - [Linux 系统调用的迷人世界](https://sysdig.com/blog/fascinating-world-linux-system-calls/)

##  3. 事件捕获写入文件

-w 写入文件, Sysdig 允许您将捕获的事件保存到磁盘，以便以后对其进行分析。语法如下：

```bash
sysdig -w dumpfile.scap
```
如果要将保存到文件的事件数限制为 100，可以使用 –n 标志：

```bash
sysdig -n 20 -w myfile.scap
```
-C 命令行标志允许您将捕获拆分为特定大小的文件。例如，使用它来生成大小为 1MB 的文件：

```bash
sysdig -C 1 -w dump.scap
```
您可以将 `-C` 标志与 `-W` 标志结合起来，以告诉 sysdig 要保留多少文件。例如，此命令行将事件捕获到大小为 1MB 的文件中，并且仅将最后 5 个文件保留在磁盘上：

```bash
sysdig -C 1 -W 5 -w dump.scap
```
可以使用 -r 标志读取以前保存的捕获文件：

```bash
sysdig -r myfile.scap
```
 -G 标志替代 -C 来指定时间跨度而不是文件大小。例如，此命令行将事件捕获到包含 1 分钟系统活动的文件中，并保留最后 5 分钟：

```bash
 sysdig -G 60 -W 5 -w dump.scap
```
如果要在每个文件中包含特定数量的事件，可以使用 `-e` 标志。例如，此命令行将事件保存到每个包含 1000 个事件的文件中，并且只保留磁盘上的最后 5 个文件：

```bash
sysdig -e 1000 -W 5 -w dump.scap
```
高级
您的应用程序在随机时间点崩溃，当这种情况发生时，您希望查看其最后 10 分钟的活动。您可以通过以下方式实现：

```bash
sysdig -G 60 -W 10 -wdump.scap proc.name=myapp
```
您想要捕获过去 24 小时内在容器中执行的所有命令。这是你如何做到的：

```bash
sysdig -G 86400 -W 1 -w dump.scap evt.type=execve or evt.type=clone and container.name=mycontainer
```
生成的文件将非常小，因为只会捕获给定容器的相关系统调用。您将能够检查生成的文件：

```bash
$ sysdig -r “dump.scap0” -c spy_users
5459 19:42:15 root) cd /usr/sbin
5459 19:42:15 root) mkdir .shm
5459 19:42:17 root) cd /usr/sbin/.shm
5459 19:42:18 root) wget XXX/l.tgz
5459 19:42:20 root) tar zxvf l.tgz
5459 19:42:20 root) gzip -d
5459 19:44:40 root) ps -x
5459 19:44:47 root) w
5459 19:44:50 root) passwd
5459 19:46:21 root) cd /usr/sbin
5459 19:46:21 root) wget XXX/rk.tar
5459 19:46:24 root) tar xvf rk.tar
5459 19:46:24 root) rm -rf rk.tar
5459 19:46:24 root) cd /usr/sbin/rk
5459 19:46:24 root) /bin/bash ./root exampleabcdef 1157
```
您想要捕获给定日志文件的最后 10 分钟输出：

```bash
sysdig -G 600 -W 1 -w dump.scap evt.is_io_write=true and fd.name contains mylogfile
```
您将能够检查生成的文件：

```bash
sysdig -r dump.scap0 -c echo_fds
```
文件轮换在使用来自跟踪文件的事件时也有效。这非常方便，因为它允许您根据自己喜欢的标准将较大的跟踪文件拆分为较小的文件。

例如，此命令行获取 `dump.scap` 文件并将其分成 5 分钟长的文件。

```bash
sysdig -r dump.scap -G 300 -z -w segments.scap
```

##  4. 过滤
Sysdig 的过滤系统功能强大且用途广泛，旨在大海捞针。过滤器在命令行末尾指定，就像在 tcpdump 中一样，可以应用于实时捕获或捕获文件。例如，让我们看一下特定命令的活动，在本例中为

```bash
$ ./sysdig proc.name=cat
21368 13:10:15.384878134 1 cat (8298) < execve res=0 exe=cat args=index.html. tid=8298(cat) pid=8298(cat) ptid=1978(bash) cwd=/root fdlimit=1024
21371 13:10:15.384948635 1 cat (8298) > brk size=0
21372 13:10:15.384949909 1 cat (8298) < brk res=10665984
21373 13:10:15.384976208 1 cat (8298) > mmap
21374 13:10:15.384979452 1 cat (8298) < mmap
21375 13:10:15.384990980 1 cat (8298) > access
21376 13:10:15.384999211 1 cat (8298) < access
21377 13:10:15.385008602 1 cat (8298) > open
21378 13:10:15.385014374 1 cat (8298) < open fd=3(/etc/ld.so.cache) name=/etc/ld.so.cache flags=0(O_NONE) mode=0
21379 13:10:15.385015508 1 cat (8298) > fstat fd=3(/etc/ld.so.cache)
21380 13:10:15.385016588 1 cat (8298) < fstat res=0
21381 13:10:15.385017033 1 cat (8298) > mmap
21382 13:10:15.385019763 1 cat (8298) < mmap
21383 13:10:15.385020047 1 cat (8298) > close fd=3(/etc/ld.so.cache)
21384 13:10:15.385020556 1 cat (8298) < close res=0
```
如您所见，sysdig 不附加到进程。它只是捕获所有内容，然后让您过滤掉您不感兴趣的内容。过滤器语句可以使用比较运算符（=、!=、<、<=、>、>=、包含、icontains、in、exists）并且可以使用布尔运算符（and、or 和 not）和括号组合。例如

```bash
#捕获 cat 和 vi 的活动
$ sysdig proc.name=cat or proc.name=vi
#显示由非 cat 程序打开的所有文件。
$ sysdig proc.name!=cat and evt.type=open
```
过滤器字段表示为“ class.field ”。获取可用类列表及其包含的字段的快速方法是

```bash
$ sysdig -l
----------------------
Field Class: fd

fd.num          the unique number identifying the file descriptor.
fd.type         type of FD. Can be 'file', 'directory', 'ipv4', 'ipv6', 'unix',
                 'pipe', 'event', 'signalfd', 'eventpoll', 'inotify' or 'signal
                fd'.
fd.typechar     type of FD as a single character. Can be 'f' for file, 4 for IP
                v4 socket, 6 for IPv6 socket, 'u' for unix socket, p for pipe, 
                'e' for eventfd, 's' for signalfd, 'l' for eventpoll, 'i' for i
                notify, 'o' for unknown.
fd.name         FD full name. If the fd is a file, this field contains the full
                 path. If the FD is a socket, this field contain the connection
                 tuple.
fd.directory    If the fd is a file, the directory that contains it.
fd.filename     If the fd is a file, the filename without the path.
fd.ip           (FILTER ONLY) matches the ip address (client or server) of the 
                fd.
fd.cip          client IP address.
fd.sip          server IP address.
fd.lip          local IP address.
fd.rip          remote IP address.
fd.port         (FILTER ONLY) matches the port (either client or server) of the
                 fd.
fd.cport        for TCP/UDP FDs, the client port.
fd.sport        for TCP/UDP FDs, server port.
fd.lport        for TCP/UDP FDs, the local port.
fd.rport        for TCP/UDP FDs, the remote port.
fd.l4proto      the IP protocol of a socket. Can be 'tcp', 'udp', 'icmp' or 'ra
                w'.
fd.sockfamily   the socket family for socket events. Can be 'ip' or 'unix'.
fd.is_server    'true' if the process owning this FD is the server endpoint in 
                the connection.
fd.uid          a unique identifier for the FD, created by chaining the FD numb
                er and the thread ID.
fd.containername
                chaining of the container ID and the FD name. Useful when tryin
                g to identify which container an FD belongs to.
fd.containerdirectory
                chaining of the container ID and the directory name. Useful whe
                n trying to identify which container a directory belongs to.
fd.proto        (FILTER ONLY) matches the protocol (either client or server) of
                 the fd.
fd.cproto       for TCP/UDP FDs, the client protocol.
fd.sproto       for TCP/UDP FDs, server protocol.
fd.lproto       for TCP/UDP FDs, the local protocol.
fd.rproto       for TCP/UDP FDs, the remote protocol.
fd.net          (FILTER ONLY) matches the IP network (client or server) of the 
                fd.
fd.cnet         (FILTER ONLY) matches the client IP network of the fd.
fd.snet         (FILTER ONLY) matches the server IP network of the fd.
fd.lnet         (FILTER ONLY) matches the local IP network of the fd.
fd.rnet         (FILTER ONLY) matches the remote IP network of the fd.
fd.connected    for TCP/UDP FDs, 'true' if the socket is connected.
fd.name_changed True when an event changes the name of an fd used by this event
                . This can occur in some cases such as udp connections where th
                e connection tuple changes.
fd.cip.name     Domain name associated with the client IP address.
fd.sip.name     Domain name associated with the server IP address.
fd.lip.name     Domain name associated with the local IP address.
fd.rip.name     Domain name associated with the remote IP address.

----------------------
Field Class: process

proc.pid        the id of the process generating the event.
proc.exe        the first command line argument (usually the executable name or
                 a custom one).
proc.name       the name (excluding the path) of the executable generating the 
                event.
proc.args       the arguments passed on the command line when starting the proc
                ess generating the event.
proc.env        the environment variables of the process generating the event.
proc.cmdline    full process command line, i.e. proc.name + proc.args.
proc.exeline    full process command line, with exe as first argument, i.e. pro
                c.exe + proc.args.
proc.cwd        the current working directory of the event.
proc.nthreads   the number of threads that the process generating the event cur
                rently has, including the main process thread.
proc.nchilds    the number of child threads that the process generating the eve
                nt currently has. This excludes the main process thread.
proc.ppid       the pid of the parent of the process generating the event.
proc.pname      the name (excluding the path) of the parent of the process gene
                rating the event.
proc.pcmdline   the full command line (proc.name + proc.args) of the parent of 
                the process generating the event.
proc.apid       the pid of one of the process ancestors. E.g. proc.apid[1] retu
                rns the parent pid, proc.apid[2] returns the grandparent pid, a
                nd so on. proc.apid[0] is the pid of the current process. proc.
                apid without arguments can be used in filters only and matches 
                any of the process ancestors, e.g. proc.apid=1234.
proc.aname      the name (excluding the path) of one of the process ancestors. 
                E.g. proc.aname[1] returns the parent name, proc.aname[2] retur
                ns the grandparent name, and so on. proc.aname[0] is the name o
                f the current process. proc.aname without arguments can be used
                 in filters only and matches any of the process ancestors, e.g.
                 proc.aname=bash.
proc.loginshellid
                the pid of the oldest shell among the ancestors of the current 
                process, if there is one. This field can be used to separate di
                fferent user sessions, and is useful in conjunction with chisel
                s like spy_user.
proc.duration   number of nanoseconds since the process started.
proc.fdopencount
                number of open FDs for the process
proc.fdlimit    maximum number of FDs the process can open.
proc.fdusage    the ratio between open FDs and maximum available FDs for the pr
                ocess.
proc.vmsize     total virtual memory for the process (as kb).
proc.vmrss      resident non-swapped memory for the process (as kb).
proc.vmswap     swapped memory for the process (as kb).
thread.pfmajor  number of major page faults since thread start.
thread.pfminor  number of minor page faults since thread start.
thread.tid      the id of the thread generating the event.
thread.ismain   'true' if the thread generating the event is the main one in th
                e process.
thread.exectime CPU time spent by the last scheduled thread, in nanoseconds. Ex
                ported by switch events only.
thread.totexectime
                Total CPU time, in nanoseconds since the beginning of the captu
                re, for the current thread. Exported by switch events only.
thread.cgroups  all the cgroups the thread belongs to, aggregated into a single
                 string.
thread.cgroup   the cgroup the thread belongs to, for a specific subsystem. E.g
                . thread.cgroup.cpuacct.
thread.vtid     the id of the thread generating the event as seen from its curr
                ent PID namespace.
proc.vpid       the id of the process generating the event as seen from its cur
                rent PID namespace.
thread.cpu      the CPU consumed by the thread in the last second.
thread.cpu.user the user CPU consumed by the thread in the last second.
thread.cpu.system
                the system CPU consumed by the thread in the last second.
thread.vmsize   For the process main thread, this is the total virtual memory f
                or the process (as kb). For the other threads, this field is ze
                ro.
thread.vmrss    For the process main thread, this is the resident non-swapped m
                emory for the process (as kb). For the other threads, this fiel
                d is zero.
proc.sid        the session id of the process generating the event.
proc.sname      the name of the current process's session leader. This is eithe
                r the process with pid=proc.sid or the eldest ancestor that has
                 the same sid as the current process.
proc.tty        The controlling terminal of the process. 0 for processes withou
                t a terminal.
proc.exepath    The full executable path of the process.
proc.vpgid      the process group id of the process generating the event, as se
                en from its current PID namespace.

----------------------
Field Class: evt

evt.num         event number.
evt.time        event timestamp as a time string that includes the nanosecond p
                art.
evt.time.s      event timestamp as a time string with no nanoseconds.
evt.datetime    event timestamp as a time string that includes the date.
evt.rawtime     absolute event timestamp, i.e. nanoseconds from epoch.
evt.rawtime.s   integer part of the event timestamp (e.g. seconds since epoch).
evt.rawtime.ns  fractional part of the absolute event timestamp.
evt.reltime     number of nanoseconds from the beginning of the capture.
evt.reltime.s   number of seconds from the beginning of the capture.
evt.reltime.ns  fractional part (in ns) of the time from the beginning of the c
                apture.
evt.latency     delta between an exit event and the correspondent enter event, 
                in nanoseconds.
evt.latency.s   integer part of the event latency delta.
evt.latency.ns  fractional part of the event latency delta.
evt.latency.human
                delta between an exit event and the correspondent enter event, 
                as a human readable string (e.g. 10.3ms).
evt.deltatime   delta between this event and the previous event, in nanoseconds
                .
evt.deltatime.s integer part of the delta between this event and the previous e
                vent.
evt.deltatime.ns
                fractional part of the delta between this event and the previou
                s event.
evt.outputtime  this depends on -t param, default is %evt.time ('h').
evt.dir         event direction can be either '>' for enter events or '<' for e
                xit events.
evt.type        The name of the event (e.g. 'open').
evt.type.is     allows one to specify an event type, and returns 1 for events t
                hat are of that type. For example, evt.type.is.open returns 1 f
                or open events, 0 for any other event.
syscall.type    For system call events, the name of the system call (e.g. 'open
                '). Unset for other events (e.g. switch or sysdig internal even
                ts). Use this field instead of evt.type if you need to make sur
                e that the filtered/printed value is actually a system call.
evt.category    The event category. Example values are 'file' (for file operati
                ons like open and close), 'net' (for network operations like so
                cket and bind), memory (for things like brk or mmap), and so on
                .
evt.cpu         number of the CPU where this event happened.
evt.args        all the event arguments, aggregated into a single string.
evt.arg         one of the event arguments specified by name or by number. Some
                 events (e.g. return codes or FDs) will be converted into a tex
                t representation when possible. E.g. 'evt.arg.fd' or 'evt.arg[0
                ]'.
evt.rawarg      one of the event arguments specified by name. E.g. 'evt.rawarg.
                fd'.
evt.info        for most events, this field returns the same value as evt.args.
                 However, for some events (like writes to /dev/log) it provides
                 higher level information coming from decoding the arguments.
evt.buffer      the binary data buffer for events that have one, like read(), r
                ecvfrom(), etc. Use this field in filters with 'contains' to se
                arch into I/O data buffers.
evt.buflen      the length of the binary data buffer for events that have one, 
                like read(), recvfrom(), etc.
evt.res         event return value, as a string. If the event failed, the resul
                t is an error code string (e.g. 'ENOENT'), otherwise the result
                 is the string 'SUCCESS'.
evt.rawres      event return value, as a number (e.g. -2). Useful for range com
                parisons.
evt.failed      'true' for events that returned an error status.
evt.is_io       'true' for events that read or write to FDs, like read(), send,
                 recvfrom(), etc.
evt.is_io_read  'true' for events that read from FDs, like read(), recv(), recv
                from(), etc.
evt.is_io_write 'true' for events that write to FDs, like write(), send(), etc.
evt.io_dir      'r' for events that read from FDs, like read(); 'w' for events 
                that write to FDs, like write().
evt.is_wait     'true' for events that make the thread wait, e.g. sleep(), sele
                ct(), poll().
evt.wait_latency
                for events that make the thread wait (e.g. sleep(), select(), p
                oll()), this is the time spent waiting for the event to return,
                 in nanoseconds.
evt.is_syslog   'true' for events that are writes to /dev/log.
evt.count       This filter field always returns 1 and can be used to count eve
                nts from inside chisels.
evt.count.error This filter field returns 1 for events that returned with an er
                ror, and can be used to count event failures from inside chisel
                s.
evt.count.error.file
                This filter field returns 1 for events that returned with an er
                ror and are related to file I/O, and can be used to count event
                 failures from inside chisels.
evt.count.error.net
                This filter field returns 1 for events that returned with an er
                ror and are related to network I/O, and can be used to count ev
                ent failures from inside chisels.
evt.count.error.memory
                This filter field returns 1 for events that returned with an er
                ror and are related to memory allocation, and can be used to co
                unt event failures from inside chisels.
evt.count.error.other
                This filter field returns 1 for events that returned with an er
                ror and are related to none of the previous categories, and can
                 be used to count event failures from inside chisels.
evt.count.exit  This filter field returns 1 for exit events, and can be used to
                 count single events from inside chisels.
evt.around      (FILTER ONLY) Accepts the event if it's around the specified ti
                me interval. The syntax is evt.around[T]=D, where T is the valu
                e returned by %evt.rawtime for the event and D is a delta in mi
                lliseconds. For example, evt.around[1404996934793590564]=1000 w
                ill return the events with timestamp with one second before the
                 timestamp and one second after it, for a total of two seconds 
                of capture.
evt.abspath     Absolute path calculated from dirfd and name during syscalls li
                ke renameat and symlinkat. Use 'evt.abspath.src' or 'evt.abspat
                h.dst' for syscalls that support multiple paths.
evt.is_open_read
                'true' for open/openat events where the path was opened for rea
                ding
evt.is_open_write
                'true' for open/openat events where the path was opened for wri
                ting

----------------------
Field Class: user

user.uid        user ID.
user.name       user name.
user.homedir    home directory of the user.
user.shell      user's shell.
user.loginuid   audit user id (auid).
user.loginname  audit user name (auid).

----------------------
Field Class: group

group.gid       group ID.
group.name      group name.

----------------------
Field Class: syslog

syslog.facility.str
                facility as a string.
syslog.facility facility as a number (0-23).
syslog.severity.str
                severity as a string. Can have one of these values: emerg, aler
                t, crit, err, warn, notice, info, debug
syslog.severity severity as a number (0-7).
syslog.message  message sent to syslog.

----------------------
Field Class: container

container.id    the container id.
container.name  the container name.
container.image the container image name (e.g. sysdig/sysdig:latest for docker,
                 ).
container.image.id
                the container image id (e.g. 6f7e2741b66b).
container.type  the container type, eg: docker or rkt
container.privileged
                true for containers running as privileged, false otherwise
container.mounts
                A space-separated list of mount information. Each item in the l
                ist has the format <source>:<dest>:<mode>:<rdrw>:<propagation>
container.mount Information about a single mount, specified by number (e.g. con
                tainer.mount[0]) or mount source (container.mount[/usr/local]).
                 The pathname can be a glob (container.mount[/usr/local/*]), in
                 which case the first matching mount will be returned. The info
                rmation has the format <source>:<dest>:<mode>:<rdrw>:<propagati
                on>. If there is no mount with the specified index or matching 
                the provided source, returns the string "none" instead of a NUL
                L value.
container.mount.source
                the mount source, specified by number (e.g. container.mount.sou
                rce[0]) or mount destination (container.mount.source[/host/lib/
                modules]). The pathname can be a glob.
container.mount.dest
                the mount destination, specified by number (e.g. container.moun
                t.dest[0]) or mount source (container.mount.dest[/lib/modules])
                . The pathname can be a glob.
container.mount.mode
                the mount mode, specified by number (e.g. container.mount.mode[
                0]) or mount source (container.mount.mode[/usr/local]). The pat
                hname can be a glob.
container.mount.rdwr
                the mount rdwr value, specified by number (e.g. container.mount
                .rdwr[0]) or mount source (container.mount.rdwr[/usr/local]). T
                he pathname can be a glob.
container.mount.propagation
                the mount propagation value, specified by number (e.g. containe
                r.mount.propagation[0]) or mount source (container.mount.propag
                ation[/usr/local]). The pathname can be a glob.
container.image.repository
                the container image repository (e.g. sysdig/sysdig).
container.image.tag
                the container image tag (e.g. stable, latest).
container.image.digest
                the container image registry digest (e.g. sha256:d977378f890d44
                5c15e51795296e4e5062f109ce6da83e0a355fc4ad8699d27).

----------------------
Field Class: fdlist

fdlist.nums     for poll events, this is a comma-separated list of the FD numbe
                rs in the 'fds' argument, returned as a string.
fdlist.names    for poll events, this is a comma-separated list of the FD names
                 in the 'fds' argument, returned as a string.
fdlist.cips     for poll events, this is a comma-separated list of the client I
                P addresses in the 'fds' argument, returned as a string.
fdlist.sips     for poll events, this is a comma-separated list of the server I
                P addresses in the 'fds' argument, returned as a string.
fdlist.cports   for TCP/UDP FDs, for poll events, this is a comma-separated lis
                t of the client TCP/UDP ports in the 'fds' argument, returned a
                s a string.
fdlist.sports   for poll events, this is a comma-separated list of the server T
                CP/UDP ports in the 'fds' argument, returned as a string.

----------------------
Field Class: k8s

k8s.pod.name    Kubernetes pod name.
k8s.pod.id      Kubernetes pod id.
k8s.pod.label   Kubernetes pod label. E.g. 'k8s.pod.label.foo'.
k8s.pod.labels  Kubernetes pod comma-separated key/value labels. E.g. 'foo1:bar
                1,foo2:bar2'.
k8s.rc.name     Kubernetes replication controller name.
k8s.rc.id       Kubernetes replication controller id.
k8s.rc.label    Kubernetes replication controller label. E.g. 'k8s.rc.label.foo
                '.
k8s.rc.labels   Kubernetes replication controller comma-separated key/value lab
                els. E.g. 'foo1:bar1,foo2:bar2'.
k8s.svc.name    Kubernetes service name (can return more than one value, concat
                enated).
k8s.svc.id      Kubernetes service id (can return more than one value, concaten
                ated).
k8s.svc.label   Kubernetes service label. E.g. 'k8s.svc.label.foo' (can return 
                more than one value, concatenated).
k8s.svc.labels  Kubernetes service comma-separated key/value labels. E.g. 'foo1
                :bar1,foo2:bar2'.
k8s.ns.name     Kubernetes namespace name.
k8s.ns.id       Kubernetes namespace id.
k8s.ns.label    Kubernetes namespace label. E.g. 'k8s.ns.label.foo'.
k8s.ns.labels   Kubernetes namespace comma-separated key/value labels. E.g. 'fo
                o1:bar1,foo2:bar2'.
k8s.rs.name     Kubernetes replica set name.
k8s.rs.id       Kubernetes replica set id.
k8s.rs.label    Kubernetes replica set label. E.g. 'k8s.rs.label.foo'.
k8s.rs.labels   Kubernetes replica set comma-separated key/value labels. E.g. '
                foo1:bar1,foo2:bar2'.
k8s.deployment.name
                Kubernetes deployment name.
k8s.deployment.id
                Kubernetes deployment id.
k8s.deployment.label
                Kubernetes deployment label. E.g. 'k8s.rs.label.foo'.
k8s.deployment.labels
                Kubernetes deployment comma-separated key/value labels. E.g. 'f
                oo1:bar1,foo2:bar2'.

----------------------
Field Class: mesos

mesos.task.name Mesos task name.
mesos.task.id   Mesos task id.
mesos.task.label
                Mesos task label. E.g. 'mesos.task.label.foo'.
mesos.task.labels
                Mesos task comma-separated key/value labels. E.g. 'foo1:bar1,fo
                o2:bar2'.
mesos.framework.name
                Mesos framework name.
mesos.framework.id
                Mesos framework id.
marathon.app.name
                Marathon app name.
marathon.app.id Marathon app id.
marathon.app.label
                Marathon app label. E.g. 'marathon.app.label.foo'.
marathon.app.labels
                Marathon app comma-separated key/value labels. E.g. 'foo1:bar1,
                foo2:bar2'.
marathon.group.name
                Marathon group name.
marathon.group.id
                Marathon group id.

----------------------
Field Class: span

span.id         ID of the span. This is a unique identifier that is used to mat
                ch the enter and exit tracer events for this span. It can also 
                be used to match different spans belonging to a trace.
span.time       time of the span's enter tracer as a human readable string that
                 includes the nanosecond part.
span.ntags      number of tags that this span has.
span.nargs      number of arguments that this span has.
span.tags       dot-separated list of all of the span's tags.
span.tag        one of the span's tags, specified by 0-based offset, e.g. 'span
                .tag[1]'. You can use a negative offset to pick elements from t
                he end of the tag list. For example, 'span.tag[-1]' returns the
                 last tag.
span.args       comma-separated list of the span's arguments.
span.arg        one of the span arguments, specified by name or by 0-based offs
                et. E.g. 'span.arg.xxx' or 'span.arg[1]'. You can use a negativ
                e offset to pick elements from the end of the tag list. For exa
                mple, 'span.arg[-1]' returns the last argument.
span.enterargs  comma-separated list of the span's enter tracer event arguments
                . For enter tracers, this is the same as evt.args. For exit tra
                cers, this is the evt.args of the corresponding enter tracer.
span.enterarg   one of the span's enter arguments, specified by name or by 0-ba
                sed offset. For enter tracer events, this is the same as evt.ar
                g. For exit tracer events, this is the evt.arg of the correspon
                ding enter event.
span.duration   delta between this span's exit tracer event and the enter trace
                r event.
span.duration.human
                delta between this span's exit tracer event and the enter event
                , as a human readable string (e.g. 10.3ms).

----------------------
Field Class: evtin

evtin.span.id   accepts all the events that are between the enter and exit trac
                ers of the spans with the given ID and are generated by the sam
                e thread that generated the tracers.
evtin.span.ntags
                accepts all the events that are between the enter and exit trac
                ers of the spans with the given number of tags and are generate
                d by the same thread that generated the tracers.
evtin.span.nargs
                accepts all the events that are between the enter and exit trac
                ers of the spans with the given number of arguments and are gen
                erated by the same thread that generated the tracers.
evtin.span.tags accepts all the events that are between the enter and exit trac
                ers of the spans with the given tags and are generated by the s
                ame thread that generated the tracers.
evtin.span.tag  accepts all the events that are between the enter and exit trac
                ers of the spans with the given tag and are generated by the sa
                me thread that generated the tracers. See the description of sp
                an.tag for information about the syntax accepted by this field.
evtin.span.args accepts all the events that are between the enter and exit trac
                ers of the spans with the given arguments and are generated by 
                the same thread that generated the tracers.
evtin.span.arg  accepts all the events that are between the enter and exit trac
                ers of the spans with the given argument and are generated by t
                he same thread that generated the tracers. See the description 
                of span.arg for information about the syntax accepted by this f
                ield.
evtin.span.p.id same as evtin.span.id, but also accepts events generated by oth
                er threads in the same process that produced the span.
evtin.span.p.ntags
                same as evtin.span.ntags, but also accepts events generated by 
                other threads in the same process that produced the span.
evtin.span.p.nargs
                same as evtin.span.nargs, but also accepts events generated by 
                other threads in the same process that produced the span.
evtin.span.p.tags
                same as evtin.span.tags, but also accepts events generated by o
                ther threads in the same process that produced the span.
evtin.span.p.tag
                same as evtin.span.tag, but also accepts events generated by ot
                her threads in the same process that produced the span.
evtin.span.p.args
                same as evtin.span.args, but also accepts events generated by o
                ther threads in the same process that produced the span.
evtin.span.p.arg
                same as evtin.span.arg, but also accepts events generated by ot
                her threads in the same process that produced the span.
evtin.span.s.id same as evtin.span.id, but also accepts events generated by the
                 script that produced the span, i.e. by the processes whose par
                ent PID is the same as the one of the process generating the sp
                an.
evtin.span.s.ntags
                same as evtin.span.id, but also accepts events generated by the
                 script that produced the span, i.e. by the processes whose par
                ent PID is the same as the one of the process generating the sp
                an.
evtin.span.s.nargs
                same as evtin.span.id, but also accepts events generated by the
                 script that produced the span, i.e. by the processes whose par
                ent PID is the same as the one of the process generating the sp
                an.
evtin.span.s.tags
                same as evtin.span.id, but also accepts events generated by the
                 script that produced the span, i.e. by the processes whose par
                ent PID is the same as the one of the process generating the sp
                an.
evtin.span.s.tag
                same as evtin.span.id, but also accepts events generated by the
                 script that produced the span, i.e. by the processes whose par
                ent PID is the same as the one of the process generating the sp
                an.
evtin.span.s.args
                same as evtin.span.id, but also accepts events generated by the
                 script that produced the span, i.e. by the processes whose par
                ent PID is the same as the one of the process generating the sp
                an.
evtin.span.s.arg
                same as evtin.span.id, but also accepts events generated by the
                 script that produced the span, i.e. by the processes whose par
                ent PID is the same as the one of the process generating the sp
                an.
evtin.span.m.id same as evtin.span.id, but accepts all the events generated on 
                the machine during the span, including other threads and other 
                processes.
evtin.span.m.ntags
                same as evtin.span.id, but accepts all the events generated on 
                the machine during the span, including other threads and other 
                processes.
evtin.span.m.nargs
                same as evtin.span.id, but accepts all the events generated on 
                the machine during the span, including other threads and other 
                processes.
evtin.span.m.tags
                same as evtin.span.id, but accepts all the events generated on 
                the machine during the span, including other threads and other 
                processes.
evtin.span.m.tag
                same as evtin.span.id, but accepts all the events generated on 
                the machine during the span, including other threads and other 
                processes.
evtin.span.m.args
                same as evtin.span.id, but accepts all the events generated on 
                the machine during the span, including other threads and other 
                processes.
evtin.span.m.arg
                same as evtin.span.id, but accepts all the events generated on 
                the machine during the span, including other threads and other 
                processes.

```
查看由 apache 以外的进程接收的传入网络连接

```bash
$ sysdig evt.type=accept and proc.name!=apache
```

```bash
$ sysdig evt.type=execve and evt.arg.ptid=bash
```

过滤器接受 `execve` 系统调用（用于执行程序），但前提是父进程名称为“bash”。`evt.arg` 和 `event.rawarg` 之间的区别在于第二个不解析 PID、FD、错误代码等，并将参数保留为原始数字形式。例如，您可以使用

```bash
$ sysdig evt.arg.res=ENOENT
```
要过滤特定的 I/O 错误代码，或者，由于错误代码是负数，这

```bash
$ sysdig " evt.rawarg.res<0 or evt.rawarg.fd<0"
```
将为您提供所有产生错误的系统调用。要获取可在过滤器中使用的所有事件及其参数的列表，请键入

```bash
> syscall(SYSCALLID ID, UINT16 nativeID)
< syscall(SYSCALLID ID)
> open()
< open(FD fd, FSPATH name, FLAGS32 flags, UINT32 mode)
> close(FD fd)
< close(ERRNO res)
> read(FD fd, UINT32 size)
< read(ERRNO res, BYTEBUF data)
> write(FD fd, UINT32 size)
< write(ERRNO res, BYTEBUF data)
> socket(FLAGS32 domain, UINT32 type, UINT32 proto)
< socket(FD fd)
> bind(FD fd)
< bind(ERRNO res, SOCKADDR addr)
> connect(FD fd)
< connect(ERRNO res, SOCKTUPLE tuple)
> listen(FD fd, UINT32 backlog)
< listen(ERRNO res)
> send(FD fd, UINT32 size)
< send(ERRNO res, BYTEBUF data)
> sendto(FD fd, UINT32 size, SOCKTUPLE tuple)
< sendto(ERRNO res, BYTEBUF data)
> recv(FD fd, UINT32 size)
< recv(ERRNO res, BYTEBUF data)
> recvfrom(FD fd, UINT32 size)
< recvfrom(ERRNO res, BYTEBUF data, SOCKTUPLE tuple)
> shutdown(FD fd, FLAGS8 how)
< shutdown(ERRNO res)
> getsockname()
< getsockname()
> getpeername()
< getpeername()
> socketpair(FLAGS32 domain, UINT32 type, UINT32 proto)
< socketpair(ERRNO res, FD fd1, FD fd2, UINT64 source, UINT64 peer)
> setsockopt()
< setsockopt()
> getsockopt()
< getsockopt()
> sendmsg(FD fd, UINT32 size, SOCKTUPLE tuple)
< sendmsg(ERRNO res, BYTEBUF data)
> sendmmsg()
< sendmmsg()
> recvmsg(FD fd)
< recvmsg(ERRNO res, UINT32 size, BYTEBUF data, SOCKTUPLE tuple)
> recvmmsg()
< recvmmsg()
> creat()
< creat(FD fd, FSPATH name, UINT32 mode)
> pipe()
< pipe(ERRNO res, FD fd1, FD fd2, UINT64 ino)
> eventfd(UINT64 initval, FLAGS32 flags)
< eventfd(FD res)
> futex(UINT64 addr, FLAGS16 op, UINT64 val)
< futex(ERRNO res)
> stat()
< stat(ERRNO res, FSPATH path)
> lstat()
< lstat(ERRNO res, FSPATH path)
> fstat(FD fd)
< fstat(ERRNO res)
> stat64()
< stat64(ERRNO res, FSPATH path)
> lstat64()
< lstat64(ERRNO res, FSPATH path)
> fstat64(FD fd)
< fstat64(ERRNO res)
> epoll_wait(ERRNO maxevents)
< epoll_wait(ERRNO res)
> poll(FDLIST fds, INT64 timeout)
< poll(ERRNO res, FDLIST fds)
> select()
< select(ERRNO res)
> select()
< select(ERRNO res)
> lseek(FD fd, UINT64 offset, FLAGS8 whence)
< lseek(ERRNO res)
> llseek(FD fd, UINT64 offset, FLAGS8 whence)
< llseek(ERRNO res)
> getcwd()
< getcwd(ERRNO res, CHARBUF path)
> chdir()
< chdir(ERRNO res, CHARBUF path)
> fchdir(FD fd)
< fchdir(ERRNO res)
> mkdir(FSPATH path, UINT32 mode)
< mkdir(ERRNO res)
> rmdir(FSPATH path)
< rmdir(ERRNO res)
> openat(FD dirfd, CHARBUF name, FLAGS32 flags, UINT32 mode)
< openat(FD fd)
> link(FSPATH oldpath, FSPATH newpath)
< link(ERRNO res)
> linkat(FD olddir, CHARBUF oldpath, FD newdir, CHARBUF newpath)
< linkat(ERRNO res)
> unlink(FSPATH path)
< unlink(ERRNO res)
> unlinkat(FD dirfd, CHARBUF name)
< unlinkat(ERRNO res)
> pread(FD fd, UINT32 size, UINT64 pos)
< pread(ERRNO res, BYTEBUF data)
> pwrite(FD fd, UINT32 size, UINT64 pos)
< pwrite(ERRNO res, BYTEBUF data)
> readv(FD fd)
< readv(ERRNO res, UINT32 size, BYTEBUF data)
> writev(FD fd, UINT32 size)
< writev(ERRNO res, BYTEBUF data)
> preadv(FD fd, UINT64 pos)
< preadv(ERRNO res, UINT32 size, BYTEBUF data)
> pwritev(FD fd, UINT32 size, UINT64 pos)
< pwritev(ERRNO res, BYTEBUF data)
> dup(FD fd)
< dup(FD res)
> signalfd(FD fd, UINT32 mask, FLAGS8 flags)
< signalfd(FD res)
> kill(PID pid, SIGTYPE sig)
< kill(ERRNO res)
> tkill(PID tid, SIGTYPE sig)
< tkill(ERRNO res)
> tgkill(PID pid, PID tid, SIGTYPE sig)
< tgkill(ERRNO res)
> nanosleep(RELTIME interval)
< nanosleep(ERRNO res)
> timerfd_create(UINT8 clockid, FLAGS8 flags)
< timerfd_create(FD res)
> inotify_init(FLAGS8 flags)
< inotify_init(FD res)
> getrlimit(FLAGS8 resource)
< getrlimit(ERRNO res, INT64 cur, INT64 max)
> setrlimit(FLAGS8 resource)
< setrlimit(ERRNO res, INT64 cur, INT64 max)
> prlimit(PID pid, FLAGS8 resource)
< prlimit(ERRNO res, INT64 newcur, INT64 newmax, INT64 oldcur, INT64 oldmax)
> fcntl(FD fd, FLAGS8 cmd)
< fcntl(FD res)
> switch(PID next, UINT64 pgft_maj, UINT64 pgft_min, UINT32 vm_size, UINT32 vm_rss, UINT32 vm_swap)
> brk(UINT64 addr)
< brk(UINT64 res, UINT32 vm_size, UINT32 vm_rss, UINT32 vm_swap)
> mmap(UINT64 addr, UINT64 length, FLAGS32 prot, FLAGS32 flags, FD fd, UINT64 offset)
< mmap(UINT64 res, UINT32 vm_size, UINT32 vm_rss, UINT32 vm_swap)
> mmap2(UINT64 addr, UINT64 length, FLAGS32 prot, FLAGS32 flags, FD fd, UINT64 pgoffset)
< mmap2(UINT64 res, UINT32 vm_size, UINT32 vm_rss, UINT32 vm_swap)
> munmap(UINT64 addr, UINT64 length)
< munmap(ERRNO res, UINT32 vm_size, UINT32 vm_rss, UINT32 vm_swap)
> splice(FD fd_in, FD fd_out, UINT64 size, FLAGS32 flags)
< splice(ERRNO res)
> ptrace(FLAGS16 request, PID pid)
< ptrace(ERRNO res, DYNAMIC addr, DYNAMIC data)
> ioctl(FD fd, UINT64 request, UINT64 argument)
< ioctl(ERRNO res)
> rename()
< rename(ERRNO res, FSPATH oldpath, FSPATH newpath)
> renameat()
< renameat(ERRNO res, FD olddirfd, CHARBUF oldpath, FD newdirfd, CHARBUF newpath)
> symlink()
< symlink(ERRNO res, CHARBUF target, FSPATH linkpath)
> symlinkat()
< symlinkat(ERRNO res, CHARBUF target, FD linkdirfd, CHARBUF linkpath)
> procexit(ERRNO status)
> sendfile(FD out_fd, FD in_fd, UINT64 offset, UINT64 size)
< sendfile(ERRNO res, UINT64 offset)
> quotactl(FLAGS16 cmd, FLAGS8 type, UINT32 id, FLAGS8 quota_fmt)
> setresuid(UID ruid, UID euid, UID suid)
< setresuid(ERRNO res)
> setresgid(GID rgid, GID egid, GID sgid)
< setresgid(ERRNO res)
> setuid(UID uid)
< setuid(ERRNO res)
> setgid(GID gid)
< setgid(ERRNO res)
> getuid()
< getuid(UID uid)
> geteuid()
< geteuid(UID euid)
> getgid()
< getgid(GID gid)
> getegid()
< getegid(GID egid)
> getresuid()
< getresuid(ERRNO res, UID ruid, UID euid, UID suid)
> getresgid()
< getresgid(ERRNO res, GID rgid, GID egid, GID sgid)
> clone()
< clone(PID res, CHARBUF exe, BYTEBUF args, PID tid, PID pid, PID ptid, CHARBUF cwd, INT64 fdlimit, UINT64 pgft_maj, UINT64 pgft_min, UINT32 vm_size, UINT32 vm_rss, UINT32 vm_swap, CHARBUF comm, BYTEBUF cgroups, FLAGS32 flags, UINT32 uid, UINT32 gid, PID vtid, PID vpid)
> fork()
< fork(PID res, CHARBUF exe, BYTEBUF args, PID tid, PID pid, PID ptid, CHARBUF cwd, INT64 fdlimit, UINT64 pgft_maj, UINT64 pgft_min, UINT32 vm_size, UINT32 vm_rss, UINT32 vm_swap, CHARBUF comm, BYTEBUF cgroups, FLAGS32 flags, UINT32 uid, UINT32 gid, PID vtid, PID vpid)
> vfork()
< vfork(PID res, CHARBUF exe, BYTEBUF args, PID tid, PID pid, PID ptid, CHARBUF cwd, INT64 fdlimit, UINT64 pgft_maj, UINT64 pgft_min, UINT32 vm_size, UINT32 vm_rss, UINT32 vm_swap, CHARBUF comm, BYTEBUF cgroups, FLAGS32 flags, UINT32 uid, UINT32 gid, PID vtid, PID vpid)
> execve()
< execve(ERRNO res, CHARBUF exe, BYTEBUF args, PID tid, PID pid, PID ptid, CHARBUF cwd, UINT64 fdlimit, UINT64 pgft_maj, UINT64 pgft_min, UINT32 vm_size, UINT32 vm_rss, UINT32 vm_swap, CHARBUF comm, BYTEBUF cgroups, BYTEBUF env)
> signaldeliver(PID spid, PID dpid, SIGTYPE sig)
> getdents(FD fd)
< getdents(ERRNO res)
> getdents64(FD fd)
< getdents64(ERRNO res)
> setns(FD fd, FLAGS32 nstype)
< setns(ERRNO res)
> flock(FD fd, FLAGS32 operation)
< flock(ERRNO res)
> cpu_hotplug(UINT32 cpu, UINT32 action)
> accept()
< accept(FD fd, SOCKTUPLE tuple, UINT8 queuepct, UINT32 queuelen, UINT32 queuemax)
> accept(INT32 flags)
< accept(FD fd, SOCKTUPLE tuple, UINT8 queuepct, UINT32 queuelen, UINT32 queuemax)
> semop(INT32 semid)
< semop(ERRNO res, UINT32 nsops, UINT16 sem_num_0, INT16 sem_op_0, FLAGS16 sem_flg_0, UINT16 sem_num_1, INT16 sem_op_1, FLAGS16 sem_flg_1)
> semctl(INT32 semid, INT32 semnum, FLAGS16 cmd, INT32 val)
< semctl(ERRNO res)
> ppoll(FDLIST fds, RELTIME timeout, SIGSET sigmask)
< ppoll(ERRNO res, FDLIST fds)
> mount(FLAGS32 flags)
< mount(ERRNO res, CHARBUF dev, FSPATH dir, CHARBUF type)
> umount(FLAGS32 flags)
< umount(ERRNO res, FSPATH name)
> semget(INT32 key, INT32 nsems, FLAGS32 semflg)
< semget(ERRNO res)
> access(FLAGS32 mode)
< access(ERRNO res, FSPATH name)
> chroot()
< chroot(ERRNO res, FSPATH path)
> tracer(INT64 id, CHARBUFARRAY tags, CHARBUF_PAIR_ARRAY args)
< tracer(INT64 id, CHARBUFARRAY tags, CHARBUF_PAIR_ARRAY args)
> setsid()
< setsid(PID res)
> mkdir(UINT32 mode)
< mkdir(ERRNO res, FSPATH path)
> rmdir()
< rmdir(ERRNO res, FSPATH path)
> notification(CHARBUF id, CHARBUF desc)
> execve()
< execve(ERRNO res, CHARBUF exe, BYTEBUF args, PID tid, PID pid, PID ptid, CHARBUF cwd, UINT64 fdlimit, UINT64 pgft_maj, UINT64 pgft_min, UINT32 vm_size, UINT32 vm_rss, UINT32 vm_swap, CHARBUF comm, BYTEBUF cgroups, BYTEBUF env, INT32 tty)
> unshare(FLAGS32 flags)
< unshare(ERRNO res)
```
##  5. 输出格式
这个单线过滤 chdir 系统调用（每次用户执行 cd 时都会调用的那些），并打印用户名和用户所在的目录。从本质上讲，它允许您在用户在文件系统中移动时跟踪她。

```bash
$ sysdig -p"user:%user.name dir:%evt.arg.path" evt.type=chdir
user:ubuntu dir:/root
user:ubuntu dir:/root/tmp
user:ubuntu dir:/root/Download
```
关于 -p 格式化语法的一些注意事项：

 - 字段必须以 % 开头
 - 您可以在字符串中添加任意文本，就像在 C printf 中所做的一样。
 - 默认情况下，仅当-p 指定的所有字段都存在于事件中时才会打印一行。但是，无论如何，您都可以在字符串前面加上 *以使其打印。在这种情况下，缺少的字段将呈现为 <NA>。

例如

```bash
$ sysdig -p"%evt.type %evt.dir %evt.arg.name" evt.type=open
open < /proc/1285/task/1399/stat
open < /proc/1285/task/1400/io
open < /proc/1285/task/1400/statm


$ sysdig -p"*%evt.type %evt.dir %evt.arg.name" evt.type=open
open > <NA>
open < /proc/1285/task/1399/stat
open > <NA>
open < /proc/1285/task/1400/io
open > <NA>
open < /proc/1285/task/1400/statm
open > <NA>


$ sysdig -A -s 65000 -p"%evt.buffer" "proc.name=cat and evt.type=write and fd.num=1"
```

打印进程的标准输出（在本例中为 cat）。请注意我们如何使用 -A 开关将结果呈现为人类可读的字符串，并使用 -s 开关来捕获超过通常每次写入的 80 个字节。谨慎使用-s，它会生成巨大的捕获文件！

```bash
显示由真实用户（即来自 bash）启动的每个程序的用户、命令名称和参数。
$ sysdig -p"%user.name) %proc.name %proc.args" evt.type=execve and evt.arg.ptid=bash
```


```bash
显示交互式用户访问的目录
$ sysdig -p"%user.name) %evt.arg.path" "evt.type=chdir"


打印对 /etc 目录的所有访问的用户、进程和文件名。
$ sysdig -p"user:%user.name process:%proc.name file:%fd.name" "evt.type=write and fd.name contains /etc/"


列出 apache 接收到的所有连接的 TCP/IP 端点信息。
$ sysdig -p"%fd.name" "proc.name=apache and evt.type=accept"
```
##  6. Chisels
Sysdig 的凿子是分析 sysdig 事件流以执行有用操作的小脚本。从本质上讲，它们使您能够使用您的 sysdig 数据做一些非常酷的事情。只需挖掘数据，然后用凿子将其塑造成漂亮的东西。得到它？惊人的！

Chisels 是用 Lua 编写的，Lua 是一种众所周知的、强大且极其高效的脚本语言。[去这里查看完整的教程](https://github.com/draios/sysdig/wiki/Chisels-User-Guide)。
