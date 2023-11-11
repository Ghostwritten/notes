#  python 模块 gevent 协程



----
## 1. 进程、线程、协程区分

我们通常所说的协程Coroutine其实是corporate routine的缩写，直接翻译为协同的例程，一般我们都简称为协程。

在linux系统中，线程就是轻量级的进程，而我们通常也把协程称为轻量级的线程即微线程。

### 1.1 进程和协程

下面对比一下进程和协程的相同点和不同点：

相同点：

 1. 相同点存在于，当我们挂起一个执行流的时，我们要保存的东西：
 2. 栈， 其实在你切换前你的局部变量，以及要函数的调用都需要保存，否则都无法恢复 寄存器状态,这个其实用于当你的执行流恢复后要做什么

而寄存器和栈的结合就可以理解为上下文，上下文切换的理解：
CPU看上去像是在并发的执行多个进程，这是通过处理器在进程之间切换来实现的，操作系统实现这种交错执行的机制称为上下文切换

操作系统保持跟踪进程运行所需的所有状态信息。这种状态，就是上下文。
在任何一个时刻，操作系统都只能执行一个进程代码，当操作系统决定把控制权从当前进程转移到某个新进程时，就会进行上下文切换，即保存当前进程的上下文，恢复新进程的上下文，然后将控制权传递到新进程，新进程就会从它上次停止的地方开始。

不同点：

 1. 执行流的调度者不同，进程是内核调度，而协程是在用户态调度，也就是说进程的上下文是在内核态保存恢复的，而协程是在用户态保存恢复的，很显然用户态的代价更低
 2. 进程会被强占，而协程不会，也就是说协程如果不主动让出CPU，那么其他的协程，就没有执行的机会。
 3. 对内存的占用不同，实际上协程可以只需要4K的栈就足够了，而进程占用的内存要大的多
 4. 从操作系统的角度讲，多协程的程序是单进程，单协程

### 1.2 线程和协程

既然我们上面也说了，协程也被称为微线程，下面对比一下协程和线程：

 1. 线程之间需要上下文切换成本相对协程来说是比较高的，尤其在开启线程较多时，但协程的切换成本非常低。
 2. 同样的线程的切换更多的是靠操作系统来控制，而协程的执行由我们自己控制。

协程只是在单一的线程里不同的协程之间切换，其实和线程很像，线程是在一个进程下，不同的线程之间做切换，这也可能是协程称为微线程的原因吧。

## 2. 简介

[Gevent](http://www.gevent.org/) 模块是一种基于协程的Python网络库，它用到Greenlet提供的，封装了libevent事件循环的高层同步API。它让开发者在不改变编程习惯的同时，用同步的方式写异步I/O的代码。

### 2.1 greenlets
　如果想了解gevent的调度流程，最重要的是对greenlet有基本的了解。下面总结一些个人认为比较重要的点：

 1. 每一个greenlet.greenlet实例都有一个parent（可指定，默认为创生新的greenlet.greenlet所在环境），当greenlet.greenlet实例执行完逻辑正常结束、或者抛出异常结束时，执行逻辑切回到其parent
 2. 可以继承greenlet.greenlet，子类需要实现run方法，当调用greenlet.switch方法时会调用到这个run方法

　　在gevent中，有两个类继承了greenlet.greenlet，分别是gevent.hub.Hub和gevent.greenlet.Greenlet。后文中，如果是greenlet.greenlet这种写法，那么指的是原生的类库greentlet，如果是greenlet（或者Greenlet）那么指gevent封装后的greenlet。

gevent中的主要模式, 它是以C扩展模块形式接入Python的轻量级协程。 全部运行在主程序操作系统进程的内部，但它们被程序员协作式地调度。

在任何时刻，只有一个协程在运行。

区别于multiprocessing、threading等提供真正并行构造的库， 这些库轮转使用操作系统调度的进程和线程，是真正的并行。

## 3. 特点


```python
基于libev的快速事件循环(Linux上epoll，FreeBSD上kqueue）。
基于greenlet的轻量级执行单元。
API的概念和Python标准库一致(如事件，队列)。
可以配合socket，ssl模块使用。
能够使用标准库和第三方模块创建标准的阻塞套接字(gevent.monkey)。
默认通过线程池进行DNS查询,也可通过c-are(通过GEVENT_RESOLVER=ares环境变量开启）。
TCP/UDP/HTTP服务器
子进程支持（通过gevent.subprocess）
线程池
```
## 4. 安装
安装和依赖
依赖于greenlet library
支持python 2.6+ 、3.3+

```bash
pip install gevent
```
## 5. 示例
### 5.1 用greenlet执行一个函数

```bash
#coding:utf-8
import time
from  greenlet import greenlet

def eat():
    print('魔降风云变 is eating')
    time.sleep(0.5)
    print('魔降风云变 finished eat')

def sleep():
    print('小马过河 is sleeping')
    time.sleep(0.5)
    print('小马过河 finished sleep')
g1 = greenlet(eat)
g1.switch()
```
--------------结果：
```bash
魔降风云变 is eating
魔降风云变 finished eat
```


**在一个任务函数中切换到另一个任务函数去执行，然后没有再切换回来**

```bash
#coding:utf-8
import time
from  greenlet import greenlet
def eat():
    print('魔降风云变 is eating')
    g2.switch()
    time.sleep(0.5)
    print('魔降风云变 finished eat')

def sleep():
    print('小马过河 is sleeping')
    time.sleep(0.5)
    print('小马过河 finished sleep')
g1 = greenlet(eat)
g2 = greenlet(sleep)
g1.switch()
```

--------------结果：

```bash
魔降风云变 is eating
小马过河 is sleeping
小马过河 finished sleep
```

 **一个任务中切换到另一个任务，另一个任务执行完了再执行切换回到这个任务执行。实现两个任务间切换并都执行结束**

```python
#coding:utf-8
import time
from  greenlet import greenlet
def eat():
    print('魔降风云变 is eating')
    g2.switch()
    time.sleep(0.5)
    print('魔降风云变 finished eat')

def sleep():
    print('小马过河 is sleeping')
    time.sleep(0.5)
    print('小马过河 finished sleep')
    g1.switch()
g1 = greenlet(eat)
g2 = greenlet(sleep)
g1.switch()
```

------------结果：

```python
魔降风云变 is eating
小马过河 is sleeping
小马过河 finished sleep
魔降风云变 finished eat
```
### 5.2 创建协程任务

```bash
``python
#coding:utf-8
import time
import gevent
def eat():
    print('魔降风云变 is eating')
    time.sleep(1)
    print('魔降风云变  finished eat')
def sleep():
    print('小马过河 is sleeping')
    time.sleep(1)
    print('小马过河 finished sleep')
g1 = gevent.spawn(eat)  # 创造一个协程任务
```
-----------结果:没有输出
协程遇到阻塞才切换，这里代码从上到下执行结束，没有遇到阻塞

```python
import time
import gevent
def eat():
    print('魔降风云变 is eating')
    time.sleep(1)
    print('魔降风云变  finished eat')
def sleep():
    print('小马过河 is sleeping')
    time.sleep(1)
    print('小马过河 finished sleep')
g1 = gevent.spawn(eat)  # 创造一个协程任务
gevent.sleep(2) #加个gevent.sleep(2)就切换到eat执行里面的代码了
```

---------结果：

```python
魔降风云变 is eating
魔降风云变  finished eat
```

#加个gevent.sleep(2)就切换到eat执行里面的代码了。eat中间time.sleep(1)照样睡。



### 5.3 同步和异步执行
两个子任务之间的 切换也就是上下文切换。
创建Greenlets,gevent对Greenlet初始化提供了一些封装.
第一个
```bash
import gevent
def test1():
  print 12
  gevent.sleep(0)
  print 34
def test2():
  print 56
  gevent.sleep(0)
  print 78
gevent.joinall([
  gevent.spawn(test1),
  gevent.spawn(test2),
])
```

```bash
$ python gevent2.py
12
56
34
78
```
第二个
```bash
import gevent
from gevent import Greenlet
def foo(message, n):
    gevent.sleep(n)
    print(message)
    thread1 = Greenlet.spawn(foo, "Hello", 1)
    thread2 = gevent.spawn(foo, "I live!", 2)
    thread3 = gevent.spawn(lambda x: (x+1), 2)
    threads = [thread1, thread2, thread3]
    gevent.joinall(threads)
```
第二个

```bash
$ python gevent3.py
Hello
I live!
```

### 5.4 同步vs异步

```bash
import gevent
import random
def task(pid):
    gevent.sleep(random.randint(0,2)*0.001)
    print('Task %s done' % pid)
def synchronous():
    for i in xrange(5):
        task(i)
def asynchronous():
    threads = [gevent.spawn(task, i) for i in xrange(5)]
    gevent.joinall(threads)
    print('Synchronous:')
    synchronous()
    print('Asynchronous:')
    asynchronous()
```

```bash
$ python gevent3.py
Synchronous:
Task 0 done
Task 1 done
Task 2 done
Task 3 done
Task 4 done
Asynchronous:
Task 2 done
Task 0 done
Task 1 done
Task 3 done
Task 4 done
```
参考：

 - [python协程详解，gevent asyncio ](https://www.cnblogs.com/machangwei-8/p/10907572.html#_label3_0)
 - [Gevent简明教程](https://softlns.github.io/2015/11/28/python-gevent/)
 - [详解python之协程gevent模块](http://codingdict.com/blog/article/2019/7/11/3560.html)
 - [gevent For the Working Python Developer](https://sdiehl.github.io/gevent-tutorial/)
 - [What the heck is gevent? (Part 1 of 4)](https://eng.lyft.com/what-the-heck-is-gevent-4e87db98a8)
