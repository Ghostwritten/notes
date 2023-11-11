![在这里插入图片描述](https://img-blog.csdnimg.cn/20201210191350374.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)

----

## 1. 简介
### 1.1 kustomize 是什么？
官网的描述：
kustomize 是 kubernetes 原生的配置管理，以无模板方式来定制应用的配置。kustomize 使用 k8s 原生概念帮助创建并复用资源配置(YAML)，允许用户以一个应用描述文件 （YAML 文件）为基础（Base YAML），然后通过 Overlay 的方式生成最终部署应用所需的描述文件。

### 1.2 kustomize 解决了什么痛点？
一般应用都会存在多套部署环境：开发环境、测试环境、生产环境，多套环境意味着存在多套 K8S 应用资源 YAML。而这么多套 YAML 之间只存在微小配置差异，比如镜像版本不同、Label 不同等，而这些不同环境下的YAML 经常会因为人为疏忽导致配置错误。再者，多套环境的 YAML 维护通常是通过把一个环境下的 YAML 拷贝出来然后对差异的地方进行修改。一些类似 Helm 等应用管理工具需要额外学习DSL 语法。总结以上，在 k8s 环境下存在多套环境的应用，经常遇到以下几个问题：

 - 如何管理不同环境或不同团队的应用的 Kubernetes YAML 资源？
 - 如何以某种方式管理不同环境的微小差异，使得资源配置可以复用，减少 copy and change 的工作量？
 - 如何简化维护应用的流程，不需要额外学习模板语法？

Kustomize 通过以下几种方式解决了上述问题：

 - kustomize 通过 Base & Overlays 方式(下文会说明)方式维护不同环境的应用配置
 - kustomize 使用 patch 方式复用 Base 配置，并在 Overlay 描述与 Base 应用配置的差异部分来实现资源复用
 - kustomize 管理的都是 Kubernetes 原生 YAML 文件，不需要学习额外的 DSL 语法


### 1.3 架构
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200813002230815.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)

## 2. 术语

 - `kustomization` 
 术语 kustomization 指的是 `kustomization.yaml` 文件，或者指的是包含kustomization.yaml 文件的目录以及它里面引用的所有相关文件路径。
 - `base` 
 base 指的是一个 `kustomization` , 任何的 kustomization 包括 `overlay`
   (后面提到)，都可以作为另一个 kustomization 的 base (简单理解为基础目录)。base
   中描述了共享的内容，如资源和常见的资源配置。**base 是需要通过 overlay 修订的基础配置，这部分是类似 Docker 镜像的只读层，通常不做修改。**
 - `overlay` 
 overlay 是一个 kustomization, 它修改(并因此依赖于)另外一个 kustomization.
   **overlay 中的 kustomization指的是一些其它的 kustomization, 称为其 base.** 没有 base, overlay 无法使用，并且一个 overlay 可以用作 另一个 overlay 的 base(基础)。简而言之，overlay声明了与 base 之间的差异。通过 overlay 来维护基于 base 的不同 `variants`(变体)，例如开发、QA和生产环境的不同 variants。**overlay 提供针对 base 修改的配置部分，覆盖相同的配置项部分，提供不同的配置值。**
 - `variant` 
 variant 是在集群中将 overlay 应用于 base 的结果。例如开发和生产环境都修改了一些共同 base以创建不同的 variant。这些 variant 使用相同的总体资源，并与简单的方式变化，例如 `deployment`的副本数、`ConfigMap`使用的数据源等。简而言之，variant 是含有同一组 base 的不同 kustomization
 - `resource` 
 在 kustomize 的上下文中，**resource 是描述 k8s API 对象的 YAML 或 JSON
   文件的相对路径**。即是指向一个声明了 kubernetes API 对象的 YAML 文件
 - `patch` 
 修改文件的一般说明。文件路径，指向一个声明了 kubernetes API patch 的 YAML 文件

## 3. 安装
### 3.1 第一种
下载：
[kustomize](https://github.com/kubernetes-sigs/kustomize/releases)

```bash
$ tar -xvf kustomize_v3.8.0_linux_amd64.tar.gz
$ mv kustomize /usr/local/bin/
$ kustomize version
{Version:kustomize/v3.8.0 GitCommit:6a50372dd5686df22750b0c729adaf369fbf193c BuildDate:2020-07-05T14:08:42Z GoOs:linux GoArch:amd64}
```
### 3.2 第二种
如果本地机器 go 的版本在 1.10.1 以上，可以通过 go get 来直接安装

```bash
go get github.com/kubernetes-sigs/kustomize
```
## 4. hello world
### 4.1 base
#### 4.1.1 base创建

```bash
#!/bin/bash
export DEMO_HOME=~/hello
export BASE=$DEMO_HOME/base
mkdir -p $BASE
curl -s -o "$BASE/#1.yaml" "https://raw.githubusercontent.com\kubernetes-sigs/kustomize/master/examples/helloWorld/{configMap,deployment,kustomization,service}.yaml"
```
此时目录结构:

```bash
~/hello
 └── base
     ├── configMap.yaml
     ├── deployment.yaml
     ├── kustomization.yaml
     └── service.yaml
```
如果下载不了
下载github的[helloworld](https://github.com/kubernetes-sigs/kustomize/tree/master/examples/helloWorld)
验证 kustomize 配置并合并打印base输出:（**注意后边与staging、deployment的区别**）

```bash
$ kustomize build $DEMO_HOME/base
apiVersion: v1
data:
  altGreeting: Good Morning!
  enableRisky: "false"
kind: ConfigMap
metadata:
  labels:
    app: my-hello
  name: the-map
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: my-hello
  name: the-service
spec:
  ports:
  - port: 8666
    protocol: TCP
    targetPort: 8080
  selector:
    app: my-hello
    deployment: hello
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: my-hello
  name: the-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-hello
      deployment: hello
  template:
    metadata:
      labels:
        app: my-hello
        deployment: hello
    spec:
      containers:
      - command:
        - /hello
        - --port=8080
        - --enableRiskyFeature=$(ENABLE_RISKY)
        env:
        - name: ALT_GREETING
          valueFrom:
            configMapKeyRef:
              key: altGreeting
              name: the-map
        - name: ENABLE_RISKY
          valueFrom:
            configMapKeyRef:
              key: enableRisky
              name: the-map
        image: monopole/hello:1
        name: the-container
        ports:
        - containerPort: 8080


```

此时会完整合并`各个yaml配置`，验证语法，合并成最终的配置。以上命令也可以等同采用 kubectl kustomize base 。

执行完成部署，该用如下命令方法:

```bash
kustomize build $DEMO_HOME/base | kubectl apply -f -
```

最新kubectl 1.14集成了kustomize功能，也可以采用:

```bash
kubectl apply -k $DEMO_HOME/base
```

可能需要修改 base/kustomization.yaml 开头添加以下行（ 参考 require apiVersion/kind in Kustomization.yaml? #738 ）:

```bash
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
```

#### 4.1.2 base配置
定制 app label 并应用于所有的 resources ：

```bash
$ kustomize build $BASE | grep -C 3 app:
kind: ConfigMap
metadata:
  labels:
    app: hello

$ sed -i.bak 's/app: hello/app: my-hello/' $BASE/kustomization.yaml
$ kustomize build $BASE | grep -C 3 app:
kind: ConfigMap
metadata:
  labels:
    app: my-hello

```

### 4.2 staging（overlays）
#### 4.2.1 配置kustomization.yaml说明变动的文件
我们通常会创建一个模拟环境(staging)和一个生产环境(production) overlay:

staging环境激活production环境没有激活的一个有风险功能
production环境设置一个较高的副本数量
web服务器的集群变量设置不同

```bash
#!/bin/bash
export OVERLAYS=$DEMO_HOME/overlays
mkdir -p $OVERLAYS/staging
mkdir -p $OVERLAYS/production
```
在 `staging` 目录下，定义一个新的使用不同标签的配置 - `staging/kustomizaton.yaml`
```bash
cat <<'EOF' >$OVERLAYS/staging/kustomization.yaml
namePrefix: staging-
commonLabels:
  variant: staging
  org: acmeCorporation
commonAnnotations:
  note: Hello, I am staging!
resources:
- ../../base
patchesStrategicMerge:
- map.yaml
EOF
```
`patchesStrategicMerge:` # 这里意思是我们要把base里的configmap用下面的configmap overlay一下
#### 4.2.2 配置变动的文件具体内容
添加 `configMap` 定制`staging`环境服务的消息，并且激活 risky 标记 - `staging/map.yaml`

```bash
cat <<EOF >$OVERLAYS/staging/map.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: the-map
data:
  altGreeting: "Have a pineapple!"
  enableRisky: "true"
EOF
```
验证 kustomize 配置并合并打印staging输出:（**注意base、deployment的区别**）
 **元数据都会随环境不同根据kustomization.yaml自动生成固定得名称**
```bash
$ kustomize build $OVERLAYS/staging
apiVersion: v1
data:  ##数据不一样，因为补丁是变得是confimap
  altGreeting: Have a pineapple!
  enableRisky: "true"
kind: ConfigMap    
metadata:  #元数据不一样
  annotations:
    note: Hello, I am staging!
  labels:
    app: my-hello
    org: acmeCorporation
    variant: staging
  name: staging-the-map
---
apiVersion: v1
kind: Service
metadata:
  annotations:
    note: Hello, I am staging!
  labels:
    app: my-hello
    org: acmeCorporation
    variant: staging
  name: staging-the-service
spec:
  ports:
  - port: 8666
    protocol: TCP
    targetPort: 8080
  selector:
    app: my-hello
    deployment: hello
    org: acmeCorporation
    variant: staging
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    note: Hello, I am staging!
  labels:
    app: my-hello
    org: acmeCorporation
    variant: staging
  name: staging-the-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-hello
      deployment: hello
      org: acmeCorporation
      variant: staging
  template:
    metadata:
      annotations:
        note: Hello, I am staging!
      labels:
        app: my-hello
        deployment: hello
        org: acmeCorporation
        variant: staging
    spec:
      containers:
      - command:
        - /hello
        - --port=8080
        - --enableRiskyFeature=$(ENABLE_RISKY)
        env:
        - name: ALT_GREETING
          valueFrom:
            configMapKeyRef:
              key: altGreeting
              name: staging-the-map
        - name: ENABLE_RISKY
          valueFrom:
            configMapKeyRef:
              key: enableRisky
              name: staging-the-map
        image: monopole/hello:1
        name: the-container
        ports:
        - containerPort: 8080

```

### 4.3 production（overlay）
#### 4.3.1 配置kustomization.yaml说明变动的文件
在 production目录下，定义一个新的使用不同标签的配置 -production/kustomizaton.yaml
```bash
cat <<'EOF' >$OVERLAYS/production/kustomization.yaml
namePrefix: production-
commonLabels:
  variant: production
  org: acmeCorporation
commonAnnotations:
  note: Hello, I am production!
bases:
- ../../base
patchesStrategicMerge:
- deployment.yaml
EOF
```
#### 4.3.2  配置变动的文件具体内容

```bash
cat <<EOF >$OVERLAYS/production/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: the-deployment
spec:
  replicas: 10
EOF
```
对比overlays
现在的目录结构如下:

```bash
~/hello
 ├── base
 │   ├── configMap.yaml
 │   ├── deployment.yaml
 │   ├── kustomization.yaml
 │   └── service.yaml
 └── overlays
     ├── production
     │   ├── deployment.yaml
     │   └── kustomization.yaml
     └── staging
         ├── kustomization.yaml
         └── map.yaml
```
对比目录可以看到 staging 和 production 的差异:

```bash
diff \
  <(kustomize build $OVERLAYS/staging) \
  <(kustomize build $OVERLAYS/production) |\
  more
```
验证 kustomize 配置并合并打印deployment输出:（**注意base、staging的区别**）

```bash
$ kustomize build $OVERLAYS/production
apiVersion: v1
data:   #数据一样
  altGreeting: Good Morning! 
  enableRisky: "false"
kind: ConfigMap      
metadata:         
  annotations:     
    note: Hello, I am production!  
  labels:
    app: my-hello
    org: acmeCorporation
    variant: production
  name: production-the-map
---
apiVersion: v1
kind: Service
metadata:
  annotations:
    note: Hello, I am production!
  labels:
    app: my-hello
    org: acmeCorporation
    variant: production
  name: production-the-service
spec:
  ports:
  - port: 8666
    protocol: TCP
    targetPort: 8080
  selector:
    app: my-hello
    deployment: hello
    org: acmeCorporation
    variant: production
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:  
  annotations:
    note: Hello, I am production!
  labels:
    app: my-hello
    org: acmeCorporation
    variant: production
  name: production-the-deployment
spec:
  replicas: 10   #副本数不一样，因为补丁是变得deployment
  selector:
    matchLabels:
      app: my-hello
      deployment: hello
      org: acmeCorporation
      variant: production
  template:
    metadata:
      annotations:
        note: Hello, I am production!
      labels:
        app: my-hello
        deployment: hello
        org: acmeCorporation
        variant: production
    spec:
      containers:
      - command:
        - /hello
        - --port=8080
        - --enableRiskyFeature=$(ENABLE_RISKY)
        env:
        - name: ALT_GREETING
          valueFrom:
            configMapKeyRef:
              key: altGreeting
              name: production-the-map
        - name: ENABLE_RISKY
          valueFrom:
            configMapKeyRef:
              key: enableRisky
              name: production-the-map
        image: monopole/hello:1
        name: the-container
        ports:
        - containerPort: 8080

```
### 4.4 部署
现在我们可以部署到kubernetes集群中:

```bash
kustomize build $OVERLAYS/staging |\
    kubectl apply -f -

kustomize build $OVERLAYS/production |\
    kubectl apply -f -
```

```bash
$ kubectl get pod
NAME                                        READY   STATUS              RESTARTS   AGE
production-the-deployment-bb7fd8b65-5g9jz   0/1     ContainerCreating   0          7s
production-the-deployment-bb7fd8b65-cvj6p   0/1     ContainerCreating   0          7s
production-the-deployment-bb7fd8b65-gp7cx   0/1     ContainerCreating   0          7s
production-the-deployment-bb7fd8b65-hdxjh   0/1     ContainerCreating   0          7s
production-the-deployment-bb7fd8b65-lfpzn   0/1     ContainerCreating   0          7s
production-the-deployment-bb7fd8b65-ngrx9   0/1     ContainerCreating   0          7s
production-the-deployment-bb7fd8b65-rmz2j   0/1     ContainerCreating   0          7s
production-the-deployment-bb7fd8b65-wj2fs   0/1     ContainerCreating   0          7s
production-the-deployment-bb7fd8b65-xw4k4   1/1     Running             0          7s
production-the-deployment-bb7fd8b65-zgmnr   0/1     ContainerCreating   0          7s
staging-the-deployment-6cd65cfc7f-57p4c     1/1     Running             0          22s
staging-the-deployment-6cd65cfc7f-mmx8r     1/1     Running             0          22s
staging-the-deployment-6cd65cfc7f-xgx6n     1/1     Running             0          22s
the-deployment-7757d64b69-7c98r             1/1     Running             0          24m
the-deployment-7757d64b69-lmvkv             1/1     Running             0          24m
the-deployment-7757d64b69-vwh8g             1/1     Running             0          24m
```
### 4.5 删除
```bash
kustomize build $OVERLAYS/staging |\
    kubectl delete -f -

kustomize build $OVERLAYS/production |\
    kubectl delete -f -
```
参考资料：
[https://cloud-atlas.readthedocs.io/zh_CN/latest/kubernetes/deployment/kustomize.html](https://cloud-atlas.readthedocs.io/zh_CN/latest/kubernetes/deployment/kustomize.html)

扩展阅读：

 - [kustomize (一) 管理yaml部署入门hello world](https://ghostwritten.blog.csdn.net/article/details/107925618)
 - [kustomize (二) ConfigMap的生成和滚动更新](https://ghostwritten.blog.csdn.net/article/details/110962982)
 - [kustomize (三) devops和开发配合管理配置数据behavior: merge、namePrefix、nameSuffix](https://ghostwritten.blog.csdn.net/article/details/110980010)
 - [kustomize (四) generatorOptions详解](https://ghostwritten.blog.csdn.net/article/details/110992002)
 - [kustomize (五) 使用vars将 k8s runtime数据注入容器](https://ghostwritten.blog.csdn.net/article/details/111029759)
 - [kustomize(六)命令行常用编排](https://ghostwritten.blog.csdn.net/article/details/111042577)
 - [kustomize (七)patches、patchesJson6902、patchesStrategicMerge详解](https://ghostwritten.blog.csdn.net/article/details/111188370)
 - [kustomize (八)生成secret](https://ghostwritten.blog.csdn.net/article/details/111211735)
 - [kustomize(九)使用终章](https://blog.csdn.net/xixihahalelehehe/article/details/111223923)
