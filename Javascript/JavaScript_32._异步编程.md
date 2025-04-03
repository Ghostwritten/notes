#  JavaScript 32. 异步编程


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/03d2e9bc5c0bc5a891cd05e9591b506d.png)


##  1. 异步的概念
异步（Asynchronous, async）是与同步（Synchronous, sync）相对的概念。

在我们学习的传统单线程编程中，程序的运行是同步的（同步不意味着所有步骤同时运行，而是指步骤在一个控制流序列中按顺序执行）。而异步的概念则是不保证同步的概念，也就是说，一个异步过程的执行将不再与原有的序列有顺序关系。

简单来理解就是：同步按你的代码顺序执行，异步不按照代码顺序执行，异步的执行效率更高。

以上是关于异步的概念的解释，接下来我们通俗地解释一下异步：异步就是从主线程发射一个子线程来完成任务。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c3fd8ec0988c400d18f69459ebcb6e30.png)
## 2. 什么时候用异步编程
在前端编程中（甚至后端有时也是这样），我们在处理一些简短、快速的操作时，例如计算 1 + 1 的结果，往往在主线程中就可以完成。主线程作为一个线程，不能够同时接受多方面的请求。所以，当一个事件没有结束时，界面将无法处理其他请求。

现在有一个按钮，如果我们设置它的 onclick 事件为一个死循环，那么当这个按钮按下，整个网页将失去响应。

为了避免这种情况的发生，我们常常用子线程来完成一些可能消耗时间足够长以至于被用户察觉的事情，比如读取一个大文件或者发出一个网络请求。因为子线程独立于主线程，所以即使出现阻塞也不会影响主线程的运行。但是子线程有一个局限：一旦发射了以后就会与主线程失去同步，我们无法确定它的结束，如果结束之后需要处理一些事情，比如处理来自服务器的信息，我们是无法将它合并到主线程中去的。

为了解决这个问题，JavaScript 中的异步操作函数往往通过回调函数来实现异步任务的结果处理。

## 3. 回调函数
回调函数就是一个函数，它是在我们启动一个异步任务的时候就告诉它：等你完成了这个任务之后要干什么。这样一来主线程几乎不用关心异步任务的状态了，他自己会善始善终。

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<p>回调函数等待 3 秒后执行。</p>
<p id="demo"></p>
<script>
function print() {
    document.getElementById("demo").innerHTML="RUNOOB!";
}
setTimeout(print, 3000);
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/07798de9d5a757b536fc1bc807007998.gif#pic_center)
这段程序中的 `setTimeout` 就是一个消耗时间较长（3 秒）的过程，它的第一个参数是个回调函数，第二个参数是毫秒数，这个函数执行之后会产生一个子线程，子线程会等待 3 秒，然后执行回调函数 `"print"`，在命令行输出 `"RUNOOB!"`。

当然，JavaScript 语法十分友好，我们不必单独定义一个函数 print ，我们常常将上面的程序写成：

```bash
setTimeout(function () {
    document.getElementById("demo").innerHTML="RUNOOB!";
}, 3000);
```
注意：既然 `setTimeout` 会在子线程中等待 3 秒，在 setTimeout 函数执行之后主线程并没有停止，所以：

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<p>回调函数等待 3 秒后执行。</p>
<p id="demo1"></p>
<p id="demo2"></p>
<script>
setTimeout(function () {
    document.getElementById("demo1").innerHTML="RUNOOB-1!";
}, 3000);
document.getElementById("demo2").innerHTML="RUNOOB-2!";
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f023648c27fa1c943e39f430c6de6f9a.gif#pic_center)

## 4. 异步 AJAX
除了 setTimeout 函数以外，异步回调广泛应用于 AJAX 编程。有关于 AJAX 详细请参见：[https://www.runoob.com/ajax/ajax-tutorial.html](https://www.runoob.com/ajax/ajax-tutorial.html)

XMLHttpRequest 常常用于请求来自远程服务器上的 XML 或 JSON 数据。一个标准的 XMLHttpRequest 对象往往包含多个回调：

```bash
var xhr = new XMLHttpRequest();
 
xhr.onload = function () {
    // 输出接收到的文字数据
    document.getElementById("demo").innerHTML=xhr.responseText;
}
 
xhr.onerror = function () {
    document.getElementById("demo").innerHTML="请求出错";
}
 
// 发送异步 GET 请求
xhr.open("GET", "https://www.runoob.com/try/ajax/ajax_info.txt", true);
xhr.send();
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

<p><strong>以下内容是通过异步请求获取的：</strong></p>
<p id="demo"></p>
<script>
var xhr = new XMLHttpRequest();
 
xhr.onload = function () {
    // 输出接收到的文字数据
    document.getElementById("demo").innerHTML=xhr.responseText;
}
 
xhr.onerror = function () {
    document.getElementById("demo").innerHTML="请求出错";
}
 
// 发送异步 GET 请求
xhr.open("GET", "/try/ajax/ajax_info.txt", true);
xhr.send();
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a610d1f27e2d04a95d558076fefb36b1.png)
`XMLHttpRequest` 的 `onload` 和 `onerror` 属性都是函数，分别在它请求成功和请求失败时被调用。如果你使用完整的 [jQuery](https://jquery.com/) 库，也可以更加优雅的使用异步 AJAX：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
<script src="https://cdn.staticfile.org/jquery/1.10.2/jquery.min.js">
</script>
<script>
$(document).ready(function(){
	$("button").click(function(){
		$.get("/try/ajax/demo_test.php",function(data,status){
			alert("数据: " + data + "\n状态: " + status);
		});
	});
});
</script>
</head>
<body>

<button>发送一个 HTTP GET 请求并获取返回结果</button>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9255589cdb8274eb2e7bdb6e1efd3c9e.gif#pic_center)

