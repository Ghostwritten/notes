

difflib模块提供用于比较序列的类和函数。 例如，它可以用于比较文件，并可以产生各种格式的不同信息，包括 HTML 和上下文以及统一格式的差异点。
class difflib.SequenceMatcher
这是一个灵活的类，可用于比较任何类型的序列对，只要序列元素为 hashable 对象。

## class difflib.Differ
**这个类的作用是比较由文本行组成的序列，并产生可供人阅读的差异或增量信息。** Differ 统一使用 SequenceMatcher 来完成行序列的比较以及相似（接近匹配）行内部字符序列的比较。
Differ 增量的每一行均以双字母代码打头：
| Code  | 意义|
|--|--|
| '- '  | 行为序列 1 所独有|
| '+ '  | 行为序列 2 所独有 |
| ' '  | 行在两序列中相同 |
| '？'  | 行不存在于任一输入序列 |

以 '?' 打头的行尝试将视线引至行以外而不存在于任一输入序列的差异。 如果序列包含制表符则这些行可能会令人感到迷惑。

### class difflib.Differ(linejunk=None, charjunk=None)
可选关键字形参 linejunk 和 charjunk 均为过滤函数 (或为 None)：

 - linejunk: 接受单个字符串作为参数的函数，如果其为垃圾字符串则返回真值。 默认值为 None，意味着没有任何行会被视为垃圾行。
 - charjunk: 接受单个字符（长度为 1 的字符串）作为参数的函数，如果其为垃圾字符则返回真值。 默认值为
   None，意味着没有任何字符会被视为垃圾字符。

这些垃圾过滤函数可加快查找差异的匹配速度，并且不会导致任何差异行或字符被忽略。 请阅读 find_longest_match() 方法的 isjunk 形参的描述了解详情。
Differ 对象是通过一个单独方法来使用（生成增量）的：

### compare(a, b)
**比较两个由行组成的序列，并生成增量（一个由行组成的序列）。**
每个序列必须包含一个以换行符结尾的单行字符串。 这样的序列可以通过文件类对象的 readlines() 方法来获取。 所生成的增量同样由以换行符结尾的字符串构成，可以通过文件类对象的 writelines() 方法原样打印出来。
Differ 示例
此示例比较两段文本。 首先我们设置文本为以换行符结尾的单行字符串构成的序列（这样的序列也可以通过文件类对象的 readlines() 方法来获取）：

```python
>>> text1 = '''  1. Beautiful is better than ugly.
  2. Explicit is better than implicit.
  3. Simple is better than complex.
  4. Complex is better than complicated.
'''.splitlines(keepends=True)
>>> len(text1)
4
>>> text1[0][-1]
'\n'
>>> text2 = '''  1. Beautiful is better than ugly.
  3.   Simple is better than complex.
  4. Complicated is better than complex.
  5. Flat is better than nested.
'''.splitlines(keepends=True)
```
接下来我们实例化一个 Differ 对象：

```python
>>> d = Differ()
```
请注意在实例化 Differ 对象时我们可以传入函数来过滤掉“垃圾”行和字符。 详情参见 Differ() 构造器说明。
最后，我们比较两个序列：

```python
>>> result = list(d.compare(text1, text2))
```
result 是一个字符串列表，让我们将其美化打印出来：

```python
>>> from pprint import pprint
>>> pprint(result)
['    1. Beautiful is better than ugly.\n',
 '-   2. Explicit is better than implicit.\n',
 '-   3. Simple is better than complex.\n',
 '+   3.   Simple is better than complex.\n',
 '?     ++\n',
 '-   4. Complex is better than complicated.\n',
 '?            ^                     ---- ^\n',
 '+   4. Complicated is better than complex.\n',
 '?           ++++ ^                      ^\n',
 '+   5. Flat is better than nested.\n']
```

作为单独的多行字符串显示出来则是这样：

```python
>>> import sys
>>> sys.stdout.writelines(result)
    1. Beautiful is better than ugly.
-   2. Explicit is better than implicit.
-   3. Simple is better than complex.
+   3.   Simple is better than complex.
?     ++
-   4. Complex is better than complicated.
?            ^                     ---- ^
+   4. Complicated is better than complex.
?           ++++ ^                      ^
+   5. Flat is better than nested.
```

## class difflib.HtmlDiff

**这个类可用于创建 HTML 表格（或包含表格的完整 HTML 文件）以并排地逐行显示文本比较，行间与行外的更改将突出显示。** 此表格可以基于完全或上下文差异模式来生成。
这个类的构造函数：
__init__(tabsize=8, wrapcolumn=None, linejunk=None, charjunk=IS_CHARACTER_JUNK)
初始化 HtmlDiff 的实例。

 - tabsize 是一个可选关键字参数，指定制表位的间隔，默认值为 8。
 - wrapcolumn 是一个可选关键字参数，指定行文本自动打断并换行的列位置，默认值为 None 表示不自动换行。
 - linejunk 和 charjunk 均是可选关键字参数，会传入 ndiff() (被 HtmlDiff 用来生成并排显示的 HTML
   差异)。 




```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import re
import os
import difflib
tex1="""tex1:
this is a test for difflib ,just try to get difference of the log
goodtest
"""
tex1_lines=tex1.splitlines()
tex2="""tex2:
this is a test for difflib ,just try to get difference of the log
goodtast
"""
tex2_lines=tex2.splitlines()
d=difflib.HtmlDiff()
q=d.make_file(tex1_lines,tex2_lines)
old_str='charset=ISO-8859-1'
new_str='charset=UTF-8'
with open('diff.html','w') as f_new:
	f_new.write(q.replace(old_str,new_str))
```

```powershell
python testhtmldiff.py
```


### make_file(fromlines, tolines, fromdesc='', todesc='', context=False, numlines=5, *, charset='utf-8')

**比较 fromlines 和 tolines (字符串列表) 并返回一个字符串，表示一个完整 HTML 文件，其中包含各行差异的表格，行间与行外的更改将突出显示。**

 - fromdesc 和 todesc 均是可选关键字参数，指定来源/目标文件的列标题字符串（默认均为空白字符串）。
 - context 和 numlines 均是可选关键字参数。 当只要显示上下文差异时就将 context 设为 True，否则默认值False 为显示完整文件。 numlines 默认为 5。 当 context 为 True 时 numlines将控制围绕突出显示差异部分的上下文行数。 当 context 为 False 时 numlines 将控制在使用 "next"，超链接时突出显示差异部分之前所显示的行数（设为零则会导致 "next"超链接将下一个突出显示差异部分放在浏览器顶端，不添加任何前导上下文）。

> 在 3.5 版更改: 增加了 charset 关键字参数。 HTML 文档的默认字符集从 'ISO-8859-1' 更改为 'utf-8'。

```python
import difflib
a = ['1', '2', "A", "abcde"]
b = ['2', '3', "A", "Abcde"]
diff = difflib.HtmlDiff()    # 创建HtmlDiff 对象
result = diff.make_file(a, b)  # 通过make_file 方法输出 html 格式的对比结果
print(result)  # 需要将result写入html中。
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200210174943697.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
### make_table(fromlines, tolines, fromdesc='', todesc='', context=False, numlines=5)

**比较 fromlines 和 tolines (字符串列表) 并返回一个字符串，表示一个包含各行差异的完整 HTML 表格，行间与行外的更改将突出显示。**
此方法的参数与 make_file() 方法的相同。

### difflib.context_diff(a, b, fromfile='', tofile='', fromfiledate='', tofiledate='', n=3, lineterm='\n')

**比较 a 和 b (字符串列表)；返回上下文差异格式的增量信息 (一个产生增量行的 generator)。**
所谓上下文差异是一种只显示有更改的行再加几个上下文行的紧凑形式。 更改被显示为之前/之后的样式。 上下文行数由 n 设定，默认为三行。
默认情况下，差异控制行（以 *** or --- 表示）是通过末尾换行符来创建的。 这样做的好处是从 io.IOBase.readlines() 创建的输入将得到适用于 io.IOBase.writelines() 的差异信息，因为输入和输出都带有末尾换行符。对于没有末尾换行符的输入，应将 lineterm 参数设为 ""，这样输出内容将统一不带换行符。
上下文差异格式通常带有一个记录文件名和修改时间的标头。 这些信息的部分或全部可以使用字符串 fromfile, tofile, fromfiledate 和 tofiledate 来指定。 修改时间通常以 ISO 8601 格式表示。 如果未指定，这些字符串默认为空。

```powershell
>>> s1 = ['bacon\n', 'eggs\n', 'ham\n', 'guido\n']
>>> s2 = ['python\n', 'eggy\n', 'hamster\n', 'guido\n']
>>> sys.stdout.writelines(context_diff(s1, s2, fromfile='before.py', tofile='after.py'))
*** before.py
--- after.py
***************
*** 1,4 ****
! bacon
! eggs
! ham
  guido
--- 1,4 ----
! python
! eggy
! hamster
  guido
```

### difflib.get_close_matches(word, possibilities, n=3, cutoff=0.6)

**返回由最佳“近似”匹配构成的列表。** 
word 为一个指定目标近似匹配的序列（通常为字符串），possibilities 为一个由用于匹配 word 的序列构成的列表（通常为字符串列表）。
可选参数 n (默认为 3) 指定最多返回多少个近似匹配； n 必须大于 0.
可选参数 cutoff (默认为 0.6) 是一个 [0, 1] 范围内的浮点数。 与 word 相似度得分未达到该值的候选匹配将被忽略。
候选匹配中（不超过 n 个）的最佳匹配将以列表形式返回，按相似度得分排序，最相似的排在最前面。

```python
>>> get_close_matches('appel', ['ape', 'apple', 'peach', 'puppy'])
['apple', 'ape']
>>> import keyword
>>> get_close_matches('wheel', keyword.kwlist)
['while']
>>> get_close_matches('pineapple', keyword.kwlist)
[]
>>> get_close_matches('accept', keyword.kwlist)
['except']
```
### difflib.ndiff(a, b, linejunk=None, charjunk=IS_CHARACTER_JUNK)

**比较 a 和 b (字符串列表)；返回 Differ 形式的增量信息 (一个产生增量行的 generator)。**
可选关键字形参 linejunk 和 charjunk 均为过滤函数 (或为 None)：
linejunk: 此函数接受单个字符串参数，如果其为垃圾字符串则返回真值，否则返回假值。 默认为 None。 此外还有一个模块层级的函数 IS_LINE_JUNK()，它会过滤掉没有可见字符的行，除非该行添加了至多一个井号符 ('#') -- 但是下层的 SequenceMatcher 类会动态分析哪些行的重复频繁到足以形成噪音，这通常会比使用此函数的效果更好。
charjunk: 此函数接受一个字符（长度为 1 的字符串)，如果其为垃圾字符则返回真值，否则返回假值。 默认为模块层级的函数 IS_CHARACTER_JUNK()，它会过滤掉空白字符（空格符或制表符；但包含换行符可不是个好主意！）。

```python
>>> diff = ndiff('one\ntwo\nthree\n'.splitlines(keepends=True),
             'ore\ntree\nemu\n'.splitlines(keepends=True))
>>> print(''.join(diff), end="")
- one
?  ^
+ ore
?  ^
- two
- three
?  -
+ tree
+ emu
```

### difflib.restore(sequence, which)
**返回两个序列中产生增量的那一个。**
给出一个由 Differ.compare() 或 ndiff() 产生的 序列，提取出来自文件 1 或 2 (which 形参) 的行，去除行前缀。
示例:

```python
>>> diff = ndiff('one\ntwo\nthree\n'.splitlines(keepends=True),
             'ore\ntree\nemu\n'.splitlines(keepends=True))
>>> diff = list(diff) # materialize the generated delta into a list
>>> print(''.join(restore(diff, 1)), end="")
one
two
three
>>> print(''.join(restore(diff, 2)), end="")
ore
tree
emu
```

### difflib.unified_diff(a, b, fromfile='', tofile='', fromfiledate='', tofiledate='', n=3, lineterm='\n')
**比较 a 和 b (字符串列表)；返回统一差异格式的增量信息 (一个产生增量行的 generator)。**
所以统一差异是一种只显示有更改的行再加几个上下文行的紧凑形式。 更改被显示为内联的样式（而不是分开的之前/之后文本块）。 上下文行数由 n 设定，默认为三行。
**默认情况下，差异控制行 (以 ---, +++ 或 @@ 表示) 是通过末尾换行符来创建的。** 这样做的好处是从 io.IOBase.readlines() 创建的输入将得到适用于 io.IOBase.writelines() 的差异信息，因为输入和输出都带有末尾换行符。
对于没有末尾换行符的输入，应将 lineterm 参数设为 ""，这样输出内容将统一不带换行符。
上下文差异格式通常带有一个记录文件名和修改时间的标头。 这些信息的部分或全部可以使用字符串 fromfile, tofile, fromfiledate 和 tofiledate 来指定。 修改时间通常以 ISO 8601 格式表示。 如果未指定，这些字符串默认为空。

```python
>>> s1 = ['bacon\n', 'eggs\n', 'ham\n', 'guido\n']
>>> s2 = ['python\n', 'eggy\n', 'hamster\n', 'guido\n']
>>> sys.stdout.writelines(unified_diff(s1, s2, fromfile='before.py', tofile='after.py'))
--- before.py
+++ after.py
@@ -1,4 +1,4 @@
-bacon
-eggs
-ham
+python
+eggy
+hamster
 guido
```

### difflib.diff_bytes(dfunc, a, b, fromfile=b'', tofile=b'', fromfiledate=b'', tofiledate=b'', n=3, lineterm=b'\n')
**使用 dfunc 比较 a 和 b (字节串对象列表)；产生以 dfunc 所返回格式表示的差异行列表（也是字节串）。 dfunc 必须是可调用对象，通常为 unified_diff() 或 context_diff()。**

允许你比较编码未知或不一致的数据。 除 n 之外的所有输入都必须为字节串对象而非字符串。 作用方式为无损地将所有输入 (除 n 之外) 转换为字符串，并调用 dfunc(a, b, fromfile, tofile, fromfiledate, tofiledate, n, lineterm)。 dfunc 的输出会被随即转换回字节串，这样你所得到的增量行将具有与 a 和 b 相同的未知/不一致编码。

> 3.5 新版功能.

### difflib.IS_LINE_JUNK(line)
**对于可忽略的行返回 True。 如果 line 为空行或只包含单个 '#' 则 line 行就是可忽略的，否则就是不可忽略的。** 此函数被用作较旧版本 ndiff() 中 linejunk 形参的默认值。

### difflib.IS_CHARACTER_JUNK(ch)
**对于可忽略的字符返回 True。 字符 ch 如果为空格符或制表符则 ch 就是可忽略的，否则就是不可忽略的。** 此函数被用作 ndiff() 中 charjunk 形参的默认值。
