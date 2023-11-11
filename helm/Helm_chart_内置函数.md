




----------

## 1. 定义 chart
Helm 的 github 上面有一个比较[完整的文档](https://github.com/helm/charts)，一个 chart 包就是一个文件夹的集合，文件夹名称就是 chart 包的名称，比如创建一个 `mychart` 的 chart 包：

```bash
$ helm create mychart
Creating mychart
$ tree mychart/
mychart/
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── ingress.yaml
│   ├── NOTES.txt
│   └── service.yaml
└── values.yaml

2 directories, 7 files
```
 templates 目录下面的文件:

 - `NOTES.txt`：chart 的 “帮助文本”。这会在用户运行 `helm install` 时显示给用户。
 - `deployment.yaml`：创建 `Kubernetes deployment` 的基本 `manifest`
 - `service.yaml`：为 `deployment` 创建 `service` 的基本 `manifest`
 - `ingress.yaml`: 创建 `ingress` 对象的资源清单文件
 - `_helpers.tpl`：放置模板助手的地方，可以在整个 chart 中重复使用

然后我们把 templates 目录下面所有文件全部删除掉，这里我们自己来创建模板文件：

```bash
$ rm -rf mychart/templates/*.*
```
##  2. 创建模板
这里我们来创建一个非常简单的模板 ConfigMap，在 templates 目录下面新建一个`configmap.yaml`文件：

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: mychart-configmap
data:
  myvalue: "Hello World"
```
实际上现在我们就有一个可安装的 chart 包了，通过`helm install`命令来进行安装：

```bash
$ helm install ./mychart/
NAME:   ringed-lynx
LAST DEPLOYED: Fri Sep  7 22:59:22 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME               DATA  AGE
mychart-configmap  1     0s
```
在上面的输出中，我们可以看到我们的 `ConfigMap` 资源对象已经创建了。然后使用如下命令我们可以看到实际的模板被渲染过后的资源文件：

```bash
$ helm get manifest ringed-lynx

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mychart-configmap
data:
  myvalue: "Hello World"
```
现在我们看到上面的 `ConfigMap` 文件是不是正是我们前面在模板文件中设计的，现在我们删除当前的`release`:

```bash
$ helm delete ringed-lynx
release "ringed-lynx" deleted
```
## 3. 添加一个简单的模板
我们可以看到上面我们定义的 ConfigMap 的名字是固定的，但往往这并不是一种很好的做法，我们可以通过插入 release 的名称来生成资源的名称，比如这里 ConfigMap 的名称我们希望是：`ringed-lynx-configmap`，这就需要用到 Chart 的模板定义方法了。

Helm Chart 模板使用的是[Go语言模板](https://pkg.go.dev/text/template)编写而成，并添加了[Sprig库](https://github.com/Masterminds/sprig)中的50多个附件模板函数以及一些其他特殊的函。

> 需要注意的是kubernetes资源对象的 `labels` 和 `name` 定义被限制 63个字符，所以需要注意名称的定义。

现在我们来重新定义下上面的 `configmap.yaml` 文件：

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
```
我们将名称替换成了`{{ .Release.Name }}-configmap`，其中包含在{{和}}之中的就是模板指令，`{{ .Release.Name }}` 将 `release` 的名称注入到模板中来，这样最终生成的 ConfigMap 名称就是以 `release` 的名称开头的了。这里的 Release 模板对象属于 Helm 内置的一种对象，还有其他很多内置的对象，稍后我们将接触到。

现在我们来重新安装我们的 Chart 包，注意观察 `ConfigMap` 资源对象的名称：

```bash
$ helm install ./mychart
helm install ./mychart/
NAME:   quoting-zebra
LAST DEPLOYED: Fri Sep  7 23:20:12 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                     DATA  AGE
quoting-zebra-configmap  1     0s
```
可以看到现在生成的名称变成了`quoting-zebra-configmap`，证明已经生效了，当然我们也可以使用命令`helm get manifest quoting-zebra`查看最终生成的清单文件的样子。

##  4. 调试

我们用模板来生成资源文件的清单，但是如果我们想要调试就非常不方便了，不可能我们每次都去部署一个`release`实例来校验模板是否正确，所幸的时 Helm 为我们提供了`--dry-run --debug`这个可选参数，在执行`helm install`的时候带上这两个参数就可以把对应的 `values` 值和生成的最终的资源清单文件打印出来，而不会真正的去部署一个release实例，比如我们来调试上面创建的 chart 包：

```bash
$ helm install . --dry-run --debug ./mychart
[debug] Created tunnel using local port: '35286'

[debug] SERVER: "127.0.0.1:35286"

[debug] Original chart version: ""
[debug] CHART PATH: /root/course/kubeadm/helm/mychart

NAME:   wrapping-bunny
REVISION: 1
RELEASED: Fri Sep  7 23:23:09 2018
CHART: mychart-0.1.0
USER-SUPPLIED VALUES:
{}

COMPUTED VALUES:
...
HOOKS:
MANIFEST:

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: wrapping-bunny-configmap
data:
  myvalue: "Hello World"
```

现在我们使用`--dry-run`就可以很容易地测试代码了，不需要每次都去安装一个 `release` 实例了，但是要注意的是这不能确保 Kubernetes 本身就一定会接受生成的模板，在调试完成后，还是需要去安装一个实际的 release 实例来进行验证的。

##  5. 内置对象

刚刚我们使用`{{.Release.Name}}`将 release 的名称插入到模板中。这里的 Release 就是 Helm 的内置对象，下面是一些常用的内置对象，在需要的时候直接使用就可以：

`Release`：这个对象描述了 release 本身。它里面有几个对象：

 - `Release.Name`：release 名称
 - `Release.Time`：release 的时间
 - `Release.Namespace`：release 的 `namespace`（如果清单未覆盖）
 - `Release.Service`：release 服务的名称（始终是 Tiller）。
 - `Release.Revision`：此 `release` 的修订版本号，从1开始累加。
 - `Release.IsUpgrade`：如果当前操作是升级或回滚，则将其设置为 `true`。
 - `Release.IsInstall`：如果当前操作是安装，则设置为 `true`。

`Values`：从`values.yaml`文件和用户提供的文件传入模板的值。默认情况下，Values 是空的。

`Chart`：`Chart.yaml`文件的内容。所有的 Chart 对象都将从该文件中获取。

`Files`：这提供对 chart 中所有非特殊文件的访问。虽然无法使用它来访问模板，但可以使用它来访问 chart 中的其他文件。请参阅 "访问文件" 部分。

 - `Files.Get` 是一个按名称获取文件的函数（`.Files.Get config.ini`）
 - `Files.GetBytes` 是将文件内容作为字节数组而不是字符串获取的函数。这对于像图片这样的东西很有用。

`Capabilities`：这提供了关于 Kubernetes 集群支持的功能的信息。

 - `Capabilities.APIVersions` 是一组版本信息。
 - `Capabilities.KubeVersion` 提供了查找 Kubernetes版本的方法。它具有以下值：`Major`，`Minor`，`GitVersion`，`GitCommit`，`GitTreeState`，`BuildDate`，`GoVersion`，`Compiler`，和`Platform`。
 - `Capabilities.TillerVersion` 提供了查找 `Tiller`版本的方法。它具有以下值：`SemVer`，`GitCommit`，和 `GitTreeState`。
`Capabilities.APIVersions.Has $version`指示集群上是否有可用的版本（例如batch/v1）或资源（例如）。apps/v1/Deployment
`Capabilities.KubeVersion`并且`Capabilities.KubeVersion.Version`是 Kubernetes 版本。
`Capabilities.KubeVersion.Major`是 Kubernetes 的主要版本。
`Capabilities.KubeVersion.Minor`是 Kubernetes 次要版本。
`Capabilities.HelmVersion`是包含 Helm 版本详细信息的对象，它是相同的输出h`elm version`
`Capabilities.HelmVersion.Version`是 semver 格式的当前 Helm 版本。
`Capabilities.HelmVersion.GitCommit`是 Helm git sha1。
`Capabilities.HelmVersion.GitTreeState`是 Helm git 树的状态。
`Capabilities.HelmVersion.GoVersion`是使用的 Go 编译器的版本。

`Template`：包含有关正在执行的当前模板的信息

 - `Template.Name`: 当前模板的命名空间文件路径（例如`mychart/templates/mytemplate.yaml`）
 - `Template.BasePath`: 当前图表的模板目录的命名空间路径（例如`mychart/templates`）


上面这些值可用于任何顶级模板，要注意内置值始终以大写字母开头。这也符合Go的命名约定。当你创建自己的名字时，你可以自由地使用适合你的团队的惯例。


---

✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>

 - [helm 官方](https://helm.sh/docs/howto/charts_tips_and_tricks/)
 - [helm 手册](https://www.bookstack.cn/read/helm-3.8.0-en/b5fa667da3c6d162.md)
 1. [helm v2  入门](https://ghostwritten.blog.csdn.net/article/details/120267913)
 2. [helm v3.8.0 命令入门指南](https://ghostwritten.blog.csdn.net/article/details/122866004) 
 3. [helm charts 入门指南](https://ghostwritten.blog.csdn.net/article/details/123400829)
 4. [helm 源大全](https://ghostwritten.blog.csdn.net/article/details/120287362)
 5. [Helm 命令](https://ghostwritten.blog.csdn.net/article/details/120289197)
 6. [Helm 内置函数](https://ghostwritten.blog.csdn.net/article/details/120302814)
 7. [Helm Values.yaml](https://ghostwritten.blog.csdn.net/article/details/122883851)
 8. [Helm 模板函数与管道](https://ghostwritten.blog.csdn.net/article/details/120305201)
 9. [Helm 控制流程](https://ghostwritten.blog.csdn.net/article/details/120305974)
 10. [Helm 命名模板](https://ghostwritten.blog.csdn.net/article/details/120311148)
 11. [Helm 其他注意事项](https://ghostwritten.blog.csdn.net/article/details/120312674)
 12. [Helm Hooks](https://ghostwritten.blog.csdn.net/article/details/120349113)
 13. [helm 自动滚动部署](https://ghostwritten.blog.csdn.net/article/details/122882715)
