#  Go 包 os 文件操作

![--------](https://i-blog.csdnimg.cn/blog_migrate/d432911b4c301d3292aadcbb6ac45123.png)





## 1 简介
Go 在 os 中提供了文件的基本操作，包括通常意义的打开、创建、读写等操作，除此以外为了追求便捷以及性能上，Go 还在 io/ioutil 以及 bufio 提供一些其他函数供开发者使用
## 2 练习
### 2.1 环境变量
#### 2.1.1 获取所有环境变量, 返回变量列表

```bash
func Environ() []string
```

```bash
package main
 
import (
    "fmt"
    "os"
    "strings"
)
 
func main() {
    envs := os.Environ()
    for _, env := range envs {
        cache := strings.Split(env, "=")
        fmt.Printf(`
        key: %s value: %s
        `, cache[0], cache[1])
    }
}
```

```bash
[root@localhost os]# go run os1.go

        key: XDG_SESSION_ID value: 27
        
        key: GUESTFISH_INIT value: \e[1;34m
        
        key: HOSTNAME value: localhost.localdomain
        ......
```
#### 2.1.2 获取指定环境变量

```bash
func Getenv(key string) string
```

```bash
package main
 
import (
    "fmt"
    "os"
)
 
func main() {
    fmt.Println(os.Getenv("GOPATH"))
}
```

```bash
[root@localhost os]#  go run os2.go
/usr/local/gopath
```

#### 2.1.3 设置环境变量
	
```bash
func Setenv(key, value string) error
```

```bash
package main
 
import (
    "fmt"
    "os"
)
 
func main() {
    fmt.Println(os.Getenv("GOPATH"))
 
    if err := os.Setenv("GOPATH", "./GO/bin"); err != nil {
        fmt.Println(err)
    } else {
        fmt.Println("success")
    }
}
```

```bash
[root@localhost os]#  go run os3.go
/usr/local/gopath
success
```
#### 2.1.4 清除所有环境变量

```bash
func os.Clearenv()
```

```bash
func main() {
 data := os.Environ() 
 fmt.Println(data)
 os.Clearenv() 
 data = os.Environ()
 fmt.Println(data) 
}
```
### 2.2 文件打开方式与打开模式（运用OpenFile函数）
//打开方式

```bash
const (
//只读模式
O_RDONLY int = syscall.O_RDONLY // open the file read-only.
//只写模式
O_WRONLY int = syscall.O_WRONLY // open the file write-only.
//可读可写
O_RDWR int = syscall.O_RDWR // open the file read-write.
//追加内容
O_APPEND int = syscall.O_APPEND // append data to the file when writing.
//创建文件,如果文件不存在
O_CREATE int = syscall.O_CREAT // create a new file if none exists.
//与创建文件一同使用,文件必须存在
O_EXCL int = syscall.O_EXCL // used with O_CREATE, file must not exist
//打开一个同步的文件流
O_SYNC int = syscall.O_SYNC // open for synchronous I/O.
//如果可能,打开时缩短文件
O_TRUNC int = syscall.O_TRUNC // if possible, truncate file when opened.
)
```

打开模式
```bash
const (
    // 单字符是被String方法用于格式化的属性缩写。
    ModeDir        FileMode = 1 << (32 - 1 - iota) // d: 目录
    ModeAppend                                     // a: 只能写入，且只能写入到末尾
    ModeExclusive                                  // l: 用于执行
    ModeTemporary                                  // T: 临时文件（非备份文件）
    ModeSymlink                                    // L: 符号链接（不是快捷方式文件）
    ModeDevice                                     // D: 设备
    ModeNamedPipe                                  // p: 命名管道（FIFO）
    ModeSocket                                     // S: Unix域socket
    ModeSetuid                                     // u: 表示文件具有其创建者用户id权限
    ModeSetgid                                     // g: 表示文件具有其创建者组id的权限
    ModeCharDevice                                 // c: 字符设备，需已设置ModeDevice
    ModeSticky                                     // t: 只有root/创建者能删除/移动文件
    // 覆盖所有类型位（用于通过&获取类型位），对普通文件，所有这些位都不应被设置
    ModeType = ModeDir | ModeSymlink | ModeNamedPipe | ModeSocket | ModeDevice
    ModePerm FileMode = 0777 // 覆盖所有Unix权限位（用于通过&获取类型位）
)
```
### 2.3 文件信息

```bash
type FileInfo interface {
    Name() string       // 文件的名字（不含扩展名）
    Size() int64        // 普通文件返回值表示其大小；其他文件的返回值含义各系统不同
    Mode() FileMode     // 文件的模式位
    ModTime() time.Time // 文件的修改时间
    IsDir() bool        // 等价于Mode().IsDir()
    Sys() interface{}   // 底层数据来源（可以返回nil）
}
```
#### 2.3.1 获取文件信息对象, 符号链接将跳转

```bash
func Stat(name string) (fi FileInfo, err error)
```

```bash
package main
 
import (
    "fmt"
    "os"
)
 
func main() {
     fi, _ := os.Stat("./1.txt")
 
     fmt.Println(fi.Size())
}
```

```bash
[root@localhost os]# go run os4.go 
6
```
#### 2.3.2 获取文件信息对象, 符号链接不跳转
package main
 
import (
    "fmt"
    "os"
)
 
func main() {
    fi, _ := os.Lstat("./1.txt")
 
    fmt.Println(fi.Size())
}
[root@localhost os]# go run os5.go
6
#### 2.3.3 （重点）根据错误，判断 文件或目录是否存在

```bash
package main
 
import (
    "fmt"
    "os"
)
 
func main() {
    if _, err := os.Open("./empty.js"); err != nil {
        // false 不存在   true 存在
        emptyErr := os.IsExist(err)
        fmt.Println(emptyErr, "\n", err)
    }
}
```

```bash
[root@localhost os]#  go run os6.go
false 
 open ./empty.js: no such file or directory
```

#### 2.3.4 IsExist 反义方法

```bash
package main
 
import (
    "fmt"
    "os"
)
 
func main() {
    if _, err := os.Open("./empty.js"); err != nil {
        // false 不存在   true 存在
        emptyErr := os.IsNotExist(err)
        fmt.Println(emptyErr, "\n", err)
    }
}
```

```bash
[root@localhost os]# go run os7.go
true 
 open ./empty.js: no such file or directory
```
#### 2.3.5 根据错误，判断是否为权限错误

```bash
package main
 
import (
    "fmt"
    "os"
)
 
func main() {
    file, _ := os.Open("./cache.js")
    _, err := file.WriteString("// new info")
 
    if err != nil {
        fmt.Println(os.IsPermission(err))
    }
    defer file.Close()
}
```

```bash
[root@localhost os]#  go run os8.go
false
```
### 2.4 文件/目录操作
属性操作
#### 2.4.1 （重点）获取当前工作目录

```bash
func Getwd() (dir string, err error)
```

```bash
package main
 
import (
    "fmt"
    "os"
)
 
func main() {
    path, _ := os.Getwd()
    fmt.Println(path)
}
```

```bash
[root@localhost os]# pwd
/root/go/os
[root@localhost os]#  go run  os9.go
/root/go/os
```
#### 2.4.2 修改当前，工作目录

```bash
func Chdir(dir string) error
```

```bash
package main
 
import (
    "fmt"
    "os"
)
 
func main() {
    path1, _ := os.Getwd()
    fmt.Println(path1)
    os.Chdir("./../")
    path, _ := os.Getwd()
    fmt.Println(path)
 
}
```

```bash
[root@localhost os]#  go run os10.go
/root/go/os
/root/go
```

#### 2.4.3 修改文件的 FileMode

```bash
func Chmod(name string, mode FileMode) error
```

#### 2.4.4 修改文件的 访问时间和修改时间

```bash
func Chtimes(name string, atime time.Time, mtime time.Time) error
```

```bash
package main
 
import (
    "fmt"
    "os"
    "time"
)
 
func main() {
    fmt.Println(os.Getwd())
 
    path := "test.txt"
    os.Chtimes(path, time.Now(), time.Now())
 
    fi, _ := os.Stat(path)
    fmt.Println(fi.ModTime())
 
}
```

```bash
[root@localhost os]# go run os11.go
/root/go/os <nil>
2020-04-02 12:12:37.525936298 +0800 CST
```
 下面是：增删改查
#### 2.4.5 （重点）创建目录

```bash
func Mkdir(name string, perm FileMode) error
```

```bash
package main
 
import (
    "fmt"
    "os"
)
 
func main() {
    if err := os.Mkdir("test", os.ModeDir); err != nil {
        fmt.Println(err)
    } else {
        fmt.Println("success")
    }
 
}
```

```bash
[root@localhost os]# go run os12.go 
success
[root@localhost os]# go run os12.go 
mkdir test: file exists
```
#### 2.4.6 递归创建目录

```bash
func MkdirAll(path string, perm FileMode) error
```

```bash
package main
 
import (
    "fmt"
    "os"
)
 
func main() {
    if err := os.MkdirAll("test01/test", os.ModeDir); err != nil {
        fmt.Println(err)
    } else {
        fmt.Println("success")
    }
 
}
```

```bash
[root@localhost os]#  go run os13.go
success
```
#### 2.4.7 移除文件或目录(单一文件)

```bash
func Remove(name string) error
```

```bash
package main
import (
    "fmt"
    "os"
)
 
func main() {
    if err := os.Remove("test"); err != nil {
        fmt.Println(err)
    } else {
        fmt.Println("success")
    }
}
```

```bash
[root@localhost os]#  go run os14.go
success
[root@localhost os]# ls test
ls: 无法访问test: 没有那个文件或目录
[root@localhost os]#  go run os14.go
remove test: no such file or directory
```
#### 2.4.8 递归删除文件或目录

```bash
func RemoveAll(path string) error
```

```bash
package main
 
import (
    "fmt"
    "os"
)
 
func main() {
    if err := os.RemoveAll("test01"); err != nil {
        fmt.Println(err)
    } else {
        fmt.Println("success")
    }
} 
```

```bash
[root@localhost os]#  go run os15.go
success
```
#### 2.4.8 文件重名或移动

```bash
func Rename(oldpath, newpath string) error
```

```bash
package main
 
import (
    "fmt"
    "os"
)
 
func main() {
 
    // 重命名
    err := os.Rename("test.txt", "test01.js")
    if err != nil {
        fmt.Println(err)
    }
    err = os.Mkdir("test", os.ModeDir)
    if err != nil {
        fmt.Println(err)
    }
 
    // 移动
    err = os.Rename("test01.js", "test/text01.txt")
    if err != nil {
        fmt.Println(err)
    }
}
```

```bash
[root@localhost os]#  go run os16.go
[root@localhost os]#  go run os16.go
rename test.txt test01.js: no such file or directory
mkdir test: file exists
rename test01.js test/text01.txt: no such file or directory
[root@localhost os]# ls test/
text01.txt
```
#### 2.4.9 修改文件大小

```bash
func Truncate(name string, size int64) error
```

```bash
package main
 
import (
    "fmt"
    "os"
)
 
func main() {
 
    path := "test/text01.txt"
    fi, err := os.Stat(path)
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
 
    size := fi.Size()
    fmt.Println(size)
 
    // 截取长度
    size = int64(float64(size) * 0.5)
 
    os.Truncate(path, size)
 
    fi, _ = os.Stat(path)
 
    fmt.Println(fi.Size())
}
```

```bash
[root@localhost os]# cat test/text01.txt
abc
123
[root@localhost os]# go run os17.go
&{text01.txt 8 420 {99442120 63721429367 0x570b80} {64768 3725573 1 33188 0 0 0 0 8 4096 8 {1585832567 99442120} {1585832567 99442120} {1585832567 101442065} [0 0 0]}}
8
4
[root@localhost os]# cat test/text01.txt   #123被删除
abc
```
#### 2.4.10 比较两个文件信息对象，是否指向同一文件

```bash
func SameFile(fi1, fi2 FileInfo) bool
```

```bash
package main
 
import (
    "fmt"
    "os"
)
 
func main() {
 
    path := "test/text01.txt"
 
    fi_1, _ := os.Stat(path)
    fi_2, _ := os.Stat(path)
 
    fmt.Println(os.SameFile(fi_1, fi_2))
}
```

```bash
[root@localhost os]# vim os18.go^C
[root@localhost os]# go run os18.go 
true
```
### 2.5 文件/目录对象
#### 2.5.1 （重点）创建文件, 如果文件存在，清空原文件

```bash
func Create(name string) (file *File, err error)
```

```bash
package main
import (
    "fmt"
    "os"
)
 
func main() {
 
    file, _ := os.Create("./new_file.js")
    fmt.Println(file.Name())
}
```

```bash
[root@localhost os]# go run os19.go
./new_file.js
[root@localhost os]# ls new_file.js 
new_file.js
```
#### 2.5.2 打开文件，获取文件对象, 以读取模式打开

Open打开一个文件用于读取。如果操作成功，返回的文件对象的方法可用于读取数据；对应的文件描述符具有O_RDONLY模式。如果出错，错误底层类型是*PathError。 
所以，Open()只能用于读取文件。

```bash
func Open(name string) (file *File, err error)
```

```bash
package main
 
import (
    "fmt"
    "os"
)
 
func main() {
 
    file, _ := os.Open("./new_file.js")
    fmt.Println(file.Name())
}
```

```bash
[root@localhost os]# go run os20.go
./new_file.js
[root@localhost os]# rm -rf new_file.js 
[root@localhost os]# go run os20.go
panic: runtime error: invalid memory address or nil pointer dereference
[signal SIGSEGV: segmentation violation code=0x1 addr=0x0 pc=0x48d6f0]

goroutine 1 [running]:
os.(*File).Name(...)
	/usr/local/go/src/os/file.go:54
main.main()
	/root/go/os/os20.go:11 +0x50
exit status 2
```
#### 2.5.3 （重点）以指定模式，打开文件

```bash
func OpenFile(name string, flag int, perm FileMode) (file *File, err error)
```

```bash
package main
 
import (
    "fmt"
    "os"
)
 
func main() {
 
    file, _ := os.OpenFile("./new_file.js", os.O_RDONLY, os.ModeAppend)
    fmt.Println(file.Name())
}
```

```bash
[root@localhost os]# cat new_file.js 
[root@localhost os]# go run os21.go
./new_file.js
```
### 2.6 文件对象属性操纵

#### 2.6.1 获取文件路径
Name
```bash
func (f *File) Name() string
```


#### 2.6.2 获取文件信息对象
Stat
```bash
func (f *File) Stat() (fi FileInfo, err error)
```


#### 2.6.3 将当前工作路径修改为文件对象目录， 文件对象必须为目录, 该接口不支持window
Chdir

```bash
func (f *File) Chdir() error
```
#### 2.6.4 修改文件模式
Chmod

```bash
func (f *File) Chmod(mode FileMode) error
```

Truncate
#### 2.6.5 修改文件对象size

```bash
func (f *File) Truncate(size int64) error
```
### 2.7 文件对象读写操作

#### 2.7.1 读取文件内容, 读入长度取决 容器切片长度
Read

```bash
func (f *File) Read(b []byte) (n int, err error)
```


```bash
package main
 
import (
    "fmt"
    "os"
)
 
func main() {
 
    bt := make([]byte, 10)
    file, _ := os.Open("./new_file.js")
 
    file.Read(bt)
    defer file.Close()
 
    fmt.Println(string(bt))
}
```

```bash
[root@localhost os]# cat new_file.js 
12345679890
nihao,my name is ghostwritten
[root@localhost os]# go run os22.go
1234567989
```


#### 2.7.2 从某位置，读取文件内容
ReadAt

```bash
func (f *File) ReadAt(b []byte, off int64) (n int, err error)
```

```bash
package main
 
import (
    "fmt"
    "os"
)
 
func main() {
 
    bt := make([]byte, 100)
    file, _ := os.Open("test/text01.txt")
 
    file.ReadAt(bt, 2)
 
    fmt.Println(string(bt))
}
```

```bash
[root@localhost os]# cat test/text01.txt 
abcdefgh
[root@localhost os]#  go run os23.go
cdefgh
```

#### 2.7.3 （重点）写入内容
Write

```bash
func (f *File) Write(b []byte) (n int, err error)
```

```bash
package main
 
import (
    "fmt"
    "os"
)
 
func main() {
 
    file, err := os.OpenFile("test/text01.txt", os.O_RDWR, os.ModeAppend)
    if err != nil {
        fmt.Println("err: ", err)
        os.Exit(1)
    }
 
    defer file.Close()
 
    if n, err := file.Write([]byte("// new info")); err != nil {
        fmt.Println(err)
    } else {
        fmt.Println(n)
    }
 
}
```

```bash
[root@localhost os]# go run os24.go
11
[root@localhost os]# cat test/text01.txt 
// new info
```

#### 2.7.4 写入字符
WriteString

```bash
func (f *File) WriteString(s string) (ret int, err error)
```

```bash
package main
 
import (
    "fmt"
    "os"
)
 
func main() {
 
    file, err := os.OpenFile("test/text01.txt", os.O_RDWR, os.ModeAppend)
    if err != nil {
        fmt.Println("err: ", err)
        os.Exit(1)
    }
 
    defer file.Close()
 
    if n, err := file.Write([]byte("// new info")); err != nil {
        fmt.Println(err)
    } else {
        fmt.Println(n)
    }
 
}
```

```bash
[root@localhost os]# go run os25.go
12
[root@localhost os]# cat test/text01.txt 
// test info
```

#### 2.7.5 从指定位置，写入
WriteAt

```bash
func (f *File) WriteAt(b []byte, off int64) (n int, err error)
```

```bash
package main
 
import (
    "fmt"
    "os"
)
 
func main() {
 
    file, err := os.OpenFile("test/text01.txt", os.O_RDWR, os.ModeAppend)
    if err != nil {
        fmt.Println("err: ", err)
        os.Exit(1)
    }
 
    defer file.Close()
 
    if n, err := file.WriteAt([]byte(" append "), 5); err != nil {
        fmt.Println(err)
    } else {
        fmt.Println(n)
    }
 
}
```

```bash
[root@localhost os]# go run os26.go
8
[root@localhost os]# cat test/text01.txt 
// te append 
```

#### 2.7.6 设置下次读写位置
Seek

```bash
func (f *File) Seek(offset int64, whence int) (ret int64, err error)
```

```bash
package main
 
import (
    "fmt"
    "os"
)
 
func main() {
 
    f, err := os.OpenFile("test/text01.txt", os.O_RDWR, os.ModeAppend)
    if err != nil {
        fmt.Println("err: ", err)
        os.Exit(1)
    }
 
    defer f.Close()
 
    f.Seek(2, 0)
    buffer := make([]byte, 5)
    // Read 后文件指针也会偏移
 
    n, err := f.Read(buffer)
    if err != nil {
        fmt.Println(nil)
        return
    }
    fmt.Printf("n is %d, buffer content is : %s\n", n, buffer)
    // 获取文件指针当前位置
    cur_offset, _ := f.Seek(0, os.SEEK_CUR)
    fmt.Printf("current offset is %d\n", cur_offset)
 
}
```

```bash
[root@localhost os]# go run os27.go
n is 5, buffer content is : 34567
current offset is 7
[root@localhost os]# cat test/text01.txt 
1234567890abcdefg
```
参考：
- [https://www.cnblogs.com/saryli/p/11691142.html](https://www.cnblogs.com/saryli/p/11691142.html)

- [https://www.cnblogs.com/wdliu/p/9239140.html](https://www.cnblogs.com/wdliu/p/9239140.html)
