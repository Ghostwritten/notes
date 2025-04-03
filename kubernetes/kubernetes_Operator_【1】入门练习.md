![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0620282117953916526d0b09683c3548.png)

---

## 1.Operator SDK是什么



`Operator` 是由 CoreOS 开发的，用来扩展 Kubernetes API，特定的应用程序控制器，它用来创建、配置和管理复杂的有状态应用，如数据库、缓存和监控系统。Operator 基于 Kubernetes 的资源和控制器概念之上构建，但同时又包含了应用程序特定的领域知识。**创建Operator 的关键是`CRD（自定义资源）`的设计。**

`Kubernetes 1.7` 版本以来就引入了自定义控制器的概念，该功能可以让开发人员扩展添加新功能，更新现有的功能，并且可以自动执行一些管理任务，这些自定义的控制器就像 Kubernetes 原生的组件一样，Operator 直接使用 Kubernetes API进行开发，也就是说他们可以根据这些控制器内部编写的自定义规则来监控集群、更改 Pods/Services、对正在运行的应用进行扩缩容。

仅有CRD的定义并没有实际作用，用户还需要提供管理CRD对象的CRD控制器（`CRD Controller`），才能实现对CRD对象的管理。CRD控制器通常可以通过Go语言进行开发，并需要遵循Kubernetes的控制器开发规范，基于客户端库client-go进行开发，需要实现`Informer`、`ResourceEventHandler`、`Workqueue`等组件具体的功能处理逻辑，详细的
开发过程请参考官方示例（[https://github.com/kubernetes/samplecontroller](https://github.com/kubernetes/samplecontroller)）和client-go库（[https://github.com/kubernetes/samplecontroller/blob/master/docs/controller-client-go.md](https://github.com/kubernetes/samplecontroller/blob/master/docs/controller-client-go.md)）的详细说明。

## 2. 基本概念
首先介绍一下本节所涉及到的基本概念。

 - `CRD` (Custom Resource Definition)：允许用户自定义 Kubernetes 资源，是一个类型；
 - `CR` (Custom Resourse)：CRD 的一个具体实例；
 - `webhook`：它本质上是一种 HTTP 回调，会注册到 apiserver 上。在 apiserver 特定事件发生时，会查询已注册的
   webhook，并把相应的消息转发过去。

按照处理类型的不同，一般可以将其分为两类：一类可能会修改传入对象，称为 `mutating webhook`；一类则会只读传入对象，称为 `validating webhook`。

 - `工作队列`：controller 的核心组件。它会监控集群内的资源变化，并把相关的对象，包括它的动作与 key，例如 Pod 的一个Create 动作，作为一个事件存储于该队列中；
 - `controller`：它会循环地处理上述工作队列，按照各自的逻辑把集群状态向预期状态推动。不同的 controller处理的类型不同，比如 replicaset controller 关注的是副本数，会处理一些 Pod 相关的事件；
 - `operator`：operator 是描述、部署和管理 kubernetes 应用的一套机制，从实现上来说，可以将其理解为 CRD配合可选的 webhook 与 controller 来实现用户业务逻辑，即 operator = CRD + webhook + controller。

## 3. 工作模式
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bc21da920c59a12f2fa34d64182fff53.png#pic_center)


## 4.工作流程
SDK提供以下工作流程来开发新的Operator：

 1. 使用SDK命令行界面（CLI）创建新的Operator项目
 2. 通过添加自定义资源定义（CRD）定义新资源API
 3. 使用SDK API监控指定的资源
 4. 在指定的处理程序中定义Operator协调逻辑(对比期望状态与实际状态)，并使用SDK API与资源进行交互
 5. 使用SDK CLI构建并生成Operator部署manifests

Operator使用SDK在用户自定义的处理程序中以高级API处理监视资源的事件，并采取措施来reconcile（对比期望状态与实际状态）应用程序的状态。

## CRD关系图
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/efafce5ee55d7d1b0c7702cad97b77ef.png)

 - 1）`CRD` 
   CRD全称是CustomResourceDefinition，即自定义资源。CRD也是K8s的一种资源，创建一个CRD即在K8s中定义了一种新的资源类型，这个资源类型可以像K8s中的原生资源一样，既可以通过kubectl命令行，也可以通过访问apiserver来进行操作。
 - 2）`Resource Event` 这里的Event是当资源本身发生变化时触发的事件，并不是K8s中的Event资源。 共有四种类型，CreateEvent，UpdateEvent，DeleteEvent，GenericEvent。其中GenericEvent用来处理未知类型的Event，比如非集群内资源事件，一般不会使用。如果控制器"订阅"了这个资源，那么资源发生变化时，比如被更新或者被删除时，控制器会获取到这个事件。Event是联系控制器和资源的数据通道。
 - 3）`controller-runtime`
   controller-runtime被用来创建K8s资源控制器，如果引入了CRD的话，单纯定义这个资源只能起到存数据的作用，并没有业务处理逻辑。通过controller-runtime可以监听资源的变化，捕获Resource Event，触发相应的处理流程，让这个自定义资源表现出和原生资源相同的行为。
 - 4）`kubebuilder`
   kubebuilder是一个根据模板生成代码的工具，使用kubebuilder可以快速渲染出一个依赖`controller-runtime`的控制器。在分析controller-runtime之前，需要先用它来生成一个controller。

为了使CRD像原生资源那样工作，需要创建对应的控制器(controller)，这个控制器需要捕获资源发生变化时的事件，完成指定的操作。理解了CRD的使用方法和运行原理，这样遇在到问题时，才能够方便定位和解决。

## 5.预备条件

 1. ​[dep](https://golang.github.io/dep/docs/installation.html) version v0.5.0+.
 2. ​[git​](https://git-scm.com/downloads)
 3. ​[go](https://golang.org/dl/) version v1.10+.
 4. [​docker](https://docs.docker.com/install/) version 17.03+.
 5. [​kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) version v1.10.0+.
 6. Access to a kubernetes v.1.10.0+ cluster.



## 6.快速开始
### 6.1 编译安装operator-sdk
首先，checkout并安装operator-sdk CLI:

```bash
$ mkdir -p $GOPATH/src/github.com/operator-framework
$ cd $GOPATH/src/github.com/operator-framework
$ git clone https://github.com/operator-framework/operator-sdk
$ cd operator-sdk
$ git checkout master
$ make dep
$ make install
```
### 6.2 二进制安装
下载地址：
[https://github.com/operator-framework/operator-sdk/tags](https://github.com/operator-framework/operator-sdk/tags)
选择好版本
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5f91143d2ce3285951e9a7c4f6b9c059.png)

### 创建app-operator项目
使用SDK CLI创建和部署app-operator:

```c
# 1.创建一个定义App 用户资源的app-operator项目。
$ mkdir -p $GOPATH/src/github.com/example-inc/
# 2.创建app-operator项目
$ cd $GOPATH/src/github.com/example-inc/
$ operator-sdk new app-operator
INFO[0000] Creating new Go operator 'app-operator'.     
INFO[0000] Created go.mod                               
INFO[0000] Created tools.go                             
INFO[0000] Created cmd/manager/main.go                  
INFO[0000] Created build/Dockerfile                     
INFO[0000] Created build/bin/entrypoint                 
INFO[0000] Created build/bin/user_setup                 
INFO[0000] Created deploy/service_account.yaml          
INFO[0000] Created deploy/role.yaml                     
INFO[0000] Created deploy/role_binding.yaml             
INFO[0000] Created deploy/operator.yaml                 
INFO[0000] Created pkg/apis/apis.go                     
INFO[0000] Created pkg/controller/controller.go         
INFO[0000] Created version/version.go                   
INFO[0000] Created .gitignore                           
INFO[0000] Validating project 
......                
$ tree -L 2
.
├── build
│   ├── bin
│   └── Dockerfile
├── cmd
│   └── manager
├── deploy
│   ├── crds
│   ├── operator.yaml
│   ├── role_binding.yaml
│   ├── role.yaml
│   └── service_account.yaml
├── go.mod
├── go.sum
├── pkg
│   ├── apis
│   └── controller
├── tools.go
└── version
    └── version.go

10 directories, 9 files
```
#### 项目结构
使用operator-sdk new命令创建新的 Operator 项目后，项目目录就包含了很多生成的文件夹和文件。

 1. `Gopkg.toml Gopkg.lock — Go Dep` 清单，用来描述当前 Operator 的依赖包。
 2. `cmd` - 包含 main.go 文件，使用 operator-sdk API 初始化和启动当前 Operator 的入口。
 3. `deploy` - 包含一组用于在 Kubernetes 集群上进行部署的通用的 Kubernetes 资源清单文件。
 4. `pkg/apis` - 包含定义的 API 和自定义资源（CRD）的目录树，这些文件允许 sdk 为 CRD生成代码并注册对应的类型，以便正确解码自定义资源对象。
 5. `pkg/controller` - 用于编写所有的操作业务逻辑的地方
 6. `vendor - golang vendor` 文件夹，其中包含满足当前项目的所有外部依赖包，通过 go dep 管理该目录。

我们主要需要编写的是pkg目录下面的 api 定义以及对应的 controller 实现

#### 管理器文件
Operator 的主要程序为 `cmd/manager/main.go` 中的管理器文件。管理器会自动注册 `pkg/apis/` 下定义的所有自定义资源 (CR) 的方案，并运行 `pkg/controller/` 下的所有控制器。

管理器可限制所有控制器监视资源的命名空间：

```bash
mgr, err := manager.New(cfg, manager.Options{Namespace: namespace})
```

默认情况下，这是 Operator 运行时所处的命名空间。要监视所有命名空间，把命名空间选项设为空：

```bash
mgr, err := manager.New(cfg, manager.Options{Namespace: ""})
```


### 添加api
```bash
# 3.为AppService用户资源添加一个新的API
$ operator-sdk add api --api-version=app.example.com/v1alpha1 --kind=AppService
INFO[0000] Generating api version app.example.com/v1alpha1 for kind AppService. 
INFO[0000] Created pkg/apis/app/group.go                
INFO[0040] Created pkg/apis/app/v1alpha1/appservice_types.go 
INFO[0040] Created pkg/apis/addtoscheme_app_v1alpha1.go 
INFO[0040] Created pkg/apis/app/v1alpha1/register.go    
INFO[0040] Created pkg/apis/app/v1alpha1/doc.go         
INFO[0040] Created deploy/crds/app.example.com_v1alpha1_appservice_cr.yaml 
INFO[0042] Created deploy/crds/app.example.com_appservices_crd.yaml 
INFO[0042] Running deepcopy code-generation for Custom Resource group versions: [app:[v1alpha1], ] 
INFO[0052] Code-generation complete.                    
INFO[0052] Running CRD generation for Custom Resource group versions: [app:[v1alpha1], ] 
INFO[0052] Created deploy/crds/app.example.com_appservices_crd.yaml 
INFO[0052] CRD generation complete.                     
INFO[0052] API generation complete.                     
```
### 添加控制器

```bash
# 4.添加一个新的控制器来监控AppService
$ cd app-operator
$ operator-sdk add controller --api-version=app.example.com/v1alpha1 --kind=AppService
INFO[0000] Generating controller version app.example.com/v1alpha1 for kind AppService. 
INFO[0000] Created pkg/controller/appservice/appservice_controller.go 
INFO[0000] Created pkg/controller/add_appservice.go     
INFO[0000] Controller generation complete.  
```
​
### 自定义API
打开源文件`pkg/apis/app/v1/appservice_types.go`，需要我们根据我们的需求去自定义结构体 `AppServiceSpec`，我们最上面预定义的资源清单中就有 `size`、`image`、`ports` 这些属性，所有我们需要用到的属性都需要在这个结构体中进行定义：

```bash
package v1alpha1

import (
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    appsv1 "k8s.io/api/apps/v1"
    corev1 "k8s.io/api/core/v1"
    appv1 "github.com/cnych/opdemo/pkg/apis/app/v1"
)

// resources、envs、ports 的定义都是直接引用的"k8s.io/api/core/v1"中定义的结构体
type AppServiceSpec struct {
	Size  	  *int32                      `json:"size"`
	Image     string                      `json:"image"`
	Resources corev1.ResourceRequirements `json:"resources,omitempty"`
	Envs      []corev1.EnvVar             `json:"envs,omitempty"`
	Ports     []corev1.ServicePort        `json:"ports,omitempty"`
}

//描述资源的状态，当然我们可以根据需要去自定义状态的描述
type AppServiceStatus struct {
     appsv1.DeploymentStatus `json:",inline"`
}

type AppService struct {
	metav1.TypeMeta   `json:",inline"`
	metav1.ObjectMeta `json:"metadata,omitempty"`

	Spec   AppServiceSpec   `json:"spec,omitempty"`
	Status AppServiceStatus `json:"status,omitempty"`
}

type AppServiceList struct {
	metav1.TypeMeta `json:",inline"`
	metav1.ListMeta `json:"metadata,omitempty"`
	Items           []AppService `json:"items"`
}

func init() {
	SchemeBuilder.Register(&AppService{}, &AppServiceList{})
}
```
定义完成后，在项目根目录下面执行如下命令：

```bash
$ operator-sdk generate k8s
```

改命令是用来根据我们自定义的 API 描述来自动生成一些代码，目录pkg/apis/app/v1/下面以zz_generated开头的文件就是自动生成的代码，里面的内容并不需要我们去手动编写。

这样我们就算完成了对自定义资源对象的 API 的声明。
### 实现业务逻辑
上面 API 描述声明完成了，接下来就需要我们来进行具体的业务逻辑实现了，编写具体的 `controller` 实现，打开源文件`pkg/controller/appservice/appservice_controller.go`，需要我们去更改的地方也不是很多，核心的就是Reconcile方法，该方法就是去不断的 `watch` 资源的状态，然后根据状态的不同去实现各种操作逻辑，核心代码如下：

/


### 构建部署
#### 集群内运行
```bash
# 5.构建并推送app-operator镜像到一个公开的registry，例如：quay.io
$ operator-sdk build quay.io/example/app-operator
$ docker push quay.io/example/app-operator
​
# 6.更新operator manifest来使用新构建的镜像
$ sed -i 's|REPLACE_IMAGE|quay.io/example/app-operator|g' deploy/operator.yaml
​
# 7.建立Service Account
$ kubectl create -f deploy/service_account.yaml
# 8.建立RBAC
$ kubectl create -f deploy/role.yaml
$ kubectl create -f deploy/role_binding.yaml
# 9.建立CRD
$ kubectl create -f deploy/crds/app_v1alpha1_appservice_crd.yaml
# 10.部署app-operator
$ kubectl create -f deploy/operator.yaml
​
# 11.创建一个AppService用户资源
# 默认控制器将监视AppService对象并为每一个CR创建一个pod
$ kubectl create -f deploy/crds/app_v1alpha1_appservice_cr.yaml
​
# 12.验证pod是否创建
$ kubectl get pod -l app=example-appservice
NAME                     READY     STATUS    RESTARTS   AGE
example-appservice-pod   1/1       Running   0          1m
​
# 13.清空
$ kubectl delete -f deploy/app_v1alpha1_appservice_cr.yaml
$ kubectl delete -f deploy/operator.yaml
$ kubectl delete -f deploy/role.yaml
$ kubectl delete -f deploy/role_binding.yaml
$ kubectl delete -f deploy/service_account.yaml
$ kubectl delete -f deploy/crds/app_v1alpha1_appservice_crd.yaml
```
#### go运行
在集群外本地运行。

这是开发循环中的首选方法，可加快部署和测试的速度。

使用 $HOME/.kube/config 中的默认 Kubernetes 配置文件在本地运行 Operator：

```bash
$ operator-sdk up local --namespace=default
```

您可借助标记 `--kubeconfig=<path/to/kubeconfig>` 来使用特定的 kubeconfig。

参考链接：
[https://www.operator.org.cn/bian-xie-operator](https://www.operator.org.cn/bian-xie-operator)
扩展链接：
[https://access.redhat.com/documentation/zh-cn/openshift_container_platform/4.2/html/operators/operator-sdk](https://access.redhat.com/documentation/zh-cn/openshift_container_platform/4.2/html/operators/operator-sdk)


[https://kubernetes.io/zh/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#specifying-a-structural-schema](https://kubernetes.io/zh/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#specifying-a-structural-schema)

