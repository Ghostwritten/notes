
![在这里插入图片描述](https://img-blog.csdnimg.cn/912442790041453e84b67d0eef32c35a.png)




## 1. 查看操作系统内核版本

```bash
$  uname -r
3.10.0-1062.4.1.el7.x86_64
```

##  2. 安装 elrepo
官网地址：[http://elrepo.org/tiki/tiki-index.php](http://elrepo.org/tiki/tiki-index.php)

```bash
yum -y update && yum -y upgrade 
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
yum -y install https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm
```
 CentOS 8则采用下面的命令
```bash
# yum install -y https://www.elrepo.org/elrepo-release-8.el8.elrepo.noarch.rpm
```
##  3. 查看内核版本列表

```bash
$ yum --disablerepo="*" --enablerepo="elrepo-kernel" list available
Loading mirror speeds from cached hostfile
 * elrepo-kernel: mirrors.tuna.tsinghua.edu.cn
elrepo-kernel                                                                                                                                                                                                                        | 3.0 kB  00:00:00     
elrepo-kernel/primary_db                                                                                                                                                                                                             | 2.0 MB  00:00:04     
可安装的软件包
kernel-lt.x86_64                                                                                                              5.4.162-1.el7.elrepo                                                                                             elrepo-kernel
kernel-lt-devel.x86_64                                                                                                        5.4.162-1.el7.elrepo                                                                                             elrepo-kernel
kernel-lt-doc.noarch                                                                                                          5.4.162-1.el7.elrepo                                                                                             elrepo-kernel
kernel-lt-headers.x86_64                                                                                                      5.4.162-1.el7.elrepo                                                                                             elrepo-kernel
kernel-lt-tools.x86_64                                                                                                        5.4.162-1.el7.elrepo                                                                                             elrepo-kernel
kernel-lt-tools-libs.x86_64                                                                                                   5.4.162-1.el7.elrepo                                                                                             elrepo-kernel
kernel-lt-tools-libs-devel.x86_64                                                                                             5.4.162-1.el7.elrepo                                                                                             elrepo-kernel
kernel-ml.x86_64                                                                                                              5.15.5-1.el7.elrepo                                                                                              elrepo-kernel
kernel-ml-devel.x86_64                                                                                                        5.15.5-1.el7.elrepo                                                                                              elrepo-kernel
kernel-ml-doc.noarch                                                                                                          5.15.5-1.el7.elrepo                                                                                              elrepo-kernel
kernel-ml-headers.x86_64                                                                                                      5.15.5-1.el7.elrepo                                                                                              elrepo-kernel
kernel-ml-tools.x86_64                                                                                                        5.15.5-1.el7.elrepo                                                                                              elrepo-kernel
kernel-ml-tools-libs.x86_64                                                                                                   5.15.5-1.el7.elrepo                                                                                              elrepo-kernel
kernel-ml-tools-libs-devel.x86_64                                                                                             5.15.5-1.el7.elrepo                                                                                              elrepo-kernel
perf.x86_64                                                                                                                   5.15.5-1.el7.elrepo                                                                                              elrepo-kernel
python-perf.x86_64                                                                                                            5.15.5-1.el7.elrepo                                                                                              elrepo-kernel
```

##  4. 安装

```bash
$ yum --enablerepo=elrepo-kernel install kernel-ml-devel kernel-ml -y
已加载插件：fastestmirror, product-id, search-disabled-repos, subscription-manager

This system is not registered with an entitlement server. You can use subscription-manager to register.

Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * elrepo: mirrors.neusoft.edu.cn
 * elrepo-kernel: mirrors.neusoft.edu.cn
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com

https://wiki.centos.org/yum-errors

If above article doesn't help to resolve this issue please use https://bugs.centos.org/.

elrepo                                                                                                                                                                                                                               | 3.0 kB  00:00:00     
elrepo/primary_db                                                                                                                                                                                                                    | 501 kB  00:00:00     
正在解决依赖关系
--> 正在检查事务
---> 软件包 kernel-ml.x86_64.0.5.15.5-1.el7.elrepo 将被 安装
---> 软件包 kernel-ml-devel.x86_64.0.5.15.5-1.el7.elrepo 将被 安装
--> 解决依赖关系完成

依赖关系解决

============================================================================================================================================================================================================================================================
 Package                                                        架构                                                  版本                                                               源                                                            大小
============================================================================================================================================================================================================================================================
正在安装:
 kernel-ml                                                      x86_64                                                5.15.5-1.el7.elrepo                                                elrepo-kernel                                                 55 M
 kernel-ml-devel                                                x86_64                                                5.15.5-1.el7.elrepo                                                elrepo-kernel                                                 14 M

事务概要
============================================================================================================================================================================================================================================================
安装  2 软件包

总下载量：69 M
安装大小：306 M
Downloading packages:
(1/2): kernel-ml-5.15.5-1.el7.elrepo.x86_64.rpm                                                                                                                                                                                      |  55 MB  00:00:08     
(2/2): kernel-ml-devel-5.15.5-1.el7.elrepo.x86_64.rpm                                                                                                                                                                                |  14 MB  00:03:32     
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
总计                                                                                                                                                                                                                        330 kB/s |  69 MB  00:03:33     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在安装    : kernel-ml-devel-5.15.5-1.el7.elrepo.x86_64                                                                                                                                                                                              1/2 
  正在安装    : kernel-ml-5.15.5-1.el7.elrepo.x86_64                                                                                                                                                                                                    2/2 
  验证中      : kernel-ml-5.15.5-1.el7.elrepo.x86_64                                                                                                                                                                                                    1/2 
  验证中      : kernel-ml-devel-5.15.5-1.el7.elrepo.x86_64                                                                                                                                                                                              2/2 

已安装:
  kernel-ml.x86_64 0:5.15.5-1.el7.elrepo                                                                                    kernel-ml-devel.x86_64 0:5.15.5-1.el7.elrepo                                                                                   

完毕！
```
##  5. 查看系统上面可以使用的内核

```bash
$ awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
0 : CentOS Linux (5.15.5-1.el7.elrepo.x86_64) 7 (Core)
1 : CentOS Linux (3.10.0-1160.45.1.el7.x86_64) 7 (Core)
2 : CentOS Linux (3.10.0-957.1.3.el7.x86_64) 7 (Core)
3 : CentOS Linux (3.10.0-693.el7.x86_64) 7 (Core)
4 : CentOS Linux (0-rescue-8c140402428842fe94fb7aa44b8df5d0) 7 (Core)
```
##  6. 修改启动顺序默认值
方法1：命令操作
```bash
$ grub2-set-default 0　
```
方法2：修改`/etc/default/grub` 文件
设置 `GRUB_DEFAULT=0`，通过上面查询显示的编号为 0 的内核作为默认内核：

```bash
sed -i s/saved/0/g /etc/default/grub
```
或者
```bash
$ vim /etc/default/grub
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved  #saved改为0
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet net.ifnames=0 biosdevname=0"
GRUB_DISABLE_RECOVERY="true"
```
## 7. 生产grub 配置文件

```bash
$ grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.15.5-1.el7.elrepo.x86_64
Found initrd image: /boot/initramfs-5.15.5-1.el7.elrepo.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-1160.45.1.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-1160.45.1.el7.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-957.1.3.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-957.1.3.el7.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-693.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-693.el7.x86_64.img
Found linux image: /boot/vmlinuz-0-rescue-8c140402428842fe94fb7aa44b8df5d0
Found initrd image: /boot/initramfs-0-rescue-8c140402428842fe94fb7aa44b8df5d0.img
done
```
## 8. 重新启动

```bash
$ reboot
$ uname -r
5.15.5-1.el7.elrepo.x86_64
```
## 9. 删除旧内核版本

```bash
$ rpm -qa | grep kernel
abrt-addon-kerneloops-2.1.11-55.el7.centos.x86_64
kernel-3.10.0-1062.4.1.el7.x86_64
kernel-ml-5.6.7-1.el7.elrepo.x86_64
kernel-debug-devel-3.10.0-1062.4.1.el7.x86_64
kernel-3.10.0-862.el7.x86_64
kernel-tools-libs-3.10.0-1062.4.1.el7.x86_64
kernel-tools-3.10.0-1062.4.1.el7.x86_64
kernel-headers-3.10.0-1062.4.1.el7.x86_64
kernel-ml-devel-5.6.7-1.el7.elrepo.x86_64
```
###  9.1 方法一：使用 yum remove 删除旧版本RPM包

```bash
$ yum remove kernel-3.10.0-1062.4.1.el7.x86_64 kernel-3.10.0-862.el7.x86_64 kernel-tools-3.10.0-1062.4.1.el7.x86_64 kernel-headers-3.10.0-1062.4.1.el7.x86_64
```
### 9.2 方法二： yum-utils 工具

> 注：如果安装的内核不多于 3 个，`yum-utils` 工具不会删除任何一个。只有在安装的内核大于 3 个时，才会自动删除旧内核.

```bash
$ yum install yum-utils 
$ package-cleanup --oldkernels
```


## 10. 一键升级

```bash
yum -y update && yum -y upgrade 
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
yum -y install https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm
yum --disablerepo="*" --enablerepo="elrepo-kernel" list available
yum --enablerepo=elrepo-kernel install kernel-ml-devel kernel-ml -y
awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
grub2-set-default 0　
grub2-mkconfig -o /boot/grub2/grub.cfg
```

