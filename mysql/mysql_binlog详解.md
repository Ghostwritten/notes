

## 1.什么是binlog

**binlog日志用于记录所有更新了数据或者已经潜在更新了数据（例如，没有匹配任何行的一个DELETE）的所有语句。语句以“事件”的形式保存，它描述数据更改。**

## 2.binlog格式

Mysql binlog日志有三种格式，分别为Statement,MiXED,以及ROW！
1.Statement：每一条会修改数据的sql都会记录在binlog中。

优点：不需要记录每一行的变化，减少了binlog日志量，节约了IO，提高性能。(相比row能节约多少性能与日志量，这个取决于应用的SQL情况，正常同一条记录修改或者插入row格式所产生的日志量还小于Statement产生的日志量，但是考虑到如果带条件的update操作，以及整表删除，alter表等操作，ROW格式会产生大量日志，因此在考虑是否使用ROW格式日志时应该跟据应用的实际情况，其所产生的日志量会增加多少，以及带来的IO性能问题。)

缺点：由于记录的只是执行语句，为了这些语句能在slave上正确运行，因此还必须记录每条语句在执行的时候的一些相关信息，以保证所有语句能在slave得到和在master端执行时候相同 的结果。另外mysql 的复制,像一些特定函数功能，slave可与master上要保持一致会有很多相关问题(如sleep()函数， last_insert_id()，以及user-defined functions(udf)会出现问题).

使用以下函数的语句也无法被复制：

* LOAD_FILE()

* UUID()

* USER()

* FOUND_ROWS()

* SYSDATE() (除非启动时启用了 --sysdate-is-now 选项)

同时在INSERT ...SELECT 会产生比 RBR 更多的行级锁

2.Row:不记录sql语句上下文相关信息，仅保存哪条记录被修改。

优点： binlog中可以不记录执行的sql语句的上下文相关的信息，仅需要记录那一条记录被修改成什么了。所以rowlevel的日志内容会非常清楚的记录下每一行数据修改的细节。而且不会出现某些特定情况下的存储过程，或function，以及trigger的调用和触发无法被正确复制的问题

缺点:所有的执行的语句当记录到日志中的时候，都将以每行记录的修改来记录，这样可能会产生大量的日志内容,比如一条update语句，修改多条记录，则binlog中每一条修改都会有记录，这样造成binlog日志量会很大，特别是当执行alter table之类的语句的时候，由于表结构修改，每条记录都发生改变，那么该表每一条记录都会记录到日志中。

3.Mixedlevel: 是以上两种level的混合使用，一般的语句修改使用statment格式保存binlog，如一些函数，statement无法完成主从复制的操作，则采用row格式保存binlog,MySQL会根据执行的每一条具体的sql语句来区分对待记录的日志形式，也就是在Statement和Row之间选择一种.新版本的MySQL中队row level模式也被做了优化，并不是所有的修改都会以row level来记录，像遇到表结构变更的时候就会以statement模式来记录。至于update或者delete等修改数据的语句，还是会记录所有行的变更。
## 3.binlog作用
**因为有了数据更新的binlog，所以可以用于实时备份，与master/slave复制**

## 4.binlog基本配制与格式设定

1.基本配制

Mysql BInlog日志格式可以通过mysql的my.cnf文件的属性binlog_format指定。如以下：

```bash
binlog_format = MIXED                 //binlog日志格式
log_bin  =目录/mysql-bin.log    //binlog日志名
expire_logs_days = 7                //binlog过期清理时间
max_binlog_size=100m                    //binlog每个日志文件大小
binlog-do-db=需要备份的数据库名，如果备份多个数据库，重复设置这个选项即可
binlog-ignore-db=不需要备份的数据库苦命，如果备份多个数据库，重复设置这个选项即可
```

2.Binlog日志格式选择

Mysql默认是使用`Statement`日志格式，推荐使用`MIXED.`

由于一些特殊使用，可以考虑使用ROWED，如自己通过binlog日志来同步数据的修改，这样会节省很多相关操作。对于binlog数据处理会变得非常轻松,相对mixed，解析也会很轻松(当然前提是增加的日志量所带来的IO开销在容忍的范围内即可)。

3.mysqlbinlog格式选择

mysql对于日志格式的选定原则:如果是采用 INSERT，UPDATE，DELETE 等直接操作表的情况，则日志格式根据 binlog_format 的设定而记录,如果是采用 GRANT，REVOKE，SET PASSWORD 等管理语句来做的话，那么无论如何 都采用 SBR 模式记录
## 5.binlog参数

```c
log_bin   设置此参数表示启用binlog功能，并指定路径名称
log_bin_index   设置此参数是指定二进制索引文件的路径与名称
binlog_do_db  此参数表示只记录指定数据库的二进制日志
binlog_ignore_db    此参数表示不记录指定的数据库的二进制日志
max_binlog_cache_size    此参数表示binlog使用的内存最大的尺寸
binlog_cache_size   此参数表示binlog使用的内存大小，可以通过状态变量binlog_cache_use和binlog_cache_disk_use来帮助测试。
binlog_cache_use   使用二进制日志缓存的事务数量
binlog_cache_disk_use   使用二进制日志缓存但超过binlog_cache_size值并使用临时文件来保存事务中的语句的事务数量
max_binlog_size    Binlog最大值，最大和默认值是1GB，该设置并不能严格控制Binlog的大小，尤其是Binlog比较靠近最大值而又遇到一个比较大事务时，为了保证事务的完整性，不可能做切换日志的动作，只能将该事务的所有SQL都记录进当前日志，直到事务结束
sync_binlog    这个参数直接影响mysql的性能和完整性
sync_binlog=0：当事务提交后，Mysql仅仅是将binlog_cache中的数据写入Binlog文件，但不执行fsync之类的磁盘同步指令通知文件系统将缓存刷新到磁盘，而让Filesystem自行决定什么时候来做同步，这个是性能最好的。
sync_binlog=n：在进行n次事务提交以后，Mysql将执行一次fsync之类的磁盘同步指令，同志文件系统将Binlog文件缓存刷新到磁盘。
Mysql中默认的设置是sync_binlog=0，即不作任何强制性的磁盘刷新指令，这时性能是最好的，但风险也是最大的。一旦系统绷Crash，在文件系统缓存中的所有Binlog信息都会丢失
```

## 6.binlog的删除

binlog的删除可以手工删除或自动删除
### 自动删除binlog
通过binlog参数（expire_logs_days ）来实现mysql自动删除binlog
expire_logs_days：控制binlog日志文件保留时间，超过保留时间的binlog日志会被自动删除
```c
mysql> show binary logs;
mysql> show variables like 'expire_logs_days';
mysql> set global expire_logs_days=3;
```
### 手工删除binlog

```bash
mysql> reset master;   //删除master的binlog
mysql> reset slave;    //删除slave的中继日志
mysql> purge master logs before '2012-03-30 17:20:00';  //删除指定日期以前的日志索引中binlog日志文件
mysql> purge master logs to 'binlog.000002';   //删除指定日志文件的日志索引中binlog日志文件
```

或者直接用操作系统命令直接删除

```bash
mysql> set sql_log_bin=1/0; //如果用户有super权限，可以启用或禁用当前会话的binlog记录
mysql> show master logs; //查看master的binlog日志
mysql> show binary logs; //查看master的binlog日志
mysql> show master status; //用于提供master二进制日志文件的状态信息
mysql> show slave hosts; //显示当前注册的slave的列表。不以--report-host=slave_name选项为开头的slave不会显示在本列表中
```

## 7.binglog的查看

通过mysqlbinlog命令可以查看binlog的内容

```bash
[root@localhost ~]# mysqlbinlog  /home/mysql/binlog/binlog.000003  | more
```

解析binlog格式

```c
1.开始事物的时间:
SET TIMESTAMP=1350355892/*!*/;
BEGIN
2.sqlevent起点
#at 1643330 :为事件的起点，是以1643330字节开始。
3.sqlevent 发生的时间点
#121016 10:51:32:是事件发生的时间，
4.serverId
server id 1 :为master 的serverId
5.sqlevent终点及花费时间，错误码
end_log_pos 1643885:为事件的终点，是以1643885 字节结束。
execTime 0: 花费的时间
error_code=0:错误码
Xid:事件指示提交的XA事务
```

binlog开启成功之后，binlog文件的位置可以在my.inf配置文件中查看。也可以在mysql的命令行中查看。命令行查看代码如下

```bash
show variables like '%log_bin%';
```

我们也可以看一下当前mysql的binlog的情况

```bash
show master status;
```

每当我们`重启一次，会自动生成一个binlog文件`，我们重启完毕之后再来执行同样的命令
存放binlog的目录下也多个了这么一个文件。
当然，我们也可以手动的来刷新binlog文件，通过 `flush logs,同样会新创建一个binlog文件`
如果我们想把这些文件全部清空，可以使用reset master 来处理
 
下面我们来简单总结一下关于binlog：
1.binlog文件会随服务的启动创建一个新文件
2.通过flush logs 可以手动刷新日志，生成一个新的binlog文件
3.通过show master status 可以查看binlog的状态
4.通过reset master 可以清空binlog日志文件
5.通过mysqlbinlog 工具可以查看binlog日志的内容
6.通过执行dml，mysql会自动记录binlog
 
