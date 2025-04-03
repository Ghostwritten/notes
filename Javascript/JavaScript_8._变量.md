#  JavaScript 8. 变量


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8bc8a39ad980a2c8f2853a4f95c4d7ff.png)



##  1. 简介

变量是用于存储信息的"容器"。

```bash
var x=5;
var y=6;
var z=x+y;
```
代数一样，JavaScript 变量可用于存放值（比如 x=5）和表达式（比如 z=x+y）。

变量可以使用短名称（比如 x 和 y），也可以使用描述性更好的名称（比如 age, sum, totalvolume）。

 - 变量必须以字母开头
 - 变量也能以 $ 和 _ 符号开头（不过我们不推荐这么做）
 - 变量名称对大小写敏感（y 和 Y 是不同的变量）


##  2. 类型

```bash
var pi=3.14;  
// 如果你熟悉 ES6，pi 可以使用 const 关键字，表示一个常量
// const pi = 3.14;
var person="John Doe";
var answer='Yes I am!';
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
var pi=3.14;
var name="Bill Gates";
var answer='Yes I am!';
document.write(pi + "<br>");
document.write(name + "<br>");
document.write(answer + "<br>");
</script>

</body>
</html>
```
输出：

```bash
3.14
Bill Gates
Yes I am!
```
##  3. 声明（创建） JavaScript 变量
我们使用 var 关键词来声明变量：

```bash
var carname;
```

变量声明之后，该变量是空的（它没有值）。

如需向变量赋值，请使用等号：

```bash
carname="Volvo";
```

不过，您也可以在声明变量时对其赋值：

```bash
var carname="Volvo"
```
在下面的例子中，我们创建了名为 carname 的变量，并向其赋值 "Volvo"，然后把它放入 id="demo" 的 HTML 段落中：
```bash
var carname="Volvo";
document.getElementById("demo").innerHTML=carname;
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

<p>点击这里来创建变量，并显示结果。</p>
<button onclick="myFunction()">点击这里</button>
<p id="demo"></p>
<script>
function myFunction(){
	var carname="Volvo";
	document.getElementById("demo").innerHTML=carname;
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/185a7d20287ef314b48681fd82b7d563.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/029f3659ef6cf133c691acb7cbc9c5fd.png)

##  4. 一条语句，多个变量
您可以在一条语句中声明很多变量。该语句以 var 开头，并使用逗号分隔变量即可：

```bash
var lastname="Doe", age=30, job="carpenter";
```

声明也可横跨多行：

```bash
var lastname="Doe",
age=30,
job="carpenter";
```

一条语句中声明的多个变量不可以同时赋同一个值:

```bash
var x,y,z=1;
x,y 为 undefined， z 为 1。
```

##  5. 重新声明 JavaScript 变量
如果重新声明 JavaScript 变量，该变量的值不会丢失：

在以下两条语句执行后，变量 carname 的值依然是 "Volvo"：

```bash
var carname="Volvo";
var carname;
```

##  6. JavaScript 算数
您可以通过 JavaScript 变量来做算数，使用的是 = 和 + 这类运算符：

实例

```bash
y=5;
x=y+2;
```
##  7. 全局变量

```bash
var carName = " Volvo";
 
// 此处可调用 carName 变量
function myFunction() {
    // 函数内可调用 carName 变量
}
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

<p>全局变量在任何脚本和函数内均可访问。</p>
<p id="demo"></p>
<script>
var carName = "Volvo";
myFunction();
function myFunction() 
{
    document.getElementById("demo").innerHTML =
		"我可以显示 " + carName;
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cf7ff95453e18a180a7d2db64b72a682.png)
##  8. 局部变量

```bash
function myFunction() {
    var carName = "Volvo";
    // 函数内可调用 carName 变量
}
```


