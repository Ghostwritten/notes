#  Linux Command gzip 压缩
![](https://i-blog.csdnimg.cn/blog_migrate/85b54082b006c96ade8e8c7c393eba3c.gif#pic_center)






## 1. 简介
[gzip](https://www.gnu.org/software/gzip/) 命令作用不能是目录,只能压缩单个文件。

## 2. 安装
下载 [gzip](https://mirrors.ocf.berkeley.edu/gnu/gzip/)
```bash
准备编译 Gzip：

./configure --prefix=/usr --bindir=/bin
编译软件包：

make
用以下命令测试结果：

make check
安装软件包：

make install
移动不需要在根文件系统的程序：

mv -v /bin/{gzexe,uncompress,zcmp,zdiff,zegrep} /usr/bin
mv -v /bin/{zfgrep,zforce,zgrep,zless,zmore,znew} /usr/bin
```


## 3. 格式

```bash
gunzip [-acfhlLnNqrtvV][-s ][文件...]
```

或者gunzip [-acfhlLnNqrtvV][-s ][目录]

## 4. 参数

```bash
-a或--ascii：使用ASCII文字模式。
-c或--stdout或--to-stdout：把解压后的文件输出到标准输出设备。
-d或--decompress或----uncompress 　解开压缩文件
-f或-force：强行解开压缩文件，不理会文件名称或硬连接是否存在，以及该文件是否为符号连接。
-h或--help：在线帮助。
-l或--list：列出压缩文件的相关信息。
-L或--license：显示版本与版权信息。
-n或--no-name：解压缩时，若压缩文件内含有原来的文件名称及时间戳记，则将其忽略不予处理。
-N或--name：解压缩时，若压缩文件内含有原来的文件名称及时间戳记，则将其回存到解开的文件上。
-q或--quiet：不显示警告信息。
-r或--recursive：递归处理，将指定目录下的所有文件及子目录一并处理。
-S或--suffix：更改压缩字尾字符串。
-t或--test：测试压缩文件是否正确无误。
-v或--verbose：显示指令执行过程。
-V或--version：显示版本信息。
```
## 5. 举例
1.Linux压缩保留源文件的方法：

```bash
gzip -c filename > filename.gz
```

2.Linux解压缩保留源文件的方法：

```bash
gunzip -c filename.gz > filename
```
3.把多文件不保留源文件压缩成.gz文件

```bash
$ ls
01.txt  02.txt  03.txt  04.txt  05.txt  06.txt  
$ gzip *
$ ls
01.txt.gz  02.txt.gz  03.txt.gz  04.txt.gz  05.txt.gz  06.txt.gz 
```
4.详细显示每个压缩的文件的信息，并不解压

```bash
$ gzip -l *
         compressed        uncompressed  ratio uncompressed_name
                 26                   0   0.0% 1.txt
                 26                   0   0.0% 2.txt
                 26                   0   0.0% 3.txt
                 26                   0   0.0% 4.txt
                 26                   0   0.0% 5.txt
                 26                   0   0.0% 6.txt
```

5.每个压缩的文件解压，并列出详细的信息

```bash
$ gzip -dv *
01.txt.gz:        0.0% -- replaced with 01.txt
02.txt.gz:        0.0% -- replaced with 02.txt
03.txt.gz:        0.0% -- replaced with 03.txt
04.txt.gz:        0.0% -- replaced with 04.txt
05.txt.gz:        0.0% -- replaced with 05.txt
06.txt.gz:        0.0% -- replaced with 06.txt
$ ls
01.txt  02.txt  03.txt  04.txt  05.txt  06.txt  
```
6.压缩一个tar备份文件，此时压缩文件的扩展名为.tar.gz

```bash
$ ls log.tar
 log.tar
$ gzip -r log.tar
$ ls
 log.tar.gz
```
7.递归的压缩目录

```bash
$  ls
1.txt  2.txt  3.txt  4.txt  5.txt  6.txt
$ cd ..
$ gzip -rv dir1/
dir1//1.txt:	  0.0% -- replaced with dir1//1.txt.gz
dir1//2.txt:	  0.0% -- replaced with dir1//2.txt.gz
dir1//3.txt:	  0.0% -- replaced with dir1//3.txt.gz
dir1//4.txt:	  0.0% -- replaced with dir1//4.txt.gz
dir1//5.txt:	  0.0% -- replaced with dir1//5.txt.gz
dir1//6.txt:	  0.0% -- replaced with dir1//6.txt.gz
$ ls dir1/
1.txt.gz  2.txt.gz  3.txt.gz  4.txt.gz  5.txt.gz  6.txt.gz
```
8.递归地解压目录

```bash
$ gzip -dr dir1/
$ ls dir1/
1.txt  2.txt  3.txt  4.txt  5.txt  6.txt
```


更多阅读：
 1. [Linux Command lz4 压缩](https://blog.csdn.net/xixihahalelehehe/article/details/125201619)
 2. [Linux Command tar 压缩](https://ghostwritten.blog.csdn.net/article/details/106056841)
 3. [Linux Command gzip 压缩](https://blog.csdn.net/xixihahalelehehe/article/details/104724596)
 4. [Linux Command zip 压缩](https://blog.csdn.net/xixihahalelehehe/article/details/125203106)


![](https://i-blog.csdnimg.cn/blog_migrate/7155171cf01db09e2c8e535f26c80801.png)



