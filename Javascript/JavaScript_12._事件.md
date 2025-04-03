#  JavaScript 12. 事件


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7d8cf590e2964ec9e2b9436360a7a6be.png)


##  1. 简介
HTML 事件可以是浏览器行为，也可以是用户行为。

以下是 HTML 事件的实例：

 - HTML 页面完成加载
 - HTML input 字段改变时
 - HTML 按钮被点击

通常，当事件发生时，你可以做些事情。

在事件触发时 JavaScript 可以执行一些代码。

HTML 元素中可以添加事件属性，使用 JavaScript 代码来添加 HTML 元素。

```bash
<button onclick="getElementById('demo').innerHTML=Date()">现在的时间是?</button>
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

<button onclick="getElementById('demo').innerHTML=Date()">现在的时间是?</button>
<p id="demo"></p>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/69e90cb48a97de6768fcb1c859c79b38.png)
实例2

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<button onclick="this.innerHTML=Date()">现在的时间是?</button>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4f81ced24eccccce79d4d64ccb9a49f8.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/72a90e26bdcbee6fe2d662422ac820e5.png)
实例3

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<p>点击按钮执行 <em>displayDate()</em> 函数.</p>
<button onclick="displayDate()">点这里</button>
<script>
function displayDate(){
	document.getElementById("demo").innerHTML=Date();
}
</script>
<p id="demo"></p>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8b70cc582210f869c03d4b943de8a34c.png)
##  2. 常见的HTML事件
|事件|	描述
|--|--|
|onchange|	HTML 元素改变
|onclick|	用户点击 HTML 元素
|onmouseover|	鼠标指针移动到指定的元素上时发生
|onmouseout	|用户从一个 HTML 元素上移开鼠标时发生
|onkeydown	|用户按下键盘按键
|onload	|浏览器已完成页面的加载

##  3. JavaScript 可以做什么?
事件可以用于处理表单验证，用户输入，用户行为及浏览器动作:

 - 页面加载时触发事件
 - 页面关闭时触发事件
 - 用户点击按钮执行动作
 - 验证用户输入内容的合法性

等等 ...
可以使用多种方法来执行 JavaScript 事件代码：

 - HTML 事件属性可以直接执行 JavaScript 代码
 - HTML 事件属性可以调用 JavaScript 函数
 - 你可以为 HTML 元素指定自己的事件处理程序
 - 你可以阻止事件的发生。

等等 ...
