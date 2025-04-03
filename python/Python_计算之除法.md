

----
##  1. 除法 /
```bash
a,b = 95,20
c = a/b
print('a=',a,'b=',b,'c=',c)
```

运行结果：

```python
a= 95 b= 20 c= 4.75
```
##  2. 四舍五入round()
`round()`的第2个位置参数表示取小数点后的保留位数，缺省值为0：

```python
a,b = 95,20
c = round(a/b)
print('a=',a,'b=',b,'c=',c)
c = round(a/b,1)
print('a=',a,'b=',b,'c=',c)
a,b = 81,20
c = round(a/b)
print('a=',a,'b=',b,'c=',c)
a,b = 81,20
c = round(a/b,1)
print('a=',a,'b=',b,'c=',c)
```
运行结果：

```python
a= 95 b= 20 c= 5 
a= 95 b= 20 c= 4.8
a= 81 b= 20 c= 4 
a= 81 b= 20 c= 4.0
```
##  3. 浮点数取整int()
`int()`方法可以看做是对float类型的数值做“类型转换”，去掉小数部分向下取整，只取整数部分：

```python
a,b = 95,20
c = int(a/b)
print('a=',a,'b=',b,'c=',c)
```
运行结果：

```powershell
a= 95 b= 20 c= 4
```
##  4. 地板除 //

```python
a,b = 95,20
c = a//b
print('a=',a,'b=',b,'c=',c)
```
运行结果：

```python
a= 95 b= 20 c= 4
```
##  5. 向上取整math.ceil()
利用math模块的ceil()方法向上取整，比如4.1取整为5：

```python
import math
a,b = 95,20
c = math.ceil(a/b)
print('a=',a,'b=',b,'c=',c)
a,b = 81,20
c = math.ceil(a/b)
print('a=',a,'b=',b,'c=',c)
```

运行结果：

```python
a= 95 b= 20 c= 5
a= 95 b= 20 c= 5
```
##  6. 取小数和整数部分math.modf()
返回一个二元组，下标0是小数部分，下标1为整数部分。

```python
import math
a,b = 95,20
c = math.modf(a/b)
print('a=',a,'b=',b,'c=',c)
```

运行结果：

```python
a= 95 b= 20 c= (0.75, 4.0) 
```

✈<font color=	#FF4500 size=4 style="font-family:Courier New">参考阅读：</font>

 - [Python除法：四舍五入，地板除，取整，取小数](http://www.juzicode.com/python-note-divide/)
 - [python 内置函数](https://blog.csdn.net/xixihahalelehehe/article/details/104913051)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4b9a14d867d96996fabb019528b11a12.gif#pic_center)

