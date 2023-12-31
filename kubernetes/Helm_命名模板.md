

---

前面我们学习了一些 Helm 模板中的一些常用使用方法，但是我们都是操作的一个模板文件，在实际的应用中，很多都是相对比较复杂的，往往会超过一个模板，如果有多个应用模板，我们应该如何进行处理呢？这就需要用到新的概念：命名模板。

**命名模板我们也可以称为子模板，是限定在一个文件内部的模板，然后给一个名称。在使用命名模板的时候有一个需要特别注意的是：模板名称是全局的**，如果我们声明了两个相同名称的模板，最后加载的一个模板会覆盖掉另外的模板，由于子 chart 中的模板也是和顶层的模板一起编译的，所以在命名的时候一定要注意，不要重名了。

为了避免重名，有个通用的约定就是为每个定义的模板添加上 chart 名称：`{{define "mychart.labels"}}`，`define`关键字就是用来声明命名模板的，加上 chart 名称就可以避免不同 chart 间的模板出现冲突的情况。

##  1. 声明define和使用命名template
使用`define`关键字就可以允许我们在模板文件内部创建一个命名模板，它的语法格式如下：

```bash
{{ define "ChartName.TplName" }}
# 模板内容区域
{{ end }}
```
比如，现在我们可以定义一个模板来封装一个 `label` 标签：

```c
{{- define "mychart.labels" }}
  labels:
    from: helm
    date: {{ now | htmlDate }}
{{- end }}
```
然后我们可以将该模板嵌入到现有的 `ConfigMap` 中，然后使用`template`关键字在需要的地方包含进来即可：

```bash
{{- define "mychart.labels" }}
  labels:
    from: helm
    date: {{ now | htmlDate }}
{{- end }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  {{- template "mychart.labels" }}
data:
  {{- range $key, $value := .Values.course }}
  {{ $key }}: {{ $value | quote }}
  {{- end }}
```
我们这个模板文件被渲染过后的结果如下所示：

$ helm install --dry-run --debug .

```bash
$ helm install --dry-run --debug .
[debug] Created tunnel using local port: '42058'

......

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: ardent-bunny-configmap
  labels:
    from: helm
    date: 2018-09-22
data:
  k8s: "devops"
  python: "django"
```
我们可以看到define区域定义的命名模板被嵌入到了`template`所在的区域，但是如果我们将命名模板全都写入到一个模板文件中的话无疑也会增大模板的复杂性。

还记得我们在创建 chart 包的时候，templates 目录下面默认会生成一个`_helpers.tpl`文件吗？我们前面也提到过 `templates` 目录下面除了`NOTES.txt`文件和以下划线`_`开头命令的文件之外，都会被当做 kubernetes 的资源清单文件，而这个下划线开头的文件不会被当做资源清单外，还可以被其他 chart 模板中调用，这个就是 `Helm` 中的`partials`文件，所以其实我们完全就可以将命名模板定义在这些`partials`文件中，默认就是`_helpers.tpl`文件了。

现在我们将上面定义的命名模板移动到 `templates/_helpers.tpl` 文件中去：

```bash
{{/* 生成基本的 labels 标签 */}}
{{- define "mychart.labels" }}
  labels:
    from: helm
    date: {{ now | htmlDate }}
{{- end }}
```
一般情况下面，我们都会在命名模板头部加一个简单的文档块，用`/**/`包裹起来，用来描述我们这个命名模板的用途的。

现在我们讲命名模板从模板文件 `templates/configmap.yaml` 中移除，当然还是需要保留 `template` 来嵌入命名模板内容，名称还是之前的 `mychart.lables`，这是因为模板名称是全局的，所以我们可以能够直接获取到。我们再用 `DEBUG` 模式来调试下是否符合预期？

###  示例：Creating Image Pull Secrets
假设凭证是在`values.yaml`文件中定义的，如下所示：

```bash
imageCredentials:
  registry: quay.io
  username: someone
  password: sillyness
  email: someone@host.com
```
然后我们定义我们的帮助模板如下：

```bash
{{- define "imagePullSecret" }}
{{- with .Values.imageCredentials }}
{{- printf "{\"auths\":{\"%s\":{\"username\":\"%s\",\"password\":\"%s\",\"email\":\"%s\",\"auth\":\"%s\"}}}" .registry .username .password .email (printf "%s:%s" .username .password | b64enc) | b64enc }}
{{- end }}
{{- end }}
```
最后，我们在更大的模板中使用辅助模板来创建 Secret 清单：

```bash
apiVersion: v1
kind: Secret
metadata:
  name: myregistrykey
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: {{ template "imagePullSecret" . }}
```

##  2. 模板范围

上面我们定义的命名模板中，没有使用任何对象，只是使用了一个简单的函数，如果我们在里面来使用 chart 对象相关信息呢：

```bash
{{/* 生成基本的 labels 标签 */}}
{{- define "mychart.labels" }}
  labels:
    from: helm
    date: {{ now | htmlDate }}
    chart: {{ .Chart.Name }}
    version: {{ .Chart.Version }}
{{- end }}
```
如果这样的直接进行渲染测试的话，是不会得到我们的预期结果的：

```bash
$ $ helm install --dry-run --debug .
[debug] Created tunnel using local port: '42058'

......

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: peeking-zorse-configmap
  labels:
    from: helm
    date: 2018-09-22
    chart:
    version:
data:
  k8s: "devops"
  python: "django"
```
chart 的名称和版本都没有正确被渲染，这是因为他们不在我们定义的模板范围内，当命名模板被渲染时，它会接收由 template 调用时传入的作用域，由于我们这里并没有传入对应的作用域，因此模板中我们无法调用到 .Chart 对象，要解决也非常简单，我们只需要在 template 后面加上作用域范围即可：

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  {{- template "mychart.labels" . }}
data:
  {{- range $key, $value := .Values.course }}
  {{ $key }}: {{ $value | quote }}
  {{- end }}
```
我们在 `template` 末尾传递了.，表示当前的最顶层的作用范围，如果我们想要在命名模板中使用.Values范围内的数据，当然也是可以的，现在我们再来渲染下我们的模板：

```bash
$ helm install --dry-run --debug .
[debug] Created tunnel using local port: '32768'

......

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: oldfashioned-mule-configmap
  labels:
    from: helm
    date: 2018-09-22
    chart: mychart
    version: 0.1.0
data:
  k8s: "devops"
  python: "django"
```
我们可以看到 chart 的名称和版本号都已经被正常渲染出来了。

##  3. include 函数

假如现在我们将上面的定义的 labels 单独提取出来放置到 `_helpers.tpl` 文件中：

```bash
{{/* 生成基本的 labels 标签 */}}
{{- define "mychart.labels" }}
from: helm
date: {{ now | htmlDate }}
chart: {{ .Chart.Name }}
version: {{ .Chart.Version }}
{{- end }}
```
现在我们将该命名模板插入到 `configmap` 模板文件的 `labels` 部分和 `data` 部分：

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  labels:
    {{- template "mychart.labels" . }}
data:
  {{- range $key, $value := .Values.course }}
  {{ $key }}: {{ $value | quote }}
  {{- end }}
  {{- template "mychart.labels" . }}
```
然后同样的查看下渲染的结果：

```bash
$ helm install --dry-run --debug .
[debug] Created tunnel using local port: '42652'

......

Error: YAML parse error on mychart/templates/configmap.yaml: error converting YAML to JSON: yaml: line 9: mapping values are not allowed in this context

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: altered-wombat-configmap
  labels:
from: helm
date: 2018-09-22
chart: mychart
version: 0.1.0
data:
  k8s: "devops"
  python: "django"
from: helm
date: 2018-09-22
chart: mychart
version: 0.1.0
```
我们可以看到渲染结果是有问题的，不是一个正常的 YAML 文件格式，这是因为template只是表示一个嵌入动作而已，不是一个函数，所以原本命名模板中是怎样的格式就是怎样的格式被嵌入进来了，比如我们可以在命名模板中给内容区域都空了两个空格，再来查看下渲染的结构：

```bash
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mortal-cricket-configmap
  labels:
  from: helm
  date: 2018-09-22
  chart: mychart
  version: 0.1.0
data:
  k8s: "devops"
  python: "django"
  from: helm
  date: 2018-09-22
  chart: mychart
  version: 0.1.0
```
我们可以看到 data 区域里面的内容是渲染正确的，但是上面 labels 区域是不正常的，因为命名模板里面的内容是属于 labels 标签的，是不符合我们的预期的，但是我们又不可能再去把命名模板里面的内容再增加两个空格，因为这样的话 data 里面的格式又不符合预期了。

为了解决这个问题，Helm 提供了另外一个方案来代替`template`，那就是使用`include`函数，在需要控制空格的地方使用`indent`管道函数来自己控制，比如上面的例子我们替换成`include`函数：

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  labels:
{{- include "mychart.labels" . | indent 4 }}
data:
  {{- range $key, $value := .Values.course }}
  {{ $key }}: {{ $value | quote }}
  {{- end }}
{{- include "mychart.labels" . | indent 2 }}
```
在 labels 区域我们需要4个空格，所以在管道函数indent中，传入参数4就可以，而在 data 区域我们只需要2个空格，所以我们传入参数2即可以，现在我们来渲染下我们这个模板看看是否符合预期呢：

```bash
$ helm install --dry-run --debug .
[debug] Created tunnel using local port: '38481'

......

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: torpid-bobcat-configmap
  labels:
    from: helm
    date: 2018-09-22
    chart: mychart
    version: 0.1.0
data:
  k8s: "devops"
  python: "django"
  from: helm
  date: 2018-09-22
  chart: mychart
  version: 0.1.0
```
可以看到是符合我们的预期，所以在 Helm 模板中我们使用 include 函数要比 template 更好，可以更好地处理 YAML 文件输出格式。

##  4. required 函数
该`required`函数允许您根据模板渲染的需要声明一个特定的值条目。如果该值为空，则模板渲染将失败并显示用户提交的错误消息。

以下`required`函数示例声明了一个条目 `for.Values.who`是必需的，并且在缺少该条目时将打印一条错误消息：

```bash
value: {{ required "A valid .Values.who entry required!" .Values.who }}
```

## 5. tpl 函数
该tpl函数允许开发人员将字符串评估为模板内的模板。这对于将模板字符串作为值传递给图表或呈现外部配置文件很有用。句法：`{{ tpl TEMPLATE_STRING VALUES }}`

```bash
# values
template: "{{ .Values.name }}"
name: "Tom"
# template
{{ tpl .Values.template . }}
# output
Tom
```
渲染外部配置文件：

```bash
# external configuration file conf/app.conf
firstName={{ .Values.firstName }}
lastName={{ .Values.lastName }}
# values
firstName: Peter
lastName: Parker
# template
{{ tpl (.Files.Get "conf/app.conf") . }}
# output
firstName=Peter
lastName=Parker
```



