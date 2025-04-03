#  Linux Command dpkg 软件包管理
tags: 软件包管理


---

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/30fe8e7419329895618a881d50951406.gif#pic_center)


##  1. 简介
[dpkg](https://zh.m.wikipedia.org/zh-hans/Dpkg) 是Debian的一个底层包管理工具，主要用于对已下载到本地和已安装的软件包进行管理。
dpkg这个机制最早由Debian Linux社区所开发出来的，通过dpkg的机制，Debian提供的软件就能够简单的安装起来，同时能提供安装后的软件信息，实在非常不错。只要派生于Debian的其它Linux distributions大多使用dpkg这个机制来管理，包括B2D，Ubuntu等。

##  2. 语法

```bash
dpkg(选项)(参数)
```

##  2. 参数
选项

 - -i：安装软件包；
 - -r：删除软件包；
 - -P：删除软件包的同时删除其配置文件；
 - -L：显示于软件包关联的文件；
 - -l：显示已安装软件包列表；
 - --unpack：解开软件包；
 - -c：显示软件包内文件列表；
 - --confiugre：配置软件包。

## 3. 示例

```bash
dpkg -r 卸载软件包.不是完全的卸载,它的配置文件还存在.
dpkg --info "软件包名" --列出软件包解包后的包名称.
dpkg -l     --列出当前系统中所有的包.可以和参数less一起使用在分屏查看.
dpkg -l |grep -i "软件包名" --查看系统中与"软件包名"相关联的包.
dpkg -s  packagename   查询已安装的包的详细信息. 
dpkg -L  packagename  查询系统中已安装的软件包所安装的位置.
dpkg -S   查询系统中某个文件属于哪个软件包.
dpkg -I   查询deb包的详细信息,在一个软件包下载到本地之后看看用不用安装(看一下呗).
dpkg -i 手动安装软件包(这个命令并不能解决软件包之前的依赖性问题),如果在安装某一个软件包的时候遇到了软件依赖的问题,可以用apt-get -f install在解决信赖性这个问题.
dpkg -reconfigure 重新配置 
dpkg -P 全部卸载(但是还是不能解决软件包的依赖性的问题)
```

参考：

 - [dpkg(1) — Linux manual page](https://man7.org/linux/man-pages/man1/dpkg.1.html)

