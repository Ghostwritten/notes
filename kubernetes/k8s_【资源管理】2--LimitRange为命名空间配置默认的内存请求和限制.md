![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3a4b075681f11c6dfa649c7c9f19ecab.png#pic_center)

----

## 1. 前言
本文介绍怎样给命名空间配置默认的内存请求和限制。 如果在一个有默认内存限制的命名空间创建容器，该容器没有声明自己的内存限制时， 将会被指定默认内存限制。 Kubernetes 还为某些情况指定了默认的内存请求，本章后面会进行介绍。


##  2. LimitRange限制命名空间资源
创建命名空间

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
在 default-mem-example 命名空间创建限制范围：
```bash
kubectl apply -f https://k8s.io/examples/admin/resource/memory-defaults.yaml --namespace=default-mem-example
```
## 3. 创建没有声明自己的内存请求和限制值pod
现在，如果在 default-mem-example 命名空间创建容器，并且该容器没有声明自己的内存请求和限制值， 它将被指定默认的内存请求 256 MiB 和默认的内存限制 512 MiB。

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
## 4. 声明容器的限制而不声明它的请求会怎么样？

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
## 5. 声明容器的内存请求而不声明内存限制会怎么样？

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

输出结果显示该容器的内存请求被设置为了容器配置文件中声明的数值。 容器的内存限制被设置为 512MiB，即命名空间的默认内存限制。

```bash
resources:
  limits:
    memory: 512Mi
  requests:
    memory: 128Mi
```
相关阅读：
- [k8s【资源管理】1--ResourceQuota为命名空间配置内存和CPU配额](https://ghostwritten.blog.csdn.net/article/details/108813649)
 - [k8s【资源管理】2--LimitRange为命名空间配置默认的内存请求和限制](https://ghostwritten.blog.csdn.net/article/details/112509245)
 - [k8s【资源管理】3--LimitRange为命名空间配置 CPU 最小和最大约束](https://ghostwritten.blog.csdn.net/article/details/112526677)
- [k8s【资源管理】4--LimitRange为配置命名空间内存最小和最大约束](https://ghostwritten.blog.csdn.net/article/details/112524133)
- [k8s【资源管理】5--配置 Pod 的服务质量（QoS）](https://blog.csdn.net/xixihahalelehehe/article/details/112537920)
