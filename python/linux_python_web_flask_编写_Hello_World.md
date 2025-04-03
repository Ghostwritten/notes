#  linux python web flask 编写 Hello World
tags: flask
<!--  catalog: ~flask 编写 Hello World~ -->

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/382394ff8a15ad44a10851dc2c84d61a.png)





## 1. 项目hello world
### 1.1 创建app
```bash
$ mkdir microblog   #创建项目
$ cd microblog
$ python3 -m venv venv  #虚拟环境已经成为内置模块,直接创建
$ source venv/bin/activate  #进入虚拟环境
(venv) $ 
(venv) $ pip install flask  #安装模块
(venv) $ mkdir app         #创建一个名为app的包来存放整个应用
```
### 1.2 创建__init __.py
包含__init__.py文件的子目录被视为一个可导入的包。 当你导入一个包时，__init__.py会执行并定义这个包暴露给外界的属性
```bash
(venv) $ vim app/'___init __.py  
from flask import Flask  #从flask中导入的类Flask

app = Flask(__name__)  #传递给Flask类的__name__变量是一个Python预定义的变量，它表示当前调用它的模块的名字

from app import routes
```
### 1.3 创建routes.py
**路由是应用程序实现的不同URL。** 在Flask中，应用程序路由的处理逻辑被编写为Python函数，称为视图函数。 视图函数被映射到一个或多个路由URL，以便Flask知道当客户端请求给定的URL时执行什么逻辑。

```bash
(venv) $ vim app/routes.py  
from app import app

@app.route('/')
@app.route('/index')
def index():
    return "Hello, World!"
```
这个视图函数简单到只返回一个字符串作为问候用语。 函数上面的两个奇怪的＠app.route行是装饰器，这是Python语言的一个独特功能。 装饰器会修改跟在其后的函数。 装饰器的常见模式是使用它们将函数注册为某些事件的回调函数。 在这种情况下，＠app.route修饰器在作为参数给出的URL和函数之间创建一个关联。 在这个例子中，有两个装饰器，它们将URL /和/index索引关联到这个函数。 这意味着，当Web浏览器请求这两个URL中的任何一个时，Flask将调用该函数并将其返回值作为响应传递回浏览器。这样做是为了在运行这个应用程序的时候会稍微有一点点意义。

### 1.4 创建microblog.py
要完成应用程序，你需要在定义Flask应用程序实例的顶层（译者注：也就是microblog目录下）创建一个命名为microblog.py的Python脚本。 它仅拥有一个导入应用程序实例的行：

```bash
(venv) $ vim microblog.py
from app import app
```
### 1.5 启动
项目结构图：

```bash
microblog/
  venv/
  app/
    __init__.py
    routes.py
  microblog.py
```
设置FLASK_APP环境变量告诉Flask如何导入它：

```bash
(venv) $ export FLASK_APP=microblog.py
```
注意：如果你使用Microsoft Windows操作系统，在上面的命令中使用set替换export。

```bash
(venv) $ flask run
 * Serving Flask app "microblog"
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```
服务启动后将处于阻塞监听状态，将等待客户端连接。 flask run的输出表明服务器正在运行在IP地址127.0.0.1上，这是本机的回环IP地址.

```bash
$ curl  http://localhost:5000/
Hello, World!
$ curl  http://localhost:5000/index
Hello, World!
```
在终端会话中直接设置的环境变量不会永久生效，因此你不得不在每次新开终端时设定 `FLASK_APP` 环境变量，从 1.0 版本开始，Flask 允许你设置只会在运行flask命令时自动注册生效的环境变量，要实现这点，你需要安装 `python-dotenv`：

```bash
(venv) $ pip install python-dotenv
```

此时，在项目的根目录下新建一个名为 `.flaskenv` 的文件，其内容是：

```bash
$  vim .flaskenv
FLASK_APP=microblog.py
```

## 2. 脚本 hello world

```python
from flask import Flask

app = Flask(__name__)


@app.route('/')
def hello_world():
    return 'Hello World!'


if __name__ == '__main__':
    app.run()
```
![!\[在这里插入图片描述\](https://img-blog.csdnimg.cn/20200514112507443.png)!\[在这里插入图片描述\](https://img-blog.csdnimg.cn/20200514112506195.png](https://i-blog.csdnimg.cn/blog_migrate/5ace38466844f60b0c5cbb5f9cf9f1e9.png)


更多阅读：
 - [linux python web flask Hello World实战](https://blog.csdn.net/xixihahalelehehe/article/details/106111115)
 - [windows python web flask Hello World实战](https://ghostwritten.blog.csdn.net/article/details/106864137)
 - [windows python web flask 模板开放实战](https://ghostwritten.blog.csdn.net/article/details/106889489)
 - [windows python web flask获取请求参数数据](https://ghostwritten.blog.csdn.net/article/details/106888653)
 - [windows python flask返回json数据](https://ghostwritten.blog.csdn.net/article/details/107428589)
 - [windows python flask与mysql数据库写入查询显示等操作详解](https://ghostwritten.blog.csdn.net/article/details/107431748)
 - [python flask-caching模块缓存详解](https://ghostwritten.blog.csdn.net/article/details/107235464)

