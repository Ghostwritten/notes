#  python stmplib与email 模块 发送邮箱



----

##  1. stmplib 简介
简单邮件传输协议 (SMTP) 是一种协议，用于处理在邮件服务器之间发送电子邮件和路由电子邮件。

Python 提供了[smtplib](https://docs.python.org/3/library/smtplib.html)模块，该模块定义了一个 SMTP 客户端会话对象，该对象可用于将邮件发送到具有 SMTP 或 ESMTP 侦听器守护程序的任何 Internet 机器。

## 2. stmplib类与方法

```python
smtplib.SMTP([host[, port[, local_hostname[, timeout]]]])
```
作为SMTP的构造函数，功能是与smtp服务器建立连接，在连接成功后，就可以向服务器发送相关请求，比如登录、校验、发送、退出等。

 - `host`参数为远程smtp主机地址，比如smtp.163.com；
 - `port`为连接端口，默认为25；
 - `local_hostname`的作用是在本地主机的FQDN（完整的域名）发送HELO/EHLO（标识用户身份）指令；
 - `timeout`为连接或尝试在多少秒超时。

smtplib模块还提供了SMTP_SSL类和LMTP类，对它们的操作与SMTP基本一致。
smtplib.SMTP提供的方法：

 - `SMTP.set_debuglevel(level)`：设置是否为调试模式。默认为False，即非调试模式，表示不输出任何调试信息。
 - `SMTP.connect([host[, port]])`：连接到指定的smtp服务器。参数分别表示smpt主机和端口。例如：SMTP.connect（“smtp.163.com”，“25”）。
 - `SMTP.docmd(cmd[, argstring])`：向smtp服务器发送指令。可选参数argstring表示指令的参数。
 - `SMTP.helo([hostname])` ：使用"helo"指令向服务器确认身份。相当于告诉smtp服务器“我是谁”。
 - `SMTP.has_extn(name)`：判断指定名称在服务器邮件列表中是否存在。出于安全考虑，smtp服务器往往屏蔽了该指令。
 - `SMTP.verify(address)` ：判断指定邮件地址是否在服务器中存在。出于安全考虑，smtp服务器往往屏蔽了该指令。
 - `SMTP.login(user, password)`：登陆到smtp服务器。现在几乎所有的smtp服务器，都必须在验证用户信息合法之后才允许发送邮件。
 - `SMTP.sendmail(from_addr, to_addrs, msg[, mail_options,rcpt_options])`：方法，实现邮件的发送功能，参数依次为是发件人、收件人、邮件内容，例如：SMTP.sendmail（“python_2014@163.com”，“demo@domail.com”，body），其中body内容自定义。
 - `SMTP.starttls([keyfile[,certfile]])`：启用TLS（安全传输）模式，所有SMTP指令都将加密传输，例如使用gmail的smtp服务时需要启动此项才能正常发送邮件，如SMTP.starttls()。
 - SMTP.quit() ：断开与smtp服务器的连接，相当于发送"quit"指令。
 - smtp.close()

### 2.1 发送txt格式邮件

 - 连接 SMTP 服务器，并使用用户名、密码登录服务器。
 - 创建 mailMessage 对象，该对象代表邮件本身。
 - 调用代表与 SMTP 服务器连接的对象的 sendmail() 方法发送邮件。

示例：

```python
#!/usr/bin/python
#coding=gbk 
  
import smtplib 
import string

HOST = "smtp.xxx.com"
SUBJECT = "test email from Python"
TO = "xxx@163.com"
FROM = "xxxx@xxx.com"
text = "Python rules them all"
BODY = string.join((
        "From: %s" % FROM,
        "To: %s" % TO,
        "Subject: %s" % SUBJECT,
        "",
        text
        ), "\r\n")
server = smtplib.SMTP()
server.connect(HOST,"25")
server.starttls()
server.login("xxxx@xxx.com","paswd")
server.sendmail(FROM, [TO],BODY)
server.quit()
```

```python
[root@localhost stmplib]$ python stmp1.py
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200415224957640.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

##  3. email 简介
[email](https://docs.python.org/3/library/email.examples.html) 模块，使用该模块可以轻松的发送带图片、视频、附件等复杂内容的邮件。
## 4. 类与方法

 - `email.mime.multipart.MIMEMultipart([_subtype[, boundary[, _subparts[,_params]]]])`，作用是生成包含多个部分的邮件体的MIME对象，参数_subtype指定要添加到”Content-type:multipart/subtype”报头的可选的三种子类型，分别为`mixed、related、alternative`，默认值为mixed。定义mixed实现构建一个带附件的邮件体；定义related实现构建内嵌资源的邮件体；定义alternative则实现构建纯文本与超文本共存的邮件体。
 - `email.mime.audio.MIMEAudio (_audiodata[, _subtype[, _encoder[, **_params]]])`，创建包含音频数据的邮件体，_audiodata包含原始二进制音频数据的字节字符串。
 - `email.mime.image.MIMEImage(_imagedata[, _subtype[, _encoder[,**_params]]])`，创建包含图片数据的邮件体，_imagedata是包含原始图片数据的字节字符串。
 - `email.mime.text.MIMEText (_text[, _subtype[,_charset]])`，创建包含文本数据的邮件体，_text是包含消息负载的字符串，_subtype指定文本类型，支持plain（默认值）或html类型的字符串。
### 4.1 实现HTML格式邮件

```python
#!/usr/bin/python
#coding: utf-8
import smtplib
from email.mime.text import MIMEText
 
Host = "smtp.xxx.com"
From = "xxx@xxx.com"    
To = [ 'xxx@163.com', 'xxxx@qq.com' ]         #定义群组
#To = [ 'xxxx@163.com' ]         #定义个体
subject = u"官网流量数据报表"   
username = "xxxx@xxx.com"
password = "xxxx"
msg = MIMEText("""
     <table width="800" border="0" cellspacing="0" cellpadding="4">
     <tr>
        <td bgcolor="#CECFAD" height="20" style="font-size:14px">*官方数据 <a
 href="monitor.domain.com"> 更多>></a></td>
     </tr>
  
     <tr>
         <td bgcolor="#EFEBOE" height="100" style="font-size:13px">
          1)日访问量:<font color=red>153433</font> 访问次数: 23651 页面浏览量: 45322 点击数: 45353 数据流量: 454Mb<br>
          2) 状态码信息<br>
          3) 访问浏览器信息<br>
          4) 页面信息<br>
          &nbsp;&nbsp;/index.php 42153<br>
          &nbsp;&nbsp;/view.php 21451<br>
          &nbsp;&nbsp;/login.php 5112<br>
          </td>
      </tr>
    </table>""","html","utf-8")
msg['Subject'] = subject 
msg['From'] = From 
#msg['To'] = To
for to in To:   
    msg[to] = to
    
try:
    smtp = smtplib.SMTP()
    smtp.connect(Host,"25")
    smtp.starttls()
    smtp.login(username, password)
    smtp.sendmail(From, To, msg.as_string())
    smtp.quit()
    print('邮件发送成功！')
except Exception as e:
    print('邮件发送失败！'+str(e))
```

```python
[root@localhost stmplib]# python stmp3.py 
邮件发送成功！
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020041523505056.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
### 4.2 实现接受多个图片格式邮件

```python
#!/usr/bin/python
#coding: utf-8
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.mime.image import MIMEImage
 
Host = "smtp.xxxx.com"
From = "xxx@xxxx.com"    
To = [ 'xxxx@163.com', 'xxxxxxxx@qq.com' ]         #定义群组
To = [ 'xxxx@163.com' ]         #定义群组
subject = u"官网流量数据报表"   
username = "xxx@xxxx.com"
password = "xxx123."

def addimg(src, imgid):
    fp = open(src, 'rb')
    msgImage = MIMEImage(fp.read())
    fp.close()
    msgImage.add_header('Content-ID', imgid)
    return msgImage

msg = MIMEMultipart('related')
msgtext = MIMEText("""
     <table width="1200" border="0" cellspacing="0" cellpadding="4">
     <tr>
        <td bgcolor="#CECFAD" height="10" style="font-size:14px">*官方数据 <a
 href="monitor.domain.com"> 更多>></a></td>
     </tr>
  
     <tr>
         <td bgcolor="#EFEBOE" height="100" style="font-size:13px">
            <img src="cid:greenday1"></td><td>
            <img src="cid:greenday2"></td>
     </tr>
     <tr>
         <td bgcolor="#EFEBOE" height="100" style="font-size:13px">
            <img src="cid:greenday3"></td><td>
            <img src="cid:greenday4"></td>
     </tr>
            
    </table>""","html","utf-8")
msg.attach(msgtext)
msg.attach(addimg("/root/python/stmplib/photo/12.jpg","greenday1"))
msg.attach(addimg("/root/python/stmplib/photo/13.jpg","greenday2"))
msg.attach(addimg("/root/python/stmplib/photo/14.jpg","greenday3"))
msg.attach(addimg("/root/python/stmplib/photo/15.jpg","greenday4"))
msg['Subject'] = subject 
msg['From'] = From 
#msg['To'] = To
for to in To:   
    msg[to] = to
    
try:
    smtp = smtplib.SMTP()
    smtp.connect(Host,"25")
    smtp.starttls()
    smtp.login(username, password)
    smtp.sendmail(From, To, msg.as_string())
    smtp.quit()
    print('邮件发送成功！')
except Exception as e:
    print('邮件发送失败！'+str(e))
```

```python
[root@localhost stmplib]# python stmp4.py
邮件发送成功！
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200416002937134.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
### 4.3 实现接受单个图片格式邮件

```python
......
#!/usr/bin/python
#coding: utf-8
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.mime.image import MIMEImage

HOST = "smtp.xxxx.com" 
From = "xxx@xxxx.com"    
To = 'xxxx@163.com'      #定义群组
subject = "官网流量数据报表"   
username = "xxx@xxxx.com"
password = "xxxxx"
msg = MIMEMultipart('related') 
msg['Subject'] = subject 
msg['From'] = From 
msg['To'] = To
msgtext = MIMEText('</pre><h1>你好</h1><pre><img src="cid:greenday1"></td>','html','utf-8') 
msg.attach(msgtext)

fp = open('/root/python/stmplib/photo/12.jpg','rb')
msgimage = MIMEImage(fp.read())
fp.close() 
msgimage.add_header('Content-ID','greenday1')
msg.attach(msgimage)

try:
    smtp = smtplib.SMTP()
    smtp.connect(HOST)
    smtp.starttls()
    smtp.login(username, password)
    smtp.sendmail(From, [To], msg.as_string())
    smtp.quit()
    print('邮件发送成功！')
except Exception as e:
    print('邮件发送失败！'+str(e))
```
![!\[在这里插入图片描述\](https://img-blog.csdnimg.cn/20200416004456753.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,siz](https://img-blog.csdnimg.cn/20200416004547616.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
### 4.4 实现发送附件与图片格式的邮件

```python
#!/usr/bin/python
#coding: utf-8
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.mime.image import MIMEImage
 
Host = "smtp.xxxx.com"
From = "xxx@xxxx.com"    
To = [ 'xxxx@163.com', 'xxxxxxxxx@qq.com' ]         #定义群组
To = [ 'xxxx@163.com' ]         #定义群组
subject = u"2020年4月日报"   
username = "xxx@xxxx.com"
password = "xxxxxx"

def addimg(src, imgid):
    fp = open(src, 'rb')
    msgImage = MIMEImage(fp.read())
    fp.close()
    msgImage.add_header('Content-ID', imgid)
    return msgImage

msg = MIMEMultipart('related')
msgtext = MIMEText("<font color=red>xxxx2020年4月日报:<br><img src=\"cid:greenday1\" border=\"1\"></br>详细内容见附件。</font>","html","utf-8")
msg.attach(msgtext)
msg.attach(addimg("/root/python/stmplib/photo/12.jpg","greenday1"))

attach = MIMEText(open("/root/python/stmplib/2020年工作日报.xlsx", "rb").read(), "base64", "utf-8")
attach["Content-Type"] = "application/octet-stream"
attach["Content-Disposition"] = "attachment; filename=\"my_daily.xlsx\"".decode("utf-8").encode("gb18030")

msg.attach(attach)
msg['Subject'] = subject 
msg['From'] = From 
#msg['To'] = To
for to in To:   
    msg[to] = to
    
try:
    smtp = smtplib.SMTP()
    smtp.connect(Host,"25")
    smtp.starttls()
    smtp.login(username, password)
    smtp.sendmail(From, To, msg.as_string())
    smtp.quit()
    print('邮件发送成功！')
except Exception as e:
    print('邮件发送失败！'+str(e))
```

参考：

 - [Python - Sending Email using SMTP](https://www.tutorialspoint.com/python/python_sending_email.htm)
 - [Three Ways to Send Emails Using Python With Code Tutorials](https://www.courier.com/blog/three-ways-to-send-emails-using-python-with-code-tutorials/)
 - [Sending Email using Python in 5 statements](https://www.youtube.com/watch?v=BsVQ_cBmEwg)
 - [Python smtplib](https://zetcode.com/python/smtplib/)
 - [Python smtplib 教程](https://geek-docs.com/python/python-tutorial/python-smtplib.html)

