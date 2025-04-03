# JavaScript 21. 调试


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d165b9ec088de08637688934191ede4d.png)


##  1. 简介
没有调试工具是很难去编写 JavaScript 程序的。

你的代码可能包含语法错误，逻辑错误，如果没有调试工具，这些错误比较难于发现。

通常，如果 JavaScript 出现错误，是不会有提示信息，这样你就无法找到代码错误的位置。

##   2. 调试工具
在程序代码中寻找错误叫做代码调试。

调试很难，但幸运的是，很多浏览器都内置了调试工具。

内置的调试工具可以开始或关闭，严重的错误信息会发送给用户。

有了调试工具，我们就可以设置断点 (代码停止执行的位置), 且可以在代码执行时检测变量。

浏览器启用调试工具一般是按下 `F12` 键，并在调试菜单中选择 "`Console`" 。

##  3. console.log() 方法
如果浏览器支持调试，你可以使用 console.log() 方法在调试窗口上打印 JavaScript 值：

```bash
a = 5;
b = 6;
c = a + b;
console.log(c);
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
	
<h1>我的第一个 Web 页面</h1>
<p>
浏览器中(Chrome, IE, Firefox) 使用 F12 来启用调试模式， 在调试窗口中点击 "Console" 菜单。
</p>
<script>
a = 5;
b = 6;
c = a + b;
console.log(c);
</script>
	
</body>
</html>
```
##  4. 设置断点
在调试窗口中，你可以设置 JavaScript 代码的断点。

在每个断点上，都会停止执行 JavaScript 代码，以便于我们检查 JavaScript 变量的值。

在检查完毕后，可以重新执行代码（如播放按钮）。

## 5. debugger 关键字
`debugger` 关键字用于停止执行 JavaScript，并调用调试函数。

这个关键字与在调试工具中设置断点的效果是一样的。

如果没有调试可用，debugger 语句将无法工作。

开启 debugger ，代码在第三行前停止执行。

```bash
var x = 15 * 5;
debugger;
document.getElementbyId("demo").innerHTML = x;
```
实例

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<head>
</head>

<body>
<p id="demo"></p>
<p>开启调试工具，在代码执行到第三行前会停止执行。</p>
<script>
var x = 15 * 5;
debugger;
document.getElementById("demo").innerHTML = x;
</script>

</body>
</html>
```
输出：

```bash
75

开启调试工具，在代码执行到第三行前会停止执行。
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9e8c63d676fbdfe21949da7561bfb7c5.png)
##  6. 浏览器的调试工具
通常，浏览器启用调试工具一般是按下 F12 键，并在调试菜单中选择 "Console" 。

各浏览器的步骤如下:

### 6.1 Chrome 浏览器

 - 打开浏览器。
 - 在菜单中选择 "更多工具"。
 - 在 "更多工具" 中选择 "开发者工具"。
 - 最后，选择 Console。

###  6.2 Firefox 浏览器

 - 打开浏览器。
 - 右击鼠标，选择 "查看元素"

###  6.3 Safari

 - 打开浏览器。
 - 右击鼠标，选择检查元素。
 - 在底部弹出的窗口中选择"控制台"。

###  6.4 Internet Explorer 浏览器。

 - 打开浏览器。
 - 在菜单中选择工具。
 - 在工具中选择开发者工具。
 - 最后，选择 Console。

###  6.5 Opera

 - 打开浏览器。
 - 点击左上角，选择"开发者工具",选择"WEB检查器"。

