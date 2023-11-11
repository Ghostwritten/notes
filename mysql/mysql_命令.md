

## 1. 查看
### 1.1 查看uuid

```bash
select @@server_uuid；
+--------------------------------------+
| @@server_uuid                        |
+--------------------------------------+
| ea5085ce-ac82-11ea-9dcd-005056b25447 |
+--------------------------------------+
1 row in set (0.00 sec)

```
### 1.2 查看binlog事件

```bash
show binlog events;
+---------------------------------+-----+----------------+------------+-------------+---------------------------------------+
| Log_name                        | Pos | Event_type     | Server_id  | End_log_pos | Info                                  |
+---------------------------------+-----+----------------+------------+-------------+---------------------------------------+
| 161cc2f8_fKGvn001-binlog.000001 |   4 | Format_desc    | 3232235526 |         123 | Server ver: 5.7.22-log, Binlog ver: 4 |
| 161cc2f8_fKGvn001-binlog.000001 | 123 | Previous_gtids | 3232235526 |         154 |                                       |
+---------------------------------+-----+----------------+------------+-------------+---------------------------------------+
2 rows in set (0.01 sec)

```
### 1.3 查看binlog日志
mysql> show binary  logs;
+---------------------------------+-----------+
| Log_name                        | File_size |
+---------------------------------+-----------+
| 161cc2f8_fKGvn001-binlog.000001 |       154 |
+---------------------------------+-----------+
1 row in set (0.00 sec)

### 1.4 flush
```bash
flush;
help flush;

flush slow logs;
flush local slow logs;

flush binary logs;
flush local binary logs;

flush privileges;
flush local privileges;
```
### 1.5 查看用户权限

```bash
$ show grants;
$  show grants for test;
```
### 1.6 查看表

```bash
$ show table mysql_user;
$ show create table mysql_user;
```

###  1.7 查看进程

```bash
show processlist;
```
###  1.8 查看打开表

```bash
show open tables where in_use > 0;
```
###  1.9 查看锁表

```bash
select * from metadata_locks;
select * from threads order by processlist_time desc limit 10;
```

###  1.10 查看锁源当前执行的SQL语句

```bash
#查出锁源 SQL 语句
SELECT THREAD_ID, SQL_TEXT AS '锁源当前执行的SQL语句' ,CURRENT_SCHEMA AS '数据库' FROM performance_schema.`events_statements_current` 
WHERE thread_id IN (SELECT THREAD_ID FROM performance_schema.threads pt WHERE processlist_id IN (SELECT blocking_pid FROM sys.innodb_lock_waits));

show full processlist；

```

### 1.11  查看表的数据量
从库里面查一下表的数据量   

```bash
select count(id） from regulation.jg_jgsx_publish_sun_detail;
```

再看下索引 

```bash
show index from  regulation.jg_jgsx_publish_sun_detail;

```
表结构

```bash
desc  regulation.jg_jgsx_publish_sun_detail;
```


## 2 查询 
### 2.1 查询在A表不在B表的数据
假设有A、B两张表。

如果查询在A表中存在，但是在B表中不存在的记录，应该如何操作？
A表
| id |
|----|
| 1  |
| 2  |
| 3  |
| 4  |
| 5  |
B表
| id | a\_id |
|----|-------|
| 1  | 3     |

#### 2.1.1 子查询方法
```bash

select A.* from A where A.id not in(select B.a_id from B);
```
#### 2.1.2 使用join方法

```bash
select * from A left join B on A.id = B.a_id;
```


