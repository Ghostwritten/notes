## 基本参数配置

```bash
bind-address    绑定的IP地址
user      用户
port     端口号
datadir   数据文件目录
basedir  msyql应用程序的目录
socket     socket文件，默认在/tmp目录下，但是建议不要这样设置，/tmp目录是一个大家都愿意破坏的目录
default-table-type   默认表类型
```
## 2.查询的Cache
**注意**：是从MySQL4.0版本开始提供的功能

```bash
query_cache_size   查询Cache的尺寸
query_cache_type   查询的Cache类型。0 OFF，不进行缓冲  1 ON，进行缓冲  2 DEMAND，对SELECT SQL_CACHE开头的查询进行缓冲
query_cache_limit  查询的结果的限制长度，小于这个长度的数据才能Cache
```


## 3.MyISAM的索引参数

```bash
key_buffer_size     MyISAM引擎的最关键的优化参数之一
key_buffer_size     (关键参数),索引块用的缓冲区大小，所有的连接程序线程共用
key_cache_block_size    每一个索引block的大小，默认1024字节,从4.1.1后才出现这个参数，原来都是直接采用1024字节作为Block的长度
```


## 4.InnoDB使用的参数
InnoDB的参数较少，笼统而不细致，内存的管理多由InnoDB引擎自己负责，

```bash
innodb_buffer_pool_size   innodb的缓冲区大小，存放数据和索引,一般设置为机器内存的50%-80% (关键参数)
innodb_log_buffer_size    InnoDB日志缓冲区大小
innodb_flush_method       刷新日志的方法
innodb_additional_mem_pool_size    innodb内存池的大小，存放着各种内部使用的数据结构
innodb_data_home_dir      InnoDB数据文件的目录
innodb_data_file_path     数据文件配置
innodb_log_files_in_group  Innodb日志的
innodb_log_file_size      Innodb日志文件的尺寸
innodb_lock_wait_timeout  等待数据锁的超时时间，避免死锁的一种措施
innodb_flush_log_at_trx_commit   日志提交方式 (关键参数)
                                 0每秒写1次日志，将数据刷入磁盘，相当于每秒提交一次事务。
                                 1每次提交事务写日志，同时将刷新相应磁盘，默认参数。
                                 2每提交事务写一次日志，但每隔一秒刷新一次相应的磁盘文件

[注]
innodb_force_recovery在Innodb的自动恢复失败后，从崩溃中强制启动，有1-6个级别，数值越低恢复的方式也保守，默认为4。尽量使用较保守方式恢复。恢复后要注释删除这一行。
```


## 5.Log的参数
MySQL的日志有6种：
**查询日志，慢查询日志，变更日志，二进制变更日志，告警日志，错误日志。**
my.cnf中可以配置日志的前缀和日志参数。日志是监控数据库系统的重要途径

```bash
log    查询日志，记录所有的MySQL的命令操作，在跟踪数据库运行时非常有帮助，但在实际环境中就不要使用了
log-update   变更日志，用文本方式记录所有改变数据的变更操作，
log-bin    二进制变更日志，更加紧凑，使用mysqlbinlog读取，操作，转换
binlog_cache_size   临时存放某次事务的SQL语句缓冲长度
max_binlog_cache_szie    最大的二进制Cache日志缓冲区尺寸
max_binlog_size   最大的二进制日志尺寸
log-error   导致无法启动的错误日志
log-warnings   告警日志
long_query_time    慢查询时间限度，超过这个限度，mysqld认为是一个慢查询
log-queries-not-using-indexes    没有使用索引查询的日志,方便记录长时间访问的查询进行优化
log-slow-queries   慢速的查询日志，
```

## 6.安全

```bash
secure_file_prive=null   -- 限制mysqld 不允许导入导出
secure_file_priv=/tmp/   -- 限制mysqld的导入导出只能发生在/tmp/目录下
secure_file_priv=''     -- 不对mysqld 的导入 导出做限制
```

## 7.实战
示例1

```bash
$ vim /etc/my.cnf

[client]
socket = /var/sock/mysqld/mysqld.sock

[mysql]
socket = /var/sock/mysqld/mysqld.sock
[mysqld]
skip-host-cache
skip-name-resolve
datadir = /var/lib/mysql  #mysql数据库
user = mysql              #mysql用户
port = 3306               #端口
bind-address = 0.0.0.0     #绑定地址
socket = /var/sock/mysqld/mysqld.sock
pid-file = /var/run/mysqld/mysqld.pid
general_log_file = /var/log/mysql/query.log
slow_query_log_file = /var/log/mysql/slow.log
log-error = /var/log/mysql/error.log
!includedir /etc/my.cnf.d/
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/docker-default.d/

symbolic-links = 0
character_set_server = utf8
explicit_defaults_for_timestamp = true
innodb_buffer_pool_size  = 1024M
innodb_data_file_path  = ibdata1:512M:autoextend
lower_case_table_names = 1
sql_mode = 'ALLOW_INVALID_DATES,NO_AUTO_CREATE_USER'
```

