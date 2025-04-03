

##  1. 界面安装
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ccf29ef6601b256e7b01a1ee0b6691b1.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d5468a5b09b407f64b6e3345cd22da1e.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f37592f71217bdbce32ea3562b2585e4.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c2f2e0a7806e56d1a2fffcb5f538cc77.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ae5a880a50f967766bef5f6597b1cd0a.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e1cb0ace75eeca6304b785a82707e35e.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/aece3cf56d4253713675cbc5080cdcc3.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0e1b3192cdecb39c7279aa5d8206d475.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0933e00a690dd2f8857afb3fe438490b.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/76ebb5ab34585b683805970191ce58dd.png)
重启reboot
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bbd71d74214caef82b1d674cc1ee2b35.png)

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

