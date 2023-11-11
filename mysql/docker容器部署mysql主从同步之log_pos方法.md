

## 1. 环境

```bash
centos
192.168.1.10  (master)
192.168.1.11 (slave)
```

## 2. 主（ master)

划分存储卷
```bash
lvcreate -L 200G -name lvdata vgdata
mkfs.xfs /dev/mapper/vgdata-lvmysql
mkdir /mysql
vim /etc/fstab
/dev/mapper/vgdata-lvmysql   /mysql xfs   defaults 0 0
mount -a
```
初始化mysql主节点配置
```bash
$ docker pull local.harbor.io/library/mysql:5.7.22
$ mkdir /mysql/data /mysql/log /mysql/config  #创建日志、配置文件、数据持久存储目录
$ docker run -tid --restart=always --net=host -e MYSQL_ROOT_PASSWORD=123456 --name mysql-test mysql:5.7.22  #启动测试容器将默认文件拷贝
$ docker cp mysql-test:/var/log/mysql /mysql/log
$ docker cp mysql-test:/var/lib/mysql /mysql/data
$ docker cp mysql-test:/etc/mysql  /mysql/config
$ vim /mysql/config/mysql/my.cnf #设置配置文件参数
[client]
port            = 3306
socket          = /var/lib/mysql/mysql.sock
 
[mysqld]
datadir=/var/lib/mysql
port            = 3306
socket          = /var/lib/mysql/mysql.sock
symbolic-links=0
skip-external-locking
key_buffer_size = 16M
max_allowed_packet = 1M
table_open_cache = 64
sort_buffer_size = 512K
net_buffer_length = 8K
read_buffer_size = 256K
read_rnd_buffer_size = 512K
myisam_sort_buffer_size = 8M
innodb_buffer_pool_size = 2G
log-bin=mysql-bin
binlog_format=mixed
server-id       = 1
 
[mysqld_safe]
log-error=/var/log/mysql/zbx3dbmaster.log
pid-file=/var/lib/mysql/zbx3dbmaster.pid
 
[mysqldump]
quick
max_allowed_packet = 16M
 
[mysql]
no-auto-rehash
 
[myisamchk]
key_buffer_size = 20M
sort_buffer_size = 20M
read_buffer = 2M
write_buffer = 2M
 
[mysqlhotcopy]
interactive-timeout

```
启动主节点容器
```bash
$ docker run -tid --restart=always --net=host -e MYSQL_ROOT_PASSWORD=123456 -v /mysql/data/mysql:/var/lib/mysql -v /mysql/log/mysql:/var/log/mysql -v /mysql/config/mysql:/etc/mysql --name mysql-master mysql:5.7.22
```
创建同步用户
```bash
docker exec -ti mysql-master mysql -uroot -p123456 -e "grant replication slave on *.* to 'db_sync'@'%' identified by 'abcdefg';"
```
查看主状态

```bash
docker exec -ti mysql-master mysql -uroot -p123456 -e "show master status\G;"
*************************** 1. row ***************************
             File: mysql-bin.000001
         Position: 154
     Binlog_Do_DB: db
 Binlog_Ignore_DB: 
Executed_Gtid_Set: 
1 row in set (0.00 sec)
```

## 3. 从 （slave）

划分存储卷
```bash
lvcreate -L 200G -name lvdata vgdata
mkfs.xfs /dev/mapper/vgdata-lvmysql
mkdir /mysql
vim /etc/fstab
/dev/mapper/vgdata-lvmysql   /mysql xfs   defaults 0 0
mount -a
```
初始化从节点配置
```bash
```bash
$ docker pull local.harbor.io/library/mysql:5.7.22
$ mkdir /mysql/data /mysql/log /mysql/config  #创建日志、配置文件、数据持久存储目录
$ docker run -tid --restart=always --net=host -e MYSQL_ROOT_PASSWORD=123456 --name mysql-test mysql:5.7.22  #启动测试容器将默认文件拷贝
$ docker cp mysql-test:/var/log/mysql /mysql/log
$ docker cp mysql-test:/var/lib/mysql /mysql/data
$ docker cp mysql-test:/etc/mysql  /mysql/config
$ vim /mysql/config/mysql/my.cnf #设置配置文件参数
[client]
port            = 3306
socket          = /var/lib/mysql/mysql.sock
 
[mysqld]
datadir=/var/lib/mysql
port            = 3306
socket          = /var/lib/mysql/mysql.sock
symbolic-links=0
skip-external-locking
key_buffer_size = 16M
max_allowed_packet = 1M
table_open_cache = 64
sort_buffer_size = 512K
net_buffer_length = 8K
read_buffer_size = 256K
read_rnd_buffer_size = 512K
myisam_sort_buffer_size = 8M
innodb_buffer_pool_size = 2G
log-bin=mysql-bin
binlog_format=mixed
server-id       = 2
 
[mysqld_safe]
log-error=/var/log/mysql/zbx3dbslave.log
pid-file=/var/lib/mysql/zbx3dbslave.pid
 
[mysqldump]
quick
max_allowed_packet = 16M
 
[mysql]
no-auto-rehash
 
[myisamchk]
key_buffer_size = 20M
sort_buffer_size = 20M
read_buffer = 2M
write_buffer = 2M
 
[mysqlhotcopy]
interactive-timeout

```
```bash
$ docker run -tid --restart=always -p3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -v /mysql/data/mysql:/var/lib/mysql -v /mysql/log/mysql:/var/log/mysql -v /mysql/config/mysql:/etc/mysql --name mysql-slave mysql:5.7.22
```
与主同步
```bash
$ docker exec -ti mysql-slave mysql -uroot -p123456 -e "stop slave;change master to master_host='192.168.1.10',master_user='db_sync',master_password='abcdefg',master_log_file='mysql-bin.000001',master_log_pos=154;start slave;"
$ docker exec -ti mysql-slave mysql -uroot -p123456 -e "show slave status\G;"
```
如果报错

```bash
Last_IO_Error: Fatal error: The slave I/O thread stops because master and slave have equal MySQL server UUIDs; these UUIDs must be different for replication to work
```
原因：uuid值相等
解决方法：修改不一样即可
修改 `/mysql/data/mysql/auto.cnf` 

```bash
[auto]
server-uuid=a97c5316-4819-11eb-a312-000c29613c38
```


