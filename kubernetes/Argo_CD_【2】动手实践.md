

---

 - [GitOps【1】理论认识](https://blog.csdn.net/xixihahalelehehe/article/details/122193489?spm=1001.2014.3001.5501)
 - [Argo CD【1】介绍与入门](https://blog.csdn.net/xixihahalelehehe/article/details/122238344?spm=1001.2014.3001.5501)
 - [Argo CD 【2】动手实践](https://blog.csdn.net/xixihahalelehehe/article/details/122242023?spm=1001.2014.3001.5501)

---

##  1. kind部署k8s

```bash
$ cat kind-config.yaml 
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
- role: worker

$ kind create cluster --config=kind-config.yaml 
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.21.1) 🖼 
 ✓ Preparing nodes 📦 📦 📦 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
 ✓ Joining worker nodes 🚜 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Thanks for using kind! 😊


$ kubectl wait --for=condition=Ready nodes --all
node/kind-control-plane condition met
node/kind-worker condition met
node/kind-worker2 condition met
node/kind-worker3 condition met

$ kubectl get nodes
NAME                 STATUS   ROLES                  AGE     VERSION
kind-control-plane   Ready    control-plane,master   2m42s   v1.21.1
kind-worker          Ready    <none>                 2m17s   v1.21.1
kind-worker2         Ready    <none>                 2m17s   v1.21.1
kind-worker3         Ready    <none>                 2m17s   v1.21.1

```

##  2. 部署 Argo CD
这里可以直接使用 Argo CD 项目中提供的部署文件进行安装。这里需要注意的是 此部署文件中 RBA 的配置中引用了 argocd 这个 namespace，所以如果你是将它部署到其他 namespace 中，那一定要进行对应的修改。

```bash
$ kubectl create ns argocd
namespace/argocd created

$ kubectl -n argocd apply -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
customresourcedefinition.apiextensions.k8s.io/applications.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/appprojects.argoproj.io created
serviceaccount/argocd-application-controller created
serviceaccount/argocd-dex-server created
serviceaccount/argocd-redis created
serviceaccount/argocd-server created
role.rbac.authorization.k8s.io/argocd-application-controller created
role.rbac.authorization.k8s.io/argocd-dex-server created
role.rbac.authorization.k8s.io/argocd-server created
clusterrole.rbac.authorization.k8s.io/argocd-application-controller created
clusterrole.rbac.authorization.k8s.io/argocd-server created
rolebinding.rbac.authorization.k8s.io/argocd-application-controller created
rolebinding.rbac.authorization.k8s.io/argocd-dex-server created
rolebinding.rbac.authorization.k8s.io/argocd-redis created
rolebinding.rbac.authorization.k8s.io/argocd-server created
clusterrolebinding.rbac.authorization.k8s.io/argocd-application-controller created
clusterrolebinding.rbac.authorization.k8s.io/argocd-server created
configmap/argocd-cm created
configmap/argocd-cmd-params-cm created
configmap/argocd-gpg-keys-cm created
configmap/argocd-rbac-cm created
configmap/argocd-ssh-known-hosts-cm created
configmap/argocd-tls-certs-cm created
secret/argocd-secret created
service/argocd-dex-server created
service/argocd-metrics created
service/argocd-redis created
service/argocd-repo-server created
service/argocd-server created
service/argocd-server-metrics created
deployment.apps/argocd-dex-server created
deployment.apps/argocd-redis created
deployment.apps/argocd-repo-server created
deployment.apps/argocd-server created
statefulset.apps/argocd-application-controller created
networkpolicy.networking.k8s.io/argocd-application-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-dex-server-network-policy created
networkpolicy.networking.k8s.io/argocd-redis-network-policy created
networkpolicy.networking.k8s.io/argocd-repo-server-network-policy created
networkpolicy.networking.k8s.io/argocd-server-network-policy created

```

查看状态

```bash
$ kubectl -n argocd get deploy
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
argocd-dex-server    0/1     1            0           82s
argocd-redis         1/1     1            1           82s
argocd-repo-server   0/1     1            0           82s
argocd-server        0/1     1            0           82s

$ kubectl get pods -n argocd
NAME                                  READY   STATUS            RESTARTS   AGE
argocd-application-controller-0       1/1     Running           0          3m20s
argocd-dex-server-76c978c87-8zz57     0/1     PodInitializing   0          3m21s
argocd-redis-5b6967fdfc-tpmqn         1/1     Running           0          3m21s
argocd-repo-server-8555f94d4f-k945r   1/1     Running           0          3m21s
argocd-server-bc59fd78c-c7xcd         1/1     Running           0          3m21s
```


获取密码：
默认情况下安装好的 Argo CD 会启用基于 Basic Auth的身份校验，我们可以在 Secret 资源中找到对应的密码。**但需要注意的是 这个名字为 `argocd-initial-admin-secret 的 sercret` 资源是等到 Pod 处于 Running 状态后才会写入**。

```bash
$ kubectl wait --for=condition=Ready pods --all -n argocd
pod/argocd-application-controller-0 condition met
pod/argocd-dex-server-5fc596bcdd-lnx65 condition met
pod/argocd-redis-5b6967fdfc-mfbrr condition met
pod/argocd-repo-server-98598b6c7-7pmgb condition met
pod/argocd-server-5b4b7b868b-bjmzz condition met

# 获取密码
$ kubectl  -n argocd get secret argocd-initial-admin-secret -o template="{{ .data.password | base64decode }}" 
-xuDhfC7ZHaJtI12

或者
$ kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
-xuDhfC7ZHaJtI12

```

> 在修改密码后，应该从Argo CD命名空间中删除`argocd-initial-admin-secret`。该秘密服务没有其他目的，只是存储初始生成的密码，并可以在任何时候安全地删除。如果必须重新生成新的管理员密码，它将在Argo CD的要求下重新创建。

##  3. 连接 Argo CD API Server
有三种方式：
### 3.1 Service Type Load Balancer¶
将“argocd-server”服务类型修改为“LoadBalancer”

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```
###  3.2 Ingress
参考[ingress文档](https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/)，了解如何使用ingress配置Argo CD。
###  3.3 端口转发
我们可以通过 `kubectl port-forward` 将 `argocd-server` 的 443 端口映射到本地的 9080  端口。

```bash
$ kubectl port-forward --address 0.0.0.0 service/argocd-server -n argocd 9080:443
```

我们选择端口转发

这样在浏览器中就可以 `ArgoCD dashboard` ，这是 `username` 是 `admin`,  以及 password 便可以前面提到的『获取密码』章节 。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7582391121c4941a2792650ce71f3f26.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d84eaf6727268bc667dc06bb73fd8fa0.png)
命令行访问：
如果你不喜欢通过浏览器进行操作，那也可以使用 Argo CD 提供的 CLI 工具。

```bash
 wget https://github.com/argoproj/argo-cd/releases/download/v2.2.1/argocd-linux-amd64
 chmod 755 argocd-linux-amd64
 mv argocd-linux-amd64 /usr/local/bin/argocd

```
登陆

```bash
$ argocd login localhost:9080
WARNING: server certificate had error: x509: certificate signed by unknown authority. Proceed insecurely (y/n)? y
Username: admin
Password: 
'admin:login' logged in successfully
Context 'localhost:9080' updated
```
命令行修改密码

```bash
$ argocd account update-password
*** Enter password of currently logged in user (admin): 
*** Enter new password for user admin: 
*** Confirm new password for user admin: 
Password updated
Context 'localhost:9080' updated
```

##  4. 部署应用
[示例项目](https://github.com/Ghostwritten/argo-cd-demo)

###  4.1 创建 app

创建目标 namespace

```bash
kubectl  create ns kustomize
```

命令行创建app
```bash
argocd app create argo-cd-demo --repo https://github.com/Ghostwritten/argo-cd-demo.git --revision kustomize --path ./kustomization --dest-server https://kubernetes.default.svc --dest-namespace kustomize 
```
其中：

 - `--repo` 指定部署应用所使用的仓库地址；
 - `--revision` 指定部署应用所使用的分支，这里我使用了一个名为 kustomize 的分支；
 - `--path` 部署应用程序用到的 manifest 所在的位置
 - `--dest-server` 目标 Kubernetes 集群的地址
 - `--dest-namespace` 应用要部署的目标 namespace

UI界面创建app
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c5e883f24e51fd03e7fcf1f6e0c32bf9.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ede0a78ada1b203febe8516d595874fc.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7afdab06540d65e8cc9b7b4296b5326c.png)
### 4.2 查看状态
当 Application 创建完成后，也可以直接在 UI 上看到具体信息：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/920a96af37fb5eb8bc49e6689864330f.png)
或者通过 argocd 在终端下进行查看：

```bash
$ argocd app get argo-cd-demo
Name:               argo-cd-demo
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          kustomize
URL:                https://localhost:9080/applications/argo-cd-demo
Repo:               https://github.com/Ghostwritten/argo-cd-demo.git
Target:             kustomize
Path:               ./kustomization
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        OutOfSync from kustomize (2c8f387)
Health Status:      Missing

GROUP  KIND        NAMESPACE  NAME          STATUS     HEALTH   HOOK  MESSAGE
       Service     kustomize  argo-cd-demo  OutOfSync  Missing        
apps   Deployment  kustomize  argo-cd-demo  OutOfSync  Missing        
```
可以看到当前的 Application 状态是 OutOfSync ，所以我们可以为它触发一次 sync 操作，进行首次部署。

###  4.3 sync
可以在 UI 上点击 SYNC 按钮，或者通过 argocd CLI 来触发同步操作。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3c7a9f1e24ec2a1eef9037b07f43201b.png)
命令：

```bash
$ argocd app sync argo-cd-demo
TIMESTAMP                  GROUP        KIND   NAMESPACE                  NAME    STATUS    HEALTH        HOOK  MESSAGE
2021-12-30T23:57:55+08:00            Service   kustomize          argo-cd-demo  OutOfSync  Missing              
2021-12-30T23:57:55+08:00   apps  Deployment   kustomize          argo-cd-demo  OutOfSync  Missing              
2021-12-30T23:57:55+08:00            Service   kustomize          argo-cd-demo  OutOfSync  Missing              service/argo-cd-demo created
2021-12-30T23:57:55+08:00   apps  Deployment   kustomize          argo-cd-demo  OutOfSync  Missing              deployment.apps/argo-cd-demo created

Name:               argo-cd-demo
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          kustomize
URL:                https://localhost:9080/applications/argo-cd-demo
Repo:               https://github.com/Ghostwritten/argo-cd-demo.git
Target:             kustomize
Path:               ./kustomization
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        Synced to kustomize (2c8f387)
Health Status:      Progressing

Operation:          Sync
Sync Revision:      2c8f387b4e5b121146330800fb0950df98ce9056
Phase:              Succeeded
Start:              2021-12-30 23:57:55 +0800 CST
Finished:           2021-12-30 23:57:55 +0800 CST
Duration:           0s
Message:            successfully synced (all tasks run)

GROUP  KIND        NAMESPACE  NAME          STATUS  HEALTH       HOOK  MESSAGE
       Service     kustomize  argo-cd-demo  Synced  Healthy            service/argo-cd-demo created
apps   Deployment  kustomize  argo-cd-demo  Synced  Progressing        deployment.apps/argo-cd-demo created
```
同步成功后，在 UI 上也能看到当前应用和同步的状态。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0198daa07b2af60e884bed06a171b115.png)
点击查看详情，可以看到应用部署的拓扑结构：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0706c6ca29c94bf1f46a8d651c6901f1.png)


命令行：

```bash
$ argocd app get argo-cd-demo
Name:               argo-cd-demo
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          kustomize
URL:                https://localhost:9080/applications/argo-cd-demo
Repo:               https://github.com/Ghostwritten/argo-cd-demo.git
Target:             kustomize
Path:               ./kustomization
SyncWindow:         Sync Allowed
Sync Policy:        <none>
Sync Status:        Synced to kustomize (2c8f387)
Health Status:      Healthy

GROUP  KIND        NAMESPACE  NAME          STATUS  HEALTH   HOOK  MESSAGE
       Service     kustomize  argo-cd-demo  Synced  Healthy        service/argo-cd-demo created
apps   Deployment  kustomize  argo-cd-demo  Synced  Healthy        deployment.apps/argo-cd-demo created


$ kubectl get all -n kustomize
NAME                               READY   STATUS    RESTARTS   AGE
pod/argo-cd-demo-7b69cdbcb-7czdj   1/1     Running   0          9m18s
pod/argo-cd-demo-7b69cdbcb-p5p47   1/1     Running   0          9m18s

NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/argo-cd-demo   ClusterIP   10.96.201.170   <none>        8888/TCP   9m19s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/argo-cd-demo   2/2     2            2           9m19s

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/argo-cd-demo-7b69cdbcb   2         2         2       9m19s

```

##   5. 验证效果

###  5.1 修改代码
接下来在 kustomize 分支，进行一些代码上的修改，并提交到 GitHub 上。此时会触发项目中基于 GitHub Action 的 CI，我们来看看其具体的配置：

```bash
 deploy:
    name: Deploy
    runs-on: ubuntu-latest
    continue-on-error: true
    needs: build

    steps:
      - name: Check out code
        uses: actions/checkout@v2

      - name: Setup Kustomize
        uses: imranismail/setup-kustomize@v1
        with:
          kustomize-version: "4.3.0"

      - name: Update Kubernetes resources
        env:
          DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
        run: |-
          cd manifests
          kustomize edit set image ghcr.io/${{ github.repository }}/argo-cd-demo:${{ github.sha }}
          cat kustomization.yaml
          kustomize build ./ > ../kustomization/manifests.yaml
          cat ../kustomization/manifests.yaml

      - uses: EndBug/add-and-commit@v7
        with:
          default_author: github_actions
          branch: kustomize
```
可以看到这里其实利用了 kustomize 这个工具，将最新的镜像写入到了部署应用所用的 `manifest.yaml` 文件中了，然后利用 `EndBug/add-and-commit@v7` 这个 action 将最新的 `manifest.yaml` 文件再提交回 GitHub 中。

### 5.2 查看状态
此时当 Sync 再次触发后，我们也就可以看到最新的部署拓扑了。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dcf66a672f59120b08052d94056a6703.png)

参考链接：
[https://mp.weixin.qq.com/s/E4OOiHKhUBV-pykkZEP5Ng](https://mp.weixin.qq.com/s/E4OOiHKhUBV-pykkZEP5Ng)
