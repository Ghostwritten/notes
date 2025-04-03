


-----
## 1. 介绍

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/50d1ac47d116368c9e3d074245ca6d59.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c00db9f87df21036b35366817cdb97aa.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a14f9ee014d53467c23d04d4e0f55c06.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7b70d29ce927652388efa1ab9a4621a8.png)
## 2. 多阶段镜像构建
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7b13b769de33c485d78b4890a90b844c.png)

```bash
root@master:~/cks/image_footprint# cat Dockerfile 
FROM ubuntu
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y golang-go
COPY app.go .
RUN CGO_ENABLED=0 go build app.go
CMD ["./app"]



root@master:~/cks/image_footprint# cat app.go 
package main

import (
    "fmt"
    "time"
    "os/user"
)

func main () {
    user, err := user.Current()
    if err != nil {
        panic(err)
    }

    for {
        fmt.Println("user: " + user.Username + " id: " + user.Uid)
        time.Sleep(1 * time.Second)
    }
}
```

```c
root@master:~/cks/image_footprint# docker  build  -t app .
```
如果报错[Docker build “Could not resolve ‘archive.ubuntu.com’” apt-get fails to install anything](https://ghostwritten.blog.csdn.net/article/details/117083378)
执行
```c

root@master:~/cks/image_footprint# docker  build --network=host -t app .
....
....
Step 4/6 : COPY app.go .
 ---> 3e1402a9aa76
Step 5/6 : RUN CGO_ENABLED=0 go build app.go
 ---> Running in 40d3a61c48c1
Removing intermediate container 40d3a61c48c1
 ---> badcd4b100b5
Step 6/6 : CMD ["./app"]
 ---> Running in 90bf3a7cd05b
Removing intermediate container 90bf3a7cd05b
 ---> 847a0ea160db
Successfully built 847a0ea160db
Successfully tagged app:latest




root@master:~/cks/image_footprint# docker run app
user: root id: 0
user: root id: 0
user: root id: 0
user: root id: 0
^Cuser: root id: 0


root@master:~/cks/image_footprint# docker images  |grep app
app                                    latest              5acf9df3a2ee        About a minute ago   678MB


#利用上一个镜像构建下一个镜像
root@node1:~/cks/image_footprint# cat Dockerfile 
FROM ubuntu
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y golang-go
COPY app.go .
RUN CGO_ENABLED=0 go build app.go


FROM alpine
COPY --from=0 /app .
CMD ["./app"]


oot@node1:~/cks/image_footprint# docker  build --network=host -t app .
Sending build context to Docker daemon  3.072kB
Step 1/8 : FROM ubuntu
 ---> 7e0aa2d69a15
Step 2/8 : ARG DEBIAN_FRONTEND=noninteractive
 ---> Using cache
 ---> ca181f94a1d8
Step 3/8 : RUN apt-get update && apt-get install -y golang-go
 ---> Using cache
 ---> 2b8ae7feb9d3
Step 4/8 : COPY app.go .
 ---> Using cache
 ---> 2bfae84136e8
Step 5/8 : RUN CGO_ENABLED=0 go build app.go
 ---> Using cache
 ---> 396e81b07b04
Step 6/8 : FROM alpine
latest: Pulling from library/alpine
540db60ca938: Pull complete 
Digest: sha256:69e70a79f2d41ab5d637de98c1e0b055206ba40a8145e7bddb55ccc04e13cf8f
Status: Downloaded newer image for alpine:latest
 ---> 6dbb9cc54074
Step 7/8 : COPY --from=0 /app .
 ---> 82a88cf8bdaa
Step 8/8 : CMD ["./app"]
 ---> Running in 4bfb018ccea7
Removing intermediate container 4bfb018ccea7
 ---> 3a81c4f3f3dc
Successfully built 3a81c4f3f3dc
Successfully tagged app:latest

功能正常
root@node1:~/cks/image_footprint# docker run app
user: root id: 0
user: root id: 0
user: root id: 0
user: root id: 0

#镜像变得超级小了
^Croot@node1:~/cks/image_footprint# docker images |grep app
app                                                                            latest              3a81c4f3f3dc        16 hours ago        7.75MB

```
## 3. 安全加固
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3c329df807aacea4714b62aafe129a7e.png)
配置Dockerfile

```bash
# 1. 修改镜像版本
# build container stage 1
FROM ubuntu
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y golang-go
COPY app.go .
RUN CGO_ENABLED=0 go build app.go

# app container stage 2
FROM alpine:3.11.6
COPY --from=0 /app .
CMD ["./app"]
```

```bash
# 2. 非roo用户执行
# build container stage 1
FROM ubuntu:20.04
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y golang-go=2:1.13~1ubuntu2
COPY app.go .
RUN pwd
RUN CGO_ENABLED=0 go build app.go

# app container stage 2
FROM alpine:3.12.0
RUN addgroup -S appgroup && adduser -S appuser -G appgroup -h /home/appuser
COPY --from=0 /app /home/appuser/
USER appuser
CMD ["/home/appuser/app"]
```

```bash
# 3.配置只读文件
# build container stage 1
FROM ubuntu:20.04
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y golang-go=2:1.13~1ubuntu2
COPY app.go .
RUN pwd
RUN CGO_ENABLED=0 go build app.go

# app container stage 2
FROM alpine:3.12.0
RUN chmod a-w /etc
RUN addgroup -S appgroup && adduser -S appuser -G appgroup -h /home/appuser
COPY --from=0 /app /home/appuser/
USER appuser
CMD ["/home/appuser/app"]
```

```bash
# 4. 禁用shell相关命令
# build container stage 1
FROM ubuntu:20.04
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y golang-go=2:1.13~1ubuntu2
COPY app.go .
RUN pwd
RUN CGO_ENABLED=0 go build app.go

# app container stage 2
FROM alpine:3.12.0
RUN addgroup -S appgroup && adduser -S appuser -G appgroup -h /home/appuser
RUN rm -rf /bin/*
COPY --from=0 /app /home/appuser/
USER appuser
CMD ["/home/appuser/app"]
```


更多细节链接: [docker build与Dockerfile](https://ghostwritten.blog.csdn.net/article/details/107517710)
