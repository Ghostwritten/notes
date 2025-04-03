

##  JavaScript 注释

单行注释以 // 开头。

本例用单行注释来解释代码：

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<h1 id="myH1"></h1>
<p id="myP"></p>
<script>
// 输出标题：
document.getElementById("myH1").innerHTML="Welcome to my Homepage";
// 输出段落：
document.getElementById("myP").innerHTML="This is my first paragraph.";
</script>
<p><b>注释：</b>注释不会被执行。</p>

</body>
</html>
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f2bdb319dd599a2f21688b40dcb1591c.png)

多行注释以 /* 开始，以 */ 结尾。

下面的例子使用多行注释来解释代码：

```bash
<!DOCTYPE html>
<html>
<head>
<title>菜鸟教程测试实例</title>
<meta charset="utf-8">
</head>
<body>

<h1 id="myH1"></h1>
<p id="myP"></p>
<script>
/*
下面的这些代码会输出
一个标题和一个段落
并将代表主页的开始
*/
document.getElementById("myH1").innerHTML="欢迎来到菜鸟教程";
document.getElementById("myP").innerHTML="这是一个段落。";
</script>
<p><b>注释：</b>注释块不会被执行。</p>

</body>
</html>
```

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<p id="myP"></p>
<script>
var x=5;   // 声明 x 并把 5 赋值给它
var y=x+2;   // 声明 y 并把 x+2 赋值给它
document.getElementById("myP").innerHTML=y // 把 y 的值写到 myP
</script>
<p><b>注释：</b>注释不会被执行。</p>

</body>
</html>
```

##   变量

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<script>
var x=5;
var y=6;
var z=x+y;
document.write(x + "<br>");
document.write(y + "<br>");
document.write(z + "<br>");
</script>

</body>
</html>
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2c9cf9a7c3b0e860514b7a8ccad8ef77.png)
变量规范：

 - 变量必须以字母开头
 - 变量也能以 $ 和 _ 符号开头（不过我们不推荐这么做）
 - 变量名称对大小写敏感（y 和 Y 是不同的变量）


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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5cdaccf0157ac405628ea974f5eb3116.png)
##  数据类型
值类型(基本类型)：字符串（String）、数字(Number)、布尔(Boolean)、对空（Null）、未定义（Undefined）、Symbol。

引用数据类型：对象(Object)、数组(Array)、函数(Function)。


### JavaScript 拥有动态类型
JavaScript 拥有动态类型。这意味着相同的变量可用作不同的类型：
var x;               // x 为 undefined
var x = 5;           // 现在 x 为数字
var x = "John";      // 现在 x 为字符串


###  JavaScript 字符串
字符串是存储字符（比如 "Bill Gates"）的变量。

字符串可以是引号中的任意文本。您可以使用单引号或双引号：

```bash
var carname="Volvo XC60";
var carname='Volvo XC60';
var answer="It's alright";
var answer="He is called 'Johnny'";
var answer='He is called "Johnny"';
```
示例：
```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<script>
var carname1="Volvo XC60";
var carname2='Volvo XC60';
var answer1='It\'s alright';
var answer2="He is called \"Johnny\"";
var answer3='He is called "Johnny"';
document.write(carname1 + "<br>")
document.write(carname2 + "<br>")
document.write(answer1 + "<br>")
document.write(answer2 + "<br>")
document.write(answer3 + "<br>")
</script>

</body>
</html>
```
###  JavaScript 数字

```bash
var x1=34.00;      //使用小数点来写
var x2=34;         //不使用小数点来写
#极大或极小的数字可以通过科学（指数）计数法来书写：
var y=123e5;      // 12300000
var z=123e-5;     // 0.00123
```
示例：

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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f8d288c6d93c6183357b94cd5b21ee4e.png)
### JavaScript 布尔

```bash
var x=true;
var y=false;
```

###  JavaScript 数组

```bash
var cars=new Array();
cars[0]="Saab";
cars[1]="Volvo";
cars[2]="BMW";
```
或者 (condensed array):

```bash
var cars=new Array("Saab","Volvo","BMW");
```
示例：

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
//var cars=["Saab","Volvo","BMW"];
for (i=0;i<cars.length;i++)
{
document.write(cars[i] + "<br>");
}
</script>

</body>
</html>
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f40be1860b55b06fced8019451b93cc5.png)
###  JavaScript 对象
对象由花括号分隔。在括号内部，对象的属性以名称和值对的形式 (name : value) 来定义。属性由逗号分隔：

```bash
var person={firstname:"John", lastname:"Doe", id:5566};
```
上面例子中的对象 (person) 有三个属性：firstname、lastname 以及 id。

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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5258122a5d1b51ae434a2d40adb87ee9.png)
###  Undefined 和 Null
Undefined 这个值表示变量不含有值。

可以通过将变量的值设置为 null 来清空变量。

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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/507cb4d25bee278cd3142ac84fd4ab1f.png)
###  声明变量类型
当您声明新变量时，可以使用关键词 "new" 来声明其类型：

```bash
var carname=new String;
var x=      new Number;
var y=      new Boolean;
var cars=   new Array;
var person= new Object;
```

##  对象

真实生活中的对象，属性和方法
真实生活中，一辆汽车是一个对象。

对象有它的属性，如重量和颜色等，方法有启动停止等:

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cebc9b8f65b0cd52d3278f767367b661.png)
在 JavaScript中，几乎所有的事物都是对象。

```bash
var car = "Fiat";
```

对象也是一个变量，但对象可以包含多个值（多个变量），每个值以 name:value 对呈现。

```bash
var car = {name:"Fiat", model:500, color:"white"};
```
###  对象定义
你可以使用字符来定义和创建 JavaScript 对象:


```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<p>创建 JavaScript 对象。</p>
<p id="demo"></p>
<script>
var person = {firstName:"John", lastName:"Doe", age:50, eyeColor:"blue"};
document.getElementById("demo").innerHTML =
	person.firstName + " 现在 " + person.age + " 岁.";
</script>

</body>
</html>
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f2871c79c08d557fa91602b1eab661df.png)


定义 JavaScript 对象可以跨越多行，空格跟换行不是必须的：

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<p>创建 JavaScript 对象。</p>
<p id="demo"></p>
<script>
var person = {
    firstName : "John",
    lastName  : "Doe",
    age       : 50,
    eyeColor  : "blue"
};
document.getElementById("demo").innerHTML =
	person.firstName + " 现在 " + person.age + " 岁。";
</script>

</body>
</html>
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8412a3f2c0c210aa3cc1aead7448760d.png)
###  对象属性
可以说 "JavaScript 对象是变量的容器"。

但是，我们通常认为 "JavaScript 对象是键值对的容器"。

键值对通常写法为 name : value (键与值以冒号分割)。

键值对在 JavaScript 对象通常称为 对象属性。

对象键值对的写法类似于：

 - PHP 中的关联数组
 - Python 中的字典
 - C 语言中的哈希表
 - Java 中的哈希映射
 - Ruby 和 Perl 中的哈希表


###  访问对象属性

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<p>
有两种方式可以访问对象属性：
</p>
<p>
你可以使用 .property 或 ["property"].
</p>
<p id="demo"></p>
<script>
var person = {
    firstName : "John",
    lastName : "Doe",
    id : 5566
};
document.getElementById("demo").innerHTML =
	person.firstName + " " + person.lastName;
</script>

</body>
</html>
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b3588f27367f3187f89df62d858b88c0.png)
###  对象方法
对象的方法定义了一个函数，并作为对象的属性存储。

对象方法通过添加 () 调用 (作为一个函数)。

该实例访问了 person 对象的 fullName() 方法:

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<p>创建和使用对象方法。</p>
<p>对象方法作为一个函数定义存储在对象属性中。</p>
<p id="demo"></p>
<script>
var person = {
    firstName: "John",
    lastName : "Doe",
    id : 5566,
    fullName : function() 
	{
       return this.firstName + " " + this.lastName;
    }
};
document.getElementById("demo").innerHTML = person.fullName();
</script>
	
</body>
</html>
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2c0a6f73b382df4ce71c7a9c413d9752.png)
如果你要访问 person 对象的 fullName 属性，它将作为一个定义函数的字符串返回：
```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<p>创建和使用对象方法。</p>
<p>对象方法是一个函数定义,并作为一个属性值存储。</p>
<p id="demo1"></p>
<p id="demo2"></p>
<script>
var person = {
    firstName: "John",
    lastName : "Doe",
    id : 5566,
    fullName : function() 
	{
       return this.firstName + " " + this.lastName;
    }
};
document.getElementById("demo1").innerHTML = "不加括号输出函数表达式："  + person.fullName;
document.getElementById("demo2").innerHTML = "加括号输出函数执行结果："  +  person.fullName();
</script>
	
</body>
</html>
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2beefb3633a7c737006184c46ee22287.png)
##  函数
函数是由事件驱动的或者当它被调用时执行的可重复使用的代码块。

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>测试实例</title>
<script>
function myFunction()
{
    alert("Hello World!");
}
</script>
</head>
 
<body>
<button onclick="myFunction()">点我</button>
</body>
</html>
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bb5338af54f6270c979b6a6ff88b6c25.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6b5646a696527c0ad0e1990e9d95b8e1.png)
###  JavaScript 函数语法

格式：

```bash
function functionname()
{
    // 执行代码
}
```
当调用该函数时，会执行函数内的代码。

可以在某事件发生时直接调用函数（比如当用户点击按钮时），并且可由 JavaScript 在任何位置进行调用。

###  调用带参数的函数
在调用函数时，您可以向其传递值，这些值被称为参数。这些参数可以在函数中使用。您可以发送任意多的参数，由逗号 (,) 分隔：

```bash
function myFunction(var1,var2)
{
代码
}
```
变量和参数必须以一致的顺序出现。第一个变量就是第一个被传递的参数的给定的值，以此类推。

```bash
<!DOCTYPE html>
<html>	
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<p>点击这个按钮，来调用带参数的函数。</p>
<button onclick="myFunction('Harry Potter','Wizard')">点击这里</button>
<script>
function myFunction(name,job){
	alert("Welcome " + name + ", the " + job);
}
</script>

</body>
</html>
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5266b99a4b31132f95ee41241b5a5e4e.png)

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<p>请点击其中的一个按钮，来调用带参数的函数。</p>
<button onclick="myFunction('Harry Potter','Wizard')">点击这里</button>
<button onclick="myFunction('Bob','Builder')">点击这里</button>
<script>
function myFunction(name,job)
{
	alert("Welcome " + name + ", the " + job);
}
</script>

</body>
</html>
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e7d5eb5cba29e15632abee654f0e889e.png)

###  带有返回值的函数
有时，我们会希望函数将值返回调用它的地方。通过使用 return 语句就可以实现。在使用 return 语句时，函数会停止执行，并返回指定的值。

```bash
function myFunction()
{
    var x=5;
    return x;
}
```
注意： 整个 JavaScript 并不会停止执行，仅仅是函数。JavaScript 将继续执行代码，从调用函数的地方。

函数调用将被返回值取代：

```bash
var myVar=myFunction();
```

myVar 变量的值是 5，也就是函数 "myFunction()" 所返回的值。

即使不把它保存为变量，您也可以使用返回值：

```bash
document.getElementById("demo").innerHTML=myFunction();
```

"demo" 元素的 innerHTML 将成为 5，也就是函数 "myFunction()" 所返回的值。

您可以使返回值基于传递到函数中的参数：

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<p>本例调用的函数会执行一个计算，然后返回结果：</p>
<p id="demo"></p>
<script>
function myFunction(a,b){
	return a*b;
}
document.getElementById("demo").innerHTML=myFunction(4,3);
</script>

</body>
</html>
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2ab52f59192e8db7bc6eb989f74eed95.png)
在您仅仅希望退出函数时 ，也可使用 return 语句。返回值是可选的：

```bash
function myFunction(a,b)
{
    if (a>b)
    {
        return;
    }
    x=a+b
}
```
如果 a 大于 b，则上面的代码将退出函数，并不会计算 a 和 b 的总和。

### 局部 JavaScript 变量
在 JavaScript 函数内部声明的变量（使用 var）是局部变量，所以只能在函数内部访问它。（该变量的作用域是局部的）。

您可以在不同的函数中使用名称相同的局部变量，因为只有声明过该变量的函数才能识别出该变量。

只要函数运行完毕，本地变量就会被删除。

### 全局 JavaScript 变量
在函数外声明的变量是全局变量，网页上的所有脚本和函数都能访问它。

JavaScript 变量的生存期
JavaScript 变量的生命期从它们被声明的时间开始。

局部变量会在函数运行以后被删除。

全局变量会在页面关闭后被删除。

### 向未声明的 JavaScript 变量分配值
如果您把值赋给尚未声明的变量，该变量将被自动作为 window 的一个属性。

这条语句：

```bash
carname="Volvo";
```

将声明 window 的一个属性 carname。

非严格模式下给未声明变量赋值创建的全局变量，是全局对象的可配置属性，可以删除。

```bash
var var1 = 1; // 不可配置全局属性
var2 = 2; // 没有使用 var 声明，可配置全局属性

console.log(this.var1); // 1
console.log(window.var1); // 1
console.log(window.var2); // 2

delete var1; // false 无法删除
console.log(var1); //1

delete var2; 
console.log(delete var2); // true
console.log(var2); // 已经删除 报错变量未定义
```

##  作用域
在 JavaScript 中, 对象和函数同样也是变量。
在 JavaScript 中, 作用域为可访问变量，对象，函数的集合。
JavaScript 函数作用域: 作用域在函数内修改。
###  JavaScript 局部作用域
```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<p>局部变量在声明的函数外不可以访问。</p>
<p id="demo"></p>
<script>
myFunction();
document.getElementById("demo").innerHTML = "carName 的类型是：" +  typeof carName;
function myFunction() 
{
    var carName = "Volvo";
}
</script>

</body>
</html>
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3bb60dd9c32ebd3939b2ae2cc6338a95.png)
因为局部变量只作用于函数内，所以不同的函数可以使用相同名称的变量。
局部变量在函数开始执行时创建，函数执行完后局部变量会自动销毁。

###  JavaScript 全局变量
变量在函数外定义，即为全局变量。
全局变量有 全局作用域: 网页中所有脚本和函数均可使用。

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<p>
如果你的变量没有声明，它将自动成为全局变量：
</p>
<p id="demo"></p>
<script>
myFunction();
document.getElementById("demo").innerHTML =
	"我可以显示 " + carName;
function myFunction() 
{
    carName = "Volvo";
}
</script>

</body>
</html> 
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6e8cca662c2b5fbcc5b8ff88047d178b.png)
###  JavaScript 变量生命周期
JavaScript 变量生命周期在它声明时初始化。

局部变量在函数执行完毕后销毁。

全局变量在页面关闭后销毁。

###  HTML 中的全局变量

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<p>
在 HTML 中, 所有全局变量都会成为 window 变量。
</p>
<p id="demo"></p>
<script>
myFunction();
document.getElementById("demo").innerHTML =
	"我可以显示 " + window.carName;
function myFunction() 
{
    carName = "Volvo";
}
</script>

</body>
</html>
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7a823e18c241b4fea524874e6481baf1.png)
##  事件
HTML 事件是发生在 HTML 元素上的事情。当在 HTML 页面中使用 JavaScript 时， JavaScript 可以触发这些事件。

HTML 事件可以是浏览器行为，也可以是用户行为。

以下是 HTML 事件的实例：

 - HTML 页面完成加载
 - HTML input 字段改变时
 - HTML 按钮被点击

通常，当事件发生时，你可以做些事情。

在事件触发时 JavaScript 可以执行一些代码。

HTML 元素中可以添加事件属性，使用 JavaScript 代码来添加 HTML 元素。


单引号:

```bash
<some-HTML-element some-event='JavaScript 代码'>
```

双引号:

```bash
<some-HTML-element some-event="JavaScript 代码">
```
在以下实例中，按钮元素中添加了 onclick 属性 (并加上代码):

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<button onclick="getElementById('demo').innerHTML=Date()">现在的时间是?</button>
<p id="demo"></p>

</body>
</html>
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e5d338c4a245a29fc4bd6b93c742e13d.png)
以上实例中，JavaScript 代码将修改 id="demo" 元素的内容。

在下一个实例中，代码将修改自身元素的内容 (使用 this.innerHTML):

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<button onclick="this.innerHTML=Date()">现在的时间是?</button>

</body>
</html>
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/59f6c352835e0b6faf221ced69858158.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/32936e1d4273c4af29afb33e6b2687cd.png)

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<p>点击按钮执行 <em>displayDate()</em> 函数.</p>
<button onclick="displayDate()">点这里</button>
<script>
function displayDate(){
	document.getElementById("demo").innerHTML=Date();
}
</script>
<p id="demo"></p>

</body>
</html>
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c6449eda8b0888a11fca6b0f9e8b934c.png)

