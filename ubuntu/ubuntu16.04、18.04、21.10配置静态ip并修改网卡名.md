

---

 - [ubuntu16.04、18.04、21.10配置静态ip并修改网卡名](https://ghostwritten.blog.csdn.net/article/details/105641217)
 - [ubuntu安装配置NFS](https://ghostwritten.blog.csdn.net/article/details/117217493)
 -  [ubuntu ssh详解](https://ghostwritten.blog.csdn.net/article/details/105639355)
 - [ubuntu apt-get镜像源](https://ghostwritten.blog.csdn.net/article/details/110931517)
 - [ubutntu iso下载](https://al.mirror.kumi.systems/ubuntureleases/)

---

## ubuntu 16.04
```bash
$ sudo vim /etc/network/interfaces
auto ens33
iface ens33 inet static
address 192.168.211.30
netmask 255.255.255.0
gateway 192.168.211.2
dns-nameservers 192.168.211.2

sudo systemctl restart networking.service
sudo reboot
```
## ubuntu 18.04
ubuntu18.04LTS设置静态IP
因为Ubuntu18.04采用的是netplan来管理network。所以可以在`/etc/netplan/`目录下创建
一个以yaml结尾的文件。比如`01-netplan.yaml`文件。
然后在此文件下写入以下配置：

```bash
network:
  version: 2
  renderer: networkd
  ethernets:
    enp3s0:
      dhcp4: no
      addresses: [192.168.1.110/24]
      gateway4:  192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 114.114.114.114]
```

特别要注意的是这里的每一行的空格一定要有的，否则会报错误而设置失败！
最后使用`sudo netplan apply`来重启网络服务就可以了。使用ip a查看你的静态IP是否设置成功了！

禁用 IPv6

```bash
sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1
sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1
sudo sysctl -w net.ipv6.conf.lo.disable_ipv6=1
```
或者
将以下 3 行内容添加到 /etc/sysctl.conf 配置文件当中：

```bash
net.ipv6.conf.all.disable_ipv6=1
net.ipv6.conf.default.disable_ipv6=1
net.ipv6.conf.lo.disable_ipv6=1
```

```bash
sysctl -p
```
## ubuntu 21.10

```bash
$ cat /etc/netplan/01-netplan.yaml 
network:
  ethernets:
    ens33:
      dhcp4: no
      addresses: [192.168.211.80/24]
      routes:
        - to: default
          via:  192.168.211.2
      nameservers:
        addresses: [8.8.8.8, 114.114.114.114]
  version: 2
  renderer: networkd
```
重启

```bash
#检查配置
netplan try
netplan try --state /etc/netplan

#重启
netplan apply
```
注意你的网卡名是ens33、enp3s0、eth0，还是其他，保持跟默认网卡名一直。

## 修改网卡名

```bash
$ vi /etc/default/grub
GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"

#启动生效
$ update-grub

#修改配置
$ cat /etc/netplan/01-netplan.yaml 
network:
  ethernets:
    eth0:
      dhcp4: no
      addresses: [192.168.211.80/24]
      routes:
        - to: default
          via:  192.168.211.2
      nameservers:
        addresses: [8.8.8.8, 114.114.114.114]
  version: 2
  renderer: networkd


#重启生效
$ netplan apply
$ sudo reboot
```

