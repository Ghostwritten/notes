

----
## 1. 排序

 - ES 默认采用相关性算分对结果进行降序排序
 - 可以通过设置 `sorting` 参数，自行设定排序
 - 如果不指定_score, 算分为 null

## 2. 单字段排序

```bash
POST /kibana_sample_data_ecommerce/_search
{
  "size": 5,
  "query": {
    "match_all": {

    }
  },
  "sort": [
    {"order_date": {"order": "desc"}}
  ]
}
```
算分为空
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e78e529e1cb8aed336f651b8064709de.png)

## 3. 多字段排序

 - 组合多个条件
 - 优先考虑写在前面的排序
 - 支持对相关性算分进行排序

```bash
POST /kibana_sample_data_ecommerce/_search
{
  "size": 5,
  "query": {
    "match_all": {

    }
  },
  "sort": [
    {"order_date": {"order": "desc"}},
    {"_doc":{"order": "asc"}},
    {"_score":{ "order": "desc"}}
  ]
}

GET kibana_sample_data_ecommerce/_mapping
```
有相关性算分
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/153e4cd110fc54b1ba477a9a54c57537.png)

## 4. 对 text 字段进行排序
默认会报错，需打开fielddata

```bash
POST /kibana_sample_data_ecommerce/_search
{
  "size": 5,
  "query": {
    "match_all": {

    }
  },
  "sort": [
    {"customer_full_name": {"order": "desc"}}
  ]
}
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/956cb8bd677d7a9a375cc91032319705.png)

```bash

#打开 text的 fielddata
PUT kibana_sample_data_ecommerce/_mapping
{
  "properties": {
    "customer_full_name" : {
          "type" : "text",
          "fielddata": true,
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
  }
}
```
再次执行查询可返回结果
## 5. 排序的过程

 - 排序是针对字段原始内容进行的。倒排索引无法发挥作用
 - 需要用到正排索引。通过文档 ID 和字段快速得到字段原始内容

ES 有 2 种实现方式

 - Fielddata
 - Doc Values (列式存储，对 Text 类型无效）

## 6. Doc Values vs. Field Data

|      | Doc Values      | Field data                  |
|------|-----------------|-----------------------------|
| 何时创建 | 索引时，和倒排索引一起创建   | 搜索时候动态创建                    |
| 创建位置 | 磁盘文件            | JVM Heap                    |
| 优点   | 避免大量内存占用        | 索引速度快，不占用额外的磁盘空间            |
| 缺点   | 降低索引速度，占用额外磁盘空间 | 文档过多时，动态创建开销大，占用过多 JVM Heap |
| 缺省值  | ES 2.x 之后       | ES1.x 及之前                   |


## 7. 打开 Fielddata

 - 默认关闭，可以通过 Mapping 设置打开。修改设置后，即时生效，无需缩减索引

其他字段类型不支持，支持对 Text 进行设定
打开后，可以对 Text 字段进行排序，但是结果无法满足预期，不建议使用
部分情况下打开，满足一些聚合分析的特定需求



## 8. 关闭 keyword的 doc values
默认启动，可以通过 Mapping 设置关闭

 - 增减索引速度 / 减少磁盘空间

如果重新打开，需要重建索引
什么时候需要关闭

 - 明确不需要做排序及聚合分析

```bash
PUT test_keyword
PUT test_keyword/_mapping
{
  "properties": {
    "user_name":{
      "type": "keyword",
      "doc_values":false
    }
  }
}

DELETE test_keyword

PUT test_text
PUT test_text/_mapping
{
  "properties": {
    "intro":{
      "type": "text",
      "doc_values":true
    }
  }
}

DELETE test_text
```
## 9. 获取 Doc Values & Fielddata 中储存的内容
Text 类型的不支持 Doc Values
Text 类型打开 Fielddata 后，可以查看分词后的数据

```bash
DELETE temp_users
PUT temp_users
PUT temp_users/_mapping
{
  "properties": {
    "name":{"type": "text","fielddata": true},
    "desc":{"type": "text","fielddata": true}
  }
}

Post temp_users/_doc
{"name":"Jack","desc":"Jack is a good boy!","age":10}

#打开fielddata 后，查看 docvalue_fields数据
POST  temp_users/_search
{
  "docvalue_fields": [
    "name","desc"
    ]
}

#查看整型字段的docvalues
POST  temp_users/_search
{
  "docvalue_fields": [
    "age"
    ]
}
```

