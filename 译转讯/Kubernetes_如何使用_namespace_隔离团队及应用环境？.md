#  Kubernetes 如何使用命名空间隔离团队及应用环境

![](https://i-blog.csdnimg.cn/blog_migrate/ab60f3417cdd7666c2ec198aa891dfa2.png)






## 1. 背景
随着团队和应用规模的扩大，我们部署的 K8s 资源会越来越多，管理和组织这些资源成为了首先要面对的问题。另外，当我们对同一个应用有多套环境诉求时，需要有一种隔离机制能让我们在同一个集群内部署多个相同名字的工作负载。而且，不同业务的工作负载会相互竞争资源，这进一步增加了集群管理的难度。

## 2. 架构图
为了方便你理解命名空间，我们一开始可以把 K8s 的命名空间比作是“子集群”。事实上，命名空间是集群的一种“软隔离”机制，它通过这种隔离机制，为我们管理和组织集群资源提供了便利。集群和命名空间的关系如图所示：
![](https://i-blog.csdnimg.cn/blog_migrate/5d96f892e156a7e4f93ff6ae294331c8.png)
不同的命名空间具备一定的隔离性，业务应用的工作负载以及其他的 K8s 对象可以部署在特定的命名空间中，在不同命名空间中，相同类型的资源可以重名。利用这种特性，我们可以把集群虚拟的命名空间和现实中的不同团队、不同应用在逻辑上联系起来。

## 3. 默认 namespace
系统级的命名空间是 K8s 集群默认创建的命名空间，主要用来隔离系统级的对象和业务对象，系统级的命名空间下面四种。

- `default`：默认命名空间，也就是在不指定命名空间时的默认命名空间。
- `kube-system`：K8s 系统级组件的命名空间，所有 K8s 的关键组件（例如 kube-proxy、coredns、metric-server 等）都在这个命名空间下。
- `kube-public`：开放的命名空间，所有用户（包括未经认证）的用户都可以读取，这个命名空间是一个约定，但不是必须。
- `kube-node-lease`：和集群扩展相关的命名空间。

## 4. 查看
查看 K8s 的命名空间

```bash
$ kubectl get ns
NAME                 STATUS   AGE
default              Active   16h
example              Active   16h
ingress-nginx        Active   16h
kube-node-lease      Active   16h
kube-public          Active   16h
kube-system          Active   16h
local-path-storage   Active   16h
```
从返回结果我们可以看出，示例应用创建了一个新的命名空间 example，同时，我们之前部署的 Ingress-Nginx 也创建了自己专属的命名空间。此外，要获取 Namespace 的详情，例如获取命名空间级别的资源配额等详细信息，你可以使用 `kubectl describe namespace` 命令：

```bash
$ kubectl describe namespace example
Name:         example
Labels:       kubernetes.io/metadata.name=example
Annotations:  <none>
Status:       Active

No resource quota.

No LimitRange resource.
```
创建一个命名空间

```bash
$ kubectl create namespace my-namespace
namespace/my-namespace created
```
注意，命名空间的名字需要符合 DNS-1123 规范，其中最重要的点是字母只支持小写，另外除了特殊字符“-”以外，其他任何特殊字符都无法使用。当你尝试创建一个不符合规范的命名空间时，K8s 将返回错误：

```bash
$ kubectl create namespace my.namespace
The Namespace "my.namespace" is invalid: metadata.name: Invalid value: "my.namespace": a lowercase RFC 1123 label must consist of lower case alphanumeric characters or '-', and must start and end with an alphanumeric character (e.g. 'my-name',  or '123-abc', regex used for validation is '[a-z0-9]([-a-z0-9]*[a-z0-9])?')
```
`K8s Manifest` 来创建命名空间

```bash
apiVersion: v1
kind: Namespace
metadata:
  name: example
```
使用 `kubectl apply -f namespace.yaml` 命令应用这段 Manifest，它的效果和 `kubectl create namespace` 是一样的。

删除命名空间

```bash
$ kubectl delete namespace my-namespace
namespace "my-namespace" deleted
```
**删除命名空间将会删除该命名空间下的所有资源，所以在实际项目中请谨慎操作**。另外，由于删除动作是异步的，因此删除中的命名空间状态会由正常的 `Active` 转变为 `Terminating`，也就是“终止中的状态”。


## 5. 如何使用
在资源的 Manifest 中设置 `Namespace` 字段

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: example   # 设置 namesapce
  labels:
    app: frontend
spec:
  ......
```
从上面的内容可以看出，metadata.namespace 字段的值为 example，这意味着该资源将部署在指定 example 命名空间下。相反，如果不指定该字段，资源就会被部署到 default 命名空间下。

此外，还有一种指定资源命名空间的方法,用 `kubectl apply -n` 参数指定命名空间

```bash
kubectl apply -f deploy -n example
```
查看 Deployment 

```bash

$ kubectl get deployment -n example
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
backend    2/2     2            2           17h
frontend   2/2     2            2           17h
postgres   1/1     1            1           17h

$ kubectl get deployment -n default
```
由于 `default` 是默认的命名空间，所以如果我们在部署、删除和查看 K8s 对象时没有指定命名空间，就会默认都指向 `default`。

## 6. 什么时候使用命名空间？
对于一些小型团队或者小型应用，直接使用 default 也是一种选择。不过，当你有下面的需求时，我建议你考虑使用命名空间。

- **环境管理**：你可以使用命名空间来隔离开发环境、预发布环境和生产环境。例如用 dev、staging、prod 命名空间来区分这三个环境。命名空间的隔离特性可以让开发环境的的操作不会影响到其他环境的工作。实际上，对于这种情况，我更建议使用不同的集群进行硬隔离，命名空间的隔离方式只是一种选择。
- **隔离**：当你有多个团队，或者多个产品线运行在同一个集群时，你可以使用命名空间来隔离这些团队或者产品线，让他们相互之间不受影响。你甚至可以为每一个开发分配一个命名空间，让他们独立进行开发工作。
- **资源控制**：你可以在命名空间级别配置 CPU、内存等资源配额。通过这个方法，你可以为某个团队或者某条业务线设置总的资源配额，而不需要关注他们具体在每一个工作负载上怎么分配这些资源。此外，命名空间的资源配额还能够合理确保每一个项目所需要的资源配额，避免出现资源恶性竞争导致业务不稳定的情况。
- **权限控制**：K8s 的 RBAC 可以帮助我们对某个用户仅授权一个或者多个命名空间，确保只有特定的用户能访问特定命名空间下的资源。
- **提高集群性能**：命名空间有利于提高集群性能。在进行资源搜索时，命名空间有利于 K8s API 缩小查找范围，对减小搜索延迟和提升性能具有一定的帮助。

## 7. 如何跨命名空间通信？
在使用命名空间隔离资源之后，就会涉及到如何通信的问题。我们之前说命名空间是一种“软隔离”机制，既然是软隔离，那么不同命名空间完全是可以相互通信的。当 K8s 创建 Service 时，会创建相应的 DNS 解析记录，格式为：`..svc.cluster.local`。当业务容器需要和当前命名空间下其他服务通信时，则可以省略`.svc.cluster.local`，也就是缩写为就可以了。

理，要跨不同的命名空间通信，我们可以指定完整的 `Service URL` 来实现。例如，要访问 dev 命名空间下的 `backend-service`，完整的 URL 为：

```bash
backend-service.dev.svc.cluster.lcoal
```
当然，你也可以通过配置 K8s 的网络策略来限制这种访问方式。

## 8. 如何规划命名空间？

如何合理地规划命名空间是我们刚开始使用 K8s 就要考虑的问题。K8s 命名空间只提供一种组织方式，在实际的使用过程，你可以按照自己的团队情况、组织架构、业务线等维度来进行划分。为了能让你在实际的工作中直接使用，我为你总结了以下几种 K8s 命名空间规划方案。

### 8.1 单一业务应用和团队
当团队规模在几人到十几人，业务应用只有 1-2 个时，你可以参考下图这种命名空间的规划方案
![](https://i-blog.csdnimg.cn/blog_migrate/c13de3aaa68233f6fd9ec884cc7dad64.png)
在这个方案中，我们将 K8s 集群划分为 3 个命名空间，分别是 `dev`、`testing` 和 `prod`。
- dev：开发环境，用于开发人员日常开发。
- testing：测试环境，用于质量人员进行测试工作。
- prod：生产环境，用于对外提供生产服务。

注意，为了让 K8s Manifest 能部署到多个命名空间，一般我们不直接在 Manifest 的 Namespace 字段里指定命名空间，而是在实际部署时指定命名空间：

```bash
$ kuebctl apply -f deploy -n dev
deployment.apps/backend created
......
Error from server (BadRequest): error when creating "deploy/ingress.yaml": admission webhook "validate.nginx.ingress.kubernetes.io" denied the request: host "_" and path "/?(.*)" is already defined in ingress example/frontend-ingress
```
上面这段命令，实际上是将示例应用部署到了一个新的 dev 命名空间里。不过在你执行时会返回一个错误，这是因为在不同的命名空间下设置了相同的 Ingress 策略，我们可以暂时忽略它。在实际的业务场景中，不同环境的访问域名是不同的，自然也就不会出现这个问题。

这样，当需要将应用部署到 QA 环境时，只需要将 `-n` 参数替换为 `testing` 即可：

```bash
$ kubectl apply -f deploy -n testing
```
最后，再将应用部署到 Prod 生产环境，这样就完成了应用从开发、测试到上线的完整生命周期。

### 8.2 多业务应用和团队
在大型的组织架构下，往往会有多个业务应用和多个团队组成，他们开发的可能是同一个大型业务，也可能是独立的应用。在这种情况下，我建议你将团队和命名空间关联起来，并且在区分环境时使用不同的集群，具体规划方案可以参考下图：
![](https://i-blog.csdnimg.cn/blog_migrate/ff66ea7378870ad05349a1df5d743ad1.png)
这个方案的特点是，dev、testing、prod 环境分别用了 3 个独立的集群，然后会在每一个集群中为每个独立的团队分配一个命名空间，让团队和环境之间相互隔离。当然，如果不同的团队开发的是同一个业务，相当于只是给业务做了微服务的拆分，那么在 testing 和 prod 环境就不需要再划分独立命名空间，只需要有一个命名空间将所有微服务汇总部署起来即可。



