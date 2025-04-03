


----

##  1. 场景
上游要处理复杂的业务逻辑和强调开发效率，所以性能一般，通过nginx作为反向代理，通过请求算法、负载均衡代理给多个上游服务提供工作，实现了水平扩展，业务无感知的情况下添加更多的上游服务，增强服务处理性能，当上游服务出现问题时，可以自动的把请求从有问题的服务转交给正常服务。上游服务不对公网提供访问

##  2. 反向代理



修改配置监听 127.0.0.1:8080，那么这个上游服务只能本地进程访问

```bash
server {
    listen       127.0.0.1:8080;
```

停掉原有的nginx进程

```bash
$ ./sbin/nginx -s stop
```

启动

```bash
$ ./sbin/nginx

```
本地访问

```bash
$ curl http://localhost:8080
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN" "http://www.w3.org/TR/REC-html40/loose.dtd">
<html xmlns:gcse="googleCustomSearch"><head><meta http-equiv="Content-Type" content="text/html; charset=UTF-8"><link rel="shortcut icon" href="dlib-icon.ico"><meta name="verify-v1" content="02MiiaFNVzS5/u0eQhsy3/knioFHsia1X3DXRpHkE6I="><meta name="google-site-verification" content="DGSSJMKDomaDaDTIRJ8jDkv0YMx9Cz7OESbXHjjr6Jw"><title>dlib C++ Library</title><script type="text/javascript" src="dlib.js"></script><link rel="stylesheet" type="text/css" href="dlib.css"></head><body><a name="top"></a><div id="page_header"><a href="http://dlib.net"><img src="dlib-logo.png"></a></div><div id="top_content"><div id="main_menu
```

但跨机器访问不了
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4d5036324783be2966aebdf53167b11d.png)
我们可以用安装OpenRestry的配置作为反向代理，[安装OpenRestry](http://openresty.org/cn/installation.html)，实现用域名`ghostwritten.com`访问

修改配置
定义上游服务`local` ，为刚才配置server 127.0.0.1:8080，专为本地访问
```bash
    upstream local {
        server 127.0.0.1:8080
   }
```
定义server

```bash
server {
    listen       80;
    server_name  ghostwritten.com;
    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://local;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
```
有了反向代理，当我们拿一些变量的时候，可能会出错，因为变量值都是对单的，反向代理与客户端连接、反向代理与上游服务连接，所以在上游服务那里的变量地址取的其实是反向代理的地址而不是客户端远程地址，如果我想拿浏览器访问地址作为我限制浏览器流量访问速度等功能，其实是拿不到的。所以我们需要处理浏览器的一些值内容重置只能被上游服务所识别的变量。

 - `proxy_set_header Host $host`   表示把浏览器访问客户主机名`Host`设置为`$host` 
 - `proxy_set_header X-Real-IP $remote_addr`  表示把浏览器访问客户地址`X-Real-IP`设置为`$host` 
 - `proxy_pass http://local;`表示通过proxy_pass把代理的请求指定到名字为`local`上游服务。

[nginx代理特性与变量](https://nginx.org/en/docs/http/ngx_http_proxy_module.html)，这里可以查看设置更为详细的变量。


完整的配置文件

```bash
$ cat openresty/nginx/conf/nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    
    upstream local {
        server 127.0.0.1:8080;
    }
    server {
        listen       80;
        server_name  ghostwritten.com;
        location / {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://local;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```
启动

```bash
$ echo 127.0.0.1 ghostwritten.com >> /etc/hosts
$ ./openresty/bin/openresty
```
访问,本地已通。代理服务80反向代理上游服务server 127.0.0.1:8080

```bash
$ curl http://ghostwritten.com
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN" "http://www.w3.org/TR/REC-html40/loose.dtd">
<html xmlns:gcse="googleCustomSearch"><head><meta http-equiv="Content-Type" content="text/html; charset=UTF-8"><link rel="shortcut icon" href="dlib-icon.ico"><meta name="verify-v1" content="02MiiaFNVzS5/u0eQhsy3/knioFHsia1X3DXRpHkE6I="><meta name="google-site-verification" content="DGSSJMKDomaDaDTIRJ8jDkv0YMx9Cz7OESbXHjjr6Jw"><title>dlib C++ Library</title><script type="text/javascript" src="dlib.js"></script><link rel="stylesheet" type="text/css" href="dlib.css"></head><body><a name="top"></a><div id="page_header"><a href="http://dlib.net"><img src="dlib-logo.png"></a></div><div id="top_content"><div id="main_menu" class="menu"><div class="menu_top"><b>The Library</b><ul class="tree"><li><a href="algorithms.html" class="menu">Algorithms</a></li><li><a href="api.html" class="menu">API Wrappers</a></li><li><a href="bayes.html" class="menu">Bayesian Nets</a></li><li><a href="compression.html" class="menu">Compression</a></li><li><a href="containers.html" class="menu">Containers</a></li><li><a href="graph_tools.html" class="menu">Graph Tools</a></li><li><a href="imaging.html" class="menu">Image Processing</a></li><li><a href="linear_algebra.html" class="menu">Linear Algebra</a></li><li><a href="ml.html" class="menu">Machine Learning</a></li><li><a href="metaprogramming.html" class="menu">Metaprogramming</a></li><li><a href="other.html" class="menu">Miscellaneous</a></li><li><a href="network.html" class="menu">Networking</a></li><li><a href="optimization.html" class="menu">Optimization</a></li><li><a href="parsing.html" class="menu">Parsing</a></li></ul><br><b>Help/Info</b><ul class="tree"><li><a href="http://blog.dlib.net" class="menu">Dlib Blog</a></li><li><a
```
但夸机器windows即便在`C:\Windows\System32\drivers\etc`目录添加`192.168.211.100 ghostwritten.com`，也无法访问。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/aedf659bdaff6a3660f8b8b0faf1fe44.png)
如果实现跨机器访问

```bash
echo 192.168.211.100 ghostwritten.com >>   /etc/hosts
```
或
windows即便在`C:\Windows\System32\drivers\etc`目录添加`192.168.211.100 ghostwritten.com`

上游服务nginx的配置文件修改

```bash
$  vim nginx/conf/nginx.conf
..........
    server {
        listen       8080;
        server_name  localhost;
.....
```

反向代理`OpenRestry`的配置文件修改

```bash
$ vim openresty/nginx/conf/nginx.conf
.......
    upstream local {
        server 192.168.211.100:8080;
    }
......
```
当然，上游服务名`local`，可以修改你自定义的名字，并且在`proxy_pass http://[上游服务名]`也要修改保持一致。我这里暂保持默认简化修改内容。

重启

```bash
$ ./openresty/bin/openresty -s reload
$ ./nginx/sbin/nginx -s reload
```
界面，域名访问成功，我们可以在消息头看的`OpenRestry`的版本
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/eb563eb55f7883ca8ea37ff633539240.png)

##  3. 代理缓存
当我们的nginx作为反向代理时，通常有动态请求，不同的用户访问同一个URL会看到不同的内容，这时会交给上游服务处理，而当不同的用户访问静态资源时，即一段时间不会发生变化的，我们可以不交给上游服务，缓存到nginx代理，我们可以设置缓存一天，或者几个小时，即便上游服务挂了，我们仍然可以浏览到一些资源。

修改openrestry的配置

```bash
$ cat openresty/nginx/conf/nginx.conf | grep -v '#' |sed '/^$/d'
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
   
    client_max_body_size 60M;
    proxy_cache_path /tmp/nginxcache levels=1:2 keys_zone=my_cache:10m max_size=10g inactive=60m use_temp_path=off;
    sendfile        on;
    tcp_nopush     on;
    keepalive_timeout  65;
    gzip  on;
    gzip_min_length 1;
    gzip_comp_level 2;
    gzip_types text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-php
        image/jpeg image/gif image/png;
 
    upstream host {
        server 192.168.211.100:8080;
    }
    server {
        listen       80;
        server_name  ghostwritten.com;
        location / {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_cache my_cache;
            proxy_cache_key $host$uri$is_args$args;
            proxy_cache_valid 200 304 302 1d;
            proxy_pass http://host;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```
配置说明：

```bash
proxy_cache_path /tmp/nginxcache levels=1:2 keys_zone=my_cache:10m max_size=10g inactive=60m use_temp_path=off;
```

 - `levels` 设置缓存文件目录层次；levels=1:2 表示两级目录
 - `keys_zone` 设置缓存名字和共享内存大小
 - `inactive` 在指定时间内没人访问则被删除，默认是1天
 - `max_size` 最大缓存空间，如果缓存空间满，默认覆盖掉缓存时间最长的资源
 - `proxy_temp_path` : 使用temp_path存储，如果不使用，则配置在`max_size`后
   use_temp_path=off;

```bash
proxy_cache my_cache;  #根keys_zone后的内容对应
proxy_cache_key $host$uri$is_args$args;  #通过key来hash，定义KEY的值
proxy_cache_valid 200 304 302 1d; #哪些状态缓存多长时间
proxy_cache_min_uses 3; #只要统一个url，不管间隔多久，总次数访问到达3次，就开始缓存。
proxy_cache_bypass $cookie_nocache $arg_nocache$arg_comment; # 如果任何一个参数值不为空，或者不等于0，nginx就不会查找缓存，直接进行代理转发
proxy_cache_methods GET;  # 默认是get和head
```

重启

```bash
./bin/openresty -s reload
```
访问界面
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cdb839de46a527cc5d28cf492b871f73.png)
停止上游服务nginx

```bash
./nginx/sbin/nginx -s stop
```
仍然可以访问
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d4a55826018277460ba6ee097ad3279f.png)

