

----
## 1. 介绍
mustache 模板，用于构造html页面内容。在实际工作中，当同一个模板中想要调用不同的函数来渲染画面，在已经自定义好了的前提下，可以在渲染页面时对传入的参数进行手动判断。【在不好判断的情况下，我们可以通过改变传入参数来进行判断】跟小白我来看看该模板的语法吧。

**[官网详解](https://mustache.github.io/mustache.1.html)**

## 2. 语法

```bash
{{data}}
{{#data}} {{/data}}
{{^data}} {{/data}}
{{.}}
{{<partials}}
{{{data}}}
{{!comments}}
```
## 3. 示例

```bash
  ...
  <script type="text/javascript" src="mustache.js"></script>
  <script type="text/javascript">
  var data = {
      "name": " xiaohua ",
      "msg": {
          "sex": " female ", 
          "age": " 22 ", 
          "hobit": " reading "
     },
     "subject": ["Ch","En","Math","physics"] 
        }  
 
   var tpl = '<p> {{name}}</p>'; 
   var html = Mustache.render(tpl, data);
 
　　alert ( html );
 </script>
 ...
```
### 3.1 {{data}}
`{{}}`就是 Mustache 的标示符，花括号里的 data 表示键名，这句的作用是直接输出与键名匹配的键值，例如：

```bash
 var tpl = '{{name}}';
var html = Mustache.render(tpl, data);
//输出：
xiaohua
```
### 3.2 {{#data}} {{/data}}
以`#`开始、以`/`结束表示区块，它会根据当前上下文中的键值来对区块进行一次或多次渲染，例如改写下 Demo 中的 tpl：

```bash
var tpl = '{{#msg}} <p>{{sex}},{{age}},{{hobit}}</p> {{/msg}}';
var html = Mustache.render(tpl, data);
//输出：
<p> female, 22, reading</p>
```

注意：如果`{{#data}} {{/data}}`中的 data 值为 `null`, `undefined`, `false`；则不渲染输出任何内容。

### 3.3 {{^data}} {{/data}}
该语法与`{{#data}} {{/data}}`类似，不同在于它是当 data值为 null, undefined, false 时才渲染输出该区块内容。

```bash
var tpl = {{^nothing}}没找到 nothing 键名就会渲染这段{{/nothing}};
var html = Mustache.render(tpl, data);
//输出：
没找到 nothing 键名就会渲染这段
```

### 3.4 {{.}}
`{{.}}`表示枚举，可以循环输出整个数组，例如：

```bash
var tpl = '{{#subject}} <p>{{.}}</p> {{/subject}}';
var html = Mustache.render(tpl, data);
//输出：
<p>Ch</p> <p>En</p> <p>Math</p> <p>physics</p>
```

### 3.5 {{>partials}}
以>开始表示子模块，如`{{> msg}}`；当结构比较复杂时，我们可以使用该语法将复杂的结构拆分成几个小的子模块，例如：

```bash
var tpl = "<h1>{{namme}}</h1> <ul>{{>msg}}</ul>"
var partials = {msg: "{{#msg}}<li>{{sex}}</li><li>{{age}}</li><li>{{hobit}}</li>{{/msg}
var html = Mustache.render(tpl, data, partials);
//输出：
 <h1>xiaohua</h1>
 <ul>
  <li>female</li>
   <li>22</li>
   <li>reading</li>
 </ul>
```

### 3.6 {{{data}}}
{{data}}输出会将等特殊字符转译，如果想保持内容原样输出可以使用{{{}}}，例如：

```bash
var tpl = '{{#msg}} <p>{{{age}}}</p> {{/msg}}'
//输出：
 <p>22</p>

```

### 3.7 {{!comments}}
!表示注释，注释后不会渲染输出任何内容。

```bash
{{!这里是注释}}
2 //输出：
 
```

在工作中，如果页面上的内容是从后台获取数据并渲染到页面上时，我们就可以使用mustache模板了，值得注意的是，render的数据一定要与键名相符合。


```bash
<br><br><br><br>
```
参考链接：
[https://www.cnblogs.com/DF-fzh/p/5979093.html](https://www.cnblogs.com/DF-fzh/p/5979093.html)
