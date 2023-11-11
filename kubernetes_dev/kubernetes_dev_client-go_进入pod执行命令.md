


---


----
## 1. 创建pod
deploy-pod.yaml
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

```bash
$ kubectl apply -f deploy-pod.yaml

$ kubectl get deployment
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
myapp   2/2     2            2           3h45m

$ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
myapp-d46f5678b-m98p2   1/1     Running   0          3h45m
myapp-d46f5678b-x9b6g   1/1     Running   0          3h45m
```




## 2. go执行进入单个pod执行命令
go.mod

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

execpod.go

```bash
package main

import (
	"flag"
	"fmt"
	"io"
	"os"
	"path/filepath"

	"golang.org/x/crypto/ssh/terminal"
	corev1 "k8s.io/api/core/v1"
	"k8s.io/client-go/kubernetes"
	"k8s.io/client-go/kubernetes/scheme"
	"k8s.io/client-go/tools/clientcmd"
	"k8s.io/client-go/tools/remotecommand"
	"k8s.io/client-go/util/homedir"
)

func main() {
     var kubeconfig *string
     if home := homedir.HomeDir(); home != "" {
            kubeconfig = flag.String("kubeconfig", filepath.Join(home, ".kube", "config"), "(optional) absolute path to the kubeconfig file")} else {
     kubeconfig = flag.String("kubeconfig", "", "absolute path to the kubeconfig file")
     }
     flag.Parse()
     
     config, err := clientcmd.BuildConfigFromFlags("", *kubeconfig)
     if err != nil {
        panic(err)
     }
     clientset, err := kubernetes.NewForConfig(config)
     if err != nil {
        panic(err)
     }

     req := clientset.CoreV1().RESTClient().Post().
         Resource("pods").
         Name("myapp-d46f5678b-m98p2").
         Namespace("default").
         SubResource("exec").
         VersionedParams(&corev1.PodExecOptions{
             Command: []string{"echo", "hello world"},
             Stdin:   true,
             Stdout:  true,
             Stderr:  true,
             TTY:     false,
         }, scheme.ParameterCodec)
     exec, err := remotecommand.NewSPDYExecutor(config, "POST", req.URL())

     if !terminal.IsTerminal(0) || !terminal.IsTerminal(1) {
        fmt.Errorf("stdin/stdout should be terminal")
     }
     
     oldState, err := terminal.MakeRaw(0)
     if err != nil {
        fmt.Println(err)
     }
     
     defer terminal.Restore(0, oldState)
 
     screen := struct {
           io.Reader
           io.Writer
     }{os.Stdin, os.Stdout}
     
     if err = exec.Stream(remotecommand.StreamOptions{
        Stdin: screen,
        Stdout: screen,
        Stderr: screen,
        Tty:    false,
     }); err != nil {
        fmt.Print(err)
     }
}
```
执行测试:

```bash
$ go run execpod.go 
hello world
```
## 3. go执行进入某个命名空间的多个pod执行命令
go.mod

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


```bash
package main
import (
	"bufio"
	"fmt"
	"io"

	"github.com/spf13/pflag"
	v1 "k8s.io/api/core/v1"
	"k8s.io/client-go/kubernetes"
	"k8s.io/client-go/kubernetes/scheme"
	restclient "k8s.io/client-go/rest"
	"k8s.io/client-go/tools/clientcmd"
	"k8s.io/client-go/tools/remotecommand"
        metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)


type App struct {
	Config    *restclient.Config
	Namespace string
	PodName   string
}

func NewApp(namespace string, podName string) *App {
	config := LoadKubernetesConfig()
	return &App{Config: config, Namespace: namespace, PodName: podName}
}


func LoadKubernetesConfig() *restclient.Config {
	kubeconfig := pflag.Lookup("kubefile").Value.String()
	// uses the current context in kubeconfig
	config, err := clientcmd.BuildConfigFromFlags("", kubeconfig)
	if err != nil {
		panic(err.Error())
	}
	return config
}

func ExecCommandInPodContainer(config *restclient.Config, namespace string, podName string, containerName string,
	command string) (string, error) {
	client, err := kubernetes.NewForConfig(config)

	reader, writer := io.Pipe()
	var cmdOutput string

	go func() {
		scanner := bufio.NewScanner(reader)
		for scanner.Scan() {
			line := scanner.Text()
			cmdOutput += fmt.Sprintln(line)
		}
	}()

	stdin := reader
	stdout := writer
	stderr := writer
	tty := false

	cmd := []string{
		"bash",
		"-c",
		command,
	}

	req := client.CoreV1().RESTClient().Post().Resource("pods").Name(podName).
		Namespace(namespace).SubResource("exec")

	option := &v1.PodExecOptions{
		Command:   cmd,
		Container: containerName,
		Stdin:     stdin != nil,
		Stdout:    stdout != nil,
		Stderr:    stderr != nil,
		TTY:       tty,
	}

	req.VersionedParams(
		option,
		scheme.ParameterCodec,
	)
	exec, err := remotecommand.NewSPDYExecutor(config, "POST", req.URL())
	if err != nil {
		return "", err
	}
	err = exec.Stream(remotecommand.StreamOptions{
		Stdin:  stdin,
		Stdout: stdout,
		Stderr: stderr,
		Tty:    tty,
	})

	if err != nil {
		return "", err
	}

	return cmdOutput, nil
}

func (orch *App) GetClusterInfo() (string, error) {
	// check clusters info
	result, err := ExecCommandInPodContainer(orch.Config, orch.Namespace, orch.PodName, "nginx", "echo `date +%Y%m%d-%H:%M` hello wold")
	if err != nil {
		fmt.Println("Error occoured" + err.Error())
	}
	return result, nil
}


func main() {
	pflag.String("kubefile", "/root/.kube/config", "Kube file to load")
        pflag.String("namespace", "default", "App Namespace")
        pflag.Parse()
        appNamespace := pflag.Lookup("namespace").Value.String()
        config := LoadKubernetesConfig()
	client, err := kubernetes.NewForConfig(config)
        if err != nil {
           fmt.Println(err.Error())
        }
        appPodList, err := client.CoreV1().Pods(appNamespace).List(metav1.ListOptions{})
        if err != nil {
           fmt.Println(err.Error())
        }

        for _, pod := range appPodList.Items {
            app := NewApp(pod.ObjectMeta.Namespace, pod.ObjectMeta.Name)
            result, err := app.GetClusterInfo()
            if err != nil {
		fmt.Println(err.Error())
	    }
            fmt.Println(result)
        }
}
```

```bash
$ go run test1.go
20201216-10:13 hello wold

20201216-10:13 hello wold
```

参考链接：
[https://godoc.org/k8s.io/client-go/util/homedir](https://godoc.org/k8s.io/client-go/util/homedir)
[https://godoc.org/k8s.io/client-go/tools/remotecommand](https://godoc.org/k8s.io/client-go/tools/remotecommand)

相关阅读：
[k8s开发之client-go常用操作使用详解](https://ghostwritten.blog.csdn.net/article/details/110358813)

