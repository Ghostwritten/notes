#  JavaScript 类型转换


![在这里插入图片描述](https://img-blog.csdnimg.cn/84ffd79ba8394eb482f91a964144086c.png)



`Number()` 转换为数字， `String()` 转换为字符串， `Boolean()` 转换为布尔值。

##  1. JavaScript 数据类型
在 JavaScript 中有 6 种不同的数据类型：

 - string
 - number
 - boolean
 - object
 - function
 - symbol

3 种对象类型：

 - Object
 - Date
 - Array

2 个不包含任何值的数据类型：

 - null
 - undefined

##  2. typeof 操作符

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p> typeof 操作符返回变量、对象、函数、表达式的类型。</p>
<p id="demo"></p>
<script>
document.getElementById("demo").innerHTML = 
    typeof "john" + "<br>" +
    typeof 3.14 + "<br>" +
    typeof NaN + "<br>" +
    typeof false + "<br>" +
    typeof [1,2,3,4] + "<br>" +
    typeof {name:'john', age:34} + "<br>" +
    typeof new Date() + "<br>" +
    typeof function () {} + "<br>" +
    typeof myCar + "<br>" +
    typeof null;
</script>

</body>
</html>
```
输出：

```bash
typeof 操作符返回变量、对象、函数、表达式的类型。

string
number
number
boolean
object
object
object
function
undefined
object
```
请注意：

 - NaN 的数据类型是 `number`
 - 数组(Array)的数据类型是 `object`
 - 日期(Date)的数据类型为 `object`
 - null 的数据类型是 `object`
 - 未定义变量的数据类型为 `undefined`

如果对象是 `JavaScript Array` 或 `JavaScript Date` ，我们就无法通过 typeof 来判断他们的类型，因为都是 返回 object。

## 3. constructor 属性
`constructor` 属性返回所有 `JavaScript` 变量的构造函数。

```bash
"John".constructor                 // 返回函数 String()  { [native code] }
(3.14).constructor                 // 返回函数 Number()  { [native code] }
false.constructor                  // 返回函数 Boolean() { [native code] }
[1,2,3,4].constructor              // 返回函数 Array()   { [native code] }
{name:'John', age:34}.constructor  // 返回函数 Object()  { [native code] }
new Date().constructor             // 返回函数 Date()    { [native code] }
function () {}.constructor         // 返回函数 Function(){ [native code] }
```
###  3.1 实例1

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p> constructor 属性返回变量或对象的构造函数。</p>
<p id="demo"></p>
<script>
document.getElementById("demo").innerHTML = 
    "john".constructor + "<br>" +
    (3.14).constructor + "<br>" +
    false.constructor + "<br>" +
    [1,2,3,4].constructor + "<br>" +
    {name:'john', age:34}.constructor + "<br>" +
    new Date().constructor + "<br>" +
    function () {}.constructor;
</script>

</body>
</html>
```
输出：

```bash
constructor 属性返回变量或对象的构造函数。

function String() { [native code] }
function Number() { [native code] }
function Boolean() { [native code] }
function Array() { [native code] }
function Object() { [native code] }
function Date() { [native code] }
function Function() { [native code] }
```
### 3.2 实例2
可以使用 constructor 属性来查看对象是否为数组 (包含字符串 "Array"):

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p>判断是否为数组。</p>
<p id="demo"></p>
<script>
var fruits = ["Banana", "Orange", "Apple", "Mango"];
document.getElementById("demo").innerHTML = isArray(fruits);
function isArray(myArray) {
    return myArray.constructor.toString().indexOf("Array") > -1;
}
</script>

</body>
</html>
```
输出：

```bash
判断是否为数组。

true
```
###  3.3 实例3
你可以使用 constructor 属性来查看对象是否为日期 (包含字符串 "Date"):

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p>判断是否为日期。</p>
<p id="demo"></p>
<script>
var myDate = new Date();
document.getElementById("demo").innerHTML = isDate(myDate);
function isDate(myDate) {
    return myDate.constructor.toString().indexOf("Date")> -1 ;
}
</script>

</body>
</html>
```
输出：

```bash
判断是否为日期。

true
```
##  4. JavaScript 类型转换
JavaScript 变量可以转换为新变量或其他数据类型：

 - 通过使用 JavaScript 函数
 - 通过 JavaScript 自身自动转换

###  4.1 将数字转换为字符串
全局方法 `String()` 可以将数字转换为字符串。

该方法可用于任何类型的数字，字母，变量，表达式：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p> String() 方法可以将数字转换为字符串。</p>
<p id="demo"></p>
<script>
var x = 123;
document.getElementById("demo").innerHTML =
    String(x) + "<br>" +
    String(123) + "<br>" +
    String(100 + 23);
</script>
</body>
</html>
```
输出：

```bash
String() 方法可以将数字转换为字符串。

123
123
123
```
Number 方法 `toString()` 也是有同样的效果。

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p>toString() 方法将数字转换为字符串。</p>
<p id="demo"></p>
<script>
var x = 123;
document.getElementById("demo").innerHTML =
    x.toString() + "<br>" +
   (123).toString() + "<br>" +
   (100 + 23).toString();
</script>

</body>
</html>
```
输出：

```bash
toString() 方法将数字转换为字符串。

123
123
123
```
你可以找到更多数字转换为字符串的方法：

|方法	|描述|
|--|--|
|toExponential()	|把对象的值转换为指数计数法。|
|toFixed()	|把数字转换为字符串，结果的小数点后有指定位数的数字。|
|toPrecision()	|把数字格式化为指定的长度。|

###  4.2 将布尔值转换为字符串
全局方法 `String()` 可以将布尔值转换为字符串。

```bash
String(false)        // 返回 "false"
String(true)         // 返回 "true"
```
Boolean 方法 toString() 也有相同的效果。

```bash
false.toString()     // 返回 "false"
true.toString()      // 返回 "true"
```
###  4.3 将日期转换为字符串
Date() 返回字符串。

```bash
Date()      // 返回 Thu Jul 17 2014 15:38:19 GMT+0200 (W. Europe Daylight Time)
```

全局方法 `String()` 可以将日期对象转换为字符串。

```bash
String(new Date())      // 返回 Thu Jul 17 2014 15:38:19 GMT+0200 (W. Europe Daylight Time)
```

Date 方法 toString() 也有相同的效果。

实例

```bash
obj = new Date()
obj.toString()   // 返回 Thu Jul 17 2014 15:38:19 GMT+0200 (W. Europe Daylight Time)
```
在 Date 方法 章节中，你可以查看更多关于日期转换为字符串的函数：
|方法	|描述|
|--|--|
|getDate()|	从 Date 对象返回一个月中的某一天 (1 ~ 31)。|
|getDay()|	从 Date 对象返回一周中的某一天 (0 ~ 6)。|
|getFullYear()|	从 Date 对象以四位数字返回年份。|
|getHours()	|返回 Date 对象的小时 (0 ~ 23)。|
|getMilliseconds()|	返回 Date 对象的毫秒(0 ~ 999)。|
|getMinutes()	|返回 Date 对象的分钟 (0 ~ 59)。|
|getMonth()	|从 Date 对象返回月份 (0 ~ 11)。|
|getSeconds()	|返回 Date 对象的秒数 (0 ~ 59)。|
|getTime()	|返回 1970 年 1 月 1 日至今的毫秒数。|

##  5. 将字符串转换为数字
全局方法 Number() 可以将字符串转换为数字。

字符串包含数字(如 "3.14") 转换为数字 (如 3.14).

空字符串转换为 0。

其他的字符串会转换为 NaN (不是个数字)。

```bash
Number("3.14")    // 返回 3.14
Number(" ")       // 返回 0
Number("")        // 返回 0
Number("99 88")   // 返回 NaN
```
|方法	|描述|
|--|--|
|parseFloat()	|解析一个字符串，并返回一个浮点数。|
|parseInt()	|解析一个字符串，并返回一个整数。|

##  6. 一元运算符 +
Operator + 可用于将变量转换为数字：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p> typeof 操作符返回变量或表达式的类型。</p>
<button onclick="myFunction()">点我</button>
<p id="demo"></p>
<script>
function myFunction() {
    var y = "5";
    var x = + y;
    document.getElementById("demo").innerHTML =
		typeof y + "<br>" + typeof x;
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/38d6ad30cb264cbcb8cc3e1ed8be8efc.png)
如果变量不能转换，它仍然会是一个数字，但值为 NaN (不是一个数字):

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p> typeof 操作符返回变量或表达式的类型。</p>
<button onclick="myFunction()">点我</button>
<p id="demo"></p>
<script>
function myFunction() {
    var y = "John";
    var x = + y;
    document.getElementById("demo").innerHTML =
		typeof x + "<br>" + x;
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/aaa28bb141924f30aab4ee1ddd68da15.png)
## 7.  将布尔值转换为数字
全局方法 Number() 可将日期转换为数字。

```bash
d = new Date();
Number(d)          // 返回 1404568027739
```

日期方法 getTime() 也有相同的效果。

```bash
d = new Date();
d.getTime()        // 返回 1404568027739
```
## 8. 自动转换类型
当 JavaScript 尝试操作一个 "错误" 的数据类型时，会自动转换为 "正确" 的数据类型。

以下输出结果不是你所期望的：

```bash
5 + null    // 返回 5         null 转换为 0
"5" + null  // 返回"5null"   null 转换为 "null"
"5" + 1     // 返回 "51"      1 转换为 "1" 
"5" - 1     // 返回 4         "5" 转换为 5
```
## 9. 自动转换为字符串
当你尝试输出一个对象或一个变量时 JavaScript 会自动调用变量的 toString() 方法：

```bash
document.getElementById("demo").innerHTML = myVar;

myVar = {name:"Fjohn"}  // toString 转换为 "[object Object]"
myVar = [1,2,3,4]       // toString 转换为 "1,2,3,4"
myVar = new Date()      // toString 转换为 "Fri Jul 18 2014 09:08:55 GMT+0200"
```

数字和布尔值也经常相互转换：

```bash
myVar = 123             // toString 转换为 "123"
myVar = true            // toString 转换为 "true"
myVar = false           // toString 转换为 "false"
```

下表展示了使用不同的数值转换为数字(Number), 字符串(String), 布尔值(Boolean):
