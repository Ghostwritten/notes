


---

##  1. replace()
我们可以使用 `str` 的 `replace()` 方法将子字符串替换成不同的输出。

`replace()` 接受两个参数，第一个参数是你要匹配字符串的 `regex` 模式，第二个参数是匹配字符串的替换字符串。

它在 replace() 中还有第三个可选参数，它接受一个整数，用来设置要执行的最大 count 替换次数。如果你把 2 作为 count 参数，`replace()` 函数将只匹配和替换字符串中的 2 个实例。

`str.replace('Hello', 'Hi')` 将用 Hi 替换字符串中所有 Hello 的实例。如果你有一个字符串 Hello World，并对其运行 replace 函数，执行后会变成 Hi World。

```python
txt = "A!!!,Quick,brown#$,fox,ju%m%^ped,ov&er&),th(e*,lazy,d#!og$$$"

def processString(txt):
  specialChars = "!#$%^&*()" 
  for specialChar in specialChars:
    txt = txt.replace(specialChar, '')
  print(txt) # A,Quick,brown,fox,jumped,over,the,lazy,dog
  txt = txt.replace(',', ' ')
  print(txt) # A Quick brown fox jumped over the lazy dog  
```
这意味着在 spChars 方括号内的任何字符都将被一个空字符串替换，使用 `txt.replace(spChars, '')`。

第一个 replace() 函数的字符串结果将是：

```perl
A,Quick,brown,fox,jumped,over,the,lazy,dog
```
下一个 replace() 调用将把所有逗号 , 的实例替换成单个空格。

```perl
A Quick brown fox jumped over the lazy dog
```


##  2. sub() 
该函数有 3 个主要参数。第一个参数接受一个 regex 模式，第二个参数是一个字符串，用来替换匹配的模式，第三个参数是要操作的字符串。

创建一个函数，将一个字符串中的所有数字转换为 X。

```python
import re

txt = "Hi, my phone number is 089992654231. I am 34 years old. I live in 221B Baker Street. I have 1,000,000 in my bank account."

def processString3(txt):
  txt = re.sub('[0-9]', 'X', txt)
  print(txt)
  
processString3(txt)
```
输出：

```python
Hi, my phone number is XXXXXXXXXXXX. I am XX years old. I live in XXXB Baker Street. I have X,XXX,XXX in my bank account.
```
##  3. re.subn()
这个函数本质上与 re.sub() 相同，但是返回一个转换后的字符串的元组和替换的数量。

```python
import re

txt = "Hi, my phone number is 089992654231. I am 34 years old. I live in 221B Baker Street. I have 1,000,000 in my bank account."

def processString4(txt):
  txt, n = re.subn('[0-9]', 'X', txt)
  print(txt)
  
processString4(txt)
```
输出：

```powershell
Hi, my phone number is XXXXXXXXXXXX. I am XX years old. I live in XXXB Baker Street. I have X,XXX,XXX in my bank account.'
```

```php
txt, n = re.subn('[0-9]', 'X', txt)
```
在上面的代码片段中，处理后的字符串被分配给 txt，替换计数器被分配给 n。

如果你想记下有多少个模式组作为指标或用于进一步处理，则 `re.subn()` 很有用。

##  4. translate() 和 maketrans()
`translate()` 和 `maketrans()` 使用一种不同于 regex 的方法，它利用字典将旧值映射到新值。

`maketrans()` 接受 3 个参数或一个字典的映射。

 - str1 - 要替换的字符串。
 - str2 - 以上字符的替换字符串。
 - str3 - 要删除的字符串。

`maketrans()` 一个原始字符串和它的替换之间的映射表。

`translate()` 接受任何 `maketrans()` 返回，然后生成翻译后的字符串。

比方说，我们想把一个字符串中的所有小写元音转换成大写和删除字符串中的所有 x、y 和 z。

```python
txt = "Hi, my name is Mary. I like zebras and xylophones."

def processString5(txt):
  transTable = txt.maketrans("aeiou", "AEIOU", "xyz")
  txt = txt.translate(transTable)
  print(txt)
  
processString5(txt)
```
输出：

```powershell
HI, m nAmE Is MAr. I lIkE EbrAs And lOphOnEs.
```
`translate()` 将所有小写元音转换为大写，并删除所有 x, y, 和 z 的实例。

使用这些方法的另一种方法是使用一个单一的映射字典而不是三个参数。

```python
txt = "Hi, my name is Mary. I like zebras and xylophones."

def processString6(txt):
  dictionary = {'a': 'A', 'e':'E', 'i': 'I', 'o': 'O', 'u': 'U', 'x': None, 'y': None, 'z': None}
  transTable = txt.maketrans(dictionary)
  txt = txt.translate(transTable)
  print(txt)

processString6(txt)
```
输出：

```powershell
HI, m nAmE Is MAr. I lIkE EbrAs And lOphOnEs.
```

这仍然会产生与 `processString5` 相同的输出，但是是用字典实现的。你可以使用任何对你更方便的方法。

总之，有多种方法可以通过使用 Python 中的内置函数或从导入库中的函数来替换字符串中的多个字符。

最常用的方法是使用 `replace()`。`re.sub()` 和 subn() 也相当容易使用和学习。translate() 使用不同的方法，因为它不依靠正则表达式来执行字符串操作，而是依靠字典和 Map。

如果你愿意，你甚至可以使用 for 循环在字符串上手动循环，并添加你自己的条件来替换，只需使用 substring() 或 split()，但这将是非常低效和多余的。Python 提供了现有的函数来为你完成这项工作，这比你自己完成繁琐的工作要容易得多。

---

✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>


 - [python 内置函数](https://blog.csdn.net/xixihahalelehehe/article/details/104913051)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/24b31ac173f842481ba1ac0b918bdda6.gif#pic_center)

