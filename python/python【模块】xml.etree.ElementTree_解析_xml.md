#  pytho 模块 xml.etree.ElementTree 解析 xml



[
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/29a495c74bc1824d8a19f1ec79770fb9.png)](https://bladerunner.fandom.com/wiki/Blade_Runner_2049)


##  1. 什么是 XML？
XML 代表可扩展标记语言。它在外观上类似于HTML，但 XML 用于数据表示，而 HTML 用于定义正在使用的数据。XML 专门设计用于在客户端和服务器之间来回发送和接收数据。

XML 创建了一种易于解释并支持层次结构的树状结构。只要页面遵循 XML，就可以将其称为 XML 文档。

 - XML 文档具有称为元素的部分，由开始和结束标记定义。标签是一种以 开头`<和结尾的标记结构>`。开始标签和结束标签之间的字符（如果有的话）是元素的内容。元素可以包含标记，包括其他元素，称为“子元素”。
 - 最大的顶级元素称为root，它包含所有其他元素。
 - 属性是存在于起始标签或空元素标签中的名称-值对。XML 属性只能有一个值，并且每个属性在每个元素上最多只能出现一次。

XML 文件:

```bash
<?xml version="1.0"?>
<collection>
    <genre category="Action">
        <decade years="1980s">
            <movie favorite="True" title="Indiana Jones: The raiders of the lost Ark">
                <format multiple="No">DVD</format>
                <year>1981</year>
                <rating>PG</rating>
                <description>
                'Archaeologist and adventurer Indiana Jones 
                is hired by the U.S. government to find the Ark of the 
                Covenant before the Nazis.'
                </description>
            </movie>
               <movie favorite="True" title="THE KARATE KID">
               <format multiple="Yes">DVD,Online</format>
               <year>1984</year>
               <rating>PG</rating>
               <description>None provided.</description>
            </movie>
            <movie favorite="False" title="Back 2 the Future">
               <format multiple="False">Blu-ray</format>
               <year>1985</year>
               <rating>PG</rating>
               <description>Marty McFly</description>
            </movie>
        </decade>
        <decade years="1990s">
            <movie favorite="False" title="X-Men">
               <format multiple="Yes">dvd, digital</format>
               <year>2000</year>
               <rating>PG-13</rating>
               <description>Two mutants come to a private academy for their kind whose resident superhero team must 
               oppose a terrorist organization with similar powers.</description>
            </movie>
            <movie favorite="True" title="Batman Returns">
               <format multiple="No">VHS</format>
               <year>1992</year>
               <rating>PG13</rating>
               <description>NA.</description>
            </movie>
               <movie favorite="False" title="Reservoir Dogs">
               <format multiple="No">Online</format>
               <year>1992</year>
               <rating>R</rating>
               <description>WhAtEvER I Want!!!?!</description>
            </movie>
        </decade>    
    </genre>

    <genre category="Thriller">
        <decade years="1970s">
            <movie favorite="False" title="ALIEN">
                <format multiple="Yes">DVD</format>
                <year>1979</year>
                <rating>R</rating>
                <description>"""""""""</description>
            </movie>
        </decade>
        <decade years="1980s">
            <movie favorite="True" title="Ferris Bueller's Day Off">
                <format multiple="No">DVD</format>
                <year>1986</year>
                <rating>PG13</rating>
                <description>Funny movie about a funny guy</description>
            </movie>
            <movie favorite="FALSE" title="American Psycho">
                <format multiple="No">blue-ray</format>
                <year>2000</year>
                <rating>Unrated</rating>
                <description>psychopathic Bateman</description>
            </movie>
        </decade>
    </genre>
```
从您上面阅读的内容中，您可以看到

 - `<collection>`是单个根元素：它包含所有其他元素，例如`<genre>`、 或`<movie>`，它们是子元素或子元素。如您所见，这些元素是嵌套的。
 - 这些子元素也可以充当父元素并包含它们自己的子元素，这些子元素称为“子子元素”。例如，您会看到`<movie>`元素包含几个“属性”，例如favorite title提供更多信息！

##  2. ElementTree 简介
到目前为止，您所了解的 XML 解析器可以完成这项工作。但是，它们不太符合 Python 的哲学，这绝非偶然。虽然 DOM 遵循 W3C 规范，而 SAX 是根据 Java API 建模的，但两者都没有特别像 Python 的感觉。

更糟糕的是，DOM 和 SAX 解析器都感觉过时了，因为它们在[CPython](https://realpython.com/cpython-source-code-guide/)解释器中的一些代码已经超过 20 年没有改变了！在撰写本文时，它们的实现仍然不完整，并且[缺少 typeshed 存根](https://github.com/python/typeshed/issues/3787)，这会破坏代码[编辑器](https://realpython.com/python-ides-code-editors-guide/)中的代码完成。

同时，Python 2.5 带来了解析和编写 XML 文档的全新视角——`ElementTree API`。它是一个轻量级、高效、优雅且功能丰富的界面，甚至一些第三方库也以此为基础。要开始使用它，您必须导入`xml.etree.ElementTree`模块，这有点拗口。因此，习惯上这样定义别名：

```bash
import xml.etree.ElementTree as ET
```
您可以通过采用不同的解析策略来使用 ElementTree API：

||非增量	|增量（阻塞）	|增量（非阻塞）|
|--|--|--|--|
|ET.parse()|	✔️	||	
|ET.fromstring()|	✔️		|
|ET.iterparse()	|	|✔️	|
|ET.XMLPullParser|		||	✔️|

##  3. 解析 XML 数据
提供的 XML 文件中，有一个描述的电影的基本集合。唯一的问题是数据很乱！这个集合有很多不同的策展人，每个人都有自己的方式将数据输入文件。本教程的主要目标是使用 Python 阅读和理解文件 - 然后解决问题。

首先，您需要使用`ElementTree`.

```bash
tree = ET.parse('movies.xml')
root = tree.getroot()
```
现在您已经初始化了树，您应该查看 XML 并打印出值以了解树的结构。

树的每个部分（包括根）都有一个描述元素的标签。此外，正如您在介绍中看到的那样，元素可能具有属性，这些属性是附加的描述符，特别用于重复标记的使用。属性还有助于验证为该标记输入的值，再次有助于 XML 的结构化格式。

```bash
root.tag
输出：
'collection'
```
在顶层，您会看到此 XML 植根于collection标记中。

```bash
root.attrib
输出：
{}
```
所以根没有属性。
###  示例
xml:

```bash
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN"
  "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd" [
    <!ENTITY custom_entity "Hello">
]>
<svg xmlns="http://www.w3.org/2000/svg"
  xmlns:inkscape="http://www.inkscape.org/namespaces/inkscape"
  viewBox="-105 -100 210 270" width="210" height="270">
  <inkscape:custom x="42" inkscape:z="555">Some value</inkscape:custom>
  <defs>
    <linearGradient id="skin" x1="0" x2="0" y1="0" y2="1">
      <stop offset="0%" stop-color="yellow" stop-opacity="1.0"/>
      <stop offset="75%" stop-color="gold" stop-opacity="1.0"/>
      <stop offset="100%" stop-color="orange" stop-opacity="1"/>
    </linearGradient>
  </defs>
  <g id="smiley" inkscape:groupmode="layer" inkscape:label="Smiley">
    <!-- Head -->
    <circle cx="0" cy="0" r="50"
      fill="url(#skin)" stroke="orange" stroke-width="2"/>
    <!-- Eyes -->
    <ellipse cx="-20" cy="-10" rx="6" ry="8" fill="black" stroke="none"/>
    <ellipse cx="20" cy="-10" rx="6" ry="8" fill="black" stroke="none"/>
    <!-- Mouth -->
    <path d="M-20 20 A25 25 0 0 0 20 20"
      fill="white" stroke="black" stroke-width="3"/>
  </g>
  <text x="-40" y="75">&custom_entity; &lt;svg&gt;!</text>
  <script>
    <![CDATA[
      console.log("CDATA disables XML parsing: <svg>")
      const smiley = document.getElementById("smiley")
      const eyes = document.querySelectorAll("ellipse")
      const setRadius = r => e => eyes.forEach(x => x.setAttribute("ry", r))
      smiley.addEventListener("mouseenter", setRadius(2))
      smiley.addEventListener("mouseleave", setRadius(8))
    ]]>
  </script>
</svg>
```

```bash
>>> import xml.etree.ElementTree as ET
>>> tree = ET.parse('movies.xml')
>>> root = tree.getroot()
>>> element = root[0]
>>> element.tag
'{http://www.inkscape.org/namespaces/inkscape}custom'

>>> element.text
'Some value'

>>> element.attrib
{'x': '42', '{http://www.inkscape.org/namespaces/inkscape}z': '555'}

>>> element.get("x")
'42'
```

##  4. For 循环
通过使用简单的“for”循环，您可以轻松地迭代根中的子元素（通常称为“子元素”）。

```bash
for child in root:
    print(child.tag, child.attrib)
```
输出：

```bash
genre {'category': 'Action'}
genre {'category': 'Thriller'}
genre {'category': 'Comedy'}
```
现在你知道根的孩子`collection`都是`genre`. 为了指定类型，XML 使用属性`category`。`genre`根据元素，有`Action`, `Thriller`, and `Comedy` movies。

通常，了解整个树中的所有元素会很有帮助。一个有用的功能是`root.iter()`。您可以将此函数放入“for”循环中，它将遍历整个树。

```bash
[elem.tag for elem in root.iter()]
```
输出：

```bash
['collection',
 'genre',
 'decade',
 'movie',
 'format',
 'year',
 'rating',
 'description',
 'movie',
 'format',
 'year',
 'rating',
 'description',
 'movie',
 'format',
 'year',
 'rating',
 'description',
 'decade',
 'movie',
 'format',
 'year',
 'rating',
 'description',
 'movie',
 'format',
 'year',
 'rating',
 'description',
 'movie',
 'format',
 'year',
 'rating',
 'description',
 'genre',
 'decade',
 'movie',
 'format',
 'year',
 'rating',
 'description',
 'decade',
 'movie',
 'format',
 'year',
 'rating',
 'description',
 'movie',
 'format',
 'year',
 'rating',
 'description',
 'genre',
 'decade',
 'movie',
 'format',
 'year',
 'rating',
 'description',
 'decade',
 'movie',
 'format',
 'year',
 'rating',
 'description',
 'movie',
 'format',
 'year',
 'rating',
 'description',
 'decade',
 'movie',
 'format',
 'year',
 'rating',
 'description',
 'decade',
 'movie',
 'format',
 'year',
 'rating',
 'description']
```
这给出了您拥有多少个元素的一般概念，但它不显示树中的属性或级别。

有一种有用的方法可以查看整个文档。任何元素都有`.tostring()`方法。如果将根传递给`.tostring()`方法，则可以返回整个文档。在ElementTree（记住别名为ET）内，`.tostring()`采用一种稍微奇怪的形式。

由于ElementTree它是一个功能强大的库，不仅可以解释 XML，因此您必须将要显示的文档的编码和解码都指定为字符串。对于 XML，使用'`utf8`'- 这是 XML 的典型文档格式类型。

```bash
print(ET.tostring(root, encoding='utf8').decode('utf8'))
```
输出：

```bash
<?xml version='1.0' encoding='utf8'?>
<collection>
    <genre category="Action">
        <decade years="1980s">
            <movie favorite="True" title="Indiana Jones: The raiders of the lost Ark">
                <format multiple="No">DVD</format>
                <year>1981</year>
                <rating>PG</rating>
                <description>
                'Archaeologist and adventurer Indiana Jones 
                is hired by the U.S. government to find the Ark of the 
                Covenant before the Nazis.'
                </description>
            </movie>
               <movie favorite="True" title="THE KARATE KID">
               <format multiple="Yes">DVD,Online</format>
               <year>1984</year>
               <rating>PG</rating>
               <description>None provided.</description>
            </movie>
            <movie favorite="False" title="Back 2 the Future">
               <format multiple="False">Blu-ray</format>
               <year>1985</year>
               <rating>PG</rating>
               <description>Marty McFly</description>
            </movie>
        </decade>
        <decade years="1990s">
            <movie favorite="False" title="X-Men">
               <format multiple="Yes">dvd, digital</format>
               <year>2000</year>
               <rating>PG-13</rating>
               <description>Two mutants come to a private academy for their kind whose resident superhero team must 
               oppose a terrorist organization with similar powers.</description>
            </movie>
            <movie favorite="True" title="Batman Returns">
               <format multiple="No">VHS</format>
               <year>1992</year>
               <rating>PG13</rating>
               <description>NA.</description>
            </movie>
               <movie favorite="False" title="Reservoir Dogs">
               <format multiple="No">Online</format>
               <year>1992</year>
               <rating>R</rating>
               <description>WhAtEvER I Want!!!?!</description>
            </movie>
        </decade>    
    </genre>

    <genre category="Thriller">
        <decade years="1970s">
            <movie favorite="False" title="ALIEN">
                <format multiple="Yes">DVD</format>
                <year>1979</year>
                <rating>R</rating>
                <description>"""""""""</description>
            </movie>
        </decade>
        <decade years="1980s">
            <movie favorite="True" title="Ferris Bueller's Day Off">
                <format multiple="No">DVD</format>
                <year>1986</year>
                <rating>PG13</rating>
                <description>Funny movie about a funny guy</description>
            </movie>
            <movie favorite="FALSE" title="American Psycho">
                <format multiple="No">blue-ray</format>
                <year>2000</year>
                <rating>Unrated</rating>
                <description>psychopathic Bateman</description>
            </movie>
        </decade>
    </genre>

    <genre category="Comedy">
        <decade years="1960s">
            <movie favorite="False" title="Batman: The Movie">
                <format multiple="Yes">DVD,VHS</format>
                <year>1966</year>
                <rating>PG</rating>
                <description>What a joke!</description>
            </movie>
        </decade>
        <decade years="2010s">
            <movie favorite="True" title="Easy A">
                <format multiple="No">DVD</format>
                <year>2010</year>
                <rating>PG--13</rating>
                <description>Emma Stone = Hester Prynne</description>
            </movie>
            <movie favorite="True" title="Dinner for SCHMUCKS">
                <format multiple="Yes">DVD,digital,Netflix</format>
                <year>2011</year>
                <rating>Unrated</rating>
                <description>Tim (Rudd) is a rising executive
                 who “succeeds” in finding the perfect guest, 
                 IRS employee Barry (Carell), for his boss’ monthly event, 
                 a so-called “dinner for idiots,” which offers certain 
                 advantages to the exec who shows up with the biggest buffoon.
                 </description>
            </movie>
        </decade>
        <decade years="1980s">
            <movie favorite="False" title="Ghostbusters">
                <format multiple="No">Online,VHS</format>
                <year>1984</year>
                <rating>PG</rating>
                <description>Who ya gonna call?</description>
            </movie>
        </decade>
        <decade years="1990s">
            <movie favorite="True" title="Robin Hood: Prince of Thieves">
                <format multiple="No">Blu_Ray</format>
                <year>1991</year>
                <rating>Unknown</rating>
                <description>Robin Hood slaying</description>
            </movie>
        </decade>
    </genre>
</collection>
```
您可以扩展该`iter()`函数的使用，以帮助查找感兴趣的特定元素。`root.iter()`将列出根下与指定元素匹配的所有子元素。在这里，您将列出movie树中元素的所有属性：

```bash
for movie in root.iter('movie'):
    print(movie.attrib)
```
输出：

```bash
{'favorite': 'True', 'title': 'Indiana Jones: The raiders of the lost Ark'}
{'favorite': 'True', 'title': 'THE KARATE KID'}
{'favorite': 'False', 'title': 'Back 2 the Future'}
{'favorite': 'False', 'title': 'X-Men'}
{'favorite': 'True', 'title': 'Batman Returns'}
{'favorite': 'False', 'title': 'Reservoir Dogs'}
{'favorite': 'False', 'title': 'ALIEN'}
{'favorite': 'True', 'title': "Ferris Bueller's Day Off"}
{'favorite': 'FALSE', 'title': 'American Psycho'}
{'favorite': 'False', 'title': 'Batman: The Movie'}
{'favorite': 'True', 'title': 'Easy A'}
{'favorite': 'True', 'title': 'Dinner for SCHMUCKS'}
{'favorite': 'False', 'title': 'Ghostbusters'}
{'favorite': 'True', 'title': 'Robin Hood: Prince of Thieves'}
```
您已经可以看到如何movies以不同的方式输入。暂时不用担心，稍后您将有机会在本教程中修复其中一个错误。

##  5. XPath 表达式
多时候元素没有属性，它们只有文本内容。使用属性`.text`，您可以打印出此内容。

现在，打印出电影的所有描述。

```bash
for description in root.iter('description'):
    print(description.text)
```
输出：

```bash
     'Archaeologist and adventurer Indiana Jones 
                is hired by the U.S. government to find the Ark of the 
                Covenant before the Nazis.'

None provided.
Marty McFly
Two mutants come to a private academy for their kind whose resident superhero team must 
               oppose a terrorist organization with similar powers.
NA.
WhAtEvER I Want!!!?!
"""""""""
Funny movie about a funny guy
psychopathic Bateman
What a joke!
Emma Stone = Hester Prynne
Tim (Rudd) is a rising executive
                 who “succeeds” in finding the perfect guest, 
                 IRS employee Barry (Carell), for his boss’ monthly event, 
                 a so-called “dinner for idiots,” which offers certain 
                 advantages to the exec who shows up with the biggest buffoon.

Who ya gonna call?
Robin Hood slaying
```
打印出 XML 是有帮助的，但 XPath 是一种用于快速轻松地搜索 XML 的查询语言。XPath 代表 XML 路径语言，顾名思义，它使用“类似路径”的语法来识别和导航 XML 文档中的节点。

了解 XPath 对于扫描和填充 XML 至关重要。`ElementTree`有一个`.findall()`函数将遍历被引用元素的直接子元素。您可以使用 XPath 表达式来指定更有用的搜索。

在这里，您将在树中搜索 1992 年上映的电影：

```bash
for movie in root.findall("./genre/decade/movie/[year='1992']"):
    print(movie.attrib)
```
输出：

```bash
{'favorite': 'True', 'title': 'Batman Returns'}
{'favorite': 'False', 'title': 'Reservoir Dogs'}
```
该函数`.findall()`总是从指定的元素开始。这种类型的功能对于“查找和替换”来说非常强大。您甚至可以搜索属性！

现在，仅打印以多种格式（属性）可用的电影。

```bash
for movie in root.findall("./genre/decade/movie/format/[@multiple='Yes']"):
    print(movie.attrib)
```
输出：

```bash
{'multiple': 'Yes'}
{'multiple': 'Yes'}
{'multiple': 'Yes'}
{'multiple': 'Yes'}
{'multiple': 'Yes'}
```
集思广益，为什么在这种情况下 print 语句会返回multiple. 想想“for”循环是如何定义的。你能改写这个循环来打印出电影标题吗？在下面试试：

**提示：使用'...'inside XPath 返回当前元素的父元素。**

```bash
for movie in root.findall("./genre/decade/movie/format[@multiple='Yes']..."):
    print(movie.attrib)
```
输出：

```bash
{'favorite': 'True', 'title': 'THE KARATE KID'}
{'favorite': 'False', 'title': 'X-Men'}
{'favorite': 'False', 'title': 'ALIEN'}
{'favorite': 'False', 'title': 'Batman: The Movie'}
{'favorite': 'True', 'title': 'Dinner for SCHMUCKS'}
```

##  6. 修改 XML
早些时候，电影标题绝对是一团糟。现在，再次打印出来：

```bash
for movie in root.iter('movie'):
    print(movie.attrib)
```
修复 `Back 2 the Future` 中的“2”。这应该是一个查找和替换的问题。编写代码以查找标题“Back 2 the Future”并将其保存为变量：

```bash
b2tf = root.find("./genre/decade/movie[@title='Back 2 the Future']")
print(b2tf)
```
输出：

```bash
<Element 'movie' at 0x10ce00ef8>
```
请注意，使用该`.find()`方法会返回树的一个元素。很多时候，编辑元素内的内容更有用。

修改title`Back 2 the Future` 元素变量的属性为“`Back to the Future`”。然后，打印出变量的属性以查看您的更改。您可以通过访问元素的属性然后为其分配新值来轻松地做到这一点：

```bash
b2tf.attrib["title"] = "Back to the Future"
print(b2tf.attrib)
```
输出：

```bash
{'favorite': 'False', 'title': 'Back to the Future'}
```
将您的更改写回 `XML`，以便它们永久固定在文档中。再次打印您的电影属性以确保您的更改有效。使用`.write()`方法来做到这一点：

```bash
tree.write("movies.xml")

tree = ET.parse('movies.xml')
root = tree.getroot()

for movie in root.iter('movie'):
    print(movie.attrib)
```
输出：

```bash
{'favorite': 'True', 'title': 'Indiana Jones: The raiders of the lost Ark'}
{'favorite': 'True', 'title': 'THE KARATE KID'}
{'favorite': 'False', 'title': 'Back to the Future'}
{'favorite': 'False', 'title': 'X-Men'}
{'favorite': 'True', 'title': 'Batman Returns'}
{'favorite': 'False', 'title': 'Reservoir Dogs'}
{'favorite': 'False', 'title': 'ALIEN'}
{'favorite': 'True', 'title': "Ferris Bueller's Day Off"}
{'favorite': 'FALSE', 'title': 'American Psycho'}
{'favorite': 'False', 'title': 'Batman: The Movie'}
{'favorite': 'True', 'title': 'Easy A'}
{'favorite': 'True', 'title': 'Dinner for SCHMUCKS'}
{'favorite': 'False', 'title': 'Ghostbusters'}
{'favorite': 'True', 'title': 'Robin Hood: Prince of Thieves'}
```
##  7. 修复属性
该`multiple`属性在某些地方不正确。用于`ElementTree`根据影片进入的格式来修复指示符。首先，打印`format`属性和 `text`以查看需要修复的部分。

```bash
for form in root.findall("./genre/decade/movie/format"):
    print(form.attrib, form.text)
```
输出：

```bash
{'multiple': 'No'} DVD
{'multiple': 'Yes'} DVD,Online
{'multiple': 'False'} Blu-ray
{'multiple': 'Yes'} dvd, digital
{'multiple': 'No'} VHS
{'multiple': 'No'} Online
{'multiple': 'Yes'} DVD
{'multiple': 'No'} DVD
{'multiple': 'No'} blue-ray
{'multiple': 'Yes'} DVD,VHS
{'multiple': 'No'} DVD
{'multiple': 'Yes'} DVD,digital,Netflix
{'multiple': 'No'} Online,VHS
{'multiple': 'No'} Blu_Ray
```
在这个标签上需要做一些工作。

您可以使用正则表达式查找逗号 - 这将判断`multiple`属性应该是“是”还是“否”。`.set()`使用该方法可以轻松地添加和修改属性。

注意：re是 Python 的标准正则表达式解释器。如果您想了解更多关于正则表达式的信息，请考虑[本教程](https://www.datacamp.com/community/tutorials/python-regular-expression-tutorial)。

```bash
import re

for form in root.findall("./genre/decade/movie/format"):
    # Search for the commas in the format text
    match = re.search(',',form.text)
    if match:
        form.set('multiple','Yes')
    else:
        form.set('multiple','No')

# Write out the tree to the file again
tree.write("movies.xml")

tree = ET.parse('movies.xml')
root = tree.getroot()

for form in root.findall("./genre/decade/movie/format"):
    print(form.attrib, form.text)
```
输出：

```bash
{'multiple': 'No'} DVD
{'multiple': 'Yes'} DVD,Online
{'multiple': 'No'} Blu-ray
{'multiple': 'Yes'} dvd, digital
{'multiple': 'No'} VHS
{'multiple': 'No'} Online
{'multiple': 'No'} DVD
{'multiple': 'No'} DVD
{'multiple': 'No'} blue-ray
{'multiple': 'Yes'} DVD,VHS
{'multiple': 'No'} DVD
{'multiple': 'Yes'} DVD,digital,Netflix
{'multiple': 'Yes'} Online,VHS
{'multiple': 'No'} Blu_Ray
```
##  8. 移动元素
一些数据被放置在错误的十年中。使用您所了解的有关 XML 的知识ElementTree来查找和修复十年数据错误。

打印出整个文档中的decade标签和标签会很有用

```bash
for decade in root.findall("./genre/decade"):
    print(decade.attrib)
    for year in decade.findall("./movie/year"):
        print(year.text, '\n')
```
输出：

```bash
{'years': '1980s'}
1981 

1984 

1985 

{'years': '1990s'}
2000 

1992 

1992 

{'years': '1970s'}
1979 

{'years': '1980s'}
1986 

2000 

{'years': '1960s'}
1966 

{'years': '2010s'}
2010 

2011 

{'years': '1980s'}
1984 

{'years': '1990s'}
1991 
```
错误十年的两年是2000年代的电影。使用 XPath 表达式找出这些电影是什么。

```bash
for movie in root.findall("./genre/decade/movie/[year='2000']"):
    print(movie.attrib)
```
输出：

```bash
{'favorite': 'False', 'title': 'X-Men'}
{'favorite': 'FALSE', 'title': 'American Psycho'}
```
您必须在动作类型中添加一个新的十年标签，即 2000 年代，才能移动 X 战警数据。该`.SubElement()`方法可用于将此标记添加到 XML 的末尾。

```bash
action = root.find("./genre[@category='Action']")
new_dec = ET.SubElement(action, 'decade')
new_dec.attrib["years"] = '2000s'

print(ET.tostring(action, encoding='utf8').decode('utf8'))
```
输出：

```bash
<?xml version='1.0' encoding='utf8'?>
<genre category="Action">
        <decade years="1980s">
            <movie favorite="True" title="Indiana Jones: The raiders of the lost Ark">
                <format multiple="No">DVD</format>
                <year>1981</year>
                <rating>PG</rating>
                <description>
                'Archaeologist and adventurer Indiana Jones 
                is hired by the U.S. government to find the Ark of the 
                Covenant before the Nazis.'
                </description>
            </movie>
               <movie favorite="True" title="THE KARATE KID">
               <format multiple="Yes">DVD,Online</format>
               <year>1984</year>
               <rating>PG</rating>
               <description>None provided.</description>
            </movie>
            <movie favorite="False" title="Back to the Future">
               <format multiple="No">Blu-ray</format>
               <year>1985</year>
               <rating>PG</rating>
               <description>Marty McFly</description>
            </movie>
        </decade>
        <decade years="1990s">
            <movie favorite="False" title="X-Men">
               <format multiple="Yes">dvd, digital</format>
               <year>2000</year>
               <rating>PG-13</rating>
               <description>Two mutants come to a private academy for their kind whose resident superhero team must 
               oppose a terrorist organization with similar powers.</description>
            </movie>
            <movie favorite="True" title="Batman Returns">
               <format multiple="No">VHS</format>
               <year>1992</year>
               <rating>PG13</rating>
               <description>NA.</description>
            </movie>
               <movie favorite="False" title="Reservoir Dogs">
               <format multiple="No">Online</format>
               <year>1992</year>
               <rating>R</rating>
               <description>WhAtEvER I Want!!!?!</description>
            </movie>
        </decade>    
    <decade years="2000s" /></genre>
```
`.append()`现在，分别使用和将 `X-Men` 电影附加到 2000 年代并将其从 1990 年代删除`.remove()`。

```bash
xmen = root.find("./genre/decade/movie[@title='X-Men']")
dec2000s = root.find("./genre[@category='Action']/decade[@years='2000s']")
dec2000s.append(xmen)
dec1990s = root.find("./genre[@category='Action']/decade[@years='1990s']")
dec1990s.remove(xmen)

print(ET.tostring(action, encoding='utf8').decode('utf8'))
```
输出：

```bash
<?xml version='1.0' encoding='utf8'?>
<genre category="Action">
        <decade years="1980s">
            <movie favorite="True" title="Indiana Jones: The raiders of the lost Ark">
                <format multiple="No">DVD</format>
                <year>1981</year>
                <rating>PG</rating>
                <description>
                'Archaeologist and adventurer Indiana Jones 
                is hired by the U.S. government to find the Ark of the 
                Covenant before the Nazis.'
                </description>
            </movie>
               <movie favorite="True" title="THE KARATE KID">
               <format multiple="Yes">DVD,Online</format>
               <year>1984</year>
               <rating>PG</rating>
               <description>None provided.</description>
            </movie>
            <movie favorite="False" title="Back to the Future">
               <format multiple="No">Blu-ray</format>
               <year>1985</year>
               <rating>PG</rating>
               <description>Marty McFly</description>
            </movie>
        </decade>
        <decade years="1990s">
            <movie favorite="True" title="Batman Returns">
               <format multiple="No">VHS</format>
               <year>1992</year>
               <rating>PG13</rating>
               <description>NA.</description>
            </movie>
               <movie favorite="False" title="Reservoir Dogs">
               <format multiple="No">Online</format>
               <year>1992</year>
               <rating>R</rating>
               <description>WhAtEvER I Want!!!?!</description>
            </movie>
        </decade>    
    <decade years="2000s"><movie favorite="False" title="X-Men">
               <format multiple="Yes">dvd, digital</format>
               <year>2000</year>
               <rating>PG-13</rating>
               <description>Two mutants come to a private academy for their kind whose resident superhero team must 
               oppose a terrorist organization with similar powers.</description>
            </movie>
            </decade></genre>
```
##  9. 构建 XML 文档
很好，所以你基本上可以将整部电影移到新的十年。将更改保存回 XML。

```bash
tree.write("movies.xml")

tree = ET.parse('movies.xml')
root = tree.getroot()

print(ET.tostring(root, encoding='utf8').decode('utf8'))
```
输出：
```bash
<?xml version='1.0' encoding='utf8'?>
<collection>
    <genre category="Action">
        <decade years="1980s">
            <movie favorite="True" title="Indiana Jones: The raiders of the lost Ark">
                <format multiple="No">DVD</format>
                <year>1981</year>
                <rating>PG</rating>
                <description>
                'Archaeologist and adventurer Indiana Jones 
                is hired by the U.S. government to find the Ark of the 
                Covenant before the Nazis.'
                </description>
            </movie>
               <movie favorite="True" title="THE KARATE KID">
               <format multiple="Yes">DVD,Online</format>
               <year>1984</year>
               <rating>PG</rating>
               <description>None provided.</description>
            </movie>
            <movie favorite="False" title="Back to the Future">
               <format multiple="No">Blu-ray</format>
               <year>1985</year>
               <rating>PG</rating>
               <description>Marty McFly</description>
            </movie>
        </decade>
        <decade years="1990s">
            <movie favorite="True" title="Batman Returns">
               <format multiple="No">VHS</format>
               <year>1992</year>
               <rating>PG13</rating>
               <description>NA.</description>
            </movie>
               <movie favorite="False" title="Reservoir Dogs">
               <format multiple="No">Online</format>
               <year>1992</year>
               <rating>R</rating>
               <description>WhAtEvER I Want!!!?!</description>
            </movie>
        </decade>    
    <decade years="2000s"><movie favorite="False" title="X-Men">
               <format multiple="Yes">dvd, digital</format>
               <year>2000</year>
               <rating>PG-13</rating>
               <description>Two mutants come to a private academy for their kind whose resident superhero team must 
               oppose a terrorist organization with similar powers.</description>
            </movie>
            </decade></genre>

    <genre category="Thriller">
        <decade years="1970s">
            <movie favorite="False" title="ALIEN">
                <format multiple="No">DVD</format>
                <year>1979</year>
                <rating>R</rating>
                <description>"""""""""</description>
            </movie>
        </decade>
        <decade years="1980s">
            <movie favorite="True" title="Ferris Bueller's Day Off">
                <format multiple="No">DVD</format>
                <year>1986</year>
                <rating>PG13</rating>
                <description>Funny movie about a funny guy</description>
            </movie>
            <movie favorite="FALSE" title="American Psycho">
                <format multiple="No">blue-ray</format>
                <year>2000</year>
                <rating>Unrated</rating>
                <description>psychopathic Bateman</description>
            </movie>
        </decade>
    </genre>

    <genre category="Comedy">
        <decade years="1960s">
            <movie favorite="False" title="Batman: The Movie">
                <format multiple="Yes">DVD,VHS</format>
                <year>1966</year>
                <rating>PG</rating>
                <description>What a joke!</description>
            </movie>
        </decade>
        <decade years="2010s">
            <movie favorite="True" title="Easy A">
                <format multiple="No">DVD</format>
                <year>2010</year>
                <rating>PG--13</rating>
                <description>Emma Stone = Hester Prynne</description>
            </movie>
            <movie favorite="True" title="Dinner for SCHMUCKS">
                <format multiple="Yes">DVD,digital,Netflix</format>
                <year>2011</year>
                <rating>Unrated</rating>
                <description>Tim (Rudd) is a rising executive
                 who “succeeds” in finding the perfect guest, 
                 IRS employee Barry (Carell), for his boss’ monthly event, 
                 a so-called “dinner for idiots,” which offers certain 
                 advantages to the exec who shows up with the biggest buffoon.
                 </description>
            </movie>
        </decade>
        <decade years="1980s">
            <movie favorite="False" title="Ghostbusters">
                <format multiple="Yes">Online,VHS</format>
                <year>1984</year>
                <rating>PG</rating>
                <description>Who ya gonna call?</description>
            </movie>
        </decade>
        <decade years="1990s">
            <movie favorite="True" title="Robin Hood: Prince of Thieves">
                <format multiple="No">Blu_Ray</format>
                <year>1991</year>
                <rating>Unknown</rating>
                <description>Robin Hood slaying</description>
            </movie>
        </decade>
    </genre>
</collection>
```
##  10. 结论
关于 XML 和使用ElementTree.

标签构建树结构并指定应该在那里描述的值。使用智能结构可以轻松读取和写入 XML。标签总是需要左括号和右括号来显示父子关系。

属性进一步描述了如何验证标签或允许布尔指定。属性通常采用非常具体的值，以便 XML 解析器（和用户）可以使用这些属性来检查标记值。

ElementTree是一个重要的 Python 库，可让您解析和导航 XML 文档。使用ElementTree将 XML 文档分解为易于使用的树结构。如有疑问，请将其打印出来 ( print(ET.tostring(root, encoding='utf8').decode('utf8'))) - 使用这个有用的打印语句一次查看整个 XML 文档。它有助于检查何时从 XML 中编辑、添加或删除。

参考：

 - [Python XML Tutorial with ElementTree: Beginner's Guide](https://www.datacamp.com/tutorial/python-xml-elementtree)
 - [A Roadmap to XML Parsers in Python](https://realpython.com/python-xml-parser/#xmletreeelementtree-a-lightweight-pythonic-alternative)
 - [How to Parse and Modify XML in Python?](https://www.edureka.co/blog/python-xml-parser-tutorial/)


