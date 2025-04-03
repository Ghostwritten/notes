#  kubernetes 资源管理策略 LimitRange
tags: LimitRange
<!-- catalog: ~LimitRange~ -->




[![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/21b5a59810fee254f7a7778dd0034c60.png#pic_center)](https://www.rottentomatoes.com/m/interstellar_2014)
*《星际穿越》拍摄花絮*

##  1. LimitRange 为 namespace 配置默认的内存请求和限制


###  1.1 LimitRange 限制 namespace 资源
创建 namespace

```bash
kubectl create namespace default-mem-example
```

创建 LimitRange 和 Pod

```bash
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
```
在 default-mem-example  namespace创建限制范围：
```bash
kubectl apply -f https://k8s.io/examples/admin/resource/memory-defaults.yaml --namespace=default-mem-example
```
### 1.2 创建没有声明自己的内存请求和限制值pod
现在，如果在 default-mem-example  namespace创建容器，并且该容器没有声明自己的内存请求和限制值， 它将被指定默认的内存请求 256 MiB 和默认的内存限制 512 MiB。

下面是具有一个容器的 Pod 的配置文件。 容器未指定内存请求和限制。


```bash
apiVersion: v1
kind: Pod
metadata:
  name: default-mem-demo
spec:
  containers:
  - name: default-mem-demo-ctr
    image: nginx
```

```bash
kubectl apply -f https://k8s.io/examples/admin/resource/memory-defaults-pod.yaml --namespace=default-mem-example
```

```bash
kubectl get pod default-mem-demo --output=yaml --namespace=default-mem-example
```

输出内容显示该 Pod 的容器有 256 MiB 的内存请求和 512 MiB 的内存限制。 这些都是 LimitRange 设置的默认值。

```bash
containers:
- image: nginx
  imagePullPolicy: Always
  name: default-mem-demo-ctr
  resources:
    limits:
      memory: 512Mi
    requests:
      memory: 256Mi
```
删除你的 Pod：

```bash
kubectl delete pod default-mem-demo --namespace=default-mem-example
```
### 1.3 声明容器的限制而不声明它的请求会怎么样？

```bash
apiVersion: v1
kind: Pod
metadata:
  name: default-mem-demo-2
spec:
  containers:
  - name: default-mem-demo-2-ctr
    image: nginx
    resources:
      limits:
        memory: "1Gi"
 
```

```bash
kubectl apply -f https://k8s.io/examples/admin/resource/memory-defaults-pod-2.yaml --namespace=default-mem-example
```

查看 Pod 的详情：

```c
kubectl get pod default-mem-demo-2 --output=yaml --namespace=default-mem-example
```

输出结果显示容器的内存请求被设置为它的内存限制相同的值。注意该容器没有被指定默认的内存请求值 256MiB。

```bash
resources:
  limits:
    memory: 1Gi
  requests:
    memory: 1Gi
```
### 1.4 声明容器的内存请求而不声明内存限制会怎么样？

```bash
apiVersion: v1
kind: Pod
metadata:
  name: default-mem-demo-3
spec:
  containers:
  - name: default-mem-demo-3-ctr
    image: nginx
    resources:
      requests:
        memory: "128Mi"
 
```

```bash
kubectl apply -f https://k8s.io/examples/admin/resource/memory-defaults-pod-3.yaml --namespace=default-mem-example
```
查看 Pod 声明：

```bash
kubectl get pod default-mem-demo-3 --output=yaml --namespace=default-mem-example
```

输出结果显示该容器的内存请求被设置为了容器配置文件中声明的数值。 容器的内存限制被设置为 512MiB，即 namespace的默认内存限制。

```bash
resources:
  limits:
    memory: 512Mi
  requests:
    memory: 128Mi
```

## 2. LimitRange 配置 CPU 最小和最大约束
创建 namespace

```bash
kubectl create namespace constraints-cpu-example
```

创建 LimitRange 和 Pod

```bash
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-min-max-demo-lr
spec:
  limits:
  - max:
      cpu: "800m"
    min:
      cpu: "200m"
    type: Container
```

```bash
kubectl apply -f https://k8s.io/examples/admin/resource/cpu-constraints.yaml --namespace=constraints-cpu-example
```

查看 LimitRange 详情：

```bash
kubectl get limitrange cpu-min-max-demo-lr --output=yaml --namespace=constraints-cpu-example
```

输出结果显示 CPU 的最小和最大限制符合预期。但需要注意的是，尽管你在 LimitRange 的配置文件中你没有声明默认值，默认值也会被自动创建。

```bash
limits:
- default:
    cpu: 800m
  defaultRequest:
    cpu: 800m
  max:
    cpu: 800m
  min:
    cpu: 200m
  type: Container
```
### 2.1 创建满足配置 CPU 最小和最大约束的 pod

```bash
apiVersion: v1
kind: Pod
metadata:
  name: constraints-cpu-demo
spec:
  containers:
  - name: constraints-cpu-demo-ctr
    image: nginx
    resources:
      limits:
        cpu: "800m"
      requests:
        cpu: "500m"
```

```bash
kubectl apply -f https://k8s.io/examples/admin/resource/cpu-constraints-pod.yaml --namespace=constraints-cpu-example
kubectl get pod constraints-cpu-demo --namespace=constraints-cpu-example
```

查看 Pod 的详情：

```bash
kubectl get pod constraints-cpu-demo --output=yaml --namespace=constraints-cpu-example
```

输出结果表明容器的 CPU 请求为 500 millicpu，CPU 限制为 800 millicpu。 这些参数满足 LimitRange 规定的限制范围。

```bash
resources:
  limits:
    cpu: 800m
  requests:
    cpu: 500m
```
### 2.2  创建超过最大 CPU 限制的 Pod

```bash
apiVersion: v1
kind: Pod
metadata:
  name: constraints-cpu-demo-2
spec:
  containers:
  - name: constraints-cpu-demo-2-ctr
    image: nginx
    resources:
      limits:
        cpu: "1.5"
      requests:
        cpu: "500m"
```

```bash
kubectl apply -f https://k8s.io/examples/admin/resource/cpu-constraints-pod-2.yaml --namespace=constraints-cpu-example
```

输出结果表明 Pod 没有创建成功，因为容器声明的 CPU 限制太大了：

```bash
Error from server (Forbidden): error when creating "examples/admin/resource/cpu-constraints-pod-2.yaml":
pods "constraints-cpu-demo-2" is forbidden: maximum cpu usage per Container is 800m, but limit is 1500m.
```
### 2.3 创建不满足最小 CPU 请求的 Pod 

```bash
apiVersion: v1
kind: Pod
metadata:
  name: constraints-cpu-demo-3
spec:
  containers:
  - name: constraints-cpu-demo-3-ctr
    image: nginx
    resources:
      limits:
        cpu: "800m"
      requests:
        cpu: "100m"
```

```bash
kubectl apply -f https://k8s.io/examples/admin/resource/cpu-constraints-pod-3.yaml --namespace=constraints-cpu-example
```

输出结果显示 Pod 没有创建成功，因为容器声明的 CPU 请求太小了：

```bash
Error from server (Forbidden): error when creating "examples/admin/resource/cpu-constraints-pod-3.yaml":
pods "constraints-cpu-demo-4" is forbidden: minimum cpu usage per Container is 200m, but request is 100m.
```
### 2.4 创建没有声明 CPU 请求和 CPU 限制的 Pod 

```bash
apiVersion: v1
kind: Pod
metadata:
  name: constraints-cpu-demo-4
spec:
  containers:
  - name: constraints-cpu-demo-4-ctr
    image: vish/stress
```

```bash
kubectl apply -f https://k8s.io/examples/admin/resource/cpu-constraints-pod-4.yaml --namespace=constraints-cpu-example
```

查看 Pod 的详情：

```bash
kubectl get pod constraints-cpu-demo-4 --namespace=constraints-cpu-example --output=yaml
```

输出结果显示 Pod 的容器有个 800 millicpu 的 CPU 请求和 800 millicpu 的 CPU 限制。 容器是怎样得到那些值的呢？

```bash
resources:
  limits:
    cpu: 800m
  requests:
    cpu: 800m
```

因为你的 Container 没有声明自己的 CPU 请求和限制，LimitRange 给它指定了 默认的 CPU 请求和限制。


## 3. LimitRange为配置 namespace 内存最小和最大约束

在 LimitRange 对象中指定最小和最大内存值。如果 Pod 不满足 LimitRange 施加的约束，则无法在 namespace 中创建它.

### 3.1 LimitRange 配置最大最小限制值
创建 namespace 

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
### 3.2 创建满足声明的最大值与最小值

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
### 3.3 尝试创建一个超过最大内存限制的 Pod

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
### 3.4 尝试创建一个不满足最小内存请求的 Pod

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
### 3.5 创建一个没有声明内存请求和限制的 Pod

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

参考：

 - [kubernetes Limit Ranges](https://kubernetes.io/docs/concepts/policy/limit-range/)
 - [kubernetes资源调度之LimitRange](https://www.cnblogs.com/tylerzhou/p/11041963.html)
 - [Restrict resource consumption with limit ranges](https://docs.openshift.com/container-platform/4.8/nodes/clusters/nodes-cluster-limit-ranges.html)
 - [Limit Ranges in Kubernetes](https://www.howtoforge.com/limit-ranges-in-kubernetes/)

