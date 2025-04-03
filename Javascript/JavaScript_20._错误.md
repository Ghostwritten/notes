#  JavaScript 20. 错误


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f9862edbe56f5055c0b8a524a3eac98e.png)


##  1. 简介

 - `try` 语句测试代码块的错误。
 - `catch` 语句处理错误。
 - `throw` 语句创建自定义错误。
 - `finally` 语句在 try 和 catch 语句之后，无论是否有触发异常，该语句都会执行。

## 2. JavaScript 错误
当 JavaScript 引擎执行 JavaScript 代码时，会发生各种错误。

 - 可能是语法错误，通常是程序员造成的编码错误或错别字。
 - 可能是拼写错误或语言中缺少的功能（可能由于浏览器差异）。
 - 可能是由于来自服务器或用户的错误输出而导致的错误。

当然，也可能是由于许多其他不可预知的因素。

当错误发生时，当事情出问题时，JavaScript 引擎通常会停止，并生成一个错误消息。

描述这种情况的技术术语是：JavaScript 将抛出一个错误。

##  3. try 和 catch
try 语句允许我们定义在执行时进行错误测试的代码块。

catch 语句允许我们定义当 try 代码块发生错误时，所执行的代码块。

JavaScript 语句 try 和 catch 是成对出现的。
语法

```bash
try {
    ...    //异常的抛出
} catch(e) {
    ...    //异常的捕获与处理
} finally {
    ...    //结束处理
}
```
### 3.1 实例

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
<script>
var txt="";
function message(){
	try {
		adddlert("Welcome guest!");
	}
	catch(err) {
		txt="本页有一个错误。\n\n";
		txt+="错误描述：" + err.message + "\n\n";
		txt+="点击确定继续。\n\n";
		alert(txt);
	}
}
</script>
</head>
<body>

<input type="button" value="查看消息" onclick="message()" />

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c6bcb3f40bac6d203b1a129153386f1f.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/31edddcb2a2ef4ffd1a818172c51f8a7.png)
##  4. finally 语句
finally 语句不论之前的 try 和 catch 中是否产生异常都会执行该代码块。
### 4.1 实例

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>
<p>不管输入是否正确，输入框都会再输入后清空。</p>
<p>请输入 5 ~ 10 之间的数字：</p>

<input id="demo" type="text">
<button type="button" onclick="myFunction()">点我</button>

<p id="p01"></p>

<script>
function myFunction() {
  var message, x;
  message = document.getElementById("p01");
  message.innerHTML = "";
  x = document.getElementById("demo").value;
  try { 
    if(x == "") throw "值是空的";
    if(isNaN(x)) throw "值不是一个数字";
    x = Number(x);
    if(x > 10) throw "太大";
    if(x < 5) throw "太小";
  }
  catch(err) {
    message.innerHTML = "错误: " + err + ".";
  }
  finally {
    document.getElementById("demo").value = "";
  }
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b5f0f26c21919c06540c5286d0a5994d.gif#pic_center)

## 5. Throw 语句
throw 语句允许我们创建自定义错误。

正确的技术术语是：创建或抛出异常（exception）。

如果把 throw 与 try 和 catch 一起使用，那么您能够控制程序流，并生成自定义的错误消息。
语法

```bash
throw exception
```

异常可以是 JavaScript 字符串、数字、逻辑值或对象。

### 5.1 实例
本例检测输入变量的值。如果值是错误的，会抛出一个异常（错误）。catch 会捕捉到这个错误，并显示一段自定义的错误消息，**代码同上**：

```bash
function myFunction() {
    var message, x;
    message = document.getElementById("message");
    message.innerHTML = "";
    x = document.getElementById("demo").value;
    try { 
        if(x == "")  throw "值为空";
        if(isNaN(x)) throw "不是数字";
        x = Number(x);
        if(x < 5)    throw "太小";
        if(x > 10)   throw "太大";
    }
    catch(err) {
        message.innerHTML = "错误: " + err;
    }
}

```

