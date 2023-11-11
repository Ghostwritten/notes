#  Linux Command cut 切割




---
![在这里插入图片描述](https://img-blog.csdnimg.cn/3f9909aef33c4218b8400b9d931e99a4.gif#pic_center)
## 1. 简介

cut 命令从文件的每一行剪切字节、字符和字段并将这些字节、字符和字段写至标准输出。

如果不指定 File 参数，cut 命令将读取标准输入。必须指定 -b、-c 或 -f 标志之一。

## 2. 参数

```bash
-b ：以字节为单位进行分割。这些字节位置将忽略多字节字符边界，除非也指定了 -n 标志。
-c ：以字符为单位进行分割。
-d ：自定义分隔符，默认为制表符。
-f ：与-d一起使用，指定显示哪个区域。
-n ：取消分割多字节字符。仅和 -b 标志一起使用。如果字符的最后一个字节落在由 -b 标志的 List 参数指示的
```

范围之内，该字符将被写出；否则，该字符将被排除


```bash
第一，字节（bytes），用选项-b
第二，字符（characters），用选项-c
第三，域（fields），用选项-f
```
## 3. 实例
### 3.1 以“字节”定位

```bash
$ who
rocrocket :0           2009-01-08 11:07
rocrocket pts/0        2009-01-08 11:23 (:0.0)
rocrocket pts/1        2009-01-08 14:15 (:0.0)
如果我们想提取每一行的第3个字节，就这样：
$ who|cut -b 3
c
c
c
```

 如果“字节”定位中，我想提取第3，第4、第5和第8个字节，怎么办?
-b支持形如3-5的写法，而且多个定位之间用逗号隔开就成了。看看例子吧：

```bash
$ who|cut -b 3-5,8
croe
croe
croe
```


### 3.2 给个以字符为定位

```bash
$ cat cut_ch.txt
星期一
星期二
星期三
星期四
$ cut -c 3 cut_ch.txt
一
二
三
四
```


### 3.3 以域为标志

```bash
$ cat /etc/passwd|head -n 5
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin

$ cat /etc/passwd|head -n 5|cut -d : -f 1
root
bin
daemon
adm
lp

$ cat /etc/passwd|head -n 5|cut -d : -f 1,3-5
root:0:0:root
bin:1:1:bin
daemon:2:2:daemon
adm:3:4:adm
lp:4:7:lp
```

### 3.4 比较一列数字的大小

```bash
cat test | cut -d " " -f 1 | sort -n  | head 
```

更多阅读：

 - [Cut Command in Linux](https://linuxize.com/post/linux-cut-command/)
 - [cut command in Linux with examples](https://www.geeksforgeeks.org/cut-command-linux-examples/)



