#  python 模块 （1）asyncio 协程异步




----
## 1. asyncio介绍
异步IO：就是发起一个IO操作（如：网络请求，文件读写等），这些操作一般是比较耗时的，不用等待它结束，可以继续做其他事情，结束时会发来通知。

协程：又称为微线程，在一个线程中执行，执行函数时可以随时中断，由程序（用户）自身控制，执行效率极高，与多线程比较，没有切换线程的开销和多线程锁机制。

python中异步IO操作是通过[asyncio](https://docs.python.org/3/library/asyncio.html)来实现的。
## 2. 协程基础
从语句上看，协程和生成器类似，都是包含了yield关键字，不同之处在于协程中yield关键词通常出现在=右边，可以产出值a（y = yield a）或不产出值时为None（y = yield）。调用方可以用send函数发送值给协程。

激活协程时在yield处暂停，等待调用方发送数据，下次继续在yield暂停。从根本上看，yield是流程控制的工具，可以实现协作式多任务，这也是后面讲解异步IO的基础。
示例1：

```python
#!/usr/bin/python

def coroutine_example(name):
    print('start coroutine...name:', name)
    x = yield name #调用next()时，产出yield右边的值后暂停；调用send()时，产出值赋给x，并往下运行
    print('send值:', x)

if __name__ == '__main__':
  coro = coroutine_example('Zarten')
  print('next的返回值:', next(coro))
  print('send的返回值:', coro.send(6))
```
输出：

```python
$ python3 ns1.py
start coroutine...name: Zarten
next的返回值: Zarten
send值: 6
Traceback (most recent call last):
  File "ns1.py", line 11, in <module>
    print('send的返回值:', coro.send(6))
StopIteration
```
必须先调用next()函数预激活协程，不然send()函数无法使用。

调用next()时，产出yield右边的值后暂停不再往yield的下一行执行（一般不需要next产出值），等待send的到来，调用send()时，产出值赋给x(可对x作进一步处理)，并往下运行。

协程结束时会跟生成器一样抛出StopIteration的异常给调用方，调用方可以捕获它后处理。
## 3. 让协程返回值以及yield from说明
获取协程的返回值

当结束协程时，会返回返回值，调用方会抛出StopIteration异常，返回值就在异常对象的value属性中。

```python
#!/usr/bin/python

def coroutine_example(name):
    print('start coroutine...name:', name)

    while True:
        x = yield name #调用next()时，产出yield右边的值后暂停；调用send()时，产出值赋给x，并往下运行
        if x is None:
            return 'zhihuID: Zarten'
        print('send值:', x)

coro = coroutine_example('Zarten')
next(coro)
print('send的返回值:', coro.send(6))
try:
    coro.send(None)
except StopIteration as e:
    print('返回值：', e.value)
```
## 4. yield from 说明
yield from跟for循环很相似，但功能更多一些

```python
#!/usr/bin/python

def for_test():
    for i in range(3):
       yield i

print(list(for_test()))

def yield_from_test():
    yield from range(3)

print(list(yield_from_test()))
```
输出：

```powershell
$ python3 ns3.py
[0, 1, 2]
[0, 1, 2]
```
其实`yield from`内部会自动捕获`StopIteration`异常，并把异常对象的value属性变成yield from表达式的值。

yield from x 表达式内部首先是调用iter(x),然后再调用next(),因此x是任何的可迭代对象。yield from 的主要功能就是打开双向通道，把最外层的调用方和最内层的子生成器连接起来。

下面代码展示：调用方发送的值在yield from表达式处直接传递给子生成器，并在yield from处等待子生成器的返回。

```python
#!/usr/bin/python

def coroutine_example(name):
    print('start coroutine...name:', name)
    x = yield name #调用next()时，产出yield右边的值后暂停；调用send()时，产出值赋给x，并往下运行
    print('send值:', x)
    return 'zhihuID: Zarten'

def grouper2():
    result2 = yield from coroutine_example('Zarten') #在此处暂停，等待子生成器的返回后继续往下执行
    print('result2的值：', result2)
    return result2

def grouper():
    result = yield from grouper2() #在此处暂停，等待子生成器的返回后继续往下执行
    print('result的值：', result)
    return result

def main():
    g = grouper()
    next(g)
    try:
        g.send(10)
    except StopIteration as e:
        print('返回值：', e.value)

if __name__ == '__main__':
    main()
```

```python
$ python3 ns4.py 
start coroutine...name: Zarten
send值: 10
result2的值： zhihuID: Zarten
result的值： zhihuID: Zarten
返回值： zhihuID: Zarten
```
从上面也可看到yield from起到一个双向通道的作用，同时子生成器也可使用`yield from`调用另一个子生成器，一直嵌套下去直到遇到yield表达式结束链式。

yield from一般用于asyncio模块做异步IO

## 5. 异步IO（asyncio）原理

asyncio 模块最大特点就是，只存在一个线程，跟 JavaScript 一样。

由于只有一个线程，就不可能多个任务同时运行。asyncio 是"多任务合作"模式（cooperative multitasking），允许异步任务交出执行权给其他任务，等到其他任务完成，再收回执行权继续往下执行，这跟 JavaScript 也是一样的。

由于代码的执行权在多个任务之间交换，所以看上去好像多个任务同时运行，其实底层只有一个线程，多个任务分享运行时间。表面上，这是一个不合理的设计，明明有多线程多进程的能力，为什么放着多余的 CPU 核心不用，而只用一个线程呢？但是就像前面说的，单线程简化了很多问题，使得代码逻辑变得简单，写法符合直觉。

syncio 模块在单线程上启动一个事件循环（event loop），时刻监听新进入循环的事件，加以处理，并不断重复这个过程，直到异步任务结束。事件循环的内部机制，可以参考 JavaScript 的模型，两者是一样的。
![https://www.wangbase.com/blogimg/asset/201911/bg2019112005.jpg](https://img-blog.csdnimg.cn/20200528110338627.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 6. asyncio API基础
下面介绍 asyncio 模块最主要的几个API。注意，必须使用 Python 3.7 或更高版本，早期的语法已经变了。

第一步，import 加载 asyncio 模块。
```python
import asyncio
```
第二步，函数前面加上 async 关键字，就变成了 async 函数。这种函数最大特点是执行可以暂停，交出执行权。

```python
async def main():
```

第三步，在 async 函数内部的异步任务前面，加上await命令。

```python
await asyncio.sleep(1)
```
上面代码中，`asyncio.sleep(1)` 方法可以生成一个异步任务，休眠1秒钟然后结束。
执行引擎遇到`await`命令，就会在异步任务开始执行之后，暂停当前 async 函数的执行，把执行权交给其他任务。等到异步任务结束，再把执行权交回 async 函数，继续往下执行。

第四步，`async.run()` 方法加载 async 函数，启动事件循环。
```python
asyncio.run(main())
```
上面代码中，`asyncio.run()` 在事件循环上监听 `async` 函数main的执行。等到 main 执行完了，事件循环才会终止。

## 7. async 函数的示例

```python
#!/usr/bin/env python3
# async.py

import asyncio

async def count():
    print("One")
    await asyncio.sleep(1)
    print("Two")

async def main():
    await asyncio.gather(count(), count(), count())

asyncio.run(main())
```

输出：

```python
$ python3 async.py
One
One
One
Two
Two
Two
```

在 async 函数main的里面，`asyncio.gather()` 方法将多个异步任务（三个 count()）包装成一个新的异步任务，必须等到内部的多个异步任务都执行结束，这个新的异步任务才会结束。

三个 count() 依次执行，打印完 One，就休眠1秒钟，把执行权交给下一个 count()，所以先连续打印出三个 One。等到1秒钟休眠结束，执行权重新交回第一个 count()，开始执行 await 命令下一行的语句，所以会接着打印出三个Two。脚本总的运行时间是1秒。

作为对比，下面是这个例子的同步版本 sync.py。

```python
#!/usr/bin/env python3
# sync.py

import time

def count():
    print("One")
    time.sleep(1)
    print("Two")

def main():
    for _ in range(3):
        count()

main()
```
输出：

```python
$ python3 sync.py 
One
Two
One
Two
One
Two
```
## 8. asyncio三种执行协程的机制：

 1. 使用asyncio.run()执行协程。一般用于执行最顶层的入口函数，如main()。
 2. await一个协程。一般用于在一个协程中调用另一协程

 如下是一个示例：

```python
 #!/usr/bin/python

import time
import asyncio

async def say_after(delay,what):
     await asyncio.sleep(delay)
     print(what)

async def main():
    print(f"started at {time.strftime('%X')}")
    await say_after(1,"hello")
    await say_after(2,"world")
    print(f"finished at {time.strftime('%X')}")

asyncio.run(main())
```
输出：

```python
$ python3 ns6.py
started at 11:42:48
hello
world
finished at 11:42:51
```
执行耗时 3秒

 3. 用`asyncio.create_task()`方法将Coroutine（协程）封装为Task（任务）。一般用于实现异步并发操作。
    需要注意的是，只有在当前线程存在事件循环的时候才能创建任务（Task）。

我们修改以上的例程，并发执行 两个say_after协程。

```python
#!/usr/bin/python

import time
import asyncio

async def say_after(delay,what):
     await asyncio.sleep(delay)
     print(what)

async def main():
    task1 = asyncio.create_task(say_after(1,"hello"))
    task2 = asyncio.create_task(say_after(2,"world"))
    print(f"started at {time.strftime('%X')}")
    await task1
    await task2
    print(f"finished at {time.strftime('%X')}")

asyncio.run(main())
```
输出：

```python
$ python3 ns7.py 
started at 14:22:09
hello
world
finished at 14:22:11
```
耗时2秒

## 9. “可等待”对象（Awaitables）
如果一个对象能够被用在await表达式中，那么我们称这个对象是可等待对象（awaitable object）。很多asyncio API都被设计成了可等待的。
主要有三类可等待对象：

协程coroutine
任务Task
未来对象Future。

## 10. Coroutine（协程）
Python的协程是可等待的（awaitable），因此能够被其他协程用在await表达式中。

```python
import asyncio
import time

async def nested():
    print("something")

async def main():
    # 如果直接调用 "nested()"，什么都不会发生.
    # 直接调用的时候只是创建了一个 协程对象 ，但这个对象没有被 await,
    # 所以它并不会执行.
    nested()
    print(f"finished at {time.strftime('%X')}")

    # 那么我们 await 这个协程，看看会是什么结果:
    await nested()  # 将会打印 "something".

asyncio.run(main())
```
输出：

```python
as8.py:11: RuntimeWarning: coroutine 'nested' was never awaited
  nested()
RuntimeWarning: Enable tracemalloc to get the object allocation traceback
finished at 14:28:28
something
```
术语coroutine或协程指代两个关系紧密的概念：

 - 协程函数（coroutine function）:由async def定义的函数；
 - 协程对象（coroutine object）：调用 协程函数返回的对象。

asyncio也支持传统的基于生成器的协程。

## 11. Task（任务）
`Task`用来 并发的 调度协程。
当一个协程通过类似 `asyncio.create_task()` 的函数被封装进一个 Task时，这个协程 会很快被自动调度执行：

```python
import asyncio

async def nested():
    print("something")
    return 42

async def main():
    # Schedule nested() to run soon concurrently
    # with "main()".
    task = asyncio.create_task(nested())

    # "task" can now be used to cancel "nested()", or
    # can simply be awaited to wait until it is complete:
    await task

asyncio.run(main())
```
输出：

```python
$ python3 ns8.py
something
```
## 12. Future（未来对象）
Future 是一种特殊的 底层 可等待对象，代表一个异步操作的最终结果。
当一个Future对象被await的时候，表示当前的协程会持续等待，直到 Future对象所指向的异步操作执行完毕。
在asyncio中，Future对象能使基于回调的代码被用于asyn/await表达式中。
一般情况下，在应用层编程中，没有必要 创建Future对象。
有时候，有些Future对象会被一些库和asyncio API暴露出来，我们可以await它们：

## 13. 执行asyncio程序

```python
asyncio.run(coro, * , debug=False)
```

这个函数运行coro参数指定的 协程，负责 管理**asyncio事件循环** ， 终止异步生成器。
在同一个线程中，当已经有asyncio事件循环在执行时，不能调用此函数。
如果`debug=True`，事件循环将运行在 调试模式。
此函数总是创建一个新的事件循环，并在最后关闭它。建议将它用作asyncio程序的主入口，并且只调用一次。
Python3.7新增
重要：这个函数是在Python3.7被临时添加到asyncio中的。


参考：

 - [Async IO in Python: A Complete Walkthrough](https://realpython.com/async-io-python/)
 - [Python Asyncio Part 1 – Basic Concepts and Patterns](https://bbc.github.io/cloudfit-public-docs/asyncio/asyncio-part-1.html)
 - [Asyncio: An Introduction](https://www.datacamp.com/tutorial/asyncio-introduction)
 - [Asyncio: Understanding Async / Await in Python](https://www.youtube.com/watch?v=bs9tlDFWWdQ)
 - [Python Asynchronous Programming - AsyncIO & Async/Await](https://www.youtube.com/watch?v=t5Bo1Je9EmE)

