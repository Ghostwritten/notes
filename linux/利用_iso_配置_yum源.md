

##  虚拟机挂载 iso
![在这里插入图片描述](https://img-blog.csdnimg.cn/61208a5de569462c8bdf88afa77b02e2.png)


## 挂载目录

```bash
mkdir /mnt/cdrom
mount -t iso9660 /dev/cdrom /mnt/cdrom

$ df -Th
Filesystem          Type      Size  Used Avail Use% Mounted on
devtmpfs            devtmpfs  3.8G     0  3.8G   0% /dev
tmpfs               tmpfs     3.8G     0  3.8G   0% /dev/shm
tmpfs               tmpfs     3.8G  9.2M  3.8G   1% /run
tmpfs               tmpfs     3.8G     0  3.8G   0% /sys/fs/cgroup
/dev/mapper/rl-root xfs        64G   53G   12G  82% /
/dev/mapper/rl-home xfs        32G  255M   31G   1% /home
/dev/nvme0n1p1      xfs      1014M  212M  803M  21% /boot
overlay             overlay    64G   53G   12G  82% /run/containerd/io.containerd.runtime.v2.task/default/d8f5a4aed9ca461be2739a78a3735cb9c00a23899a7c12d0d30c64d267950825/rootfs
overlay             overlay    64G   53G   12G  82% /run/containerd/io.containerd.runtime.v2.task/default/3fe93f5cfc97d2f31879dac92ec43a5ab783b2af63814da70757515575d61d0f/rootfs
tmpfs               tmpfs     775M     0  775M   0% /run/user/0
/dev/sr0            iso9660    10G   10G     0 100% /mnt/cdrom


$ ls /mnt/cdrom/
AppStream  BaseOS  EFI  images  isolinux  LICENSE  media.repo  TRANS.TBL
```



```bash
$ vim /etc/fstab
/dev/sr0 /mnt/cdrom iso9660 defaults 0 0
```

##  本地配置 yum
本机配置 yum 源
```bash
[BaseOS]
name=BaseOS
baseurl=file:///mnt/cdrom/BaseOS
enable=1
gpgcheck=0

[AppStream]
name=AppStream
baseurl=file:///mnt/cdrom/AppStream
enabled=1
gpgcheck=0
```
## 远程配置 yum

###  httpd 

```bash
$ yum -y install httpd

$ vim /etc/httpd/conf/httpd.conf


listen 81

# vim  /etc/httpd/conf.d/define.conf
<VirtualHost *:81>
        ServerName  www.XXX-ym.com
        DocumentRoot /mnt/cdrom
</VirtualHost>

# vim  /etc/httpd/conf.d/permission.conf
<Directory /mnt/cdrom>
	Require all granted      
</Directory>


$ systemctl  restart  httpd
$ chcon -R --reference=/var/www  /mnt/cdrom  //调整SELinux属性
```

其他节点L:

```bash
$ vim /etc/yum.repos.d/rh7dvd.repo
[rh7dvd]
Name=rh7dvd
Baseurl=http://192.168.4.254/rh7dvd  
Baseurl=ftp://192.168.4.254/rh7dvd  
Baseurl=file///mnt
Enabled=1
Gpgcheck=0
```

### nginx

