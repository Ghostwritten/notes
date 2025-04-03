#  Linux Command dd 




-----
## 1. 简介
Linux dd 命令用于读取、转换并输出数据。

dd 可从标准输入或文件中读取数据，根据指定的格式来转换数据，再输出到文件、设备或标准输出。

> 注意：指定数字的地方若以下列字符结尾，则乘以相应的数字：b=512；c=1；k=1024；w=2

## 2. 参数

```bash
参数说明:

if=文件名：输入文件名，默认为标准输入。即指定源文件。
of=文件名：输出文件名，默认为标准输出。即指定目的文件。
ibs=bytes：一次读入bytes个字节，即指定一个块大小为bytes个字节。
obs=bytes：一次输出bytes个字节，即指定一个块大小为bytes个字节。
bs=bytes：同时设置读入/输出的块大小为bytes个字节。
cbs=bytes：一次转换bytes个字节，即指定转换缓冲区大小。
skip=blocks：从输入文件开头跳过blocks个块后再开始复制。
seek=blocks：从输出文件开头跳过blocks个块后再开始复制。
count=blocks：仅拷贝blocks个块，块大小等于ibs指定的字节数。
conv=<关键字>，关键字可以有以下11种：
conversion：用指定的参数转换文件。
ascii：转换ebcdic为ascii
ebcdic：转换ascii为ebcdic
ibm：转换ascii为alternate ebcdic
block：把每一行转换为长度为cbs，不足部分用空格填充
unblock：使每一行的长度都为cbs，不足部分用空格填充
lcase：把大写字符转换为小写字符
ucase：把小写字符转换为大写字符
swap：交换输入的每对字节
noerror：出错时不停止
notrunc：不截短输出文件
sync：将每个输入块填充到ibs个字节，不足部分用空（NUL）字符补齐。
--help：显示帮助信息
--version：显示版本信息
```

## 3. 举例

### 3.1 /dev/hdb 备份至 /dev/hdd

```bash
$ dd if=/dev/hdb of=/dev/hdd
```
### 3.2 备份文件恢复至指定盘

```bash
$ dd if=/root/image of=/dev/hdb
```
### 3.3 备份 /dev/hdb 并 gzip 压缩

```bash
dd if=/dev/hdb | gzip > /root/image.gz
```
### 3.4 压缩备份恢复至指定盘

```bash
gzip -dc /root/image.gz | dd of=/dev/hdb
```
### 3.5 备份与恢复MBR
备份磁盘开始的512个字节大小的MBR信息到指定文件：

```bash
$ dd if=/dev/hda of=/root/image count=1 bs=512
```

 - count=1指仅拷贝一个块；
 - bs=512指块大小为512个字节。

恢复：

```c
#dd if=/root/image of=/dev/had
```

将备份的MBR信息写到磁盘开始部分
### 3.6 备份软盘

```bash
$ dd if=/dev/fd0 of=disk.img count=1 bs=1440k (即块大小为1.44M)
```

### 3.7 拷贝内存内容到硬盘

```bash
$ dd if=/dev/mem of=/root/mem.bin bs=1024 (指定块大小为1k)  
```

### 3.8 拷贝光盘内容到指定文件夹，并保存cd.iso

```bash
#dd if=/dev/cdrom(hdc) of=/root/cd.iso
```

### 3.9 增加swap分区文件大小

```bash
第一步：创建一个大小为256M的文件：
$ dd if=/dev/zero of=/swapfile bs=1024 count=262144
第二步：把这个文件变成swap文件：
$ mkswap /swapfile
第三步：启用这个swap文件：
$ swapon /swapfile
第四步：编辑/etc/fstab文件，使在每次开机时自动加载swap文件：
/swapfile    swap    swap    default   0 0
```

### 3.10 销毁磁盘数据

```bash
$ dd if=/dev/urandom of=/dev/hda1
```

> 注意：利用随机的数据填充硬盘，在某些必要的场合可以用来销毁数据。

### 3.11 测试硬盘的读写速度

```bash
$ dd if=/dev/zero bs=1024 count=1000000 of=/root/1Gb.file
$ dd if=/root/1Gb.file bs=64k | dd of=/dev/null
```

通过以上两个命令输出的命令执行时间，可以计算出硬盘的读、写速度。
确定硬盘的最佳块大小：

```bash
$ dd if=/dev/zero bs=1024 count=1000000 of=/root/1Gb.file
$ dd if=/dev/zero bs=2048 count=500000 of=/root/1Gb.file
$ dd if=/dev/zero bs=4096 count=250000 of=/root/1Gb.file
$ dd if=/dev/zero bs=8192 count=125000 of=/root/1Gb.file
```

通过比较以上命令输出中所显示的命令执行时间，即可确定系统最佳的块大小。
### 3.12 修复硬盘

```bash
$ dd if=/dev/sda of=/dev/sda 或dd if=/dev/hda of=/dev/hda
```

当硬盘较长时间(一年以上)放置不使用后，磁盘上会产生`magnetic flux point`，当磁头读到这些区域时会遇到困难，并可能导致I/O错误。当这种情况影响到硬盘的第一个扇区时，可能导致硬盘报废。上边的命令有可能使这些数 据起死回生。并且这个过程是安全、高效的。
### 3.13 利用netcat远程备份


```bash
$ dd if=/dev/hda bs=16065b | netcat < targethost-IP > 1234
在源主机上执行此命令备份/dev/hda
$ netcat -l -p 1234 | dd of=/dev/hdc bs=16065b
在目的主机上执行此命令来接收数据并写入/dev/hdc
$ netcat -l -p 1234 | bzip2 > partition.img
$ netcat -l -p 1234 | gzip > partition.img
以上两条指令是目的主机指令的变化分别采用bzip2、gzip对数据进行压缩，并将备份文件保存在当前目录。
将一个很大的视频文件中的第i个字节的值改成0×41（也就是大写字母A的ASCII值）
echo A | dd of=bigfile seek=$i bs=1 count=1 conv=notrunc
```

## 4. /dev/null和/dev/zero

 - /dev/null，外号叫无底洞，你可以向它输出任何数据
 - /dev/zero，是一个输入设备，你可你用它来初始化文件。该设备无穷尽地提供0，可以使用任何你需要的数目——设备提供的要多的多。他可以用于向设备或文件写入字符串0。

```bash
$ if=/dev/zero of=./test.txt bs=1k count=1
$ ls –l
total 4
-rw-r–r–    1 oracle   dba          1024 Jul 15 16:56 test.txt
$ find / -name access_log  2>/dev/null
```

```bash
禁止标准输出
$ cat $filename >/dev/null

禁止标准错误
$ rm $badname 2>/dev/null

$ cat $filename &>/dev/null

$ cat /dev/null > /var/log/messages
$ cat /dev/null > /var/log/wtmp
$: > /var/log/messages    有同样的效果， 但不会产生新的进程.（因为:是内建的）
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a21a20f1176c48162f69e3fe53b29220.gif#pic_center)


