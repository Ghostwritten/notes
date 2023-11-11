参考链接：
[sparkdev](https://www.cnblogs.com/sparkdev/p/8795141.html)

### 1. journalctl 常用命令
查看所有日志（默认情况下 ，只保存本次启动的日志）
  

```bash
journalctl 
```

查看内核日志（不显示应用日志）

```bash
 journalctl -k 
```
查看系统本次启动的日志

```bash
 journalctl -b
```

查看上一次启动的日志（需更改设置）
    在该[Journal]部分下，将该Storage=选项设置为“persistent”以启用持久记录：

  

```bash
  vim    /etc/systemd/journald.conf
    . . .
    [Journal]
    Storage=persistent
```

在您的服务器上启用了保存以前的引导时，journalctl提供了一些命令来帮助您将引导作为分割单位来使用。要查看journald知道的引导，请使用以下--list-boots选项journalctl：

```bash
$ journalctl --list-boots 
    -1 00d066e11cb3412a912cb804cee123b5 Thu 2018-02-22 17:01:47 CST—Thu 2018-02-22 17:09:15 CST
     0 63f75abbe94c4087bc2cc3cdb3b57100 Thu 2018-02-22 17:09:10 CST—Thu 2018-02-22 17:10:19 CST
```

这将为每次启动显示一行。第一列是启动的偏移量，可用于轻松引用启动journalctl。如果您需要绝对参考，则启动ID位于第二列。您可以通过在结束时列出的两个时间规范来指出引导会话引用的时间。

### 2. 使用时间限制来过滤日记数据
例如，要查看上一次启动的日志，请使用-1带有该-b标志的相对指针：

```bash
$ journalctl -b -1
```

查看指定时间的日志
可以使用--since和--until选项过滤任意时间限制，这些限制分别显示给定时间之前或之后的条目。
例如： #"显示2017年10月30号，18点10分30秒到当前时间之间的所有日志信息"

```bash
$ journalctl --since="2017-10-30 18:10:30"
```

另外，journal还能够理解部分相对值及命名简写。例如，大家可以使用“yesterday”、“today”、“tomorrow”或者“now”等表达。另外，我们也可以使用“-”或者“+”设定相对值，或者使用“ago”之前的表达。
例如获取昨天的日志如下：

```bash
journalctl –since yesterday
```

获取某一个时间段到当前时间的前一个小时的日志

```bash
journalctl --since 09:00 --until "1 hour ago" 
```

获取当前时间的前20分钟的日志

```bash
journalctl --since "20 min ago"
```

获取某一天到某一个时间段的日志信息

```bash
journalctl --since "2017-01-10" --until "2017-01-11 03:00" 
```

### 3. 根据服务或组件来进行过滤
-u选项来过滤。

例如，查看httpd服务的日志信息

```bash
$ journalctl -u httpd.service 
-- Logs begin at Thu 2018-02-22 17:01:47 CST, end at Thu 2018-02-22 17:30:01 CST. --
Feb 22 17:29:27 centos7.localdomain systemd[1]: Starting The Apache HTTP Server...
Feb 22 17:29:27 centos7.localdomain httpd[1610]: AH00558: httpd: Could not reliably determine t
Feb 22 17:29:28 centos7.localdomain systemd[1]: Started The Apache HTTP Server.
```

也可以查看httpd服务当天的运行状况

```bash
journalctl -u httpd.service --since today
```

按进程、用户或者群组ID
由于某些服务当中包含多个子进程，因此如果我们希望通过进程ID实现查询，也可以使用相关过滤机制。
这里需要指定_PID字段。例如，如果PID为8088，则可输入：

```bash
journalctl _PID=8088
```

有时候我们可能希望显示全部来自特定用户或者群组的日志条目，这就需要使用_UID或者_GID。例如，如果大家的Web服务器运行在www-data用户下，则可这样找到该用户ID：

```bash
$ id -u www-data

33

```

接下来，我们可以使用该ID返回过滤后的journal结果：

```bash
journalctl _UID=33 --since today
```


journalctl配合-p选项显示特定优先级的信息，从而过滤掉优先级较低的信息。
例如，只显示错误级别或者更高的日志条目：

```bash
$ journalctl -p err -b
    -- Logs begin at Thu 2018-02-22 17:01:47 CST, end at Thu 2018-02-22 17:40:02 CST. --
    Feb 22 17:09:10 centos7.localdomain kernel: sd 0:0:0:0: [sda] Assuming drive cache: write throu
    Feb 22 17:09:12 centos7.localdomain kernel: piix4_smbus 0000:00:07.3: SMBus Host Controller not
    Feb 22 17:09:15 centos7.localdomain rsyslogd[593]: error during parsing file /etc/rsyslog.conf,
    Feb 22 17:09:47 centos7.localdomain pulseaudio[1232]: [alsa-sink-ES1371/1] alsa-sink.c: ALSA wo
    Feb 22 17:09:47 centos7.localdomain pulseaudio[1232]: [alsa-sink-ES1371/1] alsa-sink.c: Most li
    Feb 22 17:09:47 centos7.localdomain pulseaudio[1232]: [alsa-sink-ES1371/1] alsa-sink.c: We were
    Feb 22 17:09:48 centos7.localdomain spice-vdagent[1274]: Cannot access vdagent virtio channel /
    lines 1-8/8 (END)
```

这将只显示被标记为错误、严重、警告或者紧急级别的信息。Journal的这种实现方式与标准syslog信息在级别上是一致的。大家可以使用优先级名称或者其相关量化值。以下各数字为由最高到最低优先级：

0: emerg
1: alert
2: crit
3: err
4: warning
5: notice
6: info
7: debug

例如：

```bash
$ journalctl -p 3 -b
    -- Logs begin at Thu 2018-02-22 17:01:47 CST, end at Thu 2018-02-22 17:50:01 CST. --
    Feb 22 17:09:10 centos7.localdomain kernel: sd 0:0:0:0: [sda] Assuming drive cache: write throu
    Feb 22 17:09:12 centos7.localdomain kernel: piix4_smbus 0000:00:07.3: SMBus Host Controller not
    Feb 22 17:09:15 centos7.localdomain rsyslogd[593]: error during parsing file /etc/rsyslog.conf,
```

### 4. 修改journal显示内容

分页显示（默认）或者改为正常标准输出
分页显示，其中插入省略号以代表被移除的信息，使用–no-full选

```bash
journalctl --no-full
```

. . .


大家也可以要求其显示全部信息，无论其是否包含不可输出的字符。具体方式为添加-a标记：
journalctl -a
默认情况下，journalctl会在pager内显示输出结果以便于查阅。如果大家希望利用文本操作工具对数据进行处理，则可能需要使用标准格式。在这种情况下，我们需要使用–no-pager选项：

```bash
journalctl --no-pager
```

这样就可以用一些工具过滤出自己感兴趣的信息了
### 5. 输出格式
如果大家需要对journal条目进行处理，则可能需要使用更易使用的格式以简化数据解析工作。幸运的是，journal能够以多种格式进行显示，只须添加-o选项加格式说明即可。
例如，我们可以将journal条目输出为JSON格式：

```bash
$ journalctl -b -u httpd -o json
    { "__CURSOR" : "s=8fa6a8a1c6264c7b938e4d23584ae602;i=149d;b=63f75abbe94c4087bc2cc3cdb3b57100;m=46edf6e6;t=565c9ae1d38f7;x=b3a1eaebceb26d5b", "__REALTIME_TIMESTAMP" : "1519291767535863", "__MONOTONIC_TIMESTAMP"
    { "__CURSOR" : "s=8fa6a8a1c6264c7b938e4d23584ae602;i=149e;b=63f75abbe94c4087bc2cc3cdb3b57100;m=46f3506d;t=565c9ae22927d;x=91ef081943191196", "__REALTIME_TIMESTAMP" : "1519291767886461", "__MONOTONIC_TIMESTAMP"
    { "__CURSOR" : 
```


以下为可用于显示的各类格式：
  

```bash
    cat: 只显示信息字段本身。
    export: 适合传输或备份的二进制格式。
    json: 标准JSON，每行一个条目。
    json-pretty: JSON格式，适合人类阅读习惯。
    json-sse: JSON格式，经过打包以兼容server-sent事件。
    short: 默认syslog类输出格式。
    short-iso: 默认格式，强调显示ISO 8601挂钟时间戳。
    short-monotonic: 默认格式，提供普通时间戳。
    short-precise: 默认格式，提供微秒级精度。
    verbose: 显示该条目的全部可用journal字段，包括通常被内部隐藏的字段。
```

活动进程监控
Journalctl命令还能够帮助管理员以类似于tail的方式监控活动或近期进程。这项功能内置于journalctl当中，允许大家在无需借助其它工具的前提下实现访问。
显示近期日志
要显示特定数量的记录，大家可以使用-n选项，类似为tail -n功能。默认情况下只显示最后发生的10条日志，但是也可以指定。
例如：

```bash
$ journalctl -n20
    -- Logs begin at Thu 2018-02-22 17:01:47 CST, end at Thu 2018-02-22 18:20:01 CST. --
    Feb 22 17:40:01 centos7.localdomain systemd[1]: Started Session 5 of user root.
    Feb 22 17:40:02 centos7.localdomain systemd[1]: Starting Session 5 of user root.
```

### 6. 追踪日志
要主动追踪当前正在编写的日志，大家可以使用-f标记。同样功能类似为tail -f，只要不终止，会一直监控

```bash
$ journalctl -f
```

### 7. Journal维护
存储这么多数据当然会带来巨大压力，因此我们还需要了解如何清理部分陈旧日志以释放存储空间。
查看当前日志占用磁盘的空间的总大小

```bash
$ journalctl --disk-usage 
Archived and active journals take up 8.0M on disk.
```


指定日志文件最大空间

```bash
journalctl --vacuum-size=1G
```

指定日志文件保存多久

```bash
journalctl --vacuum-time=1years
```

### 8. journalctl相关配置
大家可以配置自己的服务器以限定journal所能占用的最高容量。要实现这一点，我们需要编辑/etc/systemd/journald.conf文件。
以下条目可用于限定journal体积的膨胀速度：
  

```bash
  SystemMaxUse=: 指定journal所能使用的最高持久存储容量。
    SystemKeepFree=: 指定journal在添加新条目时需要保留的剩余空间。
    SystemMaxFileSize=: 控制单一journal文件大小，符合要求方可被转为持久存储。
    RuntimeMaxUse=: 指定易失性存储中的最大可用磁盘容量（/run文件系统之内）。
    RuntimeKeepFree=: 指定向易失性存储内写入数据时为其它应用保留的空间量（/run文件系统之内）。
    RuntimeMaxFileSize=: 指定单一journal文件可占用的最大易失性存储容量（/run文件系统之内）。
```
    通过设置上述值，大家可以控制journald对服务器空间的消耗及保留方式。





