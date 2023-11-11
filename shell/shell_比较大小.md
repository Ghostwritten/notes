#  shell  比较大小
tags: 数学



---

##  三个数值
三个数值由小到大：

compare1.sh 
```bash
#!/bin/bash

echo -n "Enter three number:"
read a b c
if [ $a -gt $b ];then
t=$a;a=$b;b=$t;
fi
if [ $a -gt $c ];then
t=$a;a=$c;c=$t;
fi
if [ $b -gt $c ];then
t=$b;b=$c;c=$t;
fi
echo "From small to big:$a,$b,$c"
```
执行：
```bash
$ bash compare1.sh 
Enter three number: 2 4 3
From small to big:2,3,4
```
