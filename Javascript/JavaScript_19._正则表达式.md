#  JavaScript 19. 正则表达式


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ad7a139fbf6d876a593e1737f0e8c054.png)


##  1. 什么是正则表达式
正则表达式是由一个字符序列形成的搜索模式。

当你在文本中搜索数据时，你可以用搜索模式来描述你要查询的内容。

正则表达式可以是一个简单的字符，或一个更复杂的模式。

正则表达式可用于所有文本搜索和文本替换的操作。
语法

```bash
/正则表达式主体/修饰符(可选)
```
实例：

```bash
var patt = /runoob/i
```
实例解析：

 - /runoob/i  是一个正则表达式。
 - runoob  是一个正则表达式主体 (用于检索)。
 - i  是一个修饰符 (搜索不区分大小写)。


##  2. 使用字符串方法
在 JavaScript 中，正则表达式通常用于两个字符串方法 : search() 和 replace()。

 - `search()` 方法用于检索字符串中指定的子字符串，或检索与正则表达式相匹配的子字符串，并返回子串的起始位置。
 - `replace()` 方法用于在字符串中用一些字符串替换另一些字符串，或替换一个与正则表达式匹配的子串。

###  2.1 search() 方法
使用正则表达式搜索 `"Runoob"` 字符串，且不区分大小写：
```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p>搜索字符串 "runoob", 并显示匹配的起始位置：</p>
<button onclick="myFunction()">点我</button>
<p id="demo"></p>
<script>
function myFunction() {
    var str = "Visit Runoob!"; 
    var n = str.search(/Runoob/i);
    document.getElementById("demo").innerHTML = n;
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a7eaa26efc1808047f1155c202a3d636.png)
###  2.2 replace() 方法
replace() 方法将接收字符串作为参数：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p>替换 "Microsoft" 为 "Runoob" :</p>
<button onclick="myFunction()">点我</button>
<p id="demo">请访问 Microsoft!</p>
<script>
function myFunction() {
    var str = document.getElementById("demo").innerHTML; 
    var txt = str.replace("Microsoft","Runoob");
    document.getElementById("demo").innerHTML = txt;
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0e6966e1ac70184c5311eaf467f3a8b2.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/066d4b7d8b074c8a84e8ea2620b2b1fa.png)
##  3. 正则表达式修饰符
修饰符 可以在全局搜索中不区分大小写:
|修饰符|	描述|
|--|--|
|i	|执行对大小写不敏感的匹配。
|g	|执行全局匹配（查找所有匹配而非在找到第一个匹配后停止）。
|m|	执行多行匹配。

##  4. 正则表达式模式
方括号用于查找某个范围内的字符：
|表达式|	描述|
|--|--|
|[abc]	|查找方括号之间的任何字符。
|[0-9]	|查找任何从 0 至 9 的数字。
|(x|y)	|查找任何以 | 分隔的选项。

元字符是拥有特殊含义的字符：
|元字符|	描述|
|--|--|
|\d	|查找数字。
|\s	|查找空白字符。
|\b|匹配单词边界。
|\uxxxx	|查找以十六进制数 xxxx 规定的 Unicode 字符。

量词:
|量词	|描述|
|--|--|
|n+|	匹配任何包含至少一个 n 的字符串。|
|n*|	匹配任何包含零个或多个 n 的字符串。|
|n?|	匹配任何包含零个或一个 n 的字符串。|

## 5. 使用 test()
`test()` 方法是一个正则表达式方法。

test() 方法用于检测一个字符串是否匹配某个模式，如果字符串中含有匹配的文本，则返回 true，否则返回 false。

以下实例用于搜索字符串中的字符 "e"：

```bash
var patt = /e/;
patt.test("The best things in life are free!");
```

字符串中含有 "e"，所以该实例输出为：

```bash
true
```
实例：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<script>
var patt1=new RegExp("e");
document.write(patt1.test("The best things in life are free"));
</script>

</body>
</html>
```
输出：

```bash
true
```
你可以不用设置正则表达式的变量，以上两行代码可以合并为一行：

```bash
/e/.test("The best things in life are free!")
```

###  5.1 判断输入字符串是否为数字、字母、下划线组成

```bash
function isValid(str) { return /^\w+$/.test(str); }
str = "1234abd__"
document.write(isValid(str));
document.write("<br>");

str2 = "$32343#"
document.write(isValid(str2));
document.write("<br>");
```
输出：

```bash
使用正则表达式的方式来判断。

true
false
```
###  5.2 判断输入字符串是否全部为字母

```bash
val = "123456"
var isletter = /^[a-zA-Z]+$/.test(val);
document.write(isletter);
document.write("<br>");

val2 = "asaaa"
var isletter2 = /^[a-zA-Z]+$/.test(val2);
document.write(isletter2);
```
输出：

```bash
使用正则表达式的方式来判断。

false
true
```
### 5.3  判断输入字符串是否全部为数字

```bash
val = "123456"
var isnum = /^\d+$/.test(val);
document.write(isnum);
document.write("<br>");

val2 = "as123"
var isnum2 = /^\d+$/.test(val2);
document.write(isnum2);
```
输出：

```bash
使用正则表达式的方式来判断。

true
false
```

##  6. 使用 exec()
`exec()` 方法是一个正则表达式方法。

exec() 方法用于检索字符串中的正则表达式的匹配。

该函数返回一个数组，其中存放匹配的结果。如果未找到匹配，则返回值为 null。

以下实例用于搜索字符串中的字母 "e":

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<script>
var patt1=new RegExp("e");
document.write(patt1.exec("The best things in life are free"));
</script>

</body>
</html>
```
输出：

```bash
e
```


完整的 RegExp 对象参考手册，请参考 [JavaScript RegExp 参考手册](https://www.runoob.com/jsref/jsref-obj-regexp.html)。
