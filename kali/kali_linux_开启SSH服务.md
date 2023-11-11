1. 查看ssh服务状态

```bash
# /etc/init.d/ssh status
```

2.修改配置文件

```bash
# vim /etc/ssh/sshd_config
```

```bash
#PermitRootLogin prohibit-password
PermitRootLogin yes
#PasswordAuthentication yes
PasswordAuthentication yes
```
3. 启动ssh服务

```bash
# /etc/init.d/ssh start
```
查看服务状态

```bash
# /etc/init.d/ssh status
```

开机启动

```bash
# update-rc.d ssh enable
```

