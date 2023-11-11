#  JavaScript 9. 数据类型



![在这里插入图片描述](https://img-blog.csdnimg.cn/0d05fff47bd34f1598800795ebf990c0.png)



 - 值类型(基本类型)：**字符串（String）、数字(Number)、布尔(Boolean)、空（Null）、未定义（Undefined）、Symbol**。

- 引用数据类型（对象类型）：**对象(Object)、数组(Array)、函数(Function)，还有两个特殊的对象：正则（RegExp）和日期（Date）**。

![在这里插入图片描述](https://img-blog.csdnimg.cn/7f5b331b13a1454d986cf1b793998117.png)

##   1. JavaScript 拥有动态类型
JavaScript 拥有动态类型。这意味着相同的变量可用作不同的类型：

```bash
var x;               // x 为 undefined
var x = 5;           // 现在 x 为数字
var x = "John";      // 现在 x 为字符串
```
变量的数据类型可以使用 `typeof` 操作符来查看：

```bash
typeof "John"                // 返回 string
typeof 3.14                  // 返回 number
typeof false                 // 返回 boolean
typeof [1,2,3,4]             // 返回 object
typeof {name:'John', age:34} // 返回 object
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

<p> typeof 操作符返回变量或表达式的类型。</p>
<p id="demo"></p>
<script>
document.getElementById("demo").innerHTML = 
	typeof "john" + "<br>" + 
	typeof 3.14 + "<br>" +
	typeof false + "<br>" +
	typeof [1,2,3,4] + "<br>" +
	typeof {name:'john', age:34};
</script>

</body>
</html>
```
输出：

```bash
typeof 操作符返回变量或表达式的类型。

string
number
boolean
object
object
```
##  2. JavaScript 字符串
字符串是存储字符（比如 "Bill Gates"）的变量。

字符串可以是引号中的任意文本。您可以使用单引号或双引号：

```bash
var carname="Volvo XC60";
var carname='Volvo XC60';
```
您可以在字符串中使用引号，只要不匹配包围字符串的引号即可：

```bash
var answer="It's alright";
var answer="He is called 'Johnny'";
var answer='He is called "Johnny"';
```
##  3. JavaScript 数字
JavaScript 只有一种数字类型。数字可以带小数点，也可以不带：

```bash
var x1=34.00;      //使用小数点来写
var x2=34;         //不使用小数点来写
```
极大或极小的数字可以通过科学（指数）计数法来书写：

```bash
var y=123e5;      // 12300000
var z=123e-5;     // 0.00123
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
var x1=34.00;
var x2=34;
var y=123e5;
var z=123e-5;
document.write(x1 + "<br>")
document.write(x2 + "<br>")
document.write(y + "<br>")
document.write(z + "<br>")
</script>

</body>
</html>
```
输出：

```bash
34
34
12300000
0.00123
```
##  4. JavaScript 布尔
布尔（逻辑）只能有两个值：true 或 false。

```bash
var x=true;
var y=false;
```

##  5. JavaScript 数组

```bash
var cars=new Array();
cars[0]="Saab";
cars[1]="Volvo";
cars[2]="BMW";
```
或者

```bash
var cars=new Array("Saab","Volvo","BMW");
```
实例：

```bash
<!DOCTYPE html>
<html>
<body>

<script>
var i;
var cars = new Array();
cars[0] = "Saab";
cars[1] = "Volvo";
cars[2] = "BMW";

for (i=0;i<cars.length;i++)
{
document.write(cars[i] + "<br>");
}
</script>

</body>
</html>


```
输出：

```bash
Saab
Volvo
BMW
```
##  6. JavaScript 对象
对象由花括号分隔。在括号内部，对象的属性以名称和值对的形式 (name : value) 来定义。属性由逗号分隔：

```bash
var person={firstname:"John", lastname:"Doe", id:5566};
```

上面例子中的对象 (person) 有三个属性：`firstname`、`lastname` 以及 `id`。

空格和折行无关紧要。声明可横跨多行：

```bash
var person={
firstname : "John",
lastname  : "Doe",
id        :  5566
};
```
对象属性有两种寻址方式：

```bash
name=person.lastname;
name=person["lastname"];
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
var person=
{
	firstname : "John",
	lastname  : "Doe",
	id        :  5566
};
document.write(person.lastname + "<br>");
document.write(person["lastname"] + "<br>");
</script>

</body>
</html>
```
输出:

```bash
Doe
Doe
```
## 7. Undefined 和 Null
Undefined 这个值表示变量不含有值。

可以通过将变量的值设置为 null 来清空变量。

```bash
cars=null;
person=null;
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
var person;
var car="Volvo";
document.write(person + "<br>");
document.write(car + "<br>");
var car=null
document.write(car + "<br>");
</script>

</body>
</html>
```
输出：

```bash
undefined
Volvo
null
```
## 8. 声明变量类型
当您声明新变量时，可以使用关键词 "`new`" 来声明其类型：

```bash
var carname=new String;
var x=      new Number;
var y=      new Boolean;
var cars=   new Array;
var person= new Object;
```

