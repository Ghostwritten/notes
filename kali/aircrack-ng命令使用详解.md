## 简介
略
```bash
aircrack-ng
主要用于WEP及WPA-PSK密码的恢复，只要airodump-ng收集到足够数量的数据包，aircrack-ng就可以自动检测数据包并判断是否可以破解

airmon-ng
用于改变无线网卡工作模式，以便其他工具的顺利使用

airodump-ng
用于捕获802.11数据报文，以便于aircrack-ng破解

aireplay-ng
在进行WEP及WPA-PSK密码恢复时，可以根据需要创建特殊的无线网络数据报文及流量

airserv-ng
可以将无线网卡连接至某一特定端口，为攻击时灵活调用做准备

airolib-ng
进行WPA Rainbow Table攻击时使用，用于建立特定数据库文件

airdecap-ng
用于解开处于加密状态的数据包

tools
其他用于辅助的工具，如airdriver-ng、packetforge-ng等
```

## 安装

```bash
apt-get install aircrack-ng
```

## 命令
### airmon-ng
作用：该命令可以用于无线网卡的“管理模式”与“监听模式”的相互切换。当使用该命令并且不加任何选项时，将会显示无线网卡的状态。同时该命令也可以list或者终止会妨碍到无线网卡操作的进程。
```bash
$ airmon-ng   # 查看无线网卡状态
PHY	Interface	Driver		Chipset
$ airmon-ng start wlan0
```
list/kill会妨碍到无线网卡操作的进程
用法：airmon-ng <check> [kill]　
列出所有可能干扰无线网卡的程序。如果参数kill被指定，将会终止所有的程序。　　

```bash
airmon-ng check
```

### airodump-ng 
```bash
airodump-ng  wlan0mon  #查看wifi
```

