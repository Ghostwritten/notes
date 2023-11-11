#  Python3 模块 PyMySQL 管理数据库



---
## 1. 介绍
[PyMySQL](https://pypi.org/project/PyMySQL/) 是在 Python3.x 版本中用于连接 MySQL 服务器的一个库，Python2中则使用mysqldb。

PyMySQL 遵循 Python 数据库 API v2.0 规范，并包含了 pure-Python MySQL 客户端库。
## 2. 安装
在使用 PyMySQL 之前，我们需要确保 PyMySQL 已安装。

PyMySQL 下载地址：https://github.com/PyMySQL/PyMySQL。

如果还未安装，我们可以使用以下命令安装最新版的 PyMySQL：
1.
```bash
$ pip3 install PyMySQL
```
2.使用 git 命令下载安装包安装(你也可以手动下载)：

```bash
$ git clone https://github.com/PyMySQL/PyMySQL
$ cd PyMySQL/
$ python3 setup.py install
```
3.如果需要制定版本号，可以使用 curl 命令来安装：

```bash
$ # X.X 为 PyMySQL 的版本号
$ curl -L https://github.com/PyMySQL/PyMySQL/tarball/pymysql-X.X | tar xz
$ cd PyMySQL*
$ python3 setup.py install
$ # 现在你可以删除 PyMySQL* 目录
```
注意：
请确保您有root权限来安装上述模块。

安装的过程中可能会出现`"ImportError: No module named setuptools"`的错误提示，意思是你没有安装`setuptools`，你可以访问`https://pypi.python.org/pypi/setuptools` 找到各个系统的安装方法。

Linux 系统安装实例：

```bash
$ wget https://bootstrap.pypa.io/ez_setup.py
$ python3 ez_setup.py
```

## 3. 语法
### 3.1 格式
```python
import pymysql
 
conn = pymysql.connect(host=None, user=None, password="",
                 database=None, port=0, unix_socket=None,
                 charset=''.....

 - List item

.)
```
参数：
 - host：表示连接的数据库的地址
 - user：表示连接使用的用户
 - password：表示用户对应的密码
 - database：表示连接哪个库
 - port：表示数据库的端口
 - unix_socket：表示使用socket连接时，socket文件的路径
 - charset：表示连接使用的字符集
 - read_default_file：读取mysql的配置文件中的配置进行连接

**注意：**

 - 端口不能加引号，因为port接受的数据类型为整型
 - charset的字符集不是utf-8，是utf8

### 3.2 连接方法
调用connect函数，将创建一个数据库连接并得到一个Connection对象，Connection对象定义了很多的方法和异常。

 - begin：开始事务
 - commit：提交事务
 - rollback：回滚事务
 - cursor：返回一个Cursor对象
 - autocommit：设置事务是否自动提交
 - set_character_set：设置字符集编码
 - get_server_info：获取数据库版本信息

在实际的编程过程中，一般不会直接调用begin、commit和rollback函数，而是通过上下文管理器实现事务的提交与回滚操作。

### 3.3 游标
#### 3.3.1 游标介绍
游标是系统为用户开设的一个数据缓存区，存放SQL语句执行的结果，用户可以用SQL语句逐一从游标中获取记录，并赋值给变量，交由Python进一步处理。
在数据库中，游标是一个十分重要的概念。游标提供了一种对从表中检索出的数据进行操作的灵活手段，就本质而言，游标实际上是一种能从包括多条数据记录的结果集中每次提取一条记录的机制。游标总是与一条SQL 选择语句相关联因为游标由结果集（可以是零条、一条或由相关的选择语句检索出的多条记录）和结果集中指向特定记录的游标位置组成。当决定对结果集进行处理时，必须声明一个指向该结果集的游标。
正如前面我们使用Python对文件进行处理，那么游标就像我们打开文件所得到的文件句柄一样，只要文件打开成功，该文件句柄就可代表该文件。对于游标而言，其道理是相同的。

#### 3.3.2 利用游标操作数据库
在进行数据库的操作之前需要创建一个游标对象，来执行sql语句。

```python
import pymysql
 
 
def connect_mysql():
 
    db_config = {
        'host':'192.168.1.190',
        'port':3306,
        'user':'root',
        'password':'Password01!',
        'charset':'utf8'
    }
 
    conn = pymysql.connect(**db_config)
 
    return conn
 
 
if __name__ == '__main__':
    conn = connect_mysql()
    cursor = conn.cursor()    # 创建游标
    sql = r" select user,host from mysql.user "  # 要执行的sql语句
    cursor.execute(sql)  # 交给 游标执行
    result = cursor.fetchall()  # 获取游标执行结果
    print(result)
```

输出：

```powershell
$ python3.8 pymysql3.py
(('api_server', '%'), ('horus', '%'), ('mgm', '%'), ('repl', '%'), ('root', '%'))
```
#### 3.3.3 游标的方法

 - cursor.fetchall()
 - cursor.fetchone()
 - cursor.fetchmany() 
 - cursor.execute(sql)
 - executemany(sql,parser)
 - corsor.rowncount 常量，表示sql语句的结果集中，返回了多少条记录
- cursor.arraysize 常量，保存了当前获取记录的下标
- corsor.close() 关闭游标

```python
# execute 执行单条sql语句
conn = connect_mysql()
cursor = conn.cursor()  # 创建游标
sql = r" select user,host from mysql.user "  # 要执行的sql语句
cursor.execute(sql)  # 交给 游标执行
     
 
# executemany 执行多条SQL语句 
conn = connect_mysql()
cursor = conn.cursor()
sql = 'insert into tmp(id) VALUES(%s)'     # %s 表示占位符
parser = [1,2,3,4,5,6]
cursor.execute('use test')
cursor.executemany(sql,parser)      # 把参数进行填充，parset是参数的列表
conn.commit()
conn.close()

conn = connect_mysql()
cursor = conn.cursor()    # 创建游标
sql = r" select user,host from mysql.user " 
cursor.execute(sql)  # 交给 游标执行
result1 = cursor.fetchone()
result2 = cursor.fetchmany(2)  # 获取2行
result3 = cursor.fetchall() 
 
print(result1)
print(result2)
print(result3)
```

#### 3.3.4 步骤

 1. 导入相应的Python模块
 2. 使用connect函数连接数据库，并返回一个Connection对象
 3. 通过Connection对象的cursor方法，返回一个Cursor对象
 4. 通过Cursor对象的execute方法执行SQL语句
 5. 如果执行的是查询语句，通过Cursor对象的fetchall语句获取返回结果
 6. 调用Cursor对象的close关闭Cursor
 7. 调用Connection对象的close方法关闭数据库连接

## 4 用法
### 4.1 数据库连接

```python
#!/usr/bin/python3
 
import pymysql
 
# 打开数据库连接
db = pymysql.connect("192.168.1.190","root","Password01!","horus" )
 
# 使用 cursor() 方法创建一个游标对象 cursor
cursor = db.cursor()
 
# 使用 execute()  方法执行 SQL 查询 
cursor.execute("SELECT VERSION();")
 
# 使用 fetchone() 方法获取单条数据.
data = cursor.fetchone()
 
print ("Database version : %s " % data)
 
# 关闭数据库连接
db.close()
```
### 4.2 创建数据库表

```python
#!/usr/bin/python3
 
import pymysql
 
# 打开数据库连接
db = pymysql.connect("localhost","testuser","test123","TESTDB" )
 
# 使用 cursor() 方法创建一个游标对象 cursor
cursor = db.cursor()
 
# 使用 execute() 方法执行 SQL，如果表存在则删除
cursor.execute("DROP TABLE IF EXISTS EMPLOYEE")
 
# 使用预处理语句创建表
sql = """CREATE TABLE EMPLOYEE (
         FIRST_NAME  CHAR(20) NOT NULL,
         LAST_NAME  CHAR(20),
         AGE INT,  
         SEX CHAR(1),
         INCOME FLOAT )"""
 
cursor.execute(sql)
 
# 关闭数据库连接
db.close()
```
### 4.3 数据库插入操作

```python
#!/usr/bin/python3
 
import pymysql
 
# 打开数据库连接
db = pymysql.connect("localhost","testuser","test123","TESTDB" )
 
# 使用cursor()方法获取操作游标 
cursor = db.cursor()
 
# SQL 插入语句
sql = """INSERT INTO EMPLOYEE(FIRST_NAME,
         LAST_NAME, AGE, SEX, INCOME)
         VALUES ('Mac', 'Mohan', 20, 'M', 2000)"""
try:
   # 执行sql语句
   cursor.execute(sql)
   # 提交到数据库执行
   db.commit()
except:
   # 如果发生错误则回滚
   db.rollback()
 
# 关闭数据库连接
db.close()
```
### 4.4 数据库查询操作

 - fetchone(): 该方法获取下一个查询结果集。结果集是一个对象
 - fetchall(): 接收全部的返回结果行.
 - rowcount: 这是一个只读属性，并返回执行execute()方法后影响的行数。
 
查询EMPLOYEE表中salary（工资）字段大于1000的所有数据：

```python
#!/usr/bin/python3
 
import pymysql
 
# 打开数据库连接
db = pymysql.connect("localhost","testuser","test123","TESTDB" )
 
# 使用cursor()方法获取操作游标 
cursor = db.cursor()
 
# SQL 查询语句
sql = "SELECT * FROM EMPLOYEE \
       WHERE INCOME > %s" % (1000)
try:
   # 执行SQL语句
   cursor.execute(sql)
   # 获取所有记录列表
   results = cursor.fetchall()
   for row in results:
      fname = row[0]
      lname = row[1]
      age = row[2]
      sex = row[3]
      income = row[4]
       # 打印结果
      print ("fname=%s,lname=%s,age=%s,sex=%s,income=%s" % \
             (fname, lname, age, sex, income ))
except:
   print ("Error: unable to fetch data")
 
# 关闭数据库连接
db.close()
```
以上脚本执行结果如下：

```python
fname=Mac, lname=Mohan, age=20, sex=M, income=2000
```
### 4.5 数据库更新操作

```python
#!/usr/bin/python3
 
import pymysql
 
# 打开数据库连接
db = pymysql.connect("localhost","testuser","test123","TESTDB" )
 
# 使用cursor()方法获取操作游标 
cursor = db.cursor()
 
# SQL 更新语句
sql = "UPDATE EMPLOYEE SET AGE = AGE + 1 WHERE SEX = '%c'" % ('M')
try:
   # 执行SQL语句
   cursor.execute(sql)
   # 提交到数据库执行
   db.commit()
except:
   # 发生错误时回滚
   db.rollback()
 
# 关闭数据库连接
db.close()
```
### 4.6 用with简化操作数据库

```python
import pymysql

class DB():
    def __init__(self, host='localhost', port=3306, db='', user='root', passwd='root', charset='utf8'):
        # 建立连接 
        self.conn = pymysql.connect(host=host, port=port, db=db, user=user, passwd=passwd, charset=charset)
        # 创建游标，操作设置为字典类型        
        self.cur = self.conn.cursor(cursor = pymysql.cursors.DictCursor)

    def __enter__(self):
        # 返回游标        
        return self.cur

    def __exit__(self, exc_type, exc_val, exc_tb):
        # 提交数据库并执行        
        self.conn.commit()
        # 关闭游标        
        self.cur.close()
        # 关闭数据库连接        
        self.conn.close()


if __name__ == '__main__':
    with DB(host='192.168.1.190',user='root',passwd='Password01!',db='horus') as db:
        db.execute('select * from tb_host;')
        print(db)
        for i in db:
            print(i)
```
参考：

 - [ython3 MySQL 数据库连接 - PyMySQL 驱动](https://www.runoob.com/python3/python3-mysql.html)
 - [Python学习笔记 - 11 - Python操作数据库](https://zhuanlan.zhihu.com/p/81132203)
 - [Connect to MySQL using PyMySQL in Python](https://www.geeksforgeeks.org/connect-to-mysql-using-pymysql-in-python/)
 - [Zetcode PyMySQL](https://zetcode.com/python/pymysql/)

