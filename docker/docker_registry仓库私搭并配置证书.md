#  docker  registry 私搭
![](https://i-blog.csdnimg.cn/blog_migrate/6d09a7922cf3f34f450e3af51b5423e8.png)




<font color=#FFA500 size=3 face="楷体">"大家好，我是幽灵代笔，本篇文章主要归纳了关于 `docker registry`  的搭建内容，通过多种方式的部署比较对比让大家更能融会贯通，从中了解`docker`更多姿势玩法"</font>


## 1. 什么是 docker-registry ?

[Registry](https://docs.docker.com/registry/) 是 [docker](https://www.aquasec.com/cloud-native-academy/docker-container/docker-containers-vs-virtual-machines/) 官方提供的工具，可以用于构建私有的镜像仓库。

`Registry` 是一个无状态的、高度可扩展的服务器端应用程序，它存储并允许您分发 Docker 映像。Registry 是开源的，在宽松的Apache 许可下。您可以在[GitHub 上找到源代码](https://github.com/distribution/distribution) 。

`Docker registry` 被组织成 [Docker 镜像存储库](https://www.aquasec.com/cloud-native-academy/container-security/image-repository/) ，其中存储库包含特定镜像的所有版本。仓库允许 Docker 用户在本地拉取镜像，以及将新镜像推送到仓库（在适用时给予足够的访问权限）。

默认情况下，[Docker  engine](https://docs.docker.com/engine/)与 [Docker Hub](https://hub.docker.com/) 交互，Docker 的公共仓库实例。但是，可以在本地运行开源 [Docker registry](https://docs.docker.com/registry/) 。

接下来，我们开始在本地环境部署 `docker-registry`。

## 预备条件

- [设置 docker 存储目录](https://blog.csdn.net/xixihahalelehehe/article/details/134633286)
- [安装 docker](https://blog.csdn.net/xixihahalelehehe/article/details/104293170)


## 4. 简单部署私有仓库

**初级**：快速部署简单的私有 registry sever

 - `regsitry server1`：192.168.211.15
 - `client1`：192.168.211.16

### 4.1 拉取 registry 镜像
```bash
docker pull registry
```

报错`net/http: TLS handshake timeout`

修改docker配置,使用国内镜像 daocloud镜像加速器

```bash
$ vim /etc/docker/daemon.json
{"registry-mirrors": ["http://d1d9aef0.m.daocloud.io"]}

$ systemctl restart docker
$ docker pull registry
```
### 4.2 创建私有仓库
- `REGISTRY_COMPATIBILITY_SCHEMA1_ENABLED=true`：可以通过接口API进行调用，查看、删除、拉取、推送。

```bash
docker run -tid --restart=always --name registry -p 5000:5000 -e REGISTRY_COMPATIBILITY_SCHEMA1_ENABLED=true -v /storage/registry:/var/lib/registry registry:latest
```

### 4.3 测试镜像推入私有仓库

```bash
$ docker pull centos
$ docker tag  centos:latest 192.168.211.15:5000/centos:latest
$ docker push 192.168.211.15:5000/centos:latest
The push refers to a repository [192.168.211.15:5000/centos]
Get https://192.168.211.15:5000/v1/_ping: http: server gave HTTP response to HTTPS client
```

在推送镜像中出现错误，因为`client`与`Registry`交互默认将采用https访问，但我们在安装 Registry 时并未配置指定任何`tls`相关的`key`和`crt`文件，https将无法访问。因此， 我们需要配置客户端的`Insecure Registry`选项（另一种解决方案需要配置Registry的证书）。

```bash
$ docker stop registry
$ vim /etc/sysconfig/docker
OPTIONS='--selinux-enabled --log-driver=journald --signature-verification=false --insecure-registry 192.168.211.15:5000'

或者
$ systemctl daemon-reload && systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
   Active: active (running) since 四 2022-10-20 18:15:48 CST; 3h 27min ago

$ vim /usr/lib/systemd/system/docker.service
.........
ExecStart=/usr/bin/dockerd -H fd:// --insecure-registry 192.168.211.15:5000 
...........
#或者创建
$ vim /etc/docker/daemon.json 
{
  "live-restore": true,
  "dns": ["8.8.8.8"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size":  "100m",
    "max-file": "5"
   },
  "insecure-registries": ["192.168.211.15:5000"]
}

#重启docker并启动registry
$ systemctl restart docker
$ docker start registry
#测试访问是否正常
$ curl -I -X GET localhost:5000/v2/
$ docker push 192.168.211.15:5000/centos:latest
$ docker rmi centos

#获取镜像列表
$ curl  https://192.168.211.15:5000/v2/_catalog
{"repositories":["centos"]}   

#拉取仓库中centos镜像至本地
$ docker pull 192.168.211.15:5000/centos:latest 
```

###  4.5 清理

```bash
$ docker stop registry  && docker rm registry 
$ rm -rf /storage/registry
```

##  5. docker 部署具有权限认证、TLS 的私有仓库 
**中级**：搭建一个拥有权限认证、TLS 的私有仓库。

 - `regsitry server1`：192.168.211.15
 - `client1`：192.168.211.16
### 5.1 配置签名证书
在Docker Registry主机中生成OpenSSL的自签名证书：

```bash
cat << EOF > ssl.conf
[ req ]
prompt             = no
distinguished_name = req_subj
x509_extensions    = x509_ext

[ req_subj ]
CN = Localhost

[ x509_ext ]
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid,issuer
basicConstraints       = CA:true
subjectAltName         = @alternate_names

[ alternate_names ]
DNS.1 = localhost
IP.1  = 192.168.211.15
EOF
```

```bash
$ openssl req -config ssl.conf -new -x509 -nodes -sha256 -days 365 -newkey rsa:4096 -keyout /certs/server-key.pem -out /certs/server-crt.pem
```


Docker Registry所在本机操作：
证书生成好了，客户端现在就不需要 `--insecure-registry` 了 

```bash
$ vim /etc/sysconfig/docker
OPTIONS='--selinux-enabled --log-driver=journald --signature-verification=false'
或者
$ systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
   Active: active (running) since 四 2022-10-20 18:15:48 CST; 3h 27min ago

$ vim /usr/lib/systemd/system/docker.service
.........
ExecStart=/usr/bin/dockerd -H fd:// --insecure-registry 192.168.211.15:5000 
...........


$ mkdir -p  /etc/docker/certs.d/192.168.211.15:5000/
$ cp /certs/server-crt.pem /etc/docker/certs.d/192.168.211.15\:5000/
$ systemctl daemon-reload && systemctl status docker
```

客户端操作：

```bash
$ mkdir -p  /etc/docker/certs.d/192.168.211.15:5000/
$ scp /certs/server-crt.pem root@192.168.211.16:/etc/docker/certs.d/192.168.211.15:5000/
$ systemctl restart docker
```
### 5.2 配置用户认证
为了相对安全，可以给仓库加上基本的身份认证。使用 [htpasswd](https://httpd.apache.org/docs/current/programs/htpasswd.html) 创建用户和登陆密码：

```bash
$ htpasswd -Bbn testuser testpassword > /auth/htpasswd
$ cat /auth/htpasswd
testuser:$2y$05$MO4iv425uurfqY2Y/X71TuNTUPu4Vrn.oNE4NxRTjsPTTU6QywiwG
```
或者借用镜像命令创建用户

```bash
$ sudo sh -c "docker run --entrypoint htpasswd registry:2.3.0 -Bbn testuser testpassword > /auth/htpasswd"
or
$ docker run --entrypoint htpasswd registry:2.3.0 -Bbn testuser testpassword > auth/htpasswd
```


### 5.3 创建 registry server

 - 包含签名证书 ，本地目录`/certs`
 - 包含持久存储，本地目录`/var/lib/registry`
 - 用户密码登录，本地目录`/auth`

```bash
docker run -d \
    -p 5000:5000 \
    --name registry \
    --restart=always \
    -v /var/lib/registry:/var/lib/registry \
    -v /auth:/auth \
    -v /certs:/certs \
    -e REGISTRY_COMPATIBILITY_SCHEMA1_ENABLED=true \
    -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/server-crt.pem \
    -e REGISTRY_HTTP_TLS_KEY=/certs/server-key.pem \
    -e REGISTRY_AUTH=htpasswd \
    -e REGISTRY_AUTH_HTPASSWD_REALM="Registry Realm" \
    -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
    registry:2.3.0
```

### 5.4 测试
测试登陆
```bash
$ docker login -u testuser -p testpassword 192.168.211.15:5000
Login Succeeded
```
测试 `push`

```bash
docker push 192.168.211.15:5000/centos:latest
```
在另一台机器`192.168.211.16`，测试 `pull`

```bash
$ mkdir -p  /etc/docker/certs.d/192.168.211.15:5000/
$ cp /certs/server-crt.pem /etc/docker/certs.d/192.168.211.15\:5000/
$ vim /etc/sysconfig/docker
OPTIONS='--selinux-enabled --log-driver=journald --signature-verification=false --insecure-registry 192.168.211.15:5000'
$ systemctl daemon-reload && systemctl restart docker
$ docker pull 192.168.211.15:5000/centos:latest
```
###  5.5 开启删除权限
查询删除权限

```bash
$ docker exec -it  registry sh -c 'cat /etc/docker/registry/config.yml'
```
```bash
version: 0.1
log:
  fields:
    service: registry
storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
```
开启删除权限

```bash
docker exec -it  registry sh -c "sed -i '/storage:/a\  delete:' /etc/docker/registry/config.yml"
docker exec -it  registry sh -c "sed -i '/delete:/a\    enabled: true' /etc/docker/registry/config.yml"
```
重启镜像

```bash
docker restart registry
```
删除镜像

```bash
$ curl -I -X DELETE "localhost/v2/hello-world/manifests/sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e899a"
#or
curl -I --header "Accept: application/vnd.docker.distribution.manifest.v2+json" -X DELETE localhost:5000/v2/<name>/manifests/<reference>
```

##  6. docker-compose 部署具有权限认证、TLS 的私有仓库 
### 6.1 准备站点证书
如果你拥有一个域名，国内各大云服务商均提供免费的站点证书。你也可以使用 `openssl` 自行签发证书。
这里假设我们将要搭建的私有仓库地址为 `docker.domain.com`，下面我们介绍使用 `openssl` 自行签发 `docker.domain.com` 的站点 `SSL` 证书。

第一步创建 CA 私钥。

```bash
$ openssl genrsa -out "root-ca.key" 4096
```

第二步利用私钥创建 CA 根证书请求文件

```bash
$ openssl req \
          -new -key "root-ca.key" \
          -out "root-ca.csr" -sha256 \
          -subj '/C=CN/ST=Shanxi/L=Datong/O=Your Company Name/CN=Your Company Name Docker Registry CA'
```

以上命令中 `-subj` 参数里的 `/C` 表示国家，如 `CN`；`/ST` 表示省`；/L` 表示城市或者地区；`/O` 表示组织名；`/CN` 通用名称。

第三步配置 `CA` 根证书，新建 `root-ca.cnf`。

```bash
[root_ca]
basicConstraints = critical,CA:TRUE,pathlen:1
keyUsage = critical, nonRepudiation, cRLSign, keyCertSign
subjectKeyIdentifier=hash
```
第四步签发根证书。

```bash
$ openssl x509 -req  -days 3650  -in "root-ca.csr" \
               -signkey "root-ca.key" -sha256 -out "root-ca.crt" \
               -extfile "root-ca.cnf" -extensions \
               root_ca
```
第五步生成站点 SSL 私钥。

```bash
$ openssl genrsa -out "docker.domain.com.key" 4096
```
第六步使用私钥生成证书请求文件。

```bash
$ openssl req -new -key "docker.domain.com.key" -out "site.csr" -sha256 \
          -subj '/C=CN/ST=Shanxi/L=Datong/O=Your Company Name/CN=docker.domain.com'
```
第七步配置证书，新建 `site.cnf` 文件。

```bash
[server]
authorityKeyIdentifier=keyid,issuer
basicConstraints = critical,CA:FALSE
extendedKeyUsage=serverAuth
keyUsage = critical, digitalSignature, keyEncipherment
subjectAltName = DNS:docker.domain.com, IP:127.0.0.1
subjectKeyIdentifier=hash
```
第八步签署站点 SSL 证书。

```bash
$ openssl x509 -req -days 750 -in "site.csr" -sha256 \
    -CA "root-ca.crt" -CAkey "root-ca.key"  -CAcreateserial \
    -out "docker.domain.com.crt" -extfile "site.cnf" -extensions server
```

这样已经拥有了 `docker.domain.com` 的网站 `SSL` 私钥 `docker.domain.com.key` 和 `SSL` 证书 `docker.domain.com.crt` 及 CA 根证书 `root-ca.crt`。
新建 `ssl` 文件夹并将 `docker.domain.com.key` `docker.domain.com.crt root-ca.crt` 这三个文件移入，删除其他文件。

### 6.2 配置私有仓库
私有仓库默认的配置文件位于 `/etc/docker/registry/config.yml`，我们先在本地编辑 `config.yml`，之后挂载到容器中。

```bash
version: 0.1
log:
  accesslog:
    disabled: true
  level: debug
  formatter: text
  fields:
    service: registry
    environment: staging
storage:
  delete:
    enabled: true
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
auth:
  htpasswd:
    realm: basic-realm
    path: /etc/docker/registry/auth/nginx.htpasswd
http:
  addr: :443
  host: https://docker.domain.com
  headers:
    X-Content-Type-Options: [nosniff]
  http2:
    disabled: false
  tls:
    certificate: /etc/docker/registry/ssl/docker.domain.com.crt
    key: /etc/docker/registry/ssl/docker.domain.com.key
health:
  storagedriver:
    enabled: true
    interval: 10s
threshold: 3
```

### 6.3 生成 http 认证文件

```bash
$ mkdir auth

$ docker run --rm \
    --entrypoint htpasswd \
    httpd:alpine \
    -Bbn username password > auth/nginx.htpasswd
```

> 将上面的 `username` `password` 替换为你自己的用户名和密码。

### 6.4 编辑 docker-compose.yml文件

```bash
version: '3'

services:
  registry:
    image: registry
    ports:
      - "443:443"
    volumes:
      - ./:/etc/docker/registry
      - registry-data:/var/lib/registry

volumes:
  registry-data:
```
### 6.5 修改 hosts

```bash
127.0.0.1 docker.domain.com
```

### 6.6 启动

```bash
$ docker-compose up -d
```

这样我们就搭建好了一个具有权限认证、TLS 的私有仓库，接下来我们测试其功能是否正常。

### 6.7 测试
由于自行签发的 CA 根证书不被系统信任，所以我们需要将 CA 根证书 `ssl/root-ca.crt` 移入 `/etc/docker/certs.d/docker.domain.com` 文件夹中。

```bash
$ sudo mkdir -p /etc/docker/certs.d/docker.domain.com

$ sudo cp ssl/root-ca.crt /etc/docker/certs.d/docker.domain.com/ca.crt
```

登录到私有仓库

```bash
$ docker login docker.domain.com
```
尝试推送、拉取镜像。

```bash
$ docker pull ubuntu:18.04

$ docker tag ubuntu:18.04 docker.domain.com/username/ubuntu:18.04

$ docker push docker.domain.com/username/ubuntu:18.04

$ docker image rm docker.domain.com/username/ubuntu:18.04

$ docker pull docker.domain.com/username/ubuntu:18.04
```

如果我们退出登录，尝试推送镜像。

```bash
$ docker logout docker.domain.com

$ docker push docker.domain.com/username/ubuntu:18.04

no basic auth credentials
```


>注意事项：如果你本机占用了 443 端口，你可以配置  [Nginx 代理](https://docs.docker.com/registry/recipes/nginx/)


## 7. docker-compose 部署 Token 认证的私有仓库

 
### 7.1 创建存储、证书、配置文件目录

```bash
mkdir -p /data/volumes/{auth_server/{config,ssl},docker_registry/data}
```
### 7.2 配置签名证书
证书可以去认证机构购买签名证书，此处我们使用`openssl`生成证书。
一般情况下，证书只支持域名访问，要使其支持IP地址访问，需要修改配置文件openssl配置文件，再进行证书生成。
在CentOS7系统中，文件存储位置为`/etc/pki/tls/openssl.cnf`。在其中的`[ v3_ca]`部分，添加`subjectAltName`选项(填入操作机实际IP)：

```bash
[ v3_ca ]
subjectAltName = IP:192.168.211.15
```

```bash
$ cd /data/volume/auth_server/ssl
$ openssl req -config ssl.conf -new -x509 -nodes -sha256 -days 365 -newkey rsa:4096 -keyout server.key -out server.pem
```



生成时，可以所有操作直接回车，不填写任何信息生成。或在Common Name 里填入仓库将使用IP和端口”`10.20.26.52:5000`”。

```bash
$ vim /etc/sysconfig/docker
OPTIONS='--selinux-enabled --log-driver=journald --signature-verification=false --insecure-registry 192.168.211.15:5000'
$ mkdir -p  /etc/docker/certs.d/192.168.211.15\:5000/
$ cp /data/volumes/auth_server/ssl/server.pem /etc/docker/certs.d/192.168.211.15\:5000/
$ systemctl daemon-reload && systemctl restart docker
```

客户端操作：

```bash
$ vim /etc/sysconfig/docker
OPTIONS='--selinux-enabled --log-driver=journald --signature-verification=false --insecure-registry 192.168.211.15:5000'
$ mkdir -p  /etc/docker/certs.d/192.168.211.15:5000/
$ scp /certs/server-crt.pem root@192.168.211.16:/etc/docker/certs.d/192.168.211.15:5000/
$ systemctl daemon-reload && systemctl restart docker
```

### 7.3 创建登陆用户
`htpasswd` 命令在`httpd`包中

```bash
$ yum -y install  httpd
$ htpasswd -Bbn admin adminpassword
admin:$2y$05$D5vyvbsZfuLKRX3PdDIyx.tx8kFOBKFlpyoBZ.O16Ovu25bI0IHf2

$ htpasswd -Bbn reader readerpassword
reader:$2y$05$QLtmKINhomZ2iDdWqfXEW.MRGjeY7mJJIyjFgfywhGfy2lOk6gFQO
```

 
### 7.4 配置认证服务的配置文件
在目录（`/data/volumes/auth_server/config`）下创建配置文件（`auth_config.yml`）

```bash
server:  # Server settings.
 
  # Address to listen on.
 
  addr: ":5001"
 
  # TLS certificate and key.
 
  certificate: "/ssl/server.pem"
 
  key: "/ssl/server.key"
 
 
 
token:  # Settings for the tokens.
 
  issuer: "Auth Service"  # Must match issuer in the Registry config.
 
  expiration: 900
 
 
 
# Static user map. 
 
users:
 
  # Password is specified as a BCrypt hash. Use htpasswd -B to generate.
 
  "admin":
 
    password: "$2y$05$D5vyvbsZfuLKRX3PdDIyx.tx8kFOBKFlpyoBZ.O16Ovu25bI0IHf2"
 
  "reader":
 
    password: "$2y$05$QLtmKINhomZ2iDdWqfXEW.MRGjeY7mJJIyjFgfywhGfy2lOk6gFQO"
 
  "": {}  # Allow anonymous (no "docker login") access.
 
 
 
acl:
 
  # Admin has full access to everything.
 
  - match: {account: "admin"}
 
    actions: ["*"]
 
  - match: {account: "reader", name: "nginx"}
 
    actions: ["pull"]

```
### 7.5 搭建 registry 和 auth 服务
采用compose模式搭建，创建compose文件（`registry-auth.yml`）

```bash
 
dockerauth:
  image: cesanta/docker_auth:stable
  container_name: docker_auth
  ports:
    - "5001:5001"
  volumes:
    - /data/volumes/auth_server/config:/config:ro
    - /var/log/docker_auth:/logs
    - /data/volumes/auth_server/ssl:/ssl
  command: /config/auth_config.yml
  restart: always
  
registry:
  image: registry:2
  container_name: docker_registry
  ports: 
    - "5000:5000" 
  volumes: 
    - /data/volumes/auth_server/ssl:/ssl
    - /data/volumes/docker_registry/data:/var/lib/registry
  restart: always
  environment:
    - REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY=/var/lib/registry
    - REGISTRY_AUTH=token
    - REGISTRY_AUTH_TOKEN_REALM=https://192.168.211.15:5001/auth
    - REGISTRY_AUTH_TOKEN_SERVICE="Docker registry"
    - REGISTRY_AUTH_TOKEN_ISSUER="Auth Service"
    - REGISTRY_AUTH_TOKEN_ROOTCERTBUNDLE=/ssl/server.pem
    - REGISTRY_HTTP_TLS_CERTIFICATE=/ssl/server.pem
    - REGISTRY_HTTP_TLS_KEY=/ssl/server.key
```
执行：

```bash
#安装docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
部署
docker-compose -f registry-auth.yml up -d
```
### 7.6 测试
登陆

```bash
$ docker login -uadmin -padminpassword 192.168.211.15:5000
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```



##  8. docker registry API 使用
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/aebd46b54ab7ca8bec10bf9a7f474262.png)

```bash
#无用户密码
$ curl -k  http://192.168.21.183:5000/v2/registry/tags/list
{"name":"registry","tags":["latest"]}

#查看API是否可用，返回200 OK代表可用
curl -I -X GET localhost:5000/v2/

#查看所有镜像
$ curl  -k -u "testuser:testpassword" https://192.168.211.15:5000/v2/_catalog
{"repositories":[]}

#推送镜像入库
$ docker push 192.168.211.15:5000/centos:latest

#查看镜像是否存在
$ curl  -k -u "testuser:testpassword" https://192.168.211.15:5000/v2/_catalog
{"repositories":["centos"]}  

#查询镜像是否存在以及标签列表
$ curl  -k -u "testuser:testpassword" https://192.168.211.15:5000/v2/centos/tags/list
{"name":"centos","tags":["latest"]}

#获取一个镜像的manifest，<name>代表镜像名，reference可以使用tag或digest
$ curl-I  -X GET 192.168.21.183:5000/v2/registry/manifests/latest
HTTP/1.1 200 OK
Content-Length: 7859
Content-Type: application/vnd.docker.distribution.manifest.v1+prettyjws
Docker-Content-Digest: sha256:d2e81c877d28a0e2bbe1e8c0e9b4608739404d14cfa0c28c6d8703a818b7bdc5
Docker-Distribution-Api-Version: registry/2.0
Etag: "sha256:d2e81c877d28a0e2bbe1e8c0e9b4608739404d14cfa0c28c6d8703a818b7bdc5"
X-Content-Type-Options: nosniff
Date: Mon, 26 Feb 2024 06:41:26 GMT

#查看一个镜像是否存在
curl -I -X HEAD localhost:5000/v2/<name>/manifests/<reference>
正常返回信息：
200 OK
Content-Length: <length of manifest>
Docker-Content-Digest: <digest>

#下载单个镜像层
curl -X GET localhost:5000/v2/<name>/blobs/<digest>

#删除一个镜像
curl -I -X DELETE localhost:5000/v2/<name>/manifests/<reference>
#删除前可以加一个--header
curl -I --header "Accept: application/vnd.docker.distribution.manifest.v2+json" -X DELETE localhost:5000/v2/<name>/manifests/<reference>
```

关于[Docker Registry API 更多细节请参考](https://docs.docker.com/registry/spec/api/#introduction)




## 9. 回收空间

```bash
docker exec -it registry sh -c  "bin/registry garbage-collect  /etc/docker/registry/config.yml"
```
清理没有tag标记的镜像
```bash
docker exec -it -u root registry bin/registry garbage-collect --delete-untagged /etc/docker/registry/config.yml
```

## 10. 镜像存储结构 
![](https://i-blog.csdnimg.cn/blog_migrate/94b4d7e5977d538d5cdc4950756fcb7a.png)
目录分为两层：`blobs`和`repositories`。

- `blobs`：镜像所有内容的实际存储，包括了镜像层和镜像元信息`manifest`；
- `repositories`是镜像元信息存储的地方，`name`代表仓库名称；
- 每一个仓库下面又分为`_layers`、`_manifests`两个部分；
- `_layers`负责记录该仓库引用了哪些镜像层文件；
- `_manifests`负责记录镜像的元信息；
- `revisions`包含了仓库下曾经上传过的所有版本的镜像元信息；
- `tags`包含了仓库中的所有标签；
- `current`记录了当前标签指向的镜像；
- `index`目录则记录了标签指向的历史镜像。

<font color=#9400D3 size=2 face="楷体">"谢谢坚持看能到最后，由于本人能力水平有限，文章或许存在错误、不足，希望大家可以后台留言。最后，本公众号文章的梳理归纳主要目的不仅仅是与大家一起成长、积累知识，而且还有应急之需便于查询。如果觉得对你有帮助，记得收藏，日后为自己的知识库迭代统一归纳，收获会更大奥。"</font>



## 一键部署仓库脚本

```bash
#!/bin/bash

reg_ip=$1
reg_n=$2
reg_port=5000

if [ $# -eq 0 ]; then
  echo "Usage: $0 [reg_ip] [registry_name]"
  echo "Please provide one or more arguments."
  exit 1
fi

BASE_DIR="$(dirname "$(readlink -f "${0}")")"
DEST_DIR='/opt/registry'
certs_dir="${DEST_DIR}/certs"
data_dir="/var/lib/registry"
mkdir -p $DEST_DIR
mkdir -p $certs_dir



# create tls certs for docker registry
create_certs() {


cat << EOF > ${DEST_DIR}/ssl.conf
[ req ]
prompt             = no
distinguished_name = req_subj
x509_extensions    = x509_ext

[ req_subj ]
CN = Localhost

[ x509_ext ]
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid,issuer
basicConstraints       = CA:true
subjectAltName         = @alternate_names

[ alternate_names ]
DNS.1 = $reg_n
IP.1  = $reg_ip
EOF




openssl req -config ${DEST_DIR}/ssl.conf -new -x509 -nodes -sha256 -days 365 -newkey rsa:4096 -keyout ${DEST_DIR}/${reg_n}.key -out ${DEST_DIR}/${reg_n}.crt
openssl x509 -inform PEM -in ${DEST_DIR}/${reg_n}.crt -out ${DEST_DIR}/${reg_n}.cert

}

# deploy docker registry
run_reg () {

cp ${DEST_DIR}/${reg_n}.key ${DEST_DIR}/${reg_n}.crt ${DEST_DIR}/${reg_n}.cert  $certs_dir


 docker run -d --restart=always --name registry-tls-certs  -v ${certs_dir}:/certs  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/${reg_n}.crt -e REGISTRY_HTTP_TLS_KEY=/certs/${reg_n}.key -e REGISTRY_COMPATIBILITY_SCHEMA1_ENABLED=true -e REGISTRY_STORAGE_DELETE_ENABLED=true  -p 443:443 -p $reg_port:5000  -v $data_dir:/var/lib/registry  registry

[ -d /etc/docker/certs.d/${reg_n}:$reg_port ]  || mkdir -p /etc/docker/certs.d/${reg_n}:${reg_port}
cp -r ${certs_dir}/${reg_n}.crt   /etc/docker/certs.d/${reg_n}:${reg_port}/
systemctl restart docker

}

# test push
push_images() {

 docker tag registry:latest ${reg_n}:${reg_port}/registry:latest
 docker push ${reg_n}:${reg_port}/registry:latest

}

create_certs
run_reg
push_images

```

参考：

 - [Docker Registry](https://docs.docker.com/registry/)
 - [registry 镜像](https://hub.docker.com/_/registry)
 - [Deploy a registry server](https://docs.docker.com/registry/deploying/)
 - [Configuring a registry](https://docs.docker.com/registry/configuration/)
 - [Docker Registry 私有仓库介绍与部署章](https://www.jianshu.com/p/07041223df66)
 - [docker 从入门到实践](https://yeasy.gitbook.io/docker_practice/repository/registry_auth)




