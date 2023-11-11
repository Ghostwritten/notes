![在这里插入图片描述](https://img-blog.csdnimg.cn/20201214235621361.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)

---

## 1. patches介绍
kustomization.yaml 支持通过 [Strategic Merge Patch](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-api-machinery/strategic-merge-patch.md) 和 [JSON patch](https://ghostwritten.blog.csdn.net/article/details/111047607)来自定义资源。自 3.1.0 起，一个 patch 可以修改多个资源。


这可以通过指定 patch 和它所修改的 target 来完成，如下所示：

```bash
patches:
- path: <PatchFile>
  target:
    group: <Group>
    version: <Version>
    kind: <Kind>
    name: <Name>
    namespace: <Namespace>
    labelSelector: <LabelSelector>
    annotationSelector: <AnnotationSelector>
```

|
op: [add,replace]

替换 /新增的方式有三种
```bash
- op: replace
path: /metadata/name
value: beautiful-country-bigdata
```

`labelSelector` 和 `annotationSelector` 都应遵循 `label selector` 中的约定。Kustomize 选择匹配target中所有字段的目标来应用 patch 。

### 1.1 patches示例

```bash
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
patches:
- path: patch.yaml
  target:
    group: apps
    version: v1
    kind: Deployment
    name: deploy.*
    labelSelector: "env=dev"
    annotationSelector: "zone=west"
- patch: |-
    - op: replace
      path: /some/existing/path
      value: new value
  target:
    kind: MyKind
    labelSelector: "env=dev"
```

## 2. patches添加
下面的示例展示了如何为所有部署资源注入 sidecar 容器。

创建一个包含 Deployment 资源的 kustomization 。

```bash
DEMO_HOME=$(mktemp -d)

cat <<EOF >$DEMO_HOME/kustomization.yaml
resources:
- deployments.yaml
EOF

cat <<EOF >$DEMO_HOME/deployments.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy1
spec:
  template:
    metadata:
      labels:
        old-label: old-value
    spec:
      containers:
        - name: nginx
          image: nginx
          args:
          - one
          - two
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy2
spec:
  template:
    metadata:
      labels:
        key: value
    spec:
      containers:
        - name: busybox
          image: busybox
EOF
```

声明 Strategic Merge Patch 文件以注入 `sidecar` 容器：

```bash
cat <<EOF >$DEMO_HOME/patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: not-important
spec:
  template:
    spec:
      containers:
        - name: istio-proxy
          image: docker.io/istio/proxyv2
          args:
          - proxy
          - sidecar
EOF
```

在 kustomization.yaml 中添加 patches 字段

```bash
cat <<EOF >>$DEMO_HOME/kustomization.yaml
patches:
- path: patch.yaml
  target:
    kind: Deployment
EOF
```

运行 `kustomize build $DEMO_HOME`，可以在输出中确认两个 Deployment 资源都已正确应用。

```bash
test 2 ==  $(kustomize build $DEMO_HOME | grep "image: docker.io/istio/proxyv2" | wc -l);  echo $?
```

输出如下：

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy1
spec:
  template:
    metadata:
      labels:
        old-label: old-value
    spec:
      containers:
      - args:
        - proxy
        - sidecar
        image: docker.io/istio/proxyv2
        name: istio-proxy
      - args:
        - one
        - two
        image: nginx
        name: nginx
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy2
spec:
  template:
    metadata:
      labels:
        key: value
    spec:
      containers:
      - args:
        - proxy
        - sidecar
        image: docker.io/istio/proxyv2
        name: istio-proxy
      - image: busybox
        name: busybox
```

## 3. patches替换
### 3.1 通过Kustomization 直接编写替换/新增资源清单属性
`deployment.yaml`和`service.yaml`

```bash
cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: java
  name: java
spec:
  selector:
    matchLabels:
      app: java
  template:
    metadata:
      labels:
        app: java
    spec:
      containers:
      - image: java
        name: java
        ports:
        - containerPort: 8080
          name: web
EOF
```

```bash
cat <<EOF > service.yaml
apiVersion: v1
kind: Service
metadata:
  name: java
spec:
  selector:
    app: java
  ports:
  - name: http
    port: 8001
    targetPort: 8001
EOF
```

```bash
cat <<EOF > kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- ./deployment.yaml
- ./service.yaml
commonLabels:
  app: bigdata
images:
- name: java
  newName: registry.cn-qingdao.aliyuncs.com/nqkj-snapshot/sky-bigdata
  newTag: develop-be43cc32
patches:
- patch: |
    - op: replace
      path: /metadata/name
      value: bigdata
    - op: replace
      path: /spec/template/spec/containers/0/name
      value: bigdata
    - op: replace
      path: /spec/template/spec/containers/0/ports/0/containerPort
      value: 8001
  target:
    group: apps
    kind: Deployment
    version: v1
- patch: |
    - op: replace
      path: /metadata/name
      value: bigdata
  target:
    kind: Service
EOF
```
运行`kubectl kustomize ./`

```bash
apiVersion: v1
kind: Service
metadata:
  labels:
    app: bigdata
  name: bigdata
spec:
  ports:
  - name: http
    port: 8001
    targetPort: 8001
  selector:
    app: bigdata
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: bigdata
  name: bigdata
spec:
  selector:
    matchLabels:
      app: bigdata
  template:
    metadata:
      labels:
        app: bigdata
    spec:
      containers:
        image: registry.cn-qingdao.aliyuncs.com/nqkj-snapshot/sky-bigdata:develop-ad8ed411
        name: bigdata
        ports:
        - containerPort: 8001
          name: web
```
### 3.2 如果通过.yaml 文件进行修改/新增资源清单属性
准备patch.yaml,内容如下

```bash
cat <<EOF>patch.yaml
- op: replace
  path: /metadata/name
  value: oauth-server
- op: replace
  path: /spec/template/spec/containers/0/name
  value: oauth-server
EOF
```

```bash
cat <<EOF > kustomization.yaml
resources:
- ./deployment.yaml
- ./service.yaml
commonLabels:  # 标签
  app: oauth-server
images:
- name: java
  newName: my-registry/my-postgres
patches:
  - path: patch.yaml
    target:
      group: apps
      kind: Deployment
      version: v1
EOF
```
kustomize查看编排结果

```bash
$ kustomize build .
apiVersion: v1
kind: Service
metadata:
  labels:
    app: oauth-server
  name: java
spec:
  ports:
  - name: http
    port: 8001
    targetPort: 8001
  selector:
    app: oauth-server
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: oauth-server
  name: oauth-server
spec:
  selector:
    matchLabels:
      app: oauth-server
  template:
    metadata:
      labels:
        app: oauth-server
    spec:
      containers:
      - image: my-registry/my-postgres
        name: oauth-server
        ports:
        - containerPort: 8080
          name: web
```
## 4. patchesJson6902

通过json 文件的方式替换/新增资源清单的属性值
patchesJson6902 不光可以使用json文件也是可以使用yaml文件与上面的patches如出一辙

> 注意： 在通过target: 匹配资源清单的同时必须加入target>name: 属性，属性值为Deployment> 的name，这是与patches的区别

我们还是使用patches所使用的deployment.yaml和service.yaml 作为基础模版文件

准备 `patch.json` 和 `patch-svc.json`

```bash
cat <<EOF >patch.json
[
    { "op": "replace", "path": "/metadata/name", "value": "oauth-server" },
    { "op": "add", "path": "/spec/template/spec/containers/0/name", "value": "oauth-server"}
     
]
EOF
```

```bash
cat <<EOF >patch-svc.json
[
  { "op": "replace", "path": "/metadata/name", "value": "oauth-server" }
]
EOF
```

```bash
cat <<EOF > kustomization.yaml
resources:
  - ../../../template
commonLabels:  # 标签
  app: oauth-server
images:
- name: java
  newName: my-registry/my-postgres
patchesJson6902:
  - path: patch.json
    target:
      group: apps
      kind: Deployment
      version: v1
      name: java
  - path: patch-svc.json # 指定json
    target:
      version: v1
      kind: Service
      name: java
EOF
```
kustomize查看编排结果

```bash
$ kustomize build .
apiVersion: v1
kind: Service
metadata:
  labels:
    app: oauth-server
  name: oauth-server
spec:
  ports:
  - name: http
    port: 8001
    targetPort: 8001
  selector:
    app: oauth-server
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: oauth-server
  name: oauth-server
spec:
  selector:
    matchLabels:
      app: oauth-server
  template:
    metadata:
      labels:
        app: oauth-server
    spec:
      containers:
      - image: my-registry/my-postgres
        name: oauth-server
        ports:
        - containerPort: 8080
          name: web
```
## 5. patchesStrategicMerge
通过.yaml 文件的方式为要生成的资源清单定义补丁

> 注意补丁的yaml 文件的name,要跟模版清单一致，下面是deployment.yaml模版清单，increase_replicas.yaml和set_memory.yaml为补丁。在kustomization声明引用补丁文件，最后合并成带补丁的资源清单。

```bash
cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
EOF
# Create a patch increase_replicas.yaml
cat <<EOF > increase_replicas.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 3
EOF

# Create another patch set_memory.yaml
cat <<EOF > set_memory.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  template:
    spec:
      containers:
      - name: my-nginx
        resources:
        limits:
          memory: 512Mi
EOF

cat <<EOF >./kustomization.yaml
resources:
- deployment.yaml
patchesStrategicMerge:
- increase_replicas.yaml
- set_memory.yaml
EOF
```
kustomize查看编排结果
```bash
$ kustomize build .
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      run: my-nginx
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - image: nginx
        limits:
          memory: 512Mi
        name: my-nginx
        ports:
        - containerPort: 80
```

## 6. Target 选择

选择名称与 name* 匹配的资源

```bash
target:
  name: name*
```

选择所有 Deployment 资源

```bash
target:
  kind: Deployment
```

选择 label 与 app=hello 匹配的资源

```bash
target:
  labelSelector: app=hello
```

选择 annotation 与 app=hello 匹配的资源

```bash
target:
  annotationSelector: app=hello
```

选择所有 label 与 app=hello 匹配的 Deployment 资源

```bash
target:
  kind: Deployment
  labelSelector: app=hello
```
更多细节：
[https://github.com/kubernetes/community/blob/master/contributors/devel/sig-api-machinery/strategic-merge-patch.md](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-api-machinery/strategic-merge-patch.md)

[kustomize API 使用手册](https://blog.csdn.net/weixin_37546425/article/details/109214976)


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



