#  JavaScript 10. 对象


![在这里插入图片描述](https://img-blog.csdnimg.cn/3679aa6f76324a15851dc4a8ec97d5b9.png)



JavaScript 对象是拥有属性和方法的数据。
真实生活中的对象，属性和方法
真实生活中，一辆汽车是一个对象。

对象有它的属性，如重量和颜色等，方法有启动停止等:
![在这里插入图片描述](https://img-blog.csdnimg.cn/9064631831a04fc8840eeb0ceb170b60.png)
##  1. JavaScript 对象

```bash
var car = "Fiat";
```

对象也是一个变量，但对象可以包含多个值（多个变量），每个值以 name:value 对呈现。

```bash
var car = {name:"Fiat", model:500, color:"white"};
```

##  2. 对象定义
你可以使用字符来定义和创建 JavaScript 对象:

```bash
var person = {firstName:"John", lastName:"Doe", age:50, eyeColor:"blue"};
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
输出：

```bash
创建 JavaScript 对象。

John 现在 50 岁.
```
定义 JavaScript 对象可以跨越多行，空格跟换行不是必须的：

```bash
var person = {
    firstName:"John",
    lastName:"Doe",
    age:50,
    eyeColor:"blue"
};
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
输出：

```bash
创建 JavaScript 对象。

John 现在 50 岁。
```
##  3. 对象属性
可以说 "JavaScript 对象是变量的容器"。

但是，我们通常认为 "JavaScript 对象是键值对的容器"。

键值对通常写法为 name : value (键与值以冒号分割)。

键值对在 JavaScript 对象通常称为 对象属性。

你可以通过两种方式访问对象属性:
第一种：

```bash
<script>
var person = {
    firstName : "John",
    lastName : "Doe",
    id : 5566
};
document.getElementById("demo").innerHTML =
	person.firstName + " " + person.lastName;
</script>
```
第二种：

```bash
<script>
var person = {
    firstName: "John",
    lastName : "Doe",
    id : 5566
};
document.getElementById("demo").innerHTML =
	person["firstName"] + " " + person["lastName"];
</script>
```
## 4. 对象方法
对象的方法定义了一个函数，并作为对象的属性存储。

对象方法通过添加 () 调用 (作为一个函数)。

该实例访问了 person 对象的 `fullName()` 方法:

```bash
name = person.fullName();
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
输出：

```bash
创建和使用对象方法。

对象方法作为一个函数定义存储在对象属性中。

John Doe
```

如果你要访问 person 对象的 fullName 属性，它将作为一个定义函数的字符串返回：

```bash
name = person.fullName;
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
输出：

```bash
创建和使用对象方法。

对象方法是一个函数定义,并作为一个属性值存储。

不加括号输出函数表达式：function() { return this.firstName + " " + this.lastName; }

加括号输出函数执行结果：John Doe
```

