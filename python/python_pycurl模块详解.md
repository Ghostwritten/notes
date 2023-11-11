# python 模块 pycurl 处理 URL



----
## 1. 介绍
[pycurl](https://pypi.org/project/pycurl/) 是功能强大的python的url库，是用c语言写的，速度很快，比urllib和httplib都快。支持操作协议有`FTP, HTTP,HTTPS,TELNET`等，通过pycurl提供的方法，可以实现探测WEB服务质量的情况，比如，响应的HTTP状态码、请求延时、HTTP头信息、下载速度等。

## 2. 安装
第一种：

```bash
[root@localhost ~]$ pip install pycurl
```
第二种：

```bash
yum install libcurl-devel

wget https://curl.haxx.se/download/curl-7.61.0.tar.gz

tar -zxvf curl-7.61.0.tar.gz

cd curl-7.61.0/

./configure 

make && make install

export LD_LIBRARY_PATH=/usr/local/lib
```
## 3. 类与方法

 - close()方法：对应libcurl包中的curl_easy_cleanup方法，无参数，实现关闭，回收，curl对象。
 - perform()方法：对应lib包中的curl_easy_perform方法，无参数，实现curl对象请求的提交。
 - setopt(option,value)方法，对应libcurl包中的curl_easy_setopt方法，参数option是通过libcurl的常量来指定的，参数value的值会依赖option，可以是一个字符串、整型，长整型，文件对象，列表或函数等。

```c
c = pycurl.Curl() #创建一个curl对象
c.setopt(pycurl.CONNECTTIMEOUT, 5) #连接的等待时间，设置为0则不等待
c.setopt(pycurl.TIMEOUT, 5) #请求超时时间
c.setopt(pycurl.NOPROGRESS, 0) #是否屏蔽下载进度条，非0则屏蔽
c.setopt(pycurl.MAXREDIRS, 5) #指定HTTP重定向的最大数
c.setopt(pycurl.FORBID_REUSE, 1) #完成交互后强制断开连接，不重用
c.setopt(pycurl.FRESH_CONNECT,1) #强制获取新的连接，即替代缓存中的连接
c.setopt(pycurl.DNS_CACHE_TIMEOUT,60) #设置保存DNS信息的时间，默认为120秒
c.setopt(pycurl.URL,"http://www.baidu.com") #指定请求的URL
c.setopt(pycurl.USERAGENT,"Mozilla/5.2 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 1.1.4322; .NET CLR 2.0.50324)") #配置请求HTTP头的User-Agent
c.setopt(pycurl.HEADERFUNCTION, getheader) #将返回的HTTP HEADER定向到回调函数getheader
c.setopt(pycurl.WRITEFUNCTION, getbody) #将返回的内容定向到回调函数getbody
c.setopt(pycurl.WRITEHEADER, fileobj) #将返回的HTTP HEADER定向到fileobj文件对象
c.setopt(pycurl.WRITEDATA, fileobj) #将返回的HTML内容定向到fileobj文件对象
```

 - getinfo(option)方法：对应libcurl包中的curl_easy_getinfo方法，参数option是通过libcurl的常量来指定的。

```c
c = pycurl.Curl() #创建一个curl对象
c.getinfo(pycurl.HTTP_CODE) #返回的HTTP状态码
c.getinfo(pycurl.TOTAL_TIME) #传输结束所消耗的总时间
c.getinfo(pycurl.NAMELOOKUP_TIME) #DNS解析所消耗的时间
c.getinfo(pycurl.CONNECT_TIME) #建立连接所消耗的时间
c.getinfo(pycurl.PRETRANSFER_TIME) #从建立连接到准备传输所消耗的时间
c.getinfo(pycurl.STARTTRANSFER_TIME) #从建立连接到传输开始消耗的时间
c.getinfo(pycurl.REDIRECT_TIME) #重定向所消耗的时间
c.getinfo(pycurl.SIZE_UPLOAD) #上传数据包大小
c.getinfo(pycurl.SIZE_DOWNLOAD) #下载数据包大小
c.getinfo(pycurl.SPEED_DOWNLOAD) #平均下载速度
c.getinfo(pycurl.SPEED_UPLOAD) #平均上传速度
c.getinfo(pycurl.HEADER_SIZE) #HTTP头部大小
```
## 4. 探索web服务质量
HTTP服务是最流行的互联网应用之一，服务质量的好坏关系到用户体验以及网站的运营服务水平，最常用的有两个标准：

 - 一为服务的可用性，比如是否处于正常提供服务状态，而不是出现404页面或500页面错误等；
 - 二是服务响应的速度，比如静态类文件下载时间都控制在毫秒级，动态CGI为秒级。

本实例使用pycurl的setopt与getinfo方法时间HTTP服务质量的探测，获取监控URL返回的HTTP状态码,HTTP状态码采用pycurl.HTTP_CODE常量，以及从HTTP请求到下载期间各环节的响应时间，通过pycurl.NAMELOOKUP_TIME、pycurl.CONNECT_TIME、pycurl.PRETRANSFER_TIME、pycurl.R等常量来时间。另外通过pycurl.WRITEHEADER、pycurl.WRITEDATA常量得到目标URL的HTTP响应头部及页面内容.
第一种方法：
```python
#!/usr/bin/python
# -*- coding:UTF-8 -*-
import os, sys
import time
import pycurl
 
#探测的目标URL
URL= "http://www.baidu.com"
#创建一个Curl对象
c = pycurl.Curl()
 
#定义请求的URL常量
c.setopt(pycurl.URL, URL)
#定义请求连接的等待时间
c.setopt(pycurl.CONNECTTIMEOUT, 5)
#定义请求超时时间
c.setopt(pycurl.TIMEOUT, 5)
#屏蔽下载进度条
c.setopt(pycurl.NOPROGRESS, 1)
#完成交互后强制断开连接，不重用
c.setopt(pycurl.FORBID_REUSE, 1)
#指定HTTP重定向的最大数为1
c.setopt(pycurl.MAXREDIRS, 1)
#设置保存DNS信息的时间为30秒
c.setopt(pycurl.DNS_CACHE_TIMEOUT, 30)
#创建一个文件对象，以“wb”方式打开，用来存储返回的http头部及页面内容
indexfile = open(os.path.dirname(os.path.realpath(__file__)) + "/content.txt", "wb")
#将返回的HTTP HEADER定向到indexfile文件
c.setopt(pycurl.WRITEHEADER, indexfile)
#将返回的HTML内容定向到indexfile文件对象
c.setopt(pycurl.WRITEDATA, indexfile)
try:
    #提交请求
    c.perform()
except Exception, e:
    print "connection error: " + str(e)
    indexfile.close()
    c.close()
    sys.exit()
    
#获取DNS解析时间
NAMELOOKUP_TIME = c.getinfo(c.NAMELOOKUP_TIME)
#获取建立连接时间
CONNECT_TIME = c.getinfo(c.CONNECT_TIME)
#获取从建立连接到准备传输所消耗的时间
PRETRANSFER_TIME = c.getinfo(c.PRETRANSFER_TIME)
#获取从建立连接到传输开始消耗的时间
STARTTRANSFER_TIME = c.getinfo(c.STARTTRANSFER_TIME)
#获取传输的总时间
TOTAL_TIME = c.getinfo(c.TOTAL_TIME)
#获取HTTP状态码
HTTP_CODE = c.getinfo(c.HTTP_CODE)
#获取下载数据包大小
SIZE_DOWNLOAD = c.getinfo(c.SIZE_DOWNLOAD)
#获取HTTP头部大小
HEADER_SIZE = c.getinfo(c.HEADER_SIZE)
#获取平均下载速度
SPEED_DOWNLOAD = c.getinfo(c.SPEED_DOWNLOAD)
#打印输出相关数据
print "HTTP状态码： %d" % (HTTP_CODE)
print "DNS解析时间: %.2f ms" % (NAMELOOKUP_TIME * 1000)
print "建立连接时间： %.2f ms" % (CONNECT_TIME * 1000)
print "准备传输时间： %.2f ms" % (PRETRANSFER_TIME * 1000)
print "传输开始时间: %.2f ms" % (STARTTRANSFER_TIME * 1000)
print "传输结束总时间： %.2f ms" %(TOTAL_TIME * 1000)
print "下载数据包大小： %d bytes/s" %(SIZE_DOWNLOAD)
print "HTTP头部大小： %d byte" %(HEADER_SIZE)
print "平均下载速度： %d bytes/s" %(SPEED_DOWNLOAD)
#关闭文件及Curl对象
indexfile.close()
c.close()
```


```python
[root@localhost pycurl]# python pycurl1.py
HTTP状态码： 200
DNS解析时间: 62.75 ms
建立连接时间： 105.37 ms
准备传输时间： 105.51 ms
传输开始时间: 150.22 ms
传输结束总时间： 340.06 ms
下载数据包大小： 237288 bytes/s
HTTP头部大小： 1208 byte
平均下载速度： 697774 bytes/s
```
查看获取的文件头部及页面内容文件content.txt,如下图所示
```bash
[root@localhost pycurl]# ls
content.txt  pycurl1.py
[root@localhost pycurl]# head -n 20  content.txt 
HTTP/1.1 200 OK
Bdpagetype: 1
Bdqid: 0x80504cae000d2c9b
Cache-Control: private
Connection: keep-alive
Content-Type: text/html;charset=utf-8
Date: Thu, 16 Apr 2020 03:45:00 GMT
Expires: Thu, 16 Apr 2020 03:44:32 GMT
P3p: CP=" OTI DSP COR IVA OUR IND COM "
P3p: CP=" OTI DSP COR IVA OUR IND COM "
Server: BWS/1.1
.........................
```
第二种方法

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Time    : 2016/10/18 9:50
# @Author  : Beam
# @Site    : 实现探测Web服务质量
# @File    : demo_pycurl.py
# @Software: PyCharm
import pycurl
import os,sys,time
def getInfo(URL):
    """
    :param URL:用户输入需要检测的URL地址
    info 字典主要用于映射dic字典
    dic字典主要存下curl结果
    :return:没return，直接print，函数可以改写，可以用于定时检测多个域名，增加一个需要检测的url列表即可
    """
    c = pycurl.Curl()
    c.setopt(pycurl.URL,URL) #定义请求的URL常量
    c.setopt(pycurl.CONNECTTIMEOUT,5) #请求等待时间最多5秒
    c.setopt(pycurl.TIMEOUT,5)   #定义请求超时时间（服务器没回应）
    c.setopt(pycurl.NOPROGRESS,1) #屏蔽下载进度条
    c.setopt(pycurl.FORBID_REUSE,1) #交互完成后强制断开连接，不重用
    c.setopt(pycurl.MAXREDIRS,1)  #指定HTTP重定向的最大数为1
    c.setopt(pycurl.DNS_CACHE_TIMEOUT,30) #设置DNS信息保存时间为30秒
    c.setopt(pycurl.USERAGENT,"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.116 Safari/537.36 OPR/40.0.2308.81 (Edition Baidu)")
    dic = {}
    info ={'NAMELOOKUP_TIME':'DNS解析时间','CONNECT_TIME':'建立连接时间','PRETRANSFER_TIME':'建立到准备传输所消耗的时间','STARTTRANSFER_TIME':'建立连接到传输开始消耗的时间','TOTAL_TIME':'传输总时间',
           'HTTP_CODE':'HTTP状态码','SIZE_DOWNLOAD':'下载数据包大小','HEADER_SIZE':'HTTP头部大小','SPEED_DOWNLOAD':'平均下载速度'}
    with open(os.path.dirname(os.path.realpath(__file__))+"/content.txt","wb") as indexfile:
        c.setopt(pycurl.WRITEHEADER,indexfile)   #将返回的HTTP HEADER定向到indexfile文件对象
        c.setopt(pycurl.WRITEDATA,indexfile)     #将返回的HTML内容定向到indexfile文件对象
        try:
            c.perform()
        except Exception,e:
            print "Connection error:" +str(e)
            c.close()
            sys.exit()
        dic['NAMELOOKUP_TIME'] = '%.2f ms' % (c.getinfo(c.NAMELOOKUP_TIME)*1000)     #获取DNS解析时间
        dic['CONNECT_TIME'] = '%.2f ms' % (c.getinfo(c.CONNECT_TIME)*1000)          #获取建立连接时间
        dic['PRETRANSFER_TIME'] = '%.2f ms' % (c.getinfo(c.PRETRANSFER_TIME)*1000)  #获取从建立到准备传输所消耗的时间
        dic['STARTTRANSFER_TIME'] = '%.2f ms' % (c.getinfo(c.STARTTRANSFER_TIME)*1000)  #获取从建立连接到传输开始消耗的时间
        dic['TOTAL_TIME'] = '%.2f ms' % (c.getinfo(c.TOTAL_TIME)*1000)              #获取传输总时间
        dic['HTTP_CODE'] = c.getinfo(c.HTTP_CODE)                #获取HTTP状态码
        dic['SIZE_DOWNLOAD'] = '%d bytes/s' % (c.getinfo(c.SIZE_DOWNLOAD))        #获取下载数据包大小
        dic['HEADER_SIZE'] = '%d bytes/s' % (c.getinfo(c.HEADER_SIZE))            #获取HTTP头部大小
        dic['SPEED_DOWNLOAD'] = '%d bytes/s' % (c.getinfo(c.SPEED_DOWNLOAD))      #获取平均下载速度
    for key in info:
        print info[key],':',dic[key]
def main():
    while True:
        URL = raw_input("请输入一个URL地址(Q for exit)：")
        if URL.lower() == 'q':
            sys.exit()
        else:
            getInfo(URL)
if __name__ == '__main__':
    main()
```

```python
[root@localhost pycurl]# python pycurl2.py
请输入一个URL地址(Q for exit)：http://baidu.com
Connection error:(52, 'Empty reply from server')
[root@localhost pycurl]# python pycurl2.py
请输入一个URL地址(Q for exit)：http://www.baidu.com
传输总时间 : 306.28 ms
建立连接到传输开始消耗的时间 : 168.19 ms
平均下载速度 : 679439 bytes/s
建立到准备传输所消耗的时间 : 107.48 ms
下载数据包大小 : 208096 bytes/s
DNS解析时间 : 63.36 ms
建立连接时间 : 107.29 ms
HTTP头部大小 : 1209 bytes/s
HTTP状态码 : 200
请输入一个URL地址(Q for exit)：
```
参考：
 - [pycurl 官方](http://pycurl.io/)
 - [实现探测Web服务质量](https://www.jianshu.com/p/862db80e992e)
 - [《python运维自动化与实践刘天斯》](https://read.douban.com/ebook/15305244/)
 - [https://github.com/pycurl/pycurl](https://github.com/pycurl/pycurl)

