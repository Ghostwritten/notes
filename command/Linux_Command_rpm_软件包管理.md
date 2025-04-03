#  Linux Command rpm 软件包管理
tags: 软件包管理



---

## 1. 简介
[rpm](https://nl.wikipedia.org/wiki/RPM_Package_Manager)命令是RPM软件包的管理工具。rpm原本是Red Hat Linux发行版专门用来管理Linux各项套件的程序，由于它遵循GPL规则且功能强大方便，因而广受欢迎。逐渐受到其他发行版的采用。RPM套件管理方式的出现，让Linux易于安装，升级，间接提升了Linux的适用度

## 2. 参数

```bash
-a：查询所有套件；
-b<完成阶段><套件档>+或-t <完成阶段><套件档>+：设置包装套件的完成阶段，并指定套件档的文件名称；
-c：只列出组态配置文件，本参数需配合"-l"参数使用；
-d：只列出文本文件，本参数需配合"-l"参数使用；
-e<套件档>或--erase<套件档>：删除指定的套件；
-f<文件>+：查询拥有指定文件的套件；
-h或--hash：套件安装时列出标记；
-i：显示套件的相关信息；
-i<套件档>或--install<套件档>：安装指定的套件档；
-l：显示套件的文件列表；
-p<套件档>+：查询指定的RPM套件档；
-q：使用询问模式，当遇到任何问题时，rpm指令会先询问用户；
-R：显示套件的关联性信息；
-s：显示文件状态，本参数需配合"-l"参数使用；
-U<套件档>或--upgrade<套件档>：升级指定的套件档；
-v：显示指令执行过程；
-vv：详细显示指令执行过程，便于排错。
```
## 3. 实例
### 3.1安装

```bash
rpm -ivh your-package.rpm
```
### 3.2 列出所有安装过的包

```bash
rpm -qa 
rpm -qa | grep sql #匹配对应的包
```
### 3.3 如何获得某个软件包的文件全名。

```bash
rpm -q mysql
```
### 3.4 一个rpm包中的文件安装到那里去了？

```bash
rpm -ql 包名
```
### 3.5 某个程序是哪个软件包安装的

```bash
rpm -qf `which 程序名`    #返回软件包的全名
rpm -qif `which 程序名`   #返回软件包的有关信息
rpm -qlf `which 程序名`   #返回软件包的文件列表
```

###  3.6 查询包来源

```bash
# curl --version 
curl 7.29.0 (x86_64-redhat-linux-gnu)...

# rpm -qa curl
curl-7.29.0-35.el7.x86_64

# rpm -qi curl-7.29.0-35.el7.x86_64 | egrep "Vendor|Packager"
```


### 3.7 查询命令来源

```bash
$ whereis redis-cli
redis-cli: /usr/bin/redis-cli /usr/share/man/man1/redis-cli.1.gz

$ rpm -qf /usr/bin/redis-cli
redis-7.0.4-1.el7.remi.x86_64
```

参考：
	

 - [rpm command cheat sheet for Linux](https://www.cyberciti.biz/howto/question/linux/linux-rpm-cheat-sheet.php)
 - [RPM 官方网站](https://rpm.org/)
 - [RPM Package Manager (Red-hat Package Manager)](https://www.techtarget.com/searchdatacenter/definition/RPM-Package-Manager-Red-hat-Package-Manager)
 - [How to create a Linux RPM package](https://www.redhat.com/sysadmin/create-rpm-package)
