#  python 模块  json 文本处理



-----
## 1. json介绍
[JSON](https://docs.python.org/3/library/json.html) 是一种轻量级的数据交换格式。易于人阅读和编写。同时也易于机器解析和生成。
数据格式可以简单地理解为键值对的集合（A collection of name/value pairs）。不同的语言中，它被理解为对象（object），纪录（record），结构（struct），字典（dictionary），哈希表（hash table），有键列表（keyed list），或者关联数组 （associative array）。
值的有序列表（An ordered list of values）。在大部分语言中，它被理解为数组（array）。

## 2. 方法
### 2.1 json.dumps
dump的功能就是把Python对象encode为json对象，一个编码过程。注意json模块提供了json.dumps和json.dump方法，区别是dump直接到文件，而dumps到一个字符串，这里的s可以理解为string。

### 2.2 json.dump
不仅可以把Python对象编码为string，还可以写入文件。因为我们不能把Python对象直接写入文件，这样会报错TypeError: expected a string or other character buffer object，我们需要将其序列化之后才可以。

### 2.3 json.loads
从Python内置对象dump为json对象我们知道如何操作了，那如何从json对象decode解码为Python可以识别的对象呢？是的用json.loads方法，当然这个是基于string的，如果是文件，我们可以用json.load方法。

实例1：

```python
#!/usr/bin/python

import json

data = [ { 'a':'A', 'b':(2, 4), 'c':3.0} ]
print 'DATA:', repr(data)

data_string = json.dumps(data)
print 'JSON:', data_string

print type(data)
print type(data_string)

with open('output.json','w') as fp:
	json.dump(data,fp)

print type(fp)
decoded_json = json.loads(data_string)
print type(decoded_json)

print decoded_json[0]['a']



loaded_json = json.load(fp)
print type(loaded_json)
print loaded_json()
```



json和Python对象转换过程中，数据类型不完全一致，有对应。

| Python         | Json   |
|----------------|--------|
| dict           | object |
| list,tuple     | array  |
| str,unicode    | string |
| int,long,float | number |
| TRUE           | TRUE   |
| FALSE          | FALSE  |
| None           | null   |

## 3. json.dumps常用参数
一些参数，可以让我们更好地控制输出。常见的比如sort_keys，indent，separators，skipkeys等。
### 3.1 sort_keys
输出时字典的是按键值排序的，而不是随机的。

```python
import json

data = [ { 'a':'A', 'b':(2, 4), 'c':3.0 } ]
print 'DATA:', repr(data)

unsorted = json.dumps(data)
print 'JSON:', json.dumps(data)
print 'SORT:', json.dumps(data, sort_keys=True)
```
输出：

```python
$ python js2.py
DATA: [{'a': 'A', 'c': 3.0, 'b': (2, 4)}]
JSON: [{"a": "A", "c": 3.0, "b": [2, 4]}]
SORT: [{"a": "A", "b": [2, 4], "c": 3.0}]
```

### 3.2 indent
就是更个缩进，让我们更好地看清结构。

```python
import json

data = [ { 'a':'A', 'b':(2, 4), 'c':3.0 } ]
print 'DATA:', repr(data)

print 'NORMAL:', json.dumps(data, sort_keys=True)
print 'INDENT:', json.dumps(data, sort_keys=True, indent=2)
```
输出:

```python
$ python js3.py
DATA: [{'a': 'A', 'c': 3.0, 'b': (2, 4)}]
NORMAL: [{"a": "A", "b": [2, 4], "c": 3.0}]
INDENT: [
  {
    "a": "A", 
    "b": [
      2, 
      4
    ], 
    "c": 3.0
  }
]
```

### 3.3 separators
提供分隔符，可以出去白空格，输出更紧凑，数据更小。默认的分隔符是(', ', ': ')，有白空格的。不同的dumps参数，对应文件大小一目了然。

```python
data = [ { 'a':'A', 'b':(2, 4), 'c':3.0 } ]
print 'DATA:', repr(data)
print 'repr(data)             :', len(repr(data))
print 'dumps(data)            :', len(json.dumps(data))
print 'dumps(data, indent=2)  :', len(json.dumps(data, indent=2))
print 'dumps(data, separators):', len(json.dumps(data, separators=(',',':')))
```
输出：

```python
DATA: [{'a': 'A', 'c': 3.0, 'b': (2, 4)}]
repr(data)             : 35
dumps(data)            : 35
dumps(data, indent=2)  : 76
dumps(data, separators): 29
```

### 3.4 ensure_ascii
支持中文
print json.dumps(servies,ensure_ascii=False)


## 4. error错误
json需要字典的的键是字符串，否则会抛出ValueError。

```python
data = [ { 'a':'A', 'b':(2, 4), 'c':3.0, ('d',):'D tuple' } ]

print 'First attempt'
try:
    print json.dumps(data)
except (TypeError, ValueError) as err:
    print 'ERROR:', err

print
print 'Second attempt'
print json.dumps(data, skipkeys=True)
```
输出：

```python
First attempt
ERROR: keys must be a string

Second attempt
[{"a": "A", "c": 3.0, "b": [2, 4]}]
```

参考：

 - [runoob Python JSON](https://www.runoob.com/python/python-json.html)
 - [w3schools.com Python JSON](https://www.w3schools.com/python/python_json.asp)
 - [programiz Python JSON](https://www.programiz.com/python-programming/json)
 - [Working With JSON Data in Python](https://realpython.com/python-json/)

