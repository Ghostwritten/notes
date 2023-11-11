#  windows python web flask 编写 Hello World
tags: flask
<!--  catalog: ~Windows Flask 编写 Hello Word~ -->

![在这里插入图片描述](https://img-blog.csdnimg.cn/f95cffd89c0e4a799d810de80f3e8f73.png)




## 1. hello world项目
### 1.1 新建一个项目

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200619214037819.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

在默认的app.py点击运行
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200619214230996.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200619214300116.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
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
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200619214918931.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200619214936765.png)

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
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200619215529315.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

#### 1.3.2 自定义配置
自定义项目名称、`static`静态文件位置、`templates`模板位置（一般不常用）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200621154725743.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
#### 1.3.3 调试模式
在app.py修改
```python
if __name__ == '__main__':
    app.run(debug=True)
```
重新运行：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200621155135152.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
#### 1.3.4 绑定IP和端口
这里我设置本机的ip地址192.168.1.4，然后设置不被占用的端口
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200621160059640.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
重新运行：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200621160203232.png)
### 1.4 templates的调用
但是这样的格式，对于维护网页成本很高，当面对复杂的网页时，因此，可以利用我们的templates文件夹进行配置。
在templates目录下创建一个hello.html文件，并复制刚才app.py中的html内容。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020061922002795.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
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
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200619220300202.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
如此看来，app.py非常简约。

### 1.5 templates的传参

修改app.py设置变量
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200619220942765.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
修改hello.py文件调用变量名
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200619221046798.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
重新运行：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020061922110327.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
### 1.6 static的访问
将一张图片放到static目录下
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200619221630650.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
我们不需要重启，可以直接访访问这张图片。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200619221735361.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
### 1.7 html文件调用加载static目录的图片
hello.html文件修改：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200619222250863.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
重新运行：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200619222338318.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
更多阅读：

 - [linux python web flask Hello World实战](https://blog.csdn.net/xixihahalelehehe/article/details/106111115)
 - [windows python web flask 模板开放实战](https://ghostwritten.blog.csdn.net/article/details/106889489)
 - [windows python web flask获取请求参数数据](https://ghostwritten.blog.csdn.net/article/details/106888653)

