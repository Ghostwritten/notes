


---
## 1. configmap更新
### 1.1 修改 ConfigMap


```bash
$ kubectl edit configmap special-config
```

待大概10秒钟时间

> Known Issue：
> 如果使用ConfigMap的subPath挂载为Container的Volume，Kubernetes不会做自动热更新:
> [https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#mounted-configmaps-are-updated-automatically](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#mounted-configmaps-are-updated-automatically)

### 1.2 ConfigMap 更新后滚动更新 Pod
更新 ConfigMap 目前并不会触发相关 Pod 的滚动更新，可以通过修改 pod annotations 的方式强制触发滚动更新。

```bash
$ kubectl patch deployment my-nginx --patch '{"spec": {"template": {"metadata": {"annotations": {"version/config": "20180411" }}}}}'
```

这个例子里我们在 `.spec.template.metadata.annotations` 中添加 `version/config`，每次通过修改 version/config 来触发滚动更新。

更新 ConfigMap 后：

 - 使用该 ConfigMap 挂载的 Env 不会同步更新
 - 使用该 ConfigMap 挂载的 Volume 中的数据需要一段时间（实测大概10秒）才能同步更新

ENV 是在容器启动的时候注入的，启动之后 kubernetes 就不会再改变环境变量的值，且同一个 namespace 中的 pod 的环境变量是不断累加的，[参考 Kubernetes中的服务发现与docker容器间的环境变量传递源码探究](https://jimmysong.io/blog/exploring-kubernetes-env-with-docker/)。为了更新容器中使用 ConfigMap 挂载的配置，需要通过滚动更新 pod 的方式来强制重新挂载 ConfigMap。

扩展链接：

 - [Kubernetes Pod 中的 ConfigMap配置更新](https://zhuanlan.zhihu.com/p/57570231)

