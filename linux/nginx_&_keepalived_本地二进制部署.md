![在这里插入图片描述](https://img-blog.csdnimg.cn/37daffd90a1c46e7ab1ae004f528a2a3.png)



## 安装 nginx


```bash
wget http://nginx.org/download/nginx-1.24.0.tar.gz
tar -xf nginx-1.24.0.tar.gz
cd nginx-1.24.0/
./configure --with-stream --prefix=/usr/local/nginx
make && make install
```

修改nginx配置文件
创建nginx日志存放目录

```bash
$ mkdir /var/log/nginx

$ vi /usr/local/nginx/conf/nginx.conf
worker_processes  2;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  10240;
}

stream {
    upstream apiserver {
        server 192.168.1.11:6443 weight=5 max_fails=3 fail_timeout=30s;
        server 192.168.1.12:6443 weight=5 max_fails=3 fail_timeout=30s;
        server 192.168.1.13:6443 weight=5 max_fails=3 fail_timeout=30s;
    }

    server {
        listen 16443;
        proxy_connect_timeout 15s;
        proxy_timeout 15s;
        proxy_pass apiserver;
    }

    log_format proxy    '$remote_addr [$time_local] '
                        '$protocol $status $bytes_sent $bytes_received '
                        '$session_time "$upstream_addr" '
                        '"$upstream_bytes_sent" "$upstream_bytes_received" "$upstream_connect_time"';

    access_log /var/log/nginx/access.log proxy;
}
```

配置nginx启动服务文件

```bash
$ vi /usr/lib/systemd/system/nginx.service
[Unit]
Description=nginx
After=network.target
  
[Service]
Type=forking
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s quit
PrivateTmp=true
  
[Install]
WantedBy=multi-user.target
```

启动并检查nginx

```bash
systemctl daemon-reload;systemctl enable nginx.service;systemctl restart nginx.service;systemctl status nginx
```

## 安装 keepalived
安装keepalived软件,二进制安装

```bash
wget https://www.keepalived.org/software/keepalived-2.2.8.tar.gz
tar -xf keepalived-2.2.8.tar.gz
cd keepalived-2.2.8
./configure --prefix=/usr/local/keepalived-2.2.8
make && make install

ln -s /usr/local/keepalived-2.2.8 /usr/local/keepalived
mkdir /etc/keepalived/
cp /usr/local/keepalived/etc/keepalived/keepalived.conf /etc/keepalived/
cp /usr/local/keepalived-2.2.8/etc/sysconfig/keepalived /etc/sysconfig/
cp /root/keepalived-2.2.8/keepalived/keepalived.service /etc/systemd/system/
ln -s /usr/local/keepalived-2.2.8/sbin/keepalived /usr/sbin/
cp /root/keepalived-2.2.8/keepalived/etc/init.d/keepalived /etc/init.d/ 
chmod 755 /etc/init.d/keepalived
systemctl enable keepalived.service
```

修改配置文件,配置文件略有不同，因为这个采用了非抢占模式
1master1节点配置

```bash
$ vi /etc/keepalived/keepalived.conf
! Configuration File for keepalived
global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id 192.168.1.11
}
vrrp_script check_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 5
    weight -60
    fall 2
    rise 2
}
vrrp_instance VI_1 {
    state BACKUP
    nopreempt
    interface ens160
    virtual_router_id 56 # VRRP 路由 ID实例，每个实例是唯一的
    priority 80   # 优先级，备服务器设置 90
    advert_int 1    # 指定VRRP 心跳包通告间隔时间，默认1秒
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    # 虚拟IP
    virtual_ipaddress {
        192.168.1.10/24
    }
    track_script {
        check_nginx
    }
}
```

其余节点需要修改priority优先级即可，其他不用动

master2节点配置

```bash
$ vi /etc/keepalived/keepalived.conf
global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id 192.168.1.12
}
vrrp_script check_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 5
    weight -60
    fall 2
    rise 2
}
vrrp_instance VI_1 {
    state BACKUP
    nopreempt
    interface ens160
    virtual_router_id 56 # VRRP 路由 ID实例，每个实例是唯一的
    priority 90   # 优先级，备服务器设置 90
    advert_int 1    # 指定VRRP 心跳包通告间隔时间，默认1秒
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    # 虚拟IP
    virtual_ipaddress {
        192.168.1.10/24
    }
    track_script {
        check_nginx
    }
}

```

master3节点配置

```bash
$ vi /etc/keepalived/keepalived.conf
global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id 192.168.1.13
}
vrrp_script check_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 5
    weight -60
    fall 2
    rise 2
}
vrrp_instance VI_1 {
    state BACKUP
    nopreempt
    interface ens160
    virtual_router_id 56 # VRRP 路由 ID实例，每个实例是唯一的
    priority 100   # 优先级，备服务器设置 90
    advert_int 1    # 指定VRRP 心跳包通告间隔时间，默认1秒
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    # 虚拟IP
    virtual_ipaddress {
        192.168.1.10/24
    }
    track_script {
        check_nginx
    }
}
```

配置检测脚本

```bash
$ vi /etc/keepalived/check_nginx.sh
#!/bin/bash

# if check error then repeat check for 12 times, else exit
# 检测次数可以适当调整
err=0
for k in $(seq 1 2)
do
    check_code=$(curl -k http://localhost:16443)
    if [[ $check_code == "" ]]; then
        err=$(expr $err + 1)
        sleep 5
        continue
    else
        err=0
        break
    fi
done

if [[ $err != "0" ]]; then
    # if apiserver is down send SIG=1
    echo 'nginx error!'
    systemctl stop   keepalived
    exit 1
else
    # if apiserver is up send SIG=0
    echo 'nginx ok'
fi

chmod +x /etc/keepalived/check_nginx.sh
```
启动并验证keepalived

```bash
systemctl enable keepalived ; systemctl restart keepalived
```

##  卸载 nginx

```bash
systemctl stop nginx
rm -rf /var/log/nginx/
rm -rf /usr/local/nginx/
rm -rf /usr/lib/systemd/system/nginx.service
```
##  卸载 keepalived

```bash
ls /usr/local/keepalived-2.2.8 && rm -rf /usr/local/keepalived-2.2.8
ls /usr/local/keepalived && rm -rf /usr/local/keepalived
ls /etc/keepalived/ && rm -rf /etc/keepalived/
ls /etc/sysconfig/keepalived && rm -rf  /etc/sysconfig/keepalived
ls /etc/systemd/system/keepalived.service && rm -rf /etc/systemd/system/keepalived.service 
ls /etc/init.d/keepalived && rm -rf  /etc/init.d/keepalived
```

