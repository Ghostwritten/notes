#  JavaScript 24. 表单


![在这里插入图片描述](https://img-blog.csdnimg.cn/adaa8c8d87a8452495952a1ca0bcde50.png)


##  1. JavaScript 表单验证
JavaScript 可用来在数据被送往服务器前对 HTML 表单中的这些输入数据进行验证。

表单数据经常需要使用 JavaScript 来验证其正确性：

 - 验证表单数据是否为空？
 - 验证输入是否是一个正确的email地址？
 - 验证日期是否输入正确？
 - 验证表单输入内容是否为数字型？

以下实例代码用于判断表单字段(fname)值是否存在， 如果不存在，就弹出信息，阻止表单提交：

```bash
function validateForm() {
    var x = document.forms["myForm"]["fname"].value;
    if (x == null || x == "") {
        alert("需要输入名字。");
        return false;
    }
}
```
以上 `JavaScript` 代码可以通过 `HTML` 代码来调用：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<script>
function validateForm() {
    var x = document.forms["myForm"]["fname"].value;
    if (x == null || x == "") {
        alert("需要输入名字。");
        return false;
    }
}
</script>
</head>
<body>

<form name="myForm" action="demo_form.php"
onsubmit="return validateForm()" method="post">
名字: <input type="text" name="fname">
<input type="submit" value="提交">
</form>

</body>
</html>
```
输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/f78b2bead5234ddb9868d5c170bea9e5.png)



## 2. JavaScript 验证输入的数字
JavaScript 常用于对输入数字的验证：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
</head>
<body>

<h1>JavaScript 验证输入</h1>

<p>请输入 1 到 10 之间的数字：</p>

<input id="numb">

<button type="button" onclick="myFunction()">提交</button>

<p id="demo"></p>

<script>
function myFunction() {
    var x, text;

    // 获取 id="numb" 的值
    x = document.getElementById("numb").value;

    // 如果输入的值 x 不是数字或者小于 1 或者大于 10，则提示错误 Not a Number or less than one or greater than 10
    if (isNaN(x) || x < 1 || x > 10) {
        text = "输入错误";
    } else {
        text = "输入正确";
    }
    document.getElementById("demo").innerHTML = text;
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/22a7b99cd91d4ec7a8f2e7cf154d39be.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/513a869527ea4b0088e06488e9419c4a.png)
## 3. HTML 表单自动验证
HTML 表单验证也可以通过浏览器来自动完成。

如果表单字段 (fname) 的值为空, `required` 属性会阻止表单提交：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
</head>
<body>

<form action="demo_form.php" method="post">
  <input type="text" name="fname" required="required">
  <input type="submit" value="提交">
</form>

<p>点击提交按钮，如果输入框是空的，浏览器会提示错误信息。</p>

</body>
</html>
```
输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/240c6ae1c00e44b79b745c5350dc83c1.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/da44afec9c654e6a9993ddd8b36dea67.png)

> Internet Explorer 9 及更早 IE 浏览器不支持表单自动验证。

## 4. 数据验证
数据验证用于确保用户输入的数据是有效的。

典型的数据验证有：

 - 必需字段是否有输入?
 - 用户是否输入了合法的数据?
 - 在数字字段是否输入了文本?

大多数情况下，数据验证用于确保用户正确输入数据。

数据验证可以使用不同方法来定义，并通过多种方式来调用。

**服务端数据验证是在数据提交到服务器上后再验证。**

**客户端数据验证是在数据发送到服务器前，在浏览器上完成验证。**

约束验证 HTML 输入属性
|属性|	描述|
|--|--|
|disabled|	规定输入的元素不可用
|max	|规定输入元素的最大值
|min	|规定输入元素的最小值
|pattern|	规定输入元素值的模式
|required	|规定输入元素字段是必需的
|type |	规定输入元素的类型

完整列表，请查看 [HTML 输入属性](https://www.runoob.com/html/html5-form-attributes.html)。

## 5. 约束验证 CSS 伪类选择器
|选择器|	描述|
|--|--|
|:disabled	|选取属性为 "disabled" 属性的 input 元素
|:invalid|	选取无效的 input 元素
|:optional	|选择没有"optional"属性的 input 元素
|:required	|选择有"required"属性的 input 元素
|:valid	|选取有效值的 input 元素

完整列表，请[查看 CSS 伪类](https://www.runoob.com/css/css-pseudo-classes.html)。


##  6. E-mail 验证
下面的函数检查输入的数据是否符合电子邮件地址的基本语法。

意思就是说，输入的数据必须包含 @ 符号和点号(.)。同时，@ 不可以是邮件地址的首字符，并且 @ 之后需有至少一个点号：

```bash
function validateForm(){
  var x=document.forms["myForm"]["email"].value;
  var atpos=x.indexOf("@");
  var dotpos=x.lastIndexOf(".");
  if (atpos<1 || dotpos<atpos+2 || dotpos+2>=x.length){
    alert("不是一个有效的 e-mail 地址");
    return false;
  }
}
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
<script>
function validateForm(){
	var x=document.forms["myForm"]["email"].value;
	var atpos=x.indexOf("@");
	var dotpos=x.lastIndexOf(".");
	if (atpos<1 || dotpos<atpos+2 || dotpos+2>=x.length){
		alert("不是一个有效的 e-mail 地址");
  		return false;
	}
}
</script>
</head>
<body>
	
<form name="myForm" action="demo-form.php" onsubmit="return validateForm();" method="post">
Email: <input type="text" name="email">
<input type="submit" value="提交">
</form>
	
</body>
</html>
```
输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/4ca336a5d3d748d9835bcc4907229d61.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/56e8e7b4f64e4b6aa33a7af025a74b1e.png)

