#  kuberenetes pod Liveness, Readiness and Startup Probes
tags: Pod,探针,健康检测



[
![在这里插入图片描述](https://img-blog.csdnimg.cn/3a330d60c83345aba9bb379affd2a492.png)](https://www.rottentomatoes.com/m/ex_machina)




## 1. Pod 状态
Pod 的生命周期是 PodStatus 对象，其中包含一个phase字段。
| 值         | 描述                                                                                |
|-----------|-----------------------------------------------------------------------------------|
| Pending   | 该Pod已被Kubernetes系统接受，但是尚未创建一个或多个Container映像。这包括计划之前的时间以及通过网络下载图像所花费的时间，这可能需要一段时间。 |
| Running   | Pod已绑定到节点，并且所有容器都已创建。至少一个容器仍在运行，或者正在启动或重新启动。                                      |
| Succeeded | Pod中的所有容器已成功终止，并且不会重新启动。                                                          |
| Failed    | Pod中的所有容器均已终止，并且至少一个容器因故障而终止。也就是说，容器要么以非零状态退出，要么被系统终止。                            |
| Unknown   | 由于某些原因，通常由于与Pod主机通信时出错而无法获得Pod的状态。                                                |

我们最常接触到的 Pod 状态是 `Pending` 和 `Running`。Pending 代表 Pod 正在创建中，而 Running 状态则要求至少有一个容器正在启动中或者正在运行。而对于 Pod 中的容器，也有下面 3 种状态。

而对于 Pod 中的容器，也有下面 3 种状态：

 - `Waiting`：容器等待中，例如正在拉取镜像。
 - `Running`：容器运行中，PID=1 的业务进程正在启动或运行。
 - `Terminated`：容器终止中，例如正在删除 Pod。

是不是感觉有点混乱？没关系，现在我们只需要关注 Pod 和容器同时处于 Running 的状态，其他的暂时忽略就可以。当 Pod 处于 Running 状态时，代表至少有一个容器处于启动或运行状态。所以，我们只需要关注容器的 Running 状态就可以间接决定 Pod 的状态。而当容器状态为 Running 时，代表 PID=1 的业务进程正在启动或运行。

不过要注意的是，当业务进程还处于启动过程时，**Pod 和容器都处于 Running 状态，但由于业务并没有完成启动，所以还不具备接收外部流量的条件。这时候，光有一个 Running 状态是不够的，我们还需要一个能够描述 Pod 是否已经就绪并准备好接收外部请求的标识，它就是 Pod 的 Ready 字段**。

## 2. Ready 状态
在 K8s 中，Pod 的 Ready 字段用来标识是否已经就绪，它是由 kubelet 直接管理的， Ready 状态被记录在了 Pod Manifest 的 `status.conditions` 字段下。

![在这里插入图片描述](https://img-blog.csdnimg.cn/ce2556b4cc1247f4ad726366645b2a45.png)
你也可以通过 `kubectl get pods` 来查看 Pod 是否处于 Ready 状态。

```bash
$ kubectl get pods -n example
NAME                        READY   STATUS    RESTARTS        AGE
backend-5969f76d6c-jf9lq    0/1     Pending   0               102m
backend-86d76d8764-cl2pf    1/1     Running   0               109m
backend-86d76d8764-pgvhd    1/1     Running   0               109m
frontend-fc597b5d9-qcbfb    1/1     Running   2 (5h32m ago)   19h
postgres-7745b57d5d-5lbzz   1/1     Running   3 (5h32m ago)   15d
```
在返回结果的 Ready 字段中，1/1 表示运行中的容器数量和总容器数量，当这两者数量相等时，代表 Pod 处于 Ready 状态，说明 Pod 已经就绪并准备好了接收外部流量。

## 3. 为什么需要健康检查
在生产环境，我们为某个业务配置了 CPU 限制，业务运行一段时间后，由于业务应用没有做好垃圾回收甚至产生死锁的情况，导致它的 CPU 占用率一直处在限制值附近。此时，虽然 Pod 的业务进程仍在运行，但实际上它已经无法得到更多的 CPU 时间片了。我的问题是，这时候它能正常处理业务请求吗？毫无疑问，它处理业务会非常缓慢，甚至会完全不可用。

除了因为资源不足导致业务不可用以外，还有一类典型的场景：在进行水平扩容时，由于新创建的 Pod 的业务进程需要的启动时间较长（例如对于大型的 Java 应用，往往需要数分钟的启动时间），如果在业务启动时就将 Pod 标记为 Ready 状态并接收外部请求流量，将会有部分请求得到错误的返回，如下图所示。

![在这里插入图片描述](https://img-blog.csdnimg.cn/a3178ef236904abba40d87658f1224fd.png)
所以，这两种情况都引出了一个非常有意思的场景：因为业务进程处于运行状态，Pod 和容器也处于 Running 状态，所以 K8s 会认为当前 Pod 是就绪的。但实际上，业务可能正处于“** 启动中”或者出现“**资源不足”的情况，暂时无法对外提供服务。

这两个例子告诉我们，在一般情况下，Pod 就绪（Ready）不等于业务健康。那么，如何才能让 K8s 感知到业务真实的健康状态呢？这时候我们就需要用到 K8s 探针。


## 4. Pod 生命周期条件
Pod具有PodStatus，该状态具有`PodConditions`数组 ，该Pod已通过或未通过。PodCondition数组的每个元素都有六个可能的字段：

 - 该`lastProbeTime`字段提供上次探测Pod条件的时间戳。
 - 该`lastTransitionTime`字段提供Pod上一次从一种状态转换为另一种状态的时间戳。
 - 该`message`字段是人类可读的消息，指示有关过渡的详细信息。
 - 该`reason`字段是条件最后一次转换的唯一单词CamelCase原因。
 - 该`status`字段是一个字符串，可能的值为“ True”，“ False”和“ Unknown”。
 - 该`type`字段是具有以下可能值的字符串：

```bash
PodScheduled：Pod已调度到一个节点；
Ready：Pod能够处理请求，应将其添加到所有匹配服务的负载平衡池中；
Initialized：所有初始化容器 已成功启动；
Unschedulable：例如，由于缺乏资源或其他限制，调度程序无法立即调度Pod；
ContainersReady：Pod中的所有容器已准备就绪。
```
## 5. 容器探针
K8s 的健康检查类型一共有三种，分别是：

 - `livenessProbe`：指示容器是否正在运行。如果活动探针失败，则kubelet将杀死Container，并且Container将接受其重新启动策略。如果容器未提供活动性探针，则默认状态为Success。
 - `readinessProbe`：指示容器是否准备好服务请求。如果就绪探针失败，则端点控制器将从与Pod匹配的所有服务的端点中删除Pod的IP地址。初始延迟之前的默认就绪状态为Failure。如果容器未提供就绪探测器，则默认状态为Success。
 - `StartupProbe`：

有三种类型的处理程序：

 - `ExecAction`：在Container中执行指定的命令。如果命令以状态代码0退出，则认为诊断成功。
 - `TCPSocketAction`：对指定端口上的容器的IP地址执行TCP检查。如果端口打开，则认为诊断成功。
 - `HTTPGetAction`：对指定端口和路径上的容器的IP地址执行HTTP
   Get请求。如果响应的状态码大于或等于200且小于400，则认为诊断成功。

每个探针具有以下三个结果之一：

 - 成功：容器通过了诊断。
 - 失败：容器无法通过诊断。
 - 未知：诊断失败，因此不应采取任何措施。


### 5.1 ReadinessProbe
Readiness 又称为就绪探针，它用来确定 Pod 是否为就绪（Ready）状态，以及是否能够接收外部流量。例如，当某一些 Pod 出现短暂的延迟或不可用时，我们希望它不接收外部流量，这时候 Readiness 就非常有用。Readiness 探针能够识别业务的健康状态，并通过健康状态来控制 Pod 是否接受外部请求流量。

我们以示例应用 Backend Deployment 服务为例，来看看怎么为工作负载配置 Readiness 探针。下面是 Backend Deployment Manifest 的部分内容。

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  ......
spec:
  ......
    spec:
      containers:
      - name: flask-backend
        image: lyzhang1999/backend:latest
        ......
        readinessProbe: 
          httpGet:
            path: /healthy
            port: 5000
            scheme: HTTP
          initialDelaySeconds: 10
          failureThreshold: 5
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
```
这里要重点关注 readinessProbe 字段，它为 K8s Readiness 探针提供了必要的信息。

我们先来看第 14 行的 `httpGet` 下的几个字段，path 字段的含义是“通过请求 healthy 接口来判断”，Port 字段指定了 Python 后端服务的监听端口 5000，`scheme` 字段代表协议。探针请求的完整路径为：`http://PodIP:5000/healthy`，相当于以 Get 的方式主动访问业务接口，当接口返回 `200-399` 状态码时，则视为本次探针请求成功。

 - `initialDelaySeconds` 的含义是在容器启动之后，延迟 10 秒钟再进行第一次探针检查。
 - `failureThreshold` 的含义是，如果连续 5 次探针失败则代表 Readiness 探针失败，Pod 状态为 NotReady，此时 Pod 不会接收外部请求。
 - `periodSeconds` 的含义是探针每 10 秒钟轮询检测 1 次。
 - `successThreshold` 的含义是只要探针成功 1 次就代表探针成功了，Pod 状态为 Ready 表示可以接收外部请求。
 - `timeoutSeconds` 代表探针的超时时间为 1 秒。

![在这里插入图片描述](https://img-blog.csdnimg.cn/fdb4c3cc90314473bc2d3325e38205ab.png)


综合这些配置信息，我们可以得出几个重要的数字。在 Pod 启动之后，如果在 60 秒内（`initialDelaySeconds + failureThreshold * periodSeconds`） 不能通过健康检查， Pod 将处于非就绪状态。

当 Pod 处于正常的运行状态时，如果业务突然产生故障，健康检查会在 50 秒内（`failureThreshold * periodSeconds`） 识别出业务的不健康状态；当 Pod 业务恢复健康时，健康检查会在 10 秒内（`successThreshold * periodSeconds`） 识别出业务已恢复。

Readiness 探针的一个非常重要的作用是，它负责找出业务状态不健康的 Pod，并且将它从 Service EndPoints 列表移除，使它无法接收到外部请求。如果 Pod 的 Readiness 探针一直无法通过，那么 Pod 将一直无法接收外部请求，如下图所示。
![在这里插入图片描述](https://img-blog.csdnimg.cn/35e6ab90cf5c42e4ae80c103d546bb8e.png)
请注意，当 Readiness 探针向业务应用发出请求时，如果在超时时间内没有收到回复，或者收到的回复的状态码大于 400，那么认为本次探测失败。试想一下，如果这时候业务已经无法自行恢复了，比如产生了死锁，此时 Readiness 探针也已经感知到业务不可用了。



![在这里插入图片描述](https://img-blog.csdnimg.cn/951c78c06cfd47e6aea90edd919a8a7e.png)
那我们能否更进一步，让 K8s 帮我们自动重启 Pod 来恢复业务呢？这就是我接下来要介绍的 Liveness 探针。

### 5.2 LivenessProbe
Liveness 又称为存活探针，相比 Readiness 探针，它还能够在检测到 Pod 处于不健康状态时自动将 Pod 重启。在示例应用 Backend Deployment 中，我们也定义了 Liveness 探针。

```bash
spec:
  ......
    spec:
      containers:
      - name: flask-backend
        image: lyzhang1999/backend:latest
        ......
        livenessProbe: 
          httpGet:
            path: /healthy
            port: 5000
            scheme: HTTP
          failureThreshold: 5
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
```
Liveness 探针和 Readiness 探针非常类似，Liveness 探针是通过 livenessProbe 字段来配置的，livenessProbe 字段下面的每个字段的作用和 Readiness 探针都几乎一致，这里就不再赘述了。当 Liveness 探针失败时，它将自动重启业务不健康的 Pod，如下图所示。

![在这里插入图片描述](https://img-blog.csdnimg.cn/4464974b88a140239aecbf9ee6edcead.png)
需要注意的是，Liveness 和 Readiness 探针是独立并行的，它们之间并没有相互等待关系。

### 5.3 StartupProbe 
相比较 Liveness 和 Readiness 探针在运行阶段的探测特性，StartupProbe 是一种专门针对业务在启动阶段设计的探针。还记得我在之前提到的一个场景吗？对于一些大型的 Java 应用来说，往往需要数分钟时间才能够完成启动。这意味着，对于启动非常慢的大型应用，如果我们按照上面的例子来配置 Readiness 和 Liveness 探针，Pod 将因为探针失败而永远无法启动。

那么，怎么解决这个问题呢？你可能首先会想到，是不是调整探针第一次探测的延迟时间就可以了？比如将 `initialDelaySeconds` 字段的值从 10 秒钟修改到 120 秒。这当然是可以的，但如果启动时间不是 120 秒，或者你不太确定呢？那似乎可以在修改了 initialDelaySeconds 的基础上再将失败次数 `failureThreshold` 或者探测间隔 periodSeconds 加大。

这种做法虽然能解决问题，但缺点也是明显的，为了兼容应用启动慢的问题，我们主动降低了 K8s 检测 Pod 健康状态的频率，这会延迟 K8s 感知故障的速度。

其实，Readiness 和 Liveness 主要用来检查的是处于运行过程中业务，因为启动过慢的问题而去调整 Readiness 和 Liveness 参数并不是最佳实践。这类问题我们可以通过 StartupProbe 来解决。

StartupProbe 探针特别适用于业务应用启动慢的场景。当 Pod 启动时，如果配置了 StartupProbe，那么 Readiness 和 Liveness 探针都将被临时禁用，直到 StartupProbe 探针返回成功才会启用 Readiness 和 Liveness 探针，这也就避免了 Readiness 和 Liveness 在应用启动阶段造成的干扰。也就是说，我们只要为业务配置合理的 StartupProbe 探针，就可以解决应用启动慢导致其他探针认为 Pod 不健康的问题。

StartupProbe 探针的配置方法和 Readiness、Liveness 探针也非常类似，下面是示例应用 Backend Deployment Manifest 的部分内容。

```bash
spec:
  ......
    spec:
      containers:
      - name: flask-backend
        image: lyzhang1999/backend:latest
        ......
        startupProbe: 
          httpGet:
            path: /healthy
            port: 5000
            scheme: HTTP
          initialDelaySeconds: 10
          failureThreshold: 5
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
```

StartupProbe 是通过 startupProbe 字段来配置的。在这个例子中，Pod 启动之后 10 秒会开始第一次探测，如果至少有 1 次探测成功，那么 StartupProbe 探针就成功，接下来才会继续启动 Readiness 和 Liveness 探针。在我们刚才提到的 Java 应用的例子中，我们可以为 StartupProbe 配置足够大的 `initialDelaySeconds` 来让业务有足够的时间完成启动过程。

如果一个工作负载内同时定义了这三种探针，情况会变得有些复杂，你可以结合下面这张图来理解。
![在这里插入图片描述](https://img-blog.csdnimg.cn/3fa7282808664c04a56a38b621e3796d.png)
Pod 启动时，三种探针的执行顺序主要可以分成三个阶段。

 - 第一阶段：Pod 已启动，容器已启动，业务进程正在启动中，此时 StartupProbe 开始工作，由于 StartupProbe
   还未成功，当前 Pod 的容器处于 Not Ready(0/1) 的状态。
   第二阶段：随着时间推移，StartupProbe 探针成功，Readine 和 Liveness 探针开始并行工作，此时由于 Readine 探针还未成功，当前 Pod 的容器仍然处于 Not Ready(0/1) 的状态。
   第三阶段：随着 Readine 和 Liveness 的探测成功，当前 Pod 的容器转为 Ready(1/1) 的状态，Service EndPoints 将 Pod 加入到列表当中，Pod 开始接收外部请求。

![在这里插入图片描述](https://img-blog.csdnimg.cn/3c79af0246494c4193b99e7ed8af4f40.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/57d1d56a726142e9b6a84d6d51f21546.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/001c888b92884085a823679df06a29a4.png)

## 6. 什么时候应该使用活动探针与就绪探针？
如果您的Container中的进程在遇到问题或变得不正常时能够自行崩溃，则不一定需要进行活动调查；kubelet将根据Pod的自动执行正确的操作`restartPolicy`。

如果您希望在探测失败时杀死并重启容器，请指定活动探测，并指定`restartPolicy Always`或`OnFailure`。

如果您仅想在探测成功后才开始向Pod发送流量，请指定就绪探测器。在这种情况下，就绪探针可能与活动探针相同，但是规范中存在就绪探针意味着Pod将在不接收任何流量的情况下启动，并且仅在探针开始成功之后才开始接收流量。如果您的容器需要在启动过程中加载大型数据，配置文件或迁移，请指定`StartupProbe`探针。

如果希望您的Container能够自行进行维护，则可以指定一个就绪探针，以检查特定于与活跃探针不同的就绪端点。

> 请注意，如果您只想在删除Pod时便能够清空请求，则不一定需要准备就绪探测器；删除后，无论是否准备就绪探针，Pod都会自动将自己置于未就绪状态。等待Pod中的Container停止时，Pod仍处于未就绪状态。

## 7. 配置技巧
### 7.1 使用命名端口

```bash
ports:
- name: liveness-port
  containerPort: 8080
  hostPort: 8080

livenessProbe:
  httpGet:
    path: /healthz
    port: liveness-port
```

### 7.2 使用启动探测器保护慢启动容器

```bash
ports:
- name: liveness-port
  containerPort: 8080
  hostPort: 8080

livenessProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 1
  periodSeconds: 10

startupProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 30
  periodSeconds: 10
```
幸亏有启动探测，应用程序将会有最多 5 分钟(30 * 10 = 300s) 的时间来完成它的启动。 一旦启动探测成功一次，存活探测任务就会接管对容器的探测，对容器死锁可以快速响应。 如果启动探测一直没有成功，容器会在 300 秒后被杀死，并且根据 restartPolicy 来设置 Pod 状态。

![在这里插入图片描述](https://img-blog.csdnimg.cn/7465ae7606a3468fb3c8c18e4ebfe5e6.gif#pic_center)

## 8. 探测方法
### 8.1 http 探测
```bash
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - args:
    - /server
    image: k8s.gcr.io/liveness
    livenessProbe:
      httpGet:
        # when "host" is not defined, "PodIP" will be used
        # host: my-host
        # when "scheme" is not defined, "HTTP" scheme will be used. Only "HTTP" and "HTTPS" are allowed
        # scheme: HTTPS
        path: /healthz
        port: 8080
        httpHeaders:
        - name: X-Custom-Header
          value: Awesome
      initialDelaySeconds: 15
      periodSeconds: 3
      timeoutSeconds: 1
    name: liveness
```
`periodSeconds` 字段指定了 kubelet 每隔 3 秒执行一次存活探测。 `initialDelaySeconds` 字段告诉 kubelet 在执行第一次探测前应该等待 3 秒。 kubelet 会向容器内运行的服务（服务会监听 8080 端口）发送一个 HTTP GET 请求来执行探测。 如果服务器上 /healthz 路径下的处理程序返回成功代码，则 kubelet 认为容器是健康存活的。 如果处理程序返回失败代码，则 kubelet 会杀死这个容器并且重新启动它。

任何大于或等于 200 并且小于 400 的返回代码标示成功，其它返回代码都标示失败。

### 8.2 cmd 探测

```bash
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```
 `periodSeconds` 字段指定了 kubelet 应该每 5 秒执行一次存活探测。 `initialDelaySeconds` 字段告诉 kubelet 在执行第一次探测前应该等待 5 秒。 kubelet 在容器内执行命令 cat `/tmp/healthy` 来进行探测。 如果命令执行成功并且返回值为 0，kubelet 就会认为这个容器是健康存活的。 如果这个命令返回非 0 值，kubelet 会杀死这个容器并重新启动它。
###  8.3 TCP 探测

```bash
apiVersion: v1
kind: Pod
metadata:
  name: goproxy
  labels:
    app: goproxy
spec:
  containers:
  - name: goproxy
    image: k8s.gcr.io/goproxy:0.1
    ports:
    - containerPort: 8080
    readinessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```
下面这个例子同时使用就绪和存活探测器。kubelet 会在容器启动 5 秒后发送第一个就绪探测。 这会尝试连接 goproxy 容器的 8080 端口。 如果探测成功，这个 Pod 会被标记为就绪状态，kubelet 将继续每隔 10 秒运行一次检测。

除了就绪探测，这个配置包括了一个存活探测。 kubelet 会在容器启动 15 秒后进行第一次存活探测。 就像就绪探测一样，会尝试连接 goproxy 容器的 8080 端口。 如果存活探测失败，这个容器会被重新启动。


参考：

 - [配置存活、就绪和启动探测器](https://kubernetes.io/zh/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
 - [Kubernetes Startup Probes - Examples & Common Pitfalls](https://loft.sh/blog/kubernetes-startup-probes-examples-common-pitfalls/)




