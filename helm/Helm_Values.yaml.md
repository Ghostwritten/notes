## 1. values.yaml

上面的内置对象中有一个对象就是 Values，该对象提供对传入 chart 的值的访问，Values 对象的值有4个来源：

 - chart 包中的 `values.yaml` 文件
 - 父 chart 包的 `values.yaml` 文件
 - 通过 `helm install` 或者 `helm upgrade` 的-f或者`--values`参数传入的自定义的 yaml 文件
 - 通过`--set` 参数传入的值

chart 的 `values.yaml` 提供的值可以被用户提供的 `values` 文件覆盖，而该文件同样可以被`--set`提供的参数所覆盖。

这里我们来重新编辑 `mychart/values.yaml` 文件，将默认的值全部清空，添加一个新的数据：(values.yaml)

```bash
course: k8s
```
然后我们在上面的 `templates/configmap.yaml` 模板文件中就可以使用这个值了：(`configmap.yaml`)

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  course: {{ .Values.course }}
```
可以看到最后一行我们是通过`{{ .Values.course }}`来获取 `course` 的值的。现在我们用 `debug` 模式来查看下我们的模板会被如何渲染：

```bash
$ helm install --dry-run --debug ./mychart
helm install --dry-run --debug .
[debug] Created tunnel using local port: '33509'

[debug] SERVER: "127.0.0.1:33509"

[debug] Original chart version: ""
[debug] CHART PATH: /root/course/kubeadm/helm/mychart

NAME:   nasal-anaconda
REVISION: 1
RELEASED: Sun Sep  9 17:37:52 2018
CHART: mychart-0.1.0
USER-SUPPLIED VALUES:
{}

COMPUTED VALUES:
course: k8s

HOOKS:
MANIFEST:

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nasal-anaconda-configmap
data:
  myvalue: "Hello World"
  course: k8s
```
我们可以看到 ConfigMap 中 `course` 的值被渲染成了 k8s，这是因为在默认的 `values.yaml` 文件中该参数值为 k8s，同样的我们可以通过`--set`参数来轻松的覆盖 course 的值：

```bash
$ helm install --dry-run --debug --set course=python ./mychart
[debug] Created tunnel using local port: '44571'

......

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: named-scorpion-configmap
data:
  myvalue: "Hello World"
  course: python
```

由于`--set` 比默认 `values.yaml` 文件具有更高的优先级，所以我们的模板生成为 `course: python`。

values 文件也可以包含更多结构化内容，例如，我们在 `values.yaml` 文件中可以创建 course 部分，然后在其中添加几个键：

```bash
course:
  k8s: devops
  python: django
```
现在我们稍微修改模板：

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  k8s: {{ .Values.course.k8s }}
  python: {{ .Values.course.python }}
```
同样可以使用 `debug` 模式查看渲染结果：

```bash
$ helm install --dry-run --debug ./mychart
[debug] Created tunnel using local port: '33801'
......
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: exhaling-turtle-configmap
data:
  myvalue: "Hello World"
  k8s: devops
  python: django
```

可以看到模板中的参数已经被 values.yaml 文件中的值给替换掉了。虽然以这种方式构建数据是可以的，但我们还是建议保持 value 树浅一些，平一些，这样维护起来要简单一点。
