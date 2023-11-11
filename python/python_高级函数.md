#  python 高级函数




---
## 1 函数模块化调用
带时间戳日志格式的函数模块化
```python
$ mkdir log1
$ touch log1/__init__.py
$ vim log1/timestamp.py
#!/usr/bin/env python3
import time
def Timer(msg):
    print(str(msg) + str(time.time() ) )
    charge = 0.02
    return charge

$ vim charge.py 
#!/usr/bin/env python3
from log1 import timestamp
print("Press RETURN for the time (costs 2 cents).")
print("Press Q RETURN to quit.")
total = 0
while True:
    kbd = input()
    if kbd.lower() == "q":
        print("You owe $" + str(total) )
        exit()
    else:
        charge = timestamp.Timer("Time is ")
        total = total+charge

[root@localhost time]$ python3.8 charge.py 
Press RETURN for the time (costs 2 cents).
Press Q RETURN to quit.

Time is 1584882110.5468674

Time is 1584882111.1065543
q
You owe $0.04
[root@localhost time]$ python3.8 charge.py 
Press RETURN for the time (costs 2 cents).
Press Q RETURN to quit.

Time is 1584882119.8083994

Time is 1584882120.2742803

Time is 1584882120.6922472
q
You owe $0.06
```

假如不调用模块化函数方法

```python
#!/usr/bin/env python3
import time
total = 0
def Timer(msg):
    print(str(msg) + str(time.time() ) )
    charge = .02
    return charge
print("Press RETURN for the time (costs 2 cents).")
print("Press Q RETURN to quit.")
while True:
    kbd = input()
    if kbd.lower() == "q":
        print("You owe $" + str(total) )
        exit()
    else:
        charge = Timer("Time is ")
        total = total+charge

[root@localhost time]# python3.8 charge2.py
Press RETURN for the time (costs 2 cents).
Press Q RETURN to quit.

Time is 1584882257.4966393

Time is 1584882258.4600565
q
You owe $0.04
```

## 2. 闭包函数
### 2.1 闭包定义：
如果在一个内部函数里，对在外部作用域（但不是在全局作用域）的变量进行引用，那么内部函数就被认为是闭包(closure)

### 2.2 闭包实例

```python
def print_msg():
    # print_msg 是外围函数
    msg = "zen of python"
    def printer():
        # printer 是嵌套函数
        print(msg)
    return printer

another = print_msg()
# 输出 zen of python
another()
```
another 就是一个闭包，闭包本质上是一个函数，它有两部分组成，printer 函数和变量 msg。闭包使得这些变量的值始终保存在内存中。

闭包，顾名思义，就是一个封闭的包裹，里面包裹着自由变量，就像在类里面定义的属性值一样，自由变量的可见范围随同包裹，哪里可以访问到这个包裹，哪里就可以访问到这个自由变量
### 2.3 闭包作用
#### 2.3.1 当闭包执行完后，仍然能够保持住当前的运行环境
比如说，如果你希望函数的每次执行结果，都是基于这个函数上次的运行结果。我以一个类似棋盘游戏的例子来说明。假设棋盘大小为50*50，左上角为坐标系原点(0,0)，我需要一个函数，接收2个参数，分别为方向(direction)，步长(step)，该函数控制棋子的运动。棋子运动的新的坐标除了依赖于方向和步长以外，当然还要根据原来所处的坐标点，用闭包就可以保持住这个棋子原来所处的坐标。


```python
#!/usr/bin/python
#--coding:utf-8
origin = [0, 0] # 坐标系统原点 
legal_x = [0, 50] # x轴方向的合法坐标 
legal_y = [0, 50] # y轴方向的合法坐标 
def create(pos=origin): 
 def player(direction,step): 
  # 这里应该首先判断参数direction,step的合法性，比如direction不能斜着走，step不能为负等 
  # 然后还要对新生成的x，y坐标的合法性进行判断处理，这里主要是想介绍闭包，就不详细写了。 
  new_x = pos[0] + direction[0]*step 
  new_y = pos[1] + direction[1]*step 
  pos[0] = new_x 
  pos[1] = new_y 
  #注意！此处不能写成 pos = [new_x, new_y]，原因在上文有说过 
  return pos 
 return player 
  
player = create() # 创建棋子player，起点为原点 
print player([1,0],10) # 向x轴正方向移动10步 
print player([0,1],20) # 向y轴正方向移动20步 
print player([-1,0],10) # 向x轴负方向移动10步 
输出为：

[10, 0] 
[10, 20] 
[0, 20] 
```
#### 2.3.2 闭包可以根据外部作用域的局部变量来得到不同的结果
这有点像一种类似配置功能的作用，我们可以修改外部的变量，闭包根据这个变量展现出不同的功能。比如有时我们需要对某些文件的特殊行进行分析，先要提取出这些特殊行。

```python
def make_filter(keep): 
    def the_filter(file_name): 
          file = open(file_name) 
          lines = file.readlines() 
          file.close() 
          filter_doc = [i for i in lines if keep in i] 
          return filter_doc 
      return the_filter

filter = make_filter("pass")
filter_result = filter("result.txt")
```

如果我们需要取得文件"result.txt"中含有"pass"关键字的行，则可以这样使用例子程序

### 2.4 使用闭包注意事项

#### 2.4.1 闭包中是不能修改外部作用域的局部变量的

```python
$ cat bibao3.py
#!/usr/bin/python

def out():
    x = 0
    def inn():
        x = 1
        print('inner x:', x)
    print('out x:', x)
    inn()
    print('out x:', x)
    
out()
[root@localhost func]# python bibao3.py
('out x:', 0)
('inner x:', 1)
('out x:', 0)
```
#### 2.4.2 闭包函数返回变量报错
```python
def foo(): 
 a = 1
 def bar(): 
  a = a + 1
  return a 
 return bar
```
这段程序的本意是要通过在每次调用闭包函数时都对变量a进行递增的操作。但在实际使用时

```python
>>> c = foo() 
>>> print c() 
Traceback (most recent call last): 
 File "<stdin>", line 1, in <module> 
 File "<stdin>", line 4, in bar 
UnboundLocalError: local variable 'a' referenced before assignment 
```
这是因为在执行代码 c = foo()时，python会导入全部的闭包函数体bar()来分析其的局部变量，python规则指定所有在赋值语句左面的变量都是局部变量，则在闭包bar()中，变量a在赋值符号"="的左面，被python认为是bar()中的局部变量。再接下来执行print c()时，程序运行至a = a + 1时，因为先前已经把a归为bar()中的局部变量，所以python会在bar()中去找在赋值语句右面的a的值，结果找不到，就会报错。解决的方法很简单


```python
def foo(): 
 a = [1] 
 def bar(): 
  a[0] = a[0] + 1
  return a[0] 
 return bar
```

**只要将a设定为一个容器就可以了**。这样使用起来多少有点不爽，所以在python3以后，在a = a + 1 之前，使用语句nonlocal a就可以了，该语句显式的指定a不是闭包的局部变量。

#### 2.4.3 循环语句中的闭包函数
在程序里面经常会出现这类的循环语句，Python的问题就在于，当循环结束以后，循环体中的临时变量i不会销毁，而是继续存在于执行环境中。还有一个python的现象是，python的函数只有在执行时，才会去找函数体里的变量的值。

```python
$ cat bibao2.py
#!/usr/bin/python
flist = []
for i in range(3): 
 def foo(x): print x + i 
 flist.append(foo) 

for f in flist:
  f(2)
[root@localhost func]# python bibao2.py
4
4
4
```

可能有些人认为这段代码的执行结果应该是2,3,4.但是实际的结果是4,4,4。这是因为当把函数加入flist列表里时，python还没有给i赋值，只有当执行时，再去找i的值是什么，这时在第一个for循环结束以后，i的值是2，所以以上代码的执行结果是4,4,4.
改写一下函数的定义

```python
$ cat bibao1.py
#!/usr/bin/python

flist = []
for i in range(3): 
 def foo(x,y=i): print x + y 
 flist.append(foo) 

for f in flist:
  f(2)
[root@localhost func]# python bibao1.py
2
3
4
```
闭包函数理解扩展参考链接：
[https://www.cnblogs.com/cicaday/p/python-closure.html](https://www.cnblogs.com/cicaday/p/python-closure.html)

## 3. 递归函数
#关于递归函数
1.调用自身函数
2.有一个结束条件
3.问题规模相比上次递归都因该有所减少，但凡递归函数，循环都可以解决
4.递归效率低


```python
$ cat digui.py
#!/usr/bin/python
def fat(n):               #第一种方式
    ret=1
    for i in range(1,n+1):
        ret=ret*i
    return ret
print(fat(5))

def fact(n):           #第二种方式
    if n==1:
        return 1
    return n*fact(n-1)
print(fact(5))
[root@localhost func]# python digui.py
120
120
```

## 4. 嵌套函数

 - 函数名可以赋值
 - 函数名可以作为函数参数，还可以作为返回值

```python
$ cat qiantao.py
#!/usr/bin/python
def f(n):
    return n*n

def foo(a,b,func):
    func(a)+func(b)

    ret=func(a)+func(b)
    return ret

print(foo(1,2,f))
[root@localhost func]$  python qiantao.py
5
```

参考：

 - [5 Advanced Features of Python and How to Use Them](https://towardsdatascience.com/5-advanced-features-of-python-and-how-to-use-them-73bffa373c84)
 - [Advanced Python Features](https://www.codingame.com/playgrounds/500/advanced-python-features)
 - [A Guide to Python Advanced Features](https://hackernoon.com/a-guide-to-python-advanced-features-02z31ly)

