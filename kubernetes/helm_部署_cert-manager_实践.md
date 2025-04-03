![](https://i-blog.csdnimg.cn/blog_migrate/1744ec13473e813566bb1409efb24636.png)




## 1. 介绍
[Cert-manager](https://cert-manager.io/) 是一个 Kubernetes 上的证书管理控制器，它可以自动化证书签发和更新的过程。它使用了 Kubernetes 中的 Custom Resource Definitions (CRDs) 来定义证书的请求和颁发，以及与之相关的资源，如私钥和证书链。Cert-manager 支持多种证书颁发机构（CA），包括 Let’s Encrypt、Venafi、HashiCorp Vault 等。

Cert-manager 可以与 Kubernetes Ingress 控制器集成，以自动化 HTTPS 证书的签发和更新。当您创建一个 Ingress 资源时，Cert-manager 可以自动创建证书请求并向证书颁发机构提交请求。一旦证书颁发机构签发证书，Cert-manager 将自动将证书安装到相应的 Ingress 资源中，并配置 TLS 终止。

Cert-manager 还支持 ACME 协议（自动证书管理环境），它是一种自动化证书颁发协议，由 Let's Encrypt 领导的开放标准。ACME 协议允许您使用 Let's Encrypt 颁发的免费证书来启用 HTTPS，而不需要手动管理证书的签发和更新过程。

Cert-manager 还提供了一个 Webhook API，使您可以将自定义证书颁发机构集成到 Cert-manager 中，以便通过 Cert-manager 进行管理。

总的来说，Cert-manager 可以帮助您简化证书管理的流程，确保您的应用程序始终使用最新的证书，并且可以在 Kubernetes 中轻松地实现自动化证书管理。


##  2. 架构
![](https://i-blog.csdnimg.cn/blog_migrate/5f4c443897aa04fe22e3c263859a6cb7.png)

Cert-manager 通过 Kubernetes 中的 Custom Resource Definitions (CRDs) 来定义证书的请求和颁发，以及与之相关的资源，如私钥和证书链。Cert-manager 运行在 Kubernetes 集群中，作为一个 Deployment 或者 StatefulSet，包含以下三个主要组件：

- cert-manager 控制器：负责监视证书颁发请求（CertificateRequest）和颁发机构（Issuer 或 ClusterIssuer）的变化，并将证书颁发请求提交到相应的颁发机构进行签发。一旦证书颁发机构签发了证书，控制器将自动将证书安装到 Kubernetes 中的相应资源中。

- 证书颁发机构：用于配置证书颁发的配置和身份验证信息。Cert-manager 支持多种证书颁发机构，包括 Let’s Encrypt、Venafi、HashiCorp Vault 等。

- Webhook 服务：使得 Cert-manager 可以扩展到自定义证书颁发机构。Webhook 服务允许您将自定义证书颁发机构集成到 Cert-manager 中，以便通过 Cert-manager 进行管理。

Cert-manager 还可以与 Kubernetes 中的 Ingress 控制器集成，以自动化 HTTPS 证书的签发和更新。当您创建一个 Ingress 资源时，Cert-manager 可以自动创建证书请求并向证书颁发机构提交请求。一旦证书颁发机构签发证书，Cert-manager 将自动将证书安装到相应的 Ingress 资源中，并配置 TLS 终止。
## 3. 安装 
下来我们安装 `Cert-manager`，它会为我们自动签发免费的 `Let’s Encrypt HTTPS` 证书，并在过期前自动续期。首先，运行 `helm repo add` 命令添加官方 Helm 仓库。


```bash
$ kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.0/cert-manager.crds.yaml
$ helm repo add jetstack https://charts.jetstack.io
"jetstack" has been added to your repositories
```
然后，运行 helm repo update 更新本地缓存。

```bash
$ helm repo update
...Successfully got an update from the "jetstack" chart repository
```
接下来，运行 `helm install` 来安装 `Cert-manager`。

```bash
$ helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --version v1.12.0 --set ingressShim.defaultIssuerName=letsencrypt-prod --set ingressShim.defaultIssuerKind=ClusterIssuer --set ingressShim.defaultIssuerGroup=cert-manager.io --set installCRDs=true

NAME: cert-manager
LAST DEPLOYED: Mon Oct 17 21:26:44 2022
NAMESPACE: cert-manager
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
cert-manager v1.12.0 has been deployed successfully!
```
查看状态

```bash
$ kubectl get all -n cert-manager
NAME                                          READY   STATUS    RESTARTS   AGE
pod/cert-manager-6c4f5bb68f-7q762             1/1     Running   0          53s
pod/cert-manager-cainjector-f5c6565d4-8vpc4   1/1     Running   0          53s
pod/cert-manager-webhook-5f44bc85f4-9b7nz     1/1     Running   0          53s

NAME                           TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
service/cert-manager           ClusterIP   10.96.3.160   <none>        9402/TCP   53s
service/cert-manager-webhook   ClusterIP   10.96.3.178   <none>        443/TCP    53s

NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cert-manager              1/1     1            1           53s
deployment.apps/cert-manager-cainjector   1/1     1            1           53s
deployment.apps/cert-manager-webhook      1/1     1            1           53s

NAME                                                DESIRED   CURRENT   READY   AGE
replicaset.apps/cert-manager-6c4f5bb68f             1         1         1       53s
replicaset.apps/cert-manager-cainjector-f5c6565d4   1         1         1       53s
replicaset.apps/cert-manager-webhook-5f44bc85f4     1         1         1       53s

```

此外，还需要为 `Cert-manager` 创建 `ClusterIssuer`，用来提供签发机构。将下面的内容保存为 `cluster-issuer.yaml`。

```bash
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: "1zoxun1@gmail.com"
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:    
    - http01:
        ingress:
          class: nginx
```
注意，这里你需要将 `spec.acme.email` 替换为你真实的邮箱地址。然后运行 `kubectl apply` 提交到集群内。

```bash
$ kubectl apply -f cluster-issuer.yaml
clusterissuer.cert-manager.io/letsencrypt-prod created
```
到这里，`Cert-manager` 就已经配置好了。

## 4. 测试
略
##  5. 删除

```bash
helm list -n cert-manager
helm delete cert-manager  -n cert-manager
```
删除 cert-manager 命名空间

```bash
$ k delete ns cert-manager
$ cat delete-ns.sh
#!/bin/bash

NAMESPACE=$1
kubectl proxy --port=8002 &
kubectl get namespace $NAMESPACE -o json |jq '.spec = {"finalizers":[]}' >temp.json
curl -k -H "Content-Type: application/json" -X PUT --data-binary @temp.json 127.0.0.1:8002/api/v1/namespaces/$NAMESPACE/finalize
```


##  离线安装

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.crds.yaml
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm fetch jetstack/cert-manager --version v1.11.0 -d .
docker pull quay.io/jetstack/cert-manager-controller:v1.11.0 && docker save -o cert-manager-controller-v1.11.0.tar quay.io/jetstack/cert-manager-controller:v1.11.0
docker pull quay.io/jetstack/cert-manager-cainjector:v1.11.0 && docker save -o cert-manager-cainjector-v1.11.0.tar quay.io/jetstack/cert-manager-cainjector:v1.11.0
docker pull quay.io/jetstack/cert-manager-webhook:v1.11.0 && docker save -o cert-manager-webhook-v1.11.0.tar quay.io/jetstack/cert-manager-webhook:v1.11.0
docker pull quay.io/jetstack/cert-manager-ctl:v1.11.0 && docker save -o cert-manager-ctl-v1.11.0.tar quay.io/jetstack/cert-manager-ctl:v1.11.0
```

参考：

- [Cert-Manager 实现 K8s 服务域名证书自动化续签](https://mp.weixin.qq.com/s/PjvqZZzRLb2n8iBy7__ZZg)
- [https://cert-manager.io/](https://cert-manager.io/)
- 
