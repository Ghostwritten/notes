#  javascript 数组

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/da3c903c0d15df8725f8c291d2aca38a.png)




##  1. 简介
数组对象是使用单独的变量名来存储一系列的值。

如果你有一组数据（例如：车名字），存在单独变量如下所示：

```bash
var car1="Saab";
var car2="Volvo";
var car3="BMW";
```

然而，如果你想从中找出某一辆车？并且不是3辆，而是300辆呢？这将不是一件容易的事！

最好的方法就是用数组。

数组可以用一个变量名存储所有的值，并且可以用变量名访问任何一个值。

数组中的每个元素都有自己的的ID，以便它可以很容易地被访问到。

实例

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<script>
var i;
var mycars = new Array();
mycars[0] = "Saab";
mycars[1] = "Volvo";
mycars[2] = "BMW";
for (i=0;i<mycars.length;i++){
	document.write(mycars[i] + "<br />");
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

##  2. 创建数组
创建一个数组，有三种方法。

下面的代码定义了一个名为 myCars的数组对象：

1: 常规方式:

```bash
var myCars=new Array();
myCars[0]="Saab";      
myCars[1]="Volvo";
myCars[2]="BMW";
```

2: 简洁方式:

```bash
var myCars=new Array("Saab","Volvo","BMW");
```

3: 字面:

```bash
var myCars=["Saab","Volvo","BMW"];
```
## 3. 访问数组
通过指定数组名以及索引号码，你可以访问某个特定的元素。	`[0]` 是数组的第一个元素。`[1]` 是数组的第二个元素。

以下实例可以访问myCars数组的第一个值：

```bash
var name=myCars[0];
```

以下实例修改了数组 myCars 的第一个元素:

```bash
myCars[0]="Opel";
```
所有的JavaScript变量都是对象。数组元素是对象。函数是对象。

因此，你可以在数组中有不同的变量类型。

你可以在一个数组中包含对象元素、函数、数组：

```bash
myArray[0]=Date.now;
myArray[1]=myFunction;
myArray[2]=myCars;
```
## 4. 数组方法和属性
使用数组对象预定义属性和方法：

```bash
var x=myCars.length             // myCars 中元素的数量
var y=myCars.indexOf("Volvo")   // "Volvo" 值的索引值
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/512e7f36bf38c3d84b88aae6d43abbd7.jpeg#pic_center)

## 5. 创建新方法

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p id="demo">单击按钮创建一个数组,调用 ucase（）方法, 并显示结果。</p>
<button onclick="myFunction()">点我</button>
<script>
Array.prototype.myUcase=function(){
	for (i=0;i<this.length;i++){
		this[i]=this[i].toUpperCase();
	}
}
function myFunction(){
	var fruits = ["Banana", "Orange", "Apple", "Mango"];
	fruits.myUcase();
	var x=document.getElementById("demo");
	x.innerHTML=fruits;
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a1246faf92dfae5cf7d0f65b7fe29401.gif#pic_center)

## 6. 实例
###  6.1 合并两个数组 - concat()

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>
	
<script>
var hege = ["Cecilie", "Lone"];
var stale = ["Emil", "Tobias", "Linus"];
var children = hege.concat(stale);
document.write(children);
</script>
	
</body>
</html>
```
输出：

```bash
Cecilie,Lone,Emil,Tobias,Linus
```
### 6.2 合并三个数组 - concat()

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<script>
var parents = ["Jani", "Tove"];
var brothers = ["Stale", "Kai Jim", "Borge"];
var children = ["Cecilie", "Lone"];
var family = parents.concat(brothers, children);
document.write(family);
</script>

</body>
</html>
```
输出：

```bash
Jani,Tove,Stale,Kai Jim,Borge,Cecilie,Lone
```
### 6.3 用数组的元素组成字符串 - join()

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p id="demo">点击按钮将数组作为字符串输出。</p>
<button onclick="myFunction()">点我</button>
<script>
function myFunction(){
	var fruits = ["Banana", "Orange", "Apple", "Mango"];
	var x=document.getElementById("demo");
	x.innerHTML=fruits.join();
}
</script>

</body>
</html>
```
输出：

```bash
Banana,Orange,Apple,Mango
```
### 6.4 删除数组的最后一个元素 - pop()

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p id="demo">单击按钮删除数组的最后一个元素。</p>
<button onclick="myFunction()">点我</button>
<script>
var fruits = ["Banana", "Orange", "Apple", "Mango"];
function myFunction(){
	fruits.pop();
	var x=document.getElementById("demo");
	x.innerHTML=fruits;
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/089e3a43013876f5e3b0b89f1c666a32.gif#pic_center)
### 6.5 数组的末尾添加新的元素 - push()

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p id="demo">单击按钮给数组添加新的元素。</p>
<button onclick="myFunction()">点我</button>
<script>
var fruits = ["Banana", "Orange", "Apple", "Mango"];
function myFunction(){
	fruits.push("Kiwi")
	var x=document.getElementById("demo");
	x.innerHTML=fruits;
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0923acaaeb37738f07c65ced541da4a7.gif#pic_center)
### 6.6 将一个数组中的元素的顺序反转排序 - reverse()

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p id="demo">单击按钮将数组反转排序。</p>
<button onclick="myFunction()">点我</button>
<script>
var fruits = ["Banana", "Orange", "Apple", "Mango"];
function myFunction(){
	fruits.reverse();
	var x=document.getElementById("demo");
	x.innerHTML=fruits;
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/81c95f882601efa435c775b77d19f31d.gif#pic_center)
### 6.7 删除数组的第一个元素 - shift()

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p id="demo">单击按钮删除数组的第一个元素。</p>
<p id="demo2"></p>
<button onclick="myFunction()">点我</button>
<script>
var fruits = ["Banana", "Orange", "Apple", "Mango"];
function myFunction(){
	var delell = fruits.shift();
	var x=document.getElementById("demo");
	x.innerHTML= '删除后数组为：' +  fruits;
	document.getElementById("demo2").innerHTML= '删除的元素是：' +  delell;
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7205c1cc4abd094665ea641b88b3cb45.gif#pic_center)
### 6.8 从一个数组中选择元素 - slice()

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p id="demo">点击按钮截取数组下标 1 到 2 的元素。</p>
<button onclick="myFunction()">点我</button>
<script>
function myFunction(){
	var fruits = ["Banana", "Orange", "Lemon", "Apple", "Mango"];
	var citrus = fruits.slice(1,3);
	var x=document.getElementById("demo");
	x.innerHTML=citrus;
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c3c1dc82b16a4a5bc54ee273205cc15d.gif#pic_center)
### 6.9 数组排序（按字母顺序升序）- sort()

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p id="demo">单击按钮升序排列数组。</p>
<button onclick="myFunction()">点我</button>
<script>
function myFunction(){
	var fruits = ["Banana", "Orange", "Apple", "Mango"];
	fruits.sort();
	var x=document.getElementById("demo");
	x.innerHTML=fruits;
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a1c061b85a80c16579a2298aa7e924c6.gif#pic_center)

### 6.10 数字排序（按数字顺序升序）- sort()

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p id="demo">单击按钮升序排列数组。</p>
<button onclick="myFunction()">点我</button>
<script>
function myFunction(){
	var points = [40,100,1,5,25,10];
	points.sort(function(a,b){return a-b});
	var x=document.getElementById("demo");
	x.innerHTML=points;
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9f745ac61face168faa9b25267ef2ebd.gif#pic_center)
### 6.11 数字排序（按数字顺序降序）- sort()

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p id="demo">单击按钮降序排列数组。</p>
<button onclick="myFunction()">点我</button>
<script>
function myFunction(){
	var points = [40,100,1,5,25,10];
	points.sort(function(a,b){return b-a});
	var x=document.getElementById("demo");
	x.innerHTML=points;
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2cac61a3f2f4d87fb0b55402793f2af9.gif#pic_center)

### 6.12 在数组的第2位置添加一个元素 - splice()
```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p id="demo">点击按钮向数组添加元素。</p>
<button onclick="myFunction()">点我</button>
<script>
function myFunction(){
	var fruits = ["Banana", "Orange", "Apple", "Mango"];
	fruits.splice(2,0,"Lemon","Kiwi");
	var x=document.getElementById("demo");
	x.innerHTML=fruits;
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/161679d212b3df1fd9457e871aceaa7f.gif#pic_center)

### 6.13 转换数组到字符串 -toString()

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p id="demo">点击按钮将数组转为字符串并返回。</p>
<button onclick="myFunction()">点我</button>
<script>
function myFunction(){
	var fruits = ["Banana", "Orange", "Apple", "Mango"];
	var str = fruits.toString();
	var x=document.getElementById("demo");
	x.innerHTML= str;
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dc9c8986fbcf13d5b4f910bb105dde69.gif#pic_center)
### 6.14 在数组的开头添加新元素 - unshift()

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p id="demo">单击按钮在数组中插入元素。</p>
<button onclick="myFunction()">点我</button>
<script>
function myFunction(){
	var fruits = ["Banana", "Orange", "Apple", "Mango"];
	fruits.unshift("Lemon","Pineapple");
	var x=document.getElementById("demo");
	x.innerHTML=fruits;
}
</script>
<p><b>注意:</b> unshift()方法不能用于 Internet Explorer 8 之前的版本，插入的值将被返回成<em> undefined </em>。</p>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/61eaabf92dc92a2ff49a834c55822b0c.gif#pic_center)

