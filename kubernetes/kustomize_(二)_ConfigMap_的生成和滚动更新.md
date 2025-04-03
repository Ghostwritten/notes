![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4f8130c365b7034b91a871d39810a3f8.png#pic_center)

----

## 1.ConfigMap声明方法
kustomize 提供了两种添加 ConfigMap 的方法：

 - 将 ConfigMap 声明为 resource
 - 通过 `ConfigMapGenerator` 声明 ConfigMap

在 kustomization.yaml 中，这两种方法的格式分别如下：

```bash
# 将 ConfigMap 声明为 resource
resources:
- configmap.yaml

# 在 ConfigMapGenerator 中声明 ConfigMap
configMapGenerator:
- name: a-configmap
  files:
    - configs/configfile
    - configs/another_configfile
```

**声明为 resource 的 ConfigMaps 的处理方式与其他 resource 相同，Kustomize 不会在为 ConfigMap 的名称添加哈希后缀。而在 ConfigMapGenerator 中声明 ConfigMap 的处理方式则与之前不同，默认将为名称添加哈希后缀，ConfigMap 中的任何更改都将触发滚动更新。**

## 2.修改base添加 ConfigMapGenerator字面值的configmap
在 [hello_world](https://ghostwritten.blog.csdn.net/article/details/107925618) 示例中，使用 ConfigmapGenerator 来替换将 ConfigMap 声明为 resource 的方法。由此生成的 ConfigMap 中的更改将导致哈希值更改和滚动更新
**注意：在helloworld运用的文件基础上修改**

```bash
cat <<'EOF' >$BASE/kustomization.yaml
commonLabels:
  app: hello
resources:
- deployment.yaml
- service.yaml
configMapGenerator:
- name: the-map
  literals:
    - altGreeting=Good Morning!
    - enableRisky="false"
EOF
```
## 2. staging创建
通过应用 ConfigMap patch 的方式建立 staging

```bash
#!/bin/bash
export OVERLAYS=$DEMO_HOME/overlays
mkdir -p $OVERLAYS/staging

cat <<'EOF' >$OVERLAYS/staging/kustomization.yaml
namePrefix: staging-
nameSuffix: -v1
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

## 3. configmap变动分析
在集群中运行的 hello-world 的 deployment 配置了来自 configMap 的数据。

deployment 按照名称引用此 ConfigMap ：

```bash
$ grep -C 2 configMapKeyRef $BASE/deployment.yaml
       - name: ALT_GREETING
          valueFrom:
            configMapKeyRef:
              name: the-map
              key: altGreeting
        - name: ENABLE_RISKY
          valueFrom:
            configMapKeyRef:
              name: the-map
              key: enableRisky

```

当 ConfigMap 中的数据需要更新时，更改群集中的实时 ConfigMap 的数据并不是一个好的做法。 由于 Deployment 无法知道其引用的 ConfigMap 已更改，这类更新是无效。

更改 Deployment 配置的推荐方法是：

 - 使用新名称创建一个新的 configMap
 - 为deployment 添加 patch，修改相应 configMapKeyRef 字段的名称值。

后一种更改会启动对 deployment 中的 pod 的滚动更新。旧的 configMap 在不再被任何其他资源引用时最终会被[垃圾回收](https://github.com/kubernetes-sigs/kustomize/issues/242)。

## 4. 如何使用 kustomize支持ConfigMap 的生成和滚动更新
staging 的 variant 包含一个 configMap 的 patch：

```bash
$ cat $OVERLAYS/staging/map.yaml
piVersion: v1
kind: ConfigMap
metadata:
  name: the-map #名字要与base目录中kustomization.yaml的configMapGenerator的名字一至
data:
  altGreeting: "Have a pineapple!"
  enableRisky: "true"
```

根据定义，此 patch 是一个命名但不一定是完整的资源规范，旨在修改完整的资源规范。

在 ConfigMapGenerator 中声明 ConfigMap 的修改。

```bash
$ grep -C 4 configMapGenerator $BASE/kustomization.yaml
  app: hello
resources:
- deployment.yaml
- service.yaml
configMapGenerator:
- name: the-map
  literals:
    - altGreeting=Good Morning!
    - enableRisky="false"
```

要使这个 patch 正常工作，**metadata/name 字段中的名称必须匹配**。

**但是，文件中指定的名称值不是群集中使用的名称值。根据设计，kustomize 修改从 ConfigMapGenerator 声明的 ConfigMaps 的名称。要查看最终在群集中使用的名称**，只需运行 kustomize：

```bash
$ kustomize build $OVERLAYS/staging | grep -B 8 -A 1 staging-the-map
kind: ConfigMap
metadata:
  annotations:
    note: Hello, I am staging!
  labels:
    app: hello
    org: acmeCorporation
    variant: staging
  name: staging-the-map-hhhhkfmgmk  #新名字
---
--
        - /hello
        - --port=8080
        - --enableRiskyFeature=$(ENABLE_RISKY)
        env:
        - name: ALT_GREETING
          valueFrom:
            configMapKeyRef:
              key: altGreeting
              name: staging-the-map-hhhhkfmgmk #新名字
        - name: ENABLE_RISKY
          valueFrom:
            configMapKeyRef:
              key: enableRisky
              name: staging-the-map-hhhhkfmgmk #新名字
        image: monopole/hello:1
```

根据 `$OVERLAYS/staging/kustomization.yaml` 中的 `namePrefix` 字段，configMap 名称以 `staging-` 为前缀。

根据 `$OVERLAYS/staging/kustomization.yaml` 中的 `nameSuffix` 字段，configMap 名称以 `-v1` 为后缀。

configMap 名称的后缀是由 map 内容的哈希生成的 - 在这种情况下，名称后缀是 5276h4th55 ：

```bash
$ kustomize build $OVERLAYS/staging |grep k25m8k5k5m
  name: staging-the-map-v1-k25m8k5k5m
              name: staging-the-map-v1-k25m8k5k5m
              name: staging-the-map-v1-k25m8k5k5m

```
现在修改 **map patch** ，更改该服务将使用的问候消息：（**configmap名称也会随之统一修改**）

```bash
$ sed -i.bak 's/pineapple/kiwi/' $OVERLAYS/staging/map.yaml
```

查看新的问候消息：

```bash
$ kustomize build $OVERLAYS/staging | grep -B 2 -A 3 kiwi
apiVersion: v1
data:
  altGreeting: Have a kiwi!
  enableRisky: "true"
kind: ConfigMap
metadata:
```

再次运行 kustomize 查看新的 configMap 名称：

```bash
$ kustomize build $OVERLAYS/staging |    grep -B 8 -A 1 staging-the-map
kind: ConfigMap
metadata:
  annotations:
    note: Hello, I am staging!
  labels:
    app: hello
    org: acmeCorporation
    variant: staging
  name: staging-the-map-v1-cd7kdh48fd  #后缀发生改变
---
--
        - /hello
        - --port=8080
        - --enableRiskyFeature=$(ENABLE_RISKY)
        env:
        - name: ALT_GREETING
          valueFrom:
            configMapKeyRef:
              key: altGreeting
              name: staging-the-map-v1-cd7kdh48fd  #后缀发生改变
        - name: ENABLE_RISKY
          valueFrom:
            configMapKeyRef:
              key: enableRisky
              name: staging-the-map-v1-cd7kdh48fd #后缀发生改变
        image: monopole/hello:1
```
确认 configMap 内容的更改将会生成以 `cd7kdh48fd` 结尾的三个新名称 - 一个在 configMap 的名称中，另两个在使用 ConfigMap 的 deployment 中：

```bash
$ test 3 ==   $(kustomize build $OVERLAYS/staging | grep cd7kdh48fd | wc -l);   echo $?
0
```
将这些资源应用于群集将导致 deployment pod 的滚动更新，将它们从 `k25m8k5k5m` map 重新定位到 `cd7kdh48fd` map 。系统稍后将垃圾收集未使用的 map。

确保一至后进行部署即可。
```bash
kustomize build $OVERLAYS/staging | kubectl apply -f -
```

##  5. ConfigMapGenerator挂载文件
### 第一种方法file
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
kubectl apply -k .
```
### 第二种方法envs
```bash
configMapGenerator:
- envs:
  - env-file
  name: service1-env-vars
```
### 第三种方法（应对绝对路径）

```bash
configMapGenerator:
- name: etcd-env
  files:
  - /etc/etcd.env  #
```

```bash
$ kustomize build  .
Error: loading KV pairs: file sources: [/etc/etcd.env]: security; file '/etc/etcd.env' is not in or below '/root/gitlab/etcdbackup/base'
```
--load_restrictor=none参数解决

```bash
kustomize build --load_restrictor=none .
```

扩展阅读：

 - [Create environment variables from env file with Kustomize](https://github.com/kubernetes-sigs/kustomize/issues/2497)
 - [kustomize (一) 管理yaml部署入门hello world](https://ghostwritten.blog.csdn.net/article/details/107925618)
 - [kustomize (二) ConfigMap的生成和滚动更新](https://ghostwritten.blog.csdn.net/article/details/110962982)
 - [kustomize (三) devops和开发配合管理配置数据behavior: merge、namePrefix、nameSuffix](https://ghostwritten.blog.csdn.net/article/details/110980010)
 - [kustomize (四) generatorOptions详解](https://ghostwritten.blog.csdn.net/article/details/110992002)
 - [kustomize (五) 使用vars将 k8s runtime数据注入容器](https://ghostwritten.blog.csdn.net/article/details/111029759)
 - [kustomize(六)命令行常用编排](https://ghostwritten.blog.csdn.net/article/details/111042577)
 - [kustomize (七)patches、patchesJson6902、patchesStrategicMerge详解](https://ghostwritten.blog.csdn.net/article/details/111188370)
 - [kustomize (八)生成secret](https://ghostwritten.blog.csdn.net/article/details/111211735)
 - [kustomize(九)使用终章](https://blog.csdn.net/xixihahalelehehe/article/details/111223923)



