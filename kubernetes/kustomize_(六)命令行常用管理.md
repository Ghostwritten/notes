![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/17fc338481f0e827d71330e997d7824c.png#pic_center)

---

## 1. 构建空间
首先构建一个工作空间：

```bash
export DEMO_HOME=$(mktemp -d)
```

创建包含pod资源的 kustomization

```bash
cat <<EOF >$DEMO_HOME/kustomization.yaml
resources:
- pod.yaml
EOF
```

创建 pod 资源pod.yaml

```bash
cat <<EOF >$DEMO_HOME/pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.29.0
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-mydb
    image: busybox:1.29.0
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
EOF
```

`myapp-pod` 包含一个init容器和一个普通容器，两者都使用 `busybox：1.29.0` 镜像。

在 `kustomization.yaml` 中添加 images 字段来更改镜像 busybox 和标签 `1.29.0` 。

## 2. 修改镜像
通过 kustomize 添加 images：

```bash
cd $DEMO_HOME
kustomize edit set image busybox=alpine:3.6
```

将images字段将被添加到kustomization.yaml：

```bash
resources:
- pod.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
images:
- name: busybox
  newName: alpine
  newTag: "3.6"

```

构建 kustomization

```bash
$ kustomize build $DEMO_HOME
.....
    image: alpine:3.6
.....

```

确认busybox镜像和标签是否被替换为alpine：3.6：

```bash
$ test 2 =  $(kustomize build $DEMO_HOME | grep alpine:3.6 | wc -l);  echo $?
0
```
## 3. 添加secret

```bash
$ kustomize edit add secret sl-demo-app --from-literal=db-password=12345
$ cat kustomization.yaml 
resources:
- pod.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
images:
- name: busybox
  newName: alpine
  newTag: "3.6"
secretGenerator:
- literals:
  - db-password=12345
  name: sl-demo-app
  type: Opaque

```
## 4. 远程target
kustomize build 可以将 URL 作为参数传入并运行.

运行效果与如下操作相同:

如果想要要立即尝试此操作，可以按照 multibases 示例运行 kustomization 运行构建。然后查看输出中的pod：

```bash
kustomize build github.com/kubernetes-sigs/kustomize/examples/multibases/dev/?ref=v1.0.6

target="github.com/kubernetes-sigs/kustomize/examples/multibases/dev/?ref=v1.0.6"
test 1 == $(kustomize build $target | grep dev-myapp-pod | wc -l); echo $?
```

在该示例中运行 overlay 将获得三个 pod（在此 overlay 结合了dev、staging 和 prod 的 bases，以便同时将它们全部发送给所有人）：

```bash
target="https://github.com/kubernetes-sigs/kustomize/examples/multibases?ref=v1.0.6"
test 3 == $(kustomize build $target | grep cluster-a-.*-myapp-pod | wc -l); echo $?
```

将 URL 作为 base ：

```bash
DEMO_HOME=$(mktemp -d)

cat <<EOF >$DEMO_HOME/kustomization.yaml
resources:
- github.com/kubernetes-sigs/kustomize/examples/multibases?ref=v1.0.6
namePrefix: remote-
EOF
```

构建该 base 以确定所有的三个 pod 都有 remote- 前缀。

```bash
test 3 == $(kustomize build $DEMO_HOME | grep remote-.*-myapp-pod | wc -l); echo $?
```

URL format
URL 需要遵循 hashicorp/go-getter URL 格式 。下面是一些遵循此约定的 Github repos 示例url。

```bash
kustomization.yaml 在根目录

github.com/Liujingfang1/mysql

kustomization.yaml 在 test 分支的根目录

github.com/Liujingfang1/mysql?ref=test

kustomization.yaml 在 v1.0.6 版本的子目录

github.com/kubernetes-sigs/kustomize/examples/multibases?ref=v1.0.6

kustomization.yaml repoUrl2 分支的子目录

github.com/Liujingfang1/kustomize/examples/helloWorld?ref=repoUrl2

kustomization.yaml commit 7050a45134e9848fca214ad7e7007e96e5042c03 的子目录

github.com/Liujingfang1/kustomize/examples/helloWorld?ref=7050a45134e9848fca214ad7e7007e96e5042c03
```
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
