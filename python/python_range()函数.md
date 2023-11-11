

---
## 1. 打印文件名
```bash
for i in range(0, 3):
    m = f'name{i}'
    print(m)
```
输出：

```bash
name0
name1
name2
```
## 2. 打印数字
```python
for i in range(5,0,-1):
    print(i)
```
输出：
```bash
5
4
3
2
1
```
while方法
```python
i=5 
while(i > 0):
    print(i)
    i=i-1 #Decrementing 
```

## 3. 创建动态变量，并给动态变量赋值
实例1
```python
j = [10, 20, 30]
for i in range(0, 3):
    m = f'name{i}'
    exec(m + '= %s' % (j[i]))
 
print(name0)
print(name1)
print(name2)
```
输出结果：

```python
10
20
30
```

实例2
```python
value = [10, 20]
var = 'my_var'
expr = var + "= %s" % (value[1])
exec(expr)
print(my_var)
```
输出结果：

```python
20
```
✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>

 - [python 内置函数](https://blog.csdn.net/xixihahalelehehe/article/details/104913051)

![在这里插入图片描述](https://img-blog.csdnimg.cn/bbad393270e04448bc8f1eaa022785eb.gif#pic_center)

