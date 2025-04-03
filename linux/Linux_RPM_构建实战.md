
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e0d1b3e9e1fbfa42ae2b56dd85612bb7.png)






##  1. 背景

有时您可能有权访问开源应用程序源代码，但可能没有 RPM 文件来将其安装在您的系统上。

在这种情况下，您可以编译源代码并从源代码安装应用程序，或者自己从源代码构建一个 RPM 文件，然后使用 RPM 文件安装应用程序。

还可能存在您想要为您开发的应用程序构建自定义 RPM 包的情况。

本教程解释了如何从源代码构建 RPM 包。

为了构建 RPM，您需要源代码，这通常意味着一个压缩的 tar 文件，其中还包含 SPEC 文件。

SPEC 文件通常包含有关如何构建 RPM、哪些文件是包的一部分以及应该安装在哪里的说明。

RPM 在构建过程中执行以下任务。

RPM 在构建过程中执行以下任务。

- 执行规范文件的准备部分中提到的命令和宏。
- 检查文件列表的内容
- 执行规范文件的构建部分中的命令和宏。文件列表中的宏也在这一步执行。
- 创建二进制包文件
- 创建源包文件

一旦 RPM 执行了上述步骤，它就会创建二进制包文件和源包文件。

二进制包文件包含所有源文件以及安装或卸载包的任何附加信息。

它通常启用所有用于安装特定于平台的软件包的选项。二进制包文件包含为特定架构编译的完整应用程序或函数库。源包通常由原始压缩的 tar 文件、spec 文件和创建二进制包文件所需的补丁组成。

让我们看看如何使用 tar 文件创建简单的源代码和 BIN RPM 包。

如果您是 rpm 软件包的新手，您可能首先想了解如何使用[rpm](https://blog.csdn.net/xixihahalelehehe/article/details/111824141) 命令在 CentOS 或 RedHat 上安装、升级和删除软件包

## 2.  构建 RPM 流程
- 计划你想要建造什么

- 收集软件包

- 根据需要打补丁

- 创建软件的可复制构建

- 升级计划

- 列出所有依赖关系

- 构建rpm

- 测试rpm

## 3. 安装
要基于我们刚刚创建的规范文件构建一个 rpm 文件，我们需要使用 rpmbuild 命令。

`rpmbuild` 命令是 `rpm-build` 软件包的一部分。如下图所示安装。

```bash
$  yum install rpm-build
```
rpm-build 依赖于以下软件包。如果您还没有安装这些，yum 会自动为您安装这些依赖项。
- elfutils-libelf
- rpm
- rpm-libs
- rpm-python

也可以安装 `rpmdevtools`，这个工具部包含一些其他工具，依赖 `rpm-build`，所以直接安装会将 rpm-build 装上：

```bash
yum install -y rpmdevtools
```

> Python 的编译打包工具是 setuptools。

## 4. 构建 RPM "工作空间"
### 4.1 rpmdev-setuptree

`rpmbuild` 命令使用一套标准化的「工作空间」 ，生成 `%_topdir` 工作目录 `~/rpmbuild`，以及配置文件 `~/.rpmmacros`：

```bash
$ rpmdev-setuptree
```
`rpmdev-setuptree`这个命令就是安装 `rpmdevtools` 带来的。可以看到运行了这个命令之后，在 `$HOME` 家目录下多了一个叫做 `rpmbuild` 的文件夹，里边内容如下：

```bash
$ tree rpmbuild
rpmbuild
├── BUILD
├── RPMS
├── SOURCES
├── SPECS
└── SRPMS
```
如果没有安装 `rpmdevtools` 的话，其实用 mkdir 命令创建这些文件夹也是可以的:`mkdir -p ~/rpmbuild/{BUILD,RPMS,SOURCES,SPECS,SRPMS}`。

|默认位置|	宏代码|	名称	|用途|
|--|--|--|--|
|~/rpmbuild/SPECS	|%_specdir	|Spec 文件目录|	保存 RPM 包配置（.spec）文件
~/rpmbuild/SOURCES|	%_sourcedir	|源代码目录	|保存源码包（如 .tar 包）和所有 patch 补丁
~/rpmbuild/BUILD	|%_builddir	|构建目录|	源码包被解压至此，并在该目录的子目录完成编译
~/rpmbuild/RPMS	|%_rpmdir	|标准 RPM 包目录	|生成/保存二进制 RPM 包
~/rpmbuild/SRPMS	|%_srcrpmdir	|源代码 RPM 包目录|	生成/保存源码 RPM 包(SRPM)
~/rpmbuild/BUILDROOT	|%_buildrootdir	|最终安装目录	|保存 %install 阶段安装的文件

`rpmbuild` 默认工作路径的确定，通常由在 `/usr/lib/rpm/macros` 这个文件里的一个叫做 `%_topdir` 的宏变量来定义。如果用户想更改这个目录名，rpm 官方并不推荐直接更改这个目录，而是在用户家目录下建立一个名为 `.rpmmacros` 的隐藏文件(Linux下隐藏文件,前面的点不能少)，然后在里面重新定义 `%_topdir`，指向一个新的目录名。这样就可以满足某些用户的差异化需求了。`.rpmmacros` 文件里内容，比如：

```bash
 michael@localhost  ~  cat .rpmmacros

%_topdir %(echo $HOME)/rpmbuild

%_smp_mflags %( \
    [ -z "$RPM_BUILD_NCPUS" ] \\\
        && RPM_BUILD_NCPUS="`/usr/bin/nproc 2>/dev/null || \\\
                             /usr/bin/getconf _NPROCESSORS_ONLN`"; \\\
    if [ "$RPM_BUILD_NCPUS" -gt 16 ]; then \\\
        echo "-j16"; \\\
    elif [ "$RPM_BUILD_NCPUS" -gt 3 ]; then \\\
        echo "-j$RPM_BUILD_NCPUS"; \\\
    else \\\
        echo "-j3"; \\\
    fi )

%__arch_install_post \
    [ "%{buildarch}" = "noarch" ] || QA_CHECK_RPATHS=1 ; \
    case "${QA_CHECK_RPATHS:-}" in [1yY]*) /usr/lib/rpm/check-rpaths ;; esac \
    /usr/lib/rpm/check-buildroot
```

### 4.2 自定义用户构建 RPM 工作空间
在用户zongxun下创建 RPM 工作空间
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
## 5. SPEC 文件
包含以下几个阶段：
- The introduction section
- The prep section
- The build section
- the install section
- the scripts section
- The clean section
- The files sections
- changelog sections

### 5.1 The introduction section
介绍部分包含有关包的信息，即rpm -qi命令显示的信息类型

```bash
# cat /root/rpmbuild/SPECS/icecast.spec
Name:           icecast
Version:        2.3.3
Release:        0
Summary:        Xiph Streaming media server that supports multiple formats.
Group:          Applications/Multimedia
License:        GPL
URL:            http://www.icecast.org/
Vendor:         Xiph.org Foundation team@icecast.org
Source:         http://downloads.us.xiph.org/releases/icecast/%{name}-%{version}.tar.gz
Prefix:         %{_prefix}
Packager: 	Karthik
BuildRoot:      %{_tmppath}/%{name}-root

%description
Icecast is a streaming media server which currently supports Ogg Vorbis
and MP3 audio streams. It can be used to create an Internet radio
station or a privately running jukebox and many things in between.
It is very versatile in that new formats can be added relatively
easily and supports open standards for commuincation and interaction.
```

通过`rpm -i`命令查看

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

### 5.2 The prep section

- prep部分(prepare的缩写)定义了为构建做准备所需的命令
- 如果你从压缩的tar文件(tarball)开始，准备部分需要提取信息源
- prep部分以一个%prep语句开始
- 这个例子使用`%setup` RPM宏来解压缩文件，它知道tar存档

### 5.3 The build section
规范文件构建部分包含构建软件的命令
。
通常，这将只包括一些命令，因为大多数真正的指令出现在Makefile中
。
构建部分以%构建语句%build开始

```bash
./configure
--etcdir="%_sysconfdir)" \
--mandir="%{_mandir) \
--i18n=0  \
--scrip=0  \
% (_make) % (?_smp_mflags)
```

###  5.4 The install section
规范文件安装部分包含安装新构建的应用程序或库所需的命令，在大多数情况下，安装部分应该清理`Buildroot`目录并运行`make install`命令，安装部分以`%install`语句开始。

```bash
%install
%{_rm} Tf %{buildroot}
%{_make} instal1 DESTDIR="%{buildroot}"
%find_lang %{name}
```
install 比cp更强大

```bash
install /etc/fstab /tmp
install -d /tmp/test
install -D /etc/nagios/nagios.cfg /tmp/nagios/nagios.cfg
```
### 5.5  the scripts section
- %pre 安装之前
- %post安装之后
- %preun卸载之前
- %postun卸载之后
  
### 5.6 The clean section
- clean部分清理其他部分中命令创建的文件
- clean部分以%clean语句开始

```bash
%clean
%{_rm} -rf %{buildroot}
```

###  5.7 The files sections
- 最后，文件部分列出了要进入二进制RPM的文件，以及已定义的文件属性。
- 文件部分以%files文件声明。
- 宏`%doc`将某些文件标记为文档
- 这允许RPM将包含文档的文件与RPM中的其他文件区分开来

### 5.8 changelog 段
定义日志版本更新说明

### 5.9 生成 SPEC 文件模板
最最最重要的 `SPEC` 文件，命名格式一般是“`软件名-版本.spec`”的形式，将其拷贝到 `SPECS` 目录下。

如果系统有 `rpmdevtools` 工具，可以用 `rpmdev-newspec -o name.spec` 命令来生成 `SPEC` 文件的模板，然后进行修改：

```bash
[root@localhost ~]# rpmdev-newspec -o myapp.spec
Skeleton specfile (minimal) has been created to "myapp.spec".
[root@localhost ~]# cat myapp-0.1.0.spec 
Name:           myapp
Version:
Release:        1%{?dist}
Summary:

License:
URL:
Source0:

BuildRequires:
Requires:

%description


%prep
%setup -q


%build
%configure
make %{?_smp_mflags}


%install
rm -rf $RPM_BUILD_ROOT
%make_install


%files
%doc
```
如果没有安装 `rpmdevtools`，也可以自己手动创建一个 `spec` 文件。

## 6. 打包命令
在 `rpmbuild/SPECS` 目录下执行打包编译，切换到该目录下执行打包编译命令。

### 6.1 rpmbuild 命令选项
- `-bp` 只解压源码及应用补丁
- `-bc` 只进行编译
- `-bi` 只进行安装到`%{buildroot}`
`-bb` 只生成二进制 rpm 包
- `-bs` 只生成源码 rpm 包
- `-ba` 生成二进制 rpm 包和源码 rpm 包
- `--target` 指定生成 rpm 包的平台，默认会生成 `i686` 和 `x86_64` 的 rpm 包，但一般我只需要 `x86_64` 的 rpm 包

### 6.2 只生成二进制格式的 rpm 包

```bash
rpmbuild -bb 软件名-版本.spec 
```
用此命令生成软件包，生成的文件会在刚才建立的RPM目录下存在。

### 6.3 只生成 `src` 格式的 rpm 包

```bash
rpmbuild -bs 软件名-版本.spec 
```

生成的文件会在刚才建立的SRPM目录下存在。

### 6.4 只需要生成完整的源文件
```bash
rpmbuild -bp 软件名-版本.spec 
```
源文件存在目录 `BUILD` 下。可能对这个命令不太明白，这个命令的作用就是把 tar 包解开然后把所有的补丁文件合并而生成一个完整的具最新功能的源文件。

### 6.5 完全打包

```bash
rpmbuild -ba 软件名-版本.spec 
```

软件包制作完成后可用 rpm 命令查询，看看效果。如果不满意的话可以再次修改软件包描述文件，重新运行以上命令产生新的 RPM 软件包。

##  7. 实例1
将所有用于生成 `rpm` 包的源代码、 shell 脚本、配置文件都拷贝到 SOURCES 目录里，注意通常情况下源码的压缩格式都为 `*.tar.gz` 格式。

### 7.1 下载源码

```bash
cd ~/rpmbuild/SOURCES
wget wget http://ftp.gnu.org/gnu/hello/hello-2.10.tar.gz
```
###  7.2 编辑SPEC文件

```bash
cd ~/rpmbuild/SPECS
vim hello.spec
```

```bash
Name:     hello
Version:  2.10
Release:  1%{?dist}
Summary:  The "Hello World" program from GNU
Summary(zh_CN):  GNU "Hello World" 程序
License:  GPLv3+
URL:      http://ftp.gnu.org/gnu/hello
Source0:  http://ftp.gnu.org/gnu/hello/%{name}-%{version}.tar.gz

BuildRequires:  gettext
Requires(post): info
Requires(preun): info

%description
The "Hello World" program, done with all bells and whistles of a proper FOSS
project, including configuration, build, internationalization, help files, etc.

%description -l zh_CN
"Hello World" 程序, 包含 FOSS 项目所需的所有部分, 包括配置, 构建, 国际化, 帮助文件等.

%prep
%setup -q

%build
%configure
make %{?_smp_mflags}

%install
make install DESTDIR=%{buildroot}
%find_lang %{name}
rm -f %{buildroot}/%{_infodir}/dir

%post
/sbin/install-info %{_infodir}/%{name}.info %{_infodir}/dir || :

%preun
if [ $1 = 0 ] ; then
/sbin/install-info --delete %{_infodir}/%{name}.info %{_infodir}/dir || :
fi

%files -f %{name}.lang
%doc AUTHORS ChangeLog NEWS README THANKS TODO
%license COPYING
%{_mandir}/man1/hello.1.*
%{_infodir}/hello.info.*
%{_bindir}/hello

%changelog
* Sun Dec 4 2016 Your Name <youremail@xxx.xxx> - 2.10-1
- Update to 2.10
* Sat Dec 3 2016 Your Name <youremail@xxx.xxx> - 2.9-1
- Update to 2.9
```
`Group`标签过去用于按照 `/usr/share/doc/rpm-/GROUPS` 分类软件包。目前该标记已丢弃，vim的模板还有这一条，删掉即可，不过添加该标记也不会有任何影响。

###  7.3 构建RPM包

```bash
rpmbuild -ba hello.spec
```
OK，执行成功，看看结果：

```bash
 michael@localhost  ~/rpmbuild  tree *RPMS
RPMS
└── x86_64
    ├── hello-2.10-1.el7.x86_64.rpm
    └── hello-debuginfo-2.10-1.el7.x86_64.rpm
SRPMS
└── hello-2.10-1.el7.src.rpm

1 directory, 3 files
```
### 7.4 安装 RPM 包

```bash
sudo rpm -ivh ~/rpmbuild/RPMS/x86_64/hello-2.10-1.el7.x86_64.rpm
```
运行：

```bash
$ hello
Hello, world!
$ which hello
/usr/bin/hello
$ rpm -qf `which hello`
hello-2.10-1.el7.centos.x86_64
$ man hello
```

### 7.5 rpmbuild 目录结构

```bash
michael@localhost  ~/rpmbuild  tree . -L 3
.
├── BUILD
│   └── hello-2.10
│       ├── ABOUT-NLS
│       ├── aclocal.m4
│       ├── AUTHORS
│       ├── build-aux
│       ├── ChangeLog
│       ├── ChangeLog.O
│       ├── config.h
│       ├── config.in
│       ├── config.log
│       ├── config.status
│       ├── configure
│       ├── configure.ac
│       ├── contrib
│       ├── COPYING
│       ├── debugfiles.list
│       ├── debuglinks.list
│       ├── debugsources.list
│       ├── doc
│       ├── elfbins.list
│       ├── GNUmakefile
│       ├── hello
│       ├── hello.1
│       ├── hello.lang
│       ├── INSTALL
│       ├── lib
│       ├── m4
│       ├── maint.mk
│       ├── Makefile
│       ├── Makefile.am
│       ├── Makefile.in
│       ├── man
│       ├── NEWS
│       ├── po
│       ├── README
│       ├── README-dev
│       ├── README-release
│       ├── src
│       ├── stamp-h1
│       ├── tests
│       ├── THANKS
│       └── TODO
├── BUILDROOT
├── RPMS
│   └── x86_64
│       ├── hello-2.10-1.el7.x86_64.rpm
│       └── hello-debuginfo-2.10-1.el7.x86_64.rpm
├── SOURCES
│   └── hello-2.10.tar.gz
├── SPECS
│   └── hello.spec
└── SRPMS
    └── hello-2.10-1.el7.src.rpm

17 directories, 37 files
```

参考：
- [7 Steps to Build a RPM Package from Source on CentOS / RedHat](https://www.thegeekstuff.com/2015/02/rpm-build-package-example/)
- [RPM 包的构建 - 实例](https://www.cnblogs.com/michael-xiang/p/10500704.html)
- [How to create an RPM package/zh-cn](https://fedoraproject.org/wiki/How_to_create_an_RPM_package/zh-cn)
- [How to create a GNU Hello RPM package/zh-cn](https://fedoraproject.org/wiki/How_to_create_a_GNU_Hello_RPM_package/zh-cn)
