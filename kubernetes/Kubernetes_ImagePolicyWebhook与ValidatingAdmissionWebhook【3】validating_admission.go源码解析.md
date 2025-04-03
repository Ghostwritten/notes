


**相关阅读**：

 1. [Kubernetes ImagePolicyWebhook与ValidatingAdmissionWebhook【1】动手实践感受区别所在](https://ghostwritten.blog.csdn.net/article/details/119712220)
 2. [KubernetesImagePolicyWebhook与ValidatingAdmissionWebhook【2】Image_Policy.go源码解析](https://ghostwritten.blog.csdn.net/article/details/119811128)
 3. [Kubernetes ImagePolicyWebhook与ValidatingAdmissionWebhook【3】validating_admission.go源码解析](https://ghostwritten.blog.csdn.net/article/details/119979444)
 4. [Kubernetes ImagePolicyWebhook与ValidatingAdmissionWebhook【4】main.go源码解析](https://ghostwritten.blog.csdn.net/article/details/119978143)
 5. [kubernetes 快速学习手册](https://ghostwritten.blog.csdn.net/article/details/108562082)

-----------

## 1. 代码依赖
[https://pkg.go.dev/k8s.io/api/admission/v1beta1#AdmissionReview](https://pkg.go.dev/k8s.io/api/admission/v1beta1#AdmissionReview)
[https://pkg.go.dev/k8s.io/api/core/v1#PhotonPersistentDiskVolumeSource.XXX_Size](https://pkg.go.dev/k8s.io/api/core/v1#PhotonPersistentDiskVolumeSource.XXX_Size)
[https://pkg.go.dev/k8s.io/api/imagepolicy/v1alpha1#ImageReview](https://pkg.go.dev/k8s.io/api/imagepolicy/v1alpha1#ImageReview)

源码：[https://github.com/kainlite/kube-image-bouncer](https://github.com/kainlite/kube-image-bouncer)

## 2. handler的validating_admission.go
我们分析完了`handler`的`Image_Policy.go`，接下来运用同样的方式分析`validating_admission.go`

我们看一下报错信息

```bash
$ kubectl describe rc nginx-latest  #警告不允许有latest标签的镜像
.....
Warning  FailedCreate  11s (x14 over 53s)  replication-controller  Error creating: admission webhook "image-bouncer-webhook.default.svc" denied the request: Images using latest tag are not allowed
```
搜索关键词：`Images using latest tag are not allowed`

我们看的在`validating_admission.go`的51行找到了



![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e1dfd16fdf5537c25e20cd564e1e35b3.png)
### 2.1 metav1.status是什么？


我们首先看一下代码的依赖去向
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f0bbe8842141bbc808baaf69235e84b5.png)
metav1是什么包，还记得在上一篇文章我提到过吧
meta 是字面标签的意思，v1表达的是版本。
包v1包含所有版本都通用的API类型，但它们分为两类，一类是对外的，也就是能为我们开发定制的，另一部分是内部的。总之，我们纵观全览，这个包定义了k8s许多资源对象。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e4b9f166b90faa3ae6345c0eb56e050f.png)
`status.message`这里描述的是设定操作状态的描述。根据代码功能需求的描述`"Images using latest tag are not allowed"`，正是我们需要表达的。

### 2.2 admissionReview.Response.Result是什么？
它的来源是"`k8s.io/api/admission/v1beta1`"，地址是[https://pkg.go.dev/k8s.io/api/admission/v1beta1](https://pkg.go.dev/k8s.io/api/admission/v1beta1)，搜索`AdmissionReview`
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4d0e0fde306b7e25d34aaf1eece3fb68.png)

`AdmissionReview`与`ImageReview`大同小异，这是定义对象类型、请求属性、返回属性。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3a5fe6c622068df0abd988f861b5184b.png)

`admissionReview.Response.Allowed` 是一个布尔值
`admissionReview.Response.Result`代表的是`*metav1.Status`类型，然而status的结构体又是包含什么呢？如图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/aa7996cfc96960737c7934108836ff51.png)
`meta.status.message`表达的是对此状态的一种描述。正好对于代码的输出内容：`"Images using latest tag are not allowed"`

再往上分析一下`usingLatest`，即假如存在usingLatest，就将是否允许运用拉取的布尔值设置为false，并输出一个关于存在`lastest`的描述性问题。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8368c20adcb813be014d042c45b71fb8.png)
`usingLatest`来自`rules.IsUsingLatestTag(container.Image)`，根据上篇文章[Kubernetes ImagePolicyWebhook与ValidatingAdmissionWebhook【2】Image_Policy.go源码解析](https://blog.csdn.net/xixihahalelehehe/article/details/119811128?spm=1001.2014.3001.5501)，详细的分析，此函数

 - 当镜像仓库名称不对的时候返回：`false，err`；
 - 当存在`lastest`的时候返回：`true，nil`；
 - 当仓库名正确并且不存在`lastest`的时候返回：`false，nill`

这个有一个与`Image_Policy.go`不一样的地方，即镜像的获取方式。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/33df46873cb2e27a424d4e9130e2e8e6.png)
那么它们的区别是什么？
`pod`来自`v1.Pod{}`,依赖包`k8s.io/api/core/v1`，`v1`代表稳定版本。
`imageReview`来自`v1alpha1.ImageReview`，`aplha1`代表随时可能抛弃或者存在bug的版本，`ImageReview`这个结构体主要的目的是为了存放检查pod的镜像的各种存在可能性结果。而pod这个结构体主要的目的是为了存放或者说是描述运行pod的各种信息，包括容器一系列的细节。

为什么他们获取的容器镜像不一样呢，自然是针对不同的准入控制对象有已经成熟的依赖包。根据他们各自所需，通过各自的API获取镜像的方式也就大同小异，回想当初我们部署`ImagePolicyWebhook`与`ValidatingAdmissionWebhook`的时候，`ImagePolicyWebhook`的针对资源对象是`image`，依赖的包自然是`k8s.io/api/imagepolicy/v1alpha1`，而`ValidatingAdmissionWebhook`则是部署一个k8s资源对象`pod`，自然依赖的包`k8s.io/api/admission/v1beta1`。


回到`validating_admission.go`，我们自上而下推断一下这个文件代码的整体流程。

 1. `c.bind`即是echo  web客户端绑定的方法，为了输出打印`admissionReview`结构体的相关性信息。根据是否报错，输出相关返回结果。
 2. `pod := v1.Pod{}`是一个获取pod列表的方法。
 3. 对`admissionReview.Response`进行初始化，并根据是否有`lastest`以及是否来自健康的仓库即代码里标注是白名单，修改标志性状态。
 4. 最后打印接受或者拒绝此镜像的信息，并通过`http`返回`admissionReview`的json格式。

那么这里有一个问题来了。`RegistryWhitelist`的逻辑判断是怎么实现的呢？
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0d5ac4b9dd80d7bdf91674c681f30ec2.png)
`rules.IsFromWhiteListedRegistry`来自rules目录下from_whitelisted_registry.go，**rules目录下的代码是定制逻辑判断规则细节的地方。而`validating_admission.go`整体看来是对此规则运行的一层包装。**
代码如下：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dc715d1cab21128d992a8e0a040f59b8.png)

 - `reference.ParseNormalizedNamed(image)`在上篇文章中，我们已经做过分析。是对镜像格式严谨式的一次检查，并最终返回它完整的名字。即 `r.Name() + ":" + r.tag + "@" + r.digest.String()`
 - `res := strings.SplitN(named.Name(), "/", 2)`指定切割成两个字符串，如果`len(res)不等于2`表示没有仓库名，返回false
 - 最后如果有仓库名呢，遍历res的字符串，判断是否与`whitelist`，即符合白名单的仓库名，返回`true，nil`


终于，`validating_admission.go`的代码分析结束。我们做一下总结吧。
## 3. 总结
1. 关于k8s的包有很多，我们无法记住它全部的使用的方式，但是我们可以先弄清楚知道包名就能推测它的作用，包含了哪些可能的方法。另外，源码包文档有清晰的关于结构体、方法、函数的目录。多看关于项目中涉及到的k8s的应用对提高k8s开发大有裨益，有些常用的方法我可以积累去保存作为笔记，比如：`c.Bind(&imageReview)`、`for _, container := range imageReview.Spec.Containers`、`rules.IsUsingLatestTag(container.Image)`、`for _, container := range pod.Spec.Containers`、`admissionReview.Response.Result = &metav1.Status`等等。
2. 关于准入控制`ImagePolicyWebhook`与`ValidatingAdmissionWebhook`的开发，目前大概有了个思路，他们有已成熟的`API`包，根据我们的需求依赖包可以满足我们如何去开发，当然，提供给`api-server`去调用做出检测，我们就需要一个`URL`接口，而`echo` 这个依赖包恰好是非常便利的处理绑定k8s资源信息。除此之外，运用的则是处理输出格式的依赖应用包或者逻辑判断所需的成熟接口方法的包。


