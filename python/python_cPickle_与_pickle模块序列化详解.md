#  python 模块 cPickle 与 pickle 序列化



----


## 1. 序列化
[cPickle](https://docs.python.org/3/library/pickle.html) 模块通过实现将任意 python 对象转换为一系列 Bytes 的算法来帮助我们。Pickle 模块也带有类似类型的程序。但是 2 之间的区别在于cPickle 更快并且在 C 中实现了算法。cPickle 优于 Pickle 的唯一缺点是它不允许用户从 Pickle 子类化。简而言之，我们可以得出结论，使用 cPickle 是出于对象序列化的动机。 

对象的序列化在很多高级编程语言中都有相应的实现，Python也不例外。程序
运行时，所有的变量都是在内存中的，例如在程序中声明一个dict对象，里面存储 着爬取的页面的链接、页面的标题、页面的摘要等信息：
  

```bash
   d = dict(url='index.html',title='首页',content='首页')
```

在程序运行的过程中爬取的页面的链接会不断变化，比如把url改成了 second.html，但是程序一结束或意外中断，程序中的内存变量都会被操作系统进 行回收。如果没有把修改过的url存储起来，下次运行程序的时候，url被初始化为 index.html，又是从首页开始，这是我们不愿意看到的。所以把内存中的变量变成 可存储或可传输的过程，就是序列化。
将内存中的变量序列化之后，可以把序列化后的内容写入磁盘，或者通过网络 传输到别的机器上，实现程序状态的保存和共享。反过来，把变量内容从序列化的 对象重新读取到内存，称为反序列化。
在Python中提供了两个模块：`cPickle`和`pickle`来实现序列化，前者是由C语 言编写的，效率比后者高很多，但是两个模块的功能是一样的。一般编写程序的时 候，采取的方案是先导入cPickle模块，如果此模块不存在，再导入pickle模块。 示例如下：

```python
 try:        
     import cPickle as pickle     
 except ImportError:        
     import pickle
```

pickle实现序列化主要使用的是`dumps`方法或`dump`方法。dumps方法可以将任 意对象序列化成一个str，然后可以将这个str写入文件进行保存。在Python

dumps将对象序列化为str
```python
>>> import cPickle as pickle     
>>> d = dict(url='index.html',title='首页',content='首页')     
>>> pickle.dumps(d)     
"(dp1\nS'content'\np2\nS'\\xca\\xd7\\xd2\\xb3'\np3\nsS'url'\np4\nS'index.html'\n     p5\nsS'title'\np6\ng3\ns."
```
dump写入文件

```python
 >>> f=open(r'D:\dump.txt','wb')     
 >>> pickle.dump(d,f)     >>>
 >>> f.close()
```
## 2. 反序列化
[pickle](https://docs.python.org/3/library/pickle.html)实现反序列化使用的是loads方法或load方法。把序列化后的文件从磁 盘上读取为一个str，然后使用loads方法将这个str反序列化为对象，或者直接使 用load方法将文件直接反序列化为对象，如下所示：

```python
>>> f=open(r'D:\dump.txt','rb')     
>>> d=pickle.load(f)     
>>> f.close()     
>>> d     
{'content': '\xca\xd7\xd2\xb3', 'url': 'index.html', 'title': '\xca\xd7\xd2\xb3'}
```
通过反序列化，存储为文件的dict对象，又重新恢复出来，但是这个变量和原 变量没有什么关系，只是内容一样。以上就是序列化操作的整个过程。
假如我们想在不同的编程语言之间传递对象，把对象序列化为`标准格式`是关 键，例如`XML`，但是现在更加流行的是序列化为`JSON格式`，既可以被所有的编程语 言读取解析，也可以方便地存储到磁盘或者通过网络传输。[python json文本操作请参考](https://blog.csdn.net/xixihahalelehehe/article/details/106550900)

参考：

 - [cPickle in Python Explained With Examples](https://www.pythonpool.com/cpickle/)
 - [Serializing Data Using the pickle and cPickle Modules](https://www.geeksforgeeks.org/serializing-data-using-the-pickle-and-cpickle-modules/)
 - [pickle and cPickle – Python object serialization](http://pymotw.com/2/pickle/)
 - [“python3 cpickle” Code Answer’s](https://www.codegrepper.com/code-examples/python/python3+cpickle)


