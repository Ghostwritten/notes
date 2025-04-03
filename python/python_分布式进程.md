

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0a705ac0962a31c0e13a741c594cfdd4.png)

##  简介
分布式进程指的是将Process进程分布到多台机器上，充分利用多台机器的性能完成复杂的任务。我们可以将这一点应用到分布式爬虫的开发中。

分布式进程在Python中依然要用到 [multiprocessing](https://blog.csdn.net/xixihahalelehehe/article/details/127552253) 模块。multiprocessing模块不但支持多进程，其中`managers`子模块还支持把多进程分布到多台机器上。可以写一个服务进程作为调度者，将任务分布到其他多个进程中，依靠网络通信进行管理。举个例子：在做爬虫程序时，常常会遇到这样的场景，我们想抓取某个网站
的所有图片，如果使用多进程的话，一般是一个进程负责抓取图片的链接地址，将链接地址存放到`Queue`中，另外的进程负责从`Queue`中读取链接地址进行下载和存储到本地。现在把这个过程做成分布式，一台机器上的进程负责抓取链接，其他机器上的进程负责下载存储。那么遇到的主要问题是将Queue暴露到网络中，让其他机器进程都可以访问，分布式进程就是将这一个过程进行了封装，我们可以将这个过程称为本地队列的网络化。整体过程如图1-24所示。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/43882c9adf9adff12c71840f244c327f.png)
要实现上面例子的功能，创建分布式进程需要分为六个步骤：
1. 建立队列`Queue`，用来进行进程间的通信。服务进程创建任务队列`task_queue`，用来作为传递任务给任务进程的通道；服务进程创建结果队列`result_queue`，作为任务进程完成任务后回复服务进程的通道。在分布式多进程环
境下，必须通过由`Queuemanager`获得的Queue接口来添加任务。

2. 把第一步中建立的队列在网络上注册，暴露给其他进程（主机），注册后获得网络队列，相当于本地队列的映像。
3. 建立一个对象（Queuemanager（BaseManager））实例`manager`，绑定端口和验证口令。
4. 启动第三步中建立的实例，即启动管理`manager`，监管信息通道。
5. 通过管理实例的方法获得通过网络访问的`Queue`对象，即再把网络队列实体化成可以使用的本地队列。
6. 创建任务到“本地”队列中，自动上传任务到网络队列中，分配给任务进程进行处理。接下来通过程序实现上面的例子（Linux版），首先编写的是服务进程（`taskManager.py`），代码如下：

```bash
import random,time,Queue
from multiprocessing.managers import BaseManager
# 第一步：建立task_queue和result_queue，用来存放任务和结果
task_queue=Queue.Queue()
result_queue=Queue.Queue()
class Queuemanager(BaseManager):
  pass
# 第二步：把创建的两个队列注册在网络上，利用register方法，callable参数关联了Queue对象，
# 将Queue对象在网络中暴露

Queuemanager.register('get_task_queue',callable=lambda:task_queue)
Queuemanager.register('get_result_queue',callable=lambda:result_queue)
# 第三步：绑定端口8001，设置验证口令‘qiye’。这个相当于对象的初始化
manager=Queuemanager(address=('',8001),authkey='qiye')
# 第四步：启动管理，监听信息通道
manager.start()
# 第五步：通过管理实例的方法获得通过网络访问的Queue对象
task=manager.get_task_queue()
result=manager.get_result_queue()
# 第六步：添加任务
for url in ["ImageUrl_"+i for i in range(10)]:
  print 'put task %s ...' %url
  task.put(url)
# 获取返回结果
print 'try get result...'
for i in range(10):
  print 'result is %s' %result.get(timeout=10)
# 关闭管理
manager.shutdown()
```
任务进程已经编写完成，接下来编写任务进程（`taskWorker.py`），创建任务进程的步骤相对较少，需要四个步骤：
1. 使用`QueueManager`注册用于获取`Queue`的方法名称，任务进程只能通过名称来在网络上获取`Queue`。
2. 连接服务器，端口和验证口令注意保持与服务进程中完全一致。
3. 从网络上获取`Queue`，进行本地化。
4. 从`task`队列获取任务，并把结果写入`result`队列

程序`taskWorker.py`代码（win/linux版）如下：

```bash
# coding:utf-8
import time
from multiprocessing.managers import BaseManager
# 创建类似的QueueManager:
class QueueManager(BaseManager):
  pass
# 第一步：使用QueueManager注册用于获取Queue的方法名称
QueueManager.register('get_task_queue')
QueueManager.register('get_result_queue')
# 第二步：连接到服务器:
server_addr = '127.0.0.1'
print('Connect to server %s...' % server_addr)
# 端口和验证口令注意保持与服务进程完全一致:
m = QueueManager(address=(server_addr, 8001), authkey='qiye')
# 从网络连接:
m.connect()
# 第三步：获取Queue的对象:
task = m.get_task_queue()
result = m.get_result_queue()
# 第四步：从task队列获取任务,并把结果写入result队列:
while(not task.empty()):
  image_url = task.get(True,timeout=5)
  print('run task download %s...' % image_url)
  time.sleep(1)
  result.put('%s--->success'%image_url)
# 处理结束:
print('worker exit.')
```
最后开始运行程序，先启动服务进程`taskManager.py`，运行结果如下：

```bash
put task ImageUrl_0 ...
put task ImageUrl_1 ...
put task ImageUrl_2 ...
put task ImageUrl_3 ...
put task ImageUrl_4 ...
put task ImageUrl_5 ...
put task ImageUrl_6 ...
put task ImageUrl_7 ...
put task ImageUrl_8 ...
put task ImageUrl_9 ...
try get result...
```
接着再启动任务进程taskWorker.py，运行结果如下：

```bash
Connect to server 127.0.0.1...
run task download ImageUrl_0...
run task download ImageUrl_1...
run task download ImageUrl_2...
run task download ImageUrl_3...
run task download ImageUrl_4...
run task download ImageUrl_5...
run task download ImageUrl_6...
run task download ImageUrl_7...
run task download ImageUrl_8...
run task download ImageUrl_9...
worker exit.
```
当任务进程运行结束后，服务进程运行结果如下：

```bash
result is ImageUrl_0--->success
result is ImageUrl_1--->success
result is ImageUrl_2--->success
result is ImageUrl_3--->success
result is ImageUrl_4--->success
result is ImageUrl_5--->success
result is ImageUrl_6--->success
result is ImageUrl_7--->success
result is ImageUrl_8--->success
result is ImageUrl_9--->success
```
其实这就是一个简单但真正的分布式计算，把代码稍加改造，启动多个worker，就可以把任务分布到几台甚至几十台机器上，实现大规模的分布式爬虫。

> 注意 由于平台的特性，创建服务进程的代码在`Linux`和`Windows`上有一些不同，创建工作进程的代码是一致的。

`taskManager.py`程序在`Windows`版下的代码如下：
```bash
# coding:utf-8
# taskManager.py for windows
import queue
from multiprocessing.managers import BaseManager
from multiprocessing import freeze_support
# 任务个数
task_number = 10
# 定义收发队列
task_queue = queue.Queue(task_number);
result_queue = queue.Queue(task_number);
def get_task():
  return task_queue

def get_result():
  return result_queue
# 创建类似的QueueManager:
class QueueManager(BaseManager):
  pass

def win_run():
  # Windows下绑定调用接口不能使用lambda，所以只能先定义函数再绑定
  QueueManager.register('get_task_queue',callable = get_task)
  QueueManager.register('get_result_queue',callable = get_result)
  # 绑定端口并设置验证口令，Windows下需要填写IP地址，Linux下不填默认为本地
  manager = QueueManager(address = ('127.0.0.1',8001))
  # 启动
  manager.start()
  try:
  # 通过网络获取任务队列和结果队列
    task = manager.get_task_queue()
    result = manager.get_result_queue()
    # 添加任务
    for url in ["ImageUrl_"+str(i) for i in range(10)]:
      print('put task %s ...' %url)
      task.put(url)
    print('try get result...')
    for i in range(10):
      print('result is %s' %result.get(timeout=10))
  except:
    print('Manager error')
  finally:
    # 一定要关闭，否则会报管道未关闭的错误
    manager.shutdown()

if __name__ == '__main__':
# Windows下多进程可能会有问题，添加这句可以缓解
  freeze_support()
  win_run()

```
输出

```bash
$ python taskManager_win.py 
put task ImageUrl_0 ...
put task ImageUrl_1 ...
put task ImageUrl_2 ...
put task ImageUrl_3 ...
put task ImageUrl_4 ...
put task ImageUrl_5 ...
put task ImageUrl_6 ...
put task ImageUrl_7 ...
put task ImageUrl_8 ...
put task ImageUrl_9 ...
try get result...
```

