httpd离线升级
old-version：

 - httpd-2.4.6-95.el7.centos.x86_64.rpm

new-version：

 - httpd-2.4.6-97.el7.centos.x86_64.rpm

```bash
已经安装的旧版本
===========================================================================================================================================
 Package                   Arch                 Version                              Repository                                       Size
===========================================================================================================================================
Installing:
 httpd                     x86_64               2.4.6-95.el7.centos                  /httpd-2.4.6-95.el7.centos.x86_64               9.4 M
Installing for dependencies:
 apr                       x86_64               1.4.8-7.el7                          base                                            104 k
 apr-util                  x86_64               1.5.2-6.el7                          base                                             92 k
 httpd-tools               x86_64               2.4.6-95.el7.centos                  base                                             93 k
 mailcap                   noarch               2.1.41-2.el7                         base                                             31 k

Transaction Summary
===========================================================================================================================================
```

需要更新的版本 及依赖关系

```bash
[root@agent1 httpd]# pwd 
/opt/httpd
[root@agent1 httpd]# ll 
total 2880
-rw-r--r--. 1 root root 2847412 Jan 25 22:54 httpd-2.4.6-97.el7.centos.4.x86_64.rpm
-rw-r--r--. 1 root root   96276 Jan 25 22:54 httpd-tools-2.4.6-97.el7.centos.4.x86_64.rpm
[root@agent1 httpd]# createrepo . 
Spawning worker 0 with 2 pkgs
Workers Finished
Saving Primary metadata
Saving file lists metadata
Saving other metadata
Generating sqlite DBs
Sqlite DBs complete
[root@agent1 httpd]# ll
total 2884
-rw-r--r--. 1 root root 2847412 Jan 25 22:54 httpd-2.4.6-97.el7.centos.4.x86_64.rpm
-rw-r--r--. 1 root root   96276 Jan 25 22:54 httpd-tools-2.4.6-97.el7.centos.4.x86_64.rpm
drwxr-xr-x. 2 root root    4096 Feb  8 13:08 repodata
[root@agent1 yum.repos.d]# pwd 
/etc/yum.repos.d
[root@agent1 yum.repos.d]# cat httpd.repo 
[httpd]
name=httpd - Plus
baseurl=file:///opt/httpd
gpgcheck=0
enabled=1
[root@agent1 yum.repos.d]# yum repolist httpd 
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * centos-sclo-rh: mirrors.huaweicloud.com
 * centos-sclo-sclo: mirrors.aliyun.com
 * epel: mirror.sjtu.edu.cn
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
repo id                                                         repo name                                                            status
httpd                                                           httpd - Plus                                                         2
repolist: 2
[root@agent1 yum.repos.d]# yum update httpd --enablerepo=httpd 
#我此处使用的是update， 使用install也是可以的。
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * centos-sclo-rh: mirrors.huaweicloud.com
 * centos-sclo-sclo: mirrors.aliyun.com
 * epel: mirror.sjtu.edu.cn
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
Resolving Dependencies
--> Running transaction check
---> Package httpd.x86_64 0:2.4.6-95.el7.centos will be updated
---> Package httpd.x86_64 0:2.4.6-97.el7.centos.4 will be an update
--> Processing Dependency: httpd-tools = 2.4.6-97.el7.centos.4 for package: httpd-2.4.6-97.el7.centos.4.x86_64
--> Running transaction check
---> Package httpd-tools.x86_64 0:2.4.6-95.el7.centos will be updated
---> Package httpd-tools.x86_64 0:2.4.6-97.el7.centos.4 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

===========================================================================================================================================
 Package                          Arch                        Version                                     Repository                  Size
===========================================================================================================================================
Updating:
 httpd                            x86_64                      2.4.6-97.el7.centos.4                       httpd                      2.7 M
Updating for dependencies:
 httpd-tools                      x86_64                      2.4.6-97.el7.centos.4                       httpd                       94 k

Transaction Summary
===========================================================================================================================================
Upgrade  1 Package (+1 Dependent package)

Total download size: 2.8 M
Is this ok [y/d/N]: y
Downloading packages:
-------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                      166 MB/s | 2.8 MB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : httpd-tools-2.4.6-97.el7.centos.4.x86_64                                                                                1/4 
  Updating   : httpd-2.4.6-97.el7.centos.4.x86_64                                                                                      2/4 
  Cleanup    : httpd-2.4.6-95.el7.centos.x86_64                                                                                        3/4 
  Cleanup    : httpd-tools-2.4.6-95.el7.centos.x86_64                                                                                  4/4 
  Verifying  : httpd-tools-2.4.6-97.el7.centos.4.x86_64                                                                                1/4 
  Verifying  : httpd-2.4.6-97.el7.centos.4.x86_64                                                                                      2/4 
  Verifying  : httpd-tools-2.4.6-95.el7.centos.x86_64                                                                                  3/4 
  Verifying  : httpd-2.4.6-95.el7.centos.x86_64                                                                                        4/4 

Updated:
  httpd.x86_64 0:2.4.6-97.el7.centos.4                                                                                                     

Dependency Updated:
  httpd-tools.x86_64 0:2.4.6-97.el7.centos.4                                                                                               

Complete!
```



---
✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>

 - [github httpd](https://github.com/apache/httpd)
 - [apache httpd](https://httpd.apache.org/)

