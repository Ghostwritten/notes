## format函数介绍

Python2.6 开始，新增了一种格式化字符串的函数 str.format()，它增强了字符串格式化的功能。

 - 基本语法是通过 {} 和 : 来代替以前的 % 。
 - format 函数可以接受不限个参数，位置可以不按顺序。
 - 字符串的参数使用{NUM}进行表示,0, 表示第一个参数,1, 表示第二个参数, 以后顺次递加;
 - 使用":", 指定代表元素需要的操作, 如":.3"小数点三位, ":8"占8个字符空间等;

还可以添加特定的字母, 如:

```bash
'b' - 二进制. 将数字以2为基数进行输出.
'c' - 字符. 在打印之前将整数转换成对应的Unicode字符串.
'd' - 十进制整数. 将数字以10为基数进行输出.
'o' - 八进制. 将数字以8为基数进行输出. 
'x' - 十六进制. 将数字以16为基数进行输出, 9以上的位数用小写字母.
'e' - 幂符号. 用科学计数法打印数字, 用'e'表示幂. 
'g' - 一般格式. 将数值以fixed-point格式输出. 当数值特别大的时候, 用幂形式打印. 
'n' - 数字. 当值为整数时和'd'相同, 值为浮点数时和'g'相同. 不同的是它会根据区域设置插入数字分隔符. 
'%' - 百分数. 将数值乘以100然后以fixed-point('f')格式打印, 值后面会有一个百分号. 
```

## format 格式转换

```bash
数字	     格式	   输出	    描述
3.1415926	{:.2f}	  3.14	    保留小数点后两位
3.1415926	{:+.2f}	  3.14	    带符号保留小数点后两位  
-1	        {:+.2f}	  -1	    带符号保留小数点后两位
2.71828	    {:.0f}	  3	        不带小数
1000000	    {:,}	  1,000,000	以逗号分隔的数字格式
0.25	    {:.2%}	  25.00%	百分比格式
1000000000	{:.2e}	  1.00E+09	指数记法
25	        {0:b}	  11001	转换成二进制
25	        {0:d}	  25	转换成十进制
25	        {0:o}	  31	转换成八进制
25	        {0:x}	  19	转换成十六进制
5	        {:0>2}    05	数字补零(填充左边, 宽度为2)
5	        {:x<4}	  5xxx	数字补x (填充右边, 宽度为4)
10	        {:x^4}	  x10x	数字补x (填充两边,优先左边, 宽度为4)
13	        {:10}	  13	右对齐 (默认, 宽度为10)
13	        {:<10}	  13	左对齐 (宽度为10)
13	        {:^10}	  13	中间对齐 (宽度为10)
```

## format实战
### 1.位置填充字符串

```c
print('hello {0} i am {1}'.format('world','python'))    # 输入结果：hello world i am python
print('hello {} i am {}'.format('world','python') )     #输入结果：hello world i am python
print('hello {0} i am {1} . a now language--{1}'.format('world','python')# 输出结果：hello world i am python . a now language-- python
```

```c
[root@localhost ~]# cat test.py
#!/usr/bin/python
#coding:utf-8

age = 25
name = 'Caroline'
 
print('{0} is {1} years old. '.format(name, age)) #输出参数
print('{0} is a girl. '.format(name))
print('{0:.3} is a decimal. '.format(1/3)) #小数点后三位
print('{0:_^11} is a 11 length. '.format(name)) #使用_补齐空位
print('{first} is as {second}. '.format(first=name, second='Wendy')) #别名替换
print('My name is {0.name}'.format(open('out.txt', 'w'))) #调用方法
print('My name is {0:8}.'.format('Fred')) #指定宽度
[root@localhost ~]# python3.8 test.py
Caroline is 25 years old. 
Caroline is a girl. 
0.333 is a decimal. 
_Caroline__ is a 11 length. 
Caroline is as Wendy. 
My name is out.txt
My name is Fred    .
```

foramt会把参数按位置顺序来填充到字符串中，第一个参数是0，然后1 ……
也可以不输入数字，这样也会按顺序来填充
同一个参数可以填充多次，这个是format比%先进的地方

### 2.key来填充

```c
obj = 'world'
name = 'python'
print('hello, {obj} ,i am {name}'.format(obj = obj,name = name)) # 输入结果：hello, world ,i am python
```
### 3.列表填充

```c
list=['world','python']
print('hello {names[0]}  i am {names[1]}'.format(names=list))  # 输出结果：hello world  i am python
print('hello {0[0]}  i am {0[1]}'.format(list))   #输出结果：hello world  i am python
```
### 4.字典填充

```c
dict={‘obj’:’world’,’name’:’python’}
print(‘hello {names[obj]} i am {names[name]}’.format(names=dict)) # hello world i am python
```
**注意**：访问字典的key，不用引号

### 5.类的属性填充

```c
class Names():
    obj='world'
    name='python'

print('hello {names.obj} i am {names.name}'.format(names=Names))#输入结果hello world i am python
```
### 6.使用魔法参数

```c
args = [‘,’,’inx’]
kwargs = {‘obj’: ‘world’, ‘name’: ‘python’}
print(‘hello {obj} {} i am {name}’.format(*args, **kwargs))#输入结果：hello world , i am python
```

**注意**：魔法参数跟你函数中使用的性质是一样的：这里format(*args, **kwargs)) 等价于：format(‘,’,’inx’,obj = ‘world’,name = ‘python’)
### 7.转义“{}”

```c
print('{{hello}} {{{0}}}'.format('world')) #输出结果：{hello} {world}
```
跟%中%%转义%一样，format中用 { 来转义{ ，用 } 来转义 }

### 8.format作为函数变量

```c
name = 'InX'
hello = 'hello,{} welcome to python world!!!'.format #定义一个问候函数
hello(name) #输入结果：hello,inx welcome to python world!!!
```
### 9.格式化datetime

```c
from datetime import datetime
now=datetime.now()
print '{:%Y-%m-%d %X}'.format(now) # 输出结果：2017-07-24 16:51:42
```
### 10.{}内嵌{}

```c
print('hello {0:>{1}} '.format('world',10)) ##输出结果：hello      world 
```
从最内层的{}开始格式化

### 11.叹号的用法

```c
print('{!s}国'.format('中')) #输出结果：中国
```

！后面可以加s r a 分别对应str() repr() ascii() 作用是在填充前先用对应的函数来处理参数
差别就是repr带有引号，str()是面向用户的，目的是可读性，repr()是面向Python解析器的，返回值表示在python内部的含义,ascii (),返回ascii编码


参考链接：
[https://blog.csdn.net/u014770372/article/details/76021988](https://blog.csdn.net/u014770372/article/details/76021988)
