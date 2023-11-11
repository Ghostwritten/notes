#  Go 包 fmt 打印输出

![](https://img-blog.csdnimg.cn/2c714f9145cd4597b7c6002b0fca379e.png)


## 1. 格式化输出函数
```bash
func Print(a ...interface{}) (n int, err error)
```
Print采用默认格式将其参数格式化并写入标准输出。如果两个相邻的参数都不是字符串，会在它们的输出之间添加空格，返回写入的字节数和遇到的任何错误。
```bash
func Printf(format string, a ...interface{}) (n int, err error)
```
Printf根据format参数生成格式化的字符串并写入标准输出，返回写入的字节数和遇到的任何错误。
```bash
func Println(a ...interface{}) (n int, err error)
```
Println采用默认格式将其参数格式化并写入标准输出。总是会在相邻参数的输出之间添加空格并在输出结束后添加换行符，返回写入的字节数和遇到的任何错误。
区别：
 - Println :可以打印出字符串，和变量
 - Printf : 只可以打印出格式化的字符串,可以输出字符串类型的变量，不可以输出整形变量和整形

### 1.1 通用

```bash
%v	相应值的默认格式	Printf("%v",person )	{zhangsan}
%+v	类似%v，但输出结构体时会添加字段名式	Printf("%+v",person )	{Name:zhangsan}
%#v	相应值的Go语法表示	Printf("#v",person )	main.Person={zhangsan}
%T	相应值的类型的Go语法表示	Printf("%T",person )	main.Person
%%	字面上的百分号，并非值的占位符	Printf("%%")	%
```
### 1.2 布尔值
```bash
%t	单词true或false	Printf("%t",true)	true
```
### 1.3 整数

```bash
%b	二进制表示	Printf("%b",5)	101
%c	该值对应的unicode码值	Printf("%c",0x4E2d)	中
%d	十进制表示	Printf("%d",0x12)	18
%o	八进制表示	Printf("%o",10)	12
%q	单引号围绕的字符字面值，由Go语法安全的转译	Printf("%q",0x4E2d)	'中'
%x	十六进制表示，字母形式为小写a-f	Printf("%x",13)	d
%X	十六进制表示，字母形式为大写A-F	Printf("%X",13)	D
%U	表示为Unicode格式：U+1234，等价于"U+%04X"	Printf("%U",0x4E2d)	U+4E2D
```
### 1.4 浮点数与复数的两个组分

```bash
%b	无小数部分、指数为二的幂的科学计数法，与strconv.FormatFloat的'b'转换格式一致。	Printf("%b",10.20)	5742089524897382p-49
%e	科学计数法，如-1234.456e+78	Printf("%e",10.20)	1.020000e+01
%E	科学计数法，如-1234.456E+78	Printf("%E",10.20)	1.020000E+01
%f	有小数部分但无指数部分，如123.456	Printf("%f",10.20)	10.200000
%g	根据实际情况采用%e或%f格式（以获得更简洁、准确的输出）	Printf("%g",10.20)	10.2
%G	根据实际情况采用%E或%F格式（以获得更简洁、准确的输出）	Printf("%G",10.20)	(10.2+2i)
```
### 1.5 字符串和 []byte

```bash
%s  输出字符串表示(string类型或[]byte)  Printf("%s",[]byte("Go语言"))  Go语言
%q  双引号围绕的字符串，由Go语法安全的转译  Printf("%q","Go语言")  "Go语言"
%x  十六进制，小写字母，每字节两个字符  Printf("%x","golang")  676f6c616e67
%X  十六进制，大写字母，每字节两个字符  Printf("%X","golang")  676F6C616E67
```
### 1.6 指针

```bash
%P	十六进制表示，前缀 0x	Printf("%p",&person)	0xc0420341c0
```
### 1.7 其他

```bash
+	总是输出数值的正负号；对%q（%+q）会生成全部是ASCII字符的输出（通过转义）	Printf("%+q","中文")	"\u4e2d\u6587"
-	在输出右边填充空白而不是默认的左边（即从默认的右对齐切换为左对齐）；		
#	切换格式：八进制数前加0（%#o）	Printf("%#0",46)	
 	十六进制数前加0x（%#x）或0X（%#X）	Printf("%#x",46)	0x2e
 	指针去掉前面的0x（%#p）；）	fmt.Printf("%#p",&person)	c0420441b0
 	对%q（%#q），如果strconv.CanBackquote返回真会输出反引号括起来的未转义字符串；	Printf("%#q",'中')	'中'
 	对%U（%#U），如果字符是可打印的，会在输出Unicode格式、空格、单引号括起来的go字面值；	Printf("%#U",'中')	U+4E2D '中'
' '	(空格)为数值中省略的正负号流出空白(% d);	Printf("% d",16)	 16
 	以十六进制(% x,% X)打印字符串或切片时，在字节之间用空格隔开	Printf("% x","abc")	61 62 63
0	使用0而不是空格填充，对于数值类型会把填充的0放在正负号后面	
```

## 2. 实例

```bash
package main

import "fmt"

func main() {
        type Person struct {
                Name string
        }
        var people = Person{Name: "mark"}

        //1.普通占位符
        //%v(相应值的默认格式)
        fmt.Printf("%v", people) //{mark}

        //%+v(打印结构体时，会添加字段名)
        fmt.Printf("%+v", people) //{Name:mark}

        //%#v(相应值的Go语法表示)
        fmt.Printf("%#v", people) //main.Person{Name:"mark"}

        //%T(相应值的类型的Go语法表示)
        fmt.Printf("%T", people) //main.Person

        //%%(字面上的百分号，并非值的占位符)
        fmt.Printf("%%") //%

        //2.布尔占位符
        //%t(true 或 false)
        fmt.Printf("%t", true) //true

        //3.整数占位符
        //%b(二进制表示)
        fmt.Printf("%b", 5) //101

        //%c(相应Unicode码点所表示的字符)
        fmt.Printf("%c", 0x4E2D) //中

        //%d(十进制表示)
        fmt.Printf("%d", 0x12) //18

        //%o(八进制表示)
        fmt.Printf("%o", 10) //12

        //%q(单引号围绕的字符字面值，由Go语法安全地转义)
        fmt.Printf("%q", 0x4E2D) //'中'

        //%x(十六进制表示，字母形式为小写a-f)
        fmt.Printf("%x", 13) //d

        //%X(十六进制表示，字母形式为小写A-F)
        fmt.Printf("%X", 13) //D

        //%U(Unicode格式：U+1234，等同于 "U+%04X")
        fmt.Printf("%U", 0x4E2D) //U+4E2D

        //4.浮点数和复数的组成部分
        //%b(无小数部分的，指数为二的幂的科学计数法)
        fmt.Printf("%b", 10.2) //5742089524897382p-49

        //%e(科学计数法,例如 -1234.456e+78)
        fmt.Printf("%e", 10.2) //1.020000e+01

        //%E(科学计数法,例如 -1234.456E+78)
        fmt.Printf("%E", 10.2) //1.020000E+01

        //%f(有小数点而无指数，例如123.456)
        fmt.Printf("%f", 10.2) //10.200000

        //%g(根据情况选择%e或%f以产生更紧凑的(无末尾的0))
        fmt.Printf("%g", 10.20) //10.2

        //%G(根据情况选择%E或%f以产生更紧凑的(无末尾的0))
        fmt.Printf("%G", 10.20+2i) //(10.2+2i)

        //5.字符串与字节切片
        //%s(输出字符串表示(string类型或[]byte))
        fmt.Printf("%s", []byte("Go语言")) //Go语言

        //%q(双引号围绕的字符串，由Go语法安全地转义)
        fmt.Printf("%q", "Go语言") //"Go语言"

        //%x(十六进制，小写字母，每字节两个字符)
        fmt.Printf("%x", "golang") //676f6c616e67

        //%X(十六进制，大写字母，每字节两个字符)
        fmt.Printf("%X", "golang") //676F6C616E67

        //6.指针
        //%p(十六进制表示，前缀0x)
        fmt.Printf("%p", &people) //0xc0420421d0
}
```

参考

-  [https://www.cnblogs.com/Survivalist/articles/10287297.html](https://www.cnblogs.com/Survivalist/articles/10287297.html)
-  [http://static.markbest.site/blog/88.html](http://static.markbest.site/blog/88.html)
- [https://blog.csdn.net/chenbaoke/article/details/39932845](https://blog.csdn.net/chenbaoke/article/details/39932845)

