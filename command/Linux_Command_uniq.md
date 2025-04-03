#  Linux Command uniq



-------------

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/988a436e6cf95d7c6ac6b6fa443287ef.gif#pic_center)


##  1. 简介
Linux uniq 命令用于检查及删除文本文件中重复出现的行列，一般与 sort 命令结合使用。

uniq 可检查文本文件中重复出现的行列
##  2. 参数

```bash
-c, --count                在每行开头增加重复次数。
-d, --repeated             所有邻近的重复行只被打印一次。
-D                         所有邻近的重复行将全部打印。
--all-repeated[=METHOD]    类似于 -D，但允许每组之间以空行分割。METHOD取值范围{none(默认)，prepend，separate}。
-f, --skip-fields=N        跳过对前N个列的比较。
--group[=METHOD]           显示所有行，允许每组之间以空行分割。METHOD取值范围：{separate(默认)，prepend，append，both}。
-i, --ignore-case          忽略大小写的差异。
-s, --skip-chars=N         跳过对前N个字符的比较。
-u, --unique               只打印非邻近的重复行。
-z, --zero-terminated      设置行终止符为NUL（空），而不是换行符。
-w, --check-chars=N        只对每行前N个字符进行比较。
--help                     显示帮助信息并退出。
--version                  显示版本信息并退出。
```
## 3. 示例
### 1. 重复相邻
文件testfile中第 2、3、5、6、7、9行为相同的行，使用 uniq 命令删除重复的行，可使用以下命令：

```bash
$ cat testfile      #原有内容  
test 30  
test 30  
test 30  
Hello 95  
Hello 95  
Hello 95  
Hello 95  
Linux 85  
Linux 85 
```

```bash
$ uniq testfile     #删除重复行后的内容  
test 30  
Hello 95  
Linux 85 
```
检查文件并删除文件中重复出现的行，并在行首显示该行重复出现的次数。使用如下命令：

```bash
$ uniq -c testfile      #删除重复行后的内容  
3 test 30             #前面的数字的意义为该行共出现了3次  
4 Hello 95            #前面的数字的意义为该行共出现了4次  
2 Linux 85            #前面的数字的意义为该行共出现了2次 
```
### 2.  重复不相邻
当重复的行并不相邻时，uniq 命令是不起作用的，即若文件内容为以下时，uniq 命令不起作用：

```bash
$ cat testfile1      # 原有内容 
test 30  
Hello 95  
Linux 85 
test 30  
Hello 95  
Linux 85 
test 30  
Hello 95  
Linux 85 
```

```bash
$ sort  testfile1 | uniq
Hello 95  
Linux 85 
test 30

$ sort testfile1 | uniq -c
   3 Hello 95  
   3 Linux 85 
   3 test 30

$ sort testfile1 | uniq -d
Hello 95  
Linux 85 
test 30  
```

