
根据[官网](https://k3s.io/)快速安装方法只需一条命令：`curl -sfL https://get.k3s.io | sh -`
```bash
[root@localhost ~]# curl -sfL https://get.k3s.io | sh -
[INFO]  Finding release for channel stable
[INFO]  Using v1.23.6+k3s1 as release
[INFO]  Downloading hash https://github.com/k3s-io/k3s/releases/download/v1.23.6+k3s1/sha256sum-amd64.txt[INFO]  Downloading binary https://github.com/k3s-io/k3s/releases/download/v1.23.6+k3s1/k3s
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
base                                                                              | 3.6 kB  00:00:00     
epel                                                                              | 4.7 kB  00:00:00     
http://mirrors.aliyuncs.com/centos/7/extras/x86_64/repodata/repomd.xml: [Errno 14] curl#7 - "Failed connect to mirrors.aliyuncs.com:80; 拒绝连接"
正在尝试其它镜像。
extras                                                                                                                                                                                                      | 2.9 kB  00:00:00     
updates                                                                                                                                                                                                     | 2.9 kB  00:00:00     
(1/7): epel/x86_64/group_gz                                                                                                                                                                                 |  96 kB  00:00:00     
(2/7): base/7/x86_64/group_gz                                                                                                                                                                               | 153 kB  00:00:01     
(3/7): extras/7/x86_64/primary_db                                                                                                                                                                           | 247 kB  00:00:01     
(4/7): epel/x86_64/updateinfo                                                                                                                                                                               | 1.0 MB  00:00:03     
(5/7): base/7/x86_64/primary_db                                                                                                                                                                             | 6.1 MB  00:00:20     
(6/7): epel/x86_64/primary_db                                                                                                                                                                               | 7.0 MB  00:00:22     
(7/7): updates/7/x86_64/primary_db                                                                                                                                                                          |  16 MB  00:00:45     
正在解决依赖关系
--> 正在检查事务
---> 软件包 yum-utils.noarch.0.1.1.31-50.el7 将被 升级
---> 软件包 yum-utils.noarch.0.1.1.31-54.el7_8 将被 更新
--> 解决依赖关系完成

依赖关系解决

=================================================================================================================================================================================================================================== Package                                                架构                                                版本                                                           源                                                 大小
===================================================================================================================================================================================================================================正在更新:
 yum-utils                                              noarch                                              1.1.31-54.el7_8                                                base                                              122 k

事务概要
===================================================================================================================================================================================================================================升级  1 软件包

总下载量：122 k
Downloading packages:
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
yum-utils-1.1.31-54.el7_8.noarch.rpm                                                                                                                                                                        | 122 kB  00:00:01     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在更新    : yum-utils-1.1.31-54.el7_8.noarch                                                                                                                                                                               1/2 
  清理        : yum-utils-1.1.31-50.el7.noarch                                                                                                                                                                                 2/2 
  验证中      : yum-utils-1.1.31-54.el7_8.noarch                                                                                                                                                                               1/2 
  验证中      : yum-utils-1.1.31-50.el7.noarch                                                                                                                                                                                 2/2 

更新完毕:
  yum-utils.noarch 0:1.1.31-54.el7_8                                                                                                                                                                                               

完毕！
已加载插件：fastestmirror
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
rancher-k3s-common-stable                                                                                                                                                                                   | 2.9 kB  00:00:00     
rancher-k3s-common-stable/primary_db                                                                                                                                                                        | 3.4 kB  00:00:01     
正在解决依赖关系
--> 正在检查事务
---> 软件包 k3s-selinux.noarch.0.1.1-1.el7 将被 安装
--> 正在处理依赖关系 container-selinux < 2:2.164.2，它被软件包 k3s-selinux-1.1-1.el7.noarch 需要
--> 正在处理依赖关系 selinux-policy-base >= 3.13.1-252，它被软件包 k3s-selinux-1.1-1.el7.noarch 需要
--> 正在处理依赖关系 container-selinux >= 2:2.107-3，它被软件包 k3s-selinux-1.1-1.el7.noarch 需要
--> 正在检查事务
---> 软件包 container-selinux.noarch.2.2.119.2-1.911c772.el7_8 将被 安装
--> 正在处理依赖关系 policycoreutils-python，它被软件包 2:container-selinux-2.119.2-1.911c772.el7_8.noarch 需要
---> 软件包 selinux-policy-targeted.noarch.0.3.13.1-229.el7_6.6 将被 升级
---> 软件包 selinux-policy-targeted.noarch.0.3.13.1-268.el7_9.2 将被 更新
--> 正在处理依赖关系 selinux-policy = 3.13.1-268.el7_9.2，它被软件包 selinux-policy-targeted-3.13.1-268.el7_9.2.noarch 需要
--> 正在处理依赖关系 selinux-policy = 3.13.1-268.el7_9.2，它被软件包 selinux-policy-targeted-3.13.1-268.el7_9.2.noarch 需要
--> 正在检查事务
---> 软件包 policycoreutils-python.x86_64.0.2.5-34.el7 将被 安装
--> 正在处理依赖关系 policycoreutils = 2.5-34.el7，它被软件包 policycoreutils-python-2.5-34.el7.x86_64 需要
--> 正在处理依赖关系 setools-libs >= 3.3.8-4，它被软件包 policycoreutils-python-2.5-34.el7.x86_64 需要
--> 正在处理依赖关系 libsemanage-python >= 2.5-14，它被软件包 policycoreutils-python-2.5-34.el7.x86_64 需要
--> 正在处理依赖关系 audit-libs-python >= 2.1.3-4，它被软件包 policycoreutils-python-2.5-34.el7.x86_64 需要
--> 正在处理依赖关系 python-IPy，它被软件包 policycoreutils-python-2.5-34.el7.x86_64 需要
--> 正在处理依赖关系 libqpol.so.1(VERS_1.4)(64bit)，它被软件包 policycoreutils-python-2.5-34.el7.x86_64 需要
--> 正在处理依赖关系 libqpol.so.1(VERS_1.2)(64bit)，它被软件包 policycoreutils-python-2.5-34.el7.x86_64 需要
--> 正在处理依赖关系 libapol.so.4(VERS_4.0)(64bit)，它被软件包 policycoreutils-python-2.5-34.el7.x86_64 需要
--> 正在处理依赖关系 checkpolicy，它被软件包 policycoreutils-python-2.5-34.el7.x86_64 需要
--> 正在处理依赖关系 libqpol.so.1()(64bit)，它被软件包 policycoreutils-python-2.5-34.el7.x86_64 需要
--> 正在处理依赖关系 libapol.so.4()(64bit)，它被软件包 policycoreutils-python-2.5-34.el7.x86_64 需要
---> 软件包 selinux-policy.noarch.0.3.13.1-229.el7_6.6 将被 升级
---> 软件包 selinux-policy.noarch.0.3.13.1-268.el7_9.2 将被 更新
--> 正在检查事务
---> 软件包 audit-libs-python.x86_64.0.2.8.5-4.el7 将被 安装
--> 正在处理依赖关系 audit-libs(x86-64) = 2.8.5-4.el7，它被软件包 audit-libs-python-2.8.5-4.el7.x86_64 需要
---> 软件包 checkpolicy.x86_64.0.2.5-8.el7 将被 安装
---> 软件包 libsemanage-python.x86_64.0.2.5-14.el7 将被 安装
---> 软件包 policycoreutils.x86_64.0.2.5-29.el7 将被 升级
---> 软件包 policycoreutils.x86_64.0.2.5-34.el7 将被 更新
---> 软件包 python-IPy.noarch.0.0.75-6.el7 将被 安装
---> 软件包 setools-libs.x86_64.0.3.3.8-4.el7 将被 安装
--> 正在检查事务
---> 软件包 audit-libs.x86_64.0.2.8.4-4.el7 将被 升级
--> 正在处理依赖关系 audit-libs(x86-64) = 2.8.4-4.el7，它被软件包 audit-2.8.4-4.el7.x86_64 需要
---> 软件包 audit-libs.x86_64.0.2.8.5-4.el7 将被 更新
--> 正在检查事务
---> 软件包 audit.x86_64.0.2.8.4-4.el7 将被 升级
---> 软件包 audit.x86_64.0.2.8.5-4.el7 将被 更新
--> 解决依赖关系完成

依赖关系解决

=================================================================================================================================================================================================================================== Package                                                   架构                                     版本                                                         源                                                           大小
===================================================================================================================================================================================================================================正在安装:
 k3s-selinux                                               noarch                                   1.1-1.el7                                                    rancher-k3s-common-stable                                    16 k
为依赖而安装:
 audit-libs-python                                         x86_64                                   2.8.5-4.el7                                                  base                                                         76 k
 checkpolicy                                               x86_64                                   2.5-8.el7                                                    base                                                        295 k
 container-selinux                                         noarch                                   2:2.119.2-1.911c772.el7_8                                    extras                                                       40 k
 libsemanage-python                                        x86_64                                   2.5-14.el7                                                   base                                                        113 k
 policycoreutils-python                                    x86_64                                   2.5-34.el7                                                   base                                                        457 k
 python-IPy                                                noarch                                   0.75-6.el7                                                   base                                                         32 k
 setools-libs                                              x86_64                                   3.3.8-4.el7                                                  base                                                        620 k
为依赖而更新:
 audit                                                     x86_64                                   2.8.5-4.el7                                                  base                                                        256 k
 audit-libs                                                x86_64                                   2.8.5-4.el7                                                  base                                                        102 k
 policycoreutils                                           x86_64                                   2.5-34.el7                                                   base                                                        917 k
 selinux-policy                                            noarch                                   3.13.1-268.el7_9.2                                           updates                                                     498 k
 selinux-policy-targeted                                   noarch                                   3.13.1-268.el7_9.2                                           updates                                                     7.0 M

事务概要
===================================================================================================================================================================================================================================安装  1 软件包 (+7 依赖软件包)
升级           ( 5 依赖软件包)

总下载量：10 M
Downloading packages:
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
(1/13): audit-libs-2.8.5-4.el7.x86_64.rpm                                                                                                                                                                   | 102 kB  00:00:00     
(2/13): audit-2.8.5-4.el7.x86_64.rpm                                                                                                                                                                        | 256 kB  00:00:01     
(3/13): audit-libs-python-2.8.5-4.el7.x86_64.rpm                                                                                                                                                            |  76 kB  00:00:00     
(4/13): libsemanage-python-2.5-14.el7.x86_64.rpm                                                                                                                                                            | 113 kB  00:00:00     
(5/13): checkpolicy-2.5-8.el7.x86_64.rpm                                                                                                                                                                    | 295 kB  00:00:00     
(6/13): container-selinux-2.119.2-1.911c772.el7_8.noarch.rpm                                                                                                                                                |  40 kB  00:00:00     
warning: /var/cache/yum/x86_64/7/rancher-k3s-common-stable/packages/k3s-selinux-1.1-1.el7.noarch.rpm: Header V4 RSA/SHA1 Signature, key ID e257814a: NOKEY                                       ] 260 kB/s | 1.0 MB  00:00:36 ETA 
k3s-selinux-1.1-1.el7.noarch.rpm 的公钥尚未安装
(7/13): k3s-selinux-1.1-1.el7.noarch.rpm                                                                                                                                                                    |  16 kB  00:00:01     
(8/13): policycoreutils-python-2.5-34.el7.x86_64.rpm                                                                                                                                                        | 457 kB  00:00:00     
(9/13): python-IPy-0.75-6.el7.noarch.rpm                                                                                                                                                                    |  32 kB  00:00:00     
(10/13): policycoreutils-2.5-34.el7.x86_64.rpm                                                                                                                                                              | 917 kB  00:00:01     
(11/13): selinux-policy-3.13.1-268.el7_9.2.noarch.rpm                                                                                                                                                       | 498 kB  00:00:01     
(12/13): setools-libs-3.3.8-4.el7.x86_64.rpm                                                                                                                                                                | 620 kB  00:00:01     
(13/13): selinux-policy-targeted-3.13.1-268.el7_9.2.noarch.rpm                                                                                                                                              | 7.0 MB  00:00:13     
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------总计                                                                                                                                                                                               656 kB/s |  10 MB  00:00:16     
从 https://rpm.rancher.io/public.key 检索密钥
导入 GPG key 0xE257814A:
 用户ID     : "Rancher (CI) <ci@rancher.com>"
 指纹       : c8cf f216 4551 26e9 b9c9 18be 925e a29a e257 814a
 来自       : https://rpm.rancher.io/public.key
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在更新    : audit-libs-2.8.5-4.el7.x86_64                                                                                                                                                                                 1/18 
  正在更新    : policycoreutils-2.5-34.el7.x86_64                                                                                                                                                                             2/18 
  正在更新    : selinux-policy-3.13.1-268.el7_9.2.noarch                                                                                                                                                                      3/18 
  正在更新    : selinux-policy-targeted-3.13.1-268.el7_9.2.noarch                                                                                                                                                             4/18 
  正在安装    : audit-libs-python-2.8.5-4.el7.x86_64                                                                                                                                                                          5/18 
  正在安装    : setools-libs-3.3.8-4.el7.x86_64                                                                                                                                                                               6/18 
  正在安装    : checkpolicy-2.5-8.el7.x86_64                                                                                                                                                                                  7/18 
  正在安装    : python-IPy-0.75-6.el7.noarch                                                                                                                                                                                  8/18 
  正在安装    : libsemanage-python-2.5-14.el7.x86_64                                                                                                                                                                          9/18 
  正在安装    : policycoreutils-python-2.5-34.el7.x86_64                                                                                                                                                                     10/18 
  正在安装    : 2:container-selinux-2.119.2-1.911c772.el7_8.noarch                                                                                                                                                           11/18 
  正在安装    : k3s-selinux-1.1-1.el7.noarch                                                                                                                                                                                 12/18 
  正在更新    : audit-2.8.5-4.el7.x86_64                                                                                                                                                                                     13/18 
  清理        : selinux-policy-targeted-3.13.1-229.el7_6.6.noarch                                                                                                                                                            14/18 
  清理        : selinux-policy-3.13.1-229.el7_6.6.noarch                                                                                                                                                                     15/18 
  清理        : policycoreutils-2.5-29.el7.x86_64                                                                                                                                                                            16/18 
  清理        : audit-2.8.4-4.el7.x86_64                                                                                                                                                                                     17/18 
  清理        : audit-libs-2.8.4-4.el7.x86_64                                                                                                                                                                                18/18 
  验证中      : audit-libs-2.8.5-4.el7.x86_64                                                                                                                                                                                 1/18 
  验证中      : k3s-selinux-1.1-1.el7.noarch                                                                                                                                                                                  2/18 
  验证中      : audit-2.8.5-4.el7.x86_64                                                                                                                                                                                      3/18 
  验证中      : policycoreutils-2.5-34.el7.x86_64                                                                                                                                                                             4/18 
  验证中      : libsemanage-python-2.5-14.el7.x86_64                                                                                                                                                                          5/18 
  验证中      : 2:container-selinux-2.119.2-1.911c772.el7_8.noarch                                                                                                                                                            6/18 
  验证中      : python-IPy-0.75-6.el7.noarch                                                                                                                                                                                  7/18 
  验证中      : checkpolicy-2.5-8.el7.x86_64                                                                                                                                                                                  8/18 
  验证中      : policycoreutils-python-2.5-34.el7.x86_64                                                                                                                                                                      9/18 
  验证中      : selinux-policy-3.13.1-268.el7_9.2.noarch                                                                                                                                                                     10/18 
  验证中      : audit-libs-python-2.8.5-4.el7.x86_64                                                                                                                                                                         11/18 
  验证中      : selinux-policy-targeted-3.13.1-268.el7_9.2.noarch                                                                                                                                                            12/18 
  验证中      : setools-libs-3.3.8-4.el7.x86_64                                                                                                                                                                              13/18 
  验证中      : selinux-policy-targeted-3.13.1-229.el7_6.6.noarch                                                                                                                                                            14/18 
  验证中      : selinux-policy-3.13.1-229.el7_6.6.noarch                                                                                                                                                                     15/18 
  验证中      : audit-libs-2.8.4-4.el7.x86_64                                                                                                                                                                                16/18 
  验证中      : policycoreutils-2.5-29.el7.x86_64                                                                                                                                                                            17/18 
  验证中      : audit-2.8.4-4.el7.x86_64                                                                                                                                                                                     18/18 

已安装:
  k3s-selinux.noarch 0:1.1-1.el7                                                                                                                                                                                                   

作为依赖被安装:
  audit-libs-python.x86_64 0:2.8.5-4.el7     checkpolicy.x86_64 0:2.5-8.el7        container-selinux.noarch 2:2.119.2-1.911c772.el7_8     libsemanage-python.x86_64 0:2.5-14.el7     policycoreutils-python.x86_64 0:2.5-34.el7    
  python-IPy.noarch 0:0.75-6.el7             setools-libs.x86_64 0:3.3.8-4.el7    

作为依赖被升级:
  audit.x86_64 0:2.8.5-4.el7        audit-libs.x86_64 0:2.8.5-4.el7        policycoreutils.x86_64 0:2.5-34.el7        selinux-policy.noarch 0:3.13.1-268.el7_9.2        selinux-policy-targeted.noarch 0:3.13.1-268.el7_9.2       

完毕！
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Creating /usr/local/bin/ctr symlink to k3s
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
Created symlink from /etc/systemd/system/multi-user.target.wants/k3s.service to /etc/systemd/system/k3s.service.
[INFO]  systemd: Starting k3s

[root@localhost ~]# k3s kubectl get node
NAME         STATUS   ROLES                  AGE   VERSION
k3s-master   Ready    control-plane,master   34s   v1.23.6+k3s1
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1e5e31da344940a39d79a002f7866515.gif#pic_center)

