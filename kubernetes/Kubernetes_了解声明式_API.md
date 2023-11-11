#  Kubernetes 了解声明式 API



## 1. Kubernetes API概述
Kubernetes API是集群系统中的重要组成部分，Kubernetes中各种资源（对象）的数据都通过该API接口被提交到后端的持久化存储（`etcd`）中，Kubernetes集群中的各部件之间通过该API接口实现解耦合，同时Kubernetes集群中一个重要且便捷的管理工具kubectl也是通过访问该API接口实现其强大的管理功能的。Kubernetes API中的资源对象都拥有通用的元数据，资源对象也可能存在嵌套现象，比如在一个Pod里面嵌套多个Container。创建一个API对象是指通过API调用创建一条有意义的记录，该记录一旦创建，Kubernetes就将确保对应的资源对象会被自动创建并托管维护。

在Kubernetes系统中，在大多数情况下，API定义和实现都符合标准的HTTP REST格式，比如通过标准的HTTP动词（POST、PUT、GET、DELETE）来完成对相关资源对象的查询、创建、修改、删除等操作。但同时，Kubernetes也为某些非标准的REST行为实现了附加的API接口，例如Watch某个资源的变化、进入容器执行某个操作等。
Kubernetes API文档官网为[https://kubernetes.io/docs/reference](https://kubernetes.io/docs/reference)，可以通过相关链接查看不同版本的API文档，例如Kubernetes 1.14版本的链接
为[https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.14](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.14)。
在Kubernetes 1.13版本及之前的版本中，在Kubernetes API中，一个API的顶层（Top Level）元素由kind、apiVersion、metadata、spec和status这5部分组成，接下来分别对这5部分进行说明。

### 1.1 kind
kind表明对象有以下三大类别。
（1`）对象`（objects）：代表系统中的一个永久资源（实体），例如Pod、RC、Service、Namespace及Node等。通过操作这些资源的属性，客户端可以对该对象进行创建、修改、删除和获取操作。
（2）`列表`（list）：一个或多个资源类别的集合。所有列表都通过`items域`获得对象数组，例如`PodLists`、`ServiceLists`、`NodeLists`。大部分被定义在系统中的对象都有一个返回所有资源集合的端点，以及零到多个返回所有资源集合的子集的端点。某些对象有可能是单例对象（singletons），例如当前用户、系统默认用户等，这些对象没有列表。
（3）`简单类别`（simple）：该类别包含作用在对象上的特殊行为和非持久实体。该类别限制了使用范围，它有一个通用元数据的有限集合，例如`Binding`、`Status`。
### 1.2 apiVersion
`apiVersion`表明API的版本号，当前版本默认只支持v1。
### 1.3. Metadata
`Metadata`是资源对象的元数据定义，是集合类的元素类型，包含一组由不同名称定义的属性。在Kubernetes中每个资源对象都必须包含以下3种Metadata。
（1）`namespace`：对象所属的命名空间，如果不指定，系统则会将对象置于名为default的系统命名空间中。
（2）`name`：对象的名称，在一个命名空间中名称应具备唯一性。
（3）`uid`：系统为每个对象都生成的唯一ID，符合RFC 4122规范的定义。
此外，每种对象都还应该包含以下几个重要元数据。
（1）`labels`：用户可定义的“标签”，键和值都为字符串的map，是对象进行组织和分类的一种手段，通常用于标签选择器，用来匹配目标对象。
（2）`annotations`：用户可定义的“注解”，键和值都为字符串的map，被Kubernetes内部进程或者某些外部工具使用，用于存储和获取关于该对象的特定元数据。
（3）`resourceVersion`：用于识别该资源内部版本号的字符串，在用于Watch操作时，可以避免在GET操作和下一次Watch操作之间造成的信息不一致，客户端可以用它来判断资源是否改变。该值应该被客户端看作不透明，且不做任何修改就返回给服务端。客户端不应该假定版本信息具有跨命名空间、跨不同资源类别、跨不同服务器的含义。
（4）`creationTimestamp`：系统记录创建对象时的时间戳，符合RFC3339规范。
（5）`deletionTimestamp`：系统记录删除对象时的时间戳，符合RFC3339规范。
（6）`selfLink`：通过API访问资源自身的URL，例如一个Pod的link可能是“`/api/v1/namespaces/ default/pods/frontend-o8bg4`”。

### 1.4. spec
spec是集合类的元素类型，用户对需要管理的对象进行详细描述的主体部分都在spec里给出，它会被Kubernetes持久化到etcd中保存，系统通过spec的描述来创建或更新对象，以达到用户期望的对象运行状态。spec的内容既包括用户提供的**配置设置**、**默认值**、**属性的初始化值**，也包括在对象创建过程中由其他相关组件（例schedulers、auto-scalers）创建或修改的对象属性，比如Pod的Service IP地址。如果spec被删除，那么该对象将会从系统中删除。

### 1.5. Status
Status用于记录对象在系统中的当前状态信息，它也是集合类元素类型，status在一个自动处理的进程中被持久化，可以在流转的过程中生成。如果观察到一个资源丢失了它的状态（Status），则该丢失的状态可能被重新构造。以Pod为例，Pod的status信息主要包括conditions、containerStatuses、hostIP、phase、podIP、startTime等，其中比较重要的两个状态属性如下。
（1）`phase`：描述对象所处的生命周期阶段，phase的典型值是`Pending`（创建中）、`Running`、`Active`（正在运行中）或`Terminated`（已终结），这几种状态对于不同的对象可能有轻微的差别，此外，关于当前phase附加的详细说明可能包含在其他域中。
（2）condition：表示条件，由条件类型和状态值组成，目前仅有一种条件类型：Ready，对应的状态值可以为True、False或Unknown。一个对象可以具备多种condition，而condition的状态值也可能不断发生变化，condition可能附带一些信息，例如最后的探测时间或最后的转变时间。
`Kubernetes从1.14`版本开始，使用`OpenAPI`（https://www.openapis.org）的格式对API进行查询，其访问地址为`http://<master-ip>: <master-port>/openapi/v2`。例如，使用命令行工
具curl进行查询：

## 2. Kubernetes API版本的演进策略

为了在兼容旧版本的同时不断升级新的API，Kubernetes提供了多版本API的支持能力，每个版本的API都通过一个版本号路径前缀进行区分，例如/api/v1beta3。在通常情况下，新旧几个不同的API版本都能涵盖所有的Kubernetes资源对象，在不同的版本之间，这些API接口存在一些细微差别。Kubernetes开发团队基于API级别选择版本而不是基于资源和域级别，是为了确保API能够清晰、连续地描述一个系统资源和行为	的视图，能够控制访问的整个过程和控制实验性API的访问。
API的版本号通常用于描述API的成熟阶段，例如：

 - ◎ v1表示GA稳定版本；
 - ◎ v1beta3表示Beta版本（预发布版本）；
 - ◎ v1alpha1表示Alpha版本（实验性的版本）。

当某个API的实现达到一个新的GA稳定版本时（如v2），旧的GA版本（如v1）和Beta版本（例如v2beta1）将逐渐被废弃，Kubernetes建议废弃的时间如下。

 - ◎ 对于旧的GA版本（如v1），Kubernetes建议废弃的时间应不少于12个月或3个大版本Release的时间，选择最长的时间。
 - ◎ 对旧的Beta版本（如v2beta1），Kubernetes建议废弃的时间应不少于9个月或3个大版本Release的时间，选择最长的时间。
 - ◎ 对旧的Alpha版本，则无须等待，可以直接废弃。

##  3. API Groups（API组）

为了更容易对API进行扩展，Kubernetes使用API Groups（API组）进行标识。API Groups以REST URL中的路径进行定义。当前支持两类API groups。
◎ `Core Groups`（核心组），也可以称之为Legacy Groups，作为Kubernetes最核心的API，其特点是没有“组”的概念，例如“v1”，在资源对象的定义中表示“`apiVersion:v1`”。
◎ 具有分组信息的API，以`/apis/$GROUP_NAME/$VERSIONURL`路径进行标识，在资源对象的定义中表示为“a`piVersion:$GROUP_NAME/$VERSION`”，例如：“apiVersion:
batch/v1”“apiVersion: extensions:v1beta1”“apiVersion: apps/v1beta1”等，详细的API列表请参见官网[https://kubernetes.io/docs/reference](https://kubernetes.io/docs/reference)，目前根据Kubernetes的不同版本有不同的API说明页面。

如果要启用或禁用特定的API组，则需要在API Server的启动参数中设置`--runtime-config`进行声明，例如，`--runtime-config=batch/v2alpha1`表示启用API组“batch/v2alpha1”；也可以设置-`-runtimeconfig=batch/v1=false`表示禁用API组“batch/v1”。多个API组的设置以逗号分隔。在当前的API Server服务中，`DaemonSets`、`Deployments`、`HorizontalPodAutoscalers`、`Ingress`、`Jobs`和`ReplicaSets`所属的API组是默认启用的。

## 4. API声明式区分

```bash
$ docker service create --name nginx --replicas 2  nginx
$ docker service update --image nginx:1.7.9 nginx
```
第一条 create 命令创建了这两个容器，而第二条 update 命令则把它们“滚动更新”成了一个新的镜像。对于这种使用方式，我们称为**命令式命令行操作**。


```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80




$ kubectl create -f nginx.yaml
```
如果要更新这两个 Pod 使用的 Nginx 镜像，该怎么办呢？`kubectl set image` 和 `kubectl edit` 命令，来直接修改 Kubernetes 里的 API 对象。不过，相信很多人都有这样的想法，我能不能通过修改本地 YAML 文件来完成这个操作呢？


```bash
...
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
```

```bash
$ kubectl replace -f nginx.yaml
```
对于上面这种先 `kubectl create`，再 `replace` 的操作，我们称为**命令式配置文件操作**。

就是说，它的处理方式，其实跟前面 Docker Swarm 的两句命令，没什么本质上的区别。只不过，它是把 Docker 命令行里的参数，写在了配置文件里而已。


```bash

...
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9


$ kubectl apply -f nginx.yaml
```
这种方式叫**声明式 API**
Kubernetes 就会立即触发这个 Deployment 的“滚动更新”，它跟 kubectl replace 命令有什么本质区别吗？

**kubectl replace 的执行过程，是使用新的 YAML 文件中的 API 对象，替换原有的 API 对象；而 kubectl apply，则是执行了一个对原有 API 对象的 PATCH 操作。**类似地，kubectl set image 和 kubectl edit 也是对已有 API 对象的修改。

更进一步地，这意味着 kube-apiserver 在响应命令式请求（比如，kubectl replace）的时候，一次只能处理一个写请求，否则会有产生冲突的可能。而对于声明式请求（比如，kubectl apply），一次能处理多个写操作，并且具备 Merge 能力。

##  5. 以Istio 项目理解声明式API的意义
![在这里插入图片描述](https://img-blog.csdnimg.cn/0d2a65e8ace64277a6857ccec28ca89a.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
**Istio 最根本的组件，是运行在每一个应用 Pod 里的 Envoy 容器。**这个 Envoy 项目是 Lyft 公司推出的一个高性能 C++ 网络代理，也是 Lyft 公司对 Istio 项目的唯一贡献。

而 Istio 项目，则把这个代理服务以 `sidecar 容器`的方式，运行在了每一个被治理的应用 Pod 中。我们知道，Pod 里的所有容器都共享同一个 Network Namespace。所以，**Envoy 容器就能够通过配置 Pod 里的 iptables 规则，把整个 Pod 的进出流量接管下来**。

这时候，Istio 的控制层（Control Plane）里的 `Pilot` 组件，就能够通过调用每个 `Envoy` 容器的 API，对这个 Envoy 代理进行配置，从而实现微服务治理。


假设这个 Istio 架构图左边的 Pod 是已经在运行的应用，而右边的 Pod 则是我们刚刚上线的应用的新版本。这时候，Pilot 通过调节这两 Pod 里的 Envoy 容器的配置，从而将 90% 的流量分配给旧版本的应用，将 10% 的流量分配给新版本应用，并且，还可以在后续的过程中随时调整。这样，一个典型的“灰度发布”的场景就完成了。比如，Istio 可以调节这个流量从 90%-10%，改到 80%-20%，再到 50%-50%，最后到 0%-100%，就完成了这个灰度发布的过程。更重要的是，在整个微服务治理的过程中，无论是对 Envoy 容器的部署，还是像上面这样对 Envoy 代理的配置，用户和应用都是完全“无感”的。这时候，你可能会有所疑惑：Istio 项目明明需要在每个 Pod 里安装一个 Envoy 容器，又怎么能做到“无感”的呢？

实际上，**Istio 项目使用的，是 Kubernetes 中的一个非常重要的功能，叫作 Dynamic Admission Control**。


在 Kubernetes 项目中，当一个 Pod 或者任何一个 API 对象被提交给 APIServer 之后，总有一些“**初始化**”性质的工作需要在它们被 Kubernetes 项目正式处理之前进行。比如，自动为所有 Pod 加上某些标签（Labels）。而这个“初始化”操作的实现，借助的是一个叫作 **Admission 的功能**。它其实是 Kubernetes 项目里一组被称为 `Admission Controller` 的代码，可以选择性地被编译进 APIServer 中，在 API 对象创建之后会被立刻调用到。


但这就意味着，**如果你现在想要添加一些自己的规则到 Admission Controller，就会比较困难。因为，这要求重新编译并重启 APIServer**。显然，这种使用方法对 Istio 来说，影响太大了。所以，Kubernetes 项目为我们额外提供了一种**“热插拔”式的 Admission 机制**，它就是 **Dynamic Admission Control**，也叫作：**Initializer**。

例如

```bash
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
```
接下来，Istio 项目要做的，就是在这个 Pod YAML 被提交给 Kubernetes 之后，在它对应的 API 对象里自动加上 Envoy 容器的配置，使这个对象变成如下所示的样子：


```bash
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
  - name: envoy
    image: lyft/envoy:845747b88f102c0fd262ab234308e9e22f693a1
    command: ["/usr/local/bin/envoy"]
    ...
```
可以看到，被 Istio 处理后的这个 Pod 里，除了用户自己定义的 myapp-container 容器之外，多出了一个叫作 envoy 的容器，它就是 Istio 要使用的 Envoy 代理。那么，Istio 又是如何在用户完全不知情的前提下完成这个操作的呢？

**Istio 要做的，就是编写一个用来为 Pod“自动注入”Envoy 容器的 Initializer。**
**首先，Istio 会将这个 Envoy 容器本身的定义，以 ConfigMap 的方式保存在 Kubernetes 当中。这个 ConfigMap**（名叫：envoy-initializer）的定义如下所示：


```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: envoy-initializer
data:
  config: |
    containers:
      - name: envoy
        image: lyft/envoy:845747db88f102c0fd262ab234308e9e22f693a1
        command: ["/usr/local/bin/envoy"]
        args:
          - "--concurrency 4"
          - "--config-path /etc/envoy/envoy.json"
          - "--mode serve"
        ports:
          - containerPort: 80
            protocol: TCP
        resources:
          limits:
            cpu: "1000m"
            memory: "512Mi"
          requests:
            cpu: "100m"
            memory: "64Mi"
        volumeMounts:
          - name: envoy-conf
            mountPath: /etc/envoy
    volumes:
      - name: envoy-conf
        configMap:
          name: envoy
```
相信你已经注意到了，**这个 ConfigMap 的 data 部分，正是一个 Pod 对象的一部分定义**。其中，我们可以看到 Envoy 容器对应的 containers 字段，以及一个用来声明 Envoy 配置文件的 volumes 字段。不难想到，**Initializer 要做的工作，就是把这部分 Envoy 相关的字段，自动添加到用户提交的 Pod 的 API 对象里。可是，用户提交的 Pod 里本来就有 containers 字段和 volumes 字段，所以 Kubernetes 在处理这样的更新请求时，就必须使用类似于 git merge 这样的操作，才能将这两部分内容合并在一起**。

所以说，**在 Initializer 更新用户的 Pod 对象的时候，必须使用 PATCH API 来完成。而这种 PATCH API，正是声明式 API 最主要的能力**。

接下来，Istio 将一个编写好的 `Initializer`，作为一个 Pod 部署在 Kubernetes 中。这个 Pod 的定义非常简单，如下所示：

```bash
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: envoy-initializer
  name: envoy-initializer
spec:
  containers:
    - name: envoy-initializer
      image: envoy-initializer:0.0.1
      imagePullPolicy: Always
```
我们可以看到，**这个 envoy-initializer 使用的 envoy-initializer:0.0.1 镜像，就是一个事先编写好的“自定义控制器”（Custom Controller）**，一个 Kubernetes 的控制器，实际上就是一个“死循环”：它不断地获取“实际状态”，然后与“期望状态”作对比，并以此为依据决定下一步的操作。

**而 Initializer 的控制器，不断获取到的“实际状态”，就是用户新创建的 Pod。而它的“期望状态”，则是：这个 Pod 里被添加了 Envoy 容器的定义**。

Go 语言风格的伪代码描述这个控制逻辑，如下所示：


```bash
for {
  // 获取新创建的Pod
  pod := client.GetLatestPod()
  // Diff一下，检查是否已经初始化过
  if !isInitialized(pod) {
    // 没有？那就来初始化一下
    doSomething(pod)
  }
}
```

 - 如果这个 Pod 里面已经添加过 Envoy 容器，那么就“放过”这个 Pod，进入下一个检查周期。
 - 而如果还没有添加过 Envoy 容器的话，它就要进行 `Initialize` 操作了，即：修改该 Pod 的 API 对象（doSomething 函数）。
这时候，你应该立刻能想到，Istio 要往这个 Pod 里合并的字段，正是我们之前保存在 `envoy-initializer` 这个 ConfigMap 里的数据（即：它的 data 字段的值）。

所以，在 Initializer 控制器的工作逻辑里，它首先会从 APIServer 中拿到这个 ConfigMap：

```go
func doSomething(pod) {
  cm := client.Get(ConfigMap, "envoy-initializer")
}
```

然后，把这个 ConfigMap 里存储的 `containers` 和 `volumes` 字段，直接添加进一个空的 Pod 对象里：

```go
func doSomething(pod) {
  cm := client.Get(ConfigMap, "envoy-initializer")
  
  newPod := Pod{}
  newPod.Spec.Containers = cm.Containers
  newPod.Spec.Volumes = cm.Volumes
}
```
现在，关键来了。
**Kubernetes 的 API 库，为我们提供了一个方法，使得我们可以直接使用新旧两个 Pod 对象，生成一个 TwoWayMergePatch**：


```go
func doSomething(pod) {
  cm := client.Get(ConfigMap, "envoy-initializer")

  newPod := Pod{}
  newPod.Spec.Containers = cm.Containers
  newPod.Spec.Volumes = cm.Volumes

  // 生成patch数据
  patchBytes := strategicpatch.CreateTwoWayMergePatch(pod, newPod)

  // 发起PATCH请求，修改这个pod对象
  client.Patch(pod.Name, patchBytes)
}
```
**有了这个 TwoWayMergePatch 之后，Initializer 的代码就可以使用这个 patch 的数据，调用 Kubernetes 的 Client，发起一个 PATCH 请求。**

这样，一个用户提交的 Pod 对象里，就会被自动加上 Envoy 容器相关的字段。当然，Kubernetes 还允许你通过配置，来指定要对什么样的资源进行这个 Initialize 操作，比如下面这个例子：

```bash
apiVersion: admissionregistration.k8s.io/v1alpha1
kind: InitializerConfiguration
metadata:
  name: envoy-config
initializers:
  // 这个名字必须至少包括两个 "."
  - name: envoy.initializer.kubernetes.io
    rules:
      - apiGroups:
          - "" // 前面说过， ""就是core API Group的意思
        apiVersions:
          - v1
        resources:
          - pods
```
**这个配置，就意味着 Kubernetes 要对所有的 Pod 进行这个 `Initialize` 操作，并且，我们指定了负责这个操作的 Initializer**，名叫：`envoy-initializer`。

而一旦这个 `InitializerConfiguration` 被创建，Kubernetes 就会把这个 `Initializer` 的名字，加在所有新创建的 Pod 的 `Metadata` 上，格式如下所示：


```go
apiVersion: v1
kind: Pod
metadata:
  initializers:
    pending:
      - name: envoy.initializer.kubernetes.io
  name: myapp-pod
  labels:
    app: myapp
...
```
可以看到，每一个新创建的 Pod，都会自动携带了 `metadata.initializers.pending` 的 Metadata 信息。

这个 Metadata，正是接下来 Initializer 的控制器判断这个 Pod 有没有执行过自己所负责的初始化操作的重要依据（也就是前面伪代码中 isInitialized() 方法的含义）。

**这也就意味着，当你在 Initializer 里完成了要做的操作后，一定要记得将这个 metadata.initializers.pending 标志清除掉。这一点，你在编写 Initializer 代码的时候一定要非常注意。**

此外，除了上面的配置方法，你还可以在具体的 Pod 的 Annotation 里添加一个如下所示的字段，从而声明要使用某个 `Initializer`：


```bash
apiVersion: v1
kind: Pod
metadata
  annotations:
    "initializer.kubernetes.io/envoy": "true"
    ...
```
在这个 Pod 里，我们添加了一个 Annotation，写明： `initializer.kubernetes.io/envoy=true`。这样，就会使用到我们前面所定义的 envoy-initializer 了。

以上，就是关于 Initializer 最基本的工作原理和使用方法了。相信你此时已经明白，**Istio 项目的核心，就是由无数个运行在应用 Pod 中的 Envoy 容器组成的服务代理网格。这也正是 Service Mesh 的含义。**     [美好的demo](https://github.com/resouer/kubernetes-initializer-tutorial)手动去完成它。

而这个机制得以实现的原理，正是借助了 Kubernetes 能够对 API 对象进行在线更新的能力，这也正是 **Kubernetes“声明式 API”的独特之处**：

 - 首先，所谓“声明式”，指的就是我只需要提交一个定义好的 API 对象来“声明”，我所期望的状态是什么样子。
 - 其次，“声明式 API”允许有多个 API 写端，以 `PATCH` 的方式对 API 对象进行修改，而无需关心本地原始 YAML文件的内容。
 - 最后，也是最重要的，有了上述两个能力，Kubernetes 项目才可以基于对 API对象的增、删、改、查，在完全无需外界干预的情况下，完成对“实际状态”和“期望状态”的调谐（Reconcile）过程。


所以说，声明式 API，才是 Kubernetes 项目编排能力“赖以生存”的核心所在，希望你能够认真理解。此外，不难看到，无论是对 `sidecar` 容器的巧妙设计，还是对 `Initializer` 的合理利用，Istio 项目的设计与实现，其实都依托于 Kubernetes 的声明式 API 和它所提供的各种编排能力。可以说，**Istio 是在 Kubernetes 项目使用上的一位“集大成者”。**  要知道，一个 Istio 项目部署完成后，会在 Kubernetes 里创建大约 43 个 API 对象。

所以，Kubernetes 社区也看得很明白：Istio 项目有多火热，就说明 Kubernetes 这套“声明式 API”有多成功。这，既是 Google Cloud 喜闻乐见的事情，也是 Istio 项目一推出就被 Google 公司和整个技术圈儿热捧的重要原因。而在使用 Initializer 的流程中，**最核心的步骤，莫过于 Initializer“自定义控制器”的编写过程。它遵循的，正是标准的“Kubernetes 编程范式”**，即：**如何使用控制器模式，同 Kubernetes 里 API 对象的“增、删、改、查”进行协作，进而完成用户业务逻辑的编写过程**。


参考：
- [https://time.geekbang.org/column/article/41876?utm_source=CCyuyue&utm_medium=0725](https://time.geekbang.org/column/article/41876?utm_source=CCyuyue&utm_medium=0725)

