#  kubernetes client-go 管理 deployment
tags: client-go,Deployment



 - [Kubernetes Deployment【1】管理与编排入门](https://ghostwritten.blog.csdn.net/article/details/121510015)
 - [Kubernetes Deployment【2】原理深入详解](https://ghostwritten.blog.csdn.net/article/details/121525931)
 - [Kubernetes Deployment【3】管理高级技巧详解](https://ghostwritten.blog.csdn.net/article/details/121562269)
 - [Kubernetes Deployment【4】client-go 管理 deployment](https://ghostwritten.blog.csdn.net/article/details/110358813)
 - [云原生圣经](https://ghostwritten.blog.csdn.net/article/details/108562082)

-----

## 1. deployment控制器实现流程：

 1. `Deployment` 控制器从 `Etcd` 中获取到所有携带了“`app: nginx`”标签的 `Pod`，然后统计它们的数量，这就是实际状态；
 2. `Deployment` 对象的 `Replicas` 字段的值就是期望状态；
 3. `Deployment` 控制器将两个状态做比较，然后根据比较结果，确定是创建 `Pod`，还是删除已有的 `Pod`

可以看到，一个 Kubernetes 对象的主要编排逻辑，实际上是在第三步的“对比”阶段完成的。这个操作，通常被叫作调谐（`Reconcile`）。这个调谐的过程，则被称作“`Reconcile Loop`”（调谐循环）或者“`Sync Loop`”（同步循环）。我们社区交流也称为“控制循环”，

## 2. 操作deployment查看创建
**注意：**
[Apps/v1beta1](https://godoc.org/k8s.io/client-go/kubernetes/typed/apps/v1beta1) 1.16版本以上不再支持，而是[Apps/v1](https://godoc.org/k8s.io/client-go/kubernetes/typed/apps/v1#AppsV1Client.Deployments)
```bash
deployments, err := clientset.Appv1().Deployments("").List(metav1.ListOptions{})
```
### 2.1 go.mod

```bash
module createdeployment

go 1.13

require (
	github.com/evanphx/json-patch v4.9.0+incompatible // indirect
	github.com/fsnotify/fsnotify v1.4.9 // indirect
	github.com/golang/groupcache v0.0.0-20191227052852-215e87163ea7 // indirect
	github.com/golang/protobuf v1.4.2 // indirect
	github.com/googleapis/gnostic v0.4.0 // indirect
	github.com/gregjones/httpcache v0.0.0-20190611155906-901d90724c79 // indirect
	github.com/imdario/mergo v0.3.11 // indirect
	github.com/json-iterator/go v1.1.10 // indirect
	github.com/pkg/errors v0.9.1 // indirect
	golang.org/x/net v0.0.0-20200707034311-ab3426394381 // indirect
	golang.org/x/sys v0.0.0-20200622214017-ed371f2e16b4 // indirect
	golang.org/x/text v0.3.3 // indirect
	golang.org/x/time v0.0.0-20200630173020-3af7569d3a1e // indirect
	google.golang.org/protobuf v1.24.0 // indirect
	k8s.io/apimachinery v0.17.0
	k8s.io/client-go v0.17.0
	k8s.io/gengo v0.0.0-20200413195148-3a45101e95ac // indirect
	k8s.io/klog/v2 v2.2.0 // indirect
	k8s.io/utils v0.0.0-20201110183641-67b214c5f920 // indirect
	sigs.k8s.io/structured-merge-diff/v4 v4.0.1 // indirect
)
```

### 2.2 client.go

```bash
package main
import (
    "k8s.io/client-go/tools/clientcmd"
    "k8s.io/client-go/kubernetes"
    appsv1 "k8s.io/api/apps/v1"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    apiv1 "k8s.io/api/core/v1"
    "k8s.io/client-go/kubernetes/typed/apps/v1"
    "flag"
    "fmt"
    "encoding/json"
)

func main() {
    //kubelet.kubeconfig  是文件对应地址
    kubeconfig := flag.String("kubeconfig", "/root/.kube/config", "(optional) absolute path to the kubeconfig file")
    flag.Parse()

    // 解析到config
    config, err := clientcmd.BuildConfigFromFlags("", *kubeconfig)
    if err != nil {
        panic(err.Error())
    }

    // 创建连接
    clientset, err := kubernetes.NewForConfig(config)
    if err != nil {
        panic(err.Error())
    }
    deploymentsClient := clientset.AppsV1().Deployments("default")

    //创建deployment
    createDeployment(deploymentsClient)

    //监听deployment
    startWatchDeployment(deploymentsClient)
}

//监听Deployment变化
func startWatchDeployment(deploymentsClient v1.DeploymentInterface) {
    w, _ := deploymentsClient.Watch(metav1.ListOptions{})
    for {
        select {
        case e, _ := <-w.ResultChan():
            fmt.Println(e.Type, e.Object)
        }
    }
}

//创建deployemnt，需要谨慎按照部署的k8s版本来使用api接口
func createDeployment(deploymentsClient v1.DeploymentInterface)  {
    var r apiv1.ResourceRequirements
    //r  =  new(*apiv1.ResourceRequirements)
    //资源分配会遇到无法设置值的问题，故采用json反解析
    j := `{"limits": {"cpu":"200m", "memory": "1Gi"}, "requests": {"cpu":"200m", "memory": "1Gi"}}`
    json.Unmarshal([]byte(j), &r)
    deployment := &appsv1.Deployment{
        ObjectMeta: metav1.ObjectMeta{
            Name: "nginx",
            Labels: map[string]string{
                "app": "nginx",
            },
        },
        Spec: appsv1.DeploymentSpec{
            Replicas: int32Ptr2(1),
            Selector: &metav1.LabelSelector{
		MatchLabels: map[string]string{
		"app": "nginx",
		},
	    },
            Template: apiv1.PodTemplateSpec{
                ObjectMeta: metav1.ObjectMeta{
                    Labels: map[string]string{
                        "app": "nginx",
                    },
                },
                Spec: apiv1.PodSpec{
                    Containers: []apiv1.Container{
                        {   Name:               "nginx",
                            Image:           "nginx:1.13.5-alpine",
                            Resources: r,
                            ImagePullPolicy: "IfNotPresent",
                            Ports: []apiv1.ContainerPort{
                                {
                                   Name: "http",
                                   Protocol: apiv1.ProtocolTCP,
                                   ContainerPort: 80,
                        },
                    },
                },
            },
        },
      },
     }, 
    }

    fmt.Println("Creating deployment...")
    result, err := deploymentsClient.Create(deployment)
    if err != nil {
        panic(err)
    }
    fmt.Printf("Created deployment %s.\n", result.GetObjectMeta().GetName())
}

func int32Ptr2(i int32) *int32 { return &i }
```



参考资料：

 - [https://godoc.org/k8s.io/client-go/kubernetes#Clientset](https://godoc.org/k8s.io/client-go/kubernetes#Clientset)
 - [https://godoc.org/k8s.io/client-go/kubernetes/typed/core/v1](https://godoc.org/k8s.io/client-go/kubernetes/typed/core/v1)
 - [https://github.com/kubernetes/kubernetes/blob/master/pkg/controller/volume/persistentvolume/pv_controller_base.go](https://github.com/kubernetes/kubernetes/blob/master/pkg/controller/volume/persistentvolume/pv_controller_base.go)
 - [https://github.com/openshift/origin/blob/master/vendor/k8s.io/kubernetes/pkg/kubelet/kubelet_pods.go](https://github.com/openshift/origin/blob/master/vendor/k8s.io/kubernetes/pkg/kubelet/kubelet_pods.go)
 - [https://www.bookstack.cn/read/huweihuang-kubernetes-notes/develop-client-go.md](https://www.bookstack.cn/read/huweihuang-kubernetes-notes/develop-client-go.md)







