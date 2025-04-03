![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/228c66bbd565f519db9a2d91d1f8af18.png#pic_center)

-----

## io.Pipe()定义

```bash
func Pipe() (*PipeReader, *PipeWriter)
```
## io.Pipe使用
`io.Pipe`会返回一个reader和writer,对reader读取（或写入writer）后，进程会被锁住，直到writer有新数据流进入或关闭（或reader把数据读走）。如下面程序会出现死锁。

```bash
package main

import (
 "io"
)

func main() {
  reader, writer := io.Pipe()
  
  // writer写入后会锁住进程
	wrtier.Write([]byte("hello"))
  defer writer.Close()

  // Reader读取后会锁住进程
  buffer := make([]byte, 100)
  reader.Read(buffer)
  println(string(buffer))
}
```
输出;

```bash
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [select]:
io.(*pipe).Write(0xc00004e060, 0xc0000160b8, 0x5, 0x5, 0x0, 0x0, 0x0)
	/usr/local/go/src/io/pipe.go:94 +0x1db
io.(*PipeWriter).Write(...)
	/usr/local/go/src/io/pipe.go:163
main.main()
	/root/go/io/test.go:11 +0x16a
exit status 2
```

处理的办法是新建一个goroutine专门写writer

```bash
package main

import (
 "io"
)

func main() {
  reader, writer := io.Pipe()
  // 创建goroutine给writer
	go func() {
		writer.Write([]byte("hello"))
		defer writer.Close()
	}()
	buffer := make([]byte, 100)
	reader.Read(buffer)
	println(string(buffer))
}
```
执行输出：

```bash
$ go run test2.go
hello
```
第二种方法

```bash
package main

import (
 "io"
)

func main() {
	reader, writer := io.Pipe()
	defer writer.Close()
  // 创建goroutine给reader
	go func() {
		buffer := make([]byte, 100)
    reader.Read(buffer)
		println(string(buffer))
	}()
	writer.Write([]byte("hello"))
}
```

执行输出：

```bash
[root@localhost io]# go run test3.go 
[root@localhost io]# 
```
上面这段代码已经可以顺利地从writer中读取中消息了。但是实际运行的过程中我们会发现这段代码在屏幕上没有任何输出。其原因是在启动的子线程中还没执行完毕时，主线程已经结束，并强制结束了其他所有线程，解决的办法是让主线程加上一个锁。

```bash
package main

import (
 "io"
)

func main() {
	reader, writer := io.Pipe()
	defer writer.Close()
	lock := make(chan int)
	// 创建goroutine给reader
	go func() {
		buffer := make([]byte, 100)
		reader.Read(buffer)
		println(string(buffer))
		lock <- 1
	}()
	writer.Write([]byte("hello"))
	<-lock
}
```
执行输出：

```bash
[root@localhost io]# go run test4.go
hello
```
## io.Pipe在进程通讯中使用
我们以Unix系统中的grep作为例子。命令

```bash
$ cat input 
hello world
welcome to world
turtle 
```

```bash
$ cat input | grep turtle
turtle
```

将以cat input的输出作为grep turtle的输入，运行grep程序。整个命令的作用就是在问进input中查找带有单词turtle的行。现在我们用go语言实现这个命令的调用

```bash
package main

import (
 "io"
 "os"
 "os/exec"
)

func main() {
	fPtr, _ := os.OpenFile("input", os.O_RDONLY, 0755)
	cmd := exec.Command("grep", "turtle")
	r, w := io.Pipe()
	cmd.Stdin = r
	cmd.Stdout = os.Stdout
	go func() {
		io.Copy(w, fPtr)
		w.Close()
	}()
	cmd.Run()
}
```
执行输出：

```bash
[root@localhost io]# go run test5.go 
turtle
```

