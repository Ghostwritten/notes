
## 1. 背景
近几年，开源数据库逐渐流行起来。由于具有免费使用、配置简单、稳定性好、性能优良等优点，开源数据库在中低端应用上占据了很大的市场份额，而 MySQL 正是开源数据库中的杰出代表。MySQL 数据库目前分为社区版（Community Server）和企业版（Enterprise），它们最重要的区别在于：社区版是自由下载而且完全免费的，但是官方不提供任何技术支持，适用于大多数普通用户；而企业版则是收费的，不能在线下载，相应地，它提供了更多的功能和更完备的技术支持，更适合于对数据库的功能和可靠性要求较高的企业客户。本篇博客将通过丰富的实例对 SQL 语言的基础进行详细介绍，MySQL，使得读者不但能够学习到标准 SQL【Structure Query Language（结构化查询语言）】 的使用，又能够学习到 MySQL 中一些扩展 SQL 的使用方法。

PS:本片博客的内容都是借鉴了深入浅出MySQL这本书，真是一本SQL入门好书，推荐。

## 2. SQL 分类：

SQL 语句主要可以划分为以下 3 个类别。

`DDL`（Data Definition Languages）语句：数据定义语言，这些语句定义了不同的数据段、数据库、表、列、索引等数据库对象的定义。常用的语句关键字主要包括 create、drop、alter等。

`DML`（Data Manipulation Language）语句：数据操纵语句，用于添加、删除、更新和查询数据库记录，并检查数据完整性，常用的语句关键字主要包括 insert、delete、udpate 和select 等。(增添改查）

`DCL`（Data Control Language）语句：数据控制语句，用于控制不同数据段直接的许可和访问级别的语句。这些语句定义了数据库、表、字段、用户的访问权限和安全级别。主要的语句关键字包括 grant、revoke 等。

### 2.1 DDL 语句：

DDL 是数据定义语言的缩写，简单来说，就是对数据库内部的对象进行创建、删除、修改的操作语言。它和 DML 语言的最大区别是 DML 只是对表内部数据的操作，而不涉及到表的定义、结构的修改，更不会涉及到其他对象。DDL 语句更多的被数据库管理员（DBA）所使用，一般的开发人员很少使用。

下面通过一些例子来介绍 MySQL 中常用 DDL 语句的使用方法。

1．创建数据库

启动 MySQL 服务之后，输入以下命令连接到 MySQL 服务器：

```bash
[mysql@db3 ~]$ mysql -uroot -p

Enter password:

Welcome to the MySQL monitor. Commands end with ; or \g.

Your MySQL connection id is 7344941 to server version: 5.1.9-beta-log

Type 'help;' or '\h' for help. Type '\c' to clear the buffer.

mysql>
```

在以上命令行中，mysql 代表客户端命令，-u 后面跟连接的数据库用户，-p 表示需要输入密码。如果数据库设置正常，并输入正确的密码，将看到上面一段欢迎界面和一个 mysql>提示符。在欢迎界面中介绍了以下几部分内容。

命令的结束符：用;或者\g 结束。

客户端的连接 ID：这个数字记录了 MySQL 服务到目前为止的连接次数，每个新连接都会自动加 1，本例中是 7344941。

MySQL 服务器的版本：本例中是“5.1.9-beta-log”，说明是 5.1.9 的测试版，如果是标准版，则会用 Standard 代替 Beta。

通过“help;”或者“\h”命令来显示帮助内容：通过“\c”命令来清除命令行 buffer。

在 mysql>提示符后面输入所要执行的的 SQL 语句，每个 SQL 语句以分号或者\g 结束，按回车键执行。

因为所有的数据都存储在数据库中，因此需要学习的第一个命令是创建数据库，语法如下所示：

CREATE DATABASE dbname

例如，创建数据库 test1，命令执行如下：

```bash
mysql> create database test1;

Query OK, 1 row affected (0.00 sec)
```

可以发现，执行完创建命令后，下面有一行提示“Query OK, 1 row affected (0.00 sec)”，这段提示可以分为 3 部分，“Query OK”表示上面的命令执行成功，读者可能奇怪，又不是执行查询操作，为什么显示查询成功？其实这是 MySQL 的一个特点，所有的 DDL 和 DML（不包括 SELECT）操作执行成功后都显示“Query OK”，这里理解为执行成功就可以了；“1 row

affected”表示操作只影响了数据库中一行的记录，“0.00 sec”则记录了操作执行的时间。这个时候，如果需要知道系统中都存在哪些数据库，可以用以下命令来查看：

```bash
mysql> show databases;
+--------------------+
| Database |
+--------------------+
| information_schema |
| cluster |
| mysql |
| test |
| test1 |
+--------------------+
5 rows in set (0.00 sec)
```

可以发现，在上面的列表中除了刚刚创建的 test1 外，还有另外 4 个数据库，它们都是安装MySQL 时系统自动创建的，其各自功能如下。

information_schema：主要存储了系统中的一些数据库对象信息。比如用户表信息、列信息、权限信息、字符集信息、分区信息等。

cluster：存储了系统的集群信息。

mysql：存储了系统的用户权限信息。

 test：系统自动创建的测试数据库，任何用户都可以使用。

在查看了系统中已有的数据库后，可以用如下命令选择要操作的数据库：USE dbname

 

然后再用以下命令来查看 test1 数据库中创建的所有数据表：mysql> show tables;

2．删除数据库

删除数据库的语法很简单，如下所示：drop database dbname;

例如，要删除 test1 数据库可以使用以下语句：mysql> drop database test1;

 

注意：数据库删除后，下面的所有表数据都会全部删除，所以删除前一定要仔细检查并做好相应备份。

3．创建表

在数据库中创建一张表的基本语法如下：

```bash
CREATE TABLE tablename (
column_name_1  column_type_1  constraints，
column_name_2  column_type_2  constraints ， 
……
column_name_n  column_type_n  constraints）
```

因为 MySQL 的表名是以目录的形式存在于磁盘上，所以表名的字符可以用任何目录名允许的字符。column_name 是列的名字，column_type 是列的数据类型，contraints 是这个列的约束条件。

例如，创建一个名称为 emp 的表。表中包括 3 个字段，ename（姓名），hiredate（雇用日期）、sal（薪水），字段类型分别为 varchar（10）、date、int（2）：

```bash
mysql> create table emp(
ename varchar(10),
hiredate date,
sal decimal(10,2),deptno int(2));
Query OK, 0 rows affected (0.02 sec)
```

表创建完毕后，如果需要查看一下表的定义，可以使用如下命令：DESC tablename

虽然 desc 命令可以查看表定义，但是其输出的信息还是不够全面，为了查看更全面的表定义信息，有时就需要通过查看创建表的 SQL 语句来得到，可以使用如下命令实现：mysql> show create table emp \G;

```bash
*************************** 1. row ***************************
Table: emp
Create Table: CREATE TABLE 'emp' (
'ename' varchar(20) DEFAULT NULL,
'hiredate' date DEFAULT NULL,
'sal' decimal(10,2) DEFAULT NULL,
'deptno' int(2) DEFAULT NULL,
KEY idx_emp_ename' ('ename')
) ENGINE=InnoDB DEFAULT CHARSET=gbk
1 row in set (0.02 sec)
ERROR:
No query specified
mysql>
```

从上面表的创建 SQL 语句中，除了可以看到表定义以外，还可以看到表的 engine（存储引擎）和 charset（字符集）等信息。“\G”选项的含义是使得记录能够按照字段竖着排列，对于内容比较长的记录更易于显示。

4．删除表

表的删除命令如下：DROP TABLE tablename

例如，要删除数据库 emp 可以使用以下命令：mysql> drop table emp;

5．修改表（重要）

对于已经创建好的表，尤其是已经有大量数据的表，如果需要对表做一些结构上的改变，我们可以先将表删除（drop），然后再按照新的表定义重建表。这样做没有问题，但是必然要做一些额外的工作，比如数据的重新加载。而且，如果有服务在访问表，也会对服务产生影响。因此，在大多数情况下，表结构的更改一般都使用 alter table 语句，以下是一些常用的命令。

（1） 修改表类型，语法如下：

```bash
ALTER TABLE tablename MODIFY [COLUMN] column_definition [FIRST | AFTER col_name]
```

例如，修改表 emp 的 ename 字段定义，将 varchar(10)改为 varchar(20)：mysql> alter table emp modify ename varchar(20);

 

（2） 增加表字段，语法如下：

```bash
ALTER TABLE tablename ADD [COLUMN] column_definition [FIRST | AFTER col_name]
```

例如，表 emp 上新增加字段 age，类型为 int(3)：mysql> alter table emp add column age int(3);

（3）删除表字段，语法如下：

```bash
ALTER TABLE tablename DROP [COLUMN] col_name
```

例如，将字段 age 删除掉：mysql> alter table emp drop column age;

（4）字段改名，语法如下：

```bash
ALTER TABLE tablename CHANGE [COLUMN] old_col_name column_definition [FIRST|AFTER col_name]
```

例如，将 age 改名为 age1，同时修改字段类型为 int(4)：mysql> alter table emp change age age1 int(4) ;

 

注意：change 和 modify 都可以修改表的定义，不同的是 change 后面需要写两次列名，不方便。但是 change 的优点是可以修改列名称，modify 则不能。

（5）修改字段排列顺序。

前面介绍的的字段增加和修改语法（ADD/CNAHGE/MODIFY）中，都有一个可选项 first|after column_name，这个选项可以用来修改字段在表中的位置，默认 ADD 增加的新字段是加在表的最后位置，而 CHANGE/MODIFY 默认都不会改变字段的位置。

例如，将新增的字段 birth date 加在 ename 之后：mysql> alter table emp add birth date after ename;

 

修改字段 age，将它放在最前面：mysql> alter table emp modify age int(3) first;

 

注意：CHANGE/FIRST|AFTER COLUMN 这些关键字都属于 MySQL 在标准 SQL 上的扩展，在其他数据库上不一定适用。

（6）表改名，语法如下：ALTER TABLE tablename RENAME [TO] new_tablename

例如，将表 emp 改名为 emp1，命令如下：mysql> alter table emp rename emp1;

### 2.2 DML 语句：

DML 操作是指对数据库中表记录的操作，主要包括表记录的插入（insert）、更新（update）、删除（delete）和查询（select），是开发人员日常使用最频繁的操作。下面将依次对它们进行介绍。

1．插入记录

表创建好后，就可以往里插入记录了，插入记录的基本语法如下：

```bash
INSERT INTO tablename (field1,field2,……fieldn) VALUES(value1,value2,……valuesn);
```

例如，向表 emp 中插入以下记录：ename 为 zzx1，hiredate 为 2000-01-01，sal 为 2000，deptno

为 1，命令执行如下：

```bash
mysql> insert into emp (ename,hiredate,sal,deptno) values('zzx1','2000-01-01','2000',1);

Query OK, 1 row affected (0.00 sec)
```

也可以不用指定字段名称，但是 values 后面的顺序应该和字段的排列顺序一致：

```bash
mysql> insert into emp values('lisa','2003-02-01','3000',2);

Query OK, 1 row affected (0.00 sec)
```

对于含可空字段、非空但是含有默认值的字段、自增字段，可以不用在 insert 后的字段列表

里面出现，values 后面只写对应字段名称的 value，这些没写的字段可以自动设置为 NULL、

默认值、自增的下一个数字，这样在某些情况下可以大大缩短 SQL 语句的复杂性。

例如，只对表中的 ename 和 sal 字段显式插入值：

```bash
mysql> insert into emp (ename,sal) values('dony',1000);

Query OK, 1 row affected (0.00 sec)
```

来查看一下实际插入值：

```bash
mysql> select * from emp;

+--------+------------+---------+--------+

| ename | hiredate | sal | deptno |

+--------+------------+---------+--------+

| zzx | 2000-01-01 | 100.00 | 1 |

| lisa | 2003-02-01 | 400.00 | 2 |

| bjguan | 2004-04-02 | 100.00 | 1 |

| dony | NULL | 1000.00 | NULL |

+--------+------------+---------+--------+
```

果然，设置为可空的两个字段都显示为 NULL。

在 MySQL 中，insert 语句还有一个很好的特性，可以一次性插入多条记录，语法如下：

```bash
INSERT INTO tablename (field1, field2,……fieldn)

VALUES

(record1_value1, record1_value2,……record1_valuesn),

(record2_value1, record2_value2,……record2_valuesn),

……

(recordn_value1, recordn_value2,……recordn_valuesn)
```



参考连接
https://www.cnblogs.com/zhangmingcheng/p/5295684.html



