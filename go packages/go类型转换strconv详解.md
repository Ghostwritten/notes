

----
## 1. 零值
变量在定义时没有明确的初始化时会赋值为 零值 。

零值是：

 - 数值类型为 0 ，
 - 布尔类型为 false ，
 - 字符串为 "" （空字符串）。

Golang 不支持隐式类型转换，即便是从窄向宽转换也不行。

```go
package main
var b byte =100// var n int = b// ./main.go:5:5: cannot use b (type byte) as type int in assignment
var n int =int(b)// 显式转换
func main(){}
```

同样不能将其他类型当 bool 值使用。

```go
package main
func main(){
    a :=100if a {// Error: non-bool a (type int) used as if conditionprintln("true")}}
```

## 2. 类型转换type_name

类型转换用于将一种数据类型的变量转换为另外一种类型的变量。Go 语言类型转换基本格式如下：

表达式 T(v) 将值 v 转换为类型 T 。

```bash
type_name(expression)

type_name 为类型，expression 为表达式。
```
将`整型转化为浮点型`，并计算结果，将结果赋值给浮点型变量：

```go
package main
import "fmt"
func main(){
    var sum int =17
    var count int =5
    var mean float32
    mean =float32(sum)/float32(count)
    fmt.Printf("mean 的值为: %f\n", mean)}
输出结果：

mean 的值为:3.400000
```

一些关于数值的转换：

```go
package main
import ("fmt""reflect")
func main(){
    var i int =42
    fmt.Printf("i value is : %v , type is : %v \n", i, reflect.TypeOf(i))
    var f float64 =float64(i)
    fmt.Printf("f value is : %v , type is : %v \n", f, reflect.TypeOf(f))
    var u uint =uint(f)
    fmt.Printf("u value is : %v , type is : %v \n", u, reflect.TypeOf(u))}
输出结果：

i value is :42, type is : int 
f value is :42, type is : float64 
u value is :42, type is : uint 
```

或者，更加简单的形式：

```go
package main
import ("fmt""reflect")
func main(){
    i :=42
    f :=float64(i)
    u :=uint(f)
    fmt.Printf("i value is : %v , type is : %v \n", i, reflect.TypeOf(i))
    fmt.Printf("f value is : %v , type is : %v \n", f, reflect.TypeOf(f))
    fmt.Printf("u value is : %v , type is : %v \n", u, reflect.TypeOf(u))}
输出结果：

i value is :42, type is : int 
f value is :42, type is : float64 
u value is :42, type is : uint 
```

## 3.类型转换strconv

### 1.类型推导
在定义一个变量却并不显式指定其类型时（使用 := 语法或者 var = 表达式语法）【全局变量不适用】， 变量的类型由（等号）右侧的值推导得出。

当右值定义了类型时，新变量的类型与其相同：

```go
package main
import ("fmt""reflect")
func main(){
    var i int
    j := i // j 也是一个 int
    fmt.Printf("i type is : %v\n", reflect.TypeOf(i))
    fmt.Printf("j type is : %v\n", reflect.TypeOf(j))}
输出结果：

i type is : int
j type is : int
```

但是当右边包含了未指名类型的数字常量时，新的变量就可能是 int 、 float64 或 complex128 。

```go
package main
import ("fmt""reflect")
func main(){
    i :=42           
    f :=3.142        
    g :=0.867+0.5i 
    fmt.Printf("i type is : %v\n", reflect.TypeOf(i))
    fmt.Printf("f type is : %v\n", reflect.TypeOf(f))
    fmt.Printf("g type is : %v\n", reflect.TypeOf(g))}
输出结果：

i type is : int
f type is : float64
g type is : complex128
```

### 2.字符串转整形
将字符串转换为 int 类型 

```bash
strconv.ParseInt(str,base,bitSize)
str：要转换的字符串 
base：进位制（2 进制到 36 进制） 
bitSize：指定整数类型（0:int、8:int8、16:int16、32:int32、64:int64） 
返回转换后的结果和转换时遇到的错误 
如果 base 为 0，则根据字符串的前缀判断进位制（0x:16，0:8，其它:10）
ParseUint 功能同 ParseInt 一样，只不过返回 uint 类型整数
Atoi 相当于 ParseInt(s, 10, 0)
```

通常使用这个函数，而不使用 ParseInt

该方法的源码是：

```go
// Itoa is shorthand for FormatInt(i, 10).
func Itoa(i int) string {returnFormatInt(int64(i),10)}
```

可以看出是FormatInt方法的简单实现。

```go
package main
import ("fmt""reflect""strconv")
func main(){
    i, ok := strconv.ParseInt("1000",10,0)if ok == nil {
        fmt.Printf("ParseInt , i is %v , type is %v\n", i, reflect.TypeOf(i))}
    ui, ok := strconv.ParseUint("100",10,0)if ok == nil {
        fmt.Printf("ParseUint , ui is %v , type is %v\n", ui, reflect.TypeOf(i))}
    oi, ok := strconv.Atoi("100")if ok == nil {
        fmt.Printf("Atoi , oi is %v , type is %v\n", oi, reflect.TypeOf(i))}}
输出结果：

ParseInt , i is 1000, type is int64
ParseUint , ui is 100, type is int64
Atoi , oi is 100, type is int64
```

### 3.整形转字符串
FormatInt int 型整数 i 转换为字符串形式 

```go
strconv.FormatInt.(i,base)
FormatUint 将 uint 型整数 i 转换为字符串形式 
strconv.FormatUint.(i,base)
base：进位制（2 进制到 36 进制） 
大于 10 进制的数，返回值使用小写字母 ‘a’ 到 ‘z’
Itoa 相当于 FormatInt(i,10)
```

```go
package main
import ("fmt""reflect""strconv")
func main(){
    var i int64
    i =0x100
    str := strconv.FormatInt(i,10)// FormatInt第二个参数表示进制，10表示十进制。
    fmt.Println(str)
    fmt.Println(reflect.TypeOf(str))}
输出结果：

256
string
```

`AppendInt` 将 int 型整数 i 转换为字符串形式，并追加到 `[]byte` 的尾部

```bash
strconv.AppendInt([]byte, i, base)
```

`AppendUint` 将 uint 型整数 i 转换为字符串形式，并追加到 dst 的尾部

```bash
strconv.AppendUint([]byte, i, base)
```

i：要转换的字符串

base：进位制

返回追加后的 []byte

```go
package main
import ("fmt""strconv")
func main(){
    b :=make([]byte,0)
    b = strconv.AppendInt(b,-2048,16)
    fmt.Printf("%s\n", b)}
输出结果：

-800
```

### 4.字节转32位整形

```go
package main
import ("bytes""encoding/binary""fmt")
func main(){
    b :=[]byte{0x00,0x00,0x03,0xe8}
    bytesBuffer := bytes.NewBuffer(b)
    var x int32
    binary.Read(bytesBuffer, binary.BigEndian,&x)
    fmt.Println(x)}// 其中binary.BigEndian表示字节序，相应的还有little endian。通俗的说法叫大端、小端。
输出结果：

1000
```

### 5.32位整形转字节

```go
package main
import ("bytes""encoding/binary""fmt""reflect")
func main(){
    var x int32
    x =106
    bytesBuffer := bytes.NewBuffer([]byte{})
    binary.Write(bytesBuffer, binary.BigEndian, x)
    b := bytesBuffer.Bytes()
    fmt.Println(b)
    fmt.Println(reflect.TypeOf(b))}
输出结果：

[000106][]uint8
```

### 6.字节转字符串

```go
package main
import ("fmt""reflect")
func main(){
    b :=[]byte{97,98,99,100}
    str :=string(b)
    fmt.Println(str)
    fmt.Println(reflect.TypeOf(str))}
输出结果：

abcd
string
```

### 7.字符串转字节

```go
package main
import ("fmt")
func main(){
    str :="abcd"
    b :=[]byte(str)
    fmt.Println(b)}
输出结果：

[979899100]
```

### 8.字符串转布尔值 ParseBool

```go
package main
import ("fmt""strconv")
func main(){
    b, err := strconv.ParseBool("1")
    fmt.Printf("string 1 转 bool ：%v , err is : %v\n", b, err)
    b, err = strconv.ParseBool("t")
    fmt.Printf("string t 转 bool ：%v , err is : %v\n", b, err)
    b, err = strconv.ParseBool("T")
    fmt.Printf("string T 转 bool ：%v , err is : %v\n", b, err)
    b, err = strconv.ParseBool("true")
    fmt.Printf("string true 转 bool ：%v , err is : %v\n", b, err)
    b, err = strconv.ParseBool("True")
    fmt.Printf("string True 转 bool ：%v , err is : %v\n", b, err)
    b, err = strconv.ParseBool("TRUE")
    fmt.Printf("string TRUE 转 bool ：%v , err is : %v\n", b, err)
    b, err = strconv.ParseBool("TRue")
    fmt.Printf("string TRue 转 bool ：%v , err is : %v\n", b, err)
    b, err = strconv.ParseBool("")
    fmt.Printf("string '' 转 bool ：%v , err is : %v\n", b, err)
    b, err = strconv.ParseBool("0")
    fmt.Printf("string 0 转 bool ：%v , err is : %v\n", b, err)
    b, err = strconv.ParseBool("f")
    fmt.Printf("string f 转 bool ：%v , err is : %v\n", b, err)
    b, err = strconv.ParseBool("F")
    fmt.Printf("string F 转 bool ：%v , err is : %v\n", b, err)
    b, err = strconv.ParseBool("false")
    fmt.Printf("string false 转 bool ：%v , err is : %v\n", b, err)
    b, err = strconv.ParseBool("False")
    fmt.Printf("string False 转 bool ：%v , err is : %v\n", b, err)
    b, err = strconv.ParseBool("FALSE")
    fmt.Printf("string FALSE 转 bool ：%v , err is : %v\n", b, err)
    b, err = strconv.ParseBool("FALse")
    fmt.Printf("string FALse 转 bool ：%v , err is : %v\n", b, err)
    b, err = strconv.ParseBool("abc")
    fmt.Printf("string abc 转 bool ：%v , err is : %v\n", b, err)}
输出结果：

string 1 转 bool ：true, err is :<nil>
string t 转 bool ：true, err is :<nil>
string T 转 bool ：true, err is :<nil>
string true 转 bool ：true, err is :<nil>
string True 转 bool ：true, err is :<nil>
string TRUE 转 bool ：true, err is :<nil>
string TRue 转 bool ：false, err is : strconv.ParseBool: parsing "TRue": invalid syntax
string '' 转 bool ：false, err is : strconv.ParseBool: parsing "": invalid syntax
string 0 转 bool ：false, err is :<nil>
string f 转 bool ：false, err is :<nil>
string F 转 bool ：false, err is :<nil>
string false 转 bool ：false, err is :<nil>
string False 转 bool ：false, err is :<nil>
string FALSE 转 bool ：false, err is :<nil>
string FALse 转 bool ：false, err is : strconv.ParseBool: parsing "FALse": invalid syntax
string abc 转 bool ：false, err is : strconv.ParseBool: parsing "abc": invalid syntax
```

ParseBool 将字符串转换为布尔值 
**

```bash
它接受真值：1, t, T, TRUE,true, True 
它接受假值：0, f, F, FALSE,false, False.
```

** 
其它任何值都返回一个错误
### 9.布尔值转换为字符串 FormatBool

```go
package main
import ("fmt""reflect""strconv")
func main(){
    t := strconv.FormatBool(true)
    f := strconv.FormatBool(false)
    fmt.Printf("t is %v , t type is %v\n", t, reflect.TypeOf(t))
    fmt.Printf("f is %v , f type is %v\n", f, reflect.TypeOf(f))}
输出结果：

t is true, t type is string
f is false, f type is string
```

### 10.AppendBool 将布尔类型转换为字符串
然后将结果追加到 []byte 的尾部，返回追加后的 []byte

```go
package main
import ("fmt""strconv")
func main(){
    rst :=[]byte{}
    fmt.Printf("[]byte{} is %s\n", rst)
    rst = strconv.AppendBool(rst,true)
    fmt.Printf("appended true []byte{} is %s\n", rst)
    rst = strconv.AppendBool(rst,false)
    fmt.Printf("appended false []byte{} is %s\n", rst)}
输出结果：

[]byte{} is 
appended true[]byte{} is true
appended false[]byte{} is truefalse
```

### 11.将字符串转换为浮点数

```bash
strconv.ParseFloat(str,bitSize)
str：要转换的字符串
bitSize：指定浮点类型（32:float32、64:float64）
```

如果 str 是合法的格式，而且接近一个浮点值，

则返回浮点数的四舍五入值（依据 IEEE754 的四舍五入标准）

如果 str 不是合法的格式，则返回“语法错误”

如果转换结果超出 bitSize 范围，则返回“超出范围”

```go
package main
import ("fmt""strconv")
func main(){
    s :="0.12345678901234567890"
    f, err := strconv.ParseFloat(s,32)
    fmt.Println(f, err)
    fmt.Println(float32(f))
    fmt.Println("-----")
    f, err = strconv.ParseFloat(s,64)
    fmt.Println(f, err)
    fmt.Println(float64(f))
    fmt.Println("-----")
    str :="abcd"
    f, err = strconv.ParseFloat(str,32)
    fmt.Println(f, err)}
输出结果：

0.12345679104328156<nil>0.12345679-----0.12345678901234568<nil>0.12345678901234568-----0 strconv.ParseFloat: parsing "abcd": invalid syntax
```

### 12.将浮点数转换为字符串值

```bash
strconv.FormatFloat(f,fmt,prec,bitSize)
f：要转换的浮点数 
fmt：格式标记（b、e、E、,f、g、G） 
prec：精度（数字部分的长度，不包括指数部分） 
bitSize：指定浮点类型（32:float32、64:float64）
```

格式标记： 

```bash
‘b’ (-ddddp±ddd，二进制指数) 
‘e’ (-d.dddde±dd，十进制指数) 
‘E’ (-d.ddddE±dd，十进制指数) 
‘f’ (-ddd.dddd，没有指数) 
‘g’ (‘e’:大指数，’f’:其它情况) 
‘G’ (‘E’:大指数，’f’:其它情况)
如果格式标记为 ‘e’，’E’和’f’，则 prec 表示小数点后的数字位数 
如果格式标记为 ‘g’，’G’，则 prec 表示总的数字位数（整数部分+小数部分）
```

```go
package main
import ("fmt""strconv")
func main(){
    f :=100.12345678901234567890123456789
    fmt.Println(strconv.FormatFloat(f,'b',5,32))
    fmt.Println(strconv.FormatFloat(f,'e',5,32))
    fmt.Println(strconv.FormatFloat(f,'E',5,32))
    fmt.Println(strconv.FormatFloat(f,'f',5,32))
    fmt.Println(strconv.FormatFloat(f,'g',5,32))
    fmt.Println(strconv.FormatFloat(f,'G',5,32))
    fmt.Println(strconv.FormatFloat(f,'b',30,32))
    fmt.Println(strconv.FormatFloat(f,'e',30,32))
    fmt.Println(strconv.FormatFloat(f,'E',30,32))
    fmt.Println(strconv.FormatFloat(f,'f',30,32))
    fmt.Println(strconv.FormatFloat(f,'g',30,32))
    fmt.Println(strconv.FormatFloat(f,'G',30,32))}
输出结果：

13123382p-171.00123e+021.00123E+02100.12346100.12100.1213123382p-171.001234588623046875000000000000e+021.001234588623046875000000000000E+02100.123458862304687500000000000000100.1234588623046875100.1234588623046875
```

### 13.AppendFloat 将浮点数 f 转换为字符串值，并将转换结果追加到 []byte 的尾部

返回追加后的 []byte

```go
package main
import ("fmt""strconv")
func main(){
    f :=100.12345678901234567890123456789
    b :=make([]byte,0)
    b = strconv.AppendFloat(b, f,'f',5,32)
    b =append(b," "...)
    b = strconv.AppendFloat(b, f,'e',5,32)
    fmt.Printf("%s\n", b)}
输出结果：

100.123461.00123e+02
```

### 14.Quote 将字符串 s 转换为“双引号”引起来的字符串

其中的特殊字符将被转换为“转义字符”

不可显示的字符”将被转换为“转义字符”

```go
package main
import ("fmt""strconv")
func main(){
    fmt.Println(strconv.Quote(`C:\Windows`))}
输出结果：

"C:\\Windows"
```

### 15.AppendQuote 将字符串 s 转换为“双引号”引起来的字符串，

并将结果追加到 []byte 的尾部，返回追加后的 []byte

其中的特殊字符将被转换为“转义字符”

```go
package main
import ("fmt""strconv")
func main(){
    s := `C:\Windows`
    b :=make([]byte,0)
    b = strconv.AppendQuote(b, s)
    fmt.Printf("%s\n", b)}
输出结果：

"C:\\Windows"
```

### 16.QuoteToASCII 将字符串 s 转换为“双引号”引起来的 ASCII 字符串

“非 ASCII 字符”和“特殊字符”将被转换为“转义字符”

```go
package main
import ("fmt""strconv")
func main(){
    asc := strconv.QuoteToASCII("Hello 世界！")
    fmt.Println(asc)}
输出结果：

"Hello \u4e16\u754c\uff01"
```

### 17.AppendQuoteToASCII 将字符串 s 转换为“双引号”引起来的 ASCII 字符串，

并将结果追加到 []byte 的尾部，返回追加后的 []byte

非 ASCII 字符”和“特殊字符”将被转换为“转义字符”

```go
package main
import ("fmt""strconv")
func main(){
    s :="Hello 世界！"
    b :=make([]byte,0)
    b = strconv.AppendQuoteToASCII(b, s)
    fmt.Printf("%s\n", b)}
输出结果：

"Hello \u4e16\u754c\uff01"
```

### 18.QuoteRune 将 Unicode 字符转换为“单引号”引起来的字符串

特殊字符”将被转换为“转义字符”

```go
package main
import ("fmt""strconv")
func main(){
    str := strconv.QuoteRune('哈')
    fmt.Println(str)}
输出结果：

'哈'
```

### 19.AppendQuoteRune 将 Unicode 字符转换为“单引号”引起来的字符串

并将结果追加到 []byte 的尾部，返回追加后的 []byte

特殊字符”将被转换为“转义字符”

```go
package main
import ("fmt""strconv")
func main(){
    b :=make([]byte,0)
    b = strconv.AppendQuoteRune(b,'哈')
    fmt.Printf("%s\n", b)}
输出结果：

'哈'
```

### 20.QuoteRuneToASCII 将 Unicode 字符转换为“单引号”引起来的 ASCII 字符串

“非 ASCII 字符”和“特殊字符”将被转换为“转义字符”

```go
package main
import ("fmt""strconv")
func main(){
    asc := strconv.QuoteRuneToASCII('哈')
    fmt.Println(asc)}
输出结果：

'\u54c8'
```

### 21.AppendQuoteRune 将 Unicode 字符转换为“单引号”引起来的 ASCII 字符串，

并将结果追加到 []byte 的尾部，返回追加后的 []byte

“非 ASCII 字符”和“特殊字符”将被转换为“转义字符”

```go
package main
import ("fmt""strconv")
func main(){
    b :=make([]byte,0)
    b = strconv.AppendQuoteRuneToASCII(b,'哈')
    fmt.Printf("%s\n", b)}
输出结果：

'\u54c8'
```

### 22.CanBackquote 判断字符串 s 是否可以表示为一个单行的“反引号”字符串

字符串中不能含有控制字符（除了 \t）和“反引号”字符，否则返回 false

```go
package main
import ("fmt""strconv")
func main(){
    b := strconv.CanBackquote("C:\\Windows\n")
    fmt.Printf("\\n is %v\n", b)
    b = strconv.CanBackquote("C:\\Windows\r")
    fmt.Printf("\\r is %v\n", b)
    b = strconv.CanBackquote("C:\\Windows\f")
    fmt.Printf("\\f is %v\n", b)
    b = strconv.CanBackquote("C:\\Windows\t")
    fmt.Printf("\\t is %v\n", b)
    b = strconv.CanBackquote("C:\\Windows`")
    fmt.Printf("` is %v\n", b)}
输出结果：

\n is false
\r is false
\f is false
\t is true
` is false
```

### 23**.UnquoteChar 将 s 中的第一个字符“取消转义”并解码

```bash
s：转义后的字符串

quote：字符串使用的“引号符”（用于对引号符“取消转义”）

value： 解码后的字符

multibyte：value 是否为多字节字符

tail： 字符串 s 除去 value 后的剩余部分

error： 返回 s 中是否存在语法错误
```

参数 quote 为“引号符”

如果设置为单引号，则 s 中允许出现 ‘ 字符，不允许出现单独的 ‘ 字符

如果设置为双引号，则 s 中允许出现 “ 字符，不允许出现单独的 “ 字符

如果设置为 0，则不允许出现 ‘ 或 “ 字符，可以出现单独的 ‘ 或 “ 字符

```go
package main
import ("fmt""strconv")
func main(){
    s := `\"大\\家\\好！\"`
    c, mb, sr, _ := strconv.UnquoteChar(s,'"')
    fmt.Printf("%-3c %v\n", c, mb)for;len(sr)>0; c, mb, sr, _ = strconv.UnquoteChar(sr,'"'){
        fmt.Printf("%-3c %v\n", c, mb)}}
输出结果：

"   false
"   false
大   true
\   false
家   true
\   false
好   true
！   true
```

### 24.Unquote 将“带引号的字符串” s 转换为常规的字符串（不带引号和转义字符）

s 可以是“单引号”、“双引号”或“反引号”引起来的字符串（包括引号本身）

如果 s 是单引号引起来的字符串，则返回该该字符串代表的字符

```go
package main
import ("fmt""strconv")
func main(){
    sr, err := strconv.Unquote("\"大\t家\t好！\"")
    fmt.Println(sr, err)
    sr, err = strconv.Unquote(`'大家好！'`)
    fmt.Println(sr, err)
    sr, err = strconv.Unquote("'好'")
    fmt.Println(sr, err)
    sr, err = strconv.Unquote("大\\t家\\t好！")
    fmt.Println(sr, err)}
输出结果：

大    家    好！ <nil>
 invalid syntax
好 <nil>
 invalid syntax
```

### 25.IsPrint 判断 Unicode 字符 r 是否是一个可显示的字符

可否显示并不是你想象的那样，比如空格可以显示，而\t则不能显示

```go
package main
import ("fmt""strconv")
func main(){
    fmt.Println(strconv.IsPrint('a'))
    fmt.Println(strconv.IsPrint('好'))
    fmt.Println(strconv.IsPrint(' '))
    fmt.Println(strconv.IsPrint('\t'))
    fmt.Println(strconv.IsPrint('\n'))
    fmt.Println(strconv.IsPrint(0))}
输出结果：

truetruetruefalsefalsefalse
```
参考连接：
[http://www.ahadoc.com/read/Golang-Detailed-Explanation/ch2.3.4.md#%E6%95%B4%E5%BD%A2%E8%BD%AC%E5%AD%97%E7%AC%A6%E4%B8%B2](http://www.ahadoc.com/read/Golang-Detailed-Explanation/ch2.3.4.md#%E6%95%B4%E5%BD%A2%E8%BD%AC%E5%AD%97%E7%AC%A6%E4%B8%B2)
