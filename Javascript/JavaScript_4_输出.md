


![在这里插入图片描述](https://img-blog.csdnimg.cn/ada439259de24b25ada6dabda62007fd.png)


##  1. 简介
JavaScript 可以通过不同的方式来输出数据：

 - 使用 `window.alert()` 弹出警告框。
 - 使用 `document.write()` 方法将内容写到 HTML 文档中。
 - 使用 `innerHTML` 写入到 HTML 元素。
 - 使用 `console.log()` 写入到浏览器的控制台。

##  2. window.alert
```bash
<!DOCTYPE html>
<html>
<body>

<h1>我的第一个页面</h1>
<p>我的第一个段落。</p>

<script>
window.alert(5 + 6);
</script>

</body>
</html>
```
输出
![在这里插入图片描述](https://img-blog.csdnimg.cn/1be3b8d1eda44356813e0ad81ef2f07c.png)

##  3. 操作 HTML 元素
请使用 "id" 属性来标识 HTML 元素，并 innerHTML 来获取或插入元素内容：

```bash
<!DOCTYPE html>
<html>
<body>

<h1>我的第一个 Web 页面</h1>

<p id="demo">我的第一个段落</p>

<script>
document.getElementById("demo").innerHTML = "段落已修改。";
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/cca4a0ea64aa4568bcb42d3a8f91264c.png)
##  4. 写到 HTML 文档

```bash
<!DOCTYPE html>
<html>
<body>

<h1>我的第一个 Web 页面</h1>

<p>我的第一个段落。</p>

<script>
document.write(Date());
</script>

</body>
</html>
```


输出
![在这里插入图片描述](https://img-blog.csdnimg.cn/b49c73764e0c42fbac07c7d5af6025ba.png)
第二例

```bash
<!DOCTYPE html>
<html>
<body>

<h1>我的第一个 Web 页面</h1>

<p>我的第一个段落。</p>

<button onclick="myFunction()">点我</button>

<script>
function myFunction() {
    document.write(Date());
}
</script>

</body>
</html>
```
输出
![在这里插入图片描述](https://img-blog.csdnimg.cn/14c7fca289734f3ab35c6b11451b202e.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/90b572f8245a4b3881f5bd00b28b9c47.png)

##  5. 写到控制台
如果您的浏览器支持调试，你可以使用 `console.log()` 方法在浏览器中显示 JavaScript 值。

浏览器中使用 `F12` 来启用调试模式， 在调试窗口中点击 "`Console`" 菜单。

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
实例 console 截图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/62e3a6571b974e0db54de86ed48d8685.png)

