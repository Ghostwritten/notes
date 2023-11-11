# Linux Command who、whois、whoami
tags: 用户管理 

## 1. whois

显示指定用户信息

```bash
$ whois root　#显示指定用户信息
$ whois ywnz.com　#查询域名描述信息
$ whois ywnz.com　#查询域名信息
$ whois -H ywnz.com　#查询域名信息省略法律声明
$ whois -p 80 ywnz.com　#指定端口查询
```

## 2. who whoami 和 who am i

 - whoami；显示的是当前用户下的用户名
 - who am i：显示的是登录时的用户名
 - who：显示当前真正登录系统中的用户（不会显示那些用su命令切换用户的登录者）

```bash
[root@kz1 ~]# who
root     pts/0        2020-03-16 21:56 (172.6.185.228)
[root@kz1 ~]# whoami
root
[root@kz1 ~]# who am i 
root     pts/0        2020-03-16 21:56 (172.6.185.228)
[root@kz1 ~]# useradd test1
[root@kz1 ~]# su - test1
[test1@kz1 ~]$ who
root     pts/0        2020-03-16 21:56 (172.6.185.228)
[test1@kz1 ~]$ whoami
test1
[test1@kz1 ~]$ who am i
root     pts/0        2020-03-16 21:56 (172.6.185.228)
```

