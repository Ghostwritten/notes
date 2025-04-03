
![](https://i-blog.csdnimg.cn/blog_migrate/b9eb7a5b93bfc487d06e08c5dfcd6bb2.png)




## 简介
[Rancher](https://ranchermanager.docs.rancher.com/) 是一个开源的企业级全栈化容器部署及管理平台。已有超过 1900 万次下载，4000+ 生产环境的应用。

简单的说，就是一个可以让你通过 web 界面管理 docker 容器的平台。定位上和 K8s 比较接近，都是通过 web 界面赋予完全的 docker 服务编排功能。

特色：

- 平台部署方便。管理 docker 的平台本身也基于 docker 部署。只要你有 docker ，一句命令就完成平台的部署了。

- 平台扩展方便。通过 agent 机制，一句 docker 命令完成 agent 部署，快速增加你的物理机。同时也支持 AWS 等云主机， 2.0 版本甚至还支持 K8s 。

- 服务部署方便。通过应用商店，2 步完成应用部署，而且还是像 docker-compose 那样各个中间件独立编排，可以随时扩容的哦。

- 自带账户权限。相比 K8s 没有账号管理，rancher 自带账号权限体系。账号可以独立创建，也可以很方便地接入 ldap 等账号体系。对于公司使用是一大利器
## 预备条件
- [Kubernetes Cluster](https://ghostwritten.blog.csdn.net/article/details/131749277)
- [Ingress Controller](https://blog.csdn.net/xixihahalelehehe/article/details/112831123)
- [helm](https://blog.csdn.net/xixihahalelehehe/article/details/128847114)

## 在线安装 Rancher Helm Chart
添加 Helm Chart 仓库
```bash
Latest：建议用于试用最新功能
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest

Stable：建议用于生产环境
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable

Alpha：即将发布的实验性预览。
helm repo add rancher-alpha https://releases.rancher.com/server-charts/alpha

>注意：不支持升级到 Alpha 版、从 Alpha 版升级或在 Alpha 版之间升级。

为 Rancher 创建命名空间

```bash
kubectl create namespace cattle-system
```

##  选择 SSL 配置
Rancher Server 默认设计为安全的，并且需要 SSL/TLS 配置。

  > 如果你想在外部终止 SSL/TLS，请[参见外部负载均衡器的 TLS 终止](https://ranchermanager.docs.rancher.com/zh/getting-started/installation-and-upgrade/installation-references/helm-chart-options#%E5%A4%96%E9%83%A8-tls-%E7%BB%88%E6%AD%A2)。

你可以从以下三种证书来源中选择一种，用于在 Rancher Server 中终止 TLS：

- `Rancher 生成的 TLS 证书`：要求你在集群中安装 cert-manager。Rancher 使用 cert-manager 签发并维护证书。Rancher 会生成自己的 CA 证书，并使用该 CA 签署证书。然后 cert-manager负责管理该证书。
- `Let's Encrypt`：Let's Encrypt 选项也需要使用 cert-manager。但是，在这种情况下，cert-manager 与 Let's Encrypt 的特殊颁发者相结合，该颁发者执行获取 Let's Encrypt 颁发的证书所需的所有操作（包括请求和验证）。此配置使用 HTTP 验证（HTTP-01），因此负载均衡器必须具有可以从互联网访问的公共 DNS 记录。
- `你已有的证书`：使用已有的 CA 颁发的公有或私有证书。Rancher 将使用该证书来保护 WebSocket 和 HTTPS 流量。在这种情况下，你必须上传名称分别为 tls.crt 和 tls.key的 PEM 格式的证书以及相关的密钥。如果你使用私有 CA，则还必须上传该 CA 证书。这是由于你的节点可能不信任此私有 CA。Rancher 将获取该 CA 证书，并从中生成一个校验和，各种 Rancher 组件将使用该校验和来验证其与 Rancher 的连接。

| 配置                | Helm Chart 选项                  | 是否需要 cert-manager |
|-------------------|--------------------------------|-------------------|
| Rancher 生成的证书（默认） | ingress.tls.source=rancher     | 是                 |
| Let’s Encrypt     | ingress.tls.source=letsEncrypt | 是                 |
| 你已有的证书            | ingress.tls.source=secret      | 否                 |


## 安装 cert-manager
> 如果你使用自己的证书文件（ingress.tls.source=secret）或使用[外部负载均衡器的 TLS 终止](https://ranchermanager.docs.rancher.com/zh/getting-started/installation-and-upgrade/installation-references/helm-chart-options#%E5%A4%96%E9%83%A8-tls-%E7%BB%88%E6%AD%A2)，你可以跳过此步骤。

仅在使用 Rancher 生成的证书（ingress.tls.source=rancher）或 Let's Encrypt 颁发的证书（ingress.tls.source=letsEncrypt）时，才需要安装 cert-manager。

> 由于 cert-manager 的最新改动，你需要升级 cert-manager 版本。如果你需要升级 Rancher 并使用低于 0.11.0 的 cert-manager 版本，请[参见升级文档](https://ranchermanager.docs.rancher.com/zh/getting-started/installation-and-upgrade/resources/upgrade-cert-manager)。

```bash
# 如果你手动安装了CRD，而不是在 Helm 安装命令中添加了 `--set installCRDs=true` 选项，你应该在升级 Helm Chart 之前升级 CRD 资源。
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.crds.yaml

# 添加 Jetstack Helm 仓库
helm repo add jetstack https://charts.jetstack.io

# 更新本地 Helm Chart 仓库缓存
helm repo update

# 安装 cert-manager Helm Chart
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.11.0
```
安装完 cert-manager 后，你可以通过检查 cert-manager 命名空间中正在运行的 Pod 来验证它是否已正确部署：

```bash
kubectl get pods --namespace cert-manager

NAME                                       READY   STATUS    RESTARTS   AGE
cert-manager-5c6866597-zw7kh               1/1     Running   0          2m
cert-manager-cainjector-577f6d9fd7-tr77l   1/1     Running   0          2m
cert-manager-webhook-787858fcdb-nlzsq      1/1     Running   0          2m
```
##  Helm 安装 Rancher
不同的证书配置需要使用不同的 Rancher 安装命令。

但是，无论证书如何配置，Rancher 在 cattle-system 命名空间中的安装名称应该总是 rancher。

> 这个安装 Rancher 的最终命令需要一个将流量转发到 Rancher 的域名。如果你使用 Helm CLI 设置概念证明，则可以在传入 hostname 选项时使用伪域名。伪域名的一个例子是 `<IP_OF_LINUX_NODE>.sslip.io`，这会把 Rancher 暴露在它运行的 IP 上。生产安装中要求填写真实的域名。

默认情况是使用 Rancher 生成 CA，并使用 cert-manager 颁发用于访问 Rancher Server 接口的证书。

由于 rancher 是 ingress.tls.source 的默认选项，因此在执行 helm install 命令时，我们不需要指定 ingress.tls.source。

- 将 hostname 设置为解析到你的负载均衡器的 DNS 名称。
- 将 bootstrapPassword 设置为 admin 用户独有的值。
- 如果你需要安装指定的 Rancher 版本，使用 --version 标志，例如 --version 2.7.0。
- 对于 Kubernetes v1.25 或更高版本，使用 Rancher v2.7.2-v2.7.4 时，将 global.cattle.psp.enabled 设置为 false。对于 Rancher v2.7.5 及更高版本来说，这不是必需的，但你仍然可以手动设置该选项。

```bash
helm install rancher rancher-<CHART_REPO>/rancher \
  --namespace cattle-system \
  --set hostname=rancher.liby.org \
  --set bootstrapPassword=admin
```
如果你安装的是 alpha 版本，Helm 会要求你在安装命令中添加 --devel 选项：

```bash
helm install rancher rancher-alpha/rancher --devel
```

等待 Rancher 运行：

```bash
kubectl -n cattle-system rollout status deploy/rancher
Waiting for deployment "rancher" rollout to finish: 0 of 3 updated replicas are available...
deployment "rancher" successfully rolled out
```
Rancher Chart 有许多选项，用于为你的具体环境自定义安装。以下是一些常见的高级方案：

- [HTTP 代理](https://ranchermanager.docs.rancher.com/zh/getting-started/installation-and-upgrade/installation-references/helm-chart-options#http-%E4%BB%A3%E7%90%86)
- [私有容器镜像仓库](https://ranchermanager.docs.rancher.com/zh/getting-started/installation-and-upgrade/installation-references/helm-chart-options#%E7%A7%81%E6%9C%89%E4%BB%93%E5%BA%93%E5%92%8C%E7%A6%BB%E7%BA%BF%E5%AE%89%E8%A3%85)
- [外部负载均衡器上的 TLS 终止](https://ranchermanager.docs.rancher.com/zh/getting-started/installation-and-upgrade/installation-references/helm-chart-options#%E5%A4%96%E9%83%A8-tls-%E7%BB%88%E6%AD%A2)

如需获取完整的选项列表，请[参见 Chart 选项](https://ranchermanager.docs.rancher.com/zh/getting-started/installation-and-upgrade/installation-references/helm-chart-options)。

## 验证 Rancher Server 是否部署成功
添加密文后，检查 Rancher 是否已成功运行：

```bash
kubectl -n cattle-system rollout status deploy/rancher
Waiting for deployment "rancher" rollout to finish: 0 of 3 updated replicas are available...
deployment "rancher" successfully rolled out
```

如果你看到 `error: deployment "rancher" exceeded its progress deadline` 这个错误，可运行以下命令来检查 deployment 的状态：

```bash
kubectl -n cattle-system get deploy rancher
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
rancher   3         3         3            3           3m

$ kubectl get pod -n cattle-system
NAME                               READY   STATUS      RESTARTS   AGE
helm-operation-5wcvf               0/2     Completed   0          7m36s
helm-operation-75hp5               0/2     Completed   0          6m33s
helm-operation-8tdf2               0/2     Completed   0          8m38s
helm-operation-rlmbd               0/2     Completed   0          9m41s
helm-operation-zqwst               0/2     Completed   0          5m30s
rancher-569b86c8f5-7gfnh           1/1     Running     0          15m
rancher-569b86c8f5-ckbhx           1/1     Running     0          15m
rancher-569b86c8f5-hnjx9           1/1     Running     0          15m
rancher-webhook-788c48b988-hcgn8   1/1     Running     0          6m23s
$ kubectl get ns
NAME                                     STATUS   AGE
cattle-fleet-clusters-system             Active   8m12s
cattle-fleet-system                      Active   9m50s
cattle-global-data                       Active   10m
cattle-global-nt                         Active   10m
cattle-impersonation-system              Active   10m
cattle-system                            Active   28m
cert-manager                             Active   31m
cluster-fleet-local-local-1a3d67d0a899   Active   6m28s
default                                  Active   51d
fleet-default                            Active   10m
fleet-local                              Active   11m
ingress-nginx                            Active   5h1m
kube-node-lease                          Active   51d
kube-public                              Active   51d
kube-system                              Active   51d
local                                    Active   10m
metrics-server                           Active   40d
p-9dclm                                  Active   10m
p-mqmpd                                  Active   10m

```

