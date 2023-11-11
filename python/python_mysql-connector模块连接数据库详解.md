#  python 模块 mysql-connector 管理数据库



----
## 1. 介绍
MySQL 是最流行的关系型数据库管理系统，mysql-connector 来连接使用 MySQL， mysql-connector 是 MySQL 官方提供的驱动器。
## 2. 安装

```bash
$  pip install mysql-connector
```
注意：如果你的 MySQL 是 8.0 版本，密码插件验证方式发生了变化，早期版本为 mysql_native_password，8.0 版本为 caching_sha2_password，所以需要做些改变：

先修改 my.ini 配置：

```bash
[mysqld]
default_authentication_plugin=mysql_native_password
```
然后在 mysql 下执行以下命令来修改密码：

```bash
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '新密码';
```
## 3. 用法
### 3.1 创建数据库连接

```python
import mysql.connector
 
mydb = mysql.connector.connect(
  host="192.168.1.190",
  user="root",
  passwd="Password01!"
)
print(mydb) 
```
输出：
```python
python my2.py
<mysql.connector.connection.MySQLConnection object at 0x7ff34aa24410>
```
### 3.2 创建数据库

```python
import mysql.connector
 
mydb = mysql.connector.connect(
  host="localhost",
  user="root",
  passwd="123456"
)
 
mycursor = mydb.cursor()
 
mycursor.execute("CREATE DATABASE runoob_db;")
```
### 3.3 查看数据库是否存在

```python
import mysql.connector
 
mydb = mysql.connector.connect(
  host="192.168.1.190",
  user="root",
  passwd="Password01!"
)
 
mycursor = mydb.cursor()
 
mycursor.execute("SHOW DATABASES;")
 
for x in mycursor:
  print(x)
```
输出：
```python
$ python my1.py
(u'information_schema',)
(u'api_server',)
(u'api_server_test',)
(u'horus',)
(u'mgm',)
(u'mysql',)
(u'mysqlmanager',)
(u'performance_schema',)
(u'redismanager',)
(u'sys',)
```
### 3.4 创建数据表

```python
import mysql.connector
 
mydb = mysql.connector.connect(
  host="localhost",
  user="root",
  passwd="123456",
  database="runoob_db"
)
mycursor = mydb.cursor()
 
mycursor.execute("CREATE TABLE sites (name VARCHAR(255), url VARCHAR(255))")
```
### 3.5 查看数据表是否已存在

```python
import mysql.connector
 
mydb = mysql.connector.connect(
  host="localhost",
  user="root",
  passwd="123456",
  database="runoob_db"
)
mycursor = mydb.cursor()
 
mycursor.execute("SHOW TABLES")
 
for x in mycursor:
  print(x)
```
### 3.6 主键设置
给 sites 表添加主键

```python
import mysql.connector
 
mydb = mysql.connector.connect(
  host="localhost",
  user="root",
  passwd="123456",
  database="runoob_db"
)
mycursor = mydb.cursor()
 
mycursor.execute("ALTER TABLE sites ADD COLUMN id INT AUTO_INCREMENT PRIMARY KEY")
```
给表创建主键

```python
import mysql.connector
 
mydb = mysql.connector.connect(
  host="localhost",
  user="root",
  passwd="123456",
  database="runoob_db"
)
mycursor = mydb.cursor()
 
mycursor.execute("CREATE TABLE sites (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255), url VARCHAR(255))")
```
### 3.7 插入数据

```python
import mysql.connector
 
mydb = mysql.connector.connect(
  host="localhost",
  user="root",
  passwd="123456",
  database="runoob_db"
)
mycursor = mydb.cursor()
 
sql = "INSERT INTO sites (name, url) VALUES (%s, %s)"
val = ("RUNOOB", "https://www.runoob.com")
mycursor.execute(sql, val)
 
mydb.commit()    # 数据表内容有更新，必须使用到该语句
 
print(mycursor.rowcount, "记录插入成功。")
```
### 3.8 批量插入
批量插入使用 executemany() 方法

```python
import mysql.connector
 
mydb = mysql.connector.connect(
  host="localhost",
  user="root",
  passwd="123456",
  database="runoob_db"
)
mycursor = mydb.cursor()
 
sql = "INSERT INTO sites (name, url) VALUES (%s, %s)"
val = [
  ('Google', 'https://www.google.com'),
  ('Github', 'https://www.github.com'),
  ('Taobao', 'https://www.taobao.com'),
  ('stackoverflow', 'https://www.stackoverflow.com/')
]
 
mycursor.executemany(sql, val)
 
mydb.commit()    # 数据表内容有更新，必须使用到该语句
 
print(mycursor.rowcount, "记录插入成功。")
print("1 条记录已插入, ID:", mycursor.lastrowid)
```
### 3.9  查询数据
#### 3.9.1 获取所有记录
```python
import mysql.connector
 
mydb = mysql.connector.connect(
  host="localhost",
  user="root",
  passwd="123456",
  database="runoob_db"
)
mycursor = mydb.cursor()
 
mycursor.execute("SELECT * FROM sites")
 
myresult = mycursor.fetchall()     # fetchall() 获取所有记录
 
for x in myresult:
  print(x)
```

输出：

```python
(1, 'RUNOOB', 'https://www.runoob.com')
(2, 'Google', 'https://www.google.com')
(3, 'Github', 'https://www.github.com')
(4, 'Taobao', 'https://www.taobao.com')
(5, 'stackoverflow', 'https://www.stackoverflow.com/')
(6, 'Zhihu', 'https://www.zhihu.com')
```
####  3.9.2 只想读取一条数据，可以使用 fetchone() 方法

```python
import mysql.connector
 
mydb = mysql.connector.connect(
  host="localhost",
  user="root",
  passwd="123456",
  database="runoob_db"
)
mycursor = mydb.cursor()
 
mycursor.execute("SELECT * FROM sites")
 
myresult = mycursor.fetchone()
 
print(myresult)
```
输出结果为：

```python
(1, 'RUNOOB', 'https://www.runoob.com')
```

####  3.9.3 排序
查询结果排序可以使用 `ORDER BY` 语句，默认的排序方式为升序，关键字为 `ASC`，如果要设置降序排序，可以设置关键字 `DESC`。

```python
import mysql.connector
 
mydb = mysql.connector.connect(
  host="localhost",
  user="root",
  passwd="123456",
  database="runoob_db"
)
mycursor = mydb.cursor()
 
sql = "SELECT * FROM sites ORDER BY name"
#sql = "SELECT * FROM sites ORDER BY name DESC" #降序
#mycursor.execute("SELECT * FROM sites LIMIT 3") #通过 "LIMIT" 语句设置查询的数据量
#mycursor.execute("SELECT * FROM sites LIMIT 3 OFFSET 1")  使用的关键字是 OFFSET指定起始位置
mycursor.execute(sql)
 
myresult = mycursor.fetchall()
 
for x in myresult:
  print(x)
```



####  3.9.4 了防止数据库查询发生 **SQL 注入的攻击**，我们可以使用 %s 占位符来转义查询的条件。
升序
```python
import mysql.connector
 
mydb = mysql.connector.connect(
  host="localhost",
  user="root",
  passwd="123456",
  database="runoob_db"
)
mycursor = mydb.cursor()
 
sql = "SELECT * FROM sites WHERE name = %s"
na = ("RUNOOB", )
 
mycursor.execute(sql, na)
 
myresult = mycursor.fetchall()
 
for x in myresult:
  print(x)
```
输出结果为：

```python
(3, 'Github', 'https://www.github.com')
(2, 'Google', 'https://www.google.com')
(1, 'RUNOOB', 'https://www.runoob.com')
(5, 'stackoverflow', 'https://www.stackoverflow.com/')
(4, 'Taobao', 'https://www.taobao.com')
(6, 'Zhihu', 'https://www.zhihu.com')
```
### 3.10 删除记录

```python
import mysql.connector
 
mydb = mysql.connector.connect(
  host="localhost",
  user="root",
  passwd="123456",
  database="runoob_db"
)
mycursor = mydb.cursor()
 
sql = "DELETE FROM sites WHERE name = 'stackoverflow'"
 
mycursor.execute(sql)
 
mydb.commit()
 
print(mycursor.rowcount, " 条记录删除")
```
为了防止数据库查询发生 SQL 注入的攻击，我们可以使用 %s 占位符来转义删除语句的条件

```python
import mysql.connector
 
mydb = mysql.connector.connect(
  host="localhost",
  user="root",
  passwd="123456",
  database="runoob_db"
)
mycursor = mydb.cursor()
 
sql = "DELETE FROM sites WHERE name = %s"
na = ("stackoverflow", )
 
mycursor.execute(sql, na)
 
mydb.commit()
 
print(mycursor.rowcount, " 条记录删除")
```
###  3.11 更新表数据

```python
import mysql.connector
 
mydb = mysql.connector.connect(
  host="localhost",
  user="root",
  passwd="123456",
  database="runoob_db"
)
mycursor = mydb.cursor()
 
sql = "UPDATE sites SET name = 'ZH' WHERE name = 'Zhihu'"
 
mycursor.execute(sql)
 
mydb.commit()
 
print(mycursor.rowcount, " 条记录被修改")
```
输出结果为：

```python
1  条记录被修改
```
为了防止数据库查询发生 SQL 注入的攻击，我们可以使用 %s 占位符来转义更新语句的条件

```python
import mysql.connector
 
mydb = mysql.connector.connect(
  host="localhost",
  user="root",
  passwd="123456",
  database="runoob_db"
)
mycursor = mydb.cursor()
 
sql = "UPDATE sites SET name = %s WHERE name = %s"
val = ("Zhihu", "ZH")
 
mycursor.execute(sql, val)
 
mydb.commit()
 
print(mycursor.rowcount, " 条记录被修改")
```

输出结果为：

```python
1  条记录被修改
```

### 3.12 删除表

```python
import mysql.connector
 
mydb = mysql.connector.connect(
  host="localhost",
  user="root",
  passwd="123456",
  database="runoob_db"
)
mycursor = mydb.cursor()
 
sql = "DROP TABLE IF EXISTS sites"  # 删除数据表 sites
 
mycursor.execute(sql)
```
参考：

 - [Python MySQL - mysql-connector 驱动](https://www.runoob.com/python3/python-mysql-connector.html)
 - [MySQL Connector/Python Developer Guide](https://dev.mysql.com/doc/connector-python/en/)


