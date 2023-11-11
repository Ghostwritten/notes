

##  1. 条件

 - Kubernetes 集群

有关Helm 和 Kubernetes 之间支持的最大版本偏差，请[参阅Helm 版本支持策略](https://www.bookstack.cn/read/helm-3.8.0-en/625996e4776bdd15.md)。

##  2. 安装
### 2.1 二进制版本安装
helm 的每个[版本](https://github.com/helm/helm/releases)都为各种操作系统提供二进制版本

```bash
$ wget https://get.helm.sh/helm-v3.8.0-linux-amd64.tar.gz
$ tar -xzvf helm-v3.8.0-linux-amd64.tar.gz
$ cp linux-amd64/helm /usr/local/bin/
$ helm version
version.BuildInfo{Version:"v3.8.0", GitCommit:"d14138609b01886f544b2025f5000351c9eb092e", GitTreeState:"clean", GoVersion:"go1.17.5"}
```
###  2.2 脚本安装
Helm 现在有一个安装程序脚本，它会自动获取最新版本的 Helm 并在[本地安装](https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3)它。

```bash
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```
###  2.3 更多安装方式
[更多安装方式请参考这里](https://www.bookstack.cn/read/helm-3.8.0-en/6d3a31117a899d97.md)

##  3. 三大概念

 - `Chart`是一个Helm 包。它包含在 Kubernetes 集群内运行应用程序、工具或服务所需的所有资源定义。可以把它想象成 Kubernetes 的 Homebrew 公式、Apt dpkg 或 Yum RPM 文件。
 - `Repository` 是可以收集和共享图表的地方。它类似于 Perl 的CPAN 存档或Fedora 包数据库，但用于 Kubernetes 包。
 - `Release`是在 Kubernetes 集群中运行的Chart的实例。一个Chart通常可以多次安装到同一个集群中。每次安装时，都会创建一个新Release。考虑一个 MySQL Chart。如果您希望在集群中运行两个数据库，则可以将该Chart安装两次。每个都有自己的发行版，而发行版又会有自己的发行版名称。

##  4. 常用方法
### 4.1 'helm repo'：使用存储库
检查[Artifact Hub](https://artifacthub.io/packages/search?kind=0)以获取可用的 Helm 图表存储库。

```bash
$ helm repo list
NAME            URL
stable          https://charts.helm.sh/stable
mumoshu         https://mumoshu.github.io/charts

$ helm repo add bitnami https://charts.bitnami.com/bitnami
```
由于图表存储库经常更改，因此您可以随时通过运行`helm repo update`.

可以使用 删除存储库`helm repo remove`。

###  4.2  'helm search': 查找图表

 - `helm search hub`搜索[Artifact Hub](https://artifacthub.io/)，其中列出了来自数十个不同存储库的 helm 图表。
 - `helm search repo`搜索您添加到本地 helm 客户端的存储库（使用helm repo
   add）。此搜索是在本地数据上完成的，不需要公共网络连接。

```bash
$ helm search hub wordpress
URL                                                 CHART VERSION APP VERSION DESCRIPTION
https://hub.helm.sh/charts/bitnami/wordpress        7.6.7         5.2.4       Web publishing platform for building blogs and ...
https://hub.helm.sh/charts/presslabs/wordpress-...  v0.6.3        v0.6.3      Presslabs WordPress Operator Helm Chart
https://hub.helm.sh/charts/presslabs/wordpress-...  v0.7.1        v0.7.1      A Helm chart for deploying a WordPress site on ...
```
以上搜索了`wordpress` Artifact Hub 上的所有图表。

没有过滤器，`helm search hub`向您显示所有可用的图表。

使用`helm search repo`，您可以在已添加的存储库中找到图表的名称：

```bash
$ helm repo add brigade https://brigadecore.github.io/charts
"brigade" has been added to your repositories
$ helm search repo brigade
NAME                          CHART VERSION APP VERSION DESCRIPTION
brigade/brigade               1.3.2         v1.2.1      Brigade provides event-driven scripting of Kube...
brigade/brigade-github-app    0.4.1         v0.2.1      The Brigade GitHub App, an advanced gateway for...
brigade/brigade-github-oauth  0.2.0         v0.20.0     The legacy OAuth GitHub Gateway for Brigade
brigade/brigade-k8s-gateway   0.1.0                     A Helm chart for Kubernetes
brigade/brigade-project       1.0.0         v1.0.0      Create a Brigade project
brigade/kashti                0.4.0         v0.4.0      A Helm chart for Kubernetes
```
Helm 搜索使用模糊字符串匹配算法，因此您可以键入部分单词或短语：

```bash
$ helm search repo kash
NAME            CHART VERSION APP VERSION DESCRIPTION
brigade/kashti  0.4.0         v0.4.0      A Helm chart for Kubernetes
```
### 4.3 helm install'：安装包
在最简单的情况下，它需要两个参数：您选择的版本名称和您要安装的图表的名称。

```bash
$ helm install happy-panda bitnami/wordpress
NAME: happy-panda
LAST DEPLOYED: Tue Jan 26 10:27:17 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
** Please be patient while the chart is being deployed **
Your WordPress site can be accessed through the following DNS name from within your cluster:
    happy-panda-wordpress.default.svc.cluster.local (port 80)
To access your WordPress site from outside the cluster follow the steps below:
1. Get the WordPress URL by running these commands:
  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace default -w happy-panda-wordpress'
   export SERVICE_IP=$(kubectl get svc --namespace default happy-panda-wordpress --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
   echo "WordPress URL: http://$SERVICE_IP/"
   echo "WordPress Admin URL: http://$SERVICE_IP/admin"
2. Open a browser and access WordPress using the obtained URL.
3. Login with the following credentials below to see your blog:
  echo Username: user
  echo Password: $(kubectl get secret --namespace default happy-panda-wordpress -o jsonpath="{.data.wordpress-password}" | base64 --decode)
```
现在`wordpress`图表已安装。请注意，安装图表会创建一个新的发布对象。上面的版本名为`happy-panda`. （如果您希望 Helm 为您生成名称，请省略发布名称并使用`--generate-name`.）

在安装过程中，helm客户端将打印有关创建了哪些资源、发布状态是什么以及您是否可以或应该采取其他配置步骤的有用信息。
Helm 按以下顺序安装资源：

```bash
命名空间
网络策略
资源配额
限制范围
PodSecurityPolicy
PodDisruptionBudget
服务帐户
秘密
秘密清单
配置映射
存储类
持久卷
PersistentVolumeClaim
自定义资源定义
集群角色
集群角色列表
集群角色绑定
ClusterRoleBindingList
角色
角色列表
角色绑定
角色绑定列表
服务
守护程序集
在下面
复制控制器
副本集
部署
Horizo​​ntalPodAutoscaler
有状态集
工作
定时任务
入口
API服务
```
Helm 不会等到所有资源都运行完才退出。许多图表需要大小超过 600M 的 Docker 镜像，并且可能需要很长时间才能安装到集群中。

要跟踪发布的状态，或重新读取配置信息，您可以使用`helm status`：

```bash
$ helm status happy-panda
NAME: happy-panda
LAST DEPLOYED: Tue Jan 26 10:27:17 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
** Please be patient while the chart is being deployed **
Your WordPress site can be accessed through the following DNS name from within your cluster:
    happy-panda-wordpress.default.svc.cluster.local (port 80)
To access your WordPress site from outside the cluster follow the steps below:
 - Get the WordPress URL by running these commands:
  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace default -w happy-panda-wordpress'
   export SERVICE_IP=$(kubectl get svc --namespace default happy-panda-wordpress --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
   echo "WordPress URL: http://$SERVICE_IP/"
   echo "WordPress Admin URL: http://$SERVICE_IP/admin"
 - Open a browser and access WordPress using the obtained URL.
 - Login with the following credentials below to see your blog:
  echo Username: user
  echo Password: $(kubectl get secret --namespace default happy-panda-wordpress -o jsonpath="{.data.wordpress-password}" | base64 --decode)
```
更多安装方式
 - 一个图表存储库（正如我们在上面看到的）
 - 本地图表存档 ( `helm install foo foo-0.1.1.tgz`)
 - 解压后的图表目录 ( `helm install foo path/to/foo`)
 - 完整网址 ( `helm install foo https://example.com/charts/foo-1.2.3.tgz`)

### 4.4 自定义chart
要查看图表上可配置的选项，请使用`helm show values`：

```bash
$ helm show values bitnami/wordpress
## Global Docker image parameters
## Please, note that this will override the image parameters, including dependencies, configured to use the global value
## Current available global Docker image parameters: imageRegistry and imagePullSecrets
##
# global:
#   imageRegistry: myRegistryName
#   imagePullSecrets:
#     - myRegistryKeySecretName
#   storageClass: myStorageClass
## Bitnami WordPress image version
## ref: https://hub.docker.com/r/bitnami/wordpress/tags/
##
image:
  registry: docker.io
  repository: bitnami/wordpress
  tag: 5.6.0-debian-10-r35
  [..]
```
然后，您可以覆盖 YAML 格式文件中的任何这些设置，然后在安装期间传递该文件。

```bash
$ echo '{mariadb.auth.database: user0db, mariadb.auth.username: user0}' > values.yaml
$ helm install -f values.yaml bitnami/wordpress --generate-name
```
以上将创建一个名为 的默认 MariaDB 用户`user0`，并授予该用户对新创建的`user0db`数据库的访问权限，但将接受该图表的所有其余默认值。

在安装过程中有两种方式传递配置数据：

 - `--values`（或-f）：指定具有覆盖的 YAML 文件。这可以指定多次，最右边的文件将优先
 - `--set`：在命令行上指定覆盖

如果两者都使用，则以更高的优先级`--set`合并值。`--values`用 指定的覆盖`--set`将持久保存在 `ConfigMap` 中。`--set`可以使用 . 查看给定版本的值`helm get values <release-name>`。可以通过使用指定`--set`的运行来清除已被清除的值。`helm upgrade--reset-values`

#### 4.4.1 格式和限制--set
该`--set`选项采用零个或多个名称/值对。最简单的用法是这样的：`--set name=value.` 等效的 YAML 是：

```bash
name: value
```
多个值由,字符分隔。于是`--set a=b,c=d`变成：

```bash
a: b
c: d
```
支持更复杂的表达式。例如，`--set outer.inner=value`翻译成这样：

```bash
outer:
  inner: value
```
列表可以通过将值括在{和中来表示}。例如，`--set name={a, b, c}`转换为：

```bash
name:
  - a
  - b
  - c
```
从 `Helm 2.5.0` 开始，可以使用数组索引语法访问列表项。例如，`--set servers[0].port=80`变为：

```bash
servers:
  - port: 80
```
可以通过这种方式设置多个值。该行`--set servers[0].port=80,servers[0].host=example`变为：

```bash
servers:
  - port: 80
    host: example
```
有时您需要在行中使用特殊字符`--set`。您可以使用反斜杠来转义字符；`--set name=value1\,value2`会变成：

```bash
name: "value1,value2"
```
`toYaml`同样，您也可以转义点序列，当图表使用该函数解析注释、标签和节点选择器时，这可能会派上用场。的语法`--set nodeSelector."kubernetes\.io/role"=master`变为：

```bash
nodeSelector:
  kubernetes.io/role: master
```
阅读有关[值文件](https://www.bookstack.cn/read/helm-3.8.0-en/9adfbde68c1a44d9.md)的更多信息




### 4.5 'helm upgrade' 和 'helm rollback'：升级版本，并在失败时恢复
当发布新版本的图表时，或者当您想要更改发布的配置时，可以使用该helm upgrade命令。

升级采用现有版本并根据您提供的信息对其进行升级。由于 Kubernetes 图表可能很大且很复杂，Helm 尝试执行侵入性最小的升级。它只会更新自上次发布以来已更改的内容。

```bash
$ helm upgrade -f panda.yaml happy-panda bitnami/wordpress
```
在上述情况下，happy-panda使用相同的图表升级版本，但使用新的 YAML 文件：

```bash
mariadb.auth.username: user1
```

我们可以`helm get values`用来查看新设置是否生效。

```bash
$ helm get values happy-panda
mariadb:
  auth:
    username: user1
```
该`helm get`命令是查看集群中发布的有用工具。正如我们在上面看到的，它表明我们的新值`panda.yaml`已部署到集群中。

现在，如果在发布期间某些事情没有按计划进行，很容易使用`helm rollback [RELEASE] [REVISION]`.

```bash
$ helm rollback happy-panda 1
```
以上将我们的`happy-panda` 回滚到它的第一个发布版本。发布版本是增量修订。每次安装、升级或回滚时，修订号都会增加 1。第一个修订号始终为 1。我们可以使用它`helm history [RELEASE]`来查看某个版本的修订号。

### 4.6 安装/升级/回滚的有用选项
在安装/升级/回滚期间，您可以指定其他几个有用的选项来自定义 Helm 的行为。请注意，这不是 cli 标志的完整列表。要查看所有标志的描述，只需运行`helm <command> --help`.

 - `--timeout`：等待 Kubernetes 命令完成的Go 持续时间值。这默认为`5m0s`.
 - `--wait`：等到所有 Pod 都处于就绪状态，PVC 被绑定，部署有最少（Desired减号maxUnavailable）的 Pod 处于就绪状态并且服务有一个 IP 地址（如果是 a 则为 `Ingress  LoadBalancer`），然后才将发布标记为成功。只要`--timeout`值，它将等待。如果达到超时，释放将被标记为`FAILED`。注意：在`Deploymentreplicas`设置为 1 并且`maxUnavailable`作为滚动更新策略的一部分未设置为 0的情况下，--wait将返回就绪状态，因为它满足了处于就绪状态的最小 Pod。
 - `--no-hooks`：这会跳过命令的运行钩子
 - `--recreate-pods`（仅适用于upgrade和rollback）：此标志将导致重新创建所有 Pod（属于部署的 Pod 除外）。（在 Helm 3 中已弃用）

###  4.7  'helm uninstall'：卸载版本

```bash
$ helm uninstall happy-panda
$ helm list
NAME            VERSION UPDATED                         STATUS          CHART
inky-cat        1       Wed Sep 28 12:59:46 2016        DEPLOYED        alpine-0.1.0
```
在以前的 `Helm` 版本中，当一个版本被删除时，它的删除记录将保留。在 `Helm 3` 中，删除也会删除发布记录。如果您希望保留删除版本记录，请使用`helm uninstall --keep-history`. Using `helm list --uninstalled`将仅显示使用该`--keep-history`标志卸载的版本。

该`helm list --all`标志将显示 Helm 保留的所有发布记录，包括失败或已删除项目的记录（如果`--keep-history`已指定）：

```bash
$  helm list --all
NAME            VERSION UPDATED                         STATUS          CHART
happy-panda     2       Wed Sep 28 12:47:54 2016        UNINSTALLED     wordpress-10.4.5.6.0
inky-cat        1       Wed Sep 28 12:59:46 2016        DEPLOYED        alpine-0.1.0
kindred-angelf  2       Tue Sep 27 16:16:10 2016        UNINSTALLED     alpine-0.1.0
```
请注意，由于现在默认删除版本，因此无法再回滚已卸载的资源。

有时，当 Helm 运行`helm uninstall.` chart开发人员可以为资源添加注释以防止其被卸载。

```bash
kind: Secret
metadata:
  annotations:
    "helm.sh/resource-policy": keep
[...]
```
注释`"helm.sh/resource-policy": keep`指示 Helm 在 helm 操作（例如`helm uninstall`、`helm upgrade`或`helm rollback`）导致其删除时跳过删除此资源。但是，此资源成为孤立资源。Helm 将不再以任何方式管理它。`helm install --replace`如果在已卸载但保留资源的版本上使用，这可能会导致问题。
### 4.8 创建自己的chart
[图表开发指南](https://www.bookstack.cn/read/helm-3.8.0-en/89342c804d7c3b76.md)解释了如何开发您自己的图表。`helm create`但是您可以使用以下命令快速入门：

```bash
$ helm create deis-workflow
Creating deis-workflow
```
当您编辑图表时，您可以通过运行来验证它的格式是否正确`helm lint`。

当需要打包图表以进行分发时，您可以运行以下`helm package`命令：

```bash
$ helm package deis-workflow
deis-workflow-0.1.0.tgz
```
现在可以通过以下方式轻松安装该图表helm install：

```bash
$ helm install deis-workflow ./deis-workflow-0.1.0.tgz
```
打包的图表可以加载到图表存储库中。有关更多详细信息，请[参阅Helm 图表存储库的文档](https://www.bookstack.cn/read/helm-3.8.0-en/11c72f6ac43a81ed.md)。

---

✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>


 - [helm官网](https://helm.sh/docs/)
 - [helm 3.8.0 gitbook](https://www.bookstack.cn/read/helm-3.8.0-en/eb6c7d10673f4231.md)
 - [helm charts 入门指南](https://ghostwritten.blog.csdn.net/article/details/123400829)


