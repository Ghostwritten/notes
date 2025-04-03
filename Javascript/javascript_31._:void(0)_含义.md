#  javascript 31. :void(0) 含义



![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cdb7980e896fe4584c64c55516cae3c7.png)


## 1.  简介
我们经常会使用到 `javascript:void(0)` 这样的代码，那么在 JavaScript 中 `javascript:void(0)` 代表的是什么意思呢？

`javascript:void(0)` 中最关键的是 void 关键字， void 是 JavaScript 中非常重要的关键字，该操作符指定要计算一个表达式但是不返回值。

语法格式如下：

```bash
void func()
javascript:void func()
```

或者

```bash
void(func())
javascript:void(func())
```
## 2. 实例1
下面的代码创建了一个超级链接，当用户点击以后不会发生任何事
```bash
<!DOCTYPE html> 
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head> 
<body>
	
    <a href="javascript:void(0)">单击此处什么也不会发生</a>
	
</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6559d69e0d07b8e77d033778e055c775.png)
## 3. 实例2
当用户链接时，void(0) 计算为 0，但 Javascript 上没有任何效果。

以下实例中，在用户点击链接后显示警告信息：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>
	
<p>点击以下链接查看结果：</p>
<a href="javascript:void(alert('Warning!!!'))">点我!</a>
	
</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2b1f1a59038c3e15ee1985794f266ea2.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d9f8a0a3f69682b406cefa12255d0996.png)
## 4. 实例3
以下实例中参数 a 将返回 `undefined` :

```bash
<!DOCTYPE html> 
<html> 
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
<script type="text/javascript">
function getValue(){
   var a,b,c;
   a = void ( b = 5, c = 7 );
   document.write('a = ' + a + ' b = ' + b +' c = ' + c );
}
</script>
</head>
<body>
	
<p>点击以下按钮查看结果：</p>
<form>
<input type="button" value="点我" onclick="getValue();" />
</form>
	
</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7fa01bba3382595adcd3b9eea6135ba9.gif#pic_center)

## 5. href="#"与href="javascript:void(0)"的区别
`#` 包含了一个位置信息，默认的锚是#top 也就是网页的上端。

而`javascript:void(0)`, 仅仅表示一个死链接。

在页面很长的时候会使用 # 来定位页面的具体位置，格式为：`# + id`。

如果你要定义一个死链接请使用 javascript:void(0) 。
```bash
<a href="javascript:void(0);">点我没有反应的!</a>
<a href="#pos">点我定位到指定位置!</a>
<br>
...
<br>
<p id="pos">尾部定位点</p>
```
输出
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d4460dcd78ece7406fe85214e479fae5.gif#pic_center)

```bash
// 阻止链接跳转，URL不会有任何变化
<a href="javascript:void(0)" rel="nofollow ugc">点击此处</a>

// 虽然阻止了链接跳转，但URL尾部会多个#，改变了当前URL。（# 主要用于配合 location.hash）
<a href="#" rel="nofollow ugc">点击此处</a>

// 同理，# 可以的话，? 也能达到阻止页面跳转的效果，但也相同的改变了URL。（? 主要用于配合 location.search）
<a href="?" rel="nofollow ugc">点击此处</a>

// Chrome 中即使 javascript:0; 也没变化，firefox中会变成一个字符串0
<a href="javascript:0" rel="nofollow ugc">点击此处</a>
```

