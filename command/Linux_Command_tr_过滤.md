#  Linux Command tr 过滤


---

![在这里插入图片描述](https://img-blog.csdnimg.cn/c2e3e5f4654249688d25423477d0de66.gif#pic_center)


## 1. 简介
tr命令可以对来自标准输入的字符进行替换、压缩和删除。它可以将一组字符变成另一组字符，经常用来编写优美的单行命令，作用很强大。

## 2. 语法

```bash
tr [选项]... SET1 [SET2]
```

## 3. 选项

```bash
-c或——complerment：取代所有不属于第一字符集的字符；
-d或——delete：删除所有属于第一字符集的字符；
-s或--squeeze-repeats：把连续重复的字符以单独一个字符表示；
-t或--truncate-set1：先删除第一字符集较第二字符集多出的字符。
```
SET 是一组字符串，一般都可按照字面含义理解。解析序列如下：

```bash
  \NNN    八进制值为NNN 的字符(1 至3 个数位)
  \\        反斜杠
  \a        终端鸣响
  \b        退格
  \f        换页
  \n        换行
  \r        回车
  \t        水平制表符
  \v        垂直制表符
  字符1-字符2    从字符1 到字符2 的升序递增过程中经历的所有字符
  [字符*]    在SET2 中适用，指定字符会被连续复制直到吻合设置1 的长度
  [字符*次数]    对字符执行指定次数的复制，若次数以 0 开头则被视为八进制数
  [:alnum:]    所有的字母和数字
  [:alpha:]    所有的字母
  [:blank:]    所有呈水平排列的空白字符
  [:cntrl:]    所有的控制字符
  [:digit:]    所有的数字
  [:graph:]    所有的可打印字符，不包括空格
  [:lower:]    所有的小写字母
  [:print:]    所有的可打印字符，包括空格
  [:punct:]    所有的标点字符
  [:space:]    所有呈水平或垂直排列的空白字符
  [:upper:]    所有的大写字母
  [:xdigit:]    所有的十六进制数
  [=字符=]    所有和指定字符相等的字符
```

## 4. 实例
将输入字符由大写转换为小写：

```bash
echo "HELLO WORLD" | tr 'A-Z' 'a-z'
hello world
```

'A-Z' 和 'a-z'都是集合，集合是可以自己制定的，例如：'ABD-}'、'bB.,'、'a-de-h'、'a-c0-9'都属于集合，集合里可以使用'\n'、'\t'，可以可以使用其他ASCII字符。

使用tr删除字符：

```bash
echo "hello 123 world 456" | tr -d '0-9'
hello  world 
```

将制表符转换为空格：

```bash
cat text | tr '\t' ' '
```

字符集补集，从输入文本中将不在补集中的所有字符删除：

```bash
echo aa.,a 1 b#$bb 2 c*/cc 3 ddd 4 | tr -d -c '0-9 \n'
 1  2  3  4
```

此例中，补集中包含了数字0~9、空格和换行符\n，所以没有被删除，其他字符全部被删除了。

用tr压缩字符，可以压缩输入中重复的字符：

```bash
echo "thissss is      a text linnnnnnne." | tr -s ' sn'
this is a text line.
```

巧妙使用tr做数字相加操作：

```c
echo 1 2 3 4 5 6 7 8 9 | xargs -n1 | echo $[ $(tr '\n' '+') 0 ]
```

删除Windows文件“造成”的'^M'字符：

```bash
cat file | tr -s "\r" "\n" > new_file
或
cat file | tr -d "\r" > new_file
```

tr可以使用的字符类：

```bash
[:alnum:]：字母和数字
[:alpha:]：字母
[:cntrl:]：控制（非打印）字符
[:digit:]：数字
[:graph:]：图形字符
[:lower:]：小写字母
[:print:]：可打印字符
[:punct:]：标点符号
[:space:]：空白字符
[:upper:]：大写字母
[:xdigit:]：十六进制字符
```

使用方式：

```bash
tr '[:lower:]' '[:upper:]'
```

更多阅读：

 - [tr command in Unix/Linux with examples](https://www.geeksforgeeks.org/tr-command-in-unix-linux-with-examples/)

