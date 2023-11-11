
## 1. 创建 CustomResourceDefinition
创建自定义资源，即自定义 restful api
当你创建新的 `CustomResourceDefinition`（CRD）时，Kubernetes API 服务器会为你所 指定的每一个版本生成一个 RESTful 的 资源路径。CRD 可以是名字空间作用域的，也可以 是集群作用域的，取决于 CRD 的 scope 字段设置。和其他现有的内置对象一样，删除 一个名字空间时，该名字空间下的所有定制对象也会被删除。CustomResourceDefinition 本身是不受名字空间限制的，对所有名字空间可用。

例如，如果你将下面的 CustomResourceDefinition 保存到 resourcedefinition.yaml 文件：：

```bash
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # 名字必需与下面的 spec 字段匹配，并且格式为 '<名称的复数形式>.<组名>'
  name: crontabs.stable.example.com
spec:
  # 组名称，用于 REST API: /apis/<组>/<版本>
  group: stable.example.com
  # 列举此 CustomResourceDefinition 所支持的版本,可以设置多个版本.
  versions:
    - name: v1
      # 每个版本都可以通过 served 标志来独立启用或禁止
      served: true
      # 其中一个且只有一个版本必需被标记为存储版本
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                image:
                  type: string
                replicas:
                  type: integer
  # 可以是 Namespaced 或 Cluster,该API的生效范围，可选项为Namespaced（由Namespace限定）和Cluster（在集群范围全局生效，不局限于任何Namespace），默认值为Namespaced
  scope: Namespaced
  names:
    # 名称的复数形式，用于 URL：/apis/<组>/<版本>/<名称的复数形式>要求全部小写
    plural: crontabs
    # 名称的单数形式，作为命令行使用时和显示时的别名,要求全部小写
    singular: crontab
    # kind 通常是单数形式的驼峰编码（CamelCased）形式。你的资源清单会使用这一形式。
    kind: CronTab
    #CRD列表，默认被设置为<kind>List格式
    listKind: CronTablist
    # shortNames 允许你在命令行使用较短的字符串来匹配资源,要求全部小写
    shortNames:
    - ct
    categories:
    - istio-io
    - network-istio-io
```
**常用参数注意**：

 - `versions`：设置此CRD支持的版本，可以设置多个版本，用列表形式表示。目前还可以设置名为version的字段，只能设置一个版本，在将来的Kubernetes版本中会被弃用，建议使用versions进行设置。如果该CRD支持多个版本，则每个版本都会在API URL“`/apis/stable.example.com`”的下一级进行体现，例如“`/apis/stable.example.com/v1`”或“/apis/stable.example.com/v1alpha3”等。每个版本都可以设置下列参数。
 - `categories`：CRD所属的资源组列表。例如，CronTab属于 istio-io组和networking-istio-io组，用户通过查询istio-io组和networkingistio-io组，也可以查询到该CRD实例

```bash
$ kubectl apply -f resourcedefinition.yaml

/apis/stable.example.com/v1/namespaces/*/crontabs/...

$ kubectl get crd | grep crontab
crontabs.stable.example.com                   2020-12-04T07:59:07Z

```
这样一个新的受名字空间约束的 RESTful API 端点会被创建在：

此端点 URL 自此可以用来创建和管理定制对象。对象的 kind 将是来自你上面创建时 所用的 spec 中指定的 `CronTab`。

创建端点的操作可能需要几秒钟。你可以监测你的 `CustomResourceDefinition` 的 Established 状况变为 true，或者监测 API 服务器的发现信息等待你的资源出现在 那里

## 2. 创建定制对象
在创建了 CustomResourceDefinition 对象之后，你可以创建定制对象（Custom Objects）。定制对象可以包含定制字段。这些字段可以包含任意的 JSON 数据。 在下面的例子中，在类别为 CrontTab 的定制对象中，设置了cronSpec 和 image 定制字段。类别 CronTab 来自你在上面所创建的 CRD 的规约。

如果你将下面的 YAML 保存到 my-crontab.yaml：

```bash
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  name: my-new-cron-object
spec:
  cronSpec: "* * * * */5"
  image: my-awesome-cron-image
```

```bash
$ kubectl apply -f my-crontab.yaml
$ kubectl get crontab
NAME                 AGE
my-new-cron-object   6s
```

```bash
$ kubectl get ct -o yaml

你可以看到输出中包含了你创建定制对象时在 YAML 文件中指定的定制字段 cronSpec 和 image：

apiVersion: v1
kind: List
items:
- apiVersion: stable.example.com/v1
  kind: CronTab
  metadata:
    creationTimestamp: 2017-05-31T12:56:35Z
    generation: 1
    name: my-new-cron-object
    namespace: default
    resourceVersion: "285"
    uid: 9423255b-4600-11e7-af6a-28d2447dc82b
  spec:
    cronSpec: '* * * * */5'
    image: my-awesome-cron-image
metadata:
  resourceVersion: ""

```
## 3. 删除 CustomResourceDefinition
当你删除某 CustomResourceDefinition 时，服务器会卸载其 RESTful API 端点，并删除服务器上存储的所有定制对象。

```bash
$ kubectl delete -f resourcedefinition.yaml
$ kubectl get crontabs 
Error from server (NotFound): Unable to list {"stable.example.com" "v1" "crontabs"}: the server could not find the requested resource (get crontabs.stable.example.com)
```
如果你在以后创建相同的 CustomResourceDefinition 时，该 CRD 会一个空的结构.

CustomResource 对象在定制字段中保存结构化的数据，这些字段和内置的字段 apiVersion、kind 和 metadata 等一起存储，不过内置的字段都会被 API 服务器隐式完成合法性检查。有了 OpenAPI v3.0 检查 能力之后，你可以设置一个模式（Schema），在创建和更新定制对象时，这一模式会被用来 对对象内容进行合法性检查。参阅下文了解这类模式的细节和局限性。

在 apiextensions.k8s.io/v1 版本中，CustomResourceDefinition 的这一结构化模式 定义是必需的。 在 CustomResourceDefinition 的 beta 版本中，结构化模式定义是可选的。

设置结构化的模式
CustomResource 对象在定制字段中保存结构化的数据，这些字段和内置的字段 apiVersion、kind 和 metadata 等一起存储，不过内置的字段都会被 API 服务器隐式完成合法性检查。有了 OpenAPI v3.0 检查 能力之后，你可以设置一个模式（Schema），在创建和更新定制对象时，这一模式会被用来 对对象内容进行合法性检查。参阅下文了解这类模式的细节和局限性。

在 apiextensions.k8s.io/v1 版本中，CustomResourceDefinition 的这一结构化模式 定义是必需的。 在 CustomResourceDefinition 的 beta 版本中，结构化模式定义是可选的。

结构化模式本身是一个 OpenAPI v3.0 验证模式，其中：

为对象根（root）设置一个非空的 type 值（藉由 OpenAPI 中的 type），对每个 object 节点的每个字段（藉由 OpenAPI 中的 properties 或 additionalProperties）以及 array 节点的每个条目（藉由 OpenAPI 中的 items）也要设置非空的 type 值， 除非：
节点包含属性 x-kubernetes-int-or-string: true
节点包含属性 x-kubernetes-preserve-unknown-fields: true
对于 object 的每个字段或 array 中的每个条目，如果其定义中包含 allOf、anyOf、oneOf 或 not，则模式也要指定这些逻辑组合之外的字段或条目（试比较例 1 和例 2)。
在 allOf、anyOf、oneOf 或 not 上下文内不设置 description、type、default、 additionalProperties 或者 nullable。此规则的例外是 x-kubernetes-int-or-string 的两种模式（见下文）。
如果 metadata 被设置，则只允许对 metadata.name 和 metadata.generateName 设置约束。

## 4. crd参数
### 4.1 自定义资源-validations
validation这个验证是为了在创建好自定义资源后，通过该资源创建对象的时候，对象的字段中存在无效值，则创建该对象的请求将被拒绝，否则会被创建。我们可以在crd文件中添加“validation:”字段来添加相应的验证机制。

准备 crontab_crd_validation.yml
主要改动为validation部分

```bash
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  # 称必须与下面的spec字段匹配，格式为: <plural>.<group>
  name: crontabs.crd.test.com
spec:
  # 用于REST API的组名称: /apis/<group>/<version>
  group: crd.test.com
  versions:
  - name: v1
    # 每个版本都可以通过服务标志启用/禁用。
    served: true
    # 必须将一个且只有一个版本标记为存储版本。
    storage: true
  scope: Namespaced # 指定crd资源作用范围在命名空间或集群
  names:
    # URL中使用的复数名称: /apis/<group>/<version>/<plural>
    plural: crontabs
    # 在CLI(shell界面输入的参数)上用作别名并用于显示的单数名称
    singular: crontab
    kind: CronTab
    # 短名称允许短字符串匹配CLI上的资源，意识就是能通过kubectl 在查看资源的时候使用该资源的简名称来获取。
    shortNames:
    - ct
  validation:
    openAPIV3Schema:
      properties:
        spec:
          properties:
            cronSpec: #必须是字符串、符合正则规则
              type: string
              pattern: '^(\d+|\*)(/\d+)?(\s+(\d+|\*)(/\d+)?){4}$'
            replicas: #设置副本数的限制
              type: integer
              minimum: 1
              maximum: 10
```
执行：
```bash
$ kubectl apply -f crontab_crd_validation.yml
customresourcedefinition.apiextensions.k8s.io/crontabs.crd.test.com created

$ kubectl get crd | grep crontab
crontabs.crd.test.com                         2020-12-04T08:11:48Z
```
创建不符合的对象：

```bash
apiVersion: crd.test.com/v1
kind: CronTab
metadata:
  name: my-test-crontab
spec:
  cronSpec: "* * * * */10"
  image: my-test-image
  replicas: 15
```
上面的replicas为15 所以创建报错限制的最大值为10；

```bash
$ kubectl apply -f yourcron.yaml 
The CronTab "my-test-crontab" is invalid: spec.replicas: Invalid value: 10: spec.replicas in body should be less than or equal to 10
```
### 4.2 自定义资源-additionalPrinterColumns
从Kubernetes 1.11开始，kubectl使用服务器端打印。服务器决定由kubectl get命令显示哪些列即在我们获取一个内置资源的时候会显示出一些列表信息(比如：kubectl get nodes)。这里我们可以使用CustomResourceDefinition自定义这些列，当我们在查看自定义资源信息的时候显示出我们需要的列表信息。通过在crd文件中添加“additionalPrinterColumns:”字段，在该字段下声明需要打印列的的信息

准备`crontab_crd_additionalPrinterColumns.yml`

```bash
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  # 称必须与下面的spec字段匹配，格式为: <plural>.<group>
  name: crontabs.crd.test.com
spec:
  # 用于REST API的组名称: /apis/<group>/<version>
  group: crd.test.com
  versions:
  - name: v1
    # 每个版本都可以通过服务标志启用/禁用。
    served: true
    # 必须将一个且只有一个版本标记为存储版本。
    storage: true
  scope: Namespaced # 指定crd资源作用范围在命名空间或集群
  names:
    # URL中使用的复数名称: /apis/<group>/<version>/<plural>
    plural: crontabs
    # 在CLI(shell界面输入的参数)上用作别名并用于显示的单数名称
    singular: crontab
    kind: CronTab
    # 短名称允许短字符串匹配CLI上的资源，意识就是能通过kubectl 在查看资源的时候使用该资源的简名称来获取。
    shortNames:
    - ct
  validation:
    openAPIV3Schema:
      properties:
        spec:
          properties:
            cronSpec: #必须是字符串、符合正则规则
              type: string
              pattern: '^(\d+|\*)(/\d+)?(\s+(\d+|\*)(/\d+)?){4}$'
            replicas: #设置副本数的限制
              type: integer
              minimum: 1
              maximum: 10
  additionalPrinterColumns:
  - name: Replicas
    type: integer
    JSONPath: .spec.replicas
  - name: Age
    type: date
    JSONPath: .metadata.creationTimestamp
```
没有`additionalPrinterColumns`之前

```bash
$  kubectl get crontab
NAME                 AGE
my-new-cron-object   4s
```
有`additionalPrinterColumns`之后

```bash
kubectl get crontab
NAME              REPLICAS   AGE
my-test-crontab   2          4s
```
### 4.3 自定义资源-subresources
一般我们要是没有在自定义资源当中配置关于资源对象的伸缩和状态信息的一些相关配置的话，那么在当我们通过该自定义资源创建对象后，又想通过“kubectl scale”来弹性的扩展该对象的容器的时候就会无能为力。而CRD可以允许我们添加该方面的相关配置声明，从而达到我们可以对自定义资源对象的伸缩处理。添加“ subresources:”字段来声明状态和伸缩信息。

准备`crontab_crd_subresources.yml`

```bash
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  # 称必须与下面的spec字段匹配，格式为: <plural>.<group>
  name: crontabs.crd.test.com
spec:
  # 用于REST API的组名称: /apis/<group>/<version>
  group: crd.test.com
  versions:
  - name: v1
    # 每个版本都可以通过服务标志启用/禁用。
    served: true
    # 必须将一个且只有一个版本标记为存储版本。
    storage: true
  scope: Namespaced # 指定crd资源作用范围在命名空间或集群
  names:
    # URL中使用的复数名称: /apis/<group>/<version>/<plural>
    plural: crontabs
    # 在CLI(shell界面输入的参数)上用作别名并用于显示的单数名称
    singular: crontab
    kind: CronTab
    # 短名称允许短字符串匹配CLI上的资源，意识就是能通过kubectl 在查看资源的时候使用该资源的简名称来获取。
    shortNames:
    - ct
  validation:
    openAPIV3Schema:
      properties:
        spec:
          properties:
            cronSpec: #必须是字符串、符合正则规则
              type: string
              pattern: '^(\d+|\*)(/\d+)?(\s+(\d+|\*)(/\d+)?){4}$'
            replicas: #设置副本数的限制
              type: integer
              minimum: 1
              maximum: 10
  additionalPrinterColumns:
  - name: Replicas
    type: integer
    JSONPath: .spec.replicas
  - name: Age
    type: date
    JSONPath: .metadata.creationTimestamp
  subresources:
    scale:
      specReplicasPath: .spec.replicas
      statusReplicasPath: .status.replicas
```
准备test_crontab.yml

```bash
apiVersion: crd.test.com/v1
kind: CronTab
metadata:
  name: my-test-crontab
spec:
  cronSpec: "* * * * */10"
  image: my-test-image
  replicas: 2
```

测试：

```bash
$ kubectl get crontab
NAME              REPLICAS   AGE
my-test-crontab   2          11s

$ kubectl scale --replicas=5 crontabs/my-test-crontab

$ kubectl get crontab
NAME              REPLICAS   AGE
my-test-crontab   5          54s
```
参考资料：
[https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/)
[https://blog.csdn.net/weixin_41806245/article/details/94451734](https://blog.csdn.net/weixin_41806245/article/details/94451734)
