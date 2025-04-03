#  Shell 判断


---

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8816bf94ae02603e75c26be21430f08c.gif#pic_center)


## 1. if
###  1.1 if 
```powershell
if else
if
if 语句语法格式：

if condition
then
    command1 
    command2
    ...
    commandN 
fi
```

写成一行（适用于终端命令提示符）：

```powershell
if [ $(ps -ef | grep -c "ssh") -gt 1 ]; then echo "true"; fi
```

末尾的fi就是if倒过来拼写，后面还会遇到类似的。

###  1.2 if else
```powershell
if else
if else 语法格式：

if condition
then
    command1 
    command2
    ...
    commandN
else
    command
fi
```
###  1.3 if else-if else
if else-if else 语法格式：

```powershell
if condition1
then
    command1
elif condition2 
then 
    command2
else
    commandN
fi
```

以下实例判断两个变量是否相等：

```powershell
a=10
b=20
if [ $a == $b ]
then
   echo "a 等于 b"
elif [ $a -gt $b ]
then
   echo "a 大于 b"
elif [ $a -lt $b ]
then
   echo "a 小于 b"
else
   echo "没有符合的条件"
fi
输出结果：

a 小于 b
```
###  1.4 if test
if else语句经常与test命令结合使用，如下所示：

```powershell
num1=$[2*3]
num2=$[1+5]
if test $[num1] -eq $[num2]
then
    echo '两个数字相等!'
else
    echo '两个数字不相等!'
fi
输出结果：

两个数字相等!
```

### 1.5 if 文件判断
| 基本的                | 意义                                                                                                        |
|--------------------|-----------------------------------------------------------------------------------------------------------|
| [ -a FILE]          | 如果FILE存在，则为真。                                                                                             |
| [ -b FILE]           | 如果FILE存在并且是块特殊FILE，则为真。                                                                                     |
| [ -c FILE]           | 如果FILE存在并且是字符特殊FILE，则为真。                                                                                    |
| [ -d FILE]           | 如果FILE存在并且是一个目录，则为真。                                                                                      |
| [ -e FILE]           | 如果FILE存在，则为真。                                                                                             |
| [ -f FILE]           | 如果FILE存在并且是常规FILE，则为真。                                                                                      |
| [ -g FILE]           | 如果FILE存在且其 SGID 位已设置，则为真。                                                                                 |
| [ -h FILE]           | 如果FILE存在并且是符号链接，则为真。                                                                                      |
| [ -k FILE]           | 如果FILE存在并且其粘性位已设置，则为真。                                                                                    |
| [ -p FILE]           | 如果FILE存在并且是命名管道 (FIFO)，则为真。                                                                               |
| [ -r FILE]           | 如果FILE存在且可读，则为真。                                                                                          |
| [ -s FILE]           | 如果FILE存在且大小大于零，则为真。                                                                                       |
| [ -t FD ]          | 如果FILE描述符FD打开并指向终端，则为真。                                                                                     |
| [ -u FILE]           | 如果FILE存在且其 SUID（设置用户 ID）位已设置，则为真。                                                                         |
| [ -w FILE]           | 如果FILE存在且可写，则为真。                                                                                          |
| [ -x FILE]           | 如果FILE存在且可执行，则为真。                                                                                         |
| [ -O FILE]           | 如果FILE存在并且归有效用户 ID 所有，则为真。                                                                                |
| [ -G FILE]           | 如果FILE存在并且归有效组 ID 所有，则为真。                                                                                 |
| [ -L FILE]           | 如果FILE存在并且是符号链接，则为真。                                                                                      |
| [ -N FILE]           | 如果FILE存在并且自上次读取后已被修改，则为真。                                                                                 |
| [ -S FILE]           | 如果FILE存在并且是一个套接字，则为真。                                                                                     |
| [FILE1- nt FILE 2 ]    | 如果FILE1的更改比FILE2更新，或者FILE1存在而FILE2不存在，则为真。                                                                |
| [FILE1 -otFILE 2 ]     | 如果FILE1早于FILE2，或者FILE2存在而FILE1不存在，则为真。                                                                    |
| [FILE1 -efFILE 2 ]     | 如果FILE1和FILE2引用相同的设备和 inode 号，则为真。                                                                        |
| [ -o选项名称]          | 如果启用了 shell 选项“OPTIONNAME”，则为真。                                                                           |
| [-z字符串]            | 如果“STRING”为零，则长度为真。                                                                                       |
| [-n字符串] 或 [字符串]    | 如果“STRING”的长度不为零，则为真。                                                                                     |
| [ 字符串 1 == 字符串 2 ] | 如果字符串相等，则为真。 为了严格遵守 POSIX ，可以使用“=”代替“==” 。                                                                |
| [ 字符串 1 ！= 字符串 2 ] | 如果字符串不相等，则为真。                                                                                             |
| [ 字符串 1 < 字符串 2 ]  | 如果“STRING1”在当前语言环境中按字典顺序排在“STRING2 ”之前，则为真。                                                               |
| [ 字符串 1 > 字符串 2 ]  | 如果“STRING1”在当前语言环境中按字典顺序排在“STRING2 ”之后，则为真。                                                               |
| [ ARG1 开 ARG2 ]    | “OP”是-eq、-ne、-lt、-le、-gt或-ge之一。如果"ARG1"分别等于、不等于、小于、小于或等于、大于或大于或等于"ARG2" ，则这些算术二元运算符返回真。 “ARG1”和“ARG2”是整数。 |


## 2. case

```bash
name=`basename $0 .sh`
case $1 in
 s|start)
        echo "start..."
        ;;
 stop)
        echo "stop ..."
        ;;
 reload)
        echo "reload..."
        ;;
 *)
        echo "Usage: $name [start|stop|reload]"
        exit 1
        ;;
esac
exit 0
```

##  3. 组合表达式

| Operation          | Effect                                                                                      |
|--------------------|---------------------------------------------------------------------------------------------|
| [ ! EXPR ]         | 如果EXPR为假，则为真。.                                                                      |
| [ ( EXPR ) ]       | 返回EXPR的值。这可用于覆盖运算符的正常优先级. |
| [ EXPR1 -a EXPR2 ] | 如果EXPR1和EXPR2都为真，则为真。                                                  |
| [ EXPR1 -o EXPR2 ] | 如果EXPR1或EXPR2为真，则为真。                                                     |


参考：

 - [Introduction to if](https://tldp.org/LDP/Bash-Beginners-Guide/html/sect_07_01.html)
 - [Conditional Statements | Shell Script](https://www.geeksforgeeks.org/conditional-statements-shell-script/)

