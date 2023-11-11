#  python 模块 configParser 配置文件



-----
## 1. 简介
[configparser](https://pypi.org/project/configparser/)是Pyhton标准库中用来解析配置文件的模块，并且内置方法和字典非常接近。Python2.x 中名为 ConfigParser，3.x 已更名小写，并加入了一些新功能。
## 2. 安装

```bash
pip install configparser
```

## 3. 方法
### 3.1 基本的读取配置文件

```bash
-read(filename) 直接读取ini文件内容
-sections() 得到所有的section，并以列表的形式返回
-options(section) 得到该section的所有option
-items(section) 得到该section的所有键值对
-get(section,option) 得到section中option的值，返回为string类型
-getint(section,option) 得到section中option的值，返回为int类型，还有相应的getboolean()和getfloat() 函数。
```

### 3.2 基本的写入配置文件

```bash
-add_section(section) 添加一个新的section
-set( section, option, value) 对section中的option进行设置，需要调用write将内容写入配置文件
```
## 4. 示例
测试配置文件test.conf内容如下：


```bash
[first]
w = 2
v: 3
c =11-3
[second]
sw=4
test: hello
```

测试配置文件中有两个区域，first和second，另外故意添加一些空格、换行。
下面解析：

```bash
>>> import ConfigParser
>>> conf=ConfigParser.ConfigParser()
>>> conf.read('test.conf')
['test.conf']
>>> conf.sections()   #获得所有区域
['first', 'second']
>>> for sn in conf.sections():
...     print conf.options(sn)       #打印出每个区域的所有属性
...
['w', 'v', 'c']
['sw', 'test']
```

获得每个区域的属性值：

```bash
for sn in conf.sections():
    print sn,'-->'
    for attr in conf.options(sn):
        print attr,'=',conf.get(sn,attr)
```

输出：

```bash
first -->
w = 2
v = 3
c = 11-3
second -->
sw = 4
test = hello
```

好了，以上就是基本的使用过程，下面是动态的写入配置,
复制代码 代码如下:

```bash
import ConfigParser
cfd=open('test2.ini','w')
conf=ConfigParser.ConfigParser()
conf.add_section('test')         #add a section
conf.set('test','run','false')  
conf.set('test','set',1)
conf.write(cfd)
cfd.close()
```

上面是向test2.ini写入配置信息

```bash
$ cat test2.ini 
[test]
run = false
set = 1
```
## 5. 实战
**test.conf**

```bash
[sec_a] 
a_key1 = 20 
a_key2 = 10 

   
[sec_b] 
b_key1 = 121 
b_key2 = b_value2 
b_key3 = $r 
b_key4 = 127.0.0.1 
```
**test2.py**

```bash
import ConfigParser
 
cf = ConfigParser.ConfigParser()
 
#read config
cf.read("test.conf")
 
# return all section
secs = cf.sections()
print 'sections:', secs
 
opts = cf.options("sec_a")
print 'options:', opts
 
kvs = cf.items("sec_a")
print 'sec_a:', kvs
 
#read by type
str_val = cf.get("sec_a", "a_key1")
int_val = cf.getint("sec_a", "a_key2")
 
print "value for sec_a's a_key1:", str_val
print "value for sec_a's a_key2:", int_val
 
#write config
#update value
cf.set("sec_b", "b_key3", "new-$r")
#set a new value
cf.set("sec_b", "b_newkey", "new-value")
#create a new section
cf.add_section('a_new_section')
cf.set('a_new_section', 'new_key', 'new_value')
 
#write back to configure file
cf.write(open("test.conf", "w"))
```

```bash
$ python test2.py
sections: ['sec_a', 'sec_b']
options: ['a_key1', 'a_key2']
sec_a: [('a_key1', '20'), ('a_key2', '10')]
value for sec_a's a_key1: 20
value for sec_a's a_key2: 10
```
参考：

 - [ConfigParser – Work with configuration files](http://pymotw.com/2/ConfigParser/)
 - [configparser --- 配置文件解析器](https://docs.python.org/zh-cn/3.7/library/configparser.html)
 - [Zedcode Python ConfigParser](https://zetcode.com/python/configparser/)
 - [极客 Python ConfigParser 教程](https://geek-docs.com/python/python-tutorial/python-configparser.html)

