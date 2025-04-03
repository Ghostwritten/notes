#  JavaScript 17. typeof


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dce7cd9bcb2494fffa3674bf8dfae46e.png)


##  1. typeof 操作符

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
typeof 操作符返回变量或表达式的类型。

```bash
string
number
boolean
object
object
```
## 2. null
null：主动释放一个变量引用的对象，表示一个变量不再指向任何对象地址。
何时使用null?当使用完一个比较大的对象时，需要对其进行释放内存时，设置为 null。

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<p>对象可以通过设置为 <b>null</b> 来清空。</p>
<p id="demo"></p>
<script>
var person = {firstName:"John", lastName:"Doe", age:50, eyeColor:"blue"};
var person = null;
document.getElementById("demo").innerHTML = typeof person;
</script>

</body>
</html>
```
输出：

```bash
对象可以通过设置为 null 来清空。

object
```
## 3. undefined
undefined：是所有没有赋值变量的默认值，自动赋值。

```bash
var person;                  // 值为 undefined(空), 类型是undefined
person = undefined;          // 值为 undefined, 类型是undefined
```

## 4. undefined 和 null 的区别
共同点：都是原始类型，保存在栈中变量本地。

不同点：

（1）undefined——表示变量声明过但并未赋过值。

它是所有未赋值变量默认值，例如：

```bash
var a;    // a 自动被赋值为 undefined
```

（2）null——表示一个变量将来可能指向一个对象。

一般用于主动释放指向对象的引用，例如：

```bash
var emps = ['ss','nn'];
emps = null;     // 释放指向数组的引用
```

4、延伸——垃圾回收站

它是专门释放对象内存的一个程序。

 - （1）在底层，后台伴随当前程序同时运行；引擎会定时自动调用垃圾回收期；
 - （2）总有一个对象不再被任何变量引用时，才释放

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
document.getElementById("demo").innerHTML =
	typeof undefined + "<br>" +
	typeof null + "<br>" +
	(null === undefined) + "<br>" +
	(null == undefined);
</script>

</body>
</html>
```
输出：

```bash
undefined
object
false
true
```

