

----
## 1. 镜像
dockerhub官网：[https://hub.docker.com/](https://hub.docker.com/)
prometheus镜像为示例
### 1.1 拉取 (docker pull)

```bash
$ docker search prom/prometheus #搜索镜像
```

```bash

$ docker pull prom/prometheus    #默认版本latest
$ docker pull docker.io/prom/prometheus   #默认版本latest
$ docker pull docker.io/prom/prometheus:2.3.1   # 指定版本
```

```bash
$ docker images #查看镜像列表
$ docker images -a  #<none>标签得镜像也能被展示
```

### 1.2 查看配置信息 (docker inspect)

```bash
$ docker inspect prom/prometheus:latest
```

### 1.3 修改tag (docker tag)

```bash
$ docker tag prom/prometheus:latest prom/prometheus:v1.0  #修改版本
$ docker tag prom/prometheus:latest docker.registry.localhost/prometheus:latest #修改仓库名，修改自己的私有仓库docker.registry.localhost（自定义）
```

### 1.4 打包与解包 (docker save|load)
打包
```bash
$ docker save -o prometheus.tar prom/prometheus:latest #第一种方式打包
$ docker save > prometheus.tar prom/prometheus:latest  #第二种方式打包
$ docker save -o monitor.tar prom/prometheus:latest prom/alertmanager:latest # 多个镜像打包
```
解包

```bash
$ docker load -i prometheus.tar  #第一种方式解包
$ docker load < prometheus.tar  #第二种方式解包
```

### 1.5 推送 (docker push)
#将本地的镜像推送至公共镜像源的自己仓库，一般区别官方镜像，为私人定制。ghostwritten为我的仓库名

```bash
$ docker push docker.io/ghostwritten/prometheus:latest  
```

将本地的镜像推送至本地搭建的私有仓库，一般为内网集群环境公用。搭建私有仓库请点击

```bash
$ docker push docker.registry.localhost/prometheus:latest  
$ docker push docker.registry.localhost/monitor/prometheus:latest  #加monitor tag方便区分镜像类型
```

### 1.6 删除 (docker rmi)

```bash
$ docker rmi prom/prometheus:latest
$ docker rmi  -f prom/prometheus:latest  #-f 为强制删除
```

## 2 容器
### 2.1 创建容器 (docker run)
[更多细节请点击](https://woj.app/4255.html)
[参数细节](https://docs.docker.com/engine/reference/run/)
```bash
 -d --detach  #后台运行容器
 -i --interactive #保持标准输入流（stdin）对容器开发
 -t --tty #为容器分配一个虚拟终端
 -e --env username="ritchie": #为容器配置环境变量
 --env-file=[]: #从指定文件读入环境变量；
 --volume , -v : #绑定一个卷
 --volume-from <container> #挂载另一个容器的存储卷与本容器
 -P: #随机端口映射，容器内部端口随机映射到主机的端口
 -p: #指定端口映射，格式为：主机(宿主)端口:容器端口
 --restart=always #自动重启
 --rm #停止即删除
 --entrypoint #运行指定程序,入口点
 --expose=[]: 开放一个端口或一组端口；
 --name="nginx-lb": #为容器指定一个名称；
 --cpuset="0-2" or --cpuset="0,1,2": #绑定容器到指定CPU运行；
 -m :#设置容器使用内存最大值；
 --net="bridge": #指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；
 --dns 8.8.8.8: #指定容器使用的DNS服务器，默认和宿主一致；
 --dns-search example.com: #指定容器DNS搜索域名，默认和宿主一致；
 --add-host #添加域名解析
 -h  --hostname "mars": #指定容器的hostname
 --link <name or id>:alias     #name和id是源容器的name和id，alias是源容器在link下的别名 添加链接到另一个容器，是单向的网络依赖。
 --read-only  #只读容器
 --privileged #root拥有真正的root权限,可以看到很多host上的设备，并且可以执行mount,甚至允许在docker容器中启动docker容器
```

```bash
docker run -d prom/prometheus:latest  #后台模式启动一个容器，容器名随机
docker run -d --name prometheus prom/prometheus:latest  #后台模式启动一个容器，容器名定义为prometheus
docker run -d -P --name prometheus prom/prometheus:latest  #后台模式启动一个容器，容器名定义为prometheus，容器的80端口映射到主机随机端口
```
限制内存使用上限

```go
$ docker run -it -m 300M --memory-swap -1 --name con1 u-stress /bin/bash
```

正常情况下， --memory-swap 的值包含容器可用内存和可用 swap。所以 --memory="300m" --memory-swap="1g" 的含义为：
容器可以使用 300M 的物理内存，并且可以使用 700M(1G -300M) 的 swap。--memory-swap 居然是容器可以使用的物理内存和可以使用的 swap 之和！

把 --memory-swap 设置为 0 和不设置是一样的，此时如果设置了 --memory，容器可以使用的 swap 大小为 --memory 值的两倍。

如果 --memory-swap 的值和 --memory 相同，则容器不能使用 swap

限制可用的 CPU 个数

```go
$ docker run -it --rm --cpus=2 u-stress:latest /bin/bash
```

指定固定的 CPU

```go
$ docker run -it --rm --cpuset-cpus="1" u-stress:latest /bin/bash
```

设置使用 CPU 的权重

```go
$ docker run -it --rm --cpuset-cpus="0" --cpu-shares=512 u-stress:latest /bin/bash
$ docker run -it --rm --cpuset-cpus="0" --cpu-shares=1024 u-stress:latest /bin/bash
```

**注意：**
当 CPU 资源充足时，设置 CPU 的权重是没有意义的。只有在容器争用 CPU 资源的情况下， CPU 的权重才能让不同的容器分到不同的 CPU 用量。--cpu-shares 选项用来设置 CPU 权重，它的默认值为 1024。我们可以把它设置为 2 表示很低的权重，但是设置为 0 表示使用默认值 1024。
### 2.2 查看容器状态 (docker ps)
[更多细节请点击](https://blog.yaodataking.com/2017/04/09/docker-ps/)


```bash
$ docker ps         #查看启动的容器
$ docker ps -n 3    #查看前三个容器
$ docker ps -q     #查看启动的容器ID
$ docker ps -a     #查看全部容器，包括停止的
$ docker ps -a -q  #查看全部容器ID
$ docker ps --filter status=running   #查看状态为启动的容器
```

```bash
$ docker top prometheus #查看容器进程
$ docker exec prometheus ps #查看容器进程
```

### 2.3 查看容器日志 (docker logs)
[更多细节请点击](https://segmentfault.com/a/1190000010086763)


```bash
$ docker logs prometheus
$ docker logs -f prometheus
$ docker logs -f --tail 200 prometheus #查看日志后200行
```

### 2.4 查看容器配置信息 (docker inspect)
[更多细节请点击](https://www.cnblogs.com/kevingrace/p/6424476.html)

```bash
$ docker inspect prometheus
docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' prometheus #指定查看具体网卡地址ip配置信息
```

### 2.5 查看容器内文件操作（docker diff）
- A开头：表示文件被添加
- C开头：表示文件被修改
- D开头：表示文件被删除
```bash
$ docker diff prometheus
```

### 2.5 重命名容器 (docker rename)

```bash
$ docker rename prometheus prometheus-new
```
**注意**：不影响运行状态

### 2.6 将容器构建镜像 (docker commit)

```bash
$ docker commit <container> <new-image>
$ docker commit prometheus prometheus:v2
$ docker commit -a "@author" -m "added somthing" prometheus prometheus:v3
```
注意：当通过容器构建的镜像，也许你再用新镜像创建容器时发现无法启动，这是因为创建镜像的容器附带了`/bin/bash`命令，当你通过这个容器构建的镜像创建容器时，启动一个shell命令它会停止它，如果你想启动某一个命令，你可以以某个命令通过`--entrypoint`作为一个入口点。

```bash
$ docker run -tid --name git-cmd --entrypoint git ubuntu:latest
$ docker -a "@ghostwritten" -m "action git "  git-cmd ubuntu-git:v1
$ docker run -tid --name git-cmd2 git-cmd ubuntu-git:v1 version
```

### 2.6 打包与解包 (docker export / import)
打包

```bash
$ docker export -o prometheus.tar prometheus
$ docker export > prometheus.tar prometheus
```
解包

```bash
$ docker import prometheus.tar prometheus-new #默认tag为latest
$ docker import prometheus.tar prometheus-new:v1.0 #为新镜像指定name和tag
```

### 2.7 启动停止容器 (docker start / stop)

```bash
$ docker start prometheus #启动
$ docker stop prometheus #停止
$ docker restart prometheus #重启
$ docker kill prometheus #强制停止
```

### 2.8 删除容器(docker rm)

```bash
$ docker rm prometheus
$ docker rm -f prometheus
```
### 2.9 更新容器运行参数（docker update）

```bash
$ docker update --restart=always prometheus
$ docker update --cpu-period=100000 --cpu-quota=20000 prometheus
```
### 3.0 容器逻辑卷管理 (docker volume)
```bash
docker volume ls        #查看容器卷列表
docker volume inspect <容器卷>   #查看容器卷配置信息
docker volume rm <容器卷>   #删除容器卷
```

###  3.1 容器网络管理 (docker network)

```bash
docker network ls     #容器网络列表
docker network inspect <容器网络> #查看容器网络配置信息
docker network rm <容器网络>   #删除容器网络
docker network rm  -f <容器网络>   #强制删除容器网络
docker network disconnect <容器网络> <容器名称>  #断开容器网络连接
docker network disconnect --force <容器网络> <容器名称>  #强制断开容器网络连接
```

###  docker service

docker swarm init
```bash
#Docker Swarm 启动了两个 Nginx 容器实例。其中，第一条 create 命令创建了这两个容器
#第二条 update 命令则把它们“滚动更新”成了一个新的镜像。
$ docker service create --name nginx --replicas 2  nginx
$ docker service update --image nginx:1.7.9 nginx
```

参考资料：
[https://docs.docker.com/engine/reference/commandline/registry/](https://docs.docker.com/engine/reference/commandline/registry/)


