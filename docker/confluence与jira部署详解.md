

## 链接：

镜像：
https://hub.docker.com/r/centos/postgresql-94-centos7/
https://hub.docker.com/r/cptactionhank/atlassian-jira-software/
https://hub.docker.com/r/cptactionhank/atlassian-confluence/
官网
https://www.cwiki.us/pages/viewpage.action?pageId=2392981
1.镜像
2.docker-compose
3.部署
4.备份还原
5.关联
6.负载均衡


## 1.安装docker

[https://blog.csdn.net/xixihahalelehehe/article/details/104293170](https://blog.csdn.net/xixihahalelehehe/article/details/104293170)

## 2.安装docker-compose

```bash
$ sudo curl -L https://github.com/docker/compose/releases/download/1.17.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
$ docker-compose --version
```

## 3..拉取镜像

```bash
$ docker pull docker.io/centos/postgresql-94-centos7
$ docker pull docker.io/cptactionhank/atlassian-confluence
$ docker pull docker.io/cptactionhank/atlassian-jira-software
```

## 4.docker-compose配置

（1）配置文件

```bash
$ cat docker-compose.yaml
version: '2'
volumes:
  jira:
  confluence:
  postgresql:
services:  
  jira:
    image: docker.io/cptactionhank/atlassian-jira-software:7.4.2 
    ports:
      - 8080:8080
    depends_on:
      - postgresql
    volumes:
      - jira:/var/atlassian/jira
    environment:
       JIRA_DB_HOST: postgresql:5432
       JIRA_DB_PASSWD: UltraApp@2017
       JIRA_DB_USER: ultraapp
       JIRA_DB_NAME: jira
  confluence:
    image: docker.io/cptactionhank/atlassian-confluence:6.3.4
    ports:
      - 8090:8090
    depends_on:
      - postgresql
    volumes:
      - confluence:/var/atlassian/confluence
    environment:
      CONFLUENCE_DB_HOST: postgresql:5432
      CONFLUENCE_DB_PASSWORD: UltraApp@2017
      CONFLUENCE_DB_USER: ultraapp
      CONFLUENCE_DB_NAME: confluence
  postgresql:
    image: docker.io/centos/postgresql-94-centos7:latest
    ports:
      - 5432:5432
    volumes:
      - postgresql:/var/lib/pgsql/data
    environment:
      POSTGRESQL_USER: ultraapp
      POSTGRESQL_PASSWORD: UltraApp@2017
      POSTGRESQL_DATABASE: confluence
      POSTGRESQL_ADMIN_PASSWORD: UltraApp@2017
     
$ docker-compose up 容器全部启动
```

## 5.图形化安装confluence

http://localhost:8090
选择安装生产安装
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cf16bec0cfc411f09cd3d4af350e1b00.png)
申请授权码,点击get。。？获取后，操作按顺序尽量不后退步骤，否则，会报system error！
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fc4fb4c72f02a82ff7b31383d5919cbd.png)
如果做有一个完整的数据库，选择my own database，
若做数据迁移，以及其他设置，需要手动创建。

但不知道两者对保存数据的区别

my own database：
对于新产生的数据，什么保存在数据库中，什么保存在卷中。

built in（没有建立连接数据库步骤）：
新产生的数据是否导入数据库，还是保存在卷中
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bcd6ef84aa18aeb8b9792d87648b4425.png)
三种创建方式：我们选择示范站点。然后就可以**看到初始化界面（图略）**
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2a681bdf8fd7a1ab6e9c6ca0d491b66c.png)

如果选择备份还原站点
点击选择文件，把zip压缩文件导入。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/996a909fcff9b37011e41982c3fc8c3e.png)
备份文件导入
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c427b8543db8f200928084af162b4f3e.png)
含有数据的页面web界面
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8416c183ef2fae3606e93e75adab1dbb.jpeg)
另外，也可以选择建立空白站点，初始化web界面后再导入备份数据。
进入web界面，点击配置，进入系统超级管理员配置界面：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200308234750771.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70
找到在管理栏的备份还原：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/42e403e76953c76a96bbe40d144381c2.png)

## 6.图形化安装jira

1.选择第二个手动安装，“我将设置它自己”
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/833a0d1baeef9440ce13e2840b67cbe2.png)
2.申请授权码
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d1967ad093857251aa8c646a772adf5a.png)

3.选择第二个设置：my own database
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0b2252a3d392fb016c169b78cd77f502.png)

4.设置自己数据库连接
hostname要设置内网地址，即本机的eth0，数据端口本地映射的端口。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6fdc183316fe5783da63337a244d7cbf.jpeg)

5.应用设置，默认即可。

注意到 “import your data”导入数据，点击。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0e91dbd77cbb6991686c9a655ef50f19.jpeg)




走到这一步，在数据卷已经自动生成对应的应用程序配置文件，
命令行把备份压缩包复制在**/var/lib/docker/cj_jira_1/_data/import**里。
点击复原
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e38c835de42f72d968a28b6e500fff69.png)

含有数据的web界面：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/24e8abede9f3a895cdbec2cb66af3754.jpeg)

同样，**jira也可以在初始化web界面进行备份还原，步骤和confluence一样**。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/89145224ee908eeabc6ef4c24e087f17.png)
5.conflunce与jira关联
1.设置域名

2.关联域名，即应用程序

3.confluence与jira各自设置。图略
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a24c38c1009a32cfc36affd2381e80f1.jpeg)
