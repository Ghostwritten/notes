

-----------
模板函数和管道是通过转换信息并将其插入到YAML文件中的强大方法。但有时候需要添加一些比插入字符串更复杂一些的模板逻辑。这就需要使用到模板语言中提供的控制结构了。

控制流程为我们提供了控制模板生成流程的一种能力，Helm 的模板语言提供了以下几种流程控制：

 - `if/else` 条件块
 - `with` 指定范围
 - `range` 循环块

除此之外，它还提供了一些声明和使用命名模板段的操作：

 - `define`在模板中声明一个新的命名模板
 - `template`导入一个命名模板
 - `block`声明了一种特殊的可填写的模板区域

##  1. if/else 条件
`if/else`块是用于在模板中有条件地包含文本块的方法，条件块的基本结构如下：

```c
{{ if PIPELINE }}
  # Do something
{{ else if OTHER PIPELINE }}
  # Do something else
{{ else }}
  # Default case
{{ end }}
```
当然要使用条件块就得判断条件是否为真，如果值为下面的几种情况，则管道的结果为 false：

 - 一个布尔类型的假
 - 一个数字零
 - 一个空的字符串
 - 一个nil（空或null）
 - 一个空的集合（map、slice、tuple、dict、array）

除了上面的这些情况外，其他所有条件都为真。

同样还是以上面的 ConfigMap 模板文件为例，添加一个简单的条件判断，如果 python 被设置为 `django`，则添加一个web: true：（`tempaltes/configmap.yaml`）

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: {{ .Values.hello | default  "Hello World" | quote }}
  k8s: {{ .Values.course.k8s | upper | quote }}
  python: {{ .Values.course.python | repeat 3 | quote }}
  {{ if eq .Values.course.python "django" }}web: true{{ end }}
```
在上面的模板文件中我们增加了一个条件语句判断`{{ if eq .Values.course.python "django" }}web: true{{ end }}`，其中运算符eq是判断是否相等的操作，除此之外，还有ne、lt、gt、and、or等运算符都是 Helm 模板已经实现了的，直接使用即可。这里我们`{{ .Values.course.python }}`的值在values.yaml文件中默认被设置为了`django`，所以正常来说下面的条件语句判断为真，所以模板文件最终被渲染后会有`web: true`这样的的一个条目：

```bash
$ helm install --dry-run --debug .
[debug] Created tunnel using local port: '40143'

......

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fallacious-prawn-configmap
data:
  myvalue: "Hello World"
  k8s: "DEVOPS"
  python: "djangodjangodjangodjangodjango"
  web: true
```
可以看到上面模板被渲染后出现了`web: true`的条目，如果我们在安装的时候覆盖下 python 的值呢，比如我们改成 `ai`:

```bash
helm install --dry-run --debug --set course.python=ai .
[debug] Created tunnel using local port: '42802'

......

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: dull-mite-configmap
data:
  myvalue: "Hello World"
  k8s: "DEVOPS"
  python: "aiaiai"
```
根据我们模板文件中的定义，如果`{{ .Values.course.python }}`的值为django的话就会新增`web: true`这样的一个条目，但是现在我们是不是通过参数`--set`将值设置为了 ai，所以这里条件判断为假，正常来说就不应该出现这个条目了，上面我们通过 debug 模式查看最终被渲染的值也没有出现这个条目，证明条件判断是正确的。

##  2. 空格控制
上面我们的条件判断语句是在一整行中的，如果平时经常写代码的同学可能非常不习惯了，我们一般会将其格式化为更容易阅读的形式，比如：

```bash
{{ if eq .Values.course.python "django" }}
web: true
{{ end }}
```
这样的话看上去比之前要清晰很多了，但是我们通过模板引擎来渲染一下，会得到如下结果：

```bash
$ helm install --dry-run --debug .
[debug] Created tunnel using local port: '44537'

......

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: bald-narwhal-configmap
data:
  myvalue: "Hello World"
  k8s: "DEVOPS"
  python: "djangodjangodjango"

  web: true
```
我们可以看到渲染出来会有多余的空行，这是因为当模板引擎运行时，它将一些值渲染过后，之前的指令被删除，但它之前所占的位置完全按原样保留剩余的空白了，所以就出现了多余的空行。YAML文件中的空格是非常严格的，所以对于空格的管理非常重要，一不小心就会导致你的YAML文件格式错误。

我们可以通过使用在模板标识{{后面添加破折号和空格{{-来表示将空白左移，而在}}前面添加一个空格和破折号-}}表示应该删除右边的空格，另外需要注意的是换行符也是空格！

使用这个语法，我们来修改我们上面的模板文件去掉多余的空格：`（templates/configmap.yaml）`

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: {{ .Values.hello | default  "Hello World" | quote }}
  k8s: {{ .Values.course.k8s | upper | quote }}
  python: {{ .Values.course.python | repeat 3 | quote }}
  {{- if eq .Values.course.python "django" }}
  web: true
  {{- end }}
```
现在我们来查看上面模板渲染过后的样子：

```bash
$ helm install --dry-run --debug .
[debug] Created tunnel using local port: '34702'

......

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mangy-olm-configmap
data:
  myvalue: "Hello World"
  k8s: "DEVOPS"
  python: "djangodjangodjango"
  web: true
```
现在是不是没有多余的空格了，另外我们需要谨慎使用`-}}`，比如上面模板文件中：

```bash
python: {{ .Values.course.python | repeat 3 | quote }}
{{- if eq .Values.course.python "django" -}}
web: true
{{- end }}
```
如果我们在if条件后面增加`-}}`，这会渲染成：

```bash
python: "djangodjangodjango"web: true
```
因为`-}}`它删除了双方的换行符，显然这是不正确的。

##  3. 使用 with 修改范围

接下来我们来看下with关键词的使用，它用来控制变量作用域。还记得之前我们的`{{ .Release.xxx }}`或者`{{ .Values.xxx }}`吗？其中的.就是表示对当前范围的引用，.Values就是告诉模板在当前范围中查找Values对象的值。而with语句就可以来控制变量的作用域范围，其语法和一个简单的if语句比较类似：

```bash
{{ with PIPELINE }}
  #  restricted scope
{{ end }}
```
`with`语句可以允许将当前范围.设置为特定的对象，比如我们前面一直使用的`.Values.course`，我们可以使用`with`来将.范围指向`.Values.course`：(`templates/configmap.yaml`)

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: {{ .Values.hello | default  "Hello World" | quote }}
  {{- with .Values.course }}
  k8s: {{ .k8s | upper | quote }}
  python: {{ .python | repeat 3 | quote }}
  {{- if eq .python "django" }}
  web: true
  {{- end }}
  {{- end }}
```
可以看到上面我们增加了一个`{{- with .Values.course }}xxx{{- end }}`的一个块，这样的话我们就可以在当前的块里面直接引用`.python`和`.k8s`了，而不需要进行限定了，这是因为该with声明将.指向了`.Values.course`，在`{{- end }}`后.就会复原其之前的作用范围了，我们可以使用模板引擎来渲染上面的模板查看是否符合预期结果。

不过需要注意的是在with声明的范围内，此时将无法从父范围访问到其他对象了，比如下面的模板渲染的时候将会报错，因为显然`.Release`根本就不在当前的.范围内，当然如果我们最后两行交换下位置就正常了，因为`{{- end }}`之后范围就被重置了：

```bash
{{- with .Values.course }}
k8s: {{ .k8s | upper | quote }}
python: {{ .python | repeat 3 | quote }}
release: {{ .Release.Name }}
{{- end }}
```
## 4. range 循环

如果大家对编程语言熟悉的话，几乎所有的编程语言都支持类似于`for`、`foreach`或者类似功能的循环机制，在 Helm 模板语言中，是使用range关键字来进行循环操作。

我们在`values.yaml`文件中添加上一个课程列表：

```bash
course:
  k8s: devops
  python: django
courselist:
- k8s
- python
- search
- golang
```
现在我们有一个课程列表，修改 ConfigMap 模板文件来循环打印出该列表：

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: {{ .Values.hello | default  "Hello World" | quote }}
  {{- with .Values.course }}
  k8s: {{ .k8s | upper | quote }}
  python: {{ .python | repeat 3 | quote }}
  {{- if eq .python "django" }}
  web: true
  {{- end }}
  {{- end }}
  courselist:
  {{- range .Values.courselist }}
  - {{ . | title | quote }}
  {{- end }}
```
可以看到最下面我们使用了一个`range`函数，该函数将会遍历`{{ .Values.courselist }}`列表，循环内部我们使用的是一个.，这是因为当前的作用域就在当前循环内，这个.从列表的第一个元素一直遍历到最后一个元素，然后在遍历过程中使用了`title`和`quote`这两个函数，**前面这个函数是将字符串首字母变成大写，后面就是加上双引号变成字符串**，所以按照上面这个模板被渲染过后的结果为：

```bash
$ helm install --dry-run --debug .
[debug] Created tunnel using local port: '34626'

......

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: dining-terrier-configmap
data:
  myvalue: "Hello World"
  k8s: "DEVOPS"
  python: "djangodjangodjango"
  web: true
  courselist:
  - "K8s"
  - "Python"
  - "Search"
  - "Golang"
```
我们可以看到`courselist`按照我们的要求循环出来了。除了 list 或者 tuple，range 还可以用于遍历具有键和值的集合（如map 或 dict），这个就需要用到变量的概念了。

##  5. 变量

前面我们已经学习了函数、管理以及控制流程的使用方法，我们知道编程语言中还有一个很重要的概念叫：变量，在 Helm 模板中，使用变量的场合不是特别多，但是在合适的时候使用变量可以很好的解决我们的问题。如下面的模板：

```bash
{{- with .Values.course }}
k8s: {{ .k8s | upper | quote }}
python: {{ .python | repeat 3 | quote }}
release: {{ .Release.Name }}
{{- end }}
```
我们在with语句块内添加了一个`.Release.Name`对象，但这个模板是错误的，编译的时候会失败，这是因为`.Release.Name`不在该with语句块限制的作用范围之内，我们可以将该对象赋值给一个变量可以来解决这个问题：

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  {{- $releaseName := .Release.Name -}}
  {{- with .Values.course }}
  k8s: {{ .k8s | upper | quote }}
  python: {{ .python | repeat 3 | quote }}
  release: {{ $releaseName }}
  {{- end }}
```
我们可以看到我们在with语句上面增加了一句`{{- $releaseName := .Release.Name -}}`，其中`$releaseName`就是后面的对象的一个引用变量，它的形式就是`$name`，赋值操作使用`:=`，这样with语句块内部的`$releaseName`变量仍然指向的是`.Release.Name`，同样，我们 `DEBUG` 下查看结果：

```bash
$ helm install --dry-run --debug .
[debug] Created tunnel using local port: '45474'

......

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nosy-seagull-configmap
data:
  k8s: "DEVOPS"
  python: "djangodjangodjango"
  release: nosy-seagull
```
可以看到已经正常了，另外变量在`range`循环中也非常有用，我们可以在循环中用变量来同时捕获索引的值：

```bash
courselist:
{{- range $index, $course := .Values.courselist }}
- {{ $index }}: {{ $course | title | quote }}
{{- end }}
```
例如上面的这个列表，我们在`range`循环中使用`$index`和`$course`两个变量来接收后面列表循环的索引和对应的值，最终可以得到如下结果：

```bash
$ helm install --dry-run --debug .
[debug] Created tunnel using local port: '38876'

......

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: vetoed-anaconda-configmap
data:
  courselist:
  - 0: "K8s"
  - 1: "Python"
  - 2: "Search"
  - 3: "Golang"
```
我们可以看到 `courselist` 下面将索引和对应的值都打印出来了，实际上具有键和值的数据结构我们都可以使用`range`来循环获得二者的值，比如我们可以对`.Values.course`这个字典来进行循环：

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  {{- range $key, $value := .Values.course }}
  {{ $key }}: {{ $value | quote }}
  {{- end }}
```
直接使用`range`循环，用变量`$key`、`$value`来接收字段`.Values.course`的键和值。这就是变量在 Helm 模板中的使用方法。
