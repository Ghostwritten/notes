上一讲，我带你一起领略了 Mutex 的架构演进之美，现在我们已经清楚 Mutex 的实现细节了。当前 Mutex 的实现貌似非常复杂，其实主要还是针对饥饿模式和公平性问题，做了一些额外处理。但是，我们在第一讲中已经体验过了，Mutex 使用起来还是非常简单的，毕竟，它只有 Lock 和 Unlock 两个方法，使用起来还能复杂到哪里去？

正常使用 Mutex 时，确实是这样的，很简单，基本不会有什么错误，即使出现错误，也是在一些复杂的场景中，比如跨函数调用 Mutex 或者是在重构或者修补 Bug 时误操作。但是，我们使用 Mutex 时，确实会出现一些 Bug，比如说忘记释放锁、重入锁、复制已使用了的 Mutex 等情况。那在这一讲中，我们就一起来看看使用 Mutex 常犯的几个错误，做到“Bug 提前知，后面早防范”。

##  常见的 4 种错误场景

我总结了一下，使用 Mutex 常见的错误场景有 4 类，分别是 **`Lock/Unlock` 不是成对出现、Copy 已使用的 Mutex、重入和死锁**。下面我们一一来看。

##  Lock/Unlock 不是成对出现
Lock/Unlock 没有成对出现，就意味着会出现死锁的情况，或者是因为 Unlock 一个未加锁的 Mutex 而导致 panic。

我们先来看看缺少 Unlock 的场景，常见的有三种情况：

 - 代码中有太多的 `if-else` 分支，可能在某个分支中漏写了 `Unlock`；
 - 在重构的时候把 Unlock 给删除了；
 - Unlock 误写成了 Lock。


我们再来看缺少 Lock 的场景，这就很简单了，一般来说就是误操作删除了 Lock。 比如先前使用 Mutex 都是正常的，结果后来其他人重构代码的时候，由于对代码不熟悉，或者由于开发者的马虎，把 Lock 调用给删除了，或者注释掉了。比如下面的代码，`mu.Lock()` 一行代码被删除了，直接 Unlock 一个未加锁的 Mutex 会 `panic`：


```bash
func foo() {
    var mu sync.Mutex
    defer mu.Unlock()
    fmt.Println("hello world!")
}
```
运行的时候 panic：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/357309278727c7203d21c4882e2f980d.png)
##  Copy 已使用的 Mutex
二种误用是 Copy 已使用的 Mutex。在正式分析这个错误之前，我先交代一个小知识点，那就是 Package sync 的同步原语在使用后是不能复制的。我们知道 Mutex 是最常用的一个同步原语，那它也是不能复制的。为什么呢？

原因在于，Mutex 是一个有状态的对象，它的 state 字段记录这个锁的状态。如果你要复制一个已经加锁的 Mutex 给一个新的变量，那么新的刚初始化的变量居然被加锁了，这显然不符合你的期望，因为你期望的是一个零值的 Mutex。关键是在并发环境下，你根本不知道要复制的 Mutex 状态是什么，因为要复制的 Mutex 是由其它 `goroutine` 并发访问的，状态可能总是在变化。


```html
type Counter struct {
    sync.Mutex
    Count int
}


func main() {
    var c Counter
    c.Lock()
    defer c.Unlock()
    c.Count++
    foo(c) // 复制锁
}

// 这里Counter的参数是通过复制的方式传入的
func foo(c Counter) {
    c.Lock()
    defer c.Unlock()
    fmt.Println("in foo")
}
```
第 12 行在调用 foo 函数的时候，调用者会复制 `Mutex` 变量 c 作为 foo 函数的参数，不幸的是，复制之前已经使用了这个锁，这就导致，复制的 Counter 是一个带状态 `Counter`。

怎么办呢？Go 在运行时，有死锁的检查机制（[checkdead()](https://golang.org/src/runtime/proc.go?h=checkdead#L4345)  方法），它能够发现死锁的 goroutine。这个例子中因为复制了一个使用了的 Mutex，导致锁无法使用，程序处于死锁的状态。程序运行的时候，死锁检查机制能够发现这种死锁情况并输出错误信息，如下图中错误信息以及错误堆栈：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7c5e0b1d370320d5b27adc85a3e41c36.png)
你肯定不想运行的时候才发现这个因为复制 `Mutex` 导致的死锁问题，那么你怎么能够及时发现问题呢？可以使用 **vet 工具**，把检查写在 `Makefile` 文件中，在持续集成的时候跑一跑，这样可以及时发现问题，及时修复。我们可以使用 `go vet` 检查这个 Go 文件：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1f1b5338c265e8b07b82f541c3ddd312.png)
你看，使用这个工具就可以发现 Mutex 复制的问题，错误信息显示得很清楚，是在调用 foo 函数的时候发生了 `lock value` 复制的情况，还告诉我们出问题的代码行数以及 `copy lock` 导致的错误。

那么，vet 工具是怎么发现 `Mutex` 复制使用问题的呢？我带你简单分析一下。

检查是通过[copylock分析器](https://github.com/golang/tools/blob/master/go/analysis/passes/copylock/copylock.go)静态分析实现的。这个分析器会分析函数调用、range 遍历、复制、声明、函数返回值等位置，有没有锁的值 copy 的情景，以此来判断有没有问题。可以说，只要是实现了 Locker 接口，就会被分析。我们看到，下面的代码就是确定什么类型会被分析，其实就是实现了 Lock/Unlock 两个方法的 Locker 接口：


```bash
var lockerType *types.Interface
  
  // Construct a sync.Locker interface type.
  func init() {
    nullary := types.NewSignature(nil, nil, nil, false) // func()
    methods := []*types.Func{
      types.NewFunc(token.NoPos, nil, "Lock", nullary),
      types.NewFunc(token.NoPos, nil, "Unlock", nullary),
    }
    lockerType = types.NewInterface(methods, nil).Complete()
  }
```

其实，有些没有实现 Locker 接口的同步原语（比如 `WaitGroup`），也能被分析。我先卖个关子，后面我们会介绍这种情况是怎么实现的。

##  重入
接下来，我们来讨论“重入”这个问题。在说这个问题前，我先解释一下个概念，叫“可重入锁”。如果你学过 Java，可能会很熟悉 ReentrantLock，就是可重入锁，这是 Java 并发包中非常常用的一个同步原语。它的基本行为和互斥锁相同，但是加了一些扩展功能。

如果你没接触过 Java，也没关系，这里只是提一下，帮助会 Java 的同学对比来学。那下面我来具体讲解可重入锁是咋回事儿。

当一个线程获取锁时，如果没有其它线程拥有这个锁，那么，这个线程就成功获取到这个锁。之后，如果其它线程再请求这个锁，就会处于阻塞等待的状态。但是，如果拥有这把锁的线程再请求这把锁的话，不会阻塞，而是成功返回，所以叫可重入锁（有时候也叫做递归锁）。只要你拥有这把锁，你可以可着劲儿地调用，比如通过递归实现一些算法，调用者不会阻塞或者死锁。

了解了可重入锁的概念，那我们来看 Mutex 使用的错误场景。**划重点了：Mutex 不是可重入的锁。**

想想也不奇怪，因为 Mutex 的实现中没有记录哪个 `goroutine` 拥有这把锁。理论上，任何 goroutine 都可以随意地 Unlock 这把锁，所以没办法计算重入条件，毕竟，“臣妾做不到啊”！

所以，一旦误用 Mutex 的重入，就会导致报错。下面是一个误用 Mutex 的重入例子：


```bash
func foo(l sync.Locker) {
    fmt.Println("in foo")
    l.Lock()
    bar(l)
    l.Unlock()
}


func bar(l sync.Locker) {
    l.Lock()
    fmt.Println("in bar")
    l.Unlock()
}


func main() {
    l := &sync.Mutex{}
    foo(l)
}
```
写完这个 Mutex 重入的例子后，运行一下，你会发现类似下面的错误。程序一直在请求锁，但是一直没有办法获取到锁，结果就是 Go 运行时发现死锁了，没有其它地方能够释放锁让程序运行下去，你通过下面的错误堆栈信息就能定位到哪一行阻塞请求锁：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a79015083c53557638163a380bfdb104.png)
学到这里，你可能要问了，虽然标准库 Mutex 不是可重入锁，但是如果我就是想要实现一个可重入锁，可以吗？可以，那我们就自己实现一个。这里的关键就是，实现的锁要能记住当前是哪个 `goroutine` 持有这个锁。我来提供两个方案。

 - 方案一：通过 `hacker` 的方式获取到 `goroutine id`，记录下获取锁的 `goroutine id`，它可以实现 Locker接口。
 - 方案二：调用 `Lock/Unlock` 方法时，由 `goroutine` 提供一个 `token`，用来标识它自己，而不是我们通过 hacker 的方式获取到 goroutine id，但是，这样一来，就不满足 Locker 接口了。


可重入锁（递归锁）解决了代码重入或者递归调用带来的死锁问题，同时它也带来了另一个好处，就是我们可以要求，只有持有锁的 goroutine 才能 unlock 这个锁。这也很容易实现，因为在上面这两个方案中，都已经记录了是哪一个 goroutine 持有这个锁。

下面我们具体来看这两个方案怎么实现。

##  方案一：goroutine id
这个方案的关键第一步是获取 goroutine id，方式有两种，分别是简单方式和 hacker 方式。

简单方式，就是通过 `runtime.Stack` 方法获取栈帧信息，栈帧信息里包含 `goroutine id`。你可以看看上面 `panic` 时候的贴图，`goroutine id` 明明白白地显示在那里。`runtime.Stack` 方法可以获取当前的 goroutine 信息，第二个参数为 true 会输出所有的 goroutine 信息，信息的格式如下：


```bash
goroutine 1 [running]:
main.main()
        ....../main.go:19 +0xb1
```
第一行格式为 `goroutine xxx`，其中 xxx 就是 `goroutine id`，你只要解析出这个 id 即可。解析的方法可以采用下面的代码：


```bash
func GoID() int {
    var buf [64]byte
    n := runtime.Stack(buf[:], false)
    // 得到id字符串
    idField := strings.Fields(strings.TrimPrefix(string(buf[:n]), "goroutine "))[0]
    id, err := strconv.Atoi(idField)
    if err != nil {
        panic(fmt.Sprintf("cannot get goroutine id: %v", err))
    }
    return id
}
```

