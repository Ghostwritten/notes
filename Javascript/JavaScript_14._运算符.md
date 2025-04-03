#  JavaScript 14. 运算符


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c1ee811755f5fa88ff585aa7e88ee801.png)


##  1. 算术运算符
与/或值之间的算术运算。
y=5，下面的表格解释了这些算术运算符：
| 运算符   | 描述     | 例子    | x 运算结果 | y 运算结果 |
|-------|--------|-------|--------|--------|
| +     | 加法     | x=y+2 | 7      | 5     
| -     | 减法     | x=y-2 | 3      | 5    
| *     | 乘法     | x=y*2 | 10     | 5     
| /     | 除法     | x=y/2 | 2.5    | 5   
| %     | 取模（余数） | x=y%2 | 1      | 5      | 
| ++    | 自增     | x=++y | 6      | 6      | 
++| 自增| x=y++ | 5      | 6     | 
| --    | 自减     | x=--y | 4      | 4      |
--|自减| x=y-- | 5      | 4     | 

##  2. 赋值运算符
赋值运算符用于给 JavaScript 变量赋值。

给定 x=10 和 y=5，下面的表格解释了赋值运算符：
| 运算符 | 例子   | 等同于   | 运算结果 | 
|-----|------|-------|------|
| =   | x=y  |       | x=5  | 
| +=  | x+=y | x=x+y | x=15 | 
| -=  | x-=y | x=x-y | x=5  | 
| *=  | x*=y | x=x*y | x=50 | 
| /=  | x/=y | x=x/y | x=2  | 
| %=  | x%=y | x=x%y | x=0  | 




##  3. 比较运算符
比较运算符在逻辑语句中使用，以测定变量或值是否相等。

x=5，下面的表格解释了比较运算符：
| 运算符   | 描述                         | 比较      | 返回值   | 实例   |
|-------|----------------------------|---------|-------|------|
| ==    | 等于                         | x==8    | false | 实例 » |
| x==5  | true                       | 实例 »    |       |      |
| ===   | 绝对等于（值和类型均相等）              | x==="5" | false | 实例 » |
| ===   | 绝对等于（值和类型均相等） | x===5 | true                       | 实例 »    |       |      |
| !=    |  不等于                       | x!=8    | true  | 实例 » |
| !==   |  不绝对等于（值和类型有一个不相等，或两个都不相等） | x!=="5" | true  | 实例 » |
| !==   |  不绝对等于（值和类型有一个不相等，或两个都不相等| x!==5 | false                      | 实例 »    |       |      |
| >     |  大于                        | x>8     | false | 实例 » |
| <     |  小于                        | x<8     | true  | 实例 » |
| >=    |  大于或等于                     | x>=8    | false | 实例 » |
| <=    |  小于或等于                     | x<=8    | true  | 实例 » |

实例：

```bash
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<p>x=5, 返回 x==8 的比较值结果。</p>
<button onclick="myFunction()">尝试一下</button>
<p id="demo"></p>
<script>
function myFunction()
{
	var x=5;
	document.getElementById("demo").innerHTML=x==8;
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0d4289c064fdf5935519188d389df7b6.png)

 如何使用
可以在条件语句中使用比较运算符对值进行比较，然后根据结果来采取行动：

```bash
if (age<18) x="Too young";
```

##  4. 逻辑运算符
逻辑运算符用于测定变量或值之间的逻辑。

给定 `x=6` 以及 `y=3`，下表解释了逻辑运算符：
| 运算符 | 描述  | 例子                       |
|-----|-----|--------------------------|
| &&  | and | (x < 10 && y > 1) 为 true |
|\|\|  | or  | (x=\=5 \|\| y==5) 为 false   |
| !   | not | !(x==y) 为 true           |

##  5. 条件运算符
JavaScript 还包含了基于某些条件对变量进行赋值的条件运算符。

语法

```bash
variablename=(condition)?value1:value2 
```
实例：

```bash
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>

<p>点击按钮检测年龄。</p>
年龄:<input id="age" value="18" />
<p>是否达到投票年龄?</p>
<button onclick="myFunction()">点击按钮</button>
<p id="demo"></p>
<script>
function myFunction()
{
	var age,voteable;
	age=document.getElementById("age").value;
	voteable=(age<18)?"年龄太小":"年龄已达到";
	document.getElementById("demo").innerHTML=voteable;
}
</script>

</body>
</html>
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3e9b9b3791534f54c6662250fd14c98c.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/37b0c0d641125a2ad0245fbedc5c07a3.png)
##  6. 用于字符串的 + 运算符

```bash
txt1="What a very";
txt2="nice day";
txt3=txt1+txt2;
输出：
What a verynice day
```

```bash
txt1="What a very ";
txt2="nice day";
txt3=txt1+txt2;
在以上语句执行后，变量 txt3包含的值是：
What a very nice day
```

```bash
txt1="What a very";
txt2="nice day";
txt3=txt1+" "+txt2;
在以上语句执行后，变量txt3 包含的值是：

What a very nice day
```
##  7. 对字符串和数字进行加法运算
```bash
x=5+5;
y="5"+5;
z="Hello"+5;
x,y, 和 z 输出结果为:

10
55
Hello5
```
