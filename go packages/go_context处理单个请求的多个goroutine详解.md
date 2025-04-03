
参考资料：
[golang服务器开发利器 context用法详解](https://www.jianshu.com/p/d24bf8b6c869)
[https://github.com/go-training/training/tree/master/example35-goroutine-with-context](https://github.com/go-training/training/tree/master/example35-goroutine-with-context)
[context使用](https://mp.weixin.qq.com/s?__biz=MzAxMTA4Njc0OQ==&mid=2651441088&idx=4&sn=34336e972cd5993bb86f66358739210a&chksm=80bb1932b7cc902469c0723f18a1030330154648763ad9fa3bd257661e8c4228139a3ea30c44&mpshare=1&scene=1&srcid=0805isxPvBlK5y8Aki5BM6cP&sharer_sharetime=1596628450552&sharer_shareid=9e1d0f93025303e47ff2523f5ebf4078&key=89e6c39d301e544ae5482d271724d60cca7c5d343422c075193ba072de4dde24be8fb545ab0f50004e840a682d0dc1347a22531a9eeec33da5edeabc8808204f29354356e4d4716f3d7167b1538467cf&ascene=1&uin=MjkwMDAzNTYzOQ==&devicetype=Windows%2010%20x64&version=62090529&lang=zh_CN&exportkey=Ae2yogMs28yo8x40hDVfe3o=&pass_ticket=aIl2B1VGLWrFVqmYgT7Tf3juCRy/IhwrmGigP1F1fsgiz9j4covUuRBpwdFXF2/0)

控制并发的两种方式

 - 使用waitGroup
 - 使用context

### 场景1：多个goroutine执行同一件事

 waitGroup使用

```go
func main() {
	var wg sync.WaitGroup

	wg.Add(2)
	go func() {
		time.Sleep(2 * time.Second)
		fmt.Println("job 1 done.")
		wg.Done()
	}()
	go func() {
		time.Sleep(1 * time.Second)
		fmt.Println("job 2 done.")
		wg.Done()
	}()
	wg.Wait()
	fmt.Println("All Done.")
}
```
	
```bash
$ go run con3.go
job 2 done.
job 1 done.
All Done.
```

### 场景2: 当主动通知停止？如何做？
使用channel + select

```go
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
				fmt.Println("got the stop channel")
				return
			default:
				fmt.Println("still working")
				time.Sleep(1 * time.Second)
			}
		}
	}()

	time.Sleep(5 * time.Second)
	fmt.Println("stop the gorutine")
	stop <- true
	time.Sleep(5 * time.Second)
}
```

```bash
$ go run con4.go
still working
still working
still working
still working
still working
stop the gorutine
got the stop channel
```
### 当多个goroutine或goroutine中有goroutine
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/41e2313f10f69a4141f74420b1109c2c.png)

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	ctx, cancel := context.WithCancel(context.Background())

	go func() {
		for {
			select {
			case <-ctx.Done():
				fmt.Println("got the stop channel")
				return
			default:
				fmt.Println("still working")
				time.Sleep(1 * time.Second)
			}
		}
	}()

	time.Sleep(5 * time.Second)
	fmt.Println("stop the gorutine")
	cancel()
	time.Sleep(5 * time.Second)
}
```

```bash
$ go run con5.go
still working
still working
still working
still working
still working
stop the gorutine
got the stop channel
```
**多个goroutine**

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	ctx, cancel := context.WithCancel(context.Background())

	go worker(ctx, "node01")
	go worker(ctx, "node02")
	go worker(ctx, "node03")

	time.Sleep(5 * time.Second)
	fmt.Println("stop the gorutine")
	cancel()
	time.Sleep(5 * time.Second)
}

func worker(ctx context.Context, name string) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println(name, "got the stop channel")
			return
		default:
			fmt.Println(name, "still working")
			time.Sleep(1 * time.Second)
		}
	}
}
```

```bash
$ go run con6.go
node03 still working
node01 still working
node02 still working
node03 still working
node02 still working
node01 still working
node01 still working
node03 still working
node02 still working
node02 still working
node01 still working
node03 still working
node03 still working
node02 still working
node01 still working
stop the gorutine
node01 got the stop channel
node03 got the stop channel
node02 got the stop channel
```

