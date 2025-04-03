#  Linux Command jq 格式转换



-----

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4c89e81b8fdcd6fd3831db30ea6b206d.gif#pic_center)


## 1. 简介
 `jq`命令可以在命令行处理json数据
## 2. 参数
常用选项

```python
-c 紧凑输出json数据
-e 根据输出结果设置命令退出状态码
-s 读取所有输入到一个数组
-r 输出原始字符串，而不是一个JSON格式
-C 高亮显示
-M 单色显示
-S 排序对象
--tab 使用tab缩进 
```
## 3. 举例
紧凑输出json数据

```bash
$ jq -c . test.json
[{"lon":113.30765,"name":"广州市","code":"4401","lat":23.422825},{"lon":113.59446,"name":"韶关市","code":"4402","lat":24.80296}]
```
根据输出结果设置命令退出状态码

```bash
$ jq -c -e  '.[0]|{names}|.names' test.json
null
$ echo $?
1
```
读取所有输出到一个数组(也就是所在json数据最外层套一个数组)

```bash
$ echo '{"safd":"fsafd"}' | jq -r .
{
  "safd": "fsafd"
}
$ echo '{"safd":"fsafd"}' | jq -s .
[
  {
    "safd": "fsafd"
  }
]
```
输出原始字符串，而不是一个JSON格式

```bash
$ echo '{"safd":"fsafd"}' | jq  .[]
"fsafd"
$ echo '{"safd":"fsafd"}' | jq -r .[]
fsafd
```
单色显示

```bash
$ echo '{"safd":"fsafd"}' | jq  .
{
  "safd": "fsafd"
}
$ echo '{"safd":"fsafd"}' | jq  -M .
{
  "safd": "fsafd"
}
```
排序对象

```bash
$ jq . test.json 
[
  {
    "lon": 113.30765,
    "name": "广州市",
    "code": "4401",
    "lat": 23.422825
  },
  {
    "lon": 113.59446,
    "name": "韶关市",
    "code": "4402",
    "lat": 24.80296
  }
]
$ jq -S . test.json 
[
  {
    "code": "4401",
    "lat": 23.422825,
    "lon": 113.30765,
    "name": "广州市"
  },
  {
    "code": "4402",
    "lat": 24.80296,
    "lon": 113.59446,
    "name": "韶关市"
  }
]
```
以table缩进

```bash
$ echo '{"safd":"fsafd"}' | jq  --tab .
{
    "safd": "fsafd"
}
```
获取上面地理json数据里的name值

```go
$ jq '.[]|{name}' test.json 
{
  "name": "广州市"
}
{
  "name": "韶关市"
}
```
获取第一个name值

```go
$ jq '.[0]|{name}' test.json 
{
  "name": "广州市"
}
```
只打印出第一个map的值：

```go
$ jq '.[0]|.[]' test.json 
113.30765
"广州市"
"4401"
23.422825
```
打印出一个map的name值

```go
$ jq '.[0]|.name' test.json 
"广州市"
```

打印出一个map的name值并已普通字符串显示

```go
$ jq -r '.[0]|.name' test.json 
广州市
```


