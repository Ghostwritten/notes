#  python 模块 BeautifulSoup 抓取 Web 数据



----
互联网上海量的数据是任何研究领域或个人兴趣的丰富资源。为了有效地收集这些数据，您需要熟练掌握[网络爬虫](https://realpython.com/python-web-scraping-practical-introduction/)。
您将学习如何：

 - 使用[requests](https://ghostwritten.blog.csdn.net/article/details/108996025)、[Beautiful Soup](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id44)从 Web抓取和解析数据
 - 从头到尾遍历网络抓取管道
 - 构建一个从 Web 获取工作机会并在控制台中显示相关信息的脚本

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e80e67f6d44e97a591304bfa456cf018.gif#pic_center)


##  1. Beautiful Soup 简介
简单来说，[beautiful Soup](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/) 是 python 的一个库，最主要的功能是从网页抓取数据。官方解释如下：

`Beautiful Soup` 提供一些简单的、python 式的函数用来处理导航、搜索、修改分析树等功能。它是一个工具箱，通过解析文档为用户提供需要抓取的数据，因为简单，所以不需要多少代码就可以写出一个完整的应用程序。 Beautiful Soup 自动将输入文档转换为 Unicode 编码，输出文档转换为 utf-8 编码。你不需要考虑编码方式，除非文档没有指定一个编码方式，这时，Beautiful Soup 就不能自动识别编码方式了。然后，你仅仅需要说明一下原始编码方式就可以了。 Beautiful Soup 已成为和 lxml、html6lib 一样出色的 python 解释器，为用户灵活地提供不同的解析策略或强劲的速度。


##  2. Beautiful Soup 安装
Beautiful Soup 3 目前已经停止开发，推荐在现在的项目中使用 Beautiful Soup 4，不过它已经被移植到 BS4 了，也就是说导入时我们需要 import bs4 。所以这里我们用的版本是 Beautiful Soup 4.3.2 (简称 BS4)，另外据说 BS4 对 Python3 的支持不够好，不过我用的是 Python2.7.7，如果有小伙伴用的是 Python3 版本，可以考虑下载 BS3 版本。 可以利用 pip 或者 easy_install 来安装，以下两种方法均可。

```bash
easy_install beautifulsoup4
#或者
pip install beautifulsoup4
```


如果想安装[最新的版本](https://pypi.org/project/beautifulsoup4/)，请直接下载安装包来手动安装，下载完成之后解压 运行下面的命令即可完成安装。

```bash
sudo python setup.py install
```
然后需要安装 `lxml`

```bash
easy_install lxml
#或者
pip install lxml
#或者
sudo apt-get install libxml2-dev libxslt-dev python-dev
wget https://github.com/lxml/lxml/releases/download/lxml-4.8.0/lxml-4.8.0.tar.gz
tar -zxvf lxml-4.8.0.tar.gz
cd lxml-4.8.0/
python3 setup.py install

```
另一个可供选择的解析器是纯 Python 实现的 html5lib , html5lib 的解析方式与浏览器相同，可以选择下列方法来安装 html5lib:

```bash
easy_install html5lib
#or
pip install html5lib
```

## 3. 创建 Beautiful Soup 对象
解析器：

 - lxml
 - html.parser

### 3.1 第一种: html变量
prettify()获取，解析器html.parser
**demo.py**
```bash
from bs4 import BeautifulSoup


html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>
"""

#第一种：创建 beautifulsoup 对象
soup = BeautifulSoup(html,'html.parser')
#soup = BeautifulSoup(html,'lxml')

print(soup.prettify())
```
执行python3 demo.py,输出：

```bash
<html>
 <head>
  <title>
   The Dormouse's story
  </title>
 </head>
 <body>
  <p class="title" name="dromouse">
   <b>
    The Dormouse's story
   </b>
  </p>
  <p class="story">
   Once upon a time there were three little sisters; and their names were
   <a class="sister" href="http://example.com/elsie" id="link1">
    <!-- Elsie -->
   </a>
   ,
   <a class="sister" href="http://example.com/lacie" id="link2">
    Lacie
   </a>
   and
   <a class="sister" href="http://example.com/tillie" id="link3">
    Tillie
   </a>
   ;
and they lived at the bottom of a well.
  </p>
  <p class="story">
   ...
  </p>
 </body>
</html>
```
###  3.2 第二种 open
将html本地化index.html，创建 beautifulsoup 对象
```bash
#soup = BeautifulSoup(open('index.html'),'html.parser')
```

###  3.3 第三种 with 
**index2.html**
```bash
<!DOCTYPE html>
<html>
    <head>
        <title>Header</title>
        <meta charset="utf-8">                   
    </head>

    <body>
        <h2>Operating systems</h2>

        <ul id="mylist" style="width:150px">
            <li>Solaris</li>
            <li>FreeBSD</li>
            <li>Debian</li>                      
            <li>NetBSD</li>           
            <li>Windows</li>         
        </ul>

        <p>
          FreeBSD is an advanced computer operating system used to 
          power modern servers, desktops, and embedded platforms.
        </p>

        <p>
          Debian is a Unix-like computer operating system that is 
          composed entirely of free software.
        </p>        

    </body>    
</html>
```
**demo2.py**
```bash
with open("index.html", "r") as f:

    contents = f.read()

    soup = BeautifulSoup(contents, 'lxml')

    print(soup.h2)
    print(soup.head)
    print(soup.li)
    print("HTML: {0}, name: {1}, text: {2}".format(soup.h2, 
        soup.h2.name, soup.h2.text))


#<h2>Operating systems</h2>
#<head>
#<title>Header</title>
#<meta charset="utf-8"/>
#</head>
#<li>Solaris</li>
#HTML: <title>Header</title>, name: title, text: Header

```
###  3.4 第四种：网页抓取

```bash
#!/usr/bin/python3

from bs4 import BeautifulSoup
import requests as req

resp = req.get("http://www.something.com")

soup = BeautifulSoup(resp.text, 'lxml')

print(soup.title)
print(soup.title.text)
print(soup.title.parent)
```
执行：

```bash
$ python3 scraping.py 
<title>Something.</title>
Something.
<head><title>Something.</title></head>
```


##  4. Beautiful Soup获取tag
以demo.py为例

 - soup.title
 - soup.head
 - soup.name
 - soup.a
 - soup.p
 - soup.attrs
 - soup.h2
 - soup.li

```bash
print(soup.title)
#<title>The Dormouse's story</title>

print(soup.head)
#<head><title>The Dormouse's story</title></head>

print(soup.a)
#<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>

print(soup.p)
#<p class="title" name="dromouse"><b>The Dormouse's story</b></p>

print(soup.name)
print(soup.head.name)
#[document]
#head

print type(soup.name)
#<type 'unicode'>

print soup.attrs 
#{} 空字典



print(type(soup.a))
#<class 'bs4.element.Tag'>
print(soup.a)
#a 标签里的内容实际上是注释，但是如果我们利用 .string 来输出它的内容
print(soup.a.string)
print(type(soup.a.string))
#<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>
# Elsie 
#<class 'bs4.element.Comment'>


print(soup.name)
print(soup.head.name)
#[document]
#head



#获取p
print(soup.p.attrs)
#{'class': ['title'], 'name': 'dromouse'}

print(soup.p['class'])
#['title']

print(soup.p.get('class'))
#['title']

soup.p['class']="newClass"
print(soup.p)
#<p class="newClass" name="dromouse"><b>The Dormouse's story</b></p>

del soup.p['class']
print(soup.p)
#<p name="dromouse"><b>The Dormouse's story</b></p>

#获取标签内部的文字
```bash
print(soup.p.string)
#The Dormouse's story

print(type(soup.p.string))
#<class 'bs4.element.NavigableString'>
```

##  5. Beautiful Soup 遍历
### 5.1 recursiveChildGenerator()

demo2.py
```bash
#!/usr/bin/python3

from bs4 import BeautifulSoup

with open("index2.html", "r") as f:

    contents = f.read()

    soup = BeautifulSoup(contents, 'lxml')

    for child in soup.recursiveChildGenerator():

        if child.name:

            print(child.name)
```
执行：

```bash
$ python3  demo2.py
html
head
title
meta
body
h2
ul
li
li
li
li
li
p
p
```

###  5.2 children 子元素




**get_children.py**
```bash
!/usr/bin/python3

from bs4 import BeautifulSoup

with open("index2.html", "r") as f:

    contents = f.read()

    soup = BeautifulSoup(contents, 'lxml')

    root = soup.html

    print(root.head.contents)
    print(root.head.contents[0])
    print(root.head.children)

    root_childs = [e.name for e in root.children if e.name is not None]
    print(root_childs)

```
执行：

```bash
$ python3 demo3.py
['\n', <title>Header</title>, '\n', <meta charset="utf-8"/>, '\n']
<list_iterator object at 0x7f76282a5f98>
['head', 'body']
```

###  5.3 descendants 后继元素

```bash
#!/usr/bin/python3

from bs4 import BeautifulSoup

with open("index2.html", "r") as f:

    contents = f.read()

    soup = BeautifulSoup(contents, 'lxml')

    root = soup.body

    root_childs = [e.name for e in root.descendants if e.name is not None]
    print(root_childs)
```
执行

```bash
$ python3 get_descendants.py 
['h2', 'ul', 'li', 'li', 'li', 'li', 'li', 'p', 'p']
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/038de3be1684dfe9380ba3321b19d469.gif#pic_center)

##  6. find_all( name , attrs , recursive , text , **kwargs )

###  6.1 tag

`find_all ()` 方法搜索当前 tag 的所有 tag 子节点，并判断是否符合过滤器的条件 1）name 参数 name 参数可以查找所有名字为 name 的 tag, 字符串对象会被自动忽略掉 A. 传字符串 最简单的过滤器是字符串。在搜索方法中传入一个字符串参数，Beautiful Soup 会查找与字符串完整匹配的内容，下面的例子用于查找文档中所有的标签。

```bash
from bs4 import BeautifulSoup
import re

html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>
"""


soup = BeautifulSoup(html,'html.parser')

print(soup.find_all('b'))
print(soup.find_all('a'))
#[<b>The Dormouse's story</b>]
#[<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>, <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>, <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

```
传正则表达式 如果传入正则表达式作为参数，Beautiful Soup 会通过正则表达式的 match () 来匹配内容。下面例子中找出所有以 b 开头的标签，这表示 和标签都应该被找到

```bash
for tag in soup.find_all(re.compile("^b")):
    print(tag.name)
# body
# b
```
传列表 如果传入列表参数，Beautiful Soup 会将与列表中任一元素匹配的内容返回。

```bash
soup.find_all(["a", "b"])
# [<b>The Dormouse's story</b>,
#  <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```
传 True True 可以匹配任何值，下面代码查找到所有的 tag, 但是不会返回字符串节点

```bash
for tag in soup.find_all(True):
    print(tag.name)

#html
#head
#title
#body
#p
#b
#p
#a
#a
#a
#p

```
传方法 如果没有合适过滤器，那么还可以定义一个方法，方法只接受一个元素参数 [4] , 如果这个方法返回 True 表示当前元素匹配并且被找到，如果不是则反回 False 下面方法校验了当前元素，如果包含 class 属性却不包含 id 属性，那么将返回 True:

```bash
def has_class_but_no_id(tag):
    return tag.has_attr('class') and not tag.has_attr('id')
```
将这个方法作为参数传入 `find_all ()` 方法，将得到所有

```bash
soup.find_all(has_class_but_no_id)
# [<p class="title"><b>The Dormouse's story</b></p>,
#  <p class="story">Once upon a time there were...</p>,
#  <p class="story">...</p>]
```

### 6.2 keyword 参数

注意：如果一个指定名字的参数不是搜索内置的参数名，搜索时会把该参数当作指定名字 tag 的属性来搜索，如果包含一个名字为 id 的参数，Beautiful Soup 会搜索每个 tag 的”id” 属性。

```bash
print(soup.find_all(id='link2'))
#[<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```
如果传入 href 参数，Beautiful Soup 会搜索每个 tag 的”href” 属性

```bash
print(soup.find_all(href=re.compile("elsie")))
[<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>]
```
使用多个指定名字的参数可以同时过滤 tag 的多个属性

```bash
soup.find_all(href=re.compile("elsie"), id='link1')
#[<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>]
```
在这里我们想用 class 过滤，不过 class 是 python 的关键词，这怎么办？加个下划线就可以

```bash
print(soup.find_all("a", class_="sister"))
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```
有些 tag 属性在搜索不能使用，比如 `HTML5` 中的 data-* 属性

```bash
data_soup = BeautifulSoup('<div data-foo="value">foo!</div>')
print(data_soup.find_all(data-foo="value"))
# SyntaxError: keyword can't be an expression
```
但是可以通过 `find_all ()` 方法的 attrs 参数定义一个字典参数来搜索包含特殊属性的 `tag`

```bash
data_soup = BeautifulSoup('<div data-foo="value">foo!</div>', 'html.parser')
print(data_soup.find_all(attrs={"data-foo": "value"}))
#[<div data-foo="value">foo!</div>]
```
###  6.3 text 参数 
通过 text 参数可以搜搜文档中的字符串内容。与 name 参数的可选值一样，text 参数接受 字符串，正则表达式，列表，True

```bash
print(soup.find_all(text="Elsie"))
# [u'Elsie']

print(soup.find_all(text=["Tillie", "Elsie", "Lacie"]))
# [u'Elsie', u'Lacie', u'Tillie']

print(soup.find_all(text=re.compile("Dormouse")))
[u"The Dormouse's story", u"The Dormouse's story"]
```
limit 参数 `find_all ()` 方法返回全部的搜索结构，如果文档树很大那么搜索会很慢。如果我们不需要全部结果，可以使用 limit 参数限制返回结果的数量。效果与 SQL 中的 limit 关键字类似，当搜索到的结果数量达到 limit 的限制时，就停止搜索返回结果。文档树中有 3 个 tag 符合搜索条件，但结果只返回了 2 个，因为我们限制了返回数量。

```bash
print(soup.find_all("a", limit=2))
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```
`recursive` 参数 调用 tag 的 `find_all ()` 方法时，Beautiful Soup 会检索当前 tag 的所有子孙节点，如果只想搜索 tag 的直接子节点，可以使用参数 `recursive=False` . 一段简单的文档:

```bash
<html>
 <head>
  <title>
   The Dormouse's story
  </title>
 </head>
...
```
是否使用 `recursive` 参数的搜索结果:

```bash
print(soup.html.find_all("title"))
#[<title>The Dormouse's story</title>]
print(soup.html.find_all("title", recursive=False))
#[]
```

##  7. find( name , attrs , recursive , text , **kwargs )
它与 `find_all ()` 方法唯一的区别是 `find_all ()` 方法的返回结果是值包含一个元素的列表，而 find () 方法直接返回结果

```bash
#!/usr/bin/python3

from bs4 import BeautifulSoup

with open("index2.html", "r") as f:

    contents = f.read()

    soup = BeautifulSoup(contents, 'lxml')

    #print(soup.find("ul", attrs={ "id" : "mylist"}))
    print(soup.find("ul", id="mylist")) 
```
执行：

```bash
$ python3 demo4.py
<ul id="mylist" style="width:150px">
<li>Solaris</li>
<li>FreeBSD</li>
<li>Debian</li>
<li>NetBSD</li>
<li>Windows</li>
</ul>
```

##  8. BeautifulSoup CSS 选择器
我们在写 CSS 时，标签名不加任何修饰，类名前加点，id 名前加 #，在这里我们也可以利用类似的方法来筛选元素，用到的方法是 `soup.select()`，返回类型是 list

###  8.1 通过标签名查找

```bash
print(soup.select('title'))
#[<title>The Dormouse's story</title>]


print(soup.select('a'))
#[<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>, <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>, <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]


print(soup.select('b'))
#[<b>The Dormouse's story</b>]
```

###  8.2 通过类名查找

```bash
print(soup.select('.sister'))
##[<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>, <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>, <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```

###  8.3 通过 id 名查找

```bash
print(soup.select('#link1'))
#[<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>]
```
###  8.4 组合查找
组合查找即和写 class 文件时，标签名与类名、id 名进行的组合原理是一样的，例如查找 p 标签中，id 等于 link1 的内容，二者需要用空格分开

```bash
print(soup.select('p #link1'))
#[<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>]
```
直接子标签查找

```bash
print(soup.select("head > title"))
#[<title>The Dormouse's story</title>]
```
###  8.5 属性查找
查找时还可以加入属性元素，属性需要用中括号括起来，注意属性和标签属于同一节点，所以中间不能加空格，否则会无法匹配到。

```bash
print(soup.select('a[class="sister"]'))
#[<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>, <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>, <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

print(soup.select('a[href="http://example.com/elsie"]'))
#[<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>]
```
同样，属性仍然可以与上述查找方式组合，不在同一节点的空格隔开，同一节点的不加空格

```bash
print(soup.select('p a[href="http://example.com/elsie"]'))
#[<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>]
```
以上的 select 方法返回的结果都是列表形式，可以遍历形式输出，然后用 get_text () 方法来获取它的内容。

```bash
soup = BeautifulSoup(html, 'lxml')
print type(soup.select('title'))
print soup.select('title')[0].get_text()

for title in soup.select('title'):
    print title.get_text()
```

```bash
#!/usr/bin/python3

from bs4 import BeautifulSoup

with open("index2.html", "r") as f:

    contents = f.read()

    soup = BeautifulSoup(contents, 'lxml')

    print(soup.select("li:nth-of-type(3)"))

#[<li>Debian</li>]
```

## 9. BeautifulSoup 追加元素
append()方法将新标签附加到 HTML 文档。

append_tag.py

```bash
#!/usr/bin/python3

from bs4 import BeautifulSoup

with open("index.html", "r") as f:

    contents = f.read()

    soup = BeautifulSoup(contents, 'lxml')

    newtag = soup.new_tag('li')
    newtag.string='OpenBSD'

    ultag = soup.ul

    ultag.append(newtag)

    print(ultag.prettify()) 


```
输出

```bash
<ul id="mylist" style="width:150px">
 <li>
  Solaris
 </li>
 <li>
  FreeBSD
 </li>
 <li>
  Debian
 </li>
 <li>
  NetBSD
 </li>
 <li>
  Windows
 </li>
 <li>
  OpenBSD
 </li>
</ul>
```

## 10. BeautifulSoup 插入元素
`insert()`方法在指定位置插入标签。

insert_tag.py

```bash
#!/usr/bin/python3

from bs4 import BeautifulSoup

with open("index2.html", "r") as f:

    contents = f.read()

    soup = BeautifulSoup(contents, 'lxml')

    newtag = soup.new_tag('li')
    newtag.string='OpenBSD'

    ultag = soup.ul

    ultag.insert(2, newtag)

    print(ultag.prettify()) 
```
输出：

```bash
<ul id="mylist" style="width:150px">
 <li>
  Solaris
 </li>
 <li>
  OpenBSD
 </li>
 <li>
  FreeBSD
 </li>
 <li>
  Debian
 </li>
 <li>
  NetBSD
 </li>
 <li>
  Windows
 </li>
</ul>
```
##  11. BeautifulSoup 替换文字
replace_with()替换元素的文本。

replace_text.py

```bash
#!/usr/bin/python3

from bs4 import BeautifulSoup

with open("index2.html", "r") as f:

    contents = f.read()

    soup = BeautifulSoup(contents, 'lxml')

    tag = soup.find(text="Windows")
    tag.replace_with("OpenBSD")

    print(soup.ul.prettify()) 
```
输出

```bash
<ul id="mylist" style="width:150px">
 <li>
  Solaris
 </li>
 <li>
  FreeBSD
 </li>
 <li>
  Debian
 </li>
 <li>
  NetBSD
 </li>
 <li>
  OpenBSD
 </li>
</ul>
```

## 12. BeautifulSoup 删除元素
`decompose()`方法从树中删除标签并销毁它。

decompose_tag.py

```bash
#!/usr/bin/python3

from bs4 import BeautifulSoup

with open("index2.html", "r") as f:

    contents = f.read()

    soup = BeautifulSoup(contents, 'lxml')

    ptag2 = soup.select_one("p:nth-of-type(2)")

    ptag2.decompose()

    print(soup.body.prettify())
```
输出

```bash
<body>
 <h2>
  Operating systems
 </h2>
 <ul id="mylist" style="width:150px">
  <li>
   Solaris
  </li>
  <li>
   FreeBSD
  </li>
  <li>
   Debian
  </li>
  <li>
   NetBSD
  </li>
  <li>
   Windows
  </li>
 </ul>
 <p>
  FreeBSD is an advanced computer operating system used to 
          power modern servers, desktops, and embedded platforms.
 </p>
</body>
```

参考：

 - [python requests【1】处理url模块](https://blog.csdn.net/xixihahalelehehe/article/details/108996025?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164922239616781685346565%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164922239616781685346565&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-108996025.nonecase&utm_term=requests&spm=1018.2226.3001.4450)
 - [python：BeautifulSoup 模块使用指南](https://www.jianshu.com/p/2b783f7914c6)
 - [Python BeautifulSoup – find all class](https://www.geeksforgeeks.org/python-beautifulsoup-find-all-class/)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/215ec1060ed41297804d7b2dde7ba0c8.gif#pic_center)

