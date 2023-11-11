### 链路聚合

输入    #所有的帮助信息中查找example

 

```bash
$ man teamd.conf      #man  teamd.conf ，搜索 /example找出需要的聚合模式
$ nmcli connection add type team autoconnect yes  ifname team0 con-name team0 config '{"runner": {"name": "activebackup"}}'   

$ nmcli connection add type team-slave ifname eth1 con-name team0-1 autoconnect yes master team0 

$ nmcli connection add type team-slave ifname eth2 con-name team0-2 autoconnect yes master team0 

$ nmcli connection modify team0 ipv4.method manual ipv4.addresses 192.168.1.1/24 connection.autoconnect yes 

$ nmcli connection up team0

$ nmcli connection up team0-1

$ nmcli connection up team0-2

$ ifconfig team0     

$ nmcli connection delete team0 

$ nmcli connection delete team0-1

$ nmcli connection delete team0-2
```
### 配置IPv6

```bash
$ nmcli connection modify 'System eth0' ipv6.method manual ipv6.addresses 2003:ac18::305/64 connection.autoconnect  yes

$ nmcli connection up 'System eth0'
```
参考链接：
[https://www.cnblogs.com/liuhedong/p/10695969.html](https://www.cnblogs.com/liuhedong/p/10695969.html)

