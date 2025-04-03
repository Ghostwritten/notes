

---

 - [http://wiki.qingscan.songboy.site/2559367](http://wiki.qingscan.songboy.site/2559367)

----
## 1. 准备

### 1.1 安装docker

```bash
curl -sSL https://get.daocloud.io/docker | sh
 
```

### 1.2 安装docker-compose
安装compose命令如下：

```bash
sudo curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```

 
设置可执行权限

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

##  2. 部署
### 2.1 运行

```bash
git clone https://github.com/78778443/QingScan  
cd QingScan/docker/latest
docker-compose up -d 
```

首次启动需要更新容器内代码
```bash
$ docker exec  qingscan sh -c 'cd /root/qingscan && git fetch && git reset --hard origin/develop && rm code/public/install/install.lock' 
From https://gitee.com/songboy/QingScan
   20dda38..3b6175f  develop    -> origin/develop
   3235af9..734163e  main       -> origin/main
HEAD is now at 3b6175f 批量导入

```
### 2.2 创建数据库
依次执行命令创建MySQL数据库
进入容器

```bash
docker exec -it  mysqlser bash
```

进入数据库

```bash
mysql -uroot -p123
```

创建数据库

```bash
CREATE DATABASE IF NOT EXISTS QingScan;
```

## 3. 访问应用
浏览器访问 http://127.0.0.1:8000/ 自动进入安装界面
1.	fortify 涉及许可证问题，镜像内不包含，需要自己将Linux版本的fortify放到/data/tools文件夹中
2.	AWVS 调用主要通过API，需要自己将API配置系统，配置管理中去

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9aa242fb7860363c353ec42e3f76fe19.png)



