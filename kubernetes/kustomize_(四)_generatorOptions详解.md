![在这里插入图片描述](https://img-blog.csdnimg.cn/20201210210619416.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)


## 1. generatorOptions功能
Kustomize 提供了修改 ConfigMapGenerator 和 SecretGenerator 行为的选项，这些选项包括：

 - 不再将基于内容生成的哈希后缀添加到资源名称后
 - 为生成的资源添加 labels
 - 为生成的资源添加 annotations

## 2. generatorOptions实例
这个示例将展示如何运用这些选项，首先创建一个工作空间：

```bash
DEMO_HOME=$(mktemp -d)
```

创建 kustomization 并且为其添加一个 `ConfigMapGenerator`

```bash
cat > $DEMO_HOME/kustomization.yaml << EOF
configMapGenerator:
- name: my-configmap
  literals:	
  - foo=bar
  - baz=qux
EOF
```

添加如下 `generatorOptions`

```bash
cat >> $DEMO_HOME/kustomization.yaml << EOF
generatorOptions:
 disableNameSuffixHash: true  #关闭哈希后缀
 labels:
   kustomize.generated.resource: somevalue
 annotations:
   annotations.only.for.generated: othervalue
EOF
```

运行 kustomize build 并且确定生成的 `ConfigMap` 。

确定没有名称后缀

```bash
$ kustomize build $DEMO_HOME
apiVersion: v1
data:
  baz: qux
  foo: bar
kind: ConfigMap
metadata:
  annotations:
    annotations.only.for.generated: othervalue
  labels:
    kustomize.generated.resource: somevalue
  name: my-configmap  #没有哈希后缀
```

或者执行

```bash
$ test 1 == $(kustomize build $DEMO_HOME | grep "name: my-configmap$" | wc -l);echo $?
0
```

确定 `label kustomize.generated.resource: somevalue`

```bash
$ test 1 == $(kustomize build $DEMO_HOME | grep -A 1 "labels" | grep "kustomize.generated.resource" | wc -l); echo $?
0
```

确定 annotation annotations.only.for.generated: othervalue

```bash
$ test 1 == $(kustomize build $DEMO_HOME | grep -A 1 "annotations" | grep "annotations.only.for.generated" | wc -l); echo $?
0
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
