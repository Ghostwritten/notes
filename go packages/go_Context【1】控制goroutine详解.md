![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f54c885c191af4e29274e539e6a9d070.png)

---

## 1. 背景 
对于 Golang 开发者来说context（上下文）包一定不会陌生。但很多时候，我们懒惰的只是见过它，或能起到什么作用，并不会去深究它。

应用场景：在 Go http 包的 Server 中，每一个请求在都有一个对应的goroutine去处理。请求处理函数通常会启动额外的goroutine用来访问后端服务，比如数据库和 RPC 服务。用来处理一个请求的goroutine通常需要访问一些与请求特定的数据，比如终端用户的身份认证信息、验证相关的 token、请求的截止时间。当一个请求被取消或超时时，所有用来处理该请求的goroutine都应该迅速退出，然后系统才能释放这些goroutine占用的资源。

> 注意：go1.6及之前版本请使用golang.org/x/net/context。go1.7及之后已移到标准库context

## 2. 什么是WaitGroup
WaitGroup以前我们在并发的时候介绍过，它是一种控制并发的方式，它的这种方式是控制多个goroutine同时完成

```bash
package main

import (
     "sync"
     "fmt"
     "time"
)
func main() {
	var wg sync.WaitGroup

	wg.Add(2)
	go func() {
		time.Sleep(2*time.Second)
		fmt.Println("1号完成")
		wg.Done()
	}()
	go func() {
		time.Sleep(2*time.Second)
		fmt.Println("2号完成")
		wg.Done()
	}()
	wg.Wait()
	fmt.Println("好了，大家都干完了，放工")
}
```
执行输出

```bash
$ go run test1.go 
1号完成
2号完成
好了，大家都干完了，放工
```

一个很简单的例子，一定要例子中的2个goroutine同时做完，才算是完成，先做好的就要等着其他未完成的，所有的goroutine要都全部完成才可以。

这是一种控制并发的方式，这种尤其适用于，好多个goroutine协同做一件事情的时候，因为每个goroutine做的都是这件事情的一部分，只有全部的goroutine都完成，这件事情才算是完成，这是等待的方式。

在实际的业务种，我们可能会有这么一种场景：需要我们主动的通知某一个goroutine结束。比如我们开启一个后台goroutine一直做事情，比如监控，现在不需要了，就需要通知这个监控goroutine结束，不然它会一直跑，就泄漏了。

## 3. chan通知
我们都知道一个goroutine启动后，我们是无法控制他的，大部分情况是等待它自己结束，那么如果这个goroutine是一个不会自己结束的后台goroutine呢？比如监控等，会一直运行的。

这种情况化，一直傻瓜式的办法是全局变量，其他地方通过修改这个变量完成结束通知，然后后台goroutine不停的检查这个变量，如果发现被通知关闭了，就自我结束。
这种方式也可以，但是首先我们要保证这个变量在多线程下的安全，基于此，有一种更好的方式：`chan + select` 。

```bash
package main
import (
       "fmt"
       "time"
)

func main() {
	stop := make(chan bool)

	go func() {
		for {
			select {
			case <-stop:
				fmt.Println("监控退出，停止了...")
				return
			default:
				fmt.Println("goroutine监控中...")
				time.Sleep(2 * time.Second)
			}
		}
	}()

	time.Sleep(10 * time.Second)
	fmt.Println("可以了，通知监控停止")
	stop<- true
	//为了检测监控过是否停止，如果没有监控输出，就表示停止了
	time.Sleep(5 * time.Second)

}
```
执行输出：

```bash
$ go run test2.go 
goroutine监控中...
goroutine监控中...
goroutine监控中...
goroutine监控中...
goroutine监控中...
可以了，通知监控停止
监控退出，停止了...
```
例子中我们定义一个stop的chan，通知他结束后台goroutine。实现也非常简单，在后台goroutine中，使用select判断stop是否可以接收到值，如果可以接收到，就表示可以退出停止了；如果没有接收到，就会执行default里的监控逻辑，继续监控，只到收到stop的通知。

有了以上的逻辑，我们就可以在其他goroutine种，给`stop chan`发送值了，例子中是在main goroutine中发送的，控制让这个监控的goroutine结束。
发送了`stop<- true`结束的指令后，我这里使用`time.Sleep(5 * time.Second)`故意停顿5秒来检测我们结束监控goroutine是否成功。如果成功的话，不会再有goroutine监控中...的输出了；如果没有成功，监控goroutine就会继续打印goroutine监控中...输出。

这种`chan+select`的方式，是比较优雅的结束一个goroutine的方式，不过这种方式也有局限性，如果有很多goroutine都需要控制结束怎么办呢？如果这些goroutine又衍生了其他更多的goroutine怎么办呢？如果一层层的无穷尽的goroutine呢？这就非常复杂了，即使我们定义很多chan也很难解决这个问题，因为goroutine的关系链就导致了这种场景非常复杂。

## 4. context实现通知
上面说的这种场景是存在的，比如一个网络请求Request，每个Request都需要开启一个goroutine做一些事情，这些goroutine又可能会开启其他的goroutine。所以我们需要一种可以跟踪goroutine的方案，才可以达到控制他们的目的，这就是Go语言为我们提供的Context，称之为上下文非常贴切，它就是goroutine的上下文。

下面我们就使用Go Context重写上面的示例。

```bash
package main

import (
      "context"
      "fmt"
      "time"
)
func main() {
	ctx, cancel := context.WithCancel(context.Background())
	go func(ctx context.Context) {
		for {
			select {
			case <-ctx.Done():
				fmt.Println("监控退出，停止了...")
				return
			default:
				fmt.Println("goroutine监控中...")
				time.Sleep(2 * time.Second)
			}
		}
	}(ctx)

	time.Sleep(10 * time.Second)
	fmt.Println("可以了，通知监控停止")
	cancel()
	//为了检测监控过是否停止，如果没有监控输出，就表示停止了
	time.Sleep(5 * time.Second)

}
```
执行输出：

```bash
$ go run test3.go 
goroutine监控中...
goroutine监控中...
goroutine监控中...
goroutine监控中...
goroutine监控中...
可以了，通知监控停止
监控退出，停止了...
```



重写比较简单，就是把原来的chan stop 换成Context，使用Context跟踪goroutine，以便进行控制，比如结束等。

`context.Background()` 返回一个空的Context，这个空的Context一般用于整个Context树的根节点。然后我们使用`context.WithCancel(parent)`函数，创建一个可取消的子Context，然后当作参数传给goroutine使用，这样就可以使用这个子Context跟踪这个goroutine
在goroutine中，使用select调用`<-ctx.Done()`判断是否要结束，如果接受到值的话，就可以返回结束goroutine了；如果接收不到，就会继续监控。

context.WithCancel(context.Background())的返回值cancel函数即为取消函数，它是cacelfunc类型，通过调用它就可以取消指令。


## 5. context控制多个goroutine

```bash
package main

import (
       "context"
       "time"
       "fmt"
)
func main() {
	ctx, cancel := context.WithCancel(context.Background())
	go watch(ctx,"【监控1】")
	go watch(ctx,"【监控2】")
	go watch(ctx,"【监控3】")

	time.Sleep(10 * time.Second)
	fmt.Println("可以了，通知监控停止")
	cancel()
	//为了检测监控过是否停止，如果没有监控输出，就表示停止了
	time.Sleep(5 * time.Second)
}

func watch(ctx context.Context, name string) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println(name,"监控退出，停止了...")
			return
		default:
			fmt.Println(name,"goroutine监控中...")
			time.Sleep(2 * time.Second)
		}
	}
}
```
执行：

```bash
$ go run test4.go 
【监控3】 goroutine监控中...
【监控1】 goroutine监控中...
【监控2】 goroutine监控中...
【监控2】 goroutine监控中...
【监控1】 goroutine监控中...
【监控3】 goroutine监控中...
【监控2】 goroutine监控中...
【监控3】 goroutine监控中...
【监控1】 goroutine监控中...
【监控2】 goroutine监控中...
【监控1】 goroutine监控中...
【监控3】 goroutine监控中...
【监控2】 goroutine监控中...
【监控3】 goroutine监控中...
【监控1】 goroutine监控中...
可以了，通知监控停止
【监控1】 监控退出，停止了...
【监控3】 监控退出，停止了...
【监控2】 监控退出，停止了...
```

示例中启动了3个监控goroutine进行不断的监控，每一个都使用了Context进行跟踪，当我们使用cancel函数通知取消时，这3个goroutine都会被结束。这就是Context的控制能力，它就像一个控制器一样，按下开关后，所有基于这个Context或者衍生的子Context都会收到通知，这时就可以进行清理操作了，最终释放goroutine，这就优雅的解决了goroutine启动后不可控的问题。

## 6. Context接口
Context的接口定义的比较简洁，我们看下这个接口的方法。

```bash
type Context interface {
	Deadline() (deadline time.Time, ok bool)

	Done() <-chan struct{}

	Err() error

	Value(key interface{}) interface{}
}
```

 - `Deadline`方法是获取设置的截止时间的意思，第一个返回式是截止时间，到了这个时间点，Context会自动发起取消请求；第二个返回值ok==false时表示没有设置截止时间，如果需要取消的话，需要调用取消函数进行取消。
 - `Done`方法返回一个只读的chan，类型为struct{}，我们在goroutine中，如果该方法返回的chan可以读取，则意味着parent context已经发起了取消请求，我们通过Done方法收到这个信号后，就应该做清理操作，然后退出goroutine，释放资源。
 - `Err`方法返回取消的错误原因，因为什么Context被取消。
 - `Value`方法获取该Context上绑定的值，是一个键值对，所以要通过一个Key才可以获取对应的值，这个值一般是线程安全的。

以上四个方法中常用的就是Done了，如果Context取消的时候，我们就可以得到一个关闭的chan，关闭的chan是可以读取的，所以只要可以读取的时候，就意味着收到Context取消的信号了，以下是这个方法的经典用法。

```bash
  func Stream(ctx context.Context, out chan<- Value) error {
  	for {
  		v, err := DoSomething(ctx)
  		if err != nil {
  			return err
  		}
  		select {
  		case <-ctx.Done():
  			return ctx.Err()
  		case out <- v:
  		}
  	}
  }
```
Context接口并不需要我们实现，Go内置已经帮我们实现了2个，我们代码中最开始都是以这两个内置的作为最顶层的partent context，衍生出更多的子Context。

```bash
var (
	background = new(emptyCtx)
	todo       = new(emptyCtx)
)

func Background() Context {
	return background
}

func TODO() Context {
	return todo
}
```
一个是`Background`，主要用于main函数、初始化以及测试代码中，作为Context这个树结构的最顶层的Context，也就是根Context。

一个是`TODO`,它目前还不知道具体的使用场景，如果我们不知道该使用什么Context的时候，可以使用这个。
他们两个本质上都是emptyCtx结构体类型，是一个不可取消，没有设置截止时间，没有携带任何值的Context。

```bash
type emptyCtx int

func (*emptyCtx) Deadline() (deadline time.Time, ok bool) {
	return
}

func (*emptyCtx) Done() <-chan struct{} {
	return nil
}

func (*emptyCtx) Err() error {
	return nil
}

func (*emptyCtx) Value(key interface{}) interface{} {
	return nil
}
```
这就是emptyCtx实现Context接口的方法，可以看到，这些方法什么都没做，返回的都是nil或者零值。

## 7. Context的继承衍生
有了如上的根Context，那么是如何衍生更多的子Context的呢？这就要靠context包为我们提供的With系列的函数了。

```bash
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
func WithValue(parent Context, key, val interface{}) Context
```
这四个With函数，接收的都有一个partent参数，就是父Context，我们要基于这个父Context创建出子Context的意思，这种方式可以理解为子Context对父Context的继承，也可以理解为基于父Context的衍生。

通过这些函数，就创建了一颗Context树，树的每个节点都可以有任意多个子节点，节点层级可以有任意多个。


 
`WithCancel`函数，传递一个父Context作为参数，返回子Context，以及一个取消函数用来取消Context。 `WithDeadline`函数，和WithCancel差不多，它会多传递一个截止时间参数，意味着到了这个时间点，会自动取消Context，当然我们也可以不等到这个时候，可以提前通过取消函数进行取消。

`WithTimeout`和`WithDeadline`基本上一样，这个表示是超时自动取消，是多少时间后自动取消Context的意思。

WithValue函数和取消Context无关，它是为了生成一个绑定了一个键值对数据的Context，这个绑定的数据可以通过Context.Value方法访问到，后面我们会专门讲。

大家可能留意到，前三个函数都返回一个取消函数CancelFunc，这是一个函数类型，它的定义非常简单。

```bash
type CancelFunc func()
```

这就是取消函数的类型，该函数可以取消一个Context，以及这个节点Context下所有的所有的Context，不管有多少层级。
## 8. WithValue传递元数据
通过Context我们也可以传递一些必须的元数据，这些数据会附加在Context上以供使用。

```bash
package main
import (
    "fmt"
    "context"
    "time"
)

var key string="name"

func main() {
	ctx, cancel := context.WithCancel(context.Background())
	//附加值
	valueCtx:=context.WithValue(ctx,key,"【监控1】")
	go watch(valueCtx)
	time.Sleep(10 * time.Second)
	fmt.Println("可以了，通知监控停止")
	cancel()
	//为了检测监控过是否停止，如果没有监控输出，就表示停止了
	time.Sleep(5 * time.Second)
}

func watch(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			//取出值
			fmt.Println(ctx.Value(key),"监控退出，停止了...")
			return
		default:
			//取出值
			fmt.Println(ctx.Value(key),"goroutine监控中...")
			time.Sleep(2 * time.Second)
		}
	}
}
```
执行输出：

```bash
$ go run test5.go
【监控1】 goroutine监控中...
【监控1】 goroutine监控中...
【监控1】 goroutine监控中...
【监控1】 goroutine监控中...
【监控1】 goroutine监控中...
可以了，通知监控停止
【监控1】 监控退出，停止了...
```
在前面的例子，我们通过传递参数的方式，把name的值传递给监控函数。在这个例子里，我们实现一样的效果，但是通过的是Context的Value的方式。

我们可以使用context.WithValue方法附加一对K-V的键值对，这里Key必须是等价性的，也就是具有可比性；Value值要是线程安全的。

这样我们就生成了一个新的Context，这个新的Context带有这个键值对，在使用的时候，可以通过Value方法读取ctx.Value(key)
## 9. 实例
### 9.1 超时取消

```bash
package main

import (
	"fmt"
	"sync"
	"time"

	"context"
)

var (
	wg sync.WaitGroup
)

func work(ctx context.Context) error {
	defer wg.Done()

	for i := 0; i < 1000; i++ {
		select {
		case <-time.After(2 * time.Second):
			fmt.Println("Doing some work ", i)

		// we received the signal of cancelation in this channel
		case <-ctx.Done():
			fmt.Println("Cancel the context ", i)
			return ctx.Err()
		}
	}
	return nil
}

func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 4*time.Second)
	defer cancel()

	fmt.Println("Hey, I'm going to do some work")

	wg.Add(1)
	go work(ctx)
	wg.Wait()

	fmt.Println("Finished. I'm going home")
}
```
## 10. 遵循规则
遵循以下规则，以保持包之间的接口一致，并启用静态分析工具以检查上下文传播。

不要将 `Contexts` 放入结构体，相反context应该作为第一个参数传入，命名为ctx。 `func DoSomething（ctx context.Context，arg Arg）error { // ... use ctx ... }`
即使函数允许，也不要传入nil的 Context。如果不知道用哪种 Context，可以使用`context.TODO()`。
使用context的Value相关方法只应该用于在程序和接口中传递的和请求相关的元数据，不要用它来传递一些可选的参数
相同的 Context 可以传递给在不同的goroutine；Context 是并发安全的。














