


------

## 1. 模板函数
比如我们需要从`.Values`中读取的值变成字符串的时候就可以通过调用`quote`模板函数来实现：(`templates/configmap.yaml`)

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  k8s: {{ quote .Values.course.k8s }}
  python: {{ .Values.course.python }}
```
模板函数遵循调用的语法为：`functionName arg1 arg2...`。在上面的模板文件中，`quote .Values.course.k8s`调用quote函数并将后面的值作为一个参数传递给它。最终被渲染为：

```bash
$ helm install --dry-run --debug .
[debug] Created tunnel using local port: '39405'
......
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: masked-saola-configmap
data:
  myvalue: "Hello World"
  k8s: "devops"
  python: django
```

我们可以看到`.Values.course.k8s`被渲染成了字符串`devops`。上节课我们也提到过 Helm 是一种 [Go 模板语言](https://pkg.go.dev/text/template)，拥有超过60多种可用的内置函数，一部分是由Go 模板语言本身定义的，其他大部分都是Sprig模板库提供的一部分，我们可以前往这两个文档中查看这些函数的用法。

比如我们这里使用的**quote函数**就是Sprig 模板库提供的一种字符串函数，用途就是用双引号将字符串括起来，如果需要双引号"，则需要添加\来进行转义，而squote函数的用途则是用双引号将字符串括起来，而不会对内容进行转义。

所以在我们遇到一些需求的时候，首先要想到的是去查看下上面的两个模板文档中是否提供了对应的模板函数，这些模板函数可以很好的解决我们的需求。

##  2. 辅助模板
有时您想在图表中创建一些可重复使用的部分，无论它们是块还是模板部分。通常，将它们保存在自己的文件中会更干净。

在该`templates/`目录中，任何以下划线 ( _) 开头的文件都不会输出 `Kubernetes` 清单文件。所以按照惯例，辅助模板和部分被放置在一个`_helpers.tpl`文件中。

##   2. 管道
模板语言除了提供了丰富的内置函数之外，其另一个强大的功能就是管道的概念。和UNIX中一样，管道我们通常称为Pipeline，是一个链在一起的一系列模板命令的工具，以紧凑地表达一系列转换。简单来说，管道是可以按顺序完成一系列事情的一种方法。比如我们用管道来重写上面的 ConfigMap 模板：（`templates/configmap.yaml`）

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  k8s: {{ .Values.course.k8s | quote }}
  python: {{ .Values.course.python }}
```
这里我们直接调用quote函数，而是调换了一个顺序，使用一个管道符|将前面的参数发送给后面的模板函数：`{{ .Values.course.k8s | quote }}`，使用管道我们可以将几个功能顺序的连接在一起，比如我们希望上面的 ConfigMap 模板中的 k8s 的 value 值被渲染后是大写的字符串，则我们就可以使用管道来修改：`（templates/configmap.yaml）`

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  k8s: {{ .Values.course.k8s | upper | quote }}
  python: {{ .Values.course.python }}
```
这里我们在管道中增加了一个`upper`函数，该函数同样是Sprig 模板库提供的，表示将字符串每一个字母都变成大写。然后我们用`debug`模式来查看下上面的模板最终会被渲染成什么样子：

```bash
$ helm install --dry-run --debug .
[debug] Created tunnel using local port: '46651'
......
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: maudlin-labradoodle-configmap
data:
  myvalue: "Hello World"
  k8s: "DEVOPS"
  python: django
```
我们可以看到之前我们的devops已经被渲染成了`"DEVOPS"`了，要注意的是使用管道操作的时候，前面的操作结果会作为参数传递给后面的模板函数，比如我们这里希望将上面模板中的 python 的值渲染为重复出现3次的字符串，则我们就可以使用到Sprig 模板库提供的`repeat`函数，不过该函数需要传入一个参数`repeat COUNT STRING`表示重复的次数：（`templates/configmap.yaml`）

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  k8s: {{ .Values.course.k8s | upper | quote }}
  python: {{ .Values.course.python | quote | repeat 3 }}
```
该repeat函数会将给定的字符串重复3次返回，所以我们将得到这个输出：

```bash
helm install --dry-run --debug .
[debug] Created tunnel using local port: '39712'

......

Error: YAML parse error on mychart/templates/configmap.yaml: error converting YAML to JSON: yaml: line 7: did not find expected key

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: piquant-butterfly-configmap
data:
  myvalue: "Hello World"
  k8s: "DEVOPS"
  python: "django""django""django"
```
我们可以看到上面的输出中 python 对应的值变成了3个相同的字符串，这显然是不符合我们预期的，我们的预期是形成一个字符串，而现在是3个字符串了，而且上面还有错误信息，根据管道处理的顺序，我们将`quote`函数放到`repeat`函数后面去是不是就可以解决这个问题了：（`templates/configmap.yaml`）

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  k8s: {{ .Values.course.k8s | upper | quote }}
  python: {{ .Values.course.python | repeat 3 | quote }}
```
现在是不是就是先重复3次`.Values.course.python`的值，然后调用`quote`函数：

```bash
helm install --dry-run --debug .
[debug] Created tunnel using local port: '33837'

......

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: braided-manatee-configmap
data:
  myvalue: "Hello World"
  k8s: "DEVOPS"
  python: "djangodjangodjango"
```
现在是不是就正常了，也得到了我们的预期结果，所以我们在使用管道操作的时候一定要注意是按照从前到后一步一步顺序处理的。

## 3. default 函数
另外一个我们会经常使用的一个函数是`default` 函数：`default DEFAULT_VALUE GIVEN_VALUE`。该函数允许我们在模板内部指定默认值，以防止该值被忽略掉了。比如我们来修改上面的 ConfigMap 的模板：（`templates/configmap.yaml`）

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: {{ .Values.hello | default  "Hello World" | quote }}
  k8s: {{ .Values.course.k8s | upper | quote }}
  python: {{ .Values.course.python | repeat 5 | quote }}
```
由于我们的`values.yaml`文件中只定义了 course 结构的信息，并没有定义 hello 的值，所以如果没有设置默认值的话是得不到`{{ .Values.hello }}`的值的，这里我们为该值定义了一个默认值：`Hello World`，所以现在如果在`values.yaml`文件中没有定义这个值，则我们也可以得到默认值：

```bash
$ helm install --dry-run --debug .
[debug] Created tunnel using local port: '42670'

......

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: orbiting-hog-configmap
data:
  myvalue: "Hello World"
  k8s: "DEVOPS"
  python: "djangodjangodjangodjangodjango"
```

我们可以看到`myvalue`值被渲染成了`Hello World`，证明我们的默认值生效了。

##  4. lookup 函数
该lookup函数可用于查找正在运行的集群中的资源。查找函数的概要是`lookup apiVersion, kind, namespace, name -> resource or resource list`。
| parameter | apiVersion | kind   | namespace | name   |
|-----------|------------|--------|-----------|--------|
| type      | string     | string | string    | string |

`name`和都是`namespace`可选的，可以作为空字符串 ( `""`) 传递。

以下参数组合是可能的：
| 行为                                   | 查找功能                                     |
|--------------------------------------|------------------------------------------|
| Kubectl Get Pod Mypod -N Mynamespace | Lookup “v1” “Pod” “mynamespace” “mypod”  |
| Kubectl Get Pods -N Mynamespace      | Lookup “v1” “Pod” “mynamespace” “”       |
| Kubectl Get Pods —all-Namespaces     | Lookup “v1” “Pod” “” “”                  |
| Kubectl Get Namespace Mynamespace    | Lookup “v1” “Namespace” “” “mynamespace” |
| Kubectl Get Namespaces               | Lookup “v1” “Namespace” “” “”            |

当`lookup`返回一个对象时，它会返回一个字典。可以进一步导航此字典以提取特定值。

以下示例将返回`mynamespace`对象的注释：

```bash
(lookup "v1" "Namespace" "" "mynamespace").metadata.annotations
```
返回对象列表时`lookup`，可以通过以下`items`字段访问对象列表：

```bash
{{ range $index, $service := (lookup "v1" "Service" "mynamespace" "").items }}
    {{/* do something with each service */}}
{{ end }}
```
当没有找到对象时，返回一个空值。这可用于检查对象是否存在。

该`lookup`函数使用 `Helm` 现有的 `Kubernetes` 连接配置来查询 Kubernetes。如果与调用 API 服务器交互时返回任何错误（例如由于缺少访问资源的权限），则 `helm` 的模板处理将失败。

请记住，Helm 不应该在 `a helm template`或 a期间联系 `Kubernetes API` 服务器`helm install|update|delete|rollback --dry-run`，因此lookup在这种情况下，该函数将返回一个空列表（即 dict）。

##  5. 运算符
对于模板，运算符（eq、ne、lt、gt、and等or）都实现为函数。在管道中，操作可以用括号（(和)）分组。
现在我们可以从函数和管道转向[带有条件、循环和范围修饰符的流控制](https://ghostwritten.blog.csdn.net/article/details/120305974)。
