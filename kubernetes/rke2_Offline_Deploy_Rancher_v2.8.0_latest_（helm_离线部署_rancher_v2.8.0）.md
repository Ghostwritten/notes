![harbor.ghostwritten.com](https://i-blog.csdnimg.cn/blog_migrate/e5813ee2feb7b81cd039eb3faff3bcc4.png)





## 1. 预备条件

- 所有支持的操作系统都使用 64-bit x86 架构。Rancher 兼容当前所有的主流 Linux 发行版。

- [查询 kubernetes 与 rancher 兼容性](https://www.suse.com/suse-rancher/support-matrix/all-supported-versions/rancher-v2-7-9/)
- 请安装 ntp（Network Time Protocol），以防止在客户端和服务器之间由于时间不同步造成的证书验证错误。

- 某些 Linux 发行版的默认防火墙规则可能会阻止 Kubernetes 集群内的通信。从 Kubernetes v1.19 开始，你必须关闭 firewalld，因为它与 Kubernetes 网络插件冲突。
- [安装 kubernetes](https://ghostwritten.blog.csdn.net/article/details/134413829) ，这里我选择 rke2 方式
- 私有镜像仓库：你可以选择[安装 harbor](https://blog.csdn.net/xixihahalelehehe/article/details/127920005) 或者 [安装 registry](https://blog.csdn.net/xixihahalelehehe/article/details/105926147)



## 2. 为什么是三个节点？​
在RKE集群中，Rancher服务器数据存储在etcd上。这个etcd数据库在所有三个节点上运行。
etcd数据库需要奇数个节点，这样它总是可以选出一个拥有大多数etcd集群的领导者。如果etcd数据库不能选出一个领导者，etcd可能会遭受分裂的大脑，需要从备份中恢复集群。如果三个etcd节点中的一个失败，剩下的两个节点可以选举一个领导者，因为它们拥有etcd节点总数的大多数。





## 3. 配置私有仓库
(每个rke2节点都要执行更新)

- [RKE2 config containerd private registry](https://blog.csdn.net/xixihahalelehehe/article/details/134078913) 

```bash
$ vim  /etc/rancher/rke2/registries.yaml
mirrors:
  docker.io:
    endpoint:
      - "https://harbor.ghostwritten.com"
configs:
  "harbor.ghostwritten.com":
    auth:
      username: admin 
      password: Harbor12345 
    tls:
      insecure_skip_verify: true 
```
如果是master 节点，重启 rke2-server

```bash
systemctl restart  rke2-server.service && systemctl status rke2-server.service
```
如果是 node 节点，重启 rke2-agent
```bash
systemctl restart  rke2-agent.service && systemctl status rke2-agent.service
```
重启后`/etc/rancher/rke2/registries.yaml`的仓库配置会传递到`/var/lib/rancher/rke2/agent/etc/containerd/config.toml`

```bash
$ cat /var/lib/rancher/rke2/agent/etc/containerd/config.toml |grep -C 4  harbor

[plugins."io.containerd.grpc.v1.cri".registry.mirrors]

[plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
  endpoint = ["https://harbor.ghostwritten.com"]
[plugins."io.containerd.grpc.v1.cri".registry.configs."harbor.ghostwritten.com".auth]
  username = "admin"
  password = "Harbor12345"
[plugins."io.containerd.grpc.v1.cri".registry.configs."harbor.ghostwritten.com".tls]
  insecure_skip_verify = true
```

## 4. 介质清单


rancher_v2.8.0/
├── cert-manager.crds.yaml
├── cert-manager-images.txt
├── cert-manager-v1.13.3.tgz
├── cert-manager.yaml
├── helm-v3.13.3-linux-amd64.tar.gz
├── images
│   ├── docker.io_fleet-agent_v0.9.0.tar
│   ├── docker.io_fleet_v0.9.0.tar
│   ├── docker.io_gitjob_v0.1.96.tar
│   ├── docker.io_mirrored-cluster-api-controller_v1.4.4.tar
│   ├── docker.io_rancher_v2.8.0.tar
│   ├── docker.io_rancher-webhook_v0.4.2.tar
│   ├── docker.io_shell_v0.1.22.tar
│   ├── quay.io_cert-manager-cainjector_v1.13.3.tar
│   ├── quay.io_cert-manager-controller_v1.13.3.tar
│   ├── quay.io_cert-manager-ctl_v1.13.3.tar
│   └── quay.io_cert-manager-webhook_v1.13.3.tar
├── images.sh
├── install_cert-manager.sh
├── install_rancher.sh
├── rancher-2.8.0.tgz
├── rancher-cleanup.tar.gz
└── rancher-images.txt

- [images.sh: 容器镜像搬运最佳脚本](https://ghostwritten.blog.csdn.net/article/details/135082812)
- rancher 卸载工具：[https://github.com/rancher/rancher-cleanup.git](https://github.com/rancher/rancher-cleanup.git)



##  5. 安装 helm
```bash
wget https://get.helm.sh/helm-v3.13.3-linux-amd64.tar.gz
tar -xzvf helm-v3.13.3-linux-amd64.tar.gz
cp linux-amd64/helm /usr/local/bin/
helm version
rm -rf linux-amd64 helm-v3.13.3-linux-amd64.tar.gz

```
## 6. 安装 cert-manager

### 6.1 下载介质
（在联网节点下载）
```bash
wget https://github.com/cert-manager/cert-manager/releases/download/v1.13.3/cert-manager.crds.yaml
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm fetch jetstack/cert-manager --version v1.13.3
helm template ./cert-manager-v1.13.3.tgz | awk '$1 ~ /image:/ {print $2}' | sed s/\"//g >> cert-manager-images.txt
```
`cert-manager-images.txt` 镜像列表：
```bash
quay.io/jetstack/cert-manager-cainjector:v1.13.3
quay.io/jetstack/cert-manager-controller:v1.13.3
quay.io/jetstack/cert-manager-webhook:v1.13.3
quay.io/jetstack/cert-manager-ctl:v1.13.3
```
###  6.2 镜像入库
修改 `images.sh` 参数:
- registry_name='harbor.ghostwritten.com'
- project='cert-manager'
- docker='/usr/bin/podman'
- images_list='cert-manager-images.txt'

```bash
sh images.sh pull 
sh images.sh save
#搬运离线节点
sh images.sh load
sh images.sh push
```

### 6.3 helm 部署
（离线环境）

为 cert-manager 创建命名空间
```bash
kubectl create namespace cert-manager
```
创建crd
```bash
$ kubectl apply -f cert-manager.crds.yaml 
customresourcedefinition.apiextensions.k8s.io/certificaterequests.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/certificates.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/challenges.acme.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/clusterissuers.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/issuers.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/orders.acme.cert-manager.io created
```


install_cert-manager.sh 内容：
```bash
helm install --debug cert-manager ./cert-manager-v1.13.3.tgz \
    --namespace cert-manager \
    --create-namespace \
    --set image.repository=harbor.ghostwritten.com/rancher/cert-manager-controller \
    --set webhook.image.repository=harbor.ghostwritten.com/rancher/cert-manager-webhook \
    --set cainjector.image.repository=harbor.ghostwritten.com/rancher/cert-manager-cainjector \
    --set startupapicheck.image.repository=harbor.ghostwritten.com/rancher/cert-manager-ctl
```

查看

```bash
$ kubectl get pod -n cert-manager
NAME                                       READY   STATUS    RESTARTS   AGE
cert-manager-79bf4c54cf-xplpn              1/1     Running   0          22s
cert-manager-cainjector-6b8d78448f-2j8n4   1/1     Running   0          22s
cert-manager-startupapicheck-grzgz         1/1     Running   0          19s
cert-manager-webhook-c78d5bb7-mkr9x        1/1     Running   0          22s
```

### 6.4 cert-manager 卸载

```bash
$ helm delete cert-manager  -n cert-manager
release "cert-manager" uninstalled

$ kubectl get job   -n cert-manager  
NAME                           COMPLETIONS   DURATION   AGE
cert-manager-startupapicheck   1/1           27m        28m

$ kubectl delete  job   -n cert-manager   cert-manager-startupapicheck
job.batch "cert-manager-startupapicheck" deleted

$ kubectl delete  ns cert-manager
namespace "cert-manager" deleted
```
## 7. 安装 rancher
### 7.1 安装 rancher v2.7.9
- [官方下载镜像入私有仓库方法](https://ranchermanager.docs.rancher.com/zh/v2.8/getting-started/installation-and-upgrade/other-installation-methods/air-gapped-helm-cli-install/publish-images)


```bash
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
helm repo update
helm fetch rancher-stable/rancher --version=v2.7.9

```
```bash
docker.io/rancher/fleet:v0.8.0
docker.io/rancher/fleet-agent:v0.8.0
docker.io/rancher/gitjob:v0.1.76
docker.io/rancher/shell:v0.1.21
docker.io/rancher/rancher-webhook:v0.3.6
docker.io/rancher/rancher:v2.7.9
docker.io/rancher/mirrored-cluster-api-controller:v1.4.4
```
####  7.1.1 镜像入库
仅 helm 安装 rancher 依赖的镜像如下 `rancher-images.txt` ：

但涉及 rancher 集群管理，比如引导安装多种 rke2、安装插件等依赖的镜像。共470个，参考：
- [https://github.com/rancher/rancher/releases/download/v2.8.0/rancher-images.txt](https://github.com/rancher/rancher/releases/download/v2.8.0/rancher-images.txt)


修改 `images.sh` 参数:
- registry_name='harbor.ghostwritten.com'
- project='rancher'
- docker='/usr/bin/podman'
- images_list='rancher-images.txt'

```bash
sh images.sh pull 
sh images.sh save
#搬运离线节点
sh images.sh load
sh images.sh push
```
#### 7.1.2 helm 安装

```bash
   helm install rancher ./rancher-2.7.9.tgz \
    --namespace cattle-system \
    --create-namespace \
    --set hostname=rancher03.ghostwritten.com \
    --set certmanager.version=1.13.3 \
    --set rancherImage=harbor.ghostwritten.com/rancher \
    --set systemDefaultRegistry=harbor.ghostwritten.com \
    --set useBundledSystemChart=true \
    --set bootstrapPassword=admin
```

### 7.2 安装 rancher v2.8.0(离线部署有问题)

- [官方下载镜像入私有仓库方法](https://ranchermanager.docs.rancher.com/zh/v2.8/getting-started/installation-and-upgrade/other-installation-methods/air-gapped-helm-cli-install/publish-images)


```bash
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
helm repo update
helm fetch rancher-stable/rancher --version=v2.8.0
```
####  7.2.1 镜像入库
仅 helm 安装 rancher 依赖的镜像如下 `rancher-images.txt` ：
```bash
docker.io/rancher/fleet-agent:v0.9.0
docker.io/rancher/fleet:v0.9.0
docker.io/rancher/gitjob:v0.1.96
docker.io/rancher/mirrored-cluster-api-controller:v1.4.4
docker.io/rancher/rancher:v2.8.0
docker.io/rancher/rancher-webhook:v0.4.2
docker.io/rancher/shell:v0.1.22
```
但涉及 rancher 集群管理，比如引导安装多种 rke2、安装插件等依赖的镜像。共470个，参考：
- [https://github.com/rancher/rancher/releases/download/v2.8.0/rancher-images.txt](https://github.com/rancher/rancher/releases/download/v2.8.0/rancher-images.txt)


修改 `images.sh` 参数:
- registry_name='harbor.ghostwritten.com'
- project='rancher'
- docker='/usr/bin/podman'
- images_list='rancher-images.txt'

```bash
sh images.sh pull 
sh images.sh save
#搬运离线节点
sh images.sh load
sh images.sh push
```

#### 7.2.2 helm 安装

```bash
   helm install rancher ./rancher-2.8.0.tgz \
    --namespace cattle-system \
    --create-namespace \
    --set hostname=rancher03.ghostwritten.com \
    --set certmanager.version=1.13.3 \
    --set rancherImage=harbor.ghostwritten.com/rancher \
    --set systemDefaultRegistry=harbor.ghostwritten.com \
    --set useBundledSystemChart=true \
    --set bootstrapPassword=admin
```
输出：

```bash
# Source: rancher/templates/issuer-rancher.yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: rancher
  labels:
    app: rancher
    chart: rancher-2.8.0
    heritage: Helm
    release: rancher
spec:
  ca:
    secretName: tls-rancher

NOTES:
Rancher Server has been installed.

NOTE: Rancher may take several minutes to fully initialize. Please standby while Certificates are being issued, Containers are started and the Ingress rule comes up.

Check out our docs at https://rancher.com/docs/

If you provided your own bootstrap password during installation, browse to https://rancher01.ghostwritten.dev to get started.

If this is the first time you installed Rancher, get started by running this command and clicking the URL it generates:


echo https://rancher01.ghostwritten.dev/dashboard/?setup=$(kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}')


To get just the bootstrap password on its own, run:

kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}{{ "\n" }}'



Happy Containering!
```

## 8. 验证

```bash
$ helm ls -n cattle-system
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
rancher         cattle-system   1               2024-01-10 05:15:14.096529535 -0500 EST deployed        rancher-2.8.0                   v2.8.0     
rancher-webhook cattle-system   1               2024-01-10 10:21:03.85680939 +0000 UTC  deployed        rancher-webhook-103.0.1+up0.4.2 0.4.2 

$ kubectl get pod -A
NAMESPACE                         NAME                                                    READY   STATUS      RESTARTS        AGE
cattle-fleet-system               fleet-controller-6b4dd5db6c-shwsp                       1/1     Running     0               8m59s
cattle-fleet-system               gitjob-75b769c6fb-bx5zg                                 1/1     Running     0               8m59s
cattle-provisioning-capi-system   capi-controller-manager-6c4d64c64-4pjvz                 1/1     Running     0               6m15s
cattle-system                     helm-operation-2jt9g                                    0/2     Completed   0               7m40s
cattle-system                     helm-operation-9sgm6                                    0/2     Completed   0               9m13s
cattle-system                     helm-operation-pt2w6                                    0/2     Completed   0               8m9s
cattle-system                     helm-operation-t2kkr                                    0/2     Completed   0               7m11s
cattle-system                     helm-operation-zt929                                    0/2     Completed   0               6m21s
cattle-system                     rancher-5ccc6b9d89-hsv6m                                1/1     Running     0               9m2s
cattle-system                     rancher-5ccc6b9d89-ph9l7                                1/1     Running     0               12m
cattle-system                     rancher-5ccc6b9d89-w2h66                                1/1     Running     0               12m
cattle-system                     rancher-webhook-dd69b4d4f-s8n9n                         1/1     Running     0               7m
cert-manager                      cert-manager-79bf4c54cf-xplpn                           1/1     Running     0               51m
cert-manager                      cert-manager-cainjector-6b8d78448f-2j8n4                1/1     Running     0               51m
cert-manager                      cert-manager-webhook-c78d5bb7-mkr9x                     1/1     Running     0               51m
kube-system                       cloud-controller-manager-rke2-master01                  1/1     Running     4 (9m25s ago)   5d2h
kube-system                       cloud-controller-manager-rke2-master02                  1/1     Running     5 (24h ago)     5d1h
kube-system                       cloud-controller-manager-rke2-master03                  1/1     Running     0               120m
kube-system                       etcd-rke2-master01                                      1/1     Running     1               5d2h
kube-system                       etcd-rke2-master02                                      1/1     Running     1               5d1h
kube-system                       etcd-rke2-master03                                      1/1     Running     0               120m
kube-system                       helm-install-rke2-canal-6v6qr                           0/1     Completed   0               5d2h
kube-system                       helm-install-rke2-coredns-b5ttn                         0/1     Completed   0               5d2h
kube-system                       helm-install-rke2-ingress-nginx-45cqw                   0/1     Completed   0               5d2h
kube-system                       helm-install-rke2-metrics-server-mq6qh                  0/1     Completed   0               5d2h
kube-system                       helm-install-rke2-snapshot-controller-crd-jn4zf         0/1     Completed   0               5d2h
kube-system                       helm-install-rke2-snapshot-controller-zt8f5             0/1     Completed   2               5d2h
kube-system                       helm-install-rke2-snapshot-validation-webhook-kgjbt     0/1     Completed   0               5d2h
kube-system                       kube-apiserver-rke2-master01                            1/1     Running     1               5d2h
kube-system                       kube-apiserver-rke2-master02                            1/1     Running     1               5d1h
kube-system                       kube-apiserver-rke2-master03                            1/1     Running     0               120m
kube-system                       kube-controller-manager-rke2-master01                   1/1     Running     5 (9m24s ago)   5d2h
kube-system                       kube-controller-manager-rke2-master02                   1/1     Running     5 (24h ago)     5d1h
kube-system                       kube-controller-manager-rke2-master03                   1/1     Running     0               120m
kube-system                       kube-proxy-rke2-master01                                1/1     Running     1 (24h ago)     5d2h
kube-system                       kube-proxy-rke2-master02                                1/1     Running     1 (24h ago)     5d1h
kube-system                       kube-proxy-rke2-master03                                1/1     Running     0               120m
kube-system                       kube-proxy-rke2-node01                                  1/1     Running     0               24h
kube-system                       kube-scheduler-rke2-master01                            1/1     Running     1 (24h ago)     5d2h
kube-system                       kube-scheduler-rke2-master02                            1/1     Running     1 (24h ago)     5d1h
kube-system                       kube-scheduler-rke2-master03                            1/1     Running     0               120m
kube-system                       rke2-canal-dwr7m                                        2/2     Running     2 (24h ago)     5d
kube-system                       rke2-canal-jjbzf                                        2/2     Running     0               121m
kube-system                       rke2-canal-kzvc9                                        2/2     Running     2 (24h ago)     5d1h
kube-system                       rke2-canal-ssvcb                                        2/2     Running     2 (24h ago)     5d2h
kube-system                       rke2-coredns-rke2-coredns-565dfc7d75-6dbr9              1/1     Running     1 (24h ago)     5d2h
kube-system                       rke2-coredns-rke2-coredns-565dfc7d75-tvf2f              1/1     Running     1 (24h ago)     5d1h
kube-system                       rke2-coredns-rke2-coredns-autoscaler-6c48c95bf9-lb2xt   1/1     Running     1 (24h ago)     5d2h
kube-system                       rke2-ingress-nginx-controller-4dhc7                     1/1     Running     1 (24h ago)     5d
kube-system                       rke2-ingress-nginx-controller-8lp6v                     1/1     Running     1 (24h ago)     5d2h
kube-system                       rke2-ingress-nginx-controller-s5rw9                     1/1     Running     0               120m
kube-system                       rke2-ingress-nginx-controller-x2p78                     1/1     Running     1 (24h ago)     5d1h
kube-system                       rke2-metrics-server-c9c78bd66-szclt                     1/1     Running     1 (24h ago)     5d2h
kube-system                       rke2-snapshot-controller-6f7bbb497d-b426h               1/1     Running     1 (24h ago)     5d2h
kube-system                       rke2-snapshot-validation-webhook-65b5675d5c-2b98t       1/1     Running     1 (24h ago)     5d2h


$ kubectl get ingress -n cattle-system
NAME      CLASS    HOSTS                        ADDRESS                                                   PORTS     AGE
rancher   <none>   rancher01.ghostwritten.dev   192.168.23.91,192.168.23.92,192.168.23.93,192.168.23.94   80, 443   17m

$ kubectl -n cattle-system rollout status deploy/rancher
deployment "rancher" successfully rolled out

$   kubectl get secret -n cattle-system tls-rancher-ingress -o jsonpath='{.data.ca\.crt}' | base64 -d | openssl x509  -noout -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 0 (0x0)
        Signature Algorithm: ecdsa-with-SHA256
        Issuer: O = dynamiclistener-org, CN = dynamiclistener-ca@1704881898
        Validity
            Not Before: Jan 10 10:18:18 2024 GMT
            Not After : Jan  7 10:18:18 2034 GMT
        Subject: O = dynamiclistener-org, CN = dynamiclistener-ca@1704881898
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    04:78:35:e2:95:be:fc:08:70:b0:89:39:77:d6:0e:
                    5f:5c:30:cc:5c:10:b8:78:55:58:c6:1c:df:58:7b:
                    8b:75:6c:36:48:08:5a:31:1c:01:be:54:ca:a4:69:
                    5d:e1:ce:98:a3:05:c5:97:fd:5f:ca:eb:ba:74:21:
                    bf:e4:ee:10:db
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment, Certificate Sign
            X509v3 Basic Constraints: critical
                CA:TRUE
            X509v3 Subject Key Identifier: 
                CE:E4:D9:15:58:B4:B1:7C:19:34:05:F7:59:52:11:1C:FE:52:4A:79
    Signature Algorithm: ecdsa-with-SHA256
         30:44:02:20:01:a8:8c:a0:ce:9b:83:1a:17:f3:62:35:e6:80:
         94:d6:50:b1:b8:a0:96:44:5e:d0:8b:de:6b:b0:e8:30:ad:d3:
         02:20:5d:0a:f0:92:36:4d:41:40:ea:00:7a:b4:de:68:ae:f9:
         a7:de:46:eb:90:8c:e7:77:43:4a:d0:af:1a:95:25:58
```


## 9. 界面预览
访问：https://rancher03.ghostwritten.com

保存密码：6DfzJKXG6LTiRrTU
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/32dfb757dbe809197394391bfeae8bc5.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0d5103c4245877dba0b48beb93736eae.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dbe9d77c9b5187de49a79b4cb7ba241d.png)

## 10. 卸载 rancher

```bash
git clone https://github.com/rancher/rancher-cleanup.git
cd rancher-cleanup
kubectl create -f deploy/rancher-cleanup.yaml
kubectl  -n kube-system logs -l job-name=cleanup-job  -f

kubectl create -f deploy/verify.yaml
kubectl  -n kube-system logs -l job-name=verify-job  -f
kubectl  -n kube-system logs -l job-name=verify-job  -f | grep -v "is deprecated"
```
非常丝滑。
## 11. 问题 rancher v2.8.0
- 离线问题:[https://github.com/rancher/rancher/issues/43779](https://github.com/rancher/rancher/issues/43779)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f6eaceb1fa379f1c9cd697eae81a9f44.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e86f79ac1da7363bd6402bd2fc79246e.png)




参考：

- [https://ranchermanager.docs.rancher.com/zh/v2.8/getting-started/installation-and-upgrade/other-installation-methods/air-gapped-helm-cli-install/install-rancher-ha](https://ranchermanager.docs.rancher.com/zh/v2.8/getting-started/installation-and-upgrade/other-installation-methods/air-gapped-helm-cli-install/install-rancher-ha)
- [https://ranchermanager.docs.rancher.com/zh/v2.8/pages-for-subheaders/install-upgrade-on-a-kubernetes-cluster](https://ranchermanager.docs.rancher.com/zh/v2.8/pages-for-subheaders/install-upgrade-on-a-kubernetes-cluster)
- [Helm & Kubernetes Offline Deploy Rancher v2.7.5 Demo （helm 离线部署 rancher 实践）](https://ghostwritten.blog.csdn.net/article/details/132695704)
