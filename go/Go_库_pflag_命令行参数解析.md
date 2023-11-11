#  Go 库 pflag 命令行参数解析

![](https://img-blog.csdnimg.cn/df2409c6893f41e4888b8bfa97855f09.png)





##  1. 背景
Go 服务开发中，经常需要给开发的组件加上各种启动参数来配置服务进程，影响服务的行为。像 kube-apiserver 就有多达 200 多个启动参数，而且这些参数的类型各不相同（例如：string、int、ip 类型等），使用方式也不相同（例如：需要支持--长选项，-短选项等），所以我们需要一个强大的命令行参数解析工具。

虽然 Go 源码中提供了一个标准库 Flag 包，用来对命令行参数进行解析，但在大型项目中应用更广泛的是另外一个包：Pflag。Pflag 提供了很多强大的特性，非常适合用来构建大型项目，一些耳熟能详的开源项目都是用 Pflag 来进行命令行参数解析的，例如：Kubernetes、Istio、Helm、Docker、Etcd 等。接下来，我们就来介绍下如何使用 Pflag。Pflag 主要是通过创建 Flag 和 FlagSet 来使用的。我们先来看下 Flag。

## 2. pflag 简介
pflag是Go的flag包的直接替代，实现了POSIX / GNU样式的--flags。pflag是Go的本机标志包的直接替代。如果您在名称“ flag”下导入pflag，则所有代码应继续运行且无需更改。

github：[https://github.com/spf13/pflag](https://github.com/spf13/pflag)
源码包：[https://godoc.org/github.com/spf13/pflag#Args](https://godoc.org/github.com/spf13/pflag#Args)

默认标志位：`--`
```bash
--flag    // boolean flags, or flags with no option default values
--flag x  // only on flags without a default value
--flag=x
```
## 3. Pflag 包 Flag 定义
Pflag 可以对命令行参数进行处理，一个命令行参数在 Pflag 包中会解析为一个 Flag 类型的变量。Flag 是一个结构体，定义如下：
```bash
type Flag struct {
    Name                string // flag长选项的名称
    Shorthand           string // flag短选项的名称，一个缩写的字符
    Usage               string // flag的使用文本
    Value               Value  // flag的值
    DefValue            string // flag的默认值
    Changed             bool // 记录flag的值是否有被设置过
    NoOptDefVal         string // 当flag出现在命令行，但是没有指定选项值时的默认值
    Deprecated          string // 记录该flag是否被放弃
    Hidden              bool // 如果值为true，则从help/usage输出信息中隐藏该flag
    ShorthandDeprecated string // 如果flag的短选项被废弃，当使用flag的短选项时打印该信息
    Annotations         map[string][]string // 给flag设置注解
}
```
Flag 的值是一个 Value 类型的接口，Value 定义如下：

```bash
type Value interface {
    String() string // 将flag类型的值转换为string类型的值，并返回string的内容
    Set(string) error // 将string类型的值转换为flag类型的值，转换失败报错
    Type() string // 返回flag的类型，例如：string、int、ip等
}
```
通过将 Flag 的值抽象成一个 interface 接口，我们就可以自定义 Flag 的类型了。只要实现了 Value 接口的结构体，就是一个新类型。

## 4. Pflag 包 FlagSet 定义
Pflag 除了支持单个的 Flag 之外，还支持 FlagSet。FlagSet 是一些预先定义好的 Flag 的集合，几乎所有的 Pflag 操作，都需要借助 FlagSet 提供的方法来完成。在实际开发中，我们可以使用两种方法来获取并使用 FlagSet：

- 方法一，调用 `NewFlagSet` 创建一个 FlagSet。
- 方法二，使用 Pflag 包定义的全局 FlagSet：`CommandLine`。实际上 CommandLine 也是由 NewFlagSet 函数创建的。

先来看下第一种方法，自定义 FlagSet。下面是一个自定义 `FlagSet` 的示例：

```bash
var version bool
flagSet := pflag.NewFlagSet("test", pflag.ContinueOnError)
flagSet.BoolVar(&version, "version", true, "Print version information and quit.")
```
我们可以通过定义一个新的 FlagSet 来定义命令及其子命令的 Flag。再来看下第二种方法，使用全局 `FlagSet`。下面是一个使用全局 FlagSet 的示例：

```bash
import (
    "github.com/spf13/pflag"
)

pflag.BoolVarP(&version, "version", "v", true, "Print version information and quit.")
```
这其中，`pflag.BoolVarP` 函数定义如下：

```bash
func BoolVarP(p *bool, name, shorthand string, value bool, usage string) {
    flag := CommandLine.VarPF(newBoolValue(value, p), name, shorthand, usage)
    flag.NoOptDefVal = "true"
}
```
可以看到 `pflag.BoolVarP` 最终调用了 `CommandLine`，`CommandLine` 是一个包级别的变量，定义为：

```bash
// CommandLine is the default set of command-line flags, parsed from os.Args.
var CommandLine = NewFlagSet(os.Args[0], ExitOnError)
```


## 5. 安装

```bash
go get github.com/spf13/pflag
```
## 6. 方法
### 6.1 支持多种命令行参数定义方式。
支持长选项、默认值和使用文本，并将标志的值存储在指针中。

```bash
var name = pflag.String("name", "colin", "Input Your Name")
```

支持长选项、短选项、默认值和使用文本，并将标志的值存储在指针中。

```bash
var name = pflag.StringP("name", "n", "colin", "Input Your Name")
```
支持长选项、默认值和使用文本，并将标志的值绑定到变量。

```bash
var name string
pflag.StringVar(&name, "name", "colin", "Input Your Name")
```
支持长选项、短选项、默认值和使用文本，并将标志的值绑定到变量。

```bash
var name string
pflag.StringVarP(&name, "name", "n","colin", "Input Your Name")
```
上面的函数命名是有规则的：
- 函数名带Var说明是将标志的值绑定到变量，否则是将标志的值存储在指针中。
- 函数名带P说明支持短选项，否则不支持短选项。

### 6.2 使用Get<Type>获取参数的值
可以使用`Get<Type>`来获取标志的值，`<Type>`代表 Pflag 所支持的类型。例如：有一个 `pflag.FlagSet`，带有一个名为 `flagname` 的 int 类型的标志，可以使用`GetInt()`来获取 int 值。需要注意 flagname 必须存在且必须是 int，例如：

```bash
i, err := flagset.GetInt("flagname")
```
### 6.3 获取非选项参数

```bash
package main

import (
    "fmt"

    "github.com/spf13/pflag"
)

var (
    flagvar = pflag.Int("flagname", 1234, "help message for flagname")
)

func main() {
    pflag.Parse()

    fmt.Printf("argument number is: %v\n", pflag.NArg())
    fmt.Printf("argument list is: %v\n", pflag.Args())
    fmt.Printf("the first argument is: %v\n", pflag.Arg(0))
}
```
执行上述代码，输出如下：

```bash
$ go run example1.go arg1 arg2
argument number is: 2
argument list is: [arg1 arg2]
the first argument is: arg1
```
在定义完标志之后，可以调用`pflag.Parse()`来解析定义的标志。解析后，可通过`pflag.Args()`返回所有的非选项参数，通过`pflag.Arg(i)`返回第 i 个非选项参数。参数下标 0 到 pflag.NArg() - 1。

### 6.4 指定了选项但是没有指定选项值时的默认值
创建一个 Flag 后，可以为这个 Flag 设置`pflag.NoOptDefVal`。如果一个 Flag 具有 NoOptDefVal，并且该 Flag 在命令行上没有设置这个 Flag 的值，则该标志将设置为 NoOptDefVal 指定的值。例如：

```bash
var ip = pflag.IntP("flagname", "f", 1234, "help message")
pflag.Lookup("flagname").NoOptDefVal = "4321"
```
上面的代码会产生结果，具体你可以参照下表
![](https://img-blog.csdnimg.cn/83fb14e8cd374531aa81863fcecf1b32.png)
### 6.5 弃用标志或者标志的简写
Pflag 可以弃用标志或者标志的简写。弃用的标志或标志简写在帮助文本中会被隐藏，并在使用不推荐的标志或简写时打印正确的用法提示。例如，弃用名为 logmode 的标志，并告知用户应该使用哪个标志代替：

```bash
// deprecate a flag by specifying its name and a usage message
pflag.CommandLine.MarkDeprecated("logmode", "please use --log-mode instead")
```
这样隐藏了帮助文本中的 logmode，并且当使用 logmode 时，打印了Flag --logmode has been deprecated, please use --log-mode instead。

### 6.6 保留名为 port 的标志，但是弃用它的简写形式

```bash
pflag.IntVarP(&port, "port", "P", 3306, "MySQL service host port.")

// deprecate a flag shorthand by specifying its flag name and a usage message
pflag.CommandLine.MarkShorthandDeprecated("port", "please use --port only")
```
这样隐藏了帮助文本中的简写 P，并且当使用简写 P 时，打印了Flag shorthand -P has been deprecated, please use --port only。usage message 在此处必不可少，并且不应为空。

### 6.7 隐藏标志
可以将 Flag 标记为隐藏的，这意味着它仍将正常运行，但不会显示在 usage/help 文本中。例如：隐藏名为 secretFlag 的标志，只在内部使用，并且不希望它显示在帮助文本或者使用文本中。代码如下：

```bash
// hide a flag by specifying its name
pflag.CommandLine.MarkHidden("secretFlag")
```
## 7. 实例
### 7.1 练习1

```bash
package main

import (
    flag "github.com/spf13/pflag"  //替换原生的flag，并兼容
    "fmt"
)

var flagvar1 int
var flagvar2 bool

func init() {
    flag.IntVar(&flagvar1, "varname1", 1, "help message for flagname")
   flag.BoolVarP(&flagvar2, "boolname1", "b", true, "help message")
}


func main() {
   var ip1 *int = flag.Int("flagname1", 1, "help message for flagname")
   
   var ip2 = flag.IntP("flagname2", "f", 2, "help message") 
   
   

   flag.Parse()
   
   fmt.Println("ip1 has value ", *ip1)
   fmt.Println("ip2 has value ", *ip2)
   fmt.Println("flagvar1 has value ", flagvar1) 
   fmt.Println("flagvar2 has value ", flagvar2) 
}
```

```bash
$ go build fplag1.go
/pflag1 -h
Usage of ./pflag1:
  -b, --boolname1       help message (default true)
      --flagname1 int   help message for flagname (default 1)
  -f, --flagname2 int   help message (default 2)
      --varname1 int    help message for flagname (default 1)
pflag: help requested
```


### 7.2 练习2
```bash
package main

import (
   "github.com/spf13/pflag"
   "net"
   "fmt"
   "time"
)

func pflagDefine() {
   //64位整数，不带单标志位的
   var pflagint64 *int64 = pflag.Int64("number1", 1234, "this is int 64, without single flag")

   //64位整数，带单标志位的
   var pflagint64p *int64 = pflag.Int64P("number2", "n", 2345, "this is int 64, without single flag")

    //这种可以把变量的定义和变量取值分开，适合于struct，全局变量等地方
   var pflagint64var int64
   pflag.Int64Var(&pflagint64var, "number3", 1234, "this is int64var")

   //上面那一种的增加短标志位版
   var pflagint64varp int64
   pflag.Int64VarP(&pflagint64varp,"number4", "m", 1234, "this is int64varp")



    //slice版本,其实是上面的增强版，但是支持多个参数，也就是导成一个slice
    var pflagint64slice *[]int64 = pflag.Int64Slice("number5", []int64{1234, 3456}, "this is int64 slice")

    //bool版本
    var pflagbool *bool = pflag.Bool("bool", true, "this is bool")

    //bytes版本
    var pflagbyte *[]byte = pflag.BytesBase64("byte64", []byte("ea"), "this is byte base64")

    //count版本
    var pflagcount *int= pflag.Count("count", "this is count")

    //duration版本
    var pflagduration *time.Duration = pflag.Duration("duration", 10* time.Second, "this is duration")

    //float版本
    var pflagfloat *float64 = pflag.Float64("float64", 123.345, "this is florat64")

    //IP版本
   var pflagip *net.IP = pflag.IP("ip1", net.IPv4(192, 168, 1, 1), "this is ip, without single flag")

   //mask版本
    var pflagmask *net.IPMask= pflag.IPMask("mask", net.IPv4Mask(255,255,255,128),"this is net mask")

   //string版本
    var pflagstring *string= pflag.String("string", "teststring", "this is string")

   //uint版本
   var pflaguint *uint64 = pflag.Uint64("uint64", 12345, "this is uint64")

   pflag.Parse()
   fmt.Println("number1 int64 is ", *pflagint64)
   fmt.Println("number2 int64 is ", *pflagint64p)
   fmt.Println("number3 int64var is ", pflagint64var)
   fmt.Println("number4 int64varp is", pflagint64varp)
   fmt.Println("number5 int64slice is", *pflagint64slice)
    fmt.Println("bool is ", *pflagbool)
    fmt.Println("byte64 is ", *pflagbyte)
    fmt.Println("count is ", *pflagcount)
    fmt.Println("duration is ", *pflagduration)
    fmt.Println("float is ", *pflagfloat)
   fmt.Println("ip1 net.ip is ", *pflagip)
    fmt.Println("mask is %s", *pflagmask)
    fmt.Println("string is ", *pflagstring)
   fmt.Println("uint64 is ", *pflaguint)

}

func main() {
   pflagDefine()

}
```

```bash
$ go build pflag2.go
$ ./pflag2 -h
Usage of ./pflag1:
      --bool                 this is bool (default true)
      --byte64 bytesBase64   this is byte base64 (default ZWE=)
      --count count          this is count
      --duration duration    this is duration (default 10s)
      --float64 float        this is florat64 (default 123.345)
      --ip1 ip               this is ip, without single flag (default 
      --mask ipMask          this is net mask (default ffffff80)
      --number1 int          this is int 64, without single flag (defa
  -n, --number2 int          this is int 64, without single flag (defa
      --number3 int          this is int64var (default 1234)
  -m, --number4 int          this is int64varp (default 1234)
      --number5 int64Slice   this is int64 slice (default [1234,3456])
      --string string        this is string (default "teststring")
      --uint64 uint          this is uint64 (default 12345)
pflag: help requested
```
### 7.3 练习3. flag.Lookup
flag包中提供了一种类似上述的”配置中心”的机制，但这种机制不需要我们显示注入“flag vars”了，我们只需按照flag提供的方法在其他package中读取对应flag变量的值即可。

```bash
$tree flaglookup
flaglookup
├── etcd
│   └── etcd.go
└── main.go
```

```bash
// flag-demo/flaglookup/main.go
package main

import (
    "flag"
    "fmt"
    "time"

    "./etcd"
)

var (
    endpoints string
    user      string
    password  string
)

func init() {
    flag.StringVar(&endpoints, "endpoints", "127.0.0.1:2379", "comma-separated list of etcdv3 endpoints")
    flag.StringVar(&user, "user", "", "etcdv3 client user")
    flag.StringVar(&password, "password", "", "etcdv3 client password")
}

func usage() {
     fmt.Println("flagdemo-app is a daemon application which provides xxx service.\n")
     fmt.Println("Usage of flagdemo-app:\n")
     fmt.Println("\t flagdemo-app [options]\n")
    fmt.Println("The options are:\n")

    flag.PrintDefaults()
}


func main() {
    flag.Usage = usage
    flag.Parse()

    go etcd.EtcdProxy()

    time.Sleep(5 * time.Second)
}

```
```bash
// flag-demo/flaglookup/etcd/etcd.go
package etcd

import (
    "flag"
    "fmt"
)

func EtcdProxy() {
    endpoints := flag.Lookup("endpoints").Value.(flag.Getter).Get().(string)
    user := flag.Lookup("user").Value.(flag.Getter).Get().(string)
    password := flag.Lookup("password").Value.(flag.Getter).Get().(string)

    fmt.Println(endpoints, user, password)
}
```

```bash
[root@localhost flaglookup]# go run main.go -endpoints 192.168.10.69:2379,10.10.12.36:2378 -user tonybai -password xyz123
192.168.10.69:2379,10.10.12.36:2378 tonybai xyz123
```

##  7.4 flag与pflag混用
混用flag及pflag时，注意使用的方法

```bash
import (
	goflag "flag"
	flag "github.com/spf13/pflag"
)

var ip *int = flag.Int("flagname", 1234, "help message for flagname")

func main() {
	flag.CommandLine.AddGoFlagSet(goflag.CommandLine)
	flag.Parse()
}
```

参考：
- [https://www.cnblogs.com/sparkdev/p/10833186.html](https://www.cnblogs.com/sparkdev/p/10833186.html)

- [黑魔法：pflag.lookup](https://juejin.im/entry/5a5c4ab7518825734a74a9af)

