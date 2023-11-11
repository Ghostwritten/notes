
![在这里插入图片描述](https://img-blog.csdnimg.cn/2ab0bdc033c94cafb760f1b9ea30eb54.png)






- 系统安装请参考，[vsphere client 6.7安装 linux centos 7.9.2009](https://blog.csdn.net/xixihahalelehehe/article/details/127616480)

##  配置主机名

```bash
hostnamectl set-hostname master
```

## 文件配置静态网卡

```bash
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
NAME=ens192
UUID=d5c2e54d-d040-4774-be02-7808e851bf94
DEVICE=ens192
ONBOOT=yes
IPADDR=192.168.10.14
GATEWAY=192.168.10.1
NETMASK=255.255.252.0
DNS1=8.8.8.8
```
重启网卡
- 不能使用`service`和`systemctl`
- 方法：`nmcli c reload +网卡名`

```bash
$ nmcli c reload ens192
$ nmcli c down ens192
$ nmcli c up ens192
```
立即生效connection，有3种方法

```bash
$ nmcli c up ethX
$ nmcli d reapply ethX
$ nmcli d connect ethX
```

查询网卡

```bash
$ nmcli
ens192: connected to ens192
        "VMware VMXNET3"
        ethernet (vmxnet3), 00:50:56:83:24:FC, hw, mtu 1500
        ip4 default
        inet4 192.168.10.14/22
        route4 0.0.0.0/0
        route4 192.168.8.0/22
        inet6 fe80::b23c:870:5369:bb05/64
        route6 fe80::/64
        route6 ff00::/8

lo: unmanaged
        "lo"
        loopback (unknown), 00:00:00:00:00:00, sw, mtu 65536

DNS configuration:
        servers: 8.8.8.8
        interface: ens192


$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:83:24:fc brd ff:ff:ff:ff:ff:ff
    inet 192.168.10.14/22 brd 192.168.11.255 scope global noprefixroute ens192
       valid_lft forever preferred_lft forever
    inet6 fe80::b23c:870:5369:bb05/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

##  命令行配置网卡

```bash
$ nmcli device
DEVICE  TYPE      STATE      CONNECTION
ens192  ethernet  connected  ens192
lo      loopback  unmanaged  --

＃设置IPv4地址
$ nmcli connection modify ens192 ipv4.addresses 192.168.10.14/24
＃设置网关
$ nmcli connection modify ens192 ipv4.gateway 192.168.10.1
＃设置DNS
$ nmcli connection modify ens192 ipv4.dns 8.8.8.8
＃设置手动进行静态设置（对于DHCP为[auto]）
$ nmcli connection modify ens192 ipv4.method manual
＃重新启动界面以重新加载设置
$ nmcli connection down ens192; nmcli connection up ens192

＃显示设置
$ nmcli device show ens192
GENERAL.DEVICE:                         ens192
GENERAL.TYPE:                           ethernet
GENERAL.HWADDR:                         00:50:56:83:24:FC
GENERAL.MTU:                            1500
GENERAL.STATE:                          100 (connected)
GENERAL.CONNECTION:                     ens192
GENERAL.CON-PATH:                       /org/freedesktop/NetworkManager/ActiveConnection/1
WIRED-PROPERTIES.CARRIER:               on
IP4.ADDRESS[1]:                         192.168.10.14/22
IP4.GATEWAY:                            192.168.10.1
IP4.ROUTE[1]:                           dst = 0.0.0.0/0, nh = 192.168.10.1, mt = 100
IP4.ROUTE[2]:                           dst = 192.168.8.0/22, nh = 0.0.0.0, mt = 100
IP4.DNS[1]:                             8.8.8.8
IP6.ADDRESS[1]:                         fe80::b23c:870:5369:bb05/64
IP6.GATEWAY:                            --
IP6.ROUTE[1]:                           dst = fe80::/64, nh = ::, mt = 100
IP6.ROUTE[2]:                           dst = ff00::/8, nh = ::, mt = 256, table=255
```

##  路由转发

```bash
$ modprobe br_netfilter
$ vim /etc/sysctl.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables = 1
$ sysctl -p
```

##  防火墙

```bash
systemctl stop firewalld
systemctl disable firewalld
systemctl status firewalld
```
##  selinux

```bash
setenforce 0 && getenforce
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```
##  修改最大文件句柄数和最大进程数

```bash
$ vim /etc/security/limits.conf
......
* soft nofile 102400
* hard nofile 102400

$ sysctl -p 
$ ulimit -n
```
##  设置 swappiness
设置swappiness，控制运行时内存的相对权重，过多的交换空间会引起GC耗时的激增.
临时设置指令：

```bash
sysctl -w vm.swappiness=10
#临时设置查看指令
cat /proc/sys/vm/swappiness
```

永久设置指令：

```bash
echo vm.swappiness = 10 >> /etc/sysctl.conf
```
## 关闭透明大页面transparent_hugepage
临时关闭指令1：

```bash
echo never > /sys/kernel/mm/transparent_hugepage/enabled
```

临时关闭指令2：

```bash
echo never > /sys/kernel/mm/transparent_hugepage/defrag
```
永久关闭，配置信息落盘到配置文件，机器重新有效。

```bash
$ vim /etc/rc.d/rc.local
.......
if test -f /sys/kernel/mm/transparent_hugepage/enabled;
then echo never > /sys/kernel/mm/transparent_hugepage/enabled
fi
if test -f /sys/kernel/mm/transparent_hugepage/defrag;
then echo never > /sys/kernel/mm/transparent_hugepage/defrag
fi

$ chmod +x /etc/rc.d/rc.local
```


##  启动或禁用服务
显示活动的服务列表
```bash
$ systemctl -t service
  UNIT                                                                  LOAD   ACTIVE SUB     DESCRIPTION
  auditd.service                                                        loaded active running Security Auditing Service
  crond.service                                                         loaded active running Command Scheduler
  dbus.service                                                          loaded active running D-Bus System Message Bus
● dnf-makecache.service                                                 loaded failed failed  dnf makecache
  dracut-shutdown.service                                               loaded active exited  Restore /run/initramfs on>
  getty@tty1.service                                                    loaded active running Getty on tty1
  import-state.service                                                  loaded active exited  Import network configurat>
  irqbalance.service                                                    loaded active running irqbalance daemon
  kdump.service                                                         loaded active exited  Crash recovery kernel arm>
  kmod-static-nodes.service                                             loaded active exited  Create list of required s>
  ldconfig.service                                                      loaded active exited  Rebuild Dynamic Linker Ca>
  lvm2-monitor.service                                                  loaded active exited  Monitoring of LVM2 mirror>
  lvm2-pvscan@8:3.service                                               loaded active exited  LVM event activation on d>
  NetworkManager.service                                                loaded active running Network Manager
  nis-domainname.service                                                loaded active exited  Read and set NIS domainna>
  polkit.service                                                        loaded active running Authorization Manager
  rngd-wake-threshold.service                                           loaded 
  .....................
```
显示所有服务清单

```bash
 systemctl list-unit-files -t service
UNIT FILE                                  STATE
auditd.service                             enabled
autovt@.service                            enabled
blk-availability.service                   disabled
console-getty.service                      disabled
container-getty@.service                   static
cpupower.service                           disabled
crond.service                              enabled
dbus-org.freedesktop.hostname1.service     static
dbus-org.freedesktop.locale1.service       static
dbus-org.freedesktop.login1.service        static
dbus-org.freedesktop.nm-dispatcher.service enabled
dbus-org.freedesktop.portable1.service     static
dbus-org.freedesktop.timedate1.service     static
dbus.service                               static
dbxtool.service                            disabled
debug-shell.service                        disabled
dm-event.service                           static
dnf-makecache.service                      static
dracut-cmdline.service                     static
dracut-initqueue.service                   static
dracut-mount.service                       static
dracut-pre-mount.service                   static
dracut-pre-pivot.service                   static
dracut-pre-trigger.service                 static
dracut-pre-udev.service                    static
dracut-shutdown.service                    static
ebtables.service                           disabled
emergency.service                          static
```
停止并关闭服务的自动启动设置

```bash
systemctl stop   smartd && systemctl disable  smartd
systemctl status  smartd
```

##  NTP时间同步
修改时区为本地时区

```bash
$ ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
$ yum -y install ntp
$ vim /etc/ntp.conf
#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst
server 192.168.10.56
fudge 192.168.10.56 stratum 10
$ systemctl start ntpd && systemctl enable ntpd
$ systemctl status ntpd
```
- 查看当前节点同步的时间服务器：`ntpq -p`

- 查看节点与时间同步服务器的偏差时间：`ntpdc -c loopinfo`

- 查看ntp状态：`ntpstat`

##  yum 与 dnf
RHEL 8 / CentOS 8上的软件包管理工具 [DNF](https://blog.csdn.net/xixihahalelehehe/article/details/123168620)（Dandified YUM）已设置为默认值。
但是，[yum]命令也作为指向[dnf]的链接而存在，因此可以以相同的用法使用[yum]或[dnf]。

实际上，RedHat上的官方文档使用`RHEL 8`的[yum]命令给出了示例
```bash
$ which yum
/usr/bin/yum

$ ll /usr/bin/yum
lrwxrwxrwx. 1 root root 5 Apr 25  2020 /usr/bin/yum -> dnf-3

$ which dnf
/usr/bin/dnf

$ ll /usr/bin/dnf
lrwxrwxrwx. 1 root root 5 Apr 25  2020 /usr/bin/dnf -> dnf-3

$ ll /usr/bin/dnf-3
-rwxr-xr-x. 1 root root 1954 Apr 25  2020 /usr/bin/dnf-3

$ rpm -aq | grep yum
yum-4.2.17-6.el8.noarch

$ rpm -q yum
yum-4.2.17-6.el8.noarch

$ rpm -ql yum
/etc/dnf/protected.d/yum.conf
/etc/yum.conf
/etc/yum/pluginconf.d
/etc/yum/protected.d
/etc/yum/vars
/usr/bin/yum
/usr/share/man/man1/yum-aliases.1.gz
/usr/share/man/man5/yum.conf.5.gz
/usr/share/man/man8/yum-shell.8.gz
/usr/share/man/man8/yum.8.gz

$ ll  /etc/yum.conf
lrwxrwxrwx. 1 root root 12 Apr 25  2020 /etc/yum.conf -> dnf/dnf.conf

$ ll /etc/yum/vars
lrwxrwxrwx. 1 root root 11 Apr 25  2020 /etc/yum/vars -> ../dnf/vars

$ rpm -aq | grep dnf
libdnf-0.39.1-5.el8.x86_64
python3-dnf-4.2.17-6.el8.noarch
python3-dnf-plugins-core-4.0.12-3.el8.noarch
python3-libdnf-0.39.1-5.el8.x86_64
dnf-4.2.17-6.el8.noarch
dnf-plugins-core-4.0.12-3.el8.noarch
dnf-data-4.2.17-6.el8.noarch

$ rpm -ql dnf
/usr/bin/dnf
/usr/lib/systemd/system/dnf-makecache.service
/usr/lib/systemd/system/dnf-makecache.timer
/usr/share/bash-completion
/usr/share/bash-completion/completions
/usr/share/bash-completion/completions/dnf
/usr/share/locale/ar/LC_MESSAGES/dnf.mo
/usr/share/locale/bg/LC_MESSAGES/dnf.mo
/usr/share/locale/bn_IN/LC_MESSAGES/dnf.mo
/usr/share/locale/ca/LC_MESSAGES/dnf.mo
/usr/share/locale/cs/LC_MESSAGES/dnf.mo
/usr/share/locale/da/LC_MESSAGES/dnf.mo
/usr/share/locale/de/LC_MESSAGES/dnf.mo
/usr/share/locale/el/LC_MESSAGES/dnf.mo
/usr/share/locale/en_GB/LC_MESSAGES/dnf.mo
/usr/share/locale/eo/LC_MESSAGES/dnf.mo
/usr/share/locale/es/LC_MESSAGES/dnf.mo
/usr/share/locale/eu/LC_MESSAGES/dnf.mo
/usr/share/locale/fa/LC_MESSAGES/dnf.mo
/usr/share/locale/fi/LC_MESSAGES/dnf.mo
/usr/share/locale/fil/LC_MESSAGES/dnf.mo
/usr/share/locale/fr/LC_MESSAGES/dnf.mo
/usr/share/locale/fur/LC_MESSAGES/dnf.mo
/usr/share/locale/gd/LC_MESSAGES/dnf.mo
/usr/share/locale/gu/LC_MESSAGES/dnf.mo
/usr/share/locale/he/LC_MESSAGES/dnf.mo
/usr/share/locale/hi/LC_MESSAGES/dnf.mo
/usr/share/locale/hr/LC_MESSAGES/dnf.mo
/usr/share/locale/hu/LC_MESSAGES/dnf.mo
/usr/share/locale/id/LC_MESSAGES/dnf.mo
/usr/share/locale/it/LC_MESSAGES/dnf.mo
/usr/share/locale/ja/LC_MESSAGES/dnf.mo
/usr/share/locale/ka/LC_MESSAGES/dnf.mo
/usr/share/locale/kk/LC_MESSAGES/dnf.mo
/usr/share/locale/ko/LC_MESSAGES/dnf.mo
/usr/share/locale/lt/LC_MESSAGES/dnf.mo
/usr/share/locale/ml/LC_MESSAGES/dnf.mo
/usr/share/locale/mr/LC_MESSAGES/dnf.mo
/usr/share/locale/ms/LC_MESSAGES/dnf.mo
/usr/share/locale/nb/LC_MESSAGES/dnf.mo
/usr/share/locale/nl/LC_MESSAGES/dnf.mo
/usr/share/locale/or/LC_MESSAGES/dnf.mo
/usr/share/locale/pa/LC_MESSAGES/dnf.mo
/usr/share/locale/pl/LC_MESSAGES/dnf.mo
/usr/share/locale/pt/LC_MESSAGES/dnf.mo
/usr/share/locale/pt_BR/LC_MESSAGES/dnf.mo
/usr/share/locale/ru/LC_MESSAGES/dnf.mo
/usr/share/locale/sk/LC_MESSAGES/dnf.mo
/usr/share/locale/sq/LC_MESSAGES/dnf.mo
/usr/share/locale/sr/LC_MESSAGES/dnf.mo
/usr/share/locale/sr@latin/LC_MESSAGES/dnf.mo
/usr/share/locale/sv/LC_MESSAGES/dnf.mo
/usr/share/locale/th/LC_MESSAGES/dnf.mo
/usr/share/locale/tr/LC_MESSAGES/dnf.mo
/usr/share/locale/uk/LC_MESSAGES/dnf.mo
/usr/share/locale/ur/LC_MESSAGES/dnf.mo
/usr/share/locale/zh_CN/LC_MESSAGES/dnf.mo
/usr/share/locale/zh_TW/LC_MESSAGES/dnf.mo
/usr/share/man/man7/dnf.modularity.7.gz
/usr/share/man/man8/dnf.8.gz
/usr/share/man/man8/yum2dnf.8.gz
/var/cache/dnf
```

##  dnf 更新软件库

在CentOS服务器成为生产系统之后，可能很难更新系统，但是至少在安装后，将CentOS服务器更新为最新版本。

```bash
$ dnf -y upgrade
CentOS-8 - AppStream                                                                     92  B/s |  38  B     00:00
Error: Failed to download metadata for repo 'AppStream': Cannot prepare internal mirrorlist: No URLs in mirrorlist
```
> 如果您仍然在管理运行CentOS 8的系统，并且尝试使用`dnf update`或`yum update`更新包，您将遇到以下错误:`Error: Failed to download metadata for repo 'appstream': Cannot prepare internal mirrorlist: No URLs in mirrorlist`

现在CentOS已经转移到Stream(一种滚动发布的Linux发行版，介于[Fedora](https://getfedora.org/)的上游开发和[RHEL](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux)的下游开发之间)，许多用户正在转向[CentOS的替代方案](https://haydenjames.io/what-centos-alternative-distro-should-you-choose/)。其他人则决定通过迁移到[CentOS Stream 8](https://www.centos.org/centos-stream/)来坚持使用CentOS。

CentOS Linux 8早死了，它在2021年12月31日达到了生命的终结(EOL)，因此它不再从官方CentOS项目获得开发资源。

这意味着在2021年12月31日之后，要更新您的CentOS安装，您需要将镜像更改为[CentOS Vault Mirror](https://vault.centos.org/)，在那里它们将被永久存档。

从CentOS 8迁移到[CentOS Stream 8](https://www.centos.org/centos-stream/)，执行以下命令:

```bash
sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
```
或者，您也可以通过运行以下命令指向基于[cloudflare](https://www.cloudflare.com/)的仓库存储库:

```bash
sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-Linux-*
sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.epel.cloud|g' /etc/yum.repos.d/CentOS-Linux-*
```
继续更新软件

```bash
$ dnf -y upgrade
CentOS-8 - AppStream                                                                    2.1 MB/s | 8.4 MB     00:03
CentOS-8 - Base                                                                         1.8 MB/s | 4.6 MB     00:02
CentOS-8 - Extras                                                                       917  B/s |  10 kB     00:11
Dependencies resolved.
========================================================================================================================
 Package                                Architecture    Version                                Repository          Size
========================================================================================================================
Installing:
 centos-linux-release                   noarch          8.5-1.2111.el8                         BaseOS              22 k
     replacing  centos-release.x86_64 8.2-2.2004.0.1.el8
     replacing  centos-repos.x86_64 8.2-2.2004.0.1.el8
 fwupd                                  x86_64          1.5.9-1.el8_4                          BaseOS             2.8 M
     replacing  dbxtool.x86_64 8-5.el8
 kernel                                 x86_64          4.18.0-348.7.1.el8_5                   BaseOS             7.0 M
 kernel-core                            x86_64          4.18.0-348.7.1.el8_5                   BaseOS              38 M
 kernel-modules                         x86_64          4.18.0-348.7.1.el8_5                   BaseOS              30 M
Upgrading:
 NetworkManager                         x86_64          1:1.32.10-4.el8                        BaseOS             2.6 M
 NetworkManager-libnm                   x86_64          1:1.32.10-4.el8                        BaseOS             1.8 M
 NetworkManager-team                    x86_64          1:1.32.10-4.el8                        BaseOS             148 k
 NetworkManager-tui                     x86_64          1:1.32.10-4.el8                        BaseOS             336 k
 authselect                             x86_64          1.2.2-3.el8                            BaseOS             133 k
 authselect-libs                        x86_64          1.2.2-3.el8                            BaseOS             222 k
 bash                                   x86_64          4.4.20-2.el8                           BaseOS             1.5 M
 bind-export-libs                       x86_64          32:9.11.26-6.el8                       BaseOS             1.1 M

....................

Upgraded:
  NetworkManager-1:1.32.10-4.el8.x86_64                       NetworkManager-libnm-1:1.32.10-4.el8.x86_64
  NetworkManager-team-1:1.32.10-4.el8.x86_64                  NetworkManager-tui-1:1.32.10-4.el8.x86_64
  authselect-1.2.2-3.el8.x86_64                               authselect-libs-1.2.2-3.el8.x86_64
  bash-4.4.20-2.el8.x86_64                                    bind-export-libs-32:9.11.26-6.el8.x86_64
  brotli-1.0.6-3.el8.x86_64                                   ca-certificates-2021.2.50-80.0.el8_4.noarch
  centos-gpg-keys-1:8-3.el8.noarch                            chkconfig-1.19.1-1.el8.x86_64
  coreutils-8.30-12.el8.x86_64                                coreutils-common-8.30-12.el8.x86_64
  cpio-2.12-10.el8.x86_64                                     crontabs-1.11-17.20190603git.el8.noarch
  crypto-policies-20210617-1.gitc776d3e.el8.noarch            cryptsetup-libs-2.3.3-4.el8.x86_64
  curl-7.61.1-22.el8.x86_64                                   cyrus-sasl-lib-2.1.27-5.el8.x86_64
  dbus-1:1.12.8-14.el8.x86_64                                 dbus-common-1:1.12.8-14.el8.noarch
  dbus-daemon-1:1.12.8-14.el8.x86_64                          dbus-libs-1:1.12.8-14.el8.x86_64
  dbus-tools-1:1.12.8-14.el8.x86_64                           device-mapper-8:1.02.177-10.el8.x86_64
  device-mapper-event-8:1.02.177-10.el8.x86_64                device-mapper-event-libs-8:1.02.177-10.el8.x86_64
  device-mapper-libs-8:1.02.177-10.el8.x86_64                 device-mapper-persistent-data-0.9.0-4.el8.x86_64
  dhcp-client-12:4.3.6-45.el8.x86_64                          dhcp-common-12:4.3.6-45.el8.noarch
  dhcp-libs-12:4.3.6-45.el8.x86_64                            dmidecode-1:3.2-10.el8.x86_64
  dnf-4.7.0-4.el8.noarch                                      dnf-data-4.7.0-4.el8.noarch
  dnf-plugins-core-4.0.21-3.el8.noarch                        dracut-049-191.git20210920.el8.x86_64
  dracut-config-rescue-049-191.git20210920.el8.x86_64         dracut-network-049-191.git20210920.el8.x86_64
  dracut-squash-049-191.git20210920.el8.x86_64                e2fsprogs-1.45.6-2.el8.x86_64
  e2fsprogs-libs-1.45.6-2.el8.x86_64                          efi-filesystem-3-3.el8.noarch
  efivar-37-4.el8.x86_64                                      efivar-libs-37-4.el8.x86_64
  elfutils-default-yama-scope-0.185-1.el8.noarch              elfutils-libelf-0.185-1.el8.x86_64
  elfutils-libs-0.185-1.el8.x86_64                            ethtool-2:5.8-7.el8.x86_64
  expat-2.2.5-4.el8.x86_64                                    file-5.33-20.el8.x86_64
  file-libs-5.33-20.el8.x86_64                                filesystem-3.8-6.el8.x86_64
  firewalld-0.9.3-7.el8.noarch                                firewalld-filesystem-0.9.3-7.el8.noarch
  freetype-2.9.1-4.el8_3.1.x86_64                             gawk-4.2.1-2.el8.x86_64
  glib2-2.56.4-156.el8.x86_64                                 glibc-2.28-164.el8.x86_64
  glibc-all-langpacks-2.28-164.el8.x86_64                     glibc-common-2.28-164.el8.x86_64
  gnupg2-2.2.20-2.el8.x86_64                                  gnupg2-smime-2.2.20-2.el8.x86_64
  gnutls-3.6.16-4.el8.x86_64                                  gpgme-1.13.1-9.el8.x86_64
  grub2-common-1:2.02-106.el8.noarch                          grub2-efi-x64-1:2.02-106.el8.x86_64
  grub2-pc-1:2.02-106.el8.x86_64                              grub2-pc-modules-1:2.02-106.el8.noarch
  grub2-tools-1:2.02-106.el8.x86_64                           grub2-tools-extra-1:2.02-106.el8.x86_64
  grub2-tools-minimal-1:2.02-106.el8.x86_64                   grubby-8.40-42.el8.x86_64
  gzip-1.9-12.el8.x86_64                                      hdparm-9.54-4.el8.x86_64
  hwdata-0.314-8.10.el8.noarch                                ima-evm-utils-1.3.2-12.el8.x86_64
  initscripts-10.00.15-1.el8.x86_64                           iproute-5.12.0-4.el8.x86_64
  iprutils-2.4.19-1.el8.x86_64                                iptables-1.8.4-20.el8.x86_64
  iptables-ebtables-1.8.4-20.el8.x86_64                       iptables-libs-1.8.4-20.el8.x86_64
  iputils-20180629-7.el8.x86_64                               irqbalance-2:1.4.0-6.el8.x86_64
  iwl100-firmware-39.31.5.1-103.el8.1.noarch                  iwl1000-firmware-1:39.31.5.1-103.el8.1.noarch
  iwl105-firmware-18.168.6.1-103.el8.1.noarch                 iwl135-firmware-18.168.6.1-103.el8.1.noarch
  iwl2000-firmware-18.168.6.1-103.el8.1.noarch                iwl2030-firmware-18.168.6.1-103.el8.1.noarch
  iwl3160-firmware-1:25.30.13.0-103.el8.1.noarch              iwl3945-firmware-15.32.2.9-103.el8.1.noarch
  iwl4965-firmware-228.61.2.24-103.el8.1.noarch               iwl5000-firmware-8.83.5.1_1-103.el8.1.noarch
  iwl5150-firmware-8.24.2.2-103.el8.1.noarch                  iwl6000-firmware-9.221.4.1-103.el8.1.noarch
  iwl6000g2a-firmware-18.168.6.1-103.el8.1.noarch             iwl6050-firmware-41.28.5.1-103.el8.1.noarch
  iwl7260-firmware-1:25.30.13.0-103.el8.1.noarch              json-c-0.13.1-2.el8.x86_64
  kbd-2.0.4-10.el8.x86_64                                     kbd-legacy-2.0.4-10.el8.noarch
  kbd-misc-2.0.4-10.el8.noarch                                kernel-tools-4.18.0-348.7.1.el8_5.x86_64
  kernel-tools-libs-4.18.0-348.7.1.el8_5.x86_64               kexec-tools-2.0.20-57.el8_5.1.x86_64
  keyutils-libs-1.5.10-9.el8.x86_64                           kmod-25-18.el8.x86_64
  kmod-libs-25-18.el8.x86_64                                  kpartx-0.8.4-17.el8.x86_64
  krb5-libs-1.18.2-14.el8.x86_64                              libarchive-3.3.3-1.el8.x86_64
  libblkid-2.32.1-28.el8.x86_64                               libcap-2.26-5.el8.x86_64
  libcap-ng-0.7.11-1.el8.x86_64                               libcom_err-1.45.6-2.el8.x86_64
  libcomps-0.1.16-2.el8.x86_64                                libcroco-0.6.12-4.el8_2.1.x86_64
  libcurl-7.61.1-22.el8.x86_64                                libdb-5.3.28-42.el8_4.x86_64
  libdb-utils-5.3.28-42.el8_4.x86_64                          libdnf-0.63.0-3.el8.x86_64
  libfdisk-2.32.1-28.el8.x86_64                               libffi-3.1-22.el8.x86_64
  libgcc-8.5.0-4.el8_5.x86_64                                 libgcrypt-1.8.5-6.el8.x86_64
  libgomp-8.5.0-4.el8_5.x86_64                                libkcapi-1.2.0-2.el8.x86_64
  libkcapi-hmaccalc-1.2.0-2.el8.x86_64                        libldb-2.3.0-2.el8.x86_64
  libmodulemd1-1.8.16-0.2.13.0.1.x86_64                       libmount-2.32.1-28.el8.x86_64
  libndp-1.7-6.el8.x86_64                                     libnfsidmap-1:2.3.3-46.el8.x86_64
  libnghttp2-1.33.0-3.el8_2.1.x86_64                          libpcap-14:1.9.1-5.el8.x86_64
  libpsl-0.20.2-6.el8.x86_64                                  libpwquality-1.4.4-3.el8.x86_64
  librepo-1.14.0-2.el8.x86_64                                 libreport-filesystem-2.9.5-15.el8.x86_64
  libseccomp-2.5.1-1.el8.x86_64                               libselinux-2.9-5.el8.x86_64
  libselinux-utils-2.9-5.el8.x86_64                           libsemanage-2.9-6.el8.x86_64
  libsepol-2.9-3.el8.x86_64                                   libsmartcols-2.32.1-28.el8.x86_64
  libsolv-0.7.19-1.el8.x86_64                                 libss-1.45.6-2.el8.x86_64
  libssh-0.9.4-3.el8.x86_64                                   libssh-config-0.9.4-3.el8.noarch
  libsss_autofs-2.5.2-2.el8_5.3.x86_64                        libsss_certmap-2.5.2-2.el8_5.3.x86_64
  libsss_idmap-2.5.2-2.el8_5.3.x86_64                         libsss_nss_idmap-2.5.2-2.el8_5.3.x86_64
  libsss_sudo-2.5.2-2.el8_5.3.x86_64                          libstdc++-8.5.0-4.el8_5.x86_64
  libtalloc-2.3.2-1.el8.x86_64                                libtdb-1.4.3-1.el8.x86_64
  libteam-1.31-2.el8.x86_64                                   libtevent-0.11.0-0.el8.x86_64
  libtirpc-1.1.4-5.el8.x86_64                                 libusbx-1.0.23-4.el8.x86_64
  libuuid-2.32.1-28.el8.x86_64                                libxcrypt-4.1.1-6.el8.x86_64
  libxml2-2.9.7-9.el8_4.2.x86_64                              libzstd-1.4.4-1.el8.x86_64
  linux-firmware-20210702-103.gitd79c2677.el8.noarch          lshw-B.02.19.2-6.el8.x86_64
  lsscsi-0.32-3.el8.x86_64                                    lua-libs-5.3.4-12.el8.x86_64
  lvm2-8:2.03.12-10.el8.x86_64                                lvm2-libs-8:2.03.12-10.el8.x86_64
  lz4-libs-1.8.3-3.el8_4.x86_64                               man-db-2.7.6.1-18.el8.x86_64
  microcode_ctl-4:20210608-1.el8.x86_64                       mokutil-1:0.3.0-11.el8.x86_64
  ncurses-6.1-9.20180224.el8.x86_64                           ncurses-base-6.1-9.20180224.el8.noarch
  ncurses-libs-6.1-9.20180224.el8.x86_64                      nettle-3.4.1-7.el8.x86_64
  nftables-1:0.9.3-21.el8.x86_64                              numactl-libs-2.0.12-13.el8.x86_64
  openldap-2.4.46-18.el8.x86_64                               openssh-8.0p1-10.el8.x86_64
  openssh-clients-8.0p1-10.el8.x86_64                         openssh-server-8.0p1-10.el8.x86_64
  openssl-1:1.1.1k-5.el8_5.x86_64                             openssl-libs-1:1.1.1k-5.el8_5.x86_64
  os-prober-1.74-9.el8.x86_64                                 p11-kit-0.23.22-1.el8.x86_64
  p11-kit-trust-0.23.22-1.el8.x86_64                          pam-1.3.1-15.el8.x86_64
  parted-3.2-39.el8.x86_64                                    pciutils-libs-3.7.0-1.el8.x86_64
  pcre-8.42-6.el8.x86_64                                      pcre2-10.32-2.el8.x86_64
  platform-python-3.6.8-41.el8.x86_64                         platform-python-pip-9.0.3-20.el8.noarch
  platform-python-setuptools-39.2.0-6.el8.noarch              policycoreutils-2.9-16.el8.x86_64
  polkit-0.115-12.el8.x86_64                                  polkit-libs-0.115-12.el8.x86_64
  popt-1.18-1.el8.x86_64                                      procps-ng-3.3.15-6.el8.x86_64
  python3-dnf-4.7.0-4.el8.noarch                              python3-dnf-plugins-core-4.0.21-3.el8.noarch
  python3-firewall-0.9.3-7.el8.noarch                         python3-gobject-base-3.28.3-2.el8.x86_64
  python3-gpg-1.13.1-9.el8.x86_64                             python3-hawkey-0.63.0-3.el8.x86_64
  python3-libcomps-0.1.16-2.el8.x86_64                        python3-libdnf-0.63.0-3.el8.x86_64
  python3-libs-3.6.8-41.el8.x86_64                            python3-libselinux-2.9-5.el8.x86_64
  python3-libxml2-2.9.7-9.el8_4.2.x86_64                      python3-linux-procfs-0.6.3-1.el8.noarch
  python3-nftables-1:0.9.3-21.el8.x86_64                      python3-perf-4.18.0-348.7.1.el8_5.x86_64
  python3-pip-wheel-9.0.3-20.el8.noarch                       python3-rpm-4.14.3-19.el8.x86_64
  python3-setuptools-wheel-39.2.0-6.el8.noarch                python3-syspurpose-1.28.21-3.el8.x86_64
  rng-tools-6.13-1.git.d207e0b6.el8.x86_64                    rpm-4.14.3-19.el8.x86_64
  rpm-build-libs-4.14.3-19.el8.x86_64                         rpm-libs-4.14.3-19.el8.x86_64
  rpm-plugin-selinux-4.14.3-19.el8.x86_64                     rpm-plugin-systemd-inhibit-4.14.3-19.el8.x86_64
  sed-4.5-2.el8.x86_64                                        selinux-policy-3.14.3-80.el8_5.2.noarch
  selinux-policy-targeted-3.14.3-80.el8_5.2.noarch            setup-2.12.2-6.el8.noarch
  shadow-utils-2:4.6-14.el8.x86_64                            shim-x64-15-15.el8_2.x86_64
  snappy-1.1.8-3.el8.x86_64                                   sqlite-libs-3.26.0-15.el8.x86_64
  squashfs-tools-4.3-20.el8.x86_64                            sssd-client-2.5.2-2.el8_5.3.x86_64
  sssd-common-2.5.2-2.el8_5.3.x86_64                          sssd-kcm-2.5.2-2.el8_5.3.x86_64
  sssd-nfs-idmap-2.5.2-2.el8_5.3.x86_64                       sudo-1.8.29-7.el8_4.1.x86_64
  systemd-239-51.el8_5.2.x86_64                               systemd-libs-239-51.el8_5.2.x86_64
  systemd-pam-239-51.el8_5.2.x86_64                           systemd-udev-239-51.el8_5.2.x86_64
  teamd-1.31-2.el8.x86_64                                     trousers-0.3.15-1.el8.x86_64
  trousers-lib-0.3.15-1.el8.x86_64                            tuned-2.16.0-1.el8.noarch
  tzdata-2021e-1.el8.noarch                                   util-linux-2.32.1-28.el8.x86_64
  vim-minimal-2:8.0.1763-16.el8.x86_64                        virt-what-1.18-12.el8.x86_64
  which-2.21-16.el8.x86_64                                    xfsprogs-5.0.0-9.el8.x86_64
  yum-4.7.0-4.el8.noarch                                      zlib-1.2.11-17.el8.x86_64

Installed:
  ModemManager-glib-1.10.8-4.el8.x86_64                            bubblewrap-0.4.0-1.el8.x86_64
  centos-linux-release-8.5-1.2111.el8.noarch                       centos-linux-repos-8-3.el8.noarch
  crypto-policies-scripts-20210617-1.gitc776d3e.el8.noarch         elfutils-debuginfod-client-0.185-1.el8.x86_64
  fwupd-1.5.9-1.el8_4.x86_64                                       gdisk-1.0.3-6.el8.x86_64
  grub2-tools-efi-1:2.02-106.el8.x86_64                            json-glib-1.4.4-1.el8.x86_64
  kernel-4.18.0-348.7.1.el8_5.x86_64                               kernel-core-4.18.0-348.7.1.el8_5.x86_64
  kernel-modules-4.18.0-348.7.1.el8_5.x86_64                       libatasmart-0.19-14.el8.x86_64
  libblockdev-2.24-7.el8.x86_64                                    libblockdev-crypto-2.24-7.el8.x86_64
  libblockdev-fs-2.24-7.el8.x86_64                                 libblockdev-loop-2.24-7.el8.x86_64
  libblockdev-mdraid-2.24-7.el8.x86_64                             libblockdev-part-2.24-7.el8.x86_64
  libblockdev-swap-2.24-7.el8.x86_64                               libblockdev-utils-2.24-7.el8.x86_64
  libbpf-0.4.0-1.el8.x86_64                                        libbytesize-1.4-3.el8.x86_64
  libevent-2.1.8-5.el8.x86_64                                      libgcab1-1.1-1.el8.x86_64
  libgudev-232-4.el8.x86_64                                        libgusb-0.3.0-1.el8.x86_64
  libibverbs-35.0-1.el8.x86_64                                     libmbim-1.20.2-1.el8.x86_64
  libmodulemd-2.13.0-1.el8.x86_64                                  libqmi-1.24.0-3.el8.x86_64
  libsecret-0.18.6-1.el8.x86_64                                    libsmbios-2.4.1-2.el8.x86_64
  libudisks2-2.9.0-7.el8.x86_64                                    libxkbcommon-0.9.1-1.el8.x86_64
  libxmlb-0.1.15-1.el8.x86_64                                      lmdb-libs-0.9.24-1.el8.x86_64
  mdadm-4.2-rc2.el8.x86_64                                         memstrack-0.1.11-1.el8.x86_64
  nspr-4.32.0-1.el8_4.x86_64                                       nss-3.67.0-7.el8_5.x86_64
  nss-softokn-3.67.0-7.el8_5.x86_64                                nss-softokn-freebl-3.67.0-7.el8_5.x86_64
  nss-sysinit-3.67.0-7.el8_5.x86_64                                nss-util-3.67.0-7.el8_5.x86_64
  pciutils-3.7.0-1.el8.x86_64                                      pinentry-1.1.0-2.el8.x86_64
  python3-unbound-1.7.3-17.el8.x86_64                              rdma-core-35.0-1.el8.x86_64
  tpm2-tss-2.3.2-4.el8.x86_64                                      udisks2-2.9.0-7.el8.x86_64
  unbound-libs-1.7.3-17.el8.x86_64                                 volume_key-libs-0.3.11-5.el8.x86_64
  xkeyboard-config-2.28-1.el8.noarch

Complete!
```

##  安装软件

```bash
dnf -y install vim wget bash-completion net-tools
```

##  使用Moduler存储库

```bash
$ dnf module list
Last metadata expiration check: 0:01:21 ago on Tue 01 Nov 2022 10:10:32 PM CST.
CentOS Linux 8 - AppStream
Name                 Stream          Profiles Summary
389-ds               1.4                      389 Directory Server (base)
ant                  1.10 [d]        common [ Java build tool
                                     d]
container-tools      rhel8 [d]       common [ Most recent (rolling) versions of podman, buildah, skopeo, runc, conmon, r
                                     d]       unc, conmon, CRIU, Udica, etc as well as dependencies such as container-se
                                              linux built and tested together, and updated as frequently as every 12 wee
                                              ks.
container-tools      1.0             common [ Stable versions of podman 1.0, buildah 1.5, skopeo 0.1, runc, conmon, CRIU
                                     d]       , Udica, etc as well as dependencies such as container-
subversion           1.14            common [ Apache Subversion
                                     d], serv
                                     er
swig                 3.0 [d]         common [ Connects C/C++/Objective C to some high-level programming languages
                                     d], comp
                                     lete
swig                 4.0             common [ Connects C/C++/Objective C to some high-level programming languages
                                     d], comp
                                     lete
.......................
.......................
varnish              6 [d]           common [ Varnish HTTP cache
                                     d]
virt                 rhel [d]        common [ Virtualization module
                                     d]

Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled

```
要安装可用模块，请按如下所示进行配置。
```bash
$  dnf module list postgresql
Last metadata expiration check: 0:13:08 ago on Tue 01 Nov 2022 10:10:32 PM CST.
CentOS Linux 8 - AppStream
Name                   Stream             Profiles                       Summary
postgresql             9.6                client, server [d]             PostgreSQL server and client module
postgresql             10 [d]             client, server [d]             PostgreSQL server and client module
postgresql             12                 client, server [d]             PostgreSQL server and client module
postgresql             13                 client, server [d]             PostgreSQL server and client module

Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled


$ dnf module install -y postgresql:10
$  dnf module list postgresql
Last metadata expiration check: 0:15:17 ago on Tue 01 Nov 2022 10:10:32 PM CST.
CentOS Linux 8 - AppStream
Name                  Stream              Profiles                         Summary
postgresql            9.6                 client, server [d]               PostgreSQL server and client module
postgresql            10 [d][e]           client, server [d] [i]           PostgreSQL server and client module
postgresql            12                  client, server [d]               PostgreSQL server and client module
postgresql            13                  client, server [d]               PostgreSQL server and client module

Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled
```
如果您想更改为已安装模块的另一个版本，请按以下步骤进行配置。
例如，从上面[2]上安装的`PostgreSQL 10`切换到`PostgreSQL 9.6`。

```bash
$ dnf module reset -y  postgresql  
$ dnf module install  -y postgresql:9.6

＃[PostgreSQL 9.6]的状态变为[e]启用
$  dnf module list postgresql
Last metadata expiration check: 0:18:06 ago on Tue 01 Nov 2022 10:10:32 PM CST.
CentOS Linux 8 - AppStream
Name                  Stream             Profiles                          Summary
postgresql            9.6 [e]            client, server [d] [i]            PostgreSQL server and client module
postgresql            10 [d]             client, server [d]                PostgreSQL server and client module
postgresql            12                 client, server [d]                PostgreSQL server and client module
postgresql            13                 client, server [d]                PostgreSQL server and client module

Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled
```

##  更新阿里云 yum 源

```bash
mv /etc/yum.repos.d /etc/yum.repos.d.bak # 先备份原有的 Yum 源
mkdir /etc/yum.repos.d
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-vault-8.5.2111.repo
yum clean all && yum makecache
```

## 用户管理
1. 要在CentOS服务器上添加普通用户帐户，请按以下步骤设置。

```bash
$ useradd centos
$ passwd centos
Changing password for user centos.
New password:                               ＃输入您要设置的任何密码  
Retype new password:
passwd: all authentication tokens updated successfully.$所有身份验证令牌已成功更新
```


2. 如果您想从普通用户切换到root用户帐户，请使用[su]命令。

```bash
[root@localhost ~]# su - centos  $切换centos账号
[centos@localhost ~]$ su -       $切换root账号
Password:                       $输入root密码
[root@localhost ~]#              $切换到root账号
```

3. 如果您想限制用户运行[su]命令，请进行如下设置。
在以下示例中，只有[wheel]组中的用户可以运行[su]命令。

```bash
[root@localhost ~]# usermod -G wheel centos
[root@localhost ~]# vi /etc/pam.d/su
[root@localhost ~]# cat  /etc/pam.d/su
#%PAM-1.0
auth            required        pam_env.so
auth            sufficient      pam_rootok.so
# Uncomment the following line to implicitly trust users in the "wheel" group.
#auth           sufficient      pam_wheel.so trust use_uid  $我们添加的配置项
# Uncomment the following line to require a user to be in the "wheel" group.
auth            required        pam_wheel.so use_uid
auth            substack        system-auth
auth            include         postlogin
account         sufficient      pam_succeed_if.so uid = 0 use_uid quiet
account         include         system-auth
password        include         system-auth
session         include         system-auth
session         include         postlogin
session         optional        pam_xauth.so
auth          sufficient      pam_rootok.so debug

[root@localhost ~]# groups centos  $查看账号所在的组
centos : centos wheel
```

我们可以创建一个账号`user01`没有在`wheel`组，并尝试切换到`root`账号

```bash
[root@localhost ~]# useradd user01
[root@localhost ~]# passwd user01
Changing password for user user01.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
[root@localhost ~]#
[root@localhost ~]# su - user01
[user01@localhost ~]$
[user01@localhost ~]$ su -
Password:
su: Permission denied  $通常是被拒绝的
[user01@localhost ~]$
```



4. 如果您要删除用户帐户，请按以下步骤设置。

```bash
[root@localhost ~]# userdel -r user01  ＃删除用户[user01]（仅删除的用户帐户）
userdel: user 'user01' does not exist
[root@localhost ~]# ll /home/
total 4
drwxr-xr-x.  3 admin    admin      78 Sep 28 10:09 admin
drwx------.  4 centos   centos    113 Dec  4 13:56 centos
drwx------. 15 localhost localhost 4096 Sep 27 16:42 localhost
drwx------.  3 tddev    users      78 Sep 28 10:09 tddev
drwx------.  5 tdops    users     143 Oct 15 16:10 tdops
drwx------.  3 tdsec    users      78 Sep 28 10:09 tdsec
drwx------.  4     1006     1006  113 Dec  4 14:16 user01
＃删除用户[user01]（已删除的用户帐户和他的主目录）
[root@localhost ~]# userdel -r user01
userdel: user 'user01' does not exist
[root@localhost ~]# userdel -r localhost
```

5. 添加到`wheel`组用户免密切换`root`账号设置步骤。

```bash
[root@localhost ~]#  vi /etc/sudoers

## Same thing without a password
%wheel        ALL=(ALL)       NOPASSWD: ALL $添加这段内容后，wheel组用户，切换到root不需要知道root密码。



[root@localhost ~]# su - centos
[centos@localhost ~]$ id
uid=1005(centos) gid=1005(centos) groups=1005(centos),10(wheel) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[centos@localhost ~]$
[centos@localhost ~]$ sudo su - $#免密切换到root账号。
[root@localhost ~]#
```
##  给用户添加sudo权限

```bash
$ sed -i '/^root.*ALL=(ALL).*ALL/a\going\tALL=(ALL) \tALL' /etc/sudoers

$ cat /etc/sudoers
root    ALL=(ALL)       ALL
going   ALL=(ALL)       ALL
```

##  Cockpit 管理控制台
- [各类系统安装 cockpit 包](https://cockpit-project.org/running.html)

```bash
dnf install -y cockpit
systemctl enable --now cockpit.socket
systemctl status --now cockpit.socket
```
- 登陆：https://192.168.10.14:9090
- 用户名：root
- 密码：<你设置的主机密码>

![在这里插入图片描述](https://img-blog.csdnimg.cn/60063612e9ee4034992a98e4859538f4.png)

概览
![在这里插入图片描述](https://img-blog.csdnimg.cn/42291094f75f4f1fbc51d79d4f4638e1.png)
磁盘、cpu、内存、网络监控
![在这里插入图片描述](https://img-blog.csdnimg.cn/833ae32e98364f8f93934690f6d2ce8b.png)
安装软件
![在这里插入图片描述](https://img-blog.csdnimg.cn/1a7b90b17410458ab2ffaed424238161.gif#pic_center)




日志
![在这里插入图片描述](https://img-blog.csdnimg.cn/a588cca5157a4599ac98ccb17f683c7c.png)
磁盘存储
![在这里插入图片描述](https://img-blog.csdnimg.cn/c01255283b67423891693d2b76270e52.png)

网络
![在这里插入图片描述](https://img-blog.csdnimg.cn/22877a6ef4434ea0827f862c058d0ac9.png)
用户
![在这里插入图片描述](https://img-blog.csdnimg.cn/f8fd01e30c224107b8cbd759938a1477.png)
服务
![在这里插入图片描述](https://img-blog.csdnimg.cn/10a512d6e3284b9d94b78494c72920dd.png)
应用
![在这里插入图片描述](https://img-blog.csdnimg.cn/873386cf704b4779bc229833740aee61.png)
诊断报表
![在这里插入图片描述](https://img-blog.csdnimg.cn/0002acb6831c4a43a7a910428245acd4.png)


kdump
![在这里插入图片描述](https://img-blog.csdnimg.cn/ea4116041cb14c2aac60cf87543f2a6c.png)
软件更新
![在这里插入图片描述](https://img-blog.csdnimg.cn/62155f16603e494aac7ac522efccb8d0.png)
界面终端
![在这里插入图片描述](https://img-blog.csdnimg.cn/ffe202b607054c248d9ae7e38a137389.png)

