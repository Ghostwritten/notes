![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/754bc83cd6e17042ae0ede3560a6714d.png)

---

## 1. 安装（全部节点)
运行以下命令更新YUM源。

```bash
rpm -Uvh  http://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm
```

运行以下命令安装MySQL。

```bash
yum -y install mysql-community-server
```

运行以下命令查看MySQL版本号。

```bash
mysql -V
```

返回结果如下，表示MySQL安装成功。

```bash
mysql  Ver 14.14 Distrib 5.7.31, for Linux (x86_64) using  EditLine wrapper
```

步骤三：配置MySQL
运行以下命令启动MySQL服务。

```bash
systemctl start mysqld
```

运行以下命令设置MySQL服务开机自启动。

```bash
systemctl enable mysqld
```

运行以下命令查看/var/log/mysqld.log文件，获取并记录root用户的初始密码。

```bash
grep 'temporary password' /var/log/mysqld.log
```

执行​命令结果示例如下。

```bash
2020-04-08T08:12:07.893939Z 1 [Note] A temporary password is generated for root@localhost: xvlo1lZs7>uI
```

说明 下一步对MySQL进行安全性配置时，会使用该初始密码。
运行下列命令对MySQL进行安全性配置。

```bash
mysql_secure_installation
```

重置root用户的密码。

```bash
Enter password for user root: #输入上一步获取的root用户初始密码
The 'validate_password' plugin is installed on the server.
The subsequent steps will run with the existing configuration of the plugin.
Using existing password for root.
Estimated strength of the password: 100 
Change the password for root ? ((Press y|Y for Yes, any other key for No) : Y #是否更改root用户密码，输入Y
New password: #输入新密码，长度为8至30个字符，必须同时包含大小写英文字母、数字和特殊符号。特殊符号可以是()` ~!@#$%^&*-+=|{}[]:;‘<>,.?/
Re-enter new password: #再次输入新密码
Estimated strength of the password: 100 
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : Y #是否继续操作，输入Y
```

删除匿名用户账号。

```bash
By default, a MySQL installation has an anonymous user, allowing anyone to log into MySQL without having to have a user account created for them. This is intended only for testing, and to make the installation go a bit smoother. You should remove them before moving into a production environment.
Remove anonymous users? (Press y|Y for Yes, any other key for No) : Y  #是否删除匿名用户，输入Y
Success.
```
禁止root账号远程登录。

```bash
Disallow root login remotely? (Press y|Y for Yes, any other key for No) : Y #禁止root远程登录，输入Y
Success.
```
删除test库以及对test库的访问权限。

```bash
Remove test database and access to it? (Press y|Y for Yes, any other key for No) : Y #是否删除test库和对它的访问权限，输入Y
- Dropping test database...
Success.
```

重新加载授权表。

```bash
Reload privilege tables now? (Press y|Y for Yes, any other key for No) : Y #是否重新加载授权表，输入Y
Success.
All done!
```

安全性配置的更多详情，请参见MySQL官方文档。
步骤四：远程访问MySQL数据库
您可以使用数据库客户端或阿里云提供的数据管理服务DMS（Data Management Service）来远程访问MySQL数据库。本节以DMS为例，介绍远程访问MySQL数据库的操作步骤。

在ECS实例上，创建远程登录MySQL的账号。
运行以下命令后，输入root用户的密码登录MySQL。

```bash
 mysql -uroot -p
```

依次运行以下命令创建远程登录MySQL的账号。示例账号为dms、密码为123456。

```bash
mysql> grant all on *.* to 'dms'@'%' IDENTIFIED BY '123456'; #使用root替换dms，可设置为允许root账号远程登录。
mysql> flush privileges;
```

说明
建议您使用非root账号远程登录MySQL数据库。
实际创建账号时，需将123456更换为符合要求的密码： 长度为8至30个字符，必须同时包含大小写英文字母、数字和特殊符号。可以使用以下特殊符号：

```bash
()` ~!@#$%^&*-+=|{}[]:;‘<>,.?/
```
## 2. 配置
该配置为主mysql配置文件，从节点配置文件只需改动`server-id       = 2`

```bash
[client]
port            = 3306
socket          = /var/lib/mysql/mysql.sock
 
[mysqld]
gtid_mode=on
enforce_gtid_consistency=1
datadir=/var/lib/mysql
port            = 3306
socket          = /var/lib/mysql/mysql.sock
symbolic-links=0
skip-external-locking
key_buffer_size = 16M
max_allowed_packet = 1M
table_open_cache = 64
sort_buffer_size = 512K
net_buffer_length = 8K
read_buffer_size = 256K
read_rnd_buffer_size = 512K
myisam_sort_buffer_size = 8M
innodb_buffer_pool_size = 2G
log-bin=mysql-bin
binlog_format=mixed
server-id       = 1
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
log_slave_updates=1
slave-skip-errors = 1062
 
 
[mysqldump]
quick
max_allowed_packet = 16M
 
[mysql]
no-auto-rehash
 
[myisamchk]
key_buffer_size = 20M
sort_buffer_size = 20M
read_buffer = 2M
write_buffer = 2M
 
[mysqlhotcopy]
interactive-timeout
```
## 3. 主授同步权限

```bash
grant replication slave on *.* to 'db_sync'@'%' identified by 'Admin@2018';

[root@node1 ~]# mysql -uroot -pAdmin@2018 -e "show master status\G;"
mysql: [Warning] Using a password on the command line interface can be insecure.
*************************** 1. row ***************************
             File: mysql-bin.000030
         Position: 480
     Binlog_Do_DB: 
 Binlog_Ignore_DB: 
Executed_Gtid_Set: 0c479f94-48b1-11eb-af7c-000c29613c39:1-6
```
## 4. 从配置同步

```bash
$ mysql -uroot -pAdmin@2018 -e "stop slave;change master to master_host='192.168.211.60',master_user='db_sync',master_password='Admin@2018',master_auto_position=1,MASTER_HEARTBEAT_PERIOD=2,MASTER_CONNECT_RETRY=1, MASTER_RETRY_COUNT=86400;set global slave_net_timeout=8;start slave;"
$ mysql -uroot -pAdmin@2018 -e "show slave status\G;"
.....
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
 ...
```

相关阅读：
- [**Orchestrator (3) orchestrator-client命令详解**](https://ghostwritten.blog.csdn.net/article/details/111881197)
- [**Orchestrator (2) 配置参数详解**](https://ghostwritten.blog.csdn.net/article/details/111881286)
- [**Orchestrator  (1) 高可用管理详解**](https://ghostwritten.blog.csdn.net/article/details/106099648)
- [**centos本地部署mysql主从同步之gtid方法**](https://ghostwritten.blog.csdn.net/article/details/111826719)
