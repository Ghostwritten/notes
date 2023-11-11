

----
## 1. 为什么需要滚动日志

一般情况下，无需手动旋转日志文件。Linux 系统会每隔一天（或间隔更长的时间）或根据日志文件的大小自动进行一次日志滚动。如果你需要滚动日志以释放存储空间，又或者将某一部分日志从当前的活动中分割出来，这很容易做到，具体要取决于文件滚动规则
## 2. 日志滚动过程

日志滚动的过程是这样的：在一组日志文件之中，编号最大的（最旧的）一个日志文件会被删除，其余的日志文件编号则依次增大并取代较旧的日志文件，而较新的文件则取代它作为当前的日志文件。这一个过程很容易就可以实现自动化，在细节上还能按需作出微调。

## 3. 日志滚动背景介绍
在 Linux 系统安装完成后就已经有很多日志文件被纳入到日志滚动的范围内了。另外，一些应用程序在安装时也会为自己产生的日志文件设置滚动规则。一般来说，日志滚动的配置文件会放置在 `/etc/logrotate.d`。

在日志滚动的过程中，活动日志会以一个新名称命名，例如 `log.1`，之前被命名为 log.1 的文件则会被重命名为 `log.2`，依此类推。在这一组文件中，最旧的日志文件（假如名为 `log.7`）会从系统中删除。`**日志滚动时文件的命名方式、保留日志文件的数量等参数是由 /etc/logrotate.d 目录中的配置文件决定的，*`*因此你可能会看到有些日志文件只保留少数几次滚动，而有些日志文件的滚动次数会到 7 次或更多。

例如 syslog 在经过日志滚动之后可能会如下所示（注意，行尾的注释部分只是说明滚动过程是如何对文件名产生影响的）：

```bash
$ ls -l /var/log/syslog*
-rw-r----- 1 syslog adm  128674 Mar 10 08:00 /var/log/syslog      <== 新文件
-rw-r----- 1 syslog adm 2405968 Mar  9 16:09 /var/log/syslog.1    <== 之前的 syslog
-rw-r----- 1 syslog adm  206451 Mar  9 00:00 /var/log/syslog.2.gz <== 之前的 syslog.1
-rw-r----- 1 syslog adm  216852 Mar  8 00:00 /var/log/syslog.3.gz <== 之前的 syslog.2.gz
-rw-r----- 1 syslog adm  212889 Mar  7 00:00 /var/log/syslog.4.gz <== 之前的 syslog.3.gz
-rw-r----- 1 syslog adm  219106 Mar  6 00:00 /var/log/syslog.5.gz <== 之前的 syslog.4.gz
-rw-r----- 1 syslog adm  218596 Mar  5 00:00 /var/log/syslog.6.gz <== 之前的 syslog.5.gz
-rw-r----- 1 syslog adm  211074 Mar  4 00:00 /var/log/syslog.7.gz <== 之前的 syslog.6.gz
```
你可能会发现，除了当前活动的日志和最新一次滚动的日志文件之外，其余的文件都已经被压缩以节省存储空间。这样设计的原因是大部分系统管理员都只需要查阅最新的日志文件，其余的日志文件压缩起来，需要的时候可以解压查阅，这是一个很好的折中方案。

## 4. 日志滚动工具logrotate
`logrotate`命令用于对系统日志进行`轮转、压缩和删除`，也可以将日志发送到指定邮箱。使用logrotate指令，可让你轻松管理系统所产生的记录文件。每个记录文件都可被设置成每日，每周或每月处理，也能在文件太大时立即处理。您必须自行编辑，指定配置文件，预设的配置文件存放在/etc/logrotate.conf文件中。

logrotate(选项)(参数)
选项

```bash
-?或--help：在线帮助；
-d或--debug：详细显示指令执行过程，便于排错或了解程序执行的情况；
-f或--force ：强行启动记录文件维护操作，纵使logrotate指令认为没有需要亦然；
-s<状态文件>或--state=<状态文件>：使用指定的状态文件；
-v或--version：显示指令执行过程；
-usage：显示指令基本用法。
```
logrotate参数

```bash
compress	通过gzip 压缩转储以后的日志
nocompress	不需要压缩时，用这个参数
copytruncate	用于还在打开中的日志文件，把当前日志备份并截断
nocopytruncate	备份日志文件但是不截断
create mode owner group	转储文件，使用指定的文件模式创建新的日志文件
nocreate	不建立新的日志文件
delaycompress	一起使用时，转储的日志文件到下一次转储时才压缩
nodelaycompress	覆盖 delaycompress 选项，转储同时压缩。
errors address	专储时的错误信息发送到指定的Email 地址
ifempty	即使是空文件也转储，这个是 logrotate 的缺省选项。
notifempty	如果是空文件的话，不转储
mail address	把转储的日志文件发送到指定的E-mail 地址
nomail	转储时不发送日志文件
olddir directory	转储后的日志文件放入指定的目录，必须和当前日志文件在同一个文件系统
noolddir	转储后的日志文件和当前日志文件放在同一个目录下
prerotate/endscript	在转储以前需要执行的命令可以放入这个对，这两个关键字必须单独成行
postrotate/endscript	在转储以后需要执行的命令可以放入这个对，这两个关键字必须单独成行
daily	指定转储周期为每天
weekly	指定转储周期为每周
monthly	指定转储周期为每月
rotate count	指定日志文件删除之前转储的次数，0 指没有备份，5 指保留5 个备份
tabootext [+] list	让logrotate 不转储指定扩展名的文件，缺省的扩展名是：.rpm-orig, .rpmsave, v, 和 ~
size size	当日志文件到达指定的大小时才转储，后缀MB.
```

### 4.1 logrotate配置文件位置

Linux系统默认安装logrotate工具，它默认的配置文件在：

```bash
/etc/logrotate.conf
/etc/logrotate.d/
```
logrotate.conf 才主要的配置文件，logrotate.d 是一个目录，该目录里的所有文件都会被主动的读入/etc/logrotate.conf中执行。
另外，如果 /etc/logrotate.d/ 里面的文件中没有设定一些细节，则会以/etc/logrotate.conf这个文件的设定来作为默认值。
实际运行时，Logrotate会调用配置文件/etc/logrotate.conf。
可以在/etc/logrotate.d目录里放置自定义好的配置文件，用来覆盖Logrotate的缺省值。

### 4.2 定时轮循机制

Logrotate是基于CRON来运行的，其脚本是`/etc/cron.daily/logrotate`，日志轮转是系统自动完成的。
logrotate这个任务默认放在cron的每日定时任务cron.daily下面 /etc/cron.daily/logrotate
/etc/目录下面还有cron.weekly/, cron.hourly/, cron.monthly/ 的目录都是可以放定时任务的

```bash
$ cat /etc/cron.daily/logrotate 
#!/bin/sh

/usr/sbin/logrotate -s /var/lib/logrotate/logrotate.status /etc/logrotate.conf
EXITVALUE=$?
if [ $EXITVALUE != 0 ]; then
    /usr/bin/logger -t logrotate "ALERT exited abnormally with [$EXITVALUE]"
fi
exit 0
```
实际运行时，Logrotate会调用配置文件`/etc/logrotate.conf`：
```bash
$ cat /etc/logrotate.conf
# see "man logrotate" for details
# rotate log files weekly
weekly

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file
dateext

# uncomment this if you want your log files compressed
#compress

# RPM packages drop log rotation information into this directory
include /etc/logrotate.d

# no packages own wtmp and btmp -- we'll rotate them here
/var/log/wtmp {
    monthly
    create 0664 root utmp
	minsize 1M
    rotate 1
}

/var/log/btmp {
    missingok
    monthly
    create 0600 root utmp
    rotate 1
}

# system-specific logs may be also be configured here.
```
这里的设置可以理解为Logrotate的缺省值，当然了，可以我们在/etc/logrotate.d目录里放置自己的配置文件，用来覆盖Logrotate的缺省值
nginx 常用日志切割配置

```powershell
$ cat /etc/logrotate.d/nginx
/data/log/nginx/*.log /data/log/nginx/*/*.log { # 对匹配上的日志文件进行切割

    weekly # 每周切割

    missingok     # 在日志轮循期间，任何错误将被忽略，例如“文件无法找到”之类的错误。

    rotate 6      # 保留 6 个备份

    compress     # 压缩

    delaycompress    # delaycompress 和 compress 一起使用时，转储的日志文件到下一次转储时才压缩

    notifempty     # 如果是空文件的话，不转储

    create 0644 www-data ymserver     # mode owner group 转储文件，使用指定的文件模式创建新的日志文件

    sharedscripts # 下面详细说

    prerotate # 在logrotate转储之前需要执行的指令，例如修改文件的属性等动作；必须独立成行

        if [ -d /etc/logrotate.d/httpd-prerotate ]; then \

            run-parts /etc/logrotate.d/httpd-prerotate; \

        fi \

    endscript

    postrotate  # 在logrotate转储之后需要执行的指令，例如重新启动 (kill -HUP) 某个服务！必须独立成行

        [ -s /run/nginx.pid ] && kill -USR1 `cat /run/nginx.pid`

    endscript

    su root ymserver # 轮训日志时切换设置的用户/用户组来执行（默认是root），如果设置的user/group 没有权限去让文件容用 create 选项指定的拥有者 ，会触发错误。

}
```
如果要配置一个每日0点执行切割任务，怎么做到？我们的logrotate默认每天执行时间已经写到了`/etc/cron.daily/`目录下面，而这个目录下面的任务执行时间上面也说了，在/etc/crontab里面定义了时6：25。我之前就有个这样的需求，看看下面的配置

```powershell
/data/log/owan_web/chn_download_stat/chn_app_rec.log {

    copytruncate

    # weekly 注释了 但是会继承/etc/logrorate.conf的全局变量，也是weekly

    missingok

    rotate 10

    compress

    delaycompress

    size=1000M # 大小到达size开始转存

    notifempty

    create 664 www-data ymserver

    su root

    dateext       //这个参数很重要！就是切割后的日志文件以当前日期为格式结尾，如xxx.log-20131216这样,如果注释掉,切割出来是按数字递增,即前面说的 xxx.log-1这种格式

    compress      //是否通过gzip压缩转储以后的日志文件，如xxx.log-20131216.gz ；如果不需要压缩，注释掉就行

}
```

然后去root的crontab配置一个0点执行的任务

```powershell
$ cat /etc/crontab
wwwadm@host:/etc/logrotate.d$ sudo crontab -l -u root

0 0 * * * /usr/sbin/logrotate /etc/logrotate.d/web_roteate -fv  >/tmp/logro.log 2>&1
```

因为logrotate的切割周期是weekly，每次切割都是根据上一个切割的时间来进行，如果距离上一次有一周时间，就会切割，但是我们设置了crontab的每天切割，既不会进入/etc/cron.daily/的每日切割，也不会每周切割。这样就能完美定制自己想要的切割日志时间

### 4.3 值得注意的一个配置是：copytruncate


copytruncate 如果没有这个选项的话，操作方式：是将原log日志文件，移动成类似log.1的旧文件， 然后创建一个新的文件。 如果设置了，操作方式：拷贝原日志文件，并且将其变成大小为0的文件。

 

区别是如果进程,比如nginx 使用了一个文件写日志，没有copytruncate的话，切割日志时， 把旧日志log->log.1 ，然后创建新日志log。这时候nginx 打开的文件描述符依然时log.1，由没有信号通知nginx 要换日志描述符，所以它会继续向log.1写日志，这样就不符合我们的要求了。 因为我们想切割日志后，nginx 自动会向新的log 文件写日志，而不是旧的log.1文件

 

**解决方法有两个：**

1.向上面的nginx 切割日志配置，在`postrotate`里面写个脚本

postrotate  # 在logrotate转储之后需要执行的指令，例如重新启动 (kill -HUP) 某个服务！必须独立成行

```powershell
[ -s /run/nginx.pid ] && kill -USR1 `cat /run/nginx.pid`

endscript
```

这样就是发信号给nginx ,让nginx 关闭旧日志文件描述符，重新打开新的日志文件描述，并写入日志

2.使用copytruncate参数，向上面说的，配置了它以后，操作方式是把log 复制一份 成为log.1，然后清空log的内容，使大小为0，那此时log依然时原来的旧log，对进程（nginx）来说，依然打开的是原来的文件描述符，可以继续往里面写日志，而不用发送信号给nginx

 

copytruncate这种方式操作的时候， 拷贝和清空之间有一个时间差，可能会丢失部分日志数据。


nocopytruncate 备份日志文件不过不截断。
## 5. 手动日志滚动

```bash
$ sudo logrotate -f /etc/logrotate.d/rsyslog
```
值得一提的是，logrotate 命令使用 /etc/logrotate.d/rsyslog 这个配置文件，并通过了 -f 参数实行“强制滚动”。因此，整个过程将会是：

◈ 删除 syslog.7.gz，
◈ 将原来的 syslog.6.gz 命名为 syslog.7.gz，
◈ 将原来的 syslog.5.gz 命名为 syslog.6.gz，
◈ 将原来的 syslog.4.gz 命名为 syslog.5.gz，
◈ 将原来的 syslog.3.gz 命名为 syslog.4.gz，
◈ 将原来的 syslog.2.gz 命名为 syslog.3.gz，
◈ 将原来的 syslog.1.gz 命名为 syslog.2.gz，
◈ 但新的 syslog 文件不一定必须创建。
你可以按照下面的几条命令执行操作，以确保文件的属主和权限正确：

```bash
$ sudo touch /var/log/syslog
$ sudo chown syslog:adm /var/log/syslog
$ sudo chmod 640 /var/log/syslog
```
你也可以把以下这一行内容添加到 `/etc/logrotate.d/rsyslog` 当中，由 logrotate 来帮你完成上面三条命令的操作：**create 0640 syslog adm**

```bash

/var/log/syslog
{
rotate 7
daily
missingok
notifempty
create 0640 syslog adm           <==
delaycompress
compress
postrotate
/usr/lib/rsyslog/rsyslog-rotate
endscript
}
```
下面是手动滚动记录用户登录信息的 wtmp 日志的示例。由于 `/etc/logrotate.d/wtmp` 中有 `rotate 2` 的配置，因此系统中只保留了两份 wtmp 日志文件。

滚动前：

```bash
$ ls -l wtmp*
-rw-r----- 1 root utmp  1152 Mar 12 11:49 wtmp
-rw-r----- 1 root utmp   768 Mar 11 17:04 wtmp.1
```

执行滚动命令：

```bash
$ sudo logrotate -f /etc/logrotate.d/wtmp
```

滚动后：

```bash
$ ls -l /var/log/wtmp*
-rw-r----- 1 root utmp     0 Mar 12 11:52 /var/log/wtmp
-rw-r----- 1 root utmp  1152 Mar 12 11:49 /var/log/wtmp.1
-rw-r----- 1 root adm  99726 Feb 21 07:46 /var/log/wtmp.report
```

需要知道的是，无论发生的日志滚动是自动滚动还是手动滚动，最近一次的滚动时间都会记录在 logrorate 的状态文件中。

```bash
$ grep wtmp /var/lib/logrotate/status
"/var/log/wtmp" 2020-3-12-11:52:57
```

参考链接：
[https://www.cnblogs.com/276815076/p/7053640.html](https://www.cnblogs.com/276815076/p/7053640.html)
[https://www.networkworld.com/article/3531969/manually-rotating-log-files-on-linux.html](https://www.networkworld.com/article/3531969/manually-rotating-log-files-on-linux.html)
