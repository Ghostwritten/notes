# 循环



---


## 1. while
示例
```powershell
#!/bin/bash

#基础

i=1

while [ $i -le 5 ]

do 
	echo $i
	let i++
done
```
输出：
```bash
$ sh while.sh
1
2
3
4
5
```

### 1.1 大小判断
```powershell
i=10
while [ $i -ne 0 ]
do
	i=`expr $i - 1`
	echo $i
#	sleep 1
done
```
输出：
```bash
sh while2.sh
9
8
7
6
5
4
3
2
1
0
```
### 1.2 100相加
```powershell
#!/bin/bash
#计算1+2+...+100的值

i=1
sum=0
while (( i<101 ))
do
    (( sum=sum+i ))
    (( i++ ))
done

echo "The total value is $sum"
```
输出：
```bash
$ bash while.sh
The total value is 5050

```


### 1.3 无限循环
```powershell
while true
do
    uptime
    sleep 1
done

```
输出：
```bash
sh while.sh

10:20:54 up 4 min,  1 user,  load average: 0.05, 0.19, 0.12
10:20:56 up 4 min,  1 user,  load average: 0.05, 0.19, 0.12
10:20:57 up 4 min,  1 user,  load average: 0.05, 0.19, 0.12
10:20:58 up 4 min,  1 user,  load average: 0.05, 0.19, 0.12
```

参考：

 - [Shell中while循环的陷阱, 变量实效,无法赋值变量](https://justcode.ikeepstudying.com/2018/02/shell%EF%BC%9A-shell%E4%B8%ADwhile%E5%BE%AA%E7%8E%AF%E7%9A%84%E9%99%B7%E9%98%B1-%E5%8F%98%E9%87%8F%E5%AE%9E%E6%95%88-%E6%97%A0%E6%B3%95%E8%B5%8B%E5%80%BC%E5%8F%98%E9%87%8F/)

### 1.4 多列输出

```bash
$ cat test.txt 
xiaoming 122458930 山西
dali  343536546232 新疆
lihua  35354646565 广东

$ cat while2.sh 
#!/bin/bash
while read  name phone origin
do
  echo "$name : $phone : $origin"
done < test.txt
```

执行：
```bash
$ bash while2.sh 
xiaoming : 122458930 : 山西
dali : 343536546232 : 新疆
lihua : 35354646565 : 广东
```

### 1.5 读行
```bash
$ cat test 
1
22
33 3
```

 - `while-read.sh`

```bash
#!/bin/bash
# while-read: read lines from a file
count=0
while read; do
	printf "%d %s\n" $REPLY
	count=$(expr $count + 1)
done <$1
```
执行：

```bash
$ bash  wile_read.sh test
1 
22 
33 3
count:3
```




## 2. for
格式：

```powershell
for var in item1 item2 ... itemN
do
    command1
    command2
    ...
    commandN
done
```

写成一行：

```powershell
for var in item1 item2 ... itemN; do command1; command2… done;
```

 - in列表可以包含替换、字符串和文件名。
 - in列表是可选的，如果不用它，for循环使用命令行的位置参数。

例如，顺序输出当前列表中的数字：

```powershell
#!/bin/bash
 
for varible1 in {1..5}
#for varible1 in 1 2 3 4 5
do
     echo "Hello, Welcome $varible1 times "
done
```

计算1～100内所有的奇数之和

```powershell
#!/bin/bash
sum=0
 
for i in {1..100..2}
do
    let "sum+=i"
done
    
echo "sum=$sum"
```
执行：
```bash
$ bash while1.sh 
sum=2500
```

通过i的按步数2不断递增，计算sum值为2500。同样可以使用seq命令实现按2递增来计算1～100内的所有奇数之和，for i in $(seq 1 2 100)，seq表示起始数为1，跳跃的步数为2，结束条件值为100。

```powershell
#!/bin/bash
sum=0
 
for i in $(seq 1 2 100)
do
    let "sum+=i"
done
    
echo "sum=$sum"
[root@localhost control]# bash while2.sh 
sum=2500
```
查看当前目录下文件以及文件夹

```powershell
#!/bin/bash
 
for file in $( ls )
#for file in *
do
   echo "file: $file"
done
```
for循环列表参数

```powershell
#!/bin/bash
echo "number of arguments is $#"
echo "What you input is: "
for argument
do
    echo "$argument"
done
```

```powershell
#!/bin/bash
for((integer = 1; integer <= 5; integer++))
do
    echo "$integer"
done
```
## 3. break
break 跳出整个循环

```bash
$ cat break1.sh 
#!/bin/bash
for ((a=1; a<=3; a++))
do
   echo "outer loop: $a"
   for ((b=1; b<=4; b++))
   do
     if [ $b -eq 3 ]
     then
       break
     fi
   echo "inter loop: $b"
   done
done
```
执行：
```bash
$ bash break1.sh
outer loop: 1
inter loop: 1
inter loop: 2
outer loop: 2
inter loop: 1
inter loop: 2
outer loop: 3
inter loop: 1
inter loop: 2

```

## 4. continue
continue跳出本次循环
```bash
$ cat continue1.sh 
#!/bin/bash
for ((a=1; a<=3; a++))
do
   echo "outer loop: $a"
   for ((b=1; b<=4; b++))
   do
     if [ $b -eq 3 ]
     then
       continue  
     fi
   echo "inter loop: $b"
   done
done
```
执行：
```bash
$ bash continue1.sh
outer loop: 1
inter loop: 1
inter loop: 2
inter loop: 4
outer loop: 2
inter loop: 1
inter loop: 2
inter loop: 4
outer loop: 3
inter loop: 1
inter loop: 2
inter loop: 4
```

参考：

 - [Looping Statements | Shell Script](https://www.geeksforgeeks.org/looping-statements-shell-script/?ref=lbp)
 - [Bash 脚本教程：循环](https://wangdoc.com/bash/loop.html)

