#  shell IFS



----
## 1. 简介
1. IFS的默认值为：空白（包括：空格，tab, 和新行)，将其ASSII码用十六进制打印出来就是：20 09 0a （见下面的shell脚本）。
2. IFS对空格的空白的处理和其他字符不一样，左右两边的纯空白会被忽略，多个连续的空白被当成一个IFS处理。
3. S*中使用IFS中的第一个字符。
4. awk中的FS（域分隔符）也和IFS有类似的用法和作用。
我写了一个shell脚本来演示IFS的用法和作用，如下：

```bash
[hjq@localhost test]$ IFS=''

[hjq@localhost test]$ set foo bar bam

[hjq@localhost test]$ echo "$@"
foo bar bam
[hjq@localhost test]$ echo "$*"
foobarbam
[hjq@localhost test]$ unset IFS
[hjq@localhost test]$ echo "$*"
foo bar bam


[hjq@localhost test]$ IFS=a
[hjq@localhost test]$ echo "$@"
foo bar bam
[hjq@localhost test]$ echo "$*"
fooabarabam
[hjq@localhost test]$ unset IFS
[hjq@localhost test]$ echo "$*"
foo bar bam
```
## 2. 实例
### 2.1 IFS 定制分隔符

```bash
#!/bin/sh

IFS='|'
text='a a a a|b b b b|c c c c'
for i in $text;do echo "i=$i";done
```

```bash
root@master:~# bash test2.sh
i=a a a a
i=b b b b
i=c c c c
```

### 2.2 换行符做分隔符

```bash
#!/bin/sh
conf="ABC
A B C
1|2|3
1 2 3"
echo "$conf"
 
echo --------------
echo IFS:
echo -n "$IFS"|xxd 
echo --------------
for c in $conf;do
    echo "line='$c'";
done
 
echo --------------
#IFS=$'\n'  表示用 换行符 做分隔
#IFS="\n" 与 IFS='\n'  都是用 n 字符作为分隔
IFS=$'\n'
echo IFS:
echo -n "$IFS"|xxd 
echo --------------
for c in $conf;do
    echo "line='$c'";
done
```

```bash
root@master:~# bash test4.sh 
ABC
A B C
1|2|3
1 2 3
--------------
IFS:
00000000: 2009 0a                                   ..
--------------
line='ABC'
line='A'
line='B'
line='C'
line='1|2|3'
line='1'
line='2'
line='3'
--------------
IFS:
00000000: 0a                                       .
--------------
line='ABC'
line='A B C'
line='1|2|3'
line='1 2 3'
```
参考：

 - [Shell中的IFS解惑](https://blog.csdn.net/whuslei/article/details/7187639)
 - [Linux Shell - What is IFS?](https://www.theunixschool.com/2020/05/linux-shell-what-is-ifs.html)
 - [The Meaning of IFS in Bash Scripting](https://www.baeldung.com/linux/ifs-shell-variable)

