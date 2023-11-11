#  Linux Command arp 地址转换协议
tags: 网络,协议



---

##  1. 简介
ARP 是 TCP/IP 协议族中的一个重要协议，用于确定对应 IP 地址的网卡物理地址。

使用 arp 命令，能够查看本地计算机或另一台计算机的 ARP 高速缓存中的当前内容。此外，使用 arp 命令可以人工方式设置静态的网卡物理地址 / IP 地址对，使用这种方式可以为缺省网关和本地服务器等常用主机进行本地静态配置，这有助于减少网络上的信息量。

按照缺省设置，ARP 高速缓存中的项目是动态的，每当向指定地点发送数据并且此时高速缓存中不存在当前项目时，ARP 便会自动添加该项目
## 2. 语法

```bash
arp（选项）（参数）
```

## 3. 选项

 - -a # 主机 ：显示 arp 缓冲区的所有条目；
 - -H # 地址类型 ：指定 arp 指令使用的地址类型；
 - -d # 主机 ：从 arp 缓冲区中删除指定主机的 arp 条目；
 - -D # 使用指定接口的硬件地址；
 - -e # 以 Linux 的显示风格显示 arp 缓冲区中的条目；
 - -i # 接口 ：指定要操作 arp 缓冲区的网络接口；
 - -s # 主机 MAC 地址 ：设置指定的主机的 IP 地址与 MAC 地址的静态映射；
 - -n # 以数字方式显示 arp 缓冲区中的条目；
 - -v # 显示详细的 arp 缓冲区条目，包括缓冲区条目的统计信息；
 - -f # 文件 ：设置主机的 IP 地址与 MAC 地址的静态映射。


主机：查询 arp 缓冲区中指定主机的 arp 条目。

## 4. 实例
显示arp 缓冲区内容

```bash
$ arp -v
Address                  HWtype  HWaddress           Flags Mask            Iface
192.168.0.134            ether   00:21:5E:C7:4D:88   C                     eth1
115.238.144.129          ether   38:22:D6:2F:B2:F1   C                     eth0
Entries: 2      Skipped: 0      Found: 2
```

添加静态 arp 映射

```bash
arp -s IP MAC-ADDRESS
arp -s 192.168.1.1 00:b1:b2:b3:b4:b5
```

 - ① arp –a：用于查看高速缓存中的所有项目。
 - ② arp -a IP：如果有多个网卡，那么使用 arp -a 加上接口的 IP 地址，就可以只显示与该接口相关的 ARP 缓存项目。
 - ③ arp -s IP 物理地址：向 ARP
   高速缓存中人工输入一个静态项目。该项目在计算机引导过程中将保持有效状态，或者在出现错误时，人工配置的物理地址将自动更新该项目。
 - ④ arp -d IP：使用本命令能够人工删除一个静态项目。

参考：

 - [TCP/IP协议——ARP详解](https://www.cnblogs.com/songwenlong/p/6103406.html)
 - [ARP详解](https://blog.csdn.net/drivermonkey/article/details/8745157)
 - [arp command in Linux with examples](https://www.geeksforgeeks.org/arp-command-in-linux-with-examples/)
 - [arp(7) — Linux manual page](https://man7.org/linux/man-pages/man7/arp.7.html)

