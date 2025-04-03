

----

##  1. Pod Presets and initializers
PodPreset 支持将数据注入到 Kubernetes 资源文件中，而无需编写自定义初始化程序。虽然 PodPreset 可能足以满足大多数用例的需求，但初始化程序为此数据注入启用了更多自定义逻辑。

在选择使用 Pod Preset 资源或 Initializers 时，请考虑以下最佳实践：

 - Initializers 只是通用准入 webhook 的一个特例。它们使您可以轻松使用现有的 Kubernetes控制器客户端代码来处理常见操作。轻松灵活地扩展 Kubernetes 的能力很重要。
 - 集群管理员需要注意他们安装的初始化程序。一个有缺陷的初始化器可以阻止每个人在集群中创建东西的能力，并且初始化器很强大，所以你需要能够信任它们的作者。
 - PodPreset是Kubernetes社区有人在Kubernetes的源代码中为你写的一个初始化器，你的集群管理员可以信任它。
 - Pod Preset 可能对大多数初始化用例有效。不是每个人都应该被要求编写初始化程序。


##  2. initializers如何工作
初始化程序需要三个对象才能起作用：

### 2.1 initializer服务进程（监视是否可做）
initializer控制器在创建时观察未初始化的工作负载。首先， t在挂起的初始化器列表中首先找到其配置的初始化器的名称。

然后初始化程序检查它是否负责初始化工作负载的命名空间。如果初始化程序没有初始化工作负载，初始化程序将忽略工作负载，否则，它将尝试为工作负载初始化工作。

初始化工作完成后，初始化器将自己从工作负载的待处理初始化器列表中删除。

###  2.2 initializerConfigMap（做什么）
configMap 通常包含一些初始化器的数据，例如注入数据或初始化器的一些其他配置数据。

###  2.3 initializerConfig（给谁做）
initializerConfig 资源用于配置启用哪些初始化程序以及哪些资源受初始化程序的约束。

您应该首先部署初始化器并确保它正常工作，然后再创建初始化器配置（我们通常将初始化器作为Kubernetes部署，您可以检查初始化器的pod是否正在运行）。否则，在 initializerConfig 中配置的任何新创建的资源都将停留在未初始化状态，然后超时。在此处查看如何解决此类问题。

您的集群中可以有多个初始化程序，每个初始化程序可以专注于不同命名空间中的不同任务。

下面是一张图，描述了上述三个对象的关系。


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/771e4a1b3fc53ff4dab1b7cf5c30ea33.png)

## 3. initializers教程
教程基于[https://github.com/kelseyhightower/kubernetes-initializer-tutorial](https://github.com/kelseyhightower/kubernetes-initializer-tutorial)，我对其进行了一些更新，使其更简单并在此处提交了所有代码[https://github.com/gyliu513/jay- work/tree/master/k8s/example/kube-initializer-tutorial](https://github.com/gyliu513/jay-%20work/tree/master/k8s/example/kube-initializer-tutorial)

现在，我们可以逐步完成教程，并尝试了解如何在遇到问题时进行故障排除。

###  3.1 配置
`initializers` 是 1.7 的 alpha 功能，因此您需要先启用它。

在 apiserver 中为 `runtime-config` 和 `Initializers` 为 `admission-control` 启用 `admissionregistration.k8s.io/v1alpha1`。

```bash
--runtime-config=admissionregistration.k8s.io/v1alpha1
```

###  3.2 部署 sidecar initializer configMap

```bash
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/configmaps# cat sidecar-initializer.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: sidecar-initializer
data:
  config: |
    containers:
    - name: sidecar-nginx
      image: nginx:1.8.1
      imagePullPolicy: IfNotPresent
```
上面的 configMap 只包含将被注入到工作负载的容器信息。您可以在此处添加更多数据，例如卷、`sidecar` 初始化程序配置等。

```bash
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/configmaps# kubectl apply -f ./sidecar-initializer.yaml
configmap "sidecar-initializer" created
```
### 3.3 部署sidecar initializer

```bash
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/deployments# cat sidecar-initializer.yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  initializers:
    pending: []
  labels:
    app: sidecar-initializer
  name: sidecar-initializer
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: sidecar-initializer
      name: sidecar-initializer
    spec:
      containers:
      - name: sidecar-initializer
        image: gcr.io/hightowerlabs/envoy-initializer:0.0.1
        imagePullPolicy: IfNotPresent
        args:
        - "-initializer-name=sidecar.initializer.kubernetes.io"
        - "-configmap=sidecar-initializer"
```
你可能会看到上面部署的 `initializers` 是 `[]` 如下：

```bash
metadata:
  initializers:
    pending: []
```

我们将它定义为上面的原因是因为初始化器应该显式地将待处理初始化器列表设置为排除自身，或者设置为一个空数组，以避免在等待初始化时卡住。
将挂起的初始化器列表设置为排除自身：

```bash
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  initializers:
    pending:
      - initializer.vaultproject.io
      # Do not include the Sidecar Initializer
      # - sidecar.initializer.kubernetes.io
  name: sidecar-initializer
```
将挂起的初始值设定项设置为空数组：


```bash
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  initializers:
    pending: []
```

然后部署sidecar deployment，使用deployment可以方便升级和自动重启。

```bash
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/deployments# kubectl apply -f ./sidecar-initializer.yaml
deployment "sidecar-initializer" created
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/deployments# kubectl get pods
NAME                                   READY     STATUS    RESTARTS   AGE
sidecar-initializer-7f4767c77b-rnw4w   1/1       Running   0          6s
```
当一个对象被 POST 时，它会根据所有现有的 initializerConfiguration 对象进行检查（解释如下）。对于它匹配的所有内容，所有 `spec.initializers[].names` 都附加到新对象的 `metadata.initializers.pending` 字段。

 `initializer controller`应该通过使用查询参数`?includeUninitialized=true` 列出并监视未初始化的对象。如果使用 client-go，只需将 `listOptions.includeUninitialized` 设置为 true。

下面是一些伪代码来描述逻辑，您可以从[https://github.com/gyliu513/roadmap/blob/master/k8s/example/kube-initializer-tutorial/sidecar-initializer/main.go#L58-L129](https://github.com/gyliu513/roadmap/blob/master/k8s/example/kube-initializer-tutorial/sidecar-initializer/main.go#L58-L129)获得更多详细信息。

```bash
// 观察所有命名空间中未初始化的部署。
restClient := clientset.AppsV1beta1().RESTClient() 
watchlist := cache.NewListWatchFromClient(restClient, "deployments", corev1.NamespaceAll, fields.Everything())
// 包装返回的监视列表以解决无法
在设置监视客户端时包含// `IncludeUninitialized` 列表选项的问题。
includeUninitializedWatchlist := &cache.ListWatch{ 
        ListFunc: func(options metav1.ListOptions) (runtime.Object, error) { 
                options.IncludeUninitialized = true 
                return watchlist.List(options) 
        }, 
        WatchFunc: func(options metav1.ListOptions) (watch.接口，错误) { 
                options.IncludeUninitialized = true 
                return watchlist.Watch(options) 
        }, 
}
resyncPeriod := 30 * time.Second
_, 控制器 := cache.NewInformer(includeUninitializedWatchlist, &v1beta1.Deployment{}, resyncPeriod, 
        cache.ResourceEventHandlerFuncs{ 
                AddFunc: func(obj interface{}) { 
                        err := initializeDeployment(obj.(*v1beta1.Deployment), c, clientset )
                        如果 err != nil { 
                                log.Println(err) 
                        } 
                }, 
        }, 
)
停止 := make(chan struct{})
去 controller.Run(stop)
signalChan := make(chan os.Signal, 1) 
signal.Notify(signalChan, syscall.SIGINT, syscall.SIGTERM) 
<-signalChan
log.Println("收到关机信号，退出中...") 
close(stop)
```

### 3.4 创建initializeConfig

```bash
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/initializer-configurations# cat sidecar.yaml
apiVersion: admissionregistration.k8s.io/v1alpha1
kind: InitializerConfiguration
metadata:
  name: sidecar
initializers:
- name: sidecar.initializer.kubernetes.io
  rules:
  - apiGroups:
    - "*"
    apiVersions:
    - "*"
    resources:
    - deployments
```
以上意味着 sidecar 初始化器只会影响部署资源。

```bash
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/initializer-configurations# kubectl apply -f ./sidecar.yaml
initializerconfiguration "sidecar" created
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/initializer-configurations# kubectl get initializerconfiguration
NAME      AGE
sidecar   5s
```

###  3.5 创建一个 nginx 应用程序
现在让我们创建一个 nginx 应用程序，看看是否可以将 sidecar 注入到 nginx 应用程序中。


```bash
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/deployments# cat nginx.yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
      name: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.8.1
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/deployments# kubectl apply -f ./nginx.yaml
deployment "nginx" created
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/deployments# kubectl get pods
NAME                                   READY     STATUS    RESTARTS   AGE
nginx-f5f94b64-g95dx                   2/2       Running   0          4s
sidecar-initializer-7f4767c77b-rnw4w   1/1       Running   0          32m
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/deployments# kubectl get pods nginx-f5f94b64-g95dx  -oyaml
...
spec:
  containers:
  - image: nginx:1.8.1
    imagePullPolicy: IfNotPresent
    name: nginx
    ports:
    - containerPort: 80
      protocol: TCP
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-5j13j
      readOnly: true
  - image: nginx:1.8.1
    imagePullPolicy: IfNotPresent
    name: sidecar-nginx
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-5j13j
      readOnly: true
...
```

从上面，我们可以看到 sidecar 初始化器 configMap 中的 sidecar 已经被注入到了 nginx 应用程序中。
但是这有一个问题，通过上述配置，所有的部署都会注入一个 sidecar，这在某些情况下可能不是预期的。
对于这种情况，我们可以更新 sidecar 初始化程序以过滤具有指定注释的应用程序，使用注释来启用或退出初始化。
现在让我们删除 sidecar 初始化器、sidecar initializerConfig 和 nginx，然后重新创建带有逻辑的 sidecar 初始化器以过滤掉一些部署。

```bash
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial# kubectl delete -f ./initializer-configurations/sidecar.yaml
initializerconfiguration "sidecar" deleted
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial# kubectl delete -f deployments/sidecar-initializer.yaml
deployment "sidecar-initializer" deleted
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial# kubectl delete -f deployments/nginx.yaml
deployment "nginx" deleted
```
### 3.6 重新部署带有注释过滤器的 sidecar  initializer

```bash
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/deployments# cat sidecar-initializer-with-annotation.yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  initializers:
    pending: []
  labels:
    app: sidecar-initializer
  name: sidecar-initializer
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: sidecar-initializer
      name: sidecar-initializer
    spec:
      containers:
      - name: sidecar-initializer
        image: gcr.io/hightowerlabs/envoy-initializer:0.0.1
        imagePullPolicy: IfNotPresent
        args:
        - "-annotation=initializer.kubernetes.io/sidecar"
        - "-require-annotation=true"
        - "-initializer-name=sidecar.initializer.kubernetes.io"
        - "-configmap=sidecar-initializer"
```

您可能会看到我们添加了一个名为 `-annotation=initializer.kubernetes.io/sidecar` 的新参数，该参数将帮助检查部署是否包含所需的注释，sidecar 初始化程序不会向部署注入数据。

```bash
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/deployments# kubectl apply -f ./sidecar-initializer-with-annotation.yaml
deployment "sidecar-initializer" created
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/deployments# kubectl get pods
NAME                                   READY     STATUS             RESTARTS   AGE
sidecar-initializer-5b7699577d-gp654   1/1       Running            0          4s
```
下面是一些关于如何使用`annotation`来过滤一些应用程序的伪代码，你可以从[https://github.com/gyliu513/jay-work/blob/master/k8s/example/kube-initializer-tutorial/sidecar-initializer/main.go#L131-L192](https://github.com/gyliu513/jay-work/blob/master/k8s/example/kube-initializer-tutorial/sidecar-initializer/main.go#L131-L192)获得更多细节-教程[/sidecar-initializer/main.go#L131-L192](https://github.com/gyliu513/jay-work/blob/master/k8s/example/kube-initializer-tutorial/sidecar-initializer/main.go#L131-L192)

```bash
if requireAnnotation {
        a := deployment.ObjectMeta.GetAnnotations()
        _, ok := a[annotation]
        if !ok {
                log.Printf("Required '%s' annotation missing; skipping envoy container injection", annotation)
                _, err = clientset.AppsV1beta1().Deployments(deployment.Namespace).Update(initializedDeployment)
                if err != nil {
                        return err
                }
                return nil
        }
}
```

###  3.7 重新部署初始化配置

```bash
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/initializer-configurations# kubectl apply -f ./sidecar.yaml
initializerconfiguration "sidecar" created
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/initializer-configurations# kubectl get initializerconfiguration
NAME      AGE
sidecar   5s
```



### 3.8 部署没有注解的nginx服务

```bash
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/deployments# kubectl apply -f ./nginx.yaml
deployment "nginx" created
```
由于上面的nginx不包含注解，所以部署创建后，不会有注入数据。

```bash
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/deployments# kubectl get pods
NAME                                   READY     STATUS    RESTARTS   AGE
nginx-7ff795885f-pklsf                 1/1       Running   0          3s
sidecar-initializer-5b7699577d-gp654   1/1       Running   0          5m
```
从上面的输出可以看出，nginx只包含一个容器，并没有注入数据。

### 3.9 部署注解的nginx服务

```bash
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/deployments# cat nginx-with-annotation.yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  annotations:
    "initializer.kubernetes.io/sidecar": "true"
  labels:
    app: nginx
    envoy: "true"
  name: nginx-with-annotation
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
        envoy: "true"
      name: nginx-with-annotation
    spec:
      containers:
      - name: nginx
        image: nginx:1.8.1
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
```
从上面，我们可以看到部署包括一个注释如下：

```bash
annotations:
  "initializer.kubernetes.io/sidecar": "true"
```
这意味着 sidecar 初始化程序将向此部署注入数据。

```bash
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/deployments# kubectl apply -f ./nginx-with-annotation.yaml
deployment "nginx-with-annotation" created
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/deployments# kubectl get pods
NAME                                     READY     STATUS    RESTARTS   AGE
nginx-7ff795885f-pklsf                   1/1       Running   0          4m
nginx-with-annotation-5cf4d7fcdb-f7b9l   2/2       Running   0          2s
sidecar-initializer-5b7699577d-gp654     1/1       Running   0          9m
```
可以看到nginx新建的pod`nginx-with-annotation-5cf4d7fcdb-f7b9l`有两个容器，说明注入已经生效。

## 4. 故障排除
有时，您可能会发现无法部署应用程序，所有应用程序都会挂起大约 30 秒然后超时。对于此类问题，通常是sidecar初始化器无法正常工作时的initializerConfig引起的，也可能是初始化器中的某些错误导致初始化器停止工作引起的。
让我们一步一步模拟错误情况。

###  4.1 删除 sidecar 初始化程序
到目前为止，我们的初始化器运行良好，现在让我们删除 sidecar 初始化器和两个 nginx，这将留下一个没有初始化器initializerConfig。

```bash
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/deployments# kubectl delete -f ./sidecar-initializer-with-annotation.yaml
deployment "sidecar-initializer" deleted
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/deployments# kubectl delete -f ./nginx-with-annotation.yaml
deployment "nginx-with-annotation" deleted
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/deployments# kubectl delete -f ./nginx.yaml
deployment "nginx" deleted

root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/deployments# kubectl get pods
No resources found.
```
initializerConfig 仍然存在。

```bash
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/deployments# kubectl get initializerconfigurations 
NAME AGE 
sidecar 10m
```
### 4.2 创建一个 nginx

```bash
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/deployments# kubectl apply -f ./nginx.yaml
Error from server (Timeout): error when creating "./nginx.yaml": Timeout: request did not complete within allowed duration
```
从上面可以看出，由于超时，nginx 创建失败。
然后我尝试获取部署，没有部署。

```bash
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/deployments# kubectl get deploy
No resources found.
```
但是当我再次创建nginx部署时，它失败了！！

```bash
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/deployments# kubectl create -f ./nginx.yaml
Error from server (AlreadyExists): error when creating "./nginx.yaml": deployments.apps "nginx" already exists
```

### 4.3 获取未初始化的部署
对于此类问题，一般都是未初始化的对象造成的，我们可以通过在kubectl中添加一个名为`--include-uninitialized`的标志来获取所有未初始化的对象，例如`kubectl get deploy --include-uninitialized `.

```bash
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/deployments# kubectl get deploy --include-uninitialized -oyaml
apiVersion: v1
items:
- apiVersion: extensions/v1beta1
  kind: Deployment
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"apps/v1beta1","kind":"Deployment","metadata":{"annotations":{},"labels":{"app":"nginx"},"name":"nginx","namespace":"default"},"spec":{"replicas":1,"template":{"metadata":{"labels":{"app":"nginx"},"name":"nginx"},"spec":{"containers":[{"image":"nginx:1.8.1","imagePullPolicy":"IfNotPresent","name":"nginx","ports":[{"containerPort":80}]}]}}}}
    creationTimestamp: 2017-09-25T08:02:23Z
    generation: 1
    initializers:
      pending:
      - name: sidecar.initializer.kubernetes.io
    labels:
      app: nginx
    name: nginx
    namespace: default
    ...
```
可以看到上面的输出包括以下内容：

```bash
initializers:
  pending:
  - name: sidecar.initializer.kubernetes.io
```
由于我没有诸如初始化程序之类的，因此部署一直未决。在此处查看更多详细信息[https://github.com/kubernetes/kubernetes/issues/51883](https://github.com/kubernetes/kubernetes/issues/51883)。

###  4.4 如何解决
我们需要先删除 initializeConfig。

```bash
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/initializer-configurations# kubectl delete -f ./sidecar.yaml
initializerconfiguration "sidecar" deleted
```
由于 nginx 部署已经包含了 `pending initializers`，所以我们需要通过删除 `pending initializers` 来编辑部署并删除部署。

```bash
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/initializer-configurations# kubectl edit deploy nginx  --include-uninitialized
deployment "nginx" edited
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/initializer-configurations# kubectl get deploy
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx     0         0         0            0           21m
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/initializer-configurations# kubectl delete deploy nginx
deployment "nginx" deleted
```
然后重新创建nginx部署，就会成功。

```bash
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/initializer-configurations# kubectl apply -f ../deployments/nginx.yaml
deployment "nginx" created
root@k8s001:~/cases/jay-work/k8s/example/kube-initializer-tutorial/initializer-configurations# kubectl get pods
NAME                     READY     STATUS    RESTARTS   AGE
nginx-7ff795885f-gdzss   1/1       Running   0          2s
```
## 5. 总结
助 Kubernetes Initializers，我们可以更灵活地自定义如何将数据注入 Kubernetes 资源的逻辑。
初始化程序可以为 isito 服务网格提供很多帮助。在 istio 0.1.X 中，如果你想部署一个由 istio 管理的应用程序，你需要先调用 `istioctl kube-inject` 命令将 sidecar 容器和卷注入到部署中，然后再调用 `kubectl apply`部署应用程序。这对最终用户来说并不方便，因为为 istio 部署一个应用程序会请求两次 API 调用。
在 istio 0.2.X 中，我们引入了一个 istio sidecar 初始化器，它可以帮助将数据动态注入到 Kubernetes 资源中，这使得最终用户可以更轻松地部署由 istio 管理的应用程序，因为这不需要调用 `istioctl kube-inject`手动注入数据，但所有数据将由 istio sidecar 初始化程序自动注入。


-------------------------------------------------------
## 6. 流程特性
### 6.1 配置需要 initialization 的资源类型 

`InitializerConfiguration`（https://kubernetes.io/docs/admin/extensible-admission-controllers/#configure-initializers-on-the-fly） 对象允许你配置被 initializers 分配到的资源类型。

举个例子来说，你可以进行创建，将“myproxy”initializer 添加到类型`apps/v1beta1.Deployment` 和 `v1.DeamonSet` 对象中。你可以尽可能多地创建 InitializerConfigurations，它们将适用于所有命名空间  。

###  6.2 API 服务器将为新资源分配 initializers
当你向 apiserver 提交 Deployment 对象时，将更新 Deployment 的`metadata.initalizers.pending` 并在其中添加“myproxy”值。此字段显示当前分配给资源的initializers 。

准确的说，不是由 apiserver 添加 initializers。有一个名为“Initializer”的准入控制插件，这使整个 initialization 流程成为可能。它通过将 `–admission-controller=Initializer` 标志添加到 kube-apiserver 来进行启用 。

###  6.3 可以编写一个控制器来检测资源情况
你开发并部署到集群的自定义控制器使用Watch API 来对新的资源进行监听，捕获并进行所需的修改。

###  6.4 等待修改资源的轮次
一旦你的控制器通过 Watch API 拦截一个对象，它只能修改对象，如果它在初始化器列表（`metadata.initializers.pending[0]`）的第一个元素上检查到它的名字的话。否则，这意味着它是一些其他 initializers 修改资源的轮次，而现在应该跳过修改。

###  6.5 完成修改, 生成下一个 initializer
完成对资源的修改后, 控制器应从对象的 `metadata.initializers.pending` 列表中删除其名称, 并将该对象保存回 API 服务器。

###  6.6 不存在更多 initializers 时, 资源准备好进行实现
当 Kubernetes API 服务器检测到该对象没有其他挂起的 initializers 时, 它会判定对象 “已initialized”。Kubernetes 调度程序和其他控制器可以检测到完全 initialized 的对象并加以利用。

您可以同时在群集上运行多个 initializers。这些自定义控制器中的每一个都将收到有关对资源 (如 Pods) 的修改的通知,  但他们会等待轮次来对对象进行修改，直到他们在列表中检测他们的名字。

###  6.7 Initializers：在 Kubernetes API 中进行实现
在不知道它们实际上是如何在 Kubernetes API 服务器之下实现的情况下，你可以对 initializers 进行开发和部署 。我们来详细讨论一下它是如何在 Kubernetes API 中进行实现的：

简而言之, 当一个 Pod 资源提交到 API 并被分配一个待定的 initializers 列表时, 直到 initialization 完成，它实际上都不会被安排。Kubernetes 调度程序, 这个所谓的调度程序其实是另一个控制器，检测在 API 服务器中显示的 pod，并分配给每个节点。

那么, 为什么调度程序和其他控制器在初始化之前无法看到该对象, 即使该对象保存在 API 服务器 （和预计数据库中）, 并且对某些其他控制器 （即 initializers）可见？

答案是存在一个叫 `includeUninitialized` 的请求参数。此参数默认为 false, 因此, API 将未初始化的对象从默认客户端（如 kubectl）和控制器 (如调度程序) 隐藏在 “监视” 或 “列表” 之类的请求中。所以你开发的 initializers 必须设置 `includeUninitialized = true` 查询参数才能监测这些对象。

Initializers 阻止创建请求。当创建对象的请求提交到 API 服务器时, 它不会马上返回, 并且请求会一直被阻止, 直到 initialization 完成。如果使用的是 kubectl, 并且对象在 uninitialized 状态下被阻止, 你会在30秒后发现 kubectl 超时。

参考链接：
- [https://github.com/kubernetes/kubernetes/pull/50497](https://github.com/kubernetes/kubernetes/pull/50497)
- [https://ahmet.im/blog/initializers/](https://ahmet.im/blog/initializers/)
- [https://kubernetes.io/docs/admin/extensible-admission-controllers/](https://kubernetes.io/docs/admin/extensible-admission-controllers/)

相关阅读：
 1. [Kubernetes 声明式API【1】初遇](https://ghostwritten.blog.csdn.net/article/details/110170353)
 2. [Kubernetes 声明式API【2】相知CRD](https://ghostwritten.blog.csdn.net/article/details/119155497)
 3. [Kubernetes 声明式API【3】热恋CRD](https://ghostwritten.blog.csdn.net/article/details/119180405)
 4. [kubernetes Operator 【1】入门练习](https://blog.csdn.net/xixihahalelehehe/article/details/104279855)
 5. [kubernetes Operator 【2】实战CRD编程](https://blog.csdn.net/xixihahalelehehe/article/details/108196187)
 6. [kubernetes Operator 【3】工作原理和编写方法](https://ghostwritten.blog.csdn.net/article/details/119250990)
 7. [kubernetes InitializerConfiguration 分析](https://blog.csdn.net/xixihahalelehehe/article/details/120866032?spm=1001.2014.3001.5501)
