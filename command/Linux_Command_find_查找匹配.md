#  Linux Command find 查找匹配



-----
## 1. 格式

```bash
find pathname -options [-print -exec -ok ...]
```

## 2. 参数

```bash
pathname: find命令所查找的目录路径。例如用.来表示当前目录，用/来表示系统根目录。
-print： find命令将匹配的文件输出到标准输出。
-exec： find命令对匹配的文件执行该参数所给出的shell命令。相应命令的形式为'command' { } ;，注意{ }和；之间的空格。
-ok： 和-exec的作用相同，只不过以一种更为安全的模式来执行该参数所给出的shell命令，在执行每一个命令之前，都会给出提示，让用户来确定是否执行。

-name
按照文件名查找文件。
-perm
按照文件权限来查找文件。
-prune
使用这一选项可以使find命令不在当前指定的目录中查找，如果同时使用-depth选项，那么-prune将被find命令忽略。
-user
按照文件属主来查找文件。
-group
按照文件所属的组来查找文件。
-mtime -n +n
按照文件的更改时间来查找文件， - n表示文件更改时间距现在n天以内，+ n表示文件更改时间距现在n天以前。find命令还有-atime和-ctime 选项，但它们都和-m time选项。
-nogroup
查找无有效所属组的文件，即该文件所属的组在/etc/groups中不存在。
-nouser
查找无有效属主的文件，即该文件的属主在/etc/passwd中不存在。
-newer file1 ! file2
查找更改时间比文件file1新但比文件file2旧的文件。
-type
查找某一类型的文件，诸如：
b - 块设备文件。
d - 目录。
c - 字符设备文件。
p - 管道文件。
l - 符号链接文件。
f - 普通文件。
-size n：[c] 查找文件长度为n块的文件，带有c时表示文件长度以字节计。-depth：在查找文件时，首先查找当前目录中的文件，然后再在其子目录中查找。
-fstype：查找位于某一类型文件系统中的文件，这些文件系统类型通常可以在配置文件/etc/fstab中找到，该配置文件中包含了本系统中有关文件系统的信息。
-mount：在查找文件时不跨越文件系统mount点。
-follow：如果find命令遇到符号链接文件，就跟踪至链接所指向的文件。
-cpio：对匹配的文件使用cpio命令，将这些文件备份到磁带设备中。
另外,下面三个的区别:
-amin n
查找系统中最后N分钟访问的文件
-atime n
查找系统中最后n*24小时访问的文件
-cmin n
查找系统中最后N分钟被改变文件状态的文件
-ctime n
查找系统中最后n*24小时被改变文件状态的文件
-mmin n
查找系统中最后N分钟被改变文件数据的文件
-mtime n
查找系统中最后n*24小时被改变文件数据的文件
```

## 4. 举例

find命令将删除当目录中访问时间在7日以来、含有数字后缀的admin.log文件。
该命令只检查三位数字，所以相应文件的后缀不要超过999。先建几个admin.log*的文件 ，才能使用下面这个命令

```bash
$ find . -name "admin.log[0-9][0-9][0-9]" -atime -7 -ok
rm { } ;
< rm ... ./admin.log001 > ? n
< rm ... ./admin.log002 > ? n
< rm ... ./admin.log042 > ? n
< rm ... ./admin.log942 > ? n
```
查找/var/logs目录中更改时间在7日以前的普通文件，并在删除之前询问它们；

```c
$ find /var/logs -type f -mtime +7 -ok rm { } ;
```

为了查找当前文件系统中的所有目录并排序；

```bash
$ find . -type d | sort
```

为了查找系统中所有的rmt磁带设备；

```bash
$ find /dev/rmt -print
```

###  4.1 exec


rm命令删除文件之前，先用ls命令看一下，确认它们是所要删除的文件。

```bash
$ find . -type f -exec ls -l { } ;
-rw-r--r-- 1 root root 34928 2003-02-25 ./conf/httpd.conf
-rw-r--r-- 1 root root 12959 2003-02-25 ./conf/magic
-rw-r--r-- 1 root root 180 2003-02-25 ./conf.d/README
```
在/logs目录中查找更改时间在5日以前的文件并删除它们：

```bash
$ find logs -type f -mtime +5 -exec rm { } ;
```


find命令在当前目录中查找所有文件名以.LOG结尾、更改时间在5日以上的文件，并删除它们，只不过在删除之前先给出提示。

```bash
$ find . -name "*.conf" -mtime +5 -ok rm { } ;
< rm ... ./conf/httpd.conf > ? n
按y键删除文件，按n键不删除。
```


在下面的例子中我们使用grep命令。find命令首先匹配所有文件名为“ passwd*”的文件，例如passwd、passwd.old、passwd.bak，然后执行grep命令看看在这些文件中是否存在一个sam用户。

```bash
$ find /etc -name "passwd*" -exec grep "sam" { } ;
sam:x:501:501::/usr/sam:/bin/bash
```

```bash
$ find $HOME -print
$ find ~ -print
```

让当前目录中文件属主具有读、写权限，并且文件所属组的用户和其他用户具有读权限的文件；

```bash
$ find . -type f -perm 644 -exec ls -l { } ;
```

为了查找系统中所有文件长度为0的普通文件，并列出它们的完整路径；

```bash
$ find / -type f -size 0 -exec ls -l { } ;
```




为了查找系统中所有属于root组的文件；

```bash
$find . -group root -exec ls -l { } ;
-rw-r--r-- 1 root root 595 10月 31 01:09 ./fie1
```



###  4.2 xargs

```bash
xargs - build and execute command lines from standard input
```

在使用find命令的-exec选项处理匹配到的文件时， find命令将所有匹配到的文件一起传递给exec执行。但有些系统对能够传递给exec的命令长度有限制，这样在find命令运行几分钟之后，就会出现溢出错误。错误信息通常是“参数列太长”或“参数列溢出”。这就是xargs命令的用处所在，特别是与find命令一起使用。
find命令把匹配到的文件传递给xargs命令，而xargs命令每次只获取一部分文件而不是全部，不像-exec选项那样。这样它可以先处理最先获取的一部分文件，然后是下一批，并如此继续下去。
在有些系统中，使用-exec选项会为处理每一个匹配到的文件而发起一个相应的进程，并非将匹配到的文件全部作为参数一次执行；这样在有些情况下就会出现进程过多，系统性能下降的问题，因而效率不高；
而使用xargs命令则只有一个进程。另外，在使用xargs命令时，究竟是一次获取所有的参数，还是分批取得参数，以及每一次获取参数的数目都会根据该命令的选项及系统内核中相应的可调参数来确定。
来看看xargs命令是如何同find命令一起使用的，并给出一些例子。
下面的例子查找系统中的每一个普通文件，然后使用xargs命令来测试它们分别属于哪类文件

```bash
$ find . -type f -print | xargs file
./.kde/Autostart/Autorun.desktop: UTF-8 Unicode English text
./.kde/Autostart/.directory: ISO-8859 text
......
```

在整个系统中查找内存信息转储文件(core dump) ，然后把结果保存到/tmp/core.log 文件中：

```bash
$ find / -name "core" -print | xargs echo "" >/tmp/core.log
```

上面这个执行太慢，我改成在当前目录下查找

```bash
$ find . -name "file*" -print | xargs echo "" > /temp/core.log
$ cat /temp/core.log
./file6
```


在当前目录下查找所有用户具有读、写和执行权限的文件，并收回相应的写权限：

```bash
$ ls -l
drwxrwxrwx 2 sam adm 4096 10月 30 20:14 file6
-rwxrwxrwx 2 sam adm 0 10月 31 01:01 http3.conf
-rwxrwxrwx 2 sam adm 0 10月 31 01:01 httpd.conf
$ find . -perm -7 -print | xargs chmod o-w
$ ls -l
drwxrwxr-x 2 sam adm 4096 10月 30 20:14 file6
-rwxrwxr-x 2 sam adm 0 10月 31 01:01 http3.conf
-rwxrwxr-x 2 sam adm 0 10月 31 01:01 httpd.conf
```

用grep命令在所有的普通文件中搜索hostname这个词：

```bash
$  find . -type f -print | xargs grep "hostname"
./httpd1.conf:# different IP addresses or hostnames and have them handled by the
./httpd1.conf:# VirtualHost: If you want to maintain multiple domains/hostnames
on your
```

用grep命令在当前目录下的所有普通文件中搜索hostnames这个词：

```bash
$ find . -name * -type f -print | xargs grep "hostnames"
./httpd1.conf:# different IP addresses or hostnames and have them handled by the
./httpd1.conf:# VirtualHost: If you want to maintain multiple domains/hostnames
on your
```


分别使用xargs和exec实现这样的需求，把当前目录下所有后缀名为.txt的文件的权限修改为777

```bash
$ find ./ -type f -name "*.txt" |xargs chmod 777
$ find ./ -type f -name "*.txt" -exec chmod 777 {} ;
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/48d0ec97e4b948238ac68617ac4ee59f.gif#pic_center)

