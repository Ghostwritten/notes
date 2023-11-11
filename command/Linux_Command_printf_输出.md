#  Linux Command printf  输出


---

## 1. 简介

printf 命令模仿 C 程序库（library）里的 printf() 程序。
标准所定义，因此使用printf的脚本比使用echo移植性好。

printf 使用引用文本或空格分隔的参数，外面可以在printf中使用格式化字符串，还可以制定字符串的宽度、左右对齐方式等。默认printf不会像 echo 自动添加换行符，我们可以手动添加 \n。

## 2. 语法

```bash
printf  format-string  [arguments...]
```

## 3. 参数

 - format-string: 为格式控制字符串
 - arguments: 为参数列表。

## 4. 举例
###  4.1 echo 与 printf

```bash
$ echo "Hello, Shell"
Hello, Shell
$ printf "Hello, Shell\n"
Hello, Shell
```

###  4.2 格式输出

```powershell
#!/bin/bash
printf "%-10s %-8s %-4s\n" 姓名 性别 体重kg  
printf "%-10s %-8s %-4.2f\n" 郭靖 男 66.1234 
printf "%-10s %-8s %-4.2f\n" 杨过 男 48.6543 
printf "%-10s %-8s %-4.2f\n" 郭芙 女 47.9876
执行脚本，输出结果如下所示：

姓名     性别   体重kg
郭靖     男      66.12
杨过     男      48.65
郭芙     女      47.99
```

`%s %c %d %f`都是格式替代符

`%-10s` 指一个宽度为10个字符（-表示左对齐，没有则表示右对齐），任何字符都会被显示在10个字符宽的字符内，如果不足则自动以空格填充，超过也会将内容全部显示出来。

`%-4.2f` 指格式化为小数，其中.2指保留2位小数。


```powershell
#!/bin/bash
# format-string为双引号
printf "%d %s\n" 1 "abc"
 
# 单引号与双引号效果一样 
printf '%d %s\n' 1 "abc" 
 
# 没有引号也可以输出
printf %s abcdef
 
# 格式只指定了一个参数，但多出的参数仍然会按照该格式输出，format-string 被重用
printf %s abc def
 
printf "%s\n" abc def
 
printf "%s %s %s\n" a b c d e f g h i j
 
# 如果没有 arguments，那么 %s 用NULL代替，%d 用 0 代替
printf "%s and %d \n"
执行脚本，输出结果如下所示：

1 abc
1 abc
abcdefabcdefabc
def
a b c
d e f
g h i
j  
 and 0
```

### 4.3 转义序列

 - \a	警告字符，通常为ASCII的BEL字符
 - \b	后退
 - \c	抑制（不显示）输出结果中任何结尾的换行字符（只在%b格式指示符控制下的参数字符串中有效），而且，任何留在参数里的字符、任何接下来的参数以及任何留在格式字符串中的字符，都被忽略
 - \f	换页（formfeed）
 - \n	换行
 - \r	回车（Carriage return）
 - \t	水平制表符
 - \v	垂直制表符
 - \\	一个字面上的反斜杠字符
 - \ddd	表示1到3位数八进制值的字符。仅在格式字符串中有效
 - \0ddd	表示1到3位的八进制值字符

```powershell
$ printf "a string, no processing:<%s>\n" "A\nB"
a string, no processing:<A\nB>

$ printf "a string, no processing:<%b>\n" "A\nB"
a string, no processing:<A
B>

$ printf "www.csdn.net \a"
www.csdn.net 
```


###  居中显示

```bash
# 定义颜色变量
RED="\033[0;31m"
GREEN="\033[0;32m"
YELLOW="\033[0;33m"
NC="\033[0m"

echo -e "${GREEN}欢迎使用命令行工具！${NC}"
echo ""
echo -e "${YELLOW}请按以下步骤执行命令实例：${NC}"

# 显示命令实例
echo ""
echo -e "${YELLOW}命令实例：${NC}"
printf "%*s\n" $(((${#command}+$(tput cols))/2)) "docker ps"
echo ""
```
输出：

```bash
欢迎使用命令行工具！

请按以下步骤执行命令实例：

命令实例：
                                                       docker ps
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/5b0047b571ad45dfa8f78c8009cb5887.gif#pic_center)

