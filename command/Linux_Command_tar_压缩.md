# Linux Command tar 压缩
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/be102254d6c0ace5f1deb358c045834d.gif#pic_center)



##  1. 简介
[tar](https://www.gnu.org/software/tar/)命令可以为linux的文件和目录创建档案。利用tar，可以为某一特定文件创建档案（备份文件），也可以在档案中改变文件，或者向档案中加入新的文件。
首先要弄清两个概念：打包和压缩。打包是指将一大堆文件或目录变成一个总的文件；压缩则是将一个大的文件通过一些压缩算法变成一个小文件。

为什么要区分这两个概念呢？这源于Linux中很多压缩程序只能针对一个文件进行压缩，这样当你想要压缩一大堆文件时，你得先将这一大堆文件先打成一个包（tar命令），然后再用压缩程序进行压缩（gzip bzip2命令）
## 2. 格式

tar[必要参数][选择参数][文件] 



##  3. 参数

```c
-A 新增压缩文件到已存在的压缩
-B 设置区块大小
-c 建立新的压缩文件
-d 记录文件的差别
-r 添加文件到已经压缩的文件
-u 添加改变了和现有的文件到已经存在的压缩文件
-x 从压缩的文件中提取文件
-t 显示压缩文件的内容
-z 支持gzip解压文件
-j 支持bzip2解压文件
-Z 支持compress解压文件
-v 显示操作过程
-l 文件系统边界设置
-k 保留原有文件不覆盖
-m 保留文件不被覆盖
-W 确认压缩文件的正确性
可选参数如下：
-b 设置区块数目
-C 切换到指定目录
-f 指定压缩文件
--help 显示帮助信息
--version 显示版本信息
```
##  4. 举例
### 4.1 将文件全部打包成tar包


```bash
[root@localhost test]$ ls -al log2012.log
---xrw-r-- 1 root root 302108 11-13 06:03 log2012.log
[root@localhost test]$ tar -cvf log.tar log2012.log 
log2012.log
[root@localhost test]$ tar -zcvf log.tar.gz log2012.log
log2012.log
[root@localhost test]$ tar -jcvf log.tar.bz2 log2012.log 
log2012.log
[root@localhost test]$ ls -al *.tar*
-rw-r--r-- 1 root root 307200 11-29 17:54 log.tar
-rw-r--r-- 1 root root   1413 11-29 17:55 log.tar.bz2
-rw-r--r-- 1 root root   1413 11-29 17:54 log.tar.gz
```

说明：

 - tar -cvf log.tar log2012.log    仅打包，不压缩！
 - tar -zcvf log.tar.gz log2012.log   打包后，以 gzip 压缩
 - tar -zcvf log.tar.bz2 log2012.log  打包后，以 bzip2 压缩

在参数 f 之后的文件档名是自己取的，我们习惯上都用 .tar 来作为辨识。 如果加 z 参数，则以 `.tar.gz` 或 `.tgz` 来代表 gzip 压缩过的 tar包； 如果加 j 参数，则以 .tar.bz2 来作为tar包名。

### 4.2 查阅上述 tar包内有哪些文件
```bash
[root@localhost test]$ tar -ztvf log.tar.gz
---xrw-r-- root/root    302108 2012-11-13 06:03:25 log2012.log
```

说明：
由于我们使用 gzip 压缩的log.tar.gz，所以要查阅log.tar.gz包内的文件时，就得要加上 z 这个参数了。

### 4.3 将tar 包解压缩

```bash
[root@localhost test3]$ ll
总计 0
[root@localhost test3]$ tar -zxvf /opt/soft/test/log.tar.gz
log2012.log
[root@localhost test3]$ ls
log2012.log
```

说明：
在预设的情况下，我们可以将压缩档在任何地方解开的

### 4.4 只将 /tar 内的 部分文件解压出来

```bash
[root@localhost test]$ tar -zcvf log30.tar.gz log2012.log log2013.log 
log2012.log
log2013.log
[root@localhost test]$ ls -al log30.tar.gz 
-rw-r--r-- 1 root root 1512 11-30 08:19 log30.tar.gz
[root@localhost test]$ tar -zxvf log30.tar.gz log2013.log
log2013.log
[root@localhost test]$ ll
-rw-r--r-- 1 root root   1512 11-30 08:19 log30.tar.gz
[root@localhost test]$ cd test3
[root@localhost test3]$ tar -zxvf /opt/soft/test/log30.tar.gz log2013.log
log2013.log
[root@localhost test3]$ ll
总计 4
-rw-r--r-- 1 root root 61 11-13 06:03 log2013.log
```

说明：
我可以透过 tar -ztvf 来查阅 tar 包内的文件名称，如果单只要一个文件，就可以透过这个方式来解压部分文件！

### 4.5 文件备份下来，并且保存其权限

```bash
[root@localhost test]# ll
总计 0
-rw-r--r-- 1 root root      0 11-13 06:03 log2014.log
-rw-r--r-- 1 root root      0 11-13 06:06 log2015.log
-rw-r--r-- 1 root root      0 11-16 14:41 log2016.log
[root@localhost test]$ tar -zcvpf log31.tar.gz log2014.log log2015.log log2016.log 
log2014.log
log2015.log
log2016.log
[root@localhost test]$ cd test6
[root@localhost test6]$ ll
[root@localhost test6]$ tar -zxvpf /opt/soft/test/log31.tar.gz 
log2014.log
log2015.log
log2016.log
[root@localhost test6]$ ll
总计 0
-rw-r--r-- 1 root root 0 11-13 06:03 log2014.log
-rw-r--r-- 1 root root 0 11-13 06:06 log2015.log
-rw-r--r-- 1 root root 0 11-16 14:41 log2016.log
```

说明：
这个 -p 的属性是很重要的，尤其是当您要保留原本文件的属性时

### 4.6 在文件夹当中，比某个日期新的文件才备份

```bash
[root@localhost soft]$ tar -N "2012/11/13" -zcvf log17.tar.gz test
tar: Treating date `2012/11/13' as 2012-11-13 00:00:00 + 0 nanoseconds
test/test/log31.tar.gz
test/log2014.log
test/linklog.log
test/log2015.log
test/log2013.log
test/log2012.log
test/log2017.log
test/log2016.log
test/log30.tar.gz
test/log.tar
test/log.tar.bz2
test/log.tar.gz
```


### 4.7 备份文件夹内容是排除部分文件
```bash
[root@localhost test]$ tree scf
scf
|-- bin
|-- doc
|-- lib
|-- service

[root@localhost test]$ tar --exclude scf/service -zcvf scf.tar.gz scf/* 
scf/bin/
scf/doc/
scf/lib/
```

###  4.8 分卷压缩与解压缩
将10G大小的log文件2017.log打包压缩并分割成多个100m的文件

```bash
# 分卷压缩gz
$ tar zcf - 2017.log |split -d -b 100m - logs.tar.gz.
# 生成文件： logs.tar.gz.00 logs.tar.gz.01
# 分卷压缩bz2
$ tar jcf - 2017.log |split -d -b 100m - logs.tar.bz2.
# 生成文件： logs.tar.bz2.00 logs.tar.bz2.01
```

```bash
合并分卷解压缩
# 解压gz分卷
$ cat logs.tar.gz* | tar zx
# 解压bz2分卷
$ cat logs.tar.bz2* | tar jx
```

更多阅读：
 1. [Linux Command lz4 压缩](https://blog.csdn.net/xixihahalelehehe/article/details/125201619)
 2. [Linux Command tar 压缩](https://ghostwritten.blog.csdn.net/article/details/106056841)
 3. [Linux Command gzip 压缩](https://blog.csdn.net/xixihahalelehehe/article/details/104724596)
 4. [Linux Command zip 压缩](https://blog.csdn.net/xixihahalelehehe/article/details/125203106)



![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/50ccb9c82ede5ebe51735624de5aff68.png)

