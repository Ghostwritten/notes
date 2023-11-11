

## 1. 下载 dlib
[dlib](http://dlib.net/)
```bash
$ wget http://dlib.net/files/dlib-19.23.zip
$ unzip dlib-19.23.zip
$ rm -rf dlib-19.23.zip
$ ls -l
总用量 0
drwx------ 2 nobody root   6 2月   7 21:54 client_body_temp
drwxr-xr-x 2 root   root 333 2月   7 21:53 conf
drwxrwxr-x 7 root   root 226 1月  25 11:16 dlib-19.23
drwx------ 2 nobody root   6 2月   7 21:54 fastcgi_temp
drwxr-xr-x 2 root   root  40 2月   7 12:11 html
drwxr-xr-x 2 root   root  82 2月   7 22:01 logs
drwx------ 2 nobody root   6 2月   7 21:54 proxy_temp
drwxr-xr-x 2 root   root  19 2月   7 12:11 sbin
drwx------ 2 nobody root   6 2月   7 21:54 scgi_temp
drwx------ 2 nobody root   6 2月   7 21:54 uwsgi_temp

$ mv dlib-19.23/docs dlib

```
##  2. 自定义配置

```bash
$ vim conf/nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       8080;
        server_name  localhost;
        access_log  logs/host.access.log  main;
        location / {
             alias  dlib/;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}

```

 - `alias` 参数使nginx url路径与dlib目录的路径保持一至

## 3. 启动

```bash
./sbin/nginx -s reload
```
界面
![在这里插入图片描述](https://img-blog.csdnimg.cn/e1af598a07f0402fae03ce8be657ad9c.png)
##  4. 模块
### 4.1 gzip压缩

F12打开网络抓包，刷新
![在这里插入图片描述](https://img-blog.csdnimg.cn/c85f7781d87648cdaab381632f0fb904.png)
首页传输的大小是26.6KB与文件的大小是一致的

```bash
$ ls -l dlib/index.html
-rwxr-xr-x 1 root root 26341 2月   7 23:42 dlib/index.html
```
我们可以把所有文件压缩传输，再通过gzip解压打开

```bash
$ cat conf/nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    gzip  on;
    gzip_min_length 1;
    gzip_comp_level 2;
    gzip_types text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-php
        image/jpeg image/gif image/png;
    server {
        listen       8080;
        server_name  localhost;
        access_log  logs/host.access.log  main;
        location / {
             alias  dlib/;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```
重启
```bash
./sbin/nginx -s reload
```

界面查看，首页传输大小只有8KB
![在这里插入图片描述](https://img-blog.csdnimg.cn/94eb7bfab79b4661aa7f9a7263965de4.png)
从响应头我们看的gzip方法
![在这里插入图片描述](https://img-blog.csdnimg.cn/867baf61974b4a6da148992f5e445a73.png)

###  4.2 audoindex目录显示
[http://nginx.org/en/docs/http/ngx_http_autoindex_module.html](http://nginx.org/en/docs/http/ngx_http_autoindex_module.html)
该ngx_http_autoindex_module`在这里插入代码片`模块处理以斜杠字符 (' /') 结尾的请求并生成目录列表。`ngx_http_autoindex_module` 通常，当 `ngx_http_index_module`模块找不到索引文件 时，会将请求传递给模块。

```bash
  location / {
             alias  dlib/;
             autoindex  on;
        }
```
效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/590c9acfcea84b3bad20b0ca0aa9cdbc.png)
### 4.3 limit_rate限制流量访问
[limit_rate介绍](http://nginx.org/en/docs/http/ngx_http_core_module.html#limit_rate)
试用场景：当某些客户访问大文件的时候，非常占用流量，但又要保持访问小文件的其他客户正常访问，所以就需要进行限速。

    location / {
         alias  dlib/;
         autoindex  on;
         limit_rate 1k;
    }

**清理缓存**后，查看效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/8b3692cd8791403cb7b0d7bf6ed2e5c7.png)
之前是54ms，现在7s，慢了许多，但它能兼顾大部分客户耐心。

### 4.4  log_format日志格式

```bash
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';
```

 - main是为日志格式命名，为什么要命名，为了针对不同域名下日志自定义，或者大文件、url等做一些特殊分析处理。
 - remote_addr  远程地址
 - remote_user  远程用户
 - time_local  时间
 - .....

```bash
server{
......
        access_log  logs/host.access.log  main;
......
```
参数配置即是记录某个域名服务选择以什么类型的日志，以什么样的格式输出到哪个目录文件。这里是以`access_log`类型日志，以`main`格式输出到`logs/host.access.log`，main在http块下有定义配置。

```bash
$ cat logs/host.access.log
192.168.211.1 - - [08/Feb/2022:00:28:10 +0800] "GET /dlib-logo.png HTTP/1.1" 200 5742 "http://192.168.211.100:8080/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.99 Safari/537.36" "-"
192.168.211.1 - - [08/Feb/2022:00:28:11 +0800] "GET /dlib-icon.ico HTTP/1.1" 200 1150 "http://192.168.211.100:8080/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.99 Safari/537.36" "-"
192.168.211.1 - - [08/Feb/2022:00:28:12 +0800] "GET /dlib-icon.ico HTTP/1.1" 200 1150 "http://192.168.211.100:8080/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.99 Safari/537.36" "-"
```
当然，我们其实把很多变量添加到main格式中，比如[ngx_http_core_module的变量](http://nginx.org/en/docs/http/ngx_http_core_module.html#variables)的`$connection`、`$content_length`.......,还有刚才运用到的第三方模块`ngx_http_gzip_module`，它存在一个变量`$gzip_ratio`，即gzip的压缩比例。
