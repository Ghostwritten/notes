
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e437b488affa3407f6d74a2a319595a2.jpeg#pic_center)


#  docker 部署 registry UI



## 1. 创建registry仓库

```bash
docker run -tid --restart=always --name registry -p 5000:5000 -e REGISTRY_COMPATIBILITY_SCHEMA1_ENABLED=true -v /storage/registry:/var/lib/registry registry:latest
docker ps
```

## 2. 将镜像推入仓库

```bash
$ docker pull centos
$ docker tag  centos:latest 192.168.211.15:5000/centos:latest
$ docker push 192.168.211.15:5000/centos:latest
The push refers to a repository [192.168.211.15:5000/centos]
Get https://192.168.211.15:5000/v1/_ping: http: server gave HTTP response to HTTPS client
```

在推送镜像中出现错误，因为client与Registry交互默认将采用https访问，但我们在install Registry时并未配置指定任何tls相关的key和crt文件，https将无法访问。因此， 我们需要配置客户端的Insecure Registry选项（另一种解决方案需要配置Registry的证书）。

```bash
$ vim /etc/sysconfig/docker
OPTIONS='--selinux-enabled --log-driver=journald --signature-verification=false --insecure-registry 192.168.211.15:5000'
或者
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
$ docker stop registry
$ systemctl restart docker
$ docker start registry
$ docker push 192.168.211.15:5000/centos:latest
$ $ 

{"repositories":["centos"]}   #获取镜像列表
```



## 3. 创建registry-web
Docker官方只提供了REST API，并没有给我们一个界面。 可以使用Portus来管理私有仓库， 同时可以使用简单的UI管理工具， Docker提供私有库“hyper/docker-registry-web”， 下载该镜像就可以使用了。

```bash
docker run -d -p 8080:8080 --name registry-web  --link registry -e REGISTRY_URL=http://registry:5000/v2  -e REGISTRY_NAME=registry:5000 hyper/docker-registry-web
```
界面：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/333937aa11bbb209ff5902b782cd3906.png)

## 4. web 对接token认证的registry server
##  4.1 搭建 docker registry web

```bash
$ mkdir -p  /data/registry-web/config
$ vim /data/registry-web/config/registry-web.yml
registry:
  # Docker registry url
  url: 'https://192.168.211.100:5000/v2'
  # web registry context path
  # empty string for root context, /app to make web registry accessible on http://host/app
  context_path: ''
  # Trust any SSL certificate when connecting to registry
  trust_any_ssl: true
  #  base64 encoded token for basic authentication
  basic_auth: ''
  # To allow image delete, should be false
  readonly: true
  # Docker registry fqdn
  name: '192.168.211.100:5000'
  # Authentication settings
  auth:
    # Enable authentication
    enabled: true
    # Allow registry anonymous access
    # allow_anonymous: true # not implemented
    # Token issuer
    # should equals to auth.token.issuer of docker registry
    issuer: 'test-issuer'
    # Private key for token signing
    # certificate used on auth.token.rootcertbundle should signed by this key
    key: /config/server.key

```
访问：`http://192.168.211.100:5000`
用户与密码：`admin`：`admin`
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/374a4d7af0fdf42126b85d55fbcbb4b8.png)
### 4.2 创建用户
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/682b3d96edf2f64b78f3395a53dbe14d.png)
### 4.3 用户授权
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/82a2a3ac11e48daf79a03e87334d920f.png)

参考：

 - [registry-web](https://hub.docker.com/r/hyper/docker-registry-web/)


## demo

```bash
docker run -d --restart=always --name registry -p 5000:5000 -e REGISTRY_COMPATIBILITY_SCHEMA1_ENABLED=true -v /storage/registry:/var/lib/registry registry:latest

cat > /etc/docker/daemon.json <<EOF
{
  "live-restore": true,
  "dns": ["8.8.8.8"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size":  "100m",
    "max-file": "5"
   },
  "insecure-registries": ["registry01.demo:5000"]
}
EOF
sytemctl restart docker

 docker run -d -p 8080:8080 --name registry-web  --link registry -e REGISTRY_URL=http://registry01.demo:5000/v2  -e REGISTRY_NAME=registry01.demo:5000 hyper/docker-registry-web
```

