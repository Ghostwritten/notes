通常 `ConfigMaps` 或 `Secrets` 作为配置文件注入到容器中，或者有其他需要滚动 pod 的外部依赖项更改。根据应用程序的不同，如果使用后续helm upgrade的 .

如果另一个文件发生更改，该`sha256sum`函数可用于确保更新部署的注释部分：

```bash
kind: Deployment
spec:
  template:
    metadata:
      annotations:
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
[...]
```
注意：如果您将此添加到库图表中，您将无法访问`$.Template.BasePath`. 相反，您可以使用`{{ include ("mylibchart.configmap") . | sha256sum }}.`

如果您总是想滚动部署，您可以使用与上面类似的注释步骤，而不是用随机字符串替换，这样它总是会更改并导致部署滚动：

```bash
kind: Deployment
spec:
  template:
    metadata:
      annotations:
        rollme: {{ randAlphaNum 5 | quote }}
[...]
```
模板函数的每次调用都会生成一个唯一的随机字符串。这意味着如果需要同步多个资源使用的随机字符串，所有相关资源都需要在同一个模板文件中。

这两种方法都允许您的部署利用内置的更新策略逻辑来避免停机。

> 注意：
> 1. 过去我们建议使用`--recreate-pods`标志作为另一个选项。该标志在 Helm 3 中被标记为已弃用，以支持上述更具声明性的方法。
> 
> 2. Helm 中有一些函数允许您生成随机数据、加密密钥等。这些很好用。但请注意，在升级期间，模板会重新执行。当模板运行生成与上次运行不同的数据时，将触发该资源的更新。

