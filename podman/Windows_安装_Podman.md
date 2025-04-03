

##  1. 准备条件
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c13fbe413bb375496649f954f972a3c1.png)

```bash
bcdedit /set hypervisorlaunchtype auto
```

```bash
C:\WINDOWS\system32>bcdedit /set hypervisorlaunchtype auto
操作成功完成。
```
重启电脑

## 2. 安装 podman
打开 `windows Terminal`  Powershell
```bash
  podman machine init
Extracting compressed file
Importing operating system into WSL (this may take a few minutes on a new WSL install)...
Configuring system...
Generating public/private ed25519 key pair.
Your identification has been saved in podman-machine-default
Your public key has been saved in podman-machine-default.pub
The key fingerprint is:
SHA256:N2DjZlZQrCz0jzfsDpcle78wPtZf3NMXnz7TyZIGsPA root@ZongXun-Windows
The key's randomart image is:
+--[ED25519 256]--+
|        .o.      |
|      .  ..      |
|     . o+..      |
|      .++=       |
|       .S+* .  . |
|       +.E=B   .*|
|        .o+.=.ooX|
|         o.ooBo==|
|         ...o.+++|
+----[SHA256]-----+
Machine init complete
To start your machine run:

        podman machine start

>  pwsh  ~                                                                                               15:11:00  
  podman machine start
Starting machine "podman-machine-default"

This machine is currently configured in rootless mode. If your containers
require root permissions (e.g. ports < 1024), or if you run into compatibility
issues with non-podman clients, you can switch using the following command:

        podman machine set --rootful

API forwarding listening on: npipe:////./pipe/docker_engine

Docker API clients default to this address. You do not need to set DOCKER_HOST.
Machine "podman-machine-default" started successfully
>  pwsh  ~                                                                                               15:11:13  
  podman machine set --rootful
```
测试

```bash
>  pwsh  ~         
 podman pull busybox
Resolved "busybox" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull docker.io/library/busybox:latest...
Getting image source signatures
Copying blob sha256:405fecb6a2fa4f29683f977e7e3b852bf6f8975a2aba647d234d2371894943da
Copying config sha256:9d5226e6ce3fb6aee2822206a5ef85f38c303d2b37bfc894b419fca2c0501269
Writing manifest to image destination
Storing signatures
9d5226e6ce3fb6aee2822206a5ef85f38c303d2b37bfc894b419fca2c0501269
```

