#  python 模块 copy 复制




-----
## 1. copy()与deepcopy()

对于简单的 object，用shallow copy 和 deep copy 没区别；而对于复杂的 object， 如 list 中套着 list 的情况，shallow copy 中的 子list，并未从原 object 真的「独立」出来。也就是说，如果你改变原 object 的子 list 中的一个元素，你的 copy 就会跟着一起变。这跟我们直觉上对「复制」的理解不同。

```python
import copy
from copy import deepcopy
origin = [1,2,[3,4]]
# origin里面有三个元素，1,2,[3,4]
copy1 = copy.copy(origin)
copy2 = deepcopy(origin)
print(copy1 == copy2)   # True
print(copy1 is copy2)   # False
# copy1 和copy2看上去相同，但已经不是同一个object
origin[2][0] = "hey!"
print(origin)   # [1, 2, ['hey!', 4]]
print(copy1)    # [1, 2, ['hey!', 4]]
print(copy2)    # [1, 2, [3, 4]]
```

可以看到 copy1，也就是 shallowcopy 跟着 origin 改变了。而 copy2 ，也就是 deepcopy 并没有变。可以看出deepcopy 更加符合我们对「复制」的直觉定义: 一旦复制出来了，就应该是独立的了。如果我们想要的是一个字面意义的「copy」，那就直接用deepcopy 即可。

## 2. 字典数据类型的copy
       字典数据类型的copy函数，当简单的值替换的时候，原始字典和复制过来的字典之间互不影响，但是当添加，删除等修改操作的时候，两者之间会相互影响。相当于上述的赋值操作，即在原来的对象的上面再贴上一个标签。

值替换(一级object，父对象)，字典数组类型的copy函数复制的互不影响

```python
a = {
    "name":["An","Assan"]
}
b = a.copy()

a["name"] = ["An"]

print(a)
print(b)
del a['name']
print(a)
print(b)
"""
{'name': ['An']}
{'name': ['An', 'Assan']}
{}
{'name': ['An', 'Assan']}
"""
```

值修改（二级object，子对象），字典数组类型的copy函数复制的互相影响

```python
a = {
    "name":["An","Assan"]
}
b = a.copy()

a["name"].append("Hunter")

print(a)
print(b)
a['name'].remove('An')
print(a)
print(b)
"""
{'name': ['An', 'Assan', 'Hunter']}
{'name': ['An', 'Assan', 'Hunter']}
{'name': ['Assan', 'Hunter']}
{'name': ['Assan', 'Hunter']}
"""
```

如果想不改变原有被复制的值，则需要使用deepcopy，复制前后两个变量独立。

```python
a = {
    "name":["An","Assan"]
}
b = deepcopy(a)
a["name"].append("Hunter")
print(a)
print(b)
"""
{'name': ['An', 'Assan', 'Hunter']}
{'name': ['An', 'Assan']}
"""
```

## 3. " = " 即一般意义的复制，浅复制

```python
list1 = [1,2,[3,4]]
list2 = list1
list1.append(5)
print(list1)
# [1, 2, [3, 4], 5]
print(list2)
# [1, 2, [3, 4], 5]
list1[2].remove(3)
print(list1)
# [1, 2, [4], 5]
print(list2)
# [1, 2, [4], 5]
```

## 4. 列表切片等价于深复制

```python
list1 = [1,2,[3,4]]
list2 = list1[:]
list1.append(5)
print(list1)
# [1, 2, [3, 4], 5]
print(list2)
# [1, 2, [3, 4]]
list1[2].remove(3)
print(list1)
# [1, 2, [4], 5]
print(list2)
# [1, 2, [4]]
```
参考链接：

 - [详解copy浅复制和deepcopy深复制](https://blog.csdn.net/brucewong0516/article/details/89838754)
 - [Python 直接赋值、浅拷贝和深度拷贝解析](https://www.runoob.com/w3cnote/python-understanding-dict-copy-shallow-or-deep.html)
 - [copy — Shallow and deep copy operations](https://docs.python.org/3/library/copy.html)
 - [copy in Python (Deep Copy and Shallow Copy)](https://www.geeksforgeeks.org/copy-python-deep-copy-shallow-copy/)

