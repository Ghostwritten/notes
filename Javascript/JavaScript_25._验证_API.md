#  JavaScript 25. 验证 API


![在这里插入图片描述](https://img-blog.csdnimg.cn/9179f7d845cc419c8ef2913489b37719.png)


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
![在这里插入图片描述](https://img-blog.csdnimg.cn/645a26685e4e4b9aa89c08e1eb8d42d3.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/c6e69b9c9c3645b391df315b005a2f57.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/5d2c8ff6a066433f8ba83dca3fce5912.png)
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
![在这里插入图片描述](https://img-blog.csdnimg.cn/212ef979577f4fad8642fd4ae212bcf0.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/8f96db742de5466bbc8431cc681139e1.png)
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
![在这里插入图片描述](https://img-blog.csdnimg.cn/7c72f7cd63d64295b1ecc78eb8e6fe1a.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/c179d0e8dd464cb5876187024b7cecea.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/a942b4e187684e0d986f9bf2be629215.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/d18ebd949b4b4a64828ca8505f3d80ac.png)

