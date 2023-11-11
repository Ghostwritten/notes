
## 1. 命令行语法
命令行语法主要有以下几种形式：

```bash
cmd -flag       // 只支持bool类型
cmd -flag=xxx
cmd -flag xxx   // 只支持非bool类型
```

以上语法对于一个或两个‘－’号是一样的，即

```bash
cmd -flag xxx （使用空格，一个 - 符号）
cmd --flag xxx （使用空格，两个 - 符号）
cmd -flag=xxx （使用等号，一个 - 符号）
cmd --flag=xxx （使用等号，两个 - 符号） 
```

对于整形 flag，合法的值可以为 1234，0664，0x1234 或 负数 等。对于布尔型 flag，可以为 `1，0，t，f，T，F，true，false，TRUE，FALSE，True，False` 等

其中，布尔类型的参数比较特殊，为了防止解析时的二义性，应该使用 等号 的方式指定
## 2. 命令行参数方法

### 2.1 flag.String(), Bool(), Int() 等flag.Xxx()方法
该种方式返回一个相**应的指针**

```go
import "flag"
var ip = flag.Int("flagname", 1234, "help message for flagname")
```

### 2.2 flag.XxxVar()方法将flag绑定到一个变量
该种方式**返回值类型**，如

```go
var flagvar int
func init() {
    flag.IntVar(&flagvar, "flagname", 1234, "help message for flagname")
}
```

### 2.3 flag.Var()绑定自定义类型
自定义类型需要实现Value接口(Receives必须为指针)，如

```go
flag.Var(&flagVal, "name", "help message for flagname")
```

对于这种类型的flag，默认值为该变量类型的初始值

### 2.4 flag.Parse()解析命令行参数到定义的flag

```go
flag.Parse()
```

解析函数将会在碰到第一个非flag命令行参数时停止，非flag命令行参数是指不满足命令行语法的参数，如命令行参数为`cmd --flag=true abc`则第一个非flag命令行参数为“abc”

调用Parse解析后，就可以直接使用flag本身(指针类型)或者绑定的变量了(值类型)

```go
fmt.Println("ip has value ", *ip)
fmt.Println("flagvar has value ", flagvar)
```

还可通过flag.Args(), flag.Arg(i)来获取非flag命令行参数


## 3 练习
### 3.1  获取“species” flag的值，默认为“gopher”

```go
var species = flag.String("species", "gopher", "the species we are studying")
```

```go
package main
 
import (
  "flag"
  "fmt"
)
  
func main() {
  username := flag.String("name", "world", "Input your username")
  flag.Parse()
  fmt.Println("Hello, ", *username)
}
```

```go
root@localhost flag]# ./hello 
Hello,  world
[root@localhost flag]# ./hello -name=nihao
Hello,  nihao
[root@localhost flag]# ./hello --name=nihao
Hello,  nihao
[root@localhost flag]# ./hello name=nihao  #注意返回值
Hello,  world
[root@localhost flag]# ./hello --name nihao
Hello,  nihao
[root@localhost flag]# ./hello -name nihao
Hello,  nihao
[root@localhost flag]# ./hello name nihao #注意返回值
Hello,  world

```

### 3.2 两个flag共享同一个变量，一般用于同时实现完整flag参数和对应简化版flag参数，需要注意初始化顺序和默认值

```go
var gopherType string

func init() {
  const (
    defaultGopher = "pocket"
    usage         = "the variety of gopher"
  )
  flag.StringVar(&gopherType, "gopher_type", defaultGopher, usage)
  flag.StringVar(&gopherType, "g", defaultGopher, usage+"(shorthand)")
}
```

### 3.3 将flag绑定用户自定义类型。按我们先前所说，只需要实现Value接口，但实际上，如果需要取值的话，需要实现Getter接口，看下接口定义就明白了：

```go
type Getter interface {
  Value
  Get(string) interface{}
}
type Value interface {
  String() string
  Set(string) error
}
```
## 4. 实战
实现一个解析并格式化命令行输入的时间集合的例子

```go
package main

import (
  "errors"
  "flag"
  "fmt"
  "strings"
  "time"
)

type interval []time.Duration

//实现String接口
func (i *interval) String() string {
  return fmt.Sprintf("%v", *i)
}

//实现Set接口,Set接口决定了如何解析flag的值
func (i *interval) Set(value string) error {
    //此处决定命令行是否可以设置多次-deltaT
  if len(*i) > 0 {
    return errors.New("interval flag already set")
  }
  for _, dt := range strings.Split(value, ",") {
    duration, err := time.ParseDuration(dt)
    if err != nil {
      return err
    }
    *i = append(*i, duration)
  }
  return nil
}

var intervalFlag interval

func init() {
  flag.Var(&intervalFlag, "deltaT", "comma-separated list of intervals to use between events")
}

func main() {
  flag.Parse()
  fmt.Println(intervalFlag)
}
```

```go
[root@localhost flag]# go build commandLine.go 
[root@localhost flag]# ./commandLine -deltaT 61m,72h,80s
[1h1m0s 72h0m0s 1m20s]
```
参考连接：

 - [https://studygolang.com/articles/3365](https://studygolang.com/articles/3365)
 - [Gopkg:flag](https://github.com/astaxie/gopkg/tree/master/flag)

