

-----------

JavaScript 没有任何打印或者输出的函数。

##  1. JavaScript 显示数据

JavaScript 可以通过不同的方式来输出数据：

 - 使用 `window.alert()` 弹出警告框。
 - 使用 `document.write()` 方法将内容写到 HTML 文档中。
 - 使用 `innerHTML` 写入到 HTML 元素。
 - 使用 `console.log()` 写入到浏览器的控制台。

##  2. 使用 window.alert()
你可以弹出警告框来显示数据：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>
	
<h1>我的第一个页面</h1>
<p>我的第一个段落。</p>
<script>
window.alert(5 + 6);
</script>
	
</body>
</html>
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/c0a06a27ff784e2296a07d2c846f8ea8.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_19,color_FFFFFF,t_70,g_se,x_16)
##  3. 操作 HTML 元素
如需从 JavaScript 访问某个 HTML 元素，您可以使用 `document.getElementById(id)` 方法。

请使用 "id" 属性来标识 HTML 元素，并 innerHTML 来获取或插入元素内容：

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>
	
<h1>我的第一个 Web 页面</h1>
<p id="demo">我的第一个段落。</p>
<script>
document.getElementById("demo").innerHTML="段落已修改。";
</script>
	
</body>
</html>
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/59389a2afff44bde9f75d3804eb37d8e.png)
以上 JavaScript 语句（在 <script> 标签中）可以在 web 浏览器中执行：

`document.getElementById("demo")` 是使用 id 属性来查找 HTML 元素的 JavaScript 代码 。

`innerHTML = "段落已修改。"` 是用于修改元素的 HTML 内容(innerHTML)的 JavaScript 代码

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>
	
<h1>我的第一个 Web 页面</h1>
<p>我的第一个段落。</p>
<script>
document.write(Date());
</script>
	
</body>
</html>
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/d241b97b760f4921a1f7db145d6bcff3.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>
	
<h1>我的第一个 Web 页面</h1>
<p>我的第一个段落。</p>
<button onclick="myFunction()">点我</button>
<script>
function myFunction() 
{
    document.write(Date());
}
</script>
	
</body>
</html>
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/3f488929d2cc4c4bbc931d10dfcd1ae8.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_16,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/8d15b75557eb415ebba17ef91c587e16.png)
##  4. 写到控制台
如果您的浏览器支持调试，你可以使用 `console.log()` 方法在浏览器中显示 JavaScript 值。

浏览器中使用 `F12` 来启用调试模式， 在调试窗口中点击 "Console" 菜单。

```bash
<!DOCTYPE html>
<html>
<body>

<h1>我的第一个 Web 页面</h1>

<script>
a = 5;
b = 6;
c = a + b;
console.log(c);
</script>

</body>
</html>
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/16e544793beb4b0bbbaa7ccd2af5b491.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)

