
## 1. math/rand
对于Random的使用，在业务中使用频率是非常高的，本文就小结下常用的方法：

在Golang中，有两个包提供了rand，分别为 "math/rand" 和 "crypto/rand",  对应两种应用场景。

一、"math/rand" 包实现了伪随机数生成器。也就是生成 整形和浮点型。

　　 该包中根据生成伪随机数是是否有种子(可以理解为初始化伪随机数)，可以分为两类：

　

 - 1、有种子。通常以时钟，输入输出等特殊节点作为参数，初始化。该类型生成的随机数相比无种子时重复概率较低。
 - 2、无种子。可以理解为此时种子为1， Seek(1)。

```go
package main
import (
    "fmt"
    "math/rand"
    "time"
)

func main() {
    for i := 0; i < 10; i++ {
        r := rand.New(rand.NewSource(time.Now().UnixNano()))
        fmt.Printf("%d ", r.Int31())
    }

    fmt.Println("")
    for i := 0; i < 10; i++ {
        fmt.Printf("%d ", rand.Int31())
    }
}
```

```bash
$ go run test.go
999449542 1913657863 267766011 1688476173 1412739146 1499168354 1585121329 858280757 661414473 759656463 
1298498081 2019727887 1427131847 939984059 911902081 1474941318 140954425 336122540 208240456 646203300 
```
常用的方法有：(以有种子的为例，无种子的直接通过 rand 报名调用对应的方法)

 - **按类型随机类：**

```bash
func (r *Rand) Int() int
func (r *Rand) Int31() int32
func (r *Rand) Int63() int64
func (r *Rand) Uint32() uint32
func (r *Rand) Float32() float32  // 返回一个取值范围在[0.0, 1.0)的伪随机float32值
func (r *Rand) Float64() float64  // 返回一个取值范围在[0.0, 1.0)的伪随机float64值
```

 - **指定随机范围类：**

```bash
func (r *Rand) Intn(n int) int
func (r *Rand) Int31n(n int32) int32
func (r *Rand) Int63n(n int64) int64
```

拓展：对于需要随机指定位数的，当位数不够是，可以通过前边补0达到长度一致，如：

```go
package main
import (
    "fmt"
    "math/rand"
)

func main() {
    for i := 0; i < 10; i++ {
        fmt.Printf("%.4d ", rand.Int31()%10000)
    }
}// 8081 7887 1847 4059 2081 1318 4425 2540 0456 3300
```
### 1.1 rand.Intn
练习1 输出10位随肌符
```bash
package main

import (
	"fmt"
	"math/rand"
	"time"
)
func main() {

  str := "0123456789abcdefghijklmnopqrstuvwxyz"
  bytes := []byte(str)
  result := []byte{}
  r := rand.New(rand.NewSource(time.Now().UnixNano())) 
  fmt.Println(&r)
  for i := 0; i < 10; i++ {
     result = append(result, bytes[r.Intn(len(bytes))])
  }
  fmt.Println(result)
  fmt.Println(string(result))
```
输出

```bash
PS D:\gitlab\cronserver\test> go run test3.go
0xc000080018
[117 100 118 110 51 114 98 101 115 104]
udvn3rbesh
PS D:\gitlab\cronserver\test> go run test3.go
0xc000080018
[103 122 113 117 53 100 53 54 49 110]
gzqu5d561n
```
练习2

```bash
package main

import ("fmt"
        "math/rand"
        "time")

func main() {
    // 初始化随机数的资源库, 如果不执行这行, 不管运行多少次都返回同样的值
    rand.Seed(time.Now().UnixNano())
    fmt.Println("A number from 1-100", rand.Intn(81))
}
```

```bash
PS D:\gitlab\cronserver\test> go run test4.go
A number from 1-100 63
PS D:\gitlab\cronserver\test> go run test4.go
A number from 1-100 22
```

## 2. crypto/rand
包实现了用于加解密的更安全的随机数生成器。
该包中常用的是 `func Read(b []byte) (n int, err error)` 这个方法， 将随机的byte值填充到b 数组中，以供b使用。示例如下：

```bash
import (
    "crypto/rand"
    "fmt"
)

func main() {
    b := make([]byte, 20)
    fmt.Println(b)       //

    _, err := rand.Read(b)
    if err != nil {
        fmt.Println(err.Error())
    }

    fmt.Println(b)
}

// [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
// [68 221 145 73 115 224 13 110 218 130 19 139 38 170 145 58 251 188 126 197]
```

