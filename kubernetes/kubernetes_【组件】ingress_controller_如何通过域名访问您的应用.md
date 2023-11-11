#  kubernetes ingress 如何通过域名访问您的应用
tags: 资源对象, Ingress


![在这里插入图片描述](https://img-blog.csdnimg.cn/63c31adcf6324e26a50cc6a1e80eb929.png)




-----

##  0. 背景
由于每个 Service 都要有一个负载均衡服务，所以这个做法实际上既浪费成本又高。作为用户，我其实更希望看到 Kubernetes 为我内置一个全局的负载均衡器。然后，通过我访问的 URL，把请求转发给不同的后端 Service。

**这种全局的、为了代理不同后端 Service 而设置的负载均衡服务，就是 Kubernetes 里的 Ingress 服务。**所以，Ingress 的功能其实很容易理解：**所谓 Ingress，就是 Service 的“Service”**。





## 1. 简介
[Ingress nginx](https://github.com/kubernetes/ingress-nginx) 是 Kubernetes 的一种 API 对象，将集群内部的 Service 通过 HTTP/HTTPS 方式暴露到集群外部，并通过规则定义 HTTP/HTTPS 的路由。Ingress 具备如下特性：集群外部可访问的 URL、负载均衡、SSL Termination、按域名路由（name-based virtual hosting）。增加了7层的识别能力，可以根据 http header, path 等进行路由转发。

举个例子，假如我现在有这样一个站点：`https://cafe.example.com`。其中，`https://cafe.example.com/coffee`，对应的是“咖啡点餐系统”。而，`https://cafe.example.com/tea`，对应的则是“茶水点餐系统”。这两个系统，分别由名叫 `coffee` 和 `tea` 这样两个 Deployment 来提供服务。

那么现在，**我如何能使用 Kubernetes 的 Ingress 来创建一个统一的负载均衡器，从而实现当用户访问不同的域名时，能够访问到不同的 Deployment 呢？**

 - `Ingress`: 配置转发规则，类似于 nginx 的配置文件
 - `Ingress Controller`: 转发，类似于 nginx，它会读取 Ingress 的规则并转化为 nginx 的配置文件

而 `Ingress Controller` 除了 nginx 外还有 `haproxy`，`ingress` 等等，我们选用 `nginx` 作为 Ingress Controller

##  2. Ingress对象定义

```bash
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: cafe-ingress
spec:
  tls:
  - hosts:
    - cafe.example.com
    secretName: cafe-secret
  rules:
  - host: cafe.example.com
    http:
      paths:
      - path: /tea
        backend:
          serviceName: tea-svc
          servicePort: 80
      - path: /coffee
        backend:
          serviceName: coffee-svc
          servicePort: 80
```
在上面这个名叫 `cafe-ingress.yaml` 文件中，最值得我们关注的，是 rules 字段。在 Kubernetes 里，这个字段叫作：`IngressRule`。

IngressRule 的 Key，就叫做：host。它必须是一个标准的**域名格式**（Fully Qualified Domain Name）的字符串，**而不能是 IP 地址**。

而 host 字段定义的值，就是这个 Ingress 的入口。这也就意味着，当用户访问 cafe.example.com 的时候，实际上访问到的是这个 Ingress 对象。这样，Kubernetes 就能使用 IngressRule 来对你的请求进行下一步转发。

而接下来 IngressRule 规则的定义，则依赖于 path 字段。你可以简单地理解为，这里的每一个 path 都对应一个后端 Service。所以在我们的例子里，我定义了两个 path，它们分别对应 coffee 和 tea 这两个 Deployment 的 Service（即：`coffee-svc` 和 `tea-svc`）。

**通过上面的讲解，不难看到，所谓 Ingress 对象，其实就是 Kubernetes 项目对“反向代理”的一种抽象。**


一个 Ingress 对象的主要内容，实际上就是一个“反向代理”服务（比如：Nginx）的配置文件的描述。而这个代理服务对应的转发规则，就是 `IngressRule`。

这就是为什么在每条 IngressRule 里，需要有一个 host 字段来作为这条 IngressRule 的入口，然后还需要有一系列 path 字段来声明具体的转发策略。这其实跟 Nginx、HAproxy 等项目的配置文件的写法是一致的。而有了 Ingress 这样一个统一的抽象，Kubernetes 的用户就无需关心 Ingress 的具体细节了。在实际的使用中，你只需要从社区里选择一个具体的 Ingress Controller，把它部署在 Kubernetes 集群里即可。

然后，这个 `Ingress Controller` 会根据你定义的 `Ingress` 对象，提供对应的代理能力。目前，业界常用的各种反向代理项目，比如 Nginx、HAProxy、Envoy、Traefik 等，都已经为 Kubernetes 专门维护了对应的 `Ingress Controller`。

接下来，我就以最常用的 Nginx Ingress Controller 为例，在我们前面用 kubeadm 部署的 Bare-metal 环境中，和你实践一下 Ingress 机制的使用过程。


## 3. 安装 ingress-controller
[安装指导](https://github.com/kubernetes/ingress-nginx/blob/master/docs/deploy/index.md)
[下载](https://github.com/kubernetes/ingress-nginx/blob/main/deploy/static/provider/baremetal/deploy.yaml)

### yaml install
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/cloud/deploy.yaml
```
由于需要爬梯子下载镜像k`8s.gcr.io/ingress-nginx/controller:v0.43.0`，所以可以优先从阿里云下载

```bash
wget  https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.43.0/deploy/static/provider/baremetal/deploy.yaml
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/nginx-ingress-controller:v0.43.0
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/nginx-ingress-controller:v0.43.0 k8s.gcr.io/ingress-nginx/controller:v0.43.0
kubectl apply -f deploy.yaml
```

内容类型包含：

```bash
kind: Namespace
kind: ServiceAccount
kind: ConfigMap
kind: ClusterRole
kind: ClusterRoleBinding
kind: Role
kind: RoleBinding
kind: Service
kind: Service
kind: Deployment
kind: ValidatingWebhookConfiguration
kind: ServiceAccount
kind: ClusterRole
kind: ClusterRoleBinding
kind: Role
kind: RoleBinding
kind: Job
kind: Job
```

查看部分内容：

```bash
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-ingress-controller
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
      annotations:
        ...
    spec:
      serviceAccountName: nginx-ingress-serviceaccount
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.20.0
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --publish-service=$(POD_NAMESPACE)/ingress-nginx
            - --annotations-prefix=nginx.ingress.kubernetes.io
          securityContext:
            capabilities:
              drop:
                - ALL
              add:
                - NET_BIND_SERVICE
            # www-data -> 33
            runAsUser: 33
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
            - name: http
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443
```
可以看到，在上述 YAML 文件中，我们定义了一个使用 `nginx-ingress-controller` 镜像的 Pod。需要注意的是，这个 Pod 的启动命令需要使用该 Pod 所在的 Namespace 作为参数。而这个信息，当然是通过 Downward API 拿到的，即：Pod 的 env 字段里的定义（`env.valueFrom.fieldRef.fieldPath`）。


而这个 Pod 本身，就是一个监听 `Ingress` 对象以及它所代理的后端 Service 变化的控制器。

当一个新的 Ingress 对象由用户创建后，`nginx-ingress-controller` 就会根据 `Ingress 对象`里定义的内容，生成一份对应的 Nginx 配置文件（`/etc/nginx/nginx.conf`），并使用这个配置文件启动一个 Nginx 服务。

而一旦 Ingress 对象被更新，`nginx-ingress-controller` 就会更新这个配置文件。需要注意的是，如果这里只是被代理的 Service 对象被更新，nginx-ingress-controller 所管理的 Nginx 服务是不需要重新加载（reload）的。这当然是因为 nginx-ingress-controller 通过[Nginx Lua方案](https://github.com/openresty/lua-nginx-module)实现了 Nginx Upstream 的动态配置。

此外，`nginx-ingress-controller` 还允许你通过 Kubernetes 的 ConfigMap 对象来对上述 Nginx 配置文件进行定制。这个 ConfigMap 的名字，需要以参数的方式传递给 nginx-ingress-controller。而你在这个 ConfigMap 里添加的字段，将会被合并到最后生成的 Nginx 配置文件当中。

可以看到，一个 Nginx Ingress Controller 为你提供的服务，其实是一个可以根据 Ingress 对象和被代理后端 Service 的变化，来自动进行更新的 Nginx 负载均衡器。


查看pod `ingress-nginx-controller`是否正常`running`

```bash
$ kubectl get pods --all-namespaces -l app.kubernetes.io/name=ingress-nginx
NAMESPACE       NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx   ingress-nginx-admission-create-tqtmk        0/1     Completed   0          31m
ingress-nginx   ingress-nginx-admission-patch-cnwtg         0/1     Completed   0          31m
ingress-nginx   ingress-nginx-controller-548df9766d-kp5qq   1/1     Running     0          31m
```
查看svc ,`30304`和`31794`是我们从外网访问的统一端口

```bash
$ kubectl get svc -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.97.119.123    <none>        80:30304/TCP,443:31794/TCP   32m
ingress-nginx-controller-admission   ClusterIP   10.111.32.235    <none>        443/TCP                      32m
```
你一定要记录下这个 Service 的访问入口，即：**宿主机的地址和 NodePort 的端口**
为了后面方便使用，我会把上述访问入口设置为环境变量：
```bash
$ IC_IP=10.168.0.2 # 任意一台宿主机的地址
$ IC_HTTPS_PORT=30304 # NodePort端口
```

可以扩容

```bash
kubectl scale --replicas=2  deploy/nginx-ingress-controller -n ingress-nginx
```

###  helm install

```bash
$ helm upgrade --install ingress-nginx ingress-nginx \
>   --repo https://kubernetes.github.io/ingress-nginx \
>   --namespace ingress-nginx --create-namespace
Release "ingress-nginx" does not exist. Installing it now.
NAME: ingress-nginx
LAST DEPLOYED: Tue Sep  5 04:56:00 2023
NAMESPACE: ingress-nginx
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The ingress-nginx controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running 'kubectl --namespace ingress-nginx get services -o wide -w ingress-nginx-controller'

An example Ingress that makes use of the controller:
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: example
    namespace: foo
  spec:
    ingressClassName: nginx
    rules:
      - host: www.example.com
        http:
          paths:
            - pathType: Prefix
              backend:
                service:
                  name: exampleService
                  port:
                    number: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
      - hosts:
        - www.example.com
        secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls

```

## 4. 测试1: 代理一个服务
**deploy-demo.yaml**

```bash
apiVersion: v1
kind: Service
metadata:
  name: myapp
  namespace: default
spec:
  selector:
    app: myapp
    release: stable
  ports:
  - name: myapp
    port: 80
    targetPort: 80
---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: default
spec:
  selector:
    matchLabels:
      app: myapp
      release: stable
   replicas: 3
   template:
     metadata:
       labels:
         app: myapp
         release: stable
      spec:
        containers:
        - name: myapp
          image: nginx
          imagePullPolicy: IfNotPresent
        ports:
        - name: myapp
    　　containerPort: 80
```

vim  ingress-myapp.yaml

```bash
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
name: ingress-myapp
namespace: default
annotations:
   kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: httpd.hequan.com
    http:
     paths:
     - path:
       backend:
        serviceName: myapp
        servicePort: 80
```

kubectl create -f  deploy-demo.yaml
kubectl create -f  ingress-myapp.yaml

**linuxhosts添加域名解析**

```bash
vim /etc/hosts
#任意node_ip
192.168.100.112   httpd.hequan.com　　
```
**windows添加C:\Windows\System32\drivers\etc\hosts**

```bash
192.168.100.112   httpd.hequan.com　　
```

访问 `httpd.hequan.com:32080`


##  5. 测试2: 代理两个服务
[介质下载](https://github.com/resouer/kubernetes-ingress/tree/master/examples/complete-example)


```bash
#部署cafe的tea与coffee服务
$ kubectl create -f cafe.yaml

#创建 Ingress 所需的 SSL 证书（tls.crt）和密钥（tls.key）
$ kubectl create -f cafe-secret.yaml

#创建ingress对象
$ kubectl create -f cafe-ingress.yaml
```
查看信息

```bash
$ kubectl get ingress
NAME           HOSTS              ADDRESS   PORTS     AGE
cafe-ingress   cafe.example.com             80, 443   2h

$ kubectl describe ingress cafe-ingress
Name:             cafe-ingress
Namespace:        default
Address:          
Default backend:  default-http-backend:80 (<none>)
TLS:
  cafe-secret terminates cafe.example.com
Rules:
  Host              Path  Backends
  ----              ----  --------
  cafe.example.com  
                    /tea      tea-svc:80 (<none>)
                    /coffee   coffee-svc:80 (<none>)
Annotations:
Events:
  Type    Reason  Age   From                      Message
  ----    ------  ----  ----                      -------
  Normal  CREATE  4m    nginx-ingress-controller  Ingress default/cafe-ingress
```
可以看到，这个 Ingress 对象最核心的部分，正是 Rules 字段。其中，我们定义的 Host 是`cafe.example.com`，它有两条转发规则（Path），分别转发给 `tea-svc` 和 `coffee-svc`。

当然，在 Ingress 的 YAML 文件里，你还可以定义多个 Host，比如`restaurant.example.com、movie.example.com`等等，来为更多的域名提供负载均衡服务。

接下来，我们就可以通过访问这个 Ingress 的地址和端口，访问到我们前面部署的应用了，比如，当我们访问`https://cafe.example.com:443/coffee`时，应该是 coffee 这个 Deployment 负责响应我的请求。我们可以来尝试一下：


```bash
$ curl --resolve cafe.example.com:$IC_HTTPS_PORT:$IC_IP https://cafe.example.com:$IC_HTTPS_PORT/coffee --insecureServer address: 10.244.1.56:80
Server name: coffee-7dbb5795f6-vglbv
Date: 03/Nov/2018:03:55:32 +0000
URI: /coffee
Request ID: e487e672673195c573147134167cf898
```
我们可以看到，访问这个 URL 得到的返回信息是：`Server name: coffee-7dbb5795f6-vglbv`。这正是 coffee 这个 Deployment 的名字。

而当我访问`https://cafe.example.com:433/tea`的时候，则应该是 tea 这个 Deployment 负责响应我的请求（Server name: tea-7d57856c44-lwbnp），如下所示：

```bash
$ curl --resolve cafe.example.com:$IC_HTTPS_PORT:$IC_IP https://cafe.example.com:$IC_HTTPS_PORT/tea --insecure
Server address: 10.244.1.58:80
Server name: tea-7d57856c44-lwbnp
Date: 03/Nov/2018:03:55:52 +0000
URI: /tea
Request ID: 32191f7ea07cb6bb44a1f43b8299415c
```
可以看到，`Nginx Ingress Controller` 为我们创建的 Nginx 负载均衡器，已经成功地将请求转发给了对应的后端 Service。以上，就是 Kubernetes 里 Ingress 的设计思想和使用方法了。

不过，你可能会有一个疑问，如果我的请求没有匹配到任何一条 IngressRule，那么会发生什么呢？首先，既然 Nginx Ingress Controller 是用 Nginx 实现的，那么它当然会为你返回一个 Nginx 的 404 页面。

不过，Ingress Controller 也允许你通过 Pod 启动命令里的`–default-backend-service` 参数，设置一条默认规则，比如：`–default-backend-service=nginx-default-backend`。这样，任何匹配失败的请求，就都会被转发到这个名叫 nginx-default-backend 的 Service。所以，你就可以通过部署一个专门的 Pod，来为用户返回自定义的 404 页面了。

##  6. 总结

 - **Ingress 实际上就是 Kubernetes 对“反向代理”的抽象**

目前，**Ingress 只能工作在七层，而 Service 只能工作在四层**。所以当你想要在 Kubernetes 里为应用进行 **TLS 配置**等 HTTP 相关的操作时，都必须通过 Ingress 来进行。

当然，正如同很多负载均衡项目可以同时提供七层和四层代理一样，将来 Ingress 的进化中，也会加入四层代理的能力。这样，一个比较完善的“反向代理”机制就比较成熟了。而 Kubernetes 提出 Ingress 概念的原因其实也非常容易理解，有了 Ingress 这个抽象，用户就可以根据自己的需求来自由选择 Ingress Controller。比如，如果你的应用对代理服务的中断非常敏感，那么你就应该考虑选择类似于 Traefik 这样支持“热加载”的 Ingress Controller 实现。

在实际的生产环境中，Ingress 带来的灵活度和自由度，对于使用容器的用户来说，其实是非常有意义的。要知道，当年在 Cloud Foundry 项目里，不知道有多少人为了给 `Gorouter` 组件配置一个 TLS 而伤透了脑筋。

## 7. 思考题
如果我的需求是，当访问`www.mysite.com`和 `forums.mysite.com`时，分别访问到不同的 Service（比如：`site-svc` 和 f`orums-svc`）。那么，这个 Ingress 该如何定义呢？请你描述出 YAML 文件中的 rules 字段。
答案：
```bash
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: cafe-ingress
spec:
  tls:
  - hosts:
    - www.mysite.com
    - forums.mysite.com
    secretName: mysite-secret
  rules:
  - host: www.mysite.com
    http:
      paths:
      - path: /mysite
        backend:
          serviceName: site-svc
          servicePort: 80
  - host: forums.mysite.com
    http:
      paths:
      - path: /forums
        backend:
          serviceName: site-svc
          servicePort: 80
```

参考：

 - [**云原生圣经**](https://ghostwritten.blog.csdn.net/article/details/108562082)
 - [Ingress](https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress/)
 - [Ingress 控制器](https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress-controllers/)
 - [https://github.com/kubernetes/ingress-nginx/blob/main/docs/deploy/index.md](https://github.com/kubernetes/ingress-nginx/blob/main/docs/deploy/index.md)
 - [https://github.com/kubernetes/ingress-nginx](https://github.com/kubernetes/ingress-nginx)
