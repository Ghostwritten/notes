
![](https://i-blog.csdnimg.cn/blog_migrate/00911abc81c213464bc5de61ed5f0616.jpeg#pic_center)



## 1. 实现目标

- 实现一个go 开发 demo web，并实现构建镜像，运行容器，推送镜像入库。

## 2. 开发应用


开发环境：linux 、windows、mac

```bash
package main

import (
	"fmt"
	"html"
	"log"
	"net/http"
)

func main() {

	fmt.Println("launching server at port 9090")

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {

		fmt.Fprintf(w, "hello, %q", html.EscapeString(r.URL.Path))
	})

	log.Fatal(http.ListenAndServe(":9090", nil))

}

```
第一个终端执行：

```bash
$ go run main.go 
launching server at port 9090
```
第二个终端模拟客户端访问，效果如下：

```bash
$ curl http://localhost:9090/docker-go
hello, "/docker-go"

$ curl http://localhost:9090/google
hello, "/google"

$ curl http://localhost:9090/google/demo1
hello, "/google/demo1"
```


## 3. 编写 Dockerfile
将 上面开发的demo 应用构建一个镜像。

首先，编写一个Dockerfile

```bash
FROM golang:latest

RUN mkdir /app

WORKDIR /app

ADD . /app

RUN go build -o main ./server.go

EXPOSE 9090

CMD /app/main

```
Dockerfile命令作用：

- FROM golang:latest 以golang的最新的镜像为基础镜像创建
- RUN mkdir /app 在容器中创建一个app文件夹
- WORKDIR /app 将这个app认为默认工作文件夹
- ADD . /app 将当前目录的下的所有文件加入到这个app工作文件夹中
- RUN go build -o main ./server.go 开始build server.go名称任意 (是你的go项目的程序入口)
- EXPOSE 9090 向外面暴露9090端口
- CMD /app/main 然后运行我们build的生成的可执行文件main.exe


## 4. 构建镜像

```bash
$ sudo nerdctl build . -t demo1:v1 
[+] Building 24.3s (10/10) FINISHED                                                                                                                                                     
 => [internal] load build definition from Dockerfile                                                                                                                               0.0s
 => => transferring dockerfile: 158B                                                                                                                                               0.0s
 => [internal] load metadata for docker.io/library/golang:latest                                                                                                                   0.9s
 => [internal] load .dockerignore                                                                                                                                                  0.0s
 => => transferring context: 2B                                                                                                                                                    0.0s
 => [internal] load build context                                                                                                                                                  0.0s
 => => transferring context: 108B                                                                                                                                                  0.0s
 => [1/5] FROM docker.io/library/golang:latest@sha256:2ff79bcdaff74368a9fdcb06f6599e54a71caf520fd2357a55feddd504bcaffb                                                             0.0s
 => => resolve docker.io/library/golang:latest@sha256:2ff79bcdaff74368a9fdcb06f6599e54a71caf520fd2357a55feddd504bcaffb                                                             0.0s
 => CACHED [2/5] RUN mkdir /app                                                                                                                                                    0.0s
 => CACHED [3/5] WORKDIR /app                                                                                                                                                      0.0s
 => CACHED [4/5] ADD . /app                                                                                                                                                        0.0s
 => CACHED [5/5] RUN go build -o main ./main.go                                                                                                                                    0.0s
 => exporting to docker image format                                                                                                                                              22.8s
 => => exporting layers                                                                                                                                                            0.0s
 => => exporting manifest sha256:88bd9d93ab4c3a1f0a0dedb3041548a363e1adf6bf5c8beef7a21af7ba58347a                                                                                  0.0s
 => => exporting config sha256:7523e47d0e4def8ba54b5df919588598e7821192380968df21a6789d4f43ad6b                                                                                    0.0s
 => => sending tarball                                                                                                                                                            22.7s
unpacking docker.io/library/demo1:v1 (sha256:88bd9d93ab4c3a1f0a0dedb3041548a363e1adf6bf5c8beef7a21af7ba58347a)...
Loaded image: docker.io/library/demo1:v1
```


当然，对于构建镜像来说，我们仍然可以使用 docker 构建镜像。

```bash
sudo docker build . -t demo1:v1 
```

## 5. 测试应用

创建容器
```bash
$ sudo nerdctl run -t -d -p 9090:9090 --name demo demo1:v1
f16ee68da8542b8cf8d30ea2c1c91ad88170361089aa694f6a76e4610d20c37c
```
- -d 表示后台运行
- -p 表示端口映射 将本地的9090端口映射到容器的9090端口
- demo 新建的容器的名称
- demo1:v1 容器所使用的镜像的名称

查看容器

```bash
$ sudo nerdctl ps
CONTAINER ID    IMAGE                             COMMAND                   CREATED           STATUS    PORTS                     NAMES
f16ee68da854    docker.io/library/demo1:v1        "/bin/sh -c /app/main"    24 seconds ago    Up        0.0.0.0:9090->9090/tcp    demo
```

测试

```bash
$ curl http://localhost:9090/demo
hello, "/demo"
```
当然，如果9090端口被占用。可以映射其他端口。例如9091：

```bash
sudo nerdctl run -t -d -p 9091:9090 --name demo demo1:v1¬
```

## 6. 场景运用

在实际开发应用，我们可能希望将配置文件或者数据目录挂载出来，这样，方便更新配置文件，或者存储大量的数据内容。

比如在`/root/demo1`目录。包含配置文件`config.yaml` 和目录`data`，可以通过 -v 命令参数实现挂载。

```bash
$ ls /root/demo1/
config.yaml  data
$ sudo nerdctl run -t -d -p 9091:9090 -v /root/demo1/config.yaml:/app/config.yaml -v /root/demo1/data:/app/data  ---name demo demo1:v1
```
另外，应用也许通过环境变量传递一些变量，例如，用户名、	其他应用访问的域名等等。可以通过 -e 命令参数实现。

```bash
$ sudo nerdctl run -t -d -p 9091:9090 -e USER=demo  -v /root/demo1/config.yaml:/app/config.yaml -v /root/demo1/data:/app/data   ---name demo demo1:v1
```

##  7. 镜像入库

当我们研发测试通过验证后，需要把镜像推送的镜像仓库，为生产应用提供拉取镜像的新版本。

首先，我们需要在测试环境镜像打包。

```bash
$ sudo nerdctl save -o demo1.tar.gz demo1:v1
$ ls demo1.tar.gz  -l
-rw-r--r-- 1 root root 316571136 Dec 19 17:46 demo1.tar.gz
```

然后，镜像软件包拷贝到适用于推送镜像入库的节点，建议该节点不是仓库节点或者生产的集群，因为，软件包和解压的介质积累过多会导致占用额外的空间，给生产环境带来磁盘爆满的风险。

镜像入库节点的镜像入库工具，同样有多种选择，docker、nerdctl、podman、ctr、crictl等。这里nerdctl 为主。

```bash
$ sudo nerdctl load -i demo1.tar.gz
unpacking docker.io/library/demo1:v1 (sha256:88bd9d93ab4c3a1f0a0dedb3041548a363e1adf6bf5c8beef7a21af7ba58347a)...
Loaded image: demo1:v1
```
以 `registry.ghostwritten.com` 仓库名为例。我们希望把镜像demo1:v1推送到  `registry.ghostwritten.com` 仓库的gpu项目下，注意，如果我们的镜像仓库是harbor部署的，那么项目名需要界面创建好，保障一存在。

首先，给镜像打一个标签。

```bash
sudo nerdctl tag demo1:v1 registry.ghostwritten.com/gpu/demo1:v1 
```
然后推送入库

```bash
sudo nerdctl push registry.ghostwritten.com/gpu/demo1:v1 
```

参考：

- [https://github.com/containerd/nerdctl/tree/main/docs](https://github.com/containerd/nerdctl/tree/main/docs)
