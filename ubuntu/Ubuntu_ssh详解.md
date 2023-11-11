## 1. ssh介绍
ssh是指Secure Shell,是一种安全的传输协议，Ubuntu客户端可以通过SSH访问远程服务器 。
SSH包：`openssh-client和openssh-server`

登陆别的机器的SSH只需要安装openssh-client（ubuntu有默认安装，如果没有）

```bash
$ sudo apt-get install openssh-client
```
本机开放SSH服务就需要安装（缺省没有安装）

```bash
$ sudo apt-get install openssh-server
$ ps -e|grep ssh
$ sudo/etc/init.d/ssh start
```
## 2. SSH配置

```bash
$ sudo apt-get install vim
$ sudo cp/etc/ssh/sshd_config /etc/ssh/sshd_config.original
$ sudo chmod a-w /etc/ssh/sshd_config.original
$ vim /etc/ssh/sshd_config
#Port 22，去掉注释，修改成一个五位的端口：
Port 22333
#PermitRootLogin yes，去掉注释，修改为：
PermitRootLogin no
$ sudo /etc/init.d/ssh restart
```

