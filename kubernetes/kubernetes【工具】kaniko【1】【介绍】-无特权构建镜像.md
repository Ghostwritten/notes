

----

 - [kubernetes【工具】kaniko【1】-无特权构建镜像](https://ghostwritten.blog.csdn.net/article/details/121659254)
 - [kubernetes【工具】kaniko【2】-demo](https://ghostwritten.blog.csdn.net/article/details/121781164)
 - [云原生圣经](https://blog.csdn.net/xixihahalelehehe/article/details/108562082?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163897659016780264044987%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=163897659016780264044987&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-2-108562082.pc_v2_rank_blog_default&utm_term=%E4%BA%91%E5%8E%9F%E7%94%9F&spm=1018.2226.3001.4450)
 

---

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8d1f1794faebdf37b0b3701b4501b51d.png)
## 什么是kaniko
[kaniko](https://github.com/GoogleContainerTools/kaniko#--single-snapshot) 是一种在容器或 [Kubernetes](https://kubernetes.io/) 集群内从 Dockerfile 构建容器镜像的工具。

kaniko 不依赖于 [Docker](https://www.docker.com/) 守护进程，而是完全在用户空间中执行 Dockerfile 中的每个命令。这使得在无法轻松或安全地运行 Docker 守护程序的环境中构建容器镜像成为可能，例如标准的 Kubernetes 集群。

镜像：`gcr.io/kaniko-project/executor`. 我们不建议在另一个映像中运行 kaniko 执行程序二进制文件，因为它可能不起作用。

##  kaniko是如何工作的

 - 1.读取指定的`Dockerfile`。
 - 2.将基本映像（在FROM指令中指定）提取到容器文件系统中。
 - 3.在独立的Dockerfile中分别运行每个命令。
 - 4.每次运行后都会对用户空间文件系统的做快照。
 - 5.每次运行时，将快照层附加到基础层。

##  工作原理
kaniko作为一个容器镜像运行，它接受三个参数：一个 Dockerfile ，一个构建上下文以及将镜像推送到的注册表。它在执行程序镜像中提取基本镜像的文件系统。然后，在Dockerfile中执行任何命令，快照用户空间中的文件系统。Kaniko在每个命令后都会将一层已更改的文件附加到基本镜像。最后，执行程序将新镜像推送到指定的注册表。由于Kaniko在执行程序镜像的用户空间中完全执行了这些操作，因此它完全避免了在用户计算机上需要任何特权访问。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2f47c4a6e6ee26ca174c770ca51709e7.png)




##  已知的问题

 - kaniko 不支持构建 Windows 容器。
 - 不支持在官方 kaniko 镜像以外的任何 Docker 镜像中运行 kaniko（即 YMMV）。 这包括将 kaniko 可执行文件从官方映像复制到另一个映像。
 - kaniko 不支持 v1 Registry API ( [Registry v1 API Deprecation](https://engineering.docker.com/2019/03/registry-v1-api-deprecation/) )

##  kaniko 构建上下文
kaniko 的构建上下文与您将发送 Docker 守护程序以进行映像构建的构建上下文非常相似；它代表一个包含 Dockerfile 的目录，kaniko 将使用该目录构建您的映像。例如，COPY Dockerfile 中的命令应该引用构建上下文中的文件。

您需要将构建上下文存储在 kaniko 可以访问的地方。目前，kaniko 支持以下存储解决方案：

 - GCS Bucket
 - S3 Bucket
 - Azure Blob Storage
 - Local Directory
 - Local Tar
 - Standard Input
 - Git Repository

> 关于 `Local Directory`的注意事项：此选项是指 kaniko 容器内的目录。如果您希望使用此选项，则需要在构建上下文中将其作为目录挂载到容器中。
> 
> 关于本地 Tar 的注意事项：此选项指的是 `kaniko` 容器中的 `tar gz`文件。如果您希望使用此选项，则需要在构建上下文中将其作为文件挂载到容器中。


关于标准输入的注意事项：`kaniko 允许的唯一标准输入是.tar.gz格式。`

如果使用 GCS 或 S3 存储桶，您首先需要创建构建上下文的压缩 tar 并将其上传到您的存储桶。运行后，kaniko 将在开始映像构建之前下载并解压构建上下文的压缩 tar。

要创建压缩的 tar，您可以运行：

```bash
tar -C <path to build context> -zcvf context.tar.gz .
```
然后，将压缩的 tar 复制到您的存储桶中。例如，我们可以使用 `gsutil` 将压缩的 tar 复制到 GCS 存储桶：

```bash
gsutil cp context.tar.gz gs://<bucket name>
```
运行 `kaniko` 时，使用`--context`带有适当前缀的标志来指定构建上下文的位置：
| Source             | Prefix                                                                | Example                                                                     |
|--------------------|-----------------------------------------------------------------------|-----------------------------------------------------------------------------|
| Local Directory    | dir://[path to a directory in the kaniko container]                   | dir:///workspace                                                            |
| Local Tar Gz       | tar://[path to a .tar.gz in the kaniko container]                     | tar://path/to/context.tar.gz                                                |
| Standard Input     | tar://[stdin]                                                         | tar://stdin                                                                 |
| GCS Bucket         | gs://[bucket name]/[path to .tar.gz]                                  | gs://kaniko-bucket/path/to/context.tar.gz                                   |
| S3 Bucket          | s3://[bucket name]/[path to .tar.gz]                                  | s3://kaniko-bucket/path/to/context.tar.gz                                   |
| Azure Blob Storage | https://[account].[azureblobhostsuffix]/[container]/[path to .tar.gz] | https://myaccount.blob.core.windows.net/container/path/to/context.tar.gz    |
| Git Repository     | git://[repository url][#reference][#commit-id]                        | git://github.com/acme/myproject.git#refs/heads/mybranch#<desired-commit-id> |


##  标准输入
如果运行 `kaniko` 并使用标准输入构建上下文，则需要添加 `docker` 或 `kubernetes -i`, `--interactive`标志。运行后，`kaniko` 将从中获取数据`STDIN`并将构建上下文创建为压缩的 `tar`。然后它会在开始镜像构建之前解压构建上下文的压缩 tar。如果在交互式运行期间没有数据通过管道传输，您将需要通过按 自己发送 `EOF` 信号`Ctrl+D`。

如何`.tar.gz`使用`docker` 以交互方式使用标准输入数据运行 `kaniko` 的完整示例：

```bash
echo -e 'FROM alpine \nRUN echo "created from standard input"' > Dockerfile | tar -cf - Dockerfile | gzip -9 | docker run \
  --interactive -v $(pwd):/workspace gcr.io/kaniko-project/executor:latest \
  --context tar://stdin \
  --destination=<gcr.io/$project/$image:$tag>
```
如何使用`.tar.gz`标准输入数据交互运行 `kaniko` 的完整示例，使用 `Kubernetes` 命令行与临时容器和完全无 `docker`：

```bash
echo -e 'FROM alpine \nRUN echo "created from standard input"' > Dockerfile | tar -cf - Dockerfile | gzip -9 | kubectl run kaniko \
--rm --stdin=true \
--image=gcr.io/kaniko-project/executor:latest --restart=Never \
--overrides='{
  "apiVersion": "v1",
  "spec": {
    "containers": [
      {
        "name": "kaniko",
        "image": "gcr.io/kaniko-project/executor:latest",
        "stdin": true,
        "stdinOnce": true,
        "args": [
          "--dockerfile=Dockerfile",
          "--context=tar://stdin",
          "--destination=gcr.io/my-repo/my-image"
        ],
        "volumeMounts": [
          {
            "name": "cabundle",
            "mountPath": "/kaniko/ssl/certs/"
          },
          {
            "name": "docker-config",
            "mountPath": "/kaniko/.docker/"
          }
        ]
      }
    ],
    "volumes": [
      {
        "name": "cabundle",
        "configMap": {
          "name": "cabundle"
        }
      },
      {
        "name": "docker-config",
        "configMap": {
          "name": "docker-config"
        }
      }
    ]
  }
}'
```
##  部署运行
### kubernetes中运行kaniko
Requirements:

 - Standard Kubernetes cluster (e.g. using [GKE](https://cloud.google.com/kubernetes-engine/))
 - Kubernetes Secret
 - A build context
 - Kubernetes secret

####  Kubernetes secret
要在 Kubernetes 集群中运行 kaniko，您需要一个标准的运行 Kubernetes 集群和一个 Kubernetes  secret，其中包含推送最终映像所需的身份验证。

推送至指定远端镜像仓库须要`credential`的支持，因此须要将`credential`以secret的方式挂载到`/kaniko/.docker/`这个目录下，文件名称为`kaniko_config.json`，内容以下:

```bash
{   
    "auths": {
        "ghost.harbor.com": {
            "auth": "YWRtaW46SGFyYm9yMTIzNDUK"
       }
    }
    
}
```
`ghost.harbor.com`是我的仓库名
`YWRtaW46SGFyYm9yMTIzNDUK`是通过`registry`用户名与密码以下命令获取：

```bash
$ echo "admin:Harbor12345"|base64
YWRtaW46SGFyYm9yMTIzNDUK
```
运行创建secret

```bash
$ kubectl create secret generic kaniko-secret --from-file=kaniko_config.json
secret/kaniko-secret created

$ kubectl  get secret kaniko-secret
NAME            TYPE     DATA   AGE
kaniko-secret   Opaque   1      23s
```
###  编写一个demo
#### 编写demo程序
```bash
$ cd /root/python-docker
$ pip3 install Flask
$ pip3 freeze | grep Flask >> requirements.txt

$ vim app.py
 #!/usr/bin/python3
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, Docker!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```
#### 本地测试

```bash
$ python3 app.py 
 * Serving Flask app 'app' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on all addresses.
   WARNING: This is a development server. Do not use it in a production deployment.
 * Running on http://192.168.211.71:8080/ (Press CTRL+C to quit)

$ curl http://192.168.211.71:8080
Hello, Docker!
```
#### 创建一个 `Dockerfile`

```bash
$ cat Dockerfile 
FROM python:3.8-slim-buster

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

CMD [ "python3", "app.py"]
```
demo目录结果：

```bash
python-docker
|____ app.py
|____ requirements.txt
|____ Dockerfile
```
#### 构建镜像

```bash
$ docker build --tag python-docker .
#如果pip安装不了,尝试以下

$ docker build --net=host --tag python-docker .
Sending build context to Docker daemon  5.632kB
Step 1/6 : FROM python:3.8-slim-buster
 ---> 5be55fb2aad1
Step 2/6 : WORKDIR /app
 ---> Using cache
 ---> bde6cbf0c2d3
Step 3/6 : COPY requirements.txt requirements.txt
 ---> Using cache
 ---> 94ec6d1bdf8b
Step 4/6 : RUN pip3 install -r requirements.txt
 ---> Using cache
 ---> 7683379b1dd5
Step 5/6 : COPY . .
 ---> Using cache
 ---> d7cd92473eff
Step 6/6 : CMD [ "python3", "app.py"]
 ---> Using cache
 ---> ac94a1193e2f
Successfully built ac94a1193e2f
Successfully tagged python-docker:latest
```

```bash
$ docker images |grep python
python-docker                                                                          latest            ac94a1193e2f   15 minutes ago   125MB
```
#### 运行容器测试

```bash
$ docker run -tid --net=host --name hello python-docker:latest

$ curl http://192.168.211.71:8080
Hello, Docker!
```
###  部署minikube

 - [Ubuntu 18.04 通过 Minikube 安装 Kubernetes v1.20](https://ghostwritten.blog.csdn.net/article/details/113527867)

### 编排kaniko pod

```bash
apiVersion: v1
kind: Pod
metadata:
  name: kaniko
spec:
  containers:
 - name: kaniko
#    image: gcr.io/kaniko-project/executor:latest
    image: registry.aliyuncs.com/kaniko-project/executor:latest
    args:
    - "--dockerfile=/python-docker/Dockerfile"
    - "--context=gs:/python-docker/"
    - "--destination=ghost.harbor.com/library/python-docker:v1.0>"
    volumeMounts:
    - name: kaniko-secret
      mountPath: /secret
    - name: python-docker
      mountPath: /python-docker
      subPath: Dockerfile    
    env:
    - name: GOOGLE_APPLICATION_CREDENTIALS
      value: /secret/kaniko-secret.json
  restartPolicy: Never
  volumes:
 - name: kaniko-secret
    secret:
      secretName: kaniko-secret
 - name: python-docker
   hostPath: 
     path: /root/python-docker
```
参数定义：
 - `--dockerfile`   指定Dockerfile
 - `--context`   定义位置获取编排位置，即上下文
 - `--destination`  远端镜像仓库
 - `--insecure=true`   仓库为私有http仓库
 - `--skip-tls-verify=true`  跳过tls验证






参考链接：
[https://coderedirect.com/questions/134536/docker-unauthorized-authentication-required-upon-push-with-successful-login](https://coderedirect.com/questions/134536/docker-unauthorized-authentication-required-upon-push-with-successful-login)
