

---

 - [Jenkins 手册指南](https://ghostwritten.blog.csdn.net/article/details/123042938)

-----

## 1. 简介
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fc068601f4ddfb3f70ac8577625d8e2d.png#pic_center)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fbfd5eb34c310a2ca344a54b43f9bff3.png#pic_center)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/01b45be2e8f18947db314bab5464c065.png#pic_center)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8510e30e388debdb1a5ca0c9d78dc81b.png#pic_center)


##  2. Jenkins安装
### 2.1 环境准备
最小硬件需求：256M、1G磁盘空间，通常根据需要Jenkins服务器至少1G内存，50G+的磁盘空间。
软件需求：由于jenkins是使用java语言编写的，所以需要安装java运行时环境(jdk)

获取安装包
可以从Jenkins官方网站及清华镜像站下载jenkins安装包

 - jenkins-2.164.3-1.1.noarch.rpm
 - jdk-8u211-linux-x64.rpm

```c
# cat /etc/redhat-release 
CentOS Linux release 7.5.18c4 (Core) 
```

###  2.2 安装jdk
先卸系统自带的jdk版本

```c
# rpm -qa | grep jdk
java-1.8.0-openjdk-headless-1.8.0.161-2.b14.el7.x86_64
copy-jdk-configs-3.3-2.el7.noarch
java-1.8.0-openjdk-1.8.0.161-2.b14.el7.x86_64

# rpm -e --nodeps java-1.8.0-openjdk-1.8.0.161-2.b14.el7.x86_64
# rpm -e --nodeps java-1.8.0-openjdk-headless-1.8.0.161-2.b14.el7.x86_64
# rpm -ivh jdk-8u211-linux-x64.rpm
# java -version
java version "1.8.0_211"
Java(TM) SE Runtime Environment (build 1.8.0_211-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.211-b12, mixed mode)
```

### 2.3 安装jenkins（本地）
创建jenkins用户组和用户

```c
# groupadd jenkins
# useradd -g jenkins -s /sbin/nologin -m jenkins  -s 不可以登录  -m创建用户home目录

# rpm -ivh jenkins-2.164.3-1.1.noarch.rpm
```

因为我的是gitlab服务和jenkins部署在一台上
所以我修改了jenkins的默认8080端口为8090

```c
# cat /etc/sysconfig/jenkins | grep JENKINS_PORT
JENKINS_PORT="8090"


启动、配置jenkins
# systemctl start jenkins
# ss -lntp | grep 8090
LISTEN     0      50          :::8090                    :::*                   users:(("java",pid=1858,fd=168))
```

在web上进行访问`http://192.168.137.100:8090`

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/82c4b6985d1fa3270c26ec23b5e7d781.png#pic_center)


```c
# cat /var/lib/jenkins/secrets/initialAdminPassword
6445fe48a5f34f06801937fd1fa44668
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0974baca4354167ae2066044fb4182d2.png#pic_center)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/208e0a224c4e0dbd836e6083826bf063.png#pic_center)


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/463376c40d8a1c0e9596e2f6fc899e1c.png#pic_center)


要重新启动下jenkins

```c
# systemctl restart jenkins
```

http://192.168.137.100:8090/login

 - 账号：admin
 - 密码：6445fe48a5f34f06801937fd1fa44668

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/36f098fc5e70d50e6a655513ff8795f2.png#pic_center)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2cb970f78f543b2744f9e3167a1c557a.png#pic_center)

修改admin的密码
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ac73abaab170e65be12d9ca89e32fd80.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3dffe6511e9d128f18e1d54dbb2eed1d.png#pic_center)


## 3. Jenkins插件管理
Jenkins本身是一个引擎、一个框架，只是提供了很简单功能，其强大的功能都是通过插件来实现的，jenkins有一个庞大的插件生态系统，为jenkins提供丰富的功能扩展。
插件安装的方式：
1、自动插件安装
在jenkins主页面，点击系统管理

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ffd5e975ab816af411271f611092b582.png#pic_center)

出现问题：解决“该Jenkins实例似乎已离线”
There were errors checking the update sites: SocketTimeoutException: connect timed out

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/69f821f18bec6acb77b8f2583f3bda39.png#pic_center)

解决方法： 修改更新地址的https为http

```c
# vim /var/lib/jenkins/hudson.model.UpdateCenter.xml
# systemctl restart jenkins.service 
# cat /var/lib/jenkins/hudson.model.UpdateCenter.xml
<?xml version='1.1' encoding='UTF-8'?>
<sites>
  <site>
    <id>default</id>
    <url>http://updates.jenkins.io/update-center.json</url>
  </site>
</sites>
# 
```

然后刷新页面

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a4350725ca484384cd1a46d6510e8fac.png#pic_center)

2、手工安装插件
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ff65c8a93c408a62d61f129138fa1fd2.png)
[https://mirrors.tuna.tsinghua.edu.cn/jenkins/plugins/ssh/latest/](https://mirrors.tuna.tsinghua.edu.cn/jenkins/plugins/ssh/latest/)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f5355043cffaffcf09741e3b81c71f41.png#pic_center)

下载`ssh.hpi`后，手动安装ssh插件，进入到插件管理页面
提前下载在win电脑上
Web页面添加插件

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d63fbae13c88f9967085c5c8ada81a6b.png#pic_center)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/23357b07124bbdc416fda3819e0ef022.png#pic_center)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/004c4358b9d662381293411d87c13c65.png#pic_center)


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cf9254da01d072031f4f00878f18c4ac.png#pic_center)

安装成功后
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/da3a462c97f51c61ee751f576677823a.png#pic_center)

## 4. Jenkins 目录与文件

Jenkins 常用的目录及文件

学习Jenkin，首先要明白一点，那就是jenkins下一切兼文件，也就是说jenkins没有数据库，所有的数据都是以文件的形式存在，所以要了解jenkins的主要目录及文件，通过命令可以查看到所有的jenkins目录及文件的位置。

```c
# rpm -ql jenkins
/etc/init.d/jenkins
/etc/logrotate.d/jenkins
/etc/sysconfig/jenkins
/usr/lib/jenkins
/usr/lib/jenkins/jenkins.war
/usr/sbin/rcjenkins
/var/cache/jenkins
/var/lib/jenkins
/var/log/jenkins
```



Jenkins的主配置文件
`/etc/sysconfig/jenkins`是jenkins的主配置文件：

```c
# cat /etc/sysconfig/jenkins | grep JENKINS_HOME   主目录
JENKINS_HOME="/var/lib/jenkins"
# cat /etc/sysconfig/jenkins | grep JENKINS_USER  jenkins用户
JENKINS_USER="jenkins"
Jenkins默认的用户为jenkins，然后使用sudo进行授权
# cat /etc/sysconfig/jenkins | grep JENKINS_PORT   端口，默认8080
JENKINS_PORT="8090"
```

主目录：`/var/lib/jenkins`


主目录下的

 - `jobs`目录：存放jobs的配置及每次构建的结果，
 - `plugins`目录：jenkins插件目录，存放已经安装的插件，
 - `Worksspace`：工作区目录，每次job执行构建时的工作目录，
 - `Users`目录：存放与用户相关的配置文件。

Jenkins主程序目录`/usr/lib/jenkins/jenkins.war`  是jenkins的主程序



Jenkins日志文件目录 ：`/var/log/jenkins/`

Jenkins启动文件 ：`/etc/init.d/jenkins`


##  5. Jenkins创建freestyle项目
构建作业是一个持续集成服务器的基本职能，构建`freestyle`的形式多种多样，可以是编译和单元测试，也可能是打包及部署，或者是其它类似的作业。
在jenkins中，构建作业是很容易建立的，而且可以根据需求安装各种插件，来创建多种形式的构建作业，下面是创建自由式作业
自由式构建作业是最灵活和可配置的选项，并且可以用于任何类型的项目，它的配置相对简单，其中很多配置在选项也可以用在其他构建作业中
在jenkins主页面，点击左侧菜单栏的“新建”或者“`New job`”
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f3d2bf415ba70580a10288e695569e5a.png#pic_center)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b71d55a76633211a51973bae35b586ef.png#pic_center)



注意：

1. job名称需要提前规划好，以便于后面的权限管理
  2. 创建job后不要轻易更改名称，因为jenkins一切皆文件，很多关于job的文件，都是以该名称命名，当名称被修改后，一般不会删除旧文件，而是会再重新创建一份新的文件。

进入job的配置页面，此时jenkins的主目录下的jobs目录已经生成了以job名称命名的文件夹

```c
# pwd
/var/lib/jenkins/jobs
# ll -h
总用量 0
drwxr-xr-x 3 jenkins jenkins 38 6月   8 13:55 My-Freestyle-job
```
Job的配置页面，主要是包括通用配置、源码管理、构建触发器、构建环境、构建、构建后操作等几个部分，根据选择的构建类型不同，可能配置项会有一些小的差别。


执行linux命令、脚本实验
先看通用配置选项
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e7f5f58523580762dd2dfacf2d098381.png#pic_center)


勾选“**丢弃旧的构建**”，这是必须提前考虑的重要方面，就是如何处理构建历史，构建作业会消耗大量的磁盘空间，尤其是存储的构建产物(比如执行java构建时会生成的JAR、WAR等)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2b1d267b46c0989125f0d5ad385456a0.png#pic_center)


该选项允许你限制在构建历史记录的作业数。可以设置jenkins只保留最近几次的构建，或者只保留指定数量的构建，此外，jenkins永远不会删除最后一个稳定和成功的构建。

 - 天数：5
 - 最大个数：5
 - 下拉至“构建”部分
 - 添加构建操作
 - 选择“执行shell”

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8c1d51a93ceb0168098c1f72c9676de3.png#pic_center)

在Command窗口
输入要执行的命令。每一行一条，可以输入多条
`Add build step`、添加其它构建
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/147e6f082c52150eed49146aa475440b.png#pic_center)

直接点击保存
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3068e20dabb9d066127b4f6427f6edff.png#pic_center)

回到job主页面
点击立即构建
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/83b90f512b5d701a9f68afcf14b97258.png#pic_center)


点击立即构建，执行job的构建任务，此时的job就是执行一条linux命令
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6e6f3f71f6031fccd5996da5cfbd6d30.png#pic_center)


 - 1：点击工作区，右侧就会列出工作的内容
 - 2：此文件为执行构建时，使用linux命令创建的文件
 - 3：点击这个可以查看本次构建的详细输出

点击`Console Output`
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c875d10baa1db59b0eba851fc32d6361.png#pic_center)


Job执行时的当前工作目录为`jenkins`主目录`+workspaces+`以`job`名称命名的文件夹，知道这一点对于后面写脚本执行部署任务时非常重要。



## 6. 连接gitlab获取仓库代码
在gitlab上复制仓库的地址
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c768b6f79044b56e12d40456f522dcb4.png#pic_center)


选择ssh的方式

```bash
git@192.168.137.100:xmlgrg_test/xm.git
```

然后再`jenkins`上的`My-Freestyle-job`的配置页面，下拉到“源码管理”部分
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c3b1c58fbabb4eb999c31cf3ee3a2b2a.png#pic_center)


发现没有
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/05fda54199dd72f0ec1ea2118d184191.png#pic_center)


那我们就去安装一个git的插件
[https://mirrors.tuna.tsinghua.edu.cn/jenkins/plugins/gitlab-plugin/](https://mirrors.tuna.tsinghua.edu.cn/jenkins/plugins/gitlab-plugin/)
没有安装最新的，最新的好像对版本有要求
[https://mirrors.tuna.tsinghua.edu.cn/jenkins/plugins/gitlab-plugin/1.5.5/gitlab-plugin.hpi](https://mirrors.tuna.tsinghua.edu.cn/jenkins/plugins/gitlab-plugin/1.5.5/gitlab-plugin.hpi)
安装的是1.5.5版本的

然后再去源码管理页面
然后再`jenkins`上的`My-Freestyle-job`的配置页面，下拉到“源码管理”部分，勾选git选项
输入git的URL
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7880fe294f80e0afeabb403083f6f34c.png#pic_center)

发现仓库地址被粘贴后，出现如图错误
显示信息为`key`认证失败，因为使用的是ssh方式连接仓库，所以需要配置ssh认证，因为jenkins服务启动的用户是`jenkins`
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/223d3dd2d75dea0791cf177cb3b5ec57.png)
因为在gitlab上配置的是root用户的公钥，处理这个报错有2个方法。

1. 在jenkins上配置使用root用户的私钥连接gitlab
2. 配置使用root用户启动jenkins  其实就是修改jenkins的配置文件`/etc/sysconfig/jenkins`中启动用户`JENKINS_USER=root`，重启jenkins服务

方式1
点击添加认证、选择jenkins
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e18af4cfd129e23ed4a8c5287cf466b1.png#pic_center)

进入认证添加页面
查看root的ssh私钥

```c
# cat /root/.ssh/id_rsa
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAwva0K+f7IexCRjJknCCZ8hvr1ml6Y4mwOIvNYpNeKXcnlfNA
YnUn+WU4vTDBBMhVjkp3yPpl0l/U1tdvK6mA6g4GKbVpZRDISlQ6rQ==
-----END RSA PRIVATE KEY-----
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b84a797404cb9ea5b957802f65ddc17c.png#pic_center)

点击添加
然后会自动回到源码管理页面
选择认证方式为刚才新建的即可
点击保存

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4b037f969e7fbc12f2978bbf8679d7ee.png#pic_center)

回到job的主页面，点击“立即构建”，构建完成后，就可以在工作区看到从gitlab仓库拉取的代码了
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6e70311e34e51e9fa46a6088754e9e09.png#pic_center)

在console output可以看到整个控制台的输出内容
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8b6e6cc3718b5dfaa91d071caa6de7cc.png#pic_center)

其实在“源码管理”配置部分，可以配置从分支获取代码，也可以配置从标签获取代码、还可以配置从某一次commit获取代码，

分支dev
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/190d4d24c060cfaf1d50b0e8ee990e42.png#pic_center)

保存，点击立即构建
## 7. linux脚本部署httpd服务
在192.168.137.101服务器上安装httpd服务

```c
# yum install -y httpd
# systemctl start httpd.service 
#  ss -lntp | grep httpd
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a232f6178fcf9f002f534ae5b8e0659b.png)
测试
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b7987abf4fd4bc8fe9a47df2b51959e5.png)
Httpd服务的默认网站放在`/var/www/html`目录下
创建一个页面

```c
# pwd
/var/www/html
# cat index.html
<h1>
this one
</h1>
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9a5ffdcbac7dc68fb1f20f38f8b2ceff.png)
配置ssh免密登录
因为要使用脚本将100 gitlab服务器上的程序代码推送到101 httpd服务器上，所以需要配置100到101的ssh免密码登录。

100服务器上，将root用户的公钥复制给远程101服务器

```c
# ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.137.101
root@192.168.137.101's password:  输入101的root密码
Now try logging into the machine, with:   "ssh 'root@192.168.137.101'"


# mkdir -pv /jenkins/gj
mkdir: 已创建目录 "/jenkins"
mkdir: 已创建目录 "/jenkins/gj"
# chown -R jenkins.jenkins /jenkins/gj/
```


编写部署脚本(deploy.sh)

```c
# cat /data/deploy.sh
#!/bin/bash
#目标服务器IP地址
host=$1
#job名称
job_name=$2
#包名
name=web-$(date +%F)-$(($RANDOM+10000))
#打包
cd /var/lib/jenkins/workspace/${job_name}&& tar czf /jenkins/gj/${name}.tar.gz ./*
#发送包到目标服务器
ssh -t ${host} "cd /var/www/ && mkdir ${name}"
scp /jenkins/gj/${name}.tar.gz $host:/var/www/${name}
#解包
ssh -t  ${host} "cd /var/www/${name} && tar xf ${name}.tar.gz && rm -f ${name}.tar.gz"
#使用软连接方式部署服务
ssh -t ${host} "cd /var/www && rm -rf html && ln -s /var/www/${name} /var/www/html"
```



赋予jenkins用户执行deploy.sh权限  

```c
# chown  jenkins.jenkins /data/deploy.sh
```

重启jenkins

```c
# systemctl restart jenkins.service 
```


Jenkins配置构建
在`My -Freestyle -job`配置页面
还是选择执行shell

```c
sh /data/deploy.sh 192.168.137.101 ${JOB_NAME}
```

最后点击保存
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b45f2b75bd0816b7d09698e977abdca9.png#pic_center)

在job页面、点击立即构建
由于权限的问题，我没有解决
所以、配置使用root用户启动`jenkins`  其实就是修改`jenkins`的配置文件`/etc/sysconfig/jenkins`中启动用户`JENKINS_USER=root`，重启jenkins服务


点击立即构建
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5498198586236ad448f02d8943f2bf55.png#pic_center)


构建成功后
在httpd服务器上查看
相应的目录下已经有项目里的数据了
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e40708afc23d0bdf4245a54d3d048964.png#pic_center)

##  8. Git push触发自动构建
在上面的job中，已经成功实现了将`gitlab`中`monitor`仓库的代码部署到`httpd`服务中，但是每次部署还需要手动去点击“立即构建”，下面将实现当`gitlab`收到`push`请求后，就触发`jenkins`构建，将仓库的变化部署到`httpd`服务中。

Jenkins job配置构建触发器
回到`My-Freestyle-job`的配置页面，下拉到构建触发器部分
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/eb7b4a6e94aee9122a98e84973093c25.png#pic_center)


最后点击保存
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/616b013be716d6884d2ebc00a8b69de8.png#pic_center)


Gitlab仓库配置`webhooks`
进入gitlab中`monitor`仓库的设置页面
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/387ffe98d8ee13c98d73fb82d3711fcb.png#pic_center)


点击`Integrations`、进入集成配置页面
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3606f454dfe59ef737200cae4f41905c.png#pic_center)


进入集成配置页面、复制`jenkins`触发器的配置页面`url`及`Token`信息

```bash
http://192.168.137.100:8090/project/My-Freestyle-job
862c1ea8d1e70436351d4b32e9091742
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d9d3b4aa554c106d3d0421124ee9f84f.png#pic_center)


点击`Add webhook` 按钮
因为我是`gitlab`和`jenkins`在同一个服务器上所有会报错

```bash
Url is blocked: Requests to localhost are not allowed
```

在主区域设置允许本地连接即可
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b5245e9551e8f93211e387b8d03eb87d.png#pic_center)


添加成功
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fe5c1723c801dec26cd8366c5e8ec905.png#pic_center)


克隆仓库到101本地
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9e463f5d6bea04d07500b207fed51bfc.png#pic_center)

更改a文件，push至gitlab

```c
# cd xm/
[root@localhost xm]# ll
总用量 0
-rw-r--r-- 1 root root 0 6月  15 13:57 a
-rw-r--r-- 1 root root 0 6月  15 13:57 b
-rw-r--r-- 1 root root 0 6月  15 13:57 c
[root@localhost xm]# vim a
[root@localhost xm]# cat a
this a 0615
[root@localhost xm]# git add .
[root@localhost xm]# git commit -m "one test 0615 101"
[root@localhost xm]# git checkout -b nnn  因为有设置master保护
切换到一个新分支 'nnn'
[root@localhost xm]# git push -u origin nnn
枚举对象: 5, 完成.
对象计数中: 100% (5/5), 完成.
使用 4 个线程进行压缩
压缩对象中: 100% (2/2), 完成.
写入对象中: 100% (3/3), 276 bytes | 276.00 KiB/s, 完成.
总共 3 （差异 0），复用 0 （差异 0）
remote: 
remote: To create a merge request for nnn, visit:
remote:   http://192.168.137.100/xmlgrg_test/xm/merge_requests/new?merge_request%5Bsource_branch%5D=nnn
remote: 
To 192.168.137.100:xmlgrg_test/xm.git
 * [new branch]      nnn -> nnn
分支 'nnn' 设置为跟踪来自 'origin' 的远程分支 'nnn'。

```

然后在jenkins上查看，构建已经完成了
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cbc312e0fe3e9fa37e17a927fd71817c.png#pic_center)


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ee2982cf762ab7d31c9cc7390d2120e7.png#pic_center)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/82230bebb8b8b3635f45e727b3c56bea.png#pic_center)


检测httpd服务下的文件也已经同步更新了
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3d2e9fa6f6b51ad1561fa9e7d417f03a.png#pic_center)

##  9. 升级jenkins
[https://mirrors.tuna.tsinghua.edu.cn/jenkins/war/latest/](https://mirrors.tuna.tsinghua.edu.cn/jenkins/war/latest/)
下载新版`Jenkins.war`文件，替换旧版本war文件，重启即可。
`Jenkins.war`文件的位置一般为`/usr/lib/jenkins/Jenkins.war`。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d3a8b3ec06fe96c1b77fc29a7be5b7d0.png)

```c
# cd /usr/lib/jenkins
# systemctl  stop jenkins
# mv jenkins.war jenkins.war-bak
# rz
# systemctl  start jenkins
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e59b7468ddc909f33c026a1c8ee045c2.png)

##  10. Jenkinsfile文件
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f4102793be6ce0dfbe5037f57a3ade04.png#pic_center)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/effd6b53c0f08274864d5d1bd2552f1d.png#pic_center)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b2d07ead78a0c198806ee9cd413a2577.png#pic_center)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9bd92b32deebee313d120e06a9b853cd.png#pic_center)


```bash
node {
    stage("Pull Code"){
        echo "获取代码"
        git branch: 'master', url: 'git@192.168.137.100:xmlgrg_test/javaone.git'
    }
	stage("打包"){		
			sh "tar czf frontend-${BUILD_NUMBER}.tar.gz ./* --exclude=./git --exclude=Jenkinsfile" 		
	}
	stage('deploy'){
		
			sh 'scp frontend-${BUILD_NUMBER}.tar.gz 192.168.137.101:/var/www/html'
			sh 'ssh 192.168.137.101 "cd /var/www/html && tar zxxf frontend-${BUILD_NUMBER}.tar.gz  && rm -rf  frontend-*"'
	}
    stage('Unit Test'){
        echo "Unit Test"
		sh 'curl http://192.168.137.101'
    }
}
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/989be3fea561513d56eaf486a44c2ec5.png#pic_center)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c961fe03c4a4bc2a1a233cce216190bb.png#pic_center)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/68aed1383ddf3bdbeea89bec7536cea6.png#pic_center)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/56420fbec925ed2b1fc60b858c05bd8a.png#pic_center)


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e68efde9855211a143c35470b79e0c97.png)
进一步改良命令

```c
pipeline {
    agent any 
    stages {

        stage('Build') { 
		
            steps {
                echo "获取代码"
				git branch: 'master', url: 'git@192.168.137.100:xmlgrg_test/javaone.git'
    
            }
        }
        stage('Test') { 
            steps {
                sh "tar czf frontend-${BUILD_NUMBER}.tar.gz ./* --exclude=./git --exclude=Jenkinsfile"
            }
        }
        stage('Deploy') { 
            steps {
                sh 'scp frontend-${BUILD_NUMBER}.tar.gz 192.168.137.101:/var/www'
				sh 'ssh 192.168.137.101 "rm -rf /var/www/html && cd /var/www && mkdir /var/www/html && tar -zxf frontend-${BUILD_NUMBER}.tar.gz -C /var/www/html && rm -rf /var/www/frontend-${BUILD_NUMBER}.tar.gz "'
            }
        }
		 stage('Unit Test') { 
            steps {
				echo "Unit Test"
				sh 'curl http://192.168.137.101'
            }
        }	
    }	
}
```
测试Git push 就会更新101服务器上的html文件
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c6787f137d28626054d898c7d42e38c5.png)
修改jenkinsfile

```c
#!/usr/bin/env groovy Jenkinsfile
pipeline {
    agent any 
    stages {

        stage('Build') { 
		
            steps {
                echo "获取代码"
				git branch: 'master', url: 'git@192.168.137.100:xmlgrg_test/javaone.git'
    
            }
        }
        stage('Test') { 
            steps {
                sh "tar czf frontend-${BUILD_NUMBER}.tar.gz ./* --exclude=./git --exclude=Jenkinsfile"
            }
        }
        stage('Deploy') { 
            steps {
                sh 'scp frontend-${BUILD_NUMBER}.tar.gz 192.168.137.101:/var/www'
				sh 'ssh 192.168.137.101 "rm -rf /var/www/html && cd /var/www && mkdir /var/www/html && tar -zxf frontend-${BUILD_NUMBER}.tar.gz -C /var/www/html && rm -rf /var/www/frontend-${BUILD_NUMBER}.tar.gz "'
            }
        }
		 stage('Unit Test') { 
            steps {
				echo "Unit Test"
				sh 'curl http://192.168.137.101'
            }
        }

    }
	
}
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1376ebcaabf61611885574df6caf0953.png#pic_center)
设置scm   方法在上面
设置Git push触发自动构建 方法在上面
结果如图

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e014d3e121ca96421682fd00238a96d3.png#pic_center)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9c123e6c257672b2fff4bbbfbc8b1b82.png)

