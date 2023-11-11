

---


Helm 使用一种称为图表的打包格式。图表是描述一组相关 Kubernetes 资源的文件的集合。单个图表可用于部署简单的东西，例如 memcached pod，或复杂的东西，例如带有 HTTP 服务器、数据库、缓存等的完整 Web 应用程序堆栈。

图表被创建为放置在特定目录树中的文件。它们可以打包到版本化档案中进行部署。

如果您想下载并查看已发布图表的文件，而不安装它，您可以使用`helm pull chartrepo/chartname.`


## 1. charts 文件结构
描述 WordPress 的图表将存储在`wordpress/`目录中。

```bash
wordpress/
  Chart.yaml          # A YAML file containing information about the chart
  LICENSE             # OPTIONAL: A plain text file containing the license for the chart
  README.md           # OPTIONAL: A human-readable README file
  values.yaml         # The default configuration values for this chart
  values.schema.json  # OPTIONAL: A JSON Schema for imposing a structure on the values.yaml file
  charts/             # A directory containing any charts upon which this chart depends.
  crds/               # Custom Resource Definitions
  templates/          # A directory of templates that, when combined with values,
                      # will generate valid Kubernetes manifest files.
  templates/NOTES.txt # OPTIONAL: A plain text file containing short usage notes
```

##  2. Chart.yaml 文件

```bash
apiVersion: The chart API version (required)
name: The name of the chart (required)
version: A SemVer 2 version (required)
kubeVersion: A SemVer range of compatible Kubernetes versions (optional)
description: A single-sentence description of this project (optional)
type: The type of the chart (optional)
keywords:
  - A list of keywords about this project (optional)
home: The URL of this projects home page (optional)
sources:
  - A list of URLs to source code for this project (optional)
dependencies: # A list of the chart requirements (optional)
  - name: The name of the chart (nginx)
    version: The version of the chart ("1.2.3")
    repository: (optional) The repository URL ("https://example.com/charts") or alias ("@repo-name")
    condition: (optional) A yaml path that resolves to a boolean, used for enabling/disabling charts (e.g. subchart1.enabled )
    tags: # (optional)
      - Tags can be used to group charts for enabling/disabling together
    import-values: # (optional)
      - ImportValues holds the mapping of source values to parent key to be imported. Each item can be a string or pair of child/parent sublist items.
    alias: (optional) Alias to be used for the chart. Useful when you have to add the same chart multiple times
maintainers: # (optional)
  - name: The maintainers name (required for each maintainer)
    email: The maintainers email (optional for each maintainer)
    url: A URL for the maintainer (optional for each maintainer)
icon: A URL to an SVG or PNG image to be used as an icon (optional).
appVersion: The version of the app that this contains (optional). Needn't be SemVer. Quotes recommended.
deprecated: Whether this chart is deprecated (optional, boolean)
annotations:
  example: A list of annotations keyed by name (optional).
```
从[v3.3.2](https://github.com/helm/helm/releases/tag/v3.3.2) 开始，不允许使用其他字段。推荐的方法是在`annotations`.


##  3. Charts and Versioning
每个图表都必须有一个版本号。版本必须遵循[SemVer 2](https://semver.org/spec/v2.0.0.html)标准。与 Helm Classic 不同，`Helm v2` 及更高版本使用版本号作为发布标记。存储库中的包由名称加版本标识。

例如，nginx版本字段设置为的图表`version: 1.2.3`将被命名为：

```bash
nginx-1.2.3.tgz
```
还支持更复杂的 `SemVer 2` 名称，例如`version: 1.2.3-alpha.1+ef365`. 但是系统明确禁止非 SemVer 名称。

> 注意： `Helm Classic` 和 `Deployment Manager` 在图表方面都非常面向 GitHub，而 Helm v2及更高版本不依赖或不需要 GitHub 甚至 Git。因此，它根本不使用 Git SHA 进行版本控制。

许多 Helm 工具（包括 CLI）使用内部的`version`字段。`Chart.yaml`生成包时，该`helm package`命令将使用它在 中找到的版本`Chart.yaml`作为包名称中的标记。系统假定图表包名称中的版本号与`Chart.yaml`. 不满足此假设将导致错误。

### 3.1 apiVersion领域_
该`apiVersion`字段应该v2用于至少需要 `Helm 3` 的 `Helm` 图表。支持先前 Helm 版本的图表有一个`apiVersion`设置，v1并且仍然可以由 Helm 3 安装。

从更改`v1`为`v2`：

定义图表依赖项的`dependencies`字段，位于图表的单独`requirements.yaml`文件中v1（请参阅[图表依赖](https://www.bookstack.cn/read/helm-3.8.0-en/89342c804d7c3b76.md#chart-dependencies)项）。
字段，type区分应用程序和库图表（请参阅图表类型）。
### 3.2 appVersion领域_
请注意，该`appVersion`字段与该字段无关version。这是一种指定应用程序版本的方法。例如，`drupal`图表可能有一个`appVersion: "8.2.1"`，表示图表中包含的 Drupal 版本（默认）是8.2.1。此字段是信息性的，对图表版本计算没有影响。强烈建议将版本用引号括起来。**它强制 YAML 解析器将版本号视为字符串。在某些情况下，不加引号可能会导致解析问题**。例如，YAML 解释1.0为浮点值，而 git commit SHA则解释1234e10为科学记数法。

从 `Helm v3.5.0` 开始，`helm create`将默认`appVersion`字段包含在引号中。

### 3.3 kubeVersion领域_
可选kubeVersion字段可以定义支持的 Kubernetes 版本的 semver 约束。Helm 将在安装图表时验证版本约束，如果集群运行不受支持的 Kubernetes 版本，则会失败。

版本约束可能包括空格分隔的 AND 比较，例如

```bash
>= 1.13.0 < 1.15.0
```

它们本身可以与 OR||运算符组合，如下例所示

```bash
>= 1.13.0 < 1.14.0 || >= 1.14.1 < 1.15.0
```

在此示例中，版本`1.14.0`被排除在外，如果已知某些版本中的错误会阻止图表正常运行，这可能是有意义的。

除了使用运算符`=` `!=` `>` `<` `>=` `<=`的版本约束之外，还支持以下速记符号

 - 闭合区间的连字符范围，其中`1.1 - 2.3.4`等价于`>= 1.1 <= 2.3.4`.
 - 通配符`x`, `X`and `*`, where`1.2.x`等价于`>= 1.2.0 < 1.3.0`.
 - 波浪号范围（允许更改补丁版本），`~1.2.3`相当于`>= 1.2.3 < 1.3.0`.
 - 插入符号范围（允许进行较小的版本更改），其中`^1.2.3`相当于`>= 1.2.3 < 2.0.0`.

有关支持的 semver 约束的详细说明，请参阅[Masterminds/semver](https://github.com/Masterminds/semver)。

###  3.4 Deprecating Charts
在图表存储库中管理图表时，有时需要弃用图表。中的可选`deprecated`字段`Chart.yaml`可用于将图表标记为已弃用。如果存储库中图表的最新版本被标记为已弃用，则整个图表被视为已弃用。稍后可以通过发布未标记为已弃用的较新版本来重用图表名称。弃用图表的工作流程是：

 - 更新图表Chart.yaml以将图表标记为已弃用，更新版本
 - 在 Chart Repository 中发布新的图表版本
 - 从源存储库中删除图表（例如 git）


##  4. Chart Types
该type字段定义图表的类型。有两种类型：`application`和`library`。应用程序是默认类型，它是可以完全操作的标准图表。库图表为图表构建器提供实用程序或功能。库图表与应用程序[图表](https://www.bookstack.cn/read/helm-3.8.0-en/5b6ede30be15ad22.md)不同，因为它不可安装并且通常不包含任何资源对象。

> 注意：应用程序图表可以用作库图表。这可以通过将类型设置为来启用library。然后该图表将呈现为可以利用所有实用程序和功能的库图表。图表的所有资源对象都不会被渲染。


##  5. Chart LICENSE, README and NOTES
图表还可以包含描述图表的安装、配置、使用和许可证的文件。

许可证是包含图表[许可证的纯文本文件](https://en.wikipedia.org/wiki/Software_license)。该图表可以包含许可证，因为它可能在模板中具有编程逻辑，因此不仅仅是配置。如果需要，图表安装的应用程序也可以有单独的许可证。

图表的 `README` 应采用 `Markdown (README.md)` 格式，通常应包含：

 - 图表提供的应用程序或服务的描述
 - 运行图表的任何先决条件或要求
 - 选项中的描述`values.yaml`和默认值
 - 可能与图表的安装或配置相关的任何其他信息

当集线器和其他用户界面显示有关图表的详细信息时，该详细信息是从`README.md`文件中的内容中提取的。

该图表还可以包含一个简短的纯文本`templates/NOTES.txt`文件，该文件将在安装后以及查看版本状态时打印出来。此文件作为模板进行评估，可用于显示使用说明、后续步骤或与图表发布相关的任何其他信息。例如，可以提供连接到数据库或访问 Web UI 的说明。由于此文件在运行时会打印到 `STDOUThelm install`或`helm status`，因此建议保持内容简短并指向 README 以获得更多详细信息。

##  6. Chart Dependencies
在 Helm 中，一个图表可能依赖于任意数量的其他图表。这些依赖关系可以使用`dependencies`字段动态链接`Chart.yaml`或引入`charts/`目录并手动管理。

dependencies使用字段管理依赖项
当前图表所需的图表在dependencies字段中定义为列表。

```bash
dependencies:
 - name: apache
    version: 1.2.3
    repository: https://example.com/charts
 - name: mysql
    version: 3.2.1
    repository: https://another.example.com/charts
```
 - 该`name`字段是您想要的图表的名称。
 - 该`version`字段是您想要的图表版本。
 - 该`repository`字段是图表存储库的完整 URL。请注意，您还必须使用`helm repo add`在本地添加该存储库。
 - 您可以使用 `repo` 的名称而不是 URL

```bash
$ helm repo add fantastic-charts https://fantastic-charts.storage.googleapis.com
```

```bash
dependencies:
  - name: awesomeness
    version: 1.0.0
    repository: "@fantastic-charts"
```
一旦你定义了依赖，你可以运行`helm dependency update`它，它会使用你的依赖文件为你下载所有指定的图表到你的`charts/`目录中。

```bash
$ helm dep up foochart
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "local" chart repository
...Successfully got an update from the "stable" chart repository
...Successfully got an update from the "example" chart repository
...Successfully got an update from the "another" chart repository
Update Complete. Happy Helming!
Saving 2 charts
Downloading apache from repo https://example.com/charts
Downloading mysql from repo https://another.example.com/charts
```
检索图表时`helm dependency update`，会将其作为图表存档存储在`charts/`目录中。因此，对于上面的示例，人们希望在图表目录中看到以下文件：

```bash
charts/
  apache-1.2.3.tgz
  mysql-3.2.1.tgz
```
##  7. Alias field in dependencies
除了上面的其他字段之外，每个需求条目都可能包含可选字段`alias`。

为依赖关系图表添加别名会将图表放入依赖关系中，使用别名作为新依赖关系的名称。

可以`alias`在他们需要访问具有其他名称的图表的情况下使用。

```bash
# parentchart/Chart.yaml
dependencies:
  - name: subchart
    repository: http://localhost:10191
    version: 0.1.0
    alias: new-subchart-1
  - name: subchart
    repository: http://localhost:10191
    version: 0.1.0
    alias: new-subchart-2
  - name: subchart
    repository: http://localhost:10191
    version: 0.1.0
```
在上面的示例中，我们将获得 3 个依赖项`parentchart`：

```bash
subchart
new-subchart-1
new-subchart-2
```
实现此目的的手动方法是在`charts/`目录中多次复制/粘贴相同的图表，但名称不同。

##  8. 依赖项中的标签和条件字段
除了上述其他字段外，每个需求条目都可能包含可选字段`tags`和`condition`.

默认加载所有图表。如果`tags`或`condition`字段存在，它们将被评估并用于控制它们应用到的图表的加载。

**条件 - 条件字段包含一个或多个 YAML 路径（以逗号分隔）。如果此路径存在于顶级父级的值中并解析为布尔值，则将根据该布尔值启用或禁用图表**。仅评估列表中找到的第一个有效路径，如果不存在路径，则条件无效。

**标签 - 标签字段是与此图表关联的标签的 YAML 列表。在顶部父级的值中，所有带有标签的图表都可以通过指定标签和布尔值来启用或禁用**。

```bash
# parentchart/Chart.yaml
dependencies:
  - name: subchart1
    repository: http://localhost:10191
    version: 0.1.0
    condition: subchart1.enabled, global.subchart1.enabled
    tags:
      - front-end
      - subchart1
  - name: subchart2
    repository: http://localhost:10191
    version: 0.1.0
    condition: subchart2.enabled,global.subchart2.enabled
    tags:
      - back-end
      - subchart2
```

```bash
# parentchart/values.yaml
subchart1:
  enabled: true
tags:
  front-end: false
  back-end: true
```
在上面的示例中，所有带有标签的图表`front-end`都将被禁用，但由于`subchart1.enabled`路径在父值中的计算结果为“真”，因此条件将覆盖`front-end`标签并`subchart1`启用。

由于`subchart2`被标记`back-end`并且该标记评估为`true`,`subchart2`将被启用。另请注意，虽然`subchart2`指定了条件，但父值中没有对应的路径和值，因此条件无效。

使用带有标签和条件的 CLI

 - `--set`可以像往常一样使用该参数来更改标签和条件值。

```bash
helm install --set tags.front-end=true --set subchart2.enabled=false
```
标签和条件解析

 - 条件（当在值中设置时）总是覆盖标签。存在的第一个条件路径获胜，该图表的后续条件路径将被忽略。
 - 标签被评估为“如果图表的任何标签为真，则启用图表”。
 - 标签和条件值必须设置在最高父级的值中。
 - `values` 中的`tags:`键必须是顶级键。tags:当前不支持全局和嵌套表。


##  9. 通过依赖项导入子值
在某些情况下，希望允许子图表的值传播到父图表并作为通用默认值共享。使用该`exports`格式的另一个好处是，它将使未来的工具能够内省用户可设置的值。

可以使用 YAML 列表`dependencies`在字段中的父图表中指定包含要导入的值的键。`import-values`列表中的每个项目都是从子图表`exports`字段导入的键。

要导入`exports`键中未包含的值，请使用子父格式。两种格式的示例如下所述。

###  9.1 exports 格式
如果子图表的`values.yaml`文件在根目录包含一个`exports`字段，则可以通过指定要导入的键将其内容直接导入父值，如下例所示：

```bash
# parent's Chart.yaml file
dependencies:
  - name: subchart
    repository: http://localhost:10191
    version: 0.1.0
    import-values:
      - data
```

```bash
# child's values.yaml file
exports:
  data:
    myint: 99
```
由于我们`data`在导入列表中指定键，Helm 在`exports`子图表的字段中查找`data`键并导入其内容。

最终的父值将包含我们导出的字段：

```bash
# parent's values
myint: 99
```

> 请注意，父键`data`不包含在父级的最终值中。如果需要指定父键，请使用 'child-parent' 格式。


### 9.2 使用子父格式
要访问未包含在`exports`子图表值的键中的值，您需要指定要导入的值的源键 ( child) 和父图表值中的目标路径 ( parent)。

下面`import-values`的示例中的 指示 Helm 获取在`child:`路径中找到的任何值并将它们复制到指定路径中的父值`parent`:

```bash
# parent's Chart.yaml file
dependencies:
  - name: subchart1
    repository: http://localhost:10191
    version: 0.1.0
    ...
    import-values:
      - child: default.data
        parent: myimports
```
在上面的示例中，在子图表 1 `default.data`的值中找到的值将被导入到`myimports`父图表值中的键，详细信息如下：

```bash
# parent's values.yaml file
myimports:
  myint: 0
  mybool: false
  mystring: "helm rocks!"
```

```bash
# subchart1's values.yaml file
default:
  data:
    myint: 999
    mybool: true
```
父图表的结果值为：

```bash
# parent's final values
myimports:
  myint: 999
  mybool: true
  mystring: "helm rocks!"
```
父级的最终值现在包含从 `subchart1` 导入的`myint`和字段。`mybool`

##  10. charts/通过目录手动管理依赖项
`charts/`如果需要对依赖项进行更多控制，可以通过将依赖关系图复制到目录中来明确表达这些依赖关系。

依赖项应该是一个解压缩的图表目录，但它的名称不能以`_or`开头.。图表加载器会忽略此类文件。

例如，如果 `WordPress` 图表依赖于 `Apache` 图表，则 Apache 图表（正确版本）将在 WordPress 图表的`charts/`目录中提供：

```bash
wordpress:
  Chart.yaml
  # ...
  charts/
    apache/
      Chart.yaml
      # ...
    mysql/
      Chart.yaml
      # ...
```
`charts/`上面的示例显示了 `WordPress` 图表如何通过将这些图表包含在其目录中来表达其对 Apache 和 `MySQL` 的依赖。

> 提示： 要将依赖项放入您的`charts/`目录，请使用`helm pull`命令

##  11. 使用依赖项的操作方面
面的部分解释了如何指定图表依赖关系，但是这对使用`helm installand` 的图表安装有何影响helm upgrade？

假设名为“A”的图表创建了以下 Kubernetes 对象

 - 命名空间“A-命名空间”
 - 有状态集“A-StatefulSet”
 - 服务“A-服务”

此外，A 依赖于创建对象的图表 B

 - 命名空间“B-命名空间”
 - 副本集“B-副本集”
 - 服务“B-服务”

安装/升级图表 A 后，将创建/修改一个 `Helm` 版本。该版本将按以下顺序创建/更新所有上述 Kubernetes 对象：

 - A-命名空间
 - B-命名空间
 - A-服务
 - B-服务
 - B-副本集
 - A-StatefulSet

这是因为当 Helm 安装/升级图表时，图表中的 Kubernetes 对象及其所有依赖项都是

 - 聚合成一个集合；然后
 - 按类型后跟名称排序；然后
 - 按该顺序创建/更新。

因此，使用图表及其依赖项的所有对象创建了一个版本。

`Kubernetes` 类型的安装顺序由 `kind_sorter.go` 中的枚举 `InstallOrder` 给出（参见[Helm 源文件](https://github.com/helm/helm/blob/484d43913f97292648c867b56768775a55e4bba6/pkg/releaseutil/kind_sorter.go)）。


##  12. 模板和值
Helm Chart 模板是用[Go 模板语言编写的](https://golang.org/pkg/text/template/)，添加了来自 [Sprig](https://github.com/Masterminds/sprig) 库的 50 个左右的附加模板函数和一些其他专用函数。

所有模板文件都存储在图表的`templates/`文件夹中。当 `Helm` 渲染图表时，它将通过模板引擎传递该目录中的每个文件。

模板的值以两种方式提供：

 - 图表开发人员可以提供一个`values.yaml`在图表内部调用的文件。该文件可以包含默认值。
 - 图表用户可以提供包含值的 YAML 文件。这可以在命令行中使用`helm install`.

当用户提供自定义值时，这些值将覆盖图表`values.yaml`文件中的值。


###  12.1 模板文件
模板文件遵循编写 Go 模板的标准约定（有关详细信息，请参阅`text/template Go` 包文档）。示例模板文件可能如下所示：

```bash
apiVersion: v1
kind: ReplicationController
metadata:
  name: deis-database
  namespace: deis
  labels:
    app.kubernetes.io/managed-by: deis
spec:
  replicas: 1
  selector:
    app.kubernetes.io/name: deis-database
  template:
    metadata:
      labels:
        app.kubernetes.io/name: deis-database
    spec:
      serviceAccount: deis-database
      containers:
        - name: deis-database
          image: {{ .Values.imageRegistry }}/postgres:{{ .Values.dockerTag }}
          imagePullPolicy: {{ .Values.pullPolicy }}
          ports:
            - containerPort: 5432
          env:
            - name: DATABASE_STORAGE
              value: {{ default "minio" .Values.storage }}
```
上面的示例大致基于[https://github.com/deis/charts](https://github.com/deis/charts)，是 Kubernetes 复制控制器的模板。它可以使用以下四个模板值（通常在values.yaml文件中定义）：

 - `imageRegistry`：Docker 映像的源注册表。
 - `dockerTag`：泊坞窗图像的标签。
 - `pullPolicy`：Kubernetes 拉取策略。
 - `storage`：存储后端，默认设置为"`minio`"

所有这些值都由模板作者定义。Helm 不需要或指定参数。

要查看许多工作图表，请查看 CNCF [Artifact Hub](https://artifacthub.io/packages/search?kind=0)。

### 12.2 预定义值
通过`values.yaml`文件（或通过`--set`标志）提供的值可以从`.Values`模板中的对象访问。但是您可以在模板中访问其他预定义的数据。

以下值是预定义的，可用于每个模板，并且不能被覆盖。与所有值一样，名称区分大小写。

 - `Release.Name`：版本名称（不是图表）
 - `Release.Namespace`: 图表发布到的命名空间。
 - `Release.Service`：进行发布的服务。
 - `Release.IsUpgrade`：如果当前操作是升级或回滚，则设置为 `true`。
 - `Release.IsInstall`：如果当前操作是安装，则设置为 `true`。
 - `Chart`:的内容`Chart.yaml`。因此，图表版本是可获得的，`Chart.Version`并且维护者在`Chart.Maintainers`.
 - `Files`：一个类似地图的对象，包含图表中的所有非特殊文件。这不会让您访问模板，但会让您访问存在的其他文件（除非使用排除它们`.helmignore`）。可以使用`{{ index .Files "file.name" }}`或使用该`{{.Files.Get name }}`功能访问文件。您还可以使用以下方式访问文件的[]byte内容`{{ .Files.GetBytes }}`
 - `Capabilities`：一个类似地图的对象，包含有关 Kubernetes 版本 ( `{{ .Capabilities.KubeVersion }}`) 和支持的 Kubernetes API 版本 ( `{{ .Capabilities.APIVersions.Has "batch/v1" }}`)的信息

> 注意：任何未知`Chart.yaml`字段都将被删除。它们将无法在Chart对象内部访问。因此，`Chart.yaml`不能用于将任意结构化的数据传递到模板中。不过，值文件可用于此目的。


### 12.3 Values files
考虑上一节中的模板`values.yaml`，提供必要值的文件如下所示：

```bash
imageRegistry: "quay.io/deis"
dockerTag: "latest"
pullPolicy: "Always"
storage: "s3"
```

值文件采用 `YAML` 格式。图表可能包含默认`values.yaml`文件。`Helm install` 命令允许用户通过提供额外的 YAML 值来覆盖值：

```bash
$ helm install --generate-name --values=myvals.yaml wordpress
```

当以这种方式传递值时，它们将被合并到默认值文件中。例如，考虑一个`myvals.yaml`如下所示的文件：

```bash
storage: "gcs"
```

当它与values.yaml图表中的 合并时，生成的内容将是：

```bash
imageRegistry: "quay.io/deis"
dockerTag: "latest"
pullPolicy: "Always"
storage: "gcs"
```

>  - 请注意，只有最后一个字段被覆盖。
注意：图表中包含的默认值文件必须命名为`values.yaml`. 但是在命令行上指定的文件可以命名为任何名称。
注意：如果在or`--set`上使用该标志，则这些值会在客户端简单地转换为 `YAML`。`helm install`  `helm upgrade`
注意：如果值文件中存在任何必需的条目，则可以使用“必需”函数在图表模板中将它们声明为必需的

然后可以使用该`.Values`对象在模板内部访问这些值中的任何一个：

```bash
apiVersion: v1
kind: ReplicationController
metadata:
  name: deis-database
  namespace: deis
  labels:
    app.kubernetes.io/managed-by: deis
spec:
  replicas: 1
  selector:
    app.kubernetes.io/name: deis-database
  template:
    metadata:
      labels:
        app.kubernetes.io/name: deis-database
    spec:
      serviceAccount: deis-database
      containers:
        - name: deis-database
          image: {{ .Values.imageRegistry }}/postgres:{{ .Values.dockerTag }}
          imagePullPolicy: {{ .Values.pullPolicy }}
          ports:
            - containerPort: 5432
          env:
            - name: DATABASE_STORAGE
              value: {{ default "minio" .Values.storage }}
```

##  12.4 Scope, Dependencies, and Values
`Values files`可以声明顶级图表以及该图表`charts/`目录中包含的任何图表的值。或者，换一种说法，值文件可以为图表及其任何依赖项提供值。例如，上面的演示 WordPress 图表同时具有mysql和apache作为依赖项。值文件可以为所有这些组件提供值：

```bash
title: "My WordPress Site" # Sent to the WordPress template
mysql:
  max_connections: 100 # Sent to MySQL
  password: "secret"
apache:
  port: 8080 # Passed to Apache
```
更高级别的图表可以访问下面定义的所有变量。因此 WordPress 图表可以访问 MySQL 密码为`.Values.mysql.password`. 但是较低级别的图表无法访问父图表中的内容，因此 MySQL 将无法访问该title属性。就此而言，它也不能访问`apache.port`.

值是命名空间的，但命名空间会被修剪。因此对于 `WordPress` 图表，它可以访问 MySQL 密码字段为`.Values.mysql.password`. 但是对于 MySQL 图表，值的范围已经缩小，命名空间前缀被删除，所以它会简单地看到密码字段为`.Values.password`.

##  12.5 Global Values
从 `2.0.0-Alpha.2` 开始，Helm 支持特殊的“`global`”值。考虑上一个示例的修改版本：

```bash
title: "My WordPress Site" # Sent to the WordPress template
global:
  app: MyWordPress
mysql:
  max_connections: 100 # Sent to MySQL
  password: "secret"
apache:
  port: 8080 # Passed to Apache
```
上面添加了一个`global`带有 `value` 的部分`app: MyWordPress`。该值可用于所有图表，如.Values.global.app。

例如，mysql模板可以访问app为`{{ .Values.global.app}}`，apache图表也可以访问。实际上，上面的值文件是这样重新生成的：

```bash
title: "My WordPress Site" # Sent to the WordPress template
global:
  app: MyWordPress
mysql:
  global:
    app: MyWordPress
  max_connections: 100 # Sent to MySQL
  password: "secret"
apache:
  global:
    app: MyWordPress
  port: 8080 # Passed to Apache
```
这提供了一种与所有子图表共享一个顶级变量的方法，这对于设置`metadata`标签等属性很有用。

如果子图表声明了一个全局变量，则该全局变量将向下传递（到子图表的子图表），但不会向上传递到父图表。子图表无法影响父图表的值。

### 12.6 架构 Files
有时，图表维护者可能想要定义其值的结构。这可以通过在`values.schema.json`文件中定义模式来完成。模式表示为JSON 模式。它可能看起来像这样：

 - helm install
 - helm upgrade
 - helm lint
 - helm template

满足此架构要求的文件示例`values.yaml`可能如下所示：

```bash
name: frontend
protocol: https
port: 443
```

> 请注意，模式应用于最终`.Values`对象，而不仅仅是`values.yaml`文件。这意味着以下yaml文件有效，鉴于图表安装了相应的`--set`选项，如下所示。

```bash
name: frontend
protocol: https
```

```c
helm install --set port=443
```
此外，根据所有子图模式检查最终`.Values`对象。这意味着父图表无法规避子图表的限制。这也适用于反向 - 如果子图的文件中未满足子图的要求，则父图必须满足这些限制才能有效。`values.yaml`


##  13. 自定义资源定义 (CRD)
Kubernetes 提供了一种机制来声明新类型的 `Kubernetes` 对象。使用 `CustomResourceDefinitions (CRD)`，Kubernetes 开发人员可以声明自定义资源类型。

在 `Helm 3` 中，`CRD` 被视为一种特殊的对象。它们在图表的其余部分之前安装，并​​且受到一些限制。

`CRD YAML` 文件应放在crds/图表内的目录中。多个 CRD（由 YAML 开始和结束标记分隔）可以放在同一个文件中。Helm 将尝试将CRD 目录中的所有文件加载到 Kubernetes 中。

`CRD` 文件不能被模板化。它们必须是纯 `YAML` 文档。

当 Helm 安装一个新图表时，它会上传 CRD，暂停直到 API 服务器使 CRD 可用，然后启动模板引擎，渲染图表的其余部分，并将其上传到 Kubernetes。由于这种排序，CRD 信息`.Capabilities`在 Helm 模板中的对象中可用，并且 Helm 模板可以创建在 CRD 中声明的对象的新实例。

例如，如果您的图表在目录中有一个 CRD ，您可以`CronTab`在目录中创建该类型的crds/实例：`CronTabtemplates/`

```bash
crontabs/
  Chart.yaml
  crds/
    crontab.yaml
  templates/
    mycrontab.yaml
```
该`crontab.yaml`文件必须包含没有模板指令的 `CRD`：

```bash
kind: CustomResourceDefinition
metadata:
  name: crontabs.stable.example.com
spec:
  group: stable.example.com
  versions:
    - name: v1
      served: true
      storage: true
  scope: Namespaced
  names:
    plural: crontabs
    singular: crontab
    kind: CronTab
```
然后模板`mycrontab.yaml`可能会创建一个新的CronTab（像往常一样使用模板）：

```bash
apiVersion: stable.example.com
kind: CronTab
metadata:
  name: {{ .Values.name }}
spec:
   # ...
```
`Helm` 将确保该`CronTab`类型已安装并且在 `Kubernetes API` 服务器中可用，然后再继续在`templates/.`

### 13.1 CRD 的限制
**与 Kubernetes 中的大多数对象不同，CRD 是全局安装的**。出于这个原因，Helm 在管理 CRD 时采取了非常谨慎的方法。CRD 受到以下限制：

 - 永远不会重新安装 CRD。如果 Helm 确定`crds/`目录中的 CRD 已经存在（无论版本如何），Helm 将不会尝试安装或升级。
 - CRD 永远不会在升级或回滚时安装。Helm 只会在安装操作时创建 CRD。
 - CRD 永远不会被删除。删除 CRD 会自动删除集群中所有命名空间中的所有 CRD 内容。因此，Helm 不会删除 CRD。

鼓励想要升级或删除 CRD 的操作员手动执行此操作并非常小心。

---

✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>


 - [https://www.bookstack.cn/read/helm-3.8.0-en/b5fa667da3c6d162.md](https://www.bookstack.cn/read/helm-3.8.0-en/b5fa667da3c6d162.md)
 - [helm v3.8.0 命令入门指南](https://ghostwritten.blog.csdn.net/article/details/122866004)
 - [helm 快速学习手册](https://ghostwritten.blog.csdn.net/article/details/122882625)

