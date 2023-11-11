#  python dict 字典




----
## 1. 字典特点
字典是另一种可变容器模型，且可存储任意类型对象。
字典的每个键值(key=>value)对用冒号(:)分割，每个对之间用逗号(,)分割，整个字典包括在花括号
例如：`d = {key1 : value1, key2  value2 }`
键必须是唯一的，但值则不必
不允许同一个键出现两次。创建时如果同一个键被赋值两次，后一个值会被记住

## 2. 字典内置函数


 - 1 cmp(dict1, dict2)比较两个字典元素。
 - 2 len(dict)计算字典元素个数，即键的总数。
 - 3 str(dict)输出字典可打印的字符串表示。
 - 4 type(variable)返回输入的变量类型，如果变量是字典就返回字典类型。

## 3. 字典内置方法

```bash
1 dict.clear()                    删除字典内所有元素
2 dict.copy()                     返回一个字典的浅复制
3 dict.fromkeys(seq[, val])       创建一个新字典，以序列 seq 中元素做字典的键，val 为字典所有键对应的初始值
4 dict.get(key, default=None)     返回指定键的值，如果值不在字典中返回default值
5 dict.has_key(key)               如果键在字典dict里返回true，否则返回false
6 dict.items()                    以列表返回可遍历的(键, 值) 元组数组
7 dict.keys()                     以列表返回一个字典所有的键
8 dict.setdefault(key, default=None)和get()类似, 但如果键不存在于字典中，将会添加键并将值设为default
9 dict.update(dict2)              把字典dict2的键/值对更新到dict里
10 dict.values()                  以列表返回字典中的所有值
11 pop(key[,default])             删除字典给定键 key 所对应的值，返回值为被删除的值。key值必须给出。 否则，返回default值。
12 popitem()                      随机返回并删除字典中的一对键和值。
```
## 4. 方法运用
### 4.1 增
```python
$ cat dict1.py
#!/usr/bin/python
#!---coding:utf-8----

dic1={'age': 18, 'name': 'alex', 'hobby': 'girl'}
dic2={'1':'111','2':'222'}
dic2={'1':'111','name':'222'}

dic1.update(dic2)
print(dic1)
print(dic2)
for k in dic2:  #等同update()
    dic1[k] = dic2[k]
print dic1

[root@localhost dict]$ python dict1.py 
{'1': '111', 'hobby': 'girl', 'age': 18, 'name': '222'}
{'1': '111', 'name': '222'}
{'1': '111', 'hobby': 'girl', 'age': 18, 'name': '222'}
```

第二种
```bash
dic2={'1':'111','2':'222'}
dic2['xx']='cccc'
print dic2
```

```bash
$ python dict15.py 
{'1': '111', 'xx': 'cccc', '2': '222'}
```

### 4.2 删

 - del
 - pop('key')
 - popitem()
 - clear()

```python
cat dict3.py
#!/usr/bin/python
# -*- coding: UTF-8 -*-

dict = {'Name': 'Zara', 'Age': 7, 'Class': 'First', 'weight': 56, 'gender': 'man'};

del dict['Name']; # 删除键是'Name'的条目
print dict

print(dict.pop('Age')) #删除字典中指定键值对，并返回该键值对的值

ret = dict.pop('Class')
print(ret)
print(dict)

a = dict.popitem() #随机删除某组键值对，并以元组方式返回值
print(a, dict)

dict.clear();     # 清空词典所有条目
del dict ;        # 删除词典
print dict

[root@localhost dict]# python dict3.py 
{'gender': 'man', 'Age': 7, 'weight': 56, 'Class': 'First'}
7
First
{'gender': 'man', 'weight': 56}
(('gender', 'man'), {'weight': 56})
<type 'dict'>

```



### 4.3 改

```python
$ cat dict2.py
#!/usr/bin/python
#!---coding:utf-8----

dict = {'Name': 'Zara', 'Age': 7, 'Class': 'First'};
dict['Age'] = 8; # 修改已存在的键值
dict['School'] = "DPS School"; # 增加新的键值对
print "dict['Age']: ", dict['Age'];
print "dict['School']: ", dict['School'];

[root@localhost dict]$ python dict2.py 
dict['Age']:  8
dict['School']:  DPS School
```
### 4.4 查

```python
$ cat dict4.py 
#!/usr/bin/python
#!---coding:utf-8----

dict={'age': 18, 'name': 'alex', 'hobby': 'girl'}
print(dict['name'])
print(list(dict.keys()))
print(list(dict.values()))
print(list(dict.items()))
it = dict.iteritems()
print it
print dict.get("name", "girl") 
print dict.get("e", 18)   
if "name" in dict:  #等同get()
    print dict["name"]
else:
    print "None"

[root@localhost dict]$ python dict4.py 
alex
['hobby', 'age', 'name']
['girl', 18, 'alex']
[('hobby', 'girl'), ('age', 18), ('name', 'alex')]
<dictionary-itemiterator object at 0x7f93657ad418>
alex
18
alex
```

### 4.5 创
dict.fromkeys()

```python
$ cat dict5.py 
#!/usr/bin/python

seq = ('name', 'age', 'sex')

dict = dict.fromkeys(seq)
print "New Dictionary : %s" %  str(dict)

dict = dict.fromkeys(seq, 10)
print "New Dictionary : %s" %  str(dict)
[root@localhost dict]$ python  dict5.py 
New Dictionary : {'age': None, 'name': None, 'sex': None}
New Dictionary : {'age': 10, 'name': 10, 'sex': 10}
```

## 5. 设置默认值

```python
$ cat dict6.py 
#!/usr/bin/python
#!---coding:utf-8----

dict = {}
dict.setdefault("a")
print dict
dict["a"] = "apple"
dict.setdefault("a","default")
print dict
[root@localhost dict]$ python  dict6.py 
{'a': None}
{'a': 'apple'}
```

## 6. 字典排序

```python
$ cat dict7.py 
#!/usr/bin/python
#!---coding:utf-8----

dict = {"a" : "apple", "b" : "grape", "c" : "orange", "d" : "banana"}
#按照key排序
print sorted(dict.items(), key=lambda d: d[0])
#按照value排序 
print sorted(dict.items(), key=lambda d: d[1])
[root@localhost dict]$ python dict7.py 
[('a', 'apple'), ('b', 'grape'), ('c', 'orange'), ('d', 'banana')]
[('a', 'apple'), ('d', 'banana'), ('b', 'grape'), ('c', 'orange')]
```

## 7. 字典拷贝
### 7.1 浅拷贝

```python
$ cat dict8.py 
#!/usr/bin/python
#!---coding:utf-8----

dict = {"a" : "apple", "b" : "grape"}
dict2 = {"c" : "orange", "d" : "banana"}
dict2 = dict.copy()
print dict2
[root@localhost dict]$ python dict8.py 
{'a': 'apple', 'b': 'grape'}
```
#### 7.2 深拷贝

```python
$ cat dict9.py 
#!/usr/bin/python
#!---coding:utf-8----

import copy
dict = {"a" : "apple", "b" : {"g" : "grape","o" : "orange"}}
dict2 = copy.deepcopy(dict)
dict3 = copy.copy(dict)   
print(copy1 == copy2)   # True
print(copy1 is copy2)   # False
dict2["b"]["g"] = "orange"  #dict2深拷贝相当于独立object
print dict
dict3["b"]["g"] = "orange"
print dict
[root@localhost dict]$ python dict9.py
True
False
{'a': 'apple', 'b': {'o': 'orange', 'g': 'grape'}}
{'a': 'apple', 'b': {'o': 'orange', 'g': 'orange'}}
```

## 8. 字典遍历
### 8.1 遍历key

```python
cat dict10.oy
#!/usr/bin/python
#!---coding:utf-8----
dict = {"a" : "apple", "b" : "banana", "c" : "grape", "d" : "orange"}

for k in dict:
 print k
for k in dict:
 print k,
for key in dict.iterkeys():
    print key
for key in dict.keys():
    print key,
[root@localhost dict]# python dict10.oy 
a
c
b
d
a c b d a
c
b
d
a c b d



```
### 8.2 遍历值

```python
$ cat dict11.py
#!/usr/bin/python
#!---coding:utf-8----
dict = {"a" : "apple", "b" : "banana", "c" : "grape", "d" : "orange"}

for k in dict:    #遍历value
  print dict[k]
for k in dict:    #遍历value不换行
  print dict[k],
for value in dict.itervalues():
    print value,
for value in dict.values():
    # d.values() -> [2, 1, 3]
    print value
[root@localhost dict]$  python dict11.py 
apple
grape
banana
orange
apple grape banana orange apple grape banana orange apple
grape
banana
orange
```

### 8.3 遍历keys和values

```python
$ cat dict12.py
#!/usr/bin/python
#!---coding:utf-8----
dict = {"a" : "apple", "b" : "banana", "c" : "grape", "d" : "orange"}

for key, value in dict.iteritems():
    # d.iteritems: an iterator over the (key, value) items
    print key,'corresponds to',dict[key]
for key, value in dict.items():
    # d.items(): list of d's (key, value) pairs, as 2-tuples
    # [('y', 2), ('x', 1), ('z', 3)]
    print key,'corresponds to',value
[root@localhost dict]$  python dict12.py 
a corresponds to apple
c corresponds to grape
b corresponds to banana
d corresponds to orange
a corresponds to apple
c corresponds to grape
b corresponds to banana
d corresponds to orange
```

## 9. 函数字典传参

```python
$ cat dict13.py
#!/usr/bin/python
#!---coding:utf-8----

dic={"m": 1,"n": 2,"q": 3}
def dics(qwe):
    print qwe
 
dics(dic)

[root@localhost dict]$ python dict13.py 
{'q': 3, 'm': 1, 'n': 2}
```

## 10. 字典推导式
语法一：

    key：字典中的key
    value：字典中的value
    dict.items()：序列
    condition：条件表达式
    key_exp：在for循环中，如果条件表达式condition成立(即条件表达式成立)，返回对应的key,value并作key_exp,value_exp处理
    value_exp：在for循环中，如果条件表达式condition成立(即条件表达式成立)，返回对应的key,value并作key_exp,value_exp处理
```powershell
{key_exp:value_exp for key,value in dict.items() if condition}
```

语法二：

    key：字典中的key
    value：字典中的value
    dict.items()：序列
    condition：条件表达式
    key_exp：在for循环中，如果条件表达式condition成立(即条件表达式成立)，返回对应的key,value并作key_exp,value_exp处理
    value_exp1：在for循环中，如果条件表达式condition成立(即条件表达式成立)，返回对应的key,value并作key_exp,value_exp1处理
    value_exp2：在for循环中，如果条件表达式condition不成立(即条件表达式不成立)，返回对应的key,value并作key_exp,value_exp2处理
```powershell
{key_exp:value_exp1 if condition else value_exp2 for key,value in dict.items()}
```

```python
$ cat dict14.py
#!/usr/bin/python
#!---coding:utf-8----

dict1 = {"a":10,"B":20,"C":True,"D":"hello world","e":"python教程"}
dict2 = {key:value for key,value in dict1.items() if key.islower()}
print(dict2)
dict3 = {key.lower():value  for key,value in dict1.items() }
print(dict3)
dict4 = {key:value if key.isupper() else "error" for key,value in dict1.items() }
print(dict4)
[root@localhost dict]$  python3.8 dict14.py 
{'a': 10, 'e': 'python教程'}
{'a': 10, 'b': 20, 'c': True, 'd': 'hello world', 'e': 'python教程'}
{'a': 'error', 'B': 20, 'C': True, 'D': 'hello world', 'e': 'error'}
```


✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>

 - [python dict 字典](https://blog.csdn.net/xixihahalelehehe/article/details/104488899)
 - [pyhon tuple 元组](https://blog.csdn.net/xixihahalelehehe/article/details/104486159)
 - [python list 列表](https://blog.csdn.net/xixihahalelehehe/article/details/104437743)

![在这里插入图片描述](https://img-blog.csdnimg.cn/9aa34081a04145ff831f7d81fabb69a9.gif#pic_center)

