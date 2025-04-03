![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a92b9b4c81ed49e2a344080ce7ca3b24.png)




# 背景


当前版本 gitab-ce 16.0.1，	准备升级最新版本（17.3.1）。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/37109d33fae4409dbeb4666dcf82bf15.png)

- [先迁移gitlab数据至测试节点](https://ghostwritten.blog.csdn.net/article/details/121929582)。


# gitlab-ce 16.0.1 升级 17.3.1 失败




```bash

#!/bin/bash
GITLAB_HOME='/data/gitlab'
docker run -tid --hostname gitlab.bsgchina.com -p 443:443 -p 80:80 -p 30022:22 --name gitlab-17.3.1-ce --restart always -v $GITLAB_HOME/config:/etc/gitlab:Z -v $GITLAB_HOME/logs:/var/log/gitlab:Z -v $GITLAB_HOME/data:/var/opt/gitlab:Z --shm-size 256m gitlab/gitlab-ce:17.3.1-ce.0

```

```bash
#!/bin/bash
GITLAB_HOME='/data/gitlab'
docker run -tid --hostname gitlab.bsgchina.com -p 443:443 -p 80:80 -p 30022:22 --name gitlab-17.3.1-ce --restart always -v $GITLAB_HOME/config:/etc/gitlab:Z -v $GITLAB_HOME/logs:/var/log/gitlab:Z -v $GITLAB_HOME/data:/var/opt/gitlab:Z --shm-size 256m gitlab/gitlab-ce:17.3.1-ce.0
```

```bash
[root@gitlab-backup2 gitlab]# docker logs -f 55de
Thank you for using GitLab Docker Image!
Current version: gitlab-ce=17.3.1-ce.0

Configure GitLab for your system by editing /etc/gitlab/gitlab.rb file
And restart this container to reload settings.
To do it use docker exec:

  docker exec -it gitlab editor /etc/gitlab/gitlab.rb
  docker restart gitlab

For a comprehensive list of configuration options please see the Omnibus GitLab readme
https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md

If this container fails to start due to permission problems try to fix it by executing:

  docker exec -it gitlab update-permissions
  docker restart gitlab

Cleaning stale PIDs & sockets
It seems you are upgrading from 16.0.1 to 17.3.1.
It is required to upgrade to the latest 16.11.x version first before proceeding.
Please follow the upgrade documentation at https://docs.gitlab.com/ee/update/index.html#upgrading-to-a-new-major-version
Thank you for using GitLab Docker Image!
Current version: gitlab-ce=17.3.1-ce.0

Configure GitLab for your system by editing /etc/gitlab/gitlab.rb file
And restart this container to reload settings.
To do it use docker exec:

  docker exec -it gitlab editor /etc/gitlab/gitlab.rb
  docker restart gitlab

For a comprehensive list of configuration options please see the Omnibus GitLab readme
https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md

If this container fails to start due to permission problems try to fix it by executing:

  docker exec -it gitlab update-permissions
  docker restart gitlab

Cleaning stale PIDs & sockets
It seems you are upgrading from 16.0.1 to 17.3.1.
It is required to upgrade to the latest 16.11.x version first before proceeding.
Please follow the upgrade documentation at https://docs.gitlab.com/ee/update/index.html#upgrading-to-a-new-major-version
Thank you for using GitLab Docker Image!
Current version: gitlab-ce=17.3.1-ce.0

Configure GitLab for your system by editing /etc/gitlab/gitlab.rb file
And restart this container to reload settings.
To do it use docker exec:

  docker exec -it gitlab editor /etc/gitlab/gitlab.rb
  docker restart gitlab

For a comprehensive list of configuration options please see the Omnibus GitLab readme
https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md

If this container fails to start due to permission problems try to fix it by executing:

  docker exec -it gitlab update-permissions
  docker restart gitlab

Cleaning stale PIDs & sockets
It seems you are upgrading from 16.0.1 to 17.3.1.
It is required to upgrade to the latest 16.11.x version first before proceeding.
Please follow the upgrade documentation at https://docs.gitlab.com/ee/update/index.html#upgrading-to-a-new-major-version
[root@gitlab-backup2 gitlab]# docker logs -f 55de 
```

# gitlab-ce 16.0.1 升级 16.11.8 失败


```bash
#!/bin/bash
GITLAB_HOME='/data/gitlab'
docker run -tid --hostname gitlab.bsgchina.com -p 443:443 -p 80:80 -p 30022:22 --name gitlab-16.11.8-ce --restart always -v $GITLAB_HOME/config:/etc/gitlab:Z -v $GITLAB_HOME/logs:/var/log/gitlab:Z -v $GITLAB_HOME/data:/var/opt/gitlab:Z --shm-size 256m gitlab/gitlab-ce:16.11.8-ce.0
```


```bash
Thank you for using GitLab Docker Image!
Current version: gitlab-ce=16.11.8-ce.0

Configure GitLab for your system by editing /etc/gitlab/gitlab.rb file
And restart this container to reload settings.
To do it use docker exec:

  docker exec -it gitlab editor /etc/gitlab/gitlab.rb
  docker restart gitlab

For a comprehensive list of configuration options please see the Omnibus GitLab readme
https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md

If this container fails to start due to permission problems try to fix it by executing:

  docker exec -it gitlab update-permissions
  docker restart gitlab

Cleaning stale PIDs & sockets
It seems you are upgrading from 16.0.1 to 16.11.8.
It is required to upgrade to the latest 16.7.x version first before proceeding.
Please follow the upgrade documentation at https://docs.gitlab.com/ee/update/#upgrade-paths
Thank you for using GitLab Docker Image!
Current version: gitlab-ce=16.11.8-ce.0

Configure GitLab for your system by editing /etc/gitlab/gitlab.rb file
And restart this container to reload settings.
To do it use docker exec:

  docker exec -it gitlab editor /etc/gitlab/gitlab.rb
  docker restart gitlab

For a comprehensive list of configuration options please see the Omnibus GitLab readme
https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md

If this container fails to start due to permission problems try to fix it by executing:

  docker exec -it gitlab update-permissions
  docker restart gitlab

Cleaning stale PIDs & sockets
It seems you are upgrading from 16.0.1 to 16.11.8.
It is required to upgrade to the latest 16.7.x version first before proceeding.
Please follow the upgrade documentation at https://docs.gitlab.com/ee/update/#upgrade-paths
```

# gitlab-ce 16.0.1 升级 16.7.9 失败


```bash
cat start-16.7.9-ce.0.sh 
#!/bin/bash
GITLAB_HOME='/data/gitlab'
docker run -tid --hostname gitlab.bsgchina.com -p 443:443 -p 80:80 -p 30022:22 --name gitlab-16.7.9-ce.0 --restart always -v $GITLAB_HOME/config:/etc/gitlab:Z -v $GITLAB_HOME/logs:/var/log/gitlab:Z -v $GITLAB_HOME/data:/var/opt/gitlab:Z --shm-size 256m gitlab/gitlab-ce:16.7.9-ce.0
```

```bash
Thank you for using GitLab Docker Image!
Current version: gitlab-ce=16.7.9-ce.0

Configure GitLab for your system by editing /etc/gitlab/gitlab.rb file
And restart this container to reload settings.
To do it use docker exec:

  docker exec -it gitlab editor /etc/gitlab/gitlab.rb
  docker restart gitlab

For a comprehensive list of configuration options please see the Omnibus GitLab readme
https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md

If this container fails to start due to permission problems try to fix it by executing:

  docker exec -it gitlab update-permissions
  docker restart gitlab

Cleaning stale PIDs & sockets
It seems you are upgrading from 16.0.1 to 16.7.9.
It is required to upgrade to the latest 16.3.x version first before proceeding.
Please follow the upgrade documentation at https://docs.gitlab.com/ee/update/#upgrade-paths
```

# gitlab-ce 16.0.1 升级 16.3.8 成功

数据保留

停止原来 gitlab

```bash
$ docker stop gitlab-16.0.1-ce.0
```
运行新版本 gitlab
```bash
$ vim start-16.3.8-ce.0.sh
#!/bin/bash
GITLAB_HOME='/data/gitlab'
docker run -tid --hostname gitlab.bsgchina.com -p 443:443 -p 80:80 -p 30022:22 --name gitlab-16.3.8-ce.0 --restart always -v $GITLAB_HOME/config:/etc/gitlab:Z -v $GITLAB_HOME/logs:/var/log/gitlab:Z -v $GITLAB_HOME/data:/var/opt/gitlab:Z --shm-size 256m gitlab/gitlab-ce:16.3.8-ce.0

$ sh start-16.3.8-ce.0.sh
```

查看日志

```bash
$ docker logs -f gitlab-16.3.8-ce.0
```

**等待10分钟。**

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/820e064679bf481f99f0e09ea9fd51f3.png)


#  gitlab-ce 16.3.8 升级 16.11.8 失败
数据保留

停止原来 gitlab

```bash
$ docker stop gitlab-16.3.8-ce.0
```
运行新版本 gitlab

```bash
$ vim start-16.11.8-ce.0_2.sh
#!/bin/bash
GITLAB_HOME='/data/gitlab'
docker run -tid --hostname gitlab.bsgchina.com -p 443:443 -p 80:80 -p 30022:22 --name gitlab-16.11.8-ce.0 -v /data/gitlab2/config:/etc/gitlab:Z -v $GITLAB_HOME/logs:/var/log/gitlab:Z -v $GITLAB_HOME/data:/var/opt/gitlab:Z --shm-size 256m gitlab/gitlab-ce:16.11.8-ce.0

$ sh start-16.11.8-ce.0_2.sh
```

```bash
Thank you for using GitLab Docker Image!
Current version: gitlab-ce=16.11.8-ce.0

Configure GitLab for your system by editing /etc/gitlab/gitlab.rb file
And restart this container to reload settings.
To do it use docker exec:

  docker exec -it gitlab editor /etc/gitlab/gitlab.rb
  docker restart gitlab

For a comprehensive list of configuration options please see the Omnibus GitLab readme
https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md

If this container fails to start due to permission problems try to fix it by executing:

  docker exec -it gitlab update-permissions
  docker restart gitlab

Cleaning stale PIDs & sockets
It seems you are upgrading from 16.3.8 to 16.11.8.
It is required to upgrade to the latest 16.7.x version first before proceeding.
Please follow the upgrade documentation at https://docs.gitlab.com/ee/update/#upgrade-paths
```

#  gitlab-ce 16.3.8 升级 16.7.9 成功

项目与用户数据保留完成。gitlab-runner 需要是否重新注册待确认。
数据保留

停止原来 gitlab

```bash
$ docker stop gitlab-16.3.8-ce.0
```
运行新版本 gitlab


```bash
$ vim start-16.7.9-ce.0_2.sh
#!/bin/bash
GITLAB_HOME='/data/gitlab'
docker run -tid --hostname gitlab.bsgchina.com -p 443:443 -p 80:80 -p 30022:22 --name gitlab-16.7.9-ce.0 -v /data/gitlab2/config:/etc/gitlab:Z -v $GITLAB_HOME/logs:/var/log/gitlab:Z -v $GITLAB_HOME/data:/var/opt/gitlab:Z --shm-size 256m gitlab/gitlab-ce:16.7.9-ce.0

$ sh start-16.7.9-ce.0_2.sh
```

项目与用户数据保留完成。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/73441cf4e36c4f5b8024050fe13a0f31.png)

#  gitlab-ce 16.7.9 升级 16.11.8 成功

项目与用户数据保留完成。gitlab-runner 需要是否重新注册待确认。

```bash
$ vim start-16.11.8-ce.0_2.sh 
#!/bin/bash
GITLAB_HOME='/data/gitlab'
docker run -tid --hostname gitlab.bsgchina.com -p 443:443 -p 80:80 -p 30022:22 --name gitlab-16.11.8-ce.0 -v /data/gitlab2/config:/etc/gitlab:Z -v $GITLAB_HOME/logs:/var/log/gitlab:Z -v $GITLAB_HOME/data:/var/opt/gitlab:Z --shm-size 256m gitlab/gitlab-ce:16.11.8-ce.0
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/2ad4c848c56146d4ad20f6d11747ae49.png)

#  gitlab-ce 16.11.8 升级 17.3.1 成功

项目与用户数据保留完成。gitlab-runner 需要是否重新注册待确认。
数据保留

停止原来 gitlab

```bash
$ docker stop gitlab-16.11.8-ce.0
```
运行新版本 gitlab


```bash
$ cat start-17.3.1-ce.0_2.sh 
#!/bin/bash
GITLAB_HOME='/data/gitlab'
docker run -tid --hostname gitlab.bsgchina.com -p 443:443 -p 80:80 -p 30022:22 --name gitlab-17.3.1-ce.0 -v /data/gitlab2/config:/etc/gitlab:Z -v $GITLAB_HOME/logs:/var/log/gitlab:Z -v $GITLAB_HOME/data:/var/opt/gitlab:Z --shm-size 256m gitlab/gitlab-ce:17.3.1-ce.0
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/38f67e32bdb848f2a995b95eeae66fc6.png)


# #  gitlab-ce 17.3.1 升级 17.5.1 成功

项目与用户数据保留完成。gitlab-runner 需要是否重新注册待确认。
数据保留

停止原来 gitlab

```bash
$ docker stop gitlab-17.3.1-ce.0
$ vim gitlab-update-17.5.1-ce.0.sh 
#!/bin/bash
GITLAB_HOME='/opt/gitlab'
docker run -tid --hostname gitlab.bsgchina.com -p 443:443 -p 80:80 -p 2222:22 --name gitlab-17.5.1-ce.0 --restart no  -v $GITLAB_HOME/config:/etc/gitlab:Z -v $GITLAB_HOME/logs:/var/log/gitlab:Z -v $GITLAB_HOME/data:/var/opt/gitlab:Z --shm-size 256m gitlab/gitlab-ce:17.5.1-ce.0 
$ sh gitlab-update-17.5.1-ce.0.sh 
$ docker logs -f gitlab-17.5.1-ce.0
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/125923bea5334248accb7cd8eb572a6f.png)


参考：

- [https://docs.gitlab.com/](https://docs.gitlab.com/)
- [https://docs.gitlab.com/ee/update/](https://docs.gitlab.com/ee/update/)
