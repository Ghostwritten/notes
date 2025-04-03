


---
##  字符串截取
```bash
#!/usr/bin/python
#---utf-8---
temp="ABCDEFGHIJKLMNOPQRSTUVWXYZ"

#访问单个元素
print(temp[0]) #A
print(temp[-1]) #Z

#普通切片操作
print(temp[0:10])   #ABCDEFGHIJ
#隔着取
print(temp[0:10:2])    #ACEGI

#输出从第一个到倒数第N个数
print(temp[0:-2])     #ABCDEFGHIJKLMNOPQRSTUVWX
print(temp[:-2])    #ABCDEFGHIJKLMNOPQRSTUVWX
print(temp[-2:])     #YZ

#逆序输出
print(temp[::-1])    #ZYXWVUTSRQPONMLKJIHGFEDCBA

#循环迭代输出
for s in temp:
    print(s,end=' ' )    #A B C D E F G H I J K L M N O P Q R S T U V W X Y Z 
```

##  字符串去空格

### 内置方法

 - `lstrip`：删除左边的空格

这个字符串方法，会删除字符串s开始位置前的空格。

```bash
>>> s.lstrip()
'string   '
```

 - `rstrip`：删除右连的空格

这个内置方法可以删除字符串末尾的所有空格，看下面演示代码：

```bash
>>> s.rstrip()
'    string'
```

 - `strip`：删除两端的空格

有的时候我们读取文件中的内容，每行2边都有空格，能不能一次性全部去掉呢，字符符有一个内置的strip()方法可以做到。

```bash
>>> s = “   这是一个字符串    ”
>>> s.strip()
'string'
```

### 一般方法

 - 使用字符串函数`replace`

```bash
>>> a = 'hello world'
>>> a.replace(' ', '')
'helloworld'
```

 - 使用字符串函数`split`

```bash
>>> a = ''.join(a.split())
>>> print(a)
helloworld
```

 - 使用正则表达式

```bash
>>> import re
>>> strinfo = re.compile()
>>> strinfo = re.compile(' ')
>>> b = strinfo.sub('', a)
>>> print(b)
helloworld
```


##  字符串转列表

 - [python中list与string的转换](https://blog.csdn.net/bufengzj/article/details/90231555)

---

参考链接：

 - [https://blog.csdn.net/haoaiqian/article/details/70228177](https://blog.csdn.net/haoaiqian/article/details/70228177)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c7552c5de8eed410a40f900566539c1e.gif#pic_center)

