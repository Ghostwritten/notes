# windows python flask与mysql数据库写入查询
tags: flask
<!--  catalog: ~flask mysql 增删改查~ -->



![在这里插入图片描述](https://img-blog.csdnimg.cn/2e08a19e3e8b495ea858999591dc9edb.png)


mysql客户端：
[sqlyog下载](https://blog.csdn.net/qq_26442553/article/details/80044909)

### 1. 创建一个数据库

#### 1.1 docker创建mysql
```bash
$ docker run --name first-mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d docker.io/mysql:5.7.22
```
#### 1.2 windows创建mysql
[参考链接](https://blog.csdn.net/chenriyang0306/article/details/54587034)

### 2. 创建并初始化一个库
连接数据库
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200718170425152.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
创建一个数据库mysqltest
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200718170530575.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
创建一个表
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200718170703524.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
添加一个user表
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200718170732269.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
### 2.1 查询并可以添加数据库信息的脚本

```python
import pymysql


def get_conn():
    return pymysql.connect(
        host='192.168.211.15',
        user='root',
        password='123456',
        database='mysqltest',
        charset='utf8'
    )

def query_data(sql):
    conn = get_conn()
    try:
        cursor = conn.cursor(pymysql.cursors.DictCursor)
        cursor.execute(sql)
        return cursor.fetchall()
    finally:
        conn.close()


def insert_or_update_data(sql):
    conn = get_conn()
    try:
        cursor = conn.cursor()
        cursor.execute(sql)
        conn.commit()
    finally:
        conn.close()

if __name__ == '__main__':
    sql = "insert user (name, sex, age, email) values('xiaoli', 'mam', 30, 'xiaoli@qq.com');"
    insert_or_update_data(sql)  #插入一个用户信息
    sql = "select * from user;"
    datas = query_data(sql)  # 查询一个信息
    import pprint
    pprint.pprint(datas)
```
执行结果：

```python
[{'age': 12,
  'email': '1234@163.com',
  'id': 1,
  'name': 'xiaoming',
  'sex': 'man'},
 {'age': 15,
  'email': 'abc@163.com',
  'id': 2,
  'name': 'xiaohong',
  'sex': 'women'},
 {'age': 18,
  'email': 'sgsgsgs@163.com',
  'id': 3,
  'name': 'ligang',
  'sex': 'man'},
 {'age': 20,
  'email': 'sgsgs@gmail.com',
  'id': 4,
  'name': 'zhaoming',
  'sex': 'man'},
 {'age': 30, 'email': 'xiaoli@qq.com', 'id': 5, 'name': 'xiaoli', 'sex': 'mam'}]
```
查看数据库user表
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200718171159625.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
### 2.2 提交页面表单信息到数据库
#### 2.2.1 app.py
```python
from flask import Flask, render_template, request
import json
import db
app = Flask(__name__)
@app.route("/show_add_user")
def show_add_user():
    return render_template("show_add_user.html")

@app.route("/do_add_user", methods=['POST'])
def do_add_user():
    print(request.form)
    name = request.form.get("name")
    sex = request.form.get("sex")
    age = request.form.get("age")
    email = request.form.get("email")
    sql = f"""            
        insert into user (name, sex, age, email) values ('{name}', '{sex}', '{age}', '{email}')
        """                 #'f'支持 '{变量}'格式
    print(sql)
    db.insert_or_update_data(sql)  #执行插入数据函数
    return "sucess"

if __name__ == '__main__':
    app.run(host='192.168.1.4',debug=True)
```

#### 2.2.2 db.py

```python
import pymysql


def get_conn():
    return pymysql.connect(
        host='192.168.211.15',
        user='root',
        password='123456',
        database='mysqltest',
        charset='utf8'
    )

def query_data(sql):
    conn = get_conn()
    try:
        cursor = conn.cursor(pymysql.cursors.DictCursor)
        cursor.execute(sql)
        return cursor.fetchall()
    finally:
        conn.close()


def insert_or_update_data(sql):
    conn = get_conn()
    try:
        cursor = conn.cursor()
        cursor.execute(sql)
        conn.commit()
    finally:
        conn.close()

if __name__ == '__main__':
    sql = "insert user (name, sex, age, email) values('xiaoli', 'mam', 30, 'xiaoli@qq.com');"
    insert_or_update_data(sql)
    sql = "select * from user;"
    datas = query_data(sql)
    import pprint
    pprint.pprint(datas)
```
#### 2.2.3 templates/show_add_user.html

```bash
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>提交用户信息</title>
</head>
<body>
<div>
    <h1>提交用户信息：</h1>
    <form action="/do_add_user" method="post">  #执行/do_add_user路由函数
    <table border="1">
        <tr>
            <th>用户名</th>
            <th><input type="text" name="name"> </th>
        </tr>
        <tr>
            <th>性别</th>
            <th><input type="text" name="sex"> </th>
        </tr>
        <tr>
            <th>年龄</th>
            <th><input type="text" name="age"> </th>
        </tr>
        <tr>
            <th>邮箱</th>
            <th><input type="text" name="email"> </th>
        </tr>
        <tr>
            <th>提交</th>
            <th><input type="submit" name="submit" value="提交"> </th>
        </tr>
    </table>
    </form>
</div>

</body>
</html>
```
执行后，界面访问`http://ip:5000/show_add_user`
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020071817370880.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
输入用户信息，
提交后返回成功
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200718173812280.png)
查看数据库user表信息，添加成功。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200718173900902.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

更多阅读：

 - [linux python web flask Hello World实战](https://blog.csdn.net/xixihahalelehehe/article/details/106111115)
 - [windows python web flask Hello World实战](https://ghostwritten.blog.csdn.net/article/details/106864137)
 - [windows python web flask 模板开放实战](https://ghostwritten.blog.csdn.net/article/details/106889489)
 - [windows python web flask获取请求参数数据](https://ghostwritten.blog.csdn.net/article/details/106888653)
 - [windows python flask返回json数据](https://ghostwritten.blog.csdn.net/article/details/107428589)
