#  Go 指针概念【1】
![](https://i-blog.csdnimg.cn/blog_migrate/ab04f194094e0f5448bfcc61c794ec62.png)




----
## 1. 基本类型指针的理解
先看这两行代码。

```bash
var n1 int = 666
fmt.Println(n1)//结果:666
fmt.Printf("%p\n"，n1)//结果:%!p(int=666)，说明不是一个地址，就是一个值
```

内存分布图如下。
![](https://i-blog.csdnimg.cn/blog_migrate/577f3f4dd98a4790005db36fa124f4a6.png)
再看看这样两行代码，这里用到了&。

```bash
var n1 int = 1
//表示取n1的地址
fmt.Println(&n1)//结果:0xc00000a0b8
fmt.Printf("%p\n"，&n1)//结果:0xc00000a0b8
```

如图所示。
![](https://i-blog.csdnimg.cn/blog_migrate/96b048c9cdc001d0920e2d22bdfcaf91.png)
## 2. 引用类型指针的理解
先看这样的代码。

```bash
var studentList = []string{"张三"， "李四"}//一个切片
fmt.Println(studentList)        //结果:[张三 李四]
fmt.Printf("%p\n"， studentList) //结果:0xc0000044a0
//去地址
fmt.Printf("%p\n"， &studentList) //结果:0xc0000044a0
```

内存分布图如下。
![](https://i-blog.csdnimg.cn/blog_migrate/2404f3d6a4f47e5d6a780ac7da814cb0.png)
## 3. 值类型和引用类型
值类型
在Go中，值类型主要有。

int，float，bool，string，数组，struct(结构体)

内存分布大致如下。
![](https://i-blog.csdnimg.cn/blog_migrate/f12573c3a0925b638d598279d0c657c1.png)
注:像字符串，数组，结构体这些属于连续存储，变量指向的是它们的第一个地址，剩下的会根据长度计算。

## 4. 引用类型
在Go中，引用类型主要有。

切片(slice)，map，管道(chan)

内存分布大致如下。
![](https://i-blog.csdnimg.cn/blog_migrate/c732af2ecc553dc6d71f7f47aabc1250.png)
## 5. 栈内存和堆内存的区别
栈内存在存储上，只能存一些简单的东西，比如数字了，字符了，浮点数了之类的，但是栈内存分配的内存程序员不用回收，由系统自己回收，并且性能很高。

堆内存在存储上就比较丰富了，随便存，像map，随便塞，但是堆内存分配的内存需要程序员自己回收，典型例子，C++，如果语言由GC由GC回收，性能稍弱那么一点点点....，但是人家能随便存啊，多随便。
## 6. &和*的意思

 1. &叫做取地址符。
 2. *叫做收地址符。

示例

var c *int//*int是一个整体，说明c这个变量只能接收int类型的
*int是一个整体，表示c这个变量只能接收int类型的地址。

代码

```bash
package main

import "fmt"

func main() {
    var c *int
    var d int = 1
    //c = d//错误需要的是d的地址
    c = &d
    fmt.Println(c)
}
```

执行结果。
![](https://i-blog.csdnimg.cn/blog_migrate/db1857a87e0ae8e6981224237520f1ed.png)
可以看到打印的也是一个地址，但是内存图还是基本类型图。
![](https://i-blog.csdnimg.cn/blog_migrate/8aca5eb3c7e842dd9954f039dc32f744.png)
如果要打印c的值，直接*c就好了，取得就是地址里面对应得值了。

```bash
fmt.Println(*c)
```
## 7. 关于函数
我们一直在强调，操作只会操作栈上面的值，函数同理。

```bash
package main

import "fmt"

func say1(x int) {
    //x int 相当于隐藏了一行代码
    //隐藏的代码时 var x int = x，一定要记住这个
    fmt.Printf("say1:%p\n"， x)
}
func say2(x *int) {
    //隐藏的代码是 var x *int = x，x是一个地址
    fmt.Printf("say2:%p\n"， x)
}
func say3(x []int) {
    //隐藏的代码是 var x []int = x，因为x是引用类型，所以x是一个地址
    fmt.Printf("say3:%p\n"， x)
}
func main() {
    say1(1)//栈上面是1，所以传进去就是1
    var x1 = 1
    say2(&x1)//say只能接收整数地址
    var x2 = []int{1， 1}
    say3(x2)//x2是引用类型，所以传进去的时候就是地址，栈上面的就是地址
}
```

执行结果。
![](https://i-blog.csdnimg.cn/blog_migrate/935aabda46db935f2dcac55ab1efd697.png)

参考：
- [https://mp.weixin.qq.com/s/M4VHGdgGhuU4OYBrkZb2aw](https://mp.weixin.qq.com/s/M4VHGdgGhuU4OYBrkZb2aw)
