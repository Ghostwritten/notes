#  python 模块 random随机生成





------------
## 1. 简介
random() 方法返回随机生成的一个实数，它在[0,1)范围内。

##  2.示例
###  2.1 random.random()
```bash
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
import random
 
# 生成第一个随机数
print "random() : ", random.random()
 
# 生成第二个随机数
print "random() : ", random.random()
```

以上实例运行后输出结果为：

```bash
random() :  0.281954791393
random() :  0.309090465205
```

### 2.2 randint(a, b)
用来生成[a,b]之间的随意整数，包括两个边界值。

```bash
In [12]: import random

In [13]: random.randint(1,6)
Out[13]: 1
```

### 2.3 random.uniform(a,b)
用来生成[a,b]之间的随意浮点数，包括两个边界值。

```bash
In [14]: import random

In [15]: random.uniform(1,6)
Out[15]: 5.497873150216069
```

### 2.4 choice(seq)
从一个非空序列选出随机一个元素。seq泛指list，tuple，字符串等

```bash
In [16]: import random

In [17]: List = [1,2,3,4,5,6]
In [18]: random.choice(List)
Out[18]: 1
```

### 2.5 randrange(start, stop[, step = 1])
这个就是random和range函数的合二为一了。但注意，range用法有变。

```bash
In [27]: import random

In [28]: random.randrange(1,6)
Out[28]: 3
```

### 2.6 random.shuffle(x[,random])
正如函数名所表示的意思，shuffle，洗牌，将一个列表中的元素打乱。

```bash
In [36]: import random

In [37]: List = [1,2,3,4,5,6]
In [38]: random.shuffle(List)
In [39]: print(List)
[2, 1, 6, 4, 5, 3]
```

### 2.7 andom.sample(sequence,k)
sample，样品，从有序列表中选k个作为一个片段返回。

```bash
In [41]: import random

In [42]: List = [1,2,3,4,5,6]
In [43]: random.sample(List,3)
Out[43]: [4, 6, 3]
```

### 2.8 random.seed ( [x] )
x:改变随机数生成器的种子seed。如果你不了解其原理，你不必特别去设定seed，Python会帮你选择seed。使用同一个种子，每次生成的随机数序列都是相同的。

```bash
In [48]: import random

In [49]: random.seed(10)
In [50]: print("Random number with seed 10: ", random.random())
Random number with seed 10:  0.5714025946899135

In [51]: random.seed(10)
In [52]: print("Random number with seed 10: ", random.random())
Random number with seed 10:  0.5714025946899135

In [53]: random.seed(10)
In [54]: print("Random number with seed 10: ", random.random())
Random number with seed 10:  0.5714025946899135
```
参考：

 - [Python - Random Module](https://www.tutorialsteacher.com/python/random-module)
 - [runoob Python random() 函数](https://www.runoob.com/python/func-number-random.html)
 - [w3schools Python Random Module](https://www.w3schools.com/python/module_random.asp)

