#  Go 通过 import 导入包

![](https://i-blog.csdnimg.cn/blog_migrate/49deb5b59523366d2dcc04dfbabbf095.png)




---
## 1 单行导入

```go
import "fmt"
import "sync" 
```

## 2 多行导入

```go
import(
    "fmt"
    "sync"
)
```
## 3 使用别名
我们导入了两个具有同一包名的包时产生冲突，此时这里为其中一个包定义别名

```go
import (
    "crypto/rand"
    mrand "math/rand" // 将名称替换为mrand避免冲突
)
```

我们导入了一个名字很长的包，为了避免后面都写这么长串的包名，可以这样定义别名

```go
import hw "helloworldtestmodule"
```

防止导入的包名和本地的变量发生冲突，比如 path 这个很常用的变量名和导入的标准包冲突。

```go
import pathpkg "path"
```
## 4 使用点操作
如里在我们程序内部里频繁使用了一个工具包，比如 fmt，那每次使用它的打印函数打印时，都要 包名+方法名。

对于这种使用高频的包，可以在导入的时，就把它定义会 "自己人"（方法是使用一个 . ），自己人的话，不分彼此，它的方法，就是我们的方法。

从此，我们打印再也不用加 fmt 了。

```go
import . "fmt"

func main() {
    Println("hello, world")
}
```

但这种用法，会有一定的隐患，就是导入的包里可能有函数，会和我们自己的函数发生冲突。
##  5. 包的初始化
每个包都允许有一个或多个的 init 函数，当这个包被导入时，会执行该包的这个 init 函数，做一些初始化任务。

对于 init 函数的执行有两点需要注意

init 函数优先于 main 函数执行

在一个包引用链中，包的初始化是深度优先的。比如，有这样一个包引用关系：main→A→B→C，那么初始化顺序为

```go
C.init→B.init→A.init→main
```
## 6. 包的匿名导入
当我们导入一个包时，如果这个包没有被使用到，在编译时，是会报错的。

但是有些情况下，我们导入一个包，只想执行包里的 init 函数，来运行一些初始化任务，此时怎么办呢？

可以使用匿名导入，用法如下，其中下划线为空白标识符，并不能被访问

// 注册一个PNG decoder

```go
import _ "image/png"
```

由于导入时，会执行 init 函数，所以编译时，仍然会将这个包编译到可执行文件中。
## 7. 导入的是路径还是包？
当我们使用 import 导入 testmodule/foo 时，初学者，经常会问，这个 foo 到底是一个包呢，还是只是包所在目录名？

```go
import "testmodule/foo"
```

为了得出这个结论，专门做了个试验（请看「第七点里的代码示例」），最后得出的结论是：

 1. 导入时，是按照目录导入。导入目录后，可以使用这个目录下的所有包。
 2. 出于习惯，包名和目录名通常会设置成一样，所以会让你有一种你导入的是包的错觉
## 8. 相对导入和绝对导入
据我了解在 Go 1.10 之前，好像是不支持相对导入的，在 Go 1.10 之后才可以。

绝对导入：从 $GOPATH/src 或 $GOROOT 或者 $GOPATH/pkg/mod 目录下搜索包并导入

相对导入：从当前目录中搜索包并开始导入。就像下面这样

```handlebars
import (
    "./module1"
    "../module2"
    "../../module3"
    "../module4/module5"
)
```
## 9. 包导入路径优先级
前面一节，介绍了三种不同的包依赖管理方案，不同的管理模式，存放包的路径可能都不一样，有的可以将包放在 GOPATH 下，有的可以将包放在 vendor 下，还有些包是内置包放在 GOROOT 下。

那么问题就来了，如果在这三个不同的路径下，有一个相同包名但是版本不同的包，我们导入的时候，是选择哪个进行导入呢？

这就需要我们搞懂，在 Golang 中包搜索路径优先级是怎样的？

这时候就需要区分，是使用哪种模式进行包的管理的。

如果使用 govendor

当我们导入一个包时，它会：

 1. 先从项目根目录的 vendor 目录中查找
 2. 然后从 $GOROOT/src 目录下查找
 3. 最后从 $GOPATH/src 目录下查找
 4. 都找不到的话，就报错。

为了验证这个过程，我在创建中创建一个 vendor 目录后，就开启了 vendor 模式了，我在 main.go 中随便导入一个包 pkg，由于这个包是我随便指定的，当然会找不到，找不到就会报错， Golang 会在报错信息中打印中搜索的过程，从这个信息中，就可以看到 Golang 的包查找优先级了。

![](https://i-blog.csdnimg.cn/blog_migrate/94c21999fcf8dbf61f2099af51790e57.png)
如果使用 go modules

你导入的包如果有域名，都会先在 $GOPATH/pkg/mod 下查找，找不到就连网去该网站上寻找，找不到或者找到的不是一个包，则报错。

而如果你导入的包没有域名（比如 "fmt"这种），就只会到 $GOROOT 里查找。

还有一点很重要，当你的项目下有 vendor 目录时，不管你的包有没有域名，都只会在 vendor 目录中想找。

![](https://i-blog.csdnimg.cn/blog_migrate/61f55444d66d672d7a69e61d4f62ff61.png)
通常vendor 目录是通过 go mod vendor 命令生成的，这个命令会将项目依赖全部打包到你的项目目录下的 verdor 文件夹中。

参考：
- [2020重学Go系列：25. Go 语言中关于包导入必学的 8 个知识点](https://mp.weixin.qq.com/s?__biz=MzAxMTA4Njc0OQ==&mid=2651439020&idx=5&sn=0cf5759092109e18fd770af012a1e604&chksm=80bb615eb7cce848ea78af8a5896ab210dbd9ed603697b16b6130a366e1925366c59013a9fb7&mpshare=1&scene=1&srcid=&sharer_sharetime=1585803160357&sharer_shareid=9e1d0f93025303e47ff2523f5ebf4078&key=8b8c3fc880ec2ebf588c733c83f7b0cc4a559647d374b6f19072b20357262ce1ef9f0bf093018a8bea2cf697060e440e08d0fe2bd8fee43e08ea1ff8d0dbace4755758d076c34f2be71e043bd9468e2c&ascene=1&uin=MjkwMDAzNTYzOQ==&devicetype=Windows%2010&version=62080079&lang=zh_CN&exportkey=AZcQuqT5bAsoqbCxMncSYL4=&pass_ticket=8WtlBAvkJRH4wtfCPwJB96c6g8pkVGEaxWiJxfIduP41vXzHHFa1sp9u%2bsRUSgx6)
