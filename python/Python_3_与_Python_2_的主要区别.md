![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/d7e523c0e3ae428b9055ada4614fd6f0.png)




Python 是一门强大且广泛使用的编程语言。从 Python 2 到 Python 3 的过渡引入了许多重要的变化，使得 Python 3 成为现代开发的首选版本。本文将总结 Python 3 与 Python 2 的主要区别，并参考官方文档对关键点进行分析。

---

## 1. 语法与关键字

### `print` 函数
在 Python 2 中，`print` 是一个语句，而在 Python 3 中，它被改为函数。

**Python 2**:
```python
print "Hello, World!"
```

**Python 3**:
```python
print("Hello, World!")
```

### 整数除法
在 Python 2 中，整数除法会进行向下取整，而 Python 3 中则返回浮点数。

**Python 2**:
```python
print 5 / 2  # 输出 2
```

**Python 3**:
```python
print(5 / 2)  # 输出 2.5
```

如果希望在 Python 3 中获得整数除法的结果，可以使用 `//`：
```python
print(5 // 2)  # 输出 2
```

---

## 2. 字符串处理

### 默认字符串类型
- Python 2 默认使用 ASCII 编码，字符串为 `str` 类型。
- Python 3 默认使用 Unicode 编码，字符串为 `str` 类型。

**Python 2**:
```python
s = "你好"  # ASCII 可能导致编码错误
```

**Python 3**:
```python
s = "你好"  # 默认支持 Unicode，无需额外处理
```

如果在 Python 3 中处理二进制数据，可以使用 `bytes` 类型：
```python
b = b"binary data"
```

### 字符串格式化
Python 3 引入了更强大的格式化方法 `str.format()` 和 f-string。

**Python 2**:
```python
print("%s is %d years old" % ("Alice", 30))
```

**Python 3**:
```python
print("{} is {} years old".format("Alice", 30))
# 或使用 f-string
name, age = "Alice", 30
print(f"{name} is {age} years old")
```

---

## 3. 输入函数

在 Python 2 中，`input()` 将输入解析为代码，而 `raw_input()` 返回字符串。

**Python 2**:
```python
name = raw_input("Enter your name: ")  # 返回字符串
```

**Python 3**:
```python
name = input("Enter your name: ")  # 默认返回字符串
```

---

## 4. 迭代器和生成器

### `range` 函数
在 Python 2 中，`range()` 返回列表；在 Python 3 中，`range()` 返回一个生成器对象，更节省内存。

**Python 2**:
```python
print(range(5))  # 输出 [0, 1, 2, 3, 4]
```

**Python 3**:
```python
print(range(5))  # 输出 range(0, 5)
print(list(range(5)))  # 转为列表 [0, 1, 2, 3, 4]
```

### `map`, `filter`, `zip`
在 Python 2 中，这些函数返回列表，而在 Python 3 中返回迭代器。

**Python 2**:
```python
print(map(str, [1, 2, 3]))  # 输出 ['1', '2', '3']
```

**Python 3**:
```python
print(list(map(str, [1, 2, 3])))  # 需要显式转换为列表
```

---

## 5. 标准库变化

### `urllib` 模块
- 在 Python 2 中，`urllib` 和 `urllib2` 是分开的模块。
- 在 Python 3 中，功能被整合到了 `urllib.request` 和 `urllib.error` 中。

**Python 2**:
```python
import urllib2
response = urllib2.urlopen("http://example.com")
```

**Python 3**:
```python
import urllib.request
response = urllib.request.urlopen("http://example.com")
```

### `configparser` 模块
在 Python 3 中，`ConfigParser` 被重命名为 `configparser`，并改进了功能。

---

## 6. 异常处理

Python 3 中，异常必须使用 `as` 关键字绑定到变量。

**Python 2**:
```python
try:
    x = 1 / 0
except ZeroDivisionError, e:
    print e
```

**Python 3**:
```python
try:
    x = 1 / 0
except ZeroDivisionError as e:
    print(e)
```

---

## 7. 移除的功能

- **`print` 语句**：被 `print()` 函数取代。
- **`long` 类型**：Python 3 将所有整数合并为 `int` 类型。
- **`xrange`**：被 `range` 取代。
- **`<>` 比较运算符**：在 Python 3 中必须使用 `!=`。

---

## 8. 其他重要改进

### 数据库操作
Python 3 中的数据库模块如 `sqlite3` 默认支持 Unicode，更适合现代开发需求。

### 多线程与并发
Python 3 引入了 `concurrent.futures` 模块，简化了多线程和多进程编程。

### 类型注解
Python 3 支持类型注解，使代码更具可读性和可维护性。

**示例**:
```python
def greet(name: str) -> str:
    return f"Hello, {name}"
```

---

## 9. 总结

从 Python 2 到 Python 3 的变化是显著的，尤其是对语法一致性、性能优化和现代化开发需求的支持。虽然 Python 2 已经在 2020 年停止支持，但了解它与 Python 3 的差异仍有助于维护旧代码或迁移项目。对于新开发，推荐使用 Python 3 的最新版本，以充分利用其特性和改进。

**参考资料**:
- [Python 官方文档](https://docs.python.org/3/index.html)


