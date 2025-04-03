#  Linux Command rsync 远程同步



----
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7633dfea202fad342a24a0d2249a0d53.gif#pic_center)
## 1. 简介
rsync命令是一个远程数据同步工具，可通过`LAN/WAN`快速同步多台主机间的文件。rsync使用所谓的“rsync算法”来使本地和远程两个主机之间的文件达到同步，这个算法只传送两个文件的不同部分，而不是每次都整份传送，因此速度相当快。
## 2. 语法
语法

```bash
rsync [OPTION]... SRC DEST
rsync [OPTION]... SRC [USER@]host:DEST
rsync [OPTION]... [USER@]HOST:SRC DEST
rsync [OPTION]... [USER@]HOST::SRC DEST
rsync [OPTION]... SRC [USER@]HOST::DEST
rsync [OPTION]... rsync://[USER@]HOST[:PORT]/SRC [DEST]
```
对应于以上六种命令格式，rsync有六种不同的工作模式：

 - 拷贝本地文件。当SRC和DES路径信息都不包含有单个冒号":"分隔符时就启动这种工作模式。如：`rsync -a /data /backup`
 - 使用一个远程shell程序(如rsh、ssh)来实现将本地机器的内容拷贝到远程机器。当DST路径地址包含单个冒号":"分隔符时启动该模式。如：`rsync -avz *.c foo:src`
 - 使用一个远程shell程序(如rsh、ssh)来实现将远程机器的内容拷贝到本地机器。当SRC地址路径包含单个冒号":"分隔符时启动该模式。如：`rsync -avz foo:src/bar /data`
 - 从远程rsync服务器中拷贝文件到本地机。当SRC路径信息包含"::"分隔符时启动该模式。如：`rsync -av root@192.168.78.192::www /databack`
 - 从本地机器拷贝文件到远程rsync服务器中。当DST路径信息包含"::"分隔符时启动该模式。如：`rsync -av /databack root@192.168.78.192::www`
 - 列远程机的文件列表。这类似于rsync传输，不过只要在命令中省略掉本地机信息即可。如：`rsync -v rsync://192.168.78.192/www`

## 3. 参数

```bash
-v, --verbose 详细模式输出。
-q, --quiet 精简输出模式。
-c, --checksum 打开校验开关，强制对文件传输进行校验。
-a, --archive 归档模式，表示以递归方式传输文件，并保持所有文件属性，等于-rlptgoD。
-r, --recursive 对子目录以递归模式处理。
-R, --relative 使用相对路径信息。

-b, --backup 创建备份，也就是对于目的已经存在有同样的文件名时，将老的文件重新命名为~filename。可以使用--suffix选项来指定不同的备份文件前缀。
--backup-dir 将备份文件(如~filename)存放在在目录下。
-suffix=SUFFIX 定义备份文件前缀。

-u, --update 仅仅进行更新，也就是跳过所有已经存在于DST，并且文件时间晚于要备份的文件，不覆盖更新的文件。
-l, --links 保留软链结。
-L, --copy-links 想对待常规文件一样处理软链结。
--copy-unsafe-links 仅仅拷贝指向SRC路径目录树以外的链结。
--safe-links 忽略指向SRC路径目录树以外的链结。
-H, --hard-links 保留硬链结。
-p, --perms 保持文件权限。
-o, --owner 保持文件属主信息。
-g, --group 保持文件属组信息。
-D, --devices 保持设备文件信息。
-t, --times 保持文件时间信息。
-S, --sparse 对稀疏文件进行特殊处理以节省DST的空间。
-n, --dry-run现实哪些文件将被传输。
-w, --whole-file 拷贝文件，不进行增量检测。
-x, --one-file-system 不要跨越文件系统边界。
-B, --block-size=SIZE 检验算法使用的块尺寸，默认是700字节。
-e, --rsh=command 指定使用rsh、ssh方式进行数据同步。
--rsync-path=PATH 指定远程服务器上的rsync命令所在路径信息。
-C, --cvs-exclude 使用和CVS一样的方法自动忽略文件，用来排除那些不希望传输的文件。
--existing 仅仅更新那些已经存在于DST的文件，而不备份那些新创建的文件。
--delete 删除那些DST中SRC没有的文件。
--delete-excluded 同样删除接收端那些被该选项指定排除的文件。
--delete-after 传输结束以后再删除。
--ignore-errors 及时出现IO错误也进行删除。
--max-delete=NUM 最多删除NUM个文件。
--partial 保留那些因故没有完全传输的文件，以是加快随后的再次传输。
--force 强制删除目录，即使不为空。
--numeric-ids 不将数字的用户和组id匹配为用户名和组名。
--timeout=time ip超时时间，单位为秒。
-I, --ignore-times 不跳过那些有同样的时间和长度的文件。
--size-only 当决定是否要备份文件时，仅仅察看文件大小而不考虑文件时间。
--modify-window=NUM 决定文件是否时间相同时使用的时间戳窗口，默认为0。
-T --temp-dir=DIR 在DIR中创建临时文件。
--compare-dest=DIR 同样比较DIR中的文件来决定是否需要备份。
-P 等同于 --partial。
--progress 显示备份过程。
-z, --compress 对备份的文件在传输时进行压缩处理。
--exclude=PATTERN 指定排除不需要传输的文件模式。
--include=PATTERN 指定不排除而需要传输的文件模式。
--exclude-from=FILE 排除FILE中指定模式的文件。
--include-from=FILE 不排除FILE指定模式匹配的文件。
--version 打印版本信息。
--address 绑定到特定的地址。
--config=FILE 指定其他的配置文件，不使用默认的rsyncd.conf文件。
--port=PORT 指定其他的rsync服务端口。
--blocking-io 对远程shell使用阻塞IO。
-stats 给出某些文件的传输状态。
--progress 在传输时现实传输过程。
--log-format=formAT 指定日志文件格式。
--password-file=FILE 从FILE中得到密码。
--bwlimit=KBPS 限制I/O带宽，KBytes per second。
-h, --help 显示帮助信息。
```
## 4. 举例

```bash
系统环境
服务端：192.168.211.15
客户端: 192.168.211.16
```

### 4.1. 详细执行输出
```bash
$ rsync -v root@192.168.211.15:/root/ansible-operator-v0.13.0.tar /data/
```
### 4.2. 假装执行输出
```bash
$ rsync -nv root@192.168.211.15:/root/ansible-operator-v0.13.0.tar /data/
 
```
### 4.3. 显示进度条输出

```bash
$ rsync --progress root@192.168.211.15:/root/ansible-operator-v0.13.0.tar /data/
```
**注意**：如果当前文件是存在的，拷贝的速度会加快
### 4.4. 递归、压缩、显示进度、保留文件属性传输

```bash
$ rsync --progress -azv root@192.168.211.15:/root/go /data/
$ ls /data
go
```
### 4.5. 同步中删除源中没有的文件
192.168.211.16

```bash
$  ls /data/shell
test.sh          #被删除的文件
```

192.168.211.15

```bash
$ ls /root/shell
compare1.sh  compare2.sh
```

192.168.211.16

```bash
$ rsync --delete --progress -azv root@192.168.211.15:/shell /data/
$ ls /data/shell
compare1.sh  compare2.sh
```
### 4.6. 无密码同步
服务端配置（192.168.211.15）

```powershell
$ vim /etc/rsyncd.conf
#global settings 
pid file = /var/run/rsyncd.pid
port = 873
lock file = /var/run/rsyncd.lock
log file = /var/log/rsync.log
gid = root
uid = root

module settings 
[data]
path = /root/data
use chroot = no
max connections = 15
read only = yes
write only = no
list = no
ignore errors = yes
timeout = 120
```
刷新服务
```powershell
$ /usr/bin/rsync --daemon
```

客户端

```powershell
$ rsync --progress -azv root@192.168.211.15::shell /data/shell #不拷贝源目录，在目的添加目录
$ rsync  --delete  --progress -azv root@192.168.211.15::shell /data/shell  #在目标中删除源中没有的文件
$ rsync --bwlimit=50 --progress -azv root@192.168.211.15::shell /data/shell  #限制流量同步
```
### 4.7. 有密码同步
服务端  

```powershell
$ vim /etc/rsyncd.conf
#This is the rsync daemon configuration 

#global settings 
pid file = /var/run/rsyncd.pid
port = 873
lock file = /var/run/rsyncd.lock
log file = /var/log/rsync.log
gid = root
uid = root

#module settings 
[monitor_data]
path = /root/monitor_data
use chroot = no
max connections = 15
read only = yes
write only = no
list = no
ignore errors = yes
timeout = 120
auth users = cloudmon
secrets file = /etc/rsyncd.passwd
```
刷新服务
```powershell
$ /usr/bin/rsync --daemon
```
配置用户密码
```powershell
$ echo "cloudmon:redhat123" > /etc/rsyncd.passwd 
$ chmod 600 /etc/rsyncd.passwd
$ mkdir -p /root/monitor_data
$ cp /root/prometheus /root/monitor_data  #传输的介质
```
客户端

```powershell
$ useradd cloudmon
$ echo "redhat123" > /home/cloudmon/rsyncd.passwd 
$ chmod 600 /home/cloudmon/rsyncd.passwd 
$ rsync -avz --progress  --password-file=/home/cloudmon/rsyncd.passwd  root@192.168.211.15::monitor_data /data
```

 

或者是

```powershell
$ export RSYNC_PASSWORD="redhat123"
$ rsync -avz --progress  --password-file=/home/cloudmon/rsyncd.passwd  root@192.168.211.15::monitor_data /data
```
### 4.8. 写入同步
服务端

```powershell
$ vim /etc/rsyncd.conf
#global settings 
pid file = /var/run/rsyncd.pid
port = 873
lock file = /var/run/rsyncd.lock
log file = /var/log/rsync.log
gid = root
uid = root

#module settings 
[write_data]
path = /root/write_data
use chroot = no
max connections = 15
read only = no
list = no
ignore errors = yes
timeout = 120
auth users = cloudmon
secrets file = /etc/rsyncd.passwd
```

```powershell
$ mkdir -p /root/write_data
$ touch /root/write_data/hello.txt #在目标(服务端)中删除源中没有的文件
```
客户端

```powershell
$ echo "123" > /root/write_file
$ export RSYNC_PASSWORD="redhat123"
$ rsync -avz --progress --delete /root/write_file cloudmon@192.168.211.15::write_data 
```

更多阅读：

 - [rsync 用法教程](https://www.ruanyifeng.com/blog/2020/08/rsync.html)
 - [rsync(1) - Linux man page](https://linux.die.net/man/1/rsync)




