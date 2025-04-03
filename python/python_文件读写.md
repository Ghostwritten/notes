# python 文件读写


##  1. 打开文件
读写文件是最常见的IO操作。Python内置了读写文件的函数，方便了文件的IO操作。文件读写之前需要打开文件，确定文件的读写模式。open函数用来打开文件，
语法如下：

```bash
open(name[.mode[.buffering]])
```
open函数使用一个文件名作为唯一的强制参数，然后返回一个文件对象。模式（mode）和缓冲区（buffering）参数都是可选的，默认模式是读模式，默认缓冲区是无。

假设有个名为qiye.txt的文本文件，其存储路径是c：\text（或者是在Linux下的~/text），那么可以像下面这样打开文件。在交互式环境的提示符“>>>”下，输入如下内容：

```bash
>>> f = open(r'c:\text\qiye.txt')
```
如果文件不存在，将会看到一个类似下面的异常回溯：

```bash
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
IOError: [Errno 2] No such file or directory: 'C:\\qiye.txt'
```

## 2. 文件模式
下面主要说一下open函数中的mode参数（如表1-1所示），通过改变mode参数可以实现对文件的不同操作。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/99c6fc1389cb87b920a5c4a7adecaac7.png)

这里主要是提醒一下‘b’参数的使用，一般处理文本文件时，是用不到‘b’参数的，但处理一些其他类型的文件（二进制文件），比如mp3音乐或者图像，那么应该在模式参数中增加‘b’，这在爬虫中处理媒体文件很常用。参
数‘rb’可以用来读取一个二进制文件。

## 3. 文件缓冲区
open函数中第三个可选参数buffering控制着文件的缓冲。如果参数是0，I/O操作就是无缓冲的，直接将数据写到硬盘上；如果参数是1，I/O操作就是有缓冲的，数据先写到内存里，只有使用`flush`函数或者`close`函数才会将数据更新到硬盘；如果参数为大于1的数字则代表缓冲区的大小（单位是字节），-1（或者是任何负数）代表使用默认缓冲区的大小。

## 4. 文件读取
文件读取主要是分为按字节读取和按行进行读取，经常用到的方法有read（）、readlines()、close()。
在“>>>”输入`f=open（r‘c：\text\qiye.txt’）`后，如果成功打开文本文件，接下来调用`read（）`方法则可以一次性将文件内容全部读到内存中，最后返回的是str类型的对象：

```bash
>> f.read()
"qiye"
```
最后一步调用close（），可以关闭对文件的引用。文件使用完毕后必须关闭，因为文件对象会占用操作系统资源，影响系统的IO操作。

```bash
>>> f.close()
```
由于文件操作可能会出现IO异常，一旦出现IO异常，后面的close（）方法就不会调用。所以为了保证程序的健壮性，我们需要使用`try...finally`来实现

```bash
try:
   f = open(r'c:\text\qiye.txt','r')
   print f.read()
finally:
   if f:
      f.close()
```
上面的代码略长，Python提供了一种简单的写法，使用`with`语句来替代`try...finally`代码块和`close（）`方法，如下所示：

```bash
with open(r'c:\text\qiye.txt','r') as fileReader:
  print fileReader.read()
```
调用`read（）`一次将文件内容读到内存，但是如果文件过大，将会出现内存不足的问题。一般对于大文件，可以反复调用`read（size）`方法，一次最多读取size个字节。如果文件是文本文件，Python提供了更加合理的做法，调用`readline（）`可以每次读取一行内容，调用`readlines（）`一次读取所有内容并按行返回列表。大家可以根据自己的具体需求采取不同的读取方式，例如小文件可以直接采取read（）方法读到内存，大文件更加安全的方式是连续调用read（size），而对于配置文件等文本文件，使用readline（）方法更加合理。

将上面的代码进行修改，采用readline（）的方式实现如下所示：

```bash
with open(r'c:\text\qiye.txt','r') as fileReader:
    for line in fileReader.readlines():
        print line.strip()
```

## 5. 文件写入
写文件和读文件是一样的，唯一的区别是在调用open方法时，传入标识符`‘w’`或者`‘wb’`表示写入文本文件或者写入二进制文件，示例如下：

```bash
f = open(r'c:\text\qiye.txt','w')
f.write('qiye')
f.close()
```

我们可以反复调用write（）方法写入文件，最后必须使用`close（）`方法来关闭文件。使用write（）方法的时候，操作系统不是立即将数据写入文件中的，而是先写入内存中缓存起来，等到空闲时候再写入文件中，最后使用close（）方法就将数据完整地写入文件中了。当然也可以使用f.flush（）方法，不断将数据立即写入文件中，最后使用`close（）`方法来关闭文件。和读文件同样道理，文件操作中可能会出现IO异常，所以还是推荐使用with语句：

```bash
with open(r'c:\text\qiye.txt','w') as fileWriter:
fileWriter.write('qiye')
```

参考：
- 《python 爬虫开发与项目实战》

