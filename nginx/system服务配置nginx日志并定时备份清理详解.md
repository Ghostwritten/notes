### 相关文章
[linux Systemd详解](https://blog.csdn.net/xixihahalelehehe/article/details/104844189)
[linux log rotation日志滚动详解](https://blog.csdn.net/xixihahalelehehe/article/details/105131841)
[/etc/init.d/functions运用实战配置systemd nginx服务详解](https://blog.csdn.net/xixihahalelehehe/article/details/106637054)

在配置system nginx服务后，对其nginx日志进行自定义并定时备份清理。

### 1. 配置systemd配置文件
```bash
$ mkdir  -p   /etc/systemd/system/nginx.service.d/
$ vim etc/systemd/system/nginx.service.d/nginx.conf
[Service]
SyslogIdentifier=nginx
```
### 2. rsyslog配置单独的日志

```bash
$ vim /etc/rsyslog.d/nginx.conf
if $programname == "nginx" then /var/log/nginx/nginx.log
& stop
```

### 3. logrotate配置定时备份并清理策略

```bash
$ vim /etc/logrotate.d/nginx
/var/log/nginx/nginx.log {
        weekly
        rotate 8
        missingok
        notifempty
        size 10M
        create 0644 root root
}
```
### 4. 重启服务

```bash
$ systemctl restart rsyslog
$ systemctl status nginx
```

### 5.查看日志是否生效

```bash
$  cat /var/log/nginx/nginx.log 
Jun 25 13:20:38 monitor1 nginx: /etc/rc.d/init.d/nginx: line 29: kill: /usr/local/nginx/sbin/nginx: arguments must be process or job IDs
Jun 25 13:20:38 monitor1 nginx: nginx is already running
```

