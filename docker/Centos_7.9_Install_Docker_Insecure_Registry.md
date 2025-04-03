
![](https://i-blog.csdnimg.cn/blog_migrate/8c68c0164578a7daf5ac01ccd9c9affb.png)




## 1. 镜像存储规划

- [linux LVM /dev/sdb mount dir /data【linux LVM 磁盘挂载目录】](https://ghostwritten.blog.csdn.net/article/details/134633286)

创建两个目录
- 一个 docker 数据存储目录 ：/data/docker，默认一般为linux为 `/var/lib/docker`，windows 为`C:\ProgramData\docker` 
- 一个registry 镜像仓库数据目录： /data/registry

```bash
mkdir /data/docker
mkdir /data/registry
```

## 2. 安装定制 docker

- [docker install 【docker 安装】](https://blog.csdn.net/xixihahalelehehe/article/details/104293170)

```bash
yum install -y yum-utils 
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum list docker-ce --showduplicates | sort -r
sudo yum -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

```
配置

```bash
cat <<EOF> /etc/docker/daemon.json 
{
   "exec-opts": ["native.cgroupdriver=systemd"],
   "data-root": "/data/docker",
   "live-restore": true,
   "log-driver": "json-file",
   "log-opts": {
     "max-size":  "100m",
     "max-file": "5"
    }
 }
EOF
```

启动

```bash
sudo systemctl start docker && systemctl enable docker  && systemctl status docker
```
验证

```bash
$ ls /data/docker/
buildkit  containers  engine-id  image  network  overlay2  plugins  runtimes  swarm  tmp  volumes
```

## 3. 部署 registry

拉取镜像

```bash
docker pull registry:2.8.3
```
创建镜像仓库

```bash
docker run -tid --restart=always --name registry -p 80:5000 -e REGISTRY_COMPATIBILITY_SCHEMA1_ENABLED=true -v /data/registry:/var/lib/registry registry:latest

```

检查状态

```bash
$ docker ps
CONTAINER ID   IMAGE            COMMAND                  CREATED         STATUS         PORTS                                       NAMES
238a044893a5   registry:2.8.3   "/entrypoint.sh /etc…"   5 seconds ago   Up 4 seconds   0.0.0.0:80->5000/tcp, :::80->5000/tcp   registry
```

配置域名解析

```bash
echo "192.168.10.22  registry01.ghostwritten.com" >> /etc/hosts
```

修改配置

```bash
$ cat /etc/docker/daemon.json
{
   "exec-opts": ["native.cgroupdriver=systemd"],
   "insecure-registries": ["registry01.ghostwritten.com"],  #添加
   "data-root": "/data/docker",
   "live-restore": true,
   "log-driver": "json-file",
   "log-opts": {
     "max-size":  "100m",
     "max-file": "5"
    }
 }

$ systemctl restart docker
```

## 4. 验证镜像仓库

检查仓库

```bash
$ curl 192.168.10.22/v2/_catalog
{"repositories":[]}
```

推送镜像

```bash
$ docker tag registry:2.8.3 registry01.ghostwritten.com/library/registry:2.8.3
$ docker push registry01.ghostwritten.com/library/registry:2.8.3
The push refers to repository [registry01.ghostwritten.com/library/registry]
ab4798a34c77: Layer already exists 
0b261c932361: Layer already exists 
d95d36f1fde7: Layer already exists 
b4fcd5c55862: Layer already exists 
cc2447e1835a: Layer already exists 
2.8.3: digest: sha256:386cdae4ba70c368b780a6e54251a14d300281a3d147a18ef08ae6fb079d150c size: 1363
```

拉取镜像镜像

登陆另一台节点，重复上面安装docker 、配置docker、配置域名解析，即可拉取镜像

```bash
docker pull registry01.ghostwritten.com/library/registry:2.8.3
```

参考：

- [https://docs.docker.com/registry/](https://docs.docker.com/registry/)
