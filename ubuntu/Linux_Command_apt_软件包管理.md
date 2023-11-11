# Linux Command apt 软件包管理
tags: 软件包管理




------------

![在这里插入图片描述](https://img-blog.csdnimg.cn/403fbbebe54c4195b3eee17b6ab64188.gif#pic_center)

##  1. 简介
apt 是一个命令行实用程序，用于在 Ubuntu、Debian 和相关 Linux 发行版上安装、更新、删除和以其他方式管理 deb 包。[apt-get](https://linux.die.net/man/8/apt-get)它将和工具中最常用的命令apt-cache与某些选项的不同默认值相结合。


![在这里插入图片描述](https://img-blog.csdnimg.cn/f75a130c05ac4ed3b0367ac54a2349a5.webp#pic_center)
## 2. apt  示例
### 2.1 apt search
```bash
apt search php  搜索选项可用于搜索给定的正则表达式
apt search mysql-5.?
apt search mysql-server-5.?
apt search httpd*
apt search ^apache
apt search ^nginx
apt search ^nginx$
```
###  2.2  apt show
```bash
apt show {pkgNamehere}  要显示或查看有关给定包的信息，包括其依赖项、安装和下载大小、包的可用来源、包内容的描述等等
apt show nginx
apt show sudo

apt show	apt-cache show	显示装细节
```
### 2.3 apt list
```bash
apt list   要列出所有包，请输入： 要显示满足特定条件的包列表
apt list | more
apt list | grep foo
apt list | grep php7-
apt list nginx
apt list 'php7*'
apt list --installed   列出所有已安装的包
apt list --installed | grep {pkgNameHere}
apt list --upgradable 列出已升级的软件包
```
###  2.4 apt depends
```bash
#列出包依赖
apt depends {pkgNameHere}   依赖选项显示一个包具有的每个依赖项的列表以及可以满足该依赖项的所有可能的其他包。
apt depends sudo
```
###  2.5 apt install
```bash
apt install	apt-get install	安装软件包
```
### 2.6 apt remove
```bash
apt remove {pkgNameHere}  移除软件包
apt remove nginx 删除或移除一个名为 nginx 的包，除系统上的配置文件外，所有文件都将被删除。

apt autoremove	apt-get autoremove	自动删除不需要的包
```

###  2.7 apt purge
```bash
apt purge	{pkgNameHere}	移除软件包及配置文件
apt purge nginx nginx-core nginx-common 删除一个包会删除所有打包的数据，但通常会留下小的（修改过的）用户配置文件，以防删除是意外的。
```
###  2.8 apt update
```bash
apt update	{pkgNameHere}	刷新存储库索引
```
###  2.9 apt upgrade

```bash
apt upgrade	{pkgNameHere}	升级所有可升级的软件包
apt full-upgrade  完整升级
```


## 3. apt-get 参数

```bash
 update - 重新获取软件包列表  
 upgrade - 进行更新  
 install - 安装新的软件包  
 remove - 移除软件包  
 autoremove - 自动移除全部不使用的软件包  
 purge - 移除软件包和配置文件  
 source - 下载源码档案  
 build-dep - 为源码包配置编译依赖  
 dist-upgrade - 发行版升级, 参见 apt-get(8)  
 dselect-upgrade - 依照 dselect 的选择更新  
 clean - 清除下载的归档文件  
 autoclean - 清除旧的的已下载的归档文件  
 check - 检验是否有损坏的依赖  
```

  
选项：  

```bash
 -h 本帮助文件。  
 -q 输出到日志 - 无进展指示  
 -qq 不输出信息，错误除外  
 -d 仅下载 - 不安装或解压归档文件  
 -s 不实际安装。模拟执行命令  
 -y 假定对所有的询问选是，不提示  
 -f 尝试修正系统依赖损坏处  
 -m 如果归档无法定位，尝试继续  
 -u 同时显示更新软件包的列表  
 -b 获取源码包后编译  
 -V 显示详细的版本号  
 -c=? 阅读此配置文件  
 -o=? 设置自定的配置选项，如 -o dir::cache=/tmp  
```

## 4. apt-get 示例
```bash
apt-get install packagename 安装包
apt-get install packagename - - reinstall 重新安装包
apt-get -f install 修复安装"-f = --fix-missing"
apt-get remove packagename 删除包
apt-get remove packagename - - purge 删除包，包括删除配置文件等
apt-get update 更新源
apt-get upgrade 更新已安装的包
apt-get dist-upgrade 升级系统
apt-get dselect-upgrade 使用 dselect 升级
apt-get build-dep packagename 安装相关的编译环境
apt-get source packagename 下载该包的源代码
apt-get clean 清理无用的包
apt-get autoclean 清理无用的包
apt-get check 检查是否有损坏的依赖
```



## 5. apt-cache 示例
```bash
apt-cache search packagename 搜索包
apt-cache show packagename 获取包的相关信息，如说明、大小、版本等
apt-cache depends packagename 了解使用依赖
apt-cache rdepends packagename 是查看该包被哪些包依赖
apt-cache stats  显示系统安装包的统计信息
apt-cache madison kubeadm  显示安装包的各个版本
apt-cache policy kubeadm  在列表中显示版本
```




## 6.  apt-mark 示例
apt-mark 用来标记安装软件包的状态. 有时一个软件的仓库出了问题, 用 apt-get 安装后无法使用, 这时就可以利用 apt-mark hold package 标记该软件包不被自动更新
```bash
apt-mark hold xxx       屏蔽指定软件包的更新
apt-mark unhold xxx     将 hold 替换为 unhold 就可以取消对这个包版本的锁定
dpkg --get-selections   查看包状态
```

参考：
 - [Using apt Commands in Linux](https://itsfoss.com/apt-command-guide/)
 - [apt Command Examples for Ubuntu/Debian Linux](https://www.cyberciti.biz/faq/ubuntu-lts-debian-linux-apt-command-examples/)

