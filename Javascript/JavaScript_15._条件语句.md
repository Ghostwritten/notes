#  JavaScript 15. 条件语句


![在这里插入图片描述](https://img-blog.csdnimg.cn/44e9e6f34c1c43669993e9171164babe.png)

##  1. 简介
在 JavaScript 中，我们可使用以下条件语句：

 - if 语句 - 只有当指定条件为 true 时，使用该语句来执行代码
 - if...else 语句 - 当条件为 true 时执行代码，当条件为 false 时执行其他代码
 - if...else if....else 语句- 使用该语句来选择多个代码块之一来执行
 - switch 语句 - 使用该语句来选择多个代码块之一来执行

##  2. if 语句
### 2.1 语法

```bash
if (condition)
{
    当条件为 true 时执行的代码
}
```

请使用小写的 if。使用大写字母（IF）会生成 JavaScript 错误！

### 2.2 实例
当时间小于 20:00 时，生成问候 "Good day"：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p>如果时间早于 20:00，会获得问候 "Good day"。</p>
<button onclick="myFunction()">点击这里</button>
<p id="demo"></p>
<script>
function myFunction(){
	var x="";
	var time=new Date().getHours();
	if (time<20){
		x="Good day";
    }
	document.getElementById("demo").innerHTML=x;
}
</script>

</body>
</html>
```

x 的结果是：
![在这里插入图片描述](https://img-blog.csdnimg.cn/b9277dfdcc83424794356e0d4516a8b5.png)

##  3. if...else 语句

### 3.1 语法

```bash
if (condition)
{
    当条件为 true 时执行的代码
}
else
{
    当条件不为 true 时执行的代码
}
```

### 3.2 实例
当时间小于 20:00 时，生成问候 "Good day"，否则生成问候 "Good evening"。

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p>点击这个按钮，获得基于时间的问候。</p>
<button onclick="myFunction()">点击这里</button>
<p id="demo"></p>
<script>
function myFunction(){
	var x="";
	var time=new Date().getHours();
	if (time<20){
	 	x="Good day";
     }
	else{
 		x="Good evening";
 	}
	document.getElementById("demo").innerHTML=x;
}
</script>
```
输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/9cbc5c5cb5cb4863bd6e2c1c4db49968.png)
##  4. if...else if...else 语句
### 4.1 语法

```bash
if (condition1)
{
    当条件 1 为 true 时执行的代码
}
else if (condition2)
{
    当条件 2 为 true 时执行的代码
}
else
{
  当条件 1 和 条件 2 都不为 true 时执行的代码
}
```

### 4.2 实例
如果时间小于 10:00，则生成问候 "Good morning"，如果时间大于 10:00 小于 20:00，则生成问候 "Good day"，否则生成问候 "Good evening"：

```bash
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<script type="text/javascript">
var d = new Date();
var time = d.getHours();
if (time<10)
{
	document.write("<b>早上好</b>");
}
else if (time>=10 && time<20)
{
	document.write("<b>今天好</b>");
}
else
{
	document.write("<b>晚上好!</b>");
}
</script>
<p>
这个例子演示了 if..else if...else 语句。
</p>

</body>
</html>
```
输出：

```bash
今天好
这个例子演示了 if..else if...else 语句。
```
##  5. switch 语句

```bash
语法
switch(n)
{
    case 1:
        执行代码块 1
        break;
    case 2:
        执行代码块 2
        break;
    default:
        与 case 1 和 case 2 不同时执行的代码
}
```
工作原理：首先设置表达式 n（通常是一个变量）。随后表达式的值会与结构中的每个 case 的值做比较。如果存在匹配，则与该 case 关联的代码块会被执行。请使用 break 来阻止代码自动地向下一个 case 运行。

实例
显示今天的星期名称。请注意 Sunday=0, Monday=1, Tuesday=2, 等等：

```bash
var d=new Date().getDay(); 
switch (d) 
{ 
  case 0:x="今天是星期日"; 
  break; 
  case 1:x="今天是星期一"; 
  break; 
  case 2:x="今天是星期二"; 
  break; 
  case 3:x="今天是星期三"; 
  break; 
  case 4:x="今天是星期四"; 
  break; 
  case 5:x="今天是星期五"; 
  break; 
  case 6:x="今天是星期六"; 
  break; 
}
x 的运行结果：

今天是星期五
```
##  6. default 关键词
请使用 default 关键词来规定匹配不存在时做的事情：

```bash
实例
如果今天不是星期六或星期日，则会输出默认的消息：

var d=new Date().getDay();
switch (d)
{
    case 6:x="今天是星期六";
    break;
    case 0:x="今天是星期日";
    break;
    default:
    x="期待周末";
}
document.getElementById("demo").innerHTML=x;
x 的运行结果：

期待周末
```

