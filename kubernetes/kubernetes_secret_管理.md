#  kubernetes secret
tags:对象,secret



 - [云原生圣经](https://ghostwritten.blog.csdn.net/article/details/108562082)


[
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d4a2c4cc80e7d4e0e72c6134a79578ec.jpeg#pic_center)](https://www.behance.net/gallery/29646521/WATERDROP-Homage-to-The-Dark-Forest)

## 1. 概览 
Secret 是一种包含少量敏感信息例如密码、令牌或密钥的对象。 这样的信息可能会被放在 Pod 规约中或者镜像中。 用户可以创建 Secret，同时系统也创建了一些 Secret。

要使用 Secret，Pod 需要引用 Secret。 

### 1.1 Pod 使用 Secret三种方式

 - **作为挂载到一个或多个容器上的 卷 中的文件**。
 - **作为容器的环境变量**
 - **由 kubelet 在为 Pod 拉取镜像时使用**


###  1.2 secret 类型
创建 Secret 时，您可以使用typeSecret 资源的字段或某些等效的kubectl命令行标志（如果可用）指定其类型。在type一个秘密被用于促进各种机密数据的程序处理。
| 内置类型                                | 用法                         |
|-------------------------------------|----------------------------|
| Opaque                              | 任意用户定义数据                   |
| kubernetes.io/service-account-token | 服务帐号令牌                     |
| kubernetes.io/dockercfg             | 序列化~/.dockercfg文件          |
| kubernetes.io/dockerconfigjson      | 序列化~/.docker/config.json文件 |
| kubernetes.io/basic-auth            | 基本身份验证凭据                   |
| kubernetes.io/ssh-auth              | 用于 SSH 身份验证的凭据             |
| kubernetes.io/tls                   | TLS 客户端或服务器的数据             |
| bootstrap.kubernetes.io/token       | 引导令牌数据                     |



## 2. kubectl 创建 Secret 
使用generic 子命令来指示OpaqueSecret 类型

```bash
# 创建本例中要使用的文件
echo -n 'admin' > ./username.txt
echo -n '1f2d1e2e67df' > ./password.txt

#第一种
# kubectl create secret 命令将这些文件打包到一个 Secret 中并在 API server 中创建了一个对象。 Secret 对象的名称必须是合法的 DNS 子域名。
kubectl create secret generic db-user-pass --from-file=./username.txt --from-file=./password.txt

# 默认的键名是文件名。你也可以使用 [--from-file=[key=]source] 参数来设置键名。
#第二种
kubectl create secret generic db-user-pass \
  --from-file=username=./username.txt \
  --from-file=password=./password.txt

#第三种
kubectl create secret generic dev-db-secret \
  --from-literal=username=devuser \
  --from-literal=password='S!B\*d$zDsb='

kubectl get secrets
kubectl describe secrets/db-user-pass
```
**说明**：
特殊字符（例如 $、\、*、= 和 !）可能会被你的 Shell 解析，因此需要转义。 在大多数 Shell 中，对密码进行转义的最简单方式是使用单引号（'）将其扩起来。 
您无需对文件中保存（--from-file）的密码中的特殊字符执行转义操作。

## 3. 手动创建 Secret
### 3.1 data
```bash
$ echo -n 'admin' | base64
YWRtaW4=
$ echo -n '1f2d1e2e67df' | base64
MWYyZDFlMmU2N2Rm
```
编写一个yaml

```bash
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```

```bash
kubectl apply -f ./secret.yaml
```
在某些情况下，你可能希望改用 stringData 字段。 此字段允许您将非 base64 编码的字符串直接放入 Secret 中， 并且在创建或更新 Secret 时将为您编码该字符串。

###  3.2 stringData

```bash
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
stringData:
  config.yaml: |-
    apiUrl: "https://my.api.com/api/v1"
    username: {{username}}
    password: {{password}}
```

```bash
$ kubectl get secret mysecret -o yaml
apiVersion: v1
kind: Secret
metadata:
  creationTimestamp: 2018-11-15T20:40:59Z
  name: mysecret
  namespace: default
  resourceVersion: "7225"
  uid: c280ad2e-e916-11e8-98f2-025000000001
type: Opaque
data:
  config.yaml: YXBpVXJsOiAiaHR0cHM6Ly9teS5hcGkuY29tL2FwaS92MSIKdXNlcm5hbWU6IHt7dXNlcm5hbWV9fQpwYXNzd29yZDoge3twYXNzd29yZH19
```
### 3.3 data and stringdata
如果在 data 和 stringData 中都指定了某一字段，则使用 stringData 中的值:

```bash
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
stringData:
  username: administrator
```
secret 中的生成结果：

```bash
apiVersion: v1
kind: Secret
metadata:
  creationTimestamp: 2018-11-15T20:46:46Z
  name: mysecret
  namespace: default
  resourceVersion: "7579"
  uid: 91460ecb-e917-11e8-98f2-025000000001
type: Opaque
data:
  username: YWRtaW5pc3RyYXRvcg==
```
其中的 YWRtaW5pc3RyYXRvcg== 解码后即是 administrator。

data 和 stringData 的键必须由字母数字字符 '-', '_' 或者 '.' 组成.

## 4 从生成器创建 Secret 
Kubectl 从 1.14 版本开始支持使用 Kustomize 管理对象。 Kustomize 提供资源生成器创建 Secret 和 ConfigMaps。 Kustomize 生成器要在当前目录内的 kustomization.yaml 中指定。 生成 Secret 之后，使用 kubectl apply 在 API 服务器上创建对象。

### 4.1 从文件生成 Secret 

```bash
cat <<EOF >./kustomization.yaml
secretGenerator:
- name: db-user-pass
  files:
  - username.txt
  - password.txt
EOF
```

应用包含 kustomization.yaml 目录以创建 Secret 对象。

```bash
kubectl apply -k .
kubectl get secrets
```


输出类似于：

```bash
NAME                             TYPE                                  DATA      AGE
db-user-pass-96mffmfh4k          Opaque                                2         51s

$ kubectl describe secrets/db-user-pass-96mffmfh4k
Name:            db-user-pass
Namespace:       default
Labels:          <none>
Annotations:     <none>

Type:            Opaque

Data
====
password.txt:    12 bytes
username.txt:    5 bytes
```
### 4.2 基于字符串值来创建 Secret

```bash
cat <<EOF >./kustomization.yaml
secretGenerator:
- name: db-user-pass
  literals:
  - username=admin
  - password=secret
EOF
```

应用包含 kustomization.yaml 目录以创建 Secret 对象。

```bash
kubectl apply -k .
kubectl get secret mysecret -o yaml
```
## 5. 解码 Secret 
输出类似于：

```bash
apiVersion: v1
kind: Secret
metadata:
  creationTimestamp: 2016-01-22T18:41:56Z
  name: mysecret
  namespace: default
  resourceVersion: "164619"
  uid: cfee02d6-c137-11e5-8d73-42010af00002
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```
解码 password 字段：

```bash
echo 'MWYyZDFlMmU2N2Rm' | base64 --decode
输出类似于：

1f2d1e2e67df
```
## 6. 编辑 Secret

```bash
kubectl edit secrets mysecret
```

## 7. 使用 Secret
### 7.1 Pod Access Secrets Loaded in a Volume

 1. 创建一个 Secret 或者使用已有的 Secret。多个 Pod 可以引用同一个 Secret。
 2. 修改你的 Pod 定义，在 spec.volumes[] 下增加一个卷。可以给这个卷随意命名， 它的spec.volumes[].secret.secretName 必须是 Secret 对象的名字。
 3. 将 spec.containers[].volumeMounts[] 加到需要用到该 Secret 的容器中。 指定 spec.containers[].volumeMounts[].readOnly = true 和spec.containers[].volumeMounts[].mountPath 为你想要该 Secret 出现的尚未使用的目录。
 4. 修改你的镜像并且／或者命令行，让程序从该目录下寻找文件。 Secret 的 data 映射中的每一个键都对应 mountPath 下的一个文件名。
 

```bash
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
```
### 7.2 将 Secret 键名映射到特定路径
 你可以使用 spec.volumes[].secret.items 字段修改每个键对应的目标路径：

```bash
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      items:
      - key: username
        path: my-group/my-username
```

将会发生什么呢：

 - username Secret 存储在 /etc/foo/my-group/my-username 文件中而不是
   /etc/foo/username 中。
 - password Secret 没有被映射
### 7.3 Secret 文件权限 
你还可以指定 Secret 将拥有的权限模式位。如果不指定，默认使用 `0644`
示例：

```bash
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      defaultMode: 256
```

之后，Secret 将被挂载到 /etc/foo 目录，而所有通过该 Secret 卷挂载 所创建的文件的权限都是 `0400`。

你还可以使用映射，如上一个示例，并为不同的文件指定不同的权限，如下所示：

```bash
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      items:
      - key: username
        path: my-group/my-username
        mode: 511
```

在这里，位于 /etc/foo/my-group/my-username 的文件的权限值为 `0777`。 由于 JSON 限制，必须以十进制格式指定模式，即 511。

## 8. 挂载的 Secret 会被自动更新


当已经存储于卷中被使用的 Secret 被更新时，被映射的键也将终将被更新。 组件 kubelet 在周期性同步时检查被挂载的 Secret 是不是最新的。 但是，它会使用其本地缓存的数值作为 Secret 的当前值。

**FEATURE STATE: Kubernetes v1.18 [alpha]**
Kubernetes 的 alpha 特性 不可变的 Secret 和 ConfigMap 提供了一种可选配置， 可以设置各个 Secret 和 ConfigMap 为不可变的。 对于大量使用 Secret 的集群（至少有成千上万各不相同的 Secret 供 Pod 挂载）， 禁止变更它们的数据有下列好处：

防止意外（或非预期的）更新导致应用程序中断
通过将 Secret 标记为不可变来关闭 kube-apiserver 对其的监视，从而显著降低 kube-apiserver 的负载，提升集群性能。
使用这个特性需要启用 ImmutableEmphemeralVolumes 特性开关 并将 Secret 或 ConfigMap 的 immutable 字段设置为 true. 例如：

```bash
apiVersion: v1
kind: Secret
metadata:
  ...
data:
  ...
immutable: true
```

**说明**： 一旦一个 Secret 或 ConfigMap 被标记为不可变，撤销此操作或者更改 data 字段的内容都是 不 可能的。 只能删除并重新创建这个 Secret。现有的 Pod 将维持对已删除 Secret 的挂载点 - 建议重新创建这些 Pod。

## 9. 以环境变量的形式使用 Secrets

将 Secret 作为 Pod 中的环境变量使用：

 1. 创建一个 Secret 或者使用一个已存在的 Secret。多个 Pod 可以引用同一个 Secret。
 2. 修改 Pod 定义，为每个要使用 Secret 的容器添加对应 Secret 键的环境变量。 使用 Secret 键的环境变量应在 env[x].valueFrom.secretKeyRef 中指定 要包含的 Secret 名称和键名。
 3. 更改镜像并／或者命令行，以便程序在指定的环境变量中查找值。
 

```bash
 apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: mycontainer
    image: redis
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: password
  restartPolicy: Never 
```

```bash
echo $SECRET_USERNAME
输出类似于：

admin
echo $SECRET_PASSWORD
输出类似于：

1f2d1e2e67df
```
## 10. 案例
### 10.1 以环境变量的形式使用 Secret
创建一个 Secret 定义：

```bash
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  USER_NAME: YWRtaW4=
  PASSWORD: MWYyZDFlMmU2N2Rm
```

生成 Secret 对象：

```bash
kubectl apply -f mysecret.yaml
```

使用 envFrom 将 Secret 的所有数据定义为容器的环境变量。 Secret 中的键名称为 Pod 中的环境变量名称：

```bash
apiVersion: v1
kind: Pod
metadata:
  name: secret-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      envFrom:
      - secretRef:
          name: mysecret
  restartPolicy: Never
```
### 10.2 包含 SSH 密钥的 Pod
创建一个包含 SSH 密钥的 Secret：

```bash
kubectl create secret generic ssh-key-secret \
  --from-file=ssh-privatekey=/path/to/.ssh/id_rsa \
  --from-file=ssh-publickey=/path/to/.ssh/id_rsa.pub
```
**注意： 发送自己的 SSH 密钥之前要仔细思考：集群的其他用户可能有权访问该密钥。 你可以使用一个服务帐户，分享给 Kubernetes 集群中合适的用户，这些用户是你要分享的。 如果服务账号遭到侵犯，可以将其收回。**
现在我们可以创建一个 Pod，令其引用包含 SSH 密钥的 Secret，并通过存储卷来使用它：

```bash
apiVersion: v1
kind: Pod
metadata:
  name: secret-test-pod
  labels:
    name: secret-test
spec:
  volumes:
  - name: secret-volume
    secret:
      secretName: ssh-key-secret
  containers:
  - name: ssh-test-container
    image: mySshImage
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: "/etc/secret-volume"
```

容器中的命令运行时，密钥的片段可以在以下目录找到：

```bash
/etc/secret-volume/ssh-publickey
/etc/secret-volume/ssh-privatekey
```

然后容器可以自由使用 Secret 数据建立一个 SSH 连接。

### 10.3 包含生产/测试凭据的 Pod
下面的例子展示的是两个 Pod。 一个 Pod 使用包含生产环境凭据的 Secret，另一个 Pod 使用包含测试环境凭据的 Secret。

你可以创建一个带有 secretGenerator 字段的 kustomization.yaml 文件，或者执行 kubectl create secret：

```bash
kubectl create secret generic prod-db-secret \
  --from-literal=username=produser \
  --from-literal=password=Y4nys7f11

kubectl create secret generic test-db-secret \
  --from-literal=username=testuser \
  --from-literal=password=iluvtests

说明：
特殊字符（例如 $、\、*、= 和 !）会被你的 Shell解释，因此需要转义。 在大多数 Shell 中，对密码进行转义的最简单方式是用单引号（'）将其括起来。 例如，如果您的实际密码是 S!B\*d$zDsb，则应通过以下方式执行命令：

kubectl create secret generic dev-db-secret --from-literal=username=devuser --from-literal=password='S!B\*d$zDsb='
注意：您无需对文件中的密码（--from-file）中的特殊字符进行转义。
```
创建 pod ：

```bash
$ cat <<EOF > pod.yaml
apiVersion: v1
kind: List
items:
- kind: Pod
  apiVersion: v1
  metadata:
    name: prod-db-client-pod
    labels:
      name: prod-db-client
  spec:
    volumes:
    - name: secret-volume
      secret:
        secretName: prod-db-secret
    containers:
    - name: db-client-container
      image: myClientImage
      volumeMounts:
      - name: secret-volume
        readOnly: true
        mountPath: "/etc/secret-volume"
- kind: Pod
  apiVersion: v1
  metadata:
    name: test-db-client-pod
    labels:
      name: test-db-client
  spec:
    volumes:
    - name: secret-volume
      secret:
        secretName: test-db-secret
    containers:
    - name: db-client-container
      image: myClientImage
      volumeMounts:
      - name: secret-volume
        readOnly: true
        mountPath: "/etc/secret-volume"
EOF
```

将 Pod 添加到同一个 kustomization.yaml 文件

```bash
$ cat <<EOF >> kustomization.yaml
resources:
- pod.yaml
EOF
```

通过下面的命令应用所有对象

```bash
kubectl apply -k .
```

两个容器都会在其文件系统上存在以下文件，其中包含容器对应的环境的值：

```bash
/etc/secret-volume/username
/etc/secret-volume/password
```
### 10.4 Secret 卷中以句点号开头的文件 
你可以通过定义以句点开头的键名，将数据“隐藏”起来。 例如，当如下 Secret 被挂载到 secret-volume 卷中：

```bash
apiVersion: v1
kind: Secret
metadata:
  name: dotfile-secret
data:
  .secret-file: dmFsdWUtMg0KDQo=
---
apiVersion: v1
kind: Pod
metadata:
  name: secret-dotfiles-pod
spec:
  volumes:
  - name: secret-volume
    secret:
      secretName: dotfile-secret
  containers:
  - name: dotfile-test-container
    image: k8s.gcr.io/busybox
    command:
    - ls
    - "-l"
    - "/etc/secret-volume"
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: "/etc/secret-volume"
```

卷中将包含唯一的叫做 .secret-file 的文件。 容器 dotfile-test-container 中，该文件处于 /etc/secret-volume/.secret-file 路径下。

**说明： 以点号开头的文件在 ls -l 的输出中会被隐藏起来； 列出目录内容时，必须使用 ls -la 才能看到它们**


## 11. service & kubernetes & secrets
在Kubernetes 1.24中，ServiceAccount令牌秘密不再自动生成。请参阅1.24更新日志文件中的“紧急升级说明”：
LegacyServiceAccountTokenNoAutoGeneration功能门是beta，默认情况下启用。启用后，不再为每个ServiceAccount自动生成包含服务帐户令牌的Secret API对象。使用TokenRequest API获取服务帐户令牌，或者如果需要非过期令牌，请按照本指南为令牌控制器创建Secret API对象，以使用服务帐户令牌填充。

- [https://www.servicenow.com/community/itom-articles/kubernetes-1-24-no-secret-created-for-service-accounts/ta-p/2454682](https://www.servicenow.com/community/itom-articles/kubernetes-1-24-no-secret-created-for-service-accounts/ta-p/2454682)
- [Service account secret is not listed. How to fix it?](https://stackoverflow.com/questions/72256006/service-account-secret-is-not-listed-how-to-fix-it)

参考资料：

 - [kubernetes secret/](https://kubernetes.io/zh/docs/concepts/configuration/secret/)
 - [Google Kubernetes Engine (GKE) secret](https://cloud.google.com/kubernetes-engine/docs/concepts/secret)
 - [Kubernetes Secrets – How to Create, Use and Access Secrets](https://phoenixnap.com/kb/kubernetes-secrets)
 - [Kubernetes Secrets | How To Create, Use, and Access](https://www.containiq.com/post/kubernetes-secrets)
 - [https://www.containiq.com/post/kubernetes-secrets](https://medium.com/avmconsulting-blog/secrets-management-in-kubernetes-378cbf8171d0)


