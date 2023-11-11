#  Go 运算符总结

![](https://img-blog.csdnimg.cn/f76e9eba9061424dad8784519d071f0c.png)





## 1.算数运算符

下表列出了所有Go语言的算术运算符。假定 A 值为 10，B 值为 20。                                                      

```bash
+	相加	A + B 输出结果 30
-	相减	A - B 输出结果 -10
*	相乘	A * B 输出结果 200
/	相除	B / A 输出结果 2
%	求余	B % A 输出结果 0
++	自增	A++ 输出结果 11
--	自减	A-- 输出结果 9
```

## 2.关系运算符

下表列出了所有Go语言的关系运算符。假定 A 值为 10，B 值为 20。

```bash
==	检查两个值是否相等，如果相等返回 True 否则返回 False。	(A == B) 为 False
!=	检查两个值是否不相等，如果不相等返回 True 否则返回 False。	(A != B) 为 True
>	检查左边值是否大于右边值，如果是返回 True 否则返回 False。	(A > B) 为 False
<	检查左边值是否小于右边值，如果是返回 True 否则返回 False。	(A < B) 为 True
>=	检查左边值是否大于等于右边值，如果是返回 True 否则返回 False。	(A >= B) 为 False
<=	检查左边值是否小于等于右边值，如果是返回 True 否则返回 False。	(A <= B) 为 True
```

## 3.逻辑运算符

下表列出了所有Go语言的逻辑运算符。假定 A 值为 True，B 值为 False。

```bash
&&	逻辑 AND 运算符。 如果两边的操作数都是 True，则条件 True，否则为 False。	(A && B) 为 False
||	逻辑 OR 运算符。 如果两边的操作数有一个 True，则条件 True，否则为 False。	(A || B) 为 True
!	逻辑 NOT 运算符。 如果条件为 True，则逻辑 NOT 条件 False，否则为 True。	!(A && B) 为 True
```

## 4.位运算符
位运算符对整数在内存中的二进制位进行操作。假定 A 为60，B 为13

```bash
&	按位与运算符"&"是双目运算符。 其功能是参与运算的两数各对应的二进位相与。	(A & B) 结果为 12, 111100 & 1101 二进制为 0000 1100
|	按位或运算符"|"是双目运算符。 其功能是参与运算的两数各对应的二进位相或	(A | B) 结果为 61, 111100 | 1101 二进制为 0011 1101
^	按位异或运算符"^"是双目运算符。 其功能是参与运算的两数各对应的二进位相异或，当两对应的二进位相异时，结果为1。	(A ^ B) 结果为 49, 二进制为 0011 0001
<<	左移运算符"<<"是双目运算符。左移n位就是乘以2的n次方。 其功能把"<<"左边的运算数的各二进位全部左移若干位，由"<<"右边的数指定移动的位数，高位丢弃，低位补0。	A << 2 结果为 240 ，二进制为 1111 0000
>>	右移运算符">>"是双目运算符。右移n位就是除以2的n次方。 其功能是把">>"左边的运算数的各二进位全部右移若干位，">>"右边的数指定移动的位数。	A >> 2 结果为 15 ，二进制为 0000 1111
```
演示示例

```bash
[root@localhost yunsuan]# cat test1.go
package main

import "fmt"

func main() {
   var a int = 60     //二进制是：111100  
   var b int = 13     //二进制是：001101

   fmt.Printf("%b\n%d\n",a&b,a&b)    //二进制是：1100,对应的十进制是12。说明&进行的是上下对应位的与操作
   fmt.Printf("%b\n%d\n",a|b,a|b)    //二进制是：111101,对应的十进制是61。说明&进行的是上下对应位的或操作
   fmt.Printf("%b\n%d\n",a^b,a^b)    //二进制是：110001,对应的十进制是49。^位运算符是上下对应位不同时，值为1
}
[root@localhost yunsuan]# go run test1.go
1100
12
111101
61
110001
49
```

左移右移运算符示例（实现计算器存储单位）：

```bash
[root@localhost yunsuan]# cat test2.go
package main

import "fmt"

const (   
    KB float64 = 1<<(10*iota)      //iota是 const 结构里面，定义常量行数的索引器，每个 const 里面，iota 都从 0 开始
    MB                             //下面是一个省略调用，继承了上面的表达式
    GB
    TB
    PB
)

func main() {
   fmt.Printf("1MB = %vKB\n",MB) 
   fmt.Printf("1GB = %vKB\n",GB)
   fmt.Printf("1TB = %vKB\n",TB)
   fmt.Printf("1PB = %vKB\n",PB)
}
[root@localhost yunsuan]# go run test2.go
1MB = 1024KB
1GB = 1.048576e+06KB
1TB = 1.073741824e+09KB
1PB = 1.099511627776e+12KB
```

## 5.赋值运算符

下表列出了所有Go语言的赋值运算符。假定 A 为21
                                            
```bash
=	简单的赋值运算符，将一个表达式的值赋给一个左值	C = A  将 A 赋值给 C，结果：21
+=	相加后再赋值	C += A 等于 C = C + A，结果：42
-=	相减后再赋值	C -= A 等于 C = C - A，结果：21
*=	相乘后再赋值	C *= A 等于 C = C * A，结果：441
/=	相除后再赋值	C /= A 等于 C = C / A，结果：21
%=	求余后再赋值	C %= A 等于 C = C % A，结果：0//不记入计算
<<=	左移后赋值	C <<= 2 等于 C = C << 2，结果：84
>>=	右移后赋值	C >>= 2 等于 C = C >> 2，结果：21
&=	按位与后赋值	C &= 2 等于 C = C & 2，结果：0  10101 & 10 =0
^=	按位异或后赋值	C ^= 2 等于 C = C ^ 2，结果：2 10101 ^ 10 = 00010
=	按位或后赋值	C |= 2 等于 C = C | 2，结果：2 10101 | 10 = 010
```

## 6.其他运算符

                                                      
```bash
&	返回变量存储地址	&a; 将给出变量的实际地址。
*	指针变量。	*a; 是一个指针变量
```
内存地址和指针的示例：打印变量类型用%T

```bash
[root@localhost yunsuan]# cat test3.go
package main

import "fmt"

func main() {
   var a int = 4
   var b int32
   var c float32
   var ptr *int

   fmt.Printf("a 变量类型为 = %T\n", a )     //输出变量类型%T
   fmt.Printf("b 变量类型为 = %T\n", b )
   fmt.Printf("c 变量类型为 = %T\n", c )

   ptr = &a    
   fmt.Printf("a 的内存地址为 = %p",ptr)     //go里面的内存块地址通常都是用十六进制表示的，因此输出：0x10414020a
   fmt.Printf("*ptr 为 %d\n", *ptr)        //这是个指向a的内存地址的指针，因此输出：4
}
[root@localhost yunsuan]# go run test3.go
a 变量类型为 = int
b 变量类型为 = int32
c 变量类型为 = float32
a 的内存地址为 = 0xc000082010*ptr 为 4
```
7.运算符优先级
有些运算符拥有较高的优先级，二元运算符的运算方向均是从左至右。下表列出了所有运算符以及它们的优先级，由上至下代表优先级由高到低：

```bash
优先级     运算符
 7      ^ !
 6      * / % << >> & &^
 5      + - | ^
 4      == != < <= >= >
 3      <-
 2      &&
 1      ||
```

参考：

 - [https://www.runoob.com/go/go-operators.html](https://www.runoob.com/go/go-operators.html)

