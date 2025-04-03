#  python flask 快速搭建 WEB 实战
tags: flask
<!--  catalog: ~flask 搭建 WEB 实战~ -->

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/056f3a82e9ee6081e428c552196a0dcd.png)





<font color=	#3CB371 size=4>视频</font>：[https://mp.weixin.qq.com/s/Az1LmhgYjURjsqJW4cBcLg](https://mp.weixin.qq.com/s/Az1LmhgYjURjsqJW4cBcLg)
<font color=	#FF4500 size=4>原创</font>：[https://www.youtube.com/watch?v=kng-mJJby8g](https://www.youtube.com/watch?v=kng-mJJby8g)
<font color=	#000000 size=4>github</font>：[https://github.com/Ghostwritten/flask_simple_demo.git](https://github.com/Ghostwritten/flask_simple_demo.git)



---


```bash
python -m pip install flask
mkdir flask_web1
cd flask_web1
mkdir flask_web1/template
mkdir flask_web1/static
touch app.py views.py
```
##  1. app.py配置首页
app.py
```bash
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "this is the home page"

if __name__=='__main__':
    app.run(debug=True, port=8080)
```
运行
访问：http://127.0.0.1:8080

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/216bc5460227775e2eb854e4060367db.png)
##  2. views.py配置首页
app.py

```bash
from flask import Flask
from views import views

app = Flask(__name__)

app.register_blueprint(views, url_prefix="/views")

if __name__=='__main__':
    app.run(debug=True, port=8080)
```

views.py

```bash
from flask import Blueprint

views = Blueprint(__name__, "views")


@views.route("/")
def home():
    return "home page"
```
运行app.py，此效果调用views.py的blueprint
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/49a2c36a06ffb8a4ccd32d32207aed31.png)

##  3. templates配置首页
```c
mkdir templates/index.html
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a0d2a958293e5c10a3a8f812171c7bcb.png)

自动创建html模板文件
```bash
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    
</body>
</html>
```
修改：

```bash
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Home</title>
</head>
<body>
    <div><h1>Home Page</h1></div>
</body>
</html>
```
views.py修改：

```bash
from flask import Blueprint, render_template

views = Blueprint(__name__, "views")


@views.route("/")
def home():
    return render_template("index.html")
```
app.py不变
运行：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4f2d3b162461a10b119de95fcdbfc9c3.png)
## 4. 设置变量
views.py

```bash
from flask import Blueprint, render_template

views = Blueprint(__name__, "views")


@views.route("/")
def home():
    return render_template("index.html",name="Tim",ago=21)
```
index.html

```bash
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Home</title>
</head>
<body>
    <div><h1>Home Page</h1></div>
    <p>hello, {{ name }}, you are {{ ago }} years old!</p>
</body>
</html>
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7c97a5f652245c6e2cdeddaf1f623e8f.png)

## 5. 接口变量

views.py

```bash
from flask import Blueprint, render_template

views = Blueprint(__name__, "views")


@views.route("/")
def home():
    return render_template("index.html",name="Tim")

@views.route("/profile/<username>")
def profile(username):
    return render_template("index.html",name=username)
```
index.html

```bash
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Home</title>
</head>
<body>
    <div><h1>Home Page</h1></div>
    <p>hello, {{ name }}</p>
</body>
</html>
```
运行：
死的用户
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3b5b9f415b4805171323bbf6c62d3fb3.png)
动态
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1e406f21ab9f99d21aeb0cdd6e327256.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8cb24c63529eb53b11d99709d8f708b3.png)

##  6. 接口传参

```bash
from flask import Blueprint, render_template, request

views = Blueprint(__name__, "views")


@views.route("/")
def home():
    return render_template("index.html",name="Tim")

@views.route("/profile")
def profile():
    args = request.args
    name = args.get('name')
    return render_template("index.html",name=name)
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c58f48e08295f5ee787067cca1222c89.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7acd2e5997a560aba08da598e5412816.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/80f01716bcbc4c9f2040d81e45a1a905.png)
##  7. 接口返回json
views.py
```bash
from flask import Blueprint, render_template, request, jsonify

views = Blueprint(__name__, "views")


@views.route("/")
def home():
    return render_template("index.html",name="Tim")

@views.route("/profile")
def profile():
    args = request.args
    name = args.get('name')
    return render_template("index.html",name=name)


@views.route("/json")
def get_json():
    return jsonify({'name': 'liming','coolness': 10})
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/54eafc5a78085bf4b686f1ac93bef0d7.png)

##  8. 接口跳转
views.py
```bash
from flask import Blueprint, render_template, request, jsonify,redirect, url_for

views = Blueprint(__name__, "views")


@views.route("/")
def home():
    return render_template("index.html",name="Tim")

@views.route("/profile")
def profile():
    args = request.args
    name = args.get('name')
    return render_template("index.html",name=name)


@views.route("/json")
def get_json():
    return jsonify({'name': 'liming','coolness': 10})

@views.route("/data")
def get_data():
    data = request.json
    return jsonify(data)

@views.route("/go-to-home")
def go_to_home():
    return redirect(url_for("views.get_json"))
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3cd0c2c838ea579b986e5e6a27d199d8.png)
回车go-to-home跳转json
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6cafa8bcea9efd5754361aee768c1dc9.png)
修改为views.py：

```bash
return redirect(url_for("views.home"))
```
go-to-home跳转home
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/74c61cf35bcf8f25b08538c844d24f93.png)

##  9. index.html添加javascript
index.html
```bash
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Home</title>
</head>
<body>
    <div>
        <h1>Home Page</h1>
       <p>hello, {{ name }}</p>
     </div>
     <script type="text/javascript"
     src="{{ url_for('static', filename='index.js')}}"
     ></script>

</body>
</html>
```

创建index.js
```bash
touch static/index.js
```
index.js写入console日志

```bash
console.log("I am running");
```
运行：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f22d26cd5b0d19122da454f728441d0c.png)

##  10. 设置风格

```bash
touch template/profile.html
```
profile.html写入风格

```bash
{% extends "index.html" %}
{% block content %}
<h1> This is the profile page!</h1>
{% endblock %}
```




views.py


```bash
from flask import Blueprint, render_template, request, jsonify,redirect, url_for

views = Blueprint(__name__, "views")


@views.route("/")
def home():
    return render_template("index.html",name="Tim")

@views.route("/profile")
def profile():
    args = request.args
    name = args.get('name')
    return render_template("profile.html")


@views.route("/json")
def get_json():
    return jsonify({'name': 'liming','coolness': 10})

@views.route("/data")
def get_data():
    data = request.json
    return jsonify(data)

@views.route("/go-to-home")
def go_to_home():
    return redirect(url_for("views.home"))
```

运行：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fb651b3f39da044e1b67a678e7daafae.png)
更多阅读：

 - [linux python web flask 编写 Hello World](https://blog.csdn.net/xixihahalelehehe/article/details/106111115)
 - [windows python web flask 编写 Hello World](https://blog.csdn.net/xixihahalelehehe/article/details/106864137)
 - [python flask 快速搭建 WEB](https://ghostwritten.blog.csdn.net/article/details/122093464)
 - [windows python web flask 模板开发快速入门](https://blog.csdn.net/xixihahalelehehe/article/details/106889489)
 - [windows python flask 与mysql 数据库写入查询](https://ghostwritten.blog.csdn.net/article/details/107431748)
 - [windows python flask 返回 json 数据](https://ghostwritten.blog.csdn.net/article/details/107428589)
 - [windows python flask 读取文件数据转换表格](https://blog.csdn.net/xixihahalelehehe/article/details/107419347)
 - [python flask template 模板应用](https://blog.csdn.net/xixihahalelehehe/article/details/106119529)
 - [windows python web route路由详解](https://blog.csdn.net/xixihahalelehehe/article/details/106886851)
 - [windows python web flask获取请求参数数据](https://blog.csdn.net/xixihahalelehehe/article/details/106888653)
 - [python flask-caching模块缓存详解](https://ghostwritten.blog.csdn.net/article/details/107235464)


