#  linux Command sort 排序



-----------------------------

![在这里插入图片描述](https://img-blog.csdnimg.cn/b6c135db4f1a4155a0e306913757c345.gif#pic_center)

##  1. 简介
Linux sort 命令用于将文本文件内容加以排序。
sort 可针对文本文件的内容，以行为单位来排序。

## 2. 参数

```bash
-b 忽略每行前面开始出的空格字符。
-c 会检查文件是否已排好序，如果乱序，则输出第一个乱序的行的相关信息，最后返回1
-C 会检查文件是否已排好序，如果乱序，不输出内容，仅返回1
-d 排序时，处理英文字母、数字及空格字符外，忽略其他的字符。
-f 排序时，将小写字母视为大写字母。
-i 排序时，除了040至176之间的ASCII字符外，忽略其他的字符。
-m 将几个排序好的文件进行合并。
-M 将前面3个字母依照月份的缩写进行排序。比如JAN小于FEB等等
-n 依照数值的大小排序。
-u 意味着是唯一的(unique)，输出的结果是去完重了的。
-o<输出文件> 将排序后的结果存入指定的文件。
-r 以相反的顺序来排序。
-t<分隔字符> 指定排序时所用的栏位分隔字符。
+<起始栏位>-<结束栏位> 以指定的栏位来排序，范围由起始栏位到结束栏位的前一栏位。
--help 显示帮助。
--version 显示版本信息。
[-k field1[,field2]] 按指定的列进行排序。
```

##  3. 示例
### 1. 默认

```bash
$ cat testfile      # testfile文件原有排序  
test 30  
Hello 95  
Linux 85 

$ sort testfile # 重排结果  
Hello 95  
Linux 85  
test 30 
```
###  2 -k 设置列

使用 -k 参数设置对第二列的值进行重排，结果如下：

```bash
$ sort testfile -k 2
test 30  
Linux 85 
Hello 95  
```

### 3. -u 去除重复行

```bash
[rocrocket@rocrocket programming]$ cat seq.txt
banana
apple
pear
orange
pear
[rocrocket@rocrocket programming]$ sort seq.txt
apple
banana
orange
pear
pear
[rocrocket@rocrocket programming]$ sort -u seq.txt
apple
banana
orange
pear
```

### 4. -r 降序
sort默认的排序方式是升序，如果想改成降序，就加个-r就搞定了。

```bash
[rocrocket@rocrocket programming]$ cat number.txt
1
3
5
2
4
[rocrocket@rocrocket programming]$ sort number.txt
1
2
3
4
5
[rocrocket@rocrocket programming]$ sort -r number.txt
5
4
3
2
1
```

### 5. -o 重定向文件
由于sort默认是把结果输出到标准输出，所以需要用重定向才能将结果写入文件，形如sort filename > newfile。
但是，如果你想把排序结果输出到原文件中，用重定向可就不行了。

```bash
[rocrocket@rocrocket programming]$ sort -r number.txt > number.txt
[rocrocket@rocrocket programming]$ cat number.txt
[rocrocket@rocrocket programming]$
```
看，竟然将number清空了。
就在这个时候，-o选项出现了，它成功的解决了这个问题，让你放心的将结果写入原文件。这或许也是-o比重定向的唯一优势所在。

```bash
rocrocket@rocrocket programming]$ cat number.txt
1
3
5
2
4
[rocrocket@rocrocket programming]$ sort -r number.txt -o number.txt
[rocrocket@rocrocket programming]$ cat number.txt
5
4
3
2
1
```
### 6. -n 数值排序
你有没有遇到过10比2小的情况。我反正遇到过。出现这种情况是由于排序程序将这些数字按字符来排序了，排序程序会先比较1和2，显然1小，所以就将10放在2前面喽。这也是sort的一贯作风。

我们如果想改变这种现状，就要使用-n选项，来告诉sort，“要以数值来排序”

```bash
[rocrocket@rocrocket programming]$ cat number.txt
1
10
19
11
2
5
[rocrocket@rocrocket programming]$ sort number.txt
1
10
11
19
2
5
[rocrocket@rocrocket programming]$ sort -n number.txt
1
2
5
10
11
19
```
### 7. -t -k

```bash
[rocrocket@rocrocket programming]$ cat facebook.txt
banana:30:5.5
apple:10:2.5
pear:90:2.3
orange:20:3.4
```
这个文件有三列，列与列之间用冒号隔开了，第一列表示水果类型，第二列表示水果数量，第三列表示水果价格。

那么我想以水果数量来排序，也就是以第二列来排序，如何利用sort实现？

```bash
[rocrocket@rocrocket programming]$ sort -n -k 2 -t : facebook.txt
apple:10:2.5
orange:20:3.4
banana:30:5.5
pear:90:2.3
```

我们使用冒号作为间隔符，并针对第二列来进行数值升序排序，结果很令人满意。

### 8. 综合

```bash
$ cat facebook.txt
google 110 5000
baidu 100 5000
guge 50 3000
sohu 100 4500
```
第一个域是公司名称，第二个域是公司人数，第三个域是员工平均工资


**我想让这个文件按公司的字母顺序排序，也就是按第一个域进行排序**

```bash
$ sort -t ‘ ‘ -k 1 facebook.txt
baidu 100 5000
google 110 5000
guge 50 3000
sohu 100 4500
```
**我想让facebook.txt按照公司人数排序**

```bash
$ sort -n -t ‘ ‘ -k 2 facebook.txt
guge 50 3000
baidu 100 5000
sohu 100 4500
google 110 5000
```
**我想让facebook.txt按照公司人数排序 ，人数相同的按照员工平均工资升序排序**

```bash
$ sort -n -t ‘ ‘ -k 2 -k 3 facebook.txt
guge 50 3000
sohu 100 4500
baidu 100 5000
google 110 5000
```
**我想让facebook.txt按照员工工资降序排序，如果员工人数相同的，则按照公司人数升序排序**

```bash
$ sort -n -t ‘ ‘ -k 3r -k 2 facebook.txt
baidu 100 5000
google 110 5000
sohu 100 4500
guge 50 3000
```

```bash
$ sort -t ‘ ‘ -k 3nr -k 2n facebook.txt
baidu 100 5000
google 110 5000
sohu 100 4500
guge 50 3000
```
**突发奇想，从公司英文名称的第二个字母开始进行排序**

```bash
$ sort -t ‘ ‘ -k 1.2 facebook.txt
baidu 100 5000
sohu 100 4500
google 110 5000
guge 50 3000
```
我们使用了`-k 1.2`，这就表示对第一个域的第二个字符开始到本域的最后一个字符为止的字符串进行排序。你会发现baidu因为第二个字母是a而名列榜首。sohu和 google第二个字符都是o，但sohu的h在google的o前面，所以两者分别排在第二和第三。guge只能屈居第四了。

**只针对公司英文名称的第二个字母进行排序，如果相同的按照员工工资进行降序排序**

```bash
$ sort -t ‘ ‘ -k 1.2,1.2 -k 3,3nr facebook.txt
baidu 100 5000
google 110 5000
sohu 100 4500
guge 50 3000
```

由于只对第二个字母进行排序，所以我们使用了`-k 1.2,1.2`的表示方式，表示我们“只”对第二个字母进行排序。（如果你问“我使用-k 1.2怎么不行？”，当然不行，因为你省略了End部分，这就意味着你将对从第二个字母起到本域最后一个字符为止的字符串进行排序）。对于员工工资进行排 序，我们也使用了`-k 3,3`，这是最准确的表述，表示我们“只”对本域进行排序，因为如果你省略了后面的3，就变成了我们“对第3个域开始到最后一个域位置的内容进行排序” 了。

**-k和-u联合**

```bash
$ cat facebook.txt
google 110 5000
baidu 100 5000
guge 50 3000
sohu 100 4500
```

这是最原始的facebook.txt文件。

```bash
$ sort -n -k 2 facebook.txt
guge 50 3000
baidu 100 5000
sohu 100 4500
google 110 5000

$ sort -n -k 2 -u facebook.txt
guge 50 3000
baidu 100 5000
google 110 5000
```

当设定以公司员工域进行数值排序，然后加-u后，sohu一行就被删除了！原来-u只识别用-k设定的域，发现相同，就将后续相同的行都删除。

```bash
$ sort  -k 1 -u facebook.txt
baidu 100 5000
google 110 5000
guge 50 3000
sohu 100 4500

$ sort  -k 1.1,1.1 -u facebook.txt
baidu 100 5000
google 110 5000
sohu 100 4500
```

这个例子也同理，开头字符是g的guge就没有幸免于难。

```bash
$ sort -n -k 2 -k 3 -u facebook.txt
guge 50 3000
sohu 100 4500
baidu 100 5000
google 110 5000
```

这里设置了两层排序优先级的情况下，使用-u就没有删除任何行。原来-u是会权衡所有-k选项，将都相同的才会删除，只要其中有一级不同都不会轻易删除的。

