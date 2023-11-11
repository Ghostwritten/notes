

----
## 1. 简介
当template模块对模板文件进行渲染时，使用的就是jinja2模板引擎，jinja2本身就是基于python的模板引擎，所以，如果想要更加灵活的编辑模板文件，最好还要了解一些与jinja2有关的基本知识点

## 2. 语法

```c
{{      }}  ：用来装载表达式，比如变量、运算表达式、比较表达式等。

{%   %}   ：用来装载控制语句，比如 if 控制结构，for循环控制结构。

{#    #}   ：用来装载注释，模板文件被渲染后，注释不会包含在最终生成的文件中。
```


## 3. {{      }}方法
### 3.1 输出变量
```bash
# cat test.j2
test jinja2 variable
test {{ testvar1 }} test
```

```bash
$ ansible test70 -m template -e "testvar1=teststr" -a "src=test.j2 dest=/opt/test"
```
输出

```bash
$ cat /opt/test
test jinja2 variable
test teststr test
```
”{{  }}”中包含的就是一个变量，当模板被渲染后，变量的值被替换到了最终的配置文件中，当然，既然在模板中定义了变量，那么就要保证在渲染模板时，可以调用到对应的变量，否则就会报错。
### 3.2 比较表达式
模板文件内容如下：

```bash
# cat test.j2
jinja2 test
{{ 1 == 1 }}
{{ 2 != 2 }}
{{ 2 > 1 }}
{{ 2 >= 1 }}
{{ 2 < 1 }}
{{ 2 <= 1 }}
 
生成文件内容如下：
# cat test
jinja2 test
True
False
True
True
False
False
```
### 3.3 逻辑运算

```bash
模板文件内容
# cat test.j2
jinja2 test
{{ (2 > 1) or (1 > 2) }}
{{ (2 > 1) and (1 > 2) }}
 
{{ not true }}
{{ not True }}
{{ not false }}
{{ not False }}
 
 
生成文件内容
# cat test
jinja2 test
True
False
 
False
False
True
True
```
### 3.4 算数运算

```bash
模板文件内容
# cat test.j2
jinja2 test
{{ 3 + 2 }}
{{ 3 - 4 }}
{{ 3 * 5 }}
{{ 2 ** 3 }}
{{ 7 / 5 }}
{{ 7 // 5 }}
{{ 17 % 5 }}
 
生成文件内容
# cat test
jinja2 test
5
-1
15
8
1.4
1
2
```
### 3.5 成员运算

```bash
模板文件内容
# cat test.j2
jinja2 test
{{ 1 in [1,2,3,4] }}
{{ 1 not in [1,2,3,4] }}
 
生成文件内容
# cat test
jinja2 test
True
False
```
### 3.6 数据类型显示

```bash
模板文件内容如下：
# cat test.j2
jinja2 test
### str
{{ 'testString' }}
{{ "testString" }}
### num
{{ 15 }}
{{ 18.8 }}
### list
{{ ['Aa','Bb','Cc','Dd'] }}
{{ ['Aa','Bb','Cc','Dd'].1 }}
{{ ['Aa','Bb','Cc','Dd'][1] }}
### tuple
{{ ('Aa','Bb','Cc','Dd') }}
{{ ('Aa','Bb','Cc','Dd').0 }}
{{ ('Aa','Bb','Cc','Dd')[0] }}
### dic
{{ {'name':'bob','age':18} }}
{{ {'name':'bob','age':18}.name }}
{{ {'name':'bob','age':18}['name'] }}
### Boolean
{{ True }}
{{ true }}
{{ False }}
{{ false }}
 
 
生成文件内容如下：
# cat test
jinja2 test
### str
testString
testString
### num
15
18.8
### list
['Aa', 'Bb', 'Cc', 'Dd']
Bb
Bb
### tuple
('Aa', 'Bb', 'Cc', 'Dd')
Aa
Aa
### dic
{'age': 18, 'name': 'bob'}
bob
bob
### Boolean
True
True
False
False
```
从上述示例模板文件可以看出，字符串、数值、列表、元组、字典、布尔值等数据类型均可在”{{  }}”使用，但是，通常我们不会像上述示例那样使用它们，因为通常我们会通过变量将对应的数据传入，而不是将数据直接写在”{{  }}”中，即使直接将数据写在”{{  }}”中，也会配合其他表达式或者函数进行处理，所以，我们可以把上述模板文件的内容改为如下内容进行测试：

```bash
jinja2 test
{{ teststr }}
{{ testnum }}
{{ testlist[1] }}
{{ testlist1[1] }}
{{ testdic['name'] }}
```
测试playbook如下：

```bash
# cat temptest.yml
---
- hosts: test70
 remote_user: root
 gather_facts: no
 vars:
   teststr: 'tstr'
   testnum: 18
   testlist: ['aA','bB','cC']
   testlist1:
   - AA
   - BB
   - CC
   testdic:
     name: bob
     age: 18
 tasks:
 - template:
     src: /testdir/ansible/test.j2
     dest: /opt/test
```
终生成的文件如下

```bash
# cat test
jinja2 test
tstr
18
bB
BB
bob
```
### 3.7 过滤器upper运用

```bash
模板文件内容
# cat test.j2
jinja2 test
{{ 'abc' | upper }}
 
 
生成文件内容
# cat test
jinja2 test
ABC
```
### 3.8 过滤器lookup运用
模板文件内容如下

```bash
# cat /testdir/ansible/test.j2
jinja2 test
 
{{ lookup('file','/testdir/testfile') }}
 
{{ lookup('env','PATH') }}
 
test jinja2
 
```

 
ansible主机中的`testfile`内容如下

```bash
# cat /testdir/testfile
testfile in ansible
These are for testing purposes only
```

 
 
生成文件内容如下

```bash
# cat test
jinja2 test
 
testfile in ansible
These are for testing purposes only
 
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
 
test jinja2
```

### 3.9 tests属性判断
模板文件内容


```bash
# cat test.j2
jinja2 test
{{ testvar1 is defined }}
{{ testvar1 is undefined }}
{{ '/opt' is exists }}
{{ '/opt' is file }}
{{ '/opt' is directory }}
 
执行命令时传入变量
# ansible test70 -m template -e "testvar1=1 testvar2=2" -a "src=test.j2 dest=/opt/test"
 
生成文件内容
# cat test
jinja2 test
True
False
True
False
True
```
## 4. {#    #}方法
如果我们需要在模板文件中对某些配置进行注释，则可以将注释信息写入到`”{#   #}”`中
模板文件内容如下：

```bash
# cat test.j2
jinja2 test
{#这是一行注释信息#}
jinja2 test
{#
这是多行注释信息，
模板被渲染以后，
最终的文件中不会包含这些信息
#}
jinja2 test
```

 
 
生成文件内容如下：

```bash
# cat test
jinja2 test
jinja2 test
jinja2 test
```
## 5. {%    %}方法
在jinja2中，使用”{%  %}”对控制语句进行包含，比如”if”控制语句、”for”循环控制语句等 都需要包含在”{%  %}”中。

### 5.1 if....endif语法

```bash
{% if 条件 %}
...
...
...
{% endif %}
```
示例

```bash
# cat test.j2
jinja2 test
 
{% if testnum > 3 %}
greater than 3
{% endif %}
```

```bash
# cat temptest.yml
---
- hosts: test70
  remote_user: root
  gather_facts: no
  tasks:
  - template:
      src: /testdir/ansible/test.j2
      dest: /opt/test
    vars:
      testnum: 5
```
生成的文件内容如下：

```bash
# cat /opt/test
jinja2 test
 
greater than 3
```
### 5.2 if…else…endif方法
语法
```bash
{% if 条件 %}
...
{% else %}
...
{% endif %}
```
或者

```bash
{% if 条件一 %}
...
{% elif 条件二 %}
...
{% elif 条件N %}
...
{% endif %}
```
或者

```bash
{% if 条件一 %}
...
{% elif 条件N %}
...
{% else %}
...
{% endif %}
```
### 5.3 三元运算
语法：

```bash
 <do something> if <something is true> else <do something else>
```
示例
```bash
# cat test.j2
jinja2 test
{{ 'a' if 2>1 else 'b' }}
```

渲染后的文件内容如下

```bash
# cat /opt/test
jinja2 test
a
```
if表达式的含义为，如果2>1这个条件为真，则使用’a’，如果2>1这个条件不成立，则使用’b’，而2必定大于1，所以条件成立，最终使用’a

### 5.4 set设置变量

```bash
# cat test.j2
jinja2 test
{% set teststr='abc' %}
{{ teststr }}
```

```bash
# ansible test70 -m template -a "src=test.j2 dest=/opt/test"
```

最终生成的文件内容如下

```bash
# cat /opt/test
jinja2 test
abc
```
### 5.5 for循环
语法：

```bash
{% for 迭代变量 in 可迭代对象 %}
{{ 迭代变量 }}
{% endfor %}
```

#### 5.5.1  for循环列表 
```bash
# cat test.j2
jinja2 test
{% for i in [3,1,7,8,2] %}
{{ i }}
{% endfor %}
```

上例中我们直接在模板文件的for循环中定义了一个列表，执行如下命令，对模板进行渲染

```c
# ansible test70 -m template -a "src=test.j2 dest=/opt/test"
```

最终生成的文件内容如下：

```bash
# cat /opt/test
jinja2 test
3
1
7
8
2
```

从生成的内容可以看出，每次循环后都会自动换行，如果不想要换行，则可以使用如下语法

```bash
# cat test.j2
jinja2 test
{% for i in [3,1,7,8,2] -%}
{{ i }}
{%- endfor %}
```


在for的结束控制符”%}”之前添加了减号”-”

在endfor的开始控制符”{%”之后添加到了减号”-”

渲染上述模板，最终的生成效果如下：

```bash
# cat test
jinja2 test
31782
```
如上所示，列表中的每一项都没有换行，而是连在了一起显示，如果你觉得这样显示有些”拥挤”，则可以稍微改进一下上述模板，如下：

```bash
jinja2 test
{% for i in [3,1,7,8,2] -%}
{{ i }}{{ ' ' }}
{%- endfor %}
```

如上例所示，我们在循环每一项时，在每一项后面加入了一个空格字符串，所以，最终生成的效果如下：

```bash
# cat test
jinja2 test
3 1 7 8 2
```

其实，还有更加简洁的写法，就是将上述模板内容修改为如下内容：

```bash
# cat test.j2
jinja2 test
{% for i in [3,1,7,8,2] -%}
{{ i~' ' }}
{%- endfor %}
```
#### 5.5.2 for循环字典

```bash
# cat test.j2
jinja2 test
{% for key,val in {'name':'bob','age':18}.iteritems() %}
{{ key ~ ':' ~ val }}
{% endfor %}
```
在循环操作字典时，先使用iteritems函数对字典进行处理，然后使用key和val两个变量作为迭代变量，分别用于存放字典中键值对的”键”和”值”，所以，直接输出两个变量的值即可，key和val是我随意起的变量名，你可以自己定义这两个迭代变量的名称，而且，上例中的iteritems函数也可以替换成items函数，但是推荐使用iteritems函数，上例最终生成内容如下：

```bash
# cat test
jinja2 test
age:18
name:bob
```
#### 5.5.3 for循环输出次数
在使用for循环时，有一些内置的特殊变量可以使用，比如，如果我想要知道当前循环操作为整个循环的第几次操作，则可以借助”loop.index”特殊变量，示例如下：

```bash
# cat test.j2
jinja2 test
{% for i in [3,1,7,8,2] %}
{{ i ~ '----' ~ loop.index }}
{% endfor %}
```

最终生成文件内容如下：

```bash
# cat test
jinja2 test
3----1
1----2
7----3
8----4
2----5
```
除了内置特殊变量”loop.index”，还有一些其他的内置变量，它们的作用如下（此处先简单的进行介绍，之后会给出示例）：

```bash
loop.index   当前循环操作为整个循环的第几次循环，序号从1开始
loop.index0   当前循环操作为整个循环的第几次循环，序号从0开始
loop.revindex  当前循环操作距离整个循环结束还有几次，序号到1结束
loop.revindex0 当前循环操作距离整个循环结束还有几次，序号到0结束
loop.first    当操作可迭代对象中的第一个元素时，此变量的值为true
loop.last    当操作可迭代对象中的最后一个元素时，此变量的值为true
loop.length   可迭代对象的长度
loop.depth   当使用递归的循环时，当前迭代所在的递归中的层级，层级序号从1开始
loop.depth0   当使用递归的循环时，当前迭代所在的递归中的层级，层级序号从0开始
loop.cycle()  这是一个辅助函数，通过这个函数我们可以在指定的一些值中进行轮询取
```

值，具体参考之后的示例

> 注：我当前使用的ansible版本为2.7.0，此版本的ansible对应的jinja2模板引擎的版本为2.7.2，上述内置变量为jinja2的2.7.2版本中的内置变量，目前，较新的jinja2版本为2.10，在2.10版的jinja2中还可以使用loop.previtem、loop.nextitem等特殊内置变量。

 
#### 5.5.4 for与range搭配
如果你只是想单纯的对一段内容循环的生成指定的次数，则可以借助range函数完成，比如，循环3次

```bash
{% for i in range(3) %}
something
...
{% endfor %}
```

当然，range函数可以指定起始`数字、结束数字、步长`等，默认的起始数字为0，

```bash
{% for i in range(1,4,2) %}
  {{i}}
{% endfor %}
```

**上例表示从1开始，到4结束（不包括4），步长为2，也就是说只有1和3会输出。**

#### 5.5.5 for循环与if控制条件结合

默认情况下，模板中的for循环不能像其他语言中的 for循环那样使用`break`或者`continue`跳出循环，但是你可以在”for”循环中添加”if”过滤条件，以便符合条件时，循环才执行真正的操作，示例如下：

```bash
{% for i in [7,1,5,3,9] if i > 3 %}
  {{ i }}
{% endfor %}
```

上述 示例表示只有列表中的数字大于3时，才输出列表中的元素，刚才在介绍if表达式时，

你可能会问，我们在for循环中使用if判断控制语句进行判断不是也可以实现上述语法的效果吗？比如，使用如下示例的写法。

```bash
{% for i in [7,1,5,3,9] %}
  {% if i>3 %}
    {{ i }}
  {%endif%}
{% endfor %}
```

没错，如果仅仅是为了根据条件进行过滤，上述两种写法并没有什么不同，但是，如果你需要在循环中使用到loop.index这种计数变量时，两种写法则会有所区别，具体区别渲染如下模板内容后则会很明显的看出来：

```bash
{% for i in [7,1,5,3,9] if i>3 %}
{{ i ~'----'~ loop.index }}
{% endfor %}
 
{% for i in [7,1,5,3,9] %}
{% if i>3 %}
{{ i ~'----'~ loop.index}}
{% endif %}
{% endfor %}
```

最终生成的内容如下



```bash
# cat test
7----1
5----2
9----3
 
7----1
5----3
9----5
```

从上述结果可以看出，当使用if内联表达式时，如果不满足对应条件，则不会进入当次迭代，所以loop.index也不会进行计算，而当使用if控制语句进行判断时，其实已经进入了当次迭代，loop.index也已经进行了计算。

 

当for循环中使用了if内联表达式时，还可以与else控制语句结合使用，示例如下：

```bash
{% for i in [7,1,5,3,9] if i>10 %}
{{ i }}
{%else%}
no one is greater than 10
{% endfor %}
```

其实，当for循环中没有使用if内联表达式时，也可以使用else块，示例如下

```bash
{% for u in userlist %}
  {{ u.name }}
{%else%}
  no one
{% endfor %}
```

上例中，只有userlist列表为空时，才会渲染else块后的内容。

 

所以，综上所述，如果因序列为空或者有条件过滤了序列中的所有项目而没有执行循环时，你可以使用else渲染一个用于替换的块。

 
#### 5.5.6 for循环递归
for循环也支持递归操作，递归示例如下：

```bash
{% set dictionary={ 'name':'bob','son':{ 'name':'tom','son':{ 'name':'jerry' } } }  %}
 
{% for key,value in dictionary.iteritems() recursive %}
  {% if key == 'name' %}
    {% set fathername=value %}
  {% endif %}
 
  {% if key == 'son' %}
    {{ fathername ~"'s son is "~ value.name}}
    {{ loop( value.iteritems() ) }}
  {% endif %}
{% endfor %}
```

如上例所示，我们定义了一个字典变量，从字典中可以看出，bob的儿子是tom，tom的儿子是jerry，然后我们使用for循环操作了这个字典，如前文所示，我们在操作字典时，使用了iteritems函数，在for循环的末尾，我们添加了`recursive` 修饰符，当for循环中有recursive时，表示这个循环是一个递归的循环，当我们需要在for循环中进行递归时，只要在需要进行递归的地方调用loop函数即可，没错，如你所见，上例中的”`loop( value.iteritems() )`”即为调用递归的部分，由于value也是一个字典，所以需要使用iteritems函数进行处理。

 

渲染上述模板内容，最终效果如下

```bash
  bob's son is tom
       
  tom's son is jerry
```

 

上文中总结的loop.depth变量和loop.depth0变量此处就不进行示例了，你可以自己动手写一个递归实验一下。

 

刚才在总结与循环有关的内置变量时，还提到了一个辅助函数，它就是”`loop.cycle()`”，它能够让我们在指定的一些值中进行轮询取值，这样说可能不够直观，不如来看一个小示例，如下：

```bash
{% set userlist=['Naruto','Kakashi','Sasuke','Sakura','Lee','Gaara','Itachi']  %}
 
{% for u in userlist %}
{{ u ~'----'~ loop.cycle('team1','team2','team3')}}
{%endfor%}
```

上例中，我们定义了一个用户列表，这个列表里面有一些人，现在，我想要将这些人分组，按照顺序将这些人轮流分配到三个组中，直到分完为止，三个组的组名分别为team1、team2、team3，渲染上例的内容，最终生成内容如下：

```bash
Naruto----team1
Kakashi----team2
Sasuke----team3
Sakura----team1
Lee----team2
Gaara----team3
Itachi----team1
```

从生成的内容可以看出，用户与三个组已经轮询的进行了结合。

 
#### 5.5.7 for与break、continue结合
刚才我们提到过，默认情况下，模板中的for循环无法使用break和continue，不过jinja2支持一些扩展，如果我们在ansible中启用这些扩展，则可以让模板中的for循环支持break和continue，方法如下：

如果想要开启对应的扩展支持，需要修改ansible的配置文件/`etc/ansible/ansible.cfg`，默认情况下未启用jinja2的扩展，如果想要启用jinja2扩展，则需要设置`jinja2_extension`选项，这个设置项默认情况下是注释的，我的默认设置如下

```bash
#jinja2_extensions = jinja2.ext.do,jinja2.ext.i18n
```

把注释符去掉，默认已经有两个扩展了，如果想要支持break和continue，则需要添加一个`loopcontrols`扩展，最终配置如下

```bash
jinja2_extensions = jinja2.ext.do,jinja2.ext.i18n,jinja2.ext.loopcontrols
```

完成上述配置步骤即可在for循环中使用break和continue控制语句，与其他语言一样，break表示结束整个循环，continue表示结束当次循环，示例如下：

```bash
{% for i in [7,1,5,3,9] %}
  {% if loop.index is even %}
    {%continue%}
  {%endif%}
  {{ i ~'----'~ loop.index }}
{% endfor %}
```

上述示例表示偶数次的循环将会跳过，even这个tests在前文中已经总结过，此处不再赘述。

break的示例如下：

```bash
{% for i in [7,1,5,3,9] %}
  {% if loop.index > 3 %}
    {%break%}
  {%endif%}
  {{i ~'---'~ loop.index}}
{% endfor %}
```

上例表示3次迭代以后的元素不会被处理

 
#### 5.5.8 for与do结合
如果我们想要在jinja2中修改列表中的内容，则需要借助jinja2的另一个扩展，这个扩展的名字就是”do”，细心如你肯定已经发现了，刚才修改`jinja2_extensions`配置的时候，默认就有这个扩展，它的名字是jinja2.ext.do，通过do扩展修改列表的示例如下：

```bash
{% set testlist=[3,5] %}
 
{% for i in testlist  %}
  {{i}}
{% endfor %}
 
{%do testlist.append(7)%}
 
{% for i in testlist  %}
  {{i}}
{% endfor %}
```

如上例所示，我们先定义了一个列表，然后遍历了这个列表，使用 do在列表的末尾添加了一个元素，数字7，然后又遍历 了它。


参考链接：
[https://www.zsythink.net/archives/2999](https://www.zsythink.net/archives/2999https://www.zsythink.net/archives/2999)
