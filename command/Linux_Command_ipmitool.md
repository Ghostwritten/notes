#  Linux Command ipmitool 
tags: 设备管理



---

## 1. 简介

`IPMI`（Intelligent Platform Management Interface）即智能平台管理接口是使硬件管理具备“智能化”的新一代通用接口标准。用户可以利用 IPMI 监视服务器的物理特征，如温度、电压、电扇工作状态、电源供应以及机箱入侵等。Ipmi 最大的优势在于它是独立于 CPU BIOS 和 OS 的，所以用户无论在开机还是关机的状态下，只要接通电源就可以实现对服务器的监控。Ipmi 是一种规范的标准，其中最重要的物理部件就是BMC(Baseboard Management Controller 如图1)，一种嵌入式管理微控制器，它相当于整个平台管理的“大脑”，通过它 ipmi 可以监控各个传感器的数据并记录各种事件的日志。

[ipmitool](https://github.com/ipmitool/ipmitool) 是一种可用在 linux 系统下的命令行方式的 ipmi 平台管理工具，它支持 ipmi 1.5 规范（最新的规范为 ipmi 2.0），通过它可以实现获取传感器的信息、显示系统日志内容、网络远程开关机等功能。Ipmitool 有两种使用方式

## 2. 参数

Options(选项)

```bash
-a     提示输入远程服务器的密码
-A <authtype>   当IPMIv1.5会话激活时，指定使用一个认证类型。
-c 使输出格式为 CSV(逗号分隔的变量)格式。但是不是针对所有命令都有效。
-C <ciphersuite>为IPMIv2 lanplus连接使用远程服务器身份验证、完整性和加密算法。请看IPMIv2说明书中的表格 22-19。默认的三个格式为：指定RAKP-HMAC-SHA1为验证，HMAC-SHA1-96为完整性，AES-CBC-128为加密算法。
-E 远程服务密码通过环境变量IPMI_PASSWORD来指定。
-f <password_file>指定一个文件，而这个文件中包含了远程服务密码。如果这个选项没有被使用，或者指定的文件不存在，那么那么密码将默认为NULL。
-h 获取基本帮助
-H <address>   远程服务地址，可以为ip地址或者是主机名。Lan和lanplus接口都需要这个操作。
-I <interface>  选择使用的IPMI接口。编译支持的接口都在使用帮助输出中可见
-L <privlvl> 力量会话特权级别。可以为CALLBACK,USER, OPERATOR, ADMIN。默认为ADMIN。
-m <local_address>  设置本地IPMB（智能平台管理总线）地址。默认的为0x20。如果是一般的操作，那么就没有必要更改它。
-o <oemtype>   选择支持的OEM（原始设备制造商）类型。这通常涉及到代码中的小窍门，以解决各种BMC在不同制造商的怪癖到位。使用命令“-o list”来查看当前支持的OEM类型的列表。
-p <port>   设置要连接的远程服务UPD端口，默认为623。
-P <password>   在命令行中指定远程服务密码。如果支持，他将会进程列表中被掩盖。注意！：不推荐在命令行中指定密码。
-t <target_address>  桥接IPMI的请求到远程目标地址。
-U <username>   远程服务用户名，默认为NULL。
-v 提高详细输出的级别。这个操作可以指定多次用来提高调试输出的级别。如果指定三次，那么你将会得到所有传入和传出的数据包。
-V 列出版本信息。
 如果没有密码方法被指定，那么ipmitool将会提示用户输入密码。如果用户没有输入密码，那么远程服务密码将会设置为NULL。
```

## 3. 开关机，重启

1. 查看开关机状态：

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) power status
```

2. 开机：

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) power on
```

3. 关机：

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) power off
```

4. 重启：

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) power reset
```

 

## 4. 用户管理

**说明**：[`ChannelNo`] 字段是可选的，`ChannoNo`为1或者8；`BMC`默认有2个用户：user id为1的`匿名用户`，user id为2的`ADMIN用户`；<>字段为必选内容；<privilege level>：2为user权限，3为Operator权限，4为Administrator权限；

1. 查看用户信息：

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) user list [ChannelNo]
```

2. 增加用户：

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) user set name <user id> <username>
```

3. 设置密码：

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) user set password <user id> <password>
```

4. 设置用户权限：

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) user priv <user id> <privilege level> [ChannelNo]
```

5. 启用/禁用用户：

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) user enable/disable <user id>
```

 

## 5. IP网络设置

说明：[`ChannelNo`] 字段是可选的，ChannoNo为`1(Share Nic网络)`或者`8（BMC独立管理网络）`；设置网络参数，必须首先设置IP为静态，然后再进行其他设置；

1. 查看网络信息：

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) lan print [ChannelNo]
```

2. 修改IP为静态还是DHCP模式：

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) lan set <ChannelNo> ipsrc <static/dhcp>
```

3. 修改IP地址：

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) lan set <ChannelNo> ipaddr <IPAddress>
```

4. 修改子网掩码：

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) lan set <ChannelNo> netmask <NetMask>
```

5. 修改默认网关：

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) lan set <ChannelNo> defgw ipaddr <默认网关>
```

## 6. SOL功能

**说明：<9.6/19.2/38.4/57.6/115.2>其中115.2代表115200，即*1000是表示的波特率。**

1. 设置SOL串口波特率：

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) sol set volatile-bit-rate <9.6/19.2/38.4/57.6/115.2>
```

2. 打开SOL功能：

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) sol activate
```

3. 关闭SOL功能：

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) sol deactivate
```

 

## 7. SEL日志查看

1. 查看SEL日志：

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) sel list
```

 

## 8. FRU信息查

看

1. 查看FRU信息：

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) fru list
```

 

## 9. SDR，Sensor信息查看

1. 查看SDR Sensor信息：

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) sdr
```

2. 查看Sensor信息：

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) sensor list
```

 

## 10. mc(管理单元BMC)状态和控制

1. 重启动BMC：

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) mc reset <warm/cold>
```

 

## 11. 设置BMC的iptables防火墙

1. 设置某一段IP可以访问BMC

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) raw 0x32 0x76 0x01 0x01 ip1(0xa 0xa 0xa 0xa) ip2(0xb 0xb 0xb 0xb)

ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) raw 0x32 0x76 0x09
```

2. 设置某个IP可以访问BMC

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) raw 0x32 0x76 0x00 0x01 ip1(0xa 0xa 0xa 0xa)

ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) raw 0x32 0x76 0x09
```

3. 取消设置

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) raw 0x32 0x76 0x08
```

4．获取防火墙设置

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) raw 0x32 0x77 0x01 0x00
```

5. 阻止/开启某个端口

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) raw 0x32 0x76 0x02 0x00/0x01 0x00 (portno)0x22 0x00
```

6. 取消某个端口的设置（6是5的对应取消操作）

```bash
ipmitool -H (BMC的管理IP地址) -I lanplus -U (BMC登录用户名) -P (BMC 登录用户名的密码) raw 0x32 0x76 0x06 0x00/0x01 0x00 (portno)0x22 0x00
```

 

```c
# service ipmi start
# ipmitool -I open shell
```
可以直接进入本地BMC shell

```bash
$ ipmitool -I lan -H -U shell
```
 输入password,进入IPMI交互模式,当然这里也可以把shell直接换成bmc命令 , 另外ipmitool支持端口,所以是否可以先做DNAT，然后远程直接管理内网机器.
ipmitool提供的功能要比windows下ipmish提供的功能多得多，用法相对复杂一些

## 12. 详细命令

引用
Ipmitool本地监控使用命令：

```bash
ipmitool –I open command
```

其中-I open表示使用OpenIPMI接口，command有以下项：

```bash
a) raw：发送一个原始的IPMI请求，并且打印回复信息。
b) Lan：配置网络（lan）信道(channel)
c) chassis ：查看底盘的状态和设置电源
d) event：向BMC发送一个已经定义的事件（event），可用于测试配置的SNMP是否成功
e) mc： 查看MC（Management Contollor）状态和各种允许的项
f) sdr：打印传感器仓库中的所有监控项和从传感器读取到的值。
g) Sensor：打印详细的传感器信息。
h) Fru：打印内建的Field Replaceable Unit (FRU)信息
i) Sel： 打印 System Event Log (SEL)
j) Pef： 设置 Platform Event Filtering (PEF)，事件过滤平台用于在监控系统发现有event时候，用PEF中的策略进行事件过滤，然后看是否需要报警。
k) Sol/isol：用于配置通过串口的Lan进行监控
l) User：设置BMC中用户的信息 。
m) Channel：设置Management Controller信道。
```

```bash
Ipmitool –I open sensor list //命令可以获取传感器中的各种监测值和该值的监测阈值，包括（CPU温度，电压，风扇转速，电源调制模块温度，电源电压等信息）
Ipmitool –I open sensor get “CPU0Temp” //可以获取ID为CPU0Temp监测值，CPU0Temp是sensor的ID，服务器不同，ID表示也不同。
Ipmitool –I open sensor thresh //设置ID值等于id的监测项的各种限制值。
Ipmitool –I open chassis status //查看底盘状态，其中包括了底盘电源信息，底盘工作状态等
Ipmitool –I open chassis restart_cause //查看上次系统重启的原因
Ipmitool –I open chassis policy list //查看支持的底盘电源相关策略。
Ipmitool –I open chassis power on //启动底盘，用此命令可以远程开机
Ipmitool –I open chassis power off //关闭底盘，用此命令可以远程开机
Ipmitool –I open chassis power reset //实现硬重启，用此命令可以远程开机
Ipmi还可以设置系统启动boot的设备，具体见ipmitool帮助文档。
Ipmitool –I open mc reset //使BMC重新硬启动
Ipmitool –I open mc info //查看BMC硬件信息
Ipmitool –I open mc getenables //列出BMC所有允许的选项
Ipmitool –I open mc setenables =[on|off]，//设置bmc相应的允许/禁止选项。
Ipmitool-I open event 1 //发送一个温度过高的消息到System Event Log中，可以发送的Event有：
1 Temperature: Upper Critical: Going High
2 Voltage Threshold: Lower Critical: Going Low
3 Memory: Correctable ECC Error Detected
Ipmitool-I open event //命令可以用测试配置的IPMI中的snmp功能是否成功。
Ipmitool -I open lan print 1 //打印现咱channel 1的信息 。
Ipmitool -I open lan set 1 ipaddr 10.10.113.95 //设置channel 1 的地址为10.10.113.95
Ipmitool -I open lan set 1 snmp public //设置channel 1 上snmp的community为public。
Ipmitool -I open lan set 1 access on //设置channel 1允许访问。
Ipmitool -I open pef info  //打印Platform Event Filtering （pef）信息
Ipmitool -I open pef status  //查看Platform Event Filtering （pef）状态
Ipmitool -I open pef policy //查看Platform Event Filtering （pef）策略设置
Ipmitool -I open sdr list fru //读取fru信息并显示。
```

[root@localhost ~]# yum install -y ipmitool #安装这个包
电源相关:
#开启电源

```bash
ipmitool -I lanplus -H 10.41.1.41 -U root -P root power on
```
#关闭电源

```bash
ipmitool -I lanplus -H 10.41.1.41 -U root -P root power off
```
#重启电源

```bash
ipmitool -I lanplus -H 10.41.1.41 -U root -P root power reset
```
#查看电源状态

```bash
ipmitool -I lanplus -H 10.41.1.41 -U root -P root power status
```
启动项相关:
#设置为BIOS启动

```bash
ipmitool -I lanplus -H 10.41.1.41 -U root -P root chassis bootparam set bootflag force_bios
```
#设置为pxe启动

```bash
ipmitool -I lanplus -H 10.41.1.41 -U root -P root chassis bootparam set bootflag force_pxe
```
#设置为光盘启动

```bash
ipmitool -I lanplus -H 10.41.1.41 -U root -P root chassis bootparam set bootflag force_cdrom
```
#设置为硬盘启动

```bash
ipmitool -I lanplus -H 10.41.1.41 -U root -P root chassis bootparam set bootflag force_disk
```
#重启BMC

```bash
ipmitool -I lanplus -H 10.41.1.41 -U root -P root mc reset cold
```
#修改BMC密码

```bash
ipmitool -I lanplus -H 10.41.1.41 -U root -P root user set password 2 new_password #new_password 这个是新密码
```
#远程文本重定向 重启服务器就能看到画面

```bash
ipmitool -I lanplus-H 10.41.1.41 -U root -P root sol activate
```

#当有这个错误时 Info: SOL payload already active on another session

```bash
ipmitool -I lanplus-H 10.41.1.41 -U root -P root sol deactivate #这命令是踢出其他的人会话
```
#获取mac地址 有些机型不准 我这个是dell的服务器 华为服务器也可以使用

```bash
"""ipmitool -I lanplus -H 10.41.1.41 -U root -P root lan print |grep "MAC Address"|awk '{print $NF}'"""
```

python脚本

```bash
from subprocess import Popen, PIPE
cmd = """ipmitool -I lanplus -H 10.41.1.41 -U root -P root lan print |grep "MAC Address"|awk '{print $NF}'"""
text =  Popen(cmd, stdout=PIPE, shell=True).stdout.read()
prefix_mac = text[:-3]
last_two  = text[-2:]
plus_one  = int(last_two, 16) - 2
plus_one2 = int(last_two, 16) - 1
new_last_two  = hex(plus_one)[2:]
new_last_two2 = hex(plus_one2)[2:]
if len(new_last_two) == 1:
        new_last_two = '0' + new_last_two
if len(new_last_two2) == 1:
        new_last_two2 = '0' + new_last_two2
new_mac = prefix_mac.replace(':','') + new_last_two
new_mac2 = prefix_mac.replace(':','') + new_last_two2
print(new_mac,new_mac2)
```
参考链接：


 - [ipmitool命令详解](https://www.cnblogs.com/machangwei-8/p/10350824.html)
 - [Using IPMItool to View System Information](https://docs.oracle.com/cd/E19464-01/820-6850-11/IPMItool.html)
 - [ipmitool](http://linux.51yip.com/search/ipmitool)



