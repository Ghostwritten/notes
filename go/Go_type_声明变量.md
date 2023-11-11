#  Go type 声明变量

## 1. type 定义
新类型声明格式：(是类型的组合) 

```bash
package main
import (
    "fmt"
    "reflect"
)
type bigint int64
func main() {
    var x bigint = 100
    fmt.Printf("x 的值是：%v\n", x)
    fmt.Printf("x 的类型是：%v\n", reflect.TypeOf(x))
    type smallint int8
    var y smallint = 1
    fmt.Printf("y 的值是：%v\n", y)
    fmt.Printf("y 的类型是：%v\n", reflect.TypeOf(y))
}
```

```bash
[root@localhost struct]# go run stru1.go 
x 的值是：100
x 的类型是：main.bigint
y 的值是：1
y 的类型是：main.smallint
```
新类型不是原类型的别名，除拥有相同数据存储结构外，它们之间没有任何关系，不会持有原类型任何信息。除非目标类型是未命名类型，否则必须显式转换。

```bash
package main
import (
    "fmt"
    "reflect"
)
type bigint int64
type myslice []int
func main() {
    x := 1234
    var b bigint = bigint(x) // 必须显式转换，除非是常量。
    var b2 int64 = int64(b)
    fmt.Printf("b2 的值是：%v , b2 的类型是：%v\n", b2, reflect.TypeOf(b2))
    var s myslice = []int{1, 2, 3} // 未命名类型，隐式转换。
    var s2 []int = s
    fmt.Printf("s2 的值是：%v , s2 的类型是：%v\n", s2, reflect.TypeOf(s2))
}
```

```bash
[root@localhost struct]# go run stru2.go 
b2 的值是：1234 , b2 的类型是：int64
s2 的值是：[1 2 3] , s2 的类型是：[]int
```
