

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/38acec3ef9d1cb5ea78a658d61c65ee5.png)


## 1. 触发器特性

 - 1、有begin end体，begin end;之间的语句可以写的简单或者复杂
   
   　　2、什么条件会触发：I、D、U
   
   　　3、什么时候触发：在增删改前或者后
   
   　　4、触发频率：针对每一行执行
   
   　　5、触发器定义在表上，附着在表上。

也就是由事件来触发某个操作，事件包括INSERT语句，UPDATE语句和DELETE语句；可以协助应用在数据库端确保数据的完整性。

## 2. 使用注意

cannot associate a trigger with a TEMPORARY table or a view.
！！尽量少使用触发器，不建议使用。
　　假设触发器触发每次执行1s，insert table 500条数据，那么就需要触发500次触发器，光是触发器执行的时间就花费了500s，而insert 500条数据一共是1s，那么这个insert的效率就非常低了。因此我们特别需要注意的一点是触发器的begin end;之间的语句的执行效率一定要高，资源消耗要小。

　　触发器尽量少的使用，因为不管如何，它还是很消耗资源，如果使用的话要谨慎的使用，确定它是非常高效的：触发器是针对每一行的；对增删改非常频繁的表上切记不要使用触发器，因为它会非常消耗资源。 

## 3. 创建触发器

```sql
CREATE
    [DEFINER = { user | CURRENT_USER }]
TRIGGER trigger_name
trigger_time trigger_event
ON tbl_name FOR EACH ROW
　　[trigger_order]
trigger_body

trigger_time: { BEFORE | AFTER }

trigger_event: { INSERT | UPDATE | DELETE }

trigger_order: { FOLLOWS | PRECEDES } other_trigger_name
```
`BEFORE`和`AFTER`参数指定了触发执行的时间，在事件之前或是之后。
`FOR EACH ROW`表示任何一条记录上的操作满足触发事件都会触发该触发器，也就是说触发器的触发频率是针对每一行数据触发一次。
 `tigger_event`详解：
　　①`INSERT`型触发器：插入某一行时激活触发器，可能通过INSERT、LOAD DATA、REPLACE 语句触发(LOAD DAT语句用于将一个文件装入到一个数据表中，相当与一系列的INSERT操作)；
　　②`UPDATE`型触发器：更改某一行时激活触发器，可能通过UPDATE语句触发；
　　③`DELETE`型触发器：删除某一行时激活触发器，可能通过DELETE、REPLACE语句触发。
 `trigger_order`是MySQL5.7之后的一个功能，用于定义多个触发器，使用follows(尾随)或precedes(在…之先)来选择触发器执行的先后顺序。 

### 3.1 创建只有一个执行语句的触发器

```sql
CREATE TRIGGER 触发器名 BEFORE|AFTER 触发事件 ON 表名 FOR EACH ROW 执行语句;
```

例1：创建了一个名为trig1的触发器，一旦在work表中有插入动作，就会自动往time表里插入当前时间

```sql
mysql> CREATE TRIGGER trig1 AFTER INSERT
    -> ON work FOR EACH ROW
    -> INSERT INTO time VALUES(NOW());
```
### 3.2 创建有多个执行语句的触发器

```sql
CREATE TRIGGER 触发器名 BEFORE|AFTER 触发事件
ON 表名 FOR EACH ROW
BEGIN
        执行语句列表
END;
```

例2：定义一个触发器，一旦有满足条件的删除操作，就会执行BEGIN和END中的语句

```sql
mysql> DELIMITER ||
mysql> CREATE TRIGGER trig2 BEFORE DELETE
    -> ON work FOR EACH ROW
    -> BEGIN
    -> 　　INSERT INTO time VALUES(NOW());
    -> 　　INSERT INTO time VALUES(NOW());
    -> END||
mysql> DELIMITER ;
```
### 3.3 NEW与OLD详解

MySQL 中定义了 NEW 和 OLD，用来表示触发器的所在表中，触发了触发器的那一行数据，来引用触发器中发生变化的记录内容，具体地：

 - ①在INSERT型触发器中，NEW用来表示将要（BEFORE）或已经（AFTER）插入的新数据；
 - ②在UPDATE型触发器中，OLD用来表示将要或已经被修改的原数据，NEW用来表示将要或已经修改为的新数据；
 - ③在DELETE型触发器中，OLD用来表示将要或已经被删除的原数据；

使用方法：

　　**`NEW.columnName （columnName为相应数据表某一列名）`**

另外，OLD是只读的，而NEW则可以在触发器中使用 SET 赋值，这样不会再次触发触发器，造成循环调用（如每插入一个学生前，都在其学号前加“2013”）。
例3：

```sql
mysql> CREATE TABLE account (acct_num INT, amount DECIMAL(10,2));
mysql> INSERT INTO account VALUES(137,14.98),(141,1937.50),(97,-100.00);

mysql> delimiter $$
mysql> CREATE TRIGGER upd_check BEFORE UPDATE ON account
    -> FOR EACH ROW
    -> BEGIN
    -> 　　IF NEW.amount < 0 THEN
    -> 　　　　SET NEW.amount = 0;
    -> 　　ELSEIF NEW.amount > 100 THEN
    -> 　　　　SET NEW.amount = 100;
    -> 　　END IF;
    -> END$$
mysql> delimiter ;

mysql> update account set amount=-10 where acct_num=137;

mysql> select * from account;
+----------+---------+
| acct_num | amount  |
+----------+---------+
|      137 |    0.00 |
|      141 | 1937.50 |
|       97 | -100.00 |
+----------+---------+

mysql> update account set amount=200 where acct_num=137;

mysql> select * from account;
+----------+---------+
| acct_num | amount  |
+----------+---------+
|      137 |  100.00 |
|      141 | 1937.50 |
|       97 | -100.00 |
+----------+---------+
```

## 4. 查看触发器

### 4.1 SHOW TRIGGERS语句查看触发器信息
```sql
mysql> SHOW TRIGGERS\G;
```
结果，显示所有触发器的基本信息；无法查询指定的触发器。
### 4.2 在information_schema.triggers表中查看触发器信息

```sql
mysql> SELECT * FROM information_schema.triggers\G
……
```
结果，显示所有触发器的详细信息；同时，该方法可以查询制定触发器的详细信息。

```sql
mysql> select * from information_schema.triggers 
    -> where trigger_name='upd_check'\G;
```

**Tips**：
　　所有触发器信息都存储在information_schema数据库下的triggers表中，可以使用SELECT语句查询，如果触发器信息过多，最好通过TRIGGER_NAME字段指定查询。

## 5. 删除触发器

```sql
DROP TRIGGER [IF EXISTS] [schema_name.]trigger_name
```
删除触发器之后最好使用上面的方法查看一遍；同时，也可以使用`database.trig`来指定某个数据库中的触发器。

**Tips：**
　　如果不需要某个触发器时一定要将这个触发器删除，以免造成意外操作，这很关键。

参考:

 - [https://www.cnblogs.com/geaozhang/p/6819648.html](https://www.cnblogs.com/geaozhang/p/6819648.html)

