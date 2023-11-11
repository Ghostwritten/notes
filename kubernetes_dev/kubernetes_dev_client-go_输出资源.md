

## 1.获取pod，pv，pvc，namespace数量并打印
### go.mod

```bash
module client

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


### client.go
```bash
package main
import (
    "flag"
//    "context"
    "fmt"
    "os"
    "path/filepath"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    "k8s.io/client-go/kubernetes"
    "k8s.io/client-go/tools/clientcmd"
)
func main() {
    var kubeconfig *string
    if home := homeDir(); home != "" {
        kubeconfig = flag.String("kubeconfig", filepath.Join(home, ".kube", "config"), "(optional) absolute path to the kubeconfig file")
    } else {
        kubeconfig = flag.String("kubeconfig", "", "absolute path to the kubeconfig file")
    }
    flag.Parse()
    // uses the current context in kubeconfig
    config, err := clientcmd.BuildConfigFromFlags("", *kubeconfig)
    if err != nil {
        panic(err.Error())
    }
    // creates the clientset
    clientset, err := kubernetes.NewForConfig(config)
    if err != nil {
        panic(err.Error())
    }
    
    pods, err1 := clientset.CoreV1().Pods("").List(metav1.ListOptions{})
    if err1 != nil {
       panic(err1.Error())
    }
    pvs, err2 := clientset.CoreV1().PersistentVolumes().List(metav1.ListOptions{})
    if err2 != nil {
       panic(err2.Error())
    }
    pvcs, err3 := clientset.CoreV1().PersistentVolumeClaims("").List(metav1.ListOptions{})
    if err3 != nil {
       panic(err3.Error())
    }
    namespaces, err4 := clientset.CoreV1().Namespaces().List(metav1.ListOptions{})
    if err4 != nil {
       panic(err4.Error())
    }
    fmt.Printf("There are %d pods in the cluster\n", len(pods.Items))
    fmt.Printf("There are %d pvs in the cluster\n", len(pvs.Items))
    fmt.Printf("There are %d pvcs in the cluster\n", len(pvcs.Items))
    fmt.Printf("There are %d namespaces in the cluster\n", len(namespaces.Items))

    fmt.Println("---------pods----------")
    for _, pod := range pods.Items {
       fmt.Printf("Name: %s, Status: %s, CreateTime: %s\n", pod.ObjectMeta.Name, pod.Status.Phase, pod.ObjectMeta.CreationTimestamp)
    }
    fmt.Println("---------pvs----------")
    for _, pv := range pvs.Items {
       fmt.Printf("Name: %s, Status: %s, CreateTime: %s\n", pv.ObjectMeta.Name, pv.Status.Phase, pv.ObjectMeta.CreationTimestamp)
    }
    fmt.Println("---------pvcs----------")
    for _, pvc := range pvcs.Items {
       fmt.Printf("Name: %s, Status: %s, CreateTime: %s\n", pvc.ObjectMeta.Name, pvc.Status.Phase, pvc.ObjectMeta.CreationTimestamp)
    }
    fmt.Println("---------namespaces----------")
    for _, namespace := range namespaces.Items {
       fmt.Printf("Name: %s, Status: %s, CreateTime: %s\n", namespace.ObjectMeta.Name, namespace.Status.Phase, namespace.ObjectMeta.CreationTimestamp)
    }
}
func homeDir() string {
    if h := os.Getenv("HOME"); h != "" {
        return h
    }
    return os.Getenv("USERPROFILE") // windows
}

```
执行：
```bash
$ go run client.go
There are 4 pods in the cluster
There are 2 pvs in the cluster
There are 2 pvcs in the cluster
There are 6 namespaces in the cluster
---------pods----------
Name: backend, Status: Running, CreateTime: 2020-10-23 02:24:45 -0700 PDT
Name: database, Status: Running, CreateTime: 2020-10-23 02:24:45 -0700 PDT
Name: frontend, Status: Running, CreateTime: 2020-10-23 02:24:45 -0700 PDT
Name: backend, Status: Running, CreateTime: 2020-10-24 02:34:47 -0700 PDT
---------pvs----------
Name: pv, Status: Bound, CreateTime: 2020-09-28 19:19:46 -0700 PDT
Name: task-pv-volume, Status: Bound, CreateTime: 2020-11-27 04:34:38 -0800 PST
---------pvcs----------
Name: pvc, Status: Bound, CreateTime: 2020-09-28 19:23:51 -0700 PDT
Name: task-pv-claim, Status: Bound, CreateTime: 2020-11-28 06:27:54 -0800 PST
---------namespaces----------
Name: app-stack, Status: Active, CreateTime: 2020-09-28 19:00:18 -0700 PDT
Name: default, Status: Active, CreateTime: 2020-09-25 23:11:56 -0700 PDT
Name: kube-node-lease, Status: Active, CreateTime: 2020-09-25 23:11:55 -0700 PDT
Name: kube-public, Status: Active, CreateTime: 2020-09-25 23:11:55 -0700 PDT
Name: kube-system, Status: Active, CreateTime: 2020-09-25 23:11:54 -0700 PDT
Name: rq-demo, Status: Active, CreateTime: 2020-10-22 20:01:59 -0700 PDT

```

## 2. 打印pod详细信息

```bash
package main

import (
	"flag"
	"fmt"
	"os"
	"path/filepath"
	"time"

	"k8s.io/apimachinery/pkg/api/errors"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/client-go/kubernetes"
	"k8s.io/client-go/tools/clientcmd"
)

func main() {
	// 配置 k8s 集群外 kubeconfig 配置文件，默认位置 $HOME/.kube/config
	var kubeconfig *string
	if home := homeDir(); home != "" {
		kubeconfig = flag.String("kubeconfig", filepath.Join(home, ".kube", "config"), "(optional) absolute path to the kubeconfig file")
	} else {
		kubeconfig = flag.String("kubeconfig", "", "absolute path to the kubeconfig file")
	}
	flag.Parse()

	//在 kubeconfig 中使用当前上下文环境，config 获取支持 url 和 path 方式
	config, err := clientcmd.BuildConfigFromFlags("", *kubeconfig)
	if err != nil {
		panic(err.Error())
	}

	// 根据指定的 config 创建一个新的 clientset
	clientset, err := kubernetes.NewForConfig(config)
	if err != nil {
		panic(err.Error())
	}
	for {
		// 通过实现 clientset 的 CoreV1Interface 接口列表中的 PodsGetter 接口方法 Pods(namespace string) 返回 PodInterface
		// PodInterface 接口拥有操作 Pod 资源的方法，例如 Create、Update、Get、List 等方法
		// 注意：Pods() 方法中 namespace 不指定则获取 Cluster 所有 Pod 列表
		pods, err := clientset.CoreV1().Pods("").List(metav1.ListOptions{})
		if err != nil {
			panic(err.Error())
		}
		fmt.Printf("There are %d pods in the k8s cluster\n", len(pods.Items))

		// 获取指定 namespace 中的 Pod 列表信息
		namespace := "default"
		pods, err = clientset.CoreV1().Pods(namespace).List(metav1.ListOptions{})
		if err != nil {
			panic(err)
		}
		fmt.Printf("\nThere are %d pods in namespaces %s\n", len(pods.Items), namespace)
		for _, pod := range pods.Items {
			fmt.Printf("Name: %s, Status: %s, CreateTime: %s\n", pod.ObjectMeta.Name, pod.Status.Phase, pod.ObjectMeta.CreationTimestamp)
		}

		// 获取指定 namespaces 和 podName 的详细信息，使用 error handle 方式处理错误信息
		namespace = "default"
		podName := "backend"
		pod, err := clientset.CoreV1().Pods(namespace).Get(podName, metav1.GetOptions{})
		if errors.IsNotFound(err) {
			fmt.Printf("Pod %s in namespace %s not found\n", podName, namespace)
		} else if statusError, isStatus := err.(*errors.StatusError); isStatus {
			fmt.Printf("Error getting pod %s in namespace %s: %v\n",
				podName, namespace, statusError.ErrStatus.Message)
		} else if err != nil {
			panic(err.Error())
		} else {
			fmt.Printf("\nFound pod %s in namespace %s\n", podName, namespace)
			maps := map[string]interface{}{
				"Name":        pod.ObjectMeta.Name,
				"Namespaces":  pod.ObjectMeta.Namespace,
				"NodeName":    pod.Spec.NodeName,
				"Annotations": pod.ObjectMeta.Annotations,
				"Labels":      pod.ObjectMeta.Labels,
				"SelfLink":    pod.ObjectMeta.SelfLink,
				"Uid":         pod.ObjectMeta.UID,
				"Status":      pod.Status.Phase,
				"IP":          pod.Status.PodIP,
				"Image":       pod.Spec.Containers[0].Image,
			}
			prettyPrint(maps)
//                        fmt.Println(maps)
		}

		time.Sleep(10 * time.Second)
	}
}
//分列输出
func prettyPrint(maps map[string]interface{}) { 
	lens := 0
	for k, _ := range maps {
		if lens <= len(k) {
			lens = len(k)
		}
            fmt.Println(lens)
	}
	for key, values := range maps {
		spaces := lens - len(key)
		v := ""
		for i := 0; i < spaces; i++ {
			v += " "
		}
		fmt.Printf("%s: %s%v\n", key, v, values)
	}
}

func homeDir() string {
	if h := os.Getenv("HOME"); h != "" {
		return h
	}
	return os.Getenv("USERPROFILE") // windows
}
```
输出：

```bash
root@master:~/k8s_dev/client_pod_details# go run client.go 
There are 21 pods in the k8s cluster

There are 6 pods in namespaces default
Name: backend, Status: Running, CreateTime: 2020-10-24 02:34:47 -0700 PDT
Name: delpersistentvolumeclaims-1606753860-xfkgx, Status: Succeeded, CreateTime: 2020-11-30 08:31:02 -0800 PST
Name: delpersistentvolumeclaims-1606753920-hftgz, Status: Succeeded, CreateTime: 2020-11-30 08:32:03 -0800 PST
Name: delpersistentvolumeclaims-1606753980-j8k76, Status: Succeeded, CreateTime: 2020-11-30 08:33:03 -0800 PST
Name: patch-demo-76b69ff5cd-htjmd, Status: Running, CreateTime: 2020-10-24 02:59:29 -0700 PDT
Name: patch-demo-76b69ff5cd-hwzd5, Status: Running, CreateTime: 2020-11-30 04:42:15 -0800 PST

Found pod backend in namespace default
Name:        backend
NodeName:    node2
IP:          192.168.104.34
Image:       nginx
Uid:         1c206880-592b-4ad1-89bb-26bd51e8ddf5
Status:      Running
Namespaces:  default
Annotations: map[cni.projectcalico.org/podIP:192.168.104.34/32]
Labels:      map[]
SelfLink:    /api/v1/namespaces/default/pods/backend
```
