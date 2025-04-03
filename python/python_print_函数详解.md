#  python print 函数



----

## 1. 语法
以下是 print() 方法的语法:

```bash
print(*objects, sep=' ', end='\n', file=sys.stdout, flush=False)
```
参数

 - objects -- 复数，表示可以一次输出多个对象。输出多个对象时，需要用 , 分隔。
 - sep -- 用来间隔多个对象，默认值是一个空格。
 - end -- 用来设定以什么结尾。默认值是换行符 \n，我们可以换成其他字符串。
 - file -- 要写入的文件对象。
 - flush -- 输出是否被缓存通常决定于 file，但如果 flush 关键字参数为 True，流会被强制刷新。

## 2. 实战
### 2.1 输出不同格式字符串
```bash
>>>print(1)  
1  
>>> print("Hello World")  
Hello World  
 
>>> a = 1
>>> b = 'runoob'
>>> print(a,b)
1 runoob
 
>>> print("aaa""bbb")
aaabbb
>>> print("aaa","bbb")
aaa bbb
>>> 
 
>>> print("www","runoob","com",sep=".")  # 设置间隔符
www.runoob.com
```

### 2.2 使用 flush 参数生成一个 Loading 的效果

```bash
import time

print("---RUNOOB EXAMPLE ： Loading 效果---")

print("Loading",end = "")
for i in range(20):
    print(".",end = '',flush = True)
    time.sleep(0.5)
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/28bcd7399011eaf7a88cd753ab8f8b36.png)
### 2.3 输出各类数据类型

```bash
>>>print("runoob")  # 输出字符串
runoob 
>>> print(100)            # 输出数字
100
>>> str = 'runoob'
>>> print(str)            # 输出变量
runoob
>>> L = [1,2,'a']         # 列表 
>>> print(L)  
[1, 2, 'a']  
>>> t = (1,2,'a')         # 元组
>>> print(t)  
(1, 2, 'a')  
>>> d = {'a':1, 'b':2}    # 字典
>>> print(d)  
{'a': 1, 'b': 2}
```
### 2.4  输出格式化字符串
python字符串格式化符号:

```bash
    符   号	 描述
      %c	 格式化字符及其ASCII码
      %s	 格式化字符串
      %d	 格式化整数
      %u	 格式化无符号整型
      %o	 格式化无符号八进制数
      %x	 格式化无符号十六进制数
      %X	 格式化无符号十六进制数（大写）
      %f	 格式化浮点数字，可指定小数点后的精度
      %e	 用科学计数法格式化浮点数
      %E	 作用同%e，用科学计数法格式化浮点数
      %g	 %f和%e的简写
      %G	 %f 和 %E 的简写
      %p	 用十六进制数格式化变量的地址
```
格式化操作符辅助指令:

```bash
符号	功能
*	    定义宽度或者小数点精度
-	    用做左对齐
+	    在正数前面显示加号( + )
<sp>	在正数前面显示空格
#	    在八进制数前面显示零('0')，在十六进制前面显示'0x'或者'0X'(取决于用的是'x'还是'X')
0	    显示的数字前面填充'0'而不是默认的空格
%	   '%%'输出一个单一的'%'
(var)	映射变量(字典参数)
m.n.	m 是显示的最小总宽度,n 是小数点后的位数(如果可用的话)
```

```bash
>>>str = "the length of (%s) is %d" %('runoob',len('runoob'))
>>> print(str)
the length of (runoob) is 6
```
### 2.5 自动换行与不换行
print 会自动在行末加上回车, 如果不需回车，只需在 print 语句的结尾添加一个逗号 , 并设置分隔符参数 end，就可以改变它的行为。

```bash
>>>for i in range(0,6):
...     print(i)
... 
0
1
2
3
4
5
>>> for i in range(0,6):
...     print(i, end=" ")
... 
0 1 2 3 4 5
```

