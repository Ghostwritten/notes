#  linux Command useradd、usermod、userdel
tags: 用户管理



---

## 1. useradd
 useradd 命令用来创建或更新用户信息。 
 

```bash
-c：加上备注文字，备注文字保存在passwd的备注栏中。
-d：指定用户登入时的主目录，替换系统默认值/home/<用户名>
-D：变更预设值。
-e：指定账号的失效日期，日期格式为MM/DD/YY，例如06/30/12。缺省表示永久有效。
-f：指定在密码过期后多少天即关闭该账号。如果为0账号立即被停用；如果为-1则账号一直可用。默认值为-1.
-g：指定用户所属的群组。值可以使组名也可以是GID。用户组必须已经存在的，期默认值为100，即users。
-G：指定用户所属的附加群组。
-m：自动建立用户的登入目录。
-M：不要自动建立用户的登入目录。
-N: 不要创建以用户名称为名的群组。
-n：取消建立以用户名称为名的群组。
-r：建立系统账号。
-s：指定用户登入后所使用的shell。默认值为/bin/bash。
-u：指定用户ID号。该值在系统中必须是唯一的。0~499默认是保留给系统用户账号使用的，所以该值必须大于499。
```
1.创建用户

```bash
useradd tester1
```

2.-N 选项，即不要生成与用户同名的群组。查看下 /etc/passwd 文件，发现 tester2 用户的初始群组ID是100。这个100是哪来的？有ID为100的群组吗？其实100作为 -N 的默认值是写在配置文件中的。不管有没有ID为100的群组，都是这个值。当然我们也可以通过修改配置文件来改变这个默认值！

```bash
useradd tester2 -N
```
3.sudo 是一个非常有权势的群组， tester3 加入到这个群组

```bash
useradd tester3 -g sudo
```
4.添加到非初始化组

```bash
useradd tester4 -G sudo
```
5.创建用户的同时创建用户的家目录，必须指定 -m 选项

```bash
 useradd -m tester5
```
6.指定家目录，但此时不生成目录 abc

```bash
useradd -d /home/abc tester6
```
7.生成目录 abcd，并且目录下默认存在文件

```bash
useradd -d /home/abcd -m tester7
```
8.创建一个带有家目录并且可以登录 bash 的用户

```bash
 useradd -m -s /bin/bash tester8
```
9.创建一个没有家目录且不能登录的用户

```bash
useradd -s /sbin/nologin tester9
```
10.创建时把用户加入不同的用户组

```bash
useradd -m -G xxx,sudo tester4
```

11.建立一个新用户账户testuser1，并设置UID为544，主目录为/usr/testuser1，属于users组：

```bash
useradd -u 544 -d /usr/test1  -g users -m  test11
```


12.新创建一个oracle用户，这初始属于oinstall组，且同时让他也属于dba组。

```bash
useradd oracle -g oinstall -G dba
```

## 2. usermod
usermod命令用于修改用户的基本信息。usermod命令不允许你改变正在线上的使用者帐号名称。当usermod命令用来改变user id，必须确认这名user没在电脑上执行任何程序。

```bash
usermod(选项)(参数)
```

选项
-c<备注>：修改用户帐号的备注文字；

```bash
$ useradd -c nihao   han
$ cat /etc/passwd |tail -1
han:x:2019:2019:nihao:/home/han:/bin/bash 
$ usermod -c hello han
$ cat /etc/passwd |tail -1
han:x:2019:2019:hello:/home/han:/bin/bash
```

把han用户的备注信息nihao改为hello

-d<登入目录>：修改用户登入时的目录；

```bash
$ usermod -d /home/qwe han
$ cat /etc/passwd |tail -1
han:x:2019:2019::/home/qwe:/bin/bash
```

-e<有效期限>：修改帐号的有效期限；

```bash
[root@centos6 ~]$ cat /etc/shadow |tail -1
han:!!:17549:0:99999:7:::
[root@centos6 ~]$ usermod -e 3 han
[root@centos6 ~]$ cat /etc/shadow |tail -1
han:!!:17549:0:99999:7::3:
```

-f<缓冲天数>：修改在密码过期后多少天即关闭该帐号；

```bash
[root@centos6 ~]$ usermod -f 4 han
[root@centos6 ~]$ cat /etc/shadow |tail -1
han:!!:17549:0:99999:7:4:3:
```

-g<群组>：修改用户所属的群组；

```bash
[root@centos6 ~]$ usermod -g yingyu han
[root@centos6 ~]$ id han
uid=2019(han) gid=506(yingyu) groups=506(yingyu)
```

-G<群组>；修改用户所属的附加群组；

```bash
[root@centos6 ~]$ usermod -G shuxue han
[root@centos6 ~]$ id han
uid=2019(han) gid=506(yingyu) groups=506(yingyu),2020(shuxue)
```

-l<帐号名称>：修改用户帐号名称；

```bash
[root@centos6 ~]$ usermod -l xiaoming han
[root@centos6 ~]$ cat /etc/passwd |tail -1
xiaoming:x:2019:506::/home/qwe:/bin/bash
[root@centos6 ~]$ id xiaoming
uid=2019(xiaoming) gid=506(yingyu) groups=506(yingyu),2020(shuxue)
```

-L：锁定用户密码，使密码无效；

```bash
[root@centos6 ~]$ usermod -L xiaoming
[root@centos6 ~]$ su lilei
[lilei@centos6 root]$ su xiaoming
Password: 
su: incorrect password
```

-s<shell>：修改用户登入后所使用的shell；

```bash
[root@centos6 ~]$ usermod -s /bin/sh xiaoming
[root@centos6 ~]$ cat /etc/passwd |tail -1
xiaoming:x:2019:506::/home/qwe:/bin/sh
```

-u<uid>：修改用户ID；

```bash
[root@centos6 ~]$ usermod -u 1995 xiaoming
[root@centos6 ~]$ cat /etc/passwd |tail -1
xiaoming:x:1995:506::/home/qwe:/bin/sh
```

-U:解除密码锁定。

```bash
[root@centos6 /home]$ usermod -U xiaoming
[root@centos6 /home]$ su lilei
[lilei@centos6 home]$ su xiaoming
Password: 
[xiaoming@centos6 home]$
```


## 3. userdel
userdel命令用于删除给定的用户，以及与用户相关的文件。若不加选项，则仅删除用户帐号，而不删除相关文件。
语法

```bash
userdel(选项)(参数)
```

选项
-f：强制删除用户，即使用户当前已登录；

```bash
[root@centos6 ~]$ userdel wangcai
userdel: user wangcai is currently used by process 2727
[root@centos6 ~]$ cat /etc/passwd|tail -1
wangcai:x:2019:2021::/home/wangcai:/bin/bash
[root@centos6 ~]$ userdel -f wangcai
userdel: user wangcai is currently used by process 2727
[root@centos6 ~]$ cat /etc/passwd|tail -1
han:x:2018:2018::/home/han:/bin/bash
```

-r：删除用户的同时，删除与用户相关的所有文件。

```bash
[root@centos6 ~]$ ls /home/
lilei  wangcai
[root@centos6 ~]$ userdel -r lilei
[root@centos6 ~]$ ls /home/
wangcai
```
刚才删除wangcai这个用户时没有加-r参数所以还保存有它的家目录，加-r用户lilei家目录就没了

**请不要轻易用-r选项；他会删除用户的同时删除用户所有的文件和目录，切记如果用户目录下有重要的文件，在删除前请备份。**
其实也有最简单的办法，但这种办法有点不安全，也就是直接在/etc/passwd中删除您想要删除用户的记录；但最好不要这样做，/etc/passwd是极为重要的文件，可能您一不小心会操作失误。
