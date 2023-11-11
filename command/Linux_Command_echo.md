#  Linux Command echo 
tags: 文本处理



---

##  1. 简介
echo是一种最常用的与广泛使用的内置于Linux的bash和C shell的命令，通常用在脚本语言和批处理文件中来在标准输出或者文件中显示一行文本或者字符串。

echo是一个内置shell命令。echo的行为与其他流行的shell一样，如Zsh和Ksh。但它们的行为与shell之间略有不同。

##  2. 转义
|选项	|描述|
|--|--|
|-n|	不输出末尾的换行符。|
|-e|	启用反斜线转义。
|\b|	退格
|\\	|反斜线
|\n|	新行
|\r	|回车
|\t	|水平制表符
|\v|	垂直制表符

##  3. 用法
### 标准输出

```bash
$ echo Tecmint is a community of Linux Nerds 
Tecmint is a community of Linux Nerds 
```
###   显示变量
比如，声明变量x并给它赋值为10。

```bash
$ x=10
$ echo The value of variable x = $x  
The value of variable x = 10 
```

###  显示转义字符

```bash
$ echo "\"It is a test\""
"It is a test"
```

###  显示文件

```bash
$ echo * 
103.odt 103.pdf 104.odt 104.pdf 105.odt 105.pdf 106.odt 106.pdf 107.odt 107.pdf 108a.odt 108.odt 108.pdf 109.odt 109.pdf 110b.odt 110.odt 110.pdf 111.odt 111.pdf 112.odt 112.pdf 113.odt linux-headers-3.16.0-customkernel_1_amd64.deb linux-image-3.16.0-customkernel_1_amd64.deb network.jpeg 
```

###  输出匹配文件
比如，让我们假设你想要打印所有的`‘.jpeg‘`文件，使用下面的命令。

```bash
$ echo *.jpeg 
network.jpeg 
```

###  定向输入文件

```bash
$ echo "Test Page" > testpage  
$ cat testpage 
Test Page 
```

###  彩色打印
你可使用[ANSI转义序列](https://en.wikipedia.org/wiki/ANSI_escape_code#Colors)更改前景色和背景色或设置下划线和粗体等文本属性。

```bash
echo -e "\033[1;37mWHITE"   #打印白色
echo -e "\033[0;30mBLACK"    #打印黑色
echo -e "\033[0;34mBLUE"      #打印蓝色
echo -e "\033[0;32mGREEN"   #打印绿色
echo -e "\033[0;36mCYAN"    #打印青蓝色
echo -e "\033[0;31mRED"      #打印红色
echo -e "\033[0;35mPURPLE"  #打印紫色。
```

###  显示命令执行结果

```bash
$ echo `date`
Tue Jun 28 11:59:39 UTC 2022
```


### \b 选项删除字符间的所有空格

 - -e‘后带上'\b'会删除字符间的所有空格。
 
> [!NOTE|style:flat|lable:Mylable|iconVisibility:hidden]
> Linux中的选项‘-e‘扮演了转义字符反斜线的翻译器。

```bash
$ echo -e "Tecmint \bis \ba \bcommunity \bof \bLinux \bNerds" 
TecmintisacommunityofLinuxNerds 
```

###  \n 选项换行

 - -e‘后面的带上‘\n’行会换行

```bash
$ echo -e "Tecmint \nis \na \ncommunity \nof \nLinux \nNerds" 
Tecmint 
is 
a 
community 
of 
Linux 
Nerds 
```

###   \t 选项加水平制表符

 - -e‘后面跟上‘\t’会在空格间加上水平制表符。

```bash
$ echo -e "Tecmint \tis \ta \tcommunity \tof \tLinux \tNerds"  
Tecmint     is  a   community   of  Linux   Nerds 
```

###   换行 \n 与水平制表符 \t

```bash
$ echo -e "\n\tTecmint \n\tis \n\ta \n\tcommunity \n\tof \n\tLinux \n\tNerds" 
    Tecmint 
    is 
    a 
    community 
    of 
    Linux 
    Nerds 
```
### \v 选项垂直制表符

 - ‘-e‘后面跟上‘\v’会加上垂直制表符。

```bash
$ echo -e "\vTecmint \vis \va \vcommunity \vof \vLinux \vNerds" 
Tecmint 
        is 
           a 
             community 
                       of 
                          Linux 
                                Nerds 
```

###  换行 \n 与垂直制表符 \v

```bash
$ echo -e "\n\vTecmint \n\vis \n\va \n\vcommunity \n\vof \n\vLinux \n\vNerds" 
 
Tecmint 
 
is 
 
a 
 
community 
 
of 
 
Linux 
 
Nerds 
```

> [!NOTE|style:flat|lable:Mylable|iconVisibility:hidden]
> 你可以按照你的需求连续使用两个或者多个垂直制表符，水平制表符与换行符。

###  \r 选项回车符

 - -e‘后面跟上‘\r’来指定输出中的回车符。（LCTT 译注：会覆写行开头的字符）

```bash
$ echo -e "Tecmint \ris a community of Linux Nerds"  
is a community of Linux Nerds 
```

###   \c 选项不换行

 - -e‘后面跟上‘\c’会抑制输出后面的字符并且最后不会换新行。

```bash
$ echo -e "Tecmint is a community \cof Linux Nerds" 
Tecmint is a community @tecmint:~$ 
```

###  -n 会在echo完后不会输出新行

```bash
$ echo -n "Tecmint is a community of Linux Nerds" 
Tecmint is a community of Linux Nerds@tecmint:~/Documents$ 
```

###  \a 选项声音警告

 - -e‘后面跟上‘\a’选项会听到声音警告。

```bash
$ echo -e "Tecmint is a community of \aLinux Nerds" 
Tecmint is a community of Linux Nerds
```


## 脚本
### feed.sh

```bash
$ cat feed.sh
#!/bin/bash
# This script acts upon the exit status given by penguin.sh

if [ "$#" != "2" ]; then
  echo -e "Usage of the feed script:\t$0 food-on-menu animal-name\n"
  exit 1
else

  export menu="$1"
  export animal="$2"

  echo -e "Feeding $menu to $animal...\n"

  feed="/nethome/anny/testdir/penguin.sh"

  $feed $menu $animal

result="$?"

  echo -e "Done feeding.\n"

case "$result" in

  1)
    echo -e "Guard: \"You'd better give'm a fish, less they get violent...\"\n"
    ;;
  2)
    echo -e "Guard: \"No wonder they flee our planet...\"\n"
    ;;
  3)
    echo -e "Guard: \"Buy the food that the Zoo provides at the entry, you ***\"\n"
    echo -e "Guard: \"You want to poison them, do you?\"\n"
    ;;
  *)
    echo -e "Guard: \"Don't forget the guide!\"\n"
    ;;
  esac

fi

echo "Leaving..."
echo -e "\a\a\aThanks for visiting the Zoo, hope to see you again soon!\n"

```
执行：
```bash
michel ~/test> feed.sh apple camel
Feeding apple to camel...

Will you read this sign?!  Don't feed the camels!

Done feeding.

Guard: "Buy the food that the Zoo provides at the entry, you ***"

Guard: "You want to poison them, do you?"

Leaving...
Thanks for visiting the Zoo, hope to see you again soon!

michel ~/test> feed.sh apple
Usage of the feed script:       ./feed.sh food-on-menu animal-name
```

参考：

 - [runoob Shell echo命令](https://www.runoob.com/linux/linux-shell-echo.html)
 - [Using the echo built-in command](https://tldp.org/LDP/Bash-Beginners-Guide/html/sect_08_01.html#tab_08_01)
 - [15 Practical Examples of ‘echo’ command in Linux](https://www.tecmint.com/echo-command-in-linux/)

