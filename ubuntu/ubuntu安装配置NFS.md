

---
 - [ubuntu16.04、18.04、21.10配置静态ip并修改网卡名](https://ghostwritten.blog.csdn.net/article/details/105641217)
 - [ubuntu安装配置NFS](https://ghostwritten.blog.csdn.net/article/details/117217493)
 - [ubuntu ssh详解](https://ghostwritten.blog.csdn.net/article/details/105639355)
 - [ubuntu apt-get镜像源](https://ghostwritten.blog.csdn.net/article/details/110931517)
 - [ubutntu iso下载](https://al.mirror.kumi.systems/ubuntureleases/)

----

## 1. 服务端配置
```c
root@master:~/cks# apt-get install nfs-kernel-server
root@master:~/cks# vim /etc/exports
/root/cks 192.168.211.0/24(rw,sync,no_subtree_check,no_root_squash)


root@master:~/cks# systemctl start nfs-utils.service 
root@master:~/cks# systemctl status nfs-utils.service 
● nfs-utils.service - NFS server and client services
   Loaded: loaded (/lib/systemd/system/nfs-utils.service; static; vendor preset: enabled)
   Active: active (exited) since Sun 2021-05-23 19:00:59 PDT; 3s ago
  Process: 66563 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
 Main PID: 66563 (code=exited, status=0/SUCCESS)

May 23 19:00:59 master systemd[1]: Starting NFS server and client services...
May 23 19:00:59 master systemd[1]: Started NFS server and client services.
root@master:~/cks# systemctl enable nfs-utils.service 
root@master:~/cks# exportfs -rv
exporting 192.168.211.0/24:/root/cks



```
## 2. 客户端配置

```c
root@node1:~# apt-get install nfs-common
root@node1:~# showmount -e 192.168.211.40
Export list for 192.168.211.40:
/root/cks 192.168.211.0/24


root@node1:~# mkdir cks
#临时挂载
root@node1:~# mount -t nfs 192.168.211.40:/root/cks /root/cks

#永久挂载
root@node1:~# echo "192.168.211.40:/root/cks /root/cks nfs defaults 0 0" >> /etc/fstab
root@node1:~# mount -a

root@node1:~# df -Th |grep 192.16
192.168.211.40:/root/cks nfs4       19G   17G  777M  96% /root/cks

```

