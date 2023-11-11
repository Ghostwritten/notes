#  python 基本函数



---
## 1. 函数简介

函数是组织好的，可重复使用的，用来实现单一，或相关联功能的代码段。

函数能提高应用的模块性，和代码的重复利用率。你已经知道Python提供了许多内建函数，比如print()。但你也可以自己创建函数，这被叫做用户自定义函数。

## 2. 定义一个函数

你可以定义一个由自己想要功能的函数，以下是简单的规则：

函数代码块以 def 关键词开头，后接函数标识符名称和圆括号()。
任何传入参数和自变量必须放在圆括号中间。圆括号之间可以用于定义参数。
函数的第一行语句可以选择性地使用文档字符串—用于存放函数说明。
函数内容以冒号起始，并且缩进。
return [表达式] 结束函数，选择性地返回一个值给调用方。不带表达式的return相当于返回 None。


```python
def functionname( parameters ):
   "函数_文档字符串"
   function_suite
   return [expression]
```
函数实例

```python
def printme( str ):
   "打印传入的字符串到标准显示设备上"
   print str
   return strt
```
## 3. pass空的语句块
pass语句在Python中表示一个空的语句块。

```python
def someFunction():
    pass
```

## 4. 函数调用

```python
$ cat fun1.py
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
# 定义函数
def printme( str ):
   "打印任何传入的字符串"
   print str
   return
 
# 调用函数
printme("我要调用用户自定义函数!")
printme("再次调用同一函数")
[root@localhost func]# python fun1.py
我要调用用户自定义函数!
再次调用同一函数
```

## 5. 参数传递
**参数类型属于对象，变量是没有类型的**

 1. 不可变参数：变量赋值 a=5 后再赋值 a=10，这里实际是新生成一个 int 值对象 10，再让 a 指向它，而 5被丢弃，不是改变a的值，相当于新生成了a。
 2. 可变参数：如 列表，字典
 3. 必备参数：必备参数须以正确的顺序传入函数。调用时的数量必须和声明时的一样。
 4. 关键字参数：关键字参数和函数调用关系紧密，函数调用使用关键字参数来确定传入的参数值。
 5. 默认参数：调用函数时，默认参数的值如果没有传入，则被认为是默认值。下例会打印默认的age，如果age没有被传入
 6. 不定长参数：一个函数能处理比当初声明时更多的参数。这些参数叫做不定长参数，和上述2种参数不同，声明时不会命名。

### 5.1 不可变参数
```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
def ChangeInt( a ):
    a = 10
 
b = 2
ChangeInt(b)
print b # 结果是 2
```
### 5.2 可变参数

```python
[root@localhost func]# cat func2.py
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
# 可写函数说明
def changeme( mylist ):
   "修改传入的列表"
   mylist.append([1,2,3,4])
   print "函数内取值: ", mylist
   return
 
# 调用changeme函数
mylist = [10,20,30]
changeme( mylist )
print "函数外取值: ", mylist
[root@localhost func]# python func2.py
函数内取值:  [10, 20, 30, [1, 2, 3, 4]]
函数外取值:  [10, 20, 30, [1, 2, 3, 4]]
```
### 5.3 必备参数：

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
#可写函数说明
def printme( str ):
   "打印任何传入的字符串"
   print str
   return
 
#调用printme函数
printme()

实例输出结果：
Traceback (most recent call last):
  File "test.py", line 11, in <module>
    printme()
TypeError: printme() takes exactly 1 argument (0 given)
```
### 5.4 关键字参数

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
#可写函数说明
def printme( str ):
   "打印任何传入的字符串"
   print str
   return
 
#调用printme函数
printme( str = "My string")
```

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
#可写函数说明
def printinfo( name, age ):
   "打印任何传入的字符串"
   print "Name: ", name
   print "Age ", age
   return
 
#调用printinfo函数
printinfo( age=50, name="miki" )
```
### 5.5 默认参数

```python
[root@localhost func]# cat fun3.py
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
#可写函数说明
def printinfo( name, age = 35 ):
   "打印任何传入的字符串"
   print "Name: ", name
   print "Age ", age
   return
 
#调用printinfo函数
printinfo( age=50, name="miki" )
printinfo( name="miki" )
[root@localhost func]# python fun3.py
Name:  miki
Age  50
Name:  miki
Age  35
```
### 5.6 不定长参数


```python
$ cat fun5.py
#!/usr/bin/python

def add(*args):
    print(args)
    sum=0
    for i in args:
        sum+=i
    print(sum)

def print_info(*args, **kwargs):
    print(args)
    print(kwargs)
    for i in kwargs:
        print('%s:%s'%(i, kwargs[i]))

def print_info2(sex='male', *args, **kwargs):
     print(kwargs)
     for i in kwargs:
         print('%s:%s'%(i, kwargs[i]))

add(1,2,3,4)
print_info('alex', 18, 'male', job='IT', hobby='girls', height=188)
print_info2('ttt',2,34,'female',name='zongxun')
[root@localhost func]$ python fun5.py
(1, 2, 3, 4)
10
('alex', 18, 'male')
{'hobby': 'girls', 'job': 'IT', 'height': 188}
hobby:girls
job:IT
height:188
{'name': 'zongxun'}
name:zongxun
```




## 6. 匿名函数 lambda
python 使用 `lambda` 来创建匿名函数。

 - lambda只是一个表达式，函数体比def简单很多。
 - lambda的主体是一个表达式，而不是一个代码块。仅仅能在lambda表达式中封装有限的逻辑进去。
 - lambda函数拥有自己的命名空间，且不能访问自有参数列表之外或全局命名空间里的参数。
 - 虽然lambda函数看起来只能写一行，却不等同于C或C++的内联函数，后者的目的是调用小函数时不占用栈内存从而增加运行效率。

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
# 可写函数说明
sum = lambda arg1, arg2: arg1 + arg2
 
# 调用sum函数
print "相加后的值为 : ", sum( 10, 20 )
print "相加后的值为 : ", sum( 20, 20 )

[root@localhost func]# python lambda.py
相加后的值为 :  30
相加后的值为 :  40
```

## 7. return 语句
return语句[表达式]退出函数，选择性地向调用方返回一个表达式。不带参数值的return语句返回None.

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
# 可写函数说明
def sum( arg1, arg2 ):
   # 返回2个参数的和."
   total = arg1 + arg2
   print "函数内 : ", total
   return total
 
# 调用sum函数
total = sum( 10, 20 )
```

## 8. 变量
全局变量和局部变量

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
total = 0 # 这是一个全局变量
# 可写函数说明
def sum( arg1, arg2 ):
   #返回2个参数的和."
   total = arg1 + arg2 # total在这里是局部变量.
   print "函数内是局部变量 : ", total
   return total
 
#调用sum函数
sum( 10, 20 )
print "函数外是全局变量 : ", total

输出结果：

函数内是局部变量 :  30
函数外是全局变量 :  0
```
### 8.1 global语句
如果你想要为一个定义在函数外的变量赋值，那么你就得告诉Python这个变量名不是局部的，而是 全局 的

```python
#!/usr/bin/python
# Filename: func_global.py

def func():
    global x

    print 'x is', x
    x = 2
    print 'Changed local x to', x

x = 50
func()
print 'Value of x is', x

$ python func_global.py
x is 50
Changed global x to 2
Value of x is 2
```
内部作用域先声明就会覆盖外部变量，不声明直接使用，就会使用外部作用域变量

```python
[root@localhost func]$ cat global1.py
#!/usr/bin/python
count = 10
def outer():
    global count
    print(count)
    count = 5
    print(count)
outer()
print count
[root@localhost func]$ python global1.py
10
5
5
```


## 9. DocStrings
文档字符串 ，它通常被简称为 `docstrings` 。DocStrings是一个重要的工具，由于它帮助你的程序文档更加简单易懂，你应该尽量使用它。

```python
#!/usr/bin/python
# Filename: func_doc.py

def printMax(x, y):
    '''Prints the maximum of two numbers.

    The two values must be integers.'''
    x = int(x) # convert to integers, if possible
    y = int(y)

    if x > y:
        print x, 'is maximum'
    else:
        print y, 'is maximum'

printMax(3, 5)
print printMax.__doc__

$ python func_doc.py
5 is maximum
Prints the maximum of two numbers.

        The two values must be integers.
```
参考：

 - [简明python](https://woodpecker.org.cn/abyteofpython_cn/chinese/ch07s07.html)
 - [w3schools Python Functions](https://www.w3schools.com/python/python_functions.asp)
 - [tutorialspoint Python - Functions](https://www.tutorialspoint.com/python/python_functions.htm)
 - [Defining Your Own Python Function](https://realpython.com/defining-your-own-python-function/)

