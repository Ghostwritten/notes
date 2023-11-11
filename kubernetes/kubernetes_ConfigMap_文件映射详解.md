#  kubernetes ConfigMap
tags: ConfigMap,对象
<!-- catalog: ~ConfigMap~ -->


[![在这里插入图片描述](https://img-blog.csdnimg.cn/83cf609310b6438390323994a4ed4dc2.png)](https://www.rottentomatoes.com/m/1084398-life_is_beautiful)

*“早安！公主！”          ——《美丽人生》*


## 1. 简介
ConfigMap 是一种 API 对象，用来将非机密性的数据保存到健值对中。使用时可以用作环境变量、命令行参数或者存储卷中的配置文件。

ConfigMap 将您的环境配置信息和 容器镜像 解耦，便于应用配置的修改。当您需要储存机密信息时可以使用 Secret 对象。

**注意**：
ConfigMap 并不提供保密或者加密功能。如果你想存储的数据是机密的，请使用 Secret ，或者使用其他第三方工具来保证你的数据的私密性，而不是用 ConfigMap。


四种方式来使用 ConfigMap 配置 Pod 中的容器：

 - 容器 entrypoint 的命令行参数
 - 容器的环境变量
 - 在只读卷里面添加一个文件，让应用来读取
 - 编写代码在 Pod 中运行，使用 Kubernetes API 来读取 ConfigMap

## 2. 创建configmap
`kubectl create configmap` 或者在 `kustomization.yaml` 中的 ConfigMap 生成器 来创建 ConfigMap。注意，kubectl 从 1.14 版本开始支持 kustomization.yaml。

格式：

```bash
kubectl create configmap <map-name> <data-source>
```
<map-name> 是要设置的 ConfigMap 名称，<data-source> 是要从中提取数据的目录、 文件或者字面值。 ConfigMap 对象的名称必须是合法的 DNS 子域名.

### 2.1 --from-file
**第一种方法**
```bash
# 创建本地目录
mkdir -p configure-pod-container/configmap/

# 将实例文件下载到 `configure-pod-container/configmap/` 目录
wget https://kubernetes.io/examples/configmap/game.properties -O configure-pod-container/configmap/game.properties
wget https://kubernetes.io/examples/configmap/ui.properties -O configure-pod-container/configmap/ui.properties

# 创建 configmap
kubectl create configmap game-config --from-file=configure-pod-container/configmap/
或者
kubectl create configmap game-config-2 --from-file=configure-pod-container/configmap/game.properties --from-file=configure-pod-container/configmap/ui.properties
```

**第二种方法**

```bash
$ kubectl create configmap game-config-3 --from-file=<my-key-name>=<path-to-file>
```
<my-key-name> 是你要在 ConfigMap 中使用的键名，<path-to-file> 是你想要键表示数据源文件的位置

```bash
$ kubectl create configmap game-config-3 --from-file=game-special-key=configure-pod-container/configmap/game.properties

$ kubectl get configmaps game-config-3 -o yaml
apiVersion: v1
kind: ConfigMap
metadata:
  creationTimestamp: 2016-02-18T18:54:22Z
  name: game-config-3
  namespace: default
  resourceVersion: "530"
  selfLink: /api/v1/namespaces/default/configmaps/game-config-3
  uid: 05f8da22-d671-11e5-8cd0-68f728db1985
data:
  game-special-key: |
    enemies=aliens
    lives=3
    enemies.cheat=true
    enemies.cheat.level=noGoodRotten
    secret.code.passphrase=UUDDLRLRBABAS
    secret.code.allowed=true
    secret.code.lives=30
```

### 2.2  --from-env-file

```bash
$ cat configure-pod-container/configmap/game-env-file.properties
enemies=aliens
lives=3
allowed="true"
```

```bash
$ kubectl create configmap game-config-env-file \
       --from-env-file=configure-pod-container/configmap/game-env-file.properties

$ kubectl get configmap game-config-env-file -o yaml
apiVersion: v1
kind: ConfigMap
metadata:
  creationTimestamp: 2017-12-27T18:36:28Z
  name: game-config-env-file
  namespace: default
  resourceVersion: "809965"
  selfLink: /api/v1/namespaces/default/configmaps/game-config-env-file
  uid: d9d1ca5b-eb34-11e7-887b-42010a8002b8
data:
  allowed: '"true"'
  enemies: aliens
  lives: "3"
```

### 2.3 --from-literal

kubectl create configmap 与 --from-literal 参数一起使用，从命令行定义文字值:

```bash
$ kubectl create configmap special-config --from-literal=special.how=very --from-literal=special.type=charm
```

```bash
$ kubectl get configmaps special-config -o yaml
apiVersion: v1
kind: ConfigMap
metadata:
  creationTimestamp: 2016-02-18T19:14:38Z
  name: special-config
  namespace: default
  resourceVersion: "651"
  selfLink: /api/v1/namespaces/default/configmaps/special-config
  uid: dadce046-d673-11e5-8cd0-68f728db1985
data:
  special.how: very
  special.type: charm
```
## 3. 基于生成器创建 ConfigMap
自 1.14 开始，kubectl 开始支持 kustomization.yaml。 你还可以基于生成器创建 ConfigMap，然后将其应用于 API 服务器上创建对象。 生成器应在目录内的 kustomization.yaml 中指定。

创建包含 ConfigMapGenerator 的 kustomization.yaml 文件
```bash
cat <<EOF >./kustomization.yaml
configMapGenerator:
- name: game-config-4
  files:
  - configure-pod-container/configmap/kubectl/game.properties
EOF
```
使用 kustomization 目录创建 ConfigMap 对象：

```bash
$ kubectl apply -k .
```

### 3.1 定义从文件生成 ConfigMap 时要使用的键
在 ConfigMap 生成器，你可以定义一个非文件名的键名。 例如，从 configure-pod-container/configmap/game.properties 文件生成 ConfigMap， 但使用 game-special-key 作为键名：

创建包含 ConfigMapGenerator 的 kustomization.yaml 文件

```bash
cat <<EOF >./kustomization.yaml
configMapGenerator:
- name: game-config-5
  files:
  - game-special-key=configure-pod-container/configmap/kubectl/game.properties
EOF
```

### 3.2 从字面值生成 ConfigMap
要基于字符串 special.type=charm 和 special.how=very 生成 ConfigMap， 可以在 kusotmization.yaml 中配置 ConfigMap 生成器：

创建带有 ConfigMapGenerator 的 kustomization.yaml 文件

```bash
cat <<EOF >./kustomization.yaml
configMapGenerator:
- name: special-config-2
  literals:
  - special.how=very
  - special.type=charm
EOF
```
## 4. configmap与pod
### 4.1  使用 ConfigMap 数据定义容器环境变量
在 ConfigMap 中将环境变量定义为键值对:

```bash
kubectl create configmap special-config --from-literal=special.how=very
```

将 ConfigMap 中定义的 special.how 值分配给 Pod 规范中的 SPECIAL_LEVEL_KEY 环境变量

```bash
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      env:
        # Define the environment variable
        - name: SPECIAL_LEVEL_KEY
          valueFrom:
            configMapKeyRef:
              # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
              name: special-config
              # Specify the key associated with the value
              key: special.how
  restartPolicy: Never
```
创建 Pod:

```bash
kubectl create -f https://kubernetes.io/examples/pods/pod-single-configmap-env-variable.yaml
```

现在，Pod 的输出包含环境变量 `SPECIAL_LEVEL_KEY=very`。

### 4.2 使用来自多个 ConfigMap 的数据定义容器环境变量

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
  namespace: default
data:
  special.how: very
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: env-config
  namespace: default
data:
  log_level: INFO
```

```bash
kubectl create -f https://kubernetes.io/examples/configmap/configmaps.yaml
```

```bash
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      env:
        - name: SPECIAL_LEVEL_KEY
          valueFrom:
            configMapKeyRef:
              name: special-config
              key: special.how
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: env-config
              key: log_level
  restartPolicy: Never
```

```bash
kubectl create -f https://kubernetes.io/examples/pods/pod-multiple-configmap-env-variable.yaml
```

现在，Pod 的输出包含环境变量 SPECIAL_LEVEL_KEY=very 和 LOG_LEVEL=INFO


### 4.3  将 ConfigMap 中的所有键值对配置为容器环境变量
pods/pod-configmap-envFrom.yaml 
```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
  namespace: default
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
```

```bash
kubectl create -f https://kubernetes.io/examples/configmap/configmap-multikeys.yaml
```
使用 envFrom 将所有 ConfigMap 的数据定义为容器环境变量，ConfigMap 中的键成为 Pod 中的环境变量名称。

pods/pod-configmap-envFrom.yaml 

```bash
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      envFrom:
      - configMapRef:
          name: special-config
  restartPolicy: Never
```

```bash
kubectl create -f https://kubernetes.io/examples/pods/pod-configmap-envFrom.yaml
```
### 4.4  将 ConfigMap 数据添加到一个卷中 
如基于文件创建 ConfigMap](#create-configmaps-from-files) 中所述，当你使用 --from-file 创建 ConfigMap 时，文件名成为存储在 ConfigMap 的 data 部分中的键， 文件内容成为键对应的值。
configmap/configmap-multikeys.yaml apiVersion: v1

```bash
kind: ConfigMap
metadata:
  name: special-config
  namespace: default
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
```

```bash
kubectl create -f https://kubernetes.io/examples/configmap/configmap-multikeys.yaml
```
#### 4.4.1  使用存储在 ConfigMap 中的数据填充数据卷
在 Pod 规约的 volumes 部分下添加 ConfigMap 名称。 这会将 ConfigMap 数据添加到指定为 `volumeMounts.mountPath` 的目录（在本例中为 /etc/config）。 command 部分引用存储在 ConfigMap 中的 `special.level`。
pods/pod-configmap-volume.yaml 

```bash
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "ls /etc/config/" ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        # Provide the name of the ConfigMap containing the files you want
        # to add to the container
        name: special-config
  restartPolicy: Never
```

```bash
kubectl create -f https://kubernetes.io/examples/pods/pod-configmap-volume.yaml
```

Pod 运行时，命令 ls /etc/config/ 产生下面的输出：

```bash
SPECIAL_LEVEL
SPECIAL_TYPE
```
**注意**： 如果在 /etc/config/ 目录中有一些文件，它们将被删除。


#### 4.4.2  将 ConfigMap 数据添加到数据卷中的特定路径
使用 path 字段为特定的 ConfigMap 项目指定预期的文件路径。 在这里，SPECIAL_LEVEL 将挂载在 config-volume 数据卷中 /etc/config/keys 目录下。
pods/pod-configmap-volume-specific-key.yaml 

```bash
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh","-c","cat /etc/config/keys" ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: special-config
        items:
        - key: SPECIAL_LEVEL
          path: keys
  restartPolicy: Never
```

创建Pod：

```bash
kubectl create -f https://kubernetes.io/examples/pods/pod-configmap-volume-specific-key.yaml
```

当 pod 运行时，命令 cat /etc/config/keys 产生以下输出：

```bash
very
```
#### 4.4.3 将 ConfigMap 数据添加到数据卷中的特定文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201125210939982.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020112521095867.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020112521110166.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201125211124301.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)


### 4.6 pod引用configmap示例


```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
data:
  # 类属性键；每一个键都映射到一个简单的值
  # 使用 --from-literal 定义的简单属性
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"
  #
  # 类文件键
  # 使用 --from-file 定义复杂属性的例子
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true
```

```bash
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo-pod
spec:
  containers:
    - name: demo
      image: game.example/demo-game
      env:
        # 定义环境变量
        - name: PLAYER_INITIAL_LIVES # 请注意这里和 ConfigMap 中的键名是不一样的
          valueFrom:
            configMapKeyRef:
              name: game-demo           # 这个值来自 ConfigMap
              key: player_initial_lives # 需要取值的键
        - name: UI_PROPERTIES_FILE_NAME
          valueFrom:
            configMapKeyRef:
              name: game-demo
              key: ui_properties_file_name
      volumeMounts:
      - name: config
        mountPath: "/config"
        readOnly: true
  volumes:
    # 您可以在 Pod 级别设置卷，然后将其挂载到 Pod 内的容器中
    - name: config
      configMap:
        # 提供你想要挂载的 ConfigMap 的名字
        name: game-demo
```

## 5. 使用ConfigMap的限制条件
使用ConfigMap的限制条件如下。

 - ◎ ConfigMap必须在Pod之前创建。
 - ◎ ConfigMap受Namespace限制，只有处于相同Namespace中的Pod才可以引用它。
 - ◎ ConfigMap中的配额管理还未能实现。
 - ◎ kubelet只支持可以被API Server管理的Pod使用ConfigMap。kubelet在本Node上通过 `--manifest-ur`l或`--config`自动创建的静态Pod将无法引用ConfigMap。
◎ 在Pod对ConfigMap进行挂载（volumeMount）操作时，在容器内部只能挂载为“目录”，无法挂载为“文件”。在挂载到容器内部后，在目录下将包含ConfigMap定义的每个item，如果在该目录下原来还有其他文件，则容器内的该目录将被挂载的ConfigMap覆盖。如果应用程序需要保留原来的其他文件，则需要进行额外的处理。可以将ConfigMap挂载到容器内部的临时目录，再通过启动脚本将配置文件复制或者链接到（cp或link命令）应用所用的实际配置目录下。

参考：

 - [kubernetes pod configmap](https://kubernetes.io/zh/docs/tasks/configure-pod-container/configure-pod-configmap/)
 - [google cloud 创建 ConfigMap](https://cloud.google.com/kubernetes-engine/docs/concepts/configmap)
 - [ConfigMaps in Kubernetes (K8s)](https://medium.com/avmconsulting-blog/configmaps-in-kubernetes-k8s-2f2ce79d7e66)

