#  Linux Command cmp 文件比较




---
## 1. 简介
cmp命令 用来比较两个文件是否有差异。
当相互比较的两个文件完全一样时，则该指令不会显示任何信息。
若发现有差异，预设会标示出第一个不通之处的字符和列数编号。
若不指定任何文件名称或是所给予的文件名为“-”，则cmp指令会从标准输入设备读取数据。

## 2. 选项

```bash
-c 或--print-chars：除了标明差异处的十进制字码之外，一并显示该字符所对应字符；
-i <字符数目>或--ignore-initial=<字符数目>：指定一个数目；
-l 或——verbose：标示出所有不一样的地方；
-s 或--quiet或——silent：不显示错误信息；
-v 或——version：显示版本信息；
--help：在线帮助。
```

##  3. 举例

```bash
$  cat test1
111
aaa
$ cat test2
111
222
aaa
bbb
$ cmp   test1 test2
test1 test2 differ: byte 5, line 2   # 有差异：第5字节，第2行

$ cmp -i 1  test1 test2
test1 test2 differ: byte 4, line 2 

$ cmp  -l  test1 test2  #标示出所有不一样的地方
5 141  62
6 141  62
7 141  62
cmp: EOF on test1
 
 $ cmp  -c  test1 test2  #除了标明差异处的十进制字码之外，一并显示该字符所对应字符
test1 test2 differ: byte 5, line 2 is 141 a  62 2

$ cmp  -s  test1 test2 #如果文件相同，返回值0，如果不同，返回值1；如果发生错误，返回值2
$ echo $?
1
```
更多阅读：

 - [cmp Command in Linux with examples](https://www.geeksforgeeks.org/cmp-command-in-linux-with-examples/)

