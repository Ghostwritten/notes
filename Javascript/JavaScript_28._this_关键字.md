#   JavaScript 28. this 关键字


![在这里插入图片描述](https://img-blog.csdnimg.cn/f72f892bb7414e938a24b6f736d9b043.png)



##  1. 简介
面向对象语言中 this 表示当前对象的一个引用。

但在 JavaScript 中 this 不是固定不变的，它会随着执行环境的改变而改变。

 - 在方法中，this 表示该方法所属的对象。
 - 如果单独使用，this 表示全局对象。
 - 在函数中，this 表示全局对象。
 - 在函数中，在严格模式下，this 是未定义的(undefined)。
 - 在事件中，this 表示接收事件的元素。
 - 类似 `call()` 和 `apply()` 方法可以将 this 引用到任何对象。



##  2. 方法中的 this
在对象方法中， `this` 指向调用它所在方法的对象。

在上面一个实例中，this 表示 person 对象。

`fullName` 方法所属的对象就是 person。

实例

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<h2>JavaScript <b>this</b> 关键字</h2>

<p>实例中，<b>this</b> 指向了 <b>person</b> 对象。</p>
<p>因为 person 对象是 fullName 方法的所有者。</p>

<p id="demo"></p>

<script>
// 创建一个对象
var person = {
  firstName: "John",
  lastName : "Doe",
  id     : 5566,
  fullName : function() {
    return this.firstName + " " + this.lastName;
  }
};

// 显示对象的数据
document.getElementById("demo").innerHTML = person.fullName();
</script>

</body>
</html>
```
输出

```bash
JavaScript this 关键字
实例中，this 指向了 person 对象。

因为 person 对象是 fullName 方法的所有者。

John Doe
```

## 3. 单独使用 this
单独使用 this，则它指向全局(Global)对象。

在浏览器中，window 就是该全局对象为 `[object Window]`:

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<h2>JavaScript <b>this</b> 关键字</h2>

<p>实例中，<b>this</b> 指向了 window 对象:</p>

<p id="demo"></p>

<script>
var x = this;
document.getElementById("demo").innerHTML = x;
</script>

</body>
</html>
```
输出：

```bash
JavaScript this 关键字
实例中，this 指向了 window 对象:

[object Window]
```
严格模式下，如果单独使用，this 也是指向全局(Global)对象。

```bash
"use strict";
var x = this;
```

## 4. 函数中使用 this（默认）
在函数中，函数的所属者默认绑定到 this 上。

在浏览器中，window 就是该全局对象为 [object Window]:

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<h2>JavaScript <b>this</b> 关键字</h2>

<p>实例中，<b>this</b> 表示 myFunction 函数的所有者：</p>

<p id="demo"></p>

<script>
document.getElementById("demo").innerHTML = myFunction();
function myFunction() {
  return this;
}
</script>

</body>
</html>
```
输出：

```bash
JavaScript this 关键字
实例中，this 表示 myFunction 函数的所有者：

[object Window]
```
## 5. 函数中使用 this（严格模式）
严格模式下函数是没有绑定到 this 上，这时候 this 是 `undefined`。

```bash
"use strict";
function myFunction() {
  return this;
}
```
## 6. 事件中的 this
在 HTML 事件句柄中，this 指向了接收事件的 HTML 元素：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<h2>JavaScript <b>this</b> 关键字</h2>

<button onclick="this.style.display='none'">点我后我就消失了</button>

</body>
</html>
```
输出
![在这里插入图片描述](https://img-blog.csdnimg.cn/8933b0a15a8f4c6eac931692e5b04df9.gif#pic_center)
## 7. 对象方法中绑定
下面实例中，this 是 person 对象，person 对象是函数的所有者：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<h2>JavaScript <b>this</b> 关键字</h2>

<p>在实例中，<b>this</b> 指向了 fullName 方法所属的对象 person。</p>

<p id="demo"></p>

<script>
// 创建一个对象
var person = {
  firstName  : "John",
  lastName   : "Doe",
  id     : 5566,
  myFunction : function() {
    return this;
  }
};

// 显示表单数据
document.getElementById("demo").innerHTML = person.myFunction();
</script>

</body>
</html>
```
输出：

```bash
JavaScript this 关键字
在实例中，this 指向了 fullName 方法所属的对象 person。

[object Object]
```
## 8. 显式函数绑定
在 JavaScript 中函数也是对象，对象则有方法，apply 和 call 就是函数对象的方法。这两个方法异常强大，他们允许切换函数执行的上下文环境（context），即 this 绑定的对象。

在下面实例中，当我们使用 person2 作为参数来调用 person1.fullName 方法时, this 将指向 person2, 即便它是 person1 的方法：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<h2>JavaScript this 关键字</h2>
<p>实例中 <strong>this</strong> 指向了 person2，即便它是 person1 的方法:</p>

<p id="demo"></p>

<script>
var person1 = {
  fullName: function() {
    return this.firstName + " " + this.lastName;
  }
}
var person2 = {
  firstName:"John",
  lastName: "Doe",
}
var x = person1.fullName.call(person2); 
document.getElementById("demo").innerHTML = x; 
</script>

</body>
</html>
```
输出：

```bash
JavaScript this 关键字
实例中 this 指向了 person2，即便它是 person1 的方法:

John Doe
```

