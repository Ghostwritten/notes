
----

## 1. Secret 是什么？
Kubernetes 的 ConfigMaps 和 Secrets 都是key:value map，但 Secrets 的内容更为敏感，比如：密码或者 ssh 秘钥。

Kubernetes 开发者以各种方式工作，Secrets 保存的信息相比 ConfigMaps,Deployments 等的配置信息需要更谨慎的隐藏。

## 2. 来自本地文件的 Secret
kustomize 可以通过三种不同的方式生成来自本地文件的 Secret 。

 - 从 env 文件中获取（NAME = VALUE，每行一个）
 - 使用文件内容来生成一个 secret
 - 从 kustomization.yaml 文件获取 secret

这里有一个示例结合所有的三种方式：

创建一个包含一些短密码的 env 文件：

```bash
cat <<EOF>foo.env
ROUTER_PASSWORD=admin
DB_PASSWORD=iloveyou
EOF
```
创建一个长密码的文本文件：

```bash
cat <<EOF>longsecret.txt
Lorem ipsum dolor sit amet,
consectetur adipiscing elit,
sed do eiusmod tempor incididunt
ut labore et dolore magna aliqua.
EOF
```

创建一个kustomization.yaml 文件, 其中包含引用上面文件的 secretGenerator, 并且另外定义一些文字 KV 对：

```bash
cat <<EOF>kustomization.yaml
secretGenerator:
- name: mysecrets
  envs:
  - foo.env
  files:
  - longsecret.txt
  literals:
  - FRUIT=apple
  - VEGETABLE=carrot
EOF
```
生成 Secret ：

```bash
$ kustomize build .
apiVersion: v1
data:
  DB_PASSWORD: aWxvdmV5b3U=
  FRUIT: YXBwbGU=
  ROUTER_PASSWORD: YWRtaW4=
  VEGETABLE: Y2Fycm90
  longsecret.txt: TG9yZW0gaXBzdW0gZG9sb3Igc2l0IGFtZXQsCmNvbnNlY3RldHVyIGFkaXBpc2NpbmcgZWxpdCwKc2VkIGRvIGVpdXNtb2QgdGVtcG9yIGluY2lkaWR1bnQKdXQgbGFib3JlIGV0IGRvbG9yZSBtYWduYSBhbGlxdWEuCg==
kind: Secret
metadata:
  name: mysecrets-467mf7hg78
type: Opaque
```
资源名称的前缀为 mysecrets（在 kustomization.yaml 中指定），后跟其内容的哈希值。

使用 base64 解码器确认这些值的原始版本。

这三种方法共同的问题是创建 Secret 所使用的敏感数据必须保存磁盘上。

这会增加额外的安全问题：对本地存储的敏感文件的查看、安装和删除权限的控制等。

参考连接：
https://github.com/kubernetes-[sigs/kustomize/blob/master/examples/zh/secretGeneratorPlugin.md](https://github.com/kubernetes-sigs/kustomize/blob/master/examples/zh/secretGeneratorPlugin.md)

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
