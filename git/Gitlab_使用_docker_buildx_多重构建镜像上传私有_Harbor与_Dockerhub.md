![在这里插入图片描述](https://img-blog.csdnimg.cn/ef8abfc46c34459480571d7044891353.png)



## 1. 预备条件


三台虚拟机

- 192.168.10.2 harbor 仓库
- 192.168.10.3 gitlab-ce
- 192.168.10.4 gitlab-runner
- 192.168.10.5 开发平台

系统： CentOS Linux release 8.5.2111
CPU： 4c
内存：8G
磁盘：40G
## 2. 安装 docker

```bash
sudo yum install -y yum-utils  device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum list docker-ce --showduplicates | sort -r
sudo yum -y install docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo systemctl start docker && sudo systemctl enable docker && sudo systemctl status docker
```

### 2.1 安装 docker buidx

 buidx在`gitlab runner` 节点安装

默认的 docker build 命令无法完成跨平台构建任务，我们需要为 docker 命令行安装 buildx 插件扩展其功能。buildx 能够使用由 [Moby BuildKit](https://github.com/moby/buildkit) 提供的构建镜像额外特性，它能够创建多个 builder 实例，在多个节点并行地执行构建任务，以及跨平台构建。

### 2.2 docker 配置
docker客户端开启实验室特性。在客户端的配置文件`~/.docker/config.json`中加入如下配置项，如果`~/.docker/config.json`文件不存在，则创建该文件。

```bash
$ cat ~/.docker/config.json
{
	"experimental": "enabled"
}

# 确认实验室性能开启。
$ docker version
```
docker服务端开启实验室特性。在配置文件`/etc/docker/daemon.json`中加入如下配置项即可，如果`/etc/docker/daemon.jso`n文件不存在，则创建该文件。

```bash
$ cat /etc/docker/daemon.json
{
	"experimental": true
}

$ systemctl daemon-reload && systemctl restart docker
$ docker version

```
### 2.3 安装 Buildx

- 首先从 [Docker buildx](https://github.com/docker/buildx/releases/) 项目的 `release` 页面找到适合自己平台的二进制文件。
- 下载二进制文件到本地并重命名为 `docker-buildx`，移动到 docker 的插件目录 `~/.docker/cli-plugins`。

```bash
wget https://github.com/docker/buildx/releases/download/v0.10.0/buildx-v0.10.0.linux-amd64
mkdir -p ~/.docker/cli-plugins
mv buildx-v0.9.1.linux-amd64 ~/.docker/cli-plugins/docker-buildx
chmod +x ~/.docker/cli-plugins/docker-buildx
```
如果想让其在系统级别可用，可将其拷贝至如下路径：
- `/usr/local/lib/docker/cli-plugins` OR `/usr/local/libexec/docker/cli-plugins`

- `/usr/lib/docker/cli-plugins` OR `/usr/libexec/docker/cli-plugins`

确认安装成功

```bash
$ docker buildx version
github.com/docker/buildx v0.9.1 ed00243a0ce2a0aee75311b06e32d33b44729689
```
### 2.4 安装模拟器
安装模拟器的主要作用是让 buildx 支持跨 CPU 架构编译。

首先查看是否已经安装模拟器

```bash
$ docker buildx ls
NAME/NODE DRIVER/ENDPOINT STATUS  BUILDKIT PLATFORMS
default * docker
  default default         running 20.10.22 linux/amd64, linux/386
```
模拟器对饮的仓库名称是：`tonistiigi/binfmt:latest`，要确保内核在`4.8`以上，`3.10.xx`不支持，[Centos 7 如何升级内核](https://blog.csdn.net/xixihahalelehehe/article/details/121618058)。

```bash
docker run --privileged --rm tonistiigi/binfmt --install all
installing: s390x OK
installing: arm OK
installing: ppc64le OK
installing: riscv64 OK
installing: mips64le OK
installing: mips64 OK
installing: arm64 OK
{
  "supported": [
    "linux/amd64",
    "linux/arm64",
    "linux/riscv64",
    "linux/ppc64le",
    "linux/s390x",
    "linux/386",
    "linux/mips64le",
    "linux/mips64",
    "linux/arm/v7",
    "linux/arm/v6"
  ],
  "emulators": [
    "qemu-aarch64",
    "qemu-arm",
    "qemu-mips64",
    "qemu-mips64el",
    "qemu-ppc64le",
    "qemu-riscv64",
    "qemu-s390x"
  ]
}

$ docker buildx ls
NAME/NODE DRIVER/ENDPOINT STATUS  BUILDKIT PLATFORMS
default * docker
  default default         running 20.10.22 linux/amd64, linux/386, linux/arm64, linux/riscv64, linux/ppc64le, linux/s390x, linux/arm/v7, linux/arm/v6
```
## 3. 安装 git
- 官方：[安装 git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
- 博客： [安装 git](https://blog.csdn.net/xixihahalelehehe/article/details/125107061)


## 4. 安装 gitlab
- 官方： [安装 gitlab](https://docs.gitlab.com/ee/install/)
- 博客：[docker 安装 gitlab](https://blog.csdn.net/xixihahalelehehe/article/details/121929582)



## 5. 部署 gitlab-runner
- 官方：[https://docs.gitlab.com/runner/install/](https://docs.gitlab.com/runner/install/)
- 博客： [安装 gitlab runner](https://blog.csdn.net/xixihahalelehehe/article/details/107755143)
## 6. 搭建 harbor
官方：[安装 harbor](https://goharbor.io/docs/2.0.0/install-config/)
博客：[安装 harbor](https://blog.csdn.net/xixihahalelehehe/article/details/121381151)
## 7. 开发应用

- kube operator demo (略)


## 8. 配置 BuildKit
如果您使用Buildx创建了一个`docker-container`或`kubernetes`构建器，您可以通过将`--config`标志传递给`docker buildx create`命令来应用自定义的BuildKit配置。


### 8.1 Registry mirror
您可以定义一个注册表镜像以用于您的生成。这样做会重定向BuildKit以从不同的主机名提取映像。以下步骤举例说明了如何将docker.io（Docker Hub）的镜像定义为mirror.gcr.io。

在`/etc/buildkitd.toml`中创建一个TOML，包含以下内容：

```bash
debug = true
[registry."docker.io"]
  mirrors = ["mirror.gcr.io"]
```
> `debug = true`打开BuildKit守护进程中的调试请求，该守护进程记录一条消息，显示何时使用镜像。

创建一个使用此BuildKit配置的`docker-container`构建器：

```bash
docker buildx create --use --bootstrap \
  --name mybuilder \
  --driver docker-container \
  --config /etc/buildkitd.toml
```
构建一个镜像

```bash
docker buildx build --load . -f - <<EOF
FROM alpine
RUN echo "hello world"
EOF
```
这个构建器的BuildKit日志现在显示它使用了GCR镜像。您可以通过响应消息包含x-goog-* HTTP头这一事实来判断。

```bash
docker logs buildx_buildkit_mybuilder0
```
输出：

```bash
...
time="2022-02-06T17:47:48Z" level=debug msg="do request" request.header.accept="application/vnd.docker.container.image.v1+json, */*" request.header.user-agent=containerd/1.5.8+unknown request.method=GET spanID=9460e5b6e64cec91 traceID=b162d3040ddf86d6614e79c66a01a577
time="2022-02-06T17:47:48Z" level=debug msg="fetch response received" response.header.accept-ranges=bytes response.header.age=1356 response.header.alt-svc="h3=\":443\"; ma=2592000,h3-29=\":443\"; ma=2592000,h3-Q050=\":443\"; ma=2592000,h3-Q046=\":443\"; ma=2592000,h3-Q043=\":443\"; ma=2592000,quic=\":443\"; ma=2592000; v=\"46,43\"" response.header.cache-control="public, max-age=3600" response.header.content-length=1469 response.header.content-type=application/octet-stream response.header.date="Sun, 06 Feb 2022 17:25:17 GMT" response.header.etag="\"774380abda8f4eae9a149e5d5d3efc83\"" response.header.expires="Sun, 06 Feb 2022 18:25:17 GMT" response.header.last-modified="Wed, 24 Nov 2021 21:07:57 GMT" response.header.server=UploadServer response.header.x-goog-generation=1637788077652182 response.header.x-goog-hash="crc32c=V3DSrg==" response.header.x-goog-hash.1="md5=d0OAq9qPTq6aFJ5dXT78gw==" response.header.x-goog-metageneration=1 response.header.x-goog-storage-class=STANDARD response.header.x-goog-stored-content-encoding=identity response.header.x-goog-stored-content-length=1469 response.header.x-guploader-uploadid=ADPycduqQipVAXc3tzXmTzKQ2gTT6CV736B2J628smtD1iDytEyiYCgvvdD8zz9BT1J1sASUq9pW_ctUyC4B-v2jvhIxnZTlKg response.status="200 OK" spanID=9460e5b6e64cec91 traceID=b162d3040ddf86d6614e79c66a01a577
time="2022-02-06T17:47:48Z" level=debug msg="fetch response received" response.header.accept-ranges=bytes response.header.age=760 response.header.alt-svc="h3=\":443\"; ma=2592000,h3-29=\":443\"; ma=2592000,h3-Q050=\":443\"; ma=2592000,h3-Q046=\":443\"; ma=2592000,h3-Q043=\":443\"; ma=2592000,quic=\":443\"; ma=2592000; v=\"46,43\"" response.header.cache-control="public, max-age=3600" response.header.content-length=1471 response.header.content-type=application/octet-stream response.header.date="Sun, 06 Feb 2022 17:35:13 GMT" response.header.etag="\"35d688bd15327daafcdb4d4395e616a8\"" response.header.expires="Sun, 06 Feb 2022 18:35:13 GMT" response.header.last-modified="Wed, 24 Nov 2021 21:07:12 GMT" response.header.server=UploadServer response.header.x-goog-generation=1637788032100793 response.header.x-goog-hash="crc32c=aWgRjA==" response.header.x-goog-hash.1="md5=NdaIvRUyfar8201DleYWqA==" response.header.x-goog-metageneration=1 response.header.x-goog-storage-class=STANDARD response.header.x-goog-stored-content-encoding=identity response.header.x-goog-stored-content-length=1471 response.header.x-guploader-uploadid=ADPycdtR-gJYwC7yHquIkJWFFG8FovDySvtmRnZBqlO3yVDanBXh_VqKYt400yhuf0XbQ3ZMB9IZV2vlcyHezn_Pu3a1SMMtiw response.status="200 OK" spanID=9460e5b6e64cec91 traceID=b162d3040ddf86d6614e79c66a01a577
time="2022-02-06T17:47:48Z" level=debug msg=fetch spanID=9460e5b6e64cec91 traceID=b162d3040ddf86d6614e79c66a01a577
time="2022-02-06T17:47:48Z" level=debug msg=fetch spanID=9460e5b6e64cec91 traceID=b162d3040ddf86d6614e79c66a01a577
time="2022-02-06T17:47:48Z" level=debug msg=fetch spanID=9460e5b6e64cec91 traceID=b162d3040ddf86d6614e79c66a01a577
time="2022-02-06T17:47:48Z" level=debug msg=fetch spanID=9460e5b6e64cec91 traceID=b162d3040ddf86d6614e79c66a01a577
time="2022-02-06T17:47:48Z" level=debug msg="do request" request.header.accept="application/vnd.docker.image.rootfs.diff.tar.gzip, */*" request.header.user-agent=containerd/1.5.8+unknown request.method=GET spanID=9460e5b6e64cec91 traceID=b162d3040ddf86d6614e79c66a01a577
time="2022-02-06T17:47:48Z" level=debug msg="fetch response received" response.header.accept-ranges=bytes response.header.age=1356 response.header.alt-svc="h3=\":443\"; ma=2592000,h3-29=\":443\"; ma=2592000,h3-Q050=\":443\"; ma=2592000,h3-Q046=\":443\"; ma=2592000,h3-Q043=\":443\"; ma=2592000,quic=\":443\"; ma=2592000; v=\"46,43\"" response.header.cache-control="public, max-age=3600" response.header.content-length=2818413 response.header.content-type=application/octet-stream response.header.date="Sun, 06 Feb 2022 17:25:17 GMT" response.header.etag="\"1d55e7be5a77c4a908ad11bc33ebea1c\"" response.header.expires="Sun, 06 Feb 2022 18:25:17 GMT" response.header.last-modified="Wed, 24 Nov 2021 21:07:06 GMT" response.header.server=UploadServer response.header.x-goog-generation=1637788026431708 response.header.x-goog-hash="crc32c=ZojF+g==" response.header.x-goog-hash.1="md5=HVXnvlp3xKkIrRG8M+vqHA==" response.header.x-goog-metageneration=1 response.header.x-goog-storage-class=STANDARD response.header.x-goog-stored-content-encoding=identity response.header.x-goog-stored-content-length=2818413 response.header.x-guploader-uploadid=ADPycdsebqxiTBJqZ0bv9zBigjFxgQydD2ESZSkKchpE0ILlN9Ibko3C5r4fJTJ4UR9ddp-UBd-2v_4eRpZ8Yo2llW_j4k8WhQ response.status="200 OK" spanID=9460e5b6e64cec91 traceID=b162d3040ddf86d6614e79c66a01a577
...
```
### 8.2 设置镜像仓库正式
如果您在BuildKit配置中指定了镜像仓库证书，则守护进程会将文件复制到`/etc/buildkit/certs`下的容器中。以下步骤显示如何将自签名镜像仓库证书添加到`BuildKit`配置。

1. 将以下配置添加到`/etc/buildkitd.toml`

```bash
# /etc/buildkitd.toml
debug = true
[registry."myregistry.com"]
  ca=["/etc/docker/certs.d/myregistry.com/myregistry.crt"]
  [[registry."myregistry.com".keypair]]
    key="/etc/docker/certs.d/myregistry.com/myregistry.key"
    cert="/etc/docker/certs.d/myregistry.com/myregistry.cert"
```
这将告诉构建器使用指定位置（`/etc/certs`）中的证书将图像推送到 `myregistry.com` 仓库。

2. 创建一个使用以下配置的docker-container构建器：

```bash
docker buildx create --use --bootstrap \
  --name mybuilder \
  --driver docker-container \
  --config /etc/buildkitd.toml
```
检查构建器的配置文件（`/etc/buildkit/buildkitd.toml`），它显示证书配置现在已在构建器中配置。

```bash
docker exec -it buildx_buildkit_mybuilder0 cat /etc/buildkit/buildkitd.toml
```

```bash
debug = true

[registry]

  [registry."myregistry.com"]
    ca = ["/etc/buildkit/certs/myregistry.com/myregistry.crt"]

    [[registry."myregistry.com".keypair]]
      cert = "/etc/buildkit/certs/myregistry.com/myregistry.cert"
      key = "/etc/buildkit/certs/myregistry.com/myregistry.key"
```
验证证书是否在容器中：

```bash
$ docker exec -it buildx_buildkit_mybuilder0 ls /etc/buildkit/certs/myregistry.com/
myregistry.crt    myregistry.cert   myregistry.key
```
现在，您可以使用此构建器推送到镜像仓库，它将使用证书进行身份验证：

```bash
$ docker buildx build --push --tag myregistry.com/myimage:latest .
```
构建并推送到 harbor 镜像仓库和 `dockerhub`
```bash
$ docker buildx build --platform linux/amd64,linux/arm64 -t $HARBOR_HOST/$HARBOR_PROJECT/$NMAE:dev-${CI_COMMIT_SHORT_SHA} -t docker.io/ghostwritten/$NMAE:dev-${CI_COMMIT_SHORT_SHA}  -f Dockerfile . --push
```
## 9. 编写 .gitlabs-ci.yaml

```bash
variables:
  PLATFORM: "linux/amd64,linux/arm64"
  HARBOR_HOST: "harbor.demo.com"
  HARBOR_PROJECT: "library"
 
image: docker:24.0.4

services:
  - docker:24.0.4-dind

stages:
  - build_push


build-push-dev-job:
  stage: build_push
  script:
    - docker buildx build --platform $PLATFORM -t $HARBOR_HOST/$HARBOR_PROJECT/xxxxxx:dev-${CI_COMMIT_SHORT_SHA} -t docker.io/ghostwritten/xxxxx:dev-${CI_COMMIT_SHORT_SHA}  -f Dockerfile-multi . --push
  only:
    - dev

```

参考：
- [Configure BuildKit](https://docs.docker.com/build/buildkit/configure/)
- [Install GitLab](https://docs.gitlab.com/ee/install/) 
- [Harbor Installation and Configuration](https://goharbor.io/docs/2.0.0/install-config/)
- [The secret gems behind building container images, Enter: BuildKit & Docker Buildx](https://blog.kubesimplify.com/the-secret-gems-behind-building-container-images-enter-buildkit-and-docker-buildx)
- 
