#  pyhon tuple 元组



-----
## 1.特点

 - Python的元组与列表类似，不同之处在于元组的元素不能修改
 - Tuple 比 list 操作速度快，遍历时请使用 tuple 代替 list
 - 元组使用小括号，列表使用方括号
 - 元组创建很简单，只需要在括号中添加元素，并使用逗号隔开即可

## 2.方法

### 2.1 创建元组

####  2.1.1（)创建元组

 [ ]切片
```python
$ cat tup1.py 
#!/usr/bin/python
#!---coding:utf-8----

tup1 = ('physics', 'chemistry', 1997, 2000)
print tup1      
print type(tup1)    #类型

$  python tup1.py 
('physics', 'chemistry', 1997, 2000)
<type 'tuple'>
```
#### 2.1.2 tuple()函数创建元组
将其它数据类型转换为元组类型，比如：字符串，列表，元组等

```python
$ cat tup5.py 
#!/usr/bin/python
#!---coding:utf-8----

#将字符串转换成元组
tup1 = tuple("hello")
print(tup1)
#将列表转换成元组
list1 = ['Python', 'Java', 'C++', 'JavaScript']
tup2 = tuple(list1)
print(tup2)
#将字典转换成元组
dict1 = {'a':100, 'b':42, 'c':9}
tup3 = tuple(dict1)
print(tup3)
#将区间转换成元组
range1 = range(1, 6)
tup4 = tuple(range1)
print(tup4)
$  python tup5.py 
('h', 'e', 'l', 'l', 'o')
('Python', 'Java', 'C++', 'JavaScript')
('a', 'c', 'b')
(1, 2, 3, 4, 5)

```

#### 2.1.3 创建空值

```python
$ cat tup6.py 
#!/usr/bin/python
#!---coding:utf-8----
print(tuple())    #第一种
test1 = tuple()   #第二种
print type(test1)
print test1

test2 = ()         #第三种
print type(test2)
print test2
$  python tup6.py 
()
<type 'tuple'>
()
<type 'tuple'>
()
```

### 2.2 访问元组

#### 2.2.1 index访问位置

和列表一样，我们可以使用索引（Index）访问元组中的某个元素（得到的是一个元素的值），也可以使用切片访问元组中的一组元素（得到的是一个新的子元组）。

使用索引访问元组元素的格式为：

```powershell
tuplename[i]
```

其中，tuplename 表示元组名字，i 表示索引值。元组的索引可以是正数，也可以是负数。

使用切片访问元组元素的格式为：

```powershell
tuplename[start : end : step]
```

其中，start 表示起始索引，end 表示结束索引，step 表示步长。

```python
#!/usr/bin/python
#!---coding:utf-8----

url = tuple("http://www.baidu.com")
#使用索引访问元组中的某个元素
print(url[3])  #使用正数索引
print(url[-1])  #使用负数索引
print(url[3: 5])  #使用正数切片
print(url[3: 7: 2])  #指定步长
print(url[-3: -1])  #使用负数切片

print(url[:2])   #切片操作
print(url[0:-1]) #掐头去尾
print(url[::-1]) #反序
print(url[::2]) #隔一个取一个
print(url[::])  #全取
[root@localhost tuple]# python tup7.py 
p
m
('p', ':')
('p', '/')
('c', 'o')
('h', 't')
('h', 't', 't', 'p', ':', '/', '/', 'w', 'w', 'w', '.', 'b', 'a', 'i', 'd', 'u', '.', 'c', 'o')
('m', 'o', 'c', '.', 'u', 'd', 'i', 'a', 'b', '.', 'w', 'w', 'w', '/', '/', ':', 'p', 't', 't', 'h')
('h', 't', ':', '/', 'w', '.', 'a', 'd', '.', 'o')
('h', 't', 't', 'p', ':', '/', '/', 'w', 'w', 'w', '.', 'b', 'a', 'i', 'd', 'u', '.', 'c', 'o', 'm')

```

#### 2.2.2 count访问统计

```python
#!/usr/bin/python
#!---coding:utf-8----

t = (90, 30, 40, 80, 100)
print('{}.count({})={}'.format('t', 90, t.count(90)))
[root@localhost tuple]# python tup9.py
t.count(90)=1
```


## 2.3 修改元组(本质:组合)
元组中的元素值是不允许修改的，但我们可以对元组进行连接组合
```python
$ cat tup2.py 
#!/usr/bin/python
#!---coding:utf-8----

tup1 = (12, 34.56)
tup2 = ('abc', 'xyz')
# 以下修改元组元素操作是非法的
#tup1[0] = 100
# 创建一个新的元组
tup3 = tup1 + tup2
print tup3

$ python tup2.py 
(12, 34.56, 'abc', 'xyz')
```

## 2.4 删除元组

元组中的元素值是不允许删除的，但我们可以使用del语句来删除整个元组

```python
$ cat tup3.py 
#!/usr/bin/python
#!---coding:utf-8----
tup=('physics', 'chemistry', 1997, 2000)
print tup
del tup
print "After deleting tup : "
print tup

$ python tup3.py 
('physics', 'chemistry', 1997, 2000)
After deleting tup : 
Traceback (most recent call last):
  File "tup3.py", line 7, in <module>
    print tup
NameError: name 'tup' is not defined
```

## 3. 元组运算符

与字符串一样，元组之间可以使用 + 号和 * 号进行运算。这就意味着他们可以组合和复制，运算后会生成一个新的元组。

```python
>>> len((1, 2, 3)) 
3                                     #计算元素个数
>>> (1, 2, 3) + (4, 5, 6)
(1, 2, 3, 4, 5, 6)              #连接
>>> ('Hi!',) * 4 
('Hi!', 'Hi!', 'Hi!', 'Hi!')      #复制
>>> 3 in (1, 2, 3) 
True                               #元素是否存在
>>> for x in (1, 2, 3): print x,
1 2 3                               #迭代
```

## 4. 无关闭分隔符
任意无符号的对象，以逗号隔开，默认为元组

```python
$ cat tup4.py 
#!/usr/bin/python
#!---coding:utf-8----
print 'abc', -4.24e93, 18+6.6j, 'xyz'
x, y = 1, 2
print "Value of x , y : ", x,y

$  python tup4.py 
abc -4.24e+93 (18+6.6j) xyz
Value of x , y :  1 2
```
## 5. 多元赋值

```python
$ cat  tup10.py
#!/usr/bin/python
#!---coding:utf-8----

m = 2,3
#m什么类型？
print(m)
print(type(m))
#x,y是什么值？
x,y = m
print(x, y)
[root@localhost tuple]# python tup10.py
(2, 3)
<type 'tuple'>
(2, 3)
```

## 6. 元组内置函数

 - 1 cmp(tuple1, tuple2)比较两个元组元素。
 - 2 len(tuple)计算元组元素个数。
 - 3 max(tuple)返回元组中元素最大值。
 - 4 min(tuple)返回元组中元素最小值。
 - 5 tuple(seq)将列表转换为元组。

```python
$ cat tup8.py
#!/usr/bin/python
#!---coding:utf-8----

tup1 = (1, 2, 3, 4, 5)
tup3 = (1, 2, 3, 4)
tup2 = ('a', 'b', 'c', 'd', 'e')

print cmp(tup1, tup2)
print cmp(tup1, tup3)
print len(tup1)
print len(tup2)
print max(tup1)
print max(tup2)
print min(tup1)
[root@localhost tuple]# python tup8.py 
-1
1
5
5
5
e
1
```
## 7. 元组遍历
###  7.1 单元组循环第一种for...in...
```python
cat tup11.py
#!/usr/bin/python
#!---coding:utf-8----

tup = ('小米', '创维', '海信','康佳','长虹')
for tv in tup:
    print(tv)
[root@localhost tuple]# python3.8 tup11.py
小米
创维
海信
康佳
长虹
```

###  7.2 单元组循环第二种enumerate()

```bash
girl_tuple = ("貂蝉", "狐狸精","范金链","翠花","小班")
 
for index, everyOne in enumerate(girl_tuple):
    print (str(index) + everyOne)

#0貂蝉
#1狐狸精
#2范金链
#3翠花
#4小班
```
###  7.3 单元组循环第三种range()或者xrange()

```bash
girl_tuple = ("貂蝉", "狐狸精","范金链","翠花","小班")
for index in range(len(girl_tuple)):
    print(girl_tuple[index])


girl_tuple = ("貂蝉", "狐狸精","范金链","翠花","小班")
for index in xrange(len(girl_tuple)):
    print(girl_tuple[index])

#貂蝉
#狐狸精
#范金链
#翠花
#小班
```
###  7.4 元组列表循环
格式1
```bash
res_list = [x for x,_ in rows]
```
示例1

```bash
lst = [(1, 2), (3, 4), (5, 6)]
res = [x for x,_ in lst]
res

>>>[1, 3, 5]
```
格式2

```bash
res_list = [x[0] for x in rows]
```
示例2

```bash
rows = [(1, 2), (3, 4), (5, 6)]
res = [x[0] for x in rows]
res

>>>[1, 3, 5]
```
示例3：map函数和operator.itemgetter

```bash
from operator import itemgetter
rows = [(1, 2), (3, 4), (5, 6)]
x = map(itemgetter(1), rows)
x
>>> [2, 4, 6]
```

---

✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>

 - [python list 列表](https://ghostwritten.blog.csdn.net/article/details/104437743)
 - [pyhon tuple 元组](https://ghostwritten.blog.csdn.net/article/details/104486159)
 - [python dict 字典](https://blog.csdn.net/xixihahalelehehe/article/details/104488899)
 - [python 基础入门快速学习手册](https://ghostwritten.blog.csdn.net/article/details/112743744)

![在这里插入图片描述](https://img-blog.csdnimg.cn/4d45a7e5c8464b4fa0a70bc19bcbc3ed.gif#pic_center)

