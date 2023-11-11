#  windows python web route 路由
tags: flask
<!--  catalog: ~flask route 路由~ -->

![在这里插入图片描述](https://img-blog.csdnimg.cn/892cb8baacd344799796956c23ea493c.png)



## 1. 路由介绍
路由是指用户请求的URL与视图函数之间的映射。Flask框架 根据HTTP请求的URL在路由表中匹配预定义的URL规则，找到对应的视图函数， 并将视图函数的执行结果返回WSGI服务器。
可见路由表在Flask应用中处于相当核心的位置。路由表的内容是由应用开发者填充。

route装饰器 ：可以使用Flask应用实例的route装饰器将一个URL规则绑定到 一个视图函数上。
## 2. 路由类型
### 2.1 一般路由url
第一种：没有名字的路由
```bash
@app.route('/')
def hello_world():
    return 'Welcome to Hello World !'
```
访问：http://127.0.0.1:5000/
第二种，定义一个名字

```python
@app.route('/hello')
def hello_world():
    return 'Welcome to Hello World !'
```
访问：http://127.0.0.1:5000/hello

第三种：一个函数多个url路由规则

```python
@app.route('/')
@app.route('/hello')
@app.route('/hello/<name>')

def hello(name=None):
    if name is None:
        name = 'World'
    return 'Hello %s' % name
```
访问：http://127.0.0.1:5000/
访问：http://127.0.0.1:5000/hello
访问：http://127.0.0.1:5000/hello/xiaoaming
访问：http://127.0.0.1:5000/ligang

第四种：`add_url_rule()`定义一个路由url
route装饰器内部也是通过调用add_url_rule()方法实现的路由注册

```python
def hello():
  return 'Welcome to Hello World !'
app.add_url_rule('/hello',view_func=hello)
```
访问：http://127.0.0.1:5000/hello
### 2.2 带参数的路由url

```python
@app.route('/hello/<name>')
def hello(name):
    return 'Hello %s' % name
```
name为随意字符串，结果输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200621163046431.png)
除了界面验证，我们也可以用命令行工具`ipython`测试：

```python
pip install ipython  #安装
ipython   #进入ipython

In [1]: import requests

In [2]: r = requests.get("http://192.168.1.4:5000/hello/zong")

In [3]: r.text
Out[3]: u'Hello zong'
```

参数是有类型的。默认是string
传递参数的语法是`/<参数类型:参数名称>/`，然后在视图函数中也要定义同名的参数
。

 - string：只接受字符串，没有任何“/或者”的文本
 - int：只接受整数
 - float：只接受浮点数，整数都不行哈
 - path：和string类似，但是接受斜杠
 - uuid：只有接受符合uuid的字符赤岸，一般用作表的主键
 - any：可以指定多种路径

示例：

```python
rom flask import Flask,request
app = Flask(__name__)
@app.route('/')
def hello_world():
    return 'Hello World!'
@app.route('/list/')
def article_list():
    return 'article list!'
@app.route('/p1/<article_id1>')
def article_detail(article_id1):
    return "请求的文章是：%s" %article_id1
@app.route('/p2/<string:article_id2>')
def article_detail2(article_id2):
    return "请求的文章是：%s" %article_id2
@app.route('/p3/<int:article_id3>')
def article_detail3(article_id3):
    return "请求的文章是：%s" %article_id3
@app.route('/p4/<path:article_id4>')
def article_detail4(article_id4):
    return "请求的文章是：%s" %article_id4
# import uuid
# print(uuid.uuid4())
@app.route('/p5/<uuid:article_id5>') #数据的唯一性，长度较长，有损效率（一般在用户表中使用）6a9221f6-afea-424a-a324-8ceaa5bdfc98
def article_detail5(article_id5):
    return "请求的文章是：%s" %article_id5
@app.route('/p6/<any(blog,user):url_path>/<id>/')
def detail(url_path,id):
    if url_path == "blog":
        return "博客详情 %s" %id
    else:
        return "用户详情 %s" %id
#通过问号形式传递参数
@app.route('/d/')
def d():
    wd = request.args.get('wd') #获取浏览器传递参数
    return '通过查询字符串的方式传递的参数是，%s'%wd #请求http://127.0.0.1:8080/d/?wd=php
if __name__ == '__main__':
    app.run()
```
### 2.3 带url_for传参的路由url

对url再次包装处理。

url_for的第一个参数是视图函数的函数名对应的字符串（endpoint），后面的参数就是你传递给url；如果传递的参数在url中已经定义了，那么这个参数就会被当成path的值传递给url；如果这个参数没有在url中定义，那么将变成查询字符串的形式。
语法格式：

```python
url_for('login')    # 返回/login
url_for('login', id='1')    # 将id作为URL参数，返回/login?id=1
url_for('hello', name='man')    # 适配hello函数的name参数，返回/hello/man
url_for('static', filename='style.css')    # 静态文件地址，返回/static/style.css
```
示例1
```python
from flask import Flask,url_for,request
@app.route('/')
    return url_for('my_list',page=1,count=2) #这样的话就会在页面上构建出/post/list/1/?count=2的信息
@app.route('/post/list/<page>/')
def my_list():
    return 'my list'
```

示例2:
```python
from flask import Flask,url_for,request
@app.route('/')
def hello_world():
    return url_for('login',next='/current') #页面返回/login/?next=%2Fcurrent登录前的信息
    # print(url_for('my_list',page=1,count=200))
    # return 'hello world'
@app.route('/login/')
def login():
    # next = request.args.get('next') #登录前的信息，在登陆之后仍旧保持
    return 'login'
@app.route('/list/<page>')
def my_list():
    return 'my list'
@app.route('/detail/<id>/')
def detail():
    return 'detail'
if __name__ == '__main__':
    app.run(debug=True)
```
### 2.4 不同http方法的路由url
当设置请求方式只能是`POST`时，`GET`就会报错。
```python
@app.route('/hello/<name>'，methods=["POST"])
def hello(name):
    return 'Hello %s' % name
```
测试：

```python
In [1]: import requests

In [2]: r = requests.get("http://192.168.1.4:5000/hello/zong")

In [3]: r.text
Out[6]: u'<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">\n<title>405 Method Not Allowed</title>\n<h1>Method Not Allowed</h1>\n<p>The method is not allowed for the requested URL.</p>\n'

In [7]: r.status_code
Out[7]: 405

In [8]: r = requests.post("http://192.168.1.4:5000/hello/zong")

In [9]: r.text
Out[9]: u'Hello zong'

In [10]: r.status_code
Out[10]: 200
```
如何想两种http方法都支持。
```python
from flask import request
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        return 'This is a POST request'
    else:
        return 'This is a GET request'
```

参考：
- [https://segmentfault.com/a/1190000014728092](https://segmentfault.com/a/1190000014728092)
