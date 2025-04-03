
## 1. 创建证书

- [参考 centos 7.9 部署 harbor 镜像仓库实践](https://blog.csdn.net/xixihahalelehehe/article/details/127920005) 创建证书。


## 2. 修改harbor.yaml

```bash
https:
  # https port for harbor, default is 443
  port: 443
  # The path of cert and key files for nginx
  certificate: /root/harbor/certs/harbor.ghostwritten.com.crt
  private_key: /root/harbor/certs/harbor.ghostwritten.com.key
```

# 3. 重建 harbor
镜像资源不会被删除。

```bash
$ docker-compose down
$ ./install.sh
```

# 4. 客户端验证

- 拷贝证书到`/etc/docker/certs.d/harbor.ghostwritten.com/`目录；
- 重启docker服务。

