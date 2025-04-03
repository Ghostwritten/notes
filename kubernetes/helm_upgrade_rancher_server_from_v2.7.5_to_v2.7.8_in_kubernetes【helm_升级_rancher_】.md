
![](https://i-blog.csdnimg.cn/blog_migrate/3ccfa182f7eb917ef486ce7540ade8a7.jpeg#pic_center)


## 1. 预备条件

- [Kubernetes Cluster](https://ghostwritten.blog.csdn.net/article/details/131749277)
- [Helm & Kubernetes Offline Deploy Rancher v2.7.5 Demo](https://blog.csdn.net/xixihahalelehehe/article/details/132695704)

> 注意：如果你是在vcenter 的虚拟机测试该应用，记得给当前版本做好快照，便于反复练习。

## 2. 目标
- rancher v2.7.5 升级 v2.7.8



当前集群 状态

```bash
$ helm ls -n cattle-system
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
rancher         cattle-system   1               2023-12-12 15:15:55.470319744 +0800 CST deployed        rancher-2.7.5                   v2.7.5     
rancher-webhook cattle-system   1               2023-12-12 07:23:47.884505024 +0000 UTC deployed        rancher-webhook-2.0.5+up0.3.5   0.3.5  

$ kubectl get pod -A
NAMESPACE                   NAME                                                    READY   STATUS      RESTARTS        AGE
cattle-fleet-local-system   fleet-agent-fbcbb8456-sfk5z                             1/1     Running     0               2d16h
cattle-fleet-system         fleet-controller-748ff89bfc-n8lct                       1/1     Running     2 (2d16h ago)   2d19h
cattle-fleet-system         gitjob-7bd88ddd7d-dwl4z                                 1/1     Running     1 (2d16h ago)   2d19h
cattle-system               rancher-bc4795d88-6dwmh                                 1/1     Running     1 (2d16h ago)   2d19h
cattle-system               rancher-bc4795d88-9wcn7                                 1/1     Running     1 (2d16h ago)   2d19h
cattle-system               rancher-bc4795d88-9xbsh                                 1/1     Running     2 (2d16h ago)   2d19h
cattle-system               rancher-webhook-7f6b5f4dd6-2jv2f                        1/1     Running     1 (2d16h ago)   2d19h
demo                        nginx                                                   1/1     Running     1 (2d16h ago)   2d16h
kube-system                 cloud-controller-manager-rancher02                      1/1     Running     7 (2d16h ago)   2d23h
kube-system                 etcd-rancher02                                          1/1     Running     2               2d23h
kube-system                 helm-install-rke2-canal-wrsjx                           0/1     Completed   0               2d23h
kube-system                 helm-install-rke2-coredns-nh95s                         0/1     Completed   0               2d23h
kube-system                 helm-install-rke2-ingress-nginx-h4p5q                   0/1     Completed   0               2d23h
kube-system                 helm-install-rke2-metrics-server-jg5fk                  0/1     Completed   0               2d23h
kube-system                 helm-install-rke2-snapshot-controller-crd-49t77         0/1     Completed   0               2d23h
kube-system                 helm-install-rke2-snapshot-controller-tkmjc             0/1     Completed   0               2d23h
kube-system                 helm-install-rke2-snapshot-validation-webhook-fnlc2     0/1     Completed   0               2d23h
kube-system                 kube-apiserver-rancher02                                1/1     Running     6               2d23h
kube-system                 kube-controller-manager-rancher02                       1/1     Running     7 (2d16h ago)   2d23h
kube-system                 kube-proxy-rancher02                                    1/1     Running     3 (2d16h ago)   2d16h
kube-system                 kube-scheduler-rancher02                                1/1     Running     2 (2d16h ago)   2d23h
kube-system                 rke2-canal-bqx25                                        2/2     Running     4 (2d16h ago)   2d23h
kube-system                 rke2-coredns-rke2-coredns-565dfc7d75-6g9wm              1/1     Running     2 (2d16h ago)   2d23h
kube-system                 rke2-coredns-rke2-coredns-autoscaler-6c48c95bf9-28tz8   1/1     Running     2 (2d16h ago)   2d23h
kube-system                 rke2-ingress-nginx-controller-4xhm8                     1/1     Running     2 (2d16h ago)   2d23h
kube-system                 rke2-metrics-server-c9c78bd66-rjwn7                     1/1     Running     2 (2d16h ago)   2d23h
kube-system                 rke2-snapshot-controller-6f7bbb497d-wft92               1/1     Running     2 (2d16h ago)   2d23h
kube-system                 rke2-snapshot-validation-webhook-65b5675d5c-2dckg       1/1     Running     2 (2d16h ago)   2d23h
```

## 3. 下载介质
- 下载 稳定charts 指定版本，请参考：[https://artifacthub.io/packages/helm/rancher-stable/rancher/2.7.9](https://artifacthub.io/packages/helm/rancher-stable/rancher/2.7.9)
- 下载最新 charts 指定版本，请参考：[https://artifacthub.io/packages/helm/rancher-latest/rancher/](https://artifacthub.io/packages/helm/rancher-latest/rancher/)

```bash
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
helm repo update
helm fetch rancher-latest/rancher --version=v2.7.8
```

## 4. 镜像入库

```bash
$ ls
images_list.txt  images.sh

$  cat images_list.txt 
docker.io/rancher/rancher:v2.7.8
docker.io/rancher/fleet-agent:v0.8.0
docker.io/rancher/fleet:v0.8.0
docker.io/rancher/gitjob:v0.1.76
docker.io/rancher/rancher-webhook:v0.3.6
docker.io/rancher/shell:v0.1.21
docker.io/rancher/mirrored-cluster-api-controller:v1.4.4
docker.io/rancher/kubectl:v1.20.2

$ cat images.sh
#!/bin/bash

#私有仓库
registry_name='harbor.fumai02.com'
BASE_DIR="$( dirname "$( readlink -f "${0}" )" )"
#项目名
project='rancher'

images_pull() {
 while read -r line 
 do
    sudo docker pull  $line
 done < images_list.txt

}

images_push() {
while read -r line
do
    image_repo=`echo $line | awk -F '/' '{print $1}'`
    image_name=`echo $line | awk -F '/' '{print $NF}' | awk -F ':' '{print $1}'`
    image_tag=`echo $line | awk -F '/' '{print $NF}' | awk -F ':' '{print $2}'`
    sudo docker tag $line ${registry_name}/${project}/${image_name}:${image_tag}
    sudo docker push ${registry_name}/${project}/${image_name}:${image_tag}

done < images_list.txt
}


images_pull
images_push
```

拉取公网镜像推送到私有仓库

```bash
$ sh images.sh 
```


## 5. 升级 rancher

使用以下命令创建 rancher：

```bash
helm upgrade rancher ./rancher-2.7.8.tgz \
    --namespace cattle-system \
    --set hostname=rancher02.ghostwritten.com \
    --set rancherImage=harbor.fumai02.com/rancher/rancher \
    --set ingress.tls.source=secret \
    --set privateCA=true \
    --set systemDefaultRegistry=harbor.fumai02.com \
    --set useBundledSystemChart=true 
```

输出：

```bash
Release "rancher" has been upgraded. Happy Helming!
NAME: rancher
LAST DEPLOYED: Fri Dec 15 15:27:55 2023
NAMESPACE: cattle-system
STATUS: deployed
REVISION: 3
TEST SUITE: None
NOTES:
Rancher Server has been installed.

NOTE: Rancher may take several minutes to fully initialize. Please standby while Certificates are being issued, Containers are started and the Ingress rule comes up.

Check out our docs at https://rancher.com/docs/

If you provided your own bootstrap password during installation, browse to https://rancher02.ghostwritten.com to get started.

If this is the first time you installed Rancher, get started by running this command and clicking the URL it generates:


echo https://rancher02.ghostwritten.com/dashboard/?setup=$(kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}')


To get just the bootstrap password on its own, run:


kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}{{ "\n" }}'



Happy Containering!
```

## 6. 检查测试
升级后效果

```bash
$ helm ls -n cattle-system
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
rancher         cattle-system   1               2023-10-26 07:31:49.883540457 +0800 CST deployed        rancher-2.7.8                   v2.7.8     
rancher-webhook cattle-system   1               2023-10-25 23:46:00.925115778 +0000 UTC deployed        rancher-webhook-2.0.6+up0.3.6   0.3.6      
$ kubectl get pod  -A
NAMESPACE                         NAME                                                    READY   STATUS      RESTARTS        AGE
cattle-fleet-local-system         fleet-agent-74466cd6dc-xc6jp                            1/1     Running     0               38m
cattle-fleet-system               fleet-controller-5989b98fc8-h6m57                       1/1     Running     0               40m
cattle-fleet-system               gitjob-d8f7cc69b-jvjh6                                  1/1     Running     0               40m
cattle-provisioning-capi-system   capi-controller-manager-6c4d64c64-wbgf7                 1/1     Running     0               38m
cattle-system                     helm-operation-bzzr7                                    0/2     Completed   0               41m
cattle-system                     helm-operation-pm6w8                                    0/2     Completed   0               30m
cattle-system                     helm-operation-qgs7j                                    0/2     Completed   0               39m
cattle-system                     helm-operation-sfxlc                                    0/2     Completed   0               42m
cattle-system                     rancher-7f87fb778f-ggh5q                                1/1     Running     0               51m
cattle-system                     rancher-7f87fb778f-hmlxq                                1/1     Running     0               46m
cattle-system                     rancher-7f87fb778f-vvzl6                                1/1     Running     0               51m
cattle-system                     rancher-webhook-7f4c7c4c4b-f4ddl                        1/1     Running     0               39m
demo                              nginx                                                   1/1     Running     1 (2d22h ago)   2d22h
kube-system                       cloud-controller-manager-rancher02                      1/1     Running     7 (2d22h ago)   3d5h
kube-system                       etcd-rancher02                                          1/1     Running     2               3d4h
kube-system                       helm-install-rke2-canal-wrsjx                           0/1     Completed   0               3d5h
kube-system                       helm-install-rke2-coredns-nh95s                         0/1     Completed   0               3d5h
kube-system                       helm-install-rke2-ingress-nginx-h4p5q                   0/1     Completed   0               3d5h
kube-system                       helm-install-rke2-metrics-server-jg5fk                  0/1     Completed   0               3d5h
kube-system                       helm-install-rke2-snapshot-controller-crd-49t77         0/1     Completed   0               3d5h
kube-system                       helm-install-rke2-snapshot-controller-tkmjc             0/1     Completed   0               3d5h
kube-system                       helm-install-rke2-snapshot-validation-webhook-fnlc2     0/1     Completed   0               3d5h
kube-system                       kube-apiserver-rancher02                                1/1     Running     6               3d4h
kube-system                       kube-controller-manager-rancher02                       1/1     Running     7 (2d22h ago)   3d5h
kube-system                       kube-proxy-rancher02                                    1/1     Running     3 (2d22h ago)   2d22h
kube-system                       kube-scheduler-rancher02                                1/1     Running     2 (2d22h ago)   3d5h
kube-system                       rke2-canal-bqx25                                        2/2     Running     4 (2d22h ago)   3d4h
kube-system                       rke2-coredns-rke2-coredns-565dfc7d75-6g9wm              1/1     Running     2 (2d22h ago)   3d4h
kube-system                       rke2-coredns-rke2-coredns-autoscaler-6c48c95bf9-28tz8   1/1     Running     2 (2d22h ago)   3d4h
kube-system                       rke2-ingress-nginx-controller-4xhm8                     1/1     Running     2 (2d22h ago)   3d4h
kube-system                       rke2-metrics-server-c9c78bd66-rjwn7                     1/1     Running     2 (2d22h ago)   3d4h
kube-system                       rke2-snapshot-controller-6f7bbb497d-wft92               1/1     Running     2 (2d22h ago)   3d4h
kube-system                       rke2-snapshot-validation-webhook-65b5675d5c-2dckg       1/1     Running     2 (2d22h ago)   3d4h
```

查看 rancher 镜像版本

```bash
$ kubectl get pod -n cattle-system -oyaml rancher-7f87fb778f-ggh5q  |grep image
    image: harbor.fumai02.com/rancher/rancher:v2.7.8
```

界面访问
![](https://i-blog.csdnimg.cn/blog_migrate/43e7da497b417035ba4cddc7a535c563.png)
查看升级前创建的 user 是否丢失
![](https://i-blog.csdnimg.cn/blog_migrate/e5dc6245b728ea02290eb0797d2b8f75.png)

通过user demo 登陆界面创建的 pod 实例仍正常可以获取状态
![](https://i-blog.csdnimg.cn/blog_migrate/781d98d31cd7985b94deffad0966792b.png)


