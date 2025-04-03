![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/162e9d827e44d6466c6bac95ac1f4c6a.png#pic_center)
---

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


---

## 1. kustomization API说明
| 字段                    | 类型                                     | 描述                                        |
|-----------------------|----------------------------------------|-------------------------------------------|
| bases                 | \[\]string                             | 此列表中的每个条目都应该解析为包含kustomization\.yaml文件的目录 |
| commonAnnotations     | map\[string\]string                    | 添加到所有资源的注解                                |
| commonLabels          | map\[string\]string                    | 要添加到所有资源和选择器的标签                           |
| components            |                                        | 创造中，未来会发布                                 |
| configMapGenerator    | \[\]\[ConfigMapArgs\]\[\]ConfigMapArgs | 此列表中的每个条目都生成一个ConfigMap                   |
| crds                  | \[\]string                             | 增加对 CRD 的支持。                              |
| generatorOptions      | generatorOptions                       | 修改生成所有ConfigMap和Secret Generator的行为       |
| images                |                                        | 修改镜像的名称、tag 或 image digest。               |
| namePrefix            | string                                 | 此字段的值为所有资源的名称前缀                           |
| namespace             | string                                 | 为所有资源添加 namespace。                        |
| nameSuffix            | string                                 | 为所有资源和引用的名称添加后缀。                          |
| patches               | \[\]string                             | Patches 在资源上添加或覆盖字段，非常实用                  |
| patchesJson6902       | json                                   | 通过json格式生成kubernetes 对象类型与patches         |
| patchesStrategicMerge | \[\]string                             | 通过文件定义的方式进行增加补丁或者修改字段 属性                  |
| replicas              | string                                 | 修改资源的副本数。                                 |
| resources             | \[\]string                             | 每个条目必须解析为现有的资源配置文件                        |
| secretGenerator       | secretGenerator                        | 生成 Secret 资源                              |
| vars                  | \[\]string                             | 通过变量引用其他文件的资源和属性\-反射功能                    |

## 2. bases
bases在v2.1.0中已弃用该字段

bases移到resonrces字段中。这使得基础（仍然是一个资源中心概念）相对于其他输入资源进行排序。

## 3. commonAnnotations
为所有资源添加注释，如果资源上已经存在注解键，该值将被覆盖。

```bash
cat <<EOF >./deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
EOF

cat <<EOF >./kustomization.yaml
namespace: my-namespace
commonAnnotations:
  oncallPager: 800-555-1212
resources:
- deployment.yaml
EOF
```
运行`kubectl kustomize ./`查看Deployment资源中设置的所有字段:

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    oncallPager: 800-555-1212
  labels:
    app: bingo
  name: dev-nginx-deployment-001
  namespace: my-namespace
spec:
....
```
## 4. commonLabels
为所有资源和 selectors 增加标签。如果资源上已经存在注解键，该值将被覆盖。

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
# Create a kustomization.yaml
cat <<EOF >./kustomization.yaml
namePrefix: dev-
commonLabels:
  app: my-nginx
resources:
- deployment.yaml
EOF
```
运行`kubectl kustomize ./`查看Deployment资源中设置的所有字段:

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
      app: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
        app: my-nginx
    spec:
      ....
```
## 5. images
修改镜像的名称、tag 或 image digest。

通过在kustomize .yaml的images字段中指定新的镜像来更改容器内使用的镜像。

```bash
cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  template:
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
EOF

cat <<EOF >./kustomization.yaml
resources:
- deployment.yaml
images:
- name: nginx
  newName: my.image.registry/nginx
  newTag: 1.4.0
EOF
```
运行kubectl kustomize ./查看正在使用的镜像是否更新:

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  template:
    spec:
      containers:
      - image: my.image.registry/nginx:1.4.0
        name: my-nginx
        ports:
        - containerPort: 80
```

## 6. namePrefix
为所有资源和引用的名称添加前缀。

> 服务名称可能会发生更改。不建议在命令参数中硬编码服务名称。对于这种用法，Kustomize可以通过vars将服务名称注入容器

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
        command: ["start", "--host", "\$(MY_SERVICE_NAME)"]
EOF

# Create a service.yaml file
cat <<EOF > service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    run: my-nginx
EOF

cat <<EOF >./kustomization.yaml
namePrefix: dev-
nameSuffix: "-001"

resources:
- deployment.yaml
- service.yaml
```
运行kubectl kustomize ./查看注入容器的服务名称是dev-my-nginx-001:

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dev-my-nginx-001
spec:
  replicas: 2
  selector:
    matchLabels:
      run: my-nginx
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - command:
        - start
        - --host
        - dev-my-nginx-001
        image: nginx
        name: my-nginx
```
## 7. namespace
为所有资源添加 namespace。

如果在资源上设置了现有 namespace，则将覆盖现有 namespace；如果在资源上未设置现有 namespace，则使用现有 namespace

```bash
cat <<EOF >./deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
EOF

cat <<EOF >./kustomization.yaml
namespace: my-namespace
resources:
- deployment.yaml
EOF
```
运行kubectl kustomize ./查看Deployment

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: dev-nginx-deployment
  namespace: my-namespace
```
## 8. nameSuffix
为所有资源和引用的名称添加后缀。

```bash
cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
EOF

# Create a service.yaml file
cat <<EOF > service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
spec:
EOF

cat <<EOF >./kustomization.yaml
nameSuffix: "-001"
resources:
- deployment.yaml
- service.yaml
```
运行`kubectl kustomize ./`查看注入容器的服务名称是`my-nginx-001`:

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx-001
spec:

---
apiVersion: v1
kind: Service
metadata:
  name: my-nginx-001
spec:
```



参考链接：

 - [https://github.com/kubernetes-sigs/kustomize/blob/master/examples/zh/README.md](https://github.com/kubernetes-sigs/kustomize/blob/master/examples/zh/README.md)
 - [https://blog.csdn.net/weixin_37546425/article/details/109214976](https://blog.csdn.net/weixin_37546425/article/details/109214976)

