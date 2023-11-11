


----
## 1. 简介
UUID（全称为Universally Unique IDentifier）是128位的全局唯一标识符。UUID是一个128比特的数值，这个数值可以通过一定的算法计算出来。为了提高效率，常用的UUID可缩短至16位。保证在一定的空间和时间上的唯一性，通常定义用来做唯一标识对象。也称为GUID，全称为：
            UUID —— Universally Unique IDentifier      Python 中叫 UUID
            GUID —— Globally Unique IDentifier          C#  中叫 GUID

 它通过MAC地址、时间戳、命名空间、随机数、伪随机数来保证生成ID的唯一性。

## 2. 五中算法

 - `uuid1()`——基于时间戳 由MAC地址、当前时间戳、随机数生成。可以保证全球范围内的唯一性，
   但MAC的使用同时带来安全性问题，局域网中可以使用IP来代替MAC。
 - `uuid2()`——基于分布式计算环境DCE（Python中没有这个函数） 
   算法与uuid1相同，不同的是把时间戳的前4位置换为POSIX的UID。实际中很少用到该方法。
 - `uuid3()`——基于名字的MD5散列值  通过计算名字和命名空间的MD5散列值得到，保证了同一命名空间中不同名字的唯一性，和不同命名空间的唯一性，但同一命名空间的同一名字生成相同的uuid。

  

 - `uuid4()`——基于随机数 由伪随机数得到，有一定的重复概率，该概率可以计算出来。
 - `uuid5()`——基于名字的SHA-1散列值 算法与uuid3相同，不同的是使用 Secure Hash Algorithm 1 算法

使用方面：

 - 首先，Python中没有基于DCE的，所以uuid2可以忽略；
 - 其次，uuid4存在概率性重复，由无映射性，最好不用；
 - 再次，若在Global的分布式计算环境下，最好用uuid1；
 - 最后，若有名字的唯一性要求，最好用uuid3或uuid5。


##  3. 示例
### 3.1 打印各个算法生成数

```bash
# -*- coding: utf-8 -*-

import uuid

name = "test_name"
namespace = "test_namespace"

print uuid.uuid1()  # 带参的方法参见Python Doc
print uuid.uuid3(namespace, name)
print uuid.uuid4()
print uuid.uuid5(namespace, name)
```
### 3.2 格式化打印

```bash
root@test1:~/python/uuid# cat test3.py
#! /usr/bin/env python3
import uuid

i = 1

while(i< 6):

  print( "No-", i, ' ', uuid.uuid4)
  i = i + 1
```

```bash
root@test1:~/python/uuid# python test3.py
('No-', 1, ' ', <function uuid4 at 0x7f90c0497488>)
('No-', 2, ' ', <function uuid4 at 0x7f90c0497488>)
('No-', 3, ' ', <function uuid4 at 0x7f90c0497488>)
('No-', 4, ' ', <function uuid4 at 0x7f90c0497488>)
('No-', 5, ' ', <function uuid4 at 0x7f90c0497488>)
```


