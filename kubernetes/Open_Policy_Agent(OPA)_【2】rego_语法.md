#  Open Policy Agent(OPA) rego 语法
tags: OPA
<!-- catalog: ~语法~ -->




---

 - [Open Policy Agent(OPA) 【1】介绍](https://blog.csdn.net/xixihahalelehehe/article/details/116905513?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164188925916780271953559%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164188925916780271953559&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-2-116905513.nonecase&utm_term=opa&spm=1018.2226.3001.4450)
 - [Open Policy Agent(OPA) 【2】rego语法](https://blog.csdn.net/xixihahalelehehe/article/details/116998878?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164188925916780271953559%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164188925916780271953559&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-116998878.nonecase&utm_term=opa&spm=1018.2226.3001.4450)
 - [Open Policy Agent(OPA) 【3】实战](https://blog.csdn.net/xixihahalelehehe/article/details/116904422?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164188925916780271953559%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164188925916780271953559&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-3-116904422.nonecase&utm_term=opa&spm=1018.2226.3001.4450)
 - [云原生圣经](https://ghostwritten.blog.csdn.net/article/details/108562082)



![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/393f9bb1b0b3f32ff9ec1a705d0bd8bc.png)


## 1. 背景
OPA专门用于推理结构化文档中表示的信息。您的服务及其用户发布的数据可以使用OPA的本机查询语言Rego进行检查和转换。

## 2. 什么是rego
`Rego`受到Datalog的启发，Datalog是一种众所周知的数十年的查询语言。Rego扩展了Datalog以支持诸如JSON之类的结构化文档模型。

Rego查询是对存储在OPA中的数据的断言。这些查询可用于定义策略，该策略枚举违反系统预期状态的数据实例

## 3. 为什么使用rego
使用Rego定义易于阅读和编写的策略。

Rego致力于为引用嵌套文档提供强大的支持，并确保查询正确无误。

Rego是声明性的，因此策略作者可以专注于应返回的查询，而不是应如何执行查询。这些查询比命令式语言中的查询更简单，更简洁。

像其他支持声明性查询语言的应用程序一样，OPA能够优化查询以提高性能。
## 4. 语法
### 4.1 基础知识

```bash
pi := 3.14159
pi
3.14159



rect := {"width": 2, "height": 4}
rect
{
  "height": 4,
  "width": 2
}

rect == {"height": 4, "width": 2}
true



v { "hello" == "world" }
undefined decision
v == true
undefined decision
v != true
undefined decision
```
根据变量来定义规则
规则本身可以直观地理解为：

```bash
rule-name IS value IF body
```
示例：
```bash
t { x := 42; y := 41; x > y }
t
true

t2 {
    x := 42
    y := 41
    x > y
}
```

规则中的表达式顺序不会影响文档的内容。

```bash
s {
    x > y
    y = 41
    x = 42
}
s
true

```
有一个例外：如果使用赋值:=，编译器将检查所赋值的变量是否尚未使用。

```bash
z {
    y := 41
    y := 42
    43 > y
}
1 error occurred: module.rego:5: rego_compile_error: var y assigned above
```

```bash
sites = [{"name": "prod"}, {"name": "smoke1"}, {"name": "dev"}]
r { sites[_].name == "prod" }
```
上述规则r断言sites该name属性等于的地方（至少）存在一个文档"prod"。

结果：`true`


我们可以使用定义集合文档而不是布尔文档的规则来概括上面的示例：

```bash
q[name] { name := sites[_].name }
[
  "prod",
  "smoke1",
  "dev"
]

p { q["prod"] }
查询p将具有相同的结果：
true

q["smoke2"]
undefined decision
```
### 4.2 标量值
标量值是Rego中最简单的术语类型。标量值可以是字符串，数字，布尔值或null。

可以仅根据标量值定义文档。这对于定义在多个位置引用的常量很有用。例如：

```bash
greeting   := "Hello"
max_height := 42
pi         := 3.14159
allowed    := true
location   := null

[greeting, max_height, pi, allowed, location]
[
  "Hello",
  42,
  3.14159,
  true,
  null
]
```
### 4.3 综合值

```bash
cube := {"width": 3, "height": 4, "depth": 5}

cube.width
3


a := 42
b := false
c := null
d := {"a": a, "x": [b, c]}

+----+-------+------+---------------------------+
| a  |   b   |  c   |             d             |
+----+-------+------+---------------------------+
| 42 | false | null | {"a":42,"x":[false,null]} |
+----+-------+------+---------------------------+
```
### 4.4 对象
对象是无序键值集合。在Rego中，任何值类型都可以用作对象键。例如，以下分配将端口号映射 到IP地址列表（表示为字符串）。

```bash
ips_by_port := {
    80: ["1.1.1.1", "1.1.1.2"],
    443: ["2.2.2.1"],
}

ips_by_port[80]
[
  "1.1.1.1",
  "1.1.1.2"
]

some port; ips_by_port[port][_] == "2.2.2.1"
+------+
| port |
+------+
| 443  |
+------+

将Rego值转换为JSON时，会将非字符串对象键编组为字符串（因为JSON不支持非字符串对象键）。
ips_by_port
{
  "443": [
    "2.2.2.1"
  ],
  "80": [
    "1.1.1.1",
    "1.1.1.2"
  ]
}
```
### 4.5 Sets
除了数组和对象外，Rego还支持设置值。集是唯一值的无序集合。就像其他复合值一样，可以根据标量，变量，引用和其他复合值来定义集合。例如：

```bash
s := {cube.width, cube.height, cube.depth}
+---------+
|    s    |
+---------+
| [5,4,3] |
+---------+
```

> 设置文档是没有键的值的集合。当序列化为JSON或不支持集合数据类型的其他格式时，OPA将集合文档表示为数组。集与数组或对象之间的重要区别在于，对集和数组或对象进行键控时，它们是无键的，即，您不能引用集内元素的索引。

比较集合时，元素的顺序无关紧要：

```bash
{1,2,3} == {3,1,2}
true

{1,2,3} == {3,x,2}
1 error occurred: 1:1: rego_unsafe_var_error: var x is unsafe
```
因为集合与对象共享大括号语法，并且使用定义了一个空对象{}，所以必须使用不同的语法来构造一个空集合：

```bash
count(set())
0
```
### 4.6 Variables

变量是Rego中的另一种术语。它们同时出现在规则的开头和正文中。

可以将出现在规则开头的变量视为规则的输入和输出。与许多编程语言不同，变量是输入或输出，而在许多编程语言中，变量同时是输入和输出。如果查询提供变量的值，则该变量为输入，如果查询不提供变量的值，则该变量为输出。


```bash
sites := [
    {"name": "prod"},
    {"name": "smoke1"},
    {"name": "dev"}
]

q[name] { name := sites[_].name }


q[x]
+----------+----------+
|    x     |   q[x]   |
+----------+----------+
| "prod"   | "prod"   |
| "smoke1" | "smoke1" |
| "dev"    | "dev"    |
+----------+----------+

q["dev"]
"dev"
```

### 4.7 References
引用用于访问嵌套文档。

本节中的示例使用“示例”部分中定义的数据。

最简单的参考不包含任何变量。例如，以下参考从示例数据中返回第一个站点文档中第二个服务器的主机名：

```bash
sites[0].servers[1].hostname
"helium"
```

### 4.8 Keys
引用可以包含变量作为键。以这种方式编写的引用用于从集合中的每个元素中选择一个值。

以下参考将在示例数据中选择所有服务器的主机名：

```bash
sites[i].servers[j].hostname
+---+---+------------------------------+
| i | j | sites[i].servers[j].hostname |
+---+---+------------------------------+
| 0 | 0 | "hydrogen"                   |
| 0 | 1 | "helium"                     |
| 0 | 2 | "lithium"                    |
| 1 | 0 | "beryllium"                  |
| 1 | 1 | "boron"                      |
| 1 | 2 | "carbon"                     |
| 2 | 0 | "nitrogen"                   |
| 2 | 1 | "oxygen"                     |
+---+---+------------------------------+
```
从概念上讲，这与以下命令式（Python）代码相同：

```bash
def hostnames(sites):
    result = []
    for site in sites:
        for server in site.servers:
            result.append(server.hostname)
    return result
```
在上面的参考中，我们有效地使用了名为i和的变量j来迭代集合。如果变量在引用外部未使用，我们更喜欢用下划线（_）字符替换它们。上面的参考可以重写为：

```bash
sites[_].servers[_].hostname
+------------------------------+
| sites[_].servers[_].hostname |
+------------------------------+
| "hydrogen"                   |
| "helium"                     |
| "lithium"                    |
| "beryllium"                  |
| "boron"                      |
| "carbon"                     |
| "nitrogen"                   |
| "oxygen"                     |
+------------------------------+
```
### 4.9 Composite Keys

```bash
s := {[1, 2], [1, 4], [2, 6]}
s[[1, 2]]
[
  1,
  2
]


s[[1, x]]
+---+-----------+
| x | s[[1, x]] |
+---+-----------+
| 2 | [1,2]     |
| 4 | [1,4]     |
+---+-----------+
```
### 4.10 Array Comprehensions
数组推导从子查询中构建数组值。数组推导具有以下形式：

```bash
[ <term> | <body> ]
```
例如，以下规则定义一个对象，其中的键是应用程序名称，值是在其中部署应用程序的服务器的主机名。服务器的主机名以数组表示。

```bash
app_to_hostnames[app_name] = hostnames {
    app := apps[_]
    app_name := app.name
    hostnames := [hostname | name := app.servers[_]
                            s := sites[_].servers[_]
                            s.name == name
                            hostname := s.hostname]
}

结果
app_to_hostnames[app]
+-----------+------------------------------------------------------+
|    app    |                app_to_hostnames[app]                 |
+-----------+------------------------------------------------------+
| "web"     | ["hydrogen","helium","beryllium","boron","nitrogen"] |
| "mysql"   | ["lithium","carbon"]                                 |
| "mongodb" | ["oxygen"]                                           |
+-----------+------------------------------------------------------+
```
### 4.11 Set Comprehensions
Set Comprehensions从子查询中构建设置值。集合理解的形式为

```bash
{ <term> | <body> }

a := [1, 2, 3, 4, 3, 4, 3, 4, 5]
b := {x | x = a[_]}

+---------------------+-------------+
|          a          |      b      |
+---------------------+-------------+
| [1,2,3,4,3,4,3,4,5] | [1,2,3,4,5] |
+---------------------+-------------+
```


参考：

 - [OPA policy-language rego](https://www.openpolicyagent.org/docs/latest/policy-language/)
 - [OPA Gatekeeper: Policy and Governance for Kubernetes](https://kubernetes.io/blog/2019/08/06/opa-gatekeeper-policy-and-governance-for-kubernetes/)
 - [Kubernetes Admission Control OPA](https://www.openpolicyagent.org/docs/v0.12.2/kubernetes-admission-control/)
 - [OPA On Kubernetes: An Introduction For Beginners](https://www.velotio.com/engineering-blog/deploy-opa-on-kubernetes)

