

**相关阅读**：

 1. [Kubernetes ImagePolicyWebhook与ValidatingAdmissionWebhook【1】动手实践感受区别所在](https://ghostwritten.blog.csdn.net/article/details/119712220)
 2. [KubernetesImagePolicyWebhook与ValidatingAdmissionWebhook【2】Image_Policy.go源码解析](https://ghostwritten.blog.csdn.net/article/details/119811128)
 3. [Kubernetes ImagePolicyWebhook与ValidatingAdmissionWebhook【3】validating_admission.go源码解析](https://ghostwritten.blog.csdn.net/article/details/119979444)
 4. [Kubernetes ImagePolicyWebhook与ValidatingAdmissionWebhook【4】main.go源码解析](https://ghostwritten.blog.csdn.net/article/details/119978143)
 5. [kubernetes 快速学习手册](https://ghostwritten.blog.csdn.net/article/details/108562082)

---------------
## 1. 函数与包的依赖
[https://golang.google.cn/pkg/regexp/](https://golang.google.cn/pkg/regexp/)
[https://pkg.go.dev/github.com/opencontainers/go-digest#Parse](https://pkg.go.dev/github.com/opencontainers/go-digest#Parse)
[https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#ObjectMeta](https://pkg.go.dev/k8s.io/apimachinery/pkg/apis/meta/v1#ObjectMeta)
[https://echo.labstack.com/guide/](https://echo.labstack.com/guide/)
[https://github.com/containers/image/tree/main/docker/reference](https://github.com/containers/image/tree/main/docker/reference)
[https://github.com/containers/image/blob/main/docker/reference/normalize.go](https://github.com/containers/image/blob/main/docker/reference/normalize.go)
[https://github.com/containers/image/blob/14246c8ecba1132af73f7cd86c373c1e7a1726f6/docker/reference/regexp.go](https://github.com/containers/image/blob/14246c8ecba1132af73f7cd86c373c1e7a1726f6/docker/reference/regexp.go)
[https://github.com/containers/image/blob/ac2483ecd4b6cdf67d1eb0505a9a638c27ed1b8c/docker/reference/helpers.go](https://github.com/containers/image/blob/ac2483ecd4b6cdf67d1eb0505a9a638c27ed1b8c/docker/reference/helpers.go)
[https://github.com/containers/image/blob/a7b943d24434256e7aa564f5b074e2d535e884fe/docker/reference/reference.go#L287](https://github.com/containers/image/blob/a7b943d24434256e7aa564f5b074e2d535e884fe/docker/reference/reference.go#L287)

如果你想 深入了解源码的来龙去脉，是十分有必要打开上面的链接。

对于运维人员来说，如何读懂开发写源代码是很痛苦。往往不知如何去理解才好。没有思路，一个接一个的包的各种接口、方法、函数把你搞的晕头转向。那么我们该如何读懂源码呢？

在上一篇文章[Kubernetes ImagePolicyWebhook与ValidatingAdmissionWebhook【1】动手实践感受区别所在](https://blog.csdn.net/xixihahalelehehe/article/details/119712220?spm=1001.2014.3001.5501)，我们熟悉了 `ImagePolicyWebhook`与`ValidatingAdmissionWebhook`准入控制的运用，接下来我们去细细的拆分他们的逻辑。

源代码地址：[https://github.com/kainlite/kube-image-bouncer](https://github.com/kainlite/kube-image-bouncer)

**那么，我们从输出的报错信息出发，逆向推理源代码的逻辑。**也许会是个不错的 原则。

## 2. handler的Image_Policy.go
我们首先看看`ImagePolicyWebhook` 检验效果的报错信息

```bash
$ kubectl describe rc nginx-latest  #警告不允许有latest标签的镜像
  Warning  FailedCreate  8s (x6 over 87s)  replication-controller  (combined from similar events): Error creating: pods "nginx-latest-f7667" is forbidden: image policy webhook backend denied one or more images: Images using latest tag are not allowed
```

搜索关键报错语句：`Images using latest tag are not allowed`

我们在`image_policy.go`的37行找到了
![在这里插入图片描述](https://img-blog.csdnimg.cn/47b2282b93a64845b54fe173af7784ec.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
### `review.Status.Reason`是什么？

![在这里插入图片描述](https://img-blog.csdnimg.cn/5a96ef2dbef74cd7a5416568c7415de1.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
从代码中：

 - `review.Status.Reason`可能是一个字符串类型，存储我们表达的信息。
 - `review.Status`可能是一个接口之类的东西
 - `review`是一个`v1alpha1.ImageReview`类型的变量

#### `v1alpha1.ImageReview`是什么？
从图中我们知道他来自专属[k8s.io/api/imagepolicy/v1alpha](https://pkg.go.dev/k8s.io/api/imagepolicy/v1alpha1)`的包。那我们只能去它的源代码库查看了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/e7bffb6796ee48eb8a5b8393620d15a0.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/11d80a734de64df5b3856d09c06db0de.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
看到这里我们知道了，`v1alpha1.ImageReview`是个结构体，结构体顾名思义是形容一类物体的统称范畴，结构体肯定包含特定属性的组成部分，它即可以抽象的，也可以是具体的。例如：

```bash
type  人 struct {
  中国人  string
  日本人  string
  美国人 string
  .......
}
```
我们还可以这样分：

```bash
type  人 struct {
  黄种人  string
  黑种人 string
  白种人 string
...........
}

type  人  struct {
   名字 string
   年龄  int
   籍贯  string
   人体结构  json
   状态   json
}
```
总之，从某一个角度，根据我们是否需要想怎么分就怎么分。


回到本文中来，`ImageReview`，是描述的什么结构体呢，是一个pod的镜像，有什么内容呢？

 1. `metav1.TypeMeta` 代表的镜像类型的元信息
 2. `metav1.ObjectMeta` 代表的镜像对象的元信息
 3. `Spec ImageReviewSpec`  代表的镜像的细节描述信息
 4. `Status ImageReviewStatus`  代表的镜像的状态

##### `metav1`是表达的什么？
`meta` 是标签的意思，`v1`表达的是版本。
最后发现meta其实也是一个包，它的调用方式是`k8s.io/apimachinery/pkg/apis/meta/v1`
这个包包含了什么？
包v1包含所有版本都通用的API类型，但它们分为两类，一类是对外的，也就是能为我们开发定制的，另一部分是内部的。总之，我们纵观全览，这个包定义了k8s许多对象。我们继续往下看

`metav1.TypeMeta`中的`TypeMeta`包含了什么？
![在这里插入图片描述](https://img-blog.csdnimg.cn/cb5ddbb6dfd04f769648a64b05a06e41.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b1ba385c79984eaead68fc20bfbe0ef1.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
原来，`TypeMeta`也是结构体，里边包含：

 - `kind`: 对象类型，这个我们非常常见，描述资源对象的类型。专业术语描述就是：Kind是一个字符串值，表示该对象所表示的REST资源，不能被更新。
 - `APIversion`: APIVersion定义了对象的这种表示的版本模式。


###### `metav1.ObjectMeta`包含了什么？
这个不用多说，一查就能知道;由于它的描述非常庞大，这里不展示原文注释，只展示这个结构体有什么，让我们心里有个大概。但陌生的内容我会加点注释。

```bash
type ObjectMeta struct {
	Name string `json:"name,omitempty" protobuf:"bytes,1,opt,name=name"`
	#GenerateName是一个可选的前缀，由服务器使用，以生成惟一的name ONLY如果没有提供name字段。
	GenerateName string `json:"generateName,omitempty" protobuf:"bytes,2,opt,name=generateName"`
	Namespace string `json:"namespace,omitempty" protobuf:"bytes,3,opt,name=namespace"`
	#SelfLink是一个表示这个对象的URL，Kubernetes将在1.20版本中停止传播该字段，该字段已经计划好了在1.21版本中被删除
	SelfLink string `json:"selfLink,omitempty" protobuf:"bytes,4,opt,name=selfLink"`
	UID types.UID `json:"uid,omitempty"protobuf:"bytes,5,opt,name=uid,casttype=k8s.io/kubernetes/pkg/types.UID"`
	#ResourceVersion表示该对象的内部版本的不透明值，客户端可使用该值确定对象何时发生更改。可以用于乐观并发、更改检测和对资源或资源集的监视操作。客户端必须将这些值视为不透明的，并将未经修改的值传回服务器。它们可能只对特定的资源或资源集有效。
	ResourceVersion string `json:"resourceVersion,omitempty" protobuf:"bytes,6,opt,name=resourceVersion"`
	#Generation表示所需状态的特定生成的序列号
	Generation int64 `json:"generation,omitempty" protobuf:"varint,7,opt,name=generation"`
	CreationTimestamp Time `json:"creationTimestamp,omitempty" protobuf:"bytes,8,opt,name=creationTimestamp"`
	DeletionTimestamp *Time `json:"deletionTimestamp,omitempty" protobuf:"bytes,9,opt,name=deletionTimestamp"`
	#DeletionGracePeriodSeconds在将该对象从系统中删除之前，允许该对象正常终止的秒数。仅当还设置了deletionTimestamp时设置。可能只会缩短。
	DeletionGracePeriodSeconds *int64 `json:"deletionGracePeriodSeconds,omitempty" protobuf:"varint,10,opt,name=deletionGracePeriodSeconds"`
	#Labels可用于组织和分类(范围和选择)对象的字符串键和值的映射。可以匹配复制控制器和服务的选择器。
	Labels map[string]string `json:"labels,omitempty" protobuf:"bytes,11,rep,name=labels"`
	#注释是一种非结构化的键值映射，存储在资源中，外部工具可以设置该资源来存储和检索任意元数据。它们是不可查询的，在修改对象时应该保留它们。
	Annotations map[string]string `json:"annotations,omitempty" protobuf:"bytes,12,rep,name=annotations"`
	
	#由该对象依赖的对象列表，如果列表中的所有对象都有被删除，该对象将被垃圾回收。如果该对象由控制器管理，那么这个列表中的一个条目将指向这个控制器，当控制器字段设置为true时。管理控制器不能多于一个。
	OwnerReferences []OwnerReference `json:"ownerReferences,omitempty" patchStrategy:"merge" patchMergeKey:"uid" protobuf:"bytes,13,rep,name=ownerReferences"`
#在 k8s 删除对应资源时同时删除关联的外部资源，那么可以通过 Finalizers 来实现。Finalizers 是由字符串组成的列表，当 Finalizers 字段存在时，相关资源不允许被强制删除。存在 Finalizers 字段的的资源对象接收的第一个删除请求设置 metadata.deletionTimestamp 字段的值， 但不删除具体资源，在该字段设置后， finalizer 列表中的对象只能被删除，不能做其他操作。当 metadata.deletionTimestamp 字段非空时，controller watch 对象并执行对应 finalizers 的动作，当所有动作执行完后，需要清空 finalizers ，之后 k8s 会删除真正想要删除的资源。
	Finalizers []string `json:"finalizers,omitempty" patchStrategy:"merge" protobuf:"bytes,14,rep,name=finalizers"`
	ClusterName string `json:"clusterName,omitempty" protobuf:"bytes,15,opt,name=clusterName"`
	#ManagedFields包含了唯一的管理器。 管理器由管理实体自身的基本信息组成，比如操作类型、API 版本、以及它管理的字段。机制追踪对象字段的变化。 当一个字段值改变时，其所有权从当前管理器（manager）转移到施加变更的管理器。 
	ManagedFields []ManagedFieldsEntry `json:"managedFields,omitempty" protobuf:"bytes,17,rep,name=managedFields"`

}
```
内容非常常见，当我们执行`kubectl descibe pod` 或者`kubectl get pod ....-oyaml`就能看到这类信息，作为运维k8s人员，大家应该知道他们的含义。

##### `ImageReviewSpec`作为`spect`的类型有什么？
如下：
```bash
type ImageReviewSpec struct {
     #Containers是正在创建的Pod的每个容器中的信息子集的列表
	Containers []ImageReviewContainerSpec `json:"containers,omitempty" protobuf:"bytes,1,rep,name=containers"`
	#annotation是从Pod的注释中提取的键值对列表。webhook有时会用到它。
	Annotations map[string]string `json:"annotations,omitempty" protobuf:"bytes,2,rep,name=annotations"`
	Namespace string `json:"namespace,omitempty" protobuf:"bytes,3,opt,name=namespace"`
}
```

##### `ImageReviewStatus` 作为`Status`的类型包含了什么？
如下：

```bash
type ImageReviewStatus struct {
    #Allowed表示允许运行所有映像
	Allowed bool `json:"allowed" protobuf:"varint,1,opt,name=allowed"`
	#显示报错信息
	Reason string `json:"reason,omitempty" protobuf:"bytes,2,opt,name=reason"`
	#对象的属性对象将添加AuditAnnotations，允许控制器使用'AddAnnotation'，key应该没有前缀(即，允许控制器将添加一个合适的前缀)。
	AuditAnnotations map[string]string `json:"auditAnnotations,omitempty" protobuf:"bytes,3,rep,name=auditAnnotations"`
}
```
**奥，看到这里终于明白了。回到我们的代码中，原来`review.Status.Reason`是留给我们设置输出报错信息的。**

同时呢，我们也根据`review.Status.Allowed`的源代码库的解释，也明白了它在我们代码中的意义。即是否允许镜像运行的一个判断标志，其实根据代码含义我们也能推测出来。假如`allow=true`，表达运行镜像运行容器，`allow=false`，输出错误，并`break`停止运行程序。
![review.Status.Reason](https://img-blog.csdnimg.cn/985789b30f2d41629bb991c9a6e8822a.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
同时，我们也发现了for循环中的`imageReview.Spec.Containers`，如果你浏览了代码库，定也明白它的含义，那就是显示正在创建的Pod的每个容器中的信息子集的列表。

我们注意到for循环中有一个`container.Image`，它是怎么来的呢？
![在这里插入图片描述](https://img-blog.csdnimg.cn/dd1f9af1b8d94a75b6fc6c17ba864943.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
发现 `ImageReviewContainerSpec`是有结构体的，里边包含`image`,同时，`ImageReviewContainerSpec`作为`containers`的切片对象，所以我们知道`container.Image`只是显示pod里的容器镜像，通过`apped`方法把所有镜像添加`images`切片中，当然，要提前做好切片初始化`images := []string{}`。

## 3.  rule目录的not_latest.go
毫无疑问，代码中`rule`的方法，我们是引用的不是源代码库，而是本项目中的另一个代码文件的方法，它在`rule`目录中。
![在这里插入图片描述](https://img-blog.csdnimg.cn/18e91639bf7f41b9a33484d52e806766.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
rules目录下有：
![在这里插入图片描述](https://img-blog.csdnimg.cn/c32bd4d3e8224c958a1ddb161f471a12.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)


我们找到了，**它不叫方法，它叫函数调用**。我们看看这个函数里写了什么逻辑。
![在这里插入图片描述](https://img-blog.csdnimg.cn/c5f0faa9f8eb4edc8ef08eb42440e3fb.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
传入镜像参数，通过调用`reference.ParseNormalizedNamed`函数返回名字，我们又要看看这个`reference.ParseNormalizedNamed`到底是什么逻辑，能明白它的准确意思，不能靠猜。所以，我们还有继续寻找代码的源头。

我们找到了这个函数所在的目录，具体是哪个文件呢？
![在这里插入图片描述](https://img-blog.csdnimg.cn/ed60675e4a2a40bbad30c0e4d370a674.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
#### normalize.go

找关键词`ParseNormalizedNamed`和`normalize.go`有点关联。
![在这里插入图片描述](https://img-blog.csdnimg.cn/4e1668cbb54d4b8e895a81ab64578234.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
果然找到了，注释翻译是：`ParseNormalizedNamed`将字符串解析为命名引用，将Docker UI中的熟悉名称转换为完全限定引用。如果值可能是一个标识符，则使用`ParseAnyReference`。

没看明白，还是看具体代码逻辑吧

#####  anchoredIdentifierRegexp.MatchString

`anchoredIdentifierRegexp.MatchString`又是什么？在这个`normalize.go`没有发现，通过关键词Regexp找，应该同目录下的`regexp.go`
![在这里插入图片描述](https://img-blog.csdnimg.cn/50e90efec7c749c0be252ff41451a45f.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
果然又找到了，注释翻译：`anchoredIdentifierRegexp`用于检查或匹配标识符值，锚定在字符串的开始和结束。问题又来了，`anchored(IdentifierRegexp)`又是啥，我点慌了，我像个鼹鼠挖地洞。

这个函数翻译描述是通过添加开始和结束分隔符锚定正则表达式。**类似传达一个正则表达式的意思。就是分隔符匹配**

![在这里插入图片描述](https://img-blog.csdnimg.cn/a450c6eedbf94f80857a0f6492a63e7a.png)
`regexp.Regexp`是包`regexp`的结构体，看看是啥,是个空结构体，**空结构体不占据内存空间，因此被广泛作为各种场景下的占位符使用。一是节省资源，二是空结构体本身就具备很强的语义，即这里不需要任何值，仅作为占位符。**
![在这里插入图片描述](https://img-blog.csdnimg.cn/3431bb16a5534fc3bc41893c471f726b.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/a7eabf3dcdcf4c8a98b7b5be730af3b4.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
奥，看到这里明白一件事，`MatchString`是`Regexp`的方法，也就是，`Regexp.MatchString`是可以调用，而`Regexp`结构体本身是`anchored`函数的传参，`anchored(ShortIdentifierRegexp)`又被定义`anchoredShortIdentifierRegexp`，所以，根据go语言的传递规则，anchoredShortIdentifierRegexp可以直接调用`Regexp`的`MatchString`，即`anchoredIdentifierRegexp.MatchString(s)`，它是什么意思呢？那就 得明白anchored的return的正则匹配规则，负责检查s（字符串）格式是否负责这个规则。也就是说看看这个规则`match(`^` + expression(res...).String() + `$`)`是什么意思。
代码中

```bash
var match = regexp.MustCompile
```
也就是

```bash
regexp.MustCompile(`^` + expression(res...).String() + `$`)
```

 - ^ 代表匹配首
 - $ 代表匹配尾

![在这里插入图片描述](https://img-blog.csdnimg.cn/eadc4a6a3b7644efa90a55d60677ff3f.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
express函数对传递的res的正则符合字符串化，并与字符串s拼接。这里好像是闭包函数的使用。与我们的代码中使用的场景联想一下，正是镜像的格式例如：`nginx:latest` 、`kainlite/kube-image-bouncer:latest`，其实**express作用的就是镜像中特殊字符为了避免被识别为正则表达式**。

回头来看，`nchoredIdentifierRegexp.MatchString(s)`的作用就是判断s是不是一个镜像。
![在这里插入图片描述](https://img-blog.csdnimg.cn/3db055bdc4e14c6494de0ab6f6bec708.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
#####  splitDockerDomain(s)
`splitDockerDomain(s)`在第90行，通过字段其实大概就能明白它的意思。字段切分。具体怎么实现的呢？
![在这里插入图片描述](https://img-blog.csdnimg.cn/437337723baa406c950253d88f4400e4.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
`strings.IndexRune(name, '/')`方法，可以参考[go strings方法](https://ghostwritten.blog.csdn.net/article/details/105097087)，`IndexRune`返回字符`/`在字符串s中第一次出现的位置,如果找不到，则返回-1。`domain`就等于`docker.io`,在代码中已初始化好，`name`等于`remainder`
![在这里插入图片描述](https://img-blog.csdnimg.cn/b2f1a94e813842489a0062a5677e8368.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
如果镜像中有‘/’呢，例如：`kainlite/kube-image-bouncer:latest`,`domain`取`kainlite`,`remainder`取`kube-image-bouncer:latest`

假如`domain`等于`index.docker.io`，就重定义为`docker.io`

假如domain等于docker.io,并且再次确认remainder不包含‘/’,就将remainder再次重新定义 `officialRepoName + "/" + remainder`


`splitDockerDomain(s)`函数分析完毕。继续`ParseNormalizedNamed(s string) (Named, error)`的分析：

定义个`remoteName` 

tagSep := strings.IndexRune(remainder, ':')，即确认：的位置，如果＞-1，即如果存在，则
tagSep等于镜像的标签，否则，就等于镜像的名字

`strings.ToLower`将s中的所有字符修改为其小写格式,如果remotename存在大写的话则不符合仓库名字规范。

##### parse

 `Parse(domain + "/" + remainder)`是什么意思呢?
 

![在这里插入图片描述](https://img-blog.csdnimg.cn/56c5188bc74147b7a84eafdfbd0ea6de.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

又是好大的一个函数。
那我们慢慢来分析吧
######  referenceRegexp.FindStringSubmatch(s)
`referenceRegexp.FindStringSubmatch(s)`是啥意思？翻译过来就是`ReferenceRegexp`是完全支持的引用格式。regexp被锚定，并具有名称、标记和摘要组件的捕获组。描述看不懂。还是具体代码逻辑吧
![在这里插入图片描述](https://img-blog.csdnimg.cn/43dc021d4bc24d8cbfd47271c719aef8.png)
`anchored`大体明白了，就是通过添加开始和结束分隔符锚定正则表达式并作为返回值。![在这里插入图片描述](https://img-blog.csdnimg.cn/fdac8671358f45edb0ce5d29b9481cb7.png)将含有正则表达式字符的字符串转换字符串形式。

`capture`函数与`anchored`大同小异，只是将含有正则表达式字符的字符串转换数组形式并作为返回值。
![在这里插入图片描述](https://img-blog.csdnimg.cn/3c96ae53cab74d54ad6f5c05e6ea1634.png)

`optional`似乎是对字符串可有可无的一种设定,并且`optional`字面意思是可选的。
![在这里插入图片描述](https://img-blog.csdnimg.cn/faf055fa4dc3490ba75e0d2a538a3e0e.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/63f02d9ad5b44c73952bc9a8232cbf21.png)

`literal`函数是什么意思？翻译是**将文本编译为文本正则表达式，转义任何regexp保留字符**。字面非常容易理解。
![在这里插入图片描述](https://img-blog.csdnimg.cn/ac2266313cf14df9b2a1f71a5463b522.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

 - `QuoteMeta`返回一个字符串，该字符串转义参数文本中的所有正则表达式元字符;返回的字符串是与文本匹配的正则表达式。
 - var match = regexp.MustCompile
 - re := regexp.MustCompile(正则字符串)
 - re就是单纯的正则表达式了。
 - ![在这里插入图片描述](https://img-blog.csdnimg.cn/f5a1d0593f9341779d7d112ad2ff285a.png)
`LiteralPrefix`返回一个字面值字符串，该字符串必须以正则表达式re的任何匹配项开始。如果字面值字符串包含整个正则表达式，则返回布尔值true。
![在这里插入图片描述](https://img-blog.csdnimg.cn/a5815a9a281446afb61bc6d4d92c5aec.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

后面的函数方法其实大同小异，这里不一一拆解。
回到函数`ReferenceRegexp.FindStringSubmatch(s)`，`FindStringSubmatch(s)`是regexp的方法`FindStringSubmatch`返回一个字符串切片，其中包含正则表达式在s中最左匹配的文本，以及它的子表达式的匹配(如果有的话)，如包注释中的'Submatch'描述中定义的那样。返回值为nil表示没有匹配。

`ReferenceRegexp`函数其实设定了一个正则表达式，使字符串s能够通过我们希望的分割的方式，形成一个切片。它的关键之处是获取s的正则表达式，融入到设定的规则之中。

 - 假如matches为nil，报错
 - 假如matches[1]长度＞255,也会报错

```bash
nameMatch := anchoredNameRegexp.FindStringSubmatch(matches[1])
```
这段代码其实是对`domain`的再度切片，如果它较多的递级目录，那我们就要对目录与仓库名称做出区分。

**这里定义了一个关于镜像仓库名与标签的结构体，这是非常重要的一步。**

```bash
ref := reference{
	namedRepository: repo,
	tag:             matches[2],
}
```
 `matches[3]`是什么？ `matches[2]`是镜像的版本标签，在这之后，又是通过@分割当然是一个哈希字符串的内容，是可有可无的。所以有下面的一个判断。

```bash
if matches[3] != "" {
	var err error
	ref.digest, err = digest.Parse(matches[3])
	if err != nil {
		return nil, err
	}
}
```
digest包正是处理哈希字符串的。
![在这里插入图片描述](https://img-blog.csdnimg.cn/7cc778e657fb4e928bd2157f36b1bb7a.png)

`getBestReferenceType(ref)`是什么？字面推断是对`ref`再次处理。
![在这里插入图片描述](https://img-blog.csdnimg.cn/e47ebe71058545a8b4e96137a9ad1171.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
对包含镜像的仓库、标签、哈希值的ref结构体再次检查判断，有什么输出什么。并重构还有该特性的结构体。一共有三类结构体。如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/54424ce7e98a479c8068298d087fca0a.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
到这里，关于`parse`的函数分析已经结束了，这一路下来学到了不少方法，不能说全盘了解，但调用的机理已经摸透。

`named, isNamed := ref.(Named)`是个非常有趣的值得摸索记住的。**我不懂，结构体跟点跟括号。后续研究。**，但根据下面的代码逻辑推理，可能：

```bash
  r.Name() + ":" + r.tag + "@" + r.digest.String()
```

**`Named`是管道，ref是结构体r`eference`结构体的定义，那么自然会承接`Reference`管道中的`refence`的`String()`**
![在这里插入图片描述](https://img-blog.csdnimg.cn/9b4ed87f120c484488f108c4d5f160e7.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/08aee34d5cf04e00b91a8d75ac18c409.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/bc711050f48a4ef3ba84c9e1e4533643.png)

![named](https://img-blog.csdnimg.cn/e4042c417e314ac8bde8ff1486036f67.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/55e57042306b435da925d9cdb9ef95ae.png)


剩下要做的就是看看这个parse函数返回值被如何用了，自然可以推理出来，如下

![在这里插入图片描述](https://img-blog.csdnimg.cn/b10f3f3d1fc94af2a5b53cb097d0573e.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
`strings.HasSuffix`判断字符串s是否以某个字符串结尾,

![在这里插入图片描述](https://img-blog.csdnimg.cn/1407286a1cfc40528d69b35735ed6124.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
`TagNameOnly`函数很好理解，就是判断镜像有没有标签，如果没有标签就添加默认"`latest`"。
那我们来看看他怎么实现的吧。

`IsNameOnly`函数在`helpers.go`![在这里插入图片描述](https://img-blog.csdnimg.cn/41a6e37b18ba4b5d8360b287082af830.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
来到这里我们又遇到了`ref.(NamedTagged)`，看到这里，也许有一丝明白过来，这个对ref的结构体属性的判断，根据结构体中的属性是否包含值，通过接口组合成一个判断类型。之前，`named, isNamed := ref.(Named)`代表返回一个完整的镜像+标签的内容，并判断是否正常返回，这里代表是否只返回标签，也就是只判断是否有标签。下面的`ref.(Canonical)`则是判断标签是否有哈希值。
![在这里插入图片描述](https://img-blog.csdnimg.cn/5ac5ebfca23f4f0aad73233e6799922e.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/8a80395b8d76421a9c057e976ae1044b.png)
最后，如果没有任何标签，即为true，继续执行`namedTagged, err := WithTag(ref, defaultTag)`
![amedTagged, err := WithTag(ref, defaultTag)](https://img-blog.csdnimg.cn/a24e50093ab44332a20fd452928ad581.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
`withtag`函数传递两个函数，获取的镜像的信息，和latest标签。
![在这里插入图片描述](https://img-blog.csdnimg.cn/a945e18b011849d286af03f1d357851a.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

这个对已经获取的`Named`的对象重新进行定义，目的是为了把默认没有标签的镜像打上`latest`标签。

当`withtag`函数执行结束，TagNameOnly(named)也就返回新的镜像结构体，然后通过String()方法，字符串化，`Strings.HasSuffix`方法判断这个字符串是否以`:latest`结尾。根据正确与否返回true或false。

`IsUsingLatestTag`函数的逻辑终于也就结束了。

回到我们的[image_policy](https://github.com/kainlite/kube-image-bouncer/blob/master/handlers/image_policy.go)代码中来。

下面就是根据判断是否有latest做出终端日志输出判断，并且打个是否allow的标签。

我们完成了哪些部分的代码分析呢？真的是九牛一毛。
![在这里插入图片描述](https://img-blog.csdnimg.cn/c62dc3d82f5843049a7f0dbb359225de.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 4. echo包

下面只需要分析明白如下内容。
![在这里插入图片描述](https://img-blog.csdnimg.cn/56918ed697694654a683f0b0d0d9fcab.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
`echo` web框架是go语言开发的一种高性能，可扩展，轻量级的web框架。
echo框架真的非常简单，几行代码就可以启动一个高性能的http服务端。
`echo.Context`表示当前 HTTP 请求的上下文。它保存请求和响应引用、路径、路径参数、数据、注册处理程序和 API 以读取请求和写入响应。由于 `Context` 是一个接口，因此很容易使用自定义 API 对其进行扩展。
`handlerFunc`定义http请求，匿名函数像变量一样返回给`echo.HandlerFunc`

![在这里插入图片描述](https://img-blog.csdnimg.cn/b23cd0a5aa2b48a092abeacaac5db7d1.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)



`c.bind(&imageReview)`意思是通过将请求参数绑定到一个名叫`imageReview`的struct对象的方式获取数据。这种方式获取请求参数支持json、xml、k/v键值对等多种方式。

**该函数最终返回一个json格式的pod拉去一个镜像的属性信息，并伴随http请求状态。**

分析到这里，我们完成了对`Image_Policy.go`的步步分析，也深刻的体会到读源码的重要性。

## 5. 总结
我对go有了以下感悟：

 1. **go的logo是一只土拨鼠，土拨鼠的习性是挖地洞，因此，我们在学习go的时候要有土拨鼠的挖洞精神，在每个包、每个函数、每个方法、每个接口、每个结构体找到它最真实的面孔**。
 2. echo web框架的使用，[请参考这篇文章](https://echo.labstack.com/guide/)
 3. **匿名函数作为返回值的使用**

匿名函数由一个不带函数名的函数声明和函数体组成。匿名函数的优越性在于可以直接使用函数内的变量，不必申明。这是它最大的特点。这里将一个函数当做一个变量一样的操作作为函数的返回值输出给`echo.HandlerFunc`

 4. 依赖包的结构体具体特性进行变量声明，方便简化代码的逻辑。例如：`var imageReview v1alpha1.ImageReview`

谁在调用`image_policy.go` ，是[main.go](https://github.com/kainlite/kube-image-bouncer/blob/master/main.go),在下一篇接文章，我们慢慢分析它的逻辑是什么。




