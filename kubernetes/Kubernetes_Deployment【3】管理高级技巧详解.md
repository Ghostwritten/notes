#  Deployment 技巧
tags: Deployment



----

 - [Kubernetes Deployment【1】管理与编排入门](https://ghostwritten.blog.csdn.net/article/details/121510015)
 - [Kubernetes Deployment【2】原理深入](https://ghostwritten.blog.csdn.net/article/details/121525931)
 - [Kubernetes Deployment【3】管理高级技巧](https://ghostwritten.blog.csdn.net/article/details/121562269)
 - [Kubernetes Deployment【4】参数大全](https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs/resources/deployment)
 - [Kubernetes Deployment【5】部署策略](https://ghostwritten.blog.csdn.net/article/details/121587974)
 - [Kubernetes Deployment【6】client-go 开发](https://ghostwritten.blog.csdn.net/article/details/110358813)
 - [云原生圣经](https://ghostwritten.blog.csdn.net/article/details/108562082)

----

##  1. patch 管理 deployment
###  1.2 patch 以 yaml 格式合并更新 Deployment

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: patch-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: patch-demo-ctr
        image: nginx
      tolerations:
      - effect: NoSchedule
        key: dedicated
        value: test-team
```
创建 `Deployment`：

```bash
$ kubectl create -f deployment-patch.yaml

$ kubectl get pods
NAME                        READY     STATUS    RESTARTS   AGE
patch-demo-28633765-670qr   1/1       Running   0          23s
patch-demo-28633765-j5qs3   1/1       Running   0          23s
```
把运行的 Pod 的名字记下来。稍后，你将看到这些 Pod 被终止并被新的 Pod 替换。

此时，每个 Pod 都有一个运行 nginx 镜像的容器。现在假设你希望每个 Pod 有两个容器：一个运行 `nginx`，另一个运行 `redis`。

创建一个名为 `patch-file-containers.yaml` 的文件。内容如下:

```bash
spec:
  template:
    spec:
      containers:
      - name: patch-demo-ctr-2
        image: redis
```
修补你的 `Deployment`：

```bash
kubectl patch deployment patch-demo --patch "$(cat patch-file-containers.yaml)"
```
查看修补后的 `Deployment`：

```bash
kubectl get deployment patch-demo --output yaml
```
输出显示 `Deployment` 中的 `PodSpec` 有两个容器:

```bash
containers:
- image: redis
  imagePullPolicy: Always
  name: patch-demo-ctr-2
  ...
- image: nginx
  imagePullPolicy: Always
  name: patch-demo-ctr
  ...
```
查看与 `patch Deployment` 相关的 `Pod`:

```bash
$ kubectl get pods
NAME                          READY     STATUS    RESTARTS   AGE
patch-demo-1081991389-2wrn5   2/2       Running   0          1m
patch-demo-1081991389-jmg7b   2/2       Running   0          1m
```

输出显示正在运行的 Pod 与以前运行的 Pod 有不同的名称。`Deployment` 终止了旧的 Pod，并创建了两个 符合更新的部署规范的新 Pod。2/2 表示每个 Pod 有两个容器:

```bash
$ kubectl get pods
NAME                          READY     STATUS    RESTARTS   AGE
patch-demo-1081991389-2wrn5   2/2       Running   0          1m
patch-demo-1081991389-jmg7b   2/2       Running   0          1m
```
仔细查看其中一个 `patch-demo Pod`:

```bash
$ kubectl get pod <your-pod-name> --output yaml
containers:
- image: redis
  ...
- image: nginx
  ...
```
###  1.2 合并类的 patch 的说明
在前面的练习中所做的 `patch` 称为策略性合并 `patch`（Strategic Merge Patch)。 请注意，`patch` 没有替换containers 列表。相反，它向列表中添加了一个新 `Container`。换句话说， `patch` 中的列表与现有列表合并。当你在列表中使用策略性合并 `patch` 时，并不总是这样。 在某些情况下，列表是替换的，而不是合并的。

对于策略性合并 `patch`，列表可以根据其 patch 策略进行替换或合并。 `patch` 策略由 `Kubernetes` 源代码中字段标记中的 `patchStrategy` 键的值指定。 例如，`PodSpec` 结构体的 `Containers` 字段的 `patchStrategy` 为 `merge`：

```bash
type PodSpec struct {
  ...
  Containers []Container `json:"containers" patchStrategy:"merge" patchMergeKey:"name" ...`
```
你还可以在 [OpenApi spec](https://raw.githubusercontent.com/kubernetes/kubernetes/master/api/openapi-spec/swagger.json) 规范中看到 `patch` 策略：

```bash
"io.k8s.api.core.v1.PodSpec": {
    ...
     "containers": {
      "description": "List of containers belonging to the pod. ...
      },
      "x-kubernetes-patch-merge-key": "name",
      "x-kubernetes-patch-strategy": "merge"
     },
```
你可以在 [Kubernetes API 文档](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.22/#podspec-v1-core) 中看到 patch 策略。

创建一个名为 `patch-file-tolerations.yaml` 的文件。内容如下:

```bash
spec:
  template:
    spec:
      tolerations:
      - effect: NoSchedule
        key: disktype
        value: ssd
```
对 `Deployment` 执行 `patch` 操作：

```bash
$ kubectl patch deployment patch-demo --patch "$(cat patch-file-containers.yaml)"

$ kubectl get deployment patch-demo --output yaml
containers:
- image: redis
  imagePullPolicy: Always
  name: patch-demo-ctr-2
  ...
- image: nginx
  imagePullPolicy: Always
  name: patch-demo-ctr
  ...

tolerations:
      - effect: NoSchedule
        key: disktype
        value: ssd
```
请注意，`PodSpec` 中的 `tolerations` 列表被替换，而不是合并。这是因为 `PodSpec` 的 `tolerations` 的字段标签中没有 `patchStrategy` 键。所以策略合并 patch 操作使用默认的 `patch` 策略，也就是 `replace`。

```bash
type PodSpec struct {
  ...
  Tolerations []Toleration `json:"tolerations,omitempty" protobuf:"bytes,22,opt,name=tolerations"`
```

###  1.3 patch 以 yaml 格式替换更新 Deployment 
策略性合并 patch 不同于 [JSON 合并 patch](https://datatracker.ietf.org/doc/html/rfc7386)。 使用 JSON 合并 patch，如果你想更新列表，你必须指定整个新列表。新的列表完全取代现有的列表。

`kubectl patch` 命令有一个 `type` 参数，你可以将其设置为以下值之一:
| Parameter value | Merge type                 |
|-----------------|----------------------------|
| json            | [JSON Patch, RFC 6902](https://tools.ietf.org/html/rfc6902)       |
| merge           | [JSON Merge Patch, RFC 7386](https://tools.ietf.org/html/rfc7386) |
| strategic       | Strategic merge patch      |

有关 JSON patch 和 JSON 合并 patch 的比较，查看 [JSON patch 和 JSON 合并 patch](http://erosb.github.io/post/json-patch-vs-merge-patch/)。

`type` 参数的默认值是 `strategic`。在前面的练习中，我们做了一个策略性的合并 `patch`。

下一步，在相同的部署上执行 JSON 合并 `patch`。创建一个名为 `patch-file-2` 的文件。内容如下:

```bash
spec:
  template:
    spec:
      containers:
      - name: patch-demo-ctr-3
        image: gcr.io/google-samples/node-hello:1.0
```
在 patch 命令中，将 `type` 设置为 `merge`：

```bash
kubectl patch deployment patch-demo --type merge --patch "$(cat patch-file-2.yaml)"
```
查看 patch 部署：

```bash
kubectl get deployment patch-demo --output yaml
```
`patch` 中指定的容器列表只有一个容器。 输出显示您的一个容器列表替换了现有的容器列表

```bash
spec:
  containers:
  - image: gcr.io/google-samples/node-hello:1.0
    ...
    name: patch-demo-ctr-3
```
列表中运行的 Pod：

```bash
$ kubectl get pods
NAME                          READY     STATUS    RESTARTS   AGE
patch-demo-1307768864-69308   1/1       Running   0          1m
patch-demo-1307768864-c86dc   1/1       Running   0          1m
```

### 1.4 patch 以 json 格式合并更新 Deployment 
`kubectl patch` 命令使用 `YAML` 或 `JSON`。它可以将 patch 作为文件，也可以直接在命令行中使用。

创建一个文件名称是 `patch-file.json` 内容如下：

```bash
{
   "spec": {
      "template": {
         "spec": {
            "containers": [
               {
                  "name": "patch-demo-ctr-2",
                  "image": "redis"
               }
            ]
         }
      }
   }
}
```
以下命令是相同的：

```bash
kubectl patch deployment patch-demo --patch "$(cat patch-file.yaml)"
```

###  1.5 patch 其他方式更新 Deployment 
```bash
kubectl patch deployment patch-demo --patch 'spec:\n template:\n  spec:\n   containers:\n   - name: patch-demo-ctr-2\n     image: redis'

kubectl patch deployment patch-demo --patch "$(cat patch-file.json)"
kubectl patch deployment patch-demo --patch '{"spec": {"template": {"spec": {"containers": [{"name": "patch-demo-ctr-2","image": "redis"}]}}}}'
```

##  2. kustomize 管理 deployment

 - [kustomize (一) 管理yaml部署入门hello world](https://ghostwritten.blog.csdn.net/article/details/107925618)

