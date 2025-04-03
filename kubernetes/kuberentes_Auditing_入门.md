#  kuberentes Auditing 入门
tags: Auditing,对象




[
![《银翼杀手2049》](https://i-blog.csdnimg.cn/blog_migrate/158382f6a6af0d97ed6a449a98fb1bcd.png)](https://bladerunner.fandom.com/wiki/Blade_Runner_2049)

{% youtube %}
https://www.youtube.com/watch?v=HXtLTxo30SY
{% endyoutube %}


##  1. 前言
[Kubernetes](https://kubernetes.io/)是一种容器编排工具，可降低部署和管理容器化应用程序的复杂性。它正迅速成为部署和管理大型应用程序和[微服务](https://www.containiq.com/post/microservices-architecture)的[行业标准](https://www.containiq.com/post/kubernetes-statistics)，并被各种组织广泛用于管理他们在云端和本地的应用程序。

作为[Kubernetes 管理员](https://www.containiq.com/post/certified-kubernetes-administrator)，您必须记录集群中发生的事件。这些记录将作为调试问题和提高[集群安全性](https://www.containiq.com/post/kubernetes-security-best-practices)的真实来源。Kubernetes  Auditing记录在您的集群中执行的操作（或某人试图执行的操作）。

在本文中，您将了解 Kubernetes  Auditing 是什么，它们为何重要，以及如何在 Kubernetes 集群中启用 Auditing日志。

日志自然是 Auditing日志的核心；在整个 Kubernetes 运行时，系统设置为跟踪性能指标、元数据和任何被认为重要的管理活动。从捕获点开始，这些日志被“流式传输”到它们的存储目的地，然后可用于回顾性分析。每个都带有时间戳以添加上下文。

在 Auditing日志记录过程中，有权访问其历史日志记录的人开始寻找任何突出的信息或任何外在表明可疑活动的信息。这些可能包括意外登录或系统崩溃。

 Auditing日志是一组记录，其中包含对[Kubernetes API](https://www.containiq.com/post/kubernetes-api)发出的所有请求的按时间顺序排列的列表。Kubernetes 存储每个用户以及[控制平面](https://www.containiq.com/post/kubernetes-control-plane)生成的操作。这些日志采用 JSON 格式，包含 HTTP 方法、发起请求的用户信息、发起的请求、处理请求的[Kubernetes 组件](https://www.containiq.com/post/kubernetes-components)等信息。包括：

 - The control plane (built-in controllers, the scheduler)
 - Node daemons (the kubelet, kube-proxy, and others)
 - Cluster services (e.g., the cluster autoscaler, kube-state-metrics,CoreDNS, etc.)
 - Users making kubectl requests
 - Applications, controllers, and operators that send requests through a kube client
 - Even the API server itself

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d522a949a15cc79f641671135997e9c6.png)


 Auditing允许集群管理员回答以下问题：

 - Kubernetes 发生了什么？
 - 什么时候发生了什么事？
 - 谁触发了事件？
 - 事件发生在什么级别？
 - 它发生在哪里，它是在哪里发起的？
 - 它去哪儿了？



##  2.  Auditing 阶段
当某个人或组件向 Kubernetes API 服务器发出请求时，该请求将经历一个或多个阶段：
| 阶段               | 说明                 |
|------------------|--------------------|
| RequestReceived  |  Auditing处理程序已收到请求。       |
| ResponseStarted  | 已发送响应标头，但尚未发送响应正文。 |
| ResponseComplete | 响应正文已完成，不再发送任何字节。  |
| Panic            | 内部服务器出错，请求未完成。     |

请求的每个阶段都会生成一个事件，该事件根据政策进行处理。政策指定是否应将事件记录为日志条目，如果需要记录，应将哪些数据包含在日志条目中。

##  3.  Auditing级别
 Auditing策略定义了关于应该记录哪些事件以及它们应该包含哪些数据的规则。 Auditing策略对象结构在 audit.k8s.ioAPI 组中定义。处理事件时，会按顺序将其与规则列表进行比较。第一个匹配规则设置事件的  Auditing级别。定义的 Auditing级别是：

 - `None`- 不记录符合此规则的事件。
 - `Metadata`- 记录请求元数据（请求用户、时间戳、资源、动词等），但不记录请求或响应正文。
 - `Request`- 记录事件元数据和请求正文，但不记录响应正文。这不适用于非资源请求。
 - `RequestResponse`- 记录事件元数据、请求和响应主体。这不适用于非资源请求。

您可以将带有策略的文件传递给kube-apiserver 使用`--audit-policy-file`标志。如果省略该标志，则不记录任何事件。请注意，必须在 Auditing策略文件中提供该`rules`字段。没有 (0) 规则的策略被视为非法。

下面是一个示例 Auditing策略文件`audit/audit-policy.yaml`：

```bash
apiVersion: audit.k8s.io/v1 # This is required.
kind: Policy
# Don't generate audit events for all requests in RequestReceived stage.
omitStages:
  - "RequestReceived"
rules:
  # Log pod changes at RequestResponse level
  - level: RequestResponse
    resources:
    - group: ""
      # Resource "pods" doesn't match requests to any subresource of pods,
      # which is consistent with the RBAC policy.
      resources: ["pods"]
  # Log "pods/log", "pods/status" at Metadata level
  - level: Metadata
    resources:
    - group: ""
      resources: ["pods/log", "pods/status"]

  # Don't log requests to a configmap called "controller-leader"
  - level: None
    resources:
    - group: ""
      resources: ["configmaps"]
      resourceNames: ["controller-leader"]

  # Don't log watch requests by the "system:kube-proxy" on endpoints or services
  - level: None
    users: ["system:kube-proxy"]
    verbs: ["watch"]
    resources:
    - group: "" # core API group
      resources: ["endpoints", "services"]

  # Don't log authenticated requests to certain non-resource URL paths.
  - level: None
    userGroups: ["system:authenticated"]
    nonResourceURLs:
    - "/api*" # Wildcard matching.
    - "/version"

  # Log the request body of configmap changes in kube-system.
  - level: Request
    resources:
    - group: "" # core API group
      resources: ["configmaps"]
    # This rule only applies to resources in the "kube-system" namespace.
    # The empty string "" can be used to select non-namespaced resources.
    namespaces: ["kube-system"]

  # Log configmap and secret changes in all other namespaces at the Metadata level.
  - level: Metadata
    resources:
    - group: "" # core API group
      resources: ["secrets", "configmaps"]

  # Log all other resources in core and extensions at the Request level.
  - level: Request
    resources:
    - group: "" # core API group
    - group: "extensions" # Version of group should NOT be included.

  # A catch-all rule to log all other requests at the Metadata level.
  - level: Metadata
    # Long-running requests like watches that fall under this rule will not
    # generate an audit event in RequestReceived.
    omitStages:
      - "RequestReceived"
```
您可以使用最小的 Auditing策略文件来记录`Metadata`级别的所有请求：

```bash
# Log all requests at the Metadata level.
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: Metadata
```
如果您正在制作自己的 Auditing配置文件，则可以使用 `Google Container-Optimized OS` 的 Auditing配置文件作为起点。您可以检查 生成 Auditing策略文件的[configure-helper.sh](https://github.com/kubernetes/kubernetes/blob/master/cluster/gce/gci/configure-helper.sh)脚本。您可以通过直接查看脚本来查看大部分 Auditing策略文件。

您还可以参考[Policy配置参考](https://kubernetes.io/docs/reference/config-api/apiserver-audit.v1/#audit-k8s-io-v1-Policy) 以获取有关定义的字段的详细信息。


##  4.  Auditing 技巧

 - 通常，`create`、`update` 和 `delete` 等写入请求会在 `RequestResponse` 级别记录。
 - 通常，`get`、`list` 和 `watch` 事件会在 `Metadata` 级别记录。
 - 有些事件被视为特殊情况。如需查看被视为特殊情况的请求的确切列表，请参阅政策脚本。在撰写本文时，以下这些是特殊情况：
   - 由 `kubelet`、`system:node-problem-detector` 或 `system:serviceaccount:kube-system:node-problem-detector` 发出的针对 `nodes/status` 资源或 `pods/status` 资源的 `update` 和 `patch` 请求会在 `Request` 级别记录。
   - 由 `system:nodes` 组中的任何身份发出的针对 `nodes/status` 资源或 `pods/status` 资源的 `update` 和 `patch` 请求会在 `Request` 级别记录。
   - `system:serviceaccount:kube-system:namespace-controller` 发出的 `deletecollection` 请求会在 `Request` 级别记录。
   - 针对 `secrets` 资源、`configmaps` 资源或 `tokenreviews` 资源的请求会在 `Metadata` 级别记录。
 - 某些请求是不会被记录的。如需查看系统不会记录的请求的确切列表，请参阅政策脚本中的 `level: None` 规则。截至撰写本文时为止，以下请求将不会进行记录：
   - `system:kube-proxy` 发出的监视 `endpoints` 资源、`services` 资源或 `services/status` 资源的请求；
   - `system:unsecured` 发出的针对 `kube-system` 命名空间中 `configmaps` 资源的 `get` 请求。
   - `kubelet` 发出的针对 `nodes` 资源或 `nodes/status` 资源的 `get` 请求。
   - `system:nodes` 组中的任何身份发出的针对 `nodes` 资源或 `nodes/status` 资源的 `get` 请求。
   - `system:kube-controller-manager`、`system:kube-scheduler` 或 `system:serviceaccount:endpoint-controller` 发出的针对 `kube-system` 命名空间中 `endpoints` 资源的 `get` 和 `update` 请求。
   - `system:apiserver` 发出的针对 `namespaces` 资源、`namespaces/status` 资源或 `namespaces/finalize` 资源的 `get` 请求。
   - `system:kube-controller-manager` 发出的针对 `metrics.k8s.io` 组中任何资源的 `get` 和 `list` 请求。
   - 对与 `/healthz*`、`/version` 或 `/swagger*` 匹配的网址发出的请求。- 

##  5.  Auditing 后端 
 Auditing后端将 Auditing事件持久化到外部存储。开箱即用的 kube-apiserver 提供了两个后端：

 - Log 后端，将事件写入文件系统
 - Webhook 后端，将事件发送到外部 HTTP API

在所有情况下， Auditing事件都遵循 [audit.k8s.io API 组](https://kubernetes.io/docs/reference/config-api/apiserver-audit.v1/#audit-k8s-io-v1-Event)中 Kubernetes API 定义的结构。

###  5.1 Log 后端
日志后端将 Auditing事件写入JSONlines格式的文件。您可以使用以下kube-apiserver标志配置日志 Auditing后端：

 - `--audit-log-path`：指定日志后端用于写入 Auditing事件的日志文件路径。不指定此标志会禁用日志后端。-表示标准输出
 - `--audit-log-maxage`：定义保留旧 Auditing日志文件的最大天数
 - `--audit-log-maxbackup`；定义要保留的 Auditing日志文件的最大数量
 - `--audit-log-maxsize`：定义 Auditing日志文件在轮换之前的最大大小（以 MB 为单位）
如果您的集群的控制平面将 kube-apiserver 作为 Pod 运行，请记住将其挂载hostPath 到策略文件和日志文件的位置，以便保留 Auditing记录。例如：

```bash
   --audit-policy-file=/etc/kubernetes/audit-policy.yaml \
    --audit-log-path=/var/log/kubernetes/audit/audit.log
```
然后挂载卷：

```bash
...
volumeMounts:
  - mountPath: /etc/kubernetes/audit-policy.yaml
    name: audit
    readOnly: true
  - mountPath: /var/log/kubernetes/audit/
    name: audit-log
    readOnly: false
```
最后配置`hostPath`：

```bash
...
volumes:
 - name: audit
  hostPath:
    path: /etc/kubernetes/audit-policy.yaml
    type: File

 - name: audit-log
  hostPath:
    path: /var/log/kubernetes/audit/
    type: DirectoryOrCreate
```
###  5.2 Webhook 后端
webhook  Auditing后端将 Auditing事件发送到远程 Web API，该 API 被假定为 Kubernetes API 的一种形式，包括身份验证方式。您可以使用以下 kube-apiserver 标志配置 webhook  Auditing后端：

 - `--audit-webhook-config-file`：使用 webhook 配置指定文件的路径。webhook 配置实际上是一个专门的 [kubeconfig](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)。

 - `--audit-webhook-initial-backoff`：指定在第一个失败请求之后重试之前等待的时间量。使用指数退避重试后续请求。 webhook 配置文件使用 kubeconfig 格式来指定服务的远程地址和用于连接它的凭据。

##  6. 事件批处理
日志和 webhook 后端都支持批处理。以 webhook 为例，这里是可用标志的列表。要为日志后端获取相同的标志，请在标志名称中webhook替换为。log默认情况下，批处理在 中启用webhook和禁用log。同样，默认情况下，在 中启用webhook和禁用节流log。

`--audit-webhook-mode`定义缓冲策略。以下之一：

 - `batch`- 缓冲事件并批量异步处理它们。这是默认设置。
 - `blocking`- 在处理每个单独的事件时阻止 API 服务器响应。
 - `blocking-strict`- 与阻塞相同，但当在 `RequestReceived` 阶段 Auditing日志记录失败时，对 kube-apiserver 的整个请求都会失败。

以下标志仅在`batch`模式中使用：

 - `--audit-webhook-batch-buffer-size`：定义批处理前要缓冲的事件数。如果传入事件的速率溢出缓冲区，则丢弃事件。
 - `--audit-webhook-batch-max-size`：定义一批中的最大事件数。
 - `--audit-webhook-batch-max-wait`：定义在队列中无条件批处理事件之前等待的最长时间。
 - `--audit-webhook-batch-throttle-qps`：定义每秒生成的最大平均批次数。
 - `--audit-webhook-batch-throttle-burst`：如果之前未充分利用允许的 QPS，则定义同时生成的最大批次数。

## 7. 参数调优
应设置参数以适应 API 服务器上的负载。

例如，如果 kube-apiserver 每秒接收 100 个请求，并且每个请求仅在`ResponseStarted`和`ResponseComplete`阶段进行 Auditing，则您应该考虑每秒生成 ≅200 个 Auditing事件。假设批处理中最多有 100 个事件，您应该将限制级别设置为每秒至少 2 个查询。假设后端最多可能需要 5 秒来写入事件，您应该将缓冲区大小设置为最多容纳 5 秒的事件；即：10 个批次，或 1000 个事件。

然而，在大多数情况下，默认参数就足够了，您不必担心手动设置它们。您可以查看 kube-apiserver 和日志中公开的以下 Prometheus 指标，以监控 Auditing子系统的状态。

 - `apiserver_audit_event_totalmetric`： 包含导出的 Auditing事件的总数。
 - `apiserver_audit_error_total`：指标包含由于导出过程中的错误而丢弃的事件总数。

### 7.1 日志条目截断
日志和 webhook 后端都支持限制记录的事件的大小。例如，以下是日志后端可用的标志列表：

 - `audit-log-truncate-enabled`：是否启用事件和批处理截断。
 - `audit-log-truncate-max-batch-size`：发送到底层后端的批处理的最大大小（以字节为单位）。
 - `audit-log-truncate-max-event-size`：发送到底层后端的 Auditing事件的最大字节数。

默认情况下，`truncate` 在webhook和中都被禁用log，集群管理员应该设置 `audit-log-truncate-enabled`或`audit-webhook-truncate-enabled`启用该功能。

##  8.  Auditing 日志格式示例
以下是 Kubernetes  Auditing日志的示例。JSON 结构中的每个键都包含对了解集群中正在发生的事情至关重要的信息。

```bash

{
  "kind": "Event",
  "apiVersion": "audit.k8s.io/v1",
  "level": "RequestResponse",
  "auditID": "fbc474df-2466-4612-ae36-69af2c927f9d",
  "stage": "ResponseComplete",
  "requestURI": "/api/v1/namespaces/default/pods/nginx",
  "verb": "get",
  "user": {
    "username": "system:node:minikube",
    "groups": [
      "system:nodes",
      "system:authenticated"
    ]
  },
  "sourceIPs": [
    "192.168.49.2"
  ],
  "userAgent": "kubelet/v1.21.2 (linux/amd64) kubernetes/092fbfb",
  "objectRef": {
    "resource": "pods",
    "namespace": "default",
    "name": "nginx",
    "apiVersion": "v1"
  },
  "responseStatus": {
    "metadata": {},
    "code": 200
  },
  "responseObject": {
    "kind": "Pod",
    "apiVersion": "v1",
    "metadata": {...},
    "spec": {...},
    "status": {...}
  },
  "requestReceivedTimestamp": "2022-01-18T06:57:18.944663Z",
  "stageTimestamp": "2022-01-18T06:57:18.968543Z",
  "annotations": {
    "authorization.k8s.io/decision": "allow",
    "authorization.k8s.io/reason": ""
  }
}
```
###  8.1 user 信息和 sourceIP
`user`密钥告诉您是哪个用户或服务帐户通过 Kubernetes API 服务器发起了请求，而`sourceIP`密钥提供了该帐户的 IP。IP 地址可以告诉您位置（例如城市、邮政编码或区号）以及用户 ISP 的名称。IP 地址并不总是可靠的，因为它们可以被发出请求的用户更改或隐藏，但在尝试阻止源自 IP 地址范围的恶意活动时，此信息可能很有用。

### 8.2 有关已启动和执行的请求的信息
`verb`、`requestURI`和`objectRef`都提供有关用户在集群中发起的请求的信息。动词键表示执行的Kubernetes HTTP 请求方法：`get`、`post`、`list`、`watch`、`patch`或`delete`。`requestURI`提供有关您在集群中发起的 API 请求的信息——例如，获取所有 pod 或创建新部署的请求。`objectRef`包含有关与请求关联的Kubernetes 对象。`objectRef`、`verb`和`requestURI`提供有关用户发起的请求的完整信息。

### 8.3 已执行操作的响应状态
`responseStatus`和`responseObject`与`annotations`键相结合，可以深入了解对用户或服务帐户发起的请求的响应。`annotations.authorization.k8s.io/decision`提供了允许或拒绝值。在对 Kubernetes 集群执行 Auditing以检测异常行为时，这些键非常有用。

##  9.  Auditing policy 场景
在大多数情况下，GKE 应用以下规则来记录来自 Kubernetes API 服务器的条目：

 - 表示 `create`、`delete` 和 `update` 请求的条目将写入管理员活动日志。
 - 表示 `get`、`list` 和 `updateStatus` 请求的条目将写入数据访问日志。

Kubernetes  Auditing政策文件开头的规则将指定不记录哪些事件。例如，此规则指定不应记录 `kubelet` 发出的针对 `nodes` 资源或 `nodes/status` 资源的任何 `get` 请求。前文中提到过，`None` 级别意味着不应记录任何匹配事件：

```bash
- level: None
  users: ["kubelet"] # legacy kubelet identity
  verbs: ["get"]
  resources:
    - group: "" # core
    resources: ["nodes", "nodes/status"]
```
在 `level: None` 规则列表之后，政策文件包含一列针对特殊情况的规则。例如，下面是一个特殊情况规则，指定在 `Metadata` 级别记录某些请求：

```bash
 - level: Metadata
    resources:
      - group: "" # core
        resources: ["secrets", "configmaps"]
      - group: authentication.k8s.io
        resources: ["tokenreviews"]
    omitStages:
      - "RequestReceived"
```
如果满足以下所有条件，则事件与规则匹配：
 - 事件与政策文件中前面的任何规则都不匹配。
 - 请求针对 `secrets`、`configmaps` 或 `tokenreviews` 类型的资源。
 - 事件不是针对调用的 `RequestReceived` 阶段。

在特殊情况规则列表之后，政策文件列出一些通用规则。 如需查看[脚本](https://github.com/kubernetes/kubernetes/blob/release-1.10/cluster/gce/gci/configure-helper.sh#L706)中的通用规则，您必须用 `known_apis` 的值替换 `${known_apis}`。替换后，您会得到如下规则：

```bash
- level: Request
  verbs: ["get", "list", "watch"]
  resources:
    - group: "" # core
    - group: "admissionregistration.k8s.io"
    - group: "apiextensions.k8s.io"
    - group: "apiregistration.k8s.io"
    - group: "apps"
    - group: "authentication.k8s.io"
    - group: "authorization.k8s.io"
    - group: "autoscaling"
    - group: "batch"
    - group: "certificates.k8s.io"
    - group: "extensions"
    - group: "metrics.k8s.io"
    - group: "networking.k8s.io"
    - group: "policy"
    - group: "rbac.authorization.k8s.io"
    - group: "settings.k8s.io"
    - group: "storage.k8s.io"
  omitStages:
    - "RequestReceived"
```
该规则适用于与政策文件中前面的任何规则均不匹配且不在 `RequestReceived` 阶段的事件。该规则指定，针对属于所列群组之一的任何资源发出的 `get`、`list` 和 `watch` 请求应该在 `Request` 级别记录。

> 注意：`get`、`list` 和 `watch` 请求实际上没有请求正文，因此该规则的真正作用是指定在 `Metadata` 级别创建日志条目。


```bash
- level: RequestResponse
  resources:
    - group: "" # core
    - group: "admissionregistration.k8s.io"
    - group: "apiextensions.k8s.io"
    - group: "apiregistration.k8s.io"
    - group: "apps"
    - group: "authentication.k8s.io"
    - group: "authorization.k8s.io"
    - group: "autoscaling"
    - group: "batch"
    - group: "certificates.k8s.io"
    - group: "extensions"
    - group: "metrics.k8s.io"
    - group: "networking.k8s.io"
    - group: "policy"
    - group: "rbac.authorization.k8s.io"
    - group: "settings.k8s.io"
    - group: "storage.k8s.io"
  omitStages:
    - "RequestReceived"
```
该规则适用于与政策文件中前面的任何规则均不匹配且不在 `RequestReceived` 阶段的事件。具体来说，此规则不适用于读取请求：`get`、`list` 和 `watch`。但此规则适用于写入请求，如 `create`、`update` 和 `delete`。该规则指定应该在 `RequestResponse` 级别记录写入请求。

下一篇是：[kuberentes Auditing 实战](https://blog.csdn.net/xixihahalelehehe/article/details/126280469)

参考：

 - [Kubernetes Auditing](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/)
 - [Google Kubernetes Engine (GKE)  audit-policy](https://cloud.google.com/kubernetes-engine/docs/concepts/audit-policy)
 - [6 Best Practices for Kubernetes Audit Logging](https://goteleport.com/blog/kubernetes-audit-logging/)
 - [Kubernetes Audit Logs | Use Cases & Best Practices](https://www.containiq.com/post/kubernetes-audit-logs)

