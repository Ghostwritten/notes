#  Kubernetes 资源管理策略 ResourceQuota
tags: ResourceQuota,策略
<!-- catalog:~ResourceQuota~ -->



![在这里插入图片描述](https://img-blog.csdnimg.cn/affde18388c34520af101c1c0a80fe09.png)
*《死亡诗社》讲述了威尔顿预科学院一向都是以传统、守旧的方法来教授学生，可是新学期来校的新文学老师基廷（Keating）却一改学校的常规，让自己班上的学生们解放思想，充分发挥学生们的能力。告诉学生们要“活在当下”，并以该原则行事。*


##  1. 简介
简而言之，ResourceQuota 提供了限制每个命名空间的资源消耗的约束。它们只能应用于命名空间级别，这意味着它们可以应用于计算资源并限制命名空间内的对象数量。

Kubernetes 资源配额由`ResourceQuota`对象定义。当应用于命名空间时，它可以限制 CPU 和内存等计算资源以及以下对象的创建：

 - Pods
 - Services
 - Secrets
 - Persistent Volume Claims (PVCs)
 - ConfigMaps


##  2. 场景
当多个用户或团队共享具有固定数量节点的集群时，一个团队可能会使用超过其公平份额的资源。

资源配额是管理员解决此问题的工具。

由`ResourceQuota`对象定义的资源配额提供限制每个命名空间的聚合资源消耗的约束。它可以按类型限制可以在命名空间中创建的对象数量，以及该命名空间中的资源可能消耗的计算资源总量。

资源配额的工作方式如下：

 - 不同的团队在不同的命名空间中工作。这可以通过RBAC强制执行。
 - 管理员为每个命名空间创建一个 ResourceQuota。
 - 用户在命名空间中创建资源（pod、服务等），配额系统会跟踪使用情况，以确保它不超过 ResourceQuota 中定义的硬资源限制。
 - 如果创建或更新资源违反了配额约束，则请求将失败并显示 HTTP 状态代码403 FORBIDDEN，并显示一条消息，解释可能违反的约束。
 - cpu如果在命名空间中为和之类的计算资源启用了配额memory，则用户必须为这些值指定请求或限制；否则，配额系统可能会拒绝创建  pod。提示：使用LimitRanger准入控制器强制对没有计算资源要求的 pod 使用默认值。



ResourceQuota 对象的名称必须是有效的 DNS 子域名。

可以使用命名空间和配额创建的策略示例如下：

 - 在容量为 32 GiB RAM 和 16 CPU 的集群中，让团队 A 使用 20 GiB 和 10  CPU，让 B 使用 10GiB 和 4
   CPU，并保留 2GiB 和 2 CPU以供将来分配。
 - 将“测试”命名空间限制为使用 1 CPU 和 1GiB RAM。让“生产”命名空间使用任意数量。

在集群总容量小于命名空间配额之和的情况下，可能会发生资源争用。这是按照先到先得的原则处理的。

争用或更改配额都不会影响已创建的资源。

##  3. 架构图
![在这里插入图片描述](https://img-blog.csdnimg.cn/683d71bb91214a858bb81d8c9049e1ca.png)


##  4. 启用资源配额
许多 Kubernetes 发行版默认启用资源配额支持。它启用时API 服务器 `--enable-admission-plugins=flagResourceQuota`作为其参数之一。

当特定命名空间中存在 ResourceQuota 时，将在该命名空间中强制执行资源配额。

## 5. 计算资源配额
| 资源名称             | 描述                                   |
|------------------|--------------------------------------|
| limits.cpu       | 在所有处于非终端状态的 Pod 中，CPU 限制的总和不能超过此值。   |
| limits.memory    | 在所有处于非终端状态的 Pod 中，内存限制的总和不能超过此值。     |
| requests.cpu     | 在所有处于非终端状态的 Pod 中，CPU 请求的总和不能超过这个值。  |
| requests.memory  | 在所有处于非终端状态的 Pod 中，内存请求的总和不能超过这个值。    |
| hugepages-\<size\> | 在所有处于非终端状态的 Pod 中，指定大小的大页面请求数不能超过该值。 |
| cpu              | 如同requests.cpu                       |
| memory           | 如同requests.memory                    |

##  6. 资源的资源配额 
除了上面提到的资源，在 1.10 版本中，增加了对 扩展资源的配额支持。

由于扩展资源不允许过度使用，因此在配额中同时指定requests 和limits为同一扩展资源指定是没有意义的。所以对于扩展资源，目前只requests.允许带前缀的配额项。

以 GPU 资源为例，如果资源名称为`nvidia.com/gpu`，并且您想限制一个命名空间中请求的 GPU 总数为 4，您可以定义配额如下：

```bash
requests.nvidia.com/gpu: 4
```

##  7. 存储资源配额
您可以限制给定命名空间中可以请求的[存储资源](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)的总和。

此外，您可以根据关联的存储类限制存储资源的消耗。


| 资源名称                                                              | 描述                                                |
|-------------------------------------------------------------------|---------------------------------------------------|
| requests.storage                                                  | 在所有持久卷声明中，存储请求的总和不能超过此值。                          |
| persistentvolumeclaims                                            | 命名空间中可以存在的PersistentVolumeClaims总数。               |
| <storage-class-name>.storageclass.storage.k8s.io/requests.storage | 在与 关联的所有持久卷声明中<storage-class-name>，存储请求的总和不能超过此值。 |


`<storage-class-name>.storageclass.storage.k8s.io/persistentvolumeclaims`	在与 `storage-class-name` 关联的所有持久卷声明中，可以存在于命名空间中的持久卷声明的总数。
例如，如果运营商想要使用与gold存储类分开的bronze存储类来配额存储，则运营商可以定义如下配额：

 - gold.storageclass.storage.k8s.io/requests.storage: 500Gi
 - bronze.storageclass.storage.k8s.io/requests.storage: 100Gi

在 1.8 版中，添加了对本地临时存储的配额支持作为 `alpha` 功能：

| 资源名称                       | 描述                                 |
|----------------------------|------------------------------------|
| requests.ephemeral-storage | 在命名空间中的所有 Pod 中，本地临时存储请求的总和不能超过此值。 |
| limits.ephemeral-storage   | 在命名空间中的所有 Pod 中，本地临时存储限制的总和不能超过此值。 |
| ephemeral-storage          | 与 相同requests.ephemeral-storage。    |


> 注意：使用 CRI 容器运行时，容器日志将计入临时存储配额。这可能会导致已用完存储配额的 pod 被意外驱逐。

##  8. 对象计数配额 
您可以使用以下语法为所有标准命名空间资源类型的某些资源的总数设置配额：

 - `count/<resource>.<group>`来自非核心组的资源
 - `count/<resource>`来自核心组的资源

以下是用户可能希望放在对象计数配额下的一组示例资源：

 - count/persistentvolumeclaims
 - count/services
 - count/secrets
 - count/configmaps
 - count/replicationcontrollers
 - count/deployments.apps
 - count/replicasets.apps
 - count/statefulsets.apps
 - count/jobs.batch
 - count/cronjobs.batch

相同的语法可用于自定义资源。例如，要在API 组中的`widgets`自定义资源上创建配额，请使用.`example.comcount/widgets.example.com`

使用`count/*`资源配额时，如果对象存在于服务器存储中，则按配额收费。这些类型的配额有助于防止存储资源耗尽。例如，鉴于服务器的大小，您可能希望限制服务器中的 Secret 数量。集群中过多的 Secrets 实际上会阻止服务器和控制器启动。您可以为作业设置配额以防止配置不当的 CronJob。在命名空间中创建过多作业的 CronJobs 可能会导致拒绝服务。

也可以对有限的资源集进行通用对象计数配额。支持以下类型：

| 资源名称                   | 描述                                                                               |
|------------------------|----------------------------------------------------------------------------------|
| configmaps             | 命名空间中可以存在的 ConfigMap 总数。                                                         |
| persistentvolumeclaims | 命名空间中可以存在的PersistentVolumeClaims总数。                                              |
| pods                   | 命名空间中可以存在的处于非终端状态的 Pod 总数。如果.status.phase in (Failed, Succeeded)为真，则 Pod 处于终端状态。 |
| replicationcontrollers | 命名空间中可以存在的 ReplicationController 总数。                                             |
| resourcequotas         | 命名空间中可以存在的 ResourceQuota 总数。                                                     |
| services               | 命名空间中可以存在的服务总数。                                                                  |
| services.loadbalancers | LoadBalancer命名空间中可以存在的类型的服务总数。                                                   |
| services.nodeports     | NodePort命名空间中可以存在的类型的服务总数。                                                       |
| secrets                | 命名空间中可以存在的 Secret 总数。                                                            |

例如，`pods`配额计算并强制限制在pods 单个命名空间中创建的非终端数量。您可能希望pods 在命名空间上设置配额，以避免用户创建许多小型 Pod 并耗尽集群的 Pod IP 供应的情况。

##  9. 配额范围
每个配额可以有一组关联的scopes. 如果配额与枚举范围的交集匹配，则配额只会衡量资源的使用情况。

将范围添加到配额时，它会将其支持的资源数量限制为与该范围相关的资源。在允许集之外的配额上指定的资源会导致验证错误。

| 范围             | 描述                                          |
|----------------|---------------------------------------------|
| Terminating    | 在哪里匹配 pod `.spec.activeDeadlineSeconds >= 0`   |
| NotTerminating | 在哪里匹配 pod  `.spec.activeDeadlineSeconds is nil` |
| BestEffort     | 匹配具有最佳服务质量的 pod。                            |
| NotBestEffort  | 匹配没有尽力而为服务质量的 pod。                          |
| PriorityClass  | 匹配引用指定优先级类的 pod 。                           |

范围将`BestEffort`配额限制为跟踪以下资源：

 - pods

`Terminating`和`NotTerminating`范围将配额限制`NotBestEffort`为`PriorityClass` 跟踪以下资源：

 - pods
 - cpu
 - memory
 - requests.cpu
 - requests.memory
 - limits.cpu
 - limits.memory

支持字段中的`scopeSelector`以下值`operator`：

 - In
 - NotIn
 - Exists
 - DoesNotExist

当定义 `scopeSelector`时，使用以下值之一作为`scopeName`，`operator`必须是`Exists`

 - Terminating
 - NotTerminating
 - BestEffort
 - NotBestEffort

如果`operator`是In或`Not In`，则该`values`字段必须至少有一个值。例如：

```bash
  scopeSelector:
    matchExpressions:
      - scopeName: PriorityClass
        operator: In
        values:
          - middle
```
如果`operator`是`Exists`或`DoesNotExist`，则不得指定该`values`字段。

##  10. pod 配置 PriorityClass 优先级消耗资源
征状态： `Kubernetes v1.17 [stable]`
可以按特定的优先级创建 Pod 。`scopeSelector` 您可以使用配额规范中的字段，根据 Pod 的优先级控制 Pod 对系统资源的消耗。

只有`scopeSelector`在配额规范中选择 pod 时才会匹配和消耗配额。

当配额使用字段限定为优先级时`scopeSelector`，配额对象被限制为仅跟踪以下资源：

 - pods
 - cpu
 - memory
 - ephemeral-storage
 - limits.cpu
 - limits.memory
 - limits.ephemeral-storage
 - requests.cpu
 - requests.memory
 - requests.ephemeral-storage

此示例创建一个配额对象并将其与特定优先级的 pod 匹配。该示例的工作原理如下：

 - 集群中的 Pod 具有三个优先级类别之一，“低”、“中”、“高”。
 - 为每个优先级创建一个配额对象。

将以下 YAML 保存到文件`quota.yml`中。

```bash
apiVersion: v1
kind: List
items:
- apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: pods-high
  spec:
    hard:
      cpu: "1000"
      memory: 200Gi
      pods: "10"
    scopeSelector:
      matchExpressions:
      - operator : In
        scopeName: PriorityClass
        values: ["high"]
- apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: pods-medium
  spec:
    hard:
      cpu: "10"
      memory: 20Gi
      pods: "10"
    scopeSelector:
      matchExpressions:
      - operator : In
        scopeName: PriorityClass
        values: ["medium"]
- apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: pods-low
  spec:
    hard:
      cpu: "5"
      memory: 10Gi
      pods: "10"
    scopeSelector:
      matchExpressions:
      - operator : In
        scopeName: PriorityClass
        values: ["low"]
```

创建 低、中、高 ResourceQuota

```bash
kubectl create -f ./quota.yml
```
验证Used配额正在0使用`kubectl describe quota`.

```bash
$ kubectl describe quota
....
Name:       pods-high
Namespace:  default
Resource    Used  Hard
--------    ----  ----
cpu         0     1k
memory      0     200Gi
pods        0     10


Name:       pods-low
Namespace:  default
Resource    Used  Hard
--------    ----  ----
cpu         0     5
memory      0     10Gi
pods        0     10


Name:       pods-medium
Namespace:  default
Resource    Used  Hard
--------    ----  ----
cpu         0     10
memory      0     20Gi
pods        0     10
```

创建一个优先级为“高”的 pod。将以下 YAML 保存到文件`high-priority-pod.yml`中。

```bash
apiVersion: v1
kind: Pod
metadata:
  name: high-priority
spec:
  containers:
  - name: high-priority
    image: ubuntu
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo hello; sleep 10;done"]
    resources:
      requests:
        memory: "10Gi"
        cpu: "500m"
      limits:
        memory: "10Gi"
        cpu: "500m"
  priorityClassName: high
```
创建 high pod

```bash
kubectl create -f ./high-priority-pod.yml
```
验证 ResourceQuota 使用情况

```bash
kubectl describe quota
Name:       pods-high
Namespace:  default
Resource    Used  Hard
--------    ----  ----
cpu         500m  1k
memory      10Gi  200Gi
pods        1     10


Name:       pods-low
Namespace:  default
Resource    Used  Hard
--------    ----  ----
cpu         0     5
memory      0     10Gi
pods        0     10


Name:       pods-medium
Namespace:  default
Resource    Used  Hard
--------    ----  ----
cpu         0     10
memory      0     20Gi
pods        0     10
```

## 11. 跨命名空间 Pod Affinity Quota
特征状态： `Kubernetes v1.24 [stable]`
运营商可以使用`CrossNamespacePodAffinity`配额范围来限制允许哪些命名空间拥有具有跨命名空间的亲和术语的 pod。具体来说，它控制允许设置哪些 pod `namespaces`或`namespaceSelect`  亲和性术语中的字段。

可能需要防止用户使用跨命名空间亲和性术语，因为具有反亲和性约束（anti-affinity constraints）的 pod 可能会阻止来自所有其他命名空间的 pod 在故障域中被调度。

使用此范围运算符可以防止某些命名空间（foo-ns在下面的示例中）具有使用跨命名空间 pod 亲和性的 pod，方法是在该命名空间中创建一个`CrossNamespaceAffinity`范围和硬限制为 0 的资源配额对象：

```bash
apiVersion: v1
kind: ResourceQuota
metadata:
  name: disable-cross-namespace-affinity
  namespace: foo-ns
spec:
  hard:
    pods: "0"
  scopeSelector:
    matchExpressions:
    - scopeName: CrossNamespaceAffinity
```
如果运营商希望默认禁止使用`namespaces` and `namespaceSelector`并且只允许特定命名空间使用，他们可以`CrossNamespaceAffinity` 通过将 `kube-apiserver` 标志 `--admission-control-config-file` 设置为以下配置文件的路径来配置为有限资源：

```bash
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
- name: "ResourceQuota"
  configuration:
    apiVersion: apiserver.config.k8s.io/v1
    kind: ResourceQuotaConfiguration
    limitedResources:
    - resource: pods
      matchScopes:
      - scopeName: CrossNamespaceAffinity
```




## 12. 创建与查看 ResourceQuota
###  12.1 创建
```bash
kubectl create namespace myspace

cat <<EOF > compute-resources.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
    requests.nvidia.com/gpu: 4
EOF

kubectl create -f ./compute-resources.yaml --namespace=myspace

cat <<EOF > object-counts.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: object-counts
spec:
  hard:
    configmaps: "10"
    persistentvolumeclaims: "4"
    pods: "4"
    replicationcontrollers: "20"
    secrets: "10"
    services: "10"
    services.loadbalancers: "2"
EOF

kubectl create -f ./object-counts.yaml --namespace=myspace
```

###  12.2 查看

```bash
$ kubectl get quota --namespace=myspace
NAME                    AGE
compute-resources       30s
object-counts           32s


$  kubectl describe quota compute-resources --namespace=myspace
Name:                    compute-resources
Namespace:               myspace
Resource                 Used  Hard
--------                 ----  ----
limits.cpu               0     2
limits.memory            0     2Gi
requests.cpu             0     1
requests.memory          0     1Gi
requests.nvidia.com/gpu  0     4

$ kubectl describe quota object-counts --namespace=myspace
Name:                   object-counts
Namespace:              myspace
Resource                Used    Hard
--------                ----    ----
configmaps              0       10
persistentvolumeclaims  0       4
pods                    0       4
replicationcontrollers  0       20
secrets                 1       10
services                0       10
services.loadbalancers  0       2
```
`Kubectl` 还使用以下语法支持所有标准命名空间资源的对象计数配额`count/<resource>.<group>`：

```bash
kubectl create namespace myspace
kubectl create quota test --hard=count/deployments.apps=2,count/replicasets.apps=4,count/pods=3,count/secrets=4 --namespace=myspace
kubectl create deployment nginx --image=nginx --namespace=myspace --replicas=2

$ kubectl describe quota --namespace=myspace
Name:                         test
Namespace:                    myspace
Resource                      Used  Hard
--------                      ----  ----
count/deployments.apps        1     2
count/pods                    2     3
count/replicasets.apps        1     4
count/secrets                 1     4
```
##  13. 默认限制优先级消耗 
可能希望 pod 具有特定优先级，例如 当且仅当存在匹配的配额对象时，应允许在命名空间中使用“`cluster-services`”。

通过这种机制，运营商能够将某些高优先级类的使用限制在有限数量的命名空间中，并且默认情况下并非每个命名空间都能够使用这些优先级类。

要强制执行此操作，应使用`kube-apiserver flag`将路径传递到以下配置文件：`--admission-control-config-file`


```bash
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
- name: "ResourceQuota"
  configuration:
    apiVersion: apiserver.config.k8s.io/v1
    kind: ResourceQuotaConfiguration
    limitedResources:
    - resource: pods
      matchScopes:
      - scopeName: PriorityClass
        operator: In
        values: ["cluster-services"]
```
然后，在命名空间中创建一个资源配额对象`kube-system`：

```bash
apiVersion: v1
kind: ResourceQuota
metadata:
  name: pods-cluster-services
spec:
  scopeSelector:
    matchExpressions:
      - operator : In
        scopeName: PriorityClass
        values: ["cluster-services"]
```
创建

```bash
kubectl apply -f https://k8s.io/examples/policy/priority-class-resourcequota.yaml -n kube-system
```
在这种情况下，如果出现以下情况，将允许创建 pod：

 - `priorityClassName`未指定Pod 。
 - `Pod 的priorityClassName`值被指定为`cluster-services`.
 - `PodpriorityClassName`设置为`cluster-services`，将在`kube-system`命名空间中创建，并且已通过资源配额检查。

如果将 Pod 创建请求`priorityClassName`设置为`cluster-services` 并且将在`kube-system`



##  14. 实例


###  14.1 pod 请求或限制  cpu、mem 超出资源配置
1. 创建 ResourceQuota 文件`quota-mem-cpu.yaml`

```bash
apiVersion: v1
kind: ResourceQuota
metadata:
  name: mem-cpu-demo
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
```

```bash
kubectl create namespace quota-mem-cpu-example
kubectl apply -f https://k8s.io/examples/admin/resource/quota-mem-cpu.yaml --namespace=quota-mem-cpu-example
kubectl get resourcequota mem-cpu-demo --namespace=quota-mem-cpu-example --output=yaml
```
2. 创建 pod 文件`quota-mem-cpu-pod.yaml`
```bash
apiVersion: v1
kind: Pod
metadata:
  name: quota-mem-cpu-demo
spec:
  containers:
  - name: quota-mem-cpu-demo-ctr
    image: nginx
    resources:
      limits:
        memory: "800Mi"
        cpu: "800m"
      requests:
        memory: "600Mi"
        cpu: "400m"
```

```bash
kubectl apply -f https://k8s.io/examples/admin/resource/quota-mem-cpu-pod.yaml --namespace=quota-mem-cpu-example
kubectl get pod quota-mem-cpu-demo --namespace=quota-mem-cpu-example
kubectl get resourcequota mem-cpu-demo --namespace=quota-mem-cpu-example --output=yaml
```

输出显示配额以及已使用了多少配额。您可以看到Pod的内存和CPU请求以及限制没有超过配额。

```bash
status:
  hard:
    limits.cpu: "2"
    limits.memory: 2Gi
    requests.cpu: "1"
    requests.memory: 1Gi
  used:
    limits.cpu: 800m
    limits.memory: 800Mi
    requests.cpu: 400m
    requests.memory: 600Mi
```
**注意，已用内存请求和此新内存请求的总和超过了内存请求配额。600 MiB + 700 MiB> 1 GiB。**

尝试创建Pod：

```bash
kubectl apply -f https://k8s.io/examples/admin/resource/quota-mem-cpu-pod-2.yaml --namespace=quota-mem-cpu-example
```

没有创建第二个Pod。输出显示创建第二个Pod将导致内存请求总数超过内存请求配额。

```bash
Error from server (Forbidden): error when creating "examples/admin/resource/quota-mem-cpu-pod-2.yaml":
pods "quota-mem-cpu-demo-2" is forbidden: exceeded quota: mem-cpu-demo,
requested: requests.memory=700Mi,used: requests.memory=600Mi, limited: requests.memory=1Gi
```




参考：

 - [kubernetes Resource Quotas](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/)
 - [kubernetes quota-memory-cpu-namespace](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/)
 - [为命名空间配置默认的内存请求和限制](https://kubernetes.io/zh/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/)
 - [Pod开销](https://kubernetes.io/zh/docs/concepts/scheduling-eviction/pod-overhead/)
 - [Kubernetes Resource Quota](https://www.densify.com/kubernetes-autoscaling/kubernetes-resource-quota)


