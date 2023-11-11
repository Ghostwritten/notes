
## 1. logrus介绍
golang标准库的日志框架非常简单,仅仅提供了print,panic和fatal三个函数对于更精细的日志级别、日志文件分割以及日志分发等方面并没有提供支持. 所以催生了很多第三方的日志库,但是在golang的世界里,没有一个日志库像slf4j那样在Java中具有绝对统治地位.golang中,流行的日志框架包括logrus、zap、zerolog、seelog等.

logrus是目前Github上star数量最多的日志库,目前(2018.12,下同)star数量为8119,fork数为1031. logrus功能强大,性能高效,而且具有高度灵活性,提供了自定义插件的功能.很多开源项目,如docker,prometheus,dejavuzhou/ginbro等,都是用了logrus来记录其日志.

zap是Uber推出的一个快速、结构化的分级日志库.具有强大的ad-hoc分析功能,并且具有灵活的仪表盘.zap目前在GitHub上的star数量约为4.3k. seelog提供了灵活的异步调度、格式化和过滤功能.目前在GitHub上也有约1.1k。

## 2. logrus特性

 - 完全兼容golang标准库日志模块：logrus拥有六种日志级别：`debug`、`info`、`warn`、`error`、`fatal`和`panic`,这是golang标准库日志模块的API的超集.如果您的项目使用标准库日志模块,完全可以以最低的代价迁移到logrus上.

第三方库需要先安装：



```bash
$ go get github.com/sirupsen/logrus
```

后使用：



```bash
package main

import (
  "github.com/sirupsen/logrus"
)

func main() {
  logrus.SetLevel(logrus.TraceLevel)

  logrus.Trace("trace msg")
  logrus.Debug("debug msg")
  logrus.Info("info msg")
  logrus.Warn("warn msg")
  logrus.Error("error msg")
  logrus.Fatal("fatal msg")
  logrus.Panic("panic msg")
}
```


logrus的使用非常简单，与标准库log类似。logrus支持更多的日志级别：



Panic：记录日志，然后panic。


Fatal：致命错误，出现错误时程序无法正常运转。输出日志后，程序退出；


Error：错误日志，需要查看原因；


Warn：警告信息，提醒程序员注意；


Info：关键操作，核心流程的日志；


Debug：一般程序中输出的调试信息；


Trace：很细粒度的信息，一般用不到；




日志级别从上向下依次增加，Trace最大，Panic最小。logrus有一个日志级别，高于这个级别的日志不会输出。默认的级别为InfoLevel。所以为了能看到Trace和Debug日志，我们在main函数第一行设置日志级别为TraceLevel。



运行程序，输出：



```bash
$ go run main.go
time="2020-02-07T21:22:42+08:00" level=trace msg="trace msg"
time="2020-02-07T21:22:42+08:00" level=debug msg="debug msg"
time="2020-02-07T21:22:42+08:00" level=info msg="info msg"
time="2020-02-07T21:22:42+08:00" level=info msg="warn msg"
time="2020-02-07T21:22:42+08:00" level=error msg="error msg"
time="2020-02-07T21:22:42+08:00" level=fatal msg="fatal msg"
exit status 1
```


由于logrus.Fatal会导致程序退出，下面的logrus.Panic不会执行到。



另外，我们观察到输出中有三个关键信息，time、level和msg：



time：输出日志的时间；


level：日志级别；


msg：日志信息。

 - 可扩展的Hook机制：允许使用者通过hook的方式将日志分发到任意地方,如本地文件系统、标准输出、logstash、elasticsearch或者mq等,或者通过hook定义日志内容和格式等.
   可选的日志输出格式：logrus内置了两种日志格式,JSONFormatter和TextFormatter,如果这两个格式不满足需求,可以自己动手实现接口Formatter,来定义自己的日志格式.
 - Field机制：logrus鼓励通过Field机制进行精细化的、结构化的日志记录,而不是通过冗长的消息来记录日志.
 - logrus是一个可插拔的、结构化的日志框架.
 - Entry: logrus.WithFields会自动返回一个 *Entry,Entry里面的有些变量会被自动加上

```bash
time:entry被创建时的时间戳
msg:在调用.Info()等方法时被添加
level
```
## 3. 基本用法
### 3.1 输出至终端平台
```go
package main

import "github.com/sirupsen/logrus"

func main() {
    logrus.WithFields(logrus.Fields{
        "animal": "walrus",
        "number": 1,
        "size":   10,
    }).Info("A walrus appears")
}
```

```bash
$  go run logrus1.go
INFO[0000] A walrus appears                              animal=walrus number=1 size=10
```
### 3.2 输出至文件

```go
package main

import (
  "github.com/sirupsen/logrus"
  "os"
)
func main() {
  log := logrus.New()
  file, err := os.OpenFile("logrus.log", os.O_CREATE|os.O_WRONLY, 0666)
  if err == nil {
     log.Out = file
  } else {
     log.Info("Failed to log to file, using default stderr")
  }
  log.Info("log-- log--")

}
```
输出：
```bash
$ cat logrus.log 
time="2020-05-26T10:29:20+08:00" level=info msg="log-- log--"
```

###  3.3 设置日志输出格式与日志输出级别
语法：

```bash
logrus.SetFormatter(&logrus.JSONFormatter{})
logrus.Infoln("JSONFormatter")

logrus.SetFormatter(&logrus.TextFormatter{})
logrus.Infoln("TextFormatter not time")
```


示例1
```go
package main

import (
    "os"
    log "github.com/sirupsen/logrus"
)

func init() {
    // 设置日志格式为json格式
    log.SetFormatter(&log.JSONFormatter{})

    // 设置将日志输出到标准输出（默认的输出为stderr,标准错误）
    // 日志消息输出可以是任意的io.writer类型
    log.SetOutput(os.Stdout)

    // 设置日志级别为warn以上
    log.SetLevel(log.WarnLevel)
}

func main() {
    log.WithFields(log.Fields{
        "animal": "walrus",
        "size":   10,
    }).Info("A group of walrus emerges from the ocean")

    log.WithFields(log.Fields{
        "omg":    true,
        "number": 122,
    }).Warn("The group's number increased tremendously!")

    log.WithFields(log.Fields{
        "omg":    true,
        "number": 100,
    }).Fatal("The ice breaks!")
}
```
输出：
```bash
$ go run logrus3.go
{"level":"warning","msg":"The group's number increased tremendously!","number":122,"omg":true,"time":"2020-05-26T10:53:39+08:00"}
{"level":"fatal","msg":"The ice breaks!","number":100,"omg":true,"time":"2020-05-26T10:53:39+08:00"}
exit status 1
```
### 3.4 自定义Logger
如果想在一个应用里面向多个地方log,可以创建Logger实例. logger是一种相对高级的用法, 对于一个大型项目, 往往需要一个全局的logrus实例,即logger对象来记录项目所有的日志.
语法：

```bash
log := logrus.New()
log.Formatter= new(logrus.JSONFormatter)
log.Infoln("my logger")
```

示例1:输出到终端

```go
package main

import (
    "github.com/sirupsen/logrus"
    "os"
)

// logrus提供了New()函数来创建一个logrus的实例.
// 项目中,可以创建任意数量的logrus实例.
var log = logrus.New()

func main() {
    // 为当前logrus实例设置消息的输出,同样地,
    // 可以设置logrus实例的输出到任意io.writer
    log.Out = os.Stdout

    // 为当前logrus实例设置消息输出格式为json格式.
    // 同样地,也可以单独为某个logrus实例设置日志级别和hook,这里不详细叙述.
    log.Formatter = &logrus.JSONFormatter{}

    log.WithFields(logrus.Fields{
        "animal": "walrus",
        "size":   10,
    }).Info("A group of walrus emerges from the ocean")
}
```
输出：

```bash
$ go run logrus4.go
{"animal":"walrus","level":"info","msg":"A group of walrus emerges from the ocean","size":10,"time":"2020-05-26T10:59:26+08:00"}
```
示例2：输出到文件

```go
package main

import (
	"github.com/sirupsen/logrus"
	"os"
)

var log = logrus.New()

func main() {
	file ,err := os.OpenFile("logrus.log", os.O_CREATE|os.O_WRONLY, 0666)
	if err == nil{
		log.Out = file
	}else{
		log.Info("Failed to log to file")
	}

	log.WithFields(logrus.Fields{
		"filename": "123.txt",
	}).Info("打开文件失败")
}
```
输出：

```bash
$ cat logrus.log 
time="2020-05-26T11:08:07+08:00" level=info msg="打开文件失败" filename=123.txt
```
### 3.5 Fields用法
logrus不推荐使用冗长的消息来记录运行信息,它推荐使用Fields来进行精细化的、结构化的信息记录. 例如下面的记录日志的方式：

```go
log.Fatalf("Failed to send event %s to topic %s with key %d", event, topic, key)
```

在logrus中不太提倡,logrus鼓励使用以下方式替代之：

```go
log.WithFields(log.Fields{
  "event": event,
  "topic": topic,
  "key": key,
}).Fatal("Failed to send event")
```

前面的WithFields API可以规范使用者按照其提倡的方式记录日志.但是WithFields依然是可选的,因为某些场景下,使用者确实只需要记录仪一条简单的消息.

通常,在一个应用中、或者应用的一部分中,都有一些固定的Field.比如在处理用户http请求时,上下文中,所有的日志都会有request_id和user_ip.为了避免每次记录日志都要使用log.WithFields(log.Fields{“request_id”: request_id, “user_ip”: user_ip}),我们可以创建一个logrus.Entry实例,为这个实例设置默认Fields,在上下文中使用这个logrus.Entry实例记录日志即可.

```go
package main

import (
	"github.com/sirupsen/logrus"
)

var log = logrus.New()

func main() {
	entry := logrus.WithFields(logrus.Fields{
		"name": "test",
	})
	entry.Info("message1")
	entry.Info("message2")
}
```
输出：

```bash
$ go run logrus6.go
INFO[0000] message1                                      name=test
INFO[0000] message2                                      name=test
```
### 3.6 hook
hook的原理是，在logrus写入日志时拦截，修改logrus.Entry

```bash
type Hook interface {
    Levels() []Level
    Fire(*Entry) error
}
```
使用示例：
自定义一个hook DefaultFieldHook,在所有级别的日志消息中加入默认字段

```go
appName="myAppName"

type DefaultFieldHook struct {
}

func (hook *DefaultFieldHook) Fire(entry *log.Entry) error {
    entry.Data["appName"] = "MyAppName"
    return nil
}

func (hook *DefaultFieldHook) Levels() []log.Level {
    return log.AllLevels
}
```

在初始化时，调用logrus.AddHook(hook)添加响应的hook即可
logrus官方仅仅内置了syslog的hook. 此外,但Github也有很多第三方的hook可供使用,文末将提供一些第三方HOOK的连接.
####  3.6.1 Logrus-Hook-Email
email这里只需用NewMailAuthHook方法得到hook,再添加即可

```go
package main

import (
	"time"

	//"github.com/logrus_mail"
        "github.com/zbindenren/logrus_mail"
	"github.com/sirupsen/logrus"
)

func main() {
	logger := logrus.New()
	hook, err := logrus_mail.NewMailAuthHook(
		"logrus_email",
		"smtp.163.com",
		25,
		"xxxxx@163.com",
		"xxxxxx@qq.com",
		"xxxxxxx@163.com",
		"xxxxxx",
	)
	if err == nil {
		logger.Hooks.Add(hook)
	}
	//生成*Entry
	var filename = "123.txt"
	contextLogger := logger.WithFields(logrus.Fields{
		"file":    filename,
		"content": "GG",
	})
	//设置时间戳和message
	contextLogger.Time = time.Now()
	contextLogger.Message = "这是一个hook发来的邮件"
	//只能发送Error,Fatal,Panic级别的log
	contextLogger.Level = logrus.ErrorLevel

	//使用Fire发送,包含时间戳，message
	hook.Fire(contextLogger)
}
```

#### 3.6.2 将日志发送到其他位置
将日志发送到日志中心也是logrus所提倡的，虽然没有提供官方支持，但是目前Github上有很多第三方hook可供使用：

[logrus_amqp](https://github.com/vladoatanasov/logrus_amqp)：Logrus hook for Activemq。
[logrus-logstash-hook](https://github.com/bshuster-repo/logrus-logstash-hook):Logstash hook for logrus。
[mgorus](https://github.com/weekface/mgorus):Mongodb Hooks for Logrus。
[logrus_influxdb](https://github.com/abramovic/logrus_influxdb):InfluxDB Hook for Logrus。
[logrus-redis-hook](https://github.com/rogierlommers/logrus-redis-hook):Hook for Logrus which enables logging to RELK stack (Redis, Elasticsearch, Logstash and Kibana)


####  3.6.3 Logrus-Hook 日志分隔
logrus本身不带日志本地文件分割功能,但是我们可以通过file-rotatelogs进行日志本地文件分割. 每次当我们写入日志的时候,logrus都会调用file-rotatelogs来判断日志是否要进行切分.
示例1：
```go
package main

import (
	"time"

	rotatelogs "github.com/lestrrat-go/file-rotatelogs"
	log "github.com/sirupsen/logrus"
)

func init() {
	path := "/Users/opensource/test/go.log"
	/* 日志轮转相关函数
	`WithLinkName` 为最新的日志建立软连接
	`WithRotationTime` 设置日志分割的时间，隔多久分割一次
	WithMaxAge 和 WithRotationCount二者只能设置一个
	  `WithMaxAge` 设置文件清理前的最长保存时间
	  `WithRotationCount` 设置文件清理前最多保存的个数
	*/
	// 下面配置日志每隔 1 分钟轮转一个新文件，保留最近 3 分钟的日志文件，多余的自动清理掉。
	writer, _ := rotatelogs.New(
		path+".%Y%m%d%H%M",
		rotatelogs.WithLinkName(path),
		rotatelogs.WithMaxAge(time.Duration(180)*time.Second),
		rotatelogs.WithRotationTime(time.Duration(60)*time.Second),
	)
	log.SetOutput(writer)
	//log.SetFormatter(&log.JSONFormatter{})
}

func main() {
	for {
		log.Info("hello, world!")
		time.Sleep(time.Duration(2) * time.Second)
	}
}
```

示例2：

```go
package main
 
import (
	"github.com/lestrrat-go/file-rotatelogs"
	"github.com/pkg/errors"
	"github.com/rifflock/lfshook"
	log "github.com/sirupsen/logrus"
	"path"
	"time"
)
 
func ConfigLocalFilesystemLogger(logPath string, logFileName string, maxAge time.Duration, rotationTime time.Duration) {
	baseLogPaht := path.Join(logPath, logFileName)
	writer, err := rotatelogs.New(
		baseLogPaht+".%Y%m%d%H%M",
		//rotatelogs.WithLinkName(baseLogPaht), // 生成软链，指向最新日志文件
		rotatelogs.WithMaxAge(maxAge), // 文件最大保存时间
		rotatelogs.WithRotationTime(rotationTime), // 日志切割时间间隔
	)
	if err != nil {
		log.Errorf("config local file system logger error. %+v", errors.WithStack(err))
	}
	lfHook := lfshook.NewHook(lfshook.WriterMap{
		log.DebugLevel: writer, // 为不同级别设置不同的输出目的
		log.InfoLevel:  writer,
		log.WarnLevel:  writer,
		log.ErrorLevel: writer,
		log.FatalLevel: writer,
		log.PanicLevel: writer,
	},&log.TextFormatter{DisableColors: true})
	log.AddHook(lfHook)
}
 
 
//切割日志和清理过期日志
func ConfigLocalFilesystemLogger1(filePath string) {
	writer, err := rotatelogs.New(
		filePath+".%Y%m%d%H%M",
		rotatelogs.WithLinkName(filePath),         // 生成软链，指向最新日志文件
		rotatelogs.WithMaxAge(time.Second*60*3),     // 文件最大保存时间
		rotatelogs.WithRotationTime(time.Second*60), // 日志切割时间间隔
	)
	if err != nil {
		log.Fatal("Init log failed, err:", err)
	}
	log.SetOutput(writer)
	log.SetLevel(log.InfoLevel)
}
 
func main()  {
	ConfigLocalFilesystemLogger1("log")
	for {
		log.Info(111)
		time.Sleep(500*time.Millisecond)
	}
}
```

### 3.7 Fatal处理
和很多日志框架一样，logrus的Fatal系列函数会执行os.Exit(1)。但是logrus提供可以注册一个或多个fatal handler函数的接口`logrus.RegisterExitHandler(handler func() {} )`，让logrus在执行os.Exit(1)之前进行相应的处理。fatal handler可以在系统异常时调用一些资源释放api等，让应用正确的关闭。

### 3.8 线程安全
默认情况下，logrus的api都是线程安全的，其内部通过互斥锁来保护并发写。互斥锁工作于调用hooks或者写日志的时候，如果不需要锁，可以调用logger.SetNoLock()来关闭之。可以关闭logrus互斥锁的情形包括：

 - 没有设置hook，或者所有的hook都是线程安全的实现。
 - 写日志到logger.Out已经是线程安全的了，如logger.Out已经被锁保护，或者写文件时，文件是以O_APPEND方式打开的，并且每次写操作都小于4k。

### 3.9 输出代码文件名
调用logrus.SetReportCaller(true)设置在输出日志中添加文件名和方法信息：



```bash
package main

import (
  "github.com/sirupsen/logrus"
)

func main() {
  logrus.SetReportCaller(true)

  logrus.Info("info msg")
}
```

输出多了两个字段file为调用logrus相关方法的文件名，method为方法名：



```bash
$ go run main.go
time="2020-02-07T21:46:03+08:00" level=info msg="info msg" func=main.main file="D:/code/golang/src/github.com/darjun/go-daily-lib/logrus/caller/main.go:10"
```



添加字段

参考链接：
[jack-life](https://blog.csdn.net/wslyk606/article/details/81670713)
[Go进阶10:logrus日志使用教程](https://mojotv.cn/2018/12/27/golang-logrus-tutorial)
[Go 每日一库之 logrus](https://zhuanlan.zhihu.com/p/105759117)

相关阅读：
[go log包详解](https://blog.csdn.net/xixihahalelehehe/article/details/105256439)



