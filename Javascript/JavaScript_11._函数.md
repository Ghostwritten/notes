#  JavaScript 11. 函数
tags: javascript



![在这里插入图片描述](https://img-blog.csdnimg.cn/af26356f07ac490f8880ff843285c0dc.png)

##  


##  1. 函数实例
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

实例2

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
输出：

```bash
本例调用的函数会执行一个计算，然后返回结果：

12
```


##  2. JavaScript 函数语法

```bash
function functionname()
{
    // 执行代码
}
```

> JavaScript 对大小写敏感。关键词 function 必须是小写的，并且必须以与函数名称相同的大小写来调用函数。


##  3. 函数参数
在调用函数时，您可以向其传递值，这些值被称为参数。

这些参数可以在函数中使用。

您可以发送任意多的参数，由逗号 (,) 分隔：

```bash
myFunction(argument1,argument2)
```
当您声明函数时，请把参数作为变量来声明：

```bash
function myFunction(var1,var2)
{
代码
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
输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/5f60ead7e05f4394a479bb39f2de6ae3.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/17e29e006a804797abbe9f00d9a04696.png)
###  3.1 参数规则

 - JavaScript 函数定义显式参数时没有指定数据类型。
 - JavaScript 函数对隐式参数没有进行类型检测。
 - JavaScript 函数对隐式参数的个数没有进行检测。

### 3.2 默认参数
ES5 中如果函数在调用时未提供隐式参数，参数会默认设置为： undefined

有时这是可以接受的，但是建议最好为参数设置一个默认值：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p>设置参数的默认值。</p>
<p id="demo"></p>
<script>
function myFunction(x, y) {
    if (y === undefined) {
        y = 0;
    }    
    return x * y;
}
document.getElementById("demo").innerHTML = myFunction(4);
</script>

</body>
</html>
```
输出：

```bash
设置参数的默认值。

0
```
或者，更简单的方式

```bash
function myFunction(x, y) {
    y = y || 0;
}
```
如果 y 已经定义，y || 0 返回 y，因为 y 是 true，否则返回 0，因为 `undefined` 为 false。

### 3.3 ES6 函数可以自带参数
ES6 支持函数带有默认参数，就判断 undefined 和 || 的操作：

```bash
function myFunction(x, y = 10) {
    // y is 10 if not passed or undefined
    return x + y;
}
 
myFunction(0, 2) // 输出 2
myFunction(5); // 输出 15, y 参数的默认值
```
### 3.4 arguments 对象
JavaScript 函数有个内置的对象 arguments 对象。

argument 对象包含了函数调用的参数数组。

通过这种方式你可以很方便的找到最大的一个参数的值：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p>查找最大的数。</p>
<p id="demo"></p>
<script>
x = findMax(1, 123, 500, 115, 44, 88);
 
function findMax() {
    var i, max = arguments[0];
    
    if(arguments.length < 2) return max;
 
    for (i = 0; i < arguments.length; i++) {
        if (arguments[i] > max) {
            max = arguments[i];
        }
    }
    return max;
}
document.getElementById("demo").innerHTML = x;
</script>

</body>
</html>
```
输出：

```bash
查找最大的数。

500
```

**或者创建一个函数用来统计所有数值的和：**

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p>计算所有参数之和：</p>
<p id="demo"></p>
<script>
function sumAll() {
    var i, sum = 0;
    for(i = 0; i < arguments.length; i++) {
        sum += arguments[i];
    }
    return sum;
} 
document.getElementById("demo").innerHTML =
	sumAll(1, 123, 500, 115, 44, 88);
</script>

</body>
</html>
```
输出：

```bash
计算所有参数之和：

871
```


##  4. 带有返回值的函数

```bash
function myFunction()
{
    var x=5;
    return x;
}
```

上面的函数会返回值 5
整个 JavaScript 并不会停止执行，仅仅是函数。JavaScript 将继续执行代码，从调用函数的地方。

函数调用将被返回值取代：

```bash
var myVar=myFunction();
```

myVar 变量的值是 5，也就是函数 "`myFunction()`" 所返回的值。

```bash
document.getElementById("demo").innerHTML=myFunction();
```
"demo" 元素的 innerHTML 将成为 5，也就是函数 "`myFunction()`" 所返回的值。

您可以使返回值基于传递到函数中的参数：
实例：

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
输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/005093712ad742e5af8e1e3e292f09e9.png)
##  5. 局部 JavaScript 变量
在 JavaScript 函数内部声明的变量（使用 var）是局部变量，所以只能在函数内部访问它。（该变量的作用域是局部的）。

您可以在不同的函数中使用名称相同的局部变量，因为只有声明过该变量的函数才能识别出该变量。

只要函数运行完毕，本地变量就会被删除。

##  6. 全局 JavaScript 变量
在函数外声明的变量是全局变量，网页上的所有脚本和函数都能访问它。

## 7. JavaScript 变量的生存期
JavaScript 变量的生命期从它们被声明的时间开始。

局部变量会在函数运行以后被删除。

全局变量会在页面关闭后被删除。

##  8. 函数表达式
JavaScript 函数可以通过一个表达式定义。

函数表达式可以存储在变量中：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p>函数可以存储在变量中:</p>
<p id="demo"></p>
<script>
var x = function (a, b) {return a * b};
document.getElementById("demo").innerHTML = x;
</script>

</body>
</html>
```
输出：

```bash
函数可以存储在变量中:

function (a, b) {return a * b}
```
实例

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p>函数存储在变量后，变量可作为函数使用：</p>
<p id="demo"></p>
<script>
var x = function (a, b) {return a * b};
document.getElementById("demo").innerHTML = x(4, 3);
</script>

</body>
</html>
```
输出：

```bash
函数存储在变量后，变量可作为函数使用：

12
```
以上函数实际上是一个 `匿名函数` (函数没有名称)。

函数存储在变量中，不需要函数名称，通常通过变量名来调用。

## 9. Function() 构造函数

```bash
var myFunction = new Function("a", "b", "return a * b");

var x = myFunction(4, 3);
```
也这样写：
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
var myFunction = function (a, b) {return a * b};
document.getElementById("demo").innerHTML = myFunction(4, 3);
</script>

</body>
</html>
```
## 10. 函数提升（Hoisting）
提升（Hoisting）是 JavaScript 默认将当前作用域提升到前面去的行为。

提升（Hoisting）应用在变量的声明与函数的声明。

因此，函数可以在声明之前调用：

```bash
myFunction(5);

function myFunction(y) {
    return y * y;
}
```
## 11. 函数调用
### 11.1 函数作为方法调用
在 JavaScript 中你可以将函数定义为对象的方法。

以下实例创建了一个对象 (myObject), 对象有两个属性 (firstName 和 lastName), 及一个方法 (fullName):

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p>myObject.fullName() 返回 John Doe:</p>
<p id="demo"></p>
<script>
var myObject = {
    firstName:"John",
    lastName: "Doe",
    fullName: function() {
		return this.firstName + " " + this.lastName;
    }
}
document.getElementById("demo").innerHTML = myObject.fullName(); 
</script>

</body>
</html>
```
输出：

```bash
myObject.fullName() 返回 John Doe:

John Doe
```
`fullName` 方法是一个函数。函数属于对象。 `myObject` 是函数的所有者。

this对象，拥有 JavaScript 代码。实例中 this 的值为 myObject 对象。

测试以下！修改 fullName 方法并返回 this 值:

```bash
var myObject = {
    firstName:"John",
    lastName: "Doe",
    fullName: function () {
        return this;
    }
}
myObject.fullName();          // 返回 [object Object] (所有者对象)
```
### 11.2 使用构造函数调用函数
如果函数调用前使用了 `new` 关键字, 则是调用了构造函数。

这看起来就像创建了新的函数，但实际上 JavaScript 函数是重新创建的对象：

```bash
// 构造函数:
function myFunction(arg1, arg2) {
    this.firstName = arg1;
    this.lastName  = arg2;
}
 
// This    creates a new object
var x = new myFunction("John","Doe");
x.firstName;                             // 返回 "John"
```

### 11.3 作为函数方法调用函数
在 JavaScript 中, 函数是对象。JavaScript 函数有它的属性和方法。

`call()` 和 `apply()` 是预定义的函数方法。 两个方法可用于调用函数，两个方法的第一个参数必须是对象本身。

```bash
<!DOCTYPE html>
<html>
<body>

<p id="demo"></p>

<script>
var myObject;
function myFunction(a, b) {
    return a * b;
}
myObject = myFunction.call(myObject, 10, 2);    // 返回 20
document.getElementById("demo").innerHTML = myObject; 
</script>

</body>
</html>
```
输出：20

实例2

```bash
<!DOCTYPE html>
<html>
<body>

<p id="demo"></p>

<script>
var myObject, myArray;
function myFunction(a, b) {
    return a * b;
}
myArray = [10, 2]
myObject = myFunction.apply(myObject, myArray);      // 返回 20
document.getElementById("demo").innerHTML = myObject; 
</script>

</body>
</html>
```
输出： 20
两个方法都使用了对象本身作为第一个参数。 两者的区别在于第二个参数： 

 - `apply`传入的是一个参数数组，也就是将多个参数组合成为一个数组传入
 - 而call则作为call的参数传入（从第二个参数开始）。

在 JavaScript 严格模式(strict mode)下, 在调用函数时第一个参数会成为 this 的值， 即使该参数不是一个对象。

在 JavaScript 非严格模式(non-strict mode)下, 如果第一个参数的值是 null 或 undefined, 它将使用全局对象替代。


### 11.4 自调用函数
函数表达式可以 "自调用"。

自调用表达式会自动调用。

如果表达式后面紧跟 `()` ，则会自动调用。

不能自调用声明的函数。

通过添加括号，来说明它是一个函数表达式：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p>函数可以自动调用：</p>
<p id="demo"></p>
<script>
(function () {
    document.getElementById("demo").innerHTML = "Hello! 我是自己调用的";
})();
</script>

</body>
</html>
```
输出：

```bash
函数可以自动调用：

Hello! 我是自己调用的
```
## 12 函数可作为一个值使用
JavaScript 函数作为一个值使用：
实例1
```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p>函数可作为一个值：</p>
<p>x = myFunction(4,3) 或 x = 12</p>
<p>两种情况下，x 的值都为 12。</p>
<p id="demo"></p>
<script>
function myFunction(a, b) {
    return a * b;
}
var x = myFunction(4, 3);
document.getElementById("demo").innerHTML = x;
</script>

</body>
</html>
```
输出：

```bash
函数可作为一个值：

x = myFunction(4,3) 或 x = 12

两种情况下，x 的值都为 12。

12
```
实例2

```bash
function myFunction(a, b) {
    return a * b;
}

var x = myFunction(4, 3) * 2;
```
## 13 函数是对象
在 JavaScript 中使用 `typeof` 操作符判断函数类型将返回 "function" 。

但是JavaScript 函数描述为一个对象更加准确。

JavaScript 函数有 属性 和 方法。

`arguments.length` 属性返回函数调用过程接收到的参数个数：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p> arguments.length 属性返回函数接收到参数的个数：</p>
<p id="demo"></p>
<script>
function myFunction(a, b) {
    return arguments.length;
}
document.getElementById("demo").innerHTML = myFunction(4, 3);
</script>

</body>
</html>
```
输出：

```bash
arguments.length 属性返回函数接收到参数的个数：

2
```
`toString()` 方法将函数作为一个字符串返回:

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p> toString() 将函数作为一个字符串返回：</p>
<p id="demo"></p>
<script>
function myFunction(a, b) {
    return a * b;
}
document.getElementById("demo").innerHTML = myFunction.toString();
</script>

</body>
</html>
```
输出：

```bash
toString() 将函数作为一个字符串返回：

function myFunction(a, b) { return a * b; }
```

## 14 箭头函数
ES6 新增了箭头函数。

箭头函数表达式的语法比普通函数表达式更简洁。

```bash
(参数1, 参数2, …, 参数N) => { 函数声明 }

(参数1, 参数2, …, 参数N) => 表达式(单一)
// 相当于：(参数1, 参数2, …, 参数N) =>{ return 表达式; }
```
当只有一个参数时，圆括号是可选的：

```bash
(单一参数) => {函数声明}
单一参数 => {函数声明}
```

没有参数的函数应该写成一对圆括号:

```bash
() => {函数声明}
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

<h2>JavaScript 箭头函数</h2>

<p>箭头函数不需要使用 function、return 关键字及大括号 {}。</p>

<p>IE11 及更早 IE 版本不支持箭头函数。</p>

<p id="demo"></p>

<script>
const x = (x, y) => x * y;
document.getElementById("demo").innerHTML = x(5, 5);
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/473db40930a24810a4126a7fc06374f1.png)
另一个方法：

```bash
const x = (x, y) => { return x * y };
```

有的箭头函数都没有自己的 this。 不适合定义一个 对象的方法。

当我们使用箭头函数的时候，箭头函数会默认帮我们绑定外层 this 的值，所以在箭头函数中 this 的值和外层的 this 是一样的。

箭头函数是不能提升的，所以需要在使用之前定义。

使用 const 比使用 var 更安全，因为函数表达式始终是一个常量。

如果函数部分只是一个语句，则可以省略 return 关键字和大括号 {}，这样做是一个比较好的习惯:

## 15. 函数闭包

```bash
function add() {
    var counter = 0;
    return counter += 1;
}
 
add();
add();
add();
 
// 本意是想输出 3, 但事与愿违，输出的都是 1 !
```

传统计数器

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p>全局变量计数。</p>
<button type="button" onclick="myFunction()">计数!</button>
<p id="demo">0</p>
<script>
var counter = 0;
function add() {
    return counter += 1;
}
function myFunction(){
    document.getElementById("demo").innerHTML = add();
}
</script>

</body>
</html>
```
输出
![在这里插入图片描述](https://img-blog.csdnimg.cn/589a53c3c85c4a54928a7366ee81d749.gif#pic_center)
闭包函数

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p>局部变量计数。</p>
<button type="button" onclick="myFunction()">计数!</button>
<p id="demo">0</p>
<script>
var add = (function () {
    var counter = 0;
    return function () {return counter += 1;}
})();
function myFunction(){
    document.getElementById("demo").innerHTML = add();
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/589a53c3c85c4a54928a7366ee81d749.gif#pic_center)
变量 add 指定了函数自我调用的返回字值。

自我调用函数只执行一次。设置计数器为 0。并返回函数表达式。

add变量可以作为一个函数使用。非常棒的部分是它可以访问函数上一层作用域的计数器。

这个叫作 JavaScript 闭包。它使得函数拥有私有变量变成可能。

计数器受匿名函数的作用域保护，只能通过 add 方法修改。
