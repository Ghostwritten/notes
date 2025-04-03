#  Go struc 结构体

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cb4863c12831c4939b3c115127c90b28.png)



## 1. struct类型
可将类型分为命名和未命名两大类：

 - 命名类型包括 bool、int、string 等
 - array、slice、map 等和具体元素类型、长度等有关，属于未命名类型。

具有相同声明的未命名类型被视为同一类型

 1. 具有相同基类型的指针。
 2. 具有相同元素类型和长度的 array。
 3. 具有相同元素类型的 slice。
 4. 具有相同键值类型的 map。
 5. 具有相同元素类型和传送方向的 channel。
 6. 具有相同字段序列 (字段名、类型、标签、顺序) 的匿名 struct。 签名相同 (参数和返回值，不包括参数名称) 的function。
 7. 方法集相同 ( 方法名、方法签名相同，和次序无关) 的 interface。

## 2. struct 特点

 1. 用来自定义复杂数据结构
 2. struct里面可以包含多个字段（属性）
 3. struct类型可以定义方法，注意和函数的区分
 4. struct类型是值类型
 5. struct类型可以嵌套
 6. Go语言没有class类型，只有struct类型
 7. 结构体是用户单独定义的类型，不能和其他类型进行强制转换
 8. golang中的struct没有构造函数，一般可以使用工厂模式来解决这个问题。
 9. 我们可以为struct中的每个字段，写上一个tag。这个tag可以通过反射的机制获取到，最常用的场景就是json序列化和反序列化。
 
### 2.1 特性
 **值传递**
结构体与数组一样，是复合类型，无论是作为实参传递给函数时，还是赋值给其他变量，都是值传递，即复一个副本。
**没有继承**
Go语言是支持面向对象编程的，但却没有继承的概念，在结构体中，可以通过组合其他结构体来构建更复杂的结构体。
**结构体不能包含自己**
一个结构体，并没有包含自身，比如Member中的字段不能是Member类型，但却可能是*Member。
  
引用指针为什么要用new或make？
[Go语言中new和make的区别](https://www.flysnow.org/2017/10/23/go-new-vs-make.html)

## 3. 结构体的声明

```bash
type Employee struct {
    firstName string
    lastName  string
    age       int
}
```
在上面的代码片段里，声明了一个结构体类型 Employee，它有 firstName、lastName 和 age 三个字段。通过把相同类型的字段声明在同一行，结构体可以变得更加紧凑。在上面的结构体中，firstName 和 lastName 属于相同的 string 类型，于是这个结构体可以重写为：

```bash
type Employee struct {
    firstName, lastName string
    age, salary         int
}
```
上面的结构体 Employee 称为 **命名的结构体**（Named Structure）。我们创建了名为 Employee 的新类型，而它可以用于创建 Employee 类型的结构体变量。

声明结构体时也可以不用声明一个新类型，这样的结构体类型称为 **匿名结构体**（Anonymous Structure）。

```bash
var employee struct {
    firstName, lastName string
    age int
}
```

## 4. 创建命名的结构体

方法有几种：

```bash
package main
import (
    "fmt"
)
type Test struct {
    int
    string
}
var a Test
func main() {
    b := new(Test) //同 var b *Test = new(Test)
    c := Test{1, "c"}
    d := Test{}
    e := &Test{}
    f := &Test{2, "f"} //同 var d *Test = &Test{2, "f"}
    fmt.Println(a, b, c, d, e, f)
    // 注: a b c d 返回 Test 类型变量；e f 返回 *Test 类型变量；若无初始化值，则默认为零值
}
```

```bash
[root@localhost struct]# ls
stru1.go  stru2.go  stru3.go
[root@localhost struct]# go run stru3.go
{0 } &{0 } {1 c} {0 } &{0 } &{2 f}
```

初始化值可以分为两种:

 - a. 有序: typeName{value1, value2, …} 必须一一对应
 - b. 无序: typeName{field1:value1, field2:value2, …} 可初始化部分值

```bash
package main

import (
    "fmt"
)

type Employee struct {
    firstName, lastName string
    age, salary         int
}

func main() {

    //creating structure using field names
    emp1 := Employee{
        firstName: "Sam",
        age:       25,
        salary:    500,
        lastName:  "Anderson",
    }

    //creating structure without using field names
    emp2 := Employee{"Thomas", "Paul", 29, 800}

    fmt.Println("Employee 1", emp1)
    fmt.Println("Employee 2", emp2)

```

```bash
该程序将输出：

Employee 1 {Sam Anderson 25 500}
Employee 2 {Thomas Paul 29 800}
```

## 5. 创建匿名结构体

```bash
package main

import (
    "fmt"
)

func main() {
    emp3 := struct {
        firstName, lastName string
        age, salary         int
    }{
        firstName: "Andreah",
        lastName:  "Nikola",
        age:       31,
        salary:    5000,
    }

    fmt.Println("Employee 3", emp3)
}
```
该程序会输出：

```bash
Employee 3 {Andreah Nikola 31 5000}
```
## 6. 结构体的零值（Zero Value）

```bash
package main

import (
    "fmt"
)

type Employee struct {
    firstName, lastName string
    age, salary         int
}

func main() {
    var emp4 Employee //zero valued structure
    fmt.Println("Employee 4", emp4)
}
```
该程序定义了 emp4，却没有初始化任何值。因此 firstName 和 lastName 赋值为 string 的零值（""）。而 age 和 salary 赋值为 int 的零值（0）。该程序会输出：

```bash
Employee 4 { 0 0}
```
当然还可以为某些字段指定初始值，而忽略其他字段。这样，忽略的字段名会赋值为零值。

```bash
package main

import (
    "fmt"
)

type Employee struct {
    firstName, lastName string
    age, salary         int
}

func main() {
    emp5 := Employee{
        firstName: "John",
        lastName:  "Paul",
    }
    fmt.Println("Employee 5", emp5)
}
```
在上面程序中的第 14 行和第 15 行，我们初始化了 firstName 和 lastName，而 age 和 salary 没有进行初始化。因此 age 和 salary 赋值为零值。该程序会输出：

```bash
Employee 5 {John Paul 0 0}
```

## 7. 访问结构体的字段
点号操作符 . 用于访问结构体的字段。

```bash
package main

import (
    "fmt"
)

type Employee struct {
    firstName, lastName string
    age, salary         int
}

func main() {
    emp6 := Employee{"Sam", "Anderson", 55, 6000}
    fmt.Println("First Name:", emp6.firstName)
    fmt.Println("Last Name:", emp6.lastName)
    fmt.Println("Age:", emp6.age)
    fmt.Printf("Salary: $%d", emp6.salary)
}
```
在线运行程序[5]

上面程序中的 emp6.firstName 访问了结构体 emp6 的字段 firstName。该程序输出：

```bash
First Name: Sam
Last Name: Anderson
Age: 55
Salary: $6000
```
还可以创建零值的 struct，以后再给各个字段赋值。

```bash
package main

import (
    "fmt"
)

type Employee struct {
    firstName, lastName string
    age, salary         int
}

func main() {
    var emp7 Employee
    emp7.firstName = "Jack"
    emp7.lastName = "Adams"
    fmt.Println("Employee 7:", emp7)
}
```
在线运行程序[6]

在上面程序中，我们定义了 emp7，接着给 firstName 和 lastName 赋值。该程序会输出：

```bash
Employee 7: {Jack Adams 0 0}
```
## 8. 结构体的指针
还可以创建指向结构体的指针。

```bash
package main

import (
    "fmt"
)

type Employee struct {
    firstName, lastName string
    age, salary         int
}

func main() {
    emp8 := &Employee{"Sam", "Anderson", 55, 6000}
    fmt.Println("First Name:", (*emp8).firstName)
    fmt.Println("Age:", (*emp8).age)
}
```

在线运行程序[7]

在上面程序中，emp8 是一个指向结构体 Employee 的指针。(*emp8).firstName 表示访问结构体 emp8 的 firstName 字段。该程序会输出：

```bash
First Name: Sam
Age: 55
```
Go 语言允许我们在访问 firstName 字段时，可以使用 emp8.firstName 来代替显式的解引用 (*emp8).firstName。

```bash
package main

import (
    "fmt"
)

type Employee struct {
    firstName, lastName string
    age, salary         int
}

func main() {
    emp8 := &Employee{"Sam", "Anderson", 55, 6000}
    fmt.Println("First Name:", emp8.firstName)
    fmt.Println("Age:", emp8.age)
}
```

在线运行程序[8]

在上面的程序中，我们使用 emp8.firstName 来访问 firstName 字段，该程序会输出：

First Name: Sam
Age: 55

## 9. 匿名字段
**声明一个 struct1 可以包含已经存在的 struct2 或者go语言中内置类型作为内置字段，称为匿名字段**，即只写了 typeName，无 varName，但是 `typeName` 不能重复。
匿名字段与面向对象程序语言中的继承声明及初始化:

```bash
type Person struct {
    string
    int
}
```
我们接下来使用匿名字段来编写一个程序。

```bash
package main

import (
    "fmt"
)

type Person struct {
    string
    int
}

func main() {
    p := Person{"Naveen", 50}
    fmt.Println(p)
}
```
在线运行程序[9]

在上面的程序中，结构体 Person 有两个匿名字段。p := Person{"Naveen", 50} 定义了一个 Person 类型的变量。该程序输出 `{Naveen 50}`。

虽然匿名字段没有名称，但其实匿名字段的名称就默认为它的类型。比如在上面的 Person 结构体里，虽说字段是匿名的，但 Go 默认这些字段名是它们各自的类型。所以 Person 结构体有两个名为 string 和 int 的字段。

```bash
package main

import (
    "fmt"
)

type Person struct {
    string
    int
}

func main() {
    var p1 Person
    p1.string = "naveen"
    p1.int = 50
    fmt.Println(p1)
}
```

在线运行程序[10]

在上面程序的第 14 行和第 15 行，我们访问了 Person 结构体的匿名字段，我们把字段类型作为字段名，分别为 "string" 和 "int"。上面程序的输出如下：

```bash
{naveen 50}
```

## 10. 嵌套结构体（Nested Structs）
结构体的字段有可能也是一个结构体。这样的结构体称为嵌套结构体。
#### 第一示例
```bash
package main

import (
    "fmt"
)

type Address struct {
    city, state string
}
type Person struct {
    name string
    age int
    address Address
}

func main() {
    var p Person
    p.name = "Naveen"
    p.age = 50
    p.address = Address {
        city: "Chicago",
        state: "Illinois",
    }
    fmt.Println("Name:", p.name)
    fmt.Println("Age:",p.age)
    fmt.Println("City:",p.address.city)
    fmt.Println("State:",p.address.state)
}
```
在线运行程序[11]

上面的结构体 Person 有一个字段 address，而 address 也是结构体。该程序输出：
#### 第二示例
```bash
Name: Naveen
Age: 50
City: Chicago
State: Illinois
```

```bash
package main
import (
    "fmt"
)
type Person struct {
    name string
    age  int
    addr string
}
type Employee struct {
    Person //匿名字段
    salary int
    int           //用内置类型作为匿名字段
    addr   string //类似于重载
}
func main() {
    em1 := Employee{Person{"rain", 23, "qingyangqu"}, 5000, 100, "gaoxingqu"}
    fmt.Println(em1)
    // var em2 Person = em1
    // Error: cannot use em1 (type Employee) as type Person in assignment （没有继承， 然也不会有多态）
    var em2 Person = em1.Person // 同类型拷贝。
    fmt.Println(em2)
}
```

```bash
[root@localhost struct]# go run stru6.go 
{{rain 23 qingyangqu} 5000 100 gaoxingqu}
{rain 23 qingyangqu}
```

#### 第三示例
```bash
package main
import "fmt"
type Person struct {
    name string
    age  int
    addr string
}
type Employee struct {
    Person //匿名字段
    salary int
    int           //用内置类型作为匿名字段
    addr   string //类似于重载
}
func main() {
    /*
       var em1 Employee = Employee{}
       em1.Person = Person{"rain", 23, "帝都"}
       em1.salary = 5000
       em1.int = 100 //使用时注意其意义，此处无
       em1.addr = "北京"
    */
    //em1 := Employee{Person{"rain", 23, "帝都"}, 5000, 100, "北京"}
    //初始化方式不一样，但是结果一样
    em1 := Employee{Person: Person{"Murphy", 23, "帝都"}, salary: 5000, int: 100, addr: "北京"}
    fmt.Println(em1)
    fmt.Println("live addr(em1.addr) = ", em1.addr)
    fmt.Println("work addr(em1.Person.addr) = ", em1.Person.addr)
    em1.int = 200 //修改匿名字段的值
}
```

```bash
[root@localhost struct]# go run stru7.go
\{{Murphy 23 帝都} 5000 100 北京}
live addr(em1.addr) =  北京
work addr(em1.Person.addr) =  帝都
```
空结构 “节省” 内存， 如用来实现 set 数据结构，或者实现没有 “状态” 只有方法的 “静态类”。

```bash
package main
func main() {
    var null struct{}
    set := make(map[string]struct{})
    set["a"] = null
}
```
不能同时嵌入某一类型和其指针类型，因为它们名字相同。

```bash
package main
type Resource struct {
    id int
}
type User struct {
    *Resource
    // Resource // Error: duplicate field Resource
    name string
}
func main() {
    u := User{
        &Resource{1},
        "Administrator",
    }
    println(u.id)
    println(u.Resource.id)
}
```

```bash
[root@localhost struct]# go run stru8.go
1
1
```


## 11. 提升字段（Promoted Fields）

```bash
type Address struct {
    city, state string
}
type Person struct {
    name string
    age  int
    Address
}
```
上面的代码片段中，`Person` 结构体有一个匿名字段 `Address`，而 `Address` 是一个结构体。现在结构体 `Address` 有 `city` 和 `state` 两个字段，访问这两个字段就像在 `Person` 里直接声明的一样，因此我们称之为`提升字段`。

```bash
package main

import (
    "fmt"
)

type Address struct {
    city, state string
}
type Person struct {
    name string
    age  int
    Address
}

func main() {
    var p Person
    p.name = "Naveen"
    p.age = 50
    p.Address = Address{
        city:  "Chicago",
        state: "Illinois",
    }
    fmt.Println("Name:", p.name)
    fmt.Println("Age:", p.age)
    fmt.Println("City:", p.city) //city is promoted field
    fmt.Println("State:", p.state) //state is promoted field
}
```
上面代码中的第 26 行和第 27 行，我们使用了语法 p.city 和 p.state，访问提升字段 city 和 state 就像它们是在结构体 p 中声明的一样。该程序会输出：

```bash
Name: Naveen
Age: 50
City: Chicago
State: Illinois
```
## 12. 导出结构体和字段
如果结构体名称以大写字母开头，则它是其他包可以访问的导出类型（Exported Type）。同样，如果结构体里的字段首字母大写，它也能被其他包访问到。

让我们使用自定义包，编写一个程序来更好地去理解它。

在你的 Go 工作区的 src 目录中，创建一个名为 `structs` 的文件夹。另外在 `structs` 中再创建一个目录 `computer`。

在 computer 目录中，在名为 `spec.go` 的文件中保存下面的程序。

```bash
package computer

type Spec struct { //exported struct
    Maker string //exported field
    model string //unexported field
    Price int //exported field
}
```
上面的代码片段中，创建了一个 `computer` 包，里面有一个导出结构体类型 `Spec`。`Spec` 有两个导出字段 `Maker` 和 `Price`，和一个未导出的字段 `model`。接下来我们会在 main 包中导入这个包，并使用 Spec 结构体。

```bash
package main

import "structs/computer"
import "fmt"

func main() {
    var spec computer.Spec
    spec.Maker = "apple"
    spec.Price = 50000
    fmt.Println("Spec:", spec)
}
```
包结构如下所示：

```bash
src
   structs
        computer
            spec.go
        main.go
```
在上述程序的第 3 行，我们导入了 computer 包。在第 8 行和第 9 行，我们访问了结构体 Spec 的两个导出字段 Maker 和 Price。执行命令 `go install structs` 和 `workspacepath/bin/structs`，运行该程序。

如果我们试图访问未导出的字段 model，编译器会报错。将 main.go 的内容替换为下面的代码。

```bash
package main

import "structs/computer"
import "fmt"

func main() {
    var spec computer.Spec
    spec.Maker = "apple"
    spec.Price = 50000
    spec.model = "Mac Mini"
    fmt.Println("Spec:", spec)
}
```

在上面程序的第 10 行，我们试图访问未导出的字段 model。如果运行这个程序，编译器会产生错误：`spec.model undefined (cannot refer to unexported field or method model)。`
## 13. 结构体相等性（Structs Equality）
结构体是值类型。如果它的每一个字段都是可比较的，则该结构体也是可比较的。如果两个结构体变量的对应字段相等，则这两个变量也是相等的。

```bash
package main

import (
    "fmt"
)

type name struct {
    firstName string
    lastName string
}


func main() {
    name1 := name{"Steve", "Jobs"}
    name2 := name{"Steve", "Jobs"}
    if name1 == name2 {
        fmt.Println("name1 and name2 are equal")
    } else {
        fmt.Println("name1 and name2 are not equal")
    }

    name3 := name{firstName:"Steve", lastName:"Jobs"}
    name4 := name{}
    name4.firstName = "Steve"
    if name3 == name4 {
        fmt.Println("name3 and name4 are equal")
    } else {
        fmt.Println("name3 and name4 are not equal")
    }
}
```

在线运行程序[13]

在上面的代码中，结构体类型 name 包含两个 string 类型。由于字符串是可比较的，因此可以比较两个 name 类型的结构体变量。

上面代码中 name1 和 name2 相等，而 name3 和 name4 不相等。该程序会输出：

```bash
name1 and name2 are equal
name3 and name4 are not equal
```

如果结构体包含不可比较的字段，则结构体变量也不可比较。

```bash
package main

import (
    "fmt"
)

type image struct {
    data map[int]int
}

func main() {
    image1 := image{data: map[int]int{
        0: 155,
    }}
    image2 := image{data: map[int]int{
        0: 155,
    }}
    if image1 == image2 {
        fmt.Println("image1 and image2 are equal")
    }
}
```

在线运行程序[14]

在上面代码中，结构体类型 image 包含一个 map 类型的字段。由于 map 类型是不可比较的，因此 image1 和 image2 也不可比较。如果运行该程序，编译器会报错：`main.go:18: invalid operation: image1 == image2 (struct containing map[int]int cannot be compared)`

## 14. struct与tag应用
在处理json格式字符串的时候，经常会看到声明struct结构的时候，属性的右侧还有小米点括起来的内容。形如：

```bash
type User struct {
    UserId   int    `json:"user_id" bson:"user_id"`
    UserName string `json:"user_name" bson:"user_name"`
}
```
这个小米点里的内容是用来干什么的呢？

### 14.1 struct成员变量标签（Tag）说明

要比较详细的了解这个，要先了解一下golang的基础，**在golang中，命名都是推荐都是用驼峰方式，并且在首字母大小写有特殊的语法含义：包外无法引用**。但是由经常需要和其它的系统进行数据交互，例如转成json格式，存储到mongodb啊等等。这个时候如果用属性名来作为键值可能不一定会符合项目要求。

所以呢就多了小米点的内容，在golang中叫标签（Tag），在转换成其它数据格式的时候，会使用其中特定的字段作为键值。例如上例在转成json格式：

```bash
package main
import (
    "encoding/json"
    "fmt"
)
type User struct {
    UserId   int
    UserName string
}
type UserTag struct {
    UserId   int    `json:"user_id" bson:"user_id"`
    UserName string `json:"user_name" bson:"user_name"`
}
func main() {
    u := &User{UserId: 1, UserName: "Murphy"}
    j, _ := json.Marshal(u)
    fmt.Printf("struct User echo : %v\n", string(j))
    u_tag := &UserTag{UserId: 1, UserName: "Murphy"}
    j_tag, _ := json.Marshal(u_tag)
    fmt.Printf("struct UserTag echo : %v\n", string(j_tag))
}
```

```bash
[root@localhost struct]# go run stru9.go
struct User echo : {"UserId":1,"UserName":"Murphy"}
struct UserTag echo : {"user_id":1,"user_name":"Murphy"}
```
可以看到直接用struct的属性名做键值。
其中还有一个bson的声明，这个是用在将数据存储到mongodb使用的。
标签是类型的组成部分。

### 14.2 struct成员变量标签（Tag）获取
那么当我们需要自己封装一些操作，需要用到Tag中的内容时，咋样去获取呢？这边可以使用反射包（reflect）中的方法来获取：

```bash
t := reflect.TypeOf(u)

field := t.Elem().Field(0)

fmt.Println(field.Tag.Get("json"))

fmt.Println(field.Tag.Get("bson"))
```
实例

```bash
package main
import (
    "encoding/json"
    "fmt"
    "reflect"
)
func main() {
    type User struct {
        //多个Key使用空格进行分开，然后使用Get方法获取不同Key的值。
        UserId   int    `json:"user_json_id" xml:"user_xml_id"`
        UserName string `json:"user_json_name" xml:"user_xml_name"`
    }
    // 输出json格式
    u := &User{UserId: 1, UserName: "tony"}
    j, _ := json.Marshal(u)
    fmt.Println(string(j))
    // 获取tag中的内容
    t := reflect.TypeOf(u)
    fmt.Println(t)
    field0 := t.Elem().Field(0)
    fmt.Println(field0.Tag.Get("json"))
    fmt.Println(field0.Tag.Get("xml"))
    field1 := t.Elem().Field(1)
    fmt.Println(field1.Tag.Get("json"))
    fmt.Println(field1.Tag.Get("xml"))
}
```

```bash
[root@localhost struct]# go run stru10.go
{"user_json_id":1,"user_json_name":"tony"}
*main.User
user_json_id
user_xml_id
user_json_name
user_xml_name
```



## 15. struct与方法
实例1定义其初始化方法
```bash
package main

import "fmt"

type Member struct { 
     Name string
}

func (m Member)setName(name string){//绑定到Member结构体的方法
    m.Name = name
}

func main() {
     m := Member{}
     m.setName("小明")
     fmt.Println(m.Name)//输出为空
}
```
```bash
[root@localhost struct]# go run stru13.go

```
```bash
package main

import "fmt"

type Member struct { 
     Name string
}

func (m *Member)setName(name string){//绑定到Member结构体的方法
    m.Name = name
}

func main() {
     m := Member{}
     m.setName("小明")
     fmt.Println(m.Name)//输出为空
}

```

```bash
[root@localhost struct]# go run stru13.go
小明
```
上面的代码中，我们会很奇怪，不是调用setName()方法设置了字段Name的值了吗？为什么还是输出为空呢？
这是因为，结构体是值传递，当我们调用setName时，方法接收器接收到是只是结构体变量的一个副本，通过副本对值进行修复，并不会影响调用者，因此，我们可以将方法接收器定义为指针变量，就可达到修改结构体的目的了。

实例2自定义方法初始化

```bash
package main

import "fmt"

type Person struct {
    Name string
    Age int
}

func (self *Person) init(name string ,age int){
     self.Name = name
     self.Age = age
}


func main() {
    var person1 = new(Person)
    person1.init("wd",22)
    //(&person1).init("wd",22)
    fmt.Println(person1)//&{wd 22}
}
```

实例3 方法返回
```bash
package main

import "fmt"

type Person struct {
    Name string
    Age int
}

func (p Person) Getname() string{ //p代表结构体本身的实列，类似python中的self,这里p可以写为self
    fmt.Println(p.Name)
    return p.Name


}

func main() {
    var person1 = new(Person)
    person1.Age = 22
    person1.Name = "wd"
    person1.Getname()// wd
}
```
实例4实现结构体中的方法
如果实现了结构体中的String方法，在使用fmt打印时候会调用该方法，类似与python中的__str__方法.

```bash
package main

import "fmt"

type Person struct {
    Name string
    Age int
}

func (self *Person) String() string{
    return self.Name

}


func main() {
    var person1 = new(Person)
    person1.Name = "wd"
    person1.Age = 22
    fmt.Println(person1)// wd
}
```

## 16.  struct与函数
自定义构造函数

```bash
package main

import "fmt"

type Student struct {
    name string
    age int
    Class string
}

func Newstu(name1 string,age1 int,class1 string) *Student {
    return &Student{name:name1,age:age1,Class:class1}
}
func main() {
   stu1 := Newstu("wd",22,"math")
   fmt.Println(stu1.name) // wd
}
```
## 17. 遍历链表
链表的遍历是通过移动指针进行遍历，当指针到最好一个节点时，其next指针为nil

```bash
package main

import "fmt"

type Node struct {
    data  int
    next  *Node
}

func Shownode(p *Node){   //遍历
    for p != nil{
        fmt.Println(*p)
        p=p.next  //移动指针
    }
}

func main() {
    var head = new(Node)
    head.data = 1
    var node1 = new(Node)
    node1.data = 2

    head.next = node1
    var node2 = new(Node)
    node2.data = 3

    node1.next = node2
    Shownode(head)
}
//{1 0xc42000e1e0}
//{2 0xc42000e1f0}
//{3 <nil>}
```

## 18. 内存对齐影响struct的大小
我们定义一个struct，这个struct有3个字段，它们的类型有byte,int32以及int64,但是这三个字段的顺序我们可以任意排列，那么根据顺序的不同，一共有6种组合。

```bash
type user1 struct {
    b byte
    i int32
    j int64
}
type user2 struct {
    b byte
    j int64
    i int32
}
type user3 struct {
    i int32
    b byte
    j int64
}
type user4 struct {
    i int32
    j int64
    b byte
}
type user5 struct {
    j int64
    b byte
    i int32
}
type user6 struct {
    j int64
    i int32
    b byte
}
```
根据这6种组合，定义了6个struct，分别位user1，user2，…，user6，那么现在大家猜测一下，这6种类型的struct占用的内存是多少，就是unsafe.Sizeof()的值。

大家可能猜测1+4+8=13，因为byte的大小为1，int32大小为4，int64大小为8，而struct其实就是一个字段的组合，所以猜测struct大小为字段大小之和也很正常。

但是，但是，我可以明确的说，这是错误的。

为什么是错误的，因为有内存对齐存在，编译器使用了内存对齐，那么最后的大小结果就不一样了。现在我们正式验证下，这几种struct的值。

```bash
package main
import (
    "fmt"
    "unsafe"
)
type user1 struct {
    b byte
    i int32
    j int64
}
type user2 struct {
    b byte
    j int64
    i int32
}
type user3 struct {
    i int32
    b byte
    j int64
}
type user4 struct {
    i int32
    j int64
    b byte
}
type user5 struct {
    j int64
    b byte
    i int32
}
type user6 struct {
    j int64
    i int32
    b byte
}
func main() {
    var u1 user1
    var u2 user2
    var u3 user3
    var u4 user4
    var u5 user5
    var u6 user6
    fmt.Println("u1 size is ", unsafe.Sizeof(u1))
    fmt.Println("u2 size is ", unsafe.Sizeof(u2))
    fmt.Println("u3 size is ", unsafe.Sizeof(u3))
    fmt.Println("u4 size is ", unsafe.Sizeof(u4))
    fmt.Println("u5 size is ", unsafe.Sizeof(u5))
    fmt.Println("u6 size is ", unsafe.Sizeof(u6))
}
```

```bash
[root@localhost struct]# go run stru11.go
u1 size is  16
u2 size is  24
u3 size is  16
u4 size is  24
u5 size is  16
u6 size is  16
```
内存对齐影响struct的大小 struct的字段顺序影响struct的大小
综合以上两点，我们可以得知，不同的字段顺序，最终决定struct的内存大小，所以有时候合理的字段顺序可以减少内存的开销。

## 19. 使用注意：
### 19.1 struct的赋值
错误实例

```bash
package main

import(
    "fmt"
)

type person struct {
    name string
    age int
}

var P person

P.name = "annie"
P.age = 20

func main() {
    fmt.Printf("The person's name is %s", P.name)
}
```

```bash
[root@localhost struct]# go run stru12.go
# command-line-arguments
./stru12.go:14:1: syntax error: non-declaration statement outside function body
```
正确写法1

```bash
package main

import(
    "fmt"
)

type person struct {
    name string
    age int
}

var P =person{"annie",20}


func main() {
    fmt.Printf("The person's name is %s", P.name)
    }
```
正确写法2

```bash
package main

import(
    "fmt"
)

type person struct {
    name string
    age int
}

var P person


func main() {
    P.name="a"
    P.age=10
    fmt.Printf("The person's name is %s", P.name)
}
```
go和c一样，所有的运算都应该在函数内进行，函数外进行的是语法错误。
函数体外初始化结构体就两个办法，要么一次性全部赋值，要么先声明（全局/局部）变量，在某个函数内进行赋值，在函数体外进行结构体成员赋值相当于函数外面进行运算了。

参考：
- [http://www.ahadoc.com/read/Golang-Detailed-Explanation/ch3.md?wd=goto](http://www.ahadoc.com/read/Golang-Detailed-Explanation/ch3.md?wd=goto)
- [https://segmentfault.com/q/1010000009276425](https://segmentfault.com/q/1010000009276425)
- [https://www.cnblogs.com/wdliu/p/9209419.html](https://www.cnblogs.com/wdliu/p/9209419.html)













