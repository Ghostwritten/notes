


## 1. 命令参数
![在这里插入图片描述](https://img-blog.csdnimg.cn/673ec6a39384420e96a2f7bc5ca0372f.png)
## 2. 重载配置文件

```bash
$ vim  conf/nginx.conf
.......
tcp_nopush     on;  #注释取消
.........

$ ./nginx -s reload
```

## 3. 热部署

```bash
$ ps -ef |grep nginx
root     1077779       1  0 21:54 ?        00:00:00 nginx: master process ./nginx
nobody   1077906 1077779  0 21:54 ?        00:00:00 nginx: worker process
root     1078508  389304  0 21:55 pts/0    00:00:00 grep --color=auto nginx
#备份nginx二进制
$ cp sbin/nginx sbin/nginx.old
#拷贝新nginx二进制覆盖原来的nginx
$ cp $new/nginx sbin/nginx
#给master进程发送
$ kill -USR2 1077779 
$ ps -ef |grep nginx
root     1077779       1  0 21:54 ?        00:00:00 nginx: master process ./nginx
nobody   1077906 1077779  0 21:54 ?        00:00:00 nginx: worker process
root     1085282 1077779  0 22:01 ?        00:00:00 nginx: master process ./nginx
nobody   1085283 1085282  0 22:01 ?        00:00:00 nginx: worker process
root     1085777  389304  0 22:02 pts/0    00:00:00 grep --color=auto nginx

#worker process死掉，说明新的请求连接到新的nginx中
#为什么有两个master process，因为还有一个master进程是老版本的，为了版本回退
$ ps -ef |grep nginx
root     1077779       1  0 21:54 ?        00:00:00 nginx: master process ./nginx
root     1085282 1077779  0 22:01 ?        00:00:00 nginx: master process ./nginx
nobody   1085283 1085282  0 22:01 ?        00:00:00 nginx: worker process
root     1087809  389304  0 22:03 pts/0    00:00:00 grep --color=auto nginx

```

##  4. 切割日志文件
把access.log切割

```bash
$ ls logs/
access.log  error.log  nginx.pid  nginx.pid.oldbin
$ mv logs/access.log logs/bak.log
$ ./sbin/nginx -reopen
#重新生成access.log,不影响启动的nginx服务
$ ls logs/
access.log  error.log  nginx.pid  nginx.pid.oldbin
```
实际生产中，我们会每一天、或每一周做切割，所以可以写一个bash脚本
例如：

```bash
#!/bin/bash

#初始化

LOGS_PATH=/opt/nginx

YESTERDAY=$(date -d "yesterday" +%Y%m%d)

#按天切割日志

mv ${LOGS_PATH}/access.log ${LOGS_PATH}/access_${YESTERDAY}.log

#向nginx主进程发送USR1信号，重新打开日志文件，否则会继续往mv后的文件写数据的。原因在于：linux系统中，内核是根据文件描述符来找文件的。如果不这样操作导致日志切割失败。

kill -USR1 `ps axu | grep "nginx: master process" | grep -v grep | awk '{print $2}'`

#删除7天前的日志

cd ${LOGS_PATH}

find . -mtime +7 -name "*20[1-9][3-9]*" | xargs rm -f

#或者

#find . -mtime +7 -name "access_*" | xargs rm -f

exit 0
```

