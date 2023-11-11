#  windows python web flask 模板开发快速入门
tags: flask
<!--  catalog: ~flask 快速入门~ -->

![在这里插入图片描述](https://img-blog.csdnimg.cn/fb9f7bceb87645e1a50dc1a631c535a4.png)




---

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200621185646962.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200621185702473.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 1. 简介
模板是一个包含响应文本的文件，其中个包含用站位变量表示的动态部分，其具体值只在请求的上下文中才能知道。

渲染是一个使用真实值替换变量，在返回最终得到的响应字符串的过程。

Jinja2 是一个强大模板引擎用来渲染模板。

## 2. 语法
利用jinja2这样的模板引擎，我们可以将一部分的程序逻辑放到模板中去。简单的说，我们可以在模板中使用python语句和表达式来操作数据的输出。但需要注意的是，jinja2并不支持所有的python语法，并且出于效率和代码组织方面的考虑，我们应适度的使用模板，仅把和输出控制有关的逻辑放到模板中。

jinja2中允许在模板中使用大部分python对象，比如字符串，列表，字典，元组，整型，浮点型，布尔值。它支持基本的运算符（+，-，*，/等），比较符号（==，!=）,逻辑符号（and,or,not和括号）以及in，is，None和布尔值（True，False）

 - {{ ... }} 用来标记变量。
 - {% ... %} 用来标记语句，比如 if 语句，for 语句等。
 - {# ... #} 用来写注释。

## 3. 方法
### 3.1 模板渲染
用到三个文件
**templates\index.html**

```csharp
<h1>
    Hello World
</h1>
```

**templates\user.html**

```csharp
{# {{ name }} 表示一种变量，模板中的占位符 #}
<h1>Hello,{{ name }}!</h1>
```
**hello.py**

```python
from flask import Flask,render_template


# 1.初始化 Flask 对象
app = Flask(__name__)
# manager = Manager(app)



# 2. 设置路由和视图函数
@app.route('/')
def index():
    # return '<h1>Hello World!<h1>'
    return render_template('index.html')


@app.route('/user/<name>')
def user(name):
    # return '<h1>Hello,%s!<h1>' % name
    return render_template('user.html',name=name)


# 3.启动服务
if __name__ == '__main__':
    app.run(debug=True)
    # manager.run()
```
测试：

```python
$ python hello.py
* Serving Flask app "hello" (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: on
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 175-682-414
```

```python
$ curl http://127.0.0.1:5000
<h1>
    Hello World
</h1>
$ curl http://127.0.0.1:5000/user/liming
<h1>Hello,liming!</h1>
```
### 3.2 变量使用
Jinja2 能识别所有类型的变量,甚至是一些复杂的类型,例如列表、字典和对象。

```python
<p>从字典中获取一个值： {{ mydict['key'] }}.</p>
<p>从列表中获取一个值： {{ mylist[3] }}.</p>
<p>通过一个变量索引，从列表中获取一个值： {{ mylist[myintvar] }}.</p>
<p>从对象的方法中获取一个值： {{ myobj.somemethod() }}.</p>
```
在模板中，jinja2支持使用”.”获取变量的属性，比如user字典中的username键值通过”.”获取即可。比如，`user.username`等价于`user[username]`
Jinja2 可以使用过滤器修改变量，过滤器名添加在变量名之后，中间使用竖线分割。


语法：
```python
{{ 变量|过滤器 }}
```

例如，下述模板以首字母大写形式显示变量 name 的值：

```python
Hello,{{ name|capitalize }}
```

Jinja2 变量过滤器
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200717180121882.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
`safe` 过滤器最值得一提。出于安全考虑，默认情况下，`Jinja2` 会把所有变量的值进行转义。比如，如果变量的值为 `<h1>Hello World!</h1>`，Jinja2 会在渲染模板时，将其转义为：`&lt;h1&gt;Hello World!&lt;/h1&gt;`。完整的过滤器列表，请参见[官方文档](https://jinja.palletsprojects.com/en/2.10.x/templates/#builtin-filters)

### 3.3 控制结构
#### 3.3.1 条件控制语句

```python
{% if user %}
    Hello,{{ user }}！
{% else %}
    Hello,Stranger!
{% endif %}
```
#### 3.3.2 for 循环

```python
<ul>
    {% for comment in comments %}
        <li>{{ comment }}</li>
    {% endfor %}

</ul>
```

```python
HTML <ul> 标签**无序 HTML 列表**
```

在for循环内，jinja2提供了多个特殊变量，常用的for循环变量如下：
| 变量名                | 作用              |
|--------------------|-----------------|
|                    |                 |
| loop\.index        | 当前迭代数，从1开始计数    |
|                    |                 |
| loop\.index\(\)    | 当前迭代数，从0开始计数    |
|                    |                 |
| loop\.revindex     | 当前反向迭代数，从1开始计数  |
|                    |                 |
| loop\.revindex\(\) | 当前反向迭代数，从0开始计数  |
|                    |                 |
| loop\.first        | 如果是第一个元素则为True  |
|                    |                 |
| loop\.last         | 如果是最后一个元素则为True |
|                    |                 |
| loop\.previtem     | 上一个迭代的条目        |
|                    |                 |
| loop\.nextitem     | 下一个迭代的条目        |
|                    |                 |
| loop\.length       | 序列中元素的数量        |

### 3.3.3  Jinja2 还支持宏 。宏类似 python 代码中的函数

```python
{% macro render_comment(comment) %}
    <li>{{ render_comment(comment) }}</li>
{% endmacro %}
<ul>
    {% for comment in comments %}
        {{ render_comment(comment) }}
    {% endfor %}

</ul>
```
#### 3.3.4 import模板中导入
为了重复使用宏，我们可以将其保存在单独的文件中，然后在需要使用的模板中导入：

```python
{% import 'macros.html' as macros %}
<ul>
    {% for comment in comments %}
        {{ macros.render_comment(comment) }}
    {% endfor %}
</ul>
```
### 3.3.5 include模板代码片段可以写入单独的文件
需要在多处重复使用的模板代码片段可以写入单独的文件，再包含在所有模板中，以避免重复：

```python
{% include 'comment.html' %}
```
另一种重复使用代码的强大方式是模板继承，它类似于 Python 代码中的类继承。首先，创 \建一个名为 base.html 的基模板：

```python
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>{% block title %}{% endblock %} - My Application </title>
    {% endblock %}
</head>
<body>
    {% block body %}
    {% endblock %}
</body>
</html>
```
### 3.3.6 block标签定义的元素可在extends衍生模板中修改
`block` 标签定义的元素可在衍生模板中修改。在本例中，我们定义了名为 `head`、`title` 和 `body` 的块。注意，title 包含在 head 中。下面这个示例是基模板的衍生模板：

```python
{% extends "base.html" %}
{% block title %}Index{% endblock %}
{% block head %}
    {{ super() }}
    <style>

    </style>
{% endblock %}
{% block body %}
    <h1>Hello,World</h1>
{% endblock %}
```
`extends` 指令声明这个模板衍生自 `base.html`。在 extends 指令之后，基模板中的 `3 个块`被重新定义，模板引擎会将其插入适当的位置。注意新定义的 head 块，在基模板中其内容不是空的，所以使用 `super() 获取原来的内容`。


## 4 实战
### 4.1 for循环、if判断方法、变量的搭配
app.py

```python
# encoding=utf-8
from flask import Flask, render_template
app = Flask(__name__)
 
 
user = {"username": "xiaoxiao",
        "bio": "A girl who loves movies."}
 
movies = [
    {'name': 'My Neighbor Totoro','year':'1988'},
    {'name': 'Three Colours trilogy', 'year': '1993'},
    {'name': 'Forrest Gump', 'year': '1994'},
    {'name': 'Perfect Blue', 'year': '1997'},
    {'name': 'The Matrix', 'year': '1999'},
    {'name': 'Memento', 'year': '2000'},
    {'name': 'The Bucket list', 'year': '2007'},
    {'name': 'Black Swan', 'year': '2010'},
    {'name': 'Gone Girl', 'year': '2014'},
    {'name': 'CoCo', 'year': '2017'}
]
 
 
@app.route('/hi')
def hi():
    return "hello flask!"
 
 
@app.route("/watchlist")
def watchlist():
    return render_template("watchlist.html", user=user, movies=movies)
 
 
if __name__ == "__main__":
    app.run('0.0.0.0',debug=True)
```
templates/watchlist.html

```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{{ user.username }}'s watchlist</title>
</head>
<body>
<a href="{{ url_for('hi') }}">← Return</a>
<h2>{{ user.username}}</h2>
{% if user.bio %}
    <i>{{ user.bio }}</i>
{% else %}
    <i>This user has not provided a bio.</i>
{% endif %}
{# 以下是电影清单（这是注释） #}
<h5>{{ user.username }}'s watchlist ({{ movies|length }}):</h5>
<ul>
    {% for movie in movies %}
        <li>{{ movie.name }} - {{ movie.year }}</li>
    {% endfor %}
</ul>
</body>
</html>
```
执行：

```python
$ python app.py 
 * Serving Flask app "app" (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: on
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 175-682-414
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200717184006610.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
HTML文件说明：

**模板中使用←是HTML的实体，HTML实体除了用来转义HTML保留符号外，通常会被用来显示不容易通过键盘输入的字符。这里的←会显示为左箭头，另外，©用来显示版权标志。**

在模板中添加python语句和表达式时，需要使用特定的定界符将其标识。

### 4.2 测试for循环中的loop变量，修改`watchlist.html`

```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{{ user.username }}'s watchlist</title>
</head>
<body>
<a href="{{ url_for('hi') }}">← Return</a>
<h2>{{ user.username}}</h2>
{% if user.bio %}
    <i>{{ user.bio }}</i>
{% else %}
    <i>This user has not provided a bio.</i>
{% endif %}
{# 以下是电影清单（这是注释） #}
<h5>{{ user.username }}'s watchlist ({{ movies|length }}):</h5>
<ul>
    {% for movie in movies %}
        <li>loop.index:{{ loop.index }}  loop.first:{{ loop.first }}
         loop.last:{{ loop.last }} - {{ movie.name }} - {{ movie.year }}</li>
    {% endfor %}
</ul>
</body>
</html>
```
效果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020071718554689.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
### 4.3 render_template_string()函数的使用

**app.py**

```python
# encoding=utf-8
from flask import Flask, render_template_string
app = Flask(__name__)
 
 
user = {"username": "xiaoxiao",
         "bio": "A girl who loves movies."}
 
movies = [
    {'name' : 'My Neighbor Totoro','year':'1988'},
    {'name': 'Three Colours trilogy', 'year': '1993'},
    {'name': 'Forrest Gump', 'year': '1994'},
    {'name': 'Perfect Blue', 'year': '1997'},
    {'name': 'The Matrix', 'year': '1999'},
    {'name': 'Memento', 'year': '2000'},
    {'name': 'The Bucket list', 'year': '2007'},
    {'name': 'Black Swan', 'year': '2010'},
    {'name': 'Gone Girl', 'year': '2014'},
    {'name': 'CoCo', 'year': '2017'}
]
 
 
@app.route('/hi')
def hi():
    return "hello flask!"
 
 
@app.route("/watchlist")
def watchlist():
    return render_template_string(
        """{% for movie in movies %}
            <li>{{ movie.name }} - {{ movie.year }}</li>
            {% endfor %}
        """, movies=movies)
 
 
if __name__ == "__main__":
    app.run(debug=True)
```
测试结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200717190615812.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
