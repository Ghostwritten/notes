#  grafana 安装
tags: grafana

![](https://i-blog.csdnimg.cn/blog_migrate/22f4851a28b092f1c34449beafee0ae6.png)






##  1. 条件
此页面列出了安装 Grafana 的最低硬件和软件要求。

要运行 Grafana，您必须拥有受支持的操作系统、满足或超过最低要求的硬件、受支持的数据库和受支持的浏览器。

###  1.1 支持的操作系统
Grafana 安装支持以下操作系统：

 - [Debian / Ubuntu](https://grafana.com/docs/grafana/latest/setup-grafana/installation/debian/)
 - [RPM-based Linux (CentOS, Fedora, OpenSuse, RedHat)](https://grafana.com/docs/grafana/latest/setup-grafana/installation/rpm/)
 - [macOS](https://grafana.com/docs/grafana/latest/setup-grafana/installation/mac/)
 - [Windows](https://grafana.com/docs/grafana/latest/setup-grafana/installation/windows/)

###  1.2 硬件建议
Grafana 不占用大量资源，在内存和 CPU 的使用上非常轻量级。

推荐的最低内存：255 MB 推荐的最低 CPU：1

某些功能可能需要更多内存或 CPU。需要更多资源的功能包括：

 - [Server side rendering of images](https://grafana.com/grafana/plugins/grafana-image-renderer/#requirements)
 - [Alerting](https://grafana.com/docs/grafana/latest/alerting/)
 - [Data source proxy](https://grafana.com/docs/grafana/latest/developers/http_api/data_source/)

###  1.3 支持的数据库
Grafana 需要一个数据库来存储其配置数据，例如用户、数据源和仪表板。具体要求取决于 Grafana 安装的大小和使用的功能。

Grafana 支持以下数据库：

 - [SQLite 3](https://www.sqlite.org/index.html)
 - [MySQL 5.7+](https://www.mysql.com/support/supportedplatforms/database.html)
 - [PostgreSQL 10+](https://www.postgresql.org/support/versioning/)

默认情况下，Grafana 安装并使用 SQLite，它是存储在 Grafana 安装位置的嵌入式数据库。

Grafana 将支持在 Grafana 版本发布时项目正式支持的这些数据库的版本。当某个版本不受支持时，Grafana 也可能会放弃对该版本的支持。有关每个项目的支持政策，请参阅上面的链接。

###  1.4 支持的网络浏览器
以下浏览器的当前版本支持 Grafana。这些浏览器的旧版本可能不受支持，因此在使用 Grafana 时应始终升级到最新版本。

 - Chrome/Chromium
 - Firefox
 - Safari
 - Microsoft Edge
 - Internet Explorer 11 is only fully supported in Grafana versions prior v6.0.

> 始终在您的浏览器中启用 JavaScript。不支持在浏览器中不启用 JavaScript 的情况下运行 Grafana。


##  2. Install on Debian or Ubuntu

###  2.1 APT 安装
如果您从 APT 存储库安装，那么 Grafana 会在您每次运行时自动更新`apt-get update`
| Grafana Version           | Package            | Repository                                              |
|---------------------------|--------------------|---------------------------------------------------------|
| Grafana Enterprise        | grafana-enterprise | https://packages.grafana.com/enterprise/deb stable main |
| Grafana Enterprise (Beta) | grafana-enterprise | https://packages.grafana.com/enterprise/deb beta main   |
| Grafana OSS               | grafana            | https://packages.grafana.com/oss/deb stable main        |
| Grafana OSS (Beta)        | grafana            | https://packages.grafana.com/oss/deb beta main          |

> 注意： Grafana Enterprise 是推荐的默认版本。它是免费提供的，包含 [OSS 版本的所有功能](https://grafana.com/products/enterprise/?utm_source=grafana-install-page)。您还可以升级到完整的 Enterprise 功能集，它支持[Enterprise 插件](https://grafana.com/grafana/plugins/?enterprise=1&utcm_source=grafana-install-page)。

### 2.2 安装最新的企业版

```bash
sudo apt-get install -y apt-transport-https
sudo apt-get install -y software-properties-common wget
sudo wget -q -O /usr/share/keyrings/grafana.key https://packages.grafana.com/gpg.key
```
为稳定版本添加此存储库：

```bash
echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://packages.grafana.com/enterprise/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```
如果您想要 beta 版本，请添加此存储库：

```bash
echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://packages.grafana.com/enterprise/deb beta main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```
添加存储库后：

```bash
sudo apt-get update
sudo apt-get install grafana-enterprise
```
###  2.3 安装最新的 OSS 版本

```bash
sudo apt-get install -y apt-transport-https
sudo apt-get install -y software-properties-common wget
sudo wget -q -O /usr/share/keyrings/grafana.key https://packages.grafana.com/gpg.key
```
为稳定版本添加此存储库：

```bash
echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

```
如果您想要 beta 版本，请添加此存储库：

```bash
echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://packages.grafana.com/oss/deb beta main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

```
添加存储库后：

```bash
sudo apt-get update
sudo apt-get install grafana
```

###  2.4 安装 .deb 包
如果你安装了这个`.deb`包，那么你需要为每个新版本手动更新 Grafana。
- 1. 在Grafana 下载页面，选择您要安装的 Grafana 版本。
   - 默认选择最新的 Grafana 版本。
   - 版本字段仅显示已完成的版本。如果要安装 beta 版本，请单击Nightly Builds，然后选择一个版本
 - 2. 选择一个版本。
    - 企业- 推荐下载。功能上与开源版本相同，但包含您可以选择使用许可证解锁的功能。
    - 开源- 功能与企业版相同，但如果您需要企业版功能，则需要下载企业版。
- 3. 根据您运行的系统，单击Linux或ARM。
- 4. 将安装页面中的代码复制并粘贴到命令行中并运行。它遵循如下所示的模式。

```bash
sudo apt-get install -y adduser
wget <.deb package url>
sudo dpkg -i grafana<edition>_<version>_amd64.deb
```

###  2.5 二进制 .tar.gz 文件安装
下载[最新.tar.gz文件](https://grafana.com/grafana/download?platform=linux)并解压。文件解压到以下载的 Grafana 版本命名的文件夹中。此文件夹包含运行 Grafana 所需的所有文件。此软件包中没有初始化脚本或安装脚本。

```bash
wget <tar.gz package url>
sudo tar -zxvf <tar.gz package>
```


###  2.6 使用 systemd 启动服务器
启动服务并验证服务是否已启动：

```bash
sudo systemctl daemon-reload
sudo systemctl start grafana-server
sudo systemctl status grafana-server
#重启
sudo systemctl restart grafana-server
```

将 Grafana 服务器配置为在引导时启动：

```bash
sudo systemctl enable grafana-server.service
```
### 2.7 在 < 1024 的端口上服务 Grafana
如果您正在使用systemd并希望在小于 `1024` 的端口上启动 Grafana，则必须添加systemd单位覆盖。

以下命令在您配置的编辑器中创建一个覆盖文件：

```bash
# Alternatively, create a file in /etc/systemd/system/grafana-server.service.d/override.conf
systemctl edit grafana-server.service
```
 添加这些附加设置以授予该CAP_NET_BIND_SERVICE功能。要阅读有关功能的更多信息，请参阅[有关功能的手册页](https://man7.org/linux/man-pages/man7/capabilities.7.html)。
 

```bash
[Service]
# Give the CAP_NET_BIND_SERVICE capability
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_BIND_SERVICE

# A private user cannot have process capabilities on the host's user
# namespace and thus CAP_NET_BIND_SERVICE has no effect.
PrivateUsers=false
```
###  2.8 使用 init.d 启动服务器
启动服务并验证服务是否已启动：

```bash
sudo service grafana-server start
sudo service grafana-server status

#重启
sudo service grafana-server restart
```

将 Grafana 服务器配置为在引导时启动：

```bash
sudo update-rc.d grafana-server defaults
```
###  2.9 执行二进制
`grafana-server`二进制 `.tar.gz` 需要工作目录是二进制和public文件夹所在的根安装目录。

通过运行启动 Grafana：

```bash
./bin/grafana-server web
```

 - 将二进制安装到`/usr/sbin/grafana-server`

- 将 Init.d 脚本安装到`/etc/init.d/grafana-server`
- 创建默认文件（环境变量）以`/etc/default/grafana-server`
- 将配置文件安装到`/etc/grafana/grafana.ini`
- 安装 systemd 服务（如果 systemd 可用）名称`grafana-server.service`
- 默认配置将日志文件设置为`/var/log/grafana/grafana.log`
- 默认配置指定一个 SQLite3 数据库在`/var/lib/grafana/grafana.db`
- 将 HTML/JS/CSS 和其他 Grafana 文件安装在`/usr/share/grafana`

##  3. docker 安装
您可以使用官方 Docker 镜像安装和运行 Grafana。我们的 docker 镜像有两个版本：

 - Grafana Enterprise: `grafana/grafana-enterprise`
 - Grafana Open Source: `grafana/grafana-oss`

每个版本都有两种变体：`Alpine` 和 `Ubuntu`。见下文。

有关配置 docker 镜像的文档，请参阅[配置 Grafana Docker 镜像](https://grafana.com/docs/grafana/v9.0/setup-grafana/configure-docker/)。

本主题还包含有关[从早期 Docker 映像版本迁移](https://grafana.com/docs/grafana/latest/setup-grafana/installation/docker/#migrate-from-previous-docker-containers-versions)的重要信息。

###  3.1 Alpine image (recommended)

 - Grafana Enterprise edition: `grafana/grafana-enterprise:<version>`
 - Grafana Open Source edition: `grafana/grafana-oss:<version>`

默认镜像基于流行的[Alpine Linux](http://alpinelinux.org/) 项目，可在[Alpine 官方镜像](https://hub.docker.com/_/alpine)中找到。Alpine Linux 比大多数发行版基础镜像要小得多，因此可以生成更苗条和更安全的镜像。

当需要尽可能小的安全性和最终图像大小时，强烈建议使用 `Alpine` 变体。需要注意的主要警告是它使用musl libc而不是glibc 和朋友，因此某些软件可能会遇到问题，具体取决于其 libc 要求的深度。但是，大多数软件对此没有问题，因此此变体通常是一个非常安全的选择

###  3.2 Ubuntu image

 - Grafana Enterprise edition: `grafana/grafana-enterprise:<version>-ubuntu`
 - Grafana Open Source edition: `grafana/grafana-oss:<version>-ubuntu`

这些镜像基于[Ubuntu](https://ubuntu.com/)，可在[Ubuntu 官方镜像](https://hub.docker.com/_/ubuntu)中找到。对于那些喜欢基于Ubuntu的映像和/或依赖于某些 Alpine 不可用的工具的人来说，它是一个替代映像。

###  3.3 运行 Grafana
您可以运行最新的 Grafana 版本、运行特定版本或基于[grafana/grafana GitHub 存储库](https://github.com/grafana/grafana)的主分支运行不稳定版本。

运行最新稳定版 Grafana

```bash
docker run -d -p 3000:3000 grafana/grafana-enterprise
```
运行特定版本的 Grafana

```bash
docker run -d -p 3000:3000 --name grafana grafana/grafana-enterprise:<version number>
```
例子：

```bash
docker run -d -p 3000:3000 --name grafana grafana/grafana-enterprise:8.2.0
```

###  3.4 容器中安装插件
您可以安装 [Grafana插件](https://grafana.com/grafana/plugins/)页面或自定义 URL 中列出的官方和社区插件。

#### 3.4.1 安装官方和社区 Grafana 插件
将您要安装的插件与`GF_INSTALL_PLUGINS`环境变量作为逗号分隔列表传递给 Docker。`grafana-cli plugins install ${plugin}`这会在 Grafana 启动时将每个插件名称发送到并安装它们。

```bash
docker run -d \
  -p 3000:3000 \
  --name=grafana \
  -e "GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-simple-json-datasource" \
  grafana/grafana-enterprise
```

> 注意：如果您需要指定插件的版本，则可以将其添加到GF_INSTALL_PLUGINS环境变量中。否则，使用最新的。例如：-e "GF_INSTALL_PLUGINS=grafana-clock-panel
1.0.1,grafana-simple-json-datasource 1.3.5"。

####  3.4.2 其他来源安装插件

> 仅在 `Grafana v5.3.1` 及更高版本中可用。

```bash
docker run -d \
  -p 3000:3000 \
  --name=grafana \
  -e "GF_INSTALL_PLUGINS=http://plugin-domain.com/my-custom-plugin.zip;custom-plugin,grafana-clock-panel" \
  grafana/grafana-enterprise
```

####  3.4.3 使用预安装的插件构建和运行 Docker 映像
在[Grafana GitHub 存储库](https://github.com/grafana/grafana)中有一个名为 的文件夹`packaging/docker/custom/`，其中包含两个 Dockerfile`Dockerfile`和`ubuntu.Dockerfile`，可用于构建自定义 Grafana 镜像。它接受`GRAFANA_VERSION`、`GF_INSTALL_PLUGINS`和`GF_INSTALL_IMAGE_RENDERER_PLUGIN`作为构建参数。

```bash
cd packaging/docker/custom
docker build \
  --build-arg "GRAFANA_VERSION=latest" \
  --build-arg "GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-simple-json-datasource" \
  -t grafana-custom -f Dockerfile .

docker run -d -p 3000:3000 --name=grafana grafana-custom
```
####  3.4.4 使用来自其他来源的预安装插件构建

```bash
cd packaging/docker/custom
docker build \
  --build-arg "GRAFANA_VERSION=latest" \
  --build-arg "GF_INSTALL_PLUGINS=http://plugin-domain.com/my-custom-plugin.zip;custom-plugin,grafana-clock-panel" \
  -t grafana-custom -f Dockerfile .

docker run -d -p 3000:3000 --name=grafana grafana-custom
```
将Dockerfile上面的示例替换`ubuntu.Dockerfile`为构建基于 Ubuntu 的自定义映像（Grafana v6.5+）。


###  3.5 重要变化
#### 3.5.1 用户 ID 更改

```bash
Version	User	User ID	Group	Group ID
< 5.1	grafana	104	grafana	107
>= 5.1	grafana	472	grafana	472
>= 7.3	grafana	472	root	0
```
以其他用户身份运行 Docker

```bash
docker run --user 104 --volume "<your volume mapping here>" grafana/grafana-enterprise:8.2.0
```
在 `docker-compose.yml` 中指定用户

```bash
version: '2'

services:
  grafana:
    image: grafana/grafana-enterprise:8.2.0
    ports:
      - 3000:3000
    user: '104'
```
####  3.5.2 修改权限
下面的命令在 Grafana 容器内运行 bash，并映射了您的卷。这使得修改文件所有权以匹配新容器成为可能。修改权限时请务必小心。

```bash
$ docker run -ti --user root --volume "<your volume mapping here>" --entrypoint bash grafana/grafana-enterprise:8.2.0
```

```bash
# in the container you just started:
chown -R root:root /etc/grafana && \
chmod -R a+r /etc/grafana && \
chown -R grafana:grafana /var/lib/grafana && \
chown -R grafana:grafana /usr/share/grafana
```
配置 Docker 镜像
有关自定义环境、日志记录、数据库等选项的详细信息，请参阅[配置 Grafana Docker](https://grafana.com/docs/grafana/latest/setup-grafana/configure-docker/) 镜像页面。

###  3.6 重启

```bash
docker restart grafana
```

> 更多关于[容器部署 grafana 的细节请参考](https://ghostwritten.blog.csdn.net/article/details/107867172)

##  4. Deploy Grafana on Kubernetes

```bash
vim  grafana.yaml
```

```bash
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: grafana-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: grafana
  name: grafana
spec:
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      securityContext:
        fsGroup: 472
        supplementalGroups:
          - 0
      containers:
        - name: grafana
          image: grafana/grafana:8.4.4
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 3000
              name: http-grafana
              protocol: TCP
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /robots.txt
              port: 3000
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 30
            successThreshold: 1
            timeoutSeconds: 2
          livenessProbe:
            failureThreshold: 3
            initialDelaySeconds: 30
            periodSeconds: 10
            successThreshold: 1
            tcpSocket:
              port: 3000
            timeoutSeconds: 1
          resources:
            requests:
              cpu: 250m
              memory: 750Mi
          volumeMounts:
            - mountPath: /var/lib/grafana
              name: grafana-pv
      volumes:
        - name: grafana-pv
          persistentVolumeClaim:
            claimName: grafana-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: grafana
spec:
  ports:
    - port: 3000
      protocol: TCP
      targetPort: http-grafana
  selector:
    app: grafana
  sessionAffinity: None
  type: LoadBalancer
```
执行：

```bash
kubectl apply -f grafana.yaml
kubectl port-forward service/grafana 3000:3000
```
`localhost:3000`在浏览器中导航到。您应该会看到一个 Grafana 登录页面。

用于`admin`登录的用户名和密码。


##  5. Install on RPM-based Linux (CentOS, Fedora, OpenSuse, Red Hat)
###  5.1 Install from YUM repository
如果您从 YUM 存储库安装，那么 Grafana 会在您每次运行时自动更新`sudo yum update`
| Grafana Version           | Package            | Repository                                       |
|---------------------------|--------------------|--------------------------------------------------|
| Grafana Enterprise        | grafana-enterprise | https://packages.grafana.com/enterprise/rpm      |
| Grafana Enterprise (Beta) | grafana-enterprise | https://packages.grafana.com/enterprise/rpm-beta |
| Grafana OSS               | grafana            | https://packages.grafana.com/oss/rpm             |
| Grafana OSS (Beta)        | grafana            | https://packages.grafana.com/oss/rpm-beta        |

> 注意： Grafana Enterprise 是推荐的默认版本。它是免费提供的，包含 OSS 版的所有功能。您还可以升级到完整的[Enterprise 功能集](https://grafana.com/products/enterprise/?utm_source=grafana-install-page)并支持[Enterprise 插件](https://grafana.com/grafana/plugins/?enterprise=1&utcm_source=grafana-install-page)。


使用您选择的方法将新文件添加到您的 YUM 存储库。下面的命令使用nano.

```bash
sudo nano /etc/yum.repos.d/grafana.repo
```

选择是否要安装 Grafana 的开源版或企业版，然后将所选版本中的信息输入到`grafana.repo`. 如果要安装 Grafana 的 beta 版本，则需要将 URL 替换为上表中的 beta URL。

对于企业版：

```bash
[grafana]
name=grafana
baseurl=https://packages.grafana.com/enterprise/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
```
对于 OSS 版本：

```bash
[grafana]
name=grafana
baseurl=https://packages.grafana.com/oss/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
```
使用以下命令之一安装 Grafana：

```bash
sudo yum install grafana

# or

sudo yum install grafana-enterprise
```

###   5.2 YUM 手动安装
在[Grafana 下载页面](https://grafana.com/grafana/download)，选择您要安装的 Grafana 版本。
默认选择最新的 Grafana 版本。
版本字段仅显示已完成的版本。如果要安装 beta 版本，请单击Nightly Builds，然后选择一个版本。

```bash
wget <rpm package url>
sudo yum localinstall <local rpm package>
你也可以直接使用 YUM 安装 Grafana：

sudo yum install <rpm package url>
```

###  5.3 RPM 安装
注意： `.rpm` 文件已签名，您可以使用此[GPG 公钥验证](https://packages.grafana.com/gpg.key)签名。

在[Grafana 下载页面](https://grafana.com/grafana/download)，选择您要安装的 Grafana 版本。

在 CentOS、Fedora、Red Hat 或 RHEL 上：
```bash
sudo yum install initscripts urw-fonts wget
wget <rpm package url>
sudo rpm -Uvh <local rpm package>
```
在 OpenSUSE 或 SUSE 上：
```bash
wget <rpm package url>
sudo rpm -i --nodeps <local rpm package>
```
###  5.4 二进制 .tar.gz 文件安装
下载[最新.tar.gz文件](https://grafana.com/grafana/download?platform=linux)并解压。这些文件被提取到以您下载的 Grafana 版本命名的文件夹中。此文件夹包含运行 Grafana 所需的所有文件。此软件包中没有初始化脚本或安装脚本。

```bash
wget <tar.gz package url>
sudo tar -zxvf <tar.gz package>
```
###  5.5 启动服务器
如果您使用`.rpm`软件包安装，则可以使用`systemd`或启动服务器`init.d`。如果您安装了二进制`.tar.gz`文件，则需要执行该二进制文件。

```bash
sudo systemctl daemon-reload
sudo systemctl start grafana-server
sudo systemctl status grafana-server
sudo systemctl enable grafana-server
```
###  5.6 使用 init.d 启动服务器
启动服务并验证服务是否已启动：

```bash
sudo service grafana-server start
sudo service grafana-server status
```

将 Grafana 服务器配置为在引导时启动：

```bash
sudo /sbin/chkconfig --add grafana-server
```

### 5.7 执行二进制
grafana-server二进制文件需要工作目录是二进制文件和public文件夹所在的根安装目录。

通过运行启动 Grafana：

```bash
./bin/grafana-server web
```
- 将二进制安装到`/usr/sbin/grafana-server`
- 将 init.d 脚本复制到`/etc/init.d/grafana-server`
- 将默认文件（环境变量）安装到`/etc/sysconfig/grafana-server`
- 将配置文件复制到`/etc/grafana/grafana.ini`
- 安装 systemd 服务（如果 systemd 可用）名称grafana-server.service
- 默认配置使用日志文件`/var/log/grafana/grafana.log`
- 默认配置指定一个 sqlite3 数据库在`/var/lib/grafana/grafana.db`


##  6. helm charts 安装 grafana
您更喜欢 Helm，请参阅[Grafana Helm 社区图表](https://github.com/grafana/helm-charts)。

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm search repo grafana
```

##  7. docker-compose 安装 grafana
略

参考：

 - [Install Grafana](https://grafana.com/docs/grafana/latest/setup-grafana/installation/)

