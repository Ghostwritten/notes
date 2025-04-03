#  Bash EOF 技巧
![](https://i-blog.csdnimg.cn/blog_migrate/57a7f04958e2f2eb54ed5c084e2f6375.png)




EOF适用场景：
- 命令行多行输出
- 脚本包装
- 类型配置文件

## 1. 命令行输出

```bash
$ cat << EOF
> Hello
> EOF
Hello
```

## 2. 写入文本
```bash
cat << EOF >1.txt
111
222
333
EOF
```
复制终端是这样的。

```bash
cat << EOF >1.txt
> 111
> 222
> 333
> EOF
```
回车后

```bash
$ cat 1.txt
111
222
333
```

## 3. 追加文本

```bash
cat << EOF >> 1.txt
444
555
666
EOF
```
查看1.txt内容

```bash
$ cat 1.txt
111
222
333
444
555
666
```
## 4. 覆盖文本

```bash
cat << EOF >1.txt
aaa
bbb
ccc
EOF
```
查看

```bash
$ cat 1.txt
aaa
bbb
ccc
```

## 5. 自定义 EOF

```bash
cat << a > 1.txt
111
222
333
a
```
输出：

```bash
$ cat 1.txt
111
222
333
```

## 6. 另一种格式
- `cat > filename <<EOF`
- `cat << EOF > filename`

```bash
cat > 1.txt <<EOF
123
456
789
EOF
```
查看

```bash
$ cat 1.txt
123
456
789
```
追加内容

```bash
cat >> 1.txt <<EOF
abc
def
ghi
EOF
```
查看内容

```bash
$ cat 1.txt
123
456
789
abc
def
ghi
```

## 7. 示例
### 7.1 配置文件
或者`cat << EOF > /usr/local/mysql/my.cnf`
```bash
cat > /usr/local/mysql/my.cnf << EOF         
[client]
port = 3306
socket = /usr/local/mysql/var/mysql.sock

[mysqld]
port = 3306
socket = /usr/local/mysql/var/mysql.sock

basedir = /usr/local/mysql/
datadir = /data/mysql/data
pid-file = /data/mysql/data/mysql.pid
user = mysql
bind-address = 0.0.0.0
server-id = 1
sync_binlog=1
log_bin = mysql-bin

[myisamchk]
key_buffer_size = 8M
sort_buffer_size = 8M
read_buffer = 4M
write_buffer = 4M

sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES 
port = 3306
EOF
```

### 7.2 新建分区并挂载

```bash
$ cat auto_add_disk.sh         
#!/bin/bash
fdisk  /dev/sdb  <<EOF
n
p
1
 
 
wq
EOF
 
/sbin/mkfs .ext4  /dev/sdb1  &&   /bin/mkdir  -p  /data  &&  /bin/mount  /dev/sdb1  /data
echo  'LABEL=data_disk /data ext4 defaults 0 2'  >>  /etc/fstab
```

### 7.3 设置变量

```bash
$ sql=$(cat <<EOF
SELECT foo, bar FROM db
WHERE foo='baz'
EOF
)

$ echo -e "$sql"
```

###  7.4 输出脚本

```bash
cat <<EOF > print.sh
#!/bin/bash
echo \$PWD
echo $PWD
EOF
```
查看内容

```bash
$ cat print.sh
#!/bin/bash
echo $PWD
echo /home/user
```

### 7.5 匹配输出

```bash
$ cat <<EOF | grep 'b' | tee b.txt
> foo
> bar
> baz
> EOF
bar
baz

$ cat b.txt
bar
baz
```

### 7.6 json 文本

```bash
cat >> /etc/docker/daemon.json < EOF
{
   "exec-opts": ["native.cgroupdriver=systemd"],
   "log-driver": "json-file",
   "log-opts": {
   "max-size":  "100m"
    },
   "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ]
 }
 EOF
```
查看

```bash
$ cat /etc/docker/daemon.json
{
   "exec-opts": ["native.cgroupdriver=systemd"],
   "log-driver": "json-file",
   "log-opts": {
   "max-size":  "100m"
    },
   "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ]
 }
```
参考：
- [How does "cat << EOF" work in bash?](https://stackoverflow.com/questions/2500436/how-does-cat-eof-work-in-bash)
- [What is Cat EOF in Bash Script?](https://linuxhint.com/what-is-cat-eof-bash-script/)
