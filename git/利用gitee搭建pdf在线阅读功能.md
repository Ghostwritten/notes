

##  1. 简介
pdf在线阅读功能，可以存储整理自己的小存量pdf，并可以快速浏览，以及在编写文章中进行链接引用，也方便路人下载与观看。

 - 利用gitee就可以实现pdf在线阅读功能，这是我的[mimipdf](https://gitee.com/ghostwritten/mimipdf)仓库。
 - 查看在线阅读pdf效果：[LexingtonClassAircraftCarrier.pdf](https://ghostwritten.gitee.io/mimipdf/web/viewer.html?file=LexingtonClassAircraftCarrier.pdf)

##  2. 工具

 - [码云](https://gitee.com/)Pages(gitee pages)是一个免费的静态网页托管服务, 除此之外你还可以使用gitee pages托管博客、项目官网等.之后我们将使用gitee pages来托管pdf.js. 当然github同样可以实现这样的效果。
 - [pdf.js](https://mozilla.github.io/pdf.js/getting_started/#download)是一款使用HTML5 Canvas安全地渲染pdf文件以及遵从网页标准的网页浏览器渲染pdf文件的javascript库.该插件不需要任何本地支持，对浏览器的兼容性也比较好.

##  3. 注册gitee并创建仓库
注册[gitee](https://gitee.com/)

本地运行
```bash
$ ssh-keygen -t rsa -C 'xxxxx@outlook.com'
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/XH/.ssh/id_rsa):
/c/Users/XH/.ssh/id_rsa already exists.
Overwrite (y/n)? y
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/XH/.ssh/id_rsa.
Your public key has been saved in /c/Users/XH/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:OY+Ek3Ww1gKxW0fC5bvbcQ2XlHT2kBaGb+YpPQn7qxM xxxxxx@outlook.com
The key's randomart image is:
+---[RSA 3072]----+
|      ooo.o  .++o|
|       o.B  ..+o+|
|      . * =  o o.|
|       B = .. = .|
|      = S .  B.+.|
|       o + .E *+ |
|        . o .+...|
|           o.o.  |
|          . oo.. |
+----[SHA256]-----+


$ cat /c/Users/XH/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC6PcgwDYK//ZRdY48q8C1kKw7OsdGggUQff1GW/e3JE6RqWdIZN1Y
................................
pt9HhYq3f/ocvbnx0RHJcs1F82lYZZh7iLZwHhxV5L47pwHLs8YkJ5WM8PnRtOymS0JNuoaam+rXp5ORhY3+ATsk7bcNVA6eNLb2Z+IWKemdoSWs8Jt/XDthJ/B8Jp5z3kDmGsClZ0UHpsgPY/i2IrpXk= xxxxx@outlook.com

```
![在这里插入图片描述](https://img-blog.csdnimg.cn/cf044404647444ebb59f52be0d436f5b.png)

##  4. 初始化mimipdf库
在本地创建空项目[mimipdf](https://gitee.com/ghostwritten/mimipdf),并初始化git

```bash
mkdir mimipdf
cd mimipdf
git init
```
在gitee创建名为mimipdf的仓库, 本地连接到远程仓库

```bash
git config --global user.name "xxxxx"
git config --global user.email "xxxxx@outlook.com"
git remote add origin https://gitee.com/xxxx/mimipdf.git
```
下载[pdf.js](https://mozilla.github.io/pdf.js/getting_started/#download)的源码,并解压到本地Npdf仓库下.

```bash
XH@DESKTOP-2FKN21J MINGW64 /f/gitee/mimipdf (master)
$ ls pdfjs-2.13.216-dist/
build/  LICENSE  web/
```
将你自己要展示的`pdf`文件放入web文件夹下,将文件上传到gitee的仓库

##  5. 上传更新内容

```bash
git add *
git commit -m "add pdf.js"
git push origin master
```

##  6. 创建Gitee Pages
gitee的`mimipdf`仓库中选择Service下的`Gitee Pages.`进入选择创建Pages.

![在这里插入图片描述](https://img-blog.csdnimg.cn/32e9c694ef41462d9acfd64cf7a32212.png)
当然，你必须有实名认证的条件

 - 真实姓名
 - 身份证号
 - 身份证正反照片
 - 手持身份证

![在这里插入图片描述](https://img-blog.csdnimg.cn/cd4b6b2397bd428db8951c71a5524ce7.png)
当认证通过以后，最终可以生成如下内容(部署成功)
![在这里插入图片描述](https://img-blog.csdnimg.cn/801727ce7dd648f9919e6a4cbcbc267b.png)
直接点击启动即可。

访问我的pdf书籍：

```bash
https://ghostwritten.gitee.io/mimipdf/web/viewer.html?file=LexingtonClassAircraftCarrier.pdf
```
如图在线PDF：
![在这里插入图片描述](https://img-blog.csdnimg.cn/f15d534d082b432480c9299086c0d835.png)

---
![在这里插入图片描述](https://img-blog.csdnimg.cn/aeded96672dd466a988a15c18a1599eb.gif#pic_center)


