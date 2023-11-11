


**相关阅读**：

 1. [Kubernetes ImagePolicyWebhook与ValidatingAdmissionWebhook【1】动手实践感受区别所在](https://ghostwritten.blog.csdn.net/article/details/119712220)
 2. [KubernetesImagePolicyWebhook与ValidatingAdmissionWebhook【2】Image_Policy.go源码解析](https://ghostwritten.blog.csdn.net/article/details/119811128)
 3. [Kubernetes ImagePolicyWebhook与ValidatingAdmissionWebhook【3】validating_admission.go源码解析](https://ghostwritten.blog.csdn.net/article/details/119979444)
 4. [Kubernetes ImagePolicyWebhook与ValidatingAdmissionWebhook【4】main.go源码解析](https://ghostwritten.blog.csdn.net/article/details/119978143)
 5. [kubernetes 快速学习手册](https://ghostwritten.blog.csdn.net/article/details/108562082)

------------

## 1. 包的依赖
 1. [go fmt包详解](https://blog.csdn.net/xixihahalelehehe/article/details/104507198?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163021100016780366538558%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=163021100016780366538558&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-2-104507198.pc_v2_rank_blog_default&utm_term=fmt&spm=1018.2226.3001.4450)
 2. [go os包文件操作详解](https://ghostwritten.blog.csdn.net/article/details/105264641)
 3. [go strings包详解](https://blog.csdn.net/xixihahalelehehe/article/details/105097087?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163021130616780255286869%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=163021130616780255286869&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-1-105097087.pc_v2_rank_blog_default&utm_term=strings&spm=1018.2226.3001.4450)
 4. [echo包详解](https://echo.labstack.com/guide/)
 5. [echo源码](https://pkg.go.dev/github.com/labstack/echo)
 6. [https://pkg.go.dev/io](https://pkg.go.dev/io)
 7. [https://pkg.go.dev/os](https://pkg.go.dev/os)


项目源码：[https://github.com/kainlite/kube-image-bouncer](https://github.com/kainlite/kube-image-bouncer)
##  2. main.go
首先我们了解一下依赖包的各个用途。
```bash
import (
	"fmt"  //打印输出
	"os"  //系统管理
	"strings"  //字符串管理

	"github.com/labstack/echo"  //可扩展，轻量级的web框架
	"github.com/labstack/echo/middleware"  //echo web框架中间件
	"github.com/labstack/gommon/log"  //自定义的日志输出
	"gopkg.in/urfave/cli.v1"   //用于在 Go 中构建命令行应用程序

	"github.com/kainlite/kube-image-bouncer/handlers"
)
```
下面的代码其实是以cli包与echo包为主的运行，如果具体学习cli的用法可以参考[https://pkg.go.dev/gopkg.in/urfave/cli.v1](https://pkg.go.dev/gopkg.in/urfave/cli.v1)
```go
func main() {
//定义参数
	var cert, key, whitelist string  //证书，密钥，白名单
	var port int  //webhook的端口
	var debug bool //debug模式

//定义命令对象
	app := cli.NewApp()
	app.Name = "kube-image-bouncer"
	app.Usage = "webhook endpoint for kube image policy admission controller"

//定义命令参数
	app.Flags = []cli.Flag{
		cli.StringFlag{
			Name:        "cert, c",
			Usage:       "Path to the certificate to use",
			EnvVar:      "BOUNCER_CERTIFICATE",
			Destination: &cert,
		},
		cli.StringFlag{
			Name:        "key, k",
			Usage:       "Path to the key to use",
			EnvVar:      "BOUNCER_KEY",
			Destination: &key,
		},
		cli.StringFlag{
			Name:        "registry-whitelist",
			Usage:       "Comma separated list of accepted registries",
			EnvVar:      "BOUNCER_REGISTRY_WHITELIST",
			Destination: &whitelist,
		},
		cli.IntFlag{
			Name:        "port, p",
			Value:       1323,
			Usage:       "Port to listen to",
			EnvVar:      "BOUNCER_PORT",
			Destination: &port,
		},
		cli.BoolFlag{
			Name:        "debug",
			Usage:       "Enable extra debugging",
			EnvVar:      "BOUNCER_DEBUG",
			Destination: &debug,
		},
	}

//定义命令的调用方式
	app.Action = func(c *cli.Context) error {
		e := echo.New()
		e.POST("/image_policy", handlers.PostImagePolicy())
		e.POST("/", handlers.PostValidatingAdmission())

		e.Use(middleware.LoggerWithConfig(middleware.LoggerConfig{
			Format: "method=${method}, uri=${uri}, status=${status}\n",
		}))

		if debug {
			e.Logger.SetLevel(log.DEBUG)
		}

		if whitelist != "" {
			handlers.RegistryWhitelist = strings.Split(whitelist, ",")
			fmt.Printf(
				"Accepting only images from these registries: %+v\n",
				handlers.RegistryWhitelist)
			fmt.Println("WARN: this feature is implemented only by the ValidatingAdmissionWebhook code")
		} else {
			fmt.Println("WARN: accepting images from ALL registries")
		}

		var err error
		if cert != "" && key != "" {
			err = e.StartTLS(fmt.Sprintf(":%d", port), cert, key)
		} else {
			err = e.Start(fmt.Sprintf(":%d", port))
		}

		if err != nil {
			return cli.NewExitError(err, 1)
		}

		return nil
	}

//运行app对象
	app.Run(os.Args)
}
```

main.go主要设计两个依赖包的用法：

 - cli.v1
 - echo

### 2.1 先分析cli.v1的应用
创建一个`newapp`，`go build` 生成二进制命令则是`app.name`的值。
![在这里插入图片描述](https://img-blog.csdnimg.cn/999d70fe86ea438f8ab58331e4d84baa.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/4bd97ba6a45543838a847fa038245e30.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
这里是除了配置命令的名字，用法说明，还有配置命令行参数，这里有包含四个，分别是`证书，密钥、白名单、端口。`，白名单是我们需要的仓库，即来自白名单的仓库镜像才是我们可以用的，端口是我们程序占用的端口。那证书与密钥又是用来开启`https`。
![在这里插入图片描述](https://img-blog.csdnimg.cn/bc86d5874d944cd89f439b4e2f630138.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
每个参数进行设置详细的类型说明。
![在这里插入图片描述](https://img-blog.csdnimg.cn/c978b38fc27d4f3dbcc1d09258d94e43.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_15,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b400493ed59c4d8faefa317b9a0bbc8c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_12,color_FFFFFF,t_70,g_se,x_16)
`app.Action`是我们要执行的操作。
![在这里插入图片描述](https://img-blog.csdnimg.cn/712d6348190b430c99d8946d38dd2445.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/a10732a0274846148c4d3a0305e00e55.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)

`e := echo.New()`创建一个名叫e的实例。
![在这里插入图片描述](https://img-blog.csdnimg.cn/a8d17d0b525c4f7aa8e6de869259306f.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/ab798563f18942c5a87ccdc7a9ad27c3.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
`e.use`是使用将中间件添加到在路由器之后运行的链中，也就是说这里是自定义log输出。
具体解释参考[https://studygolang.com/articles/11740](https://studygolang.com/articles/11740)
![在这里插入图片描述](https://img-blog.csdnimg.cn/52b77d1b44344033ad10467f76333995.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/6a234afbec6940bfa3161158280b40fa.png)
### 2.2  LoggerWithConfig是这么含义？
它来自`github.com/labstack/echo/middleware`，通过查找，在[https://github.com/labstack/echo/blob/master/middleware/logger.go](https://github.com/labstack/echo/blob/master/middleware/logger.go)文件中。描述功能为一个中间件配置的日志。如下：

```go

type (
	// LoggerConfig defines the config for Logger middleware.
	LoggerConfig struct {
		// Skipper defines a function to skip middleware.
		Skipper Skipper
		Format string `yaml:"format"`

		// Optional. Default value DefaultLoggerConfig.CustomTimeFormat.
		CustomTimeFormat string `yaml:"custom_time_format"`

		// Output is a writer where logs in JSON format are written.
		// Optional. Default value os.Stdout.
		Output io.Writer

		template *fasttemplate.Template
		colorer  *color.Color
		pool     *sync.Pool
	}
)

var (
	// DefaultLoggerConfig is the default Logger middleware config.
	DefaultLoggerConfig = LoggerConfig{
		Skipper: DefaultSkipper,
		Format: `{"time":"${time_rfc3339_nano}","id":"${id}","remote_ip":"${remote_ip}",` +
			`"host":"${host}","method":"${method}","uri":"${uri}","user_agent":"${user_agent}",` +
			`"status":${status},"error":"${error}","latency":${latency},"latency_human":"${latency_human}"` +
			`,"bytes_in":${bytes_in},"bytes_out":${bytes_out}}` + "\n",
		CustomTimeFormat: "2006-01-02 15:04:05.00000",
		colorer:          color.New(),
	}
)

// LoggerWithConfig returns a Logger middleware with config.
// See: `Logger()`.
func LoggerWithConfig(config LoggerConfig) echo.MiddlewareFunc {
	// Defaults
	if config.Skipper == nil {
		config.Skipper = DefaultLoggerConfig.Skipper
	}
	if config.Format == "" {
		config.Format = DefaultLoggerConfig.Format
	}
	if config.Output == nil {
		config.Output = DefaultLoggerConfig.Output
	}

	config.template = fasttemplate.New(config.Format, "${", "}")
	config.colorer = color.New()
	config.colorer.SetOutput(config.Output)
	config.pool = &sync.Pool{
		New: func() interface{} {
			return bytes.NewBuffer(make([]byte, 256))
		},
	}
```


`middleware.LoggerConfig{Format: "method=${method}, uri=${uri}, status=${status}\n",}`等于`config LoggerConfig`

`config.Skipper`并没有明确的定义，但这里是给予了默认值`DefaultLoggerConfig.Skipper`即`defaultSkipper`，暂认为是一种配置标识。
`config.Format`给出了输出选择属性，即`method=${method}, uri=${uri}, status=${status}\n`
`config.Output`默认输出一种json格式。

**当我们推理到这里的时候，就会发现我们又要开启对各个莫名其妙的调用包的理解之旅了，因为它是我们的成长之路。**

首先我们先明确一下`LoggerWithConfig`函数传入了什么？返回了什么？以什么格式返回输出？

 - 传入了什么？`config LoggerConfig`代码已写出。
 - 返回了什么？`bytes.NewBuffer(make([]byte, 256))`
![在这里插入图片描述](https://img-blog.csdnimg.cn/0e26b52454de419595c61c76b34d46a8.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
 - 以什么格式返回输出？ `echo.MiddlewareFunc`

![在这里插入图片描述](https://img-blog.csdnimg.cn/c34e4a8f75c947bd9cbf63dadda8f4f3.png)

```bash
type Context interface {
	Request() *http.Request
	SetRequest(r *http.Request)
	Response() *Response
	IsTLS() bool
	IsWebSocket() bool
	Scheme() string
	RealIP() string
	Path() string
	SetPath(p string)
	Param(name string) string
	ParamNames() []string
	SetParamNames(names ...string)
	ParamValues() []string
	SetParamValues(values ...string)
	QueryParam(name string) string
	QueryParams() url.Values
	QueryString() string
	FormValue(name string) string
	FormParams() (url.Values, error)
	FormFile(name string) (*multipart.FileHeader, error)
	MultipartForm() (*multipart.Form, error)
	Cookie(name string) (*http.Cookie, error)
	SetCookie(cookie *http.Cookie)
	Cookies() []*http.Cookie
	Get(key string) interface{}
	Set(key string, val interface{})
	Bind(i interface{}) error
	Validate(i interface{}) error
	Render(code int, name string, data interface{}) error
	HTML(code int, html string) error
	HTMLBlob(code int, b []byte) error
	String(code int, s string) error
	JSON(code int, i interface{}) error
	JSONPretty(code int, i interface{}, indent string) error
	JSONBlob(code int, b []byte) error
	JSONP(code int, callback string, i interface{}) error
	JSONPBlob(code int, callback string, b []byte) error
	XML(code int, i interface{}) error
	XMLPretty(code int, i interface{}, indent string) error
	XMLBlob(code int, b []byte) error
	Blob(code int, contentType string, b []byte) error
	Stream(code int, contentType string, r io.Reader) error
	File(file string) error
	Attachment(file string, name string) error
	Inline(file string, name string) error
	NoContent(code int) error
	Redirect(code int, url string) error
	Error(err error)
	Handler() HandlerFunc
	SetHandler(h HandlerFunc)
	Logger() Logger
	Echo() *Echo
	Reset(r *http.Request, w http.ResponseWriter)
}
```


`NewBuffer`使用`buf`作为它的初始内容来创建和初始化一个新的Buffer。新的Buffer获得了buf的所有权，调用者在调用之后不应该使用buf。NewBuffer用于准备一个Buffer来读取现有数据。它还可以用于设置用于写入的内部缓冲区的初始大小。要做到这一点，buf应该具有所需的容量，但长度为零。

再次，我们分析`LoggerWithConfig`的过程


####  2.2.1  `fasttemplate.New(config.Format, "${", "}")`是什么？
来自[https://github.com/valyala/fasttemplate/blob/master/template.go](https://github.com/valyala/fasttemplate/blob/master/template.go)
New使用给定的`startTag`和`endTag`作为标签开始和结束来解析给定的模板。返回的模板可以通过使用`Execute*`方法并发运行`goroutines`来执行。如果给定的模板无法解析，则会出现新的恐慌。如果模板可能包含错误，请使用`NewTemplate`。

 - `config.Format`即是`template`
 - `"${", "}"`即是开始结束标识符

![在这里插入图片描述](https://img-blog.csdnimg.cn/32cef8c563a04617b61608bd4d6557ce.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/a31548e878d14967802edf1937807274.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/0c4504de119046d58a99011f416da46e.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
reset方法中重定义t结构体的变量名后，利用 `unsafeString2Bytes()`对各个变量进行了处理，那么它的逻辑什么？
在`fasttemplate`包中的`unsafe.go`代码我们找到了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/0b675465160d4e80a1e070415e37a49b.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
这里用到了reflect包和unsafe包，他们的作用分别是：

 - `reflect`：反射就是用来检测存储在接口变量内部(值`value`；类型`concrete type) pair`对的一种机制。
 - `unsafe`: 指向不同类型数据的指针，是无法直接相互转换的，必须借助`unsafe.Pointer`

![在这里插入图片描述](https://img-blog.csdnimg.cn/12f9b3c7b74445eaaeeee10cecf3c42b.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
`StringHeader`是字符串的运行时表示。它不能安全或便携地使用，它的表示形式可能会在以后的版本中改变。而且，Data字段不足以保证它引用的数据不会被垃圾收集，因此程序必须保持一个独立的、正确键入的指向底层数据的指针.
![在这里插入图片描述](https://img-blog.csdnimg.cn/a3c59d64d0de46a39f9057a347e62c80.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
`Uintptr`是一个整数类型，它大到足以容纳任何指针的位模式
len当然是整数类型

![在这里插入图片描述](https://img-blog.csdnimg.cn/cd227b01386e41d8bade3c433a0b14ae.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
`SliceHeader`是片的运行时表示。它不能安全或便携地使用，它的表示形式可能会在以后的版本中改变。而且，Data字段不足以保证它引用的数据不会被垃圾收集，因此程序必须保持一个独立的、正确键入的指向底层数据的指针。

`unsafeString2Bytes`其实就是将字符串的格式转换为切片类型，而go格式转换需要`unsafe.Point`方法。

分析到这里，我们要倒推一下s是什么？
`unsafeString2Bytes(s string)`---->`unsafeString2Bytes(template)`----`template是Template的结构体一部分`----->New(template, `startTag, endTag string)`------------>`fasttemplate.New(config.Format, "${", "}")`---->LoggerWithConfig(config LoggerConfig) ------>iddleware.LoggerWithConfig(middleware.LoggerConfig{
			Format: "method=${method}, uri=${uri}, status=${status}\n",
		})

`template`即是`Format: "method=${method}, uri=${uri}, status=${status}\n"`

回到[fasttemplate代码](https://github.com/valyala/fasttemplate/blob/master/template.go)中的`reset`函数中，继续往下分析。

 - 当s、a、b都转换为切片格式之后，利用`bytes.count()`方法统计s中a的数量。
 - 随后，对`t.texts`与`t.tags`初始化切片分配`tagcount`内存容量。
 - 判断s中是否有a起始标志并返回它的位置，如果为空不存在，即返回-1，当＜0时，直接将s合并给t.texts格式并退出。
 - 如果n＞0，把s中第一个a之前的切片输出给`t.texts`
 - s = s[n+len(a):]则是重定义s中出现a之后的切片重新赋值给s
 - 然后判断s中是否有b结尾标志的数量，假如没有，则报错。因为有起始必有结尾才行。
 - `append(t.tags, unsafeBytes2String(s[:n]))`是把起始到结尾之间的切片赋值给`t.tags`,
 - 最后s再次重新定义为s中b之后的切片内容。

**回到包含reset函数的`NewTemplate`函数中看，NewTemplate其实是通过`"${"`与`"}"`对`template`格式进行一次检验，因为reset没有确切的返回值，只是返回是否报错而已，但是reset设定了t结构体的属性值内容，虽然没有在NewTemplate返回输出，但包含`NewTemplate`函数的`New`函数输出了**。t结构体包含的template、startTag、endTag、texts、tags   都有了具体内容。 

`new`函数返回输出在[echo.middleware代码](https://github.com/labstack/echo/blob/master/middleware/logger.go)中`LoggerWithConfig`函数有体现，给了`config.template`，即属于`*fasttemplate.Template`类型，是原来t的同一个结构体而已。

我们继续分析`LoggerWithConfig`。

#### 2.2.2  `config.colorer = color.New()`代码中`new()`又如何理解呢？
它来自[https://github.com/labstack/gommon/blob/master/color/color.go](https://github.com/labstack/gommon/blob/master/color/color.go)
![在这里插入图片描述](https://img-blog.csdnimg.cn/4608cd28fc864957b3060072a51968c3.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
在New函数中，我们应当知道new函数的语法：`func new(Type) *Type`，**内建函数 new 用来分配内存，第一个参数是一个类型，不是一个值，返回值是一个指向分配零值的指针**。

在关于c的`SetOutPut`方法中，`io.writer`利用了流的读写，bytes.Buffer是一个可变字节的类型，可以让我们很容易的对字节进行操作，比如读写，追加等。bytes.Buffer实现了io.Writer接口，所以我么可以很容易的进行读写操作。
[**io的源码**](https://pkg.go.dev/io)

`os.File`是打开文件
**[https://pkg.go.dev/os](https://pkg.go.dev/os)**
![在这里插入图片描述](https://img-blog.csdnimg.cn/950feee663d146f386897dfa1f3de7e5.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_18,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/9724631535134a4384e89d297a18319a.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
如果文件描述符是terminal则返回true,c.disabled=true,表示如果不是终端，则不开启的标志，也就是`LoggerWithConfig`函数能用到颜色输出的地方，只有终端，当人打开去看的时候，如果是终端输出日志，就给输出特定的颜色。

#### 2.2.3  sync.Pool又是什么？
通常用golang来构建高并发场景下的应用，但是由于golang内建的GC机制会影响应用的性能，为了减少GC，golang提供了对象重用的机制，也就是`sync.Pool`对象池。
[具体细节参考这篇文章](https://www.huaweicloud.com/articles/859ec483eb449ca338af0eb7967a9b8f.html)。

### 2.3 白名单、证书、密钥的判断
**回到[main.go](https://github.com/kainlite/kube-image-bouncer/blob/master/main.go)代码中来。**
![在这里插入图片描述](https://img-blog.csdnimg.cn/7a2917be127b476fa915e9740e603f10.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)

 - 假如whitelist不为空，通过`“，”`分割，因为白名单仓库可能是多个的，然后打印输出
 - 如果whitelist为空，说明介绍所有的镜像仓库来源
 - 假如证书与密钥存在，开启`https server`
 - 假如证书与密钥不存在，开启http server![在这里插入图片描述](https://img-blog.csdnimg.cn/92a03fd74f0245e0825435e61e96f0ca.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_19,color_FFFFFF,t_70,g_se,x_16)

最后，`app.run(os.args)`启动服务并显示文本。
![在这里插入图片描述](https://img-blog.csdnimg.cn/fa576eb07e744b2cb7034207471319a7.png)

## 3. 总结
编写一个webhook，[cli.v1](https://pkg.go.dev/gopkg.in/urfave/cli.v)可以帮助实现：

 - app.Name实现构建二进制的命令名字；
 - app.Usage描述使用方法；
 - app.Flags实现配置参数说明；
 - app.Action实现具体代码逻辑；
 - app.Run运行启动。

初次之外，[echo](https://pkg.go.dev/github.com/labstack/echo)，启动一个web客户端，并通过设定API接口实现具体逻辑。

handler目录写API接口的逻辑，主要包含获取需求的对象以及如何展示有用的对象信息。
rule目录写具体对获取的对象设定规则的逻辑。



