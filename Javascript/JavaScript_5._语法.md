

--------------
##  JavaScript 字面量
数字（Number）字面量 可以是整数或者是小数，或者是科学计数(e)。
字符串（String）字面量 可以使用单引号或双引号
表达式字面量 用于计算：
```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>
	
<p id="demo1"></p>
<p id="demo2"></p>
<p id="demo3"></p>
<script>
document.getElementById("demo1").innerHTML = 'John Doe';
document.getElementById("demo2").innerHTML = 123e5;
document.getElementById("demo3").innerHTML = 5 * 10;
</script>
	
</body>
</html>
```
输出

```bash
John Doe

12300000

50
```

数组（Array）字面量 定义一个数组：

```bash
[40, 100, 1, 5, 25, 10]
```

对象（Object）字面量 定义一个对象：

```bash
{firstName:"John", lastName:"Doe", age:50, eyeColor:"blue"}
```

函数（Function）字面量 定义一个函数：

```bash
function myFunction(a, b) { return a * b;}
```

##  JavaScript 变量

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
var length;
length = 6;
document.getElementById("demo").innerHTML = length;
</script>

</body>
</html>
```
输出；
6

##  JavaScript 操作符
JavaScript使用 算术运算符 来计算值:
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
document.getElementById("demo").innerHTML = (5 + 6) * 10;
</script>

</body>
</html>
```
输出
110

JavaScript使用赋值运算符给变量赋值：

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
var x, y, z;
x = 5;
y = 6;
z = (x + y) * 10;
document.getElementById("demo").innerHTML = z;
</script>

</body>
</html>
```
输出
110

JavaScript语言有多种类型的运算符：

类型	实例	描述

 - 赋值，算术和位运算符	=  +  -  *  /	在 JS 运算符中描述
 - 条件，比较及逻辑运算符	==  != <  > 	在 JS 比较运算符中描述

##  JavaScript 语句
在 HTML 中，JavaScript 语句向浏览器发出的命令。

语句是用分号分隔：

```bash
x = 5 + 6;
y = x * 10;
```
##  JavaScript 关键字

JavaScript 关键字用于标识要执行的操作。

和其他任何编程语言一样，JavaScript 保留了一些关键字为自己所用。

var 关键字告诉浏览器创建一个新的变量：

```bash
var x = 5 + 6;
var y = x * 10;
```
JavaScript 同样保留了一些关键字，这些关键字在当前的语言版本中并没有使用，但在以后 JavaScript 扩展中会用到。

以下是 JavaScript 中最​​重要的保留字（按字母顺序）：
| abstract | else       | instanceof | super        |
|----------|------------|------------|--------------|
| boolean  | enum       | int        | switch       |
| break    | export     | interface  | synchronized |
| byte     | extends    | let        | this         |
| case     | false      | long       | throw        |
| catch    | final      | native     | throws       |
| char     | finally    | new        | transient    |
| class    | float      | null       | true         |
| const    | for        | package    | try          |
| continue | function   | private    | typeof       |
| debugger | goto       | protected  | var          |
| default  | if         | public     | void         |
| delete   | implements | return     | volatile     |
| do       | import     | short      | while        |
| double   | in         | static     | with         |


##  JavaScript 注释
不是所有的 JavaScript 语句都是"命令"。双斜杠 // 后的内容将会被浏览器忽略：

```bash
// 我不会执行
```
## JavaScript 数据类型
JavaScript 有多种数据类型：数字，字符串，数组，对象等等：

```bash
var length = 16;                                  // Number 通过数字字面量赋值
var points = x * 10;                              // Number 通过表达式字面量赋值
var lastName = "Johnson";                         // String 通过字符串字面量赋值
var cars = ["Saab", "Volvo", "BMW"];              // Array  通过数组字面量赋值
var person = {firstName:"John", lastName:"Doe"};  // Object 通过对象字面量赋值
```
##  JavaScript 函数
JavaScript 语句可以写在函数内，函数可以重复引用：

引用一个函数 = 调用函数(执行函数内的语句)。

```bash
function myFunction(a, b) {
    return a * b;                                // 返回 a 乘以 b 的结果
}
```
##  JavaScript 字母大小写
JavaScript 对大小写是敏感的。

当编写 JavaScript 语句时，请留意是否关闭大小写切换键。

函数 getElementById 与 getElementbyID 是不同的。

同样，变量 myVariable 与 MyVariable 也是不同的。
