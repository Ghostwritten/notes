# Docker Swarm 部署界面 UI Portainer



---

## 1. 创建集群
第一台机器：

```bash
docker swarm init
```

第二台机器：

```bash
token=$(ssh -o StrictHostKeyChecking=no 172.17.0.46 "docker swarm join-token -q worker") && echo $token
docker swarm join 172.17.0.46:2377 --token $token
```
第一台机器：

```bash
docker node ls
```

## 2. Deploy Portainer
配置了集群后，下一阶段是部署Portainer。Portainer作为运行在Docker集群或Docker主机上的容器部署。
要完成这个场景，需要将Portainer部署为Docker服务。通过部署Docker服务，Swarm将确保该服务始终在管理器上运行，即使主机宕机。

该服务对外公开`9000`端口，并将内部Portainer数据保存在“`/host/data`”目录下。当Portainer启动时，它使用docker进行连接。`sock`文件到`Docker Swarm manager`。

还有一个附加的约束，即容器只能在管理器节点上运行
第一台执行：
```bash
docker service create \
    --name portainer \
    --publish 9000:9000 \
    --constraint 'node.role == manager' \
    --mount type=bind,src=/host/data,dst=/data \
     --mount type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
    portainer/portainer \
    -H unix:///var/run/docker.sock
```
另一种运行Portainer的方法是直接在主机上运行。在本例中，该命令在端口9000上暴露Portainer仪表板，将数据持久化到主机，并通过Docker连接到它正在运行的Docker主机。袜子文件。

```bash
docker run -d -p 9000:9000 --name=portainer \
  -v "/var/run/docker.sock:/var/run/docker.sock" \
  -v /host/data:/data \
  portainer/portainer
```
## 3. 访问 Portainer Dashboard
随着Portainer的运行，现在可以通过UI访问仪表板并管理集群。仪表板运行在Port 9000上，可以通过这个链接访问
第一个屏幕要求您为admin用户创建一个密码
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/527a0a5bede708e1777186ddeeb75f04.png)
配置完成后，第二个屏幕将要求您使用定义的密码登录到仪表板。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3534daa58e038904f4f9f9bbe7ff401b.png)
## 4. 部署模板
Portainer的众多特性之一是，它可以基于预定义的容器部署服务。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3dec8559c8390ab21e7f01a08820c4c0.png)
在这种情况下，您将部署nginx模板。

 - 通过“应用模板”页签查看可用的模板。
 - 选择nginx模板
 - 例如，为容器输入一个友好的名称`nginx-web`
 - 勾选“显示高级选项”，将80端口绑定到主机端口80
 - 创建容器对象
 - 访问容器通过80端口

## 5. 管理容器
将部署一个Nginx实例。使用指示板，您将看到状态并能够控制集群。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/89f8786688eee5bad0ede23e580a2dfd.png)

参考：

 - [Swarmpit web user interface for your Docker Swarm cluster](https://dockerswarm.rocks/swarmpit/)
 - [Docker UI SWIRL](https://github.com/cuigh/swirl)
 - [Docker Portainer](https://www.portainer.io/)

