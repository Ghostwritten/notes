#  python 模块 re 正则表达式



## 1. 原理
[Python re 正则表达式](https://docs.python.org/3/library/re.html)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/27062117e520422e2aacd9859ff30913.png)
正则表达式的大致匹配过程是：依次拿出表达式和文本中的字符比较，如果每一个字符都能匹配，则匹配成功；一旦有匹配不成功的字符则匹配失败。如果表达式中有量词或边界，这个过程会稍微有一些不同，但也是很好理解的，看下图中的示例以及自己多使用几次就能明白。

##  2. 语法
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/14fee623f9ace32fc6516373b99fb7db.png)
### 2.1 数量词的贪婪模式与非贪婪模式
正则表达式通常用于在文本中查找匹配的字符串。Python里数量词默认是贪婪的（在少数语言里也可能是默认非贪婪），总是尝试匹配尽可能多的字符；非贪婪的则相反，总是尝试匹配尽可能少的字符。例如：正则表达式"ab*"如果用于查找"abbbc"，将找到"abbb"。而如果使用非贪婪的数量词"ab*?"，将找到"a"。
### 2.2 反斜杠的困扰
与大多数编程语言相同，正则表达式里使用"\"作为转义字符，这就可能造成反斜杠困扰。假如你需要匹配文本中的字符"\"，那么使用编程语言表示的正则表达式里将需要4个反斜杠"\\\\"：前两个和后两个分别用于在编程语言里转义成反斜杠，转换成两个反斜杠后再在正则表达式里转义成一个反斜杠。Python里的原生字符串很好地解决了这个问题，这个例子中的正则表达式可以使用r"\\"表示。同样，匹配一个数字的"\\d"可以写成r"\d"。有了原生字符串，你再也不用担心是不是漏写了反斜杠，写出来的表达式也更直观。
### 2.3 匹配模式
正则表达式提供了一些可用的匹配模式，比如忽略大小写、多行匹配等，这部分内容将在Pattern类的工厂方法re.compile(pattern[, flags])中一起介绍。

---

 - [replace与正则表达式](https://blog.csdn.net/m0_37360684/article/details/84102044)


##  3. 模式

###  3.1  `I`    IGNORECASE
忽略大小写的匹配模式, 样例如下：

```bash
s = 'hello World!'

regex = re.compile("hello world!", re.I)
print regex.match(s).group()
#output> 'hello World!'

#在正则表达式中指定模式以及注释
regex = re.compile("(?#注释)(?i)hello world!")
print regex.match(s).group()
#output> 'hello World!'
```

### 3.2  `L`    LOCALE
   字符集本地化。这个功能是为了支持多语言版本的字符集使用环境的，比如在转义符\w，在英文环境下，它代表[a-zA-Z0-9_]，即所以英文字符和数字。如果在一个法语环境下使用，缺省设置下，不能匹配"é"或   "ç"。加上这L选项和就可以匹配了。不过这个对于中文环境似乎没有什么用，它仍然不能匹配中文字符。
###  3.3 `M`    MULTILINE
多行模式, 改变 `^` 和 `$` 的行为

```bash
s = '''first line
second line
third line'''

# ^
regex_start = re.compile("^\w+")
print regex_start.findall(s)
# output> ['first']

regex_start_m = re.compile("^\w+", re.M)
print regex_start_m.findall(s)
# output> ['first', 'second', 'third']

#$
regex_end = re.compile("\w+$")
print regex_end.findall(s)
# output> ['line']

regex_end_m = re.compile("\w+$", re.M)
print regex_end_m.findall(s)
# output> ['line', 'line', 'line']
```

### 3.4 `S`  DOTALL
此模式下 '.' 的匹配不受限制，可匹配任何字符，包括换行符

```bash
s = '''first line
second line
third line'''
#
regex = re.compile(".+")
print regex.findall(s)
# output> ['first line', 'second line', 'third line']

# re.S
regex_dotall = re.compile(".+", re.S)
print regex_dotall.findall(s)
# output> ['first line\nsecond line\nthird line']
```

### 3.5 `X`    VERBOSE
 冗余模式， 此模式忽略正则表达式中的空白和#号的注释，例如写一个匹配邮箱的正则表达式

```bash
email_regex = re.compile("[\w+\.]+@[a-zA-Z\d]+\.(com|cn)")

email_regex = re.compile("""[\w+\.]+  # 匹配@符前的部分
                            @  # @符
                            [a-zA-Z\d]+  # 邮箱类别
                            \.(com|cn)   # 邮箱后缀  """, re.X)
```
### 3.6 `U`    UNICODE
使用 \w, \W, \b, \B 这些元字符时将按照 UNICODE 定义的属性.
正则表达式的模式是可以同时使用多个的，在 python 里面使用按位或运算符 | 同时添加多个模式，如 `re.compile('', re.I|re.M|re.S)`，每个模式在 re 模块中其实就是不同的数字

```bash
print re.I
# output> 2
print re.L
# output> 4
print re.M
# output> 8
print re.S
# output> 16
print re.X
# output> 64
print re.U
# output> 32
```


##  4. 函数 （参见 python 模块 re 文档）

### 4.1 compile(pattern, flags=0)

给定一个正则表达式 `pattern`，指定使用的模式 `flags` 默认为0 即不使用任何模式,然后会返回一个 `SRE_Pattern` (参见 第四小节 re 内置对象用法) 对象

```bash
regex = re.compile(".+")
print regex
# output> <_sre.SRE_Pattern object at 0x00000000026BB0B8>
```
这个对象可以调用其他函数来完成匹配，一般来说推荐使用 `compile` 函数预编译出一个正则模式之后再去使用，这样在后面的代码中可以很方便的复用它，当然大部分函数也可以不用 compile 直接使用，具体见 `findall` 函数

```bash
s = '''first line
second line
third line'''
#
regex = re.compile(".+")
# 调用 findall 函数
print regex.findall(s)
# output> ['first line', 'second line', 'third line']
# 调用 search 函数
print regex.search(s).group()
# output> first lin
```

###  4.2 escape(pattern)   
转义 如果你需要操作的文本中含有正则的元字符，你在写正则的时候需要将元字符加上反斜扛 `\` 去匹配自身， 而当这样的字符很多时，写出来的正则表达式就看起来很乱而且写起来也挺麻烦的，这个时候你可以使用这个函数,用法如下

```bash
s = ".+\d123"
#
regex_str = re.escape(".+\d123")
# 查看转义后的字符
print regex_str
# output> \.\+\\d123

# 查看匹配到的结果
for g in re.findall(regex_str, s):
    print g
# output> .+\d123
```

###  4.3 findall(pattern, string, flags=0) 
参数 `pattern` 为正则表达式, `string` 为待操作字符串, `flags` 为所用模式，函数作用为在待操作字符串中寻找所有匹配正则表达式的字串，返回一个列表，如果没有匹配到任何子串，返回一个空列表。

```bash
s = '''first line
second line
third line'''

# compile 预编译后使用 findall
regex = re.compile("\w+")
print regex.findall(s)
# output> ['first', 'line', 'second', 'line', 'third', 'line']

# 不使用 compile 直接使用 findall
print re.findall("\w+", s)
# output> ['first', 'line', 'second', 'line', 'third', 'line']
```

###  4.4 finditer(pattern, string, flags=0) 
参数和作用与 findall 一样，不同之处在于 `findall` 返回一个列表， [finditer 返回一个迭代器](http://www.cnblogs.com/huxi/archive/2011/07/01/2095931.html)， 而且迭代器每次返回的值并不是字符串，而是一个 `SRE_Match` 对象，这个对象的具体用法见 `match` 函数。

```bash
s = '''first line
second line
third line'''

regex = re.compile("\w+")
print regex.finditer(s)
# output> <callable-iterator object at 0x0000000001DF3B38>
for i in regex.finditer(s):
    print i
# output> <_sre.SRE_Match object at 0x0000000002B7A920>
#         <_sre.SRE_Match object at 0x0000000002B7A8B8>
#         <_sre.SRE_Match object at 0x0000000002B7A920>
#         <_sre.SRE_Match object at 0x0000000002B7A8B8>
#         <_sre.SRE_Match object at 0x0000000002B7A920>
#         <_sre.SRE_Match object at 0x0000000002B7A8B8>
```

###  4.5 match(pattern, string, flags=0) 
使用指定正则去待操作字符串中寻找可以匹配的子串, 返回匹配上的第一个字串，并且不再继续找，需要注意的是 `match` 函数是从字符串开始处开始查找的，如果开始处不匹配，则不再继续寻找，返回值为 一个 `SRE_Match` (参见 第四小节 re 内置对象用法) 对象，找不到时返回 None

```bash
s = '''first line
second line
third line'''

# compile
regex = re.compile("\w+")
m = regex.match(s)
print m
# output> <_sre.SRE_Match object at 0x0000000002BCA8B8>
print m.group()
# output> first

# s 的开头是 "f", 但正则中限制了开始为 i 所以找不到
regex = re.compile("^i\w+")
print regex.match(s)
# output> None
```
###  4.6 purge()   
当你在程序中使用 re 模块，无论是先使用 compile 还是直接使用比如 findall 来使用正则表达式操作文本，re 模块都会将正则表达式先编译一下， 并且会将编译过后的正则表达式放到缓存中，这样下次使用同样的正则表达式的时候就不需要再次编译， 因为编译其实是很费时的，这样可以提升效率，而默认缓存的正则表达式的个数是 100, 当你需要频繁使用少量正则表达式的时候，缓存可以提升效率，而使用的正则表达式过多时，缓存带来的优势就不明显了 (参考 《python re.compile对性能的影响》http://blog.trytofix.com/article/detail/13/)， 这个函数的作用是清除缓存中的正则表达式，可能在你需要优化占用内存的时候会用到。

###  4.7 search(pattern, string, flags=0) 
 函数类似于 `match`，不同之处在于不限制正则表达式的开始匹配位置

```bash
 s = '''first line
second line
third line'''

# 需要从开始处匹配 所以匹配不到 
print re.match('i\w+', s)
# output> None

# 没有限制起始匹配位置
print re.search('i\w+', s)
# output> <_sre.SRE_Match object at 0x0000000002C6A920>

print re.search('i\w+', s).group()
# output> irst
```

###  4.8 split(pattern, string, maxsplit=0, flags=0)
参数 maxsplit 指定切分次数， 函数使用给定正则表达式寻找切分字符串位置，返回包含切分后子串的列表，如果匹配不到，则返回包含原字符串的一个列表

```bash
s = '''first 111 line
second 222 line
third 333 line'''

# 按照数字切分
print re.split('\d+', s)
# output> ['first ', ' line\nsecond ', ' line\nthird ', ' line']

# \.+ 匹配不到 返回包含自身的列表
print re.split('\.+', s, 1)
# output> ['first 111 line\nsecond 222 line\nthird 333 line']

# maxsplit 参数
print re.split('\d+', s, 1)
# output> ['first ', ' line\nsecond 222 line\nthird 333 line']
```

##   5. demo
###  5.1 正则表达式包含变量

 - re.compile(r’表达式’+变量+’表达式’)
 - re.compile(r’表达式(%s)表达式’ %变量)

```python
#!/usr/bin/python

import re

url = "medium.com"
regex3 = re.compile(r"^((/|.)*(%s))" %url)
regex4 = re.compile(r"^((/|.)*medium.com)")
regex5 = re.compile(r"^((/|.)*"+ url +')')

string3 = '/medium.com/darpa.mil/'

mo3 = regex3.search(string3)
mo4 = regex4.search(string3)
mo5 = regex5.search(string3)
print(mo3.group())
print(mo4.group())
print(mo5.group())
```
输出：

```python
/medium.com
/medium.com
/medium.com
```
----
✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>

 - [python正则表达式详解](https://www.cnblogs.com/dyfblog/p/5880728.html)
 - [runoob Python 正则表达式](https://www.runoob.com/python/python-reg-expressions.html)
 - [Python Regex: re.search() VS re.findall()](https://www.geeksforgeeks.org/python-regex-re-search-vs-re-findall/)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/73b98450d13cdda7b7e4df9355d6d1d4.gif#pic_center)

