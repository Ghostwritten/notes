
![](https://i-blog.csdnimg.cn/blog_migrate/0b3d11833631d118b9b681a5f82cdc67.png)

## 第一种

创建代理配置文件
```bash
mkdir -p  /etc/systemd/system/docker.service.d/
cat <<EOF >  /etc/systemd/system/docker.service.d/http-proxy.conf
[Service]
Environment="HTTP_PROXY=http://192.168.21.101:7890"
Environment="HTTPS_PROXY=http://192.168.21.101:7890"
Environment="NO_PROXY=localhost,127.0.0.1,.coding.net,.tencentyun.com,.myqcloud.com,harbor.bsgchina.com"
EOF
```
重启docker

```bash
systemctl daemon-reload && systemctl restart docker && systemctl status docker
```
测试

```bash
$ docker pull registry.k8s.io/pause:3.9
3.9: Pulling from pause
Digest: sha256:7031c1b283388d2c2e09b57badb803c05ebed362dc88d84b480cc47f72a21097
Status: Image is up to date for registry.k8s.io/pause:3.9
registry.k8s.io/pause:3.9
```


- [Use a proxy server with the Docker CLI](https://docs.docker.com/engine/cli/proxy/)
- [Configure the daemon to use a proxy](https://docs.docker.com/engine/daemon/proxy/)
