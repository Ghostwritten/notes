

---
##  1. nfs介绍
[红帽官方NFS讲解](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/managing_file_systems/mounting-nfs-shares_managing-file-systems)
### 1.1 简介

NFS 是Network File System的缩写，即网络文件系统。一种使用于分散式文件系统的协议，由Sun公司开发，于1984年向外公布。功能是通过网络让不同的机器、不同的操作系统能够彼此分享彼此的数据，让应用程序在客户端通过网络访问位于服务器磁盘中的数据，是在类Unix系统间实现磁盘文件共享的一种方法。
NFS 的基本原则是“允许不同的客户端及服务端通过一组RPC分享相同的文件系统”，它是独立于操作系统，允许不同硬件及操作系统的系统共同进行文件的分享。
NFS在文件传送或信息传送过程中依赖于RPC协议。RPC，远程过程调用 (Remote Procedure Call) 是能使客户端执行其他系统中程序的一种机制。NFS本身是没有提供信息传输的协议和功能的，但NFS却能让我们通过网络进行资料的分享，这是因为NFS使用了一些其它的传输协议。而这些传输协议用到这个RPC功能的。可以说NFS本身就是使用RPC的一个程序。或者说NFS也是一个RPC SERVER。所以只要用到 NFS的地方都要启动RPC服务，不论是NFS SERVER或者NFS CLIENT。这样SERVER和CLIENT才能通过RPC来实现PROGRAM PORT的对应。可以这么理解RPC和NFS的关系：NFS是一个文件系统，而RPC是负责负责信息的传输。

NFS协议从诞生到现在为止，已经有多个版本，如NFS V2（rfc1094）,NFS V3（rfc1813）（最新的版本是V4（rfc3010）。

### 1.2 各NFS协议版本的主要区别
#### V3相对V2的主要区别：

 - 文件尺寸：V2最大只支持32BIT的文件大小(4G),而NFS V3新增加了支持64BIT文件大小的技术。
 - 文件传输尺寸：V3没有限定传输尺寸，V2最多只能设定为8k，可以使用-rsize and -wsize 来进行设定。
 - 完整的信息返回：V3增加和完善了许多错误和成功信息的返回，对于服务器的设置和管理能带来很大好处。
 - 增加了对TCP传输协议的支持：V2只提供了对UDP协议的支持，在一些高要求的网络环境中有很大限制，V3增加了对TCP协议的支持
 - 异步写入特性：改进了SERVER的mount性能
 - 有更好的I/O WRITES 性能。
 - 更强网络运行效能，使得网络运作更为有效
 - 更强的灾难恢复功能。

异步写入特性（v3新增加）介绍：
NFS V3 能否使用异步写入，这是可选择的一种特性。NFS V3客户端发发送一个异步写入请求到服务器，在给客户端答复之前服务器并不是必须要将数据写入到存储器中（稳定的）。服务器能确定何时去写入数据或者将多个写入请求聚合到一起并加以处理，然后写入。客户端能保持一个数据的copy以防万一服务器不能完整的将数据写入。当客户端希望释放这个copy的时候，它会向服务器通过这个操作过程，以确保每个操作步骤的完整。异步写入能够使服务器去确定最好的同步数据的策略。使数据能尽可能的同步的提交何到达。与V2比较来看，这样的机制能更好的实现数据缓冲和更多的平行（平衡）。而NFS V2的SERVER在将数据写入存储器之前不能再相应任何的写入请求。

#### V4相对V3的改进：

 - 改进了INTERNET上的存取和执行效能
 - 在协议中增强了安全方面的特性
 - 增强的跨平台特性

### 1.3 NFS优点

 - 简单容易掌握
 - 方便快速部署简单维护容易
 - 可靠—从软件层面上看，数据可靠性高，经久耐用

### 1.4 NFS缺点

 - 局限性是存在单点故障，如果NFSserver宕机了所有客户端都不能访问共享目录，我们可以通过rsync来进行数据的同步。或者是通过负载均衡的高可用方案。
 - 在高并发的场合，NFS效率性能有限，一般几千万以下pv的网站不是瓶颈。
 - 服务器共享文件的客户端认证是基于IP和主机名的安全性一般（但用于内网则问题不大）
 - NFS数据是明文的，对数据完整性不做验证（一般是存放于内网，提供内网的服务器使用。所以安全性相对不是一个问题）
 - 多机器挂载服务器时，连接管理维护麻烦。尤其NFS服务端出问题后，所有客户端都挂掉状态（可使用autofs自动挂载解决。）

### 1.5 生产应用场景

   中小型网站（2000万pv以下）线上应用，都有用武之地。门户网站也会有其他方面的应用，。因为门户网站的并发量是超级的大。所以有更加会用专业的存储来做这件事情。

----
## 2. NFS原理
NFS服务器端其实是随机选择端口来进行数据传输。NFS服务器时通过远程过程调用`（remote procedure call 简称RPC）`协议/服务来实现的。也就是说RPC服务会统一管理NFS的端口，客户端和服务端通过RPC来先沟通NFS使用了哪些端口，之后再利用这些端口（小于1024）来进行数据的传输。
### 2.1 rpc与nfs关系
  `pc（portmap）`就是用来统一管理NFS端口的服务，并且统一对外的端口是111。NFS服务端需要先启动rpc，再启动NFS，这样NFS才能够到RPC去注册端口信息。客户端的RPC可以通过向服务端的RPC请求获取服务端的NFS端口信息。当获取到了NFS端口信息后，就会以实际端口进行数据的传输。（由于NFS端口为随机的。）
### 2.2 RPC和NFS如何通讯
 因为NFS有很多功能，不同的功能需要使用不同的端口。因此NFS无法固定端口。而RPC会记录NFS端口的信息，这样我们就能够通过RPC实现服务端和客户端的RPC来沟通端口信息。
 在启动NFS SERVER之前，首先要启动RPC服务（即portmap服务，下同）否则NFS SERVER就无法向RPC服务区注册，另外，如果RPC服务重新启动，原来已经注册好的NFS端口数据就会全部丢失。因此此时RPC服务管理的NFS程序也要重新启动以重新向RPC注册。特别注意：一般修改NFS配置文档后，是不需要重启NFS的，直接在命令执行/etc/init.d/nfs  reload或exportfs –rv即可使修改的/etc/exports生效。
###  2.3 客户端NFS和服务端NFS通讯过程

 1. 首先服务器端启动RPC服务，并开启111端口
 2. 启动NFS服务，并向RPC注册端口信息
 3. 客户端启动RPC（portmap服务），向服务端的RPC(portmap)服务请求服务端的NFS端口
 4. 服务端的RPC(portmap)服务反馈NFS端口信息给客户端。
 5. 客户端通过获取的NFS端口来建立和服务端的NFS连接并进行数据的传输。

----
## 3. 安装部署NFS
### 3.1 系统环境
```bash
系统环境：CentOS Linux 7 (Core)
NFS Server IP：192.168.211.15
NFS Client IP：192.168.211.16

$ systemctl disable firewalld
$ systemctl stop firewalld
$ sed -ri '#^SELINUX=#cSELINUX=Disabled' /etc/selinux/config
$ setenforce 0
```
### 3.2 安装NFS服务（服务端、客户端）

```bash
nfs-utils-* :包括基本的NFS命令与监控程序
portmap-* ：支持安全NFS RPC服务的连接
$ yum -y install nfs-utils portmap
```

### 3.3 NFS系统守护进程
NFS需要启动的Daemons：

 - pc.nfsd: 基本的NFS守护进程，主要负责Clinet登录权限检查。
 - rpm.mountd: RPC安装守护进程，负责NFS的文件系统，当Client端通过rpc.nfsd登录NFS Server后，对Client存取Server的文件前，还必须通过文件使用权限的验证，他会读取NFS的配置文件/etc/exports来对比Client权限。
 
 NFS server的服务进程

```bash
nfs-utils： 提供rpc.nfsd及rpc.mountd这两个NFS Daemons的套件
```

```bash
portmap： NFS其实可以看作一个RPC Server Program，主要功能是进行端口映射。当Client尝试连接并使用RPC服务提供的服务（NFS）时，portmap会将所管理的服务对应的端口提供给客户点，从而是Client可以通过该端口请求服务。
```
### 3.4 配置NFS服务
#### 3.4.1 NFS常用文件（记住）：

```bash
/etc/exports NFS服务的主要配置文件
/usr/sbin/exportfs NFS服务的管理命令
/usr/sbin/showmount 客户端的查看命令
/var/lib/nfs/etab 记录NFS分享出来的目录的完整权限设定值
/var/lib/nfs/xtab 记录曾经登录过的Clinent 信息
```
####  2.4.2 配置文件/etc/exports（服务端）
**注意**：NFS服务的配置文件是/etc/exports，这个文件是NFS的主要配置文件，不过系统并没有默认值，所以这个文件不一定存在，可能要自己手动创建，写入相应配置内容。

etc/exports文件内容格式：

```bash
[客户端1 选项（访问权限,用户映射,其他）] [客户端2 选项（访问权限,用户映射,其他）]
```

 输出目录：输出目录是指NFS系统中需要共享给客户端使用的目录

 - 客户端： 客户端是指网络中可以访问这个NFS Server的主机，客户端常用的指定方式如下：
   
    - 指定IP地址：192.168.211.15
    - 指定子网中的主机：192.168.211.0/24
    - 指定域名的主机：www.nfstest.com
    - 指定域中的所有主机：*.nfstest.com
    - 所有主机：*

选项：
选项用来设置输出目录的访问权限、用户映射等。
NFS主要有3类选项：

 - 访问权限选项：   设置输出目录只读：ro ；设置输出目录读写：rw
 - 用户映射选项：
    - `all_squash`： 将远程访问的所有普通用户及属组都映射为匿名用户或用户组(nfsnobody)；
    - `no_all_squash`： 与all_squash相反（default）；
    - `root_squash`： 将root用户及属组都映射问匿名用户或用户组（default）；
    - `no_root_squash`：客户机用root访问该共享文件夹时，不映射root用户；
    - `anonuid=xxx`： 将远程访问的所有用户都映射为匿名用户，并指定用户问本地用户（UID=xxx）；
    - `anongid=xxx`： 将远程访问的所有用户都映射为匿名用户组，并指定用户问本地用户组（GID=xxx）；
 - 其他选项：
    - `secure`： 限制客户端只能从小于1024的tcp端口连接NFS Server（default）；
    - `insecure`： 允许客户端从大于1024的tcp端口连接NFS Server；
    - `sync`： 将数据同步下乳内存缓冲区与磁盘中，效率低，但是可以保证数据的一致性；
    - `async`： 将数据先保存在内存缓冲区中，必要时才写入磁盘；
    - `wdelay`： 检查是否有相关的写操作，如果有则见这些写操作一起执行，可以提高效率（default）；
    - `no_wdelay`： 若有写操作立即执行，应与sync配合使用；
    - `subtree`： 若输出目录是一个子目录，则NFS Server将检查其父目录权限（default）；
    - `no_subtree`： 若输出目录是一个子目录，则NFS Server将不检查其父目录权限；

编辑配置文件

```bash
$ vim/etc/exports
/opt *(rw,no_root_squash)
/data 192.168.211.15(rw) 192.168.211.0/24(ro)
```
### 3.4.3 启动服务
```bash
$ systemctl start rpcbind
$ systemctl start nfs-server
```
**注意**：要停止NFS时，要先停止NFS服务再停止portmap，对于系统中有其他服务（如NIS）需要使用时，不要停止portmap。

### 3.4.4 查看服务状态

```bash
systemctl status rpcbind && systemctl status nfs-server
rpcinfo  -p
netstat  -lnt
```

### 3.4.5 查看是否应用配置

```bash
$ exportfs -rv
```
---
## 4. Client挂载目录（客户端）
### 4.1 启动rpcbind服务

```bash
$ systemctl enable rpcbind
$ systemctl start rpcbind
```
### 4.2 showmount远程服务器rpc提供的可挂载nfs信息

```bash
$ showmount -e 192.168.211.15
```

### 4.3 创建挂载点目录，命令行挂载共享文件

```bash
$ mkdir /data      #自定义目录，不需要与源目录保持一致
$ mount -t nfs 192.168.211.15:/data /data/  #执行挂载
$ df -h          #查看挂载
```

### 4.4 进行增删改操作，测试客户端是否拥有写的权限

```bash
$ echo "123" > /data/test
$ ll /data/
-rw-r--r-- 1 nfsnobody nfsnobody 4 9月   6 03:41 test
```

### 4.5 检查nfs服务端是否存在数据
```bash
$ ll /data/
-rw-r--r-- 1 nfsnobody nfsnobody 4 9月   6 03:41 test
```

### 4.6 如果希望NFS文件共享服务能一直有效则永久挂载

```bash
$ echo '192.168.211.15:/data       /data      nfs     defaults        0 0' >> /etc/fstab
$ tail -1 /etc/fstab
192.168.211.15:/data       /data                nfs     defaults        0 0
$ umount /data/  #验证fstab是否ok，前提要先卸载挂载
$ df -h          #检查卸载是否成功
$ mount -a  #执行挂载，默认执行/etc/fstab文件
$ df Th    #检查挂载是否成功
```
---
## 5. 问题
#### 5.1 卸载的时候如果提示`”umount.nfs: /data: device is busy”` 

```bash
$ umount -lf /data  #强制卸载
```
---
参考链接：
[https://www.cnblogs.com/zeq912/p/9606105.html](https://www.cnblogs.com/zeq912/p/9606105.html)
[https://czero000.github.io/2015/12/21/nfs-detail.html](https://czero000.github.io/2015/12/21/nfs-detail.html)
[https://blog.51cto.com/atong/1343950](https://blog.51cto.com/atong/1343950)




