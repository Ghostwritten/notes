#  Linux Command du 查看文件大小



---

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4e130d6c197c7c6a559af2ed53025285.gif#pic_center)


##  1. 简介
Linux du显示每个文件和目录的磁盘使用空间。，但是与df命令不同的是Linux du命令是对文件和目录磁盘使用的空间的查看。

## 2. 格式

```bash
du [选项][文件]
```

##  3. 参数

```bash
-a或-all  显示目录中个别文件的大小。   

-b或-bytes  显示目录或文件大小时，以byte为单位。   

-c或--total  除了显示个别目录或文件的大小外，同时也显示所有目录或文件的总和。 

-k或--kilobytes  以KB(1024bytes)为单位输出。

-m或--megabytes  以MB为单位输出。   

-s或--summarize  仅显示总计，只列出最后加总的值。

-h或--human-readable  以K，M，G为单位，提高信息的可读性。

-x或--one-file-xystem  以一开始处理时的文件系统为准，若遇上其它不同的文件系统目录则略过。 

-L<符号链接>或--dereference<符号链接> 显示选项中所指定符号链接的源文件大小。   

-S或--separate-dirs   显示个别目录的大小时，并不含其子目录的大小。 

-X<文件>或--exclude-from=<文件>  在<文件>指定目录或文件。   

--exclude=<目录或文件>         略过指定的目录或文件。    

-D或--dereference-args   显示指定符号链接的源文件大小。   

-H或--si  与-h参数相同，但是K，M，G是以1000为换算单位。   

-l或--count-links   重复计算硬件链接的文件。  
```

## 4. 实例

###  4.1 显示目录或者文件所占空间 

```bash
[root@localhost test]# du
[root@data2 ~]# du
8	./.ssh
4	./.cache/abrt
4	./.cache
0	./.config/abrt
0	./.config
1100	./cos
0	./.distlib/resource-cache
0	./.distlib
456	./pip-package
4	./.pip
7696	./xtrabackup
4	./.docker
48	./ansible-check/rds-os-precheck
52	./ansible-check/rds_check_az1
48	./ansible-check/rds_check_az2
176	./ansible-check

```
### 4.2 显示指定文件所占空间

[root@localhost test]# du log2012.log 

300     log2012.log

### 4.3 查看指定目录的所占空间

```bash
[root@localhost test]# du scf
4       scf/lib
4       scf/service/deploy/product
4       scf/service/deploy/info
12      scf/service/deploy
16      scf/service
4       scf/doc
4       scf/bin
32      scf
```
### 4.4 显示多个文件所占空间

```bash
[root@localhost test]# du log30.tar.gz log31.tar.gz 
4       log30.tar.gz
4       log31.tar.gz
```
### 4.5 只显示总和的大小

```bash
[root@localhost test]# du -s
1288    .
[root@localhost test]# du -s scf
32      scf
[root@localhost test]# cd ..
[root@localhost soft]# du -s test
1288    test
```
### 4.6 方便阅读的格式显示

```bash
[root@localhost soft]# du -h test
608K    test/test6
308K    test/test4
4.0K    test/scf/lib
4.0K    test/scf/service/deploy/product
4.0K    test/scf/service/deploy/info
12K     test/scf/service/deploy
16K     test/scf/service
4.0K    test/scf/doc
4.0K    test/scf/bin
32K     test/scf
8.0K    test/test3
1.3M    test
```

### 4.7 文件和目录都显示

```bash
[root@localhost soft]# du -ah test
4.0K    test/log31.tar.gz
4.0K    test/test13.tar.gz
0       test/linklog.log
0       test/test6/log2014.log
300K    test/test6/linklog.log
0       test/test6/log2015.log
4.0K    test/test6/log2013.log
300K    test/test6/log2012.log
```
### 4.8 显示指定文件占用磁盘空间
还统计它们的总和

```bash
[root@localhost test]# du -c log30.tar.gz log31.tar.gz 
4       log30.tar.gz
4       log31.tar.gz
8       总计
```
### 4.9 按照空间大小排序

```bash
[root@localhost test]# du|sort -nr|more
1288    .
608     ./test6
308     ./test4
32      ./scf
16      ./scf/service
```
### 4.10 输出当前目录下各个子目录所使用的空间

```bash
[root@localhost test]# du -h  --max-depth=1
608K    ./test6
308K    ./test4
32K     ./scf
8.0K    ./test3
1.3M    .
```

