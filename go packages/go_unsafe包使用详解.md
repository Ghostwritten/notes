

---
## 1.unsafe 作用

从golang的定义来看，unsafe 是类型安全的操作。顾名思义，它应该非常谨慎地使用; unsafe可能很危险，但也可能非常有用。例如，当使用系统调用和Go结构必须具有与C结构相同的内存布局时，您可能别无选择，只能使用unsafe。关于指针操作，在unsafe包官方定义里有四个描述：

 - 任何类型的指针都可以被转化为Pointer
 - Pointer可以被转化为任何类型的指针
 - uintptr可以被转化为Pointer
 - Pointer可以被转化为uintptr

额外在加上一个规则：指向不同类型数据的指针，是无法直接相互转换的，必须借助`unsafe.Pointer`(类似于C的 void指针)代理一下再转换也就是利用上述的1，2规则。
举例：

```go
func Float64bits(f float64) uint64 {
  // 无法直接转换，报错：Connot convert expression of type *float64 to type *uint64
  // return *(*uint64)(&f)   

 // 先把*float64 转成 Pointer(描述1)，再把Pointer转成*uint64(描述2)
  return *(*uint64)(unsafe.Pointer(&f)) 
}
```

## 2. unsafe的定义：

整体代码比较简单，**2个类型定义和3个uintptr的返回函数**

```go
package unsafe
//ArbitraryType仅用于文档目的，实际上并不是unsafe包的一部分,它表示任意Go表达式的类型。
type ArbitraryType int
//任意类型的指针，类似于C的*void
type Pointer *ArbitraryType
//确定结构在内存中占用的确切大小
func Sizeof(x ArbitraryType) uintptr
//返回结构体中某个field的偏移量
func Offsetof(x ArbitraryType) uintptr
//返回结构体中某个field的对其值（字节对齐的原因）
func Alignof(x ArbitraryType) uintptr
```
例子：

```go
package main

import (
    "fmt"
    "unsafe"
)

type Human struct {
    sex  bool
    age  uint8
    min  int
    name string
}

func main() {
    h := Human{
        true,
        30,
        1,
        "hello",
    }
    i := unsafe.Sizeof(h)
    j := unsafe.Alignof(h.age
)
    k := unsafe.Offsetof(h.name)
    fmt.Println(i, j, k)
    fmt.Printf("%p\n", &h)
    var p unsafe.Pointer
    p = unsafe.Pointer(&h)
    fmt.Println(p)
}
[root@localhost unsafe]#  go run unsafe.go
32 1 16
0xc00000c060
0xc00000c060
```

## 3. Pointer使用

前面已经说了，pointer是任意类型的指针，可以指向任意类型数据。参照Float64bits的转换和上述栗子的unsafe.Pointer(&h)，所以主要用于转换各种类型

## 4. uintptr 

golang中uintptr的定义是 `type uintptr uintptr uintptr`是golang的内置类型，是能存储指针的整型

根据描述3**，一个unsafe.Pointer指针也可以被转化为uintptr类型，然后保存到指针型数值变量中（注：这只是和当前指针相同的一个数字值，并不是一个指针），然后用以做必要的指针数值运算。（uintptr是一个无符号的整型数，足以保存一个地址）**
这种转换虽然也是可逆的，但是将**uintptr转为unsafe.Pointer指针可能会破坏类型系统，因为并不是所有的数字都是有效的内存地址。**
许多将unsafe.Pointer指针转为uintptr，然后再转回为unsafe.Pointer类型指针的操作也是不安全的。比如下面的例子需要将变量x的地址加上b字段地址偏移量转化为*int16类型指针，然后通过该指针更新x.b：

```go
package main

import (
    "fmt"
    "unsafe"
)

func main() {

    var x struct {
        a bool
        b int16
        c []int
    }

    /**
    unsafe.Offsetof 函数的参数必须是一个字段 x.f, 然后返回 f 字段相对于 x 起始地址的偏移量, 包括可能的空洞.
    */

    /**
    uintptr(unsafe.Pointer(&x)) + unsafe.Offsetof(x.b)
    指针的运算
    */
    // 和 pb := &x.b 等价
    pb := (*int16)(unsafe.Pointer(uintptr(unsafe.Pointer(&x)) + unsafe.Offsetof(x.b)))
    *pb = 42
    fmt.Println(x.b) // "42"
}
```

上面的写法尽管很繁琐，但在这里并不是一件坏事，因为这些功能应该很谨慎地使用。不要试图引入一个uintptr类型的临时变量，因为它可能会破坏代码的安全性（注：这是真正可以体会unsafe包为何不安全的例子）。

下面段代码是错误的：

```go
// NOTE: subtly incorrect!
tmp := uintptr(unsafe.Pointer(&x)) + unsafe.Offsetof(x.b)
pb := (*int16)(unsafe.Pointer(tmp))
*pb = 42
```

产生错误的原因很微妙。

有时候垃圾回收器会移动一些变量以降低内存碎片等问题。这类垃圾回收器被称为移动GC。当一个变量被移动，所有的保存改变量旧地址的指针必须同时被更新为变量移动后的新地址。从垃圾收集器的视角来看，一个unsafe.Pointer是一个指向变量的指针，因此当变量被移动是对应的指针也必须被更新；但是uintptr类型的临时变量只是一个普通的数字，所以其值不应该被改变。上面错误的代码因为引入一个非指针的临时变量tmp，导致垃圾收集器无法正确识别这个是一个指向变量x的指针。当第二个语句执行时，变量x可能已经被转移，这时候临时变量tmp也就不再是现在的&x.b地址。第三个向之前无效地址空间的赋值语句将彻底摧毁整个程序！

参考链接：
[https://studygolang.com/articles/18436](https://studygolang.com/articles/18436)

扩展链接：
[https://mp.weixin.qq.com/s?__biz=MzAxMTA4Njc0OQ==&mid=2651438794&idx=3&sn=3ab3a434e2ca96c19b52f320db587e03&chksm=80bb6038b7cce92e948c605148549e01a469d3c06f49120413ece380f91387dd7dac9004261b&mpshare=1&scene=1&srcid=&sharer_sharetime=1584035150927&sharer_shareid=9e1d0f93025303e47ff2523f5ebf4078&key=9bd4673607b34344798fdfd474e0dbb7b935c4b90ecd2d367ce77d35071df58750c1a110ec23a2e471a75ebb558386f4ef4e27bcaf5105430637714ce4e7a8dc362afa0d82a7a4ee23ffe5e337926936&ascene=1&uin=MjkwMDAzNTYzOQ%3D%3D&devicetype=Windows+10&version=62080079&lang=zh_CN&exportkey=AWaaaRIza9NoxoUmA5IIxOk%3D&pass_ticket=pOs7If%2FmacdTI4QyshJJLBE%2FIJNE%2BmjmCnL9CjZ%2Fw2s6Jz4C0KYT1mmUJPpfM6IV](https://mp.weixin.qq.com/s?__biz=MzAxMTA4Njc0OQ==&mid=2651438794&idx=3&sn=3ab3a434e2ca96c19b52f320db587e03&chksm=80bb6038b7cce92e948c605148549e01a469d3c06f49120413ece380f91387dd7dac9004261b&mpshare=1&scene=1&srcid=&sharer_sharetime=1584035150927&sharer_shareid=9e1d0f93025303e47ff2523f5ebf4078&key=9bd4673607b34344798fdfd474e0dbb7b935c4b90ecd2d367ce77d35071df58750c1a110ec23a2e471a75ebb558386f4ef4e27bcaf5105430637714ce4e7a8dc362afa0d82a7a4ee23ffe5e337926936&ascene=1&uin=MjkwMDAzNTYzOQ==&devicetype=Windows%2010&version=62080079&lang=zh_CN&exportkey=AWaaaRIza9NoxoUmA5IIxOk=&pass_ticket=pOs7If/macdTI4QyshJJLBE/IJNE%2bmjmCnL9CjZ/w2s6Jz4C0KYT1mmUJPpfM6IV)
