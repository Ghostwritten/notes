![](https://i-blog.csdnimg.cn/blog_migrate/34501aea5bd9401ba3d01f22045e25ef.png)


## 预备条件

- [docker registry仓库私搭并配置证书](https://blog.csdn.net/xixihahalelehehe/article/details/105926147)
- [centos 7.9 部署 harbor 镜像仓库实践](https://blog.csdn.net/xixihahalelehehe/article/details/127920005)
- [harbor 部署入门指南](https://blog.csdn.net/xixihahalelehehe/article/details/121381151)
- [Podman 部署私有镜像仓库](https://blog.csdn.net/xixihahalelehehe/article/details/127962045)

## 设置

```bash
$ vim /etc/hosts
192.168.23.47 registry.ghostwritten.com

$ vim /etc/containers/registries.conf
...
[[registry]]
location = "registry.ghostwritten.com"
insecure = true

$ sudo systemctl restart podman
$  podman login -u admin  registry.ghostwritten.com
Password: 
Login Succeeded!

$ podman pull registry.ghostwritten.com/k8s-public/curlimages/curl:7.85.0
Trying to pull registry.ghostwritten.com/k8s-public/curlimages/curl:7.85.0...
Getting image source signatures
Copying blob df9b9388f04a done  
Copying blob b6a21241ef17 done  
Copying blob 17b4f9600428 done  
Copying blob b07eadfb9305 done  
Copying blob 4ad5b64acddc done  
Copying blob ffaca4cc7d5a done  
Copying blob e02c9f2ce612 done  
Copying blob 8e26c9329104 done  
Copying blob 1000da985f9b done  
Copying blob a4bd3c08ea7a done  
Copying blob a32f5307f213 done  
Copying config dddbb581f8 done  
Writing manifest to image destination
Storing signatures
dddbb581f872d3d5a5083a31888badf4aace9bd73a6393d2289f9d47667b92c2
```



