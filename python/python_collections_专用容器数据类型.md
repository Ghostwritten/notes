#  python 模块 collections 专用容器数据类型



---

##  1. 简介
官方说法：collections模块实现了特定目标的容器，以提供Python标准内建容器[dict](https://blog.csdn.net/xixihahalelehehe/article/details/104488899?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164942342716780264033255%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164942342716780264033255&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-104488899.nonecase&utm_term=%E5%AD%97%E5%85%B8&spm=1018.2226.3001.4450) ,[list](https://blog.csdn.net/xixihahalelehehe/article/details/104437743?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164942334216782246440259%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164942334216782246440259&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-6-104437743.nonecase&utm_term=%E5%88%97%E8%A1%A8&spm=1018.2226.3001.4450) , [set](https://blog.csdn.net/xixihahalelehehe/article/details/104913051?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164942345216780271989786%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164942345216780271989786&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-25-104913051.nonecase&utm_term=set%20&spm=1018.2226.3001.4450) , 和[tuple](https://blog.csdn.net/xixihahalelehehe/article/details/104486159?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164942350116782248567035%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164942350116782248567035&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-104486159.nonecase&utm_term=%E5%85%83%E7%BB%84&spm=1018.2226.3001.4450)的替代选择。

通俗说法：Python内置的数据类型和方法，collections模块在这些内置类型的基础提供了额外的高性能数据类型，比如基础的字典是不支持顺序的，collections模块的OrderedDict类构建的字典可以支持顺序，collections模块的这些扩展的类用处非常大，熟练掌握该模块，可以大大简化Python代码，提高Python代码逼格和效率。此外，本文的最后一部分需要一些有关Python 中[面向对象编程](https://blog.csdn.net/xixihahalelehehe/article/details/106245475?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164942364216780271534896%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164942364216780271534896&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-106245475.nonecase&utm_term=%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E7%BC%96%E7%A8%8B&spm=1018.2226.3001.4450)的基本知识。

 - [中文文档](https://docs.python.org/zh-cn/3/library/collections.html#module-collections)
 - [英文文档](https://docs.python.org/3/libra)


![在这里插入图片描述](https://img-blog.csdnimg.cn/cd87778918a2486296ceb4148a9edc58.png)

##  2. 子类
用`collections.__all__`查看所有的子类，一共包含9个

```bash
import collections
print(collections.__all__)
['deque', 'defaultdict', 'namedtuple', 'UserDict', 'UserList', 
'UserString', 'Counter', 'OrderedDict', 'ChainMap']
```
这个模块实现了特定目标的容器，以提供Python标准内建容器dict , list , set , 和tuple 的替代选择。
| 名字 |含义  |
|--|--|
|[namedtuple()](https://docs.python.org/3/library/collections.html#collections.namedtuple)  |  创建命名元组子类的工厂函数|
|[deque](https://docs.python.org/3/library/collections.html#collections.deque)|类似列表(list)的容器，实现了在两端快速添加(append)和弹出(pop)|
|[ChainMap](https://docs.python.org/3/library/collections.html#collections.ChainMap)|类似字典(dict)的容器类，将多个映射集合到一个视图里面|
|[Counter](https://docs.python.org/3/library/collections.html#collections.Counter)|字典的子类，提供了可哈希对象的计数功能|
|[OrderedDict](https://docs.python.org/3/library/collections.html#collections.OrderedDict)|字典的子类，保存了他们被添加的顺序|
|[defaultdict](https://docs.python.org/3/library/collections.html#collections.defaultdict)|字典的子类，提供了一个工厂函数，为字典查询提供一个默认值|
|[UserDict](https://docs.python.org/3/library/collections.html#collections.UserDict)|封装了字典对象，简化了字典子类化|
|[UserList](https://docs.python.org/3/library/collections.html#collections.UserList)|封装了列表对象，简化了列表子类化
|[UserString](https://docs.python.org/3/library/collections.html#collections.UserString)|封装了字符串对象，简化了字符串子类化|

##  3. 计数器-Counter
### 3.1 基础介绍
一个计数器工具提供快速和方便的计数，`Counter`是一个`dict`的子类，用于计数可哈希对象。它是一个集合，元素像字典键(key)一样存储，它们的计数存储为值。计数可以是任何整数值，包括0和负数，Counter类有点像其他语言中的`bags`或`multisets`。

```python
#计算top10的单词
from collections import Counter
import re
text = 'remove an existing key one level down remove an existing key one level down'
words = re.findall(r'\w+', text)
print(Counter(words).most_common(10))
//[('remove', 2), ('an', 2), ('existing', 2), ('key', 2), ('one', 2), ('level', 2), ('down', 2)]


#计算列表中单词的个数
cnt = Counter()
for word in ['red', 'blue', 'red', 'green', 'blue', 'blue']:
    cnt[word] += 1
print(cnt）
//Counter({'red': 2, 'blue': 3, 'green': 1})


L = ['red', 'blue', 'red', 'green', 'blue', 'blue'] 
print(Counter(L))
//Counter({'red': 2, 'blue': 3, 'green': 1}


```
元素从一个`iterable` 被计数或从其他的`mapping (or counter)`初始化：

```bash
from collections import Counter

#字符串计数
Counter('gallahad') 
Counter({'g': 1, 'a': 3, 'l': 2, 'h': 1, 'd': 1})

#字典计数
Counter({'red': 4, 'blue': 2})  
Counter({'red': 4, 'blue': 2})

#是个啥玩意计数
Counter(cats=4, dogs=8)
Counter({'cats': 4, 'dogs': 8})

Counter(['red', 'blue', 'red', 'green', 'blue', 'blue'])
Counter({'red': 2, 'blue': 3, 'green': 1})
```
###  3.2 elements()
描述：返回一个迭代器，其中每个元素将重复出现计数值所指定次。 元素会按首次出现的顺序返回。 如果一个元素的计数值小于1，elements() 将会忽略它。

```bash
from collections import Counter
import re

c = Counter(a=4, b=2, c=0, d=-2)
print(list(c.elements()))
#['a', 'a', 'a', 'a', 'b', 'b']

print(sorted(c.elements()))
#['a', 'a', 'a', 'a', 'b', 'b']

c = Counter(a=4, b=2, c=0, d=5)
print(list(c.elements()))
#['a', 'a', 'a', 'a', 'b', 'b', 'd', 'd', 'd', 'd', 'd']
```
### 3.3 most_common()
返回一个列表，其中包含n个最常见的元素及出现次数，按常见程度由高到低排序。 如果 n 被省略或为None，`most_common()` 将返回计数器中的所有元素，计数值相等的元素按首次出现的顺序排序，经常用来计算top词频的词语。

```bash
a=Counter('abracadabra').most_common(3)
print(a)
#[('a', 5), ('b', 2), ('r', 2)]

b=Counter('abracadabra').most_common(5)
print(b)
#[('a', 5), ('b', 2), ('r', 2), ('c', 1), ('d', 1)]
```
###  3.4 subtract()
从迭代对象或映射对象减去元素。像`dict.update()` 但是是减去，而不是替换。输入和输出都可以是0或者负数。

```python
c = Counter(a=4, b=2, c=0, d=-2)
d = Counter(a=1, b=2, c=3, d=4)
c.subtract(d)
print(c)
#Counter({'a': 3, 'b': 0, 'c': -3, 'd': -6})

#减去一个abcd
str0 = Counter('aabbccdde')
print(str0)
#Counter({'a': 2, 'b': 2, 'c': 2, 'd': 2, 'e': 1})

str0.subtract('abcd')
print(str0)
#Counter({'a': 1, 'b': 1, 'c': 1, 'd': 1, 'e': 1}
```
###  3.5 字典方法
通常字典方法都可用于`Counter`对象，除了有两个方法工作方式与字典并不相同。

 - `fromkeys(iterable)`

这个类方法没有在Counter中实现。

 - `update([iterable-or-mapping])`

从迭代对象计数元素或者从另一个映射对象 (或计数器) 添加。 像 `dict.update()` 但是是加上，而不是替换。另外，迭代对象应该是序列元素，而不是一个 (key, value) 对。

```kotlin
sum(c.values())                 # total of all counts
c.clear()                       # reset all counts
list(c)                         # list unique elements
set(c)                          # convert to a set
dict(c)                         # convert to a regular dictionary
c.items()                       # convert to a list of (elem, cnt) pairs
Counter(dict(list_of_pairs))    # convert from a list of (elem, cnt) pairs
c.most_common()[:-n-1:-1]       # n least common elements
+c                              # remove zero and negative counts
```
###  3.6 数学操作
这个功能非常强大，提供了几个数学操作，可以结合 Counter 对象，以生产 `multisets` (计数器中大于0的元素）。 加和减，结合计数器，通过加上或者减去元素的相应计数。交集和并集返回相应计数的最小或最大值。每种操作都可以接受带符号的计数，但是输出会忽略掉结果为零或者小于零的计数。

```kotlin
c = Counter(a=3, b=1)
d = Counter(a=1, b=2)
c + d                       # add two counters together:  c[x] + d[x]
Counter({'a': 4, 'b': 3})
c - d                       # subtract (keeping only positive counts)
Counter({'a': 2})
c & d                       # intersection:  min(c[x], d[x]) 
Counter({'a': 1, 'b': 1})
c | d                       # union:  max(c[x], d[x])
Counter({'a': 3, 'b': 2})
```
单目加和减（一元操作符）意思是从空计数器加或者减去。

```kotlin
c = Counter(a=2, b=-4)
+c
Counter({'a': 2})
-c
Counter({'b': 4})
```
写一个计算文本相似的算法，加权相似

```perl
from collections import Counter
import re

def str_sim(str_0,str_1,topn):
    topn = int(topn)
    collect0 = Counter(dict(Counter(str_0).most_common(topn)))
    collect1 = Counter(dict(Counter(str_1).most_common(topn)))       
    jiao = collect0 & collect1
    bing = collect0 | collect1       
    sim = float(sum(jiao.values()))/float(sum(bing.values()))        
    return(sim)         

str_0 = '定位手机定位汽车定位GPS定位人定位位置查询'         
str_1 = '导航定位手机定位汽车定位GPS定位人定位位置查询'         

print(str_sim(str_0,str_1,5))
#0.75
```

##  4. 双向队列-deque
双端队列，可以快速的从另外一侧追加和推出对象,`deque`是一个双向链表，针对list连续的数据结构插入和删除进行优化。它提供了两端都可以操作的序列，这表示在序列的前后你都可以执行添加或删除操作。双向队列(deque)对象支持以下方法：
###  4.1 append()
添加 x 到右端。
```kotlin
from collections import deque

d = deque('ghi')  
d.append('j') 
print(d)
#deque(['g', 'h', 'i', 'j'])
```

###  4.2 appendleft()
添加 x 到左端。
```bash
from collections import deque
d = deque('ghi')  
d.appendleft('f')
print(d)
#deque(['f', 'g', 'h', 'i', 'j'])
```




### 4.3 clear()
移除所有元素，使其长度为0.

```bash
from collections import deque
d = deque('ghi')
d.clear()
print(d)
#deque([])
```
### 4.4 copy()
创建一份浅拷贝。

```bash
from collections import deque
d = deque('xiaoweuge')
y = d.copy()
print(y)
#deque(['x', 'i', 'a', 'o', 'w', 'e', 'u', 'g', 'e'])
```
###  4.5 count()
计算 deque 中元素等于 x 的个数。

```bash
from collections import deque
d = deque('xiaoweuge-shuai')
print(d.count('a'))
#2
```
###  4.6 extend()
扩展deque的右侧，通过添加iterable参数中的元素。

```bash
from collections import deque
a = deque('abc')
b = deque('cd')
a.extend(b)
a
#deque(['a', 'b', 'c', 'c', 'd'])

#与append 的区别
a = deque('abc')
b = deque('cd')
a.append(b)
print(a)
#deque(['a', 'b', 'c', deque(['c', 'd'])])
```

###  4.7 extendleft()
扩展deque的左侧，通过添加`iterable`参数中的元素。注意，左添加时，在结果中iterable参数中的顺序将被反过来添加。

```bash
from collections import deque
a = deque('abc')
b = deque('cd')
a.extendleft(b)
print(a)
#deque(['d', 'c', 'a', 'b', 'c'])
```
###  4.8 index()
返回 x 在 deque 中的位置（在索引 `start` 之后，索引 stop 之前）。 返回第一个匹配项，如果未找到则引发 `ValueError`。

```bash
from collections import deque
d = deque('xiaoweuge')
a=d.index('w')
print(a)
#4
```

###  4.9 insert()
在位置 i 插入 x 。

如果插入会导致一个限长 `deque` 超出长度 maxlen 的话，就引发一个 `IndexError`。

```bash
from collections import deque
a = deque('abc')
e=a.insert(1,'X')
print(e)
print(a)
#None
#deque(['a', 'X', 'X', 'b', 'c'])
```

###  4.10 pop()
移去并且返回一个元素，deque 最右侧的那一个。 如果没有元素的话，就引发一个 IndexError。

```bash
from collections import deque
d = deque('xiaoweuge')
a = d.pop()
print(a)
print(d)
#e
#deque(['x', 'i', 'a', 'o', 'w', 'e', 'u', 'g'])

```
### 4.11 popleft()

```bash
from collections import deque
d = deque('xiaoweuge')
a = d.popleft()
print(a)
print(d)
#x
#deque(['i', 'a', 'o', 'w', 'e', 'u', 'g', 'e'])

```
###  4.12 remove(value)
移除找到的第一个 value。 如果没有的话就引发 ValueError。

```bash
from collections import deque
a = deque('abca')
a.remove('a')
print(a)
#deque(['b', 'c', 'a'])
```
###  4.13 reverse()
将deque逆序排列。返回 None 。

```bash
#逆序排列
from collections import deque
d = deque('ghi') # 创建一个deque
a = list(reversed(d))
b = deque(reversed(d))

print(a)
print(b)
#['i', 'h', 'g']
#deque(['i', 'h', 'g'])
```
###  4.14 rotate()
向右循环移动 n 步。 如果 n 是负数，就向左循环。

如果deque不是空的，向右循环移动一步就等价于 `d.appendleft(d.pop())` ， 向左循环一步就等价于 `d.append(d.popleft())` 。

```bash
from collections import deque
# 向右边挤一挤
d = deque('ghijkl')
d.rotate(1)                      
print(d)
#deque(['l', 'g', 'h', 'i', 'j', 'k'])

# 向左边挤一挤
d.rotate(-1)                     
print(d)
#deque(['g', 'h', 'i', 'j', 'k', 'l'])

#看一个更明显的
x = deque('12345')
print(x)
deque(['1', '2', '3', '4', '5'])
x.rotate()
print(x)
#deque(['5', '1', '2', '3', '4'])

d = deque(['12','av','cd'])
d.rotate(1)
print(d)
#deque(['cd', '12', 'av'])
```

###  4.15 maxlen
Deque的最大尺寸，如果没有限定的话就是 None 。

```bash
from collections import deque

d=deque(maxlen=10)
for i in range(20):
   d.append(i)
print(d)  
#deque([10, 11, 12, 13, 14, 15, 16, 17, 18, 19], maxlen=10)
```
除了以上操作，deque还支持迭代、封存、`len(d)`、`reversed(d)`、`copy.deepcopy(d)`、`copy.copy(d)`、成员检测运算符 in 以及下标引用例如通过 d[0] 访问首个元素等。 索引访问在两端的复杂度均为 O(1) 但在中间则会低至 O(n)。 如需快速随机访问，请改用列表。

Deque从版本3.5开始支持 `__add__()`, `__mul__()`, 和 `__imul__()` 。

##  5. 有序字典-OrderedDict
使用dict时，Key是无序的。在对dict做迭代时，我们无法确定Key的顺序。如果要保持Key的顺序，可以用`OrderedDict`。
有序词典就像常规词典一样，但有一些与排序操作相关的额外功能,`popitem()` 方法有不同的签名。它接受一个可选参数来指定弹出哪个元素。`move_to_end()` 方法，可以有效地将元素移动到任一端。

有序词典就像常规词典一样，但有一些与排序操作相关的额外功能。由于内置的 `dict` 类获得了记住插入顺序的能力（在 Python 3.7 中保证了这种新行为），它们变得不那么重要了。

一些与 dict 的不同仍然存在：

 - 常规的 dict 被设计为非常擅长映射操作。 跟踪插入顺序是次要的。
 - OrderedDict 旨在擅长重新排序操作。 空间效率、迭代速度和更新操作的性能是次要的。
 - 算法上， OrderedDict 可以比 dict 更好地处理频繁的重新排序操作。 这使其适用于跟踪最近的访问（例如在 LRU cache 中）。
 - 对于 `OrderedDict` ，相等操作检查匹配顺序。
 - `OrderedDict` 类的 `popitem()` 方法有不同的签名。它接受一个可选参数来指定弹出哪个元素。
 - `OrderedDict` 类有一个 `move_to_end()` 方法，可以有效地将元素移动到任一端。
 - Python 3.8之前， dict 缺少 `__reversed__()` 方法。

| 传统字典方法      | OrderedDict方法   | 差异                                                    |
|-------------|-----------------|-------------------------------------------------------|
| clear       | clear           |                                                       |
| copy        | copy            |                                                       |
| fromkeys    | fromkeys        |                                                       |
| get         | get             |                                                       |
| items       | items           |                                                       |
| keys        | keys            |                                                       |
| pop         | pop             |                                                       |
| popitem     | popitem         | OrderedDict 类的 popitem() 方法有不同的签名。它接受一个可选参数来指定弹出哪个元素。 |
| setdefault  | setdefault      |                                                       |
| update      | update          |                                                       |
| values      | values          |                                                       |
|| move_to_end | 可以有效地将元素移动到任一端。 |                                                       


###  5.1 popitem
语法：`popitem(last=True)`

功能：有序字典的 `popitem()` 方法移除并返回一个 (key, value) 键值对。 如果 last 值为真，则按 LIFO 后进先出的顺序返回键值对，否则就按 FIFO 先进先出的顺序返回键值对。

```bash
from collections import OrderedDict
d = OrderedDict.fromkeys('abcde')
d.popitem()
print(d)
#OrderedDict([('a', None), ('b', None), ('c', None), ('d', None)])

d = OrderedDict.fromkeys('abcde')
a = ''.join(d.keys())
print(a)
#abcde

d.popitem(last=False)
b = ''.join(d.keys())
print(b)
#bcde
```

###  5.2 move_to_end
可以有效地将元素移动到任一端。
```bash
from collections import OrderedDict
d = OrderedDict.fromkeys('abcde')
d.move_to_end('b')   #最右端
e = ''.join(d.keys())
print(e)
#acdeb
print(d)
OrderedDict([('a', None), ('c', None), ('d', None), ('e', None), ('b', None)])


d.move_to_end('b', last=False)
g = ''.join(d.keys())
print(g)
#'bacde'
```

###  5.3 reversed()
相对于通常的映射方法，有序字典还另外提供了逆序迭代的支持，通过`reversed()` 。

```bash
from collections import OrderedDict
d = OrderedDict.fromkeys('abcde')
j = list(reversed(d))
print(j)
#['e', 'd', 'c', 'b', 'a']
```
##  6. 命名元组-namedtuple
Python 的collections模块提供了一个名为的[工厂函数](https://en.wikipedia.org/wiki/Factory_%28object-oriented_programming%29)[namedtuple()](https://realpython.com/python-namedtuple/)，该函数专门设计用于在处理元组时使您的代码更加Pythonic 。使用[namedtuple()](https://realpython.com/python-namedtuple/)，您可以创建不可变的序列类型，允许您使用描述性字段名称和点表示法而不是不清楚的整数索引来访问它们的值。

1、参数介绍

```bash
namedtuple(typename,field_names,*,verbose=False, rename=False, module=None)
```

 - `typename`：该参数指定所创建的`tuple`子类的类名，相当于用户定义了一个新类。
 - `field_names`：该参数是一个字符串序列，如 ['x'，'y']。此外，field_names也可直接使用单个字符串代表所有字段名，多个字段名用空格、逗号隔开，如 'x y' 或 'x,y'。任何有效的 Python标识符都可作为字段名（不能以下画线开头）。有效的标识符可由字母、数字、下画线组成，但不能以数字、下面线开头，也不能是关键字（如 return、global、pass、raise 等）。
 - `rename`：如果将该参数设为 True，那么无效的字段名将会被自动替换为位置名。例如指定`['abc','def','ghi','abc']`，它将会被替换为 `['abc', '_1','ghi','_3']`，这是因为 def字段名是关键字，而 abc 字段名重复了。
 - `verbose`：如果该参数被设为 True，那么当该子类被创建后，该类定义就被立即打印出来。
 - `module`：如果设置了该参数，那么该类将位于该模块下，因此该自定义类的 `__module__` 属性将被设为该参数值。

###  6.1 与类对比
`namedtuple`是一个非常简单的元类，通过它我们可以非常方便地定义我们想要的类。
它的用法很简单，我们直接来看例子。比如如果我们想要定义一个学生类，这个类当中有name、score、age这三个字段，那么这个类会写成：

```bash
class Student:
    def __init__(self, name=None, score=None, age=None):
        self.name = name
        self.score = score
        self.age = age
```
namedtuple代码：

```bash
from collections import namedtuple
# 这个是类，columns也可以写成'name score age'，即用空格分开
Student = namedtuple('Student', ['name', 'score', 'age'])

# 这个是实例
student = Student(name='xiaoming', score=99, age=10)
print(student.name)
```

###  6.2 加法
```bash
from collections import namedtuple
# 定义命名元组类：Point
Point = namedtuple('Point', ['x', 'y'])
# 初始化Point对象，即可用位置参数，也可用命名参数
p = Point(11, y=22)
# 像普通元组一样用根据索引访问元素
print(p[0] + p[1]) 
33
#执行元组解包，按元素的位置解包
a, b = p
print(a, b) 
11, 22
#根据字段名访问各元素
print(p.x + p.y) 
33
print(p) 
Point(x=11, y=22)
```

>  备注:在Python中，带有前导下划线的方法通常被认为是“私有的”。但是，namedtuple提供的其他方法(如`._asdict()`、`._make()`、`._replace()`等)是公开的。

除了继承元组的方法，命名元组还支持三个额外的方法和两个属性。为了防止字段名冲突，方法和属性以下划线开始。

### 6.3  三个方法

#### 6.3.1  _make(iterable)

```bash
from collections import namedtuple

Point = namedtuple('Point', ['x', 'y'])

t = [14, 55]
Point._make(t)
```
####  6.3.2 _asdict()
返回一个新的 dict ，它将字段名称映射到它们对应的值：

```bash
from collections import namedtuple

Point = namedtuple('Point', ['x', 'y'])
p = Point(x=11, y=22)
p._asdict()
#OrderedDict([('x', 11), ('y', 22)])
```

#### 6.3.3  _replace(**kwargs)
返回一个新的命名元组实例，并将指定域替换为新的值

```bash
p = Point(x=11, y=22)
p._replace(x=33)

Point(x=33, y=22)
```

###  6.4 两个属性

#### 6.4.1 _fields

字符串元组列出了字段名。用于提醒和从现有元组创建一个新的命名元组类型。

```bash

```

#### 6.4.2 _field_defaults

字典将字段名称映射到默认值。

```bash
Account = namedtuple('Account', ['type', 'balance'], defaults=[0])
Account._field_defaults
{'balance': 0}
Account('premium')
Account(type='premium', balance=0)
```
###  6.5 defaults参数

```bash
>>> from collections import namedtuple

>>> # Define default values for fields
>>> Person = namedtuple("Person", "name job", defaults=["Python Developer"])
>>> person = Person("Jane")
>>> person
Person(name='Jane', job='Python Developer')

>>> # Create a dictionary from a named tuple
>>> person._asdict()
{'name': 'Jane', 'job': 'Python Developer'}

>>> # Replace the value of a field
>>> person = person._replace(job="Web Developer")
>>> person
Person(name='Jane', job='Web Developer')
```


### 6.6 商取余divmod()函数
为了正确看待代码可读性问题，请考虑[divmod()](https://docs.python.org/3/library/functions.html#divmod). 这个内置函数接受两个（非复数）数字并返回一个元组，其中包含输入值的整数除法得到的商和余数：
```bash
>>> divmod(12, 5)
(2, 2)
```
namedtuple

```bash
>>> from collections import namedtuple

>>> def custom_divmod(x, y):
...     DivMod = namedtuple("DivMod", "quotient remainder")
...     return DivMod(*divmod(x, y))
...

>>> result = custom_divmod(12, 5)
>>> result
DivMod(quotient=2, remainder=2)

>>> result.quotient
2
>>> result.remainder
2
```
###  6.7 两个坐标 
创建Point具有两个坐标 (x和y)的示例 2D 的不同方法namedtuple()：

```bash
>>> from collections import namedtuple

>>> # Use a list of strings as field names
>>> Point = namedtuple("Point", ["x", "y"])
>>> point = Point(2, 4)
>>> point
Point(x=2, y=4)

>>> # Access the coordinates
>>> point.x
2
>>> point.y
4
>>> point[0]
2

>>> # Use a generator expression as field names
>>> Point = namedtuple("Point", (field for field in "xy"))
>>> Point(2, 4)
Point(x=2, y=4)

>>> # Use a string with comma-separated field names
>>> Point = namedtuple("Point", "x, y")
>>> Point(2, 4)
Point(x=2, y=4)

>>> # Use a string with space-separated field names
>>> Point = namedtuple("Point", "x y")
>>> Point(2, 4)
Point(x=2, y=4)
```
##  7. 默认字典-defaultdict

在Python字典中收集数据通常是很有用的。

在字典中获取一个 key 有两种方法, 第一种 get , 第二种 通过 [] 获取.

使用dict时，如果引用的Key不存在，就会抛出KeyError。如果希望key不存在时，返回一个默认值，就可以用`defaultdict`。

```bash
>>> favorites = {"pet": "dog", "color": "blue", "language": "Python"}

>>> favorites["fruit"]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'fruit'
```

```bash
>>> favorites = {"pet": "dog", "color": "blue", "language": "Python"}

>>> favorites.setdefault("fruit", "apple")
'apple'

>>> favorites
{'pet': 'dog', 'color': 'blue', 'language': 'Python', 'fruit': 'apple'}

>>> favorites.setdefault("pet", "cat")
'dog'

>>> favorites
{'pet': 'dog', 'color': 'blue', 'language': 'Python', 'fruit': 'apple'}
```

在此示例中，您使用`.setdefault()`为 生成默认值`fruit`。由于该键在 中不存在`favorites`，`.setdefault()`因此创建它并为其分配 的值`apple`。如果您`.setdefault()`使用现有键调用，则调用不会影响字典，并且您的键将保留原始值而不是默认值。


当我使用普通的字典时，用法一般是dict={},添加元素的只需要dict[element] =value即，调用的时候也是如此，dict[element] = xxx,但前提是element字典里，如果不在字典里就会报错

这时defaultdict就能排上用场了，defaultdict的作用是在于，当字典里的key不存在但被查找时，返回的不是keyError而是一个默认值

```bash
>>> from collections import defaultdict
>>> dd = defaultdict(lambda: 'N/A')
>>> dd['key1'] = 'abc'
>>> dd['key1'] # key1存在
'abc'
>>> dd['key2'] # key2不存在，返回默认值
'N/A'
```
####  7.1 int()
int()您可以创建一个合适的计数器来计算不同的对象：
示例1

```bash
>>> from collections import defaultdict

>>> counter = defaultdict(int)
>>> counter
defaultdict(<class 'int'>, {})
>>> counter["dogs"]
0
>>> counter
defaultdict(<class 'int'>, {'dogs': 0})

>>> counter["dogs"] += 1
>>> counter["dogs"] += 1
>>> counter["dogs"] += 1
>>> counter["cats"] += 1
>>> counter["cats"] += 1
>>> counter
defaultdict(<class 'int'>, {'dogs': 3, 'cats': 2})
```
示例2
```bash
from collections import defaultdict

s = 'mississippi'
d = defaultdict(int)
for k in s:
    d[k] += 1
j = sorted(d.items())
print(j)
#[('i', 4), ('m', 1), ('p', 2), ('s', 4)]
```
####  7.2 set

```bash
from collections import defaultdict

s = [('red', 1), ('blue', 2), ('red', 3), ('blue', 4), ('red', 1), ('blue', 4)]
d = defaultdict(set)
for k, v in s:
    d[k].add(v)

y = sorted(d.items())
print(y)
#[('blue', {2, 4}), ('red', {1, 3})]
```
####  7.3 list()

```bash
>>> from collections import defaultdict

>>> pets = [
...     ("dog", "Affenpinscher"),
...     ("dog", "Terrier"),
...     ("dog", "Boxer"),
...     ("cat", "Abyssinian"),
...     ("cat", "Birman"),
... ]

>>> group_pets = defaultdict(list)

>>> for pet, breed in pets:
...     group_pets[pet].append(breed)
...

>>> for pet, breeds in group_pets.items():
...     print(pet, "->", breeds)
...
dog -> ['Affenpinscher', 'Terrier', 'Boxer']
cat -> ['Abyssinian', 'Birman']
```

##  8. 映射链-ChainMap
1、ChainMap是什么
ChainMap最基本的使用，可以用来合并两个或者更多个字典，当查询的时候，从前往后依次查询。

ChainMap：将多个字典视为一个，解锁Python超能力。

ChainMap是由Python标准库提供的一种数据结构，允许你将多个字典视为一个。换句话说:ChainMap是一个基于多dict的可更新的视图，它的行为就像一个普通的dict。

ChainMap类用于快速链接多个映射，以便将它们视为一个单元。它通常比创建新字典和多次调用update()快得多。

我知道的用例包括：

 - 通过多个字典搜索
 - 提供链缺省值
 - 经常计算字典子集的性能关键的应用程序

```bash
>>> from collections import ChainMap

>>> cmd_proxy = {}  # The user doesn't provide a proxy
>>> local_proxy = {"proxy": "proxy.local.com"}
>>> global_proxy = {"proxy": "proxy.global.com"}

>>> config = ChainMap(cmd_proxy, local_proxy, global_proxy)
>>> config["proxy"]
'proxy.local.com'
```
ChainMap允许您为应用程序的代理配置定义适当的优先级。键查找搜索cmd_proxy，然后local_proxy，最后global_proxy，返回手头键的第一个实例。在此示例中，用户未在命令行中提供代理，因此您的应用程序使用local_proxy.

###  8.1 maps
通常，ChainMap对象的行为类似于常规`dict`对象。但是，它们具有一些附加功能。例如，它们有一个`.maps`包含内部映射列表的公共属性：

```bash
>>> from collections import ChainMap

>>> numbers = {"one": 1, "two": 2}
>>> letters = {"a": "A", "b": "B"}

>>> alpha_nums = ChainMap(numbers, letters)
>>> alpha_nums.maps
[{'one': 1, 'two': 2}, {'a': 'A', 'b': 'B'}]
```
instance 属性.maps使您可以访问内部映射列表。此列表是可更新的。您可以手动添加和删除映射、遍历列表等等。

###  8.2 new_child() 与parents
另外，ChainMap提供一个.new_child()方法和一个.parents属性：

```bash
>>> from collections import ChainMap

>>> dad = {"name": "John", "age": 35}
>>> mom = {"name": "Jane", "age": 31}
>>> family = ChainMap(mom, dad)
>>> family
ChainMap({'name': 'Jane', 'age': 31}, {'name': 'John', 'age': 35})

>>> son = {"name": "Mike", "age": 0}
>>> family = family.new_child(son)

>>> for person in family.maps:
...     print(person)
...
{'name': 'Mike', 'age': 0}
{'name': 'Jane', 'age': 31}
{'name': 'John', 'age': 35}

>>> family.parents
ChainMap({'name': 'Jane', 'age': 31}, {'name': 'John', 'age': 35})
```
使用.new_child()，您可以创建一个新ChainMap对象，其中包含一个新映射 ( son)，后跟当前实例中的所有映射。作为第一个参数传递的映射成为映射列表中的第一个映射。如果您不传递地图，则该方法使用空字典。

该parents属性返回一个新ChainMap对象，其中包含当前实例中除第一个之外的所有地图。当您需要跳过键查找中的第一个映射时，这很有用。

###  8.3 ChainMap 变异操作
最后一个要强调的特性ChainMap是变异操作，例如更新键、添加新键、删除现有键、弹出键和清除字典，作用于内部映射列表中的第一个映射：

```bash
>>> from collections import ChainMap

>>> numbers = {"one": 1, "two": 2}
>>> letters = {"a": "A", "b": "B"}

>>> alpha_nums = ChainMap(numbers, letters)
>>> alpha_nums
ChainMap({'one': 1, 'two': 2}, {'a': 'A', 'b': 'B'})

>>> # Add a new key-value pair
>>> alpha_nums["c"] = "C"
>>> alpha_nums
ChainMap({'one': 1, 'two': 2, 'c': 'C'}, {'a': 'A', 'b': 'B'})

>>> # Pop a key that exists in the first dictionary
>>> alpha_nums.pop("two")
2
>>> alpha_nums
ChainMap({'one': 1, 'c': 'C'}, {'a': 'A', 'b': 'B'})

>>> # Delete keys that don't exist in the first dict but do in others
>>> del alpha_nums["a"]
Traceback (most recent call last):
  ...
KeyError: "Key not found in the first mapping: 'a'"

>>> # Clear the dictionary
>>> alpha_nums.clear()
>>> alpha_nums
ChainMap({}, {'a': 'A', 'b': 'B'})
```

##  9. 自定义内置插件：UserString、UserList和UserDict
有时您需要自定义内置类型，例如字符串、列表和字典，以添加和修改某些行为。从Python 2.2开始，您可以通过直接对这些类型进行子类化来做到这一点。但是，您可能会在使用这种方法时遇到一些问题，稍后您会看到。

Pythoncollections提供了三个方便的包装类来模拟内置数据类型的行为：

 - UserString
 - UserList
 - UserDict

通过结合常规方法和特殊方法，您可以使用这些类来模拟和自定义字符串、列表和字典的行为。

```bash
>>> class LowerDict(dict):
...     def __setitem__(self, key, value):
...         key = key.lower()
...         super().__setitem__(key, value)
...

>>> ordinals = LowerDict({"FIRST": 1, "SECOND": 2})
>>> ordinals["THIRD"] = 3
>>> ordinals.update({"FOURTH": 4})

>>> ordinals
{'FIRST': 1, 'SECOND': 2, 'third': 3, 'FOURTH': 4}

>>> isinstance(ordinals, dict)
True
```
[]当您使用带方括号 ( )的字典式赋值插入新键时，此字典可以正常工作。但是，当您将初始字典传递给[类构造函数](https://realpython.com/python-class-constructor/)或使用[.update()](https://docs.python.org/3/library/stdtypes.html#dict.update). 这意味着您需要覆盖.[__init__()](https://docs.python.org/3/reference/datamodel.html#object.__init__), .update()，可能还有其他一些方法才能使您的自定义字典正常工作。

现在看一下相同的字典，但UserDict用作基类：

```bash
>>> from collections import UserDict

>>> class LowerDict(UserDict):
...     def __setitem__(self, key, value):
...         key = key.lower()
...         super().__setitem__(key, value)
...

>>> ordinals = LowerDict({"FIRST": 1, "SECOND": 2})
>>> ordinals["THIRD"] = 3
>>> ordinals.update({"FOURTH": 4})

>>> ordinals
{'first': 1, 'second': 2, 'third': 3, 'fourth': 4}

>>> isinstance(ordinals, dict)
False
```
有用！您的自定义字典现在将所有新键转换为小写字母，然后再将它们插入字典。请注意，由于您不dict直接继承自，因此您的类不会返回dict上面示例中的实例。

UserDict将常规字典存储在名为 的实例属性中.data。然后它围绕该字典实现其所有方法。UserList并UserString以相同的方式工作，但它们的.data属性分别包含一个list和一个str对象。

如果您需要自定义这些类中的任何一个，那么您只需要覆盖适当的方法并根据需要更改它们的功能。

通常，您应该使用UserDict, UserList, 并且UserString当您需要一个行为几乎与底层包装的内置类相同的类并且您想要自定义其标准功能的某些部分时。

使用这些类而不是内置等效类的另一个原因是访问底层.data属性以直接操作它。

直接从内置类型继承的能力在很大程度上取代了UserDict,UserList和UserString. 然而，内置类型的内部实现使得很难在不重写大量代码的情况下安全地继承它们。在大多数情况下，使用来自collections. 它将使您免于几个问题和奇怪的行为。

##  10. 结论
在 Python 的collections模块中，您可以使用几种专门的容器数据类型来解决常见的编程问题，例如计数对象、创建队列和堆栈、处理字典中缺少的键等等。


---
✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>

 - [python list 列表](https://blog.csdn.net/xixihahalelehehe/article/details/104437743?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164942334216782246440259%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164942334216782246440259&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-6-104437743.nonecase&utm_term=%E5%88%97%E8%A1%A8&spm=1018.2226.3001.4450)
 - [python dict字典](https://blog.csdn.net/xixihahalelehehe/article/details/104488899?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164942342716780264033255%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164942342716780264033255&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-104488899.nonecase&utm_term=%E5%AD%97%E5%85%B8&spm=1018.2226.3001.4450)
 - [pyhon tuple 元组](https://blog.csdn.net/xixihahalelehehe/article/details/104486159?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164942350116782248567035%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164942350116782248567035&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-104486159.nonecase&utm_term=%E5%85%83%E7%BB%84&spm=1018.2226.3001.4450)
 - [python 面向对象](https://blog.csdn.net/xixihahalelehehe/article/details/106245475?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164942364216780271534896%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164942364216780271534896&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-106245475.nonecase&utm_term=%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E7%BC%96%E7%A8%8B&spm=1018.2226.3001.4450)
 
![在这里插入图片描述](https://img-blog.csdnimg.cn/bfef2b0846964030aa34c8073c8bfc1f.gif#pic_center)


