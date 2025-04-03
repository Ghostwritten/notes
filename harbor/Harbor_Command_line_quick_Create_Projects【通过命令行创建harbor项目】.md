![](https://i-blog.csdnimg.cn/blog_migrate/9cd3682ffc03ea35edb1408466595d8f.png)


## 1. harbor 1.x版本

因为客户vpn没有开放服务器端口访问权限，现在只能通过命令行方式创建harbor项目，然后将项目推送到harbor仓库。

这个是json文件： `{"project_name": "k8s","metadata": {"public": "true"}}` ，其中k8s可以根据自己项目的名称来定义。

这个是执行命令

```bash
curl -u "admin:Harbor12345" -X POST -H "Content-Type: application/json" "192.168.131.73:5000/api/projects" -d @createproject.json  
```

创建harbor镜像库

首先先创建一个json文件内容，然后执行下面这行命令。

 

批量执行脚本：

```bash
#!/bin/bash
for line in `cat $2`
do
 curl -u 'admin:Harbor12345' -X 'POST' -H 'Content-Type: application/json' ''$1'/api/projects' -d '{"project_name": "'$line'","metadata": {"public": "true"}}'
done
```

## 2. harbor2.x版本

```bash
curl -u "admin:Harbor12345" -X POST -H "Content-Type: application/json" "192.168.131.73:5000/api/v2.0/projects" -d '{"project_name": "k8s","metadata": {"public": "true"}}'

```

### 2.1 脚本1

```bash
#!/bin/bash

harbor_url=$1


for line in `cat projects.txt`
do
 curl -u 'admin:Harbor12345' -X 'POST' -H 'Content-Type: application/json' "$1/api/projects" -d '{"project_name": "'$line'","metadata": {"public": "true"}}'
done
```

### 2.2 脚本2

```bash
$ vim  create_harbor_projects.sh 
#!/bin/bash
 
url="http://xxxxxx"
user="admin"
passwd="xxx"
 
 
harbor_projects=(
calico \
coredns \
csiplugin \
elastic \
fluent \
grafana \
istio \
jaegertracing \
jenkins \
jimmidyson \
joosthofman \
kubeedge \
kubesphere \
kubespheredev \
library \
minio \
mirrorgooglecontainers \
nginxdemos \
openebs \
osixia \
prom \
thanosio \
weaveworks \
)
 
for project in ${harbor_projects[@]} ; do curl -u "${user}:${passwd}" -X POST -H "Content-Type: application/json" "${url}/api/v2.0/projects" -d "{\"project_name\": \"${project}\", \"metadata\": {\"public\": \"true\"}, \"storage_limit\": -1}"; done
```

