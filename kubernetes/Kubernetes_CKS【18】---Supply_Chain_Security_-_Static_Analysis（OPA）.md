

---
## 1. 介绍
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/71bf125138d525fe786fd6c1b11238bf.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a9ede8e538d2877607284b32d35099ad.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/99dbd9f1ec3de19999b8b62c8f0d80f7.png)

**static analysis CI/CD**
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/963a8960accf228da7779a25722f2154.png)
**manual check**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cb8e41cf190fa73d5e2eee1337d286c3.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ec24fd81402696fee07c42f664867297.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0619e9f272337df7f7d7b333c6828f39.png)
##  2. Kubesec
[Kubesec官网/](https://kubesec.io/)
github：[https://github.com/shyiko/kubesec](https://github.com/shyiko/kubesec)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5dfd9c5d268853efecda91de5f607941.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7494444e32339f9fb9f8f746d96bdf1d.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/61e246604af32b62eb49cdcb37d89fb2.png)
退出2格
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7412695bb793019bfea1f1e468c6200b.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/47a185617ce61e3cf92af2ce1b0c3ca9.png)
##  3. OPA Conftest
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c15b8c3436d702a81b7a118b1cac98ad.png)

## 4. OPA Conftest for K8s YAML

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ee5e4ed972a57d891d083723d42fe779.png)

```bash
root@node1:~/cks/static-analysis/conftest/kubernetes# ls
deploy.yaml  policy  run.sh


root@node1:~/cks/static-analysis/conftest/kubernetes# cat deploy.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: test
  name: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: test
    spec:
      containers:
        - image: httpd
          name: httpd
          resources: {}
status: {}


root@node1:~/cks/static-analysis/conftest/kubernetes# cat policy/deployment.rego 
# from https://www.conftest.dev
package main

deny[msg] {
  input.kind = "Deployment"
  not input.spec.template.spec.securityContext.runAsNonRoot = true
  msg = "Containers must not run as root"
}

deny[msg] {
  input.kind = "Deployment"
  not input.spec.selector.matchLabels.app
  msg = "Containers must provide app label for pod selectors"
}



root@node1:~/cks/static-analysis/conftest/kubernetes# cat run.sh 
docker run --rm -v $(pwd):/project openpolicyagent/conftest test deploy.yaml

#报错了
root@node1:~/cks/static-analysis/conftest/kubernetes# bash run.sh 
Unable to find image 'openpolicyagent/conftest:latest' locally
latest: Pulling from openpolicyagent/conftest
540db60ca938: Already exists 
c348a0913279: Pull complete 
634fd801ca52: Pull complete 
77bd1e540306: Pull complete 
b7a9709315e9: Pull complete 
Digest: sha256:209991f4471b8a3cfa3726eca9272d299ca28f9298ca19014d7ec2e6aefe07c8
Status: Downloaded newer image for openpolicyagent/conftest:latest
FAIL - deploy.yaml - main - Containers must not run as root

2 tests, 1 passed, 0 warnings, 1 failure, 0 exceptions


# 修改deploy.yaml文件
root@node1:~/cks/static-analysis/conftest/kubernetes# vim deploy.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: test
  name: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: test
    spec:
      securityContext:  #添加此行
        runAsNonRoot: true  #添加此行
      containers:
        - image: httpd
          name: httpd
          resources: {}
status: {}


root@node1:~/cks/static-analysis/conftest/kubernetes# bash run.sh 

2 tests, 2 passed, 0 warnings, 0 failures, 0 exceptions


#修改spec.selector.mathLabels.label
root@node1:~/cks/static-analysis/conftest/kubernetes# cat deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: test
  name: test
spec:
  replicas: 1
  selector:
    matchLabels:
      test: test  #修改此行
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: test
    spec:
      securityContext: 
        runAsNonRoot: true
      containers:
        - image: httpd
          name: httpd
          resources: {}
status: {}


root@node1:~/cks/static-analysis/conftest/kubernetes# bash run.sh 
FAIL - deploy.yaml - main - Containers must provide app label for pod selectors

2 tests, 1 passed, 0 warnings, 1 failure, 0 exceptions

```


## 5. OPA Conftest for Dockerfile
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/72281a194faeb405660fb8149778819f.png)

```bash
root@node1:~/cks/static-analysis/conftest/docker# ls 
Dockerfile  policy/     run.sh      
root@node1:~/cks/static-analysis/conftest/docker# ls policy/
base.rego      commands.rego  
root@node1:~/cks/static-analysis/conftest/docker# cat Dockerfile 
FROM ubuntu
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y golang-go
COPY app.go .
RUN go build app.go
CMD ["./app"]

root@node1:~/cks/static-analysis/conftest/docker# cat policy/base.rego 
# from https://www.conftest.dev
package main

denylist = [
  "ubuntu"
]

deny[msg] {
  input[i].Cmd == "from"
  val := input[i].Value
  contains(val[i], denylist[_])

  msg = sprintf("unallowed image found %s", [val])
}
root@node1:~/cks/static-analysis/conftest/docker# cat policy/commands.rego 
# from https://www.conftest.dev

package commands

denylist = [
  "apk",
  "apt",
  "pip",
  "curl",
  "wget",
]

deny[msg] {
  input[i].Cmd == "run"
  val := input[i].Value
  contains(val[_], denylist[_])

  msg = sprintf("unallowed commands found %s", [val])
}
root@node1:~/cks/static-analysis/conftest/docker# cat run.sh 
docker run --rm -v $(pwd):/project openpolicyagent/conftest test Dockerfile --all-namespaces

#test都未通过
root@node1:~/cks/static-analysis/conftest/docker# bash run.sh 
FAIL - Dockerfile - commands - unallowed commands found ["apt-get update && apt-get install -y golang-go"]
FAIL - Dockerfile - main - unallowed image found ["ubuntu"]

2 tests, 0 passed, 0 warnings, 2 failures, 0 exceptions


#修改镜像
root@node1:~/cks/static-analysis/conftest/docker# cat Dockerfile 
FROM apline  #修改此行
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y golang-go
COPY app.go .
RUN go build app.go
CMD ["./app"]

#通过一个
root@node1:~/cks/static-analysis/conftest/docker# bash run.sh 
FAIL - Dockerfile - commands - unallowed commands found ["apt-get update && apt-get install -y golang-go"]

2 tests, 1 passed, 0 warnings, 1 failure, 0 exceptions


#修改policy/commonds.rego删除apt
root@node1:~/cks/static-analysis/conftest/docker# cat policy/commands.rego
# from https://www.conftest.dev

package commands

denylist = [
  "apk",
  "pip",
  "curl",
  "wget",
]

deny[msg] {
  input[i].Cmd == "run"
  val := input[i].Value
  contains(val[_], denylist[_])

  msg = sprintf("unallowed commands found %s", [val])
}


#全部通过
root@node1:~/cks/static-analysis/conftest/docker# bash run.sh 

2 tests, 2 passed, 0 warnings, 0 failures, 0 exceptions

```

