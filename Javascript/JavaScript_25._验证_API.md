#  JavaScript 25. 验证 API


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f0a01d12c97127fd5f1e93b4515d163c.png)


##  1. 约束验证 DOM 方法

 - `checkValidity()`	如果 input 元素中的数据是合法的返回 true，否则返回 false。
 - `setCustomValidity()` 设置 input 元素的 `validationMessage` 属性，用于自定义错误提示信息的方法。
使用 setCustomValidity 设置了自定义提示后`，validity.customError` 就会变成 true，`checkValidity` 总是会返回 false。如果要重新判断需要取消自定义提示，方式如下：

   - `setCustomValidity('')` 
   - `setCustomValidity(null)` 
   - `setCustomValidity(undefined)`

以下实例如果输入信息不合法，则返回错误信息：

```bash
<input id="id1" type="number" min="100" max="300" required>
<button onclick="myFunction()">验证</button>
 
<p id="demo"></p>
 
<script>
function myFunction() {
    var inpObj = document.getElementById("id1");
    if (inpObj.checkValidity() == false) {
        document.getElementById("demo").innerHTML = inpObj.validationMessage;
    }
}
</script>
```
实例

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
</head>
<body>

<p>输入数字并点击验证按钮:</p>

<input id="id1" type="number" min="100" max="300" required>
<button onclick="myFunction()">验证</button>

<p>如果输入的数字小于 100 或大于300，会提示错误信息。</p>

<p id="demo"></p>

<script>
function myFunction() {
    var inpObj = document.getElementById("id1");
    if (inpObj.checkValidity() == false) {
        document.getElementById("demo").innerHTML = inpObj.validationMessage;
    } else {
        document.getElementById("demo").innerHTML = "输入正确";
    }
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/868b2b49cfc7f5c4a640b2bb4d71fab3.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/614eeeace7427e82b3add3a9820c286b.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c680fea8c103fa1f5c9fb8bcb40e2cbb.png)
##  2. 约束验证 DOM 属性
|属性	|描述|
|--|--|
|validity|	布尔属性值，返回 input 输入值是否合法
|validationMessage|	浏览器错误提示信息
|willValidate	|指定 input 是否需要验证

##  3. Validity 属性
input 元素的 validity 属性包含一系列关于 validity 数据属性:
|属性|	描述|
|--|--|
|customError	|设置为 true, 如果设置了自定义的 validity 信息。|
|patternMismatch|	设置为 true, 如果元素的值不匹配它的模式属性。|
|rangeOverflow	|设置为 true, 如果元素的值大于设置的最大值。|
|rangeUnderflow|	设置为 true, 如果元素的值小于它的最小值。|
|stepMismatch	|设置为 true, 如果元素的值不是按照规定的 step 属性设置。|
|tooLong	|设置为 true, 如果元素的值超过了 maxLength 属性设置的长度。|
|typeMismatch	|设置为 true, 如果元素的值不是预期相匹配的类型。|
|valueMissing	|设置为 true，如果元素 (required 属性) 没有值。|
|valid	|设置为 true，如果元素的值是合法的。|

### 3.1 实例 1
如果输入的值大于 100，显示一个信息：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
</head>
<body>

<p>输入数字并点击验证按钮:</p>

<input id="id1" type="number" max="100">
<button onclick="myFunction()">验证</button>

<p>如果输入的数字大于 100 ( input 的 max 属性), 会显示错误信息。</p>

<p id="demo"></p>

<script>
function myFunction() {
    var txt = "";
    if (document.getElementById("id1").validity.rangeOverflow) {
        txt = "输入的值太大了";
    } else {
        txt = "输入正确";
    }
    document.getElementById("demo").innerHTML = txt;
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/84fe79e53d2d80652b34af57e625392b.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/be38a26fbf1373d532842877182ea300.png)
### 3.2 实例 2
如果输入的值小于 100，显示一个信息：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
</head>
<body>

<p>输入数字并点击验证按钮:</p>
<input id="id1" type="number" min="100" required>
<button onclick="myFunction()">验证</button>

<p>如果输入的数字小于 100 ( input 的 min 属性), 会显示错误信息。</p>

<p id="demo"></p>

<script>
function myFunction() {
    var txt = "";
	var inpObj = document.getElementById("id1");
	if(!isNumeric(inpObj.value)) {
		txt = "你输入的不是数字";
	} else if (inpObj.validity.rangeUnderflow) {
        txt = "输入的值太小了";
    } else {
        txt = "输入正确";
    }
    document.getElementById("demo").innerHTML = txt;
}

// 判断输入是否为数字
function isNumeric(n) {
    return !isNaN(parseFloat(n)) && isFinite(n);
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7ea173ca286d3b11879ef80bbcc01e3b.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/74c6150e98b2b066e6db39911aafc133.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9141a2e4e6f6d994f15d6a9599413226.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b53a8a35736b10bd076472f8a0425475.png)

