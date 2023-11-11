


-----
## 1 regexp.Match
判断在 b 中能否找到正则表达式 pattern 所匹配的子串
```go
func Match(pattern string, b []byte) (matched bool, err error)
```

 - pattern：要查找的正则表达式
 - b：要在其中进行查找的 []byte
 - matched：返回是否找到匹配项
 - err：返回查找过程中遇到的任何错误

```go
package main

import (
       "regexp"
       "fmt"
)
func main() {
     fmt.Println(regexp.Match("H.* ", []byte("Hello World!")))
}
```
     
```go
[root@localhost regexp]# go run re1.go
true <nil>
```
## 2 regexp.MatchReader()
判断在 r 中能否找到正则表达式 pattern 所匹配的子串
```go
func MatchReader(pattern string, r io.RuneReader) (matched bool, err error)
```

 - pattern：要查找的正则表达式
 - r：要在其中进行查找的 RuneReader 接口
 - matched：返回是否找到匹配项
 - err：返回查找过程中遇到的任何错误

```go
package main

import (
       "regexp"
       "fmt"
       "bytes"
)
func main() {
      r := bytes.NewReader([]byte("Hello World!"))
      fmt.Println(regexp.MatchReader("H.* ", r))
}
```
     
```go
[root@localhost regexp]# go run re2.go
true <nil>
```
## 3 regexp.MatchString()
判断在 s 中能否找到正则表达式 pattern 所匹配的子串
```go
func MatchString(pattern string, s string) (matched bool, err error)
```

 - pattern：要查找的正则表达式
 - r：要在其中进行查找的字符串
 - matched：返回是否找到匹配项
 - err：返回查找过程中遇到的任何错误

```go
package main

import (
       "regexp"
       "fmt"
)
func main() {
     fmt.Println(regexp.MatchString("H.* ", "Hello World!"))

}
```
     
```go
[root@localhost regexp]# go run re3.go
true <nil>
```
## 4 regexp.QuoteMeta
QuoteMeta 将字符串 s 中的“特殊字符”转换为其“转义格式”
 特殊字符有：\.+*?()|[]{}^$，这些字符用于实现正则语法，所以当作普通字符使用时需要转换

```go
func QuoteMeta(s string) string
```

```go
package main

import (
       "regexp"
       "fmt"
)
func main() {
     fmt.Println(regexp.QuoteMeta("(?P:Hello) [a-z]"))
}
```
     
```go
[root@localhost regexp]# go run re4.go
\(\?P:Hello\) \[a-z\]
```
## 5 regexp.Compile()创建 Regexp 对象
 Regexp 结构表示一个编译后的正则表达式
Regexp 的公开接口都是通过方法实现的
多个 goroutine 并发使用一个 RegExp 是安全的

```go
type Regexp struct {
// 私有字段
}
```
 Compile 用来解析正则表达式 expr 是否合法，如果合法，则返回一个 Regexp 对象

```go
 func Compile(expr string) (*Regexp, error)
```

```go
package main

import (
       "regexp"
       "fmt"
)
func main() {
     reg, err := regexp.Compile(`\w+`)
     fmt.Printf("%q,%v\n", reg.FindString("Hello World!"), err)

}
```

```go
[root@localhost regexp]# go run re5.go
"Hello",<nil>
```
## 6 regexp.CompilePOSIX()创建regexp对象
CompilePOSIX 的作用和 Compile 一样，不同的是，CompilePOSIX 使用 POSIX 语法，同时，它采用最左最长方式搜索，而 Compile 采用最左最短方式搜索，POSIX 语法不支持 Perl 的语法格式：\d、\D、\s、\S、\w、\W

```go
func CompilePOSIX(expr string) (*Regexp, error)
```

```go
package main

import (
       "regexp"
       "fmt"
)
func main() {
     reg, err := regexp.CompilePOSIX(`[[:word:]]+`)
     fmt.Printf("%q,%v\n", reg.FindString("Hello World!"), err)

}
```
     
```go
[root@localhost regexp]# go run re6.go
"Hello",<nil>
```

## 7 regexp.MustCompile创建regexp对象
MustCompile 的作用和 Compile 一样，不同的是，当正则表达式 str 不合法时，MustCompile 会抛出异常，而 Compile 仅返回一个 error 值。

```go
func MustCompile(str string) *Regexp
```

```go
package main

import (
       "regexp"
       "fmt"
)
func main() {
     reg := regexp.MustCompile(`\w+`)
     fmt.Println(reg.FindString("Hello World!"))
}
```
     
```go
[root@localhost regexp]# go run re7.go
Hello
```
 ## regexp.MustCompilePOSIX()创建对象
  MustCompilePOSIX 的作用和 CompilePOSIX 一样，不同的是，当正则表达式 str 不合法时，MustCompilePOSIX 会抛出异常，而 CompilePOSIX 仅返回一个 error 值。

```go
func MustCompilePOSIX(str string) *Regexp
```

```go
package main

import (
       "regexp"
       "fmt"
)
func main() {
     reg := regexp.MustCompilePOSIX(`[[:word:]].+ `)
     fmt.Printf("%q\n", reg.FindString("Hello World!"))
}
```
```go
[root@localhost regexp]# go run re8.go
"Hello "
```
## 8 Find()方法
在 byte 中查找 re 中编译好的正则表达式，并返回第一个匹配的内容

```go
func (re *Regexp) Find(b []byte) []byte
```

```go
package main

import (
       "regexp"
       "fmt"
)
func main() {
     reg := regexp.MustCompile(`\w+`)
     fmt.Printf("%q", reg.Find([]byte("Hello World!")))
}
```
     
```go
[root@localhost regexp]# go run re9.go
"Hello"
```
 ## FindString()方法
 在 string 中查找 re 中编译好的正则表达式，并返回第一个匹配的内容

```go
func (re *Regexp) FindString(s string) string
```

```go
package main

import (
       "regexp"
       "fmt"
)
func main() {
     reg := regexp.MustCompile(`\w+`)
     fmt.Println(reg.FindString("Hello World!"))
}
```
     
```go
[root@localhost regexp]# go run re10.go
Hello
```
##  9 FindAll()方法
 在 byte 中查找 re 中编译好的正则表达式，并返回所有匹配的内容{{匹配项}, {匹配项}, ...}，只查找前 n 个匹配项，如果 n < 0，则查找所有匹配项。

```go
func (re *Regexp) FindAll(b []byte, n int) [][]byte
```

```go
package main

import (
       "regexp"
       "fmt"
)
func main() {
     reg := regexp.MustCompile(`\w+`)
     fmt.Printf("%q", reg.FindAll([]byte("Hello World!"), -1))
}
```
     
```go
[root@localhost regexp]# go run re11.go
["Hello" "World"]
```
##  10 FindAllString()方法
 在 string 中查找 re 中编译好的正则表达式，并返回所有匹配的内容{{匹配项}, {匹配项}, ...}，只查找前 n 个匹配项，如果 n < 0，则查找所有匹配项。

```go
func (re *Regexp) FindAllString(s string, n int) []string
```

```go
package main

import (
       "regexp"
       "fmt"
)
func main() {
     reg := regexp.MustCompile(`\w+`)
     fmt.Printf("%q", reg.FindAllString(("Hello World!"), -1))
}
```
     
```go
[root@localhost regexp]# go run re12.go
["Hello" "World"]
```
## 11 FindIndex()方法
在 b 中查找 re 中编译好的正则表达式，并返回第一个匹配的位置{起始位置, 结束位置}

```go
func (re *Regexp) FindIndex(b []byte) (loc []int)
```

```go
package main

import (
       "regexp"
       "fmt"
)
func main() {
     reg := regexp.MustCompile(`\w+`)
     fmt.Printf("%q", reg.FindIndex([]byte("Hello World!")))
}
```
     
```go
[root@localhost regexp]# go run re13.go
['\x00' '\x05']
```
## 12 FindStringIndex()方法

在 s 中查找 re 中编译好的正则表达式，并返回第一个匹配的位置{起始位置, 结束位置}

```go
func (re *Regexp) FindStringIndex(s string) (loc []int)
```

```go
package main

import (
       "regexp"
       "fmt"
)
func main() {
     reg := regexp.MustCompile(`\w+`)
     fmt.Printf("%q", reg.FindStringIndex("Hello World!"))
}
```
     
```go
[root@localhost regexp]# go run re14.go
['\x00' '\x05']
```

## 13 FindReaderIndex()方法
在 r 中查找 re 中编译好的正则表达式，并返回第一个匹配的位置{起始位置, 结束位置}

```go
func (re *Regexp) FindReaderIndex(r io.RuneReader) (loc []int)
```

```go
package main

import (
       "regexp"
       "fmt"
       "bytes"
)
func main() {
     r := bytes.NewReader([]byte("Hello World!"))
     reg := regexp.MustCompile(`\w+`)
     fmt.Println(reg.FindReaderIndex(r))
}
```
  
```go
[root@localhost regexp]# go run re15.go
[0 5]
```
 ## 14 FindAllIndex()方法   
在 b 中查找 re 中编译好的正则表达式，并返回所有匹配的位置 {{起始位置, 结束位置}, {起始位置, 结束位置}, ...}，只查找前 n 个匹配项，如果 n < 0，则查找所有匹配项

```go
func (re *Regexp) FindAllIndex(b []byte, n int) [][]int
```

```go
package main

import (
       "regexp"
       "fmt"
)
func main() {
     reg := regexp.MustCompile(`\w+`)
     fmt.Println(reg.FindAllIndex([]byte("Hello World!"), -1))
}
```
     
```go
[root@localhost regexp]# go run re16.go
[[0 5] [6 11]]
```
  ## 15 FindAllStringIndex()方法    


在 b 中查找 re 中编译好的正则表达式，并返回所有匹配的位置 {{起始位置, 结束位置}, {起始位置, 结束位置}, ...},只查找前 n 个匹配项，如果 n < 0，则查找所有匹配项.

```go
func (re *Regexp) FindAllStringIndex(s string, n int) [][]int
```

```go
package main

import (
       "regexp"
       "fmt"
)
func main() {
     reg := regexp.MustCompile(`\w+`)
     fmt.Println(reg.FindAllStringIndex("Hello World!", -1))
}
```
```go
[root@localhost regexp]# go run re17.go
[[0 5] [6 11]]
```

参考连接：
[https://blog.csdn.net/weiyuefei/article/details/78589764](https://blog.csdn.net/weiyuefei/article/details/78589764)
