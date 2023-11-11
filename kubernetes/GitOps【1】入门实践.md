

---
## 安装GitOps

```bash
curl --silent --location "https://github.com/weaveworks/weave-gitops/releases/download/v0.6.0/gitops-$(uname)-$(uname -m).tar.gz" | tar xz -C /tmp
sudo mv /tmp/gitops /usr/local/bin
gitops version
```
或者，macOS 用户可以使用 Homebrew：

```bash
brew tap weaveworks/tap
brew install weaveworks/tap/gitops
```
##  创建 GitHub Repositories
Weave GitOps 可用于将多个应用程序部署到多个 Kubernetes 集群，每个应用程序在自己的单独存储库中都有部署清单。为了便于管理，我们将一个或多个集群的GitOps 自动化存储在单个配置存储库中。

虽然您可以将自动化添加到任何现有存储库，包括带有应用程序部署清单的存储库，但我们建议为此目的使用新的或空的存储库，我们的指南将采用这种方法。
![在这里插入图片描述](https://img-blog.csdnimg.cn/830ffec9cc2143b099e2554d9a3abfdf.png)
##  Fork podinfo 示例应用程序存储库
们将对示例应用程序进行更改，以展示 GitOps 对账的实际效果。因此，我们将首先 fork podinfo 示例存储库。

转到[https://github.com/wego-example/podinfo-deploy](https://github.com/wego-example/podinfo-deploy)并分叉存储库。
![在这里插入图片描述](https://img-blog.csdnimg.cn/59c18921b29e4d449f97e753a837d558.png)
Podinfo是一个用 Go 编写的简单 Web 应用程序，由前端和后端组件组成；它旨在展示在 Kubernetes 中运行微服务的最佳实践。可以在[此处](https://github.com/stefanprodan/podinfo)找到完整的应用程序源。

```bash
.
├── README.md
├── backend
│   ├── deployment.yaml
│   ├── hpa.yaml
│   └── service.yaml
├── frontend
│   ├── deployment.yaml
│   └── service.yaml
└── namespace.yaml
2 directories, 7 files
```
##  kind创建集群

```bash
$ kind create cluster
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.21.1) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-kind"
You can now use your cluster with:
kubectl cluster-info --context kind-kind

Have a nice day! 👋
```
##  在 Kubernetes 集群上安装 Weave GitOps

格式
```bash
gitops install --config-repo git@github.com:<username>/gitops-config
```

```bash
gitops install --config-repo https://github.com/Ghostwritten/gitops-config.git
```
运行安装命令，指定在步骤 1 中创建的配置存储库的位置。

安装大约需要 2 分钟，具体取决于您的系统。

完成后，您将看到：

```bash
...
◎ verifying installation
✔ image-reflector-controller: deployment ready
✔ image-automation-controller: deployment ready
✔ source-controller: deployment ready
✔ kustomize-controller: deployment ready
✔ helm-controller: deployment ready
✔ notification-controller: deployment ready
✔ install finished
Deploy key generated and uploaded to git provider
► Writing manifests to disk
► Committing and pushing gitops updates for application
► Pushing app changes to repository
► Applying manifests to the cluster
```
如果报错：

```bash
✗ helm-controller: deployment not ready
✗ image-automation-controller: deployment not ready
✗ image-reflector-controller: deployment not ready
✗ kustomize-controller: deployment not ready
✗ notification-controller: deployment not ready
✗ source-controller: deployment not ready
✗ install failed
 and error: exit status 1

$ kubectl get pods -n wego-system
NAME                                           READY   STATUS             RESTARTS   AGE
helm-controller-59dcbc6dcb-8dtz7               0/1     ImagePullBackOff   0          174m
image-automation-controller-747996c677-rnhlq   0/1     ImagePullBackOff   0          174m
image-reflector-controller-f55d746df-q2742     0/1     ImagePullBackOff   0          174m
kustomize-controller-5b7b8b44f6-4t9h7          0/1     ErrImagePull       0          174m
notification-controller-77f68bf8f4-hlk8w       0/1     ImagePullBackOff   0          174m
source-controller-679665b8d6-5l2kq             0/1     ErrImagePull       0          174m

```
解决方法：

```bash
$ for i in `kubectl get pods -n wego-system   -o jsonpath='{.items[*].spec.containers[0].image}'`;do docker pull  $i;done

$ for i in `kubectl get pods -n wego-system   -o jsonpath='{.items[*].spec.containers[0].image}'`;do kind load  docker-image $i --name kind ;done

#检查运行pod状态
$ kubectl get pods -n wego-system
NAME                                           READY   STATUS    RESTARTS   AGE
helm-controller-59dcbc6dcb-8dtz7               1/1     Running   0          3h31m
image-automation-controller-747996c677-rnhlq   1/1     Running   0          3h31m
image-reflector-controller-f55d746df-q2742     1/1     Running   0          3h31m
kustomize-controller-5b7b8b44f6-4t9h7          1/1     Running   0          3h31m
notification-controller-77f68bf8f4-hlk8w       1/1     Running   0          3h31m
source-controller-679665b8d6-5l2kq             1/1     Running   0          3h31m

```

这将向`.weave-gitops`您的配置存储库提交一个新文件夹，其中包含以下文件以管理指定集群上的 Weave GitOps 运行时：

```bash
.
└── clusters
    └── kind-kind
        ├── system
        │   ├── flux-source-resource.yaml
        │   ├── flux-system-kustomization-resource.yaml
        │   ├── flux-user-kustomization-resource.yaml
        │   ├── gitops-runtime.yaml
        │   ├── wego-app.yaml
        │   └── wego-system.yaml
        └── user
            └── .keep
```

 - `flux-source-resource`：一个[GitRepository](https://fluxcd.io/docs/concepts/#sources)源，它“定义了包含系统所需状态和获取它的要求的存储库的来源”。这包括`interval`检查可用新版本的频率。
 - `flux-system-kustomization-resource`：一个[Flux Kustomization](https://fluxcd.io/docs/concepts/#kustomization)，它“代表了 Flux应该在集群中协调的一组本地 Kubernetes 资源（例如 `kustomize`覆盖）”。这将部署在指定路径下找到的资源，在本例中为`/system`文件夹，在集群和 `Git` 中声明的状态之间进行协调。其中“如果您使 `kubectl edit/patch/delete` 对集群进行任何更改，它们将被立即还原。” 基于`interval`值。
 - `flux-user-kustomization-resource`：另一个[Flux Kustomization](https://fluxcd.io/docs/concepts/#kustomization)，这次是`/user`文件夹中的任何内容，本指南后面将包含对我们示例应用程序的引用。
 - `gitops-runtime`: 创建`wego-system`命名空间并部署 `Flux` 运行时。
 - `wego-app`：它部署了我们的集群上 Web UI（当前未公开）。
 - `wego-system`：它创建了我们的应用程序自定义资源定义 (CRD)

要了解有关这些文件的更多信息，请参阅[GitOps 自动化](https://docs.gitops.weave.works/docs/gitops-automation/index.html)。

##  启动 GitOps Dashboard Web UI 
Weave GitOps 提供了一个 Web UI 来帮助管理应用程序的生命周期管理。

```bash
gitops ui run
```
运行上述命令将在浏览器中打开位于`http://0.0.0.0:9001/`的仪表板。

您将看到一个空的应用程序视图，如下图所示。

