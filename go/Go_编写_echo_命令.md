#  Go 编写 echo 命令

第一种

```bash
package main

import (
    "fmt"
    "os"
)

func main() {
    var s, sep string
    for i := 1; i < len(os.Args); i++ {
        s += sep + os.Args[i]
        sep = " "
    }
    fmt.Println(s)
}
```

```bash
$ go build echo1.go
$./echo1 

$ ./echo1 1
1
$ ./echo1 13 4
13 4
$ /echo1 "hello world"
hello world
```
第二种 利用

```bash
package main

import (
    "fmt"
    "os"
)

func main() {
    s, sep := "", ""
    for _, arg := range os.Args[1:] {
        s += sep + arg
        sep = " "
    }
    fmt.Println(s)
```

}

```bash
$ go build echo2.go
$ ./echo2 3
3
$ ./echo2 3 4
3 4
$ ./echo2 3 4 hello
3 4 hello
```
第三种

```bash
package main

import (
  "fmt"
  "strings"
  "os"
)
func main() {
    fmt.Println(strings.Join(os.Args[1:], " "))
}
```

```bash
$ ./echo3  1 
1
$ ./echo3  1  2
1 2
$ ./echo3  1  2 hello
1 2 hello
```
参考
- 书籍： go圣经
