![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/35c97ce38449be57fe8052f91dc35da2.png#pic_center)


---

## 1. 前言

本页介绍如何设置在命名空间中运行的容器使用的内存的最小值和最大值。 你可以在 LimitRange 对象中指定最小和最大内存值。如果 Pod 不满足 LimitRange 施加的约束，则无法在命名空间中创建它

## 2. LimitRange 配置最大最小限制值
创建命名空间 

```bash
kubectl create namespace constraints-mem-example
```
创建 LimitRange 和 Pod

```bash
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-min-max-demo-lr
spec:
  limits:
  - max:
      memory: 1Gi
    min:
      memory: 500Mi
    type: Container
```

```bash
kubectl apply -f https://k8s.io/examples/admin/resource/memory-constraints.yaml --namespace=constraints-mem-example
```
查看 LimitRange 的详情：

```bash
kubectl get limitrange mem-min-max-demo-lr --namespace=constraints-mem-example --output=yaml
```

输出显示预期的最小和最大内存约束。 但请注意，即使你没有在 LimitRange 的配置文件中指定默认值，也会自动创建它们。

```bash
  limits:
  - default:
      memory: 1Gi
    defaultRequest:
      memory: 1Gi
    max:
      memory: 1Gi
    min:
      memory: 500Mi
    type: Container
```
## 3. 创建满足声明的最大值与最小值

```bash
apiVersion: v1
kind: Pod
metadata:
  name: constraints-mem-demo
spec:
  containers:
  - name: constraints-mem-demo-ctr
    image: nginx
    resources:
      limits:
        memory: "800Mi"
      requests:
        memory: "600Mi"
```

```bash
kubectl apply -f https://k8s.io/examples/admin/resource/memory-constraints-pod.yaml --namespace=constraints-mem-example
```

确认下 Pod 中的容器在运行：

```bash
kubectl get pod constraints-mem-demo --namespace=constraints-mem-example
```

查看 Pod 详情：

```bash
kubectl get pod constraints-mem-demo --output=yaml --namespace=constraints-mem-example
```

输出结果显示容器的内存请求为600 MiB，内存限制为800 MiB。这些满足了 LimitRange 设定的限制范围。

```bash
resources:
  limits:
     memory: 800Mi
  requests:
    memory: 600Mi
```
## 4. 尝试创建一个超过最大内存限制的 Pod

```bash
apiVersion: v1
kind: Pod
metadata:
  name: constraints-mem-demo-2
spec:
  containers:
  - name: constraints-mem-demo-2-ctr
    image: nginx
    resources:
      limits:
        memory: "1.5Gi"
      requests:
        memory: "800Mi"
```

```bash
kubectl apply -f https://k8s.io/examples/admin/resource/memory-constraints-pod-2.yaml --namespace=constraints-mem-example
```
输出结果显示 Pod 没有创建成功，因为容器声明的内存限制太大了：

```bash
Error from server (Forbidden): error when creating "examples/admin/resource/memory-constraints-pod-2.yaml":
pods "constraints-mem-demo-2" is forbidden: maximum memory usage per Container is 1Gi, but limit is 1536Mi.
```
## 5. 尝试创建一个不满足最小内存请求的 Pod

```bash
apiVersion: v1
kind: Pod
metadata:
  name: constraints-mem-demo-3
spec:
  containers:
  - name: constraints-mem-demo-3-ctr
    image: nginx
    resources:
      limits:
        memory: "800Mi"
      requests:
        memory: "100Mi"
```


```bash
kubectl apply -f https://k8s.io/examples/admin/resource/memory-constraints-pod-3.yaml --namespace=constraints-mem-example
```
输出结果显示 Pod 没有创建成功，因为容器声明的内存请求太小了：

```bash
Error from server (Forbidden): error when creating "examples/admin/resource/memory-constraints-pod-3.yaml":
pods "constraints-mem-demo-3" is forbidden: minimum memory usage per Container is 500Mi, but request is 100Mi.
```
## 6. 创建一个没有声明内存请求和限制的 Pod

```bash
apiVersion: v1
kind: Pod
metadata:
  name: constraints-mem-demo-4
spec:
  containers:
  - name: constraints-mem-demo-4-ctr
    image: nginx
```

```bash
kubectl apply -f https://k8s.io/examples/admin/resource/memory-constraints-pod-4.yaml --namespace=constraints-mem-example
```
查看 Pod 详情：

```bash
kubectl get pod constraints-mem-demo-4 --namespace=constraints-mem-example --output=yaml
```

输出结果显示 Pod 的内存请求为1 GiB，内存限制为1 GiB。容器怎样获得哪些数值呢？

```bash
resources:
  limits:
    memory: 1Gi
  requests:
    memory: 1Gi
```

因为你的容器没有声明自己的内存请求和限制，它从 LimitRange 那里获得了 默认的内存请求和限制。


相关阅读：
- [k8s【资源管理】1--ResourceQuota为命名空间配置内存和CPU配额](https://ghostwritten.blog.csdn.net/article/details/108813649)
 - [k8s【资源管理】2--LimitRange为命名空间配置默认的内存请求和限制](https://ghostwritten.blog.csdn.net/article/details/112509245)
 - [k8s【资源管理】3--LimitRange为命名空间配置 CPU 最小和最大约束](https://ghostwritten.blog.csdn.net/article/details/112526677)
- [k8s【资源管理】4--LimitRange为配置命名空间内存最小和最大约束](https://ghostwritten.blog.csdn.net/article/details/112524133)
- [k8s【资源管理】5--配置 Pod 的服务质量（QoS）](https://blog.csdn.net/xixihahalelehehe/article/details/112537920)
