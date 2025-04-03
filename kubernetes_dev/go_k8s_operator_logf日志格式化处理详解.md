![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8a64b846fd187b0b603fea82bd5520e1.png)

-----

## 1. 简介
操作员SDK生成的操作员使用该logr界面进行记录。此日志界面具有多个后端，例如`zap`，SDK默认在生成的代码中使用这些后端。`logr.Logger`公开结构化的日志记录方法，这些方法可帮助创建机器可读的日志并向日志记录添加大量信息。
## 2. 默认的Zap记录器
在搭建新项目时，Operator SDK使用`zap`基于`logr`后端的后端。为了帮助配置和使用此记录器，SDK包含一些帮助器功能。

在下面的简单示例中，我们使用来将zap标志`BindFlags()`集添加到操作员的命令行标志中，然后使用来设置控制器运行时记录器`zap.Options{}`。

默认情况下，`zap.Options{}`将返回准备用于生产的记录器。它使用`JSON`编码器，从该`info`级别开始记录日志。要自定义默认行为，用户可以使用zap标志集并在命令行上指定标志。zap标志集包括以下标志，可用于配置记录器：

 - `--zap-devel`：开发模式默认值（encoder = consoleEncoder，logLevel = Debug，stackTraceLevel = Warn）生产模式默认值（encoder = jsonEncoder，logLevel =
   Info，stackTraceLevel = Error）
 - `--zap-encoder`：Zap日志编码（“ json”或“ console”）
 - `--zap-log-level`：Zap Level配置日志的详细程度。可以是'debug'，'info'，'error'或任何大于0的整数值之一，对应于增加详细程度的自定义调试级别”）
 - `--zap-stacktrace-level`：Zap级别以及在其之上捕获堆栈跟踪的信息（“ info”或“ error”之一）
有关更详细的标志信息，请查阅控制器运行时[godocs](https://godoc.org/github.com/kubernetes-sigs/controller-runtime/pkg/log/zap#Options.BindFlags)。

## 3. 示例
### 第一种方法
```bash
package main

import (
	"sigs.k8s.io/controller-runtime/pkg/log/zap"  
	logf "sigs.k8s.io/controller-runtime/pkg/log"
	"flag"
)

var globalLog = logf.Log.WithName("global")
func main() {
	// Add the zap logger flag set to the CLI. The flag set must
	// be added before calling flag.Parse().
	opts := zap.Options{}
	opts.BindFlags(flag.CommandLine)
	flag.Parse()

	logger := zap.New(zap.UseFlagOptions(&opts))
	logf.SetLogger(logger)

	scopedLog := logf.Log.WithName("scoped")

	globalLog.Info("Printing at INFO level")
	globalLog.V(1).Info("Printing at DEBUG level")
	scopedLog.Info("Printing at INFO level")
	scopedLog.V(1).Info("Printing at DEBUG level")
}
```

```bash
$ go run main.go
INFO[0000] Running the operator locally in namespace default.
{"level":"info","ts":1587741740.407766,"logger":"global","msg":"Printing at INFO level"}
{"level":"info","ts":1587741740.407855,"logger":"scoped","msg":"Printing at INFO level"}
```

输出将日志级别覆盖为1（调试）

```bash
$ go run main.go --zap-log-level=debug
INFO[0000] Running the operator locally in namespace default.
{"level":"info","ts":1587741837.602911,"logger":"global","msg":"Printing at INFO level"}
{"level":"debug","ts":1587741837.602964,"logger":"global","msg":"Printing at DEBUG level"}
{"level":"info","ts":1587741837.6029708,"logger":"scoped","msg":"Printing at INFO level"}
{"level":"debug","ts":1587741837.602973,"logger":"scoped","msg":"Printing at DEBUG level"}
```
###  第二种方法

```bash
package main
import (
    "github.com/operator-framework/operator-sdk/pkg/log/zap"
    "github.com/spf13/pflag"
    logf "sigs.k8s.io/controller-runtime/pkg/log"
)
var globalLog = logf.Log.WithName("global")
func main() {
    pflag.CommandLine.AddFlagSet(zap.FlagSet())
    pflag.Parse()
    logf.SetLogger(zap.Logger())
    scopedLog := logf.Log.WithName("scoped")
    globalLog.Info("Printing at INFO level")
    globalLog.V(1).Info("Printing at DEBUG level")
    scopedLog.Info("Printing at INFO level")
    scopedLog.V(1).Info("Printing at DEBUG level")
}
```

```bash
$ go run main.go
{"level":"info","ts":1559866292.307987,"logger":"global","msg":"Printing at INFO level"}
{"level":"info","ts":1559866292.308039,"logger":"scoped","msg":"Printing at INFO level"}
Output overriding the log level to 1 (debug)
$ go run main.go --zap-level=1
{"level":"info","ts":1559866310.065048,"logger":"global","msg":"Printing at INFO level"}
{"level":"debug","ts":1559866310.0650969,"logger":"global","msg":"Printing at DEBUG level"}
{"level":"info","ts":1559866310.065119,"logger":"scoped","msg":"Printing at INFO level"}
{"level":"debug","ts":1559866310.065123,"logger":"scoped","msg":"Printing at DEBUG level"}
```

## 4. 自定义zap记录器
为了使用自定义的zap记录器，zap可以使用`controller-runtime`来将其包装在logr实现中。

以下是说明`zap-logfmt`在日志记录中使用的示例。

例
在您的`main.go`文件中，将main函数的日志替换为当前的实现：
第一种：
...

```bash
// Add the zap logger flag set to the CLI. The flag set must
// be added before calling flag.Parse().
	opts := zap.Options{}
	opts.BindFlags(flag.CommandLine)
	flag.Parse()

	logger := zap.New(zap.UseFlagOptions(&opts))
	logf.SetLogger(logger)
...
```
第二种

```go
...
// Add the zap logger flag set to the CLI. The flag set must
// be added before calling pflag.Parse().
pflag.CommandLine.AddFlagSet(zap.FlagSet())
// Add flags registered by imported packages (e.g. glog and
// controller-runtime)
pflag.CommandLine.AddGoFlagSet(flag.CommandLine)
pflag.Parse()
// Use a zap logr.Logger implementation. If none of the zap
// flags are configured (or if the zap flag set is not being
// used), this defaults to a production zap logger.
// The logger instantiated here can be changed to any logger
// implementing the logr.Logger interface. This logger will
// be propagated through the whole operator, generating
// uniform and structured logs.
logf.SetLogger(zap.Logger())
...
```

带有：

```bash
	import(
	...
	zaplogfmt "github.com/sykesm/zap-logfmt"
	uzap "go.uber.org/zap"
	"go.uber.org/zap/zapcore"
	logf "sigs.k8s.io/controller-runtime/pkg/log"
	"sigs.k8s.io/controller-runtime/pkg/log/zap"
	...
)
	configLog := uzap.NewProductionEncoderConfig()
	configLog.EncodeTime = func(ts time.Time, encoder zapcore.PrimitiveArrayEncoder) {
		encoder.AppendString(ts.UTC().Format(time.RFC3339Nano))
	}
	logfmtEncoder := zaplogfmt.NewEncoder(configLog)

	// Construct a new logr.logger.
	logger := zap.New(zap.UseDevMode(true), zap.WriteTo(os.Stdout), zap.Encoder(logfmtEncoder))
	logf.SetLogger(logger)
```

注意：对于此示例，您需要将模块添加"`github.com/sykesm/zap-logfmt`"到项目中。运行`go get -u github.com/sykesm/zap-logfmt`。完整代码：`main.go`


```bash
package main

import (
        "time"
        "os"
	zaplogfmt "github.com/sykesm/zap-logfmt"
	uzap "go.uber.org/zap"
	"go.uber.org/zap/zapcore"
	logf "sigs.k8s.io/controller-runtime/pkg/log"
	"sigs.k8s.io/controller-runtime/pkg/log/zap"
)

var globalLog = logf.Log.WithName("global")
func main() {
	// Add the zap logger flag set to the CLI. The flag set must
	// be added before calling flag.Parse().
	configLog := uzap.NewProductionEncoderConfig()
	configLog.EncodeTime = func(ts time.Time, encoder zapcore.PrimitiveArrayEncoder) {
		encoder.AppendString(ts.UTC().Format(time.RFC3339Nano))
	}
	logfmtEncoder := zaplogfmt.NewEncoder(configLog)

	// Construct a new logr.logger.
	logger := zap.New(zap.UseDevMode(true), zap.WriteTo(os.Stdout), zap.Encoder(logfmtEncoder))
	logf.SetLogger(logger)
	scopedLog := logf.Log.WithName("scoped")

	globalLog.Info("Printing at INFO level")
	globalLog.V(1).Info("Printing at DEBUG level")
	scopedLog.Info("Printing at INFO level")
	scopedLog.V(1).Info("Printing at DEBUG level")
}
```

使用自定义zap记录器进行输出

```bash
$ go run main.go
ts=2020-04-30T20:35:59.551268Z level=info logger=global msg="Printing at INFO level"
ts=2020-04-30T20:35:59.551314Z level=debug logger=global msg="Printing at DEBUG level"
ts=2020-04-30T20:35:59.551318Z level=info logger=scoped msg="Printing at INFO level"
ts=2020-04-30T20:35:59.55132Z level=debug logger=scoped msg="Printing at DEBUG level"
```

通过使用`sigs.k8s.io/controller-runtime/pkg/log`，您的记录器将通过传播`controller-runtime`。由controller-runtime代码产生的任何日志都将通过您的记录器，因此具有相同的格式和目的地。

在本地运行时设置标志
当在本地运行时`make run ENABLE_WEBHOOKS=false`，您可以使用`ARGSvar`将其他标志（包括zap标志）传递给操作员。例如：

```bash
$ make run ARGS="--zap-encoder=console" ENABLE_WEBHOOKS=false
```

确保按照下图所示确定run目标。ARGSMakefile

```bash
# Run against the configured Kubernetes cluster in ~/.kube/config
run: generate fmt vet manifests
	go run ./main.go $(ARGS)
```

## 5. 部署到集群时设置标志
将操作员部署到群集时，可以使用文件中`args`操作员container规范中的数组来设置其他标志，`config/default/manager_auth_proxy_patch.yaml`例如：

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: controller-manager
  namespace: system
spec:
  template:
    spec:
      containers:
      - name: kube-rbac-proxy
        image: gcr.io/kubebuilder/kube-rbac-proxy:v0.5.0
        args:
        - "--secure-listen-address=0.0.0.0:8443"
        - "--upstream=http://127.0.0.1:8080/"
        - "--logtostderr=true"
        - "--v=10"
        ports:
        - containerPort: 8443
          name: https
      - name: manager
        args:
        - "--metrics-addr=127.0.0.1:8080"
        - "--enable-leader-election"
        - "--zap-encoder=console"
        - "--zap-log-level=debug"
```

## 6. 创建结构化的日志语句
使用两种方法可以创建结构化日志logr。您可以在每个日志记录中使用`log.WithValues(keyValues)`包括`keyValues`键值对列表的新记录器interface{}。或者，您可以keyValues直接将其包含在log语句中，因为所有logrlog语句都带有一些消息和keyValues。签名`logr.Error()`具有`error-type`参数，可以为nil。

来自的示例`memcached_controller.go`：

```bash
package memcached

import (
	"github.com/go-logr/logr"
)


// MemcachedReconciler reconciles a Memcached object
type MemcachedReconciler struct {
	client.Client
	Log    logr.Logger
	Scheme *runtime.Scheme
}

func (r *MemcachedReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
	log := r.Log.WithValues("memcached", req.NamespacedName)

	// Fetch the Memcached instance
	memcached := &cachev1alpha1.Memcached{}
	err := r.Get(ctx, req.NamespacedName, memcached)
	if err != nil {
		if errors.IsNotFound(err) {
			// Request object not found, could have been deleted after reconcile request.
			// Owned objects are automatically garbage collected. For additional cleanup logic use finalizers.
			// Return and don't requeue
			log.Info("Memcached resource not found. Ignoring since object must be deleted")
			return ctrl.Result{}, nil
		}
		// Error reading the object - requeue the request.
		log.Error(err, "Failed to get Memcached")
		return ctrl.Result{}, err
	}

	// Check if the deployment already exists, if not create a new one
	found := &appsv1.Deployment{}
	err = r.Get(ctx, types.NamespacedName{Name: memcached.Name, Namespace: memcached.Namespace}, found)
	if err != nil && errors.IsNotFound(err) {
		// Define a new deployment
		dep := r.deploymentForMemcached(memcached)
		log.Info("Creating a new Deployment", "Deployment.Namespace", dep.Namespace, "Deployment.Name", dep.Name)
		err = r.Create(ctx, dep)
		if err != nil {
			log.Error(err, "Failed to create new Deployment", "Deployment.Namespace", dep.Namespace, "Deployment.Name", dep.Name)
			return ctrl.Result{}, err
		}
		// Deployment created successfully - return and requeue
		return ctrl.Result{Requeue: true}, nil
	} else if err != nil {
		log.Error(err, "Failed to get Deployment")
		return ctrl.Result{}, err
	}

	...
}
```

日志记录如下所示（从log.Error()上方）：

```bash
2020-04-27T09:14:15.939-0400	ERROR	controllers.Memcached	Failed to create new Deployment	{"memcached": "default/memcached-sample", "Deployment.Namespace": "default", "Deployment.Name": "memcached-sample"}
```

## 7. 非默认日志记录
如果您不想logr用作日志记录工具，则可以logr从操作员的代码（包括中的logr设置代码）中删除特定的语句而不会出现问题main.go，然后添加自己的语句。请注意，删除`logr`安装代码将阻止`controller-runtime`日志记录。

