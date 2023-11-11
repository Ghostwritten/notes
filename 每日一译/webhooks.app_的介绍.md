# webhooks.app 的介绍
![在这里插入图片描述](https://img-blog.csdnimg.cn/7fc67d19479d46ee9c46e4a429977eaf.png)





文章详细介绍 webhook 应用程序的用途：它的架构、如何在本地和 Kubernetes 集群上部署它，以及它所依赖的各种项目。

##  1. 简介

[webhooks.app](https://webhooks.app/)是一个遵循微服务架构的开源应用程序。它的目的是为演示提供一个 webhook 端点。在本系列文章中，我将介绍应用程序以及我使用（并将使用）使其在 Kubernetes 集群中的生产环境中运行的几个步骤。

## 2. 申请

webhooks.app是一个应用程序，其目的是提供永远在线的安全 webhook 端点（基本上是一个等待从 HTTP POST 请求接收 json 有效负载的服务器）。Webhook 是应用程序相互通信的一种方式，通常以 json 有效负载的形式交换信息。

让我们考虑以下示例：我们有一个[Harbor](https://goharbor.io/)容器注册表，并且希望每次将新图像推送到给定项目时都得到通知。Harbor 允许指定一个 webhook（url 和身份验证令牌），并且每次推送新图像时都会向该 webhook 发送一个 json 有效负载。

![在这里插入图片描述](https://img-blog.csdnimg.cn/27b2fb16b3384bf9a337bfcc71ef28f6.png)
![指定 webhook，以便每次将新图像推送到 api 项目时，Harbor 都会向该 webhook 发送有效负载](https://img-blog.csdnimg.cn/c3183b688f3744a7976247fb2b13e1e5.png)

 - 许多其他注册表允许使用 webhook 与外部系统进行通信（您也可以使用 [Docker Hub](https://hub.docker.com/) 存储库之一来使用它）
 - Harbor 还可以将有效负载发送到 webhook 端点以处理其他类型的事件（图像扫描的结果、超出配额……）

为了说明这一点，我们可以标记一个虚拟图像并将其发送到 Harbor 注册表：

```bash
$ docker login -u admin Harbor.techwhale.io 
$ docker image pull nginx:1.20 
$ docker image tag nginx:1.20 Harbor.techwhale.io/api/pong:2.0 
$ docker image pushharbor.techwhale.io/api/pong: 2.0
```
我们马上就可以在 `webhooks.app` 的仪表板中看到 Harbor 发送的负载内容：
![webhook 端点接收到的 json 有效负载的内容](https://img-blog.csdnimg.cn/aa32056582ff44ed8d41ee1a1687e412.png)
`webhooks.app` 仅允许您查看传入负载中的内容，并且不会以任何方式处理负载。这对于演示目的很有用，因此我们可以检查有效负载的内容，这可以是开发实际处理此有效负载的服务器之前的第一步。

##  4. 整体架构
该应用程序托管在 GitLab 的`web-hook` 组中。每个微服务都有自己的存储库：

 - web-hook：包含应用程序详细信息的元存储库，例如在本地运行它的方式以及您可以贡献的说明
 - www : 嗯……前端网络
 - api：操作数据并与底层数据库通信的微服务
 - ws : websocket 服务器，以确保每次应用程序接收到新的有效负载时实时更新前端

除了这些微服务之外，还使用了其他组件：

 - `NATS` 消息代理
 - `MongoDB`数据库
 - 应用前的 [Traefik](https://traefik.io/) 反向代理

下面的架构说明了应用程序的架构：

![在这里插入图片描述](https://img-blog.csdnimg.cn/ddfb073860634000b0da44985dba8a59.png)
基本上：

 - Traefik 将 api 和 web 前端暴露给外部世界
 - 每次将有效负载发送到 api 时，都会将其保存在数据库中并发布在 NATS 中的专用主题上
 - websocket 服务器收到来自 NATS 的消息并通过 `websocket` 将更新发送到浏览器

> 注意：架构中使用 `NATS` 来解耦 `api` 和 `ws` 组件。它允许设置 `Pub/Sub` 系统，`api`是 `NATS` 的发布者，而ws是NATS 的订阅者：api 发送的消息因此被 ws 接收。如果需要，以后可以轻松添加其他订阅者。

## 5. 每个微服务的简单 CI 设置
api、www和ws之间的每个存储库都包含一个 `.gitlab-ci.yml` 文件，该文件定义了在 `git push` 上触发的操作。例如，文件[https://gitlab.com/web-hook/api/-/blob/main/.gitlab-ci.yml](https://gitlab.com/web-hook/api/-/blob/main/.gitlab-ci.yml)定义了api的 CI（持续集成）管道。当新代码被推送到 api 存储库的主分支时，会发生以下操作：

 - 创建一个新标签（遵循 SemVer 格式）
 - 使用此标记构建和标记新容器映像
 - 使用此标记更新配置存储库（在以下专门介绍 GitOps 方法的文章中详细介绍）

 因此，CI 的主要结果是创建了一个新镜像，该镜像托管在微服务存储库的容器注册表中。

![已经为 api 微服务构建的容器镜像的历史](https://img-blog.csdnimg.cn/e21909bf573b4517aed0dd01d6b251b0.png)
## 6.  Docker Compose 本地运行它
在`web-hook` 存储库中，您可以找到在本地运行整个项目的所有说明。下面详细介绍如何使用 Docker Compose 在容器中运行应用程序。

首先将存储库克隆到一个新文件夹中。

```bash
mkdir webhooks && cd webhooks
for repo in www api ws web-hook; do
  git clone git@gitlab.com:web-hook/$repo.git
done
```
接下来使用 `web-hook` 存储库中的 `docker-compose.yml` 文件运行应用程序：

```bash
cd web-hook
docker compose up
```
几秒钟后，您可以访问将 Web 浏览器指向 `localhost` 的应用程序，然后单击底部的`Get my webhook`按钮获取您自己的 `Webhook` 端点。
![在这里插入图片描述](https://img-blog.csdnimg.cn/9cc3d73a53a84164bd4a6f9c3cf6b87f.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/0fc9f6ee62f14b5eadf4d88c85a1b9e9.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/4dad8ff9fbf34654af09d28db8fdf8fa.png)

> 以上图片是访问应用程序并获取您自己的 webhook 端点（url 和身份验证令牌）


让我们跳到仪表板：

![在这里插入图片描述](https://img-blog.csdnimg.cn/b4029e72dabc44a3afb4a2015d68e321.png)
从那里你有一个 curl 示例命令，你可以使用它来立即从命令行发送有效负载：

```bash
$ curl -XPOST -H "Authorization: Bearer 7227a2c2ebdbc267438e7ece2bdd0a" http://localhost/data -d '{"ok": "hello"}' -H 'Content-Type: application/json'
```
仪表板将使用有效负载的内容实时更新：

![在这里插入图片描述](https://img-blog.csdnimg.cn/a329c323c8bf47f8828f7a5df7002799.png)

翻译来源：

 - https://itnext.io/journey-of-a-microservice-application-in-the-kubernetes-world-bdfe795532ef

