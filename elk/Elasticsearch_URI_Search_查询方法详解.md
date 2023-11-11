
## 1. 通过 URI query 实现搜索

```bash
GET /users/_search?q=2012&df=title&sort=year:desc&from=0&size=10&timeout=1s
{
  "profile": "true"
}
```

 - q 指定查询语句，使用 Query String Syntax
 - df 默认字段，不指定时
 - Sort 排序 /from 和 size 用于分页
 - Profile 可以查看查询时如何被执行的
## 2. Query String Synctax
###  2.1 指定字段 vs 泛查询


`q=title:2012` / `q=2012`



```bash
//只对title字段进行查询
GET  /movies/_search?q=2012&df=title

//泛查询 ，正对_all ,所有字段
GET  /movies/_search?q=2012
{
"profile": "true"
}

//对自定字段进行查询  跟 df 等效
GET  /movies/_search?q=title:2012
{
"profile": "true"
}
```
### 2.2 Term vs Phrase

`Beautiful Mind` 等效于 `Beautiful OR Mind`

`“Beautiful Mind”`, 等效于 `Beautiful AND Mind`。Phrase 查询，还要求前后顺序保存一致

```bash
//使用引号。Phrase
GET  /movies/_search?q=title:"Beautiful Mind"
{
 "profile": "true"
}

//查找美丽心灵，Mind为泛查询 
// 意思就是说 title 是Term  查询 "Beautiful" ，对所有字段查询"Mind"
GET  /movies/_search?q=title:Beautiful Mind
{
 "profile": "true"
}
```
### 2.3 分组和引号

 - `title:(Beautiful AND Mind)`
 - `title=”Beautiful Mind”`

```bash
//分组，Bool 查询 type：BooleanQuery
GET  /movies/_search?q=title:(Beautiful Mind)
{
"profile": "true"
}
```
### 2.4 布尔操作

 - AND / OR / NOT 或者 && / || / !
 - 必须大写
 - title:(matrix NOT reloaded)

```bash
// type：BooleanQuery
// title 里面必须包括Beautiful 跟 Mind
GET  /movies/_search?q=title:(Beautiful AND Mind)
{
    "profile": "true"
}

// type：BooleanQuery 
//必须包括Beautiful 但不包括 Mind
GET  /movies/_search?q=title:(Beautiful NOT Mind)
{
    "profile": "true"
}

// type：BooleanQuery
//包括Beautiful必须有Mind，没有Beautiful但有mind
GET  /movies/_search?q=title:(Beautiful %2BMind)
{
    "profile": "true"
}
```
### 2.5 分组

 + +表示 must
 + -表示 must_not
 + title:(+matrix -reloaded)

### 2.6 范围查询

 区间表示：[] 闭区间 ，{} 开区间

 - `year:{2019 TO 2018}`
 - `year:[* TO 2018]`

### 2.7 算数符号

 - `year:>2010`
 - `year(>2010 && <=2018)`
 - `year:(+>2010 +<=2018)`

```bash
//范围查询，区间写法  / 数学写法
GET  /movies/_search?q=year:>=1980
{
"profile": "true"
}
```
### 2.8 通配符查询
**（通配符查询效率低，占用内容大，不建议使用。特别是放在最前面）**

 - ？代表 1 个字符，* 代表 0 或多个字符
 - `title:mi?d`
 - `title:be*`

```bash
//通配符查询
GET  /movies/_search?q=title:b*
{
"profile": "true"
}
```

### 2.9 正则表达
 - `title:[bt]oy`
### 2.10 模糊匹配与近似查询
 - title:befutifl~1
 - title:”lord rings” ~2

```bash
//模糊匹配 
//用户输错,还能找到
GET  /movies/_search?q=ttile:beautifl~1
{
  "profile": "true"
}
// 近似度匹配 可查出     Lord of the Rings
GET  /movies/_search?q=ttile:"Lord Rings" ~2
{
"profile": "true"
}
```

参考资料：
极客时间：Elasticsearch核心技术与实战
相关阅读：
[初学elasticsearch入门](https://blog.csdn.net/xixihahalelehehe/article/details/109380768)
[Elasticsearch本地安装与简单配置](https://blog.csdn.net/xixihahalelehehe/article/details/109385145)
[docker-compose安装elasticsearch集群](https://blog.csdn.net/xixihahalelehehe/article/details/109389780)
[Elasticsearch 7.X之文档、索引、REST API详解](https://blog.csdn.net/xixihahalelehehe/article/details/109406518)
[Elasticsearch节点，集群，分片及副本详解](https://blog.csdn.net/xixihahalelehehe/article/details/109406875)
[Elasticsearch倒排索引介绍](https://blog.csdn.net/xixihahalelehehe/article/details/109440345)
[elasticsearch Analyzer 进行分词详解](https://blog.csdn.net/xixihahalelehehe/article/details/109447777)
[elasticsearch search API详解](https://blog.csdn.net/xixihahalelehehe/article/details/109449425)
