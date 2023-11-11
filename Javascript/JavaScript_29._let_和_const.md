#  JavaScript 29. let 和 const


![在这里插入图片描述](https://img-blog.csdnimg.cn/6030ac71dba140c191a689c76eb419c3.png)


##  前言
ECMAScript 2015(ECMAScript 6)
ES2015(ES6) 新增加了两个重要的 JavaScript 关键字: `let` 和 `const`。

 - let 声明的变量只在 let 命令所在的代码块内有效。
 - const 声明一个只读的常量，一旦声明，常量的值就不能改变。

在 ES6 之前，JavaScript 只有两种作用域： `全局变量` 与 `函数内的局部变量`。

## var 全局变量
全局变量在 JavaScript 程序的任何地方都可以访问。
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

```bash
全局变量在任何脚本和函数内均可访问。

我可以显示 Volvo
```

## var 局部变量

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
输出:

```bash
局部变量在声明的函数外不可以访问。

carName 的类型是：undefined
```
##  var 与 let 块级作用域(Block Scope)
使用 var 关键字声明的变量不具备块级作用域的特性，它在 `{}` 外依然能被访问到。

```bash
{ 
    var x = 2; 
}
// 这里可以使用 x 变量
```
在 ES6 之前，是没有块级作用域的概念的。

ES6 可以使用 let 关键字来实现块级作用域。

let 声明的变量只在 let 命令所在的代码块 {} 内有效，在 {} 之外不能访问。

```bash
{ 
    let x = 2;
}
// 这里不能使用 x 变量
```

## var 与 let 重新定义变量
使用 var 关键字重新声明变量可能会带来问题。

在块中重新声明变量也会重新声明块外的变量：

```bash
var x = 10;
// 这里输出 x 为 10
{ 
    var x = 2;
    // 这里输出 x 为 2
}
// 这里输出 x 为 2
```
let 关键字就可以解决这个问题，因为它只在 let 命令所在的代码块 {} 内有效。

```bash
var x = 10;
// 这里输出 x 为 10
{ 
    let x = 2;
    // 这里输出 x 为 2
}
// 这里输出 x 为 10
```
##  let 浏览器支持
Internet Explorer 11 及更早版本的浏览器不支持 let 关键字。

下表列出了各个浏览器支持 let 关键字的最低版本号。
![在这里插入图片描述](https://img-blog.csdnimg.cn/bd6f56cd7ab14b5ea874291e307b1403.png)
##  var 与 let 循环作用域
使用 var 关键字：

```bash
var i = 5;
for (var i = 0; i < 10; i++) {
    // 一些代码...
}
// 这里输出 i 为 10
```
使用 let 关键字：

```bash
let i = 5;
for (let i = 0; i < 10; i++) {
    // 一些代码...
}
// 这里输出 i 为 5
```
##  let 局部变量
在函数体内使用 var 和 let 关键字声明的变量有点类似。

它们的作用域都是 局部的:

```bash
// 使用 var
function myFunction() {
    var carName = "Volvo";   // 局部作用域
}

// 使用 let
function myFunction() {
    let carName = "Volvo";   //  局部作用域
}
```
##  let 全局变量
在函数体外或代码块外使用 var 和 let 关键字声明的变量也有点类似。

它们的作用域都是 全局的:

```bash
// 使用 var
var x = 2;       // 全局作用域

// 使用 let
let x = 2;       // 全局作用域
```
##  HTML 代码中使用全局变量
在 JavaScript 中, 全局作用域是针对 JavaScript 环境。

在 HTML 中, 全局作用域是针对 window 对象。

使用 var 关键字声明的全局作用域变量属于 window 对象:

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<h2>JavaScript 全局变量</h2>

<p>使用 var 关键字声明的全局作用域变量属于 window 对象。</p>

<p id="demo"></p>

<script>
var carName = "Volvo";

// 可以使用 window.carName 访问变量
document.getElementById("demo").innerHTML = "I can display " + window.carName;
</script>

</body>
</html>
```
输出：

```bash
JavaScript 全局变量
使用 var 关键字声明的全局作用域变量属于 window 对象。

I can display Volvo
```
使用 let 关键字声明的全局作用域变量不属于 window 对象:

```bash
let carName = "Volvo";
// 不能使用 window.carName 访问变量
```
## var 与 let 重置变量
使用 var 关键字声明的变量在任何地方都可以修改：

```bash
var x = 2;
 
// x 为 2
 
var x = 3;
 
// 现在 x 为 3
```
在相同的作用域或块级作用域中，不能使用 let 关键字来重置 var 关键字声明的变量:

```bash
var x = 2;       // 合法
let x = 3;       // 不合法

{
    var x = 4;   // 合法
    let x = 5   // 不合法
}
```
在相同的作用域或块级作用域中，不能使用 let 关键字来重置 let 关键字声明的变量:

```bash
let x = 2;       // 合法
let x = 3;       // 不合法

{
    let x = 4;   // 合法
    let x = 5;   // 不合法
}
```
在相同的作用域或块级作用域中，不能使用 var 关键字来重置 let 关键字声明的变量:

```bash
let x = 2;       // 合法
var x = 3;       // 不合法

{
    let x = 4;   // 合法
    var x = 5;   // 不合法
}
```
let 关键字在不同作用域，或不同块级作用域中是可以重新声明赋值的:

```bash
let x = 2;       // 合法

{
    let x = 3;   // 合法
}

{
    let x = 4;   // 合法
}
```
## var 变量提升
JavaScript 中，var 关键字定义的变量可以在使用后声明，也就是变量可以先使用再声明

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<h2>JavaScript 变量提升</h2>

<p>var 关键字定义的变量可以在使用后声明，也就是变量可以先使用再声明:</p>

<p id="demo"></p>

<script>
carName = "Volvo";
document.getElementById("demo").innerHTML = carName;
var carName;
</script>

</body>
</html>
```
输出

![在这里插入图片描述](https://img-blog.csdnimg.cn/d42e4660dda4416a8aa0ad70e2c34bbb.png)

##  const 关键字
const 用于声明一个或多个常量，声明时必须进行初始化，且初始化后值不可再修改：

```bash
const PI = 3.141592653589793;
PI = 3.14;      // 报错
PI = PI + 10;   // 报错
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

<h2>JavaScript const</h2>

<p>const 用于声明一个或多个常量，声明时必须进行初始化，且初始化后值不可再修改。</p>

<p id="demo"></p>

<script>
try {
    const PI = 3.141592653589793;
    PI = 3.14;
}
catch (err) {
    document.getElementById("demo").innerHTML = err;
}
</script>

</body>
</html>
```
输出：

```bash
JavaScript const
const 用于声明一个或多个常量，声明时必须进行初始化，且初始化后值不可再修改。

TypeError: Assignment to constant variable.
```
const定义常量与使用let 定义的变量相似：

 - 二者都是块级作用域
 - 都不能和它所在作用域内的其他变量或函数拥有相同的名称

两者还有以下两点区别：

 - const声明的常量必须初始化，而let声明的变量不用
 - const 定义常量的值不能通过再赋值修改，也不能再次声明。而 let 定义的变量值可以修改。

```bash
var x = 10;
// 这里输出 x 为 10
{ 
    const x = 2;
    // 这里输出 x 为 2
}
// 这里输出 x 为 10
```
const 声明的常量必须初始化：

```bash
// 错误写法
const PI;
PI = 3.14159265359;

// 正确写法
const PI = 3.14159265359;
```

##  并非真正的常量
const 的本质: const 定义的变量并非常量，并非不可变，它定义了一个常量引用一个值。使用 const 定义的对象或者数组，其实是可变的。下面的代码并不会报错

```bash
// 创建常量对象
const car = {type:"Fiat", model:"500", color:"white"};
 
// 修改属性:
car.color = "red";
 
// 添加属性
car.owner = "Johnson";
```
但是我们不能对常量对象重新赋值：

```bash
const car = {type:"Fiat", model:"500", color:"white"};
car = {type:"Volvo", model:"EX60", color:"red"};    // 错误
```
以下实例修改常量数组：

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<h2>JavaScript const</h2>

<p>以下实例修改常量数组:</p>

<p id="demo"></p>

<script>
// 创建常量数组
const cars = ["Saab", "Volvo", "BMW"];
 
// 修改元素
cars[0] = "Toyota";
 
// 添加元素
cars.push("Audi");

// 显示数组
document.getElementById("demo").innerHTML = cars; 
</script>

</body>
</html>
```
输出：

```bash
JavaScript const
以下实例修改常量数组:

Toyota,Volvo,BMW,Audi
```
但是我们不能对常量数组重新赋值：

```bash
const cars = ["Saab", "Volvo", "BMW"];
cars = ["Toyota", "Volvo", "Audi"];    // 错误
```
## const 浏览器支持
Internet Explorer 10 及更早版本的浏览器不支持 const 关键字。

下表列出了各个浏览器支持 const 关键字的最低版本号。
![在这里插入图片描述](https://img-blog.csdnimg.cn/3a6a8e3016ee412291d6a7ae59cad143.png)
## var、let、const 重置变量
使用 var 关键字声明的变量在任何地方都可以修改：

```bash
var x = 2;    //  合法
var x = 3;    //  合法
x = 4;        //  合法
```
在相同的作用域或块级作用域中，不能使用 `const` 关键字来重置 `var` 和 `let`关键字声明的变量:

```bash
var x = 2;         // 合法
const x = 2;       // 不合法
{
    let x = 2;     // 合法
    const x = 2;   // 不合法
}
```
在相同的作用域或块级作用域中，不能使用 const 关键字来重置 const 关键字声明的变量:

```bash
const x = 2;       // 合法
const x = 3;       // 不合法
x = 3;             // 不合法
var x = 3;         // 不合法
let x = 3;         // 不合法

{
    const x = 2;   // 合法
    const x = 3;   // 不合法
    x = 3;         // 不合法
    var x = 3;     // 不合法
    let x = 3;     // 不合法
}
```
const 关键字在不同作用域，或不同块级作用域中是可以重新声明赋值的:

```bash
const x = 2;       // 合法

{
    const x = 3;   // 合法
}

{
    const x = 4;   // 合法
}
```
##  const 冻结
文中说到 const 定义的变量并非不可改变，比如使用const声明对象，可以改变对象值。

那么什么情况能彻底“锁死”变量呢？

可以使用Object.freeze()方法来 冻结变量 ，如：

```bash
const obj = {
  name:"1024kb"
}
Object.freeze(obj)
// 此时对象obj被冻结，返回被冻结的对象
```
需要注意的是，被冻结后的对象不仅仅是不能修改值，同时也

 - 不能向这个对象添加新的属性
 - 不能修改其已有属性的值
 - 不能删除已有属性
 - 不能修改该对象已有属性的可枚举性、可配置性、可写性

## 总结
使用var关键字声明的全局作用域变量属于window对象。

使用let关键字声明的全局作用域变量不属于window对象。

使用var关键字声明的变量在任何地方都可以修改。

在相同的作用域或块级作用域中，不能使用let关键字来重置var关键字声明的变量。

在相同的作用域或块级作用域中，不能使用let关键字来重置let关键字声明的变量。

let关键字在不同作用域，或不用块级作用域中是可以重新声明赋值的。

在相同的作用域或块级作用域中，不能使用const关键字来重置var和let关键字声明的变量。

在相同的作用域或块级作用域中，不能使用const关键字来重置const关键字声明的变量

const 关键字在不同作用域，或不同块级作用域中是可以重新声明赋值的:

var关键字定义的变量可以先使用后声明。

let关键字定义的变量需要先声明再使用。

const关键字定义的常量，声明时必须进行初始化，且初始化后不可再修改。
