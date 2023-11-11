

-----
## 1. 前端背景
网页主要由三部分组成：结 构 （Structure）、表现 （Presentation）和行为 （Behavior）。。对应的标准 也分三方面：结构化标准语言主要包括XHTML和XML，表现标准语言主要包括CSS， 行为标准主要包括对象模型（如W3C DOM）、ECMAScript等。本节我们主要讲解 `HTML、CSS、JavaScript、Xpath`和JSON等5个部分。

## 2. html
什么是HTML标记语言？HTML不是编程语言，是一种表示网页信息的符号标记语 言。标记语言是一套标记，HTML使用标记来描述网页。Web浏览器的作用是读取 HTML文档，并以网页的形式显示出它们。浏览器不会显示HTML标记，而是使用标记 来解释页面的内容。HTML语言的特点包括：

 - ·可以设置文本的格式，比如标题、字号、文本颜色、段落，等等。
 - ·可以创建列表。
 - ·可以插入图像和媒体。
 - ·可以建立表格。
 - ·超链接，可以使用鼠标点击超链接来实现页面之间的跳转。

### 2.1 HTML的基本结构
下面从HTML的基本结构、文档设置标记、图像标记、表格和超链接五个方面讲
解。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200908171647170.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)



从谷歌首页的源代码中可以分析出HTML的基本结构：

```bash
·<html>内容</html>：HTML文档是由<html></html>包裹，这是HTML文档的 文档标记，也称为HTML开始标记。这对标记分别位于网页的最前端和最后端， <html>在最前端表示网页的开始，</html>在最后端表示网页的结束。
·<head>内容</head>：HTML文件头标记，也称为HTML头信息开始标记。用来包 含文件的基本信息，比如网页的标题、关键字，在<head></head>内可以放 <title></title>、<meta></meta>、<style></style>等标记。注意：在 <head></head>标记内的内容不会在浏览器中显示。

·<title>内容</title>：HTML文件标题标记。网页的“主题”，显示在浏览器的窗口的左上边。
·<body>内容</body>：<body>...</body>是网页的主体部分，在此标记之间 可以包含如<p></p>、<h1></h1>、<br>、<hr>等等标记，正是由这些内容组成了 我们所看见的网页。
·<meta>内容</meta>：页面的元信息（meta-information）。提供有关页面 的元信息，比如针对搜索引擎和更新频度的描述和关键词。注意meta标记必须放在 head元素里面。
```

### 2.2 文档设置标记

文档设置标记分为格式标记和文本标记。下面通过一个标准的HTML文档对格式 标记进行讲解，文档如下所示：
 

```bash
  <html>     
  <head>        
  <title>Python爬虫开发与项目实战</title>        
  <meta charset="UTF-8">     
  </head>    
   <body>     文档设置标记<br>     
   <p>这是段落。</p>     
   <p>这是段落。</p>     
   <p>这是段落。</p>     
   <hr>     
   <center>居中标记1</center>     
   <center>居中标记2</center>    
    <hr>     
    <pre>     [00:00](music)     [00:28]你我皆凡人，生在人世间；     [00:35]终日奔波苦，一刻不得闲；     [00:43]既然不是仙，难免有杂念；
     </pre>     
     <hr>     
     <p>     [00:00](music)     [00:28]你我皆凡人，生在人世间；     [00:35]终日奔波苦，一刻不得闲；     [00:43]既然不是仙，难免有杂念；     
     </p>     
     <hr>     
     <br>     
     <ul>     
     <li>Coffee</li>     
     <li>Milk</li>     
     </ul>     
     <ol type="A">     
     <li>Coffee</li>     
     <li>Milk</li>     
     </ol>        
     <dl>        
     <dt>计算机</dt>        
     <dd>用来计算的仪器 ... ...</dd>        
     <dt>显示器</dt>        
     <dd>以视觉方式显示信息的装置 ... ...</dd>        
     </dl>        
     <div >                
     <h3>这是标题</h3>                
     <p>这是段落。</p>     
     </div>     
     </body>    
     </html>
```
运行效果图

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020090817263795.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
格式标记包括：

```bash
·<br>：强制换行标记。让后面的文字、图片、表格等等，显示在下一行。
·<p>：换段落标记。换段落，由于多个空格和回车在HTML中会被等效为一个空 格，所以HTML中要换段落就要用<p>，<p>段落中也可以包含<p>段落。例如： <p>This is a paragraph.</p>。
·<center>：居中对齐标记。让段落或者是文字相对于父标记居中显示。
·<pre>：预格式化标记。保留预先编排好的格式，常用来定义计算机源代码。
和<p>进行一下对比，就可以理解。
·<li>：列表项目标记。每一个列表使用一个<li>标记，可用在有序列表 （<ol>）和无序列表（<ul>）中。
·<ul>：无序列表标记。<ul>声明这个列表没有序号。
·<ol>：有序列表标记。可以显示特定的一些顺序。有序列表的type属性 值“1”表示阿拉伯数字1.2.3等等；默认type属性值“A”表示大小字母A、B、C等 等；上面的程序使用属性“a”，这表示小写字母a、b、c等等；“Ⅰ”表示大写罗 马数字Ⅰ、Ⅱ、Ⅲ、Ⅳ等等；“ⅰ”表示小写罗马数字ⅰ、ⅱ、ⅲ、ⅳ等等。注 意：列表可以进行嵌套。
·<dl><dt><dd>：定义型列表。对列表条目进行简短说明。
·<hr>：水平分割线标记。可以用作段落之间的分割线。
·<div>：分区显示标记，也称为层标记。常用来编排一大段的HTML段落，也可 以用于将表格式化，和<p>很相似，可以多层嵌套使用。
```
接下来通过一个HTML文档对文本标记进行讲解，文档如下所示：

```bash
  <html>     
  <head>        
  <title>Python爬虫开发与项目实战</title>        
  <meta charset="UTF-8">     
  </head>     
  <body>     Hn标题标记---->>     
  <br>        
  <h1>Python爬虫</h1>        
  <h2>Python爬虫</h2>
    <h3>Python爬虫</h3>        
    <h4>Python爬虫</h4>        
    <h5>Python爬虫</h5>        
    <h6>Python爬虫</h6>     font标记---->>     
    <font size="1">Python爬虫</font>     
    <font size="3">Python爬虫</font>     
    <font size="7">Python爬虫</font>     
    <font size="7" color="red" face="微软雅黑">Python爬虫</font>     
    <font size="7" color="red" face="宋体">Python爬虫</font>     
    <font size="7" color="red" face="新细明体">Python爬虫</font>     
    <br>     B标记加粗---->>     
    <b>Python爬虫</b>     
    <br>     i标记斜体---->>     
    <i>Python爬虫</i>     
    <br>     sub下标标记---->>     
    2<sub>2</sub>     
    <br>     sup上标标记---->>     
    2<sup>2</sup>     <br>     引用标记---->>     
    <cite>Python爬虫</cite>     
    <br>     em标记表示强调，显示为斜体---->>     
    <em>Python爬虫</em>     
    <br>     strong标记表示强调，加粗显示---->>     
    <strong>Python爬虫</strong>     
    <br>     small标记，可以显示小一号字体，可以嵌套使用---->>     
    <small>Python爬虫</small>     
    <small><small>Python爬虫</small></small>     
    <small><small><small>Python爬虫</small></small></small>     
    <br>     big标记，显示大一号的字体---->>     
    <big>Python爬虫</big>
 <big><big>Python爬虫</big></big>     
 <br>     u标记是显示下划线---->>     
 <big><big><big><u>Python爬虫</u></big></big></big>     
 <br>     
 </body>     
 </html>
```

  <html>     <head>        <title>Python爬虫开发与项目实战</title>        <meta charset="UTF-8">     </head>     <body>     Hn标题标记---->>     <br>        <h1>Python爬虫</h1>        <h2>Python爬虫</h2>
        <h3>Python爬虫</h3>        <h4>Python爬虫</h4>        <h5>Python爬虫</h5>        <h6>Python爬虫</h6>     font标记---->>     <font size="1">Python爬虫</font>     <font size="3">Python爬虫</font>     <font size="7">Python爬虫</font>     <font size="7" color="red" face="微软雅黑">Python爬虫</font>     <font size="7" color="red" face="宋体">Python爬虫</font>     <font size="7" color="red" face="新细明体">Python爬虫</font>     <br>     B标记加粗---->>     <b>Python爬虫</b>     <br>     i标记斜体---->>     <i>Python爬虫</i>     <br>     sub下标标记---->>     2<sub>2</sub>     <br>     sup上标标记---->>     2<sup>2</sup>     <br>     引用标记---->>     <cite>Python爬虫</cite>     <br>     em标记表示强调，显示为斜体---->>     <em>Python爬虫</em>     <br>     strong标记表示强调，加粗显示---->>     <strong>Python爬虫</strong>     <br>     small标记，可以显示小一号字体，可以嵌套使用---->>     <small>Python爬虫</small>     <small><small>Python爬虫</small></small>     <small><small><small>Python爬虫</small></small></small>     <br>     big标记，显示大一号的字体---->>     <big>Python爬虫</big>
     <big><big>Python爬虫</big></big>     <br>     u标记是显示下划线---->>     <big><big><big><u>Python爬虫</u></big></big></big>     <br>     </body>     </html>



其中文本标记包括：

```bash
·<hn>：标题标记。共有6个级别，n的范围为1～6，不同级别对应不同显示大小 的标题，h1最大，h6最小。
·<font>：字体设置标记。用来设置字体的格式，一般有三个常用属性： size（字体大小），<font size=“14px”>；color（颜色），<font color=“red”>；face（字体），<font face=“微软雅黑”>。
·<b>：粗字体标记。
·<i>：斜字体标记。
·<sub>：文字下标字体标记。
·<sup>：文字上标字体标记。
·<tt>：打印机字体标记。
·<cite>：引用方式的字体，通常是斜体。
·<em>：表示强调，通常显示为斜体字。
·<strong>：表示强调，通常显示为粗体字。
·<small>：小型字体标记。
·<big>：大型字体标记。
·<u>：下划线字体标记。
```
### 2.3 图像标记

```bash
<img>称为图像标记，用来在网页中显示图像。使用方法为：<img src=“路
径/文件名.图片格式”width=“属性值”height=“属性值”border=“属性 值”alt=“属性值”>。<img>标记主要包括以下属性：
·src属性用来指定我们要加载的图片的路径、图片的名称以及图片格式。
·width属性用来指定图片的宽度，单位为px、em、cm、mm。
·height属性用来指定图片的高度，单位为px、em、cm、mm。
·border属性用来指定图片的边框宽度，单位为px、em、cm、mm。
```

·alt属性有三个作用：

 - 1.当网页上的图片被加载完成后，鼠标移动到上面去， 会显示这个图片指定的属性文字；
 - 2.如果图像没有下载或者加载失败，会用文字来 代替图像显示；
 - 3.搜索引擎可以通过这个属性的文字来抓取图片。

我们可以在浏览器上访问博客园首页，对博客园首页的图片进行审查，就可以 看到的img标记的使用方法，如图2-5所示。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200908173857881.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
注意：

```bash
<img>为单标记，不需要使用</img>闭合。在加载图像文件的时 候，文件的路径、文件名或者文件格式错误，将无法加载图片。
```
### 2.4 表格
表格的基本结构包括`<table>、<caption>、<tr>、<td>和<th>`等标记。
<table>标记的基本格式为

```python
<table属性1=“属性值1”属性2=“属性值 2”......>表格内容</table>
```

table标记有以下常见属性：

 - `width`属性：表示表格的宽度，它的值可以是像素（px）也可以是父级元素的 百分比（%）。
 - `height`属性：表示表格的高度，它的值可以是像素（px）也可以是父级元素的 百分比（%）。
 - `border`属性：表示表格外边框的宽度。
 - `align`属性用来表示表格的显示位置。left居左显示，center居中显示， right居右显示。
 - `cellspacing`属性：单元格之间的间距，默认是2px，单位为像素。
 - `cellpadding`属性：单元格内容与单元格边框的显示距离，单位为像素。
 - ·`frame`属性：用来控制表格边框最外层的四条线框。`void`（默认值）表示无边框；`above`表示仅顶部有边框；`below`表示仅有底部边框；`hsides`表示仅有顶部边框和底部边框；`lhs`表示仅有左侧边框；`rhs`表示仅有右侧边框；`vsides`表示仅有左右 侧边框；`border`表示包含全部4个边框。
 - `·rules`属性：用来控制是否显示以及如何显示单元格之间的分割线。属性值`none`（默认值）表示无分割线；`all`表示包括所有分割线；`rows`表示仅有行分割线；`clos`表示仅有列分割线；`groups`表示仅在行组和列组之间有分割线。

 `<caption>`**标记用于在表格中使用标题**。<caption>属性的插入位置，直接位 于<table>属性之后，<tr>表格行之前。`<caption>`标记中align属性可以取四个 值：

```python
top表示标题放在表格的上部；
bottom表示标题放在表格的下部；
left表示标 题放在表格的左部；
right表示标题放在表格的右部。
```

 
`<tr>`**标记用来定义表格的行**，对于每一个表格行，都是由一对<tr>...</tr>标 记表示，每一行<tr>标记内可以嵌套多个<td>或者<th>标记。<tr>标记中的常见属 性包括：

 - ·bgcolor属性用来设置背景颜色，格式为bgcolor=“颜色值”。

 - ·align属性用来设置垂直方向对齐方式，格式为align=“值”。值为bottom
   时，表示靠顶端对齐；值为top时，表示靠底部对齐；值为middle时，表示居中对 齐。

 - ·valign属性用来设置水平方向对齐方式，格式为valign=“值”。值为left
   时，表示靠左对齐；值为right时，表示靠右对齐；值为center时，表示居中对 齐。

`<td>和<th>`**都是单元格的标记**，其必须嵌套在<tr>标记内，成对出现。<th>是 表头标记，通常位于首行或者首列，<th>中的文字默认会被加粗，而<td>不会。 <td>是数据标记，表示该单元格的具体数据。<td>和<th>两者的标记属性都是一样 的，常用属性如下：

```python
·bgcolor设置单元格背景。
·align设置单元格对齐方式。
·valign设置单元格垂直对齐方式。
·width设置单元格宽度。
·height设置单元格高度。
·rowspan设置单元格所占行数。
·colspan设置单元格所占列数。
```
下面通过一个HTML文档来演示表格的使用，文档如下：

```python
 <html>     
 <head>       
 <title>学生信息表</title>        
 <meta charset="UTF-8">     
 </head>      
 <body>        
 <table width="960" align="center" border="1" rules="all" cellpadding="15">                
 <tr>                        
 <th>学号</th>                       
 <th>班级</th>                        
 <th>姓名</th>                        
 <th>年龄</th>
 <th>籍贯</th>                     
 </tr>                
  <tr align="center">                        
  <td>1500001</td>                        
  <td>(1)班</td>                        
   <td>张三</td>                        
   <td>16</td>                        
   <td>上海</td>                
   </tr>                
   <tr align="center">                        
   <td>1500011</td>                        
   <td>(2)班</td>                        
   <td>李四</td>                        
   <td>15</td>                        
   <td bgcolor="# ccc">浙江</td>                
   </tr>        
   </table>        
   <br/>        
   <table width="960" align="center" border="1" rules="all" cellpadding=        "15">                
   <tr bgcolor="# ccc">                        
   <th>学号</th>                        
   <th>班级</th>                        
   <th>姓名</th>                        
   <th>年龄</th>                        
   <th>籍贯</th>                
   </tr>                
   <tr align="center">                        
   <td>1500001</td>                        
   <td>(1)班</td>                        
   <td>张三</td>                        
   <td>16</td>                        
   <td bgcolor="red"><font color="white">上海</font></td>                
   </tr>                
   <tr align="center">                        
   <td>1500011</td>                        
   <td>(2)班</td>                        
   <td>李四</td>                        
   <td>15</td>
    <td>浙江</td>                
    </tr>        
    </table>
    </body>     
    </html>
```

 
  <html>     
     <head>       
     <title>学生信息表</title>        
     <meta charset="UTF-8">     
     </head>      
     <body>        
     <table width="960" align="center" border="1" rules="all" cellpadding="15">                
     <tr>                        
     <th>学号</th>                       
     <th>班级</th>                        
     <th>姓名</th>                        
     <th>年龄</th>
     <th>籍贯</th>                     
     </tr>                
      <tr align="center">                        
      <td>1500001</td>                        
      <td>(1)班</td>                        
       <td>张三</td>                        
       <td>16</td>                        
       <td>上海</td>                
       </tr>                
       <tr align="center">                        
       <td>1500011</td>                        
       <td>(2)班</td>                        
       <td>李四</td>                        
       <td>15</td>                        
       <td bgcolor="# ccc">浙江</td>                
       </tr>        
       </table>        
       <br/>        
       <table width="960" align="center" border="1" rules="all" cellpadding=        "15">                
       <tr bgcolor="# ccc">                        
       <th>学号</th>                        
       <th>班级</th>                        
       <th>姓名</th>                        
       <th>年龄</th>                        
       <th>籍贯</th>                
       </tr>                
       <tr align="center">                        
       <td>1500001</td>                        
       <td>(1)班</td>                        
       <td>张三</td>                        
       <td>16</td>                        
       <td bgcolor="red"><font color="white">上海</font></td>                
       </tr>                
       <tr align="center">                        
       <td>1500011</td>                        
       <td>(2)班</td>                        
       <td>李四</td>                        
       <td>15</td>
        <td>浙江</td>                
        </tr>        
        </table>
        </body>     
        </html>


相关链接：
 1. [html快速基础入门详解](https://ghostwritten.blog.csdn.net/article/details/108472621)
 2. [CSS快速入门详解](https://ghostwritten.blog.csdn.net/article/details/108743337)

