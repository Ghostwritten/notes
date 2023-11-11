

##  错误包需要具有哪些功能？
###  1. 应该能支持错误堆栈
我们来看下面一段代码，假设保存在[bad.go](https://github.com/marmotedu/gopractise-demo/blob/master/errors/bad.go)文件中：

```bash
package main

import (
  "fmt"
  "log"
)

func main() {
  if err := funcA(); err != nil {
    log.Fatalf("call func got failed: %v", err)
    return
  }

  log.Println("call func success")
}

func funcA() error {
  if err := funcB(); err != nil {
    return err
  }

  return fmt.Errorf("func called error")
}

func funcB() error {
  return fmt.Errorf("func called error")
}
```
执行上面的代码：

```bash
$ go run bad.go
2021/07/02 08:06:55 call func got failed: func called error
exit status 1
```
这时我们想定位问题，但不知道具体是哪行代码报的错误，只能靠猜，还不一定能猜到。为了解决这个问题，我们可以加一些 Debug 信息，来协助我们定位问题。这样做在测试环境是没问题的，但是在线上环境，一方面修改、发布都比较麻烦，另一方面问题可能比较难重现。这时候我们会想，要是能打印错误的堆栈就好了。例如：

```bash

2021/07/02 14:17:03 call func got failed: func called error
main.funcB
  /home/colin/workspace/golang/src/github.com/marmotedu/gopractise-demo/errors/good.go:27
main.funcA
  /home/colin/workspace/golang/src/github.com/marmotedu/gopractise-demo/errors/good.go:19
main.main
  /home/colin/workspace/golang/src/github.com/marmotedu/gopractise-demo/errors/good.go:10
runtime.main
  /home/colin/go/go1.16.2/src/runtime/proc.go:225
runtime.goexit
  /home/colin/go/go1.16.2/src/runtime/asm_amd64.s:1371
exit status 1
```
通过上面的错误输出，我们可以很容易地知道是哪行代码报的错，从而极大提高问题定位的效率，降低定位的难度。所以，在我看来，一个优秀的 errors 包，首先需要支持错误堆栈。

### 2. 能够支持不同的打印格式
例如%+v、%v、%s等格式，可以根据需要打印不同丰富度的错误信息。

###  3. 能支持 Wrap/Unwrap 功能，也就是在已有的错误上，追加一些新的信息
例如`errors.Wrap(err, "open file failed")` 。Wrap 通常用在调用函数中，调用函数可以基于被调函数报错时的错误 Wrap 一些自己的信息，丰富报错信息，方便后期的错误定位，例如：

```bash
func funcA() error {
    if err := funcB(); err != nil {
        return errors.Wrap(err, "call funcB failed")
    }

    return errors.New("func called error")
}

func funcB() error {
    return errors.New("func called error")
}
```
这里要注意，不同的错误类型，Wrap 函数的逻辑也可以不同。另外，在调用 Wrap 时，也会生成一个错误堆栈节点。我们既然能够嵌套 error，那有时候还可能需要获取被嵌套的 error，这时就需要错误包提供`Unwrap`函数。

### 4. 错误包应该有Is方法

在实际开发中，我们经常需要判断某个 error 是否是指定的 error。在 Go 1.13 之前，也就是没有 `wrapping error` 的时候，我们要判断 error 是不是同一个，可以使用如下方法：

```bash
if err == os.ErrNotExist {
  // normal code
}
```
### 5. 错误包应该支持  As  函数
在 `Go 1.13` 之前，没有 `wrapping error` 的时候，我们要把 `error` 转为另外一个 `error`，一般都是使用 `type assertion` 或者 `type switch`，也就是类型断言。例如：


```bash
if perr, ok := err.(*os.PathError); ok {
  fmt.Println(perr.Path)
}
```
但是现在，返回的 err 可能是嵌套的 error，甚至好几层嵌套，这种方式就不能用了。所以，我们可以通过实现 As 函数来完成这种功能。现在我们把上面的例子，用 As 函数实现一下：

```bash
var perr *os.PathError
if errors.As(err, &perr) {
  fmt.Println(perr.Path)
}
```
这样就可以完全实现类型断言的功能，而且还更强大，因为它可以处理 `wrapping error`。
最后，**能够支持两种错误创建方式：非格式化创建和格式化创建**。例如：

```bash
errors.New("file not found")
errors.Errorf("file %s not found", "iam-apiserver")
```
上面，我们介绍了一个优秀的错误包应该具备的功能。一个好消息是，Github 上有不少实现了这些功能的错误包，其中`github.com/pkg/errors`包最受欢迎。所以，我基于`github.com/pkg/errors`包进行了二次封装。


### 1.1 常见调用方式
模板

```go
n, err := Foo(0)  
 
if err != nil { 
    //  错误处理  
} else { 
    //  使用返回值 n  
} 
```

练习1

```go
package main

import (  
   "fmt"
   "os"
)

func main() {  
   f, err := os.Open("/test.txt")
   if err != nil {
       fmt.Println(err)
       return
   }
   fmt.Println(f.Name(), "opened successfully")
}
```

```go
[root@localhost error]# go run err3.go
open /test.txt: no such file or directory
```

### 1.2 自定义error方法
#### 1.2.1 函数调用error
```go
func Foo(param int)(n int, err error) { 
    // ...  
} 
```

### 1.2.2 自定义Error模板1
```go
type fileError struct {
}

func (fe *fileError) Error() string {
    return "文件错误"
}
```
练习1
 模拟一个错误

```go
package main

import "fmt"

type fileError struct {
}

func (fe *fileError) Error() string {   //自定义会覆盖原来的Error接口
    return "文件错误"
}

//只是模拟一个错误
func openFile() ([]byte, error) {
    return nil, &fileError{}
}

func main() {
    conent, err := openFile()
    if err != nil {
        fmt.Println(err)
    } else {
        fmt.Println(string(conent))
    }
}
```

```go
[root@localhost error]# go run err1.go
文件错误
```

### 1.2 自定义Error模板2
自定义中添加一个字符串

```go
type fileError struct {
    s string
}

func (fe *fileError) Error() string {
    return fe.s
}
```
练习2
声明fileError的时候，设置好要提示的错误文字

```go
package main

import "fmt"

type fileError struct {
  s string
}

func (fe *fileError) Error() string {
    return fe.s
}

//只是模拟一个错误
func openFile() ([]byte, error) {
    return nil, &fileError{"文件错误，自定义"} 
}

func main() {
    conent, err := openFile()
    if err != nil {
        fmt.Println(err)
    } else {
        fmt.Println(string(conent))
    }
}
```

```go
[root@localhost error]# go run err2.go
文件错误，自定义
```
练习3
添加一个时间刻度

```go
package main

import (
    "fmt"
    "time"
)

type MyError struct {
    When time.Time
    What string
}

func (e MyError) Error() string {
    return fmt.Sprintf("%v: %v", e.When, e.What)
}

func oops() error {
    return MyError{
        time.Date(1989, 3, 15, 22, 30, 0, 0, time.UTC),
        "the file system has gone away",
    }
}

func main() { 
    if err := oops(); err != nil { 
       fmt.Println(err) 
    }
}
```

```go
[root@localhost error]# go run err4.go
1989-03-15 22:30:00 +0000 UTC: the file system has gone away
```
练习三
没有打开文件报错
```go
package main

import (
    "fmt"
    "os"
)

type PathError struct {
 Op   string
 Path string
 Err  error
}

func (e *PathError) Error() string { 
     return e.Op + " " + e.Path + ": " + e.Err.Error() 
} 

func main() {
 f, err := os.Open("/test.txt")
 if errObject, ok := err.(*os.PathError); ok {
   fmt.Println("错误输出：",err, "文件路径：", errObject.Path)
   return
 }
 fmt.Println(f.Name(), "opened successfully")
} 
```

```go
[root@localhost error]# go run err5.go
错误输出： open /test.txt: no such file or directory 文件路径： /test.txt
```

## 2 errors包
### 2.1 获取error包
```go
go get github.com/pkg/errors
```
### 2.2 errors.New()
errors.New()接收合适的错误信息来创建
先声明再使用
l练习1

```go
package main

import (
    "errors"
    "fmt"
)

var errNotFound error = errors.New("Not found error")

func main() {
    fmt.Printf("error: %v", errNotFound)
}
```

```go
[root@localhost error]# go run errs1.go
error: Not found error
```
练习2
函数如何调用err
直接使用

```go
package main

import (
    "errors"
    "fmt"
)

func Sqrt(f float64) (float64, error) {
        if f < 0 {
           return 0, errors.New("math - square root of negative number")
        }else {
           return 1, errors.New("math - square root of 10")
        }
}

func main() {
    if _, err := Sqrt(-1); err != nil {
       fmt.Printf("Error: %s\n", err)
    }
}
```

```go
[root@localhost error]# go run errs2.go
Error: math - square root of negative number
```
## 3 自定义error与errors.New()使用比较
比较自定义error与errors.New()函数根据需求其实各有优点。

```go
package main

import (
	"errors"
	"fmt"
)

type MsgError struct {
	Code int
	Msg  string
}
func (msg *MsgError) Error() string {
	return fmt.Sprintf("%s", msg.Msg)
}

func f1(code int) (int, error) {
	if code == 1 {
		return -1, errors.New("msg test error")
	}
	return code, nil
}


func f2(code int) (int, error) {
	if code == 1 {
		return -1, &MsgError{code, "struct msg test error"}
	}
	return code, nil
}

func main() {
	for _, v := range []int{1, 2, 3, 4, 5, 6} {
	      if code, err := f1(v); err != nil {
			fmt.Println(err)
		} else {
			fmt.Println("success:", code)
		}
	}
	for _, i := range []int{1, 2, 3} {
		if code, err := f2(i); err != nil {
			fmt.Println(err)
		} else {
			fmt.Println("success:", code)
		}
	}
}
```

```go
[root@localhost error]# go run errs3.go
msg test error
success: 2
success: 3
success: 4
success: 5
success: 6
struct msg test error
success: 2
success: 3
```

