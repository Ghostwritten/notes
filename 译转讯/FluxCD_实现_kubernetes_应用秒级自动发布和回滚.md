#  FluxCD 实现 kubernetes 应用秒级自动发布和回滚
tags: 实践

![](https://i-blog.csdnimg.cn/blog_migrate/f8d4381a7440c9066abdb8d6a39fc097.png)





在上一篇中：[Kubernetes 实现自动扩容和自愈应用实践](https://blog.csdn.net/xixihahalelehehe/article/details/128415006)

我通过kind 部署一套kubernetes 集群
创建一个deployment应用
创建 service
创建 ingress-nginx
创建 metric-server

测试了 kubernetes 机制的自愈与自动扩容。下面我体验 GitOps 在应用发布，再次之前我们熟悉一下传统 kuberentes 应用发布。
##  1. 传统 K8s 应用发布流程
- 使用 kubectl set image 命令；
- 修改本地的 Manifest ；
- 修改集群内 Manifest 。

### 1.1 通过 `kubectl set image` 命令更新应用

```bash
$ kubectl set image deployment/hello-world-flask hello-world-flask=ghostwritten/hello-world-flask:v1
deployment.apps/hello-world-flask image updated
```
新镜像版本修改了 Python 的返回内容，你可以直接使用。当 K8s 接收到镜像更新的指令时，K8s 会用新的镜像版本重新创建 Pod。你可以使用 kubectl get pods 来查看 Pod 的更新情况：

```bash
$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
hello-world-flask-8f94845dc-qsm8b   1/1     Running   0          3m38s
hello-world-flask-8f94845dc-spd6j   1/1     Running   0          3m21s
hello-world-flask-64dd645c57-rfhw5   0/1     ContainerCreating   0          1s
hello-world-flask-64dd645c57-ml74f   0/1     ContainerCreating   0          0s
```
在更新 Pod 的过程中，K8s 会确保先创建新的 Pod ，然后再终止旧镜像版本的 Pod。
你可以打开浏览器访问 127.0.0.1 ，查看返回内容：

```bash
Hello, my v1 version docker images! hello-world-flask-8f94845dc-bpgnp
```
更新镜像成功。
从本质上来看，`kubectl set image` 是修改了集群内已部署的 Deployment 工作负载的 Image 字段，继而触发了 K8s 对 Pod 的变更。有时候，我们在一次发布中希望变更的内容不仅仅是镜像版本，可能还有副本数、端口号等等。这时候，我们可以对新的 Manifest 文件再执行一次 `kubectl apply` ，K8s 会比对它们之间的差异，然后做出变更。

###  1.2 通过修改本地的 Manifest 更新应用
重新把镜像版本修改为 `latest`，创建`new-hello-world-flask.yaml`

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: hello-world-flask
  name: hello-world-flask
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-world-flask
  template:
    metadata:
      labels:
        app: hello-world-flask
    spec:
      containers:
      - image: lyzhang1999/hello-world-flask:latest
        name: hello-world-flask
```
更新：

```bash
kubectl apply -f new-hello-world-flask.yaml
```
kubectl apply 命令会自动处理两种情况：

-  kubectl apply 命令会自动处理两种情况：
- 如果资源存在，那就更新资源。

###  1.3 通过修改集群内 Manifest 更新应用

```bash
$ kubectl edit deployment hello-world-flask
```
在实际项目的实践中，负责更新应用的同学早期可能会在自己的电脑上操作，然后把这部分操作挪到 CI 过程，例如使用 `Jenkins` 来执行。但是，随着项目的发展，我们会需要发布流程更加自动化、安全、可追溯。这时候，我们应该考虑用 `GitOps` 的方式来发布应用。


## 2. 从零搭建 GitOps 发布工作流
GitOps 就是以 Git 版本控制为理念的 DevOps 实践。我们会将 Manifest 存储在 Git 仓库中作为期望状态，一旦修改并提交了 Manifest ，那么 GitOps 工作流就会自动比对 Git 仓库和集群内工作负载的实际差异，并进行部署。

###  2.1 安装 FluxCD 并创建工作流
要实现 GitOps 工作流，首先我们需要一个能够帮助我们监听 Git 仓库变化，自动部署的工具。这节课，我以 FluxCD 为例，带你一步步构建出一个 GitOps 工作流。

首先，我们需要在集群内安装 FluxCD：

```bash
 kubectl apply -f https://ghproxy.com/https://raw.githubusercontent.com/lyzhang1999/resource/main/fluxcd/fluxcd.yaml
```
由于安装 FluxCD 的工作负载比较多，你可以使用 `kubectl wait` 来等待安装完成：

```bash
$ kubectl wait --for=condition=available --timeout=300s --all deployments -n flux-system
deployment.apps/helm-controller condition met
deployment.apps/image-automation-controller condition met
deployment.apps/image-reflector-controller condition met
deployment.apps/kustomize-controller condition met
deployment.apps/notification-controller condition met
deployment.apps/source-controller condition met
```
接下来，在本地创建 `fluxcd-demo` 目录：

```bash
mkdir fluxcd-demo && cd fluxcd-demo
```
然后，我们在 fluxcd-demo 目录下创建 `deployment.yaml` 文件，并将下面的内容保存到这个文件里：

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: hello-world-flask
  name: hello-world-flask
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-world-flask
  template:
    metadata:
      labels:
        app: hello-world-flask
    spec:
      containers:
      - image: ghostwritten/hello-world-flask:latest
        name: hello-world-flask
```
最后，在 Github 或 Gitlab 中创建 `fluxcd-demo` 仓库。为了方便测试，我们需要将仓库设置为公开权限，主分支为 `Main`，并将我们创建的 Manifest 推送至远端仓库：

```bash
$ ls
deployment.yaml
$ git init
......
Initialized empty Git repository in /root/kind/fluxcd-demo/.git/
$ git add -A && git commit -m "Add deployment"
[master (root-commit) 538f858] Add deployment
 1 file changed, 19 insertions(+)
 create mode 100644 deployment.yaml
$ git branch -M main
$ git remote add origin https://github.com/ghostwritten/fluxcd-demo.git
$ git push -u origin main
```
这是我的[仓库地址，](https://github.com/Ghostwritten/fluxcd-demo)你可以参考一下。下一步，我们为 `FluxCD` 创建仓库连接信息，将下面的内容保存为 `fluxcd-repo.yaml`：

```bash
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: GitRepository
metadata:
  name: hello-world-flask
spec:
  interval: 5s
  ref:
    branch: main
  url: https://github.com/ghostwritten/fluxcd-demo
```
注意，要将 URL 字段修改为你实际仓库的地址并使用 `HTTPS` 协议，branch 字段设置 `main` 分支。这里的 `interval` 代表每 5 秒钟主动拉取一次仓库并把它作为制品存储。
接着，使用 `kubectl apply` 将其 GitRepository 对象部署到集群内：

```bash
$ kubectl apply -f fluxcd-repo.yaml
gitrepository.source.toolkit.fluxcd.io/hello-world-flask created
```
你可以使用 `kubectl get gitrepository` 来检查配置是否生效：

```bash
$ kubectl get gitrepository
NAME                URL                                           AGE     READY   STATUS
hello-world-flask   https://github.com/ghostwritten/fluxcd-demo   4m36s   True    stored artifact for revision 'main/6bb7e63390ccb65b5c75c612142996029606684f'
```
接下来，我们还需要为 `FluxCD` 创建部署策略。将下面的内容保存为 `fluxcd-kustomize.yaml`：

```bash
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: hello-world-flask
spec:
  interval: 5s
  path: ./
  prune: true
  sourceRef:
    kind: GitRepository
    name: hello-world-flask
  targetNamespace: default
```
在上面的配置中，`interval` 参数表示 FluxCD 会每 5 秒钟运行一次工作负载差异对比，path 参数表示我们的 `deployment.yaml` 位于仓库的根目录中。FluxCD 在对比期望状态和集群实际状态的时候，如果发现差异就会触发重新部署。

```bash
$ kubectl apply -f fluxcd-kustomize.yaml
kustomization.kustomize.toolkit.fluxcd.io/hello-world-flask created
```
同样地，你可以使用 `kubectl get kustomization` 来检查配置是否生效：

```bash
$ kubectl get kustomization
NAME                AGE     READY   STATUS
hello-world-flask   8m21s   True    Applied revision: main/8260f5a0ac1e4ccdba64e074d1ee2c154956f12d
```
配置完成后，接下来，我们就可以正式**体验 GitOps 的秒级自动发布和回滚**了。

### 2.2 自动发布
现在，我们修改 `fluxcd-demo` 仓库的 `deployment.yaml` 文件，将 `image` 字段的镜像版本从 `latest` 修改为 `v1`：

```bash
......
    spec:
      containers:
      - image: ghostwritten/hello-world-flask:v1 # 修改此处
        name: hello-world-flask
......
```
然后，我们将修改推送到远端仓库：

```bash
$ git add -A && git commit -m "Update image tag to v1"
$ git push origin main
```
你可以使用 k`ubectl describe kustomization hello-world-flask` 查看触发重新部署的事件：

```bash
NAME                AGE   READY   STATUS
hello-world-flask   10m   True    Applied revision: main/40d38a5168ccc481c40cdf532e17be8e5d8db0d7
[root@kind1 fluxcd-demo]# k describe kustomization hello-world-flask
Name:         hello-world-flask
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  kustomize.toolkit.fluxcd.io/v1beta2
Kind:         Kustomization
Metadata:
  Creation Timestamp:  2022-12-25T07:30:03Z
  Finalizers:
    finalizers.fluxcd.io
  Generation:  1
  Managed Fields:
    API Version:  kustomize.toolkit.fluxcd.io/v1beta2
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:finalizers:
          .:
          v:"finalizers.fluxcd.io":
    Manager:      gotk-kustomize-controller
    Operation:    Update
    Time:         2022-12-25T07:30:03Z
    API Version:  kustomize.toolkit.fluxcd.io/v1beta2
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .:
          f:kubectl.kubernetes.io/last-applied-configuration:
      f:spec:
        .:
        f:force:
        f:interval:
        f:path:
        f:prune:
        f:sourceRef:
          .:
          f:kind:
          f:name:
        f:targetNamespace:
    Manager:      kubectl-client-side-apply
    Operation:    Update
    Time:         2022-12-25T07:30:03Z
    API Version:  kustomize.toolkit.fluxcd.io/v1beta2
    Fields Type:  FieldsV1
    fieldsV1:
      f:status:
        f:conditions:
        f:inventory:
          .:
          f:entries:
        f:lastAppliedRevision:
        f:lastAttemptedRevision:
        f:observedGeneration:
    Manager:         gotk-kustomize-controller
    Operation:       Update
    Subresource:     status
    Time:            2022-12-25T07:40:13Z
  Resource Version:  863001
  UID:               7b8c805e-9304-4b31-9b39-f9da0d50818d
Spec:
  Force:     false
  Interval:  5s
  Path:      ./
  Prune:     true
  Source Ref:
    Kind:            GitRepository
    Name:            hello-world-flask
  Target Namespace:  default
Status:
  Conditions:
    Last Transition Time:  2022-12-25T07:40:13Z
    Message:               Applied revision: main/40d38a5168ccc481c40cdf532e17be8e5d8db0d7
    Reason:                ReconciliationSucceeded
    Status:                True
    Type:                  Ready
  Inventory:
    Entries:
      Id:                   default_hello-world-flask_apps_Deployment
      V:                    v1
  Last Applied Revision:    main/40d38a5168ccc481c40cdf532e17be8e5d8db0d7
  Last Attempted Revision:  main/40d38a5168ccc481c40cdf532e17be8e5d8db0d7
  Observed Generation:      1
Events:
  Type    Reason                   Age                    From                  Message
  ----    ------                   ----                   ----                  -------
  Normal  Progressing              10m                    kustomize-controller  Deployment/default/hello-world-flask configured
  Normal  ReconciliationSucceeded  10m                    kustomize-controller  Reconciliation finished in 152.673132ms, next run in 5s
  Normal  ReconciliationSucceeded  10m                    kustomize-controller  Reconciliation finished in 72.890257ms, next run in 5s
  Normal  ReconciliationSucceeded  10m                    kustomize-controller  Reconciliation finished in 83.435609ms, next run in 5s
  Normal  ReconciliationSucceeded  9m59s                  kustomize-controller  Reconciliation finished in 81.227343ms, next run in 5s
  Normal  ReconciliationSucceeded  9m54s                  kustomize-controller  Reconciliation finished in 80.675433ms, next run in 5s
  Normal  ReconciliationSucceeded  9m49s                  kustomize-controller  Reconciliation finished in 69.683234ms, next run in 5s
  Normal  ReconciliationSucceeded  9m44s                  kustomize-controller  Reconciliation finished in 71.393216ms, next run in 5s
  Normal  ReconciliationSucceeded  9m39s                  kustomize-controller  Reconciliation finished in 78.597139ms, next run in 5s
  Normal  ReconciliationSucceeded  9m34s                  kustomize-controller  Reconciliation finished in 75.629941ms, next run in 5s
  Normal  ReconciliationSucceeded  10s (x112 over 9m29s)  kustomize-controller  (combined from similar events): Reconciliation finished in 76.895861ms, next run in 5s
```
从返回的结果可以看出，我们将镜像版本修改为了 `v1`，并且，FluxCD 最后一次部署仓库的 `Commit ID` 是 `40d38a5168ccc481c40cdf532e17be8e5d8db0d7`，这对应了我们最后一次的提交记录，说明变更已经生效了。

现在，我们访问 `127.0.0.1`，可以看到 `v1` 镜像版本的输出内容：

```bash
$ curl 127.0.0.1
Hello, my v1 version docker images! hello-world-flask-99c86b7d8-6zt48
```
通过上面的配置，我们让 FluxCD 自动完成了监听修改、比较和重新部署三个过程。怎么样，GitOps 的发布流程是不是比手动发布方便多了呢？接下来我们再感受一下 **GitOps 的快速回滚能力**。

### 2.3 发布回滚
要回滚 `fluxcd-demo` 仓库，首先需要找到上一次的提交记录。我们可以使用 `git log` 来查看它：

```bash
git log
commit 40d38a5168ccc481c40cdf532e17be8e5d8db0d7 (HEAD -> main, origin/main)
Author: ghostwritten <1zoxun1@gmail.com>
Date:   Sun Dec 25 15:39:18 2022 +0800

    Update image tag to v1

commit 6bb7e63390ccb65b5c75c612142996029606684f
Author: ghostwritten <1zoxun1@gmail.com>
Date:   Sun Dec 25 15:18:26 2022 +0800

    Add deployment
```
可以看到，上一次的 `commit id` 为 `6bb7e63390ccb65b5c75c612142996029606684f`，接下来使用 `git reset` 来回滚到上一次提交，并强制推送到 Git 仓库：

```bash
$ git reset --hard 6bb7e63390ccb65b5c75c612142996029606684f
HEAD is now at 6bb7e63 Add deployment

$ git push origin main -f
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
remote: This repository moved. Please use the new location:
remote:   https://github.com/Ghostwritten/fluxcd-demo.git
To https://github.com/ghostwritten/fluxcd-demo.git
 + 40d38a5...6bb7e63 main -> main (forced update)
```
再次使用 `kubectl describe kustomization hello-world-flask` 查看触发重新部署的事件：

```bash

$ kubectl describe kustomization hello-world-flask
..........
  Force:     false
  Interval:  5s
  Path:      ./
  Prune:     true
  Source Ref:
    Kind:            GitRepository
    Name:            hello-world-flask
  Target Namespace:  default
Status:
  Conditions:
    Last Transition Time:  2022-12-25T07:48:32Z
    Message:               Applied revision: main/6bb7e63390ccb65b5c75c612142996029606684f
    Reason:                ReconciliationSucceeded
    Status:                True
    Type:                  Ready
  Inventory:
    Entries:
      Id:                   default_hello-world-flask_apps_Deployment
      V:                    latest
  Last Applied Revision:    main/6bb7e63390ccb65b5c75c612142996029606684f
  Last Attempted Revision:  main/6bb7e63390ccb65b5c75c612142996029606684f
  Observed Generation:      1
......
```
从返回结果的 `Last Applied Revision` 可以看出，FluxCD 已经检查到了变更，并已经进行了同步。再次打开浏览器访问 `127.0.0.1`，可以看到返回结果已回滚到了 `latest` 镜像对应的内容：

```bash
Hello, my first docker images! hello-world-flask-56fbff68c8-c8dc4
```
到这里，我们就成功实现了 `GitOps` 的发布和回滚。
