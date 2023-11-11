#  python 模块 sys




---
## 1. sys介绍
“sys”即“system”，“系统”之意。该模块提供了一些接口，用于访问 Python 解释器自身使用和维护的变量，同时模块中还提供了一部分函数，可以与解释器进行比较深度的交互。

## 2. 常用方法
### 2.1 sys.argv
“argv”即“argument value”的简写，是一个列表对象，其中存储的是在命令行调用 Python 脚本时提供的“命令行参数”。

这个列表中的第一个参数是被调用的脚本名称，也就是说，调用 Python 解释器的“命令”（python）本身并没有被加入这个列表当中。

```python
import sys

print("The list of command line arguments:\n", sys.argv)
```
输出：

```python
$ python sy1.py 
('The list of command line arguments:\n', ['sy1.py'])

$ python sy1.py arg1 arg2 arg3
('The list of command line arguments:\n', ['sy1.py', 'arg1', 'arg2', 'arg3'])
```
### 2.2 sys.platform
查看sys模块中的sys.platform属性可以得到关于运行平台更详细的信息。
win

```python
>>> import sys
>>> sys.platform
'win32'
```

在 Linux 上：

```python
>>> sys.platform
'linux'
```
比较一下os.name的结果，sys.platform的信息更加准确。

### 2.3 sys.byteorder
`“byteord2er”`即“字节序”，指的是在计算机内部存储数据时，数据的低位字节存储在存储空间中的高位还是低位。

“小端存储”时，数据的低位也存储在存储空间的低位地址中，此时sys.byteorder的值为“little”。如果不注意，在按地址顺序打印内容的时候，可能会把小端存储的内容打错。当前大部分机器都是使用的小端存储。

```powershell
>>> sys.byteorder
'little'
```
而另外还存在一种存储顺序是“大端存储”，即数据的高位字节存储在存储空间的低位地址上，此时sys.byteorder的值为“big”。

```powershell
>>> sys.byteorder
'big'
```
### 2.4 .executable
该属性是一个字符串，在正常情况下，其值是当前运行的 Python 解释器对应的可执行程序所在的绝对路径。

```powershell
>>> sys.executable
'/usr/bin/python'
```
### 2.5 sys.modules
该属性是一个字典，包含的是各种已加载的模块的模块名到模块具体位置的映射。

通过手动修改这个字典，可以重新加载某些模块；但要注意，切记不要大意删除了一些基本的项，否则可能会导致 Python 整个儿无法运行。

```python
>>> sys.modules
{'copy_reg': <module 'copy_reg' from '/usr/lib64/python2.7/copy_reg.pyc'>, 'sre_compile': <module 'sre_compile' from '/usr/lib64/python2.7/sre_compile.pyc'>
```

### 2.6 sys.builtin_module_names
该属性是一个字符串元组，其中的元素均为当前所使用的的 Python 解释器内置的模块名称。

注意区别`sys.modules`和`sys.builtin_module_names`——前者的关键字（keys）列出的是导入的模块名，而后者则是解释器内置的模块名。

```bash
sys.builtin_module_names
('__builtin__', '__main__', '_ast', '_codecs', '_sre', '_symtable', '_warnings', '_weakref', 'errno', 'exceptions', 'gc', 'imp', 'marshal', 'posix', 'pwd', 'signal', 'sys', 'thread', 'zipimport')
```
### 2.7 sys.path
该属性是一个由字符串组成的列表，其中各个元素表示的是 Python 搜索模块的路径；在程序启动期间被初始化。

其中第一个元素（也就是path[0]）的值是最初调用 Python 解释器的脚本所在的绝对路径；如果是在交互式环境下查看sys.path的值，就会得到一个空字符串。

```bash
#!/usr/bin/python
import sys
print sys.path
print sys.path[0]
print sys.path[1]
```
输出：

```python
$ python sy2.py 
['/root/python/sys', '/usr/lib/python2.7/site-packages/s3cmd-2.0.2-py2.7.egg', '/usr/lib/python2.7/site-packages/python_magic-0.4.15-py2.7.egg', '/usr/lib/python2.7/site-packages/UNKNOWN-0.0.0-py2.7.egg', '/usr/lib64/python27.zip', '/usr/lib64/python2.7', '/usr/lib64/python2.7/plat-linux2', '/usr/lib64/python2.7/lib-tk', '/usr/lib64/python2.7/lib-old', '/usr/lib64/python2.7/lib-dynload', '/usr/lib64/python2.7/site-packages', '/usr/lib64/python2.7/site-packages/gtk-2.0', '/usr/lib/python2.7/site-packages']
/root/python/sys
/usr/lib/python2.7/site-packages/s3cmd-2.0.2-py2.7.egg
```

### 2.8 sys.stdin
即 Python 的标准输入通道。通过改变这个属性为其他的类文件（file-like）对象，可以实现输入的重定向，也就是说可以用其他内容替换标准输入的内容。

所谓“标准输入”，实际上就是通过键盘输入的字符。
sys.stdin.readline() 相当于input，区别在于input不会读入'\n'
实例1
```python
import sys
print('Plase input your name: ')
name = sys.stdin.readline()
print('Hello ', name)
```

输出：

```python
Plase input your name: 
hello world   
('Hello ', 'hello world\n')
```
实例2

```python
while True:
    n = int(input('Please input a number:\n'))
    sn = list(map(int,input('Please input some numbers:\n').split()))
    print(n)
    print(sn,'\n')
```

输出:

```python
Please input a number:
34 
Please input some numbers:
34 353
34
[34, 353] 
```

### 2.9 sys.stdout
与上一个“标准输入”类似，sys.stdout则是代表“标准输出”的属性。

通过将这个属性的值修改为某个文件对象，可以将本来要打印到屏幕上的内容写入文件。
实例1
```python
import sys
# 以附加模式打开文件，若不存在则新建
with open("count_log.txt", 'a', encoding='utf-8') as f:
    sys.stdout = f
    for i in range(10):
        print("count = ", i)
```
输出：

```python
$ cat count_log.txt 
count =  0
count =  1
count =  2
count =  3
count =  4
count =  5
count =  6
count =  7
count =  8
count =  9
```

实例2：

```python
import sys

sys.stdout.write('hello'+'\n')
print 'hello'
```

```python
$ python sy7.py
hello
hello
```

### 2.10 sys.err
与前面两个属性类似，只不过该属性标识的是标准错误，通常也是定向到屏幕的，可以粗糙地认为是一个输出错误信息的特殊的标准输出流。由于性质类似，因此不做演示。

此外，sys模块中还存在几个“私有”属性：sys.__stdin__，sys.__stdout__，sys.__stderr__。这几个属性中保存的就是最初定向的“标准输入”、“标准输出”和“标准错误”流。在适当的时侯，我们可以借助这三个属性将sys.stdin、sys.stdout和sys.err恢复为初始值。

### 2.11 sys.exit(n)
功能：执行到主程序末尾，解释器自动退出，但是如果需要中途退出程序，可以调用sys.exit函数，带有一个可选的整数参数返回给调用它的程序，表示你可以在主程序中捕获对sys.exit的调用。（0是正常退出，其他为异常）

```python
#!/usr/bin/env python

import sys

def exitfunc(value):
    print value
    sys.exit(0)

print "hello"

try:
    sys.exit(1)
except SystemExit,value:
    exitfunc(value)

print "come?"
```
输出：

```python
$ python sy11.py 
hello
1
```
参考：

 - [纯洁的微笑](http://www.ityouknow.com/python/2019/10/09/python-sys-demonstration-028.html)
 - [Python 标准输出 sys.stdout重定向](https://www.cnblogs.com/turtle-fly/p/3280519.html)
 - [python 模块运用快速学习手册](https://ghostwritten.blog.csdn.net/article/details/112751406)
 - [python argparse参数配置](https://ghostwritten.blog.csdn.net/article/details/121199110)
 - [python click模块参数处理](https://ghostwritten.blog.csdn.net/article/details/106124675)
 - [geeksforgeeks Python sys Module](https://www.geeksforgeeks.org/python-sys-module/)

