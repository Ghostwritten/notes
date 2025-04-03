

---
##  1. 简介
access日志记录了Nginx非常重要的信息,我们可以使用Nginx来分析定位问题;也可以用它来分析用户的运行数据;但是如果想要实时分析access.log相对比较困难

有一款工具叫[GoAccess](https://goaccess.io/),它可以以图像化的方式通过WebSoxket协议实时的把access.log的变迁反应到浏览器中方便我们分析问题;接下来我们来演示如何使用GoAccess工具来分析Nginx的access.log的日志中。

样板：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/03e11914a4a05ba010cee19c0175a821.png)
默认access.log

```bash
$ tail nginx/logs/host.access.log 
192.168.211.100 - - [08/Feb/2022:14:50:09 +0800] "GET / HTTP/1.0" 200 26341 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.99 Safari/537.36" "192.168.211.1"
192.168.211.100 - - [08/Feb/2022:14:50:10 +0800] "GET /dlib-icon.ico HTTP/1.0" 200 1150 "http://ghostwritten.com/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.99 Safari/537.36" "192.168.211.1"
192.168.211.100 - - [08/Feb/2022:14:59:54 +0800] "GET /dlib.js HTTP/1.0" 200 2240 "http://ghostwritten.com/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.99 Safari/537.36" "192.168.211.1"
192.168.211.100 - - [08/Feb/2022:14:59:54 +0800] "GET /plus.gif HTTP/1.0" 200 59 "http://ghostwritten.com/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.99 Safari/537.36" "192.168.211.1"
192.168.211.100 - - [08/Feb/2022:14:59:58 +0800] "GET /dlib.css HTTP/1.0" 200 6634 "http://ghostwritten.com/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.99 Safari/537.36" "192.168.211.1"
192.168.211.100 - - [08/Feb/2022:14:59:59 +0800] "GET /dlib-logo.png HTTP/1.0" 200 5701 "http://ghostwritten.com/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.99 Safari/537.36" "192.168.211.1"
192.168.211.100 - - [08/Feb/2022:15:00:17 +0800] "GET / HTTP/1.0" 200 26341 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.99 Safari/537.36" "192.168.211.1"
192.168.211.100 - - [08/Feb/2022:15:00:18 +0800] "GET /dlib-icon.ico HTTP/1.0" 200 1150 "http://ghostwritten.com/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.99 Safari/537.36" "192.168.211.1"
192.168.211.100 - - [08/Feb/2022:15:00:19 +0800] "GET /dlib-icon.ico HTTP/1.0" 200 1150 "http://ghostwritten.com/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.99 Safari/537.36" "192.168.211.1"
192.168.211.100 - - [08/Feb/2022:15:45:34 +0800] "GET / HTTP/1.0" 200 26341 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.99 Safari/537.36" "192.168.211.1"
```
##  2. 安装goaccess
不同方式安装[请参考](https://goaccess.io/download#distro)

```bash
 yum install goaccess
```

##  3. 启动goaccess　　

 - `--log-format=COMBINED`
也就是说nginx的access.log非常的灵活,我们可以自己添加各种不同的各模块的内置变量加入到access.log中;所以当我们修改了access.log的格式的时候,我们需要在`--log-format=COMBINED`中重新定义我们添加的格式;在这个例子中,我们没有添加任何的access.log的配置,那么GoAccess是怎么使用的尼？它实际上会去使用`goaccess access.log -o report.html --log-format=COMBINED` 中-o这个参数,生成一个新的html文件;把当前我们access.log日志中的内容以html图表的形式展示出来;那么当access.log变迁的时候尼？GoAccess会新起一个websocket进程通过端口的形式把新的access.log形式推送到我们的客户端;其中access.log要与自己定义的名字相对应比如我这里配置文件中定义的为`host.access.log`

接下来我们还要在上游服务器的`nginx.conf`中添加一个location,每当我们访问`/report.html`的时候;我们需要用alias把它重定向到代理服务器的`report.html`中;

上游服务配置添加：

```bash
$ vim nginx/conf/nginx.conf
.......
        location /report.html {
            alias /opt/openresty/nginx/html/report.html;
        }
.......
```
启动带参数的goaccess

```bash
$ goaccess nginx/logs/host.access.log  -o /opt/openresty/nginx/html/report.html --real-time-html --time-format='%H:%M:%S'--date-format='%d/%b/%Y' --log-format=COMBINED
```
默认goaccess在开启实时`real-time-html`后会监听端口`7890`的websocket，如果服务器不允许请求7890端口，你就看不到那个页面是实时更新的——你会发现访问的页面最后更新时间始终不变。这一点人很多忽略了，很多人以为是哪个生成html静态文件是实时更新的，其实根本不是，那个文件本身一旦生成就不动了，真正更新的实时内容是从websocket过来的。
访问代理域名后缀加`/report.html`
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1696d3adc457920c63cbf7ae4d88b04c.png)
当我访问一次`http://192.168.211.100:8080/`，goaccess界面会自动更新。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ba92ad74d0fcb401c245cf8b948b5571.png)

