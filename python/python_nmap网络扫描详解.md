#  python 模块 nmap 网络扫描



## 1. 简介
[NMap](https://pypi.org/project/python-nmap/)，也就是`Network Mapper`，最早是Linux下的网络扫描和嗅探工具包。源码：[https://github.com/nmmapper/python3-nmap](https://github.com/nmmapper/python3-nmap)

nmap是一个网络连接端扫描软件，用来扫描网上电脑开放的网络连接端。确定哪些服务运行在哪些连接端，并且推断计算机运行哪个操作系统（这是亦称 fingerprinting）。它是网络管理员必用的软件之一，以及用以评估网络系统安全。

正如大多数被用于网络安全的工具，nmap 也是不少黑客及骇客（又称脚本小子）爱用的工具 。系统管理员可以利用nmap来探测工作环境中未经批准使用的服务器，但是黑客会利用nmap来搜集目标电脑的网络设定，从而计划攻击的方法。

Nmap 常被跟评估系统漏洞软件Nessus 混为一谈。Nmap 以隐秘的手法，避开闯入检测系统的监视，并尽可能不影响目标系统的日常操作。

Nmap 在黑客帝国(The Matrix)中，连同SSH1的32位元循环冗余校验漏洞，被崔妮蒂用以入侵发电站的能源管理系统。

## 2. 功能
基本功能有三个:

 - 一是探测一组主机是否在线；
 - 其次是扫描 主机端口，嗅探所提供的网络服务；
 - 还可以推断主机所用的操作系统

 Nmap可用于扫描仅有两个节点的LAN，直至500个节点以上的网络。Nmap 还允许用户定制扫描技巧。通常，一个简单的使用ICMP协议的ping操作可以满足一般需求；也可以深入探测UDP或者TCP端口，直至主机所 使用的操作系统；还可以将所有探测结果记录到各种格式的日志中， 供进一步分析操作。

进行ping扫描，打印出对扫描做出响应的主机,不做进一步测试(如端口扫描或者操作系统探测)：

```bash
$ nmap -sP 192.168.1.0/24
```

仅列出指定网络上的每台主机，不发送任何报文到目标主机：

```bash
$ nmap -sL 192.168.1.0/24
```

探测目标主机开放的端口，可以指定一个以逗号分隔的端口列表(如-PS22，23，25，80)：

```bash
$ nmap -PS 192.168.1.234
```

使用UDP ping探测主机：

```bash
$ nmap -PU 192.168.1.0/24
```

使用频率最高的扫描选项：SYN扫描,又称为半开放扫描，它不打开一个完全的TCP连接，执行得很快：

```bash
$ nmap -sS 192.168.1.0/24
```
[nmao命令详解](https://www.cnblogs.com/weihua2616/p/6599629.html)

## 3. 安装
官网下载：[https://pypi.org/project/python-nmap/#files](https://pypi.org/project/python-nmap/#files)

```bash
$ pip install nmap
```
## 4. 方法
python-nmap模块的两个常用类，一个是PortScanner()类，实现一个nmap工具的端口扫描功能封装;另一个为PortScannerHostDict()类，实现存储与访问主机扫描结果

### 4.1 PortScanner()类常用方法
#### 4.1.1 scan()方法
`scan(self, hosts='127.0.0.1', ports=None, arguments='-sV')`方法：
实现指定主机、端口、namp命令行参数的扫描。

 - hosts为字符串类型，表示扫描的主机地址，格式可用"scanme.nmap.org"、"192.116.0-255.1-127"、"216.163.128.20/20"表示;
 - ports为字符串类型，表示扫描的端口,可以用"22,53,110,143-4564"表示;
 - namp命令行参数，格式为"-sU -sX -sC"

例如：

```bash
#!/usr/bin/python

import nmap

nm = nmap.PortScanner()
ret = nm.scan('192.168.1.121-122', '22,80')
print ret
```

```bash
$ python nm1.py
{'nmap': {'scanstats': {'uphosts': '2', 'timestr': 'Sat May  9 17:04:43 2020', 'downhosts': '0', 'totalhosts': '2', 'elapsed': '7.54'}, 'scaninfo': {'tcp': {'services': '22,80', 'method': 'syn'}}
.......
```
#### 4.1.2 command_line()方法
command_line(self)方法,返回的扫描方法映射到具体的nmap命令行，如：

```python
#!/usr/bin/python
import nmap
nm = nmap.PortScanner()
nm.scan('192.168.1.121-122', '22,80')
print nm.command_line()
```

```python
$ python nm1.py 
nmap -oX - -p 22,80 -sV 192.168.1.121-122
```
#### 4.1.3 scaninfo()方法
scaninfo(self)方法，返回nmap扫描信息，格式为字典类型，如：

```python
#!/usr/bin/python
import nmap
nm = nmap.PortScanner()
nm.scan('192.168.1.121-122', '22,80')
print nm.scaninfo()
```

```python
$ python nm1.py 
{'tcp': {'services': '22,80', 'method': 'syn'}}
```
#### 4.1.4 all_hosts()方法
all_hosts(self)方法，返回nmap扫描的主机清单，格式为列表类型，例如：

```python
#!/usr/bin/python
import nmap
nm = nmap.PortScanner()
re = nm.scan('192.168.1.121-122', '22,80')
print nm.all_hosts()
```

```python
$ python nm1.py 
['192.168.1.121', '192.168.1.122']
```
### 4.2 PortScannerHostDict()类常用方法
#### 4.2.1 hostname()方法
hostname(self)方法，返回扫描对象的主机名，如：

```bash
$ vim nm3.py 
```

```python
#!/usr/bin/env python
#--coding:utf-8---
import nmap # 导入 nmap.py 模块  
nm = nmap.PortScanner() # 实例化nmap.PortScanner对象  
nm.scan('127.0.0.1', '22-443') # 扫描127.0.0.1,端口号从22至443  
print nm['127.0.0.1'].hostname() # 获取一个主机127.0.0.1的主机名，通常为用户记录  
print nm['127.0.0.1'].hostnames() # 获取主机127.0.0.1的主机名列表，返回一个字典类型  
## [{'name':'hostname1', 'type':'PTR'}, {'name':'hostname2', 'type':'user'}]
print nm['127.0.0.1'].state() # 获取主机127.0.0.1的状态 (up|down|unknown|skipped)  
print nm['127.0.0.1'].all_protocols() # 获取执行的协议 ['tcp', 'udp'] 包含 (ip|tcp|udp|sctp)  
print nm['127.0.0.1']['tcp'].keys() # 获取tcp协议所有的端口号  
print nm['127.0.0.1'].all_tcp() # 获取tcp协议所有的端口号 (按照端口号大小进行排序)  
print nm['127.0.0.1'].all_udp() # 获取udp协议所有的端口号 (按照端口号大小进行排序)  
print nm['127.0.0.1'].all_sctp() # 获取sctp协议所有的端口号 (按照端口号大小进行排序)  
print nm['127.0.0.1'].has_tcp(22) # 主机127.0.0.1是否有关于22端口的任何信息  
print nm['127.0.0.1']['tcp'][22] # 获取主机127.0.0.1关于22端口的信息  
print nm['127.0.0.1'].tcp(22) # 获取主机127.0.0.1关于22端口的信息  
print nm['127.0.0.1']['tcp'][22]['state'] # 获取主机22端口的状态 (open)  
```

```bash
$ python nm3.py 
localhost
[{'type': 'PTR', 'name': 'localhost'}]
up
['tcp']
[25, 22, 111]
[22, 25, 111]
[]
[]
True
{'product': 'OpenSSH', 'state': 'open', 'version': '7.4', 'name': 'ssh', 'conf': '10', 'extrainfo': 'protocol 2.0', 'reason': 'syn-ack', 'cpe': 'cpe:/a:openbsd:openssh:7.4'}
{'product': 'OpenSSH', 'state': 'open', 'version': '7.4', 'name': 'ssh', 'conf': '10', 'extrainfo': 'protocol 2.0', 'reason': 'syn-ack', 'cpe': 'cpe:/a:openbsd:openssh:7.4'}
open
```
## 5. 实战
### 5.1 测试多个主机的状态、协议、端口

```python
$ vim nm4.py
#!/usr/bin/python

import nmap
nm = nmap.PortScanner()
re = nm.scan('192.168.1.190-192', '22,80')
for host in nm.all_hosts():
    print('----------------------------------------------------')
    print('Host : %s (%s)' % (host, nm[host].hostname()))
    print('State : %s' % nm[host].state())
    for proto in nm[host].all_protocols():
        print('----------')
        print('Protocol : %s' % proto)

        lport = nm[host][proto].keys()
        lport.sort()
        for port in lport:
            print ('port : %s\tstate : %s' % (port, nm[host][proto][port]['state']))

print(nm.csv())
```

```bash
$ python nm4.py 
----------------------------------------------------
Host : 192.168.1.190 ()
State : up
----------
Protocol : tcp
port : 22	state : open
port : 80	state : open
----------------------------------------------------
Host : 192.168.1.191 ()
State : up
----------
Protocol : tcp
port : 22	state : open
port : 80	state : filtered
----------------------------------------------------
Host : 192.168.1.192 ()
State : up
----------
Protocol : tcp
port : 22	state : open
port : 80	state : open

host;hostname;hostname_type;protocol;port;name;state;product;extrainfo;reason;version;conf;cpe
192.168.1.190;;;tcp;22;ssh;open;OpenSSH;protocol 2.0;syn-ack;7.4;10;cpe:/a:openbsd:openssh:7.4
192.168.1.190;;;tcp;80;http;open;Apache httpd;(CentOS);syn-ack;2.4.6;10;cpe:/a:apache:http_server:2.4.6
192.168.1.191;;;tcp;22;ssh;open;OpenSSH;protocol 2.0;syn-ack;7.4;10;cpe:/a:openbsd:openssh:7.4
192.168.1.191;;;tcp;80;http;filtered;;;no-response;;3;
192.168.1.192;;;tcp;22;ssh;open;OpenSSH;protocol 2.0;syn-ack;7.4;10;cpe:/a:openbsd:openssh:7.4
192.168.1.192;;;tcp;80;http;open;Apache httpd;(CentOS);syn-ack;2.4.6;10;cpe:/a:apache:http_server:2.4.6
192.168.1.193;;;tcp;22;ssh;filtered;;;no-response;;3;
192.168.1.193;;;tcp;80;http;filtered;;;no-response;;3;

```
### 5.2 测试主机状态

```bash
$ vim nm5.py 
```

```python
#!/usr/bin/python

import nmap
nm = nmap.PortScanner()
nm.scan(hosts='192.168.1.0/24', arguments='-n -sP -PE -PA21,23,80,3389')
hosts_list = [(x, nm[x]['status']['state']) for x in nm.all_hosts()]
for host, status in hosts_list:
    print(host + ":" + status)
```

```bash
$ python nm5.py
192.168.1.0:up
192.168.1.1:up
192.168.1.10:up
192.168.1.100:up
192.168.1.101:up
192.168.1.102:up
192.168.1.103:up
192.168.1.104:up
```

参考：

 - [xiaix's Blog](https://xiaix.me/python-nmapxiang-jie/)
 - [python-nmap : nmap from python](https://xael.org/pages/python-nmap-en.html)
 - [Port scanner using ‘python-nmap’](https://www.geeksforgeeks.org/port-scanner-using-python-nmap/)
 - [Using the Nmap Port Scanner with Python](https://www.studytonight.com/network-programming-in-python/integrating-port-scanner-with-nmap)
 - [Python Nmap Module Fully Explained with 8 Programs](https://www.pythonpool.com/python-nmap/)

