#  Shell 数组



------

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fb61b26323d1e67b574c4db47e302d91.gif#pic_center)


## 1. 简介
数组中可以存放多个值。Bash Shell 只支持一维数组（不支持多维数组），初始化时不需要定义数组大小（与 PHP 类似）。

与大部分编程语言类似，数组元素的下标由 0 开始。

Shell 数组用括号来表示，元素用"空格"符号分割开，语法格式如下：

```bash
array_name=(value1 value2 ... valuen)
```
## 2. 创建数组

```bash
my_array=(A B "C" D)
```

```bash
array_name[0]=value0
array_name[1]=value1
array_name[2]=value2
```

一次性赋值

```bash
ARRAY=(value1 value2 ... valueN)

# 等同于

ARRAY=(
  value1
  value2
  value3
)
```
也可以指定位置赋值

```bash
$ array=(a b c)
$ array=([2]=c [0]=a [1]=b)

$ days=(Sun Mon Tue Wed Thu Fri Sat)
$ days=([0]=Sun [1]=Mon [2]=Tue [3]=Wed [4]=Thu [5]=Fri [6]=Sat)
```
单独某些位置指定也可以。

```bash
names=(hatter [5]=duchess alice)
```
上面例子中，`hatter`是数组的0号位置，`duchess`是5号位置，`alice`是6号位置。

没有赋值的数组元素的默认值是空字符串。

定义数组的时候，可以使用通配符。

```bash
$ mp3s=( *.mp3 )
```

上面例子中，将当前目录的所有 MP3 文件，放进一个数组。

先用declare -a命令声明一个数组，也是可以的。

```bash
$ declare -a ARRAYNAME
```

read -a命令则是将用户的命令行输入，存入一个数组。

```bash
$ read -a dice
```

上面命令将用户的命令行输入，存入数组dice。

## 3. 读取数组

### 3.1 读取单个元素
```bash
#!/bin/bash

my_array=(A B "C" D)

echo "第一个元素为: ${my_array[0]}"
echo "第二个元素为: ${my_array[1]}"
echo "第三个元素为: ${my_array[2]}"
echo "第四个元素为: ${my_array[3]}"
```
执行脚本，输出结果如下所示：

```bash
$ chmod +x test.sh 
$ ./test.sh
第一个元素为: A
第二个元素为: B
第三个元素为: C
第四个元素为: D
```
### 3.2 读取所有元素
使用@ 或 * 可以获取数组中的所有元素，例如：

```bash
#!/bin/bash

my_array[0]=A
my_array[1]=B
my_array[2]=C
my_array[3]=D

echo "数组的元素为: ${my_array[*]}"
echo "数组的元素为: ${my_array[@]}"
```

执行脚本，输出结果如下所示：

```bash
$ chmod +x test.sh 
$ ./test.sh
数组的元素为: A B C D
数组的元素为: A B C D
```

for循环遍历

```bash
foo=(a b c d e f)
for i in "${foo[@]}"; do
  echo $i
done
```

`@`和`*`放不放在双引号之中，是有差别的。

```bash
$ activities=( swimming "water skiing" canoeing "white-water rafting" surfing )
$ for act in ${activities[@]}; \
do \
echo "Activity: $act"; \
done

Activity: swimming
Activity: water
Activity: skiing
Activity: canoeing
Activity: white-water
Activity: rafting
Activity: surfing
```

上面的例子中，数组`activities`实际包含5个成员，但是`for...in`循环直接遍历`${activities[@]}`，导致返回7个结果。为了避免这种情况，一般把`${activities[@]}`放在双引号之中。

```bash
$ for act in "${activities[@]}"; \
do \
echo "Activity: $act"; \
done

Activity: swimming
Activity: water skiing
Activity: canoeing
Activity: white-water rafting
Activity: surfing
```

上面例子中，`${activities[@]}`放在双引号之中，遍历就会返回正确的结果。

`${activities[*]}`不放在双引号之中，跟`${activities[@]}`不放在双引号之中是一样的。

```bash
$ for act in ${activities[*]}; \
do \
echo "Activity: $act"; \
done

Activity: swimming
Activity: water
Activity: skiing
Activity: canoeing
Activity: white-water
Activity: rafting
Activity: surfing
```

`${activities[*]}`放在双引号之中，所有成员就会变成单个字符串返回。

```bash
$ for act in "${activities[*]}"; \
do \
echo "Activity: $act"; \
done

Activity: swimming water skiing canoeing white-water rafting surfing
```

所以，拷贝一个数组的最方便方法，就是写成下面这样。

```bash
$ hobbies=( "${activities[@]}" )
```

上面例子中，数组activities被拷贝给了另一个数组hobbies。

这种写法也可以用来为新数组添加成员。

```bash
$ hobbies=( "${activities[@]}" diving )
```
上面例子中，新数组hobbies在数组activities的所有成员之后，又添加了一个成员。

##  4 默认位置
如果读取数组成员时，没有读取指定哪一个位置的成员，默认使用0号位置。

```bash
$ declare -a foo
$ foo=A
$ echo ${foo[0]}
A
```

上面例子中，foo是一个数组，赋值的时候不指定位置，实际上是给foo[0]赋值。

引用一个不带下标的数组变量，则引用的是0号位置的数组元素。

```bash
$ foo=(a b c d e f)
$ echo ${foo}
a
$ echo $foo
a
```
上面例子中，引用数组元素的时候，没有指定位置，结果返回的是0号位置。

## 5. 获取数组的长度
获取数组长度的方法与获取字符串长度的方法相同，例如：

```bash
#!/bin/bash


my_array[0]=A
my_array[1]=B
my_array[2]=C
my_array[3]=D

echo "数组元素个数为: ${#my_array[*]}"
echo "数组元素个数为: ${#my_array[@]}"
```

执行脚本，输出结果如下所示：

```bash
$ chmod +x test.sh 
$ ./test.sh
数组元素个数为: 4
数组元素个数为: 4
```
当读取具体数组成员

```bash
$ a[100]=foo

$ echo ${#a[*]}
1

$ echo ${#a[@]}
1
```

上面例子中，把字符串赋值给100位置的数组元素，这时的数组只有一个元素。

> [!NOTE|style:flat|lable:Mylable|iconVisibility:hidden]
> 如果用这种语法去读取具体的数组成员，就会返回该成员的字符串长度。这一点必须小心。

```bash
$ a[100]=foo
$ echo ${#a[100]}
3
```

上面例子中，${#a[100]}实际上是返回数组第100号成员a[100]的值（foo）的字符串长度。

## 6. 提取数组序号
`${!array[@]}`或`${!array[*]}`，可以返回数组的成员序号，即哪些位置是有值的。

```bash
$ arr=([5]=a [9]=b [23]=c)
$ echo ${!arr[@]}
5 9 23
$ echo ${!arr[*]}
5 9 23
```

上面例子中，数组的5、9、23号位置有值。

利用这个语法，也可以通过for循环遍历数组。

```bash
arr=(a b c d)

for i in ${!arr[@]};do
  echo ${arr[i]}
done
```

## 7. 提取数组成员
`${array[@]:position:length}`的语法可以提取数组成员。

```bash
$ food=( apples bananas cucumbers dates eggs fajitas grapes )
$ echo ${food[@]:1:1}
bananas
$ echo ${food[@]:1:3}
bananas cucumbers dates
```

上面例子中，`${food[@]:1:1}`返回从数组1号位置开始的1个成员，`${food[@]:1:3}`返回从1号位置开始的3个成员。

如果省略长度参数length，则返回从指定位置开始的所有成员。

```csharp
$ echo ${food[@]:4}
eggs fajitas grapes
```

上面例子返回从4号位置开始到结束的所有成员。

## 8. 追加数组成员
数组末尾追加成员，可以使用+=赋值运算符。它能够自动地把值追加到数组末尾。否则，就需要知道数组的最大序号，比较麻烦。

```bash
$ foo=(a b c)
$ echo ${foo[@]}
a b c

$ foo+=(d e f)
$ echo ${foo[@]}
a b c d e f
```

## 9. 删除数组
删除一个数组成员，使用`unset`命令。

```bash
$ foo=(a b c d e f)
$ echo ${foo[@]}
a b c d e f

$ unset foo[2]
$ echo ${foo[@]}
a b d e f
```

上面例子中，删除了数组中的第三个元素，下标为2。

将某个成员设为空值，可以从返回值中“隐藏”这个成员。

```bash
$ foo=(a b c d e f)
$ foo[1]=''
$ echo ${foo[@]}
a c d e f
```

上面例子中，将数组的第二个成员设为空字符串，数组的返回值中，这个成员就“隐藏”了。

> [!NOTE|style:flat|lable:Mylable|iconVisibility:hidden]
> 这里是“隐藏”，而不是删除，因为这个成员仍然存在，只是值变成了空值。

```bash
$ foo=(a b c d e f)
$ foo[1]=''
$ echo ${#foo[@]}
6
$ echo ${!foo[@]}
0 1 2 3 4 5
```

上面代码中，第二个成员设为空值后，数组仍然包含6个成员。

由于空值就是空字符串，所以下面这样写也有隐藏效果，但是不建议这种写法。

```bash
$ foo[1]=
```

上面的写法也相当于“隐藏”了数组的第二个成员。

直接将数组变量赋值为空字符串，相当于“隐藏”数组的第一个成员。

```bash
$ foo=(a b c d e f)
$ foo=''
$ echo ${foo[@]}
b c d e f
```

上面的写法相当于“隐藏”了数组的第一个成员。

`unset ArrayName`可以清空整个数组。

```bash
$ unset ARRAY

$ echo ${ARRAY[*]}
<--no output-->
```

## 10. 关联数组
Bash 的新版本支持关联数组。关联数组使用字符串而不是整数作为数组索引。

`declare -A`可以声明关联数组。

```bash
declare -A colors
colors["red"]="#ff0000"
colors["green"]="#00ff00"
colors["blue"]="#0000ff"
```

关联数组必须用带有`-A`选项的declare命令声明创建。相比之下，整数索引的数组，可以直接使用变量名创建数组，关联数组就不行。

访问关联数组成员的方式，几乎与整数索引数组相同。

```bash
echo ${colors["blue"]}
```


参考：

 - [Unix / Linux - Using Shell Arrays](https://www.tutorialspoint.com/unix/unix-using-arrays.htm)
 - [Arrays in unix shell?](https://stackoverflow.com/questions/1878882/arrays-in-unix-shell)
 - [Array Basics in Shell Scripting | Set 1](https://www.geeksforgeeks.org/array-basics-shell-scripting-set-1/)
 - [阮一峰老师的bash 数组](https://wangdoc.com/bash/array.html)





