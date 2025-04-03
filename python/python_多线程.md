#  python 多线程

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3cba108e6b09fdae30266b2e838a22a8.png)


## 1. 多线程
多线程类似于同时执行多个不同程序，多线程运行有如下优点：

 - ·可以把运行时间长的任务放到后台去处理。
 - ·用户界面可以更加吸引人，比如用户点击了一个按钮去触发某些事件的处理， 可以弹出一个进度条来显示处理的进度。
 - ·程序的运行速度可能加快。
 - ·在一些需要等待的任务实现上，如用户输入、文件读写和网络收发数据等，线
   程就比较有用了。在这种情况下我们可以释放一些珍贵的资源，如内存占用等。

Python的标准库提供了两个模块：`thread`和 [threading](https://blog.csdn.net/xixihahalelehehe/article/details/106824914)，thread是低级模 块，threading是高级模块，对thread进行了封装。绝大多数情况下，我们只需要 使用threading这个高级模块。
## 2. 用threading模块创建多线程
`threading`模块一般通过两种方式创建多线程：第一种方式是把一个函数传入 并创建`Thread实例`，然后调用start方法开始执行；第二种方式是直接从 threading.Thread继承并创建线程类，然后重写__init__方法和run方法。
首先介绍第一种方法，通过一个简单例子演示创建多线程的流程，程序如下：

```python
#coding:utf-8
import random     
import time, threading     
# 新线程执行的代码:     
def thread_run(urls):        
    print 'Current %s is running...' % threading.current_thread().name        
    for url in urls:                
        print '%s ---->>> %s' % (threading.current_thread().name,url)                
        time.sleep(random.random())        
    print '%s ended.' % threading.current_thread().name          
print '%s is running...' % threading.current_thread().name     
t1 = threading.Thread(target=thread_run, name='Thread_1',args=(['url_1','url_2','url_3'],))     
t2 = threading.Thread(target=thread_run, name='Thread_2',args=(['url_4','url_5','url_6'],))     
t1.start()     
t2.start()     
t1.join()     
t2.join()     
print '%s ended.' % threading.current_thread().name
```

```python
$ python thr1.py
MainThread is running...
Current Thread_1 is running...
Thread_1 ---->>> url_1
Current Thread_2 is running...
Thread_2 ---->>> url_4
Thread_2 ---->>> url_5
Thread_1 ---->>> url_2
Thread_1 ---->>> url_3
Thread_2 ---->>> url_6
Thread_2 ended.
Thread_1 ended.
MainThread ended.
```
第二种方式从`threading.Thread`继承创建线程类，下面将方法一的程序进行重 写，程序如下：
 
 

```python
#coding:utf-8

import random     
import threading     
import time     

class myThread(threading.Thread):        

      def __init__(self,name,urls):                
         threading.Thread.__init__(self,name=name)                
         self.urls = urls             
      def run(self):                
          print 'Current %s is running...' % threading.current_thread().name                
          for url in self.urls:                        
              print '%s ---->>> %s' % (threading.current_thread().name,url)                        
              time.sleep(random.random())                
          print '%s ended.' % threading.current_thread().name     

print '%s is running...' % threading.current_thread().name     
t1 = myThread(name='Thread_1',urls=['url_1','url_2','url_3'])     
t2 = myThread(name='Thread_2',urls=['url_4','url_5','url_6'])     
t1.start()     
t2.start()     
t1.join()     
t2.join()
```

## 3. 线程同步
如果多个线程共同对某个数据修改，则可能出现不可预料的结果，为了保证数 据的正确性，需要对多个线程进行同步。使用Thread对象的`Lock`和`RLock`可以实现 简单的线程同步，这两个对象都有`acquire`方法和`release`方法，对于那些每次只允 许一个线程操作的数据，可以将其操作放到acquire和release方法之间。
对于Lock对象而言，如果一个线程连续两次进行`acquire`操作，那么由于第一 次`acquire`之后没有`release`，第二次`acquire`将挂起线程。这会导致Lock对象永远 不会release，使得线程死锁。RLock对象允许一个线程多次对其进行acquire操 作，因为在其内部通过一个`counter`变量维护着线程acquire的次数。而且每一次的 acquire操作必须有一个release操作与之对应，在所有的release操作完成之后， 别的线程才能申请该`RLock`对象。下面通过一个简单的例子演示线程同步的过程：

```python
import threading

mylock = threading.RLock()     
num=0     
class myThread(threading.Thread):        
      def __init__(self, name):                
          threading.Thread.__init__(self,name=name)             
      def run(self):                
          global num                
          while True:                        
                mylock.acquire()                        
                print '%s locked, Number: %d'%(threading.current_thread().name, num)                        
                if num>=4:                                
                   mylock.release()                                
                   print '%s released, Number: %d'%(threading.current_thread().name, num)
                   break                        
                num+=1                        
                print '%s released, Number: %d'%(threading.current_thread().name, num)                        
                mylock.release()               

if __name__== '__main__':        
   thread1 = myThread('Thread_1')        
   thread2 = myThread('Thread_2')        
   thread1.start()        
   thread2.start()
```

```python
$ python th3.py
Thread_1 locked, Number: 0
Thread_1 released, Number: 1
Thread_1 locked, Number: 1
Thread_1 released, Number: 2
Thread_1 locked, Number: 2
Thread_1 released, Number: 3
Thread_1 locked, Number: 3
Thread_1 released, Number: 4
Thread_1 locked, Number: 4
Thread_1 released, Number: 4
Thread_2 locked, Number: 4
Thread_2 released, Number: 4
```
## 4 全局解释器锁（GIL）
在Python的原始解释器CPython中存在着GIL（Global Interpreter Lock， 全局解释器锁），因此在解释执行Python代码时，会产生互斥锁来限制线程对共享 资源的访问，直到解释器遇到I/O操作或者操作次数达到一定数目时才会释放GIL。 由于全局解释器锁的存在，在进行多线程操作的时候，不能调用多个CPU内核，只能 利用一个内核，**所以在进行CPU密集型操作的时候，不推荐使用多线程，更加倾向于 多进程。那么多线程适合什么样的应用场景呢？对于IO密集型操作，多线程可以明 显提高效率，例如Python爬虫的开发，绝大多数时间爬虫是在等待socket返回数 据，网络IO的操作延时比CPU大得多**。

参考：

 - [gevent调度流程解析](https://www.cnblogs.com/xybaby/p/6370799.html)
 - [Python process, thread and coroutine](https://pythonmana.com/2020/12/20201217122757825b.html)
 - [Difference between Multi-Processing, Multi-threading and Coroutine](https://sekiro-j.github.io/post/tcp/)
 - [Choosing between Python threads vs coroutines vs processes](https://blog.vijayprasanna13.me/posts/python-threads-coroutines-processes/)
 - [python process thread coroutine](https://copyfuture.com/blogs-details/202204300024361017)
 - [Combining Coroutines with Threads and Processes](https://pymotw.com/3/asyncio/executors.html)
