1. 进入information_schema 数据库（存放了其他的数据库的信息）

```bash
use information_schema;
```
2. 查询所有数据的大小：

```bash
select concat(round(sum(data_length/1024/1024),2),'MB') as data from tables;
+---------+
| data    |
+---------+
| 64.43MB |
+---------+

```

 
3. 查看指定数据库的大小：
比如查看数据库home的大小

```bash
select concat(round(sum(data_length/1024/1024),2),'MB') as data from tables where table_schema='home';
+---------+
| data    |
+---------+
| 39.56MB |
+---------+

```

 
4. 查看指定数据库的某个表的大小
比如查看数据库home中 members 表的大小

```bash
select concat(round(sum(data_length/1024/1024),2),'MB') as data from tables where table_schema='home' and table_name='members';
+--------+
| data   |
+--------+
| 0.02MB |
+--------+

```


5. 查看多个库的大小，并定义名称
以GB为单位
```bash
SELECT table_schema   "DB Name",     Round(Sum(data_length + index_length) / 1024 / 1024 / 1024, 1) "DB Size in GB"  FROM   information_schema.tables  GROUP  BY table_schema;
+--------------------+---------------+
| api_server         | DB Size in GB |
+--------------------+---------------+
| api_server         |           0.0 |
| horus              |           0.0 |
| information_schema |           0.0 |
| mgm                |           0.0 |
| mysql              |           0.0 |
| mysqlmanager       |           0.0 |
| performance_schema |           0.0 |
| redismanager       |           0.0 |
| sys                |           0.0 |
+--------------------+---------------+
```
以MB为单位
```bash
SELECT table_schema   "DB Name",     Round(Sum(data_length + index_length) / 1024 / 1024 , 1) "DB Size in MB"  FROM   information_schema.tables  GROUP  BY table_schema;
+--------------------+---------------+
| DB Name            | DB Size in MB |
+--------------------+---------------+
| api_server         |          49.1 |
| horus              |          11.7 |
| information_schema |           0.2 |
| mgm                |          15.1 |
| mysql              |           2.5 |
| mysqlmanager       |           0.1 |
| performance_schema |           0.0 |
| redismanager       |           0.0 |
| sys                |           0.0 |
+--------------------+---------------+

```

6. 查看所有数据库容量大小

```bash
select
table_schema as '数据库',
sum(table_rows) as '记录数',
sum(truncate(data_length/1024/1024, 2)) as '数据容量(MB)',
sum(truncate(index_length/1024/1024, 2)) as '索引容量(MB)'
from information_schema.tables
group by table_schema
order by sum(data_length) desc, sum(index_length) desc;

+--------------------+-----------+------------------+------------------+
| 数据库             | 记录数    | 数据容量(MB)     | 索引容量(MB)     |
+--------------------+-----------+------------------+------------------+
| api_server         |    120039 |            39.13 |             9.48 |
| mgm                |     27036 |            13.19 |             1.79 |
| horus              |     17270 |             8.90 |             2.66 |
| mysql              |      4253 |             2.20 |             0.15 |
| information_schema |      NULL |             0.10 |             0.00 |
| mysqlmanager       |       429 |             0.10 |             0.00 |
| redismanager       |        46 |             0.01 |             0.00 |
| sys                |         6 |             0.01 |             0.00 |
| performance_schema |   1344833 |             0.00 |             0.00 |
+--------------------+-----------+------------------+------------------+

```
7. 查看所有数据库各表容量大小
```bash
select
table_schema as '数据库',
table_name as '表名',
table_rows as '记录数',
truncate(data_length/1024/1024, 2) as '数据容量(MB)',
truncate(index_length/1024/1024, 2) as '索引容量(MB)'
from information_schema.tables
order by data_length desc, index_length desc;

+--------------------+------------------------------------------------------+-----------+------------------+------------------+
| 数据库             | 表名                                                 | 记录数    | 数据容量(MB)     | 索引容量(MB)     |
+--------------------+------------------------------------------------------+-----------+------------------+------------------+
| api_server         | tbl_auto_backup_info                                 |     46944 |            15.23 |             0.00 |
| api_server         | tbl_task                                             |     34604 |            14.40 |             7.03 |
| mgm                | tbl_task                                             |     25152 |            11.50 |     
```

8. 查看指定数据库容量大小
例：查看mysql库容量大小

```bash
select
table_schema as '数据库',
sum(table_rows) as '记录数',
sum(truncate(data_length/1024/1024, 2)) as '数据容量(MB)',
sum(truncate(index_length/1024/1024, 2)) as '索引容量(MB)'
from information_schema.tables
where table_schema='mysql';

+-----------+-----------+------------------+------------------+
| 数据库    | 记录数    | 数据容量(MB)     | 索引容量(MB)     |
+-----------+-----------+------------------+------------------+
| mysql     |      4253 |             2.20 |             0.15 |
+-----------+-----------+------------------+------------------+
```
9. 查看指定数据库各表容量大小
例：查看mysql库各表容量大小

```bash
select
table_schema as '数据库',
table_name as '表名',
table_rows as '记录数',
truncate(data_length/1024/1024, 2) as '数据容量(MB)',
truncate(index_length/1024/1024, 2) as '索引容量(MB)'
from information_schema.tables
where table_schema='mysql'
order by data_length desc, index_length desc;

+-----------+---------------------------+-----------+------------------+------------------+
| 数据库    | 表名                      | 记录数    | 数据容量(MB)     | 索引容量(MB)     |
+-----------+---------------------------+-----------+------------------+------------------+
| mysql     | help_topic                |       518 |             1.51 |             0.07 |
| mysql     | proc                      |        48 |             0.28 |             0.00 |
| mysql     | innodb_index_stats        |       551 |             0.10 |             0.00 |
| mysql     | help_keyword              |       726 |             0.09 |             0.07 |
| mysql     | help_relation             |      2233 |             0.07 |             0.00 |

```

