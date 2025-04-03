#  python 模块 dnspython



----
## 1 dnspython模块介绍
`dnspython（http://www.dnspython.org/`）是Python实现的一个DNS 工具包，它支持几乎所有的记录类型，可以用于查询、传输并动态更新 `ZONE`信息，同时支持`TSIG`（事务签名）验证消息和`EDNS0`（扩展 DNS）。在系统管理方面，我们可以利用其查询功能来实现DNS服务监 控以及解析结果的校验，可以代替`nslookup`及`dig`等工具，轻松做到与现 有平台的整合，下面进行详细介绍。
## 2 安装

```bash
[root@localhost ipy]# pip install dnspython
Downloading/unpacking dnspython
  Downloading dnspython-1.16.0-py2.py3-none-any.whl (188kB): 188kB downloaded
Installing collected packages: dnspython
Successfully installed dnspython
Cleaning up...
```
## 3. 模块域名解析方法详解
`dnspython`模块提供了大量的DNS处理方法，最常用的方法是域名 查询。dnspython提供了一个DNS解析器类——`resolver`，使用它的query 方法来实现域名的查询功能。`query`方法的定义如下：

```bash
query（self， qname， rdtype=1， rdclass=1， tcp=False， source=None，
raise_on_no_answer=True， source_port=0）
```

```bash
qname参数为查询的域名。

rdtype参数用来指定RR资源的类型， 常用的有以下几种： ·

A记录，将主机名转换成IP地址； ·

MX记录，邮件交换记录，定义邮件服务器的域名； ·

CNAME记录，指别名记录，实现域名间的映射； ·

NS记录，标记区域的域名服务器及授权子域； ·

PTR记录，反向解析，与A记录相反，将IP转换成主机名； ·

SOA记录，SOA标记，一个起始授权区的定义。

rdclass参数用于指定网络类型，可选的值有IN、CH与HS，其中IN为默 认，使用最广泛。

tcp参数用于指定查询是否启用TCP协议，默认为 False（不启用）。

source与source_port参数作为指定查询源地址与端 口，默认值为查询设备IP地址和0。

raise_on_no_answer参数用于指定当 查询无应答时是否触发异常，默认为True。
```
## 4 常见解析类型示例说明：
常见的DNS解析类型包括`A、MX、NS、CNAME`等。利用 dnspython的`dns.resolver.query`方法可以简单实现这些DNS类型的查询， 为后面要实现的功能提供数据来源，比如对一个使用DNS轮循业务的域 名进3行可用性监控，需要得到当前的解析结果。
### 4.1 A记录 实现A记录查询方法源码

```c

[root@localhost dnspython]# cat dnspython1.py
#/usr/bin/env python
#_*_coding:utf-8_*_
import dns.resolver
domain=raw_input('Please input an domain:  ')#输入域名地址
A=dns.resolver.query(domain,'A') #指定查询类型为A记录
for i in A.response.answer:  #通过response.answer方法获取查询信息
    print i
    for j in i.items:  #遍历回应信息
        print j
```
```c
[root@localhost dnspython]# python dnspython1.py
Please input an domain:  www.baidu.com
www.baidu.com. 5 IN CNAME www.a.shifen.com.
www.a.shifen.com.
www.a.shifen.com. 5 IN A 39.156.66.18
www.a.shifen.com. 5 IN A 39.156.66.14
39.156.66.18
39.156.66.14
```
### 4.2 MX记录 实现MX记录查询方法源码

```c
[root@localhost dnspython]# cat dnspython2.py
#/usr/bin/env python
#_*_coding:utf-8_*_

import dns.resolver
domain=raw_input('Please input an domain: ')
MX=dns.resolver.query(domain,'MX') #指定查询类型为MX
for i in MX: #遍历回应结果，输出MX记录的preference及exchanger信息
    print 'MX preference=' ,i.preference, 'mail exchange=', i.exchange
```
```c
[root@localhost dnspython]# python dnspython2.py
Please input an domain: baidu.com
MX preference= 20 mail exchange= mx50.baidu.com.
MX preference= 15 mail exchange= mx.n.shifen.com.
MX preference= 10 mail exchange= mx.maillb.baidu.com.
MX preference= 20 mail exchange= jpmx.baidu.com.
MX preference= 20 mail exchange= mx1.baidu.com.
```
### 4.3 NS记录 实现NS记录查询方法源码

```c
[root@localhost dnspython]# cat dnspython3.py
#/usr/bin/env python
#_*_conding:utf-8_*_

import dns.resolver
domain=raw_input('Please input an domain: ')
ns=dns.resolver.query(domain,'NS')
for i in ns.response.answer:
    for j in i.items:
        print j.to_text()
 ```
```c
[root@localhost dnspython]# python dnspython3.py
Please input an domain: baidu.com
ns7.baidu.com.
ns3.baidu.com.
ns2.baidu.com.
ns4.baidu.com.
dns.baidu.com.
```
### 4.4 CNAME记录 实现CNAME记录查询方法源码

```c
[root@localhost dnspython]# cat dnspython4.py
#/usr/bin/env python
#_*_coding:utf-8_*_
import dns.resolver
domain=raw_input('Please input a domain: ')
cname=dns.resolver.query(domain,'CNAME')  #指定查询类型为CNAME
for i in cname.response.answer: #结果将回应cname后的目标域名
    for j in i.items:
        print j.to_text()
```
 ```c    
[root@localhost dnspython]# python dnspython4.py
Please input a domain: www.baidu.com
www.a.shifen.com.
```
### 4.5 DNS域名轮循业务监控
大部分的DNS解析都是一个域名对应一个IP地址，但是通过DNS 轮循技术可以做到一个域名对应多个IP，从而实现最简单且高效的负载 平衡，不过此方案最大的弊端是目标主机不可用时无法被自动剔除，因 此做好业务主机的服务可用监控至关重要。

#### 1.步骤

 - 1）实现域名的解析，获取域名所有的A记录解析IP列表；
 - 2）对IP列表进行HTTP级别的探测
 ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9f89bc9bb7b1e56dc659fc9c7a02cd10.png)
#### 2.代码解析
本示例第一步通过**dns.resolver.query（）方法获取业务域名A记录 信息，查询出所有IP地址列表，再使用httplib模块的request（）方法以 GET方式请求监控页面，监控业务所有服务的IP是否服务正常。**

```c
[root@localhost dnspython]# cat dnspython5.py
#/usr/bin/env python
#_*_coding:utf-8_*_
import dns.resolver
import os
import httplib
iplist=[] #定义域名ip列表变量
appdomain="www.baidu.com"  #定义业务域名
def get_iplist(domain=""): #域名解析函数，解析成功IP将被追加到iplist
    try:
        A=dns.resolver.query(domain,'A')
    except Exception, e:
        print "dns resolver error: " +str(e) 
        return
    for i in A.response.answer:
        for j in i.items:
            iplist.append(j)  #追加到iplist列表
    return True
def checkip(ip):
    checkurl=str(ip)+": 80"
    getcontent=""
    httplib.socket.setdefaulttimeout(5) #定义http连接5秒超时(5)秒
    conn=httplib.HTTPConnection(checkurl) #创建http连接对象
    try:
        conn.request("GET","/",headers={"Host": appdomain}) #发起URL请求，添加host主机头
        r=conn.getresponse()
        getcontent=r.read(15) #获取URL页面前15个字符用来可用性效验
    finally:
        if getcontent=="<!DOCTYPE html>": #监控URL页的内容一般是事先定义好的比如 http 200等
            print str(ip)+"\033[32;1m [ok]\033[0m"
        else:
            print str(ip)+"\033[31;1m [Error]\033[0m"
        print r.reason  #获取response的状态码
        print r.status  #获取response的状态  ok 或者error
if __name__=="__main__":
    if get_iplist(appdomain) and len(iplist)>0: #域名解析正确且至少返回一个IP
        for ip in iplist:
            checkip(ip)
    else:
        print "dns resolver error."
```

```c
[root@localhost dnspython]# python dnspython5.py
www.a.shifen.com. [ok]
OK
200
39.156.66.14 [ok]
OK
200
39.156.66.18 [ok]
OK
200
```
参考：

 - [Download dnspython](https://pypi.org/project/dnspython/)
 - [python运维开发常用模块(三)DNS处理模块dnspython](https://www.cnblogs.com/benjamin77/p/10815477.html)
 - [PyOps — Dnspython Toolkit](https://blog.devgenius.io/pyops-dnspython-toolkit-590a368b5c2)
 - [Python - 模块域名解析(dnspython)](https://blog.csdn.net/chen1415886044/article/details/108320745)
