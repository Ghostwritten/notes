
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cb9fd6f0924c82eefdf718d60aec9929.jpeg#pic_center)


## 1. 简介
一个更加直接搬运镜像的方法。

## 2. 对比

通常方法:
- 拉镜像 
- 打包镜像
- 搬运镜像
- 解压镜像
- 打标签
- 推送镜像

新方法:

- 打包镜像仓库目录
- 搬运仓库包
- 解压包

## 3. 预备条件

| 角色 | ip  | cpu| mem| disk|ios|kernel|
|--|--|--|--|--|--|--|
|registry01 |192.168.23.80  |4  |8G|100G| Rocky 8.8 |4.18+|
| registry02 |192.168.23.81 |4  |8G|100G|Rocky 8.8|4.18+|




## 4. 安装 registry

- [设置 docker 存储目录](https://cblog.csdn.net/xixihahalelehehe/article/details/134633286)
- [安装 docker](https://blog.csdn.net/xixihahalelehehe/article/details/104293170)

192.168.23.80执行：

```bash
cat > /etc/docker/daemon.json<<EOF
{
  "live-restore": true,
  "dns": ["8.8.8.8"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size":  "100m",
    "max-file": "5"
   },
  "insecure-registries": ["192.168.23.80:5000"]
}
systemctl restart docker
docker run -tid --restart=always --name registry -p 5000:5000 -e REGISTRY_COMPATIBILITY_SCHEMA1_ENABLED=true -v /storage/registry:/var/lib/registry registry:latest
docker run -tid -p 8080:8080 --name registry-web  --link registry -e REGISTRY_URL=http://192.168.23.80:5000/v2  -e REGISTRY_NAME=192.168.23.80:5000 hyper/docker-registry-web
docker tag registry:latest 192.168.23.80:5000/registry:latest
docker push 192.168.23.80:5000/registry:latest
```

登陆 192.168.23.80:8080
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7c93f00faf9c9898314e24ea2d154496.png)



192.168.23.81执行：

192.168.23.81为空的镜像仓库
```bash
cat > /etc/docker/daemon.json<<EOF
{
  "live-restore": true,
  "dns": ["8.8.8.8"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size":  "100m",
    "max-file": "5"
   },
  "insecure-registries": ["192.168.23.81:5000"]
}
systemctl restart docker
docker run -tid --restart=always --name registry -p 5000:5000 -e REGISTRY_COMPATIBILITY_SCHEMA1_ENABLED=true -v /storage/registry:/var/lib/registry registry:latest
docker run -tid -p 8080:8080 --name registry-web  --link registry -e REGISTRY_URL=http://192.168.23.81:5000/v2  -e REGISTRY_NAME=192.168.23.1:5000 hyper/docker-registry-web
```


## 5. 仓库搬运
192.168.23.80执行：
```bash
cd /storage/registry/
tar zcvf docker-storage.tar.gz docker
```
搬运包至192.168.23.81的/storage/registry/

```bash
cd /storage/registry/
tar zxvf docker-storage.tar.gz
docker restart registy-web
```
界面访问：192.168.23.81:8080
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/09b0064f3ef20566ae98897d0f19ce5d.png)

## 6. 测试

```bash
$ docker pull 192.168.23.81:5000/registry:latest
latest: Pulling from registry
Digest: sha256:12202eb78732e22f8658d595bd6e3d47ef9f13ede78e94e90974c020c7d7c1b3
Status: Downloaded newer image for 192.168.23.81:5000/registry:latest
```
拉取镜像成功。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f27711fa67cf30d71450812e17cc2bb7.jpeg#pic_center)

