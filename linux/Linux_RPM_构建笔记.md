

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

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b52fc9eab8c0c7877c7489825d486c34.png)

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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e1e013c0cec9b440c57a7307c8cc867c.png)
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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fe0cae4328a148a242ba0ec43bdefd01.png)
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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9627a84c281e89f0a3b1e5f942a2b829.png)
###  the install section
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/54aac4b85f9bf7c6857293acda6ded52.png)
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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/95ac153632ce3e5d6af74dbdcfc9137c.png)
###  The files sections
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/346c51a6fdaa4d87bef4c3ffce2c4132.png)
###  changelog 段
定义日志版本更新说明


Build RPMS with the rpmbuild command
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0cc4b5cfa747217228de1bac09d2339e.png)

```bash
rpmbuild -bp nignx.spec
rpmbuild -bc nginx.spec
rpmbuild -bi nginx.spec
rpmbuild -ba nginx.spec
```

##  The spec file
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0064f8d2a17f782258dc987086ab34c6.png)

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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b370cb0f0a7c508b72c3f506125a15b3.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/267d42a5b95320ed2366893aceba351d.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c4d40c41d9a914647e94686241b348d3.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/97d28ee6797235c0903bde3bfea2f450.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/acd2eedc8182a58e93fd1f1213445b53.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/82ff14671d9b042130f624f79e0c4da0.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/82c63574664a4aaf9ac82c0c3feec3a4.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b8ad596249edb83006bc188a0cfb2e12.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c49e22e90852e9147085015dda42c9e2.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1714d5a555461dd41a21745e22da21d6.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/487e4b553bcdf4b76ce7fdf0532007de.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e82e916593c56691949eecc13f92932f.png)


```bash
rpmbuild  --clean nginx.spec
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a25a820658f8f7c8d037d6d74d9e18fa.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f23f0f81dc09e81bbde104ef841f04a3.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/90541440eee510c9a588364e15ae937f.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7d053e58886ddd3b11de281683896097.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6985fe9f65cabeeb0e92f962e06dea13.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c878b15dafa68c40ca0026df63f1a278.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/46e9957ec1987a8d8290a253a9d72d54.png)

```bash
rpm -K xxx.rpm  #检查安装包的来源
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/393611c720af9c4b25314009d8fc622a.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/73e7cfaa1703fb0cbbc80b3d55389b78.png)

