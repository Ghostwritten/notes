#  JavaScript 23. 使用误区


![在这里插入图片描述](https://img-blog.csdnimg.cn/7cf3203498a14793a3806a905951c8b2.png)


## 1. 赋值运算符应用错误
在 JavaScript 程序中如果你在 if 条件语句中使用赋值运算符的等号 (=) 将会产生一个错误结果, 正确的方法是使用比较运算符的两个等号 (==)。

if 条件语句返回 false (是我们预期的)因为 x 不等于 10:

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p id="demo"></p>
<script>
var x = 0;
document.getElementById("demo").innerHTML = Boolean(x == 10);
</script>

</body>
</html>
```
输出：
false

if 条件语句返回 true (不是我们预期的)因为条件语句执行为 x 赋值 10，10 为 true:
```bash
var x = 0;
document.getElementById("demo").innerHTML = Boolean(x = 10);
```
输出
true

if 条件语句返回 false (不是我们预期的)因为条件语句执行为 x 赋值 0，0 为 false:
```bash
var x = 0;
document.getElementById("demo").innerHTML = Boolean(x = 0);
```
输出
false

##  2. 比较运算符常见错误
在常规的比较中，数据类型是被忽略的，以下 if 条件语句返回 true：
```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p id="demo"></p>
<script>
var x = 10;
var y = "10";
document.getElementById("demo").innerHTML = Boolean(x == y);
</script>

</body>
</html>
```
输出
true

在严格的比较运算中，=== 为恒等计算符，同时检查表达式的值与类型，以下 if 条件语句返回 false：

```bash
var x = 10;
var y = "10";
if (x === y)
```
这种错误经常会在 switch 语句中出现，switch 语句会使用恒等计算符(===)进行比较:

以下实例会执行 alert 弹窗：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p id="demo"></p>
<script>
var x = 10;
switch(x) {
    case 10: alert("Hello");
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/a898349055174ad1ad0d649087986120.png)

以下实例由于类型不一致不会执行 alert 弹窗：

```bash
var x = 10;
switch(x) {
    case "10": alert("Hello");
}
```
## 3. 加法与连接注意事项
加法是两个数字相加。

连接是两个字符串连接。

JavaScript 的加法和连接都使用 + 运算符。

接下来我们可以通过实例查看两个数字相加及数字与字符串连接的区别：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p id="demo"></p>
<script>
var x = 10 + "5";
document.getElementById("demo").innerHTML = x;
</script>

</body>
</html>
```
输出
105

使用变量相加结果也不一致:
```bash
var x = 10;
var y = 5;
var z = x + y;           // z 的结果为 15

var x = 10;
var y = "5";
var z = x + y;           // z 的结果为 "105"
```
## 4. 浮点型数据使用注意事项
JavaScript 中的所有数据都是以 64 位浮点型数据(float) 来存储。

所有的编程语言，包括 JavaScript，对浮点型数据的精确度都很难确定：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p id="demo"></p>
<script>
var x = 0.1;
var y = 0.2;
var z = x + y;
document.getElementById("demo").innerHTML = z;
</script>

</body>
</html>
```
输出：

```bash
0.30000000000000004
```
为解决以上问题，可以用整数的乘除法来解决：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p id="demo"></p>
<script>
var x = 0.1;
var y = 0.2;
var z = (x * 10 + y *10) / 10;
document.getElementById("demo").innerHTML = z;
</script>

</body>
</html>
```
输出：
0.3

## 5. JavaScript 字符串分行
JavaScript 允许我们在字符串中使用断行语句:

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p id="demo"></p>
<script>
document.getElementById("demo").innerHTML =
	"Hello World!";
</script>

</body>
</html>
```
输出：

```bash
Hello World!
```
但是，在字符串中直接使用回车换行是会报错的：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p id="demo"></p>
<script>
document.getElementById("demo").innerHTML = "Hello 
World!";
</script>

</body>
</html>
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/00604b6a72ef42cf90a75e5794cba962.png)
字符串断行需要使用**反斜杠**(\)，如下所示:

```bash
document.getElementById("demo").innerHTML = "Hello \
World!";
```
## 6. 错误的使用分号
以下实例中，if 语句失去方法体，原 if 语句的方法体作为独立的代码块被执行，导致错误的输出结果。

由于分号使用错误，if 语句中的代码块就一定会执行：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p id="demo"></p>
<script>
var x = 5;
if (x == 19);{
	document.getElementById("demo").innerHTML = "Hello";
}
</script>

</body>
</html>
```
输出：
Hello

## 7. return 语句使用注意事项
JavaScript 默认是在一行的末尾自动结束。

以下两个实例返回结果是一样的(一个有分号一个没有):

实例 1 不带分号

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p id="demo"></p>
<script>
document.getElementById("demo").innerHTML = myFunction(55);
function myFunction(a) {
    var power = 10
    return a * power
}
</script>

</body>
</html>
```
输出

```bash
550
```

实例 2 带分号

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p id="demo"></p>
<script>
document.getElementById("demo").innerHTML = myFunction(55);
function myFunction(a) {
    var power = 10;
    return a * power;
}
</script>

</body>
</html>
```
输出

```bash
550
```

JavaScript 也可以使用多行来表示一个语句，也就是说一个语句是可以分行的。

以下实例返回相同的结果:

```bash
function myFunction(a) {
    var
    power = 10; 
    return a * power;
}
```
但是，以下实例结果会返回 `undefined`：

实例 4

```bash
function myFunction(a) {
    var
    power = 10; 
    return
    a * power;
}
```
但是由于这样的语句是完整的:

```bash
return
```

JavaScript 将自动关闭语句:

```bash
return;
```

在 JavaScript 中，分号是可选的 。

由于 return 是一个完整的语句，所以 JavaScript 将关闭 return 语句。

## 8. 数组中使用名字来索引
许多程序语言都允许使用名字来作为数组的索引。

使用名字来作为索引的数组称为关联数组(或哈希)。

JavaScript 不支持使用名字来索引数组，只允许使用数字索引。

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p id="demo"></p>
<script>
var person = [];
person[0] = "John";
person[1] = "Doe";
person[2] = 46; 
document.getElementById("demo").innerHTML =
	person[0] + " " + person.length;
</script>

</body>
</html>
```
输出：

```bash
John 3
```
在 JavaScript 中, 对象 使用 名字作为索引。

如果你使用名字作为索引，当访问数组时，JavaScript 会把数组重新定义为标准对象。

执行这样操作后，数组的方法及属性将不能再使用，否则会产生错误:

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p>
如果你使用名字作为索引，当访问数组时，JavaScript 会把数组重新定义为标准对象，数组的方法及属性将不能再使用，否则会产生错误:。
</p>
<p id="demo"></p>
<script>
var person = [];
person["firstName"] = "John";
person["lastName"] = "Doe";
person["age"] = 46; 
document.getElementById("demo").innerHTML =
	person[0] + " " + person.length;
</script>

</body>
</html>
```
输出：

```bash
如果你使用名字作为索引，当访问数组时，JavaScript 会把数组重新定义为标准对象，数组的方法及属性将不能再使用，否则会产生错误:。

undefined 0
```
## 9. 定义数组元素，最后不能添加逗号
数组最后一个值的后面添加逗号虽然语法没有问题，但是在不同的浏览器可能得到不同的结果。

```bash
var colors = [5, 6, 7,]; //这样数组的长度可能为3 也可能为4。
```

正确的定义方式：

```bash
points = [40, 100, 1, 5, 25, 10];
```
## 10. 定义对象，最后不能添加逗号
错误的定义方式：

```bash
websites = {site:"菜鸟教程", url:"www.runoob.com", like:460,}
```

正确的定义方式：

```bash
websites = {site:"菜鸟教程", url:"www.runoob.com", like:460}
```
## 11. Undefined 不是 Null
在 JavaScript 中, null 用于对象, undefined 用于变量，属性和方法。

对象只有被定义才有可能为 null，否则为 undefined。

如果我们想测试对象是否存在，在对象还没定义时将会抛出一个错误。

错误的使用方式：

```bash
if (myObj !== null && typeof myObj !== "undefined") 
```

正确的方式是我们需要先使用 typeof 来检测对象是否已定义：

```bash
if (typeof myObj !== "undefined" && myObj !== null) 
```
## 12. 程序块作用域
在每个代码块中 JavaScript 不会创建一个新的作用域，一般各个代码块的作用域都是全局的。

以下代码的的变量 i 返回 10，而不是 undefined：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p id="demo"></p>
<script>
for (var i = 0; i < 10; i++) {
    // some code
}
document.getElementById("demo").innerHTML = i;
</script>

</body>
</html>
```
输出：
10
