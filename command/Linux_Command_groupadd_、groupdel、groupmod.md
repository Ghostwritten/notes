#  Linux Command groupadd 、groupdel、groupmod
tags: 用户管理



---
## 1. groupadd

groupadd命令用于创建一个新的工作组，新工作组的信息将被添加到系统文件中

```bash
-g：指定新建工作组的id；
-r：创建系统工作组，系统工作组的组ID小于500；
-K：覆盖配置文件“/ect/login.defs”；
-o：允许添加组ID号不唯一的工作组。
 -n NEW_GROUP：更改组名为 NEW_GROUP
```
1.创建组群test

```bash
[root@localhost ~]# groupadd test1
```

2.创建组群test，并且设置该组群GID为800

```bash
[root@localhost ~]# grouadd -g 800 test1
```

3.创建系统组群test

```bash
[root@localhost ~]# groupadd -r test1
```



## 2. groupdel
1.groupdel 命令用于删除用户组

```bash
[root@localhost ~]# groupdel test1
```
## 3. groupmod

```bash
-g <群组识别码> 　设置欲使用的群组识别码。 
-o 　重复使用群组识别码。 
-n <新群组名称> 　设置欲使用的群组名称。
```
1.将 test1 组更名为 test2

```bash
[root@localhost ~]# groupmod -n test1 test2
```
2.将 test2 组的 GID 改为 3000

```bash
[root@localhost ~]# groupmod -g 3000 test2
```

