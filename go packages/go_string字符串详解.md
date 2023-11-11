

---
## 1. 前言
字符串是不可变值类型，内部用指针指向 UTF-8 字节数组。

 - 默认值是空字符串 “”。
 - 用索引号访问某字节，如 s[i]。
 - 不能用序号获取字节元素指针，&s[i] 非法。
 - 不可变类型，无法修改字节数组。
 - 字节数组尾部不包含 NULL。
 
 
## 2. 特殊处理string
### 2.1 使用索引号访问字符 (byte)

```go
package main
func main() {
    s := "abc"
    println(s[0] == '\x61', s[1] == 'b', s[2] == 0x63)
}
输出结果:

true true true
```

### 2.3 使用 “ ` “ 定义不做转义处理的原始字符串，支持跨行

```go
package main
func main() {
    s := `a
b\r\n\x00
c`
    println(s)
}
输出结果:
a
b\r\n\x00
c
```

### 2.4 连接跨行字符串时，”+” 必须在上一行末尾，否则导致编译错误

```go
package main
import (
    "fmt"
)
func main() {
    s := "Hello, " +
        "World!"
    // s2 := "Hello, "
    // +"World!" 
    //./main.go:11:2: invalid operation: + untyped string
    fmt.Println(s)
}

输出结果:
Hello, World!
```

### 2.5 支持用两个索引号 ([]) 返回子串

串依然指向原字节数组，仅修改了指针和度属性。

```go
package main
import (
    "fmt"
)
func main() {
    s := "Hello, World!"
    s1 := s[:5]  // Hello
    s2 := s[7:]  // World!
    s3 := s[1:5] // ello
    fmt.Println(s, s1, s2, s3)
}
输出结果：

Hello, World! Hello World! ello
```

### 2.6 单引号字符常量表示 Unicode Code Point

支持 \uFFFF、\U7FFFFFFF、\xFF 格式， 对应 rune 类型，UCS-4。

```go
package main
import (
    "fmt"
)
func main() {
    fmt.Printf("%T\n", 'a')
    var c1, c2 rune = '\u6211', '们'
    println(c1 == '我', string(c2) == "\xe4\xbb\xac")
}
输出结果:
int32 // rune 是 int32 的别名
true true
```

### 2.7 要修改字符串，可先将其转换成 []rune 或 []byte

完成后再转换为 string。无论哪种转换，都会重新分配内存，并复制字节数组。

```go
package main
func main() {
    s := "abcd"
    bs := []byte(s)
    bs[1] = 'B'
    println(string(bs))
    u := "电脑"
    us := []rune(u)
    us[1] = '话'
    println(string(us))
}
输出结果:
aBcd
电话
```

### 2.8 for 循环遍历字符串时，也有 byte 和 rune 两种方式

```go
package main
import (
    "fmt"
)
func main() {
    s := "abc汉字"
    for i := 0; i < len(s); i++ { // byte
        fmt.Printf("%c,", s[i])
    }
    fmt.Println()
    for _, r := range s { // rune
        fmt.Printf("%c,", r)
    }
    fmt.Println()
}
输出结果:
a,b,c,æ,±,,å,­,,
a,b,c,汉,字,
```

## 3. strings处理

### 3.1 判断是不是以某个字符串开头

```go
package main
import (
    "fmt"
    "strings"
)
func main() {
    str := "hello world"
    res0 := strings.HasPrefix(str, "http://")
    res1 := strings.HasPrefix(str, "hello")
    fmt.Printf("res0 is %v\n", res0)
    fmt.Printf("res1 is %v\n", res1)
}
输出结果：

res0 is false

res1 is true
```

### 3.2 判断是不是以某个字符串结尾

```go
package main
import (
    "fmt"
    "strings"
)
func main() {
    str := "hello world"
    res0 := strings.HasSuffix(str, "http://")
    res1 := strings.HasSuffix(str, "world")
    fmt.Printf("res0 is %v\n", res0)
    fmt.Printf("res1 is %v\n", res1)
}
输出结果：

res0 is false

res1 is true
```

### 3.3 判断str在s中首次出现的位置，如果没有返回-1

```go
package main
import (
    "fmt"
    "strings"
)
func main() {
    str := "hello world"
    res0 := strings.Index(str, "o")
    res1 := strings.Index(str, "i")
    fmt.Printf("res0 is %v\n", res0)
    fmt.Printf("res1 is %v\n", res1)
}
输出结果：

res0 is 4

res1 is -1
```

### 3.4 判断str在s中最后一次出现的位置，如果没有返回-1

```go
package main
import (
    "fmt"
    "strings"
)
func main() {
    str := "hello world"
    res0 := strings.LastIndex(str, "o")
    res1 := strings.LastIndex(str, "i")
    fmt.Printf("res0 is %v\n", res0)
    fmt.Printf("res1 is %v\n", res1)
}
输出结果：

res0 is 7

res1 is -1
```

### 3.5 字符串替换

```go
package main
import (
    "fmt"
    "strings"
)
func main() {
    str := "hello world world"
    res0 := strings.Replace(str, "world", "golang", 2)
    res1 := strings.Replace(str, "world", "golang", 1)
    //trings.Replace("原字符串", "被替换的内容", "替换的内容", 替换次数)
    fmt.Printf("res0 is %v\n", res0)
    fmt.Printf("res1 is %v\n", res1)
}
输出结果：

res0 is hello golang golang

res1 is hello golang world
```

### 3.6 求str含s的次数

```go
package main
import (
    "fmt"
    "strings"
)
func main() {
    str := "hello world world"
    countTime0 := strings.Count(str, "o")
    countTime1 := strings.Count(str, "i")
    fmt.Printf("countTime0 is %v\n", countTime0)
    fmt.Printf("countTime1 is %v\n", countTime1)
}
输出结果：

countTime0 is 3

countTime1 is 0
```

### 3.7 重复 n 次 str

```go
package main
import (
    "fmt"
    "strings"
)
func main() {
    str := "hello world "
    res0 := strings.Repeat(str, 0)
    res1 := strings.Repeat(str, 1)
    res2 := strings.Repeat(str, 2)
    // strings.Repeat("原字符串", 重复次数)
    fmt.Printf("res0 is %v\n", res0)
    fmt.Printf("res1 is %v\n", res1)
    fmt.Printf("res2 is %v\n", res2)
}
输出结果：

res0 is

res1 is hello world

res2 is hello world hello world
```

### 3.8 str 转为大写

```go
package main
import (
    "fmt"
    "strings"
)
func main() {
    str := "hello world "
    res := strings.ToUpper(str)
    fmt.Printf("res is %v\n", res)
}
输出结果：

res is HELLO WORLD
```

### 3.9 str 转为小写

```go
package main
import (
    "fmt"
    "strings"
)
func main() {
    str := "HELLO WORLD "
    res := strings.ToLower(str)
    fmt.Printf("res is %v\n", res)
}
输出结果：

res is hello world
```

### 3.10 去掉 str 首尾的空格

```go
package main
import (
    "fmt"
    "strings"
)
func main() {
    str := "     hello world     "
    res := strings.TrimSpace(str)
    fmt.Printf("res is %v\n", res)
}

输出结果：
res is hello world
```

### 3.11 去掉字符串首尾指定的字符

```go
package main
import (
    "fmt"
    "strings"
)
func main() {
    str := "hi , hello world , hi"
    res := strings.Trim(str, "hi")
    fmt.Printf("res is %v\n", res)
}
输出结果：

res is , hello world ,
```

### 3.12 去掉字符串首指定的字符

```go
package main
import (
    "fmt"
    "strings"
)
func main() {
    str := "hi , hello world , hi"
    res := strings.TrimLeft(str, "hi")
    fmt.Printf("res is %v\n", res)
}
输出结果：

res is , hello world , hi
```

### 3.13 去掉字符串尾指定的字符

```go
package main
import (
    "fmt"
    "strings"
)
func main() {
    str := "hi , hello world , hi"
    res := strings.TrimRight(str, "hi")
    fmt.Printf("res is %v\n", res)
}
输出结果：

res is hi , hello world ,
```

### 3.13 返回str空格分隔的所有子串的slice,

```go
package main
import (
    "fmt"
    "strings"
)
func main() {
    str := "hello world ，hello golang"
    res := strings.Fields(str)
    fmt.Printf("res is %v\n", res)
}
输出结果：

res is [hello world ，hello golang]
```

### 3.14 返回str 指定字符分隔的所有子串的slice

```go
package main
import (
    "fmt"
    "strings"
)
func main() {
    str := "hello world ，hello golang"
    res := strings.Split(str, "o")
    fmt.Printf("res is %v\n", res)
}
输出结果：

res is [hell w rld ，hell g lang]
```

### 3.15 用指定字符将 string 类型的 slice 中所有元素链接成一个字符串

```go
package main
import (
    "fmt"
    "strings"
)
func main() {
    str := []string{"hello", "world", "hello", "golang"}
    res := strings.Join(str, "++")
    fmt.Printf("res is %v\n", res)
    /*
        num := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 0}
        res1 := strings.Join(num, "++")
        //  cannot use num (type []int) as type []string in argument to strings.Join
        fmt.Println(res1)
    */
}
输出结果：

res is hello++world++hello++golang
```
参考连接：
[http://www.ahadoc.com/read/Golang-Detailed-Explanation/ch2.3.2.md#string%E7%9A%84%E5%BA%95%E5%B1%82%E5%B8%83%E5%B1%80](http://www.ahadoc.com/read/Golang-Detailed-Explanation/ch2.3.2.md#string%E7%9A%84%E5%BA%95%E5%B1%82%E5%B8%83%E5%B1%80)









