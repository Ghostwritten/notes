
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/03e24f75aa3fbd9fcf0037fcd2d61c72.png)



##  1. pod 状态

创建一个 `pod-status.yaml`

```bash

apiVersion: v1
kind: Pod
metadata:
  name: running
  labels:
    app: nginx
spec:
  containers:
    - name: web
      image: nginx
      ports:
        - name: web
          containerPort: 80
          protocol: TCP

---
apiVersion: v1
kind: Pod
metadata:
  name: backoff
spec:
  containers:
    - name: web
      image: nginx:not-exist

---
apiVersion: v1
kind: Pod
metadata:
  name: error
spec:
  containers:
    - name: web
      image: nginx
      command: ["sleep", "a"]
```
然后，使用 `kubectl apply` 命令将它应用到集群内。

```bash
$ kubectl apply -f pod-status.yaml
pod/running created
pod/backoff created
pod/error created
```
接下来，使用 `kubectl get pods` 查看刚才创建的 3 个 Pod。

```bash
NAME                             READY   STATUS             RESTARTS        AGE
backoff                          0/1     ImagePullBackOff   0               50s
error                            0/1     CrashLoopBackOff   1               4s
running                          1/1     Running            0               50s
```
在这个例子中，我们一共创建了 3 个 Pod，它们的状态包含 `ImagePullBackOff`、`CrashLoopBackOff` 和 `Running`。

### 1.1 容器启动错误类型
- ErrImagePull
- ImageInspectError
- ErrImageNeverPull
- RegistryUnavailable
- InvalidImageName

### 1.2  ImagePullBackOff 错误
原因导致：

- 镜像名称或者版本错误，在我们刚才创建的 `backoff` Pod 中，我们指定的镜像为 `nginx:not-exist`，但是实际上镜像版本 `not-exist` 并不存在，自然也就会抛出错误。
- 指定了私有镜像，但又没有提供拉取凭据。

我们可以使用 `kubectl describe` 命令来查看错误的详情。

```bash
$ kubectl describe pod backoff
Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  10m                  default-scheduler  Successfully assigned default/backoff to kind-control-plane
  Normal   Pulling    8m43s (x4 over 10m)  kubelet            Pulling image "nginx:not-exist"
  Warning  Failed     8m40s (x4 over 10m)  kubelet            Failed to pull image "nginx:not-exist": rpc error: code = NotFound desc = failed to pull and unpack image "docker.io/library/nginx:not-exist": failed to resolve reference "docker.io/library/nginx:not-exist": docker.io/library/nginx:not-exist: not found
  Warning  Failed     8m40s (x4 over 10m)  kubelet            Error: ErrImagePull
  Warning  Failed     8m11s (x6 over 10m)  kubelet            Error: ImagePullBackOff
  Normal   BackOff    4s (x42 over 10m)    kubelet            Back-off pulling image "nginx:not-exist"
```
从返回结果里 `Event` 事件中的第三行我们可以发现，集群抛出了 `nginx:not-exist: not found` 的异常，这样我们也就定位到了具体的错误。

### 1.3 CrashLoopBackOff
`CrashLoopBackOff` 是一种典型的容器运行阶段的错误。除此之外，你可能还会看到类似的 `RunContainerError` 错误。出现这个错误的原因主要有下面两个。

- 容器内的应用程序在启动时出现了错误，例如配置读取失败导致无法启动。
- 配置出错，例如配置了错误的容器启动命令。

在上面创建的 error Pod 的例子中，我故意错误地配置了容器的启动命令，这样我们也就看到了 `CrashLoopBackOff` 异常。对于运行阶段的错误，大部分错误都来源于业务本身的启动阶段，所以，我们只需要查看 Pod 的日志一般就能够找到问题所在。比如，我们尝试来查看 error Pod 的日志。

```bash
$ kubectl logs error
sleep: invalid time interval 'a'
Try 'sleep --help' for more information.
```
从返回的日志来看，sleep 命令抛出了一个异常，也就是参数错误。在生产环境下，我们一般会用 Deoloyment 工作负载来管理 Pod，在 Pod 出现运行阶段异常的情况下，Pod 名称会随着重新启动而出现变化，这时候你可以在查看日志时增加 `--previous` 参数，以此查看之前的 Pod 日志。

```bash
$ kubectl logs pod-name --previous
```

### 1.4 Pending
有时候，你可能不会看到启动和运行的错误状态，但查看状态时，会看到 Pod 处于 `Pending` 状态。你可以尝试将下面的内容保存为 `pending-pod.yaml` 文件，并通过 `kubectl apppy -f` 将这个例子部署到集群内。

```bash
apiVersion: v1
kind: Pod
metadata:
  name: pending
spec:
  containers:
    - name: web
      image: nginx
      resources:
        requests:
          cpu: 32
          memory: 64Gi
```
接下来，尝试查看 Pod 的状态。

```bash
$ kubectl get pods
NAME                             READY   STATUS             RESTARTS         AGE
pending                          0/1     Pending            0                15s
```
从返回结果我们会发现，Pod 没有抛出任何异常，但它的状态处于 `Pending`，同时 `READY 0/1` 表示 Pod 没有准备好接收外部流量。出现 Pending 状态主要的原因可能有下面三种。

 1. 集群资源不足以调度 Pod。
 2. Pod 正在等待 PVC 持久化存储卷。
 3. Pod 资源用量超过了命名空间的资源配额。

在上面的例子中，我们为 Pod 配置了 32 核 64G 的资源请求配额，这显然超出了集群资源。此时 Pod 会处于 Pending 状态，并且 Kubernetes 会一直尝试调度，一旦加入了新的节点并满足资源要求，Pod 就会被重新启动。Pending 状态其实也算是容器启动异常的一种情况，但它并不能算是错误，只是暂时无法调度。要查明 Pending 状态的具体原因，你可以参考寻找容器启动错误的方法，通过 kubectl describe 命令来查看。

```bash
$ kubectl describe pod pending
Events:
  Type     Reason            Age    From               Message
  ----     ------            ----   ----               -------
  Warning  FailedScheduling  11m    default-scheduler  0/1 nodes are available: 1 Insufficient cpu, 1 Insufficient memory. preemption: 0/1 nodes are available: 1 No preemption victims found for incoming pod.
  Warning  FailedScheduling  6m45s  default-scheduler  0/1 nodes are available: 1 Insufficient cpu, 1 Insufficient memory. preemption: 0/1 nodes are available: 1 No preemption victims found for incoming pod.
```
从返回结果的 Event 事件中我们可以得出结论，Pending 出现的原因是没有符合 CPU 资源条件的 Node 节点，Kubernetes 尝试调度了两次，异常情况相同。

## 2. Service 连接状态
有时候，即便是 Pod 处于运行且处于就绪状态，我们也无法从外部请求到业务服务。这时候就要关注 Service 的连接状态了。Service 是 Kubernetes 的核心组件，正常情况下它都是可用的。在生产环境下，流量的流向一般是从 Ingress 到 Service 再到 Pod。所以，当无法在外部访问到 Pod 的业务服务时，我们可以先从最内层也就是 Pod 开始检查，最简单的方式就是直连 Pod 并发起请求，查看 Pod 是否能够正常工作。

要在本地访问 Pod，我们可以使用 `kubectl port-forward` 进行端口转发，以我们刚才创建的 Nginx Pod 为例。


```bash
$ kubectl port-forward pod/running 8081:80
```
如果在本地访问 8081 端口请求能够成功，则代表 Pod 和业务层面是正常的。接下来，我们进一步检查 Service 的连接状态。同样地，最简单的方式也是通过端口转发直连 Service 发起请求。

```bash
$ kubectl port-forward service/<service-name> local_port:service_pod
```
如果请求 Service 能够正确返回内容，说明 Service 这一层也是正常的。如果无法返回内容，这时候通常可能有两个原因。
- Service Selector 选择器没有正确匹配到 Pod。
- Service 的 Port 和 TargetPort 配置错误。

通过修复这两项配置，你应该就能修复 Service 到 Pod 的连接问题了。

## 3. Ingress 连接状态
到这里，如果仍然无法从 Ingress 访问业务服务，那么就需要继续排查 Ingress 了。首先，确认 Ingress 控制器的 Pod 是否处于运行状态。

```bash
$ kubectl get pods -n ingress-nginx
NAME                                        READY   STATUS      RESTARTS        AGE
ingress-nginx-controller-8544b85788-c9m2g   1/1     Running     6 (4h35m ago)   1d
```
在确认 Ingress 控制器并无异常之后，基本上可以确认是 Ingress 策略配置错误导致的故障了。你可以通过 `kubectl describe ingress` 命令来查看 Ingress 策略。

```bash
$ kubectl describe ingress ingress_name
Name:             ingress_name
Namespace:        default
Rules:
  Host        Path  Backends
  ----        ----  --------
              /     running-service:80 (<error: endpoints "running-service" not found>)
```

