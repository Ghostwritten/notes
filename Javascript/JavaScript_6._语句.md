

---
##  1. JavaScript 语句
JavaScript 语句是发给浏览器的命令。这些命令的作用是告诉浏览器要做的事情。下面的 JavaScript 语句向 id="demo" 的 HTML 元素输出文本 "你好 Dolly" ：

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<h1>我的网页</h1>
<p id="demo">我的第一个段落。</p>
<script>
document.getElementById("demo").innerHTML = "你好 Dolly";
</script>

</body>
</html>
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/c92933d67d874029a5a1e5dc847a5098.png)
##  2. 分号 ;
分号用于分隔 JavaScript 语句。
通常我们在每条可执行的语句结尾添加分号。
使用分号的另一用处是在一行中编写多条语句。

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<h1>我的网页</h1>
<p id="demo1"></p>
<p id="demo2"></p>
<script>
a = 1;
b = 2;
c = a + b;
document.getElementById("demo1").innerHTML = c;
x = 1; y = 2; z = x + y;
document.getElementById("demo2").innerHTML = z;
</script>

</body>
</html>
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/22ec8bfa5e8f47d5b0213d4cb26b7bef.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_9,color_FFFFFF,t_70,g_se,x_16)
##  3. JavaScript 代码
JavaScript 代码是 JavaScript 语句的序列，浏览器按照编写顺序依次执行每条语句。本例向网页输出一个标题和两个段落：

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<h1>我的 Web 页面</h1>
<p id="demo">一个段落。</p>
<div id="myDIV">一个 DIV。</div>
<script>
document.getElementById("demo").innerHTML="你好 Dolly";
document.getElementById("myDIV").innerHTML="你最近怎么样?";
</script>

</body>
</html>
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/64aaa7ac9d4142d09d08bb15a5e012d3.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_12,color_FFFFFF,t_70,g_se,x_16)
##  4. JavaScript 代码块
JavaScript 可以分批地组合起来。代码块以左花括号开始，以右花括号结束。代码块的作用是一并地执行语句序列。本例向网页输出一个标题和两个段落：

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<h1>我的 Web 页面</h1>
<p id="myPar">我是一个段落。</p>
<div id="myDiv">我是一个div。</div>
<p>
<button type="button" onclick="myFunction()">点击这里</button>
</p>
<script>
function myFunction(){
	document.getElementById("myPar").innerHTML="你好世界！";
	document.getElementById("myDiv").innerHTML="你最近怎么样?";
}
</script>
<p>当您点击上面的按钮时，两个元素会改变。</p>

</body>
</html>
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/f8499c11a8f54291beb67535b7e86cb3.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_14,color_FFFFFF,t_70,g_se,x_16)
点击后
![在这里插入图片描述](https://img-blog.csdnimg.cn/943f51be5e5948e997b6bac56a520e26.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_15,color_FFFFFF,t_70,g_se,x_16)

