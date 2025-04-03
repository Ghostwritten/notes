![](https://i-blog.csdnimg.cn/blog_migrate/421eaa0bda6326eeec628b3770e7ea73.jpeg#pic_center)






## 1. 定义配置
```bash
$ mkdir /root/haproxy
$ vim /root/haproxy/haproxy.cfg
defaults
    mode            tcp
    log             global
    option          tcplog
    option          dontlognull
    option http-server-close
    option          redispatch
    retries         3
    timeout http-request 10s
    timeout queue   1m
    timeout connect 10s
    timeout client  1m
    timeout server  1m
    timeout http-keep-alive 10s
    timeout check   10s
    maxconn         3000
frontend    upm-ui
    bind        0.0.0.0:32010
    mode        tcp
    log         global
    default_backend upm_server

backend     upm_server
    balance roundrobin
    server upm1 172.168.16.21:32010 check inter 5s rise 2 fall 3
    server upm2 172.168.16.21:32010 check inter 5s rise 2 fall 3

listen stats
    mode    http
    bind    0.0.0.0:1080
    stats   enable
    stats   hide-version
    stats uri /haproxyamdin?stats
    stats realm Haproxy\ Statistics
    stats auth admin:admin
    stats admin if TRUE
```
    
 
## 2. 启动容器

```bash
 $ nerdctl  run -p 1080:1080 -p 32010:32010 -d --name haproxy-master -v /root/haproxy/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg --privileged=true haproxy
```

## 3. 查看日志

```bash
$ nerdctl logs haproxy-master
[NOTICE]   (1) : haproxy version is 2.9.7-5742051
[NOTICE]   (1) : path to executable is /usr/local/sbin/haproxy
[WARNING]  (1) : config : log format ignored for frontend 'upm-ui' since it has no log address.
[WARNING]  (1) : config : log format ignored for proxy 'stats' since it has no log address.
[NOTICE]   (1) : New worker (8) forked
[NOTICE]   (1) : Loading success.
```

## 4. 测试访问

```bash
$  curl http://192.168.24.13:32010
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8"/>
....
    }
</style>
<body>
<div id="app" style="overflow: auto;"></div>

</body>
</html>
```

参考：

- [https://www.haproxy.org/download/2.9/src/](https://www.haproxy.org/download/2.9/src/)


