

参考资料：

 - [阳明](https://www.qikqiak.com/k8s-book/docs/43.Helm%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8.html)
 - [官方文档（含有更多细节）](https://v2.helm.sh/docs/using_helm/#initialize-helm-and-install-tiller)
 - [kubernetes 快速学习手册](https://ghostwritten.blog.csdn.net/article/details/108562082)

------------

## 1. 前提条件

 1. 一个 Kubernetes 集群
 2. 决定将哪些安全配置应用于您的安装（如果有）
 3. 安装和配置 Helm 和 Tiller，集群端服务。

`Helm` 将通过读取您的 `Kubernetes` 配置文件（通常是`$HOME/.kube/config`）来找出安装 `Tiller` 的位置。这与`kubectl`使用的文件相同。

要找出 Tiller 将安装到哪个集群，您可以运行 `kubectl config current-context`或`kubectl cluster-info`。

```bash
$ kubectl config current-context
my-cluster
```

## 2. 功能

 - 创建新的 chart
 - chart 打包成 tgz 格式
 - 上传 chart 到 chart 仓库或从仓库中下载 chart
 - 在Kubernetes集群中安装或卸载 chart
 - 管理用Helm安装的 chart 的发布周期

##  3. 概念

Helm 有三个重要概念：

 - `chart`：包含了创建Kubernetes的一个应用实例的必要信息
 - `config`：包含了应用发布配置信息
 - `release`：是一个 chart 及其配置的一个运行实例

##  4. Helm组件

Helm 有以下两个组成部分：
![在这里插入图片描述](https://img-blog.csdnimg.cn/ddc6b141628245e2b2dd945e55634644.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s)
`Helm Client` 是用户命令行工具，其主要负责如下：

 - 本地 `chart` 开发
 - 仓库管理
 - List item
 - 与 `Tiller sever` 交互
 - 发送预安装的 chart
 - 查询 release 信息
 - 要求升级或卸载已存在的 release

`Tiller Server`是一个部署在Kubernetes集群内部的 server，其与 `Helm client`、`Kubernetes API server` 进行交互。Tiller server 主要负责如下：

 - 监听来自 `Helm client` 的请求
 - 通过 chart 及其配置构建一次发布
 - 安装 chart 到Kubernetes集群，并跟踪随后的发布
 - 通过与Kubernetes交互升级或卸载 chart
 - 简单的说，client 管理 charts，而 server 管理发布 release

## 5. 安装

 - [官方安装](https://v2.helm.sh/docs/using_helm/#installing-helm)

###  5.1 安装客户端
我们可以在[Helm Realese](https://github.com/helm/helm/releases)页面下载二进制文件，这里下载的`v2.10.0`版本，解压后将可执行文件helm拷贝到`/usr/local/bin`目录下即可，这样Helm客户端就在这台机器上安装完成了。

现在我们可以使用Helm命令查看版本了，会提示无法连接到服务端Tiller：

```bash
$ helm  version
version.BuildInfo{Version:"v2.10.0", GitCommit:"d506314abfb5d21419df8c7e7e68012379db2354", GitTreeState:"clean", GoVersion:"go1.16.5"}
```
要安装 `Helm` 的服务端程序，我们需要使用到kubectl工具，所以先确保kubectl工具能够正常的访问 kubernetes 集群的apiserver

### 5.2 安装服务端
Helm 的服务器部分 `Tiller` 通常在您的 Kubernetes 集群内部运行。但对于开发，它也可以在本地运行，并配置为与远程 Kubernetes 集群通信。

安装`tiller`到集群中的最简单方法就是运行 `helm init`. 这将验证helm的本地环境是否正确设置（并在必要时进行设置）。然后它将连接到kubectl默认情况下连接到的任何集群( `kubectl config view`)。连接后，它将安装tiller到 `kube-system`命名空间中。


```bash
$ helm init
```
由于 Helm 默认会去`gcr.io`拉取镜像，所以如果你当前执行的机器没有配置科学上网的话可以实现下面的命令代替：

```bash
$ helm init --upgrade --tiller-image cnych/tiller:v2.10.0
$HELM_HOME has been configured at /root/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
To prevent this, run `helm init` with the --tiller-tls-verify flag.
For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation
Happy Helming!
```
如果一直卡住或者报 google api 之类的错误，可以使用下面的命令进行初始化：

```bash
$ helm init --upgrade --tiller-image cnych/tiller:v2.10.0 --stable-repo-url https://cnych.github.io/kube-charts-mirror/
```
这个命令会把默认的 google 的仓库地址替换成我同步的一个镜像地址。

如果在安装过程中遇到了一些其他问题，比如初始化的时候出现了如下错误：

```bash
E0125 14:03:19.093131   56246 portforward.go:331] an error occurred forwarding 55943 -> 44134: error forwarding port 44134 to pod d01941068c9dfea1c9e46127578994d1cf8bc34c971ff109dc6faa4c05043a6e, uid : unable to do port forwarding: socat not found.
2018/01/25 14:03:19 (0xc420476210) (0xc4203ae1e0) Stream removed, broadcasting: 3
2018/01/25 14:03:19 (0xc4203ae1e0) (3) Writing data frame
2018/01/25 14:03:19 (0xc420476210) (0xc4200c3900) Create stream
2018/01/25 14:03:19 (0xc420476210) (0xc4200c3900) Stream added, broadcasting: 5
Error: cannot connect to Tiller
```
解决方案：在节点上安装`socat`可以解决

```bash
$ sudo yum install -y socat
```
Helm 服务端正常安装完成后，Tiller默认被部署在kubernetes集群的kube-system命名空间下：

```bash
$ kubectl get pod -n kube-system -l app=helm
NAME                             READY     STATUS    RESTARTS   AGE
tiller-deploy-86b844d8c6-44fpq   1/1       Running   0          7m
```
此时，我们查看 Helm 版本就都正常了：

```bash
$ helm version
Client: &version.Version{SemVer:"v2.10.0", GitCommit:"9ad53aac42165a5fadc6c87be0dea6b115f93090", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.10.0", GitCommit:"9ad53aac42165a5fadc6c87be0dea6b115f93090", GitTreeState:"clean"}
```
另外一个值得注意的问题是RBAC，我们的 kubernetes 集群是1.10.0版本的，默认开启了RBAC访问控制，所以我们需要为Tiller创建一个`ServiceAccount`，让他拥有执行的权限，详细内容可以查看 Helm 文档中的[Role-based Access Control](https://helm.sh/docs/intro/)。 创建`rbac.yaml`文件：

```bash
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```
然后使用kubectl创建：

```bash
$ kubectl create -f rbac-config.yaml
serviceaccount "tiller" created
clusterrolebinding.rbac.authorization.k8s.io "tiller" created
```
创建了`tiller`的 `ServceAccount` 后还没完，因为我们的 Tiller 之前已经就部署成功了，而且是没有指定 `ServiceAccount` 的，所以我们需要给 Tiller 打上一个 `ServiceAccount` 的补丁：

```bash
$ kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
```
上面这一步非常重要，不然后面在使用 Helm 的过程中可能出现`Error: no available release name found`的错误信息。
至此, Helm客户端和服务端都配置完成了，接下来我们看看如何使用吧。

##  6. helm init 
### 6.1  --history-max
`--history-max`建议在 `helm init` 上进行设置，因为如果未按最大限制清除，则 helm 历史记录中的配置映射和其他对象的数量可能会增加。如果没有设置最大历史记录，历史记录将无限期保存，留下大量记录供掌舵和分蘖维护。

```bash
$ helm init --history-max 200
```
###  6.2 --node-selectors
`--node-selectors`标志允许我们指定调度 Tiller pod 所需的节点标签

```bash
$ helm init --node-selectors "beta.kubernetes.io/os"="linux"
```
已安装的部署清单将包含我们的节点选择器标签。

```bash
...
spec:
  template:
    spec:
      nodeSelector:
        beta.kubernetes.io/os: linux
...
```
### 6.3  --override
`--override`允许您指定 Tiller 部署清单的属性。与`--set` 中其他地方使用的命令不同， `helm init --override`操作最终清单的指定属性（没有“值”文件）。因此，您可以为部署清单中的任何有效属性指定任何有效值。

 - **覆盖注释**

```bash
helm init --override metadata.annotations."deployment\.kubernetes\.io/revision"="1"
```
输出：

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
...
```

 - **覆盖亲和力**

```bash
helm init --override "spec.template.spec.affinity.nodeAffinity.preferredDuringSchedulingIgnoredDuringExecution[0].weight"="1" --override "spec.template.spec.affinity.nodeAffinity.preferredDuringSchedulingIgnoredDuringExecution[0].preference.matchExpressions[0].key"="e2e-az-name"
```
指定的属性组合到“`preferredDuringSchedulingIgnoredDuringExecution`”属性的第一个列表项中。

```bash
...
spec:
  strategy: {}
  template:
    ...
    spec:
      affinity:
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - preference:
              matchExpressions:
              - key: e2e-az-name
                operator: ""
            weight: 1
...
```

###  6.4 --output
该`--output`标志允许我们跳过 Tiller 部署清单的安装，只需将部署清单以 JSON 或 YAML 格式输出到标准输出。然后可以使用类似工具修改输出jq 并使用kubectl.

在下面的示例中，我们helm init使用--output json标志执行。

```bash
helm init --output json
```

跳过 Tiller 安装，清单以 JSON 格式输出到标准输出。

```bash
"apiVersion": "apps/v1",
"kind": "Deployment",
"metadata": {
    "creationTimestamp": null,
    "labels": {
        "app": "helm",
        "name": "tiller"
    },
    "name": "tiller-deploy",
    "namespace": "kube-system"
},
...
```

### 6.5 其他参数

 - 使用`--canary-image`标志安装金丝雀版本 ： `helm init --canary-image`
 - 安装特定的镜像（版本） `--tiller-image`
 - 安装到特定集群 `--kube-context`
 - 安装到特定的命名空间中 `--tiller-namespace`
 - 使用服务帐户安装 Tiller `--service-account`（对于启用 RBAC 的集群）
 - 在不安装服务帐户的情况下安装 Tiller `--automount-service-account false`

##  7. 部署示例
我们现在了尝试创建一个 Chart：

```bash
$ helm create hello-helm
Creating hello-helm
$ tree hello-helm
hello-helm
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── ingress.yaml
│   ├── NOTES.txt
│   └── service.yaml
└── values.yaml

2 directories, 7 files
```
我们通过查看`templates`目录下面的`deployment.yaml`文件可以看出默认创建的 Chart 是一个 `nginx` 服务，具体的每个文件是干什么用的，我们可以前往 [Helm 官方文档](https://v2.helm.sh/docs/developing_charts/#charts)进行查看，后面会和大家详细讲解的。比如这里我们来安装 `1.7.9` 这个版本的 `nginx`，则我们更改 `value.yaml` 文件下面的 `image tag` 即可，将默认的 stable 更改为 1.7.9，为了测试方便，我们把 Service 的类型也改成 NodePort

```bash
...
image:
  repository: nginx
  tag: 1.7.9
  pullPolicy: IfNotPresent

nameOverride: ""
fullnameOverride: ""

service:
  type: NodePort
  port: 80
...
```
现在我们来尝试安装下这个 Chart :

```bash
$ helm install ./hello-helm
NAME:   iced-ferret
LAST DEPLOYED: Thu Aug 30 23:39:45 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Service
NAME                    TYPE       CLUSTER-IP     EXTERNAL-IP  PORT(S)  AGE
iced-ferret-hello-helm  ClusterIP  10.100.118.77  <none>       80/TCP   0s

==> v1beta2/Deployment
NAME                    DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
iced-ferret-hello-helm  1        0        0           0          0s

==> v1/Pod(related)
NAME                                     READY  STATUS   RESTARTS  AGE
iced-ferret-hello-helm-58cb69d5bb-s9f2m  0/1    Pending  0         0s


NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app=hello-helm,release=iced-ferret" -o jsonpath="{.items[0].metadata.name}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl port-forward $POD_NAME 8080:80

$ kubectl get pods -l app=hello-helm
NAME                                      READY     STATUS    RESTARTS   AGE
iced-ferret-hello-helm-58cb69d5bb-s9f2m   1/1       Running   0          2m
$ kubectl get svc -l app=hello-helm
NAME                       TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
iced-ferret-hello-helm   NodePort   10.104.127.141   <none>        80:31236/TCP   3m
```
等到 Pod 创建完成后，我们可以根据创建的 `Service` 的 `NodePort` 来访问该服务了，然后在浏览器中打开`http://k8s.haimaxy.com:31236`就可以正常的访问我们刚刚部署的 nginx 应用了。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/430ad7ec5997ff45e3ac7dc694a1804b.png)
查看release：

```bash
$ helm list
NAME             REVISION    UPDATED                     STATUS      CHART               APP VERSION    NAMESPACE
winning-zebra    1           Thu Aug 30 23:50:29 2018    DEPLOYED    hello-helm-0.1.0    1.0            default

$ helm ls
NAME            	REVISION	UPDATED                 	STATUS  	CHART       	APP VERSION	NAMESPACE
wintering-rodent	1       	Thu Oct 18 15:06:58 2018	DEPLOYED	mysql-0.10.1	5.7.14     	default
```
打包chart：

```handlebars
$ helm package hello-helm
Successfully packaged chart and saved it to: /root/course/kubeadm/helm/hello-helm-0.1.0.tgz
```

然后我们就可以将打包的tgz文件分发到任意的服务器上，通过`helm fetch`就可以获取到该 Chart 了。
然后我们就可以将打包的tgz文件分发到任意的服务器上，通过helm fetch就可以获取到该 Chart 了。

删除release：

```bash
$ helm delete winning-zebra
release "winning-zebra" deleted
```

然后我们看到kubernetes集群上的该 nginx 服务也已经被删除了。

```bash
$ kubectl get pods -l app=hello-helm
No resources found.
```


