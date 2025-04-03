
![](https://i-blog.csdnimg.cn/blog_migrate/68858961c8b9f499d0af73278d31a434.jpeg#pic_center)



## 1. 简介
很多情况下，针对一个项目会有很多镜像需要搬运，打包，解压，打标签，推送入库。该项目将针对多个镜像进行管理操作。方便工程师快速梳理介质。

## 2. 功能

- 批量下载多个镜像，将需要下载的镜像列表放到 $images_list；
- 批量解压多个镜像包，多个目录的镜像包；
- 批量打包 $images_list 的镜像；
- 批量拉取 $images_list 的镜像；
- 批量修改 $images_list 的镜像标签并推送私有仓库；


## 3. 代码
```bash
#!/bin/bash

action=$1
registry_name='harbor.ghostwritten.com'
BASE_DIR="$( dirname "$( readlink -f "${0}" )" )"
project='rancher'

images_list='cert-manager-images.txt'
images_dir="${BASE_DIR}/images"

docker=$(command -v docker || command -v podman || command -v nerdctl)

if [ -z "$docker" ]; then
    echo "docker, podman or nerdctl not found."
    exit 1
fi

if [ "$(basename $docker)" = "nerdctl" ]; then
   docker="/usr/local/bin/nerdctl --insecure-registry"
fi


#对多个项目目录的images目录的tar格式进行解压

images_load() {

output=$(ls $BASE_DIR)

# 将结果按空格分割为数组，遍历每一个对象进行判断
for obj in `ls ${images_dir}`
do
   sudo $docker load -i ${images_dir}/$obj 
done

}


images_load_1() {

output=$(ls $BASE_DIR)

# 将结果按空格分割为数组，遍历每一个对象进行判断
for obj in $output
do
    # 检查对象是否为目录
    if [ -d "${BASE_DIR}/$obj" ]; then
       for image in  `ls -l ${BASE_DIR}/$obj/images/*.tar | awk '{print $NF}'`
       do
         #echo $image
         sudo $docker load -i  $image | tee -a images_load.txt
       done
    fi
done

}

#解压当前目录的tar 包镜像
images_load_2() {

 
 > images_load.txt
 for i in `ls *.tar`
 do 
   sudo $docker load -i $i | tee -a images_load.txt
 done


}

images_list() {

  cat images_load.txt | grep Loaded | awk '{print $3}' > $images_list
}

images_push() {
while read -r line
do
    image_repo=`echo $line | awk -F '/' '{print $1}'`
    image_name=`echo $line | awk -F '/' '{print $NF}' | awk -F ':' '{print $1}'`
    image_tag=`echo $line | awk -F '/' '{print $NF}' | awk -F ':' '{print $2}'`
    sudo $docker tag $line ${registry_name}/${project}/${image_name}:${image_tag}
    $docker push ${registry_name}/${project}/${image_name}:${image_tag}

done < ${images_list}
}

images_push_2() {
while read -r line
do
    image_repo=`echo $line | awk -F '/' '{print $1}'`
    image_project=`echo $line | awk -F '/' '{print $(NF-1)}' `
    image_name=`echo $line | awk -F '/' '{print $NF}' | awk -F ':' '{print $1}'`
    image_tag=`echo $line | awk -F '/' '{print $NF}' | awk -F ':' '{print $2}'`
    sudo $docker tag $line ${registry_name}/${image_project}/${image_name}:${image_tag}
    #sudo $docker tag $line ${registry_name}/${image_name}:${image_tag}
    $docker push ${registry_name}/${image_project}/${image_name}:${image_tag}
    #$docker push ${registry_name}/${image_name}:${image_tag}

done < ${images_list}
}

images_pull() {
 while read -r line 
 do
    sudo $docker pull  $line
 done < $images_list

}

images_save() {

images_dir=${BASE_DIR}/images
mkdir $images_dir
while read -r line
do
    image_repo=`echo $line | awk -F '/' '{print $1}'`
    image_name=`echo $line | awk -F '/' '{print $NF}' | awk -F ':' '{print $1}'`
    image_tag=`echo $line | awk -F '/' '{print $NF}' | awk -F ':' '{print $2}'`
    sudo $docker save -o $images_dir/${image_repo}_${image_name}_${image_tag}.tar $line
done < $images_list
}

case $action in 
   pull) 
     images_pull
     ;;
   push)
     #images_push
     images_push_2
     ;;
   save)
     images_save
     ;;
   load)
   #  images_load_1
    # images_load_2
     images_load
    # images_list 
     ;;
   *)
     echo "Usage: $name [pull|push|save|load]]"
     exit 1
     ;;
esac
exit 0

```

## 4. 示例

### 4.1 拉取 kube-prometheus-stack 55.4.1 版本的镜像
imaegs_list.txt
```bash
docker.io/grafana/grafana:10.2.2
quay.io/kiwigrid/k8s-sidecar:1.25.2
quay.io/prometheus/alertmanager:v0.26.0
quay.io/prometheus/node-exporter:v1.7.0
quay.io/prometheus-operator/prometheus-config-reloader:v0.70.0
quay.io/prometheus-operator/prometheus-operator:v0.70.0
quay.io/prometheus/prometheus:v2.48.0
registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.10.1
```

```bash
sh images.sh pull
sh images.sh save
sh images.sh load
sh images.sh push
```


