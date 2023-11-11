# Linux Command xargs 



---
## 1. 简介

`xargs`命令是给其他命令传递参数的一个过滤器，也是组合多个命令的一个工具。

 - 它擅长将标准输入数据转换成命令行参数，xargs能够处理管道或者stdin并将其转换成特定命令的命令参数。
 - xargs也可以将单行或多行文本输入转换为其他格式，例如多行变单行，单行变多行。
 - xargs的默认命令是echo，空格是默认定界符。这意味着通过管道传递给xargs的输入将会包含换行和空白，
 - 通过xargs的处理，换行和空白将被空格取代。xargs是构建单行命令的重要组件之一。

xargs命令的作用，是将标准输入转为命令行参数。

```bash
$ echo "hello world" | xargs echo hello world
hello world hello world
```

## 2. 格式

```bash
$ xargs [-options] [command]
```

xargs的作用在于，大多数命令（比如rm、mkdir、ls）与管道一起使用时，都需要xargs将标准输入转为命令行参数。

```bash
#1. 创建多文件
$ echo "one two three" | xargs mkdir
$ ls 
one two three

$ cat foo.txt
one
two
three

#2. 创建多文件
$ cat foo.txt | xargs -I % sh -c 'echo %; mkdir %'
one 
two
three

$ ls 
one two three

```

上面的代码等同于mkdir one two three。如果不加xargs就会报错，提示mkdir缺少操作参数。
## 3. 单独使用
```bash
$ echo   "a\nb\nc\nd" 
a\nb\nc\nd

$ echo  "a\nb\nc\nd" | xargs 
anbncnd

$ echo -e  "a\nb\nc\nd" 
a
b
c
d

$ echo -e "a\nb\nc\nd" | xargs 
a b c d

$ cat test.txt
a b c d e f g
h i j k l m n
o p q
r s t
u v w x y z

$ cat test.txt | xargs
a b c d e f g h i j k l m n o p q r s t u v w x y z
```



## 4. -d 自定义定界符

```bash
$ echo "nameXnameXnameXname" | xargs -dX
name name name name

$ echo "nameXnameXnameXname" | xargs -dX -n2
name name
name name

$ echo -e "a\tb\tc" | xargs -d "\t"
a b c
```



## 5. -p 询问执行
-p参数打印出要执行的命令，询问用户是否要执行
```bash
$ echo 'one two three' | xargs -p touch
touch one two three ?...y
$ ls
one two three


```
## 6. -t 立即执行
-t参数则是打印出最终要执行的命令，然后直接执行，不需要用户确认。

```bash
echo 'one two three' | xargs -t rm
rm one two three 
```



## 7. -I 替换字符串
xargs的一个选项-I，使用-I指定一个替换字符串{}，这个字符串在xargs扩展时会被替换掉，当-I与xargs结合使用，每一个参数命令都会被执行一次：



```bash
$ cat sk.sh
#!/bin/bash
echo $*

$ cat arg.txt
aaa
bbb
ccc

$ cat arg.txt | xargs -I {} ./sk.sh -p {} -l
-p aaa -l
-p bbb -l
-p ccc -l
```

复制所有图片文件到 /data/images 目录下：

```bash
ls *.jpg | xargs -n1 -I cp {} /data/images
```
## 8. -L 

```bash
$ cat test.txt | xargs -L 1
a b c d e f g
h i j k l m n
o p q
r s t
u v w x y z

[root@localhost shell]# cat test.txt | xargs -L 1
a b c d e f g
h i j k l m n
o p q
r s t
u v w x y z

$ cat test.txt | xargs -L 2
a b c d e f g h i j k l m n
o p q r s t
u v w x y z

$ cat test.txt | xargs -L 3
a b c d e f g h i j k l m n o p q
r s t u v w x y z

$ cat test.txt | xargs -L 4
a b c d e f g h i j k l m n o p q r s t
u v w x y z

$ cat test.txt | xargs -L 5
a b c d e f g h i j k l m n o p q r s t u v w x y z

$ cat test.txt | xargs -L 6
a b c d e f g h i j k l m n o p q r s t u v w x y z
```
## 9. -n 分列

```bash
$ cat test.txt
a b c d e f g
h i j k l m n
o p q
r s t
u v w x y z

$ cat test.txt | xargs -n 1
a
b
c
d
e
f
g
h
i
j
k
l
m
n
o
p
q
r
s
t
u
v
w
x
y
z

$ cat test.txt | xargs -n 2
a b
c d
e f
g h
i j
k l
m n
o p
q r
s t
u v
w x
y z

$ cat test.txt | xargs -n3
a b c
d e f
g h i
j k l
m n o
p q r
s t u
v w x
y z
```


快速关闭 Docker 容器
```bash
$ docker ps -q | xargs -n 1 --max-procs 0 docker kill
```

`--max-procs`参数指定同时用多少个进程并行执行命令。

 - `--max-procs 2`表示同时最多使用两个进程
 - `--max-procs 0`表示不限制进程数。

> xargs默认只用一个进程执行命令。如果命令要执行多次，必须等上一次执行完，才能执行下一次。

## 10. xargs 结合 find 使用

> 用rm 删除太多的文件时候，可能得到一个错误信息：`/bin/rm Argument list too long.`用xargs去避免这个问题： 由于xargs默认将空格作为分隔符，所以不太适合处理文件名，因为文件名可能包含空格。

**find命令有一个特别的参数-print0，指定输出的文件列表以null分隔。然后，xargs命令的-0参数表示用null当作分隔符**
```bash
$ find . -type f -name "*.log" -print0 | xargs -0 rm -f
```
xargs -0将`\0`作为定界符。

统计一个源代码目录中所有php文件的行数：

```bash
find . -type f -name "*.php" -print0 | xargs -0 wc -l
```

查找所有的jpg 文件，并且压缩它们：

```bash
find . -type f -name "*.jpg" -print | xargs tar -czvf images.tar.gz
```

与`-exec`相比，谁删除文件更快？当然`xargs`
```bash
time find . -type f -name "*.txt" -exec rm {} \;
0.35s user 0.11s system 99% cpu 0.467 total

time find ./foo -type f -name "*.txt" | xargs rm
0.00s user 0.01s system 75% cpu 0.016 total
```

删除14分钟以前的文件

```bash
find /tmp -mtime +14 | xargs rm
```

## 11. xargs 其他应用

假如你有一个文件包含了很多你希望下载的URL，你能够使用xargs下载所有链接：

```bash
cat url-list.txt | xargs wget -c
```
grep abc在所有的txt文件中

```bash
$ find . -name "*.txt" | xargs grep "abc"
```

更多阅读：

 - [xargs(1) — Linux manual page](https://man7.org/linux/man-pages/man1/xargs.1.html)
 - [How to Use the Linux xargs Command](https://phoenixnap.com/kb/xargs-command)

![在这里插入图片描述](https://img-blog.csdnimg.cn/02ed68580d694883901d7c2c866af945.gif#pic_center)

