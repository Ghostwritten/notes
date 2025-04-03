#  python ljust()、center() 、rjust() 字符串填充对齐


---
## 1. python ljust() 左对齐
###  1.1 简介
Python ljust() 方法返回一个原字符串左对齐,并使用空格填充至指定长度的新字符串。如果指定的长度小于原字符串的长度则返回原字符串。

###  1.2 语法

```bash
str.ljust(width[, fillchar])
```
参数：
 - width -- 指定字符串长度。
 - fillchar -- 填充字符，默认为空格

###  1.3 实例
1.ljust_1.py

```bash
txt = "banana"

x = txt.ljust(20)

print(x, "is my favorite fruit.")
```
输出：

```bash
banana              is my favorite fruit.
```


2.ljust_2.py

```bash
#!/usr/bin/python
str = "Hi";
print(str.ljust(5, '0'))
```
输出

```bash
Hi000
```

##  2. python center() 居中

```bash
txt = "banana"

x = txt.center(10,'0')

print(x)
```
输出：

```bash
00banana00
```

##  3. python rjust() 右对齐

```bash
txt = "banana"

x = txt.rjust(10,'0')

print(x, "is my favorite fruit.")
```
输出：

```bash
0000banana is my favorite fruit.
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4c96ed3b8dc4bcb7c121476859f9ff62.gif#pic_center)


