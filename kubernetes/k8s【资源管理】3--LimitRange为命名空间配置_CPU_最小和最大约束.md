![在这里插入图片描述](https://img-blog.csdnimg.cn/20210112150414532.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)

---

## 1. LimitRange配置 CPU 最小和最大约束
创建命名空间

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
## 2. 创建满足配置 CPU 最小和最大约束的pod

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
## 3. 尝试创建一个超过最大 CPU 限制的 Pod

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
## 4. 尝试创建一个不满足最小 CPU 请求的 Pod 

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
## 5. 创建一个没有声明 CPU 请求和 CPU 限制的 Pod 

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

因为你的 Container 没有声明自己的 CPU 请求和限制，LimitRange 给它指定了 默认的 CPU 请求和限制


相关阅读：
- [k8s【资源管理】1--ResourceQuota为命名空间配置内存和CPU配额](https://ghostwritten.blog.csdn.net/article/details/108813649)
 - [k8s【资源管理】2--LimitRange为命名空间配置默认的内存请求和限制](https://ghostwritten.blog.csdn.net/article/details/112509245)
 - [k8s【资源管理】3--LimitRange为命名空间配置 CPU 最小和最大约束](https://ghostwritten.blog.csdn.net/article/details/112526677)
- [k8s【资源管理】4--LimitRange为配置命名空间内存最小和最大约束](https://ghostwritten.blog.csdn.net/article/details/112524133)
- [k8s【资源管理】5--配置 Pod 的服务质量（QoS）](https://blog.csdn.net/xixihahalelehehe/article/details/112537920)
