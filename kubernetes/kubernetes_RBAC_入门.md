#  kubernetes RBAC 入门
tags: 对象

[![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7f6156ef9645f98818df37d18acb3fdf.png)](https://movie.douban.com/subject/1291561/?i=0)

*不可以吃太胖哦，会被杀掉的！    ——《千与千寻》*



## 1. 简介
RBAC（Role-Based Access Control，基于角色的访问控制）在Kubernetes的`1.5`版本中引入，在`1.6`版本时升级为`Beta`版本，在1.8版本时升级为`GA`。作为kubeadm安装方式的默认选项，足见其重要程度。相对于其他访问控制方式，新的RBAC具有如下优势。

 - ◎ 对集群中的资源和非资源权限均有完整的覆盖。
 - ◎ 整个RBAC完全由几个API对象完成，同其他API对象一样，可以用kubectl或API进行操作。
 - ◎ 可以在运行时进行调整，无须重新启动API Server。要使用RBAC授权模式，需要在API Server的启动参数中加上`--authorization-mode=RBAC`。

```bash
 kube-apiserver --authorization-mode=Example,RBAC --<其他选项> --<其他选项>
```

下面对RBAC的原理和用法进行说明。

## 2. API 对象
RBAC API 声明了四种 Kubernetes 对象：`Role`、`ClusterRole`、`RoleBinding` 和 `ClusterRoleBinding`。你可以像使用其他 Kubernetes 对象一样， 通过类似 kubectl 这类工具 描述对象, 或修补对象。

### 2.1 角色（Role）
一个角色就是一组权限的集合，这里的权限都是许可形式的，不存在拒绝的规则。在一个命名空间中，可以用角色来定义一个角色，如果是集群级别的，就需要使用ClusterRole了。角色只能对命名空间内的资源进行授权，在下面例子中定义的角色具备读取Pod的权限：

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" 标明 core API 组
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```
rules中的参数说明如下。
◎ `apiGroups`：支持的API组列表，例如“`apiVersion:batch/v1`”“`apiVersion: extensions:v1beta1`”“`apiVersion: apps/v1beta1`”等，
详细的API组说明参见第9章的说明。
◎ `resources`：支持的资源对象列表，例如pods、deployments、
jobs等。
◎ `verbs`：对资源对象的操作方法列表，例如get、watch、list、delete、replace、patch等，详细的操作方法说明参见第9章的说明。
### 2.2 集群角色（ClusterRole）
集群角色除了具有和角色一致的命名空间内资源的管理能力，因其集群级别的范围，还可以用于以下特殊元素的授权。
◎ 集群范围的资源，例如Node。
◎ 非资源型的路径，例如“/healthz”。
◎ 包含全部命名空间的资源，例如pods（用于kubectl get pods --
all-namespaces这样的操作授权）。
下面的集群角色可以让用户有权访问任意一个或所有命名空间的secrets（视其绑定方式而定）：

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" 被忽略，因为 ClusterRoles 不受名字空间限制
  name: secret-reader
rules:
- apiGroups: [""]
  # 在 HTTP 层面，用来访问 Secret 对象的资源的名称为 "secrets"
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```
### 2.3 角色绑定（RoleBinding）和集群角色绑定（ClusterRoleBinding）
角色绑定或集群角色绑定用来把一个角色绑定到一个目标上，绑定目标可以是`User`（用户）、`Group`（组）或者`Service Account`。使用`RoleBinding`为某个命名空间授权，使用`ClusterRoleBinding`为集群范围内授权。
RoleBinding可以引用Role进行授权。下面的例子中的RoleBinding将在default命名空间中把pod-reader角色授予用户jane，这一操作可以让jane读取default命名空间中的Pod：

```bash
apiVersion: rbac.authorization.k8s.io/v1
# 此角色绑定允许 "jane" 读取 "default" 名字空间中的 Pods
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
# 你可以指定不止一个“subject（主体）”
- kind: User
  name: jane # "name" 是不区分大小写的
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" 指定与某 Role 或 ClusterRole 的绑定关系
  kind: Role # 此字段必须是 Role 或 ClusterRole
  name: pod-reader     # 此字段必须与你要绑定的 Role 或 ClusterRole 的名称匹配
  apiGroup: rbac.authorization.k8s.io
```
`RoleBinding`也可以引用`ClusterRole`，对属于同一命名空间内ClusterRole定义的资源主体进行授权。一种常见的做法是集群管理员为集群范围预先定义好一组ClusterRole，然后在多个命名空间中重复使用这些ClusterRole。例如，在下面的例子中，虽然`secret-reader`是一个集群角色，但是因为使用了RoleBinding，所以dave只能读取development命名空间中的secret：

```bash
apiVersion: rbac.authorization.k8s.io/v1
# 此角色绑定使得用户 "dave" 能够读取 "default" 名字空间中的 Secrets
# 你需要一个名为 "secret-reader" 的 ClusterRole
kind: RoleBinding
metadata:
  name: read-secrets
  # RoleBinding 的名字空间决定了访问权限的授予范围。
  # 这里仅授权在 "development" 名字空间内的访问权限。
  namespace: development
subjects:
- kind: User
  name: dave # 'name' 是不区分大小写的
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```
集群角色绑定中的角色只能是集群角色，用于进行集群级别或者对所有命名空间都生效的授权。下面的例子允许manager组的用户读取任意Namespace中的secret：

```bash
apiVersion: rbac.authorization.k8s.io/v1
# 此集群角色绑定允许 “manager” 组中的任何人访问任何名字空间中的 secrets
kind: ClusterRoleBinding
metadata:
  name: read-secrets-global
subjects:
- kind: Group
  name: manager # 'name' 是不区分大小写的
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d92070aaf5d04005360791d55f7e1233.png)

创建了绑定之后，你不能再修改绑定对象所引用的 Role 或 ClusterRole。 试图改变绑定对象的 roleRef 将导致合法性检查错误。 如果你想要改变现有绑定对象中 roleRef 字段的内容，必须删除重新创建绑定对象。

这种限制有两个主要原因：

 1. 针对不同角色的绑定是完全不一样的绑定。要求通过删除/重建绑定来更改 roleRef,
    这样可以确保要赋予绑定的所有主体会被授予新的角色（而不是在允许修改 roleRef的情况下导致所有现有主体胃镜验证即被授予新角色对应的权限）。
 2. 将 roleRef 设置为不可以改变，这使得可以为用户授予对现有绑定对象的 update 权限，这样可以让他们管理主体列表，同时不能更改被授予这些主体的角色。

命令 `kubectl auth reconcile` 可以创建或者更新包含 RBAC 对象的清单文件， 并且在必要的情况下删除和重新创建绑定对象，以改变所引用的角色。 更多相关信息请参照命令用法和示例

## 3. 对资源的引用
多数资源可以用其名称的字符串来表达，也就是Endpoint中的URL相对路径，例如pods。然而，某些Kubernetes API包含下级资源，例如Pod的日志（logs）。Pod日志的Endpoint是`GET/api/v1/namespaces/{namespace}/pods/{name}/log`。
在这个例子中，Pod是一个命名空间内的资源，log就是一个下级资源。要在一个RBAC角色中体现，就需要用斜线“/”来分隔资源和下级资源。若想授权让某个主体同时能够读取Pod和Pod log，则可以配置resources为一个数组：

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-and-pod-logs-reader
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list"]
```
对于某些请求，也可以通过 resourceNames 列表按名称引用资源。 在指定时，可以将请求限定为资源的单个实例。 下面的例子中限制可以 "get" 和 "update" 一个名为 my-configmap 的 ConfigMap：

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: configmap-updater
rules:
- apiGroups: [""]
  # 在 HTTP 层面，用来访问 ConfigMap 的资源的名称为 "configmaps"
  resources: ["configmaps"]
  resourceNames: ["my-configmap"]
  verbs: ["update", "get"]
```

可想而知，resourceName这种用法对list、watch、create或deletecollection操作是无效的，这是因为必须要通过URL进行鉴权，而资源名称在list、watch、create deletecollection请求中只是请求Body数据的一部分。

## 4. 聚合的 ClusterRole
你可以将若干 ClusterRole 聚合（Aggregate） 起来，形成一个复合的 ClusterRole。 某个控制器作为集群控制面的一部分会监视带有 aggregationRule 的 ClusterRole 对象集合。`aggregationRule` 为控制器定义一个标签 选择算符供后者匹配 应该组合到当前 ClusterRole 的 roles 字段中的 ClusterRole 对象。

下面是一个聚合 ClusterRole 的示例：

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: monitoring
aggregationRule:
  clusterRoleSelectors:
  - matchLabels:
      rbac.example.com/aggregate-to-monitoring: "true"
rules: [] # 控制面自动填充这里的规则
```
如果你创建一个与某现有聚合 ClusterRole 的标签选择算符匹配的 ClusterRole， 这一变化会触发新的规则被添加到聚合 ClusterRole 的操作。 下面的例子中，通过创建一个标签同样为 `rbac.example.com/aggregate-to-monitoring: true` 的 ClusterRole，新的规则可被添加到 "monitoring" ClusterRole 中。

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: monitoring-endpoints
  labels:
    rbac.example.com/aggregate-to-monitoring: "true"
# 当你创建 "monitoring-endpoints" ClusterRole 时，
# 下面的规则会被添加到 "monitoring" ClusterRole 中
rules:
- apiGroups: [""]
  resources: ["services", "endpoints", "pods"]
  verbs: ["get", "list", "watch"]
```

默认的面向用户的角色 使用 ClusterRole 聚合。 这使得作为集群管理员的你可以为扩展默认规则，包括为定制资源设置规则， 比如通过 `CustomResourceDefinitions` 或聚合 API 服务器提供的定制资源。
例如，下面的 ClusterRoles 让默认角色 "admin" 和 "edit" 拥有管理自定义资源 "CronTabs" 的权限， "view" 角色对 CronTab 资源拥有读操作权限。 你可以假定 CronTab 对象在 API 服务器所看到的 URL 中被命名为 "crontabs"。

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: aggregate-cron-tabs-edit
  labels:
    # 添加以下权限到默认角色 "admin" 和 "edit" 中
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
rules:
- apiGroups: ["stable.example.com"]
  resources: ["crontabs"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: aggregate-cron-tabs-view
  labels:
    # 添加以下权限到 "view" 默认角色中
    rbac.authorization.k8s.io/aggregate-to-view: "true"
rules:
- apiGroups: ["stable.example.com"]
  resources: ["crontabs"]
  verbs: ["get", "list", "watch"]
```
## 5. Role 示例
以下示例均为从 Role 或 CLusterRole 对象中截取出来，我们仅展示其 rules 部分。

允许读取在核心 API 组下的 "Pods"：

```bash
rules:
- apiGroups: [""]
  # 在 HTTP 层面，用来访问 Pod 的资源的名称为 "pods"
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

允许读/写在 "extensions" 和 "apps" API 组中的 Deployment（在 HTTP 层面，对应 URL 中资源部分为 "deployments"）：

```bash
rules:
- apiGroups: ["extensions", "apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

允许读取核心 API 组中的 "pods" 和读/写 "batch" 或 "extensions" API 组中的 "jobs"：

```bash
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["batch", "extensions"]
  resources: ["jobs"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

允许读取名称为 "my-config" 的 ConfigMap（需要通过 RoleBinding 绑定以 限制为某名字空间中特定的 ConfigMap）：

```bash
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  resourceNames: ["my-config"]
  verbs: ["get"]
```

允许读取在核心组中的 "nodes" 资源（因为 Node 是集群作用域的，所以需要 ClusterRole 绑定到 ClusterRoleBinding 才生效）：

```bash
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]
```

允许针对非资源端点 `/healthz` 和其子路径上发起 GET 和 POST 请求 （必须在 ClusterRole 绑定 ClusterRoleBinding 才生效）：

```bash
rules:
  - nonResourceURLs: ["/healthz", "/healthz/*"] # nonResourceURL 中的 '*' 是一个全局通配符
    verbs: ["get", "post"]
```
## 6. 对主体的引用
`RoleBinding` 或者 `ClusterRoleBinding` 可绑定角色到某 *主体（Subject）*上。 主体可以是组，用户或者 服务账号。

Kubernetes 用字符串来表示用户名。 用户名可以是普通的用户名，像 "alice"；或者是邮件风格的名称，如 "bob@example.com"， 或者是以字符串形式表达的数字 ID。 你作为 Kubernetes 管理员负责配置 身份认证模块 以便后者能够生成你所期望的格式的用户名。

> 注意： 前缀 system: 是 Kubernetes 系统保留的，所以你要确保 所配置的用户名或者组名不能出现上述 system: 前缀。
> 除了对前缀的限制之外，RBAC 鉴权系统不对用户名格式作任何要求。


## 7. RoleBinding 示例
下面示例是 RoleBinding 中的片段，仅展示其 subjects 的部分。

对于名称为 alice@example.com 的用户：

```bash
subjects:
- kind: User
  name: "alice@example.com"
  apiGroup: rbac.authorization.k8s.io
```

对于名称为 frontend-admins 的用户组：

```bash
subjects:
- kind: Group
  name: "frontend-admins"
  apiGroup: rbac.authorization.k8s.io
```

对于 kube-system 名字空间中的默认服务账号：

```bash
subjects:
- kind: ServiceAccount
  name: default
  namespace: kube-system
```

对于 "qa" 名字空间中所有的服务账号：

```bash
subjects:
- kind: Group
  name: system:serviceaccounts:qa
  apiGroup: rbac.authorization.k8s.io
```

对于在任何名字空间中的服务账号：

```bash
subjects:
- kind: Group
  name: system:serviceaccounts
  apiGroup: rbac.authorization.k8s.io
```

对于所有已经过认证的用户：

```bash
subjects:
- kind: Group
  name: system:authenticated
  apiGroup: rbac.authorization.k8s.io
```

对于所有未通过认证的用户：

```bash
subjects:
- kind: Group
  name: system:unauthenticated
  apiGroup: rbac.authorization.k8s.io
```

对于所有用户：

```bash
subjects:
- kind: Group
  name: system:authenticated
  apiGroup: rbac.authorization.k8s.io
- kind: Group
  name: system:unauthenticated
  apiGroup: rbac.authorization.k8s.io
```
## 8. 默认 Roles 和 Role Bindings
API 服务器创建一组默认的 ClusterRole 和 ClusterRoleBinding 对象。 这其中许多是以 `system:` 为前缀的，用以标识对应资源是直接由集群控制面管理的。 所有的默认 ClusterRole 和 ClusterRoleBinding 都有 `kubernetes.io/bootstrapping=rbac-defaults` 标签。

> 注意： 在修改名称包含 system: 前缀的 ClusterRole 和 ClusterRoleBinding 时要格外小心。
> 对这些资源的更改可能导致集群无法继续工作。

| 默认 ClusterRole              | 默认 ClusterRoleBinding                           | 描述                                                                                          |
|-----------------------------|-------------------------------------------------|---------------------------------------------------------------------------------------------|
| system:basic\-user          | system:authenticated 组                          | 允许用户以只读的方式去访问他们自己的基本信息。在 1\.14 版本之前，这个角色在 默认情况下也绑定在 system:unauthenticated 上。               |
| system:discovery            | system:authenticated 组                          | 允许以只读方式访问 API 发现端点，这些端点用来发现和协商 API 级别。 在 1\.14 版本之前，这个角色在默认情况下绑定在 system:unauthenticated 上。 |
| system:public\-info\-viewer | system:authenticated 和 system:unauthenticated 组 | 允许对集群的非敏感信息进行只读访问，它是在 1\.14 版本中引入的。                                                         |

## 9. 面向用户的角色
一些默认的 ClusterRole 不是以前缀 system: 开头的。这些是面向用户的角色。 它们包括超级用户（`Super-User`）角色（`cluster-admin`）、 使用 ClusterRoleBinding 在集群范围内完成授权的角色（`cluster-status`）、 以及使用 RoleBinding 在特定名字空间中授予的角色（admin、edit、view）。

面向用户的 ClusterRole 使用 ClusterRole 聚合以允许管理员在 这些 ClusterRole 上添加用于定制资源的规则。如果想要添加规则到 admin、edit 或者 view， 可以创建带有以下一个或多个标签的 ClusterRole：

```bash
metadata:
  labels:
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
    rbac.authorization.k8s.io/aggregate-to-view: "true"
```

## 10. 核心组件角色
| 默认 ClusterRole                                                                                                                                        | 默认 ClusterRoleBinding                 | 描述                                                                                                                                             |
|-------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------|
| cluster\-admin                                                                                                                                        | system:masters 组                      | 允许超级用户在平台上的任何资源上执行所有操作。 当在 ClusterRoleBinding 中使用时，可以授权对集群中以及所有名字空间中的全部资源进行完全控制。 当在 RoleBinding 中使用时，可以授权控制 RoleBinding 所在名字空间中的所有资源，包括名字空间本身。 |
| admin                                                                                                                                                 | 无                                     | 允许管理员访问权限，旨在使用 RoleBinding 在名字空间内执行授权。 如果在 RoleBinding 中使用，则可授予对名字空间中的大多数资源的读/写权限， 包括创建角色和角色绑定的能力。 但是它不允许对资源配额或者名字空间本身进行写操作。                   |
| edit                                                                                                                                                  | 无                                     | 允许对名字空间的大多数对象进行读/写操作。 它不允许查看或者修改角色或者角色绑定。 不过，此角色可以访问 Secret，以名字空间中任何 ServiceAccount 的身份运行 Pods， 所以可以用来了解名字空间内所有服务账号的 API 访问级别。                 |
| view                                                                                                                                                  | 无                                     |
| system:kube\-scheduler                                                                                                                                | system:kube\-scheduler user           | 允许访问 scheduler 组件所需要的资源。                                                                                                                       |
| system:volume\-scheduler                                                                                                                              | system:kube\-scheduler user           | 允许访问 kube\-scheduler 组件所需要的卷资源。                                                                                                                |
| system:kube\-controller\-manager                                                                                                                      | system:kube\-controller\-manager user | 允许访问控制器管理器 组件所需要的资源。 各个控制回路所需要的权限在控制器角色 详述。                                                                                                    |
| system:node                                                                                                                                           | 无                                     | 允许访问 kubelet 所需要的资源，包括对所有 Secret 的读操作和对所有 Pod 状态对象的写操作。                                                                                        |
| 你应该使用 Node 鉴权组件 和 NodeRestriction 准入插件 而不是 system:node 角色。同时基于 kubelet 上调度执行的 Pod 来授权 kubelet 对 API 的访问。 system:node 角色的意义仅是为了与从 v1\.8 之前版本升级而来的集群兼容。 |
|                                                                                                                                                       |
| system:node\-proxier                                                                                                                                  | system:kube\-proxy user               | 允许访问 kube\-proxy 组件所需要的资源                                                                                                                      |
## 11. 其他组件角色
| 默认 ClusterRole                         | 默认 ClusterRoleBinding                | 描述                                                      |
|----------------------------------------|--------------------------------------|---------------------------------------------------------|
| system:auth\-delegator                 | 无                                    | 允许将身份认证和鉴权检查操作外包出去。 这种角色通常用在插件式 API 服务器上，以实现统一的身份认证和鉴权。 |
| system:heapster                        | 无                                    | 为 Heapster 组件（已弃用）定义的角色。                                |
| system:kube\-aggregator                | 无                                    | 为 kube\-aggregator 组件定义的角色。                             |
| system:kube\-dns                       | 在 kube\-system 名字空间中的 kube\-dns 服务账号 | 为 kube\-dns 组件定义的角色。                                    |
| system:kubelet\-api\-admin             | 无                                    | 允许 kubelet API 的完全访问权限。                                 |
| system:node\-bootstrapper              | 无                                    | 允许访问执行 kubelet TLS 启动引导 所需要的资源。                         |
| system:node\-problem\-detector         | 无                                    | 为 node\-problem\-detector 组件定义的角色。                      |
| system:persistent\-volume\-provisioner | 无                                    | 允许访问大部分 动态卷驱动 所需要的资源                                    |

## 12. 内置控制器的角色
Kubernetes 控制器管理器 运行内建于 Kubernetes 控制面的控制器。 当使用 --use-service-account-credentials 参数启动时, kube-controller-manager 使用单独的服务账号来启动每个控制器。 每个内置控制器都有相应的、前缀为 system:controller: 的角色。 如果控制管理器启动时未设置 --use-service-account-credentials， 它使用自己的身份凭据来运行所有的控制器，该身份必须被授予所有相关的角色。 这些角色包括:

```bash
system:controller:attachdetach-controller
system:controller:certificate-controller
system:controller:clusterrole-aggregation-controller
system:controller:cronjob-controller
system:controller:daemon-set-controller
system:controller:deployment-controller
system:controller:disruption-controller
system:controller:endpoint-controller
system:controller:expand-controller
system:controller:generic-garbage-collector
system:controller:horizontal-pod-autoscaler
system:controller:job-controller
system:controller:namespace-controller
system:controller:node-controller
system:controller:persistent-volume-binder
system:controller:pod-garbage-collector
system:controller:pv-protection-controller
system:controller:pvc-protection-controller
system:controller:replicaset-controller
system:controller:replication-controller
system:controller:resourcequota-controller
system:controller:root-ca-cert-publisher
system:controller:route-controller
system:controller:service-account-controller
system:controller:service-controller
system:controller:statefulset-controller
system:controller:ttl-controller
```
## 13. 对角色绑定创建或更新的限制
例如，下面的 ClusterRole 和 RoleBinding 将允许用户 user-1 把名字空间 user-1-namespace 中的 admin、edit 和 view 角色赋予其他用户：

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: role-grantor
rules:
- apiGroups: ["rbac.authorization.k8s.io"]
  resources: ["rolebindings"]
  verbs: ["create"]
- apiGroups: ["rbac.authorization.k8s.io"]
  resources: ["clusterroles"]
  verbs: ["bind"]
  # 忽略 resourceNames 意味着允许绑定任何 ClusterRole
  resourceNames: ["admin","edit","view"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: role-grantor-binding
  namespace: user-1-namespace
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: role-grantor
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: user-1
```

当启动引导第一个角色和角色绑定时，需要为初始用户授予他们尚未拥有的权限。 对初始角色和角色绑定进行初始化时需要：

使用用户组为 system:masters 的凭据，该用户组由默认绑定关联到 cluster-admin 这个超级用户角色。
如果你的 API 服务器启动时启用了不安全端口（使用 --insecure-port）, 你也可以通过 该端口调用 API ，这样的操作会绕过身份验证或鉴权

## 14. 命令行工具
### 14.1 kubectl create role
创建 Role 对象，定义在某一名字空间中的权限。例如:

创建名称为 "pod-reader" 的 Role 对象，允许用户对 Pods 执行 get、watch 和 list 操作：

```bash
kubectl create role pod-reader --verb=get --verb=list --verb=watch --resource=pods
```

创建名称为 "pod-reader" 的 Role 对象并指定 resourceNames：

```bash
kubectl create role pod-reader --verb=get --resource=pods --resource-name=readablepod --resource-name=anotherpod
```

创建名为 "foo" 的 Role 对象并指定 apiGroups：

```bash
kubectl create role foo --verb=get,list,watch --resource=replicasets.apps
```

创建名为 "foo" 的 Role 对象并指定子资源权限:

```bash
kubectl create role foo --verb=get,list,watch --resource=pods,pods/status
```

创建名为 "my-component-lease-holder" 的 Role 对象，使其具有对特定名称的 资源执行 get/update 的权限：

```bash
kubectl create role my-component-lease-holder --verb=get,list,watch,update --resource=lease --resource-name=my-component
```

### 14.2 kubectl create clusterrole
创建 ClusterRole 对象。例如：

创建名称为 "pod-reader" 的 ClusterRole对象，允许用户对 Pods 对象执行 get、watch和list` 操作：

```bash
kubectl create clusterrole pod-reader --verb=get,list,watch --resource=pods
```

创建名为 "pod-reader" 的 ClusterRole 对象并指定 resourceNames：

```bash
kubectl create clusterrole pod-reader --verb=get --resource=pods --resource-name=readablepod --resource-name=anotherpod
```

创建名为 "foo" 的 ClusterRole 对象并指定 apiGroups：

```bash
kubectl create clusterrole foo --verb=get,list,watch --resource=replicasets.apps
```

创建名为 "foo" 的 ClusterRole 对象并指定子资源:

```bash
kubectl create clusterrole foo --verb=get,list,watch --resource=pods,pods/status
```

创建名为 "foo" 的 ClusterRole 对象并指定 nonResourceURL：

```bash
kubectl create clusterrole "foo" --verb=get --non-resource-url=/logs/*
```

创建名为 "monitoring" 的 ClusterRole 对象并指定 aggregationRule：

```bash
kubectl create clusterrole monitoring --aggregation-rule="rbac.example.com/aggregate-to-monitoring=true"
```

### 14.3 kubectl create rolebinding
在特定的名字空间中对 Role 或 ClusterRole 授权。例如：

在名字空间 "acme" 中，将名为 admin 的 ClusterRole 中的权限授予名称 "bob" 的用户:

```bash
kubectl create rolebinding bob-admin-binding --clusterrole=admin --user=bob --namespace=acme
```

在名字空间 "acme" 中，将名为 view 的 ClusterRole 中的权限授予名字空间 "acme" 中名为 myapp 的服务账号：

```bash
kubectl create rolebinding myapp-view-binding --clusterrole=view --serviceaccount=acme:myapp --namespace=acme
```

在名字空间 "acme" 中，将名为 view 的 ClusterRole 对象中的权限授予名字空间 "myappnamespace" 中名称为 myapp 的服务账号：

```bash
kubectl create rolebinding myappnamespace-myapp-view-binding --clusterrole=view --serviceaccount=myappnamespace:myapp --namespace=acme
```

### 14.4 kubectl create clusterrolebinding
在整个集群（所有名字空间）中用 ClusterRole 授权。例如：

在整个集群范围，将名为 cluster-admin 的 ClusterRole 中定义的权限授予名为 "root" 用户：

```bash
kubectl create clusterrolebinding root-cluster-admin-binding --clusterrole=cluster-admin --user=root
```

在整个集群范围内，将名为 system:node-proxier 的 ClusterRole 的权限授予名为 "system:kube-proxy" 的用户：

```bash
kubectl create clusterrolebinding kube-proxy-binding --clusterrole=system:node-proxier --user=system:kube-proxy
```

在整个集群范围内，将名为 view 的 ClusterRole 中定义的权限授予 "acme" 名字空间中 名为 "myapp" 的服务账号：

```bash
kubectl create clusterrolebinding myapp-view-binding --clusterrole=view --serviceaccount=acme:myapp
```

### 14.5 kubectl auth reconcile
使用清单文件来创建或者更新 rbac.authorization.k8s.io/v1 API 对象。

尚不存在的对象会被创建，如果对应的名字空间也不存在，必要的话也会被创建。 已经存在的角色会被更新，使之包含输入对象中所给的权限。如果指定了 --remove-extra-permissions，可以删除额外的权限。

已经存在的绑定也会被更新，使之包含输入对象中所给的主体。如果指定了 --remove-extra-permissions，则可以删除多余的主体。

例如:

测试应用 RBAC 对象的清单文件，显示将要进行的更改：

```bash
kubectl auth reconcile -f my-rbac-rules.yaml --dry-run
```

应用 RBAC 对象的清单文件，保留角色中的额外权限和绑定中的其他主体：

```bash
kubectl auth reconcile -f my-rbac-rules.yaml
```

应用 RBAC 对象的清单文件, 删除角色中的额外权限和绑定中的其他主体：

```bash
kubectl auth reconcile -f my-rbac-rules.yaml --remove-extra-subjects --remove-extra-permissions
```
## 15. 服务账号权限
为特定应用的服务账户授予角色（最佳实践）

这要求应用在其 Pod 规约中指定 serviceAccountName， 并额外创建服务账号（包括通过 API、应用程序清单、kubectl create serviceaccount 等）。

例如，在名字空间 "my-namespace" 中授予服务账号 "my-sa" 只读权限：

```bash
kubectl create rolebinding my-sa-view \
  --clusterrole=view \
  --serviceaccount=my-namespace:my-sa \
  --namespace=my-namespace
```

将角色授予某名字空间中的 "default" 服务账号

如果某应用没有指定 serviceAccountName，那么它将使用 "default" 服务账号。

说明： "default" 服务账号所具有的权限会被授予给名字空间中所有未指定 serviceAccountName 的 Pod。
例如，在名字空间 "my-namespace" 中授予服务账号 "default" 只读权限：

```bash
kubectl create rolebinding default-view \
  --clusterrole=view \
  --serviceaccount=my-namespace:default \
  --namespace=my-namespace
```

许多插件组件 在 kube-system 名字空间以 "default" 服务账号运行。 要允许这些插件组件以超级用户权限运行，需要将集群的 cluster-admin 权限授予 kube-system 名字空间中的 "default" 服务账号。

说明： 启用这一配置意味着在 kube-system 名字空间中包含以超级用户账号来访问 API 的 Secrets。

```bash
kubectl create clusterrolebinding add-on-cluster-admin \
  --clusterrole=cluster-admin \
  --serviceaccount=kube-system:default
```

将角色授予名字空间中所有服务账号

如果你想要名字空间中所有应用都具有某角色，无论它们使用的什么服务账号， 可以将角色授予该名字空间的服务账号组。

例如，在名字空间 "my-namespace" 中的只读权限授予该名字空间中的所有服务账号：

```bash
kubectl create rolebinding serviceaccounts-view \
  --clusterrole=view \
  --group=system:serviceaccounts:my-namespace \
  --namespace=my-namespace
```

在集群范围内为所有服务账户授予一个受限角色（不鼓励）

如果你不想管理每一个名字空间的权限，你可以向所有的服务账号授予集群范围的角色。

例如，为集群范围的所有服务账号授予跨所有名字空间的只读权限：

```bash
kubectl create clusterrolebinding serviceaccounts-view \
  --clusterrole=view \
  --group=system:serviceaccounts
```

授予超级用户访问权限给集群范围内的所有服务帐户（强烈不鼓励）

如果你不关心如何区分权限，你可以将超级用户访问权限授予所有服务账号。

警告： 这样做会允许所有应用都对你的集群拥有完全的访问权限，并将允许所有能够读取 Secret（或创建 Pod）的用户对你的集群有完全的访问权限。

```bash
kubectl create clusterrolebinding serviceaccounts-cluster-admin \
  --clusterrole=cluster-admin \
  --group=system:serviceaccounts
```
## 16. 宽松的 RBAC 权限
下面的策略允许 所有 服务帐户充当集群管理员。 容器中运行的所有应用程序都会自动收到服务帐户的凭据，可以对 API 执行任何操作， 包括查看 Secrets 和修改权限。这一策略是不被推荐的。

```bash
kubectl create clusterrolebinding permissive-binding \
  --clusterrole=cluster-admin \
  --user=admin \
  --user=kubelet \
  --group=system:serviceaccounts
```

## 17. 示例
###  容器网络接口（CNI）weavework

```bash
controlplane $ cat /opt/weave-kube.yaml
apiVersion: v1
kind: List
items:
 - apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: weave-net
      annotations:
        cloud.weave.works/launcher-info: |-
          {
            "original-request": {
              "url": "/k8s/v1.10/net.yaml?k8s-version=v1.16.0",
              "date": "Mon Oct 28 2019 18:38:09 GMT+0000 (UTC)"
            },
            "email-address": "support@weave.works"
          }
      labels:
        name: weave-net
      namespace: kube-system
 - apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: weave-net
      annotations:
        cloud.weave.works/launcher-info: |-
          {
            "original-request": {
              "url": "/k8s/v1.10/net.yaml?k8s-version=v1.16.0",
              "date": "Mon Oct 28 2019 18:38:09 GMT+0000 (UTC)"
            },
            "email-address": "support@weave.works"
          }
      labels:
        name: weave-net
    rules:
      - apiGroups:
          - ''
        resources:
          - pods
          - namespaces
          - nodes
        verbs:
          - get
          - list
          - watch
      - apiGroups:
          - networking.k8s.io
        resources:
          - networkpolicies
        verbs:
          - get
          - list
          - watch
      - apiGroups:
          - ''
        resources:
          - nodes/status
        verbs:
          - patch
          - update
 - apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: weave-net
      annotations:
        cloud.weave.works/launcher-info: |-
          {
            "original-request": {
              "url": "/k8s/v1.10/net.yaml?k8s-version=v1.16.0",
              "date": "Mon Oct 28 2019 18:38:09 GMT+0000 (UTC)"
            },
            "email-address": "support@weave.works"
          }
      labels:
        name: weave-net
    roleRef:
      kind: ClusterRole
      name: weave-net
      apiGroup: rbac.authorization.k8s.io
    subjects:
      - kind: ServiceAccount
        name: weave-net
        namespace: kube-system
 - apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      name: weave-net
      annotations:
        cloud.weave.works/launcher-info: |-
          {
            "original-request": {
              "url": "/k8s/v1.10/net.yaml?k8s-version=v1.16.0",
              "date": "Mon Oct 28 2019 18:38:09 GMT+0000 (UTC)"
            },
            "email-address": "support@weave.works"
          }
      labels:
        name: weave-net
      namespace: kube-system
    rules:
      - apiGroups:
          - ''
        resourceNames:
          - weave-net
        resources:
          - configmaps
        verbs:
          - get
          - update
      - apiGroups:
          - ''
        resources:
          - configmaps
        verbs:
          - create
 - apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: weave-net
      annotations:
        cloud.weave.works/launcher-info: |-
          {
            "original-request": {
              "url": "/k8s/v1.10/net.yaml?k8s-version=v1.16.0",
              "date": "Mon Oct 28 2019 18:38:09 GMT+0000 (UTC)"
            },
            "email-address": "support@weave.works"
          }
      labels:
        name: weave-net
      namespace: kube-system
    roleRef:
      kind: Role
      name: weave-net
      apiGroup: rbac.authorization.k8s.io
    subjects:
      - kind: ServiceAccount
        name: weave-net
        namespace: kube-system
 - apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: weave-net
      annotations:
        cloud.weave.works/launcher-info: |-
          {
            "original-request": {
              "url": "/k8s/v1.10/net.yaml?k8s-version=v1.16.0",
              "date": "Mon Oct 28 2019 18:38:09 GMT+0000 (UTC)"
            },
            "email-address": "support@weave.works"
          }
      labels:
        name: weave-net
      namespace: kube-system
    spec:
      minReadySeconds: 5
      selector:
        matchLabels:
          name: weave-net
      template:
        metadata:
          labels:
            name: weave-net
        spec:
          containers:
            - name: weave
              command:
                - /home/weave/launch.sh
              env:
                - name: IPALLOC_RANGE
                  value: 10.32.0.0/24
                - name: HOSTNAME
                  valueFrom:
                    fieldRef:
                      apiVersion: v1
                      fieldPath: spec.nodeName
              image: 'docker.io/weaveworks/weave-kube:2.6.0'
              readinessProbe:
                httpGet:
                  host: 127.0.0.1
                  path: /status
                  port: 6784
              resources:
                requests:
                  cpu: 10m
              securityContext:
                privileged: true
              volumeMounts:
                - name: weavedb
                  mountPath: /weavedb
                - name: cni-bin
                  mountPath: /host/opt
                - name: cni-bin2
                  mountPath: /host/home
                - name: cni-conf
                  mountPath: /host/etc
                - name: dbus
                  mountPath: /host/var/lib/dbus
                - name: lib-modules
                  mountPath: /lib/modules
                - name: xtables-lock
                  mountPath: /run/xtables.lock
            - name: weave-npc
              env:
                - name: HOSTNAME
                  valueFrom:
                    fieldRef:
                      apiVersion: v1
                      fieldPath: spec.nodeName
              image: 'docker.io/weaveworks/weave-npc:2.6.0'
              resources:
                requests:
                  cpu: 10m
              securityContext:
                privileged: true
              volumeMounts:
                - name: xtables-lock
                  mountPath: /run/xtables.lock
          hostNetwork: true
          hostPID: true
          restartPolicy: Always
          securityContext:
            seLinuxOptions: {}
          serviceAccountName: weave-net
          tolerations:
            - effect: NoSchedule
              operator: Exists
          volumes:
            - name: weavedb
              hostPath:
                path: /var/lib/weave
            - name: cni-bin
              hostPath:
                path: /opt
            - name: cni-bin2
              hostPath:
                path: /home
            - name: cni-conf
              hostPath:
                path: /etc
            - name: dbus
              hostPath:
                path: /var/lib/dbus
            - name: lib-modules
              hostPath:
                path: /lib/modules
            - name: xtables-lock
              hostPath:
                path: /run/xtables.lock
                type: FileOrCreate
      updateStrategy:
        type: RollingUpdate
```

参考：

 - [kubernetes Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
 - [Extend Kubernetes with Custom Resource Definitions and RBAC for ServiceAccounts](https://thorsten-hans.com/custom-resource-definitions-with-rbac-for-serviceaccounts)
 - [Demystifying RBAC in Kubernetes](https://www.cncf.io/blog/2018/08/01/demystifying-rbac-in-kubernetes/)

