

 - preparing to build RPMS
 - Planning for RPMS
 - Explaining the build process
 - using building files
 - seeing the results
 - verifying your RPMS
 - writing spec files
 - defiining package information
 - controlling the build
 - listing the files in the package
 - defining spec file macros

---

##  preparing to build RPMS

The main tasks in building RPMs
are:

 - Planning what you want to build
 - Gathering the software to package
 - *Patching the software as needed*
- *Creating a reproducible build of the software* 
- *Planning for upgrades*
- Outlining any dependencies
- Building the RPMS 
- Testing the RPMs


- 计划你想要建造什么
- 收集软件包
- 根据需要打补丁
- 创建软件的可复制构建
- 升级计划
- 列出所有依赖关系
- 构建rpm
- 测试rpm

场景：cacti 0.8.7  --> 0.8.8 升级，删除原来，打补丁.....需求

RPM capability 能力 封装  构建 厕所

###  Planning what you want to build
- An application
- Customized or patched?
- A programming library
- A set of system configuration files
- Or a documentation package
- Creating a binary RPM or a source RPM or both?

- 一个应用程序
- 定制或者修补吗?
- 一个编程库
- 一组系统配置文件
- 或者文件包
- 创建二进制RPM或源RPM或两者?

src.rpm  tar.gz , spec

##  Building RPMS
1.Set up the directory structure 
2.Place the sources in the right directory
3.Create a command what to do spec file that tells the rpmbuild
4. Build the source and binary RPMs

1.设置目录结构
2.将源代码放在正确的目录中
3.创建一个命令做什么规格文件告诉rpmbuild
4. 构建源代码和二进制rpm

###  Set up the directory structure 

![在这里插入图片描述](https://img-blog.csdnimg.cn/9da6445c63da4020b2a862e4dd7fe1ed.png)

```bash
rpmbuild --showrc | grep macros
```
如何构建自己的`rpmbuild`车间

```bash
[root@k3smaster ~]# rpmbuild --showrc |grep _topdir
-14: _builddir  %{_topdir}/BUILD
-14: _buildrootdir      %{_topdir}/BUILDROOT
-14: _rpmdir    %{_topdir}/RPMS
-14: _sourcedir %{_topdir}/SOURCES
-14: _specdir   %{_topdir}/SPECS
-14: _srcrpmdir %{_topdir}/SRPMS
-14: _topdir    %{getenv:HOME}/rpmbuild
[root@k3smaster ~]# su - mage
su: user mage does not exist
[root@k3smaster ~]# useradd zongxun
[root@k3smaster ~]# su - zongxun
[zongxun@k3smaster ~]$ rpmbuild --showrc | grep _topdir
-14: _builddir  %{_topdir}/BUILD
-14: _buildrootdir      %{_topdir}/BUILDROOT
-14: _rpmdir    %{_topdir}/RPMS
-14: _sourcedir %{_topdir}/SOURCES
-14: _specdir   %{_topdir}/SPECS
-14: _srcrpmdir %{_topdir}/SRPMS
-14: _topdir    %{getenv:HOME}/rpmbuild
[zongxun@k3smaster ~]$ ls
[zongxun@k3smaster ~]$ vim .rpmmacros
[zongxun@k3smaster ~]$ cat .rpmmacros
%_topdir        /home/zongxun
[zongxun@k3smaster ~]$ vim .rpmmacros
[zongxun@k3smaster ~]$ cat .rpmmacros
%_topdir        /home/zongxun/rpmbuild
[zongxun@k3smaster ~]$ mkdir -pv rpmbuild/{BUILD,RPMS,SOURCES,SPECS,SRPMS}
mkdir: created directory ‘rpmbuild’
mkdir: created directory ‘rpmbuild/BUILD’
mkdir: created directory ‘rpmbuild/RPMS’
mkdir: created directory ‘rpmbuild/SOURCES’
mkdir: created directory ‘rpmbuild/SPECS’
mkdir: created directory ‘rpmbuild/SRPMS’
[zongxun@k3smaster ~]$ rpmbuild --showrc | grep _topdir
-14: _builddir  %{_topdir}/BUILD
-14: _buildrootdir      %{_topdir}/BUILDROOT
-14: _rpmdir    %{_topdir}/RPMS
-14: _sourcedir %{_topdir}/SOURCES
-14: _specdir   %{_topdir}/SPECS
-14: _srcrpmdir %{_topdir}/SRPMS
-14: _topdir    /home/zongxun/rpmbuild
```




###  The introduction section
![在这里插入图片描述](https://img-blog.csdnimg.cn/dc45ca5bef374e1ebb0f071041576cd7.png)
source  BUILD
BUILDROOT

BuildRequires 依赖包


查看rpm文件

```bash
rpm -qi httpd
Name        : httpd
Version     : 2.4.6
Release     : 97.el7.centos.5
Architecture: x86_64
Install Date: Tue 08 Nov 2022 01:48:37 PM CST
Group       : System Environment/Daemons
Size        : 9821136
License     : ASL 2.0
Signature   : RSA/SHA256, Fri 25 Mar 2022 02:21:56 AM CST, Key ID 24c6a8a7f4a80eb5
Source RPM  : httpd-2.4.6-97.el7.centos.5.src.rpm
Build Date  : Thu 24 Mar 2022 10:59:42 PM CST
Build Host  : x86-02.bsys.centos.org
Relocations : (not relocatable)
Packager    : CentOS BuildSystem <http://bugs.centos.org>
Vendor      : CentOS
URL         : http://httpd.apache.org/
Summary     : Apache HTTP Server
Description :
The Apache HTTP Server is a powerful, efficient, and extensible
web server.
```



###  The prep section
准备阶段
![在这里插入图片描述](https://img-blog.csdnimg.cn/218f4e91335e4f5ba5db06b4a6b5541f.png)
The prep section, short for prepare, defines the commands necessary to prepare for the build
If you are starting with a compressed tar archive (a tarball) of the sources, the prep section needs
to extract the sources
The prep section starts with a %prep statement
This example uses the %setup RPM macro, which knows about tar archives, to extract the files
- prep部分(prepare的缩写)定义了为构建做准备所需的命令
- 如果你从压缩的tar文件(tarball)开始，准备部分需要提取信息源
- prep部分以一个%prep语句开始
- 这个例子使用`%setup` RPM宏来解压缩文件，它知道tar存档

###  The build section
![在这里插入图片描述](https://img-blog.csdnimg.cn/133c8285a15a4242842a6c8fede3d290.png)
###  the install section
![在这里插入图片描述](https://img-blog.csdnimg.cn/7db8728d43524c3cb82cf4a490b47a6a.png)
###  脚本段
- %pre 安装之前
- %post安装之后
- %preun卸载之前
- %postun卸载之后

BUILDROOT/


install 比cp更强大

```bash
install /etc/fstab /tmp
install -d /tmp/test
install -D /etc/nagios/nagios.cfg /tmp/nagios/nagios.cfg
```

### The clean section
![在这里插入图片描述](https://img-blog.csdnimg.cn/1f2fd057393f4b8eaa3d466881e4e230.png)
###  The files sections
![在这里插入图片描述](https://img-blog.csdnimg.cn/c2e280424bce4d9b9e62b6d77e60efbd.png)
###  changelog 段
定义日志版本更新说明


Build RPMS with the rpmbuild command
![在这里插入图片描述](https://img-blog.csdnimg.cn/e8e28897ebbe425785c8d45fece2e00e.png)

```bash
rpmbuild -bp nignx.spec
rpmbuild -bc nginx.spec
rpmbuild -bi nginx.spec
rpmbuild -ba nginx.spec
```

##  The spec file
![在这里插入图片描述](https://img-blog.csdnimg.cn/51a4a3909b594cf49b24936b274ff15b.png)

> 在注视当中要用%%表示%

rpmbuildd ba 既生成源码包又生成二进制包

```bash
rpm2cpio xxxx.rpm  cpio文件（乱码）
rpm3cpio xxxx.rpm | cpio -t
```


```bash
rpmbuild -rebuild  xxx.src.rpm
```
展开src包

```bash
rpm2cpio nginx-1.0.14-3.src.rpm | cpio -id
```
去哪里寻找src.rpm包

 - rpmfind.net
 - rpmone.com


##  defiining package information
![在这里插入图片描述](https://img-blog.csdnimg.cn/e3a0e870cf474f1caaec5d30dda5e043.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/8acf212c5c604b35967d072d7980a264.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/4282ae6cb5504dca86643d54ac798971.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/39cb2f36937548f08111a9192fd8b795.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/bb52f594dc804848984d8a41feb261f4.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/1762e8f7cb7a49aba83c2194fad72027.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/44bdc8dfb5b44a958c6c0096704074ea.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/df3a581677a84919ba52edc7047156d2.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/04bfc424f7ca4d588de4da4afb66353c.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/e3c2c50f03a84c7a913c903243bc6512.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/c4c0a3c07fc244e5a14f434297dd69db.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/6ce64ad3141f48d782800c2405d2cf2a.png)


```bash
rpmbuild  --clean nginx.spec
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/71cfec8054b14415b167c820730f1172.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/508304963a37496fb42f389ced0904ff.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b64ab2461a134ed18b01d73145a7f445.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/c0490398463e4195b92ff522edebfdf9.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/ea29b785044c4e5fb52975905fbc5034.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/185ea9059eeb46b0ad5a14ff8a9e02d3.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/7285ff91f8934f819fbda65af1b805fb.png)

```bash
rpm -K xxx.rpm  #检查安装包的来源
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/8fd8fd3c2df24cc1a45255e3c04057cf.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/3f4e3864a110416892e41b65b27b344b.png)

