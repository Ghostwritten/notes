
## 1. 介绍
  创建klog的决定并不是一件容易的事，但是由于glog中存在一些缺陷，所以有必要这样做。最终，由于没有积极开发glog而创建了fork。这可以在glog自述文件中看到
  这使我们无法在没有分叉的情况下解决许多用例。下面列出了需要开发功能的因素：

 - glog 提出了很多“陷阱”，并介绍了容器化环境中的挑战，所有这些都没有得到很好的记录。
 - glog 没有提供一种简单的方法来测试日志，这降低了使用它的软件的稳定性
 - 长期目标是实现一个日志接口，该接口允许我们添加上下文，更改输出格式等。

历史背景可以在这里找到：

[https://github.com/kubernetes/kubernetes/issues/61006](https://github.com/kubernetes/kubernetes/issues/61006)
[https://github.com/kubernetes/kubernetes/issues/70264](https://github.com/kubernetes/kubernetes/issues/70264)
[https://groups.google.com/forum/#!msg/kubernetes-sig-architecture/wCWiWf3Juzs/hXRVBH90CgAJ](https://groups.google.com/forum/#!msg/kubernetes-sig-architecture/wCWiWf3Juzs/hXRVBH90CgAJ)
[https://groups.google.com/forum/#!msg/kubernetes-dev/7vnijOMhLS0/1oRiNtigBgAJ](https://groups.google.com/forum/#!msg/kubernetes-dev/7vnijOMhLS0/1oRiNtigBgAJ)

## 2. 使用方法

 - 替代进口"`github.com/golang/glog`"与"`k8s.io/klog/v2`"

使用`klog.InitFlags(nil)`明确的初始化全局标志，因为我们不再使用init()方法注册标志
现在，您可以使用log_file而不是log_dir用于登录到单个文件（请参阅examples/log_file/usage_log_file.go）
如果您想将使用klog记录的所有内容重定向到其他地方（例如syslog！），则可以使用klog.SetOutput()method并提供一个io.Writer。（请参阅examples/set_output/usage_set_output.go）
有关更多日志记录约定（[**请参阅日志记录约定**](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-instrumentation/logging.md)）
## 3. 实例
### 3.1 输出到文件
**usage_log_file.go**

```bash
package main

import (
	"flag"

	"k8s.io/klog/v2"
)

func main() {
	klog.InitFlags(nil)
	// By default klog writes to stderr. Setting logtostderr to false makes klog
	// write to a log file.
	flag.Set("logtostderr", "false") //日志输出到stderr，不输出到日志文件。false为关闭
	flag.Set("log_file", "myfile.log")
	flag.Parse()
	klog.Info("nice to meet you")
	klog.Flush()
}
```
输出结果：

```bash
Log file created at: 2020/11/23 06:35:58
Running on machine: master
Binary: Built with gc go1.12 for linux/amd64
Log line format: [IWEF]mmdd hh:mm:ss.uuuuuu threadid file:line] msg
I1123 06:35:58.691771    3168 test.go:16] nice to meet you
Log file created at: 2020/11/23 06:36:05
Running on machine: master
Binary: Built with gc go1.12 for linux/amd64
Log line format: [IWEF]mmdd hh:mm:ss.uuuuuu threadid file:line] msg
I1123 06:36:05.890354    3356 test.go:16] nice to meet you
```
### 3.2 输出至终端
**usage_set_output.go**
```bash
package main

import (
	"bytes"
	"flag"
	"fmt"
	"k8s.io/klog/v2"
)

func main() {
	klog.InitFlags(nil)
	flag.Set("logtostderr", "false")
	flag.Set("alsologtostderr", "false")
	flag.Parse()

	buf := new(bytes.Buffer)
	klog.SetOutput(buf)
	klog.Info("nice to meet you")
	klog.Flush()

	fmt.Printf("LOGGED: %s", buf.String())
}
```
输出结果：

```bash
go run usage_set_output.go
LOGGED: I1123 06:50:17.709793   23045 test2.go:18] nice to meet you
```
### 3.3 输出自定义格式

```bash
package main

import (
	"flag"

	"k8s.io/klog/v2"
	"k8s.io/klog/v2/klogr"
)

type myError struct {
	str string
}

func (e myError) Error() string {
	return e.str
}

func main() {
	klog.InitFlags(nil)
	flag.Set("v", "3")
	flag.Parse()
	log := klogr.New().WithName("MyName").WithValues("user", "you")
	log.Info("hello", "val1", 1, "val2", map[string]int{"k": 1})
	log.V(3).Info("nice to meet you")
	log.Error(nil, "uh oh", "trouble", true, "reasons", []float64{0.1, 0.11, 3.14})
	log.Error(myError{"an error occurred"}, "goodbye", "code", -1)
	klog.Flush()
}
```
输出结果：
```bash
root@master:~/k8s_dev/klog# go run test3.go
I1123 07:08:38.580111   47818 test3.go:23] MyName "msg"="hello" "user"="you" "val1"=1 "val2"={"k":1}
I1123 07:08:38.580544   47818 test3.go:24] MyName "msg"="nice to meet you" "user"="you" 
E1123 07:08:38.580742   47818 test3.go:25] MyName "msg"="uh oh" "error"=null "user"="you" "reasons"=[0.1,0.11,3.14] "trouble"=true
E1123 07:08:38.580773   47818 test3.go:26] MyName "msg"="goodbye" "error"="an error occurred" "user"="you" "code"=-1
```
### 3.4 klog与glog

```bash
package main

import (
	"flag"

	"github.com/golang/glog"
	"k8s.io/klog/v2"
)

func main() {
	flag.Set("alsologtostderr", "true")
	flag.Parse()

	klogFlags := flag.NewFlagSet("klog", flag.ExitOnError)
	klog.InitFlags(klogFlags)

	// Sync the glog and klog flags.
	flag.CommandLine.VisitAll(func(f1 *flag.Flag) {
		f2 := klogFlags.Lookup(f1.Name)
		if f2 != nil {
			value := f1.Value.String()
			f2.Value.Set(value)
		}
	})

	glog.Info("hello from glog!")
	klog.Info("nice to meet you, I'm klog")
	glog.Flush()
	klog.Flush()
}
```
输出结果：

```bash
I1123 07:24:40.459265   69727 test4.go:26] hello from glog!
I1123 07:24:40.460089   69727 test4.go:27] nice to meet you, I'm klog
```
### 3.5 klogv1与klgov2

```bash
package main

import (
	"flag"

	klogv1 "k8s.io/klog"
	klogv2 "k8s.io/klog/v2"
)

// OutputCallDepth is the stack depth where we can find the origin of this call
const OutputCallDepth = 6

// DefaultPrefixLength is the length of the log prefix that we have to strip out
const DefaultPrefixLength = 53

// klogWriter is used in SetOutputBySeverity call below to redirect
// any calls to klogv1 to end up in klogv2
type klogWriter struct{}

func (kw klogWriter) Write(p []byte) (n int, err error) {
	if len(p) < DefaultPrefixLength {
		klogv2.InfoDepth(OutputCallDepth, string(p))
		return len(p), nil
	}
	if p[0] == 'I' {
		klogv2.InfoDepth(OutputCallDepth, string(p[DefaultPrefixLength:]))
	} else if p[0] == 'W' {
		klogv2.WarningDepth(OutputCallDepth, string(p[DefaultPrefixLength:]))
	} else if p[0] == 'E' {
		klogv2.ErrorDepth(OutputCallDepth, string(p[DefaultPrefixLength:]))
	} else if p[0] == 'F' {
		klogv2.FatalDepth(OutputCallDepth, string(p[DefaultPrefixLength:]))
	} else {
		klogv2.InfoDepth(OutputCallDepth, string(p[DefaultPrefixLength:]))
	}
	return len(p), nil
}

func main() {
	// initialize klog/v2, can also bind to a local flagset if desired
	klogv2.InitFlags(nil)

	// In this example, we want to show you that all the lines logged
	// end up in the myfile.log. You do NOT need them in your application
	// as all these flags are set up from the command line typically
	flag.Set("logtostderr", "false")     // By default klog logs to stderr, switch that off
	flag.Set("alsologtostderr", "false") // false is default, but this is informative
	flag.Set("stderrthreshold", "FATAL") // stderrthreshold defaults to ERROR, we don't want anything in stderr
	flag.Set("log_file", "myfile.log")   // log to a file

	// parse klog/v2 flags
	flag.Parse()
	// make sure we flush before exiting
	defer klogv2.Flush()

	// BEGIN : hack to redirect klogv1 calls to klog v2
	// Tell klog NOT to log into STDERR. Otherwise, we risk
	// certain kinds of API errors getting logged into a directory not
	// available in a `FROM scratch` Docker container, causing us to abort
	
	var klogv1Flags flag.FlagSet
	klogv1.InitFlags(&klogv1Flags)
	klogv1Flags.Set("logtostderr", "false")     // By default klog v1 logs to stderr, switch that off
	klogv1Flags.Set("stderrthreshold", "FATAL") // stderrthreshold defaults to ERROR, use this if you
	// don't want anything in your stderr

	klogv1.SetOutputBySeverity("INFO", klogWriter{}) // tell klog v1 to use the writer
	// END : hack to redirect klogv1 calls to klog v2

	// Now you can mix klogv1 and v2 in the same code base
	klogv2.Info("hello from klog (v2)!")
	klogv1.Info("hello from klog (v1)!")
	klogv1.Warning("beware from klog (v1)!")
	klogv1.Error("error from klog (v1)!")
	klogv2.Info("nice to meet you (v2)")
}
```
输出结果：

```bash
$ cat myfile.log 
Log file created at: 2020/11/23 07:32:22
Running on machine: master
Binary: Built with gc go1.12 for linux/amd64
Log line format: [IWEF]mmdd hh:mm:ss.uuuuuu threadid file:line] msg
I1123 07:32:22.759786   80245 test5.go:70] hello from klog (v2)!
I1123 07:32:22.760099   80245 test5.go:71]  klog (v1)!
W1123 07:32:22.760459   80245 test5.go:72] m klog (v1)!
E1123 07:32:22.760602   80245 test5.go:73]  klog (v1)!
I1123 07:32:22.760724   80245 test5.go:74] nice to meet you (v2)
```

### 3.6 获取环境变量

```bash
package main

import (
	"flag"
	"fmt"
	"os"

	"k8s.io/klog/v2"
)

func main() {
	infoLogLine := getEnvOrDie("KLOG_INFO_LOG")
	warningLogLine := getEnvOrDie("KLOG_WARNING_LOG")
	errorLogLine := getEnvOrDie("KLOG_ERROR_LOG")
	fatalLogLine := getEnvOrDie("KLOG_FATAL_LOG")

	klog.InitFlags(nil)
	flag.Parse()
	klog.Info(infoLogLine)
	klog.Warning(warningLogLine)
	klog.Error(errorLogLine)
	klog.Flush()
	klog.Fatal(fatalLogLine)
}

func getEnvOrDie(name string) string {
	val, ok := os.LookupEnv(name)
	if !ok {
		fmt.Fprintf(os.Stderr, name+" could not be found in environment")
		os.Exit(1)
	}
	return val
}
```
输出结果：

```bash
root@master:~/k8s_dev/klog# export KLOG_INFO_LOG='test1'
root@master:~/k8s_dev/klog# export KLOG_WARNING_LOG='test_warning'
root@master:~/k8s_dev/klog# export  KLOG_ERROR_LOG='test_error'
root@master:~/k8s_dev/klog# export KLOG_FATAL_LOG='test_fatal'
root@master:~/k8s_dev/klog# go run test7.go
I1123 08:09:11.652294  130337 test7.go:19] test1
W1123 08:09:11.652583  130337 test7.go:20] test_warning
E1123 08:09:11.653002  130337 test7.go:21] test_error
F1123 08:09:11.653282  130337 test7.go:23] test_fatal
goroutine 1 [running]:
k8s.io/klog/v2.stacks(0xc00008c001, 0xc0000ae000, 0x36, 0x40)
	/usr/local/gopath/pkg/mod/k8s.io/klog/v2@v2.4.0/klog.go:1026 +0xb1
k8s.io/klog/v2.(*loggingT).output(0x5ad160, 0xc000000003, 0x0, 0x0, 0xc00009c000, 0x59484b, 0x8, 0x17, 0x40d000)
	/usr/local/gopath/pkg/mod/k8s.io/klog/v2@v2.4.0/klog.go:975 +0x168
k8s.io/klog/v2.(*loggingT).printDepth(0x5ad160, 0xc000000003, 0x0, 0x0, 0x0, 0x0, 0x1, 0xc000076370, 0x1, 0x1)
	/usr/local/gopath/pkg/mod/k8s.io/klog/v2@v2.4.0/klog.go:732 +0x171
k8s.io/klog/v2.(*loggingT).print(...)
	/usr/local/gopath/pkg/mod/k8s.io/klog/v2@v2.4.0/klog.go:714
k8s.io/klog/v2.Fatal(...)
	/usr/local/gopath/pkg/mod/k8s.io/klog/v2@v2.4.0/klog.go:1482
main.main()
	/root/k8s_dev/klog/test7.go:23 +0x47f

goroutine 19 [chan receive]:
```

参考连接：
[https://github.com/kubernetes/klog](https://github.com/kubernetes/klog)
