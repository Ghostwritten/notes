#  python flask template 模板应用
tags: flask
<!--  catalog: ~flask template 模板应用 ~ -->

![在这里插入图片描述](https://img-blog.csdnimg.cn/e0487f22ea9b44d790f87882ef5afac6.png)




## 1. templates模板运用
前面介绍了创建一个[hello world项目](https://blog.csdn.net/xixihahalelehehe/article/details/106111115)，
下面介绍模板
格式:

```bash
user = {'username': 'Miguel'}
```
原先的视图函数返回简单的字符串，我现在要将其扩展为包含完整HTML页面元素的字符串，如下所示：

```bash
$ vim app/routes.py
from app import app

@app.route('/')
@app.route('/index')
def index():
    user = {'username': 'Miguel'}
    return '''
<html>
    <head>
        <title>Home Page - Microblog</title>
    </head>
    <body>
        <h1>Hello, ''' + user['username'] + '''!</h1>
    </body>
</html>'''
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200514144236981.png)
板有助于实现页面展现和业务逻辑之间的分离。 在Flask中，模板被编写为单独的文件，存储在应用程序包内的templates文件夹中。 在确定你在microblog目录后，创建一个存储模板的目录：

```bash
(venv) $ mkdir app/templates
```
在下面可以看到你的第一个模板，它的功能与上面的index()视图函数返回的HTML页面相似。 把这个文件写在`app/templates/index.html`中：

```bash
<html>
    <head>
        <title>{{ title }} - Microblog</title>
    </head>
    <body>
        <h1>Hello, {{ user.username }}!</h1>
    </body>
</html>
```
网页渲染转移到HTML模板之后，视图函数就能被简化：

```bash
from flask import render_template
from app import app

@app.route('/')
@app.route('/index')
def index():
    user = {'username': 'Miguel'}
    return render_template('index.html', title='Home', user=user)
```
将模板转换为完整的HTML页面的操作称为渲染。 为了渲染模板，需要从`Flask`框架中导入一个名为`render_template()`的函数。 该函数需要传入模板文件名和模板参数的变量列表，并返回模板中所有占位符都用实际变量值替换后的字符串结果。

render_template()函数调用Flask框架原生依赖的Jinja2模板引擎。 Jinja2用render_template()函数传入的参数中的相应值替换{{...}}块。

## 2. 条件语句
在渲染过程中使用实际值替换占位符，只是Jinja2在模板文件中支持的诸多强大操作之一。 模板也支持在{%...％}块内使用控制语句。 index.html模板的下一个版本添加了一个条件语句：

```bash
<html>
    <head>
        {% if title %}
        <title>{{ title }} - Microblog</title>
        {% else %}
        <title>Welcome to Microblog!</title>
        {% endif %}
    </head>
    <body>
        <h1>Hello, {{ user.username }}!</h1>
    </body>
</html>
```
## 3. 循环
使用模拟对象的把戏来创建一些模拟用户和动态：

```bash
$ vim app/routes.py
from flask import render_template
from app import app

@app.route('/')
@app.route('/index')
def index():
    user = {'username': 'Miguel'}
    posts = [
        {
            'author': {'username': 'John'},
            'body': 'Beautiful day in Portland!'
        },
        {
            'author': {'username': 'Susan'},
            'body': 'The Avengers movie was so cool!'
        }
    ]
    return render_template('index.html', title='Home', user=user, posts=posts)
```
用户动态列表拥有的元素数量由视图函数决定。 那么模板不能对有多少个用户动态进行任何假设，因此需要准备好以通用方式渲染任意数量的用户动态。
Jinja2提供了for控制结构来应对这类问题：

```bash
$ vim app/templates/index.html
```
```bash
<html>
    <head>
        {% if title %}
        <title>{{ title }} - Microblog</title>
        {% else %}
        <title>Welcome to Microblog</title>
        {% endif %}
    </head>
    <body>
        <h1>Hi, {{ user.username }}!</h1>
        {% for post in posts %}
        <div><p>{{ post.author.username }} says: <b>{{ post.body }}</b></p></div>
        {% endfor %}
    </body>
</html>
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200514161548771.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 4. 模板的继承
绝大多数Web应用程序在页面的顶部都有一个**导航栏**，其中带有一些**常用的链接**，例如**编辑配置文件，登录，注销**等。我可以轻松地用HTML标记语言将导航栏添加到`index.html`模板上，但随着应用程序的增长，我将需要在其他页面重复同样的工作。尽量不要编写重复的代码，这是一个良好的编程习惯，毕竟我真的不想在诸多HTML模板上保留同样的代码。

Jinja2有一个模板继承特性，专门解决这个问题。从本质上来讲，**就是将所有模板中相同的部分转移到一个基础模板中，然后再从它继承过来。**

所以我现在要做的是定义一个名为`base.html`的基本模板，其中包含一个简单的导航栏，以及我之前实现的标题逻辑。 您需要在模板文件`app/templates/base.html`中编写代码如下：

```bash
<html>
    <head>
      {% if title %}
      <title>{{ title }} - Microblog</title>
      {% else %}
      <title>Welcome to Microblog</title>
      {% endif %}
    </head>
   <body>
        <div>Microblog: <a href="/index">Home</a></div>
        <hr>
        {% block content %}{% endblock %}
    </body>
</html>
```

在这个模板中，我使用block控制语句来定义派生模板可以插入代码的位置。 block被赋予一个唯一的名称，派生的模板可以在提供其内容时进行引用。

通过从基础模板base.html继承HTML元素，我现在可以简化模板`index.html`了：

```bash
{% extends "base.html" %}

{% block content %}
    <h1>Hi, {{ user.username }}!</h1>
    {% for post in posts %}
    <div><p>{{ post.author.username }} says: <b>{{ post.body }}</b></p></div>
    {% endfor %}
{% endblock %}
```

自从基础模板base.html接手页面的布局之后，我就可以从index.html中删除所有这方面的元素，只留下内容部分。 extends语句用来建立了两个模板之间的继承关系，这样Jinja2才知道当要求呈现index.html时，需要将其嵌入到base.html中。 而两个模板中匹配的block语句和其名称content，让Jinja2知道如何将这两个模板合并成在一起。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200514162304931.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
参考：
- [The-Flask-Mega-Tutorial-zh第二章](https://github.com/luhuisicnu/The-Flask-Mega-Tutorial-zh/blob/master/docs/%E7%AC%AC%E4%BA%8C%E7%AB%A0%EF%BC%9A%E6%A8%A1%E6%9D%BF.md)


更多阅读：

 - [linux python web flask Hello World实战](https://blog.csdn.net/xixihahalelehehe/article/details/106111115)
 - [windows python web flask Hello World实战](https://ghostwritten.blog.csdn.net/article/details/106864137)
 - [windows python web flask 模板实战](https://ghostwritten.blog.csdn.net/article/details/106889489)
 - [windows python web flask获取请求参数数据](https://ghostwritten.blog.csdn.net/article/details/106888653)
 - [windows python flask返回json数据](https://ghostwritten.blog.csdn.net/article/details/107428589)
 - [windows python flask与mysql数据库写入查询显示等操作详解](https://ghostwritten.blog.csdn.net/article/details/107431748)
 - [python flask-caching模块缓存详解](https://ghostwritten.blog.csdn.net/article/details/107235464)


