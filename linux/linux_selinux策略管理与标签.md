

## 1. selinux 策略
#打开`80/tcp` 、`443/tcp`端口

```bash
firewall-cmd --permanent --add-service=http --add-service=https
firewall-cmd --reload
```

文件系统权限：任何`DocumentRoot`必须由apache用户或用户组读取，大部分情况下，不允许apache用户或组写入。
selinux：默认selinux策略会限制httpd读取上下文，web服务器默认上下文是`httpd_sys_content_t`

```bash
semanage fcontext -a -t httpd_sys_content_t '/new/location(/.*)?'
```

`selinux-policy-devel`的`httpd_selinux man page`详解

允许documentRoot写

```bash
setfacl -R -m g:webmasters:rwX /var/www/html
setfacl -R -m d:g:webmasters:rwx /var/www/html
```

大写的“X”位仅对目录设置执行

创建webmasters组

```bash
mkdir -p -m 2775 /new/docroot
chgrp webmasters /new/docroot
```

---------------------

我将serverx系统配置为将一个传入到端口，443/tcp的请求从desktopX转发到端口22/tcp,我的desktopx的ip地址为172.25.x.10

serverx添加永久性规则

```bash
firewall-cmd --permanent --add-rich-rule 'rule family=ipv4 source address=172.25.x.10/32 forward-port port=443 protocol=tcp to-port=22'
firewall-cmd --reload
```



-----
## 2. selinux标签协议

selinux：文件、进程、网络流量标记（端口）

安装

```bash
yum -y install selinux-policy-devel
mandb
man -k _selinux
```

监听网络端口标签


查看本地

```bash
semanage port -l
```


管理端口标签

向现有端口标签添加端口语法：

```bash
semanage port -a -t port_label -p tcp|udp PORTNUMBER
```

例如：允许gopher服务侦听端口71/tcp

```bash
semanage port -a -t gopher_port -p tcp 71
```

删除

```bash
semanage port -d -t gopher_port -p tcp 71
```

修改

```bash
semanage port -m -t gopher_port -p tcp 71
```
---
✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>

-  [red hat linux selinux 细节参考](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/selinux_users_and_administrators_guide/index)



![在这里插入图片描述](https://img-blog.csdnimg.cn/f83763541592452294366d320af38f16.gif#pic_center)

