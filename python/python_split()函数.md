

---
## 1. 简介

Python中有split()和os.path.split()两个函数，具体作用如下：  
- `split()`：拆分字符串。通过指定分隔符对字符串进行切片，并返回分割后的字符串列表（list）；
- `os.path.split()`：按照路径将文件名和路径分割开。

## 2. 语法

### 2.1 split()函数

格式：
```bash
str.split(str="",num=string.count(str))[n]
```

参数说明：  
- `str`：   表示为分隔符，默认为空格，但是不能为空('')。若字符串中没有分隔符，则把整个字符串作为列表的一个元素  
- `num`：表示分割次数。如果存在参数num，则仅分隔成 num+1 个子字符串，并且每一个子字符串可以赋给新的变量  
- `[n]`：   表示选取第n个分片

> 注意：当使用空格作为分隔符时，对于中间为空的项会自动忽略

### 2.2 os.path.split()函数  

格式：
```bash
os.path.split('PATH')
```


参数说明：

*  PATH指一个文件的全路径作为参数；
*  如果给出的是一个目录和文件名，则输出路径和文件名；
*  如果给出的是一个目录名，则输出路径和为空文件名。

## 3. demo

### 3.1 split() demo
1. url切割

```bash
`>>> u` `=` `"www.doiido.com.cn"`

`#使用默认分隔符`

`>>>` `print` `u.split()`

`[``'www.doiido.com.cn'``]`

`#以"."为分隔符`

`>>>` `print` `u.split(``'.'``)`

`[``'www'``,` `'doiido'``,` `'com'``,` `'cn'``]`

`#分割0次`

`>>>` `print` `u.split(``'.'``,``0``)`

`[``'www.doiido.com.cn'``]`

`#分割一次`

`>>>` `print` `u.split(``'.'``,``1``)`

`[``'www'``,` `'doiido.com.cn'``]`

`#分割两次`

`>>>` `print` `u.split(``'.'``,``2``)`

`[``'www'``,` `'doiido'``,` `'com.cn'``]`

`#分割两次，并取序列为1的项`

`>>>` `print` `u.split(``'.'``,``2``)[``1``]`

`doiido`

`#分割最多次（实际与不加num参数相同）`

`>>>` `print` `u.split(``'.'``,``-``1``)`

`[``'www'``,` `'doiido'``,` `'com'``,` `'cn'``]`

`#分割两次，并把分割后的三个部分保存到三个文件`

`>>> u1,u2,u3` `=` `u.split(``'.'``,``2``)`

`>>>` `print` `u1`

`www`

`>>>` `print` `u2`

`doiido`

`>>>` `print` `u3`

`com.cn`
```


2. 去掉换行符

```bash
`>>> c` `=` `'''say`

`hello`

`baby'''`

`>>>` `print` `c`

`say`

`hello`

`baby`

`>>>` `print` `c.split(``'\n'``)`

`[``'say'``,` `'hello'``,` `'baby'``]`
```

3. 多字符替换
```bash
`>>> string``=``"hello boy<[www.doiido.com]>byebye"`

`>>>` `print` `string.split(``"["``)[``1``].split(``"]"``)[``0``]`

`www.doiido.com`

`>>>` `print` `string.split(``"["``)[``1``].split(``"]"``)[``0``].split(``"."``)`

`[``'www'``,` `'doiido'``,` `'com'``]`
```

### 3.2 os.path.split() demo
1. 分离文件名和路径
```bash
`>>>` `import` `os`

`>>>` `print` `os.path.split(``'/dodo/soft/python/'``)`

`(``'/dodo/soft/python'``, '')`

`>>>` `print` `os.path.split(``'/dodo/soft/python'``)`

`(``'/dodo/soft'``,` `'python'``)`
```

----

✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>


 - [python 内置函数](https://blog.csdn.net/xixihahalelehehe/article/details/104913051)

![在这里插入图片描述](https://img-blog.csdnimg.cn/0bd917e7997442b6a8d498db7d766984.gif#pic_center)

