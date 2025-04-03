


##  1. Argo CD ApplicationSets  简介
[Argo CD ApplicationSets](https://argocd-applicationset.readthedocs.io/en/stable/) 是“App of Apps”部署模式的演变。它采用了“App of Apps”的理念并将其扩展为更加灵活并处理广泛的用例。ArgoCD ApplicationSets 作为自己的控制器运行，并补充了 Argo CD 应用程序 CRD 的功能。

ApplicationSet 提供以下功能：

- 使用单个清单来定位多个 Kubernetes 集群。
- 使用单个清单从单个或多个 git 存储库部署多个应用程序。
- 改进对单体存储库模式（也称为“monorepo”）的支持。这是您在单个存储库中定义许多应用程序和/或环境的地方。

在多租户集群中，它提高了集群中团队使用 Argo CD 部署应用程序的能力（无需提权）。
ApplicationSets 通过创建、更新、管理和删除 Argo CD 应用程序与 Argo CD 交互。ApplicationSets 的工作是确保 Argo CD Application 与声明的 ApplicationSet 资源保持一致。ApplicationSets 可以被认为是一种“应用程序工厂”。它接受一个 ApplicationSet 并输出一个或多个 Argo CD 应用程序。

ArgoCD ApplicationSet 来实现 GitOps 自动多环境管理，并通过 ArgoCD Generator 来达到“代码即环境”的效果。

## 2. Generators
ApplicationSet控制器由“生成器”组成。这些“生成器”指示 ApplicationSet 如何通过提供的 repo 或 repos 生成应用程序，它还指示将应用程序部署到哪里。我将探索 3 个“生成器”： 
- List Generator
- Cluster Generator
- Git Generator
- Matrix generator

每个“生成器”处理不同的场景和用例。每个“生成器”都会为您提供相同的最终结果：部署松散耦合在一起以便于管理的 Argo CD 应用程序。你使用什么取决于很多因素，比如管理的集群数量、git repo 布局和环境差异。

### 2.1 List Generator
“List Generator”根据固定列表生成 Argo CD 应用程序清单。这是最直接的，因为它只是将您在元素部分中指定的键/值传递到 ApplicatonSet 清单的模板部分。请参阅以下示例：

```bash
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: bgd
  namespace: openshift-gitops
spec:
  generators:
  - list:
      elements:
      - cluster: cluster1
        url: https://api.cluster1.chx.osecloud.com:6443
     - cluster: cluster2
        url: https://api.cluster2.chx.osecloud.com:6443
      - cluster: cluster3
        url: https://api.cluster3.chx.osecloud.com:6443
  template:
    metadata:
      name: '{{cluster}}-bgd'
    spec:
      project: default
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
      source:
        repoURL: https://github.com/christianh814/gitops-examples
        targetRevision: master
        path: applicationsets/list-generator/overlays/{{cluster}}
      destination:
        server: '{{url}}'
        namespace: bgd
```
这里 `{{cluster}}` 和 `{{url}}` 的每次迭代都将被上面的元素替换。这将产生 3 个应用程序。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b4d77f9f759cf0728bce4203cbb81346.png)
这些集群必须已经在 Argo CD 中定义，以便为这些值生成应用程序。ApplicationSet 控制器不创建集群。

您可以看到我的 ApplicationSet 向定义的每个集群部署了一个应用程序。您可以在配置中使用列表元素的任意组合。它不一定是集群或覆盖。由于这只是一个简单的键/值对生成器，您可以根据需要混合和匹配。

### 2.2 Cluster Generator
Argo CD 将其管理的集群信息存储在 Secret 中。您可以通过查看 openshift-gitops 命名空间中的机密列表来列出您的集群。

```bash
$ oc get secrets -n openshift-gitops -l argocd.argoproj.io/secret-type=cluster
NAME                                           TYPE DATA   AGE
cluster-api.cluster1.chx.osecloud.com-74873278 Opaque   3  23m
cluster-api.cluster2.chx.osecloud.com-2320437559   Opaque   3  23m
cluster-api.cluster3.chx.osecloud.com-2066075908   Opaque   3  23m
```
当您使用 argocd CLI 列出这些集群时，控制器会读取密钥以收集所需的信息。

```bash
$ argocd cluster list
SERVER                                  NAME    VERSION  STATUS  MESSAGE
https://api.cluster1.chx.osecloud.com:6443  cluster1 1.20 Successful  
https://api.cluster2.chx.osecloud.com:6443  cluster2 1.20 Successful  
https://api.cluster3.chx.osecloud.com:6443  cluster3 1.20 Successful
```
ApplicationSet 控制器也是如此。它使用这些相同的 Secret 来生成将在清单的模板部分中使用的参数。此外，您可以使用标签选择器将特定配置定位到特定集群。然后您可以标记相应的秘密。这是一个例子：

```bash
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: bgd
  namespace: openshift-gitops
spec:
  generators:
  - clusters:
      selector:
        matchLabels:
          bgd: dev
  template:
    metadata:
      name: '{{name}}-bgd'
    spec:
      project: default
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
      source:
        repoURL: https://github.com/christianh814/gitops-examples
        targetRevision: master
        path: applicationsets/cluster-generator/overlays/dev/
      destination:
        server: '{{server}}'
        namespace: bgd
```
在这里，在 `.spec.generators.clusters` 下，您可以看到我将选择器设置为 `bgd=dev`。任何匹配此标签的集群都将部署应用程序。`{{name}}` 和 `{{server}}` 资源由机密中相应的名称和服务器字段填充。
最初，当我应用它时，我什么也看不到。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/21f3b1082d21aeee19178b47e79aa6a0.png)
如果我查看控制器日志，您会看到它显示它生成了 0 个应用程序。

```bash
$ oc logs -l app.kubernetes.io/name=argocd-applicationset-controller | grep generated 
time="2021-04-01T01:25:31Z" level=info msg="generated 0 applications" generator="&{0xc000745b80 0xc000118000 0xc000afa000 openshift-gitops 0xc0009bd810 0xc000bb01e0}"
```
这是因为我们使用标签 `bgd=dev` 来指示我们要将此应用程序部署到哪个集群。让我们来看看其中的秘密。

```bash
$ oc get secrets  -l bgd=dev -n openshift-gitops
No resources found in openshift-gitops namespace.
```
让我们标记 `cluster1`，然后对其进行验证，以将此应用程序部署到该集群。


```bash
$ oc label secret cluster-api.cluster1.chx.osecloud.com-74873278 bgd=dev -n openshift-gitops
secret/cluster-api.cluster1.chx.osecloud.com-74873278 labeled

$ oc get secrets  -l bgd=dev -n openshift-gitops
NAME                                         TYPE DATA   AGE
cluster-api.cluster1.chx.osecloud.com-74873278   Opaque   3  131m
```
查看 UI，我应该会看到已部署的应用程序。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/26085be6f6e57a4529b34b0a9604284f.png)
现在要将此应用程序部署到另一个集群，您只需标记要部署到的集群的秘密即可。 

```bash
$ oc label secret cluster-api.cluster3.chx.osecloud.com-2066075908 bgd=dev -n openshift-gitops
secret/cluster-api.cluster3.chx.osecloud.com-2066075908 labeled

$ oc get secrets  -l bgd=dev -n openshift-gitops
NAME                                           TYPE DATA   AGE
cluster-api.cluster1.chx.osecloud.com-74873278 Opaque   3  135m
cluster-api.cluster3.chx.osecloud.com-2066075908   Opaque   3  135m
```
现在我已将我的应用程序部署到 cluster1 和 cluster3，我所要做的就是标记相应的秘密。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bd3a6579d85452d1888b0ca76c6504d5.png)
如果您想将所有集群作为目标，只需将 .spec.generators.clusters 设置为一个空对象 {}。下面的示例片段。

```bash
spec:
  generators:
  - clusters: {}
```
这将针对 Argo CD 管理的所有集群，包括 Argo CD 正在运行的集群，称为“in cluster”。


### 2.3 Git Generator
Git 生成器根据 Git 存储库的组织方式来确定应用程序的部署方式。Git 生成器有两个子生成器：目录和文件。

#### 2.3.1 Directory Generator
Git 目录生成器根据 git 存储库中的目录结构生成使用的参数。ApplicationSet 控制器将根据存储在存储库中特定目录中的清单创建应用程序。这是一个示例 ApplicationSet 清单。

```bash
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: pricelist
  namespace: openshift-gitops
spec:
  generators:
  - git:
      repoURL: https://github.com/christianh814/gitops-examples
      revision: master
      directories:
      - path: applicationsets/git-dir-generator/apps/*
  template:
    metadata:
      name: '{{path.basename}}'
    spec:
      project: default
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
      source:
        repoURL: https://github.com/christianh814/gitops-examples
        targetRevision: master
        path: '{{path}}'
      destination:
        server: https://api.cluster1.chx.osecloud.com:6443
        namespace: pricelist
```
此 ApplicationSet 部署了一个由 Helm 图表和 YAML 协同工作的应用程序。要了解这是如何工作的，最好看一下我的目录结构的树视图。

```bash
$ tree applicationsets/git-dir-generator/apps
applicationsets/git-dir-generator/apps
├── pricelist-config
│   ├── kustomization.yaml
│   ├── pricelist-config-ns.yaml
│   └── pricelist-config-rb.yaml
├── pricelist-db
│   ├── Chart.yaml
│   └── values.yaml
└── pricelist-frontend
    ├── kustomization.yaml
    ├── pricelist-deploy.yaml
    ├── pricelist-job.yaml
    ├── pricelist-route.yaml
    └── pricelist-svc.yaml

3 directories, 10 files
```
应用程序的名称是根据目录的名称生成的，在config中表示为`{{path.basename}}`，即`pricelist-config`、`pricelist-db`和`pricelist-frontend`。

表示为 `{{path}}` 的每个应用程序的路径将基于配置中 `.spec.generators.git.directories.path` 下定义的内容。应用此配置后，它将在 UI 中显示 3 个应用程序。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/66446897824432e7438d30d059d44dec.png)
现在，当您添加带有 Helm 图表或裸 YAML 清单的目录时，它会在您推送到跟踪的 git 存储库时自动添加。

#### 2.3.2 File Generator
Git 文件生成器是下一个子类型。该生成器也基于存储在您的 git 存储库中的内容，但它不会读取目录结构，而是读取配置文件。该文件可以任意命名，但必须是 JSON 格式。看看我的目录结构。

```bash
$ tree applicationsets/git-generator/
applicationsets/git-generator/
├── appset-bgd.yaml
├── base
│   ├── bgd-deployment.yaml
│   ├── bgd-namespace.yaml
│   ├── bgd-route.yaml
│   ├── bgd-svc.yaml
│   └── kustomization.yaml
├── cluster-config
│   ├── cluster1
│   │   └── config.json
│   ├── cluster2
│   │   └── config.json
│   └── cluster3
│       └── config.json
└── overlays
    ├── cluster1
    │   ├── bgd-deployment.yaml
    │   └── kustomization.yaml
    ├── cluster2
    │   ├── bgd-deployment.yaml
    │   └── kustomization.yaml
    └── cluster3
        ├── bgd-deployment.yaml
        └── kustomization.yaml
```
请注意，此结构包含一个 `cluster-config` 目录。在此目录中有一个 `config.json` 文件，其中包含有关如何通过提供传递给 ApplicationSets 清单中的模板所需的信息来部署应用程序的信息。您可以根据需要配置 `config.json` 文件，只要它是有效的 JSON。这是集群 1 的示例。

```bash
{
  "cluster": {
    "name": "cluster1",
    "server": "https://api.cluster1.chx.osecloud.com:6443",
    "overlay": "cluster1"
  }
}
```
基于此配置，我可以构建 ApplicationSet YAML。

```bash
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: bgd
  namespace: openshift-gitops
spec:
  generators:
  - git:
      repoURL: https://github.com/christianh814/gitops-examples
      revision: master
      files:
      - path: "applicationsets/git-generator/cluster-config/**/config.json"
  template:
    metadata:
      name: '{{cluster.name}}-bgd'
    spec:
      project: default
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
      source:
        repoURL: https://github.com/christianh814/gitops-examples
        targetRevision: master
        path: applicationsets/git-generator/overlays/{{cluster.overlay}}
      destination:
        server: '{{cluster.server}}'
        namespace: bgd
```
此配置采用您存储的配置文件（在 .spec.generators.git.files.path 部分下表示），并读取配置文件以用作模板部分的参数。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/555c3b29f8fbe9f79f92c5af80a1e084.png)
参考：
- [Getting Started with ApplicationSets](https://cloud.redhat.com/blog/getting-started-with-applicationsets)
- [ApplicationSet Controller](https://argocd-applicationset.readthedocs.io/en/stable/)
