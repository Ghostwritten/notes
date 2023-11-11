

----
## 1. 温习静态与动态类型
### 1.1 静态类型
   静态类型就是变量声明时的赋予的类型。比如：

```go
type MyInt int // int 就是静态类型

type A struct{
    Name string  // string就是静态
}
var i *int  // *int就是静态类型
```
### 1.2. 动态类型
动态类型：运行时给这个变量复制时，这个值的类型(如果值为nil的时候没有动态类型)。一个变量的动态类型在运行时可能改变，这主要依赖于它的赋值（前提是这个变量时接口类型）。

  

```go
  var A interface{} // 静态类型interface{}
	A = 10            // 静态类型为interface{}  动态为int
	A = "String"      // 静态类型为interface{}  动态为string
	var M *int
	A = M             // 猜猜这个呢？
```
实例1
步骤：

 1. 定义一个接口
 2. 定义一个类型
 3. 通过类型实现这个这个接口
 4. 定义一个函数作用返回这个类型实例
 5. 把返回的类型实例放入定义的接口，对返回值做一个判断

```go
package main
import (
    "fmt"
    "os"
)
//定义一个接口
type Abc interface{
	String() string
}
// 类型
type Efg struct{
	data string
}
// 类型Efg实现Abc接口
func (e *Efg)String()string{
	return e.data
}
// 获取一个*Efg实例
func GetEfg() *Efg{
	return nil
}
// 比较
func CheckAE(a Abc) bool{
	return a == nil
}

func main() {
	efg := GetEfg()
	b := CheckAE(efg)
	fmt.Println(b)
	os.Exit(1)
}
```

```go
[root@localhost reflect]# go run ref1.go
false
exit status 1
```
## 2 反射简介
在计算机科学领域，反射是指一类应用，它们能够自描述和自控制。也就是说，这类应用通过采用某种机制来实现对自己行为的描述（self-representation）和监测（examination），并能根据自身行为的状态和结果，调整或修改应用所描述行为的状态和相关的语义。
每种语言的反射模型都不同，并且有些语言根本不支持反射。Golang语言实现了反射，反射机制就是在运行时动态的调用对象的方法和属性，官方自带的reflect包就是反射相关的，只要包含这个包就可以使用。
多插一句，Golang的gRPC也是通过反射实现的。

## 3 什么时候用反射？
有时候你想在运行时使用变量来处理变量，这些变量使用编写程序时不存在的信息。也许你正试图将来自文件或网络请求的数据映射到变量中。也许创建一个适用于不同类型的tool。在这些情况下，你需要使用反射。**反射使您能够在运行时检查类型。它还允许您在运行时检查，修改和创建变量，函数和结构。**

 反射的使用场景：**写测试用例的时候可以使用反射** 

其他场景尽量不要使用反射，原因如下： 
1. 业务代码中写反射，会增加复杂性，让人难以理解 
2. 反射性能比较低，比正常代码要慢一到两个数量级 
3. 反射并不能在编译时检查出错误，在运行时可能会出现panic 
所以在正常代码中尽量不要使用反射。


## 4. interface 和 反射关系
在讲反射之前，先来看看Golang关于类型设计的一些原则

 - 变量包括（type, value）两部分 理解这一点就知道为什么nil != nil了
 - type 包括 static type和concrete type. 简单来说 static
   type是你在编码是看见的类型(如int、string)，concrete type是runtime系统看见的类型
 - 类型断言能否成功，取决于变量的concrete type，而不是static type. 因此，一个
   reader变量如果它的concrete type也实现了write方法的话，它也可以被类型断言为writer.

  接下来要讲的反射，就是建立在类型之上的，**Golang的指定类型的变量的类型是静态的（也就是指定int、string这些的变量，它的type是static type）**，在创建变量的时候就已经确定，**反射主要与Golang的interface类型相关（它的type是concrete type），只有interface类型才有反射一说。**
  在Golang的实现中，每个interface变量都有一个对应pair，pair中记录了实际变量的值和类型:

```go
(value, type)
```
**value是实际变量值，type是实际变量的类型。一个interface{}类型的变量包含了2个指针，一个指针指向值的类型【对应concrete type】，另外一个指针指向实际的值【对应value】。**
例如，创建类型为*os.File的变量，然后将其赋给一个接口变量r：

```go
tty, err := os.OpenFile("/dev/tty", os.O_RDWR, 0)

var r io.Reader
r = tty
```

**接口变量r**的pair中将记录如下信息：(tty, *os.File)，这个pair在接口变量的连续赋值过程中是不变的，将接口变量r赋给另一个接口变量w:

```go
var w io.Writer
w = r.(io.Writer)
```
接口变量w的pair与r的pair相同，都是:(tty, *os.File)，即使w是空接口类型，pair也是不变的。
**interface及其pair的存在，是Golang中实现反射的前提，理解了pair，就更容易理解反射。反射就是用来检测存储在接口变量内部(值value；类型concrete type) pair对的一种机制。**


## 5 三个代表
package reflect里定义了很多函数。从整体上看需要注意的有三个:

```go
func ValueOf(i interface{}) Value
func TypeOf(i interface{}) Type
func (v Value) Interface() (i interface{})
```

## 6 两个基本点
同样在reflect里的各种类型中,最基本的有两个:`Typeof`和`Valueof`。
### 6.1 基础常用

```go
package main

import (
    "reflect"
    "fmt"
)

func main() {
    
    fmt.Printf("-----test1-----\n") 
    str := "wd"
    res_type := reflect.TypeOf(str)
    res_value := reflect.ValueOf(str)
    fmt.Println(res_type) //string
    fmt.Println(res_value) 

    fmt.Printf("-----test2-----\n") 
    var num float64 = 1.2345
    fmt.Println("type: ", reflect.TypeOf(num))
    fmt.Println("value: ", reflect.ValueOf(num))
    
    
    fmt.Printf("-----test3-----\n") 
    type myStruct struct {
    field1 int
    field2 string
    }
    var test myStruct
    testType := reflect.TypeOf(test)
    fmt.Println(testType.Name())
    for i := 0; i < testType.NumField(); i++ {
       fmt.Println(testType.Field(i).Name) 
    }
    
    fmt.Printf("-----test4-----\n") 
    type Addr struct {
    Province string
    City string
    Telephone string
    }
    addr := Addr{"HuNan","ShaoYang","15511111111"}
    reflectValue := reflect.ValueOf(addr)
    refectValueOfAddr, _ := reflectValue.Interface().(Addr)
    fmt.Printf("Province: %s\n", refectValueOfAddr.Province)
    fmt.Printf("City: %s\n", refectValueOfAddr.City)
    fmt.Printf("Telephone: %s\n", refectValueOfAddr.Telephone)

}
```

```go
[root@localhost reflect]# go run ref2.go
-----test1-----
string
wd
-----test2-----
type:  float64
value:  1.2345
-----test3-----
myStruct
field1
field2
-----test4-----
Province: HuNan
City: ShaoYang
Telephone: 15511111111
```
### 6.2 typeof

```python
// 通用方法

func (t *rtype) String() string // 获取 t 类型的字符串描述，不要通过 String 来判断两种类型是否一致。

func (t *rtype) Name() string // 获取 t 类型在其包中定义的名称，未命名类型则返回空字符串。

func (t *rtype) PkgPath() string // 获取 t 类型所在包的名称，未命名类型则返回空字符串。

func (t *rtype) Kind() reflect.Kind // 获取 t 类型的类别。

func (t *rtype) Size() uintptr // 获取 t 类型的值在分配内存时的大小，功能和 unsafe.SizeOf 一样。

func (t *rtype) Align() int  // 获取 t 类型的值在分配内存时的字节对齐值。

func (t *rtype) FieldAlign() int  // 获取 t 类型的值作为结构体字段时的字节对齐值。

func (t *rtype) NumMethod() int  // 获取 t 类型的方法数量。

func (t *rtype) Method() reflect.Method  // 根据索引获取 t 类型的方法，如果方法不存在，则 panic。
// 如果 t 是一个实际的类型，则返回值的 Type 和 Func 字段会列出接收者。
// 如果 t 只是一个接口，则返回值的 Type 不列出接收者，Func 为空值。

func (t *rtype) MethodByName(string) (reflect.Method, bool) // 根据名称获取 t 类型的方法。

func (t *rtype) Implements(u reflect.Type) bool // 判断 t 类型是否实现了 u 接口。

func (t *rtype) ConvertibleTo(u reflect.Type) bool // 判断 t 类型的值可否转换为 u 类型。

func (t *rtype) AssignableTo(u reflect.Type) bool // 判断 t 类型的值可否赋值给 u 类型。

func (t *rtype) Comparable() bool // 判断 t 类型的值可否进行比较操作


####注意对于：数组、切片、映射、通道、指针、接口 
func (t *rtype) Elem() reflect.Type // 获取元素类型、获取指针所指对象类型，获取接口的动态类型
```
示例：
```go
package main

import (
    "fmt"
    "reflect"
)

type Skills interface {
    reading()
    running()
}

type Student struct {
    Name string
    Age   int

}

func (self Student) runing(){
    fmt.Printf("%s is running\n",self.Name)
}
func (self Student) reading(){
    fmt.Printf("%s is reading\n" ,self.Name)
}
func main() {
    stu1 := Student{Name:"wd",Age:22}
    inf := new(Skills)
    stu_type := reflect.TypeOf(stu1)
    inf_type := reflect.TypeOf(inf).Elem()   // 特别说明，引用类型需要用Elem()获取指针所指的对象类型
    fmt.Println(stu_type.String())  //main.Student
    fmt.Println(stu_type.Name()) //Student
    fmt.Println(stu_type.PkgPath()) //main
    fmt.Println(stu_type.Kind()) //struct
    fmt.Println(stu_type.Size())  //24
    fmt.Println(inf_type.NumMethod())  //2
    fmt.Println(inf_type.Method(0),inf_type.Method(0).Name)  // {reading main func() <invalid Value> 0} reading
    fmt.Println(inf_type.MethodByName("reading")) //{reading main func() <invalid Value> 0} true

}
```
其他方法
```go
// 数值
func (t *rtype) Bits() int  // 获取数值类型的位宽，t 必须是整型、浮点型、复数型
------------------------------
// 数组
func (t *rtype) Len() int  // 获取数组的元素个数
------------------------------
// 映射
func (t *rtype) Key() reflect.Type // 获取映射的键类型
------------------------------
// 通道
func (t *rtype) ChanDir() reflect.ChanDir // 获取通道的方向
------------------------------
// 结构体
func (t *rtype) NumField() int  // 获取字段数量
func (t *rtype) Field(int) reflect.StructField  // 根据索引获取字段
func (t *rtype) FieldByName(string) (reflect.StructField, bool)  // 根据名称获取字段
func (t *rtype) FieldByNameFunc(match func(string) bool) (reflect.StructField, bool)  // 根据指定的匹配函数 math 获取字段
func (t *rtype) FieldByIndex(index []int) reflect.StructField  // 根据索引链获取嵌套字段
------------------------------
// 函数
func (t *rtype) NumIn() int // 获取函数的参数数量
func (t *rtype) In(int) reflect.Type // 根据索引获取函数的参数信息
func (t *rtype) NumOut() int // 获取函数的返回值数量
func (t *rtype) Out(int) reflect.Type // 根据索引获取函数的返回值信息
func (t *rtype) IsVariadic() bool  // 判断函数是否具有可变参数。
// 如果有可变参数，则 t.In(t.NumIn()-1) 将返回一个切片。
```
示例：
```go
package main

import (
    "fmt"
    "reflect"
)

type Skills interface {
    reading()
    running()
}

type Student struct {
    Name string
    Age   int

}

func (self Student) runing(){
    fmt.Printf("%s is running\n",self.Name)
}
func (self Student) reading(){
    fmt.Printf("%s is reading\n" ,self.Name)
}
func main() {
    stu1 := Student{Name:"wd",Age:22}
    stu_type := reflect.TypeOf(stu1)
    fmt.Println(stu_type.NumField())  //2
    fmt.Println(stu_type.Field(0))  //{Name  string  0 [0] false}
    fmt.Println(stu_type.FieldByName("Age"))  //{{Age  int  16 [1] false} true
}
```
### 6.3 valueof
常量类型

```go
const (
        Invalid Kind = iota
        Bool
        Int
        Int8
        Int16
        Int32
        Int64
        Uint
        Uint8
        Uint16
        Uint32
        Uint64
        Uintptr
        Float32
        Float64
        Complex64
        Complex128
        Array
        Chan
        Func
        Interface
        Map
        Ptr
        Slice
        String
        Struct
        UnsafePointer
)
```

```go
package main

import (
"reflect"
    "fmt"
)

func main() {
    str := "wd"
    val := reflect.ValueOf(str).Kind()
    fmt.Println(val)//string
}
```
用于获取值方法：

```go
func (v Value) Int() int64 // 获取int类型值，如果 v 值不是有符号整型，则 panic。

func (v Value) Uint() uint64 // 获取unit类型的值，如果 v 值不是无符号整型（包括 uintptr），则 panic。

func (v Value) Float() float64 // 获取float类型的值，如果 v 值不是浮点型，则 panic。

func (v Value) Complex() complex128 // 获取复数类型的值，如果 v 值不是复数型，则 panic。

func (v Value) Bool() bool // 获取布尔类型的值，如果 v 值不是布尔型，则 panic。

func (v Value) Len() int // 获取 v 值的长度，v 值必须是字符串、数组、切片、映射、通道。

func (v Value) Cap() int  // 获取 v 值的容量，v 值必须是数值、切片、通道。

func (v Value) Index(i int) reflect.Value // 获取 v 值的第 i 个元素，v 值必须是字符串、数组、切片，i 不能超出范围。

func (v Value) Bytes() []byte // 获取字节类型的值，如果 v 值不是字节切片，则 panic。

func (v Value) Slice(i, j int) reflect.Value // 获取 v 值的切片，切片长度 = j - i，切片容量 = v.Cap() - i。
// v 必须是字符串、数值、切片，如果是数组则必须可寻址。i 不能超出范围。

func (v Value) Slice3(i, j, k int) reflect.Value  // 获取 v 值的切片，切片长度 = j - i，切片容量 = k - i。
// i、j、k 不能超出 v 的容量。i <= j <= k。
// v 必须是字符串、数值、切片，如果是数组则必须可寻址。i 不能超出范围。

func (v Value) MapIndex(key Value) reflect.Value // 根据 key 键获取 v 值的内容，v 值必须是映射。
// 如果指定的元素不存在，或 v 值是未初始化的映射，则返回零值（reflect.ValueOf(nil)）

func (v Value) MapKeys() []reflect.Value // 获取 v 值的所有键的无序列表，v 值必须是映射。
// 如果 v 值是未初始化的映射，则返回空列表。

func (v Value) OverflowInt(x int64) bool // 判断 x 是否超出 v 值的取值范围，v 值必须是有符号整型。

func (v Value) OverflowUint(x uint64) bool  // 判断 x 是否超出 v 值的取值范围，v 值必须是无符号整型。

func (v Value) OverflowFloat(x float64) bool  // 判断 x 是否超出 v 值的取值范围，v 值必须是浮点型。

func (v Value) OverflowComplex(x complex128) bool // 判断 x 是否超出 v 值的取值范围，v 值必须是复数型。
```
设置值方法：

```go
func (v Value) SetInt(x int64)  //设置int类型的值

func (v Value) SetUint(x uint64)  // 设置无符号整型的值

func (v Value) SetFloat(x float64) // 设置浮点类型的值

func (v Value) SetComplex(x complex128) //设置复数类型的值

func (v Value) SetBool(x bool) //设置布尔类型的值

func (v Value) SetString(x string) //设置字符串类型的值

func (v Value) SetLen(n int)  // 设置切片的长度，n 不能超出范围，不能为负数。

func (v Value) SetCap(n int) //设置切片的容量

func (v Value) SetBytes(x []byte) //设置字节类型的值

func (v Value) SetMapIndex(key, val reflect.Value) //设置map的key和value，前提必须是初始化以后，存在覆盖、不存在添加
```
其他方法：

```go
##########结构体相关：
func (v Value) NumField() int // 获取结构体字段（成员）数量

func (v Value) Field(i int) reflect.Value  //根据索引获取结构体字段

func (v Value) FieldByIndex(index []int) reflect.Value // 根据索引链获取结构体嵌套字段

func (v Value) FieldByName(string) reflect.Value // 根据名称获取结构体的字段，不存在返回reflect.ValueOf(nil)

func (v Value) FieldByNameFunc(match func(string) bool) Value // 根据匹配函数 match 获取字段,如果没有匹配的字段，则返回零值（reflect.ValueOf(nil)）


########通道相关：
func (v Value) Send(x reflect.Value)// 发送数据（会阻塞），v 值必须是可写通道。

func (v Value) Recv() (x reflect.Value, ok bool) // 接收数据（会阻塞），v 值必须是可读通道。

func (v Value) TrySend(x reflect.Value) bool // 尝试发送数据（不会阻塞），v 值必须是可写通道。

func (v Value) TryRecv() (x reflect.Value, ok bool) // 尝试接收数据（不会阻塞），v 值必须是可读通道。

func (v Value) Close() // 关闭通道


########函数相关
func (v Value) Call(in []Value) (r []Value) // 通过参数列表 in 调用 v 值所代表的函数（或方法）。函数的返回值存入 r 中返回。
// 要传入多少参数就在 in 中存入多少元素。
// Call 即可以调用定参函数（参数数量固定），也可以调用变参函数（参数数量可变）。

func (v Value) CallSlice(in []Value) []Value // 调用变参函数
```
示例一：获取和设置普通类型的值

```go
package main

import (
    "reflect"
    "fmt"
)

func main() {
    str := "wd"
    age := 11
    fmt.Println(reflect.ValueOf(str).String()) //获取str的值，结果wd
    fmt.Println(reflect.ValueOf(age).Int())   //获取age的值，结果age
    str2 := reflect.ValueOf(&str)        //获取Value类型
    str2.Elem().SetString("jack")     //设置值
    fmt.Println(str2.Elem(),age) //jack 11
}
```
示例二：简单结构体操作

```go
package main

import (
    "fmt"
    "reflect"
)

type Skills interface {
    reading()
    running()
}

type Student struct {
    Name string
    Age   int

}

func (self Student) runing(){
    fmt.Printf("%s is running\n",self.Name)
}
func (self Student) reading(){
    fmt.Printf("%s is reading\n" ,self.Name)
}
func main() {
    stu1 := Student{Name:"wd",Age:22}
    stu_val := reflect.ValueOf(stu1) //获取Value类型
    fmt.Println(stu_val.NumField()) //2
    fmt.Println(stu_val.Field(0),stu_val.Field(1)) //wd 22
    fmt.Println(stu_val.FieldByName("Age")) //22
    stu_val2 := reflect.ValueOf(&stu1).Elem()   
    stu_val2.FieldByName("Age").SetInt(33)  //设置字段值 ，结果33
    fmt.Println(stu1.Age)
    
}
```
示例三：通过反射调用结构体中的方法，通过reflect.Value.Method(i int).Call()或者reflect.Value.MethodByName(name string).Call()实现

```go
package main

import (
    "fmt"
    "reflect"
)

type Student struct {
    Name string
    Age int
}

func (this *Student) SetName(name string) {
    this.Name = name
    fmt.Printf("set name %s\n",this.Name )
}

func (this *Student) SetAge(age int) {
    this.Age = age
    fmt.Printf("set age %d\n",age )
}

func (this *Student) String() string {
    fmt.Printf("this is %s\n",this.Name)
    return this.Name
}

func main() {
    stu1 := &Student{Name:"wd",Age:22}
    val := reflect.ValueOf(stu1)       //获取Value类型，也可以使用reflect.ValueOf(&stu1).Elem() 
    val.MethodByName("String").Call(nil)  //调用String方法

    params := make([]reflect.Value, 1)
    params[0] = reflect.ValueOf(18)
    val.MethodByName("SetAge").Call(params)  //通过名称调用方法

    params[0] = reflect.ValueOf("jack")   
    val.Method(1).Call(params)    //通过方法索引调用

    fmt.Println(stu1.Name,stu1.Age)
}
//this is wd
//set age 18
//set name jack
//jack 18
```
## 7. 细化 typeof与ValueOf输出
### 7.1 typeof示例1

```bash
package main


import (
  "reflect"
  "fmt"
)

func reflectType(x interface{}) {
    v := reflect.TypeOf(x)
    fmt.Printf("你传入的变量类型是:%v\n",v)
}


func main() {
    var a int = 666
    var b float64 = 3.14
    var c string = "hello world"
    var d [3]int = [3]int{1, 2, 6}
    var e []int = []int{1,2,6,88}
    var f map[string]interface{} = map[string]interface{}{
        "Name":"张三",
        "Age":18,
}
    reflectType(a)
    reflectType(b)
    reflectType(c)
    reflectType(d)
    reflectType(e)
    reflectType(f)
}
```
输出
```bash
[root@localhost reflect]# go run test.go
你传入的变量类型是:int
你传入的变量类型是:float64
你传入的变量类型是:string
你传入的变量类型是:[3]int
你传入的变量类型是:[]int
你传入的变量类型是:map[string]interface {}
```
### 7.2 typeof示例2

```bash
package main


import (
  "reflect"
  "fmt"
)


func reflectType(x interface{}) {
v := reflect.TypeOf(x)
fmt.Printf("你传入的变量类型是:%v | Name:%v | Kind:%v\n", v, v.Name(), v.Kind())
}


type Student struct {
    Name string
    Age int
}

func main() {
    var a int
    var b *int
    var c []int
    var d map[string]interface{}
    var e Student
    reflectType(a)
    reflectType(b)
    reflectType(c)
    reflectType(d)
    reflectType(e)
}
```
输出

```bash
[root@localhost reflect]# go run test2.go
你传入的变量类型是:int | Name:int | Kind:int
你传入的变量类型是:*int | Name: | Kind:ptr
你传入的变量类型是:[]int | Name: | Kind:slice
你传入的变量类型是:map[string]interface {} | Name: | Kind:map
你传入的变量类型是:main.Student | Name:Student | Kind:struct
```

### 7.3 ValueOf示例1

```bash
package main


import (
  "reflect"
  "fmt"
)


type Student struct {
    Name string
    Age int
}

func reflectType(x interface{}) {
    v := reflect.ValueOf(x)
    k := v.Kind()
    switch k {
    case reflect.Int:
        fmt.Printf("我是Int类型,我的值是%v\n", v.Int())
    case reflect.Slice:
        fmt.Printf("我是切片类型,我的值是%v\n",v.Slice(1,2))
    case reflect.Map:
        fmt.Printf("我是切片类型,我的值是%v\n",v.MapKeys())
    //case :可以继续case下去
  }
}



func main() {
  var a int = 1
  var c []int = []int{1, 5,7, 19}
  var d map[string]interface{} = map[string]interface{}{
    "Name": "你好",
    "Age":  18,
  }
  var e Student
  reflectType(a)

  reflectType(c)
  reflectType(d)
  reflectType(e)
}
```

输出：

```bash
[root@localhost reflect]# go run test3.go 
我是Int类型,我的值是1
我是切片类型,我的值是[5]
我是切片类型,我的值是[Name Age]
```

## 8. 结构体反射
 typeof示例1
```bash
package main


import (
  "reflect"
  "fmt"
)

type Student struct {
    Name   string   `json:"name" describe:"姓名"`
    Age    int      `json:"age" describe:"年龄"`
    Gender bool     `json:"gender" describe:"性别"`
    Hobby  []string `json:"hobby" describe:"爱好"`
}

func main() {
    //实例化结构体
    var s1 = Student{
        Name:   "张三",
        Age:    18,
        Gender: true,
        Hobby:  []string{"吃", "喝", "pia", "玩"},
}
    var t = reflect.TypeOf(s1)
    fmt.Println(t.Name())     //Student
    fmt.Println(t.Kind())     //struct
    fmt.Println(t.NumField()) //结果:4,表示多少个字段
    for i := 0; i < t.NumField(); i++ {
        field := t.Field(i)//每个结构体对象
        /*
            {Name  string json:"name" describe:"姓名" 0 [0] false}
            {Age  int json:"age" describe:"年龄" 16 [1] false}
            {Gender  bool json:"gender" describe:"性别" 24 [2] false}
            {Hobby  []string json:"hobby" describe:"爱好" 32 [3] false}
        */
        //fmt.Println(field)
        fmt.Println("------")
        fmt.Printf("field.Name:%v\n",field.Name)
        fmt.Printf("field.Index:%v\n",field.Index)
        fmt.Printf("field.Type:%v\n",field.Type)
        fmt.Printf("field.Tag:%v\n",field.Tag.Get("describe"))

    }
}
```
输出

```bash
[root@localhost reflect]# go run test4.go
Student
struct
4
------
field.Name:Name
field.Index:[0]
field.Type:string
field.Tag:姓名
------
field.Name:Age
field.Index:[1]
field.Type:int
field.Tag:年龄
------
field.Name:Gender
field.Index:[2]
field.Type:bool
field.Tag:性别
------
field.Name:Hobby
field.Index:[3]
field.Type:[]string
field.Tag:爱好
```

### 8.1 单独反射指定字段信息
typeof示例1
```bash
package main

import (
  "fmt"
  "reflect"
)



type Student struct {
    Name   string   `json:"name" describe:"姓名"`
    Age    int      `json:"age" describe:"年龄"`
    Gender bool     `json:"gender" describe:"性别"`
    Hobby  []string `json:"hobby" describe:"爱好"`
}
func main() {
    //实例化结构体
    var s1 = Student{
        Name:   "张三",
        Age:    18,
        Gender: true,
        Hobby:  []string{"吃", "喝", "pia", "玩"},
}
    var t = reflect.TypeOf(s1)
    genderField, ok := t.FieldByName("Gender")
    if ok {
        fmt.Println(genderField.Name)                //Gender
        fmt.Println(genderField.Index)               //[2]
        fmt.Println(genderField.Type)                //bool
        fmt.Println(genderField.Tag.Get("describe")) //性别
    }
}
```
输出

```bash
[root@localhost reflect]# go run test5.go
Gender
[2]
bool
性别
```
valueof示例1

```bash
package main

import (
  "fmt"
  "reflect"
)



type Student struct {
    Name   string   `json:"name" describe:"姓名"`
    Age    int      `json:"age" describe:"年龄"`
    Gender bool     `json:"gender" describe:"性别"`
    Hobby  []string `json:"hobby" describe:"爱好"`
}

func main() {
    //实例化结构体
    var s1 = Student{
        Name:   "张三",
        Age:    18,
        Gender: true,
        Hobby:  []string{"吃", "喝", "pia", "玩"},
}
    var v = reflect.ValueOf(s1)
    for i := 0; i < v.NumField(); i++ {
        field :=v.Field(i)
        fmt.Println("------")
        fmt.Printf("Kind:%v\n",field.Kind())
        fmt.Printf("值:%v\n",field.Interface())
    }
}
```
输出

```bash
[root@localhost reflect]# go run test6.go
------
Kind:string
值:张三
------
Kind:int
值:18
------
Kind:bool
值:true
------
Kind:slice
值:[吃 喝 pia 玩]
```


参考连接：
[https://www.cnblogs.com/wdliu/p/9222283.html](https://www.cnblogs.com/wdliu/p/9222283.html)
[https://my.oschina.net/90design/blog/1614820](https://my.oschina.net/90design/blog/1614820)
[https://www.jianshu.com/p/8215e3bc1402](https://www.jianshu.com/p/8215e3bc1402)










