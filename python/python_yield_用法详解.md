#  python yield 生成器




---
## 1. 背景

您可能听说过，带有 yield 的函数在 Python 中被称之为 generator（生成器），何谓 generator ？

我们先抛开 generator，以一个常见的编程题目来展示 yield 的概念。

## 2. 如何生成斐波那契數列
斐波那契（Fibonacci）數列是一个非常简单的递归数列，除第一个和第二个数外，任意一个数都可由前两个数相加得到。用计算机程序输出斐波那契數列的前 N 个数是一个非常简单的问题，许多初学者都可以轻易写出如下函数：

### 清单 1. 简单输出斐波那契數列前 N 个数

```bash
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
def fab(max): 
    n, a, b = 0, 0, 1 
    while n < max: 
        print b 
        a, b = b, a + b 
        n = n + 1
fab(5)
```
执行以上代码，我们可以得到如下输出：

1 
1 
2 
3 
5
结果没有问题，但有经验的开发者会指出，直接在 fab 函数中用 print 打印数字会导致该函数可复用性较差，因为 fab 函数返回 None，其他函数无法获得该函数生成的数列。

要提高 fab 函数的可复用性，最好不要直接打印出数列，而是返回一个 List。以下是 fab 函数改写后的第二个版本：

### 清单 2. 输出斐波那契數列前 N 个数第二版

实例

```bash
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
def fab(max): 
    n, a, b = 0, 0, 1 
    L = [] 
    while n < max: 
        L.append(b) 
        a, b = b, a + b 
        n = n + 1 
    return L
 
for n in fab(5): 
    print n
```
改写后的 fab 函数通过返回 List 能满足复用性的要求，但是更有经验的开发者会指出，该函数在运行中占用的内存会随着参数 max 的增大而增大，如果要控制内存占用，最好不要用 List

来保存中间结果，而是通过 `iterable` 对象来迭代。例如，在 Python2.x 中，代码：
### 清单 3. 通过 iterable 对象来迭代

```bash
for i in range(1000): pass
```

会导致生成一个 1000 个元素的 List，而代码：

```bash
for i in xrange(1000): pass
```

则不会生成一个 1000 个元素的 List，而是在每次迭代中返回下一个数值，内存空间占用很小。因为 xrange 不返回 List，而是返回一个 iterable 对象。

> 注意：python3时已经没有xrange()了，在python3中，range()就是xrange()了，你可以在python3中查看range()的类型，它已经是个<class 'range'>了，而不是一个list了，毕竟这个是需要优化的

利用 iterable 我们可以把 fab 函数改写为一个支持 iterable 的 class，以下是第三个版本的 Fab：

### 清单 4. 第三个版本

实例

```bash
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
class Fab(object): 
 
    def __init__(self, max): 
        self.max = max 
        self.n, self.a, self.b = 0, 0, 1 
 
    def __iter__(self): 
        return self 
 
    def next(self): 
        if self.n < self.max: 
            r = self.b 
            self.a, self.b = self.b, self.a + self.b 
            self.n = self.n + 1 
            return r 
        raise StopIteration()
 
for n in Fab(5): 
    print n
```

**Fab 类通过 next() 不断返回数列的下一个数，内存占用始终为常数**：

```bash
1 
1 
2 
3 
5
```
然而，使用 class 改写的这个版本，代码远远没有第一版的 fab 函数来得简洁。如果我们想要保持第一版 fab 函数的简洁性，同时又要获得 iterable 的效果，yield 就派上用场了：
### 清单 5. 使用 yield 的第四版

实例

```bash
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
def fab(max): 
    n, a, b = 0, 0, 1 
    while n < max: 
        yield b      # 使用 yield
        # print b 
        a, b = b, a + b 
        n = n + 1
 
for n in fab(5): 
    print n
```
第四个版本的 fab 和第一版相比，仅仅把 print b 改为了 yield b，就在保持简洁性的同时获得了 iterable 的效果。

调用第四版的 fab 和第二版的 fab 完全一致：

```bash
1 
1 
2 
3 
5
```

简单地讲，yield 的作用就是把一个函数变成一个 generator，带有 yield 的函数不再是一个普通函数，Python 解释器会将其视为一个 generator，调用 fab(5) 不会执行 fab 函数，而是返回一个 iterable 对象！在 for 循环执行时，每次循环都会执行 fab 函数内部的代码，执行到 yield b 时，fab 函数就返回一个迭代值，下次迭代时，代码从 yield b 的下一条语句继续执行，而函数的本地变量看起来和上次中断执行前是完全一样的，于是函数继续执行，直到再次遇到 yield。

也可以手动调用 fab(5) 的 next() 方法（因为 fab(5) 是一个 generator 对象，该对象具有 next() 方法），这样我们就可以更清楚地看到 fab 的执行流程：
清单 6. 执行流程

>>>f = fab(5) 
>>> f.next() 
1 
>>> f.next() 
1 
>>> f.next() 
2 
>>> f.next() 
3 
>>> f.next() 
5 
>>> f.next() 
Traceback (most recent call last): 
 File "<stdin>", line 1, in <module> 
StopIteration
当函数执行结束时，generator 自动抛出 StopIteration 异常，表示迭代完成。在 for 循环里，无需处理 StopIteration 异常，循环会正常结束。

我们可以得出以下结论：

一个带有 yield 的函数就是一个 generator，它和普通函数不同，生成一个 generator 看起来像函数调用，但不会执行任何函数代码，直到对其调用 next()（在 for 循环中会自动调用 next()）才开始执行。虽然执行流程仍按函数的流程执行，但每执行到一个 yield 语句就会中断，并返回一个迭代值，下次执行时从 yield 的下一个语句继续执行。看起来就好像一个函数在正常执行的过程中被 yield 中断了数次，每次中断都会通过 yield 返回当前的迭代值。

yield 的好处是显而易见的，把一个函数改写为一个 generator 就获得了迭代能力，比起用类的实例保存状态来计算下一个 next() 的值，不仅代码简洁，而且执行流程异常清晰。

如何判断一个函数是否是一个特殊的 generator 函数？可以利用 isgeneratorfunction 判断：

### 清单 7. 使用 isgeneratorfunction 判断

```bash
>>>from inspect import isgeneratorfunction 
>>> isgeneratorfunction(fab) 
True
```

要注意区分 fab 和 fab(5)，fab 是一个 generator function，而 fab(5) 是调用 fab 返回的一个 generator，好比类的定义和类的实例的区别：

### 清单 8. 类的定义和类的实例

```bash
>>>import types 
>>> isinstance(fab, types.GeneratorType) 
False 
>>> isinstance(fab(5), types.GeneratorType) 
True
```

fab 是无法迭代的，而 fab(5) 是可迭代的：

```bash
>>>from collections import Iterable 
>>> isinstance(fab, Iterable) 
False 
>>> isinstance(fab(5), Iterable) 
True
```

每次调用 fab 函数都会生成一个新的 generator 实例，各实例互不影响：

```bash
>>>f1 = fab(3) 
>>> f2 = fab(5) 
>>> print 'f1:', f1.next() 
f1: 1 
>>> print 'f2:', f2.next() 
f2: 1 
>>> print 'f1:', f1.next() 
f1: 1 
>>> print 'f2:', f2.next() 
f2: 1 
>>> print 'f1:', f1.next() 
f1: 2 
>>> print 'f2:', f2.next() 
f2: 2 
>>> print 'f2:', f2.next() 
f2: 3 
>>> print 'f2:', f2.next() 
f2: 5
```

 return 的作用
在一个 generator function 中，如果没有 return，则默认执行至函数完毕，如果在执行过程中 return，则直接抛出 StopIteration 终止迭代。

另一个例子
另一个 yield 的例子来源于文件读取。如果直接对文件对象调用 read() 方法，会导致不可预测的内存占用。好的方法是利用固定长度的缓冲区来不断读取文件内容。通过 yield，我们不再需要编写读文件的迭代类，就可以轻松实现文件读取：

### 清单 9. 另一个 yield 的例子

实例

```bash
def read_file(fpath): 
    BLOCK_SIZE = 1024 
    with open(fpath, 'rb') as f: 
        while True: 
            block = f.read(BLOCK_SIZE) 
            if block: 
                yield block 
            else: 
                return
```

以上仅仅简单介绍了 yield 的基本概念和用法，yield 在 Python 3 中还有更强大的用法，我们会在后续文章中讨论。

## 3. next停止的位置

```bash
def foo():
    print("starting...")
    while True:
        res = yield 4
        print("res:",res)
g = foo()
print(next(g))
print("*"*20)
print(next(g))
```

输出：

```bash
starting...
4
********************
res: None
4
```

解释代码运行顺序，相当于代码单步调试：

 - 1.程序开始执行以后，因为foo函数中有yield关键字，所以foo函数并不会真的执行，而是先得到一个生成器g(相当于一个对象)
 - 2.直到调用next方法，foo函数正式开始执行，先执行foo函数中的print方法，然后进入while循环
 - 3.程序遇到yield关键字，然后把yield想想成return,return了一个4之后，程序停止，并没有执行赋值给res操作，此时next(g)语句执行完成，所以输出的前两行（第一个是while上面的print的结果,第二个是return出的结果）是执行print(next(g))的结果，
 - 4.程序执行print("*"*20)，输出20个*
 - 5.又开始执行下面的print(next(g)),这个时候和上面那个差不多，不过不同的是，这个时候是从刚才那个next程序停止的地方开始执行的，也就是要执行res的赋值操作，这时候要注意，**这个时候赋值操作的右边是没有值的（因为刚才那个是return出去了，并没有给赋值操作的左边传参数），所以这个时候res赋值是None,所以接着下面的输出就是res:None,**
 - 6.程序会继续在while里执行，又一次碰到yield,这个时候同样return 出4，然后程序停止，print函数输出的4就是这次return出的4.

 

**到这里你可能就明白yield和return的关系和区别了，带yield的函数是一个生成器，而不是一个函数了，这个生成器有一个函数就是next函数，next就相当于“下一步”生成哪个数，这一次的next开始的地方是接着上一次的next停止的地方执行的，所以调用next的时候，生成器并不会从foo函数的开始执行，只是接着上一步停止的地方开始，然后遇到yield后，return出要生成的数，此步就结束。**

```bash
def foo():
    print("starting...")
    while True:
        res = yield 4
        print("res:",res)
g = foo()
print(next(g))
print("*"*20)
print(g.send(7))
```

再看一个这个生成器的send函数的例子，这个例子就把上面那个例子的最后一行换掉了，输出结果：

```bash
starting...
4
********************
res: 7
4
```

先大致说一下send函数的概念：此时你应该注意到上面那个的紫色的字，还有上面那个res的值为什么是None，这个变成了7，到底为什么，这是因为，send是发送一个参数给res的，因为上面讲到，return的时候，并没有把4赋值给res，下次执行的时候只好继续执行赋值操作，只好赋值为None了，而如果用send的话，开始执行的时候，先接着上一次（return 4之后）执行，先把7赋值给了res,然后执行next的作用，遇见下一回的yield，return出结果后结束。

 

 - 5.程序执行g.send(7)，程序会从yield关键字那一行继续向下运行，send会把7这个值赋值给res变量
 - 6.由于send方法中包含next()方法，所以程序会继续向下运行执行print方法，然后再次进入while循环
 - 7.程序执行再次遇到yield关键字，yield会返回后面的值后，程序再次暂停，直到再次调用next方法或send方法。

 

 
参考：

 - [Python yield 使用浅析](https://www.runoob.com/w3cnote/python-yield-used-analysis.html)
 - [python中yield的用法详解](https://blog.csdn.net/mieleizhi0522/article/details/82142856)

