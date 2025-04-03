#  windows python web flask获取请求参数数据
tags: flask
<!--  catalog: ~flask 接口获取参数~ -->

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/370d1468a33dbc52a2ab524ff66f760a.png)





![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/157e8307b0d30c2cc0b8977f60255b32.png)

设置一个路由url

### 1. 获取请求中的参数值
```python
@app.route('/data')
def test_data():
    print(request.args)
    print(request.args.get("a"), request.args.get("b"))
    return 'success'
if __name__ == '__main__':
    app.run(host='192.168.1.4')
```
运行结果：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/09223cc022596fab69b47d9bb6eee89c.png)
可以看到打印的参数的信息
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b560991049905c82d7b4b7807e4eeab4.png)
### 2. 获取请求中的header
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/943d57d6b2416bb5722a9c2e55ec0a61.png)
### 3. 获取请求中的user-Agent
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e5999385004abf20c90ab47e72ef4103.png)
### 4. 获取请求中一组json数据

```python
def test_data():
    print(request.data)
    import json
    print(json.loads(request.data))
    return 'success'
if __name__ == '__main__':
    app.run(host='192.168.1.4')
```
ipython命令行发起一组请求数据：

```python
In [1]: import requests

In [2]: url = "http://192.168.1.4:5000/data"

In [3]: import json

In [4]: data=json.dumps({"dataa":123,"datab":"xxx"})

In [5]: requests.get(url, data=data)  #发起请求
Out[5]: <Response [200]>
```
如图，已获取客户端发出请求的数据。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4440d4df7f550a6c269de405e01ea9e4.png)
### 5. 获取请求中的cookies

```python
def test_data():
    # print(request.args)
    # print(request.args.get("a"), request.args.get("b"))
    # print(request.headers)
    # print(request.headers.get("User-Agent"))
    # print(request.data)
    # import json
    # print(json.loads(request.data))
    print(request.cookies)
    print(request.cookies.get("token"))
    return 'success'
if __name__ == '__main__':
    app.run(host='192.168.1.4')
```
客户端发起包含cookies的请求

```python
In [8]: requests.get(url, data=data, cookies={"token":"tokenxxx"})
Out[8]: <Response [200]>
```
如图获取的`cookies`的值
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d56c8c338816a5dbfd9e7a131220e7d3.png)
### 6. 获取请求中的form的值
修改app.py文件
```python
@app.route('/data', methods=["POST", "GET"])
def test_data():
    print(request.form)
    print(request.form.get("username"), request.form.get("password"))
    return 'success'
if __name__ == '__main__':
    app.run(host='192.168.1.4')
```
创建一个静态html文件
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2a0b2ce5da63d515117b80f1250c1f55.png)
添加内容
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/26cb0b9f17127026ba2889922cfdc9d6.png)
重新运行获取表单：
分别输入userxxx 与123456

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/14bb492454e6e20db53fe73a46b9b7bc.png)
如图，我们已经获取表单提交的客户信息。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/21fc83c3f66765982b78e2d1941bafcf.png)

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

