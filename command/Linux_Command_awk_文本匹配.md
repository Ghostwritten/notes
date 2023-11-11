#  Linux Command awk 文本匹配



----
## 1. 简介
awk是一个强大的文本分析工具，相对于[grep](https://ghostwritten.blog.csdn.net/article/details/105743476)的查找，[sed](https://blog.csdn.net/xixihahalelehehe/article/details/105140323)的编辑，awk在其对数据分析并生成报告时，显得尤为强大。简单来说awk就是把文件逐行的读入，以空格为默认分隔符将每行切片，切开的部分再进行各种分析处理。

awk有3个不同版本: `awk、nawk和gawk`，未作特别说明，一般指gawk，gawk 是 AWK 的 [GNU](https://www.gnu.org/) 版本。
awk其名称得自于它的创始人 `Alfred Aho` 、`Peter Weinberger` 和 `Brian Kernighan` 姓氏的首个字母。实际上 AWK 的确拥有自己的语言： AWK 程序设计语言 ， 三位创建者已将它正式定义为“样式扫描和处理语言”。它允许您创建简短的程序，这些程序读取输入文件、为数据排序、处理数据、对输入执行计算以及生成报表，还有无数其他的功能。


## 2. 原理

AWK 工作流程可分为三个部分：

 - 读输入文件之前执行的代码段（由BEGIN关键字标识）。
 - 主循环执行输入文件的代码段。
 - 读输入文件之后的代码段（由END关键字标识）。

命令结构:

```bash
awk 'BEGIN{ commands } pattern{ commands } END{ commands }'
```
下面的流程图描述出了 AWK 的工作流程：![在这里插入图片描述](https://img-blog.csdnimg.cn/20200424234927558.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

 - 1、通过关键字 BEGIN 执行 BEGIN 块的内容，即 BEGIN 后花括号 {} 的内容。
 - 2、完成 BEGIN 块的执行，开始执行body块。
 - 3、读入有 \n 换行符分割的记录。
 - 4、将记录按指定的域分隔符划分域，填充域，$0 则表示所有域(即一行内容)，$1 表示第一个域，$n 表示第 n 个域。
 - 5、依次执行各 BODY 块，pattern 部分匹配该行内容成功后，才会执行 awk-commands 的内容。
 - 6、循环读取并执行各行直到文件结束，完成body块执行。
 - 7、开始 END 块执行，END 块可以输出最终结果。

## 3. 命令
###  3.1 awk [选项] '{编辑指令}' 文件

```bash
awk '{print $1,$2}' /etc/rc.local      //输出文件的第1、2列
head -7 /etc/passwd > passwd.txt
awk -F ":" '{print $1","$7}' passwd.txt //以“：”作为分隔符，，打印第1和7列
awk -F [:/] '{print $1,$10}' passwd.txt //以“：”或“/”作为分隔符，，打印第1和10列
awk -F: '{print NR,NF}' passwd.txt  // 打印行数与字段数
free | awk '/Mem/{printf("RAM Usage: %.2f\n"), $3/$2*100}'| awk '{print $3}'
```
### 3.2 与管道结合
```bash
ifconfig eth0 | grep "inet" | awk '{print $2}
df -hT / | tail -1 | awk '{print $6}'
uname -a | awk '{print $1,$3,$12}'      //输出第1、3、12字段
seq 100 | awk 'NR%7==0||NR~/7/{print}'                     #列出100以内整数中7的倍数或是含7的数
seq 100 | awk '$0%7==0||$0~/7/{print}'
行号与每行的实际文本值是一致的，那么根据NR或者$0行值进行判断都是可以的。输出100以内7的倍数或是包含7的数
 ```

### 3.3 awk [选项] '[条件]{编辑指令}' 文件

```bash
awk -F: '/bash$/{print}' passwd.txt   #以bash结尾
awk -F: '/^[a-d]/{print $1,$6}' passwd.txt   #以a-d任意字符开头的第1和6列字段打印
awk -F: '/^a|nologin$/{print $1,$7}' passwd.txt  #以a开头或nologin结尾的第1和7列字段打印
awk -F: '$6~/bin$/{print $1,$6}' passwd.txt    #以bin结尾的打印第1和6列
awk -F: '$7!~/nologin$/{print $1,$7}' passwd.txt   #输出其中登录Shell不以nologin结尾（对第7个字段做!~反向匹配）的用户名、登录Shell信息
awk -F: 'NR==3{print}' passwd.txt        #输出第3行（行号NR等于3）的用户记录
awk -F: 'NR%2==1{print}' passwd.txt      #输出奇数行（行号NR除以2余数为1）的用户记录
awk -F: 'NR<=3{print}' passwd.txt        #输出前3行文本
awk -F: 'NR>=5{print}' passwd.txt        #输出从第5行开始到文件末尾的所有行
awk -F: '$1=="sync"{print}' passwd.txt      #输出用户名为“sync”的行
awk -F: '$1==ENVIRON["USER"]{print $1,$6,$7}' passwd.txt  #输出当前用户的用户名、宿主目录、登录Shell信息
awk -F: 'NR>=3&&NR<=5{print}' passwd.txt
awk -F: '(NR>=3)&&(NR<=5){print}' passwd.txt
awk -F: 'NR==3||NR==5{print}' passwd.txt
awk -F: '$7!~/nologin$/||$1~/^[ad]/{print}' passwd.txt  #输出“登录Shell不以nologin结尾”或者“用户名以a或d开头”的文本
awk -F: '$3<3||$3%2==0{print}' passwd.txt    #输出UID小于3或者UID是偶数的用户记录
awk -F: '$3>=501&&$3<=505{print}' /etc/passwd  #输出UID大于501并且小于505的行
awk -F: '/^127|^192/{print}' /etc/hosts   #输出/etc/hosts映射文件内以127或者192开头的记录
```

###  3.3 awk  [选项]  ' BEGIN{编辑指令 } {编辑指令} END{编辑指令}'  文件

```bash

#打印A乘于2.56的值
awk 'BEGIN{A=1024;print A*2.56}'    

#统计系统中使用bash作为登录Shell的用户总个数
awk 'BEGIN{x=0}/\<bash$/{x++} END{print x}' /etc/passwd    

#输出的内容包括三个部分：列表头、用户信息、列表尾
awk -F: 'BEGIN{print "User\tUID\tHome"} {print $1"\t"$3"\t"$6} END{print "Total "NR" lines."}' /etc/passwd    

#打印行数
awk -F"\t" 'BEGIN{x=0}{x+=NF}END{print x}' /etc/passwd

#以统计passwd.txt文件中以“:”分隔的总字段个数，需要每处理一行时将当前行的字段数（内置变量
awk -F: 'BEGIN{x=0}{x+=NF} END{print "Total "x" fields."}' passwd.txt   

#统计/etc/passwd文件中UID小于或等于500的用户个数
awk -F: 'BEGIN{i=0}{if($3<=500){i++}}END{print i}' /etc/passwd   

#统计/etc/passwd文件中登录Shell是“/bin/bash”的用户个数
awk -F: 'BEGIN{i=0}{if($7~/bash$/){i++}}END{print i}' /etc/passwd 

#统计/etc/passwd文件中登录Shell不是“/bin/bash”的用户个数
awk -F: 'BEGIN{i=0}{if($7!~/bash$/){i++}}END{print i}' /etc/passwd  

#分别统计/etc/passwd文件中UID小于或等于500、UID大于500的用户个数
awk -F: 'BEGIN{i=0;j=0}{if($3<=500){i++}else{j++}}END{print i,j}' /etc/passwd  

#分别统计/etc/passwd文件中登录Shell是“/bin/bash”、 登录Shell不是“/bin/bash”的用户个数
 awk -F: 'BEGIN{i=0;j=0}{if($7~/bash$/){i++}else{j++}} END{print i,j}' /etc/passwd 

分别统计/etc/passwd文件中登录Shell是“/bin/bash”、“/sbin/nologin”、其他的用户个数
awk -F: 'BEGIN{i=0;j=0;k=0}{if($7~/bash$/){i++} else if($7~/nologin$/){j++}else{k++}}END{print i,j,k}' /etc/passwd
#提取IP地址及访问量
awk  '{ip[$1]++} END{for(i in ip) {print i,ip[i]}}' /var/log/httpd/access_log   

#用户登陆次数
who | awk '{IP[$1]++}END{for(i in IP){print i,IP[i]}}' 

列出黑名单)根据此可作防火墙 
awk '{IP[$1]++}END{for(i in IP){print IP[i],i}}' /var/log/httpd/access_log | awk '$1>=500{print $2}' >>ip.log

#对第1）步的结果根据访问量排名
awk  '{ip[$1]++} END{for(i in ip) {print i,ip[i]}}' /var/log/httpd/access_log | sort -nr -k 2 

求最大值：
awk 'BEGIN {max = 0} {if ($1+0 > max+0) max=$1} END {print "Max=", max}' test.txt
Max= 118

求最小值：
awk 'BEGIN {min = 65536} {if ($1+0 < min+0) min=$1} END {print "Min=", min}' test.txt
Min= 9

求和：
cat test.txt|awk '{sum+=$1} END {print "Sum= ", sum}'
Sum=  236

求平均值：
cat test.txt|awk '{sum+=$1} END {print "Avg= ", sum/NR}'
Avg=  39.3333

 ```
 
### 3.4 第n到末尾参数输出

```bash
$ echo "1 2 3 4 5" | awk '{for(i=1;i<3;i++)$i="";print}'
  3 4 5

$ echo "1 2 3 4 5" | awk '{$1=$2="";print}'
  3 4 5

$ echo "1 2 3 4 5" | cut -d" " -f3-
3 4 5

$ echo "1 2 3 4 5" | awk '{$1=$2=""}1'
  3 4 5
```

###  3.5 多行合并一行

```bash
$  cat test
MD_Q9_G1_F
MD_Q9_G1_Ua
MD_Q9_G1_Ub
MD_Q9_G1_Uc
MD_Q9_G1_Uab
MD_Q9_G1_Ubc
MD_Q9_G1_Uca
MD_Q9_G1_Ia
MD_Q9_G1_Ib
MD_Q9_G1_Ic
MD_Q9_G1_Pa
MD_Q9_G1_Pb
MD_Q9_G1_Pc
MD_Q9_G1_Psum
MD_Q9_G1_Qa
MD_Q9_G1_Qb
MD_Q9_G1_Qc
MD_Q9_G1_Qsum
MD_Q9_G1_Sa
MD_Q9_G1_Sb
MD_Q9_G1_Sc
MD_Q9_G1_Ssum
MD_Q9_G1_PFa
MD_Q9_G1_PFb
MD_Q9_G1_PFc
MD_Q9_G1_PF

$ awk '{ORS=NR%50?",":RS}1' test
MD_Q9_G1_F,MD_Q9_G1_Ua,MD_Q9_G1_Ub,MD_Q9_G1_Uc,MD_Q9_G1_Uab,MD_Q9_G1_Ubc,MD_Q9_G1_Uca,MD_Q9_G1_Ia,MD_Q9_G1_Ib,MD_Q9_G1_Ic,MD_Q9_G1_Pa,MD_Q9_G1_Pb,MD_Q9_G1_Pc,MD_Q9_G1_Psum,MD_Q9_G1_Qa,MD_Q9_G1_Qb,MD_Q9_G1_Qc,MD_Q9_G1_Qsum,MD_Q9_G1_Sa,MD_Q9_G1_Sb,MD_Q9_G1_Sc,MD_Q9_G1_Ssum,MD_Q9_G1_PFa,MD_Q9_G1_PFb,MD_Q9_G1_PFc,MD_Q9_G1_PF,

$ awk 'NR%50{printf $0",";next}1'  test
MD_Q9_G1_F,MD_Q9_G1_Ua,MD_Q9_G1_Ub,MD_Q9_G1_Uc,MD_Q9_G1_Uab,MD_Q9_G1_Ubc,MD_Q9_G1_Uca,MD_Q9_G1_Ia,MD_Q9_G1_Ib,MD_Q9_G1_Ic,MD_Q9_G1_Pa,MD_Q9_G1_Pb,MD_Q9_G1_Pc,MD_Q9_G1_Psum,MD_Q9_G1_Qa,MD_Q9_G1_Qb,MD_Q9_G1_Qc,MD_Q9_G1_Qsum,MD_Q9_G1_Sa,MD_Q9_G1_Sb,MD_Q9_G1_Sc,MD_Q9_G1_Ssum,MD_Q9_G1_PFa,MD_Q9_G1_PFb,MD_Q9_G1_PFc,MD_Q9_G1_PF,

awk '{OFS=",";ORS=NR%50?OFS:RS}1' test
MD_Q9_G1_F,MD_Q9_G1_Ua,MD_Q9_G1_Ub,MD_Q9_G1_Uc,MD_Q9_G1_Uab,MD_Q9_G1_Ubc,MD_Q9_G1_Uca,MD_Q9_G1_Ia,MD_Q9_G1_Ib,MD_Q9_G1_Ic,MD_Q9_G1_Pa,MD_Q9_G1_Pb,MD_Q9_G1_Pc,MD_Q9_G1_Psum,MD_Q9_G1_Qa,MD_Q9_G1_Qb,MD_Q9_G1_Qc,MD_Q9_G1_Qsum,MD_Q9_G1_Sa,MD_Q9_G1_Sb,MD_Q9_G1_Sc,MD_Q9_G1_Ssum,MD_Q9_G1_PFa,MD_Q9_G1_PFb,MD_Q9_G1_PFc,MD_Q9_G1_PF,
```



### 3.6 获取外部变量
####  3.6.1 获得普通外部变量
格式：

```bash
awk '{action}'  变量名=变量值   
```
这样传入变量，可以在action中获得值。 **注意：变量名与值放到’{action}’后面。**

```bash
$ test='awk code'                           
$ echo | awk  '{print test}' test="$test"
awk code
```

#### 3.6.2 BEGIN 获取变量
格式：

```bash
awk –v 变量名=变量值 [–v 变量2=值2 …] 'BEGIN{action}’  
```

> 注意：用-v 传入变量可以在3中类型的action 中都可以获得到，但顺序在  action前面。

```bash
$ test='awk code'                                
$ echo | awk -v test="$test" 'BEGIN{print test}'
awk code
$ echo | awk -v test="$test" '{print test}'    
awk code
```

```bash
 errnum=$(($errnum+1))
 awk   -F " " -v b="$i" -v d=$errnum '{if ($1==b) $3=d}1{print $0 > "alert_list"}' alert_list
```

#### 3.6.3 获取环境变量

```bash
$ awk  'BEGIN{for (i in ENVIRON) {print i"="ENVIRON[i];}}'
AWKPATH=.:/usr/share/awk
SSH_ASKPASS=/usr/libexec/openssh/gnome-ssh-askpass
SELINUX_LEVEL_REQUESTED=
SELINUX_ROLE_REQUESTED=
LANG=en_US.UTF-8
.......
```
只需要调用：
awk内置变量 `ENVIRON`,就可以直接获得环境变量。它是一个字典数组。环境变量名 就是它的键值。

### 3.7 内建变量

包含：FS,NF,NR,RT,RS,ORS,OFS

#### 3.7.1 FS 指定字段un列分隔符（Font Space）

```bash
$ echo "111|222|333" | awk '{print $1}'
111|222|333
$ echo "111|222|333" | awk 'BEGIN{FS="|"}{print $1}'
111
```

#### 3.7.2 OFS 指定输出字段列分隔符（Output Font space）

```bash
$ echo "111 222 333" |awk 'BEGIN{OFS="|";}{print $1,$2,$3}'
111|222|333
```


#### 3.7.3 RS 指定行分隔符 默认分隔符为\n（Row Space）

```bash
$ echo "111 222|333 444|555 666" | awk 'BEGIN{RS="|"}{print $0}'
111 222
333 444
555 666
```


#### 3.7.4 ORS 指定输出行分隔符

```bash
$ awk 'BEGIN{ORS="|";}{print $0;}' test.txt
111 222|333 444|555 666
```

#### 3.7.5 RT 代指分隔符

```bash
$ echo "111 222|333 444|555 666" | awk 'BEGIN{RS="|"}{print $0,RT}'
111 222 |
333 444 |
555 666 |
```

#### 3.7.6 NF 每行字段总数(Number of Font)

```bash
$ cat test.txt
111 222
333 444
555 666
$ awk '{print NF}' test.txt
2
2
2
$ awk '{print $NF}' test.txt
222
444
666
```

#### 3.7.7 NR 当前行数(Number of Row)

```bash
$ cat test.txt
111 222
333 444
555 666 777
$ awk '{print NR}' test.txt
1
2
3
$ awk '{print $NR}' test.txt
111
444
777
```

## 4. 脚本
关于awk脚本，我们需要注意两个关键词`BEGIN`和`END`。

**BEGIN{ 这里面放的是执行前的语句 }
END {这里面放的是处理完所有的行后要执行的语句 }**
**{这里面放的是处理每一行时要执行的语句}**

假设有这么一个文件（学生成绩表）：

```bash
$ cat score.txt
Marry   2143 78 84 77
Jack    2321 66 78 45
Tom     2122 48 77 71
Mike    2537 87 97 95
Bob     2415 40 57 62
```

```bash
$ cat cal.awk
#!/bin/awk -f
#运行前
BEGIN {
    math = 0
    english = 0
    computer = 0
 
    printf "NAME    NO.   MATH  ENGLISH  COMPUTER   TOTAL\n"
    printf "---------------------------------------------\n"
}
#运行中
{
    math+=$3
    english+=$4
    computer+=$5
    printf "%-6s %-6s %4d %8d %8d %8d\n", $1, $2, $3,$4,$5, $3+$4+$5
}
#运行后
END {
    printf "---------------------------------------------\n"
    printf "  TOTAL:%10d %8d %8d \n", math, english, computer
    printf "AVERAGE:%10.2f %8.2f %8.2f\n", math/NR, english/NR, computer/NR
}
```

我们来看一下执行结果：

```bash
$ awk -f cal.awk score.txt
NAME    NO.   MATH  ENGLISH  COMPUTER   TOTAL
---------------------------------------------
Marry  2143     78       84       77      239
Jack   2321     66       78       45      189
Tom    2122     48       77       71      196
Mike   2537     87       97       95      279
Bob    2415     40       57       62      159
---------------------------------------------
  TOTAL:       319      393      350
AVERAGE:     63.80    78.60    70.00
```
## 5. 自定义函数
用户自定义函数的语法格式为：

```bash
function function_name(argument1, argument2, ...)
{
    function body
}
```

 - function_name 是用户自定义函数的名称。函数名称应该以字母开头，其后可以是数字、字母或下划线的自由组合。AWK
   保留的关键字不能作为用户自定义函数的名称。
 - 自定义函数可以接受多个输入参数，这些参数之间通过逗号分隔。参数并不是必须的。我们也可以定义没有任何输入参数的函数。
 - function body 是函数体部分，它包含 AWK 程序代码。
 
 求最大值与最小值

```bash
 # 返回最小值
function find_min(num1, num2)
{
  if (num1 < num2)
    return num1
  return num2
}

# 返回最大值
function find_max(num1, num2)
{
  if (num1 > num2)
    return num1
  return num2
}

# 主函数
function main(num1, num2)
{
  # 查找最小值
  result = find_min(10, 20)
  print "Minimum =", result

  # 查找最大值
  result = find_max(10, 20)
  print "Maximum =", result
}

# 脚本从这里开始执行
BEGIN {
  main(10, 20)
}  
```

执行 functions.awk 文件，可以得到如下的结果：

```bash
$ awk -f functions.awk 
Minimum = 10
Maximum = 20
```
## 6. 数组
AWK 可以使用关联数组这种数据结构，索引可以是数字或字符串。

AWK关联数 组也不需要提前声明其大小，因为它在运行时可以自动的增大或减小。

数组使用的语法格式：

```bash
array_name[index]=value
```

 - array_name：数组的名称
 - index：数组索引
 - value：数组中元素所赋予的值
### 6.1 创建数组
接下来看一下如何创建数组以及如何访问数组元素：

```bash
$ awk 'BEGIN {
sites["runoob"]="www.runoob.com";
sites["google"]="www.google.com"
print sites["runoob"] "\n" sites["google"]
}'
```

执行以上命令，输出结果为：

```bash
www.runoob.com
www.google.com
```
### 6.2 删除数组元素
我们可以使用 delete 语句来删除数组元素，语法格式如下：

```bash
delete array_name[index
```

下面的例子中，数组中的 google 元素被删除（删除命令没有输出）：

```bash
$ awk 'BEGIN {
sites["runoob"]="www.runoob.com";
sites["google"]="www.google.com"
delete sites["google"];
print fruits["google"]
}'
```
输出为空。
### 6.3 多维数组
下面是模拟二维数组的例子：

```bash
$ awk 'BEGIN {
array["0,0"] = 100;
array["0,1"] = 200;
array["0,2"] = 300;
array["1,0"] = 400;
array["1,1"] = 500;
array["1,2"] = 600;
# 输出数组元素
print "array[0,0] = " array["0,0"];
print "array[0,1] = " array["0,1"];
print "array[0,2] = " array["0,2"];
print "array[1,0] = " array["1,0"];
print "array[1,1] = " array["1,1"];
print "array[1,2] = " array["1,2"];
}'
```

执行上面的命令可以得到如下结果：

```bash
array[0,0] = 100
array[0,1] = 200
array[0,2] = 300
array[1,0] = 400
array[1,1] = 500
array[1,2] = 600
```
## 7. split 切割

```bash
$ ip=$(kubectl get pod www -o yaml |grep podIP | awk '{split($0,a,":"); print a[2]}'); echo $ip
10.32.0.6
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/e94a6acb70e04d89b39edcc4c295fcea.gif#pic_center)


