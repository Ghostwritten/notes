#  python open 文本操作函数



---
## 1. 背景
IO在计算机中指的是Input/Output，也就是输入输出。凡是用到数据交换的地 方，都会涉及IO编程，例如磁盘、网络的数据传输。在IO编程中，Stream（流）是 一种重要的概念，分为输入流（Input Stream）和输出流（Output Stream）。我 们可以把流理解为一个水管，数据相当于水管中的水，但是只能单向流动，所以数 据传输过程中需要架设两个水管，一个负责输入，一个负责输出，这样读写就可以 实现同步。

## 2. 文件读写

### 2.1 打开文件
文件读写之前需要打开文件，确定文件的读写模式。open函数用来打开文件， 语法如下：
 

```bash
open(name[.mode[.buffering]])
```

open函数使用一个文件名作为唯一的强制参数，然后返回一个文件对象。模式 （mode）和缓冲区（buffering）参数都是可选的，默认模式是读模式，默认缓冲 区是无。
假设有个名为qiye.txt的文本文件，其存储路径是c：\text（或者是在Linux 下的~/text），那么可以像下面这样打开文件。在交互式环境的提示 符“>>>”下，输入如下内容：

```bash
 >>> f = open(r'c:\text\qiye.txt')
```

如果文件不存在，将会看到一个类似下面的异常回溯：

```bash
 Traceback (most recent call last):        File "<stdin>", line 1, in <module>     IOError: [Errno 2] No such file or directory: 'C:\\qiye.txt'
```

### 2.2 文件模式
下面主要说一下open函数中的mode参数（如表1-1所示），通过改变mode参数 可以实现对文件的不同操作。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200807235248271.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
这里主要是提醒一下‘b’参数的使用，一般处理文本文件时，是用不 到‘b’参数的，但处理一些其他类型的文件（二进制文件），比如mp3音乐或者图 像，那么应该在模式参数中增加‘b’，这在爬虫中处理媒体文件很常用。参 数‘rb’可以用来读取一个二进制文件。
示例：
`encoding='utf-8'`支持中文读写格式
```python
f = open('test1.txt','wt',encoding='utf-8')
f.write('hello world, 周董')
f.close()
f = open('test1.txt','rt',encoding='utf-8')
s = f.read()
print(s)
```

### 2.3 文件缓冲区
open函数中第三个可选参数buffering控制着文件的缓冲。如果参数是0，I/O 操作就是无缓冲的，直接将数据写到硬盘上；如果参数是1，I/O操作就是有缓冲 的，数据先写到内存里，只有使用flush函数或者close函数才会将数据更新到硬 盘；如果参数为大于1的数字则代表缓冲区的大小（单位是字节），-1（或者是任何 负数）代表使用默认缓冲区的大小。

```python
f=open(“demo.txt”,’w’,buffering=1) #先缓存至内存
f=open(“demo.txt”,’w,’,buffering=0) #直接写入磁盘
```
### 2.4 文件读取

文件读取主要是分为按字节读取和按行进行读取，经常用到的方法有 read（）、readlines（）、close（）。

优雅1
```python
  try:       
     f = open(r'c:\text\qiye.txt','r')        
     print f.read()     
 finally:        
     if f:                
        f.close()
```
优雅2
```python
 with open(r'c:\text\qiye.txt','r') as fileReader:        
        print fileReader.read()
```
调用`read（）`一次将文件内容读到内存，但是如果文件过大，将会出现内存不 足的问题。一般对于大文件，可以反复调用`read（size）`方法，一次最多读取size 个字节。如果文件是文本文件，Python提供了更加合理的做法，调用readline（） 可以每次读取一行内容，调用`readlines（）`一次读取所有内容并按行返回列表。 大家可以根据自己的具体需求采取不同的读取方式.总之：

 - 小文件可以直接采取 read（）方法读到内存，
 - 大文件更加安全的方式是连续调用read（size），
 - 而对于 配置文件等文本文件，使用readline（）方法更加合理。

```python
 with open(r'c:\text\qiye.txt','r') as fileReader:        
       for line in fileReader.readlines():                
             print line.strip()
```
### 2.5 文件写入
写文件和读文件是一样的，唯一的区别是在调用open方法时，传入标识
符‘w’或者‘wb’表示写入文本文件或者写入二进制文件，示例如下：

```python
  f = open(r'c:\text\qiye.txt','w')     
  f.write('qiye')     
  f.close()
```
我们可以反复调用write（）方法写入文件，最后必须使用close（）方法来关 闭文件。使用write（）方法的时候，操作系统不是立即将数据写入文件中的，而是 先写入内存中缓存起来，等到空闲时候再写入文件中，最后使用close（）方法就将 数据完整地写入文件中了。当然也可以使用f.flush（）方法，不断将数据立即写 入文件中，最后使用close（）方法来关闭文件。和读文件同样道理，文件操作中可 能会出现IO异常，所以还是推荐使用with语句：

```python
 with open(r'c:\text\qiye.txt','w') as fileWriter:        
       fileWriter.write('qiye')
```

