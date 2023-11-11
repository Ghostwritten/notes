#  Shell while getopts



---

## 1. 简介
`getpots`是Shell命令行参数解析工具，旨在从Shell Script的命令行当中解析参数。getopts被Shell程序用来分析位置参数，**option包含需要被识别的选项字符，如果这里的字符后面跟着一个冒号，表明该字符选项需要一个参数，其参数需要以空格分隔。冒号和问号不能被用作选项字符。getopts每次被调用时，它会将下一个选项字符放置到变量中，OPTARG则可以拿到参数值；如果option前面加冒号，则代表忽略错误**。

## 2. 格式

```bash
getopts optstring name [arg...]
```

 - `optstring` 列出了对应的Shell Script可以识别的所有参数。比如：如果 Shell Script可以识别-a，-f以及-s参数，则optstring就是afs；如果对应的参数后面还跟随一个值，则在相应的optstring后面加冒号。比如，a:fs   表示a参数后面会有一个值出现，-a  value的形式。另外，getopts执行匹配到a的时候，会把value存放在一个叫OPTARG的Shell Variable当中。如果optstring是以冒号开头的，命令行当中出现了optstring当中没有的参数将不会提示错误信息。
 - `name`表示的是参数的名称，每次执行getopts，会从命令行当中获取下一个参数，然后存放到name当中。如果获取到的参数不在optstring当中列出，则name的值被设置为?。命令行当中的所有参数都有一个index，第一个参数从1开始，依次类推。另外有一个名为OPTIND的Shell Variable存放下一个要处理的参数的index。

## 3. 示例
### 3.1  常规带参数的脚本
在shell脚本中，对于简单的参数，常常会使用$1，$2，...,$n来处理即可，具体如下：
```bash
[root@bobo tmp]# cat test.sh                     
#!/bin/bash
 
SYSCODE=$1
APP_NAME=$2
MODE_NAME=$3
 
echo "${SYSCODE}下的${APP_NAME}分布在${MODE_NAME}里面"
 
[root@bobo tmp]# sh test.sh caiwu reops kebank_uut
caiwu下的reops分布在kebank_uut里面
```
### 3.2 getopts的用法

```bash
[root@bobo tmp]# cat test.sh
#!/bin/bash
 
func() {
    echo "Usage:"
    echo "test.sh [-j S_DIR] [-m D_DIR]"
    echo "Description:"
    echo "S_DIR,the path of source."
    echo "D_DIR,the path of destination."
    exit -1
}
 
upload="false"
 
while getopts 'h:j:m:u' OPT; do
    case $OPT in
        j) S_DIR="$OPTARG";;
        m) D_DIR="$OPTARG";;
        u) upload="true";;
        h) func;;
        ?) func;;
    esac
done
 
echo $S_DIR
echo $D_DIR
echo $upload
```
执行脚本

```bash
[root@bobo tmp]# sh test.sh -j /data/usw/web -m /opt/data/web
/data/usw/web
/opt/data/web
false
 
[root@bobo tmp]# sh test.sh -j /data/usw/web -m /opt/data/web -u
/data/usw/web
/opt/data/web
true
 
[root@bobo tmp]# sh test.sh -j /data/usw/web
/data/usw/web
 
false
 
[root@bobo tmp]# sh test.sh -m /opt/data/web                 
 
/opt/data/web
false
 
[root@bobo tmp]# sh test.sh -h
test.sh: option requires an argument -- h
Usage:
test.sh [-j S_DIR] [-m D_DIR]
Description:
S_DIR,the path of source.
D_DIR,the path of destination.
 
[root@bobo tmp]# sh test.sh j
 
 
false
 
[root@bobo tmp]# sh test.sh j m
 
 
false
```
getopts后面跟的字符串就是参数列表，每个字母代表一个选项，如果字母后面跟一个：，则就表示这个选项还会有一个值，比如上面例子中对应的-j /data/usw/web 和-m /opt/data/web 。而getopts字符串中没有跟随:的字母就是开关型选项，不需要指定值，等同于true/false,只要带上了这个参数就是true。

getopts识别出各个选项之后，就可以配合case进行操作。操作中，有两个"常量"，一个是`OPTARG`，用来获取当前选项的值；另外一个就是`OPTIND`，表示当前选项在参数列表中的位移。case的最后一项是?，用来识别非法的选项，进行相应的操作，我们的脚本中输出了帮助信息。

### 3.3 getopts与shift的结合
：当选项参数识别完成以后，就能识别剩余的参数了，我们可以使用shift进行位移，抹去选项参数。

```bash
[root@bobo tmp]# cat test.sh
#!/bin/bash
  
func() {
    echo "func:"
    echo "test.sh [-j S_DIR] [-m D_DIR]"
    echo "Description:"
    echo "S_DIR, the path of source."
    echo "D_DIR, the path of destination."
    exit -1
}
  
upload="false"
  
echo $OPTIND
  
while getopts 'j:m:u' OPT; do
    case $OPT in
        j) S_DIR="$OPTARG";;
        m) D_DIR="$OPTARG";;
        u) upload="true";;
        ?) func;;
    esac
done
  
echo $OPTIND
shift $(($OPTIND - 1))
echo $1
```
执行：

```bash
[root@bobo tmp]# sh test.sh -j /data/usw/web beijing
1              #执行的是第一个"echo $OPTIND"
3              #执行的是第二个"echo $OPTIND"
beijing        #此时$1是"beijing"
 
[root@bobo tmp]# sh test.sh -m /opt/data/web beijing                
1              #执行的是第一个"echo $OPTIND"
3              #执行的是第二个"echo $OPTIND"
beijing
 
[root@bobo tmp]# sh test.sh -j /data/usw/web -m /opt/data/web beijing
1              #执行的是第一个"echo $OPTIND"
5              #执行的是第二个"echo $OPTIND"
beijing
 
                  参数位置： 1        2       3       4        5     6
[root@bobo tmp]# sh test.sh -j /data/usw/web -m /opt/data/web -u beijing
6
beijing
```
在上面的脚本中，我们位移的长度等于case循环结束后的OPTIND - 1，OPTIND的初始值为1。当选项参数处理结束后，其指向剩余参数的第一个。getopts在处理参数时，处理带值的选项参数，OPTIND加2；处理开关型变量时，OPTIND则加1。

如上执行的脚本：1）第一个脚本执行，-j的参数位置为1，由于-j后面带有参数，即处理带值选项参数，所以其OPTIND为1+2=3；2）第二个脚本执行，-m参数位置为1，由于其后带有参数，所以其OPTIND也为1+2=3；3）第三个脚本执行，-m的参数位置 (观察最后一个参数的位置) 为3，由于其后面带有参数，所以其OPTIND为3+2=5；4）第四个脚本执行，-u参数位置为5，由于其后面不带参数，即为处理开关型变量，所以其OPTIND为5+1=6。

### 3.4 getopts与shift结合2

```bash
[root@bobo tmp]# cat test.sh
#!/bin/bash
 
echo $*
while getopts ":a:bc:" opt
do
    case $opt in
      a)
      echo $OPTARG
      echo $OPTIND
      ;;
      b)
      echo "b $OPTIND"
      ;;
      c)
      echo "c $OPTIND"
      ;;
      ?)
      echo "error"
      exit 1
    esac
done
 
echo $OPTIND
shift $(( $OPTIND-1 ))
echo $0
echo $*
 
[root@bobo tmp]# sh test.sh -a beijing -b -c shanghai
-a beijing -b -c shanghai           #执行的是第一个"echo $*",即打印"传递给脚本的所有参数的列表"
beijing                             #执行的是"echo $OPTARG", OPTARG表示存储相应选项的参数，这里指-a的参数"beijing"
3                                   #-a参数位置为1，是处理带值选项参数，即-a参数的OPTIND为1+2=3
b 4                                 #-b参数位置为3，是处理开关型变量(即后面没有跟参数)，即-b参数的OPTIND为3+1=4
c 6                        #-c参数位置为4，是处理带值选项参数，即-a参数的OPTIND为4+2=3
6                          #执行的是"echo $OPTIND",此时打印的是脚本执行的最后一个参数(即-c)的OPTIND的index索引值。
test.sh                    #执行的是"echo $0",即打印脚本名称。$0是脚本本身的名字；
                           #执行的是最后一个"echo $*",即打印"传递给脚本的所有参数的列表"。由于前面执行了shift $(( $OPTIND-1 )),即每执行一步，位置参数减1，所以到最后$*就为零了。
[root@bobo tmp]#
```
对比无 shift

```bash
[root@bobo tmp]# cat test.sh
#!/bin/bash
# getopts-test.sh
 
while getopts :d:s ha
do
  case "$ha" in
      d)
        echo "d option value is $OPTARG"
        echo "d option index is $(($OPTIND-1))"
        ;;
      s)
        echo "s option..."
        echo "s option index is $(($OPTIND-1))"
        ;;
      [?])
        print "Usage: $0 [-s] [-d value] file ..."
        exit 1
        ;;
  esac
done
 
执行脚本：
[root@bobo tmp]# sh test.sh -d 100 -s
d option value is 100                  #打印的是对应选项的参数，即-d的参数值
d option index is 2                    #-d参数位置为1，是处理带值选项参数，即-d参数的OPTIND为1+2=3。所以$(($OPTIND-1))为2
s option...
s option index is 3                    #-s参数位置为3，是处理带值选项参数，即-s参数的OPTIND为3+1=4。所以$(($OPTIND-1))为2
```
### 3.6  getopts 忽略错误

```bash
[root@bobo tmp]# cat test.sh
#!/bin/bash
while getopts :ab:c: OPTION;do              #ab参数前面的:表示忽略错误
    case $OPTION in
      a)echo "get option a"
      ;;
      b)echo "get option b and parameter is $OPTARG"
      ;;
      c)echo "get option c and parameter is $OPTARG"
      ;;
      ?)echo "get a non option $OPTARG and OPTION is $OPTION"
      ;;
    esac
done
 
[root@bobo tmp]# sh test.sh -a haha
get option a
 
[root@bobo tmp]# sh test.sh -b hehe
get option b and parameter is hehe
 
[root@bobo tmp]# sh test.sh -a haha -b hehe     #由于getopts解析时ab参数在一起，-a和-b都跟参数时，-a在前面执行后，-b参数就不会执行了。
get option a
 
[root@bobo tmp]# sh test.sh -b haha -a hehe     #将-b参数放在前面执行，-a参数放在后面执行，两个参数就都可以执行了。
get option b and parameter is haha
get option a
 
[root@bobo tmp]# sh test.sh -ab hehe      
get option a
get option b and parameter is hehe
 
[root@bobo tmp]# sh test.sh -ab hehe -c heihei
get option a
get option b and parameter is hehe
get option c and parameter is heihei
 
[root@bobo tmp]# sh test.sh -ab hehe -c heihei -u liu
get option a
get option b and parameter is hehe
get option c and parameter is heihei
get a non option u and OPTION is ?
```

 
### 3.7 getopts 参数捆绑
稍微修改下脚本，将abc参数放在一起

```bash
[root@bobo tmp]# cat test.sh
#!/bin/bash
while getopts :abc: OPTION;do         
    case $OPTION in
      a)echo "get option a"
      ;;
      b)echo "get option b and parameter is $OPTARG"
      ;;
      c)echo "get option c and parameter is $OPTARG"
      ;;
      ?)echo "get a non option $OPTARG and OPTION is $OPTION"
      ;;
    esac
done
 
[root@bobo tmp]# sh test.sh -a haha
get option a
[root@bobo tmp]# sh test.sh -a haha -b hehe
get option a
[root@bobo tmp]# sh test.sh -a haha -c heihei        
get option a
[root@bobo tmp]# sh test.sh -a haha -b hehe -c heihei
get option a
[root@bobo tmp]# sh test.sh -a haha -c hehe -b heihei
get option a
 
[root@bobo tmp]# sh test.sh -b hehe
get option b and parameter is
[root@bobo tmp]# sh test.sh -b haha -a hehe
get option b and parameter is
[root@bobo tmp]# sh test.sh -b haha -c hehe
get option b and parameter is
[root@bobo tmp]# sh test.sh -b haha -a hehe -c heihei
get option b and parameter is
[root@bobo tmp]# sh test.sh -b haha -c hehe -a heihei
get option b and parameter is
 
[root@bobo tmp]# sh test.sh -c haha
get option c and parameter is haha
[root@bobo tmp]# sh test.sh -c haha -a hehe
get option c and parameter is haha
get option a
[root@bobo tmp]# sh test.sh -c haha -b heihei
get option c and parameter is haha
get option b and parameter is
[root@bobo tmp]# sh test.sh -c haha -a hehe -b heihei
get option c and parameter is haha
get option a
[root@bobo tmp]# sh test.sh -c haha -b hehe -c heihei
get option c and parameter is haha
get option b and parameter is
 
[root@bobo tmp]# sh test.sh -abc hehe
get option a
get option b and parameter is
get option c and parameter is hehe
```
参考：

 - [Shell脚本中的while getopts用法小结](https://www.cnblogs.com/kevingrace/p/11753294.html)
 - [Shell Scripting Tutorial - Tips Getopts](https://www.shellscript.sh/tips/getopts/index.html)
 - [How to Use getopts to Parse Linux Shell Script Options](https://www.howtogeek.com/778410/how-to-use-getopts-to-parse-linux-shell-script-options/)

