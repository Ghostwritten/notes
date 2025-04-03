

----
## 1 yum介绍

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0512cf4af2db81bfd6a44451d1179ff7.png#pic_center)



[yum](https://man7.org/linux/man-pages/man8/yum.8.html) 是Yellow dog Updater, Modified 的简称，是杜克大学为了提高RPM 软件包安装性而开发的一种软件包管理器。
yum 的理念是使用一个中心仓库(repository)管理一部分甚至一个distribution 的应用程序相互关系，
根据计算出来的软件依赖关系进行相关的升级、安装、删除等等操作，减少了Linux 用户一直头痛的dependencies 的问题。
这一点上，yum 和apt 相同。apt 原为debian 的deb 类型软件管理所使用，但是现在也能用到RedHat 门下的rpm 了。

## 2 Yum 特点：

 - 可以同时配置多个资源库(Repository)
 - 简洁的配置文件(/etc/yum.conf)
 - 自动解决增加或删除rpm包时遇到的倚赖性问题
 - 使用方便
 - 保持与RPM数据库的一致性

## 3 yum命令
概括了部分常用的命令包括：
#自动搜索最快镜像插件：
  
```bash
  yum install yum-fastestmirror
```
#安装yum图形窗口插件：

```bash
 yum install yumex
```
#查看可能批量安装的列表：
  
```bash
  yum grouplist
```
### 3.1. 安装

```bash
yum install  //全部安装
yum install package1 //安装指定的安装包package1
yum groupinsall group1 //安装程序组group1
```

### 3.2. 更新和升级

```bash
yum update //全部更新
yum update package1 //更新指定程序包package1
yum check-update //检查可更新的程序
yum upgrade package1 //升级指定程序包package1
yum groupupdate group1 //升级程序组group1
```

### 3.3. 查找和显示

```bash
yum info package1 //显示安装包信息package1
yum list //显示所有已经安装和可以安装的程序包
yum list package1 //显示指定程序包安装情况package1
yum  list python --showduplicates| sort -r
yum groupinfo group1 //显示程序组group1信息
yum search string //根据关键字string查找安装包
```

### 3.4. 删除程序

```bash
yum remove | erase package1 //删除程序包package1
yum groupremove group1 //删除程序组group1
yum deplist package1 //查看程序package1依赖情况
```

### 3.5. 清除缓存
yum clean packages //清除缓存目录下的软件包

```bash
yum clean headers //清除缓存目录下的 headers
yum clean oldheaders //清除缓存目录下旧的 headers
yum clean, yum clean all (= yum clean packages; yum clean oldheaders) //清除缓存目录下的软件包及旧的headers
```

## 4 yum源类型
本地搭建
开源社区
## 5 yum源使用方式

```bash
ftp
http
file
```

## 6 生成repo文件方式
第一种:直接编辑配置文件

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
第二种：创建快捷配置文件

```bash
$ yum-config-manager --add http://192.168.4.254/rh7dvd ftp://192.168.4.254/rh7dvd  file///mnt
$ echo "gpgcheck=0" > > <yum文件>
```
第三种：写脚本方式

```bash
#！/binbash
rm -rf /etc/yum.repos.d/*
echo"[rhel]
Name=rhel linux
Baseurl=http://xxxxx
Gpgcheck=0
Enabled=1 " > /etc/yum.repos.d/rh7.dvd
$ yum clean all && yum repolist 
```
## 7 制作yum源
### 7.1  自定义yum源

```bash
 createrepo /data/（里面包含根据自己需求囊括的软件包）
 #！/binbash
rm -rf /etc/yum.repos.d/*
echo"[rh7dvd]
Name=zzzz
Baseurl=file:///opt/yum
Gpgcheck=0
Enabled=1 " > /etc/yum.repos.d/data.repo
yum clean all && yum repolist 
```
多台机器公用一个yum源,要在本地搭建httpd
```bash
$ yum -y install httpd

$ vim /etc/httpd/conf/httpd.conf


listen 81

# vim  /etc/httpd/conf.d/define.conf
<VirtualHost *:81>
        ServerName  www.XXX-ym.com
        DocumentRoot /opt/yum
</VirtualHost>

# vim  /etc/httpd/conf.d/permission.conf
<Directory /opt/yum>
	Require all granted      
</Directory>


$ systemctl  restart  httpd
$ chcon -R --reference=/var/www  /opt/yum  //调整SELinux属性
```

### 7.2 用iso搭建yum源

```bash
$ mount -t iso9660 -o,loop /usr/rhel-server-6.5-x86_64-dvd.iso /mnt/cdrom
$ vim /etc/yum.repos.d/iso.repol
name=Red Hat Enterprise Linux - Source
baseurl=file:///mnt/cdrom/
enabled=1
gpgcheck=1
gpgkey=file:///mnt/cdrom/RPM-GPG-KEY-redhat-release
$ yum clean all && yum repolist
```
### 7.3 添加rpm包新添加到yum源

```bash
1，将所有的rpm包拷贝到一个文件夹中
 2，通过rpm命令手工安装createrepo软件
 3，运行命令createrepo -v /rpm-directory
   如果有分组信息，则在运行命令的时候使用-g参数指定分组文件
   createrepo -g /tmp/*comps.xml /rpm-directory
 4,CentOS/RHEL的分组信息保存在光盘repodata/目录下，文件名以comps.xml结尾的xml文件
```

## 8 利用开源yum源

```bash
$ wget -O /etc/yum.repos.d/epel.repo http://mirrors.cloud.tencent.com/repo/epel-7.repo
$ cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg  
EOF
$ yum clean all && yum makecache
```
## 9 yum配置文件
yum的配置文件的详解：

yum 的配置文件分为两部分：`main` 和`repository`

 - main 部分定义了全局配置选项，整个yum 配置文件应该只有一个main。常位于/etc/yum.conf 中。
 - repository 部分定义了每个源/服务器的具体配置，可以有一到多个。常位于/etc/yum.repo.d 目录下的各文件中

```bash
[main]
cachedir=/var/cache/yum   //yum 缓存的目录，yum 在此存储下载的rpm 包和数据库，默认设置为/var/cache/yum
keepcache=0             //安装完成后是否保留软件包，0为不保留（默认为0），1为保留
debuglevel=2             //Debug 信息输出等级，范围为0-10，缺省为2
logfile=/var/log/yum.log     //yum 日志文件位置。用户可以到/var/log/yum.log 文件去查询过去所做的更新。
pkgpolicy=newest    //包的策略。一共有两个选项，newest 和last，这个作用是如果你设置了多个repository，而同一软件在不同的repository 中同时存在，yum 应该安装哪一个，如果是newest，则yum 会安装最新的那个版本。如果是last，则yum 会将服务器id 以字母表排序，并选择最后的那个服务器上的软件安装。一般都是选newest。
distroverpkg=redhat-release    //指定一个软件包，yum 会根据这个包判断你的发行版本，默认是redhat-release，也可以是安装的任何针对自己发行版的rpm 包。
tolerant=1          //有1和0两个选项，表示yum 是否容忍命令行发生与软件包有关的错误，比如你要安装1,2,3三个包，而其中3此前已经安装了，如果你设为1,则yum 不会出现错误信息。默认是0。
exactarch=1        //有1和0两个选项，设置为1，则yum 只会安装和系统架构匹配的软件包，例如，yum 不会将i686的软件包安装在适合i386的系统中。默认为1。
retries=6         //网络连接发生错误后的重试次数，如果设为0，则会无限重试。默认值为6.
obsoletes=1          //这是一个update 的参数，具体请参阅yum(8)，简单的说就是相当于upgrade，允许更新陈旧的RPM包。
plugins=1          //是否启用插件，默认1为允许，0表示不允许。我们一般会用yum-fastestmirror这个插件。
bugtracker_url=http://bugs.centos.org/set_project.php?project_id=16&ref=http://bugs.centos.org/bug_report_page.php?category=yum
# Note: yum-RHN-plugin doesn't honor this.
metadata_expire=1h
installonly_limit = 5
# PUT YOUR REPOS HERE OR IN separate files named file.repo
# in /etc/yum.repos.d

# 除了上述之外，还有一些可以添加的选项，如：
exclude=selinux*　　// 排除某些软件在升级名单之外，可以用通配符，列表中各个项目要用空格隔开，这个对于安装了诸如美化包，中文补丁的朋友特别有用。
gpgcheck=1　　// 有1和0两个选择，分别代表是否是否进行gpg(GNU Private Guard) 校验，以确定rpm 包的来源是有效和安全的。这个选项如果设置在[main]部分，则对每个repository 都有效。默认值为0。
```
## 10.执行报错
yum 命令死锁

```bash
Another app is currently holding the yum lock; waiting for it to exit… 
The other application is: PackageKit 
Memory : 40 M RSS (698 MB VSZ) 
Started: Wed Jul 15 13:59:01 2015 - 06:29 ago 
State : Sleeping, pid: 2305
```

可以通过执行`rm -rf /var/run/yum.pid` 来强行解除锁定，然后你的yum就可以运行了。


```bash
$ rm -f /var/lib/rpm/__db.00*　 //删除rpm数据文件
或者删除
$ rm -f /var/lib/rpm/__db.*
$ rpm --rebuileddb　//从新rpm数据文件
$ yum clean all  && yum update
```

---


✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>

 - **yum更多细节参阅**：[红帽官方软件包管理](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-yum)

