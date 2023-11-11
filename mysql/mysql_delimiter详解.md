其实就是告诉MySQL解释器，该段命令是否已经结束了，mysql是否可以执行了。默认情况下，其实就是告诉MySQL解释器，该段命令是否已经结束了，mysql是否可以执行了。默认情况下，delimiter是分号;。在命令行客户端中，如果有一行命令以分号结束，那么回车后，mysql将会执行该命令。

```sql
DELIMITER $$   
DROP TRIGGER IF EXISTS `updateegopriceondelete`$$   
CREATE   
    TRIGGER `updateegopriceondelete` AFTER  DELETE ON  `customerinfo`   
    FOR EACH ROW BEGIN   
DELETE FROM egoprice  WHERE customerId=OLD.customerId;   
    END$$   
DELIMITER ;   
```

**其中`DELIMITER` 定好结束符为"$$", 然后最后又定义为";", MYSQL的默认结束符为";". 
详细解释: 
其实就是告诉mysql解释器，该段命令是否已经结束了，mysql是否可以执行了。 
默认情况下，delimiter是分号;。在命令行客户端中，如果有一行命令以分号结束， 
那么回车后，mysql将会执行该命令**。如输入下面的语句 

```sql
mysql> select * from test_table; 
```

然后回车，那么MySQL将立即执行该语句。 
但有时候，不希望MySQL这么做。在为可能输入较多的语句，且语句中包含有分号。 
如试图在命令行客户端中输入如下语句 

```sql
mysql> CREATE FUNCTION `SHORTEN`(S VARCHAR(255), N INT)   
mysql>     RETURNS varchar(255)   
mysql> BEGIN   
mysql> IF ISNULL(S) THEN   
mysql>     RETURN '';   
mysql> ELSEIF N<15 THEN   
mysql>     RETURN LEFT(S, N);   
mysql> ELSE   
mysql>     IF CHAR_LENGTH(S) <=N THEN   
mysql>    RETURN S;   
mysql>     ELSE   
mysql>    RETURN CONCAT(LEFT(S, N-10), '...', RIGHT(S, 5));   
mysql>     END IF;   
mysql> END IF;   
mysql> END;   
```

默认情况下，不可能等到用户把这些语句全部输入完之后，再执行整段语句。 
因为mysql一遇到分号，它就要自动执行。 
即，在语句RETURN '';时，mysql解释器就要执行了。 
这种情况下，就需要事先把delimiter换成其它符号，如//或$$。 

```sql
mysql> delimiter //   
mysql> CREATE FUNCTION `SHORTEN`(S VARCHAR(255), N INT)   
mysql>     RETURNS varchar(255)   
mysql> BEGIN   
mysql> IF ISNULL(S) THEN   
mysql>     RETURN '';   
mysql> ELSEIF N<15 THEN   
mysql>     RETURN LEFT(S, N);   
mysql> ELSE   
mysql>     IF CHAR_LENGTH(S) <=N THEN   
mysql>    RETURN S;   
mysql>     ELSE   
mysql>    RETURN CONCAT(LEFT(S, N-10), '...', RIGHT(S, 5));   
mysql>     END IF;   
mysql> END IF;   
mysql> END;//   
```

这样只有当//出现之后，mysql解释器才会执行这段语句 .
例子

```sql
mysql> delimiter //   
  
mysql> CREATE PROCEDURE simpleproc (OUT param1 INT)   
-> BEGIN   
-> SELECT COUNT(*) INTO param1 FROM t;   
-> END;   
-> //   
Query OK, 0 rows affected (0.00 sec)   
  
mysql> delimiter ;   
  
mysql> CALL simpleproc(@a);   
Query OK, 0 rows affected (0.00 sec)   
  
mysql> SELECT @a;   
+------+   
| @a |   
+------+   
| 3 |   
+------+   
1 row in set (0.00 sec)   
```

默认情况下，delimiter “;” 用于向 MySQL 提交查询语句。在存储过程中每个 SQL 语句的结尾都有个 “;”，如果这时候，每逢 “;” 就向 MySQL 提交的话，当然会出问题了。于是更改 MySQL 的 delimiter，上面 MySQL 存储过程就编程这样子了： 

```sql
delimiter //;     -- 改变 MySQL delimiter 为：“//”   
  
drop procedure if exists pr_stat_agent //   
  
-- call pr_stat_agent ('2008-07-17', '2008-07-18')   
  
create procedure pr_stat_agent   
(   
   pi_date_from  date   
  ,pi_date_to    date   
)   
begin   
   -- check input   
   if (pi_date_from is null) then   
      set pi_date_from = current_date();   
   end if;   
  
   if (pi_date_to is null) then   
      set pi_date_to = pi_date_from;   
   end if;   
  
   set pi_date_to = date_add(pi_date_from, interval 1 day);   
  
   -- stat   
   select agent, count(*) as cnt   
     from apache_log   
    where request_time >= pi_date_from   
      and request_time <  pi_date_to   
    group by agent   
    order by cnt desc;   
end; //   
  
delimiter ; //   -- 改回默认的 MySQL delimiter：“;”   
```

当然，MySQL delimiter 符号是可以自由设定的，你可以用 “/” 或者“” 等。但是 MySQL 存储过程中比较常见的用法是 “//” 和 “”。上面的这段在 SQLyog 中的代码搬到 MySQL 命令客户端（MySQL Command Line Client）却不能执行。 
真是奇怪了！最后终于发现问题了，在 MySQL 命令行下运行 “delimiter //; ” 则 MySQL 的 delimiter 实际上是 “//;”，而不是我们所预想的 “//”。其实只要运行指令 “delimiter //” 就 OK 了。 
 

```sql
mysql> delimiter //     -- 末尾不要符号 “;”   
mysql>   
mysql> drop procedure if exists pr_stat_agent //   
Query OK, 0 rows affected (0.00 sec)   
  
mysql>   
mysql> -- call pr_stat_agent ('2008-07-17', '2008-07-18')   
mysql>   
mysql> create procedure pr_stat_agent   
    -> (   
    ->    pi_date_from  date   
    ->   ,pi_date_to    date   
    -> )   
    -> begin   
    ->    -- check input   
    ->    if (pi_date_from is null) then   
    ->       set pi_date_from = current_date();   
    ->    end if;   
    ->   
    ->    if (pi_date_to is null) then   
    ->       set pi_date_to = pi_date_from;   
    ->    end if;   
    ->   
    ->    set pi_date_to = date_add(pi_date_from, interval 1 day);   
    ->   
    ->    -- stat   
    ->    select agent, count(*) as cnt   
    ->      from apache_log   
    ->     where request_time >= pi_date_from   
    ->       and request_time <  pi_date_to   
    ->     group by agent   
    ->     order by cnt desc;   
    -> end; //   
Query OK, 0 rows affected (0.00 sec)   
  
mysql>   
mysql> delimiter ;  -- 末尾不要符号 “//”   
```

参考连接：
[https://blog.csdn.net/yuxin6866/article/details/52722913](https://blog.csdn.net/yuxin6866/article/details/52722913)
