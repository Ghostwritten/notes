#  python 协程
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fb3c7d69df3c265d5b2dd8d669982be8.png)



## 1. 协程
协程 （coroutine），又称微线程，是一种用户级的轻量级线程。协程 拥有自己的寄存器上下文和栈。协程调度切换时，将寄存器上下文和栈保存到其他 地方，在切回来的时候，恢复先前保存的寄存器上下文和栈。因此协程能保留上一 次调用时的状态，每次过程重入时，就相当于进入上一次调用的状态。

在并发编程 中，协程与线程类似，每个协程表示一个执行单元，有自己的本地数据，与其他协 程共享全局数据和其他资源。协程需要用户自己来编写调度逻辑，对于CPU来说，协程其实是单线程，所以 CPU不用去考虑怎么调度、切换上下文，这就省去了CPU的切换开销，所以协程在一 定程度上又好于多线程。那么在Python中是如何实现协程的呢？	

Python通过`yield`提供了对协程的基本支持，但是不完全，而使用第三方 `gevent`库是更好的选择，gevent提供了比较完善的协程支持。gevent是一个基于 协程的Python网络函数库，使用`greenlet`在`libev`事件循环顶部提供了一个有高级 别并发性的API。主要特性有以下几点：

 - ·基于libev的快速事件循环，Linux上是`epoll`机制。
 - ·基于`greenlet`的轻量级执行单元。
 - ·API复用了Python标准库里的内容。
 - ·支持SSL的协作式`sockets`。
 - ·可通过线程池或`c-ares`实现`DNS`查询。
 - ·通过`monkey patching`功能使得第三方模块变成协作式。
 
gevent对协程的支持，本质上是greenlet在实现切换工作。greenlet工作流 程如下：假如进行访问网络的IO操作时，出现阻塞，greenlet就显式切换到另一段 没有被阻塞的代码段执行，直到原先的阻塞状况消失以后，再自动切换回原来的代 码段继续处理。因此，greenlet是一种合理安排的串行方式。

由于IO操作非常耗时，经常使程序处于等待状态，有了`gevent`为我们自动切换 协程，就保证总有greenlet在运行，而不是等待IO，这就是协程一般比多线程效率 高的原因。由于切换是在IO操作时自动完成，所以`gevent`需要修改Python自带的一 些标准库，将一些常见的阻塞，如`socket`、`select`等地方实现协程跳转，这一过程 在启动时通过monkey patch完成。下面通过一个的例子来演示gevent的使用流 程，代码如下：

```bash
from gevent import monkey; monkey.patch_all()
import gevent
import urllib.request
def run_task(url):
  print('Visit --> %s' % url)
  try:
      response = urllib.request.urlopen(url)
      data = response.read()
      print('%d bytes received from %s.' % (len(data), url))
  except Exception as e:
      print(e)

if __name__=='__main__':
  urls = ['https://github.com/','https://www.python.org/','http://www.cnblogs.com/']
  greenlets = [gevent.spawn(run_task, url) for url in urls ]
  gevent.joinall(greenlets)
```
输出：

```bash
$ python coroutine1.py 
Visit --> https://github.com/
Visit --> https://www.python.org/
Visit --> http://www.cnblogs.com/
50895 bytes received from https://www.python.org/.
75936 bytes received from http://www.cnblogs.com/.
307979 bytes received from https://github.com/.
```
以上程序主要用了gevent中的spawn方法和joinall方法。spawn方法可以看做
是用来形成协程，joinall方法就是添加这些协程任务，并且启动运行。从运行结
果来看，3个网络操作是并发执行的，而且结束顺序不同，但其实只有一个线程。

`gevent`中还提供了对池的支持。当拥有动态数量的`greenlet`需要进行并发管理
（限制并发数）时，就可以使用池，这在处理大量的网络和IO操作时是非常需要
的。接下来使用`gevent`中`pool`对象，对上面的例子进行改写，程序如下：

```bash
from gevent import monkey
monkey.patch_all()
import urllib.request
from gevent.pool import Pool

def run_task(url):
  print('Visit --> %s' % url)
  try:
    response = urllib.request.urlopen(url)
    data = response.read()
    print('%d bytes received from %s.' % (len(data), url))
  except Exception as e:
    print(e)
  return('url:%s --->finish'% url)

if __name__=='__main__':
  pool = Pool(2)
  urls = ['https://github.com/','https://www.python.org/','http://www.cnblogs.com/']
  results = pool.map(run_task,urls)
  print(results)
```
输出信息：

```bash
$ python coroutine2.py 
Visit --> https://github.com/
Visit --> https://www.python.org/
50895 bytes received from https://www.python.org/.
Visit --> http://www.cnblogs.com/
307979 bytes received from https://github.com/.
76015 bytes received from http://www.cnblogs.com/.
['url:https://github.com/ --->finish', 'url:https://www.python.org/ --->finish', 'url:http://www.cnblogs.com/ --->finish']
```
通过运行结果可以看出，Pool对象确实对协程的并发数量进行了管理，先访问
了前两个网址，当其中一个任务完成时，才会执行第三个。

参考：
- [Python爬虫开发与项目实战](https://www.aliyundrive.com/s/AnqmkH58UsD)
 - [gevent调度流程解析](https://www.cnblogs.com/xybaby/p/6370799.html)
 - [Python process, thread and coroutine](https://pythonmana.com/2020/12/20201217122757825b.html)
 - [Difference between Multi-Processing, Multi-threading and Coroutine](https://sekiro-j.github.io/post/tcp/)
 - [Choosing between Python threads vs coroutines vs processes](https://blog.vijayprasanna13.me/posts/python-threads-coroutines-processes/)
 - [python process thread coroutine](https://copyfuture.com/blogs-details/202204300024361017)
 - [Combining Coroutines with Threads and Processes](https://pymotw.com/3/asyncio/executors.html)
