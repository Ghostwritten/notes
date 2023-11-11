# Linux Command passwd、gpasswd
tags: 用户管理



---

## 1. passwd
Linux passwd命令用来更改使用者的密码

###  1.1 参数
```bash
-d 删除密码
-f 强制执行
-k 更新只能发送在过期之后
-l 停止账号使用
-S 显示密码信息
-u 启用已被停止的账户
-x 设置密码的有效期
-g 修改群组密码
-i 过期后停止用户账号
选择参数：

--help 显示帮助信息
--version 显示版本信息
```
###  1.2 交互修改密码
passwd 命令后面不接任何参数或用户名，则表示修改当前用户的密码；请看下面的例子；

```bash
$ passwd   注：没有加任何用户，我是用root用户来执行的passwd 表示修改root用户的密码；下面也有提示；
Changing password for user root.
New UNIX password: 注：请输入新密码；
Retype new UNIX password: 注：验证新密码；
passwd: all authentication tokens updated successfully. 注：修改root密码成功；
```

如果是普通用户执行passwd 只能修改自己的密码；

如果新建用户后，要为新用户创建密码，则用 passwd 用户名 ，注意要以root用户的权限来创建；

```bash
$ passwd beinan 注：更改或创建beinan用户的密码；
Changing password for user beinan.
New UNIX password: 注：请输入新密码；
Retype new UNIX password: 注：再输入一次；
passwd: all authentication tokens updated successfully. 注：成功；
```
### 1.3 非交互修改密码

```bash
echo "123" | passwd --stdin liming
```

### 1.4 查看用户密码的状态

```bash
$ passwd -S liming
liming PS 2013-01-06 0 99999 7 -1 (Password set, SHA512 crypt.)
```
意思依次是：用户名 密码 设定时间(2013*01-06) 密码修改间隔时间(0) 密码有效期(99999) 警告时间(7) 密码不失效(-1)，密码已使用

"`-S`"选项会显示出密码状态，这里的密码修改间隔时间、密码有效期、警告时间、密码宽限时间其实分别是 /etc/shadow 文件的第四、五、六、七个字段的内容。

### 1.5 修改密码 60 天变更、10 天密码失效

```bash
$ passwd -x 60 -i 10 liming
$ passwd -S liming
liming PS 2013-01-06 0 60 7 10 (Password set, SHA512 crypt.)
```
这里显示 SHA512 为密码加密方式，CentOS 6.3 加密方式已经从 MD5 加密更新到 SHA512 加密

###  1.6 锁定用户

```bash
$ passwd -I lamp
Locking password for user .
passwd:Successg

$ passwd -S lamp
lamp LK 2013-01-06 0 99999 7 -1 (Password locked.)

$ grep "lamp" /etc/shadow
lamp:!! $6$ZTq7o/9o $lj07iZ0bzW.D1zBa9CsY43d04onskUCzjwiFMNt8PX4GXJoHX9zA1S C9.i Yzh9LZA4fEM2lg92hM9w/p6NS50.:15711:0:99999:7:::
```

锁定其实就是在加密密码之前加入了"!!"，让密码失效而已

###  1.7 解锁用户

```bash
$ passwd -u lamp
Unlocking password for user lamp.
passwd:Success
$ passwd -S lamp
lamp PS 2013-01-06 0 99999 7 -1 (Password set, SHA512 crypt.)
#可以看到，锁定状态消失
$ grep "lamp" /etc/shadow
lamp: $6$ZTq7cV9o $lj07iZ0bzW.D1zBa9CsY43d04onskUCzjwiFMNt8PX4GXJoHX9zA1S C9.iYz h9LZA4fEM2lg92hM9w/p6NS50.:15711:0:99999:7:::
```

## 2. gpasswd
gpasswd命令 是Linux下工作组文件/etc/group和/etc/gshadow管理工具。

```bash
-a：添加用户到组；
-d：从组删除用户；
-A：指定管理员；
-M：指定组成员和-A的用途差不多；
-r：删除密码；
-R：限制用户登入组，只有组中的成员才可以用newgrp加入该组。
```
1.将userA添加到groupB用户组里面：  
```bash
gpasswd -a userA groupB
```
**注意**：添加用户到某一个组可以使用  usermod -G groupB userA 这个命令可以添加一个用户到指定的组，但是以前添加的组就会清空掉， 所以想要添加一个用户到一个组，同时保留以前添加的组时，请使用gpasswd这个命令来添加操作用户。
2.将userA设置为groupA的群组管理员：

```bash
gpasswd -A userA groupA
```

