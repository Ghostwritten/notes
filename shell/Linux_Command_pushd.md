#  Linux Command pushd



---
## 1. 简介
pushed命令用于将目录加入堆栈中，加入记录到目录栈顶部，并切换到该目录，如果不加任何参数， 则会将位于记录栈最上面的两个目录对换位置。
## 2. 格式

```bash
pushed [参数] [目录]
```
## 3. 参数

```bash
-n	将右起第n个目录移到堆栈顶（n从0开始计数）
+n	将左起第n个目录移到堆栈顶（n从0开始计数）
```
## 4. 实例

```bash
显示Shell默认的目录堆栈：

[root@linuxcool ~]# dirs
/var/log
向Shell目录堆栈中添加目录/etc：

[root@linuxcool ~]# pushed /etc
显示Shell目录堆栈内容，每行一个记录，并加序号：

[root@linuxcool ~]# dirs -v
0 /sbin
1 /etc
2 /var/log
将第一个记录移到堆栈顶：

[root@linuxcool ~]# pushed -1
/etc /var/log /sbin

要禁止默认更改目录，请使用-n选项，例如，要将/usr/local目录添加到堆栈但不更改到堆栈
[root@linuxcool ~]# pushd -n /usr/local
/etc /usr/local /var/log /sbin

从上到下（或从左到右）计数时，目录的索引为2：相当于cd

pushd +2

从下到上计数时，/var/www目录的索引为1：

pushd -1.

```
更多阅读：

 - [pushd Command in Linux with Examples](https://www.geeksforgeeks.org/pushd-command-in-linux-with-examples/)
 - [Pushd and Popd Commands in Linux](https://linuxize.com/post/popd-and-pushd-commands-in-linux/)

