


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/72a9461e9101ab71a6f7c72f37e19cde.png)


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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c6f42d58055556c1694dac330abd709b.png)

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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/97bee8f9857cb13f7cc8a91e8ada9f76.png)
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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/520f8dc08daf0d6454fb4d24b0b6520f.png)
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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8bf6e9eac35d4c01d2d714751a6dc985.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ac0f3c3481dd76ab784fa0d213391984.png)

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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d65f294bc1368f4dcd267e8c6b0f2a16.png)

