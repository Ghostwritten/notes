#  Linux Command sed 文本处理



----
## 1. 简介

Sream EDitor流式编辑器（非交互）===【vim（交互） word,记事本】
非交互，基于模式匹配过滤及修改文本
逐行处理，并将结果输出到屏幕
可实现对文本的输出，删除，替换，复制，粘贴，导出，导入等操作

## 2. 语法

```bash
sed [选项] '条件命令' 文件
```

##  3. 参数

```bash
-e：多点编辑，可以执行多个子命令
-f：从脚本文件中读取命令（sed操作可以事先写入脚本，然后通过-f读取并执行）
-l：指定行的长度
-n：屏蔽sed的默认输出功能
-i：修改源文件
-r：sed支持扩展正则
-{}:可组合多个命令
-p:打印行
-s:字符串替换   sed 's/ 旧  / 新  /'  a.txt
-d：删除行

#文本块处理：针对整行的插入追加替换操作
i: （insert）插入到行前
a: （append）追加到行后
c: （change）替换修改某行
r：从文件中读取输入行
w：另存为某文件
H：复制（模式空间追加复制
h：复制（模式空间覆盖
G：粘贴（保持空间追加模式空间
G：粘贴（保持空间覆盖模式空间
w	 将所选的行写入文件
x	 交换暂存缓冲区与模式空间的内容
y	 将字符替换为另一字符（不能对正则表达式使用y命令）
```

##  4. 举例
### 4.1  p：打印行

```c
$ sed '1p' /etc/rc.local 
#!/bin/bash
# THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES
# Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure
# that this script will be executed during boot.
touch /var/lock/subsys/local

$ sed -n '1p' /etc/rc.local   $打印单行
#!/bin/bash

$  sed -n '1p;4p' /etc/rc.local $打印1和4行
#!/bin/bash
# that this script will be executed during boot.
$ sed -n '2,+3p' /etc/rc.local $打印从第二行数三行
# THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES
# Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure
# that this script will be executed during boot.
touch /var/lock/subsys/local
$ sed -n '/local$/p' /etc/rc.local $打印以local结尾的行
touch /var/lock/subsys/local
$ sed -n 'p;n' /etc/rc.local $打印奇数行
#!/bin/bash
# Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure
touch /var/lock/subsys/local
$ sed -n 'n;p' /etc/rc.local $打印偶数行
# THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES
# that this script will be executed during boot.
$ sed -n '$=' /etc/rc.local $显示函数
6
```
### 4.2 d：删除行

```c
$ cat  /etc/rc.local 
#!/bin/bash
# THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES
# Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure
# that this script will be executed during boot.
touch /var/lock/subsys/local
$ sed '1,4d' /etc/rc.local $并未真1和4行删除
touch /var/lock/subsys/local
$ cat  /etc/rc.local 
#!/bin/bash
# THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES
# Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure
# that this script will be executed during boot.
touch /var/lock/subsys/local
$ sed -i  '1,4d' /etc/rc.local 真删除和4行
$ cat /etc/rc.local
touch /var/lock/subsys/local

其他删除方法
sed '/init/d' /etc/rc.local                 $删除init的行
sed '/init/d;/bin/d' /etc/rc.local          $删除init和bin的行
sed '/init/!d' /etc/rc.local                $删除不包含init的行
sed '$d' /etc/rc.local                      $删除最后一行
sed '/^$/d' /etc/rc.local                   $删除空行
```

### 4.3 s：字符串替换 
```c
sed 's/ll/TARENA/' /etc/rc.local          $替换所有行的第一“||”
sed 's/ll/TARENA/g'  /etc/rc.local        $替换所有行所有的“||”
sed '3s/script/SCRIPT/2' /etc/rc.local    $替换第三那行的第二个“script”
sed 's/init//g' /etc/rc.local             $删除文件所有的“init”
sed 's/script\|stuff\|e//g' /etc/rc.local $删除所有的“script”、所有的“stuff”、所有的字母e
sed '3,5s/^#//' /etc/rc.local            $解除/etc/rc.local文件第3~5行的注释（去掉开头的$
sed '6,7s/^/#/' /etc/rc.local           $将/etc/rc.local文件的第6~7行注释掉（行首添加$ ）
sed 's/.//2;s/.$//'/etc/rc.local       $删除文件中每行的第二个、最后一个字符
```
替换指定行的下一行内容
```bash
$ cat test.txt
i love you
name id year
//n命令-->移动到匹配行的下一行
sed -i '/i love you/{n;s/year/year1/;}'  test.txt
i love you
name id year1
```
### 4.4 -r：sed支持扩展正则
```c
sed -r 's/^(.)(.)(.*)/\2\1\3/' /etc/rc.local  $每行文本拆分为“第1个字符”“第2个字符”“剩下的所有字符”三个部分，然后通过替换操作重排顺序为“2-1-3
sed -r 's/^(.)(.*)(.)$/\3\2\1/' /etc/rc.local   $第一个字符和最后一个字符对调
sed -r 's/[0-9]//g;s/^( )+//' /etc/rc.local   $删除所有数字、行首空格
sed 's/[A-Z]/(&)/g' /etc/rc.local            $使用“&”可调用s替换操作中的整个查找串
sed -i '/^id:/s/3/5/' /etc/inittab       $将默认运行级别修改为5 ----------查看效果grep "^id:" /etc/inittab
cat /etc/sysconfig/network-scripts/ifcfg-eth0
sed '/^IPADDR/s/192.168.4.4/172.16.16.4/’/etc/sysconfig/network-scripts/ifcfg-eth0 | grep "^IPADDR"  $将IP地址192.168.4.4修改为172.16.16.4
sed -r -i '/^IPADDR/s/192.168.4.(.*)/172.16.16.\1/' /etc/sysconfig/network-scripts/ifcfg-eth0  $要求只修改网段地址时，可以利用扩展正则表达式的 \1、\2、……等调用，分别对应此前第1个、第2个、…… 以 ()包围的表达式所匹配的内容
grep "^IPADDR" /etc/sysconfig/network-scripts/ifcfg-eth0
sed -i 's#/var/www/html#/opt/wwwroo$' /etc/httpd/conf/httpd.conf $更改网站根目录
grep "^DocumentRoot" /etc/httpd/conf/httpd.conf $查看修改结果
```
###  4.5 多行注释
```bash
$ cat tt.txt 
Aprl:1000M:2000
Jan:400G:1000
Feb:30K:100\
May:45K:111

$ sed '/Aprl/,+2s/\(.*\)/#&/' tt.txt 
#Aprl:1000M:2000
#Jan:400G:1000
#Feb:30K:100\
May:45K:111

$ sed '/Aprl/,/Feb/s/\(.*\)/#&/' tt.txt 
#Aprl:1000M:2000
#Jan:400G:1000
#Feb:30K:100\
May:45K:111
```


###  4.6 r：从文件中读取输入行

```c
$ cat a.txt 
192.168.1.110 www.baidu.com

$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.1.120 oss.com

$ sed '/oss/r a.txt' /etc/hosts 
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.1.120 oss.com
192.168.1.110 www.baidu.com
```
### 4.7  w：另存为某文件
```c
$ cat a.txt 
192.168.1.110 www.baidu.com
$ sed -i 'w c.txt' a.txt 
$ cat c.txt 
192.168.1.110 www.baidu.com
$ sed -i '/oss/w d.txt' /etc/hosts
$ cat d.txt 
192.168.1.120 oss.yun.ccb.com
```
###  4.8 H:h:G:g 复制粘贴

H：复制（模式空间追加复制
h：复制（模式空间覆盖
G：粘贴（保持空间追加模式空间
g：粘贴（保持空间覆盖模式空间
```c
[root@monitor1 ~]# sed '1h;1g'  mm
aaaa
bbbb
cccc

[root@monitor1 ~]# sed '1h;2g'  mm
aaaa
aaaa
cccc

[root@monitor1 ~]# sed '1h;3g'  mm
aaaa
bbbb
aaaa
[root@monitor1 ~]# sed '2h;1g'  mm

bbbb
cccc

[root@monitor1 ~]# sed '3h;1g'  mm

bbbb
cccc
[root@monitor1 ~]# sed '1H;1G'  mm
aaaa

aaaa
bbbb
cccc
[root@monitor1 ~]# sed '1H;2G'  mm
aaaa
bbbb

aaaa
cccc
[root@monitor1 ~]# sed '2H;1G'  mm
aaaa

bbbb
cccc
```
### 4.9 行前后添加
```bash
sed '1i 添加的内容' file $这是在第一行前添加字符串
sed '$i 添加的内容' file $这是在最后一行行前添加字符串
sed '$a添加的内容' file $这是在最后一行行后添加字符串
```

```c
$ cat mm
aaaa
bbbb
cccc

$ sed '1i 0000' mm
0000
aaaa
bbbb
cccc

$ sed '$i 0000' mm
aaaa
bbbb
0000
cccc

$sed '$a 0000' mm
aaaa
bbbb
cccc
0000
```

```bash
$ cat test 
aaa
bbb
ccc
```
有某aaa字符串行的下一行加

```bash
$ sed '/aaa/a111' test 
aaa
111
bbb
ccc
```
有某aaa字符串行的上一行加

```bash
$ sed '/aaa/i111' test 
111
aaa
bbb
ccc
```

### 4.10 多匹配模式

```bash
$ nl 1.txt 
     1  This is my cat,my cat's name is betty
     2  This is my dog,my dog's name is frank
     3  This is my fish, my fish's name is george
     4  This is my goat,my goat's name is adam
```

#### 4.11.1 管道多匹配
去掉 1.txt 中第三行及后面的内容，并把 name 替换成 mingzi：

```bash
$ nl 1.txt | sed '3,$d' | sed 's/name/mingzi/g'
     1  This is my cat,my cat's mingzi is betty
     2  This is my dog,my dog's mingzi is frank
```

#### 4.11.2  -e 多匹配
去掉 1.txt 中第三行及后面的内容，并把 name 替换成 mingzi：

```bash
$ nl 1.txt | sed  -e  '3,$d'  -e  's/name/mingzi/g'   
     1  This is my cat,my cat's mingzi is betty
     2  This is my dog,my dog's mingzi is frank
```

#### 4.12.3 ; 多匹配

去掉 1.txt 中第三行及后面的内容，并把 name 替换成 mingzi：

```bash
$ nl 1.txt | sed '3,$d; s/name/mingzi/g'       
     1  This is my cat,my cat's mingzi is betty
     2  This is my dog,my dog's mingzi is frank
```


### 4.11  替换某种特定格式的行
把下面

```bash
![[Pasted image 20230424183144.png]]

![[Pasted image 20230424183401.png]]

![[Pasted image 20230424183551.png]]
```
替换为

```bash
(img/Tanzu_NSX_Avi_ALB_install/Pasted%20image%2020230424183144.png)]

(img/Tanzu_NSX_Avi_ALB_install/Pasted%20image%2020230424183401.png)]

(img/Tanzu_NSX_Avi_ALB_install/Pasted%20image%2020230424183551.png)]
```

命令

```bash
sed -i 's/!\[\[Pasted image \(.*\)\.png\]\]/(img\/Tanzu_NSX_Avi_ALB_install\/Pasted%20image%20\1.png)]/g' your_file.md
```


![在这里插入图片描述](https://img-blog.csdnimg.cn/198a1290fb1a4b92a4ae2dcf6cc3877e.gif#pic_center)

