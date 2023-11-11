

## 1. 配置网卡

```bash
$ cat /etc/sysconfig/network-scripts/ifcfg-ens160 
TYPE=Ethernet
BOOTPROTO=static
IPV6INIT=no
NAME=ens160
DEVICE=ens160
ONBOOT=yes
IPADDR=192.168.23.41
NETMASK=255.255.240.0
GATEWAY=192.168.21.1

$  cat /etc/sysconfig/network-scripts/ifcfg-ens192 
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=eui64
NAME=ens192
DEVICE=ens192
ONBOOT=yes
IPADDR=192.168.23.3
PREFIX=20
GATEWAY=192.168.21.1
DNS1=8.8.8.8
```

## 2. 修改网卡名

### 2.1 配置grub
`/etc/sysconfig/grub`  添加 `net.ifnames=0 biosdevname=0`
```bash
$ cat /etc/sysconfig/grub 
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet net.ifnames=0 biosdevname=0"
GRUB_DISABLE_RECOVERY="true"
```
###  2.2 重载grub
重载grub 配置文件

```bash
grub2-mkconfig -o /boot/grub2/grub.cfg
```
### 2.3 配置网卡

```bash
$ cat /etc/sysconfig/network-scripts/ifcfg-eth0 
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=eth0
DEVICE=eth0
ONBOOT=yes
IPADDR=192.168.211.50
GATEWAY=192.168.211.2
DNS1=192.168.211.2
```

###  2.4 重启

```bash
reboot
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/084f9f346c3246e0bd66466bfbd5dc6b.gif#pic_center)

