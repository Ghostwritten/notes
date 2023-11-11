

## 1. xtrabackup介绍

xtrabackup是Percona公司CTO Vadim参与开发的一款基于InnoDB的在线热备工具，具有开源，免费，支持在线热备，备份恢复速度快，占用磁盘空间小等特点，并且支持不同情况下的多种备份形式。
xtrabackup的官方下载地址为http://www.percona.com/software/percona-xtrabackup。

xtrabackup包含两个主要的工具，即xtrabackup和innobackupex，二者区别如下：

 - （1）xtrabackup只能备份innodb和xtradb两种引擎的表，而不能备份myisam引擎的表；

- （2）innobackupex是一个封装了xtrabackup的Perl脚本，支持同时备份innodb和myisam，但在对myisam备份时需要加一个全局的读锁。还有就是myisam不支持增量备份。

### 1.1 备份过程
innobackupex备份过程如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200308201232229.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
在图1中，备份开始时首先会开启一个后台检测进程，实时检测mysql redo的变化，一旦发现redo中有新的日志写入，立刻将日志记入后台日志文件xtrabackup_log中。之后复制innodb的数据文件和系统表空间文件ibdata1，待复制结束后，执行flush tables with read lock操作，复制.frm，MYI，MYD，等文件（执行flush tableswith read lock的目的是为了防止数据表发生DDL操作，并且在这一时刻获得binlog的位置）最后会发出unlock tables，把表设置为可读可写状态，最终停止xtrabackup_log。
### 1.2 全备恢复
这一阶段会启动xtrabackup内嵌的innodb实例，回放xtrabackup日志xtrabackup_log，将提交的事务信息变更应用到innodb数据/表空间，同时回滚未提交的事务(这一过程类似innodb的实例恢复）。恢复过程如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200308201625902.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
### 1.3 增量备份

innobackupex增量备份过程中的"增量"处理，其实主要是相对innodb而言，对myisam和其他存储引擎而言，它仍然是全拷贝(全备份)

"增量"备份的过程主要是通过拷贝innodb中有变更的"页"（这些变更的数据页指的是"页"的LSN大于xtrabackup_checkpoints中给定的LSN）。增量备份是基于全备的，第一次增备的数据必须要基于上一次的全备，之后的每次增备都是基于上一次的增备，最终达到一致性的增备。增量备份的过程如下，和全备的过程很类似，区别仅在第2步。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200308201733555.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
### 1.4 增量备份恢复

和全备恢复类似，也需要两步，一是数据文件的恢复，如图4，这里的数据来源由3部分组成：全备份，增量备份和xtrabackup log。二是对未提交事务的回滚，如图5所示：
![图4 innobackupex 增量备份恢复过程1](https://img-blog.csdnimg.cn/20200308201819665.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![图5 innobackupex增量备份恢复过程2](https://img-blog.csdnimg.cn/20200308201913726.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

## 2. 参数详解
命令参数说明:

```c
--defaults-file=/etc/my.cnf    备份数据库的配置文件my.cnf的路径
--user=root       备份操作用户名，一般都是root用户,但可以改 
--apply-log    应用备份时产生的日志，为整体拷贝恢复做准备
--copy-back    把完整备份文件拷贝到目标目录,由--defaults-file指定的my.cnf决定
--use-memory=4G    为了加快恢复速度,设置可用内存参数,可以不用
--databases    指定还原的数据库和某个库的表
--password=$pwdr    备份操作用户名的密码
--host=$hosts    主机ip，本地可以不加,ssh传输,要开放端口
--parallel=4        并行个数，根据主机配置选择合适的，默认是1个，多个可以加快备份速度。
--throttle=400    运行io限制数,一般来说并行能增加速度,但是IO也高,限制能减少影响
--stream=tar    压缩类型，这里选择tar格式，好像也只能是tar
$backupdir      备份存放的目录
2>$backupdir/backtip.log    备份日志，将备份过程中的输出信息重定向到log
|gzip >$backupdir/$backname.tar.gz    备份文件后用管道再压缩,最后成为一个压缩文件,减少占用空间
--databases    指定备份的数据库和某个库的表,例如这样:--databases="db db.table",不过要注意的是,他依然会把mysql库备份一遍,因为有很多公共信息还是依赖到他.
--no-lock       可以在备份非innodb表和frm文件时不锁表，但是master和slave的pos信息就无法记录，因为write_binlog_info和write_slave_info函数只在mysql_lockall函数中调用，而mysql_lockal调用了flush tables with read lock （不适合生产环境）
--use-memory preparing    进程可以通过分配更多的内存来提高速度，这取决于系统的可用内存，默认值是100MB。总的来说分配的内存越多越好。
--slave-info    备份目录下会多生成一个xtrabackup_slave_info 文件, 这里会保存主日志文件以及偏移, 文件内容类似于:CHANGE MASTER TO MASTER_LOG_FILE='', MASTER_LOG_POS=0
--slave-info    会将master的binlog文件名和偏移量位置保存到xtrabackup_slave_info文件中
--safe-slave-backup      会暂停slave的SQL线程直到没有打开的临时表的时候开始备份。备份结束后SQL线程会自动启动，这样操作的目的主要是确保一致性的复制状态
--incremental-basedir            指向全备目录，
--incremental                         指向增量备份的目录
```

## 5.innobackupex使用示例
下载地址：
[https://www.percona.com/downloads/Percona-XtraBackup-LATEST/](https://www.percona.com/downloads/Percona-XtraBackup-LATEST/)
### 5.1 安装使用xtrabackup
安装比较简单，我们使用二进制编译好的就行了，这种工具无需源码编译，因为没有什么功能需要俺们定制。

```bash
$ wget http://www.percona.com/redir/downloads/XtraBackup/LATEST/binary/Linux/x86_64/percona-xtrabackup-2.1.8-733-Linux-x86_64.tar.gz 
$ tar xf percona-xtrabackup-2.1.8-733-Linux-x86_64.tar.gz -C /usr/local/
$ mv /usr/local/percona-xtrabackup-2.1.8-Linux-x86_64/ /usr/local/xtrabackup
$ echo "export PATH=\$PATH:/usr/local/xtrabackup/bin" >> /etc/profile
$ source /etc/profile
```
### 5.2 全量备份

1.创建备份用户：

```bash
mysql> create user 'backup'@'%' identified by 'yayun';
Query OK, 0 rows affected (0.01 sec)
mysql> grant reload,lock tables,replication client,create tablespace,super on *.* to 'backup'@'%';
Query OK, 0 rows affected (0.00 sec)
```
备份数据存放在/data/backup/下面，innobackupex会自动创建一个文件夹，是当前系统的时间戳
2.测试数据就是yayun库中的t1表

```bash
mysql> select * from yayun.t1;
+------+-------+
| id   | name  |
+------+-------+
|    1 | yayun |
|    2 | atlas |
+------+-------+
2 rows in set (0.00 sec)
```


```bash
$ innobackupex --user=backup --password=yayun --socket=/tmp/mysqld.sock --defaults-file=/etc/my.cnf /data/backup/
[root@MySQL-01 backup]# cd 2014-04-07_23-05-04/
[root@MySQL-01 2014-04-07_23-05-04]# ll
total 845888
-rw-r--r-- 1 root root       261 Apr  7 23:05 backup-my.cnf
drwx------ 2 root root      4096 Apr  7 23:06 employees
drwx------ 2 root root      4096 Apr  7 23:06 host
-rw-r----- 1 root root 866123776 Apr  7 23:05 ibdata1
drwx------ 2 root root      4096 Apr  7 23:06 menagerie
drwxr-xr-x 2 root root      4096 Apr  7 23:06 mysql
drwxr-xr-x 2 root root      4096 Apr  7 23:06 performance_schema
drwx------ 2 root root      4096 Apr  7 23:06 sakila
drwx------ 2 root root      4096 Apr  7 23:06 test
drwx------ 2 root root      4096 Apr  7 23:06 world_innodb
drwxr-xr-x 2 root root      4096 Apr  7 23:06 world_myisam
-rw-r--r-- 1 root root        13 Apr  7 23:06 xtrabackup_binary
-rw-r--r-- 1 root root        24 Apr  7 23:06 xtrabackup_binlog_info
-rw-r----- 1 root root        95 Apr  7 23:06 xtrabackup_checkpoints
-rw-r----- 1 root root      2560 Apr  7 23:06 xtrabackup_logfile
drwx------ 2 root root      4096 Apr  7 23:06 yayun
```
对应数据库的名字，比如yayun，还有一个以时间戳命名的目录

```c
[root@MySQL-01 2014-04-07_23-05-04]$ cat xtrabackup_checkpoints 
backup_type = full-backuped
from_lsn = 0
to_lsn = 5324782783
last_lsn = 5324782783
compact = 0
[root@MySQL-01 2014-04-07_23-05-04]# cat xtrabackup_binlog_info 
mysql-bin.000014        2983
```


3.删除数据库，然后恢复全备（线上不要这样搞）

```bash
mysql> drop database yayun;
Query OK, 1 row affected (0.04 sec)
```
4.恢复全备

恢复备份到mysql的数据文件目录，这一过程要先**关闭mysql数据库，重命名或者删除原数据文件目录都可以，再创建一个新的数据文件目录，将备份数据复制到新的数据文件目录下，赋权，修改权限，启动数据库**

```bash
[root@MySQL-01 ~]$ /etc/init.d/mysqld stop
Shutting down MySQL.....                                   [  OK  ]
[root@MySQL-01 ~]$ mv /data/mysql /data/mysql_bak
[root@MySQL-01 ~]$ mkdir /data/mysql
[root@MySQL-01 ~]$ 
[root@MySQL-01 ~]$ innobackupex --apply-log /data/backup/2014-04-07_23-05-04/ 
```

以上对应的目录就是innobackupex全备份自己创建的目录。

```bash
[root@MySQL-01 ~]$ innobackupex --defaults-file=/etc/my.cnf --copy-back --rsync /data/backup/2014-04-07_23-05-04/
```
可以看见innobackupex: completed OK!说明已经成功恢复，修改数据目录权限，启动mysql，效验数据是否正常，查看yayun库下面的t1表中的数据。

```bash
[root@MySQL-01 ~]$ chown -R mysql.mysql /data/mysql
[root@MySQL-01 ~]$ /etc/init.d/mysqld start
mysql> use yayun
mysql> select * from t1;
+------+-------+
| id   | name  |
+------+-------+
|    1 | yayun |
|    2 | atlas |
+------+-------+
2 rows in set (0.00 sec)
```
发现数据已经成功恢复。
### 5.3 增量备份

在进行增量备份时，**首先要进行一次全量备份，第一次增量备份是基于全备的，之后的增量备份是基于上一次的增量备份，以此类推。**

全备份放在`/data/backup/full`,增量备份放在`/data/backup/incremental`
```c
[root@MySQL-01 ~]$ tree /data/backup/
/data/backup/
├── full
└── incremental
先来一次全备份
[root@MySQL-01 ~]$ innobackupex --user=backup --password=yayun --socket=/tmp/mysqld.sock --defaults-file=/etc/my.cnf /data/backup/full/
```
为了测试效果，我们在t1表中插入数据

```c
mysql> select * from t1;
+------+-------+
| id   | name  |
+------+-------+
|    1 | yayun |
|    2 | atlas |
+------+-------+
2 rows in set (0.00 sec)

mysql> insert into t1 select 1,'love sql';
Query OK, 1 row affected (0.01 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> select * from t1;                  
+------+----------+
| id   | name     |
+------+----------+
|    1 | yayun    |
|    2 | atlas    |
|    1 | love sql |
+------+----------+
3 rows in set (0.00 sec)
```

现在来一次增量备份1


```c
[root@MySQL-01 ~]$ innobackupex --user=backup --password=yayun --socket=/tmp/mysqld.sock --defaults-file=/etc/my.cnf --incremental /data/backup/incremental/ --incremental-basedir=/data/backup/full/2014-04-07_23-37-20/ --parallel=2
```
我们看看增量备份的大小以及文件内容

```c
[root@MySQL-01 ~]# du -sh /data/backup/full/2014-04-07_23-37-20/
1.2G    /data/backup/full/2014-04-07_23-37-20/
[root@MySQL-01 ~]# du -sh /data/backup/incremental/2014-04-07_23-42-46/
3.6M    /data/backup/incremental/2014-04-07_23-42-46/
```
看见增量备份的数据很小吧，就是备份改变的数据而已。

```bash
[root@MySQL-01 2014-04-07_23-42-46]# cat xtrabackup_checkpoints 
backup_type = incremental
from_lsn = 5324784718
to_lsn = 5324785066
last_lsn = 5324785066
compact = 0
```

我们再次向t1表插入数据，然后创建增量备份2

```c
mysql> select * from t1;
+------+----------+
| id   | name     |
+------+----------+
|    1 | yayun    |
|    2 | atlas    |
|    1 | love sql |
+------+----------+
3 rows in set (0.00 sec)

mysql> insert into t1 select 1,'mysql dba';
Query OK, 1 row affected (0.00 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> select * from t1;                   
+------+-----------+
| id   | name      |
+------+-----------+
|    1 | yayun     |
|    2 | atlas     |
|    1 | love sql  |
|    1 | mysql dba |
+------+-----------+
4 rows in set (0.00 sec)
```
创建增量备份2（这次是基于上次的增量备份）

```c
[root@MySQL-01 ~]# innobackupex --user=backup --password=yayun --socket=/tmp/mysqld.sock --defaults-file=/etc/my.cnf --incremental /data/backup/incremental/ --incremental-basedir=/data/backup/incremental/2014-04-07_23-42-46/ --parallel=2
[root@MySQL-01 ~]# ls -ltr /data/backup/full/
total 4
drwxr-xr-x 12 root root 4096 Apr  7 23:38 2014-04-07_23-37-20
[root@MySQL-01 ~]# ls -ltr /data/backup/incremental/
total 8
drwxr-xr-x 12 root root 4096 Apr  7 23:43 2014-04-07_23-42-46
drwxr-xr-x 12 root root 4096 Apr  7 23:51 2014-04-07_23-51-15
```
### 5.4 增量备份恢复

增量备份的恢复大体为3个步骤

 - *恢复完全备份
 - *恢复增量备份到完全备份（开始恢复的增量备份要添加`--redo-only`参数，到最后一次增量备份去掉--redo-only参数）
 - *对整体的完全备份进行恢复，回滚那些未提交的数据

恢复完全备份（`注意这里一定要加--redo-only参数，该参数的意思是只应用xtrabackup日志中已提交的事务数据，不回滚还未提交的数据`）

```c
[root@MySQL-01 ~]# innobackupex --apply-log --redo-only /data/backup/full/2014-04-07_23-37-20/
```
将增量备份1应用到完全备份

```c
[root@MySQL-01 ~]# innobackupex --apply-log --redo-only /data/backup/full/2014-04-07_23-37-20/ --incremental-dir=/data/backup/incremental/2014-04-07_23-42-46/
```

将增量备份2应用到完全备份（注意恢复最后一个增量备份时需要去掉--redo-only参数，回滚xtrabackup日志中那些还未提交的数据）

```c
[root@MySQL-01 ~]# innobackupex --apply-log /data/backup/full/2014-04-07_23-37-20/ --incremental-dir=/data/backup/incremental/2014-04-07_23-51-15/
```

把所有合在一起的完全备份整体进行一次apply操作，回滚未提交的数据：

```c
[root@MySQL-01 ~]# innobackupex --apply-log /data/backup/full/2014-04-07_23-37-20/
xtrabackup: starting shutdown with innodb_fast_shutdown = 1
```
把恢复完的备份复制到数据库目录文件中，赋权，然后启动mysql数据库，检测数据正确性

```c
[root@MySQL-01 ~]# /etc/init.d/mysqld stop
Shutting down MySQL.                                       [  OK  ]
[root@MySQL-01 ~]# mv /data/mysql /data/mysql_bak
[root@MySQL-01 ~]# mkdir /data/mysql
[root@MySQL-01 ~]# innobackupex --defaults-file=/etc/my.cnf --copy-back --rsync /data/backup/full/2014-04-07_23-37-20/
```
查看数据是否正确
```c
mysql> select * from t1;
+------+-----------+
| id   | name      |
+------+-----------+
|    1 | yayun     |
|    2 | atlas     |
|    1 | love sql  |
|    1 | mysql dba |
+------+-----------+
4 rows in set (0.00 sec)
```
参考链接：
[https://www.cnblogs.com/gomysql/p/3650645.HTML](https://www.cnblogs.com/gomysql/p/3650645.HTML)
