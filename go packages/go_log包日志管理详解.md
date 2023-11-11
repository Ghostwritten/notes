

----
## 1. Log三类方法

 - Print()   用于输出日志
 - Fatal()   输出日志的同时，调用os.Exit(1)方法退出，小提示：如果函数下存在`defer`不会执行
 - Panic()   输出日志的同时，调用panic方法，但`defer`会执行
 
 三类区别：
 **Fatal会保存日志并终止程序,Panic会保存日志并丢出异常终止程序，Print会保存日志但是程序继续**
 三类源代码
 

```go
func Println(v ...interface{}) {
	std.Output(2, fmt.Sprintln(v...))
}

func Fatalln(v ...interface{}) {
	std.Output(2, fmt.Sprintln(v...))
	os.Exit(1)
}

func Panicln(v ...interface{}) {
	s := fmt.Sprintln(v...)
	std.Output(2, s)
	panic(s)
}
```

跟fmt.Print不同的地方在于,fmt.Print,fmt.Fatal,fmt.Pacni属于Stdout输出，log.Print属于Stderr输出
### 1.1 Print()输出日志

```go
package main

import (
	"log"
)

func sayPrint() {
	log.Print("sayPrint")  //Stderr
}

func main() {
	sayPrint()
}
```

```go
[root@localhost log]# go run logprint.go
2020/04/01 21:51:01 sayPrint
```
### 1.2 Fatal()输出日志并调用os.Exit(1)方法退出
```go
package main

import (
	"log"
	"fmt"
)

func sayPrint() {
	log.Print("sayPrint")  //Stderr
}

func sayFalal() {
        defer fmt.Println("end")
	log.Fatal("sayFalal")
}


func main() {
	sayPrint()
        sayFalal()
}
```

```go
[root@localhost log]# go run logfatal.go
2020/04/01 21:54:49 sayPrint
2020/04/01 21:54:49 sayFalal
exit status 1
```
如果没有`defer`则会打印`end`
### 1.3 Panic() 输出日志并调用panic方法

```go
package main

import (
	"log"
	"fmt"
)

func sayPrint() {
	log.Print("sayPrint")  //Stderr
}

func sayFalal() {
        defer fmt.Println("end")
	log.Fatal("sayFalal")
}

func sayPanic() {
	defer fmt.Println("end")
	log.Panic("sayOne")
}


func main() {
	sayPrint()
    sayPanic()
    sayFalal()
}
```

```go
[root@localhost log]# go run logpanic.go
2020/04/01 22:00:03 sayPrint
2020/04/01 22:00:03 sayOne
end
panic: sayOne

goroutine 1 [running]:
log.Panic(0xc000043f08, 0x1, 0x1)
	/usr/local/go/src/log/log.go:338 +0xac
main.sayPanic()
	/root/go/log/logpanic.go:19 +0xe9
main.main()
	/root/go/log/logpanic.go:25 +0x62
exit status 2
```

有输出日志内容、时间、以及报错，还不满足我们得日常需求。
给日志添加新格式
## 2 三个函数
### 2.1 SetFlags()函数
//设置flag格式

```go
func SetFlags(flag int)
```
flag格式

```bash
const (
	Ldate         = 1 << iota     //日期示例： 2009/01/23
	Ltime                         //时间示例: 01:23:23
	Lmicroseconds                 //毫秒示例: 01:23:23.123123.
	Llongfile                     //绝对路径和行号: /a/b/c/d.go:23
	Lshortfile                    //文件和行号: d.go:23.
	LUTC                          //日期时间转为0时区的
	LstdFlags     = Ldate | Ltime //Go提供的标准抬头信息
)
```
#### 添加输出文件名与代码行

```go
package main

import (
	"log"
)

func Print() {
	log.Println("ghostwritten的博客:","https://blog.csdn.net/xixihahalelehehe")
	log.Printf("ghostwritten的web：%s\n","http://www.smoothies.com.cn/")
}

func init(){
	log.SetFlags(log.Ldate|log.Lshortfile)
}

func main() {
	Print()
}
```

```go
[root@localhost log]#  go run logset.go
2020/04/01 logset.go:8: ghostwritten的博客: https://blog.csdn.net/xixihahalelehehe
2020/04/01 logset.go:9: ghostwritten的web：http://www.smoothies.com.cn/
```
#### 输出UTC时间

```go
package main

import (
	"log"
)

func Print() {
	log.Println("ghostwritten的博客:","https://blog.csdn.net/xixihahalelehehe")
	log.Printf("ghostwritten的web：%s\n","http://www.smoothies.com.cn/")
}

func init(){
	log.SetFlags(log.Ldate|log.Ltime|log.LUTC|log.Lshortfile)
}

func main() {
	Print()
}
```

```go
[root@localhost log]# go run logset2.go
2020/04/01 14:17:40 logset2.go:8: ghostwritten的博客: https://blog.csdn.net/xixihahalelehehe
2020/04/01 14:17:40 logset2.go:9: ghostwritten的web：http://www.smoothies.com.cn/
```
### 2.2 SetPrefix()配置log属性的输出
#### 对日志区分业务模块

```go
package main
import (
	"log"
)

func Print() {
	log.Println("ghostwritten的博客:","https://blog.csdn.net/xixihahalelehehe")
	log.Printf("ghostwritten的web：%s\n","http://www.smoothies.com.cn/")
}

func init(){
        log.SetPrefix("【UserCenter】")
        log.SetFlags(log.LstdFlags | log.Lshortfile |log.LUTC)
}


func main() {
	Print()
}
```
```go
[root@localhost log]# go run logprefix.go
【UserCenter】2020/04/01 14:22:44 logprefix.go:8: ghostwritten的博客: https://blog.csdn.net/xixihahalelehehe
【UserCenter】2020/04/01 14:22:44 logprefix.go:9: ghostwritten的web：http://www.smoothies.com.cn/
```

### 2.3 log.New()创建日志对象
#### 2.3.1 实现原理
日志包log的这些函数都是类似的，关键的输出日志就在于std.Output方法。

```go
func New(out io.Writer, prefix string, flag int) *Logger {
	return &Logger{out: out, prefix: prefix, flag: flag}
}

var std = New(os.Stderr, "", LstdFlags)
```
从以上源代码可以看出，变量std其实是一个`*Logger`，通过`log.New`函数创建，默认输出到`os.Stderr`设备，前缀为空，日志抬头信息为标准抬头`LstdFlags`。

os.Stderr对应的是UNIX里的标准错误警告信息的输出设备，同时被作为默认的日志输出目的地。初次之外，还有标准输出设备os.Stdout以及标准输入设备os.Stdin。

```go
var (
	Stdin  = NewFile(uintptr(syscall.Stdin), "/dev/stdin")
	Stdout = NewFile(uintptr(syscall.Stdout), "/dev/stdout")
	Stderr = NewFile(uintptr(syscall.Stderr), "/dev/stderr")
)
```
以上就是定义的UNIX的标准的三种设备，分别用于输入、输出和警告错误信息。理解了os.Stderr，现在我们看下Logger这个结构体，日志的信息和操作，都是通过这个Logger操作的。

```go
type Logger struct {
	mu     sync.Mutex // ensures atomic writes; protects the following fields
	prefix string     // prefix to write at beginning of each line
	flag   int        // properties
	out    io.Writer  // destination for output
	buf    []byte     // for accumulating text to write
}
```

 1. 字段mu是一个互斥锁，主要是是保证这个日志记录器Logger在多goroutine下也是安全的。
 2. 字段prefix是每一行日志的前缀
 3. 字段flag是日志抬头信息
 4. 字段out是日志输出的目的地，默认情况下是os.Stderr。
 5. 字段buf是一次日志输出文本缓冲，最终会被写到out里。

#### 2.3.2 练习
1一个info类型自定义格式的日志文件
```go
package main

import (
	"log"
	"os"
)

func newLog() {
	//创建日志文件
	fileName := "app.log"
	logFile, err := os.Create(fileName)
	if err != nil {
		panic(err)
	}
	defer logFile.Close()

	//自定义输出日志
	//Ldate Ltime
	log := log.New(logFile, "[Info]", log.Ldate|log.Lmicroseconds|log.Llongfile)
	log.Println(" Hello World newLog")
}

func main() {
	newLog()
}
```

```go
[root@localhost log]#  go run lognew1.go
[root@localhost log]# cat app.log 
[Info]2020/04/01 22:52:36.143571 /root/go/log/lognew1.go:20:  Hello World newLog
```
2.一个输出info或debug级别的自定义格式的日志文件

```go
package main
import (
    "log"
    "os"
)
func main(){
    // 定义一个文件
    fileName := "ll.log"
    logFile,err  := os.Create(fileName)
    defer logFile.Close()
    if err != nil {
        log.Fatalln("open file error !")
    }
    // 创建一个日志对象
    debugLog := log.New(logFile,"[Debug]",log.LstdFlags)
    debugLog.Println("A debug message here")
    //配置一个日志格式的前缀
    debugLog.SetPrefix("[Info]")
    debugLog.Println("A Info Message here ")
    //配置log的Flag参数
    debugLog.SetFlags(debugLog.Flags()|log.Lshortfile)
    debugLog.Println("A different prefix")
}
```

[root@localhost log]# go run lognew2.go

```go
[root@localhost log]# cat ll.log 
[Debug]2020/04/01 23:00:47 A debug message here
[Info]2020/04/01 23:00:47 A Info Message here 
[Info]2020/04/01 23:00:47 lognew2.go:22: A different prefix
```
3.一个终端输出info、debug、err类型级别并自定义格式的日志并err级别写入文件

```go
package main

import (
	"log"
	"os"
        "io"
)

var (
	Info *log.Logger
	Warning *log.Logger
	Error * log.Logger
)

func init(){
	errFile,err:=os.OpenFile("errors.log",os.O_CREATE|os.O_WRONLY|os.O_APPEND,0666)
	if err!=nil{
		log.Fatalln("打开日志文件失败：",err)
	}

	Info = log.New(os.Stdout,"Info:",log.Ldate | log.Ltime | log.Lshortfile)
	Warning = log.New(os.Stdout,"Warning:",log.Ldate | log.Ltime | log.Lshortfile)
	Error = log.New(io.MultiWriter(os.Stderr,errFile),"Error:",log.Ldate | log.Ltime | log.Lshortfile)

}

func main() {
	Info.Println("ghostwritten的博客:","https://blog.csdn.net/xixihahalelehehe")
	Warning.Printf("ghostwritten的网站：%s\n","http://www.smoothies.com.cn/")
	Error.Println("欢迎关注留言")
}
```

```go
[root@localhost log]# go run lognew3.go
Info:2020/04/01 23:12:46 lognew3.go:28: ghostwritten的博客: https://blog.csdn.net/xixihahalelehehe
Warning:2020/04/01 23:12:46 lognew3.go:29: ghostwritten的网站：http://www.smoothies.com.cn/
Error:2020/04/01 23:12:46 lognew3.go:30: 欢迎关注留言
[root@localhost log]# cat errors.log 
Error:2020/04/01 23:08:11 lognew3.go:30: 欢迎关注留言
```

