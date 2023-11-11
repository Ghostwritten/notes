#  python 内置函数




## 1. 数学相关

```bash
abs(a) : 求取绝对值。abs(-1)
max(list) : 求取list最大值。max([1,2,3])
min(list) : 求取list最小值。min([1,2,3])
sum(list) : 求取list元素的和。 sum([1,2,3]) >>> 6
sorted(list) : 排序，返回排序后的list。
len(list) : list长度,len([1,2,3])
divmod(a,b): 获取商和余数。 divmod(5,2) >>> (2,1)
pow(a,b) : 获取乘方数。pow(2,3) >>> 8
round(a,b) : 获取指定位数的小数。a代表浮点数，b代表要保留的位数。round(3.1415926,2) >>> 3.14
range(a[,b]) : 生成一个a到b的数组,左闭右开。 range(1,10) >>> [1,2,3,4,5,6,7,8,9]
```

## 2. 类型转换

```bash
int(str) : 转换为int型。int('1') >>> 1
float(int/str) : 将int型或字符型转换为浮点型。float('1') >>> 1.0
str(int) : 转换为字符型。str(1) >>> '1'
bool(int) : 转换为布尔类型。 str(0) >>> False str(None) >>> False
bytes(str,code) : 接收一个字符串，与所要编码的格式，返回一个字节流类型。bytes('abc', 'utf-8') >>> b'abc' bytes(u'爬虫', 'utf-8') >>> b'\xe7\x88\xac\xe8\x99\xab'
list(iterable) : 转换为list。 list((1,2,3)) >>> [1,2,3]
iter(iterable)： 返回一个可迭代的对象。 iter([1,2,3]) >>> <list_iterator object at 0x0000000003813B00>
dict(iterable) : 转换为dict。 dict([('a', 1), ('b', 2), ('c', 3)]) >>> {'a':1, 'b':2, 'c':3}
enumerate(iterable) : 返回一个枚举对象。
tuple(iterable) : 转换为tuple。 tuple([1,2,3]) >>>(1,2,3)
set(iterable) : 转换为set。 set([1,4,2,4,3,5]) >>> {1,2,3,4,5} set({1:'a',2:'b',3:'c'}) >>> {1,2,3}
hex(int) : 转换为16进制。hex(1024) >>> '0x400'
oct(int) : 转换为8进制。 oct(1024) >>> '0o2000'
bin(int) : 转换为2进制。 bin(1024) >>> '0b10000000000'
chr(int) : 转换数字为相应ASCI码字符。 chr(65) >>> 'A'
ord(str) : 转换ASCI字符为相应的数字。 ord('A') >>> 65
```

## 3. 相关操作

```bash
eval() : 执行一个表达式，或字符串作为运算。 eval('1+1') >>> 2
exec() : 执行python语句。 exec('print("Python")') >>> Python
filter(func, iterable) : 通过判断函数fun，筛选符合条件的元素。 filter(lambda x: x>3, [1,2,3,4,5,6]) >>> <filter object at 0x0000000003813828>
map(func, *iterable) : 将func用于每个iterable对象。 map(lambda a,b: a+b, [1,2,3,4], [5,6,7]) >>> [6,8,10]
zip(*iterable) : 将iterable分组合并。返回一个zip对象。 list(zip([1,2,3],[4,5,6])) >>> [(1, 4), (2, 5), (3, 6)]
type()：返回一个对象的类型。
id()： 返回一个对象的唯一标识值。
hash(object)：返回一个对象的hash值，具有相同值的object具有相同的hash值。 hash('python') >>> 7070808359261009780
help()：调用系统内置的帮助系统。
isinstance()：判断一个对象是否为该类的一个实例。
issubclass()：判断一个类是否为另一个类的子类。
globals() : 返回当前全局变量的字典。
next(iterator[, default]) : 接收一个迭代器，返回迭代器中的数值，如果设置了default，则当迭代器中的元素遍历后，输出default内容。
reversed(sequence) ： 生成一个反转序列的迭代器。 reversed('abc') >>> ['c','b','a']
```

## 4. 内置函数
### 4.1 add()添加
```python
s=['alex', 2,3]
s2=set(s)
print(s2)
s2.add('uu')
print(s2)
output:
{3, 'alex', 2}
{3, 'alex', 2, 'uu'}
```
### 4.2 append()向列表的尾部添加一个新的元素
```python
#!/usr/bin/python
aList = [123, 'xyz', 'zara', 'abc'];
aList.append( 2009 );
print "Updated List : ", aList;

output：
Updated List :  [123, 'xyz', 'zara', 'abc', 2009]
```

### 4.3 all()判断列表是否有为空的元素

```python
print(all([1,2,3,'ef']))
True
print(all([1,2,3,'ef','']))
False
```

### 4.4 clear() 函数用于删除字典内所有元素

```python
#!/usr/bin/python

dict = {'Name': 'Zara', 'Age': 7};

print "Start Len : %d" %  len(dict)
dict.clear()
print "End Len : %d" %  len(dict)
以上实例输出结果为：

Start Len : 2
End Len : 0
```
### 4.5 del()删除列表的元素

```python
#!/usr/bin/python
 
list1 = ['physics', 'chemistry', 1997, 2000]
 
print list1
del list1[2]
print "After deleting value at index 2 : "
print list1
```
### 4.6 remove是用来移除指定值. 并且,一次只会移除一个

```python
s=['alex', 2,3]
s2=set(s)
print(s2)
s2.remove('alex')
print(s2) 	
output:
{3, 2, 'alex'}
{3, 2}
```

###  4.7 () 函数用来执行一个字符串表达式并返回表达式的值
参数

 - expression -- 表达式。
 - globals -- 变量作用域，全局命名空间，如果被提供，则必须是一个字典对象。
 - locals -- 变量作用域，局部命名空间，如果被提供，可以是任何映射对象。 返回值

```python
>>>x = 7
>>> eval( '3 * x' )
21
>>> eval('pow(2,2)')
4
>>> eval('2 + 2')
4
>>> n=81
>>> eval("n + 4")
85
```


```python
a=str({'beijing':{'1':111}})
print(type(a))
print(a)
a=eval(a)
print()
print(a['beijing'])

output:
<class 'str'>
{'beijing': {'1': 111}}

{'1': 111}
```

### 4.8 filter()过滤元组值，不能修改

```python
#!/usr/bin/python
#---coding:utf-8---
str = ['a','b','c','d']
def funl(s):
    if s != 'a':
        return s
ret = filter(funl,str)   #ret是一个迭代器
print(list(ret))

[root@localhost func]# python filter.py
['b', 'c', 'd']
```
### 4.9 flush() 刷新缓冲区
即将缓冲区中的数据立刻写入文件，同时清空缓冲区，不需要是被动的等待输出缓冲区写入。
一般情况下，文件关闭后会自动刷新缓冲区，但有时你需要在关闭前刷新它，这时就可以使用 flush() 方法。

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
# 打开文件
fo = open("runoob.txt", "wb")
print "文件名为: ", fo.name
 
# 刷新缓冲区
fo.flush()
 
# 关闭文件
fo.close()
```
### 4.10 input()和raw_input()
raw_input()随便输都是字符串，而input()必须按照Python的规则来~
raw_input()

```python
name=raw_input('输入姓名：')
age=raw_input('输入年龄')
我们输入汉字的姓名和数字的年龄
输入姓名：许嵩
输入年龄：31
许嵩 31
```
年龄的格式是`string`
 
input()

```python
name=input('输入姓名：')
age=input('输入年龄：')
我们还是输入汉字的姓名和数字的年龄
输入姓名：'许嵩'
输入年龄：31
许嵩 31
```

input()输入严格按照Python的语法，是字符就自觉的加 ' ' ，数字就是数字

**raw_input()input()区分使用
字符的时候可以用raw_input()，当然不怕麻烦也可以用input()手动加''
int类型的时候最好用input()**

```python
$ cat input1.py
#!/usr/bin/python3.8
#-----coding:utf-8---
name = input("Name:")
age = int(input('Age:'))
job = input("Job:")
salary = input('Salary:')
if salary.isdigit():  #长的像不像数字，比如200d,'200'
    salary = int(salary)
#else:
    print("must input digit")
    exit()
msg = '''
-------- info of %s -------
Name: %s
Age : %s
Job : %s
Salary: %f
You will be retired in %s years
-------- end ------------------
''' % (name, name, age, job, salary ,65-age)

print(msg)
[root@localhost func]# python3.8 input1.py
Name:liming
Age:18
Job:IT
Salary:800

-------- info of liming -------
Name: liming
Age : 18
Job : IT
Salary: 800.000000
You will be retired in 47 years
-------- end ------------------
```

### 4.11 isatty() 检测文件是否连接到一个终端设备
如果是返回 True，否则返回 False。

```python
fileObject.isatty()
```

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

# 打开文件
fo = open("runoob.txt", "wb")
print "文件名为: ", fo.name

ret = fo.isatty()
print "返回值 : ", ret

# 关闭文件
fo.close()
以上实例输出结果为：
文件名为:  runoob.txt
返回值 :  False
```

### 4.12 isinstance()与type()
`isinstance()` 函数来判断一个对象是否是一个已知的类型
`isinstance()` 与 `type()` 区别：

 - type() 不会认为子类是一种父类类型，不考虑继承关系。
 - isinstance() 会认为子类是一种父类类型，考虑继承关系。

如果要判断两个类型是否相同推荐使用 isinstance()。

以下是 isinstance() 方法的语法:

```powershell
isinstance(object, classinfo)
```

参数
object -- 实例对象。
classinfo -- 可以是直接或间接类名、基本类型或者由它们组成的元组。

```python
>>>a = 2
>>> isinstance (a,int)
True
>>> isinstance (a,str)
False
>>> isinstance (a,(str,int,list))    # 是元组中的一个返回 True
True
```

type() 与 isinstance()区别：

```python
class A:
    pass
 
class B(A):
    pass
 
isinstance(A(), A)    # returns True
type(A()) == A        # returns True
isinstance(B(), A)    # returns True
type(B()) == A        # returns False
```

对于基本类型来说 classinfo 可以是：
int，float，bool，complex，str(字符串)，list，dict(字典)，set，tuple
要注意的是，classinfo 的字符串是 str 而不是 string，字典也是简写 dict。
实例

```python
arg=123
isinstance(arg, int)    #输出True
isinstance(arg, str)    #输出False
isinstance(arg, string) #报错
```

### 4.13 lower() 方法转换字符串中所有大写字符为小写

```python
#!/usr/bin/python
str = "THIS IS STRING EXAMPLE....WOW!!!";
print str.lower();
以上实例输出结果如下：
this is string example....wow!!!
```


### 4.14 map()迭代函数以列表返回
1.对可迭代函数'iterable'中的每一个元素应用‘function’方法，将结果作为list返回
```python
$ cat map1.py
#!/usr/bin/python
def add100(x):
     return x+100
 
hh = [11,22,33]
print map(add100,hh)
[root@localhost func]# python map1.py 
[111, 122, 133]
```
2.如果给出了额外的可迭代参数，则对每个可迭代参数中的元素‘并行’的应用‘function’

```python
$ cat map2.py
#!/usr/bin/python
def abc(a, b, c):
     return a*10000 + b*100 + c
 
list1 = [11,22,33]
list2 = [44,55,66]
list3 = [77,88,99]
print map(abc,list1,list2,list3)
[root@localhost func]# python map2.py
[114477, 225588, 336699]
```
3.如果'function'给出的是‘None’，自动假定一个‘identity’函数

```python
$ cat map3.py
#!/usr/bin/python

list1 = [11,22,33]
print map(None,list1)
list1 = [11,22,33]
list2 = [44,55,66]
list3 = [77,88,99]
print map(None,list1,list2,list3)
[root@localhost func]# python map3.py
[11, 22, 33]
[(11, 44, 77), (22, 55, 88), (33, 66, 99)]
```
4.map(f, iterable)基本上等于：[f(x) for x in iterable]
```python
$ cat map4.py
#!/usr/bin/python
def add100(x):
    return x + 100

list1 = [11,22,33]
print map(add100,list1)

print [add100(i) for i in list1]
[root@localhost func]# python map4.py
[111, 122, 133]
[111, 122, 133]
```


### 4.15 reduce() 函数会对参数序列中元素进行累积
函数将一个数据集合（链表，元组等）中的所有数据进行下列操作：用传给 reduce 中的函数 function（有两个参数）先对集合中的第 1、2 个元素进行操作，得到的结果再与第三个数据用 function 函数运算，最后得到一个结果。

```python
reduce(function, iterable[, initializer])
```

参数

 - function -- 函数，有两个参数
 - iterable -- 可迭代对象
 - initializer -- 可选，初始参数

```python
from functools import reduce
def add1(x,y):
    return x + y

print(reduce(add1,range(1,10)))

output:
45
```

### 4.16 pop()函数用于移除列表中的一个元素（默认最后一个元素）并且返回该元素的值

```python
#!/usr/bin/python3
#coding=utf-8
 
list1 = ['Google', 'Runoob', 'Taobao']
list_pop=list1.pop(1)
print "删除的项为 :", list_pop
print "列表现在为 : ", list1
以上实例输出结果如下：

删除的项为 : Runoob
列表现在为 :  ['Google', 'Taobao']
```

### 4.17 pow()获取乘方数


```python
$ cat pow1.py
#!/usr/bin/python

import math

print pow(2,3)
print math.pow(2,3)
print 2**3
print 2*3
print pow(2,3,1)

$ python pow1.py
8
8.0
8
6
0
```
pow(x,y,z) 当 z 这个参数不存在时 x,y 不限制是否为 float 类型, 而当使用第三个参数的时候要保证前两个参数只能为整数。

### 4.18 seek()用于移动文件读取指针到文件指定的位置

```python
file. seek(offset[, whence])
```

 - whence：0,1,2三个参数，0表示文件开头，1表示当前位置，2表示文件结尾
 - offset:偏移量，可正可负，正数表示向后移动offset位，负数表示向前移动offset位。

```python
cat log.txt
122222222222222222
1324222222222
24242424242
gsgssgsgssxxnxnxn

sgsgsgsgs

#!/usr/bin/python


#path = 'E:\log.txt'
f = open('E:\log.txt','rb')
print(f.tell())
print(f.read(1))
print(f.tell())

f.seek(5)
print(f.tell())
print(f.read(1))

f.seek(2,1)
print(f.tell())
#print(f.read(1))

f.seek(-1,1)
print(f.tell())
print(f.read(1))

输出：
0
b'1'
1
5
b'6'
8
7
b'8'
```



### 4.19 set集合创建 
set() 函数创建一个无序不重复元素集，可进行关系测试，删除重复数据，还可以计算交集、差集、并集等。
set 语法：
class set([iterable])
参数说明：
iterable -- 可迭代对象对象；

集合的相关操作
#### 4.19.1 创建集合
由于集合没有自己的语法格式，只能通过集合的工厂方法set()和frozenset创建

#字典与列表的set   

```python
s=set('alex li')
s1= ['alvin', 'ee', 'alvin']
s2=set(s1)
print(s2,type(s2))
print(s,type(s))

output：
{'alvin', 'ee'} <class 'set'>
{'e', 'a', 'x', 'l', 'i', ' '} <class 'set'>
```

#列表中的列表不可哈希

```python
$ cat set1.py
#!/usr/bin/python
s=[[23,2], 'sgs',34]
s2=set(s)
print(s2)
[root@localhost func]# python set1.py
Traceback (most recent call last):
  File "set1.py", line 3, in <module>
    s2=set(s)
TypeError: unhashable type: 'list'
```

交集并集差集

```python
>>>x = set('runoob')
>>> y = set('google')
>>> x, y
(set(['b', 'r', 'u', 'o', 'n']), set(['e', 'o', 'g', 'l']))   # 重复的被删除
>>> x & y         # 交集
set(['o'])
>>> x | y         # 并集
set(['b', 'e', 'g', 'l', 'o', 'n', 'r', 'u'])
>>> x - y         # 差集
set(['r', 'b', 'u', 'n'])
>>>
```



#### 4.19.2 访问集合
由于集合本身是无序的，所以不能为集合创建索引或切片操作，只能循环遍历或使用in、not in 来 访问或者判断集合元素

```python
s=['alex', 2,3]
s2=set(s)
print(s2)

print(2 in s)
print(4 in s)
print('a' in s）

s2.update('12','zc')
print(s2)

s2.update([12],['wda'])
print(s2)

output;
{3, 2, 'alex'}
True
False
False
{2, 3, '2', 'c', 'alex', 'z', '1'}
{2, 3,  '2', 'c', 'alex', 'z', '1', 12, 'wda'}
```




output:



s=['alex', 2,3]
s2=set(s)
print(s2)
s2.remove('alex')
print(s2) 	
output:
{3, 2, 'alex'}
{3, 2}


**********************
4.集合类型操作符
print(set('alex')==set('alexxxxxx'))
print(set('alex')<set('alexxxxxxw'))
print(set('alex')and set('alexxxxxxw'))
print(set('alex') or set('alexxxxxxw'))

output:
True
True
{'w', 'e', 'x', 'a', 'l'}
{'e', 'l', 'x', 'a'}

*******************************
#### 4.19.3 交集与并集、差集、对称差集（反向交集）、父集

```python
#intersection（）
a = set([ 1,2,3,4,5 ])
b = set([4,5,6,7,8])
print(a.intersection(b))
print(a.union(b))
print(a.difference(b))
print(b.difference(a))
print(a.symmetric_difference(b))
print(a.issubset(b))
print(a & b)
print(a | b)
print(a - b)
print(b - a)
print(a ^ b)

output;
{4, 5}
{1, 2, 3, 4, 5, 6, 7, 8}
{1, 2, 3}
{8, 6, 7}
{1, 2, 3, 6, 7, 8}
False
```
### 4.20 strip()它返回的是字符串的副本并删除前导和后缀字符
无参数
```python
a=" \rzha ng\n\t "
print(len(a))

b=a.strip()
print(b)
print(len(b))

输出：
11
zha ng
6
```
有参数

```python
a="rrbbrrddrr"
b=a.strip("r")
print(b)

输出：bbrrdd
```
### 4.21 lstrip()与rstrip分别删除前缀与后缀
```python
$ cat strip1.py
#!/usr/bin/python
a=" zhangkang "
print(a.lstrip(),len(a.lstrip()))
print(a.rstrip(),len(a.rstrip()))

[root@localhost func]# python lstrip.py
('zhangkang ', 10)
(' zhangkang', 10)
```

### 4.22 truncate() 方法用于截断文件
如果指定了可选参数 size，则表示截断文件为 size 个字符。 如果没有指定 size，则从当前位置起截断；截断之后 size 后面的所有字符被删除。
truncate() 方法语法如下：

```python
fileObject.truncate( [ size ])
```
文件 runoob.txt 的内容如下：
```python
1:www.runoob.com
2:www.runoob.com
3:www.runoob.com
4:www.runoob.com
5:www.runoob.com
```
循环读取文件的内容：

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
# 打开文件
fo = open("runoob.txt", "r+")
print "文件名为: ", fo.name
line = fo.readline()
print "读取第一行: %s" % (line)
# 截断剩下的字符串
fo.truncate()
# 尝试再次读取数据
line = fo.readline()
print "读取数据: %s" % (line)
# 关闭文件
fo.close()

以上实例输出结果为：
文件名为:  runoob.txt
读取第一行: 1:www.runoob.com
读取数据:
```


以下实例截取 runoob.txt 文件的10个字节：

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
# 打开文件
fo = open("runoob.txt", "r+")
print "文件名为: ", fo.name
# 截取10个字节
fo.truncate(10)
str = fo.read()
print "读取数据: %s" % (str)
# 关闭文件
fo.close()
以上实例输出结果为：
文件名为:  runoob.txt
读取数据: 1:www.runo
```
### 4.23 startswith()与endswith()
startswith判断文本是否以某个或某几个字符开始;
endswith判断文本是否以某个或某几个字符结束;

str.endswith(suffix[, start[, end]])
suffix -- 该参数可以是一个字符串或者是一个元素。
start -- 字符串中的开始位置。
end -- 字符中结束位置。


```python
text = 'Happy National Day!'   
print text.startswith('A')      # False
print text.startswith('H')      # True
print text.startswith('Hap')    # True
print text.startswith('')       # True
 
print text.endswith('A')        # False
print text.endswith('!')        # True
print text.endswith('Day!')     # True
```

startswith()和endswith()函数的参数可以包在一个括号中一次列出多个，各个参数之间是或的关系：


```python
text = 'Happy National Day!'      
print text.startswith(('A','H'))   # True
print text.endswith(('y','!'))     # True
```


endswith典型的应用场景是用来判断是否是某一文件类型（图片或.exe、.sh执行文件）


```python
import os 
import cv2
 
for item in os.listdir('/home/xxx/TestImage/'):
    if item.endswith(('.jpg','.png','gif')):
        img = cv2.imread(item)
        print True
```

```python
#!/usr/bin/python
 
str = "this is string example....wow!!!";
 
suffix = "wow!!!";
print str.endswith(suffix);
print str.endswith(suffix,20);
suffix = "is";
print str.endswith(suffix, 2, 4);
print str.endswith(suffix, 2, 6);
[root@localhost func]# python startwith.py
True
True
True
False
```

### 4.24 tuple() 函数将列表转换为元组。


```python
$ cat tuple.py
#!/usr/bin/python

print tuple([1,2,3,4])
print tuple({1:2,3:4})
print tuple((1,2,3,4))  
aList = [123, 'xyz', 'zara', 'abc'];
aTuple = tuple(aList) 
print "Tuple elements : ", aTuple
[root@localhost func]# python tuple.py
(1, 2, 3, 4)
(1, 3)
(1, 2, 3, 4)
Tuple elements :  (123, 'xyz', 'zara', 'abc')
```
### 4.25 with()
一般读写文件
```python
#!/usr/bin/env python

fileReader = open('students.txt', 'r')

for row in fileReader:
    print(row.strip())

fileReader.close()
```

基本也实现了读取文件的功能。但是有的时候，上述代码在运行的时候会`抛出异常，导致无法关闭文件句柄`，这个时候，我就会加上异常处理程序，代码就改成了这样：

```python
#!/usr/bin/env python

try:
    fileReader = open('students.txt', 'r')

    for row in fileReader:
        print(row.strip())
except:
    print('Read file failed')
finally:
    fileReader.close()
```

也还好，解决了抛出一大堆异常的问题。但是上述代码怎么看都有点累赘啰嗦的感觉。那怎么办？
从Python 2.5开始引入了with语句，一种与异常处理相关的功能。

with语句适用于对资源进行访问的场合，确保不管使用过程中是否发生异常都会执行必要的“清理”操作，释放资源，比如文件使用后自动关闭、线程中锁的自动获取和释放等。比如上面的代码，通过使用with语句改造，就变成了下面这个样子：

```python
#!/usr/bin/env python
with open('students.txt', 'r') as fileReader:
    for row in fileReader:
        print(row.strip())
```
## 5. 函数混合
### 5.1 list 、filter、lambda

```python
$cat test1.py
#!/usr/bin/python

x = range(-5, 5)
new_list = []
for num in x:
   if num < 0:
      new_list.append(num)
print new_list

all_less_than_zero = list(filter(lambda num: num< 0, x))
print all_less_than_zero
[root@localhost func]# python test1.py
[-5, -4, -3, -2, -1]
[-5, -4, -3, -2, -1]
```

### 5.2 reduce、lambda

```python
$ cat test2.py
#!/usr/bin/python

product = 1
x = [1, 2, 3, 4]
for num in x:
   product = product * num
print product

product = reduce((lambda x, y: x * y),[1, 2, 3, 4])
print product
[root@localhost func]# python test2.py
24
24
```
### 5.3 lambda、map、list

```python
cat test3.py
#!/usr/bin/python

square = lambda x: x * x
print square(3)

x = [1, 2, 3, 4]
def square(num):
   return num * num
print list(map(square, x))

print(list(map(lambda num: num * num,x)))
[root@localhost func]# python test3.py
9
[1, 4, 9, 16]
[1, 4, 9, 16]

```


参考：

 - [runoob Python 内置函数](https://www.runoob.com/python/python-built-in-functions.html)
 - [Python3常用内置函数](https://www.cnblogs.com/Lands-ljk/p/5753748.html)
 - [python docs Built-in Functions](https://docs.python.org/3/library/functions.html)
 - [w3school Python Built in Functions](https://www.w3schools.com/python/python_ref_functions.asp)
 - [geeksforgeeks Python Built in Functions](https://www.geeksforgeeks.org/python-built-in-functions/)
