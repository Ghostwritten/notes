

---
## 1. 采集并删除pvc
###  go.mod
```bash
module cronserver

go 1.13

require (
	github.com/imdario/mergo v0.3.8 // indirect
	github.com/spf13/pflag v1.0.5
	golang.org/x/oauth2 v0.0.0-20200107190931-bf48bf16ab8d // indirect
	golang.org/x/time v0.0.0-20191024005414-555d28b269f0 // indirect
	k8s.io/api v0.0.0-20200214081623-ecbd4af0fc33
	k8s.io/apimachinery v0.17.3
	k8s.io/client-go v11.0.0+incompatible
	k8s.io/klog v1.0.0
	k8s.io/utils v0.0.0-20200229041039-0a110f9eb7ab // indirect
)

replace k8s.io/client-go => k8s.io/client-go v0.0.0-20200214082307-e38a84523341
```

### client.go

```bash
package main
import (
        "context"
        "time"
	"fmt"

	"github.com/spf13/pflag"
        v1 "k8s.io/api/core/v1"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	restclient "k8s.io/client-go/rest"
	"k8s.io/client-go/tools/clientcmd"
	"k8s.io/klog"
	clientset "k8s.io/client-go/kubernetes"
)


// GetPersistentVolumeClaims 方法将会从 Kuberenetes 自动获取到 PersistentVolumeClaims 服务对象
func GetPersistentVolumeClaims(name string) *v1.PersistentVolumeClaim {
	_, kubeClient := loadKubernetesClientsForDelPersistentVolumeClaims()
	PersistentVolumeClaims, err := kubeClient.CoreV1().PersistentVolumeClaims("").Get(context.TODO(),name,metav1.GetOptions{})
	if err != nil {
		klog.Errorf("Error getting PersistentVolumeClaims %v", err)
	}
	return PersistentVolumeClaims
}

// ListersistentVolumeClaims 方法将会从Kuberenetes自动获取到 PersistentVolumeClaims 服务对象
func ListPersistentVolumeClaims() *v1.PersistentVolumeClaimList {
	_, kubeClient := loadKubernetesClientsForDelPersistentVolumeClaims()
	PersistentVolumeClaims, err := kubeClient.CoreV1().PersistentVolumeClaims("").List(context.TODO(),metav1.ListOptions{})
	if err != nil {
		klog.Errorf("Error getting PersistentVolumeClaims %v", err)
	}

	return PersistentVolumeClaims
}

// DelPersistentVolumeClaims 方法将会从Kuberenetes自动删除 PersistentVolumeClaims 服务对象
func DelPersistentVolumeClaims(pvc v1.PersistentVolumeClaim)  {

	_, kubeClient := loadKubernetesClientsForDelPersistentVolumeClaims()
	pvcname := pvc.GetName()
        namespace := pvc.GetNamespace()
	pvcstatusPhase := string(pvc.Status.Phase)
        pvcCreationTime := pvc.GetCreationTimestamp()
	age := int(time.Since(pvcCreationTime.Time).Seconds())
        //pvc.Labels[PVC_WAIT_KEY] = PVC_WAIT_GC_VALUE
        //pvc.Labels[PVC_DELETE_TIME] = time.Now().In(unitv1alpha1.GetCstZone()).Format(layout: "2006-01-02 15:04:05.999999999")
        //fmt.Println(pvc.Labels)
         

// 删除条件pvc STATUS is "Released" and age 大于72小时
	if pvcstatusPhase == "Pending" {
		if age > 20 {

		   err := kubeClient.CoreV1().PersistentVolumeClaims(namespace).Delete(context.TODO(),pvcname,&metav1.DeleteOptions{})

		   if err != nil {
				klog.Errorf("Delete pvc error: %v", err)
	           }
                }
        }
}

func loadKubernetesClientsForDelPersistentVolumeClaims() (*restclient.Config, *clientset.Clientset) {
	klog.Infof("starting getting PersistentVolumeClaims")
	kubeconfig := pflag.Lookup("kubefile").Value.String()
	// uses the current context in kubeconfig
	config, err := clientcmd.BuildConfigFromFlags("", kubeconfig)
	if err != nil {
		panic(err.Error())
	}
	kubeClient, errClient := clientset.NewForConfig(config)
	if errClient != nil {
		klog.Errorf("Error received creating client %v", errClient)
	}
	return config, kubeClient
}


func main() {
	pflag.String("kubefile", "/Users/jamesjiang/.kube/config", "Kube file to load")
	pflag.String("timetoexec", "800", "Seconds to execute in a single period")
	pflag.String("option", "backupBinaryLog", "Function options")
	pflag.Parse()
        option := pflag.Lookup("option").Value.String()
	if option == "DelPersistentVolumeClaims" {
		fmt.Println("【DelpersistentVolumeClaims】")
                PersistentVolumeClaims :=  ListPersistentVolumeClaims()
		for _, pvc := range PersistentVolumeClaims.Items {
		    DelPersistentVolumeClaims(pvc)
		}
	}
}
```
测试

```bash
$   kubectl get pvc
NAME            STATUS    VOLUME           CAPACITY   ACCESS MODES   STORAGECLASS   AGE
cron-pv-claim   Pending                                              manual         6m30s
pvc             Bound     pv               512M       RWX            shared         62d
task-pv-claim   Bound     task-pv-volume   10Gi       RWO            manual         40h
$ go build .
$ ./cronserver --kubefile /root/.kube/config --option DelPersistentVolumeClaims
$   kubectl get pvc
NAME            STATUS    VOLUME           CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc             Bound     pv               512M       RWX            shared         62d
task-pv-claim   Bound     task-pv-volume   10Gi       RWO            manual         40h
```

## 2. 通过label标签筛选删除pvc

> （pod、pv也通用）

### go.mode
```bash
module hello

go 1.13

require (
	github.com/imdario/mergo v0.3.8 // indirect
	github.com/spf13/pflag v1.0.5
	golang.org/x/oauth2 v0.0.0-20200107190931-bf48bf16ab8d // indirect
	golang.org/x/time v0.0.0-20191024005414-555d28b269f0 // indirect
	k8s.io/api v0.0.0-20200214081623-ecbd4af0fc33
	k8s.io/apimachinery v0.17.3
	k8s.io/client-go v11.0.0+incompatible
	k8s.io/klog v1.0.0
	k8s.io/utils v0.0.0-20200229041039-0a110f9eb7ab // indirect
)

replace k8s.io/client-go => k8s.io/client-go v0.0.0-20200214082307-e38a84523341
```
### client.go

```bash
package main
import (
        "context"
//        "time"
	"fmt"

	"github.com/spf13/pflag"
        v1 "k8s.io/api/core/v1"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	restclient "k8s.io/client-go/rest"
        "k8s.io/apimachinery/pkg/labels"
	"k8s.io/client-go/tools/clientcmd"
	"k8s.io/klog"
	clientset "k8s.io/client-go/kubernetes"
)


// GetPersistentVolumeClaims 方法将会从 Kuberenetes 自动获取到 PersistentVolumeClaims 服务对象
func GetPersistentVolumeClaims(name string) *v1.PersistentVolumeClaim {
	_, kubeClient := loadKubernetesClientsForDelPersistentVolumeClaims()
	PersistentVolumeClaims, err := kubeClient.CoreV1().PersistentVolumeClaims("").Get(context.TODO(),name,metav1.GetOptions{})
	if err != nil {
		klog.Errorf("Error getting PersistentVolumeClaims %v", err)
	}
	return PersistentVolumeClaims
}

// ListersistentVolumeClaims 方法将会从Kuberenetes自动获取到 PersistentVolumeClaims 服务对象
func ListPersistentVolumeClaims() *v1.PersistentVolumeClaimList {
	_, kubeClient := loadKubernetesClientsForDelPersistentVolumeClaims()
	PersistentVolumeClaims, err := kubeClient.CoreV1().PersistentVolumeClaims("").List(context.TODO(),metav1.ListOptions{})
	if err != nil {
		klog.Errorf("Error getting PersistentVolumeClaims %v", err)
	}

	return PersistentVolumeClaims
}

// ListNamespace 方法将会从Kuberenetes自动获取到 Namespace 服务对象
func ListNamespaces() *v1.NamespaceList {
	_, kubeClient := loadKubernetesClientsForDelPersistentVolumeClaims()
        Namespaces, err := kubeClient.CoreV1().Namespaces().List(context.TODO(),metav1.ListOptions{})
	if err != nil {
		klog.Errorf("Error getting Namespaces %v", err)
	}

	return Namespaces
}


// DelPersistentVolumeClaims 方法将会从Kuberenetes自动删除 PersistentVolumeClaims 服务对象
func DelLabelPersistentVolumeClaims(namespace string)  {

        
	var (
		PVC_WAIT_GC_VALUE = "PVC_WAIT_GC_VALUE"
                PVC_DELETE_TIME = "295200s"
	)
	_, kubeClient := loadKubernetesClientsForDelPersistentVolumeClaims()
        labelPvc := labels.SelectorFromSet(labels.Set(map[string]string{"PVC_WAIT_KEY": PVC_WAIT_GC_VALUE, "PVC_DELETE_TIME": PVC_DELETE_TIME}))
	listPvcOptions := metav1.ListOptions{
		LabelSelector: labelPvc.String(),
	}
	PersistentVolumeClaims, err := kubeClient.CoreV1().PersistentVolumeClaims(namespace).List(context.TODO(), listPvcOptions)
        for _, PersistentVolumeClaim := range PersistentVolumeClaims.Items {
            fmt.Printf("Start to delete pvc %s in %s namespace! \n", PersistentVolumeClaim.ObjectMeta.Name, namespace)
        }
	err = kubeClient.CoreV1().PersistentVolumeClaims(namespace).DeleteCollection(context.TODO(), &metav1.DeleteOptions{}, listPvcOptions)
	if err != nil {
			klog.Errorf("Drop pvc labels err %v",err)
	}
}

func loadKubernetesClientsForDelPersistentVolumeClaims() (*restclient.Config, *clientset.Clientset) {
	klog.Infof("starting getting PersistentVolumeClaims")
	kubeconfig := pflag.Lookup("kubefile").Value.String()
	// uses the current context in kubeconfig
	config, err := clientcmd.BuildConfigFromFlags("", kubeconfig)
	if err != nil {
		panic(err.Error())
	}
	kubeClient, errClient := clientset.NewForConfig(config)
	if errClient != nil {
		klog.Errorf("Error received creating client %v", errClient)
	}
	return config, kubeClient
}


func main() {
	pflag.String("kubefile", "/Users/jamesjiang/.kube/config", "Kube file to load")
	pflag.String("timetoexec", "800", "Seconds to execute in a single period")
	pflag.String("option", "backupBinaryLog", "Function options")
	pflag.Parse()
        option := pflag.Lookup("option").Value.String()
	if option == "DelLabelPersistentVolumeClaims" {
		fmt.Println("【DelpersistentVolumeClaims】")
                Namespaces :=  ListNamespaces()
		for _, namespace := range Namespaces.Items {
                   namespace := namespace.ObjectMeta.Name
		   DelLabelPersistentVolumeClaims(namespace)
		}
	}
}
```

```bash
go build .
```

### 测试
创建一个pvc，labels分别为`PVC_WAIT_KEY: PVC_WAIT_GC_VALUE`和`PVC_DELETE_TIME: 295200s`

pvc.yaml
```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: backup-pv-claim
  labels:
    PVC_WAIT_KEY: PVC_WAIT_GC_VALUE
    PVC_DELETE_TIME: 295200s
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

```bash
$ kubectl apply -f pvc.yaml
persistentvolumeclaim/backup-pv-claim created

$ kubectl get pvc
NAME              STATUS    VOLUME           CAPACITY   ACCESS MODES   STORAGECLASS   AGE
backup-pv-claim   Pending                                              manual         4s

$ ./cronserver --option DelLabelPersistentVolumeClaims --kubefile /root/.kube/config
【DelpersistentVolumeClaims】
I1130 22:59:55.328428   60651 client.go:76] starting getting PersistentVolumeClaims
I1130 22:59:55.403363   60651 client.go:76] starting getting PersistentVolumeClaims
Start to delete pvc backup-pv-claim in default namespace!  //删除pvc日志
I1130 22:59:55.512610   60651 client.go:76] starting getting PersistentVolumeClaims
```

