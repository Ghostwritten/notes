
使用正确的 SELinux 上下文和权限创建目录`/home/student/local/mysql`。

创建/home/student/local/mysql目录。

```bash
[student@workstation ~]$ mkdir -vp /home/student/local/mysql
mkdir: 创建的目录/home/student/local
mkdir: 创建的目录/home/student/local/mysql
```

/home/student/local/mysql为目录及其内容添加适当的 SELinux 上下文。

```bash
[student@workstation ~]$ sudo semanage fcontext -a  -t container_file_t '/home/student/local/mysql(/.*)?'
```

将 SELinux 策略应用于新创建的目录。

```bash
[student@workstation ~]$ sudo restorecon -R /home/student/local/mysql
```

验证该/home/student/local/mysql目录的 SELinux 上下文类型是否为container_file_t.

```bash
[student@workstation ~]$ ls -ldZ /home/student/local/mysql
drwxrwxr-x。2 名学生学生unconfined_u:object_r:container_file_t:s06 月 26 日 14:33 /home/student/local/mysql
```

将目录的所有者更改/home/student/local/mysql为mysql用户和mysql组：

```bash
[student@workstation ~]$ podman unshare chown 27:27 /home/student/local/mysql
```



> 容器中运行进程的用户必须能够将文件写入目录。 
> 权限应使用容器中的数字用户 ID (UID) 定义。Red Hat 提供的 MySQL 服务，UID 是 27。commandpodman
> unshare提供了一个会话，可以在与容器内运行的进程相同的用户命名空间内执行命令。

创建具有持久存储的 MySQL 容器实例。

使用您的 Red Hat 帐户登录到 Red Hat Container Catalog。如果您需要注册 Red Hat，请参阅附录 D 中的说明，创建红帽帐户.

```bash
[student@workstation ~]$ podman login registry.redhat.io
用户名：your-username
密码：your-password
登录成功！
```

拉取 MySQL 容器镜像。

```bash
[student@workstation ~]$ podman pull registry.redhat.io/rhel8/mysql-80:1
试图拉 registry.redhat.io/rhel8/mysql-80:1...
获取图像源签名
检查图像目标是否支持签名
复制 blob 028bdc977650 完成
...输出省略...
将清单写入图像目标
存储签名
4ae3a3f4f409a8912cab9fbf71d3564d011ed2e68f926d50f88f2a3a72c809c5
```

创建一个新的容器指定挂载点来存储 MySQL 数据库数据：

```bash
[student@workstation ~]$ podman run --name persist-db \
> -d -v /home/student/local/mysql:/var/lib/mysql/data \
> -e MYSQL_USER=user1 -e MYSQL_PASSWORD=mypa55 \
> -e MYSQL_DATABASE=items -e MYSQL_ROOT_PASSWORD=r00tpa55 \
> registry.redhat.io/rhel8/mysql-80:1
6e0ef134315b510042ca757faf869f2ba19df27790c601f95ec2fd9d3c44b95d
```

此命令将/home/student/local/mysql目录从主机挂载到/var/lib/mysql/data容器中的目录。默认情况下，MySQL 数据库将数据存储在该/var/lib/mysql/data目录中。

验证容器是否正确启动。

```bash
[student@workstation ~]$ podman ps --format="{{.ID}} {{.Names}} {{.Status}}"
6e0ef134315b   persist-db 启动 3 分钟前
```

验证该/home/student/local/mysql目录是否包含该items目录。

```bash
[student@workstation ~]$ ls -ld /home/student/local/mysql/items
drwxr-x---。2 100026 100026 8 月 6 日 07:31 /home/student/local/mysql/items
```

该目录存储与此容器创建的数据库items相关的数据。items如果该items目录不存在，则表示在创建容器期间未正确定义挂载点。

或者，您可以运行相同的命令来`podman unshare`检查容器中的数字用户 ID (UID)。

```bash
[student@workstation ~]$ podman unshare ls -ld /home/student/local/mysql/items
drwxr-x---。2 27 27 6 Apr 8 07:31 /home/student/local/mysql/items
```

