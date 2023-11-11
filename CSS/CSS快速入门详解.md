

---
## 1. CSS简介
CSS指层叠样式表 （`Cascading Style Sheets`），用来定义如何显示HTML元 素，一般和HTML配合使用。CSS样式表的目的是为了解决内容与表现分离的问题， 即使同一个HTML文档也能表现出外观的多样化。在HTML中使用CSS样式的方式，一 般有三种做法：

 - ·内联样式表：CSS代码直接写在现有的HTML标记中，直接使用style属性改变 样式。例如，`<body style="background-color:green; margin:0; padding:0;"></body>`。
 - ·嵌入式样式表：CSS样式代码写在`<style type="text/css"></style>`标记之间，一般情况下嵌入式CSS样式写在<head></head>之间。
·
 - 外部样式表：CSS代码写一个单独的外部文件中，这个CSS样式文件以“.css”为扩展名，在<head>内（不是在<style>标记内）使用<link>标记将CSS样式文件链接到HTML文件内。例如，`<link rel="StyleSheet" type="text/css"href="style.css">`。

## 2. CSS选择器
CSS规则由两个主要的部分构成：**选择器，以及一条或多条声明**。选择器通常是 需要改变样式的HTML元素。每条声明由一个属性和一个值组成。属性 （property）是希望设置的样式属性（style attribute）。每个属性有一个值。 属性和值由冒号分开。例如：`h1{color：blue；font-size：12px}`。其中h1为选 择器，color和font-size是属性，blue和12px是属性值，这句话的意思是将h1标 记中的颜色设置为蓝色，字体大小为12px。根据选择器的定义方式，可以将样式表 的定义分成三种方式：

 - `HTML标记定义`：上面举的例子就是使用的这种方式。假如想修改`<p>...</p>`的样式，可以定义CSS：p{属性：属性值；属性1：属性值1}。p可以叫做选择器，定义了标记中内容所执行的样式。一个选择器可以控制若干个样式属性，他们之间 需要用英语的“；”隔开，最后一个可以不加“；”。
·
 - `ID选择器定义`：ID选择器可以为标有特定ID的HTML元素指定特定的样式。
   HTML元素以ID属性来设置ID选择器，CSS中ID选择器以“#”来定义。假如定义为`#word{text-align：center；color：red；}`，就将HTML中ID为word的元素设置 为居中，颜色为红色。
 - ·`class选择器定义`：class选择器用于描述一组元素的样式，class选择器有别
   于ID选择器，它可以在多个元素中使用。class选择器在HTML中以class属性表示，在CSS中，class选择器以一个点“.”号显示。例如，`.center{text-align：center；}`将所有拥有center类的HTML元素设为居中。当然也可以指定特定的HTML元素使用class，例如，p.center{text-align：center；}是对所有的p元素使用 class=“center”，让该元素的文本居中。
## 3. CSS常见的属性
介绍完选择器，接着说一下CSS中一些常见的属性。常见属性主要说明一下颜色 属性、字体属性、背景属性、文本属性和列表属性。
### 3.1 颜色属性
颜色属性color用来定义文本的颜色，可以使用以下方式定义颜色：
·颜色名称，如color：green。
·十六进制，如color：#ff6600。
·简写方式，如color：#f60。
·RGB方式，如rgb（255，255，255），红（R）、绿（G）、蓝（B）的取值范 围均为0～255
·RGBA方式，如color：rgba（255，255，255，1），RGBA表示Red（红色）、 Green（绿色）、Blue（蓝色）和Alpha的（色彩空间）透明度。

### 3.2 字体属性
可以使用字体属性定义文本形式，有如下方法：

 - ·`font-size`定义字体大小，如font-size：14px。
 - ·`font-family`定义字体，如font-family：微软雅黑，serif。字体之间可以使用“，”隔开，以确保当字体不存在的时候直接使用下一个字体。
 - ·`font-weight`定义字体加粗，取值有两种方式。一种是使用名称，如normal（默认值）、bold（粗）、bolder（更粗）、lighter（更细）；一种是使用数字，如100、200、300～900，400=normal，而700=bold。

### 3.3 背景属性
可以用背景属性定义背景颜色、背景图片、背景重复和背景的位置，内容如
下：

 - ·`background-color`用来定义背景的颜色，用法参考颜色属性。
 - ·`background-image`用来定义背景图片，如background-image：url（图片路径），也可以设置为background-image：none，表示不使用图片。
 - ·`background-repeat`用来定义背景重复方式。取值为repeat，表示整体重复平铺；取值为repeat-x，表示只在水平方向平铺；取值为repeat-y，表示只在垂直 方向平铺；取值为no-repeat，表示不重复。
 - ·`background-position`用来定义背景位置。在横向上，可以取left、center、right；在纵向上可以取top、center、bottom。

·简写方式可以简化背景属性的书写，同时定义多个属性，格式为background： 背景颜色url（图像）重复位置。如background：#f60url（images/bg、jpg）
no-repeat top center。


### 3.4 文本属性
可以用文本属性设置行高、缩进和字符间距，具体如下：

 - ·`text-align`设置文本对齐方式，属性值可以取left、center、right。
 - ·`line-height`设置文本行高，属性值可以取具体数值，来设置固定的行高值。 也可以取百分比，是基于字体大小的百分比行高。
 - ·`text-indent`代表首行缩进，如text-indent：50px，意思是首行缩进50个像 素。
 - ·`letter-spacing`用来设置字符间距。属性值默认是normal，规定字符间没有额外的空间；可以设置具体的数值（可以是负值），如letter-spacing：3px；可以取inherit，从父元素继承letter-spacing属性的值。

### 3.5 列表
在HTML中，有两种类型的列表：**无序和有序**。其实使用CSS，可以列出进一步 的样式，并可用图像作列表项标记。接下来主要讲解以下几种属性：

 - ·`list-style-type`用来指明列表项标记的类型。常用的属性值有：none（无标
   记）、disc（默认，标记是实心圆）、circle（标记是空心圆）、square（标记是实心方块）、decimal（标记是数字）、decimal-leading-zero（0开头的数字标记）、lower-roman（小写罗马数字i、ii、iii、iv、v等）、upper-roman（大写罗马数字I、II、III、IV、V等）、lower-alpha（小写英文字母a、b、c、d、e等）、upper-alpha（大写英文字母A、B、C、D、E等）。例如，`ul.a{liststyle-type：circle；}`是将class选择器的值为a的ul标记设置为空心圆标记。
 - ·`list-style-position`用来指明列表项中标记的位置。属性值可以取inside、outside和inherit。inside指的是列表项标记放置在文本以内，且环绕文本根据标记对齐。outside为默认值，保持标记位于文本的左侧，列表项标记放置在文本以外，且环绕文本不根据标记对齐。inherit规定应该从父元素继承liststyle-position属性的值。
 - ·`list-style-image`用来设置设置图像列表标记。属性值可以为URL（图像的路径）、none（默认无图形被显示）、inherit（从父元素继承list-style-image属性的值）。例如，`ul{list-style-image：url（‘image.gif’）；}`，意思是给ul标记前面的标记设置为image.gif图片。

## 4. 实例
接下来通过一个综合的例子将所有知 识点进行融合，采用嵌入式样式表的方式，HTML文档如下：

```bash
<!DOCTYPE html>     
<html>     
<head>     
<meta charset="utf-8">     
<title>Python爬虫开发</title>     
<style>     
h1     
{        
background-color:# 6495ed;/*--背景颜色--*/        
color:red;/* 字体颜色 */        
text-align:center;/* 文本居中 */        
font-size:40px;/* 字体大小*/     
}     
p     
{
        background-color:# e0ffff;        
        text-indent:50px;/* 首行缩进 */        
        font-family:"Times New Roman", Times, serif;/* 设置字体 */     
}     
p.ex {color:rgb(0,0,255);}     
div     
{        
background-color:# b0c4de;     
}          
ul.a {list-style-type:square;}     
ol.b {list-style-type:upper-roman;}     
ul.c{list-style-image:url('http://www.cnblogs.com/images/logo_small.gif');}     
</style>     
</head>          
<body>     
<h1>CSS background-color演示</h1>     
<div>     该文本插入在 div 元素中。     
<p>该段落有自己的背景颜色。</p>     
<p class="ex">这是一个类为"ex"的段落。这个文本是蓝色的。
</p>     我们仍然在同一个 div 中。     
</div>     <p>无序列表实例:</p>          
<ul class="a">        
<li>Coffee</li>        
<li>Tea</li>        
<li>Coca Cola</li>     
</ul>         
<p>有序列表实例:</p>     
<ol class="b">        
<li>Coffee</li>        
<li>Tea</li>        
<li>Coca Cola</li>     
</ol>          
<p>图片列表示例</p>
 <ul class="c">        
 <li>Coffee</li>        
 <li>Tea</li>        
 <li>Coca Cola</li>     
 </ul>     
 </body>     
 </html>
```
效果:

 
<!DOCTYPE html>     
<html>     
<head>     
<meta charset="utf-8">     
<title>Python爬虫开发</title>     
<style>     
h1     
{        
background-color:# 6495ed;/*--背景颜色--*/        
color:red;/* 字体颜色 */        
text-align:center;/* 文本居中 */        
font-size:40px;/* 字体大小*/     
}     
p     
{
        background-color:# e0ffff;        
        text-indent:50px;/* 首行缩进 */        
        font-family:"Times New Roman", Times, serif;/* 设置字体 */     
}     
p.ex {color:rgb(0,0,255);}     
div     
{        
background-color:# b0c4de;     
}          
ul.a {list-style-type:square;}     
ol.b {list-style-type:upper-roman;}     
ul.c{list-style-image:url('http://www.cnblogs.com/images/logo_small.gif');}     
</style>     
</head>          
<body>     
<h1>CSS background-color演示</h1>     
<div>     该文本插入在 div 元素中。     
<p>该段落有自己的背景颜色。</p>     
<p class="ex">这是一个类为"ex"的段落。这个文本是蓝色的。
</p>     我们仍然在同一个 div 中。     
</div>     <p>无序列表实例:</p>          
<ul class="a">        
<li>Coffee</li>        
<li>Tea</li>        
<li>Coca Cola</li>     
</ul>         
<p>有序列表实例:</p>     
<ol class="b">        
<li>Coffee</li>        
<li>Tea</li>        
<li>Coca Cola</li>     
</ol>          
<p>图片列表示例</p>
 <ul class="c">        
 <li>Coffee</li>        
 <li>Tea</li>        
 <li>Coca Cola</li>     
 </ul>     
 </body>     
 </html>

相关链接：

 1. [html快速基础入门详解](https://ghostwritten.blog.csdn.net/article/details/108472621)
 2. [CSS快速入门详解](https://ghostwritten.blog.csdn.net/article/details/108743337)
