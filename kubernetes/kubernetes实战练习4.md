## 将service从源代码部署到 Kubernetes
您是一名开发人员，想尝试 Kubernetes 吗？

在本教程中，我们将向您展示从源代码到正在运行的 Kubernetes 集群将服务部署到 Kubernetes 的基础知识。

###  容器
Kubernetes 不会直接运行您的源代码。相反，您将容器交给 Kubernetes。

容器包含 

 - 1源代码的编译版本
 - 2运行源代码所需的任何/所有运行时依赖项。

在本教程中，我们将使用 Docker 作为我们的容器格式。我们已经用 Python 创建了一个简单的 Web 应用程序，`hello-webapp`. 要将 `webapp` 打包为 Docker 容器，我们创建了一个`Dockerfile`.

我们Dockerfile为您创建了一个，因此只需键入以下命令即可查看其内容：

```bash
controlplane $ cd hello-webapp
controlplane $ cat Dockerfile
# Run server
FROM alpine:3.5
RUN apk add --no-cache python py2-pip py2-gevent
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . /app
WORKDIR /app
EXPOSE 8080
ENTRYPOINT ["python"]
CMD ["app.py"]
controlplane $ ls
Dockerfile  README.md  deployment.yaml  requirements.txt  uuid.txt
LICENSE     app.py     k8s              service.yaml
controlplane $ docker build -t hello-webapp:v1 .
Sending build context to Docker daemon  135.7kB
Step 1/9 : FROM alpine:3.5
3.5: Pulling from library/alpine
8cae0e1ac61c: Already exists 
Digest: sha256:66952b313e51c3bd1987d7c4ddf5dba9bc0fb6e524eed2448fa660246b3e76ec
Status: Downloaded newer image for alpine:3.5
 ---> f80194ae2e0c
Step 2/9 : RUN apk add --no-cache python py2-pip py2-gevent
 ---> Running in 83294f8e9223
fetch http://dl-cdn.alpinelinux.org/alpine/v3.5/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.5/community/x86_64/APKINDEX.tar.gz
(1/14) Installing libbz2 (1.0.6-r5)
(2/14) Installing expat (2.2.0-r1)
(3/14) Installing libffi (3.2.1-r2)
(4/14) Installing gdbm (1.12-r0)
(5/14) Installing ncurses-terminfo-base (6.0_p20171125-r1)
(6/14) Installing ncurses-terminfo (6.0_p20171125-r1)
(7/14) Installing ncurses-libs (6.0_p20171125-r1)
(8/14) Installing readline (6.3.008-r4)
(9/14) Installing sqlite-libs (3.15.2-r2)
(10/14) Installing python2 (2.7.15-r0)
(11/14) Installing py2-greenlet (0.4.10-r3)
(12/14) Installing py2-gevent (1.1.2-r0)
(13/14) Installing py-setuptools (29.0.1-r0)
(14/14) Installing py2-pip (9.0.0-r1)
Executing busybox-1.25.1-r2.trigger
OK: 63 MiB in 25 packages
Removing intermediate container 83294f8e9223
 ---> 4737825cb785
Step 3/9 : COPY requirements.txt .
 ---> 14704e6183c2
Step 4/9 : RUN pip install -r requirements.txt
 ---> Running in 4b4798d29aa9
Collecting flask (from -r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/e8/6d/994208daa354f68fd89a34a8bafbeaab26fda84e7af1e35bdaed02b667e6/Flask-1.1.4-py2.py3-none-any.whl (94kB)
Collecting requests (from -r requirements.txt (line 2))
  Downloading https://files.pythonhosted.org/packages/92/96/144f70b972a9c0eabbd4391ef93ccd49d0f2747f4f6a2a2738e99e5adc65/requests-2.26.0-py2.py3-none-any.whl (62kB)
Collecting Jinja2<3.0,>=2.10.1 (from flask->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/7e/c2/1eece8c95ddbc9b1aeb64f5783a9e07a286de42191b7204d67b7496ddf35/Jinja2-2.11.3-py2.py3-none-any.whl (125kB)
Collecting Werkzeug<2.0,>=0.15 (from flask->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/cc/94/5f7079a0e00bd6863ef8f1da638721e9da21e5bacee597595b318f71d62e/Werkzeug-1.0.1-py2.py3-none-any.whl (298kB)
Collecting click<8.0,>=5.1 (from flask->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/d2/3d/fa76db83bf75c4f8d338c2fd15c8d33fdd7ad23a9b5e57eb6c5de26b430e/click-7.1.2-py2.py3-none-any.whl (82kB)
Collecting itsdangerous<2.0,>=0.24 (from flask->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/76/ae/44b03b253d6fade317f32c24d100b3b35c2239807046a4c953c7b89fa49e/itsdangerous-1.1.0-py2.py3-none-any.whl
Collecting urllib3<1.27,>=1.21.1 (from requests->-r requirements.txt (line 2))
  Downloading https://files.pythonhosted.org/packages/af/f4/524415c0744552cce7d8bf3669af78e8a069514405ea4fcbd0cc44733744/urllib3-1.26.7-py2.py3-none-any.whl (138kB)
Collecting idna<3,>=2.5; python_version < "3" (from requests->-r requirements.txt (line 2))
  Downloading https://files.pythonhosted.org/packages/a2/38/928ddce2273eaa564f6f50de919327bf3a00f091b5baba8dfa9460f3a8a8/idna-2.10-py2.py3-none-any.whl (58kB)
Collecting certifi>=2017.4.17 (from requests->-r requirements.txt (line 2))
  Downloading https://files.pythonhosted.org/packages/37/45/946c02767aabb873146011e665728b680884cd8fe70dde973c640e45b775/certifi-2021.10.8-py2.py3-none-any.whl (149kB)
Collecting chardet<5,>=3.0.2; python_version < "3" (from requests->-r requirements.txt (line 2))
  Downloading https://files.pythonhosted.org/packages/19/c7/fa589626997dd07bd87d9269342ccb74b1720384a4d739a1872bd84fbe68/chardet-4.0.0-py2.py3-none-any.whl (178kB)
Collecting MarkupSafe>=0.23 (from Jinja2<3.0,>=2.10.1->flask->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/b9/2e/64db92e53b86efccfaea71321f597fa2e1b2bd3853d8ce658568f7a13094/MarkupSafe-1.1.1.tar.gz
Installing collected packages: MarkupSafe, Jinja2, Werkzeug, click, itsdangerous, flask, urllib3, idna, certifi, chardet, requests
  Running setup.py install for MarkupSafe: started
    Running setup.py install for MarkupSafe: finished with status 'done'
Successfully installed Jinja2-2.11.3 MarkupSafe-1.1.1 Werkzeug-1.0.1 certifi-2021.10.8 chardet-4.0.0 click-7.1.2 flask-1.1.4 idna-2.10 itsdangerous-1.1.0 requests-2.26.0 urllib3-1.26.7
You are using pip version 9.0.0, however version 21.3.1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
Removing intermediate container 4b4798d29aa9
 ---> 99831ef71094
Step 5/9 : COPY . /app
 ---> 2d6006fe0c32
Step 6/9 : WORKDIR /app
 ---> Running in dd5a0ef82c6a
Removing intermediate container dd5a0ef82c6a
 ---> 2f02dd608139
Step 7/9 : EXPOSE 8080
 ---> Running in 41dd76e23535
Removing intermediate container 41dd76e23535
 ---> 5e8a803e2b31
Step 8/9 : ENTRYPOINT ["python"]
 ---> Running in 1445b35d4c4b
Removing intermediate container 1445b35d4c4b
 ---> 43808464f62f
Step 9/9 : CMD ["app.py"]
 ---> Running in e24f5500e614
Removing intermediate container e24f5500e614
 ---> 761f1945fd23
Successfully built 761f1945fd23
Successfully tagged hello-webapp:v1
```

