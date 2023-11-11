#  windows python web flask获取请求参数数据
tags: flask
<!--  catalog: ~flask 接口获取参数~ -->

![在这里插入图片描述](https://img-blog.csdnimg.cn/f8078c88020141bb9143de0334af567e.png)





![在这里插入图片描述](https://img-blog.csdnimg.cn/20200621174334172.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

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
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200621175118650.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
可以看到打印的参数的信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200621175234790.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
### 2. 获取请求中的header
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020062117572149.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
### 3. 获取请求中的user-Agent
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200621175931199.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
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
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200621180747150.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
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
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020062118150471.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
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
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200621181900824.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
添加内容
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200621182420289.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
重新运行获取表单：
分别输入userxxx 与123456

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200621182929756.png)
如图，我们已经获取表单提交的客户信息。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200621182832839.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

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

