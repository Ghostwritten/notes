---
## 1. samba介绍
[红帽官方samba讲解](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/managing_file_systems/assembly_mounting-an-smb-share-on-red-hat-enterprise-linux_managing-file-systems)
Samba 是在Linux和UNIX系统上实现SMB协议的一个免费软件，由服务器及客户端程序构成。

Samba最大的功能就是可以用于Linux与windows系统直接的文件共享和打印共享，Samba既可以用于windows与Linux之间的文件共享，也可以用于Linux与Linux之间的资源共享。

Samba由两个主要程序组成，它们是`smbd`和`nmbd`。这两个守护进程在服务器启动到停止期间持续运行，功能各异。Smbd和nmbd使用的全部配置信息全都保存在smb.conf文件中。Smb.conf向smbd和nmbd两个守护进程说明输出什么以便共享，共享输出给谁及如何进行输出。

 - SMB是Samba 的核心启动服务，主要负责建立 Linux Samba服务器与Samba客户机之间的对话，
   验证用户身份并提供对文件和打印系统的访问，只有SMB服务启动，才能实现文件的共享，监听139 TCP端口
 - NMB服务是负责解析用的，类似与DNS实现的功能，NMB可以把Linux系统共享的工作组名称与其IP对应起来，如果NMB服务没有启动，就只能通过IP来访问共享文件，监听137和138
   UDP端口。

　　例如，某台Samba服务器的IP地址为10.0.0.163，对应的工作组名称为davidsamba，那么在Windows的IE浏览器输入下面两条指令都可以访问共享文件。其实这就是Windows下查看Linux Samba服务器共享文件的方法。
　　\\10.0.0.163\共享目录名称
　　\\davidsamba\共享目录名称

　　Samba服务器可实现如下功能：WINS和DNS服务； 网络浏览服务； Linux和Windows域之间的认证和授权； UNICODE字符集和域名映射；满足CIFS协议的UNIX共享等。

Samba提供了基于CIFS的四个服务：

 - 文件和打印服务
 - 授权与被授权
 - 名称解析
 - 浏览服务

前两项服务由smbd提供，后两项服务则由nmbd提供。 简单地说，smbd进程的作用是处理到来的SMB软件包，为使用该软件包的资源与Linux进行协商，nmbd进程使主机(或工作站)能浏览Linux服务器。

---
## 2. 部署
### 2.1 系统环境

```bash
CentOS Linux 7 (Core)
smb server : 192.168.211.15
smb client : 192.168.211.16
windows
SELINUX=disabled
$ systemctl stop firewalld && systemctl enable firewalld
```

### 2.2 安装

```bash
$ yum -y install samba samba-client 
$ systemctl start smb
$ systemctl start nmb
$ ps -ef | grep -E 'smb|nmb'
$ netstat -tunlp | grep -E 'smbd|nmbd'  #smbd应用进程主要监听139和445端口， nmbd应用进程主要监听137与138端口。
```
### 2.3 Samba 配置服务
需求：系统分区时，单独划分一个`/storage`的分区，分区下有`logge`r和`shared`两个文件夹;

 - logger文件夹/storage/logger下对应的管理员账号为`logadmin`，用户账号为`loguser`;
 - shared文件夹/storage/shared下对应的管理员账号为`admin`，用户账户号为`shared`;

#### 2.2.1 创建共享目录

```bash
$ mkdir -p /storage/logger  /storage/shared
```

####  2.2.2 创建系统用户

```bash
$ useradd -s /sbin/nologin logadmin
$ useradd -s /sbin/nologin admin
$ useradd -g admin -s /sbin/nologin shared
```
#### 2.2.3 建Samba用户
密码设置`redhat`
```bash
# 创建logadmin用户
$ smbpasswd -a logadmin
New SMB password:
Retype new SMB password:
Added user logadmin.

# 创建loguser用户
$ smbpasswd -a loguser
New SMB password:
Retype new SMB password:
Added user loguser.

# 创建管理员用户
$ smbpasswd -a admin
New SMB password:
Retype new SMB password:
Added user admin.

# 创建shared用户
$ smbpasswd -a shared
New SMB password:
Retype new SMB password:
Added user shared.
```
#### 2.2.4 修改目录属性

```bash
# 修改所属主组
$ chown logadmin.logadmin logger
$ chown admin.admin shared

# 修改目录权限
$ chmod -R 777 logger
$ chmod -R 777 shared
```
#### 2.2.5 配置主配置文件
配置Samba服务

```bash
$ vi /etc/samba/smb.conf
```

```bash
# 全局参数
[global]
　　# 工作组名称 
    workgroup = SC.LOCAL
　　# 服务器介绍信息，参数%v 为显示 SMB 版本号
    server string = Samba Server Version %v
　　# 设置Samba Server的NetBIOS名称。
    netbios name = Linuxidc-Server
　　# 定义日志文件的存放位置与名称，参数%m 为来访的主机名
    log file = /var/log/samba/%m.log
　　# 定义日志文件的最大容量为 10240KB
    max log size = 10240
   # 安全验证的方式，总共有 4 种 
　　 security = user
　　# 定义用户后台的类型，共有 3 种
    passdb backend = tdbsam

[logger]
    comment = Logs Directories
    path = /storage/logger/
    public = no
    admin users = logadmin
    valid users = @logadmin
    browseable = yes
    writable = yes
    create mask = 0777
    directory mask = 0777
    force directory mode = 0777
    force create mode = 0777

# 共享参数
[shared]
    # 共享文件目录描述
    comment = Shared Directories
    # 共享文件目录
    path = /storage/shared/
    # 是否允许guest访问
    public = no
    # 指定管理用户
    admin users = admin
    # 可访问的用户组、用户
    valid users = @admin
    # 是否浏览权限
    browseable = yes
    # 是否可写权限
    writable = yes
    # 文件权限设置
    create mask = 0777
    directory mask = 0777
    force directory mode = 0777
    force create mode = 0777
```

### 2.3 重启服务
```bash
$ systemctl restart smb
$ systemctl restart nmb
```
### 2.4 Samba 测试访问
#### 2.4.1 windows
使用`Windows`客户机通过UNC路径访问Samba服务， 如： \\192.168.211.15
进入不同的目录会输入相应的用户与密码

```bash
logger目录：logadmin/redhat
shared目录：admin/redhat
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427200838795.png)
#### 2.4.2 linux
`smbclient`命令 属于samba套件，它提供一种命令行使用交互式方式访问samba服务器的共享资源。
 语法

```bash
smbclient(选项)(参数)
```

选项

```bash
-B<ip地址>：传送广播数据包时所用的IP地址；
-d<排错层级>：指定记录文件所记载事件的详细程度；
-E：将信息送到标准错误输出设备；
-h：显示帮助；
-i<范围>：设置NetBIOS名称范围；
-I<IP地址>：指定服务器的IP地址；
-l<记录文件>：指定记录文件的名称；
-L：显示服务器端所分享出来的所有资源；
-M<NetBIOS名称>：可利用WinPopup协议，将信息送给选项中所指定的主机；
-n<NetBIOS名称>：指定用户端所要使用的NetBIOS名称；
-N：不用询问密码；
-O<连接槽选项>：设置用户端TCP连接槽的选项；
-p<TCP连接端口>：指定服务器端TCP连接端口编号；
-R<名称解析顺序>：设置NetBIOS名称解析的顺序；
-s<目录>：指定smb.conf所在的目录；
-t<服务器字码>：设置用何种字符码来解析服务器端的文件名称；
-T<tar选项>：备份服务器端分享的全部文件，并打包成tar格式的文件；
-U<用户名称>：指定用户名称；
-w<工作群组>：指定工作群组名称。
```

 安装`samba-client`
```bash
$ yum -y install samba-client
```
显示共享所有内容
```bash
$ smbclient -L //192.168.211.15 
Enter SAMBA\root's password: 
Anonymous login successful

	Sharename       Type      Comment
	---------       ----      -------
	logger          Disk      Logs Directories
	shared          Disk      Shared Directories
	IPC$            IPC       IPC Service (Samba Server Version 4.9.1)
Reconnecting with SMB1 for workgroup listing.
Anonymous login successful

	Server               Comment
	---------            -------

	Workgroup            Master
	---------            -------
	SAMBA                MONITOR
	SC.LOCAL             LINUXIDC-SERVER

```
直接进入logadmin用户可以进入的logger目录
```bash
$ smbclient -U logadmin%redhat //192.168.211.15/logger  
```

隐形输入密码进入共享目录logger
```bash
$ smbclient -U logadmin //192.168.211.15/logger   
Enter SAMBA\logadmin's password: 
Try "help" to get a list of possible commands.
smb: \> ls
```
显示`logadmin`用户可以查看的共享文件
```bash
$ smbclient -L 192.168.211.15 -U logadmin%redhat  
```
一次性使用smbclient命令
```bash
$ smbclient -c "ls" //192.168.211.15/logger -U logadmin%redhat 
$ smbclient -c "mkdir test" //192.168.211.15/logger -U logadmin%redhat 
```
挂载共享文件，方便快速查看

```bash
$ mount -t cifs -o username=logadmin,password=redhat //192.168.211.15/logger /tmp/log   
$ ls /tmp/log/
test
```
---
##  3. 配置文件详解
### 3.1 全局参数

```bash
全局参数 [global]

config file = /usr/local/samba/lib/smb.conf.%m
#config file可以让你使用另一个配置文件来覆盖缺省的配置文件。如果文件 不存在，则该项无效。这个参数很有用，可以使得samba配置更灵活，可以让一台samba服务器模拟多台不同配置的服务器。比如，你想让PC1（主机名）这台电脑在访问Samba Server时使用它自己的配置文件，那么先在/etc/samba/host/下为PC1配置一个名为smb.conf.pc1的文件，然后在smb.conf中加入：config file=/etc/samba/host/smb.conf.%m。这样当PC1请求连接Samba Server时，smb.conf.%m就被替换成smb.conf.pc1。这样，对于PC1来说，它所使用的Samba服务就是由smb.conf.pc1定义的，而其他机器访问Samba Server则还是应用smb.conf。

workgroup = WORKGROUP
#设定 Samba Server 所要加入的工作组或者域。

server string = Samba Server Version %v
#设定 Samba Server 的注释，可以是任何字符串，也可以不填。宏%v表示显示Samba的版本号。

netbios name = smbserver
#设置Samba Server的NetBIOS名称。如果不填，则默认会使用该服务器的DNS名称的第一部分。netbios name和workgroup名字不要设置成一样了。

interfaces = lo eth0 192.168.12.2/24 192.168.13.2/24
#设置Samba Server监听哪些网卡，可以写网卡名，也可以写该网卡的IP地址。

hosts allow = 127. 192.168.1. 192.168.10.1
#表示允许连接到Samba Server的客户端，多个参数以空格隔开。可以用一个IP表示，也可以用一个网段表示。hosts deny 与hosts allow 刚好相反。

#例如：
# 表示容许来自172.17.2.*.*的主机连接，但排除172.17.2.50
hosts allow=172.17.2.EXCEPT172.17.2.50
# 表示容许来自172.17.2.0/255.255.0.0子网中的所有主机连接
hosts allow=172.17.2.0/255.255.0.0
# 表示容许来自M1和M2两台计算机连接
hosts allow=M1，M2
# 表示容许来自SC域的所有计算机连接
hosts allow=@SC

max connections = 0
#max connections用来指定连接Samba Server的最大连接数目。如果超出连接数目，则新的连接请求将被拒绝。0表示不限制。

deadtime = 0
#deadtime用来设置断掉一个没有打开任何文件的连接的时间。单位是分钟，0代表Samba Server不自动切断任何连接。

time server = yes/no
#time server用来设置让nmdb成为windows客户端的时间服务器。

log file = /var/log/samba/log.%m
#设置Samba Server日志文件的存储位置以及日志文件名称。在文件名后加个宏%m（主机名），表示对每台访问Samba Server的机器都单独记录一个日志文件。如果pc1、pc2访问过Samba Server，就会在/var/log/samba目录下留下log.pc1和log.pc2两个日志文件。

max log size = 50
#设置Samba Server日志文件的最大容量，单位为kB，0代表不限制。



security = user
#设置用户访问Samba Server的验证方式，一共有四种验证方式。

#share：用户访问Samba Server不需要提供用户名和口令, 安全性能较低。
#user：Samba Server共享目录只能被授权的用户访问,由Samba Server负责检查账号和密码的正确性。账号和密码要在本Samba Server中建立。
#server：依靠其他Windows NT/2000或Samba Server来验证用户的账号和密码,是一种代理验证。此种安全模式下,系统管理员可以把所有的Windows用户和口令集中到一个NT系统上,使用Windows NT进行Samba认证, 远程服务器可以自动认证全部用户和口令,如果认证失败,Samba将使用用户级安全模式作为替代的方式。
#domain：域安全级别,使用主域控制器(PDC)来完成认证。


passdb backend = tdbsam
#passdb backend就是用户后台的意思。
#目前有三种后台：smbpasswd、tdbsam和ldapsam。sam应该是security account manager（安全账户管理）的简写。

#smbpasswd：该方式是使用smb自己的工具smbpasswd来给系统用户（真实用户或者虚拟用户）设置一个Samba密码，客户端就用这个密码来访问Samba的资源。smbpasswd文件默认在/etc/samba目录下，不过有时候要手工建立该文件。
#tdbsam：该方式则是使用一个数据库文件来建立用户数据库。数据库文件叫passdb.tdb，默认在/etc/samba目录下。passdb.tdb用户数据库可以使用smbpasswd –a来建立Samba用户，不过要建立的Samba用户必须先是系统用户。我们也可以使用pdbedit命令来建立Samba账户。pdbedit命令的参数很多，我们列出几个主要的。

# pdbedit –a username：新建Samba账户。
# pdbedit –x username：删除Samba账户。
# pdbedit –L：列出Samba用户列表，读取passdb.tdb数据库文件。
# pdbedit –Lv：列出Samba用户列表的详细信息。
# pdbedit –c “[D]” –u username：暂停该Samba用户的账号。
# pdbedit –c “[]” –u username：恢复该Samba用户的账号。
 
#ldapsam：该方式则是基于LDAP的账户管理方式来验证用户。首先要建立LDAP服务，然后设置“passdb backend = ldapsam:ldap://LDAP Server”



encrypt passwords = yes/no
# 是否将认证密码加密。因为现在windows操作系统都是使用加密密码，所以一般要开启此项。不过配置文件默认已开启。

smb passwd file = /etc/samba/smbpasswd
#用来定义samba用户的密码文件。smbpasswd文件如果没有那就要手工新建。

username map = /etc/samba/smbusers
#用来定义用户名映射，比如可以将root换成administrator、admin等。不过要事先在smbusers文件中定义好。比如：root = administrator admin，这样就可以用administrator或admin这两个用户来代替root登陆Samba Server，更贴近windows用户的习惯。

guest account = nobody
# 用来设置guest用户名。

socket options = TCP_NODELAY SO_RCVBUF=8192 SO_SNDBUF=8192
#用来设置服务器和客户端之间会话的Socket选项，可以优化传输速度。

domain master = yes/no
#设置Samba服务器是否要成为网域主浏览器，网域主浏览器可以管理跨子网域的浏览服务。

local master = yes/no
#local master用来指定Samba Server是否试图成为本地网域主浏览器。如果设为no，则永远不会成为本地网域主浏览器。但是即使设置为yes，也不等于该Samba Server就能成为主浏览器，还需要参加选举。

preferred master = yes/no
#设置Samba Server一开机就强迫进行主浏览器选举，可以提高Samba Server成为本地网域主浏览器的机会。如果该参数指定为yes时，最好把domain master也指定为yes。使用该参数时要注意：如果在本Samba Server所在的子网有其他的机器（不论是windows NT还是其他Samba Server）也指定为首要主浏览器时，那么这些机器将会因为争夺主浏览器而在网络上大发广播，影响网络性能。如果同一个区域内有多台Samba Server，将上面三个参数设定在一台即可。

os level = 200
#设置samba服务器的os level。该参数决定Samba Server是否有机会成为本地网域的主浏览器。os level从0到255，winNT的os level是32，win95/98的os level是1。Windows 2000的os level是64。如果设置为0，则意味着Samba Server将失去浏览选择。如果想让Samba Server成为PDC，那么将它的os level值设大些。

domain logons = yes/no
#设置Samba Server是否要做为本地域控制器。主域控制器和备份域控制器都需要开启此项。

logon . = %u.bat
#当使用者用windows客户端登陆，那么Samba将提供一个登陆档。如果设置成%u.bat，那么就要为每个用户提供一个登陆档。如果人比较多，那就比较麻烦。可以设置成一个具体的文件名，比如start.bat，那么用户登陆后都会去执行start.bat，而不用为每个用户设定一个登陆档了。这个文件要放置在[netlogon]的path设置的目录路径下。

wins support = yes/no
#设置samba服务器是否提供wins服务。

wins server = wins服务器IP地址
#设置Samba Server是否使用别的wins服务器提供wins服务。

wins proxy = yes/no
#设置Samba Server是否开启wins代理服务。

dns proxy = yes/no
#设置Samba Server是否开启dns代理服务。

load printers = yes/no
#设置是否在启动Samba时就共享打印机。

printcap name = cups
#设置共享打印机的配置文件。

printing = cups
#设置Samba共享打印机的类型。现在支持的打印系统有：bsd, sysv, plp, lprng, aix, hpux, qnx

```
### 3.2 共享文件配置参数详解

```bash
共享参数 [共享名]：

comment = 任意字符串
#comment是对该共享的描述，可以是任意字符串。

path = 共享目录路径
#说明：path用来指定共享目录的路径。可以用%u、%m这样的宏来代替路径里的unix用户和客户机的Netbios名，用宏表示主要用于[homes]共享域。
#例如：如果我们不打算用home段做为客户的共享，而是在/home/share/下为每个Linux用户以他的用户名建个目录，作为他的共享目录，这样path就可以写成：path = /home/share/%u; 。
#用户在连接到这共享时具体的路径会被他的用户名代替，要注意这个用户名路径一定要存在，否则，客户机在访问时会找不到网络路径。
#同样，如果我们不是以用户来划分目录，而是以客户机来划分目录，为网络上每台可以访问samba的机器都各自建个以它的netbios名的路径，作为不同机器的共享资源，就可以这样写：path = /home/share/%m 。

browseable = yes/no
#browseable用来指定该共享是否可以浏览。

writable = yes/no
#writable用来指定该共享路径是否可写。

available = yes/no
#说明：available用来指定该共享资源是否可用。

admin users = 该共享的管理者
#说明：admin users用来指定该共享的管理员（对该共享具有完全控制权限）。在samba 3.0中，如果用户验证方式设置成“security=share”时，此项无效。
#例如：admin users =bobyuan，jane（多个用户中间用逗号隔开）。

valid users = 允许访问该共享的用户
#说明：valid users用来指定允许访问该共享资源的用户。
#例如：valid users = bobyuan，@bob，@tech（多个用户或者组中间用逗号隔开，如果要加入一个组就用“@+组名”表示。）

invalid users = 禁止访问该共享的用户
#说明：invalid users用来指定不允许访问该共享资源的用户。
#例如：invalid users = root，@bob（多个用户或者组中间用逗号隔开。）

write list = 允许写入该共享的用户
#说明：write list用来指定可以在该共享下写入文件的用户。
#例如：write list = bobyuan，@bob

public = yes/no
#说明：public用来指定该共享是否允许guest账户访问。

guest ok = yes/no
#说明：意义同“public”。


```

