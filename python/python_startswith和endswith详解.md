

---
## 1. startswith
Python startswith() 方法用于检查字符串是否是以指定子字符串开头，如果是则返回 True，否则返回 False。如果参数 beg 和 end 指定值，则在指定范围内检查。

### 1.1 语法
startswith()方法语法：

```bash
str.startswith(str, beg=0,end=len(string));
```

### 1.2 参数

 - str -- 检测的字符串。
 - strbeg -- 可选参数用于设置字符串检测的起始位置。
 - strend -- 可选参数用于设置字符串检测的结束位置。

返回值
如果检测到字符串则返回True，否则返回False。

### 1.3 实例
以下实例展示了startswith()函数的使用方法：

```bash
#!/usr/bin/python

str = "this is string example....wow!!!";
print str.startswith( 'this' );
print str.startswith( 'is', 2, 4 );
print str.startswith( 'this', 2, 4 );
以上实例输出结果如下：

True
True
False
```
## 2. endswith
Python endswith() 方法用于判断字符串是否以指定后缀结尾，如果以指定后缀结尾返回True，否则返回False。可选参数"start"与"end"为检索字符串的开始与结束位置。

### 2.1 语法
endswith()方法语法：

```bash
str.endswith(suffix[, start[, end]])
```

### 2.2 参数

 - suffix -- 该参数可以是一个字符串或者是一个元素。
 - start -- 字符串中的开始位置。
 - end -- 字符中结束位置。

返回值
如果字符串含有指定的后缀返回True，否则返回False。

### 2.3 实例
以下实例展示了endswith()方法的实例：

实例(Python 2.0+)

```bash
#!/usr/bin/python
 
str = "this is string example....wow!!!";
 
suffix = "wow!!!";
print str.endswith(suffix);
print str.endswith(suffix,20);
 
suffix = "is";
print str.endswith(suffix, 2, 4);
print str.endswith(suffix, 2, 6);
以上实例输出结果如下：

True
True
True
False
```

