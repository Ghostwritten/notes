#  python 模块 httplib 处理HTTP请求


------
## 1. 介绍
httplib是提供了Web客户端的功能和接口。这样httplib将会完成Web浏览器的基本功能。

模块urllib,urllib2,httplib的区别（细节另起文章）
httplib实现了http和https的客户端协议，但是在python中，`模块urllib和urllib2对httplib`进行了更上层的封装。

## 2. 安装

```c
[root@localhost ~]# pip install httplib2
Downloading/unpacking httplib2
  Downloading httplib2-0.17.0.tar.gz (220kB): 220kB downloaded
  Running setup.py (path:/tmp/pip_build_root/httplib2/setup.py) egg_info for package httplib2
    
Installing collected packages: httplib2
  Running setup.py install for httplib2
    
Successfully installed httplib2
Cleaning up...
```
## 3. class httplib.HTTPConnection
### 3.1 HTTPConnection()
#### 3.1.1 语法
```c
 class httplib.HTTPConnection(host[,port[, strict[, timeout[, source_address]]]]) 
```
#### 3.1.2 用法
    该类用于创建一个http类型的请求链接
#### 3.1.3 参数
 - host: 请求的服务器host，不能带http://开头
 - port: 服务器web服务端口
 - strict: 是否严格检查请求的状态行，就是http1.0/1.1
   协议版本的那一行，即请求的第一行，默认为False，为True时检查错误会抛异常。
 - timeout: 单次请求的超时时间，没有时默认使用httplib模块内的全局的超时时间

#### 3.1.4 返回
    HTTPConnection类会实例并返回一个HTTPConnection对象
#### 3.1.5 详解 
  HttpConnection的实例表示与HTTP服务器的事务。实例化时需要传递主机和可选的端口号。如果没有端口号，试图以host:port格式从主机字符串提取，如果提取失败则使用默认的HTTP端口(80)。
  参数strict默认为false，表示在无法解析状态行时(status line)不能被HTTP/1.0或1.1解析时不抛出BadStatusLine异常；可选参数timeout表示即阻塞在多少秒后超时，如果没有给出默认使用全局超时设置。可选参数source_address表示HTTP的源地址(host, port)。




实例代代码：

```c
import httplib
conn =httplib.HTTPConnection('www.baidu.com')
print conn
conn = httplib.HTTPConnection('www.baidu.com:80')
print conn
conn =httplib.HTTPConnection('www.baidu.com','80')
print conn
conn =httplib.HTTPConnection('www.baidu.com','80',True)
print conn
conn =httplib.HTTPConnection('www.baidu.com','80',True,10)
print conn
conn =httplib.HTTPConnection('www.baidu.com:80',True,10)
print conn

```
输出：

```bash
<httplib.HTTPConnection instance at 0x7feb97b90050>
<httplib.HTTPConnection instance at 0x7feb97b96830>
<httplib.HTTPConnection instance at 0x7feb97b96710>
<httplib.HTTPConnection instance at 0x7feb97b96830>
<httplib.HTTPConnection instance at 0x7feb97b96710>
<httplib.HTTPConnection instance at 0x7feb97b96830>
```

### 3.2 HTTPSConnection()
#### 3.2.1 语法
```bash
httplib.HTTPSConnection('www.baidu.com',443,key_file,cert_file,True,10)
```
#### 3.2.2 用法
该类用于创建一个https类型的请求链接
#### 3.2.3 参数
 - key_file:一个包含PEM格式的私钥文件
 - cert_file:一个包含PEM格式的认证文件
 - other：其它同http参数

#### 3.2.4 返回
    同样返回一个HTTPSConnection对象
**注意**：
    要创建https链接，必须要保证底层的`socket`模块是支持`ssl`的编译模式，即编译时ssl选项的开关是开着的
#### 3.2.5 详解
   HttpConnection的子类，使用SSL与安全服务器通信。默认端口为443。key_file是包含PEM格式私钥的文件名称。 `cert_file`中是PEM格式的证书链文件。

#### 3.2.6 实例代码

```c
import httplib
conn = httplib.HTTPSConnection('www.baidu.com',443,key_file,cert_file,True,10)
```

### 3.3 HTTPConnection.request()
#### 3.3.1 语法

```c
 HTTPConnection.request( method , url [ , body [ , headers ]] )
```

#### 3.3.2 用法
 调用request方法会向服务器发送一次请求
#### 3.3.3 参数
   

 - method: 请求的方式，如'GET','POST','HEAD','PUT','DELETE'等
 - url: 请求的网页路径。如：'/index.html'
 - body: 请求是否带数据，该参数是一个字典
 - headers: 请求是否带头信息，该参数是一个字典，不过键的名字是指定的http头关键字

#### 3.3.4 返回
  无返回，其实就是相对于向服务其发送数据，但是没有最后回车
#### 3.3.5 代码

```c
import httplib
conn =httplib.HTTPConnection('www.baidu.com:80',True,10)
print conn.request('get','/','',{'user-agent':'test'})
```
### 3.4 HTTPConnection.getresponse()
#### 3.4.1 说明
获取一个http响应对象，相当于执行最后的2个回车
#### 3.4.2 返回
HTTPResponse对象（下面会用到）
#### 3.4.3 代码

```c
[root@localhost httplib2]# cat http2.py
#!/usr/bin/python
import httplib

conn=httplib.HTTPConnection('www.baidu.com',80,False,10)
conn.request('get','/','',{'user-agent':'test'})
res = conn.getresponse()
print res
[root@localhost httplib2]# python http2.py
<httplib.HTTPResponse instance at 0x7f459bd107e8>
```
### 3.5 HTTPConnection.connect()
说明：对象创建之后连接到指定的服务器

### 3.6 HTTPConnection.close()
说明：关闭与服务器的连接  
 代码：

```c
 #!/usr/bin/python
import httplib
conn=httplib.HTTPConnection('www.baidu.com',80,False,10)
conn.request('get','/','',{'user-agent':'test'})
res = conn.getresponse()
print res
conn.close()
```
### 3.7 HTTPConnection.set_debuglevel( level )
 说明： 设置高度的级别。参数level 的默认值为0 ，表示不输出任何调试信息

```c
 #!/usr/bin/python
import httplib
conn=httplib.HTTPConnection('www.baidu.com',80,False,10)
conn.request('get','/','',{'user-agent':'test'})
debug = conn.set_debuglevel(0)
print debug
conn.close()
```
## 4.  class httplib.HTTPResponse
 `HTTPResponse`表示服务器对客户端请求的响应。往往通过调用`HTTPConnection.getresponse()`来创建，实例连接成功之后返回的类，不能由用户实例化。
它有如下方法和属性：

###  4.1 HTTPResponse.read([amt])
说明： 获得http响应的内容部分，即网页源码
原型：body = res.read([amt])amt: 读取指定长度的字符，默认为空，即读取所有内容
返回：网页内容字符串
获取响应的消息体。如果请求的是一个普通的网页，那么该方法返回的是页面的html。可选参数amt表示从响应流中读取指定字节的数据。

```c
[root@localhost httplib2]# cat http5.py
#!/usr/bin/python
import httplib
conn=httplib.HTTPConnection('www.baidu.com',80,False,10)
conn.request('GET','')
res = conn.getresponse()
print res.read()
[root@localhost httplib2]# python http5.py
<!DOCTYPE html><!--STATUS OK-->
<html>
<head>
	<meta http-equiv="content-type" content="text/html;charset=utf-8">
	<meta http-equiv="X-UA-Compatible" content="IE=Edge">
	<link rel="dns-prefetch" href="//s1.bdstatic.com"/>
	·········
```
获取执指定的响应头。Name表示头域(headerfield)名，可选参数default在头域名不存在的情况下作为默认值返回。

###  4.2  HTTPResponse.getheaders()
说明; 获得所有的响应头内容，是一个元组列表[(name,value),(name2,value2)]
 附代码：

```c
[root@localhost httplib2]# cat http6.py
#!/usr/bin/python
import httplib
conn=httplib.HTTPConnection('www.baidu.com',80,False,10)
conn.request('GET','')
res = conn.getresponse()
print res.getheaders()
[root@localhost httplib2]# python http6.py
[('content-length', '14615'), ('traceid', '1585662997043926733812101108609502278751'), ('set-cookie', 'BAIDUID=C1DA400A878388A271CC54BCDE99F6F5:FG=1; expires=Thu, 31-Dec-37 23:55:55 GMT; max-age=2147483647; path=/; domain=.baidu.com, BIDUPSID=C1DA400A878388A271CC54BCDE99F6F5; expires=Thu, 31-Dec-37 23:55:55 GMT; max-age=2147483647; path=/; domain=.baidu.com, PSTM=1585662997; expires=Thu, 31-Dec-37 23:55:55 GMT; max-age=2147483647; path=/; domain=.baidu.com, BAIDUID=C1DA400A878388A2E4674A2D94B0BB5F:FG=1; max-age=31536000; expires=Wed, 31-Mar-21 13:56:37 GMT; domain=.baidu.com; path=/; version=1; comment=bd'), ('accept-ranges', 'bytes'), ('vary', 'Accept-Encoding'), ('server', 'BWS/1.1'), ('connection', 'keep-alive'), ('x-ua-compatible', 'IE=Edge,chrome=1'), ('pragma', 'no-cache'), ('cache-control', 'no-cache'), ('date', 'Tue, 31 Mar 2020 13:56:37 GMT'), ('p3p', 'CP=" OTI DSP COR IVA OUR IND COM ", CP=" OTI DSP COR IVA OUR IND COM "'), ('content-type', 'text/html')]
```
###   4.3 HTTPResponse.msg()
说明：获取所有的响应头信息。包含响应头的mimetools.Message实例

```c
[root@localhost httplib2]# cat http7.py
#!/usr/bin/python
import httplib
conn=httplib.HTTPConnection('www.baidu.com',80,False,10)
conn.request('GET','')
res = conn.getresponse()
print res.msg
```

```c
[root@localhost httplib2]# python http7.py
Accept-Ranges: bytes
Cache-Control: no-cache
Connection: keep-alive
Content-Length: 14615
Content-Type: text/html
Date: Tue, 31 Mar 2020 13:59:33 GMT
P3p: CP=" OTI DSP COR IVA OUR IND COM "
P3p: CP=" OTI DSP COR IVA OUR IND COM "
Pragma: no-cache
Server: BWS/1.1
Set-Cookie: BAIDUID=03C55CB4B2F53EEB89BC70720ABA5DB8:FG=1; expires=Thu, 31-Dec-37 23:55:55 GMT; max-age=2147483647; path=/; domain=.baidu.com
Set-Cookie: BIDUPSID=03C55CB4B2F53EEB89BC70720ABA5DB8; expires=Thu, 31-Dec-37 23:55:55 GMT; max-age=2147483647; path=/; domain=.baidu.com
Set-Cookie: PSTM=1585663173; expires=Thu, 31-Dec-37 23:55:55 GMT; max-age=2147483647; path=/; domain=.baidu.com
Set-Cookie: BAIDUID=03C55CB4B2F53EEB1D3F571BA752A14E:FG=1; max-age=31536000; expires=Wed, 31-Mar-21 13:59:33 GMT; domain=.baidu.com; path=/; version=1; comment=bd
Traceid: 1585663173045617562611241334006733438813
Vary: Accept-Encoding
X-Ua-Compatible: IE=Edge,chrome=1
```

### 4.4 HTTPResponse.msg
说明：获取服务器所使用的http协议版本。11表示http/1.1；10表示http/1.0

```c
[root@localhost httplib2]# cat http8.py
#!/usr/bin/python
import httplib
conn=httplib.HTTPConnection('www.baidu.com',80,False,10)
conn.request('GET','')
res = conn.getresponse()
print res.version
```

```bash
[root@localhost httplib2]# python http8.py
11
```
### 4.5  HTTPResponse.status
说明： 获取响应的状态码。如：200表示请求成功

```c
[root@localhost httplib2]# cat http9.py
#!/usr/bin/python
import httplib
conn=httplib.HTTPConnection('www.baidu.com',80,False,10)
conn.request('GET','')
res = conn.getresponse()
print res.status
```

```c
[root@localhost httplib2]# python http9.py
200
```
### 4.6 HTTPResponse.reason
说明：返回服务器处理请求的结果说明。一般为”OK”

```c
[root@localhost httplib2]# cat http10.py
#!/usr/bin/python
import httplib
conn=httplib.HTTPConnection('www.baidu.com',80,False,10)
conn.request('GET','')
res = conn.getresponse()
print res.reason
```

```c
[root@localhost httplib2]# python http10.py
OK
```
## 5. class httplib.HTTPMessage
  `HTTPMessage`实例用于**保存HTTP响应头**。它使用mimetools.Message类实现，并提供了处理HTTP头的工具函数。它不直接实例化的用户。不能由用户实例化。

##  6. 异常处理

```c
exception httplib.HTTPException
```
Exception的子类，此模块中的其他异常的基类。下面的类默认是该类的直接子类。

```c
httplib.NotConnected

httplib.InvalidURL

httplib.UnknownProtocol

httplib.UnknownTransferEncoding

httplib.UnimplementedFileMode

httplib.IncompleteRead

httplib.ImproperConnectionState

httplib.CannotSendRequest

ImproperConnectionState的一个子类。

httplib.CannotSendHeader

ImproperConnectionState的一个子类。

httplib.ResponseNotReady

ImproperConnectionState的一个子类。

httplib.BadStatusLine
```

服务器返回的HTTP状态码不认识时产生。


## 7. 实战

```c
[root@localhost httplib2]# cat http11.py
#!/usr/bin/python
#coding:utf-8
import httplib, urllib
conn = None
try:
    params = urllib.urlencode({'name': 'qiye', 'age': 22})
    headers = {"Content-type": "application/x-www-form-urlencoded"
    , "Accept": "text/plain"}
    conn = httplib.HTTPConnection("www.zhihu.com", 80, timeout=3)
    conn.request("POST", "/login", params, headers)
    response = conn.getresponse()
    print response.getheaders() # 获取头信息
    print response.status
    print response.read()
except Exception, e:
    print e
finally:
    if conn:
        conn.close()
```

```c
[root@localhost httplib2]# python http11.py
[('x-edge-timing', '0.000'), ('content-length', '278'), ('via', 'vcache3.cn2204[,0]'), ('x-cdn-provider', 'alibaba'), ('eagleid', '3ad79e1715856640307233330e'), ('server', 'Tengine'), ('connection', 'keep-alive'), ('location', 'https://www.zhihu.com/login'), ('date', 'Tue, 31 Mar 2020 14:13:50 GMT'), ('content-type', 'text/html'), ('timing-allow-origin', '*')]
301
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html>
<head><title>301 Moved Permanently</title></head>
<body bgcolor="white">
<h1>301 Moved Permanently</h1>
<p>The requested resource has been assigned a new permanent URI.</p>
<hr/>Powered by Tengine</body>
</html>
```

参考：

 - [Python中操作HTTP请求的urllib模块详解](https://cloud.tencent.com/developer/article/1504178)
 - [腾讯云 python httplib](https://cloud.tencent.com/developer/section/1368319)
 - [Python httplib.HTTPConnection() Examples](https://www.programcreek.com/python/example/187/httplib.HTTPConnection)

