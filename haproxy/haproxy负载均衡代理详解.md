

-----
## 1. 介绍
HAProxy提供`高可用性`、`负载均衡`以及基于`TCP`和`HTTP`应用的代理，支持虚拟主机，它是免费、快速并且可靠的一种解决方案。根据官方数据，其最高极限支持`10G`的并发。

HAProxy特别适用于那些负载特大的web站点， 这些站点通常又需要`会话保持或七层`处理。HAProxy运行在当前的硬件上，完全可以支持数以`万计`的并发连接。并且它的运行模式使得它可以很简单安全的整合进您当前的架构中， 同时可以保护你的web服务器不被暴露到网络上。

其支持从4层至7层的网络交换，即覆盖所有的TCP协议。就是说，Haproxy 甚至还支持 `Mysql` 的均衡负载。。

如果说在功能上，能以proxy反向代理方式实现 WEB均衡负载，这样的产品有很多。包括 `Nginx，ApacheProxy，lighttpd，Cheroke` 等。但要明确一点的，Haproxy 并不是 Http 服务器。以上提到所有带反向代理均衡负载的产品，都清一色是 WEB 服务器。简单说，就是他们能自个儿提供静态(html,jpg,gif..)或动态(php,cgi..)文件的传输以及处理。而Haproxy 仅仅，而且专门是一款的用于均衡负载的应用代理。其自身并不能提供http服务。

但其配置简单，拥有非常不错的服务器健康检查功能还有专门的系统状态监控页面，当其代理的后端服务器出现故障, HAProxy会自动将该服务器摘除，故障恢复后再自动将该服务器加入。自`1.3版本`开始还引入了`frontend,backend,`frontend根据任意HTTP请求头内容做规则匹配，然后把请求定向到相关的backend。

## 2. 功能

 - `内容交换` : 可以根据`请求(request)的任何一部分` 来选择一组服务器, 比如请求的 URI , Host头(header) , cookie , 以及其他任何东西. 当然，对那些静态分离的站点来说，对此特性还有更多的需求。
 - `全透明代理` : `可以用 客户端IP地址 或者任何其他地址`来连接后端服务器. 这个特性仅在Linux2.4/2.6内核打了cttproxy 补丁后才可以使用. 这个特性也使得为某特殊服务器处理部分流量同时又不修改服务器的地址成为可能。
 - `基于树的更快的调度器` : 1.2.16以上的版本要求所有的超时都设成同样的值以支持数以万计的全速连接. 这个特性已经移植到1.2.17.
 - `内核TCP拼接` : 避免了内核到用户然后用户到内核端的数据拷贝, 提高了吞吐量同时又降低了CPU使用率 . Haproxy1.3支持Linux L7SW 以满足在商用硬件上数Gbps 的吞吐的需求。
 - `连接拒绝` : 因为维护一个连接的打开的开销是很低的，有时我们很需要限制攻击蠕虫(attack bots)，也就是说限制它们的连接打开从而限制它们的危害。 这个已经为一个陷于小型DDoS攻击的网站开发了而且已经拯救了很多站点。
 - `细微的头部处理` : 使得编写基于header的规则更为简单，同时可以处理URI的某部分。 快而可靠的头部处理 : 使用完全RFC2616兼容的完整性检查对一般的请求全部进行分析和索引仅仅需要不到2ms 的时间。
 - `模块化设计` : 允许更多人加入进此项目，调试也非常简单. poller已经分离, 已经使得它们的开发简单了很多，HTTP已经从TCP分离出来了，这样增加新的七层特性变得非常简单. 其他子系统也会很快实现模块化。
 - `投机I/O 处理` : 在一个套接字就绪前就尝试从它读取数据。poller仅推测哪个可能就绪哪个没有，尝试猜测，并且如果成功，一些开销很大的系统调用就可以省去了。如果失败，就会调用这些系统调用。已知的使用Linux epoll()已经净提升起码10%了。
 - `ACLs` : 使用任意规则的任意组合作为某动作的执行条件。
 - `TCP 协议检查` : 结合ACL来对请求的任意部分进行检查，然后再进行转发。这就可以执行协议验证而不是盲目的进行转发。比如说允许SSL但拒绝SSH。
 - `更多的负载均衡算法` : 现在，动态加权轮循(Dynamic Round Robin)，加权源地址哈希(Weighted Source Hash)，加权URL哈希和加权参数哈希(Weighted Parameter Hash)已经实现。其他算法比如Weighted Measured Response Time也很快会实现。
----
## 3. 安装部署
### 3.1 系统环境

```bash
Oracle Linux Server 7.4
web1 server : 192.168.1.120
web2 server : 192.168.1.121
haproxy server: 192.168.1.121
```

### 3.2 安装web服务
web1 server执行：

```bash
$ yum -y install httpd
$ echo web1 > /var/www/html/index.html
$ systemctl start httpd && systemctl enable httpd
```
web2 server执行：

```bash
$ yum -y install httpd
$ echo web2 > /var/www/html/index.html
$ systemctl start httpd && systemctl enable httpd
```
### 3.3 安装haproxy
官网：[http://www.haproxy.org/](http://www.haproxy.org/)

```bash
$ yum -y install gcc c++
$ wget http://www.haproxy.org/download/2.1/src/haproxy-2.1.4.tar.gz
$ tar -zxvf haproxy-2.1.4.tar.gz
$ cd haproxy-2.1.4
$ uname -r 
3.10.0-693.el7.x86_64
$ make TARGET=linux3100 ARCH=x86_64 PREFIX=/usr/local/haproxy
$ make install PREFIX=/usr/local/haproxy

参数说明：
使用uname -r查看内核，如：3.10.0-693.el7.x86_64，此时该TARGET参数就为linux3100
ARCH=x86_64 #系统位数
PREFIX=/usr/local/haprpxy #/usr/local/haprpxy为haprpxy安装路径
```

如果是YUM安装的话配置文件会自动生成在/etc/haproxy/haproxy.cfg。如果是编译安装的就可以通过复制安装包的模板文件做修改，这样可以方便快速手动配置，并且为haproxy创建一个系统用户。Haproxy的日志在/var/log/message里。

```bash
$ useradd -r haproxy
$ cp ./examples/haproxy.init /etc/init.d/haproxy
$ chmod 755 /etc/init.d/haproxy
$ cp ./examples/option-http_proxy.cfg /usr/local/haproxy/haproxy.cfg
```
### 3.4 Haproxy日志配置
创建日志目录

```bash
$ mkdir /usr/local/haproxy/logs
```

系统设置接收外来日志

```bash
$ vim /etc/sysconfig/rsyslog
SYSLOGD_OPTIONS="-r -m 0 -c 2"
```

 - -m  0 : 表示给日志添加-- MARK --标记，0表示关闭标记
 - -r :  选项以允许接受外来日志消息
 - -c : 指定rsyslog运行(兼容)的版本号, 当然也可以省略, 默认为-c 0, (命令行兼容sysklogd)
开启rsyslog的日志记录功能

```bash
$ vim /etc/rsyslog.conf  #Haproxy默认是把日志输出到/var/log/message中，不太方便查阅，所以配置文件中有配置rsyslog
$ModLoad imudp
$UDPServerRun 514
local2.*           /usr/local/haproxy/logs/haproxy.log
#或者 local2.*           /var/log/haproxy.log
```
重启rsyslog生效

```bash
$ systemctl restart rsyslog
```
### 3.5Haproxy简单配置
通过该Haproxy实现后端两台web节点轮询

```bash
$ cat /usr/local/haproxy/haproxy.cfg
```

```bash
global
    log         127.0.0.1 local2
    chroot      /usr/local/haproxy
    pidfile     /usr/local/haproxy/logs/haproxy.pid
    maxconn     4000
    ulimit-n    65535
    user        haproxy
    group       haproxy
    daemon
    nbproc      1

defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8  #记录客户端真实IP
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s  #haproxy连接后端服务器超时时间
    timeout client          10m  #客户端和haproxy的非活动超时时间
    timeout server          10m  #haproxy和后端服务器的非活动超时时间
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000

frontend  test
    bind  *:8080
    default_backend             test

backend test
    balance     roundrobin
    server  app1 192.168.1.120:80 weight 1 check inter 2000 fall 3 rise 3
    server  app2 192.168.1.121:80 weight 1 check inter 2000 fall 3 rise 3 

listen mysqlserver  #名称随意写
    bind 0.0.0.0:3306
    balance roundrobin
    server server1 192.168.1.190:3306 weight 1
    server server2 192.168.1.191:3306 weight 1

listen  admin_stats  #配置状态监控页面
  bind 0.0.0.0:8000
  mode http
  stats refresh 30s
  stats uri /status
  stats auth admin:admin123
  stats hide-version
  stats admin if TRUE
```
### 3.6 启动服务

```bash
$ /usr/local/haproxy/sbin/haproxy -f /usr/local/haproxy/haproxy.cfg
```
### 3.7 查看Haproxy状态监控
浏览器：http://192.168.1.121:8000/status
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/760032a97a4687ae8de3e18be2b36d81.png)
### 3.8 查看haproxy代理的web
浏览器：http://192.168.1.121:8080
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/08f6cc4576f45533929ef96176432f26.png)
刷新一次
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f804dc6fe42cb9356a9e583740bc7d7e.png)
## 4. 模板配置文件详解

```c
####################全局配置信息########################
#######参数是进程级的，通常和操作系统（OS）相关#########
global
       maxconn 20480                   #默认最大连接数
       log 127.0.0.1 local3            #[err warning info debug]
       chroot /var/haproxy             #chroot运行的路径
       uid 99                          #所属运行的用户uid
       gid 99                          #所属运行的用户组
       daemon                          #以后台形式运行haproxy
       nbproc 1                        #进程数量(可以设置多个进程提高性能)
       pidfile /var/run/haproxy.pid    #haproxy的pid存放路径,启动进程的用户必须有权限访问此文件
       ulimit-n 65535                  #ulimit的数量限制
       #####################默认的全局设置######################
       ##这些参数可以被利用配置到frontend，backend，listen组件##
defaults
       log global
       mode http                       #所处理的类别 (#7层 http;4层tcp  )
       maxconn 20480                   #最大连接数
       option httplog                  #日志类别http日志格式
       option httpclose                #每次请求完毕后主动关闭http通道
       option dontlognull              #不记录健康检查的日志信息
       option forwardfor               #如果后端服务器需要获得客户端真实ip需要配置的参数，可以从Http Header中获得客户端ip
       option redispatch               #serverId对应的服务器挂掉后,强制定向到其他健康的服务器
       option abortonclose             #当服务器负载很高的时候，自动结束掉当前队列处理比较久的连接
       stats refresh 30                #统计页面刷新间隔
       retries 3                       #3次连接失败就认为服务不可用，也可以通过后面设置
       balance roundrobin              #默认的负载均衡的方式,轮询方式
      #balance source                  #默认的负载均衡的方式,类似Nginx的ip_hash
      #balance leastconn               #默认的负载均衡的方式,最小连接
       contimeout 5000                 #连接超时
       clitimeout 50000                #客户端超时
       srvtimeout 50000                #服务器超时
       timeout check 2000              #心跳检测超时
       ####################监控页面的设置#######################
listen admin_status                    #Frontend和Backend的组合体,监控组的名称，按需自定义名称
        bind 0.0.0.0:65532             #监听端口
        mode http                      #http的7层模式
        log 127.0.0.1 local3 err       #错误日志记录
        stats refresh 5s               #每隔5秒自动刷新监控页面
        stats uri /admin?stats         #监控页面的url
        stats realm itnihao itnihao   #监控页面的提示信息
        stats auth admin:admin         #监控页面的用户和密码admin,可以设置多个用户名
        stats auth admin1:admin1       #监控页面的用户和密码admin1
        stats hide-version             #隐藏统计页面上的HAproxy版本信息
        stats admin if TRUE            #手工启用/禁用,后端服务器(haproxy-1.4.9以后版本)
       errorfile 403 /etc/haproxy/errorfiles/403.http
       errorfile 500 /etc/haproxy/errorfiles/500.http
       errorfile 502 /etc/haproxy/errorfiles/502.http
       errorfile 503 /etc/haproxy/errorfiles/503.http
       errorfile 504 /etc/haproxy/errorfiles/504.http
       #################HAProxy的日志记录内容设置###################
       capture request  header host           len 40
       capture request  header Content-Length len 10
       capture request  header Referer        len 200
       capture response header Server         len 40
       capture response header Content-Length len 10
       capture response header Cache-Control  len 8
       #######################网站监测listen配置#####################
       ###########此用法主要是监控haproxy后端服务器的监控状态############
listen site_status
       bind 0.0.0.0:1081                    #监听端口
       mode http                            #http的7层模式
       log 127.0.0.1 local3 err             #[err warning info debug]
       monitor-uri /site_status             #网站健康检测URL，用来检测HAProxy管理的网站是否可以用，正常返回200，不正常返回503
       acl site_dead nbsrv(server_web) lt 2 #定义网站down时的策略当挂在负载均衡上的指定backend的中有效机器数小于1台时返回true
       acl site_dead nbsrv(server_blog) lt 2
       acl site_dead nbsrv(server_bbs)  lt 2
       monitor fail if site_dead            #当满足策略的时候返回503，网上文档说的是500，实际测试为503
       monitor-net 192.168.16.2/32          #来自192.168.16.2的日志信息不会被记录和转发
       monitor-net 192.168.16.3/32
       ########frontend配置############
       #####注意，frontend配置里面可以定义多个acl进行匹配操作########
frontend http_80_in
       bind 0.0.0.0:80      #监听端口，即haproxy提供web服务的端口，和lvs的vip端口类似
       mode http            #http的7层模式
       log global           #应用全局的日志配置
       option httplog       #启用http的log
       option httpclose     #每次请求完毕后主动关闭http通道，HA-Proxy不支持keep-alive模式
       option forwardfor    #如果后端服务器需要获得客户端的真实IP需要配置次参数，将可以从Http Header中获得客户端IP
       ########acl策略配置#############
       acl itnihao_web hdr_reg(host) -i ^(www.itnihao.cn|ww1.itnihao.cn)$
       #如果请求的域名满足正则表达式中的2个域名返回true -i是忽略大小写
       acl itnihao_blog hdr_dom(host) -i blog.itnihao.cn
       #如果请求的域名满足www.itnihao.cn返回true -i是忽略大小写
       #acl itnihao    hdr(host) -i itnihao.cn
       #如果请求的域名满足itnihao.cn返回true -i是忽略大小写
       #acl file_req url_sub -i  killall=
       #在请求url中包含killall=，则此控制策略返回true,否则为false
       #acl dir_req url_dir -i allow
       #在请求url中存在allow作为部分地址路径，则此控制策略返回true,否则返回false
       #acl missing_cl hdr_cnt(Content-length) eq 0
       #当请求的header中Content-length等于0时返回true
      ########acl策略匹配相应#############
       #block if missing_cl
       #当请求中header中Content-length等于0阻止请求返回403
       #block if !file_req || dir_req
       #block表示阻止请求，返回403错误，当前表示如果不满足策略file_req，或者满足策略dir_req，则阻止请求
       use_backend  server_web  if itnihao_web
       #当满足itnihao_web的策略时使用server_web的backend
       use_backend  server_blog if itnihao_blog
       #当满足itnihao_blog的策略时使用server_blog的backend
       #redirect prefix http://blog.itniaho.cn code 301 if itnihao
       #当访问itnihao.cn的时候，用http的301挑转到http://192.168.16.3
       default_backend server_bbs
       #以上都不满足的时候使用默认server_bbs的backend
       ##########backend的设置##############
       #下面我将设置三组服务器 server_web，server_blog，server_bbs
###########backend server_web#################
backend server_web
       mode http            #http的7层模式
       balance roundrobin   #负载均衡的方式，roundrobin平均方式
       cookie SERVERID      #允许插入serverid到cookie中，serverid后面可以定义
       option httpchk GET /index.html #心跳检测的文件
       server web1 192.168.16.2:80 cookie web1 check inter 1500 rise 3 fall 3 weight 1
       #服务器定义，cookie 1表示serverid为web1，check inter 1500是检测心跳频率rise 3是3次正确认为服务器可用，
       #fall 3是3次失败认为服务器不可用，weight代表权重
       server web2 192.168.16.3:80 cookie web2 check inter 1500 rise 3 fall 3 weight 2
       #服务器定义，cookie 1表示serverid为web2，check inter 1500是检测心跳频率rise 3是3次正确认为服务器可用，
       #fall 3是3次失败认为服务器不可用，weight代表权重
#############backend server_blog##############
backend server_blog
       mode http            #http的7层模式
       balance roundrobin   #负载均衡的方式，roundrobin平均方式
       cookie SERVERID      #允许插入serverid到cookie中，serverid后面可以定义
       option httpchk GET /index.html #心跳检测的文件
       server blog1 192.168.16.2:80 cookie blog1 check inter 1500 rise 3 fall 3 weight 1
       #服务器定义，cookie 1表示serverid为web1，check inter 1500是检测心跳频率rise 3是3次正确认为服务器可用，fall 3是3次失败认为服务器不可用，weight代表权重
       server blog2 192.168.16.3:80 cookie blog2 check inter 1500 rise 3 fall 3 weight 2
        #服务器定义，cookie 1表示serverid为web2，check inter 1500是检测心跳频率rise 3是3次正确认为服务器可用，fall 3是3次失败认为服务器不可用，weight代表权重
#############backend server_bbs##############
backend server_bbs
       mode http            #http的7层模式
       balance roundrobin   #负载均衡的方式，roundrobin平均方式
       cookie SERVERID      #允许插入serverid到cookie中，serverid后面可以定义
       option httpchk GET /index.html #心跳检测的文件
       server bbs1 192.168.16.2:80 cookie bbs1 check inter 1500 rise 3 fall 3 weight 1
       #服务器定义，cookie 1表示serverid为web1，check inter 1500是检测心跳频率rise 3是3次正确认为服务器可用，fall 3是3次失败认为服务器不可用，weight代表权重
       server bbs2 192.168.16.3:80 cookie bbs2 check inter 1500 rise 3 fall 3 weight 2
        #服务器定义，cookie 1表示serverid为web2，check inter 1500是检测心跳频率rise 3是3次正确认为服务器可用，fall 3是3次失败认为服务器不可用，weight代表权重
```
更新的配置详解参考 [马哥教育](https://blog.51cto.com/mageedu/1744420)


参考连接：
[http://www.liangxiansen.cn/2017/03/06/haproxy/](http://www.liangxiansen.cn/2017/03/06/haproxy/)
[http://www.linuxe.cn/post-379.html](http://www.linuxe.cn/post-379.html)
[https://www.cnblogs.com/f-ck-need-u/p/8540805.html](https://www.cnblogs.com/f-ck-need-u/p/8540805.html)
[https://www.cnblogs.com/MacoLee/p/5853413.html](https://www.cnblogs.com/MacoLee/p/5853413.html)

[https://docs.pingcap.com/zh/tidb/stable/haproxy-best-practices#haproxy-%E5%9C%A8-tidb-%E4%B8%AD%E7%9A%84%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5](https://docs.pingcap.com/zh/tidb/stable/haproxy-best-practices#haproxy-%E5%9C%A8-tidb-%E4%B8%AD%E7%9A%84%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5)

