![](https://i-blog.csdnimg.cn/direct/601cbfb490b44fb29e2137eb3dd695ab.jpeg#pic_center)





**👋 这篇文章内容：实现shell 脚本批量清理docker registry的镜像。**

🔔：你可以在这里阅读：[https://github.com/Ghostwritten/registry-clean.git](https://github.com/Ghostwritten/registry-clean.git)
# 1. 安装 docker 

- [docker 安装](https://blog.csdn.net/xixihahalelehehe/article/details/104293170)


# 2. 配置 docker

```bash
$ cat /etc/docker/daemon.json 
{
   "exec-opts": ["native.cgroupdriver=systemd"],
   "insecure-registries": ["registry.ghostwritten.com"],
   "live-restore": true,
   "log-driver": "json-file",
   "log-opts": {
     "max-size":  "100m",
     "max-file": "5"
    }
 }

# 配置代理
$ cat /usr/lib/systemd/system/docker.service.d/proxy.conf 
[Service]
Environment="HTTP_PROXY=http://192.168.21.101:7890"
Environment="HTTPS_PROXY=http://192.168.21.101:7890"
Environment="NO_PROXY=localhost,127.0.0.1,.coding.net,.tencentyun.com,.myqcloud.com,*.bsgchina.com"
```

```bash
$ systemctl daemon-reload && systemctl restart docker
```

# 4. 配置域名解析

服务端(192.168.21.2)配置
```bash
$ cat  /etc/unbound/unbound.conf
...
    local-data: "registry.ghostwritten.com A 192.168.21.25"
    local-data-ptr: "192.168.21.25 registry.ghostwritten.com"
....

$ systemctl restart unbound
```

客户端

```bash
$ cat /etc/resolv.conf 
# Generated by NetworkManager
nameserver 192.168.21.2
```

# 5. 部署 registry

测试删除镜像

```bash
$ docker run -tid --restart=always --name registry -p 80:5000 -e REGISTRY_COMPATIBILITY_SCHEMA1_ENABLED=true -e REGISTRY_STORAGE_DELETE_ENABLED=true -v /data/registry:/var/lib/registry registry:latest

```


# 6. Registry API 管理

```bash

$ curl  -q -s   'http://registry.ghostwritten.com/v2/registry/tags/list' | jq .
{
  "name": "registry",
  "tags": [
    "latest"
  ]
}

$ curl -I -X GET  'http://registry.ghostwritten.com/v2/registry/manifests/latest'
HTTP/1.1 200 OK
Content-Length: 6863
Content-Type: application/vnd.docker.distribution.manifest.v1+prettyjws
Docker-Content-Digest: sha256:f538bbbf8ff45e9872789dfcfbbcd48a44a9c87d21324595efb4df2e6b666c8c
Docker-Distribution-Api-Version: registry/2.0
Etag: "sha256:f538bbbf8ff45e9872789dfcfbbcd48a44a9c87d21324595efb4df2e6b666c8c"
X-Content-Type-Options: nosniff
Date: Wed, 18 Sep 2024 09:02:51 GMT

通过API直接匹配标签删除不掉镜像
$ curl -I -X DELETE  http://registry.ghostwritten.com/v2/registry/manifests/latest
HTTP/1.1 400 Bad Request
Content-Type: application/json; charset=utf-8
Docker-Distribution-Api-Version: registry/2.0
X-Content-Type-Options: nosniff
Date: Wed, 18 Sep 2024 08:58:49 GMT
Content-Length: 98

#获取ID
$ docker inspect registry.ghostwritten.com/registry:latest | jq -r '.[0].RepoDigests[]' |grep registry.ghostwritten.com | awk -F '@' '{print $2}'
sha256:4d2514be50520c34f3b207fc03dd734a9b72403624d8572f8c40e9185575e453


$ curl -I -X GET  'http://registry.ghostwritten.com/v2/registry/manifests/sha256:4d2514be50520c34f3b207fc03dd734a9b72403624d8572f8c40e9185575e453'
HTTP/1.1 200 OK
Content-Length: 1363
Content-Type: application/vnd.docker.distribution.manifest.v2+json
Docker-Content-Digest: sha256:4d2514be50520c34f3b207fc03dd734a9b72403624d8572f8c40e9185575e453
Docker-Distribution-Api-Version: registry/2.0
Etag: "sha256:4d2514be50520c34f3b207fc03dd734a9b72403624d8572f8c40e9185575e453"
X-Content-Type-Options: nosniff
Date: Wed, 18 Sep 2024 09:12:51 GMT



$ curl -I -X DELETE  'http://registry.ghostwritten.com/v2/registry/manifests/sha256:4d2514be50520c34f3b207fc03dd734a9b72403624d
8572f8c40e9185575e453'
HTTP/1.1 202 Accepted
Docker-Distribution-Api-Version: registry/2.0
X-Content-Type-Options: nosniff
Date: Wed, 18 Sep 2024 09:12:59 GMT
Content-Length: 0

$ curl   -s   'http://registry.ghostwritten.com/v2/_catalog' | jq .
{
  "repositories": [
    "library/busybox",
    "registry"
  ]
}

$ curl  -q -s   'http://registry.ghostwritten.com/v2/registry/tags/list' | jq .
{
  "name": "registry",
  "tags": null
}

$ rm -rf  /data/registry/docker/registry/v2/repositories/registry
$ curl -q -s http://registry.ghostwritten.com/v2/_catalog | jq .
{
  "repositories": [
    "library/busybox"
  ]
}

```
推送镜像

```bash
$ docker push registry.ghostwritten.com/registry:latest
The push refers to repository [registry.ghostwritten.com/registry]
3715a40eaff0: Layer already exists 
e48d56ef719d: Layer already exists 
1670fc7fdd22: Layer already exists 
f3965cc04390: Layer already exists 
d62a02444d39: Layer already exists 
latest: digest: sha256:4d2514be50520c34f3b207fc03dd734a9b72403624d8572f8c40e9185575e453 size: 1363
```

查询镜像标签

```bash
$ curl  -q -s   'http://registry.ghostwritten.com/v2/registry/tags/list' | jq .
{
  "name": "registry",
  "tags": [
    "latest"
  ]
}
```

# 7. 批量清理镜像

首先，这是一个清理的演示，先批量推送镜像入库。未来应对可能存在各类情况。

- 有没有项目名的镜像
- 多个标签的镜像


```bash
$ cat registry-images-push.sh
docker push  registry.ghostwritten.com/demo/nginx:1.26
docker push  registry.ghostwritten.com/demo/nginx:latest
docker push registry.ghostwritten.com/library/busybox:1.36.1
docker push registry.ghostwritten.com/registry:latest
docker push registry.ghostwritten.com/library/busybox:1.35.0
```

推送镜像入库。

```bash
$ sh registry-images-push.sh
```

要删除的镜像我们存放到 registry-images-clean.txt 文件中。我们会检验以下几类删除场景。

- 删除有个多个标签的镜像检验是否会误删除；
- 删除没有项目名的镜像；
- 删除不存在的镜像。

```bash
$ cat registry-images-clean.txt
registry.ghostwritten.com/library/busybox:1.36.1
registry.ghostwritten.com/registry:latest
registry.ghostwritten.com/demo/nginx:1.26
registry.ghostwritten.com/demo/nginx:noexist
```



这个脚本的目的是从Docker注册表中批量删除镜像。在执行脚本之前，必须创建一个名为images.txt的文件，其中应该包含计划删除的映像的名称。例如，文件中的一个条目可能如下所示:registry.demo.com/library/nginx:latest。

```bash
$ cat registry-images-clean.sh 
#!/bin/bash

# Script Name: registry-images-clean.sh
# Author: ghostwritten
# Date: 2024-9-18
# Description: This script is designed to facilitate the batch deletion of images from a Docker registry. Prior to executing the script, you must create a file named images.txt, which should contain the names of the images scheduled for deletion. For example, an entry in the file might look like: registry.demo.com/library/nginx:latest. 

name=`basename $0 .sh`

action=$1


images_list() {


image_repos=`cat registry-images-clean.txt  | awk -F '/' '{print $1}' |  uniq`

for image_repo in $image_repos
do 
 repositories=$(curl -q -s http://${image_repo}/v2/_catalog | jq -r '.repositories[]')

 if [[ -n $repositories ]] ; then
  echo "$image_repo 镜像仓库列表:"

  for repo in $repositories; do

    tags=$(curl -q -s http://${image_repo}/v2/${repo}/tags/list | jq -r '.tags[]')

    if [ -z "$tags" ]; then
        echo "  No tags found"
    else
        for tag in $tags; do
            echo "  $repo:$tag"
        done
    fi
  done
 else
   echo "$image_repo 镜像仓库是空库."
 fi
done

}

images_rm() {

while IFS= read -r line
do
    image_repo=$(echo "$line" | awk -F '/' '{print $1}')
    
    image_project=$(echo "$line" | awk -F '/' '{ if (NF > 2) print $(NF-1) "/"; else print "" }')


    image_name=$(echo "$line" | awk -F '/' '{print $NF}' | awk -F ':' '{print $1}')
    
    image_tag=$(echo "$line" | awk -F '/' '{print $NF}' | awk -F ':' '{print $2}')


    response=$(curl -s "http://${image_repo}/v2/${image_project}${image_name}/tags/list")
    if echo "$response" | jq -e ".tags | index(\"${image_tag}\")" > /dev/null; then
       echo "镜像 ${image_project}${image_name}:${image_tag} 存在，准备删除..."

       source_data=`docker inspect registry | jq -r '.[0].Mounts[] | select(.Type == "bind") | .Source'`
       MANIFEST_DIGEST=`ls ${source_data}/docker/registry/v2/repositories/${image_project}${image_name}/_manifests/tags/${image_tag}/index/sha256/`

       curl -s -I -X DELETE  "http://${image_repo}/v2/${image_project}${image_name}/manifests/sha256:${MANIFEST_DIGEST}" 2>&1 > /dev/null

    
      response=$(curl -s "http://${image_repo}/v2/${image_project}${image_name}/tags/list")
      if ! echo "$response" | jq -e '.tags | length > 0' > /dev/null; then
         rm -rf ${source_data}/docker/registry/v2/repositories/${image_project}${image_name}
      fi
    else
       echo "镜像 ${image_repo}/${image_project}${image_name}:${image_tag} 不存在。"
fi


done < registry-images-clean.txt


docker exec registry  bin/registry garbage-collect /etc/docker/registry/config.yml  >> /dev/null
docker restart registry


}

case $action in
 l|ls)
        images_list
        ;;
 d|rm)
        images_rm
        ;;
 *)
        echo "Usage: $name [ls|rm]"
        exit 1
        ;;
esac
exit 0
```


首先，检查当前registry镜像仓库有哪些镜像。

```bash
$ sh registry-images-clean.sh ls
registry.ghostwritten.com 镜像仓库列表:
  demo/nginx:latest
  demo/nginx:1.26
  library/busybox:1.35.0
  library/busybox:1.36.1
  registry:latest
registest.bsgchina.com 镜像仓库是空库.
```
现在，我们开始删除 registry-images-clean.txt 的镜像。

```bash
$  registry-images-clean.sh rm
镜像 library/busybox:1.36.1 存在，准备删除...
镜像 registry:latest 存在，准备删除...
镜像 demo/nginx:1.26 存在，准备删除...
镜像 registry.ghostwritten.com/demo/nginx:noexist 不存在。
镜像 demo/nginx:noexist 存在，准备删除...
```

执行结束后，再次检查镜像仓库列表。

```bash
sh registry-images-clean.sh ls
registry.ghostwritten.com 镜像仓库列表:
  demo/nginx:latest
  library/busybox:1.35.0
registest.bsgchina.com 镜像仓库是空库.
```

# 8. 其他

- 除了通过API 查看registry镜像列表，你可以[部署一个 registry UI 查看镜像](https://blog.csdn.net/xixihahalelehehe/article/details/107406198)。
- 另外，当你想要迁移一套具有自己镜像介质的镜像仓库。你可以参考这篇[不需要推拉镜像的镜像仓库迁移教程](https://ghostwritten.blog.csdn.net/article/details/136328119)。



参考：

- [Open Container Initiative Distribution Specification](https://github.com/opencontainers/distribution-spec/blob/v1.0.1/spec.md#pull)
- [https://docs.docker.com/registry/#introduction](https://docs.docker.com/registry/#introduction)
- [https://github.com/Ghostwritten/registry-clean.git](https://github.com/Ghostwritten/registry-clean.git)

