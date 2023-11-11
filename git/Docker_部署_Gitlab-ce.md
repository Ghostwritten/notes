#  Docker 部署 Gitlab-ce



---

## 1. 简介
[Gitlab](https://about.gitlab.com/)是一个开源的Git代码仓库系统，可以实现自托管的Github项目，即用于构建私有的代码托管平台和项目管理系统。系统基于Ruby on Rails开发，速度快、安全稳定。它拥有与Github类似的功能，能够浏览源代码，管理缺陷和注释。可以管理团队对仓库的访问，它非常易于浏览提交过的版本并提供一个文件历史库。团队成员可以利用内置的简单聊天程序(Wall)进行交流。它还提供一个代码片段收集功能可以轻松实现代码复用，便于日后有需要的时候进行查找。
![在这里插入图片描述](https://img-blog.csdnimg.cn/f25bc959c1d144a199a5f55ce6263385.png?)





## 2. 安装 docker
[install docker in ubuntu](https://docs.docker.com/engine/install/ubuntu/) 


卸载旧版本

```bash
 sudo apt-get remove docker docker-engine docker.io containerd runc
 rm -rf /var/lib/docker/
```
配置docker源

```bash
 sudo apt-get update
 sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
#下载gpg证书
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

#其他版本
 echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
安装 docker

```bash
#查看版本
 apt-cache madison docker-ce
 sudo apt-get update
 sudo apt-get install docker-ce docker-ce-cli containerd.io
 或者指定特定版本
 sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io
```
## 3. docker 安装 GitLab-ce
GitLab的安装可以直接run，或者通过docker-compose文件指定安装流程，这里使用前者进行快速简单安装，后者后续更新。

拉取GitLab-ce镜像，查看镜像信息

```bash
$ docker pull gitlab/gitlab-ce
$ docker image ls
```

#配置存储位置

```bash
$ mkdir /opt/gitlab
$ export GITLAB_HOME=/opt/gitlab
$ echo $GITLAB_HOME
/opt/gitlab
```
运行gitlab

```bash
$ docker run -d --hostname gitlab.example.com -p 443:443 -p 80:80 -p 22:22 --name gitlab --restart always -v $GITLAB_HOME/config:/etc/gitlab:Z -v $GITLAB_HOME/logs:/var/log/gitlab:Z -v $GITLAB_HOME/data:/var/opt/gitlab:Z --shm-size 256m gitlab/gitlab-ce:latest                 
```
`:Z`能够确保有足够的权限的权限创建文件
正常要等1~2分钟。

> 假如报错 ```bash /opt/gitlab/embedded/bin/runsvdir-start: line 24: ulimit:
> pending signals: cannot modify limit: Operation not permitted 
> /opt/gitlab/embedded/bin/runsvdir-start: line 37:
> /proc/sys/fs/file-max: Read-only file system Configuring GitLab
> package... Configuring GitLab...  ```
> 解决方法：
> ` chmod 2770 /opt/gitlab/data/git-data/repositories` 
> `docker restart gitlab` 
> 
> 查看容器运行情况，出现gitlab运行信息表明启动成功

查看容器状态
```bash
$ docker ps
```

查看进程状态
```bash
$ docker exec gitlab-ce gitlab-ctl status
run: alertmanager: (pid 783) 757s; run: log: (pid 694) 780s
run: gitaly: (pid 293) 841s; run: log: (pid 312) 840s
run: gitlab-exporter: (pid 763) 759s; run: log: (pid 622) 796s
run: gitlab-kas: (pid 420) 829s; run: log: (pid 432) 828s
run: gitlab-workhorse: (pid 753) 759s; run: log: (pid 495) 810s
run: logrotate: (pid 263) 854s; run: log: (pid 273) 851s
run: nginx: (pid 520) 805s; run: log: (pid 535) 804s
run: postgres-exporter: (pid 792) 757s; run: log: (pid 720) 772s
run: postgresql: (pid 321) 835s; run: log: (pid 417) 833s
run: prometheus: (pid 772) 758s; run: log: (pid 655) 784s
run: puma: (pid 436) 823s; run: log: (pid 443) 822s
run: redis: (pid 276) 847s; run: log: (pid 285) 846s
run: redis-exporter: (pid 765) 758s; run: log: (pid 634) 792s
run: sidekiq: (pid 451) 817s; run: log: (pid 467) 816s
run: sshd: (pid 32) 864s; run: log: (pid 31) 864s
```


浏览器进入`http://192.168.211.70:8080`，使用root账户登录并设置密码即可进入管理员界面
![在这里插入图片描述](https://img-blog.csdnimg.cn/6f147f63b1574e67a4b3b472ffbd1329.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)

### 3.1 定义环境变量
环境变量`GITLAB_OMNIBUS_CONFIG`添加到Docker run来预先配置GitLab Docker镜像。这个变量包含任何`gitlab.rb`设置,，`GITLAB_OMNIBUS_CONFIG`中包含的设置不写入`gitlab.rb`配置文件。
下面是一个设置外部URL并在启动容器时启用LFS的例子:

```bash
docker run --detach \
  --hostname gitlab.example.com \
  --env GITLAB_OMNIBUS_CONFIG="external_url 'http://my.domain.com/'; gitlab_rails['lfs_enabled'] = true;" \
  --publish 443:443 --publish 80:80 --publish 22:22 \
  --name gitlab \
  --restart always \
  --volume $GITLAB_HOME/config:/etc/gitlab \
  --volume $GITLAB_HOME/logs:/var/log/gitlab \
  --volume $GITLAB_HOME/data:/var/opt/gitlab \
  --shm-size 256m \
  gitlab/gitlab-ee:latest
```

### 3.2 映射不同端口
如果你想使用一个不同于80 (HTTP)或443 (HTTPS)的主机端口，你需要添加一个单独的——publish指令到docker运行命令。

```bash
$ docker run --detach \
  --hostname gitlab.example.com \
  --publish 8929:8929 --publish 2289:22 \
  --name gitlab \
  --restart always \
  --volume $GITLAB_HOME/config:/etc/gitlab \
  --volume $GITLAB_HOME/logs:/var/log/gitlab \
  --volume $GITLAB_HOME/data:/var/opt/gitlab \
  --shm-size 256m \
  gitlab/gitlab-ee:latest

$ docker exec -it gitlab /bin/bash

$ vi /etc/gitlab/gitlab.rb
# For HTTP
external_url "http://gitlab.example.com:8929"

or

# For HTTPS (notice the https)
external_url "https://gitlab.example.com:8929"

gitlab_rails['gitlab_shell_ssh_port'] = 2289

#重载配置
$ gitlab-ctl reconfigure
```

> 端映射格式为“`hostPort:containerPort`”。更多信息请[参阅Docker的文档](https://docs.docker.com/engine/reference/run/#/expose-incoming-ports)

此`external_url`中指定的端口必须与Docker发布给主机的端口相匹配。此外，如果NGINX监听端口没有在NGINX `['listen_port']`中显式设置，它将从`external_url`中拉出。要了解更多信息，[请参阅NGINX文档](https://docs.gitlab.com/omnibus/settings/nginx.html)。


### 3.3  对接外部 postgres

创建数据存储目录
```bash
mkdir -p /opt/gitlab/postgresql/data
```
创建 postgres
```bash
docker run --name=postgresql -tid -p 5432:5432 --restart=always -e POSTGRES_DB=gitlab -e POSTGRES_USER=gitlab -e  POSTGRES_PASSWORD='Bsgchina@2023' -e POSTGRES_ADMIN_PASSWORD='Bsgchina@2023' -v /opt/gitlab/postgresql/data:/var/lib/postgresql/data --name postgres postgres:latest
```
测试连接

```bash
yum -y install postgresql
psql -h 192.168.21.21 -p 5432 -W -U gitlab gitlab 
```


创建 gitlab-ce
```bash
docker run -tid --hostname gitlab.example.com -p 443:443 -p 80:80 -p 2200:22 --name gitlab-ce --restart always -v $GITLAB_HOME/config:/etc/gitlab:Z -v $GITLAB_HOME/logs:/var/log/gitlab:Z -v $GITLAB_HOME/data:/var/opt/gitlab:Z  gitlab/gitlab-ce:latest
```


定义配置文件 `gitlab.rb` 

```bash
$ vim $GITLAB_HOME/config/gitlab.rb
.....
external_url 'http://gitlab.example.com'
.....
### GitLab database settings
###! Docs: https://docs.gitlab.com/omnibus/settings/database.html
###! **Only needed if you use an external database.**
postgresql['enable'] = false
gitlab_rails['db_adapter'] = "postgresql"
gitlab_rails['db_encoding'] = "utf-8"
# gitlab_rails['db_collation'] = nil
gitlab_rails['db_database'] = "gitlab"
gitlab_rails['db_username'] = "gitlab"
gitlab_rails['db_password'] = "Bsgchina@2023"
gitlab_rails['db_host'] = "192.168.21.21"
gitlab_rails['db_port'] = 5432
gitlab_rails['db_pool'] = 100
....
```

执⾏以下命令初始化 GitLab，此命令执⾏会消耗⼀段时间，请耐心等待，当提示：gitlab Reconfigured! 时，说明已经初始化完成。

```bash
docker exec gitlab-ce gitlab-ctl reconfigure
```
执⾏以下命令启动 GitLab。

```bash
docker exec gitlab-ce gitlab-ctl start
```


##  4. Docker-compose 安装 gitlab
使用[Docker Compose](https://docs.docker.com/compose/)，你可以轻松地配置、安装和升级基于Docker的GitLab安装:

 - 安装[docker-compose](https://docs.docker.com/compose/install/)
 - 编排docker-compose.yml

```bash
version: '3.6'
services:
  web:
    image: 'gitlab/gitlab-ee:latest'
    restart: always
    hostname: 'gitlab.example.com'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'https://gitlab.example.com'
        # Add any other gitlab.rb configuration here, each on its own line
    ports:
      - '80:80'
      - '443:443'
      - '22:22'
    volumes:
      - '$GITLAB_HOME/config:/etc/gitlab'
      - '$GITLAB_HOME/logs:/var/log/gitlab'
      - '$GITLAB_HOME/data:/var/opt/gitlab'
    shm_size: '256m'
```

```bash
docker-compose up -d
```


## 5. 界面

访问：`http://192.168.21.21/`
![在这里插入图片描述](https://img-blog.csdnimg.cn/f06b023506e04001ba5fef30f5e850ad.png)


### 4.1 获取密码

```bash
$ cat /etc/gitlab/initial_root_password 
# WARNING: This value is valid only in the following conditions
#          1. If provided manually (either via `GITLAB_ROOT_PASSWORD` environment variable or via `gitlab_rails['initial_root_password']` setting in `gitlab.rb`, it was provided before database was seeded for the first time (usually, the first reconfigure run).
#          2. Password hasn't been changed manually, either via UI or via command line.
#
#          If the password shown here doesn't work, you must reset the admin password following https://docs.gitlab.com/ee/security/reset_user_password.html#reset-your-root-password.

Password: mmPPA7vlzRPgdEgQXu1LnWbok6OUNgiAgoZvhYnCgrw=

# NOTE: This file will be automatically deleted in the first reconfigure run after 24 hours.
```
`mmPPA7vlzRPgdEgQXu1LnWbok6OUNgiAgoZvhYnCgrw=`是默认的初始密码，
我们可以在`/etc/gitlab/gitlab.rb`配置文件中设置自己的root密码，也可以用默认的密码登陆再修改自己想要的密码。**要注意该文件24小时后自动删除**。


### 4.2 修改密码
![在这里插入图片描述](https://img-blog.csdnimg.cn/cfc035ff0011484e9ed966f428708659.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_14,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/d75d95a641b94b78a27be45cc1f8ec6d.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
修改重新登陆即可。


---

 - [参考官方部署](https://docs.gitlab.com/ee/install/docker.html)
 - [Create a Continuous Integration (CI) Pipeline in Gitlab](https://blog.csdn.net/xixihahalelehehe/article/details/121941628?spm=1001.2014.3001.5501)
 - [gitlab-runner部署](https://blog.csdn.net/xixihahalelehehe/article/details/107755143?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164005071616780261915680%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164005071616780261915680&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-4-107755143.pc_v2_rank_blog_default&utm_term=gitlab&spm=1018.2226.3001.4450)
 - [git 本地项目上传github或gitlab详解](https://blog.csdn.net/xixihahalelehehe/article/details/105240194?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164005071616780261915680%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164005071616780261915680&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-2-105240194.pc_v2_rank_blog_default&utm_term=gitlab&spm=1018.2226.3001.4450)
 - [git与gitlab快速学习手册](https://ghostwritten.blog.csdn.net/article/details/121107739)



## 6. 备份还原

创建 demo 项目，并写入一个文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/dc8d147ec6d34203801679c5561e7acf.png)


开始备份
```bash
$ docker exec  -ti gitlab-ce  gitlab-rake gitlab:backup:create
2023-05-24 11:33:11 UTC -- Dumping database ... 
Dumping PostgreSQL database gitlabhq_production ... [DONE]
2023-05-24 11:33:13 UTC -- Dumping database ... done
2023-05-24 11:33:13 UTC -- Dumping repositories ... 
{"command":"create","gl_project_path":"root/demo","level":"info","msg":"started create","relative_path":"@hashed/6b/86/6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b.git","storage_name":"default","time":"2023-05-24T11:33:13.841Z"}
{"command":"create","gl_project_path":"root/demo.wiki","level":"info","msg":"started create","relative_path":"@hashed/6b/86/6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b.wiki.git","storage_name":"default","time":"2023-05-24T11:33:13.842Z"}
{"command":"create","error":"manager: repository empty: repository skipped","gl_project_path":"root/demo.wiki","level":"warning","msg":"skipped create","relative_path":"@hashed/6b/86/6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b.wiki.git","storage_name":"default","time":"2023-05-24T11:33:13.845Z"}
{"command":"create","gl_project_path":"root/demo","level":"info","msg":"completed create","relative_path":"@hashed/6b/86/6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b.git","storage_name":"default","time":"2023-05-24T11:33:13.853Z"}
{"command":"create","gl_project_path":"root/demo.design","level":"info","msg":"started create","relative_path":"@hashed/6b/86/6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b.design.git","storage_name":"default","time":"2023-05-24T11:33:13.858Z"}
{"command":"create","error":"manager: repository empty: repository skipped","gl_project_path":"root/demo.design","level":"warning","msg":"skipped create","relative_path":"@hashed/6b/86/6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b.design.git","storage_name":"default","time":"2023-05-24T11:33:13.858Z"}
2023-05-24 11:33:13 UTC -- Dumping repositories ... done
2023-05-24 11:33:13 UTC -- Dumping uploads ... 
2023-05-24 11:33:13 UTC -- Dumping uploads ... done
2023-05-24 11:33:13 UTC -- Dumping builds ... 
2023-05-24 11:33:13 UTC -- Dumping builds ... done
2023-05-24 11:33:13 UTC -- Dumping artifacts ... 
2023-05-24 11:33:13 UTC -- Dumping artifacts ... done
2023-05-24 11:33:13 UTC -- Dumping pages ... 
2023-05-24 11:33:13 UTC -- Dumping pages ... done
2023-05-24 11:33:13 UTC -- Dumping lfs objects ... 
2023-05-24 11:33:13 UTC -- Dumping lfs objects ... done
2023-05-24 11:33:13 UTC -- Dumping terraform states ... 
2023-05-24 11:33:13 UTC -- Dumping terraform states ... done
2023-05-24 11:33:13 UTC -- Dumping container registry images ... [DISABLED]
2023-05-24 11:33:13 UTC -- Dumping packages ... 
2023-05-24 11:33:13 UTC -- Dumping packages ... done
2023-05-24 11:33:13 UTC -- Creating backup archive: 1684927991_2023_05_24_16.0.1_gitlab_backup.tar ... 
2023-05-24 11:33:13 UTC -- Creating backup archive: 1684927991_2023_05_24_16.0.1_gitlab_backup.tar ... done
2023-05-24 11:33:13 UTC -- Uploading backup archive to remote storage  ... [SKIPPED]
2023-05-24 11:33:13 UTC -- Deleting old backups ... [SKIPPED]
2023-05-24 11:33:13 UTC -- Deleting tar staging files ... 
2023-05-24 11:33:13 UTC -- Cleaning up /var/opt/gitlab/backups/backup_information.yml
2023-05-24 11:33:13 UTC -- Cleaning up /var/opt/gitlab/backups/db
2023-05-24 11:33:13 UTC -- Cleaning up /var/opt/gitlab/backups/repositories
2023-05-24 11:33:13 UTC -- Cleaning up /var/opt/gitlab/backups/uploads.tar.gz
2023-05-24 11:33:13 UTC -- Cleaning up /var/opt/gitlab/backups/builds.tar.gz
2023-05-24 11:33:13 UTC -- Cleaning up /var/opt/gitlab/backups/artifacts.tar.gz
2023-05-24 11:33:13 UTC -- Cleaning up /var/opt/gitlab/backups/pages.tar.gz
2023-05-24 11:33:13 UTC -- Cleaning up /var/opt/gitlab/backups/lfs.tar.gz
2023-05-24 11:33:13 UTC -- Cleaning up /var/opt/gitlab/backups/terraform_state.tar.gz
2023-05-24 11:33:13 UTC -- Cleaning up /var/opt/gitlab/backups/packages.tar.gz
2023-05-24 11:33:13 UTC -- Deleting tar staging files ... done
2023-05-24 11:33:13 UTC -- Deleting backups/tmp ... 
2023-05-24 11:33:13 UTC -- Deleting backups/tmp ... done
2023-05-24 11:33:13 UTC -- Warning: Your gitlab.rb and gitlab-secrets.json files contain sensitive data 
and are not included in this backup. You will need these files to restore a backup.
Please back them up manually.
2023-05-24 11:33:13 UTC -- Backup 1684927991_2023_05_24_16.0.1 is done.
2023-05-24 11:33:13 UTC -- Deleting backup and restore PID file ... done
[root@gitlab ~]# ls -l /opt/gitlab/data/backups
total 432
-rw------- 1 polkitd render 440320 May 24 19:33 1684927991_2023_05_24_16.0.1_gitlab_backup.tar
[root@gitlab ~]# du -sh /opt/gitlab/data/backups
432K	/opt/gitlab/data/backups

```
我们尝试删除项目
![在这里插入图片描述](https://img-blog.csdnimg.cn/42a2dee86f9a41a1a34bb99252e1e2e9.png)

从默认备份恢复（backups目录下只有一个备份文件时）：

```bash
$ docker exec  -ti gitlab-ce gitlab-rake gitlab:backup:restore
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
ALTER TABLE
[DONE]
Source backup for the database ci doesn't exist. Skipping the task
2023-05-24 11:52:17 UTC -- Restoring database ... done
2023-05-24 11:52:17 UTC -- Restoring repositories ... 
{"command":"restore","gl_project_path":"root/demo","level":"info","msg":"started restore","relative_path":"@hashed/6b/86/6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b.git","storage_name":"default","time":"2023-05-24T11:52:17.405Z"}
{"command":"restore","gl_project_path":"root/demo.wiki","level":"info","msg":"started restore","relative_path":"@hashed/6b/86/6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b.wiki.git","storage_name":"default","time":"2023-05-24T11:52:17.406Z"}
{"command":"restore","error":"manager: repository skipped: restore bundle: doesn't exist","gl_project_path":"root/demo.wiki","level":"warning","msg":"skipped restore","relative_path":"@hashed/6b/86/6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b.wiki.git","storage_name":"default","time":"2023-05-24T11:52:17.412Z"}
{"command":"restore","gl_project_path":"root/demo.design","level":"info","msg":"started restore","relative_path":"@hashed/6b/86/6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b.design.git","storage_name":"default","time":"2023-05-24T11:52:17.422Z"}
{"command":"restore","error":"manager: repository skipped: restore bundle: doesn't exist","gl_project_path":"root/demo.design","level":"warning","msg":"skipped restore","relative_path":"@hashed/6b/86/6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b.design.git","storage_name":"default","time":"2023-05-24T11:52:17.427Z"}
{"command":"restore","gl_project_path":"root/demo","level":"info","msg":"completed restore","relative_path":"@hashed/6b/86/6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b.git","storage_name":"default","time":"2023-05-24T11:52:17.447Z"}
2023-05-24 11:52:17 UTC -- Restoring repositories ... done
2023-05-24 11:52:17 UTC -- Restoring uploads ... 
2023-05-24 11:52:17 UTC -- Restoring uploads ... done
2023-05-24 11:52:17 UTC -- Restoring builds ... 
2023-05-24 11:52:17 UTC -- Restoring builds ... done
2023-05-24 11:52:17 UTC -- Restoring artifacts ... 
2023-05-24 11:52:17 UTC -- Restoring artifacts ... done
2023-05-24 11:52:17 UTC -- Restoring pages ... 
2023-05-24 11:52:17 UTC -- Restoring pages ... done
2023-05-24 11:52:17 UTC -- Restoring lfs objects ... 
2023-05-24 11:52:17 UTC -- Restoring lfs objects ... done
2023-05-24 11:52:17 UTC -- Restoring terraform states ... 
2023-05-24 11:52:17 UTC -- Restoring terraform states ... done
2023-05-24 11:52:17 UTC -- Restoring packages ... 
2023-05-24 11:52:17 UTC -- Restoring packages ... done
This task will now rebuild the authorized_keys file.
You will lose any data stored in the authorized_keys file.
Do you want to continue (yes/no)? yes
2023-05-24 11:53:11 UTC -- Deleting tar staging files ... 
2023-05-24 11:53:11 UTC -- Cleaning up /var/opt/gitlab/backups/backup_information.yml
2023-05-24 11:53:11 UTC -- Cleaning up /var/opt/gitlab/backups/db
2023-05-24 11:53:11 UTC -- Cleaning up /var/opt/gitlab/backups/repositories
2023-05-24 11:53:11 UTC -- Cleaning up /var/opt/gitlab/backups/uploads.tar.gz
2023-05-24 11:53:11 UTC -- Cleaning up /var/opt/gitlab/backups/builds.tar.gz
2023-05-24 11:53:11 UTC -- Cleaning up /var/opt/gitlab/backups/artifacts.tar.gz
2023-05-24 11:53:11 UTC -- Cleaning up /var/opt/gitlab/backups/pages.tar.gz
2023-05-24 11:53:11 UTC -- Cleaning up /var/opt/gitlab/backups/lfs.tar.gz
2023-05-24 11:53:11 UTC -- Cleaning up /var/opt/gitlab/backups/terraform_state.tar.gz
2023-05-24 11:53:11 UTC -- Cleaning up /var/opt/gitlab/backups/packages.tar.gz
2023-05-24 11:53:11 UTC -- Deleting tar staging files ... done
2023-05-24 11:53:11 UTC -- Deleting backups/tmp ... 
2023-05-24 11:53:11 UTC -- Deleting backups/tmp ... done
2023-05-24 11:53:11 UTC -- Warning: Your gitlab.rb and gitlab-secrets.json files contain sensitive data 
and are not included in this backup. You will need to restore these files manually.
2023-05-24 11:53:11 UTC -- Restore task is done.
2023-05-24 11:53:11 UTC -- Deleting backup and restore PID file ... done
```

现在刷新页面，项目又回来了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/5423ca96e1e54bbcac6d8486ec1b4c87.png)


