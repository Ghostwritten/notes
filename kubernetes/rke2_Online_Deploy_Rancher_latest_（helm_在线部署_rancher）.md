![3.13.3](https://i-blog.csdnimg.cn/blog_migrate/eeaf06ab0c85378199a6348db9a75453.png)




## 1. 简介
[Rancher](https://ranchermanager.docs.rancher.com/zh/) 是一个 Kubernetes 管理工具，让你能在任何地方和任何提供商上部署和运行集群。

Rancher 可以创建来自 Kubernetes 托管服务提供商的集群，创建节点并安装 Kubernetes，或者导入在任何地方运行的现有 Kubernetes 集群。

Rancher 基于 Kubernetes 添加了新的功能，包括统一所有集群的身份验证和 RBAC，让系统管理员从一个位置控制全部集群的访问。

此外，Rancher 可以为集群和资源提供更精细的监控和告警，将日志发送到外部提供商，并通过应用商店（Application Catalog）直接集成 Helm。如果你拥有外部 CI/CD 系统，你可以将其与 Rancher 对接。没有的话，你也可以使用 Rancher 提供的 Fleet 自动部署和升级工作负载。

Rancher 是一个 全栈式 的 Kubernetes 容器管理平台，为你提供在任何地方都能成功运行 Kubernetes 的工具。
## 2. 预备条件

- [安装 kubernetes](https://ghostwritten.blog.csdn.net/article/details/134413829) ，这里我选择 rke2 方式
## 3. 安装 helm

```bash
wget https://get.helm.sh/helm-v3.13.3-linux-amd64.tar.gz
tar -xzvf helm-v3.13.3-linux-amd64.tar.gz
cp linux-amd64/helm /usr/local/bin/
helm version
rm -rf linux-amd64 helm-v3.13.3-linux-amd64.tar.gz

```






## 4. 安装 cert-manager

- [https://cert-manager.io/docs/releases/](https://cert-manager.io/docs/releases/)

### 4.1 yaml 安装
```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.3/cert-manager.yaml

or
wget  https://github.com/cert-manager/cert-manager/releases/download/v1.13.3/cert-manager.yaml
kubectl apply -f cert-manager.yaml
```
### 4.2 helm 安装

查询helm charts 版本：[https://artifacthub.io/packages/search?org=cert-manager](https://artifacthub.io/packages/search?org=cert-manager)

```bash
helm repo add jetstack https://charts.jetstack.io

helm repo update

helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace

# Windows Powershell
helm install cert-manager jetstack/cert-manager `
  --namespace cert-manager `
  --create-namespace
```


## 5. 安装 rancher

```bash
Latest：建议用于试用最新功能
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest

Stable：建议用于生产环境
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable

Alpha：即将发布的实验性预览。
helm repo add rancher-alpha https://releases.rancher.com/server-charts/alpha

>注意：不支持升级到 Alpha 版、从 Alpha 版升级或在 Alpha 版之间升级。
```
这里我选择 latest
```bash
$ helm repo list
NAME            URL                                              
rancher-latest  https://releases.rancher.com/server-charts/latest
jetstack        https://charts.jetstack.io 
```


```bash
kubectl create namespace cattle-system
helm install rancher rancher-latest/rancher \
  --namespace cattle-system \
  --set hostname=rancher01.demo.com \
  --set bootstrapPassword=admin
```
输出：

```bash
helm install rancher rancher-latest/rancher \
>   --namespace cattle-system \
>   --set hostname=rancher01.demo.com \
>   --set bootstrapPassword=admin
NAME: rancher
LAST DEPLOYED: Wed Jan 10 12:40:44 2024
NAMESPACE: cattle-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Rancher Server has been installed.

NOTE: Rancher may take several minutes to fully initialize. Please standby while Certificates are being issued, Containers are started and the Ingress rule comes up.

Check out our docs at https://rancher.com/docs/

If you provided your own bootstrap password during installation, browse to https://rancher01.demo.com to get started.

If this is the first time you installed Rancher, get started by running this command and clicking the URL it generates:


echo https://rancher01.demo.com/dashboard/?setup=$(kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}')


To get just the bootstrap password on its own, run:


kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}{{ "\n" }}'

```

## 6. 验证

查看集群pod

```bash
$ helm ls -n cattle-system
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
rancher         cattle-system   1               2024-01-10 12:40:44.827222381 +0800 CST deployed        rancher-2.8.0                   v2.8.0     
rancher-webhook cattle-system   1               2024-01-10 04:48:50.16424492 +0000 UTC  deployed        rancher-webhook-103.0.1+up0.4.2 0.4.2      

$ kubectl get pod -A
NAMESPACE                         NAME                                                    READY   STATUS      RESTARTS        AGE
cattle-fleet-system               fleet-controller-59cdb866d7-jhz56                       1/1     Running     0               109m
cattle-fleet-system               gitjob-f497866f8-kvnqr                                  1/1     Running     0               109m
cattle-provisioning-capi-system   capi-controller-manager-6f87d6bd74-h8ztb                1/1     Running     0               107m
cattle-system                     rancher-59b9f4f9b6-2bbj8                                1/1     Running     1 (113m ago)    116m
cattle-system                     rancher-59b9f4f9b6-njj6g                                1/1     Running     0               116m
cattle-system                     rancher-59b9f4f9b6-sstkl                                1/1     Running     0               116m
cattle-system                     rancher-webhook-65f5455d9c-x8c5s                        1/1     Running     0               108m
cert-manager                      cert-manager-77c645c9cd-gjklf                           1/1     Running     0               179m
cert-manager                      cert-manager-cainjector-6678d4cbcd-m67hs                1/1     Running     0               179m
cert-manager                      cert-manager-webhook-996c79df8-h24sb                    1/1     Running     0               179m
kube-system                       cloud-controller-manager-rancher02                      1/1     Running     3 (3h39m ago)   29d
kube-system                       etcd-rancher02                                          1/1     Running     2               29d
kube-system                       helm-install-rke2-canal-wrsjx                           0/1     Completed   0               29d
kube-system                       helm-install-rke2-coredns-nh95s                         0/1     Completed   0               29d
kube-system                       helm-install-rke2-ingress-nginx-h4p5q                   0/1     Completed   0               29d
kube-system                       helm-install-rke2-metrics-server-jg5fk                  0/1     Completed   0               29d
kube-system                       helm-install-rke2-snapshot-controller-crd-49t77         0/1     Completed   0               29d
kube-system                       helm-install-rke2-snapshot-controller-tkmjc             0/1     Completed   0               29d
kube-system                       helm-install-rke2-snapshot-validation-webhook-fnlc2     0/1     Completed   0               29d
kube-system                       kube-apiserver-rancher02                                1/1     Running     1               29d
kube-system                       kube-controller-manager-rancher02                       1/1     Running     3 (3h39m ago)   29d
kube-system                       kube-proxy-rancher02                                    1/1     Running     2 (3h39m ago)   3h38m
kube-system                       kube-scheduler-rancher02                                1/1     Running     1 (3h40m ago)   29d
kube-system                       rke2-canal-bqx25                                        2/2     Running     2 (3h40m ago)   29d
kube-system                       rke2-coredns-rke2-coredns-565dfc7d75-6g9wm              1/1     Running     1 (3h40m ago)   29d
kube-system                       rke2-coredns-rke2-coredns-autoscaler-6c48c95bf9-28tz8   1/1     Running     1 (3h40m ago)   29d
kube-system                       rke2-ingress-nginx-controller-4xhm8                     1/1     Running     1 (3h40m ago)   29d
kube-system                       rke2-metrics-server-c9c78bd66-rjwn7                     1/1     Running     1 (3h40m ago)   29d
kube-system                       rke2-snapshot-controller-6f7bbb497d-wft92               1/1     Running     1 (3h40m ago)   29d
kube-system                       rke2-snapshot-validation-webhook-65b5675d5c-2dckg       1/1     Running     1 (3h40m ago)   29d

$ kubectl get ingress -n cattle-system
NAME      CLASS    HOSTS                ADDRESS         PORTS     AGE
rancher   <none>   rancher01.demo.com   192.168.23.79   80, 443   116m

$ kubectl -n cattle-system rollout status deploy/rancher
deployment "rancher" successfully rolled out
```

获取自动创建的ca.crt证书的过期时间，时间是2034年1月7日

```bash
$ kubectl get secret -n cattle-system tls-rancher-ingress -o jsonpath='{.data.ca\.crt}' | base64 -d | openssl x509  -noout -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 0 (0x0)
        Signature Algorithm: ecdsa-with-SHA256
        Issuer: O = dynamiclistener-org, CN = dynamiclistener-ca@1704861915
        Validity
            Not Before: Jan 10 04:45:15 2024 GMT
            Not After : Jan  7 04:45:15 2034 GMT
        Subject: O = dynamiclistener-org, CN = dynamiclistener-ca@1704861915
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    04:bf:8b:2f:74:de:9b:35:40:c6:4a:0d:44:5d:f8:
                    a7:27:b6:41:54:32:7e:a2:0d:d3:50:a8:6d:5f:a3:
                    74:95:1e:ea:fd:82:77:f2:42:2e:b3:35:71:9b:cb:
                    db:74:d0:80:e7:4a:90:9d:8f:d7:09:a6:e4:51:70:
                    29:32:e1:05:e8
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment, Certificate Sign
            X509v3 Basic Constraints: critical
                CA:TRUE
            X509v3 Subject Key Identifier: 
                63:00:61:8F:F9:84:AF:1F:D5:EB:93:E0:8E:A0:51:16:3F:AC:A5:D1
    Signature Algorithm: ecdsa-with-SHA256
         30:46:02:21:00:8e:83:8b:5f:48:c5:b7:4a:9c:48:54:03:17:
         70:a2:16:74:d2:c1:bd:15:bd:0c:5b:cb:00:57:35:a5:69:e9:
         7a:02:21:00:80:ca:2c:34:41:ec:3c:4f:4a:b1:f3:00:97:1b:
         18:91:98:5c:3c:37:4c:b7:28:c8:cc:20:ea:7c:44:30:e3:86
```

界面访问
![3.13.3](https://i-blog.csdnimg.cn/blog_migrate/161b8ef4114479e1f684dd4c01a1cfd6.png)

密码：
KR5gy93UdgHKNVcr

bootstrap password 密码是我们部署的时候传参`bootstrapPassword=admin`命令。你还可以通过以下命令查看。
第一种：

```bash
kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}{{ "\n" }}'
```
第二种：
```bash
kubectl get secret --namespace cattle-system bootstrap-secret  -oyaml
apiVersion: v1
data:
  bootstrapPassword: YWRtaW4=
kind: Secret
metadata:
  annotations:
    field.cattle.io/projectId: local:p-4ss74
    helm.sh/hook: pre-install,pre-upgrade
    helm.sh/hook-weight: "-5"
    helm.sh/resource-policy: keep
  creationTimestamp: "2024-01-10T04:40:48Z"
  name: bootstrap-secret
  namespace: cattle-system
  resourceVersion: "78945"
  uid: 2f0f6924-1feb-479a-ac34-68006eac85e9
type: Opaque

$ echo  "YWRtaW4=" | base64 -d
admin
```

登陆继续。进入首页。

## 7. 界面预览
![3.13.3](https://i-blog.csdnimg.cn/blog_migrate/906e93749ecba89a1b50e79b885806df.png)
![3.13.3](https://i-blog.csdnimg.cn/blog_migrate/f20231ff47d205aafa95dcf3bbc6d5c3.png)
![3.13.3](https://i-blog.csdnimg.cn/blog_migrate/a866c04dcdf2111a38c7ad4151401fa0.png)
![3.13.3](https://i-blog.csdnimg.cn/blog_migrate/bf6bf9ed0ce3d636283b7154f4d576e4.png)

## 版本

- v2.8.0
- v2.8.1:[https://github.com/rancher/rancher/releases/tag/v2.8.1](https://github.com/rancher/rancher/releases/tag/v2.8.1)

参考：

- [https://ranchermanager.docs.rancher.com/zh/v2.8/pages-for-subheaders/install-upgrade-on-a-kubernetes-cluster](https://ranchermanager.docs.rancher.com/zh/v2.8/pages-for-subheaders/install-upgrade-on-a-kubernetes-cluster)
- [Helm Deploy Online Rancher Demo](https://ghostwritten.blog.csdn.net/article/details/132701099)
- [rancher 手册](https://ghostwritten.blog.csdn.net/article/details/135285243)
