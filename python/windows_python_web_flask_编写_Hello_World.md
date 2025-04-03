#  windows python web flask 编写 Hello World
tags: flask
<!--  catalog: ~Windows Flask 编写 Hello Word~ -->

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cb2a7b65c0c5e8e118f054938915874d.png)




## 1. hello world项目
### 1.1 新建一个项目

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7590035dc22c917e131174d3fbaf129c.png)

在默认的app.py点击运行
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/59f1b2c4f2086a93577de54249f216e2.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2d6d171e531bbdac88639a1e7ffd32a2.png)
### 1.2 hello world项目修改无效的原因
修改返回内容，发现仍然是`“hello world”`，后来发现是端口占用

```bash
C:\Users\XH>netstat -aon|findstr "5000"
  TCP    127.0.0.1:5000         0.0.0.0:0              LISTENING       21380
  TCP    127.0.0.1:5000         0.0.0.0:0              LISTENING       21488
  TCP    127.0.0.1:5000         0.0.0.0:0              LISTENING       19480
  TCP    127.0.0.1:5000         0.0.0.0:0              LISTENING       20848
  TCP    127.0.0.1:5000         0.0.0.0:0              LISTENING       19816
  TCP    127.0.0.1:5000         0.0.0.0:0              LISTENING       6900
  TCP    127.0.0.1:5000         0.0.0.0:0              LISTENING       19568
  TCP    127.0.0.1:5000         0.0.0.0:0              LISTENING       20688
  TCP    127.0.0.1:5000         0.0.0.0:0              LISTENING       12516
  TCP    127.0.0.1:5000         0.0.0.0:0              LISTENING       16360
  TCP    127.0.0.1:5000         0.0.0.0:0              LISTENING       19900
  TCP    127.0.0.1:50000        0.0.0.0:0              LISTENING       4820

C:\Users\XH>netstat -aon|findstr "5000"
```
通过进程号查询占用进程命令，原来是多余的python.exe

```bash
C:\Users\XH>tasklist|findstr  19900
python.exe                   19900 Console                    2     27,332 K
```
杀死进程

```bash
C:\Users\XH>taskkill  /f  /pid  21380
成功: 已终止 PID 为 21380 的进程。
```
### 1.3 修改hello world成功的效果
#### 1.3.1 修改显示内容
`app.py`重新测试修改返回值。

```python
from flask import Flask

app = Flask(__name__)


@app.route('/')
def hello_world():
    return 'Welcome to Hello World !'

@app.route('/hello')
def hello_world2():
    return 'Hello World 2!'
    
if __name__ == '__main__':
    app.run()
```
重新运行
**注意：要关闭之前的启动进程。CTRL + C或者点击重新运行。**
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ff6ec26cdc9684b0d9ff8596b2fa3e75.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/defbe4cd8675c3eec2bcc0af024f98cd.png)

app.py设置一个`html`内容

```python
from flask import Flask

app = Flask(__name__)


@app.route('/')
def hello_world():
    return 'Welcome to Hello World !'

@app.route('/hello')
def hello_world2():
    return '''
    <html>
    <body>
    <h1 style="color:#e00;">hello world</h>
    </body>
    </html>
    '''
if __name__ == '__main__':
    app.run()
```
重新运行：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c73393b627906af9083f44508863908d.png)

#### 1.3.2 自定义配置
自定义项目名称、`static`静态文件位置、`templates`模板位置（一般不常用）
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5338d88833a844d4c2e8c80b9bb230c8.png)
#### 1.3.3 调试模式
在app.py修改
```python
if __name__ == '__main__':
    app.run(debug=True)
```
重新运行：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4e4cd93a3704dc93be3edf18bafce543.png)
#### 1.3.4 绑定IP和端口
这里我设置本机的ip地址192.168.1.4，然后设置不被占用的端口
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3b549033d3296a0501601e6a3210c736.png)
重新运行：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/baa30a1946fa84b43d1b3d03301d93ac.png)
### 1.4 templates的调用
但是这样的格式，对于维护网页成本很高，当面对复杂的网页时，因此，可以利用我们的templates文件夹进行配置。
在templates目录下创建一个hello.html文件，并复制刚才app.py中的html内容。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ede6e5a8e8e954665309759b19cae903.png)
然后app.py调用templates的文件。

```python
from flask import Flask, render_template  #多添加一个方法

app = Flask(__name__)


@app.route('/')
def hello_world():
    return 'Welcome to Hello World !'

@app.route('/hello')
def hello_world2():
    return render_template("hello.html")  #方法中指定文件

if __name__ == '__main__':
    app.run()
```
重新运行：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fae0cee4322106ade55db000bd285eca.png)
如此看来，app.py非常简约。

### 1.5 templates的传参

修改app.py设置变量
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/56f807ea81d50d5e86703b37985fd64c.png)
修改hello.py文件调用变量名
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6cbcdedc39d1d21be6cdbdd00fb686c5.png)
重新运行：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2833ff4b69cfd13bb661258dca4c83b5.png)
### 1.6 static的访问
将一张图片放到static目录下
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0042779e1a4cc9e683014f0ffb4c34b3.png)
我们不需要重启，可以直接访访问这张图片。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e5cda9690e79d22dd09bd6d066e30d67.png)
### 1.7 html文件调用加载static目录的图片
hello.html文件修改：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f49c2ea0f49e43b3b181f458a91bc694.png)
重新运行：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/355830737ae0a43370e906d0dea5cea6.png)
更多阅读：

 - [linux python web flask Hello World实战](https://blog.csdn.net/xixihahalelehehe/article/details/106111115)
 - [windows python web flask 模板开放实战](https://ghostwritten.blog.csdn.net/article/details/106889489)
 - [windows python web flask获取请求参数数据](https://ghostwritten.blog.csdn.net/article/details/106888653)

