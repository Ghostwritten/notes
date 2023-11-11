#  Linux Command sshpass 远程主机
tags： 远程

1、直接远程连接某主机

```bash
sshpass -p {密码} ssh {用户名}@{主机IP}
```

2、远程连接指定ssh的端口

```bash
sshpass -p {密码} ssh -p ${端口} {用户名}@{主机IP} 
```

3、从密码文件读取文件内容作为密码去远程连接主机

```bash
sshpass -f ${密码文本文件} ssh {用户名}@{主机IP} 
```

4、从远程主机上拉取文件到本地

```bash
sshpass -p {密码} scp {用户名}@{主机IP}:${远程主机目录} ${本地主机目录}
```

5、将主机目录文件拷贝至远程主机目录

```bash
sshpass -p {密码} scp ${本地主机目录} {用户名}@{主机IP}:${远程主机目录} 
```

6、远程连接主机并执行命令

```bash
sshpass -p {密码} ssh -o StrictHostKeyChecking=no {用户名}@{主机IP} 'rm -rf /tmp/test'
root@k8s2-master:~#  sshpass -p foo ssh -o StrictHostKeyChecking=no root@192.168.211.51 "hostname"
k8s2-node1

```

**-o StrictHostKeyChecking=no ：忽略密码提示**

7. 拷贝密钥

```bash
sshpass -p 123456 ssh-copy-id -f root@192.168.211.51
```

参考：

 - [SSH password automation in Linux with sshpass](https://www.redhat.com/sysadmin/ssh-automation-sshpass)
 - [sshpass: Login To SSH Server / Provide SSH Password Using A Shell Script](https://www.cyberciti.biz/faq/noninteractive-shell-script-ssh-password-provider/)
 - [sshpass：一个很棒的免交互 SSH 登录工具，但不要用在生产服务器上](https://linux.cn/article-8086-1.html)

