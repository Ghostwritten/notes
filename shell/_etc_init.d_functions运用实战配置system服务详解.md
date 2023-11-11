[/etc/init.d/functions详解](https://www.cnblogs.com/image-eye/archive/2011/10/26/2220405.html)

```bash
functions这个脚本是给/etc/init.d里边的文件使用的。提供了一些基础的功能，看看里边究竟有些什么。首先会设置umask，path，还有语言环境，然后会设置success,failure,warning,normal几种情况下的字体颜色。下面再看看提供的重要方法：
checkpid:检查是否已存在pid，如果有一个存在，返回0（通过查看/proc目录）
daemon:启动某个服务。/etc/init.d目录部分脚本的start使用到这个
killproc:杀死某个进程。/etc/init.d目录部分脚本的stop使用到这个
pidfileofproc:寻找某个进程的pid
pidofproc:类似上面的，只是还查找了pidof命令
status:返回一个服务的状态
echo_success,echo_failure,echo_passed,echo_warning分别输出各类信息
success,failure,passed,warning分别记录日志并调用相应的方法
action:打印某个信息并执行给定的命令，它会根据命令执行的结果来调用 success,failure方法
strstr:判断$1是否含有$2
confirm:显示 "Start service $1 (Y)es/(N)o/(C)ontinue? [Y]"的提示信息，并返回选择结果
```

## 练习#调用函数
#调用函数
```bash
source /etc/init.d/functions
. /etc/init.d/functions
```

#以守护进程形式启动

```bash
daemon /usr/local/nginx-1.16.0/sbin/nginx
```

#退出当前进程

```c
killproc /usr/local/nginx-1.16.0/sbin/nginx
```

#查看进程

```bash
pidofproc  /usr/local/nginx-1.16.0/sbin/nginx
```

nginx.sh

```bash
#!/bin/bash

# chkconfig: - 85 15

# description: nginx is a World Wide Web server. It is used to serve
. /etc/init.d/functions

exec=/usr/local/nginx/sbin/nginx
lock=/var/lock/subsys/nginx
prog=nginx
RETVAL=0

start(){
     pidofproc $exec > /dev/null
     [ $? = 0 ] && echo "$prog is already running" && exit
     daemon $exec
     [ $? = 0 ] && echo  "start $prog success" && touch $lock
}

reload(){
     pidofproc $exec > /dev/null
     [ $? = 0 ] && echo "$prog is running" && killproc $exec -HUP
     [ $? = 0 ] && echo "$prog does not run" && daemon $exec 
}

stop(){
      pidofproc $exec > /dev/null
      [ $? != 0 ] && echo "$prog have been stopped" && exit
      kill $exec 
      [ $? = 0 ] && echo "stop $prog success" && rm -rf $lock
}

case $1 in
 
     start)
       start
     ;;
    
     stop)
       stop
     ;;

     restart)
       stop
       start
     ;;
  
     reload)
       reload
     ;;
  
     status)
       status nginx
     ;;

     *)
      echo "USAGE: nginx {start | stop | restart | reload | status}"
     ;;
esac

exit 0

```

复制脚本到init.d目录

```bash
cp /home/shell/nginx.sh  /etc/init.d/nginx
```

查看当前系统启动数据

```bash
chkconfig --list
systemctl list-unit-files
```
添加服务到系统服务

```bash
chkconfig --add  nginx
```

服务操作

```bash
$ service  nginx start
Starting nginx (via systemctl):                            [  OK  ]
```

```bash
$ systemctl status nginx
● nginx.service - SYSV: nginx is a World Wide Web server. It is used to serve
   Loaded: loaded (/etc/rc.d/init.d/nginx; bad; vendor preset: disabled)
   Active: active (exited) since Thu 2020-06-11 16:11:25 CST; 4min 14s ago
     Docs: man:systemd-sysv-generator(8)

Jun 11 16:11:25 monitor1 systemd[1]: Starting SYSV: nginx is a World Wide Web server. It is used to serve...
Jun 11 16:11:25 monitor1 nginx[1361]: nginx is already running
Jun 11 16:11:25 monitor1 systemd[1]: Started SYSV: nginx is a World Wide Web server. It is used to serve.
```
其他操作
```bash

service nginx start
service nginx restart
service nginx reload
service nginx stop
systemctl start nginx
systemctl restart nginx
systemctl reload nginx 
systemctl status nginx
systemctl stop nginx
```

参考连接：
[https://blog.51cto.com/11726705/2403439](https://blog.51cto.com/11726705/2403439)


