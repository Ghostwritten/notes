#  JavaScript 16. 循环语句


![在这里插入图片描述](https://img-blog.csdnimg.cn/eeb2aa8171064d77b2682db9b07941cf.png)


##  1. 简介
JavaScript 支持不同类型的循环：

 - for - 循环代码块一定的次数
 - for/in - 循环遍历对象的属性
 - while - 当指定的条件为 true 时循环指定的代码块
 - do/while - 同样当指定的条件为 true 时循环指定的代码块

一般写法：

```bash
document.write(cars[0] + "<br>"); 
document.write(cars[1] + "<br>"); 
document.write(cars[2] + "<br>"); 
document.write(cars[3] + "<br>"); 
document.write(cars[4] + "<br>"); 
document.write(cars[5] + "<br>");
```

使用for循环

```bash
cars=["BMW","Volvo","Saab","Ford"];
for (var i=0;i<cars.length;i++)
{ 
    document.write(cars[i] + "<br>");
}
```
输出：

```bash
BMW
Volvo
Saab
Ford
```

##  2. For 循环
语法：

```bash
for (语句 1; 语句 2; 语句 3)
{
    被执行的代码块
}
```

 - 语句 1 （代码块）开始前执行
 - 语句 2 定义运行循环（代码块）的条件
 - 语句 3 在循环（代码块）已被执行之后执行

实例：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p>点击按钮循环代码5次。</p>
<button onclick="myFunction()">点击这里</button>
<p id="demo"></p>
<script>
function myFunction(){
	var x="";
	for (var i=0;i<5;i++){
		x=x + "该数字为 " + i + "<br>";
	}
	document.getElementById("demo").innerHTML=x;
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/1ff16b25f07e46ac9e3380e31508edb4.png)
###  2.1 语句 1
通常我们会使用语句 1 初始化循环中所用的变量 (var i=0)。

语句 1 是可选的，也就是说不使用语句 1 也可以。

您可以在语句 1 中初始化任意（或者多个）值：

```bash
for (var i=0,len=cars.length; i<len; i++)
{ 
    document.write(cars[i] + "<br>");
}
```
或者
```bash
var i=2,len=cars.length;
for (; i<len; i++)
{ 
    document.write(cars[i] + "<br>");
}
```
###  2.2 语句 2
通常语句 2 用于评估初始变量的条件。

语句 2 同样是可选的。

如果语句 2 返回 true，则循环再次开始，如果返回 false，则循环将结束。

###  2.3 语句 3
通常语句 3 会增加初始变量的值。

语句 3 也是可选的。

语句 3 有多种用法。增量可以是负数 (i--)，或者更大 (i=i+15)。

语句 3 也可以省略（比如当循环内部有相应的代码时）：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<script>
cars=["BMW","Volvo","Saab","Ford"];
var i=0,len=cars.length;
for (; i<len; ){
	document.write(cars[i] + "<br>");
	i++;
}
</script>

</body>
</html>
```
输出：

```bash
BMW
Volvo
Saab
Ford
```
##  3. For/In 循环

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>
	
<p>点击下面的按钮，循环遍历对象 "person" 的属性。</p>
<button onclick="myFunction()">点击这里</button>
<p id="demo"></p>
<script>
function myFunction(){
	var x;
	var txt="";
	var person={fname:"Bill",lname:"Gates",age:56}; 
	for (x in person){
		txt=txt + person[x];
	}
	document.getElementById("demo").innerHTML=txt;
}
</script>
	
</body>
</html>
```
输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/1ed80b28a09542d8a301fa0fb1bd5569.png)
##  4. while 循环
###  4.1 语法

```bash
while (条件)
{
    需要执行的代码
}
```
### 4.2 实例

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p>点击下面的按钮，只要 i 小于 5 就一直循环代码块。</p>
<button onclick="myFunction()">点击这里</button>
<p id="demo"></p>
<script>
function myFunction(){
	var x="",i=0;
	while (i<5){
		x=x + "该数字为 " + i + "<br>";
		i++;
	}
	document.getElementById("demo").innerHTML=x;
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/d6274df6f1a640cfbd41456d62283186.png)
##  5. do/while 循环
do/while 循环是 while 循环的变体。该循环会在检查条件是否为真之前执行一次代码块，然后如果条件为真的话，就会重复这个循环。

###  5.1 语法

```bash
do
{
    需要执行的代码
}
while (条件);
```
###  5.2 实例
下面的例子使用 do/while 循环。该循环至少会执行一次，即使条件为 false 它也会执行一次，因为代码块会在条件被测试前执行：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p>点击下面的按钮，只要 i 小于 5 就一直循环代码块。</p>
<button onclick="myFunction()">点击这里</button>
<p id="demo"></p>
<script>
function myFunction(){
	var x="",i=0;
	do{
		x=x + "该数字为 " + i + "<br>";
	    i++;
	}
	while (i<5)  
	document.getElementById("demo").innerHTML=x;
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2b0be68d6606405988b57cf8f4211a6d.png)
##  6. 比较 for 和 while
如果您已经阅读了前面那一章关于 for 循环的内容，您会发现 while 循环与 for 循环很像。

本例中的循环使用 for 循环来显示 cars 数组中的所有值：

```bash
cars=["BMW","Volvo","Saab","Ford"];
var i=0;
for (;cars[i];)
{
    document.write(cars[i] + "<br>");
    i++;
}
```

while 
```bash
cars=["BMW","Volvo","Saab","Ford"];
var i=0;
while (cars[i])
{
    document.write(cars[i] + "<br>");
    i++;
}
```

##  7. break 语句
break 语句可用于跳出循环。

break 语句跳出循环后，会继续执行该循环之后的代码（如果有的话）：

```bash
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p>点击按钮，测试带有 break 语句的循环。</p>
<button onclick="myFunction()">点击这里</button>
<p id="demo"></p>
<script>
function myFunction(){
	var x="",i=0;
	for (i=0;i<10;i++){
		if (i==3){
    			break;
			}
    	x=x + "该数字为 " + i + "<br>";
    }
	document.getElementById("demo").innerHTML=x;
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/d02a35446db2400e95821741a1e1c917.png)
由于这个 if 语句只有一行代码，所以可以省略花括号：

```bash
for (i=0;i<10;i++)
{
    if (i==3) break;
    x=x + "The number is " + i + "<br>";
}
```
##  8. continue 语句
continue 语句中断当前的循环中的迭代，然后继续循环下一个迭代。 以下例子在值为 3 时，直接跳过：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p>点击下面的按钮来执行循环，该循环会跳过 i=3 的数字。</p>
<button onclick="myFunction()">点击这里</button>
<p id="demo"></p>
<script>
function myFunction(){
	var x="",i=0;
	for (i=0;i<10;i++){
  		if (i==3){
    		continue;
    	}
		x=x + "该数字为 " + i + "<br>";
  	}
	document.getElementById("demo").innerHTML=x;
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/4f5f9fa77ba046488da1baf54d54cbca.png)
for 实例

```bash
for (i=0;i<=10;i++)
{
    if (i==3) continue;
    x=x + "The number is " + i + "<br>";
}
```
while 实例

```bash
while (i < 10){
  if (i == 3){
    i++;    //加入i++不会进入死循环
    continue;
  }
  x= x + "该数字为 " + i + "<br>";
  i++;
}
```
## 9. JavaScript 标签
可以对 JavaScript 语句进行标记。

如需标记 JavaScript 语句，请在语句之前加上冒号：

```bash
label:
statements
```

break 和 continue 语句仅仅是能够跳出代码块的语句。

语法:

```bash
break labelname; 
 
continue labelname;
```

 - break 的作用是跳出代码块, 所以 break 可以使用于循环和 switch 等
 - continue 的作用是进入下一个迭代, 所以 continue 只能用于循环的代码块。
 - continue 语句（带有或不带标签引用）只能用在循环中。
 - break 语句（不带标签引用），只能用在循环或 switch 中。

通过标签引用，break 语句可用于跳出任何 JavaScript 代码块：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<script>
cars=["BMW","Volvo","Saab","Ford"];
list:{
	document.write(cars[0] + "<br>"); 
	document.write(cars[1] + "<br>"); 
	document.write(cars[2] + "<br>"); 
	break list;
	document.write(cars[3] + "<br>"); 
	document.write(cars[4] + "<br>"); 
	document.write(cars[5] + "<br>"); 
}
</script>

</body>
</html>
```
输出：

```bash
BMW
Volvo
Saab
```

