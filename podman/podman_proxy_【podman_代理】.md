![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/be161e4e10038740a7a4813e49f5768e.png)



## 方法1: 为当前用户设置环境变量

您可以为当前用户设置 HTTP_PROXY 和 HTTPS_PROXY 环境变量,Podman 将自动读取这些环境变量并使用代理。

```bash
# Bash
export HTTP_PROXY="http://代理地址:端口"  
export HTTPS_PROXY="https://代理地址:端口"

# 对于 bash, 也可以在 ~/.bashrc 中添加上述命令使其永久有效

# Fish
set -x HTTP_PROXY "http://代理地址:端口"
set -x HTTPS_PROXY "https://代理地址:端口"

# 对于 fish,也可以在 ~/.config/fish/config.fish 中添加以上命令
```

如果您的代理需要身份验证,可以在 URL 中添加用户名和密码。格式如下:

```bash
http://用户名:密码@代理地址:端口
```

## 方法2：为 Podman 服务设置配置文件
您还可以通过编辑 /etc/containers/registries.conf 配置文件为 Podman 服务设置代理。在该文件中添加如下内容:

```bash
[registries.search]
registries = ['docker.io', 'quay.io']

[registries.insecure]
registries = []

[registries.block]
registries = []

[registries.unqualified-search-registries]

[registry.mirrors]

[registry.configs]

[registry.configs.REGISTRY_NAME.HOSTNAME/HOSTPATH]  
unqualified-search-registries = ["registry.fedoraproject.org", "registry.access.redhat.com", "docker.io"]
blocked=false 

[registry.configs.REGISTRY_NAME.HOSTNAME]
http-proxy="http://代理地址:端口"
https-proxy="https://代理地址:端口"
```

替换 REGISTRY_NAME.HOSTNAME 为您要配置的注册表,如 docker.io。如果代理需要身份验证,则使用类似 http://user:password@proxy.example.com:8080 的格式。

## 方法3: 为单个 Podman 命令设置代理

您也可以为单个 Podman 命令临时设置代理,方法是在命令前添加 --build-arg 参数。例如:

```bash
podman --build-arg HTTP_PROXY="http://代理地址:端口" --build-arg HTTPS_PROXY="https://代理地址:端口" pull nginx
```

## 方法四: 配置 http-proxy.conf
```bash
$ systemctl status podman
● podman.service - Podman API Service
   Loaded: loaded (/usr/lib/systemd/system/podman.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/podman.service.d
           └─http-proxy.conf
   Active: inactive (dead) since Mon 2023-11-20 18:45:12 CST; 3 months 22 days ago
     Docs: man:podman-system-service(1)
  Process: 50669 ExecStart=/usr/bin/podman $LOGGING system service (code=exited, status=0/SUCCESS)
 Main PID: 50669 (code=exited, status=0/SUCCESS)

Nov 20 18:45:07 downlaod systemd[1]: Starting Podman API Service...
Nov 20 18:45:07 downlaod systemd[1]: Started Podman API Service.
Nov 20 18:45:07 downlaod podman[50669]: time="2023-11-20T18:45:07+08:00" level=info msg="/usr/bin/podman filtering at log level>
Nov 20 18:45:07 downlaod podman[50669]: time="2023-11-20T18:45:07+08:00" level=info msg="Not using native diff for overlay, thi>
Nov 20 18:45:07 downlaod podman[50669]: time="2023-11-20T18:45:07+08:00" level=info msg="Setting parallel job count to 13"
Nov 20 18:45:07 downlaod podman[50669]: time="2023-11-20T18:45:07+08:00" level=info msg="Using systemd socket activation to det>
Nov 20 18:45:07 downlaod podman[50669]: time="2023-11-20T18:45:07+08:00" level=info msg="API service listening on \"/run/podman>
Nov 20 18:45:12 downlaod systemd[1]: podman.service: Succeeded.

$ cat /etc/systemd/system/podman.service.d/http-proxy.conf 
[Service]
Environment="HTTP_PROXY=http://192.168.21.101:7890"
Environment="HTTPS_PROXY=http://192.168.21.101:7890"
Environment="NO_PROXY=localhost,127.0.0.1,.coding.net,.tencentyun.com,.myqcloud.com,harbor.bsgchina.com"
```

