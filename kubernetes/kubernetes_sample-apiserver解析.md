##  简介
`kubernetes/apiserver`同步自`kubernertes`主代码树的`taging/src/k8s.io/apiserver`目录，它提供了创建K8S风格的API Server所需要的库。包括`kube-apiserver`、`kube-aggregator`、`service-catalog`在内的很多项目都依赖此库。

`apiserver`库的目的主要是用来构建`API Aggregation`中的`Extension API Server`。它提供的特性包括：

 1. 将`authn/authz`委托给主`kube-apiserver`
 2. 支持kuebctl兼容的API发现
 3. 支持`admisson control`链
 4. 支持版本化的API类型


K8S提供了一个样例`kubernetes/sample-apiserver`，但是这个例子依赖于主`kube-apiserver`。即使不使用`authn/authz`或API聚合，也是如此。你需要通过`--kubeconfig`来指向一个主`kube-apiserver`，样例中的`SharedInformer`依赖于会连接到主kube-apiserver来访问K8S资源。

如果您想构建一个扩展 API 服务器以与 API 聚合一起使用，或者构建一个独立的 Kubernetes 风格的 API 服务器，您可以使用此代码。

但是，请考虑另外两个选项：

 - `CRD`：如果您只想向 kubernetes 集群添加资源，请考虑使用 `Custom Resource Definition aka CRD`。它们需要更少的编码和变基。[在此处](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)阅读自定义资源定义与扩展 API 服务器之间的差异。
 - `Apiserver-builder`：如果您想构建一个扩展 API 服务器，请考虑使用[apiserver-builder](https://github.com/kubernetes-sigs/apiserver-builder-alpha)而不是这个repo。Apiserver-builder 是一个完整的框架，用于生成 `apiserver`、客户端库和安装程序。

