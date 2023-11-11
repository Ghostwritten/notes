
## 1. 介绍
ServrMux 本质上是一个 HTTP 请求路由器（或者叫多路复用器，Multiplexor）。它把收到的请求与一组预先定义的 URL 路径列表做对比，然后在匹配到路径的时候调用关联的处理器（Handler）。

处理器（Handler）负责输出HTTP响应的头和正文。任何满足了http.Handler接口的对象都可作为一个处理器。通俗的说，对象只要有个如下签名的`ServeHTTP`方法即可：

```bash
 ServeHTTP(http.ResponseWriter, *http.Request)
```
    
Go 语言的 HTTP 包自带了几个函数用作常用处理器，比如`FileServer`，`NotFoundHandler` 和 `RedirectHandler`。我们从一个简单具体的例子开始：

```bash
$ mkdir handler-example
$ cd handler-example
$ touch main.go
```

```bash
//File: main.go
package main

import (
  "log"
  "net/http"
)

func main() {
  mux := http.NewServeMux()

  rh := http.RedirectHandler("http://example.org", 307)
  mux.Handle("/foo", rh)

  log.Println("Listening...")
  http.ListenAndServe(":3000", mux)
}
```
快速地过一下代码：    

 - 在 main 函数中我们只用了 `http.NewServeMux` 函数来创建一个空的 `ServeMux`。

 - 然后我们使用 `http.RedirectHandler` 函数创建了一个新的处理器，这个处理器会对收到的所有请求，都执行307重定向操作到`http://example.org`。

 - 接下来我们使用 `ServeMux.Handle` 函数将处理器注册到新创建的 `ServeMux`，所以它在 URL 路径/foo上收到所有的请求都交给这个处理器。

 - 最后我们创建了一个新的服务器，并通过 `http.ListenAndServe` 函数监听所有进入的请求，通过传递刚才创建的ServeMux来为请求去匹配对应处理器。

继续，运行一下这个程序:

```bash
$ go run main.go
Listening...
```
然后在浏览器中访问 `http://localhost:3000/foo`，你应该能发现请求已经成功的重定向了。
明察秋毫的你应该能注意到一些有意思的事情：`ListenAndServer` 的函数签名是 `ListenAndServe(addr string, handler Handler)` ，但是第二个参数我们传递的是个 `ServeMux`。
 我们之所以能这么做，是因为 ServeMux 也有 ServeHTTP 方法，因此它也是个合法的 Handler。 对我来说，将 ServerMux 用作一个特殊的Handler是一种简化。它不是自己输出响应而是将请求传递给注册到它的其他 Handler。这乍一听起来不是什么明显的飞跃 - 但在 Go 中将 Handler 链在一起是非常普遍的用法。
## 2. 自定义处理器（Custom Handlers）-方法

让我们创建一个自定义的处理器，功能是将以特定格式输出当前的本地时间：

```bash
type timeHandler struct {
  format string
}

func (th *timeHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
  tm := time.Now().Format(th.format)
  w.Write([]byte("The time is: " + tm))
}
```
这个例子里代码本身并不是重点。

真正的重点是我们有一个对象（本例中就是个timerHandler结构体，但是也可以是一个字符串、一个函数或者任意的东西），我们在这个对象上实现了一个 `ServeHTTP(http.ResponseWriter, *http.Request)` 签名的方法，这就是我们创建一个处理器所需的全部东西。

我们把这个集成到具体的示例里：

```bash
//File: main.go

package main

import (
  "log"
  "net/http"
  "time"
)

type timeHandler struct {
  format string
}

func (th *timeHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
  tm := time.Now().Format(th.format)
  w.Write([]byte("The time is: " + tm))
}

func main() {
  mux := http.NewServeMux()

  th := &timeHandler{format: time.RFC1123}
  mux.Handle("/time", th)

  log.Println("Listening...")
  http.ListenAndServe(":3000", mux)
}
```
main函数中，我们像初始化一个常规的结构体一样，初始化了`timeHandler`，用 `&` 符号获得了其地址。随后，像之前的例子一样，我们使用 `mux.Handle` 函数来将其注册到 ServerMux。现在当我们运行这个应用，ServerMux 将会将任何对 /time的请求直接交给 `timeHandler.ServeHTTP` 方法处理。

访问一下这个地址看一下效果：`http://localhost:3000/time` 。

注意我们可以在多个路由中轻松的复用 timeHandler：

```bash
func main() {
  mux := http.NewServeMux()

  th1123 := &timeHandler{format: time.RFC1123}
  mux.Handle("/time/rfc1123", th1123)

  th3339 := &timeHandler{format: time.RFC3339}
  mux.Handle("/time/rfc3339", th3339)

  log.Println("Listening...")
  http.ListenAndServe(":3000", mux)
}
```
## 3. 将函数作为处理器
对于简单的情况（比如上面的例子），定义个新的有 ServerHTTP 方法的自定义类型有些累赘。我们看一下另外一种方式，我们借助 `http.HandlerFunc` 类型来让一个常规函数满足作为一个 Handler 接口的条件。

任何有 `func(http.ResponseWriter, *http.Request)` 签名的函数都能转化为一个 HandlerFunc 类型。这很有用，因为 HandlerFunc 对象内置了 ServeHTTP 方法，后者可以聪明又方便的调用我们最初提供的函数内容。

如果你听起来还有些困惑，可以尝试看一下[相关的源代码]`golang.org/src/pkg/net…`。你将会看到让一个函数对象满足 Handler 接口是非常简洁优雅的。
 
 让我们使用这个技术重新实现一遍`timeHandler`应用：

```bash
 //File: main.go
package main

import (
  "log"
  "net/http"
  "time"
)

func timeHandler(w http.ResponseWriter, r *http.Request) {
  tm := time.Now().Format(time.RFC1123)
  w.Write([]byte("The time is: " + tm))
}

func main() {
  mux := http.NewServeMux()

  // Convert the timeHandler function to a HandlerFunc type
  th := http.HandlerFunc(timeHandler)
  // And add it to the ServeMux
  mux.Handle("/time", th)

  log.Println("Listening...")
  http.ListenAndServe(":3000", mux)
}
```
实际上，将一个函数转换成 HandlerFunc 后注册到 ServeMux 是很普遍的用法，所以 Go 语言为此提供了个便捷方式：`ServerMux.HandlerFunc` 方法。

我们使用便捷方式重写 main() 函数看起来是这样的：

```bash
func main() {
  mux := http.NewServeMux()

  mux.HandleFunc("/time", timeHandler)

  log.Println("Listening...")
  http.ListenAndServe(":3000", mux)
}
```
绝大多数情况下这种用函数当处理器的方式工作的很好。但是当事情开始变得更复杂的时候，就会有些产生一些限制了。 你可能已经注意到了，跟之前的方式不同，我们不得不将时间格式硬编码到 timeHandler 的方法中。如果我们想从 main() 函数中传递一些信息或者变量给处理器该怎么办？  一个优雅的方式是将我们处理器放到一个闭包中，将我们要使用的变量带进去：

```bash
//File: main.go
package main

import (
  "log"
  "net/http"
  "time"
)

func timeHandler(format string) http.Handler {
  fn := func(w http.ResponseWriter, r *http.Request) {
    tm := time.Now().Format(format)
    w.Write([]byte("The time is: " + tm))
  }
  return http.HandlerFunc(fn)
}

func main() {
  mux := http.NewServeMux()

  th := timeHandler(time.RFC1123)
  mux.Handle("/time", th)

  log.Println("Listening...")
  http.ListenAndServe(":3000", mux)
}
```
timeHandler 函数现在有了个更巧妙的身份。除了把一个函数封装成 Handler(像我们之前做到那样)，我们现在使用它来返回一个处理器。这种机制有两个关键点：
 
首先是创建了一个fn，这是个匿名函数，将 format 变量封装到一个闭包里。闭包的本质让处理器在任何情况下，都可以在本地范围内访问到 format 变量。
    
 其次我们的闭包函数满足 `func(http.ResponseWriter, *http.Request)` 签名。如果你记得之前我们说的，这意味我们可以将它转换成一个HandlerFunc类型（满足了http.Handler接口）。我们的timeHandler 函数随后转换后的 HandlerFunc 返回。
 
 在上面的例子中我们已经可以传递一个简单的字符串给处理器。但是在实际的应用中可以使用这种方法传递数据库连接、模板组，或者其他应用级的上下文。使用全局变量也是个不错的选择，还能得到额外的好处就是编写更优雅的自包含的处理器以便测试。
 
你也可能见过相同的写法，像这样：

```bash
func timeHandler(format string) http.Handler {
  return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    tm := time.Now().Format(format)
    w.Write([]byte("The time is: " + tm))
  })
}
```

或者在返回时，使用一个到 `HandlerFunc` 类型的隐式转换：

```bash
func timeHandler(format string) http.HandlerFunc {
  return func(w http.ResponseWriter, r *http.Request) {
    tm := time.Now().Format(format)
    w.Write([]byte("The time is: " + tm))
  }
}
```
## 4.  更便利的 DefaultServeMux
你可能已经在很多地方看到过 `DefaultServeMux`, 从最简单的 Hello World 例子，到 go 语言的源代码中。我花了很长时间才意识到 DefaultServerMux 并没有什么的特殊的地方。`DefaultServerMux` 就是我们之前用到的 ServerMux，只是它随着 net/httpp 包初始化的时候被自动初始化了而已。Go 源代码中的相关行如下：

```bash
var DefaultServeMux = NewServeMux()
```
net/http 包提供了一组快捷方式来配合 `DefaultServeMux：http.Handle` 和 `http.HandleFunc`。这些函数与我们之前看过的类似的名称的函数功能一样，唯一的不同是他们将处理器注册到`DefaultServerMux` ，而之前我们是注册到自己创建的 `ServeMux`。

此外，`ListenAndServe`在没有提供其他的处理器的情况下（也就是第二个参数设成了 nil），内部会使用 `DefaultServeMux`。
    因此，作为最后一个步骤，我们使用 DefaultServeMux 来改写我们的 timeHandler应用：
```bash
//File: main.go
package main

import (
  "log"
  "net/http"
  "time"
)

func timeHandler(format string) http.Handler {
  fn := func(w http.ResponseWriter, r *http.Request) {
    tm := time.Now().Format(format)
    w.Write([]byte("The time is: " + tm))
  }
  return http.HandlerFunc(fn)
}

func main() {
  // Note that we skip creating the ServeMux...

  var format string = time.RFC1123
  th := timeHandler(format)

  // We use http.Handle instead of mux.Handle...
  http.Handle("/time", th)

  log.Println("Listening...")
  // And pass nil as the handler to ListenAndServe.
  http.ListenAndServe(":3000", nil)
}
```
参考链接：

 - [https://golang.org/src/net/http/server.go?s=35455:35502#L1221()](https://golang.org/src/net/http/server.go?s=35455:35502#L1221%28%29)

相关阅读：
 - [golang net/http包之设置 http response 响应头详解](https://ghostwritten.blog.csdn.net/article/details/109829882)
 - [go web net/http【1】使用详解](https://ghostwritten.blog.csdn.net/article/details/104853033)
 - [go net/http【2】ServeMux 和 Handler使用详解](https://ghostwritten.blog.csdn.net/article/details/112463115)
