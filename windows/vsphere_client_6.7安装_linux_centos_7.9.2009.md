

##  1. 界面安装
![在这里插入图片描述](https://img-blog.csdnimg.cn/9e3608e6304344c486093b900c5e4d85.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/650b65a87dcd4d45ba2868e4dc4e004c.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/efe12582f01440d497da548bf13542db.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b3d3926b1eaf43f4a22b841f9aa31ea6.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/fe941863a2174d9596dc2160f7639da5.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/f0e0fd1ea9f342779feb8c310222aad5.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/1cb3ca71847443268688b5825166ebb7.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/ce64d5988c874542b5c49ecae83dbdb1.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/9c298bf2c32e4937b42b22681ef9fed8.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/322c15efe33f499fa6aab496764141b6.png)
重启reboot
![在这里插入图片描述](https://img-blog.csdnimg.cn/c236fb841e12443cbea0bf678d3fd390.png)

```bash
[root@localhost ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens192
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
UUID=d06ef1c4-cc10-4f3f-a483-f84480fb267e
DEVICE=ens192
ONBOOT=yes
IPADDR=192.168.10.13
GATEWAY=192.168.10.1
NETMASK=255.255.252.0
DNS1=8.8.8.8
```

## 2. 基础配置

```bash
yum -y install vim wget curl bash-complation

systemctl stop firewalld && systemctl disable firewalld
systemctl stop NetworkManager && systemctl disalbe NetworkManager
setenforce 0 && getenforce
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

 - [centos 配置静态地址并修改网卡名](https://blog.csdn.net/xixihahalelehehe/article/details/124737800)

