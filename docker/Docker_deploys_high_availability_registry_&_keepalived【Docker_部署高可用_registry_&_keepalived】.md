![](https://i-blog.csdnimg.cn/blog_migrate/dc79534f9e670b83c4c41b8c2f3286be.jpeg#pic_center)



## 1. 简介

私有镜像仓库可以方便企业，或个人开发者共享内部镜像而不会泄漏私有代码，而且可以加速镜像的拉取。能更加方便地集成到容器化的 CI/CD 中去。也可建立自己的公共镜像仓库。 

 Docker Registry的优点如下：
 - Docker Registry的最大优点就是简单，只需要运行一个容器就能集中管理一个集群范围内的镜像，其他机器就能从该镜像仓库下载镜像了。 
- 在安全性方面，Docker Registry支持TLS和基于签名的身份验证。
- Docker Registry也提供了Restful API，以提供外部系统调用和管理镜像库中的镜像


## 2. 架构


## 3. 预备条件

| 角色 | ip  | cpu| mem| disk|ios|kernel|
|--|--|--|--|--|--|--|
|registry01 |192.168.23.51  |4  |8G|100G| Rocky 8.8 |4.18+|
| registry02 |192.168.23.52 |4  |8G|100G|Rocky 8.8|4.18+|
| push01 |192.168.23.53 |4  |8G|100G|Rocky 8.8|4.18+|


## 4. 配置/etc/hosts

192.168.23.51节点
```bash
#192.168.23.51 registry01.dev.com
#192.168.23.52 registry01.dev.com
192.168.23.50 registry01.dev.com
```
192.168.23.52节点
```bash
#192.168.23.51 registry01.dev.com
#192.168.23.52 registry01.dev.com
192.168.23.50 registry01.dev.com
```

## 5. 安装 registry

- [设置 docker 存储目录](https://cblog.csdn.net/xixihahalelehehe/article/details/134633286)
- [安装 docker](https://blog.csdn.net/xixihahalelehehe/article/details/104293170)
- [docker install private registry](https://blog.csdn.net/xixihahalelehehe/article/details/136256102)[这里我选择它，一键创建]
- [docker registry仓库私搭并配置证书](https://blog.csdn.net/xixihahalelehehe/article/details/105926147)


192.168.23.51节点执行部署完镜像仓库后，拷贝证书至192.168.23.52节点

```bash
scp -r /etc/docker/certs.d/registry01.dev.com:80 root@192.168.23.52:/etc/docker/certs.d/
scp -r /registry/certs/ root@192.168.23.52:/registry/
```

192.168.23.52节点安装镜像仓库。

```bash
docker run -d --privileged=true --restart=always --name registry-tls-certs -v /registry/certs:/certs -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/registry01.dev.com.crt -e REGISTRY_HTTP_TLS_KEY=/certs/registry01.dev.com.key -e REGISTRY_COMPATIBILITY_SCHEMA1_ENABLED=true -e REGISTRY_STORAGE_DELETE_ENABLED=true -p 443:443 -p 80:5000 -v /data/registry:/var/lib/registry/docker/registry registry
```

## 6. yum 安装 keepalived

两节点安装keepalived
```bash
yum -y install keepalived
```
192.168.23.51 主节点配置文件

```bash
cat > /etc/keepalived/keepalived.conf <<EOF
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
   router_id 192.168.23.51
}
vrrp_script check_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 5
    weight -60
    fall 2
    rise 2
}
vrrp_instance VI_1 {
    state MASTER
    interface ens192
    virtual_router_id 55 # VRRP 路由 ID实例，每个实例是唯一的
    priority 100    # 优先级，备服务器设置 90
    advert_int 1    # 指定VRRP 心跳包通告间隔时间，默认1秒
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    # 虚拟IP
    virtual_ipaddress {
        192.168.23.50/20l
    }
    track_script {
        check_nginx
    }
}
EOF
```

192.168.23.52 从节点配置文件（两节点配置文件不一样奥）

```bash
cat > /etc/keepalived/keepalived.conf <<EOF
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
   router_id 192.168.23.52
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
    interface ens192
    virtual_router_id 55 # VRRP 路由 ID实例，每个实例是唯一的
    priority 95
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.23.50/20
    }
    track_script {
        check_nginx
    }
}
EOF
```

两节点配置检查脚本

```bash
cat > /etc/keepalived/check_nginx.sh<<EOF 
#!/bin/bash
# if check error then repeat check for 12 times, else exit
# 检测次数可以适当调整
err=0
for k in $(seq 1 2)
do
    check_code=$(curl -k http://localhost:80)
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
    echo 'nginx error!'
    systemctl stop   keepalived
    exit 1
else
    echo 'nginx ok'
fi
EOF
```

192.168.23.51与192.168.23.52 分别启动服务

```bash
systemctl start keepalived $$ systemctl enable keepalived && systemctl status keepalived
```

虚拟网卡创建成功,目前vip（192.168.23.50）在192.168.23.51节点。

```bash
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:81:df:01 brd ff:ff:ff:ff:ff:ff
    inet 192.168.23.51/20 brd 192.168.31.255 scope global noprefixroute ens192
       valid_lft forever preferred_lft forever
    inet 192.168.23.50/20 scope global secondary ens192
```

测试推送镜像

```bash
$ docker pull busybox:latest
latest: Pulling from library/busybox
9ad63333ebc9: Pull complete 
Digest: sha256:6d9ac9237a84afe1516540f40a0fafdc86859b2141954b4d643af7066d598b74
Status: Downloaded newer image for busybox:latest
docker.io/library/busybox:latest

$ docker tag  busybox:latest registry01.dev.com:80/busybox:latest
$ docker push registry01.dev.com:80/busybox:latest
The push refers to repository [registry01.dev.com:80/busybox]
2e112031b4b9: Pushed 
latest: digest: sha256:d319b0e3e1745e504544e931cde012fc5470eba649acc8a7b3607402942e5db7 size: 527
```

## 7. 验证vip漂移

### 7.1 原主坏测试推送拉取镜像

1、停止主节点上的镜像仓库容器，模拟宕机，检查节点是否漂移到备用节点（192.168.23.52）。



```bash
$ docker ps
CONTAINER ID   IMAGE      COMMAND                   CREATED          STATUS          PORTS                                                                          NAMES
28bd293407b1   registry   "/entrypoint.sh /etc…"   51 minutes ago   Up 51 minutes   0.0.0.0:443->443/tcp, :::443->443/tcp, 0.0.0.0:80->5000/tcp, :::80->5000/tcp   registry-tls-certs
$ docker stop registry-tls-certs
registry-tls-certs

#过20秒，发现vip消失。
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:81:df:01 brd ff:ff:ff:ff:ff:ff
    inet 192.168.23.51/20 brd 192.168.31.255 scope global noprefixroute ens192
       valid_lft forever preferred_lft forever
    inet6 fe80::7e7d:1560:124e:cb76/64 scope link dadfailed tentative noprefixroute 
       valid_lft forever preferred_lft forever
```
登陆192.168.23.52节点，发现新的vip已启动。说明vip飘逸成功。

```bash
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:81:3c:0f brd ff:ff:ff:ff:ff:ff
    inet 192.168.23.52/20 brd 192.168.31.255 scope global noprefixroute ens192
       valid_lft forever preferred_lft forever
    inet 192.168.23.50/20 scope global secondary ens192
       valid_lft forever preferred_lft forever
    inet6 fe80::7e7d:1560:124e:cb76/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

接下来，在主节点的镜像仓库坏掉的情况下，我们拉取公网新的惊喜测试推送镜像

```bash
$ docker pull busybox:1.35.0
1.35.0: Pulling from library/busybox
2354422721e4: Pull complete 
Digest: sha256:02289a9972c5024cd2f083221f6903786e7f4cb4a9a9696f665d20dd6892e5d6
Status: Downloaded newer image for busybox:1.35.0
docker.io/library/busybox:1.35.0
$ docker tag busybox:1.35.0 registry01.dev.com:80/busybox:1.35.0
$ docker push registry01.dev.com:80/busybox:1.35.0
The push refers to repository [registry01.dev.com:80/busybox]
3e5d40008713: Pushed 
1.35.0: digest: sha256:163c71948c2d428e7f8f7901eb2d2fae677c50be454eda4fe8fb4596ee3f1d31 size: 527
```
测试拉取镜像也没问题

```bash
docker pull registry01.dev.com:80/busybox:latest
latest: Pulling from busybox
Digest: sha256:d319b0e3e1745e504544e931cde012fc5470eba649acc8a7b3607402942e5db7
Status: Image is up to date for registry01.dev.com:80/busybox:latest
registry01.dev.com:80/busybox:latest
```

### 7.2 原主恢复自动抢回vip

2、恢复主节点的registry服务，检查是vip是否被抢占回去，为何会当主正常被抢，因为两者权重（priority），主是100，从是95.

登陆192.168.23.51启动 registry
```bash
$ docker start registry-tls-certs
#过20秒,vip被抢回。
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:81:df:01 brd ff:ff:ff:ff:ff:ff
    inet 192.168.23.51/20 brd 192.168.31.255 scope global noprefixroute ens192
       valid_lft forever preferred_lft forever
    inet 192.168.23.50/20 scope global secondary ens192
       valid_lft forever preferred_lft forever
```

再次测试上传新镜像。

```bash
$ docker pull busybox:1.36.1
1.36.1: Pulling from library/busybox
Digest: sha256:6d9ac9237a84afe1516540f40a0fafdc86859b2141954b4d643af7066d598b74
Status: Downloaded newer image for busybox:1.36.1
docker.io/library/busybox:1.36.1
$ docker tag busybox:1.36.1 registry01.dev.com:80/busybox:1.36.1
$ docker push registry01.dev.com:80/busybox:1.36.1
The push refers to repository [registry01.dev.com:80/busybox]
2e112031b4b9: Layer already exists 
1.36.1: digest: sha256:d319b0e3e1745e504544e931cde012fc5470eba649acc8a7b3607402942e5db7 size: 527
```
结束。

## 8. 镜像同步

防止镜像介质占用过多的存储空间，我们单独选择一台机器（192.168.23.53）负责推送镜像。如何保障192.168.23.51和192.168.23.52的镜像介质相同呢，registry没有像harbor那么智能，可以做到实时或者定时同步。那么在推送镜像的时候，我们可以指定节点传输介质，传两次（悲催，不过可以写个脚本一键执行）。

- 镜像批量推送方法参考：[容器镜像搬运最佳脚本](https://blog.csdn.net/xixihahalelehehe/article/details/135082812)

修改/etc/hosts，执行push一次。
```bash
192.168.23.51 registry01.dev.com
#192.168.23.52 registry01.dev.com
#192.168.23.50 registry01.dev.com
```
再修改/etc/hosts，执行push一次。
```bash
#192.168.23.51 registry01.dev.com
192.168.23.52 registry01.dev.com
#192.168.23.50 registry01.dev.com
```

如何检查镜像的一致性。

- [部署一个registry-ui 界面查看](https://blog.csdn.net/xixihahalelehehe/article/details/107406198)
- [调用接口进行统计展示镜像列表](https://blog.csdn.net/xixihahalelehehe/article/details/105926147)

参考：
- https://www.cnblogs.com/deny/p/14184953.html
- https://www.cnblogs.com/cqqfboy/p/14503549.html


![](https://i-blog.csdnimg.cn/blog_migrate/fd1a917ea0db8c4138741e9c3900b514.jpeg#pic_center)

