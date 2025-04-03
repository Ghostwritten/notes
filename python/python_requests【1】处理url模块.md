#  python 模块 requests (1) 处理url



----
## 1. 简介
[requests](https://pypi.org/project/requests/) 是python的一个HTTP客户端库，跟urllib，urllib2类似，那为什么要用requests而不用urllib2呢？



## 2. 安装

```bash
pip install requests
```

## 3. 方法

import requests  
	 HTTP请求：GET、POST、PUT、DELETE、HEAD、OPTIONS
```bash
requests.get("http://xxxx.com/")
requests.post("http://xxxx.com/post", data = {'key':'value'})
requests.put("http://xxxx.com/put", data = {'key':'value'})
requests.delete("http://xxxx.com/delete")
requests.head("http://xxxx.com/get")
requests.options("http://xxxx.com/get")

 为URL传递参数
	requests模块使用params关键字参数，以一个字典的形式来提供参数。
>>> payload = {'key1':'value','key2':'value2'}
>>> res = requests.get("http://httpbin.org/get",params=payload)
>>> res.url
u'http://httpbin.org/get?key2=value2&key1=value'

>>> r = requests.get('http://www.zhidaow.com')  # 发送请求
>>> r.status_code  # 返回码 
200
>>> r.headers['content-type']  # 返回头部信息
'text/html; charset=utf8'
>>> r.encoding  # 编码信息
'utf-8'
>>> r.text  #内容部分（PS，由于编码问题，建议这里使用r.content）
u'<!DOCTYPE html>\n<html xmlns="http://www.w3.org/1999/xhtml"...'
...
>>> res.json()
>>> res.raw
```
response.text和response.content的区别在于:

 - response.text是解过码的字符串(比如html代码)。当requests发送请求到一个网页时，requests库会推测目标网页的编码，并对其解码，转为字符串(str)。这种方法比较容易出现乱码。
 - response.content是未解码的二进制格式(bytes)，不仅支持文本内容，还适用于二进制文件内容如图片和音乐等。如果需要把文本内容转化为字符串，一般使用response.content.decode('utf-8')方法即可。

### 3.1 发送带参数的get请求

```powershell
import requests

params = {
    "wd": "python", "pn": 10,
}

response = requests.get('https://www.baidu.com/s', params=params)
print(response.url)
print(response.text)
```

### 3.2 发送带数据的post请求

```python
import requests


post_data = {'username': 'value1', 'password': 'value2'}

response = requests.post("http://xxx.com/login/", data=post_data)
response.raise_for_status()
```

### 3.3 post也可以用于上传文件
```bash
>>> import requests
>>> url = 'http://httpbin.org/post'
>>> files = {'file': open('report.xls', 'rb')}
>>> r = requests.post(url, files=files)
```
### 3.4 设置与查看请求头(headers)

很多网站都有反爬机制，如果一个请求不携带请求头headers, 很可能被禁止访问。我们可以按如下方式设置请求头。你也可以通过打印response.headers查看当前请求头。

```python
import requests
headers = {

    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_4) AppleWebKit/"
                 "537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36"
}

response1 =requests.get("https://www.baidu.com", headers=headers)
response2 =requests.post("https://www.xxxx.com", data={"key": "value"}, 
headers=headers)

print(response1.headers)
print(response1.headers['Content-Type'])
print(response2.text)
```
### 3.5 代理Proxy
有的网站反爬机制会限制单位时间内同一IP的请求次数，这时我们可以通过设置IP proxy代理来应对这个反爬机制。requests里设置proxy也非常简单，如下面代码所示。

```python
import requests

proxies = {
  "http": "http://10.10.1.10:3128",
  "https": "http://10.10.1.10:1080",
}

requests.get("http://example.org", proxies=proxies)
```
## 3.6 保存cookies

```python
cookies = requests.cookies.RequestsCookieJar()
def login(endpointip):
    endpoint = "http://" + endpointip + ":8080/upcdb_manager/v1.0/login"
    header = {}
    header['Content-Type'] = "application/json"
    payload = {
        "loginName": username,
        "password": password
    }
    response = requests.request("POST", endpoint, data=json.dumps(payload),headers=header)
    return response.cookies
if __name__ == '__main__':
    cookies = login('192.168.1.19')
    url = 'www.baidu.com/xxx'
   r = requests.get(url,cookies=cookies)
```
✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>


 - [python requests【2】高阶](https://ghostwritten.blog.csdn.net/article/details/124088523)
 - [w3schools Python Requests Module](https://www.w3schools.com/python/module_requests.asp)
 - [runoob Python requests 模块](https://www.runoob.com/python3/python-requests.html)
 - [realpython Python’s Requests Library (Guide)](https://realpython.com/python-requests/)


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e13c43e289ce6c14c9def3ac741e61cf.gif#pic_center)

