


## 1. 临时设置环境变量
通过设置环境变量，可以使单个命令使用另一种语言LANG

```bash
$ LANG=fr_FR.utf8 date
mar. mai 24 12:16:51 CDT 2022
```
后续命令将恢复为使用系统的默认语言进行输出。该locale命令可用于确定LANG和其他相关环境变量的当前值。


## 2. 默认语言设置
系统的默认语言设置为美国英语，使用 Unicode 的 `UTF-8` 编码作为其字符集 ( `en_US.utf8`)，但这可以在安装期间或安装后更改。

从命令行，root用户可以使用命令更改系统范围的区域设置`localectl`。如果不带参数运行localectl，它会显示当前系统范围的区域设置。

要设置系统范围的默认语言，请运行命令，其中`localectl set-locale LANG=locale`语言环境LANG是本章“语言代码参考”表中环境变量的适当值。更改将在用户下次登录时生效，并存储在`/etc/locale.conf`.

```bash
 localectl set-locale LANG=fr_FR.utf8
```
## 3. 语言包
称为 langpacks 的特殊 RPM 包安装语言包，添加对特定语言的支持。这些语言包使用依赖项自动安装额外的 RPM 包，其中包含系统上其他软件包的本地化、词典和翻译。

要列出已安装和可能已安装的语言包，请使用以下命令。

```bash
$ yum list langpacks-*
  更新订阅管理存储库。
  更新订阅管理存储库。
  安装包
  langpacks-en.noarch 1.0-12.el8 @AppStream
  可用套餐
  langpacks-af.noarch 1.0-12.el8 rhel-8-for-x86_64-appstream-rpms
  langpacks-am.noarch 1.0-12.el8 rhel-8-for-x86_64-appstream-rpms
  langpacks-ar.noarch 1.0-12.el8 rhel-8-for-x86_64-appstream-rpms
  langpacks-as.noarch 1.0-12.el8 rhel-8-for-x86_64-appstream-rpms
  langpacks-ast.noarch 1.0-12.el8 rhel-8-for-x86_64-appstream-rpms
```

安装

```bash
 yum install langpacks-fr
```
用于`yum repoquery --whatsupplements`确定语言包可能安装哪些 RPM 包。

```bash
$ yum repoquery --whatsupplements langpacks-fr
  更新订阅管理存储库。
  更新订阅管理存储库。
  上次元数据过期检查：0:01:33 前，美国中部标准时间 2019 年 2 月 6 日星期三上午 10:47:24。
  glibc-langpack-fr-0:2.28-18.el8.x86_64
  gnome-getting-started-docs-fr-0:3.28.2-1.el8.noarch
  hunspell-fr-0:6.2-1.el8.noarch
  连字符-fr-0:3.0-1.el8.noarch
  libreoffice-langpack-fr-1:6.0.6.1-9.el8.x86_64
  手册页-fr-0:3.70-16.el8.noarch
  mythes-fr-0:2.3-10.el8.noarch
```

## 4. 安装浏览器 chromium

```bash
sudo yum install chromium
```

