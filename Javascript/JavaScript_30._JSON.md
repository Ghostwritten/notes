#  JavaScript 30. JSON


![在这里插入图片描述](https://img-blog.csdnimg.cn/3a9f921275af42768afdb70fb836a03c.png)


## 1. 什么是 JSON?

 - JSON 是用于存储和传输数据的格式。
 - JSON 通常用于服务端向网页传递数据 。
 - JSON 英文全称 JavaScript Object Notation
 - JSON 是一种轻量级的数据交换格式。
 - JSON是独立的语言 *
 - JSON 易于理解。

##  2. JSON 语法规则

 - 数据为 键/值 对。
 - 数据由逗号分隔。
 - 大括号保存对象
 - 方括号保存数组

##  3. JSON 数据 - 一个名称对应一个值
JSON 数据格式为 键/值 对，就像 JavaScript 对象属性。

键/值对包括字段名称（在双引号中），后面一个冒号，然后是值：

```bash
"name":"Runoob"
```

## 4. JSON 对象
JSON 对象保存在大括号内。

就像在 JavaScript 中, 对象可以保存多个 键/值 对：

```bash
{"name":"Runoob", "url":"www.runoob.com"}
```
## 5. JSON 数组
JSON 数组保存在中括号内。

就像在 JavaScript 中, 数组可以包含对象：

```bash
"sites":[
    {"name":"Runoob", "url":"www.runoob.com"}, 
    {"name":"Google", "url":"www.google.com"},
    {"name":"Taobao", "url":"www.taobao.com"}
]
```
在以上实例中，对象 "sites" 是一个数组，包含了三个对象。

每个对象为站点的信息（网站名和网站地址）。

## 6. SON 字符串转换为 JavaScript 对象
通常我们从服务器中读取 JSON 数据，并在网页中显示数据。

简单起见，我们网页中直接设置 JSON 字符串 (你还可以阅读我们的 [JSON 教程](https://www.runoob.com/json/json-tutorial.html)):

首先，创建 JavaScript 字符串，字符串为 JSON 格式的数据：

```bash
var text = '{ "sites" : [' +
'{ "name":"Runoob" , "url":"www.runoob.com" },' +
'{ "name":"Google" , "url":"www.google.com" },' +
'{ "name":"Taobao" , "url":"www.taobao.com" } ]}';
```
然后，使用 JavaScript 内置函数 `JSON.parse()` 将字符串转换为 JavaScript 对象:

```bash
var obj = JSON.parse(text)
```
最后，在你的页面中使用新的 JavaScript 对象：

```bash
var text = '{ "sites" : [' +
    '{ "name":"Runoob" , "url":"www.runoob.com" },' +
    '{ "name":"Google" , "url":"www.google.com" },' +
    '{ "name":"Taobao" , "url":"www.taobao.com" } ]}';
    
obj = JSON.parse(text);
document.getElementById("demo").innerHTML = obj.sites[1].name + " " + obj.sites[1].url;
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

<h2>为 JSON 字符串创建对象</h2>
<p id="demo"></p>
<script>
var text = '{ "sites" : [' +
	'{ "name":"Runoob" , "url":"www.runoob.com" },' +
	'{ "name":"Google" , "url":"www.google.com" },' +
	'{ "name":"Taobao" , "url":"www.taobao.com" } ]}';
	
obj = JSON.parse(text);
document.getElementById("demo").innerHTML = obj.sites[1].name + " " + obj.sites[1].url;
</script>

</body>
</html>
```
输出

```bash
为 JSON 字符串创建对象
Google www.google.com
```

## 8. 相关函数

|函数|	描述|
|--|--|
|[JSON.parse()](https://www.runoob.com/js/javascript-json-parse.html)|	用于将一个 JSON 字符串转换为 JavaScript 对象。
|[JSON.stringify()](https://www.runoob.com/js/javascript-json-stringify.html)	|用于将 JavaScript 值转换为 JSON 字符串。


## 9. JSON 与 JS 对象的关系

```bash
var obj = {a: 'Hello', b: 'World'}; //这是一个js对象，注意js对象的键名也是可以使用引号包裹的,这里的键名就不用引号包含
var json = '{"a": "Hello", "b": "World"}'; //这是一个 JSON 字符串，本质是一个字符串
```
要实现从JSON字符串转换为JS对象，使用 JSON.parse() 方法：

```bash
var obj = JSON.parse('{"a": "Hello", "b": "World"}'); //结果是 {a: 'Hello', b: 'World'}  一个对象
```
要实现从JS对象转换为JSON字符串，使用 JSON.stringify() 方法：

```bash
var json = JSON.stringify({a: 'Hello', b: 'World'}); //结果是 '{"a": "Hello", "b": "World"}'  一个JSON格式的字符串
```

