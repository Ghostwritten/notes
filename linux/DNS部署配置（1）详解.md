


--------------------

## 1. 什么是DNS？
 DNS( Domain Name System)是“域名系统”的英文缩写，是一种组织成域层次结构的计算机和网络服务命名系统，使用的是UDP协议的53号端口，它用于TCP/IP网络，它所提供的服务是用来将主机名和域名转换为IP地址的工作


## 2. DNS服务基本概念
  在介绍DNS服务器工作原理之前我们先来了解几个DNS相关的概念：
  1. `FQDN：Full Qualified Domain Name`，完全限定域名，即每个域在全球网络都是唯一的；另外值得提到的一点是域并不是指诸如`www.google.com`这样的域名，而google.com才是域；
2. 域的分类
（1）根域：标识为(.)点 ,全球13组根域名服务器以英文字母A到M依序命名，域名格式为“字母.root-servers.net”。其中有11个是以任播技术在全球多个地点设立镜像站。比如中国大陆在北       京有两台编号为L的镜像，编号为F、I、J的镜像各一台，共5台；香港有编号为D、J的镜像各2台，编号为A、F、I、L的镜像各一台，共8台；台湾则有编号为F、I、J各一台，共3台。
  （2）顶级域：顶级域（Top Level Domain，简称TLD）分为三类
     1> 通用顶级域：诸如 .com(商业机构) .org(非营利性组织) .net(网络服务机构)等
     2> 国家顶级域：诸如 .cn(中国) .uk(英国) .us(美国) .jp(小日本)
     3> 反向域（基础建设顶级域）：.arpa,即从IP到FQDN的反向解析
  3. DNS服务器查询的类型：
   （1）递归：客户端仅发出一次请求，让DNS服务器去查询返回结果；
   （2）迭代：要发出多次请求去分别查询不同的DNS服务器；
  4. DNS名称解析方式：
   （1）正向解析：即将FQDN转化为IP。
   （2）反向解析：即将IP转化为FQDN。
  
  5. DNS服务器类型：
   （1）主DNS服务器：负责解析至少一个域。
   （2）辅助（从）DNS服务器：负责解析至少一个，是主DNS服务器的辅助。
   （3）缓存DNS服务器：不负责解析域，只是缓存域名解析结果。
  6. DNS返回的结果类型：
   （1）肯定答案：查询的域存在，会被缓存下来。
   （2）否定答案：不存在查询的域名，因此不存在与其查询的域名对应的IP；会被缓存下来。
   （3）权威答案：所查询的域名的结果是由负责解析这个域的DNS服务器所返回的答案。
   （4）非权威答案：在缓存中查询的结果。
  7. DNS的监听端口：tcp的53号端口，udp的53号端口。
## 3. DNS解析原理
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/96a9eefb847a1099110aa23b5b2e89cd.png)
 

（1）当用户在浏览器中输入www.qq.com域名访问该网站时，操作系统会先检查自己本地的hosts文件是否有这个网址映射关系，如果有，就先调用这个IP地址映射，完成域名解析。 
 
（2）如果hosts里没有这个域名的映射，则查找本地DNS解析器缓存，是否有这个网址映射关系，如果有，直接返回，完成域名解析。 
 
（3）如果hosts与本地DNS解析器缓存都没有相应的网址映射关系，首先会找TCP/ip参数中设置的首选DNS服务器，在此我们叫它本地DNS服务器，此服务器收到查询时，如果要查询的域名，包含在本地配置区域资源中，则返回解析结果给客户机，完成域名解析，此解析具有权威性。 
 
（4）如果要查询的域名，不由本地DNS服务器区域解析，但该服务器已缓存了此网址映射关系，则调用这个IP地址映射，完成域名解析，此解析不具有权威性。 
 
（5）如果本地DNS服务器本地区域文件与缓存解析都失效，则根据本地DNS服务器的设置（是否设置转发器）进行查询，如果未用转发模式，本地DNS就把请求发至13台根DNS，根DNS服务器收到请求后会判断这个域名(.com)是谁来授权管理，并会返回一个负责该顶级域名服务器的一个IP。本地DNS服务器收到IP信息后，将会联系负责.com域的这台服务器。这台负责.com域的服务器收到请求后，如果自己无法解析，它就会找一个管理.com域的下一级DNS服务器地址(qq.com)给本地DNS服务器。当本地DNS服务器收到这个地址后，就会找qq.com域服务器，重复上面的动作，进行查询，直至找到www.qq.com主机。 
 
（6）如果用的是转发模式，此DNS服务器就会把请求转发至上一级DNS服务器，由上一级服务器进行解析，上一级服务器如果不能解析，或找根DNS或把转请求转至上上级，以此循环。不管是本地DNS服务器用是是转发，还是根提示，最后都是把结果返回给本地DNS服务器，由此DNS服务器再返回给客户机
## 4. DNS配置文件

```bash
/etc/named.conf          主配置文件 服务器主要运行参数
/etc/named.rfc1912.zones  区域文件，主要指定要解析哪个域名
/var/named/xxx.xx        数据文件，用来正向和反向的解析
执行程序：/usr/sbin/named
系统服务：named
默认端口：TCP/UDP 53
运行时的虚拟环境：/var/named/chroot
软件包：bind（域名服务包）；bind-chroot（提供虚拟机支持）
bind-chroot是bind的一个功能,使 bind可以在一个chroot的模式下运行.也就是说,bind运行时的/(根)目录,并不是系统真正的/(根)目录,只是系统中的一个子目录而已.这 样做的目的是为了提高安全性.因为在chroot的模式下,bind可以访问的范围仅限于这个子目录的范围里,无法进一步提升,进入到系统的其他目录中。 bind的默认启动方式就是chroot方式

```

## 5. 资源记录的类型
### （1）A记录（Address）正向解析
A记录是将一个主机名（全称域名FQDN）和一个IP地址关联起来。这也是大多数客户端程序默认的查询类型。
### （2）PTR记录（Pointer）反向解析
PTR记录将一个IP地址对应到主机名（全称域名FQDN）。这些记录保存在in-addr.arpa域中。
### （3）CNAME记录(Canonical Name)别名
别名记录，也称为规范名字(Canonical Name)。这种记录允许您将多个名字映射到同一台计算机。
### （4）MX记录（Mail eXchange）
MX记录是邮件交换记录，它指向一个邮件服务器，用于电子邮件系统发邮件时根据 收信人的地址后缀来定位邮件服务器。MX记录也叫做邮件路由记录，用户可以将该域名下的邮件服务器指向到自己的mail server上，然后即可自行操控所有的邮箱设置。
      当有多个MX记录（即有多个邮件服务器）时，则需要设置数值来确定其优先级。通过设置优先级数字来指明首选服务器，数字越小表示优先级越高。
### （5）NS记录（Name Server）
NS（Name Server）记录是域名服务器记录，也称为授权服务器，用来指定该域名由哪个DNS服务器来进行解析。
将网站的NS记录指向到目标地址，在设置NS记录的同时还需要设置目标网站的指向，否则NS记录将无法正常解析
NS记录优先于A记录。即，如果一个主机地址同时存在NS记录和A记录，则A记录不生效。
 
 
## 6. DNS服务的配置方法
提示：本次DNS环境配置是在centos7.x中进行的。
（1）配置前的准备工作
### 6.1  环境配置
配置好本地光盘yum源或者配置网络yum源
设置好防火墙开放UDP的53端口，或者直接关闭防火墙
防火墙永久关闭：

```bash
$ /etc/init.d/iptables stop
$ service iptables stop
```
关闭selinux
selinux临时关闭：

```bash
$ setenforce 0
```

selinux永久关闭：

```bash
$ sed –i“7s/enforcing/disabled/g”/etc/selinux/config
```

### 6.2 安装bind软件

```bash
$  yum -y install bind
```

 
### 6.3 修改主配置文件/etc/named.conf两个地方为{any}

```bash
$ vim /etc/named.conf
options {
        listen-on port 53 { any; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        allow-query     { any; };
 
```

### 6.4 修改区域文件/etc/named.rfc1912.zones
配置文件说明：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9d59d3bc9b39a5ca798fbfdbb0dc0d41.png)
 

```bash
$ vim /etc/named.rfc1912.zones
zone "long.com" IN {
        type master;
        file "named.zheng";   正向解析文件名（名称可以自定义）
        allow-update { none; };
};
 
zone "115.168.192.in-addr.arpa" IN {
        type master;
        file "named.fan";    反向解析文件名（名称可以自定义）
        allow-update { none; };
};
```

提示：上面的配置文件可以只保留两个地方，一个正向解析域名，一个反向解析域名，其余都可以删除

```bash
$ cd /var/named/
$ ls
data  dynamic  named.ca  named.empty  named.localhost  named.loopback  slaves
```

 
生成上面的`/etc/named.rfc1912.zones`配置文件中指定的正反解析文件

```bash
$ cp -a named.localhost named.zheng
$ cp -a named.loopback named.fan
```

 
### 6.5 修改上面的正向解析文件和反向解析文件
解析文件named.*的说明：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d18e27782a3f7f62baa8e6030621c072.png)
 
正向解析文件named.zheng的修改

```bash
$  vim named.zheng
$TTL 1D
@       IN SOA  long.com. rname.invalid. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      dns.long.com.
dns     A       192.168.115.120    dns服务器的IP地址
www     A       192.168.115.130   www服务器的IP地址
        AAAA    ::1
~                   
```

 
反向解析文件named.fan的修改:

```bash
$ vim named.fan
$TTL 1D
@       IN SOA  long.com. rname.invalid. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      dns.long.com.
120     PTR     dns.long.com.
130     PTR     www.long.com.
```

 

```bash
$ systemctl start named.service    启动dns服务
```

 ### 客户端验证
找一个客户端，把DNS修改成成我们的DNS 服务器IP地址，然后保存退出，重启网卡

```bash
$ systemctl restart network.service
$ cat /etc/resolv.conf   查看DNS已经修改成我们搭建的了
# Generated by NetworkManager
search long.com
nameserver 192.168.115.120
```

 

```bash
$ yum install bind-utils   安装nslookup命令的软件包
$ nslookup   
> 192.168.115.130     查看用ip能否解析成域名
Server:  192.168.115.120
Address: 192.168.115.120#53
 
130.115.168.192.in-addr.arpa name = www.long.com.
> www.long.com      查看用域名能否解析成IP地址
Server:  192.168.115.120
Address: 192.168.115.120#53
 
Name: www.long.com
Address: 192.168.115.130
```
### 6.6 网页解析
再创建一台虚拟机作为网页服务器，把IP地址修改为我们DNS服务器解析的IP地址即92.168.115.130，然后安装httpd服务

```bash
$ yum -y install httpd
$ systemctl start httpd.service
 
```

在客户机上输入网址解析即可

```bash
$ yum -y install elinks
$ elinks www.long.com   
```

输入这个地址后就会弹出下面的网页服务窗口
 
到了这里我们的DNS服务器就已经搭建完成了。


