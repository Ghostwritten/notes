

---
## 1. 介绍
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210521103027758.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210521103115435.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210521103209601.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

**static analysis CI/CD**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210521103350754.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
**manual check**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210521103521209.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210521103617298.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210521103632553.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
##  2. Kubesec
[Kubesec官网/](https://kubesec.io/)
github：[https://github.com/shyiko/kubesec](https://github.com/shyiko/kubesec)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210521103718806.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210521104443432.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210521104423263.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
退出2格
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210521104458954.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210521104510199.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
##  3. OPA Conftest
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210521104642528.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

## 4. OPA Conftest for K8s YAML

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210521104726938.png)

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
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210521110323680.png)

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

