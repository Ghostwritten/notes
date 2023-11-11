#  python 模块 threading 多线程



-----
## 1. 简介
多线程类似于同时执行多个不同程序，多线程运行有如下优点：

 - 使用线程可以把占据长时间的程序中的任务放到后台去处理。
 - 用户界面可以更加吸引人，这样比如用户点击了一个按钮去触发某些事件的处理，可以弹出一个进度条来显示处理的进度
 - 程序的运行速度可能加快
 - 在一些等待的任务实现上如用户输入、文件读写和网络收发数据等，线程就比较有用了。在这种情况下我们可以释放一些珍贵的资源如内存占用等等。

线程在执行过程中与进程还是有区别的。每个独立的进程有一个程序运行的入口、顺序执行序列和程序的出口。但是线程不能够独立执行，必须依存在应用程序中，由应用程序提供多个线程执行控制。

每个线程都有他自己的一组CPU寄存器，称为线程的上下文，该上下文反映了线程上次运行该线程的CPU寄存器的状态。

指令指针和堆栈指针寄存器是线程上下文中两个最重要的寄存器，线程总是在进程得到上下文中运行的，这些地址都用于标志拥有线程的进程地址空间中的内存。

 - 线程可以被抢占（中断）。
 - 在其他线程正在运行时，线程可以暂时搁置（也称为睡眠） -- 这就是线程的退让。

## 2. 方法与函数
线程模块
Python通过两个标准库thread和[threading](https://docs.python.org/3/library/threading.html)提供对线程的支持。thread提供了低级别的、原始的线程以及一个简单的锁。
threading 模块提供的其他方法：

 - threading.currentThread(): 返回当前的线程变量。
 - threading.enumerate():返回一个包含正在运行的线程的list。正在运行指线程启动后、结束前，不包括启动前和终止后的线程。
 - threading.activeCount():返回正在运行的线程数量，与len(threading.enumerate())有相同的结果。

除了使用方法外，线程模块同样提供了Thread类来处理线程，Thread类提供了以下方法:

 - run(): 用以表示线程活动的方法。
 - start():启动线程活动。
 - join([time]): 等待至线程中止。这阻塞调用线程直至线程的join()方法被调用中止-正常退出或者抛出未处理的异常-或者是可选的超时发生。
 - isAlive(): 返回线程是否活动的。
 - getName(): 返回线程名。
 - setName(): 设置线程名。

## 3. 示例
创建 Thread 对象有 2 种手段。

 - 直接创建 Thread ，将一个 callable 对象从类的构造器传递进去，这个 callable 就是回调函数，用来处理任务。
 - 编写一个自定义类继承 Thread，然后复写 run() 方法，在 run() 方法中编写任务处理代码，然后创建这个 Thread的子类。


###  3.1 直接创建 Thread 对象。
格式
```bash
class threading.Thread(group=None, target=None, name=None, args=(), kwargs={}, *, daemon=None)
```
test1

```bash
import threading
import time

def test():

    for i in range(5):
        print('test ',i)
        time.sleep(1)


thread = threading.Thread(target=test)
thread.start()

for i in range(5):
    print('main ', i)
    time.sleep(1)
```

```bash
$ python test1.py
test  0
main  0
main  1
test  1
test  2
main  2
test  3
main  3
test  4
main  4
```
### 3.2 Thread 设置名字
每一个 Thread 都有一个 name 的属性，代表的就是线程的名字，这个可以在构造方法中赋值。
如果在构造方法中没有个 name 赋值的话，默认就是 “Thread-N” 的形式，N 是数字。

## 4. 实战
### 4.1 多个函数同时执行(多进程的方法，并发)

```python
#!/usr/bin/python
#coding=utf-8
import time
import threading

def fun1(a):
    print(a)

def fun2():
    print(222)

threads = []
threads.append(threading.Thread(target=fun1,args=(u'hello world',)))
threads.append(threading.Thread(target=fun2))
print(threads)
if __name__ == '__main__':
   for t in threads:
       t.setDaemon(True)
       t.start()
```
执行：

```python
root@master:~# python3 test2.py
[<Thread(Thread-1, initial)>, <Thread(Thread-2, initial)>]
hello world
222
```
### 4.2 在class中创建线程

```bash
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
import threading
import time
 
exitFlag = 0
 
class myThread (threading.Thread):   #继承父类threading.Thread
    def __init__(self, threadID, name, counter):
        threading.Thread.__init__(self)
        self.threadID = threadID
        self.name = name
        self.counter = counter
    def run(self):                   #把要执行的代码写到run函数里面 线程在创建后会直接运行run函数 
        print "Starting " + self.name
        print_time(self.name, self.counter, 5)
        print "Exiting " + self.name
 
def print_time(threadName, delay, counter):
    while counter:
        if exitFlag:
            (threading.Thread).exit()
        time.sleep(delay)
        print "%s: %s" % (threadName, time.ctime(time.time()))
        counter -= 1
 
# 创建新线程
thread1 = myThread(1, "Thread-1", 1)
thread2 = myThread(2, "Thread-2", 2)
 
# 开启线程
thread1.start()
thread2.start()
 
print "Exiting Main Thread"
```

```bash
$ python test4.py 
Starting Thread-1
 Starting Thread-2
Exiting Main Thread
Thread-1: Sat Oct 10 16:20:59 2020
Thread-1: Sat Oct 10 16:21:00 2020
Thread-2: Sat Oct 10 16:21:00 2020
Thread-1: Sat Oct 10 16:21:01 2020
Thread-1: Sat Oct 10 16:21:02 2020
Thread-2: Sat Oct 10 16:21:02 2020
Thread-1: Sat Oct 10 16:21:03 2020
Exiting Thread-1
Thread-2: Sat Oct 10 16:21:04 2020
Thread-2: Sat Oct 10 16:21:06 2020
Thread-2: Sat Oct 10 16:21:08 2020
Exiting Thread-2
```

参考：

 - [An Intro to Threading in Python](https://realpython.com/intro-to-python-threading/)
 - [Python - Multithreaded Programming](https://www.tutorialspoint.com/python/python_multithreading.htm)
 - [Multithreading in Python | Set 1](https://www.geeksforgeeks.org/multithreading-python-set-1/)
 - [Multiprocessing vs. Threading in Python: What Every Data Scientist Needs to Know](https://blog.floydhub.com/multiprocessing-vs-threading-in-python-what-every-data-scientist-needs-to-know/)

