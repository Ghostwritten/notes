#  Linux Command dig 查询DNS



-------------
## 1. 简介

dig，和nslookup作用有些类似，都是DNS查询工具

什么是CNAME：一个域名可以有两种类型的指向，如果一个 域名指向 称为一个 记录 （Record）的话，那么就有两种 记录类型 （Record Type），分别是：

A记录 ：指向一个IP地址
CNAME ：指向一个其他的域名

*******************************

## 2. 组成

输出结果大致分成4个部分，实际上可能还包括更多的内容，总共会有以下6个部分：

 - Header : 包括软件版本，全局变量以及除消息头以外的其他部分的信息，比如上例中，显示有1个QUERY，2个ANSWER
 - QUESTION SECTION : 请求参数信息，也就是你的输入
 - ANSWER SECTION : 从DNS查询到的信息，也就是输出，显示 i.zhouliang.pro 是CNAME，指向
   mydomain.lofter.com ，而后者是一个A记录，指向一个IP地址
 - AUTHORITY SECTION : 包含DNS域名服务器的授权信息，上例中不包含这一部分，如果用这个命令就可以看到 dig
   @ns1.redhat.com redhat.com ，这里的 @ 符号用于指定查询所使用的DNS服务器
 - ADDITIONAL SECTION : 包含AUTHORITY SECTION中的域名服务器的IP地址，同样，上例中也不包含这一部分
 - Stats section : 最下方的一部分，显示了查询时间等额外信息

如果你设置的dnsserver是一个域名，那么dig会首先通过**默认的上连DNS服务器去查询对应的IP地址**，然后再以设置的dnsserver为上连DNS服务器。
没有设置@dnsserver，那么dig就会依次使用**/etc/resolv.conf**里的地址作为上连DNS服务器。
对querytype有所了解，你可以设置**A/AAAA/PTR/MX/ANY**等值，默认是查询A记录。

```c
+nocomments     – 不显示注释
+noauthority    – 不显示AUTHORITY SECTION
+noadditional   – 不显示ADDITIONAL SECTION
+nostats        – 不显示Stats section
+noanswer       – 不显示ANSWER SECTION
+noall          - 不显示所有的信息，一般会这样用 dig zhouliang.pro +noall +answer
和上面参数对应还有 +comments ， +answer 等，后文有示例，此处不赘述。另外，还有如下两个参数需要了解：

+short    - 显示简短的信息
-t       指定查询的记录类型，可以是CNAME、A、MX、NS，分别表示CNAME、A记录、MX记录、DNS服务器，默认是A
-x       表示反向查找，也就是根据IP地址查找域名
-c       可以设置协议类型（class），包括IN(默认)、CH和HS
-f       dig支持从一个文件里读取内容进行批量查询
-4和-6   用于设置仅适用哪一种作为查询包传输协议，分别对应着IPv4和IPv6
-q       显式设置你要查询的域名
```
## 3. 实例
```bash
$ dig www.oolec.com
```

即查询域名的A记录，查询的dns服务器将采用系统配置的服务器，即`/etc/resovle.conf` 中的。

如果要查询其他类型的记录，比如MX，CNAME，NS，PTR等，只需将类型加在命令后面即可
```bash
$ dig www.oolec.com mx
$ dig www.oolec.com ns
```
此外，如果你是一个系统管理员，部署好了一台dns服务器之后想对它进行解析测试，就必须要显式指定待测试的dns服务器地址了，例如
```bash
dig @202.106.0.20 www.oolec.com a
```
默认情况下dig将采用udp协议进行查询，如果要采用tcp方式，可以加上 +tcp参数
```bash
dig www.oolec.com a +tcp
```
另外一个重要的功能是+trace参数，使用这个参数之后将显示从根域逐级查询的过程
```bash
dig www.oolec.com a +trace
```
google-DNS来查baidu.com的A记录

```bash
dig @8.8.8.8 www.baidu.com A     //命令格式为dig @dnsserver name querytype
```
```bash
$ cat querylist 
www.baidu.com
www.sohu.com
$ dig -f querylist -c IN -t A
```
.查看域名

```bash
$ dig i.zhouliang.pro +noall +anwser

; <<>> DiG 9.8.3-P1 <<>> i.zhouliang.pro +noall +answer
;; global options: +cmd
i.zhouliang.pro.            10034    IN    CNAME    mydomain.lofter.com.
mydomain.lofter.com.        9183     IN    A        54.248.125.234
```

第一行是CNAME，先将 i.zhouliang.pro 解析成 mydomain.lofter.com ，第二行是A记录，将 mydomain.lofter.com 解析成IP地址。这是一个完整的域名解析过程


2.查找域名的MX记录：

```bash
$ dig zhouliang.pro -t MX +short
10 mxdomain.qq.com.
```

从输出可以看出，我用了QQ提供的域名邮箱服务


3.查找域名对应的CNAME：

```bash
$ dig i.zhouliang.pro -t CNAME +short
mydomain.lofter.com.
```

4.根据IP地址反向查找域名

```bash
$ dig -x 8.8.8.8 +short

; <<>> DiG 9.8.3-P1 <<>> -x 8.8.8.8 +noall +answer
;; global options: +cmd
8.8.8.8.in-addr.arpa.    79605    IN    PTR    google-public-dns-a.google.com.
```

5.查询域名的解析DNS服务器地址

```bash
$ dig www. ns +short
```

参考：

 - [How to Use Linux dig Command (DNS Lookup)](https://phoenixnap.com/kb/linux-dig-command-examples)
 - [Dig Command in Linux (DNS Lookup)](https://linuxize.com/post/how-to-use-dig-command-to-query-dns-in-linux/)
 - [dig(1) - Linux man page](https://linux.die.net/man/1/dig)

