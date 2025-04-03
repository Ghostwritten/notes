
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/264c8775e68eb54bbcd037f892831919.jpeg#pic_center)



## 简介
ntp全名 network time protocol 。NTP服务器可以为其他主机提供时间校对服务
## ntp和ntpdate区别
- 两个服务都是centos自带的（centos7中不自带ntp）。ntp的安装包名是ntp；ntpdate的安装包是ntpdate。他们并非由一个安装包提供。

- ntp守护进程为ntpd，配置文件是/etc/ntp.conf

- ntpdate用于客户端的时间矫正，非NTP服务器可以不启动NTP。


## 环境准备

两台服务器，一台作为NTP服务器，另一台作为client端向服务器同步时间测试。
　　
- NTP服务器：156.0.26.6
- client端：156.0.0.27

```bash
1 # For more information about this file, see the man pages
 2 # ntp.conf(5), ntp_acc(5), ntp_auth(5), ntp_clock(5), ntp_misc(5), ntp_mon(5).
 3 
 4 driftfile /var/lib/ntp/drift  #默认即可。driftfile用来指定记录本机与上层NTP server之间的频率误差。单位是百万分之一秒。
 5 
 6 # Permit time synchronization with our time source, but do not
 7 # permit the source to query or modify the service on this system.
 8 restrict default nomodify notrap nopeer noquery 
　　　　#restrict用来管理权限控制。格式为 restrict [单个ip|网络|default] parameter
　　　　　　parameter：
　　　　　　　　ignore：拒绝所有的ntp连接
　　　　　　　　nomodify：客户端不能使用ntpc和ntpq这两个程序来更改服务器的时间参数，但客户端可以通过此主机来进行网络校时。
　　　　　　　　noquery：客户端不能使用ntpc和ntpq等命令来查询时间服务器，等于不提供网络校时服务。
　　　　　　　　notrap：不提供trap这个网络时间登陆的功能
　　　　　　　　notrust：拒绝没有认证的客户端
　　示例：restrict 156.0.26.7  nomodify
 9 
10 # Permit all access over the loopback interface.  This could
11 # be tightened as well, but to do so would effect some of
12 # the administrative functions.
13 restrict 127.0.0.1  #以下两条默认，放行本机来源
14 restrict ::1
15 
16 # Hosts on local network are less restricted.
17 #restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap
18 
19 # Use public servers from the pool.ntp.org project.
20 # Please consider joining the pool (http://www.pool.ntp.org/join.html).
21 server 0.centos.pool.ntp.org iburst #以下四条为默认，注释掉即可
22 server 1.centos.pool.ntp.org iburst
23 server 2.centos.pool.ntp.org iburst
24 server 3.centos.pool.ntp.org iburst
25 　　server：用来设置上层NTP服务器，说白了就是client向谁请求NTP时间同步。
　　　　特别注意，在内网环境中由于无法连接到内网，所以没有办法向例如国家授时服务中心210.72.145.44同步时间
　　　　只能将内网中的某台主机设置为server,用以向其他内网服务器提供NTP服务。
　　server 127.127.1.0 prefer #以本机时间作为时间服务。内网中这个配置一定要加上，否则会导致NTP服务不可用
　　　　　　　　　　　　　　　　　#prefer代表这台主机优先级最高。

26 #broadcast 192.168.1.255 autokey    # broadcast server
27 #broadcastclient            # broadcast client
28 #broadcast 224.0.1.1 autokey        # multicast server
29 #multicastclient 224.0.1.1        # multicast client
30 #manycastserver 239.255.254.254        # manycast server
31 #manycastclient 239.255.254.254 autokey # manycast client
32 
33 # Enable public key cryptography.
34 #crypto
35 
36 includefile /etc/ntp/crypto/pw
37 
38 # Key file containing the keys and key identifiers used when operating
39 # with symmetric key cryptography. 
40 keys /etc/ntp/keys ##除了restrict来限制客户端连接外，还可以通过秘钥方式来给客户端认证。
41 
42 # Specify the key identifiers which are trusted.
43 #trustedkey 4 8 42
44 
45 # Specify the key identifier to use with the ntpdc utility.
46 #requestkey 8
47 
48 # Specify the key identifier to use with the ntpq utility.
49 #controlkey 8
50 
51 # Enable writing of statistics records.
52 #statistics clockstats cryptostats loopstats peerstats
53 
54 # Disable the monitoring facility to prevent amplification attacks using ntpdc
55 # monlist command when default restrict does not include the noquery flag. See
56 # CVE-2013-5211 for more details.
57 # Note: Monitoring will not be disabled with the limited restriction flag.
58 disable monitor
```

服务器端需要修改/etc/ntp.conf，添加以下内容

```bash
server 127.127.1.0 prefer  #设置本机为NTP服务器
restrict 156.0.26.7        #允许客户端156.0.26.7向本机请求时间同步
restrict 156.0.26.0 mask 255.255.255.0 #允许客户端156.0.26.0网段的所有主机向本机请求时间同步
```

客户端需要修改/etc/ntp.conf，添加以下内容

```bash
server 156.0.26.6    #指名上层NTP服务器
restrict 156.0.26.6       #放行156.0.26.6
```

## 启动

```bash
service ntpd start
```
ntp默认监听于UDP的123端口
```bash
netstat -tunlp |grep ntp 
```

此处需要使用156.0.26.7这台Client服务器，在此主机上可以使用命令ntpstat或ntpq -p 这两个命令。

ntpstat：这条命令可以查看我们的客户端（此处为156.0.26.7）是否与server（156.0.26.6） 连接成功。
ntpq  -p：此命令可以列出当前主机的NTP和上层NTP的状态。


```bash
remote：上层NTP服务器的IP或者主机名。主要最左边的*

　　*：代表目前正在使用中的上层FTP

　　+：已经连接成功，且可以作为下一个提供时间服务的候选人。

refid：它指的是给远程服务器(156.0.26.6)提供时间同步的服务器，本机上级的上级NTP服务器。

　　为什么显示为LOCAL(0)？

　　　　由于此处为内网环境，无法去外网公用的主机同步时间，因此我们在156.0.26.6上设置的是156.0.26.6自身作为时间服务器同时，使用server 127.127.1.0 prefer指名他的上级NTP为自身。

 st：就是stratum层级。 类似于DNS，NTP是层级结构,有顶端的服务器，最多有15层。 为了减缓负荷和网络堵塞，原则上应该避免直接连接到级别为1的服务器。

when：几秒之前同步过时间

poll：在过多长时间去同步时间。

reach：已经同步时间的次数

delay：网络传输过程中的延迟，单位是10的-6次方秒。

offset：时间修正值，这是个最关键的值, 它告诉了我们本地机和服务器之间的时间差别.。单位是10的-3次方。

jitter：linux系统时间（软件时间）与BIOS硬件时间的差异时间，单位是10的-6次方秒。在主机和NTP服务器同步时间欧，可以使用 hwclock -w将系统时间写入BIOS. 
```

