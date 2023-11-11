##  1. NOTES.txt 
我们前面在使用 `helm install` 命令的时候，Helm 都会为我们打印出一大堆介绍信息，这样当别的用户在使用我们的 chart 包的时候就可以根据这些注释信息快速了解我们的 chart 包的使用方法，这些信息就是编写在 `NOTES.txt` 文件之中的，这个文件是纯文本的，但是它和其他模板一样，具有所有可用的普通模板函数和对象。

现在我们在前面的示例中 templates 目录下面创建一个 NOTES.txt 文件：

```bash
Thank you for installing {{ .Chart.Name }}.

Your release is named {{ .Release.Name }}.

To learn more about the release, try:

  $ helm status {{ .Release.Name }}
  $ helm get {{ .Release.Name }}
```
我们可以看到我们在 `NOTES.txt` 文件中也使用 `Chart` 和 `Release` 对象，现在我们在 `mychart` 包根目录下面执行安装命令查看是否能够得到上面的注释信息：

```bash
$ helm install .
Error: release nomadic-deer failed: ConfigMap in version "v1" cannot be handled as a ConfigMap: v1.ConfigMap: Data: ReadString: expects " or n, but found [, error found in #10 byte of ...|rselist":[{"0":"K8s"|..., bigger context ...|:{"app":"mychart","chart":"mychart","courselist":[{"0":"K8s"},{"1":"Python"},{"2":"Search"},{"3":"Go|...
```
我们可以看到出现了上面的错误信息，但是如果我们去执行 debug 命令来调试的话是没有任何问题的，是可以正常渲染的，但是为什么在正式安装的时候确出现了问题呢？这是因为我们在 debug 调试阶段只是检验模板是否可以正常渲染，并没有去检查对应的 kubernetes 资源对象对 yaml 文件的格式要求，所以我们说 debug 测试通过，并不代表 chart 就真正的是可用状态，比如我们这里是一个 ConfigMap 的资源对象，ConfigMap 对 data 区域的内容是有严格要求的，比如我们这里出现了下面这样的内容：

```bash
web: true
courselist:
- 0: "K8s"
- 1: "Python"
- 2: "Search"
- 3: "Golang"
```
这的确是一个合法的 yaml 文件的格式，但是对 `ConfigMap` 就不合法了，需要把 `true` 变成字符串，下面的字典数组变成一个多行的字符串，这样就是一个合法的 `ConfigMap`了：(`templates/config.yaml`)

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  labels:
{{- include "mychart.labels" . | indent 4}}
data:
  app: mychart
  myvalue: {{ .Values.hello | default  "Hello World" | quote }}
  {{- $releaseName := .Release.Name }}
  {{- with .Values.course }}
  k8s: {{ .k8s | upper | quote }}
  python: {{ .python | repeat 5 | quote }}
  release: {{ $releaseName }}
  {{- if eq .python "django" }}
  web: "true"
  {{- end }}
  {{- end }}
  courselist: |
    {{- range $index, $course := .Values.courselist }}
    {{ $course | title | quote }}
    {{- end }}
  {{- range $key, $val := .Values.course }}
  {{ $key }}: {{ $val | upper | quote }}
  {{- end }}
{{- include "mychart.labels" . | indent 2 }}
```
现在我们再来执行安装命令：

```bash
$ helm install .
NAME:   nosy-pig
LAST DEPLOYED: Sat Sep 29 19:26:16 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                DATA  AGE
nosy-pig-configmap  11    0s


NOTES:
Thank you for installing mychart.

Your release is named nosy-pig.

To learn more about the release, try:

  $ helm status nosy-pig
  $ helm get nosy-pig
```
现在已经安装成功了，而且下面的注释部分也被渲染出来了，我们可以看到 `NOTES.txt` 里面使用到的模板对象都被正确渲染了。

为我们创建的 chart 包提供一个清晰的 `NOTES.txt` 文件是非常有必要的，可以为用户提供有关如何使用新安装 `chart` 的详细信息，这是一种非常友好的方式方法。
