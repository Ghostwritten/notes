![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/13367d3dd2744954b4878d90a013871d.png)





在 Python 3 中，输入与输出是程序与外界交互的重要方式。无论是读取用户输入、打印结果，还是处理文件，Python 3 提供了一组强大且直观的工具。本篇博客将优雅地总结 Python 3 的输入与输出方法，帮助您更好地理解和应用。

---

## 1. 输入与 `input()`

在 Python 3 中，`input()` 函数用于从用户获取输入。与 Python 2 中的 `raw_input()` 不同，`input()` 会将用户输入的内容作为字符串返回。

### 示例：
```python
name = input("请输入你的名字: ")
print(f"你好, {name}!")
```

### 提示：
- 如果需要特定类型的输入，例如整数或浮点数，可以结合 `int()` 或 `float()` 进行类型转换：
  ```python
  age = int(input("请输入你的年龄: "))
  print(f"明年你将是 {age + 1} 岁。")
  ```

## 2. 输出与 `print()`

`print()` 是 Python 的核心输出函数，用于在控制台显示信息。它功能强大且灵活。

### 基本用法：
```python
print("Hello, World!")
```

### 格式化输出：
Python 3 支持多种格式化字符串的方式。

#### 使用 f-string（推荐）：
```python
name = "Alice"
age = 25
print(f"{name} 的年龄是 {age}。")
```

#### 使用 `str.format()`：
```python
print("{} 的年龄是 {}。".format("Alice", 25))
```

#### 使用占位符：
```python
print("%s 的年龄是 %d。" % ("Alice", 25))
```

### `print()` 的关键参数：
- **`sep`**: 指定多个值之间的分隔符。
  ```python
  print("Python", "is", "awesome", sep="-")
  # 输出: Python-is-awesome
  ```

- **`end`**: 指定输出的结尾字符，默认是换行符。
  ```python
  print("Hello", end=" ")
  print("World!")
  # 输出: Hello World!
  ```

- **`file`**: 指定输出目标，例如文件对象。
  ```python
  with open("output.txt", "w") as f:
      print("Hello, File!", file=f)
  ```

## 3. 文件输入与输出

Python 3 提供了强大的文件读写能力，通过内置的 `open()` 函数可以轻松实现。

### 打开文件：
`open()` 函数的基本语法如下：
```python
file = open(filename, mode, encoding=None)
```
- **`filename`**: 文件路径。
- **`mode`**: 文件操作模式，例如：
  - `"r"`: 只读（默认）。
  - `"w"`: 写入，覆盖文件内容。
  - `"a"`: 追加。
  - `"b"`: 二进制模式（如 `"rb"`）。
- **`encoding`**: 文本文件的编码方式（如 `"utf-8"`）。

### 读取文件：
```python
with open("example.txt", "r", encoding="utf-8") as file:
    content = file.read()
    print(content)
```

#### 常用读取方法：
- `read(size)`：读取指定字节数。
- `readline()`：逐行读取。
- `readlines()`：读取所有行并返回一个列表。

### 写入文件：
```python
with open("example.txt", "w", encoding="utf-8") as file:
    file.write("Hello, World!\n")
```

### 文件迭代：
```python
with open("example.txt", "r", encoding="utf-8") as file:
    for line in file:
        print(line.strip())
```

### 提示：
始终使用 `with` 语句处理文件，确保文件在使用后正确关闭。

## 4. 字符编码处理

Python 3 默认使用 `utf-8` 编码。无论是输入还是输出，都建议明确指定编码，尤其是在处理多语言文本时。

### 示例：
```python
with open("example.txt", "w", encoding="utf-8") as file:
    file.write("你好，世界！")

with open("example.txt", "r", encoding="utf-8") as file:
    content = file.read()
    print(content)
```

---

## 5. 进阶：格式化字符串与 `repr()`

当需要调试或显示对象的精确表示时，可以使用 `repr()`：
```python
value = 42
print(repr(value))  # 输出: 42
```

结合 `repr()` 和格式化字符串：
```python
print(f"The result is {value!r}")
```

---

## 6. 总结

Python 3 的输入与输出功能设计直观且灵活，涵盖了从控制台交互到文件处理的多种场景。通过熟练掌握这些工具，您可以轻松构建功能丰富、用户友好的 Python 程序。


