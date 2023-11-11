
## 1. 文档
索引和文档偏向于开发的视角 ，逻辑
节点和分片偏向于运维的视角，物理

### 1.1 什么是文档（document）
使用案例出发，Elasticsearch 是面向文档，**文档是所有搜索数据的最小单元**。

 1. 案例一：每个公司都有业务日志平台，比如交易业务日志。文档：每一条日志文件中的日志项，就是文档
 2. 案例二：可以搜索并播放电影的在线视频网站文档：每一个电影的具体信息，就是文档
 3. 案例三：可以搜索并下载文件的云存储网站，类似百度云文档：每一个文件具体内容信息，就是文档

等等案例很多，那么文档就是类似数据库里面的一条长长的存储记录。文档（Document）是索引信息的基本单位。

### 1.2 文档格式（json）
文档被序列化成为 JSON 格式，物理保存在一个索引中。JSON 是一种常见的互联网数据交换格式：

 - 文档字段名：JSON 格式由 name/value pairs 组成，对应的 name 就是文档字段名
 - 文档字段类型：每个字段都有对应的字段类型：String、integer、long 等，并支持数据&嵌套

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201031190536623.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)

### 1.3 文档id
每个文档都会有一个 Unique ID，其字段名称为 `_id` ：

 - 自行设置指定 ID 或通过 Elasticsearch 自动生成
 - 其值不会被索引

注意：该 id 字段的值可以在某些查询 term, terms, match, querystring, simplequerystring 等中访问，但不能在 aggregations，scripts 或 sorting 中使用。如果需要对 id 字段进行排序或汇总，建议新建一个文档字段复制 id 字段的内容
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201031190439773.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)

### 1.4 文档元数据
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201031190805125.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)


## 2. 索引
### 2.1 什么是索引
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201031191110831.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
type不同版本差异：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201031191740795.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)

其中 _type 文档所属类型名，需要关注版本不同之间区别：

 - 7.0 之前，一个索引可以设置多个 types
 - 7.0 开始，被 Deprecated 了。一个索引只能创建一个 type，值为 _doc
### 2.2 索引的不同意思
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201031191148722.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
### 2.3 索引测试

```bash
GET http://localhost:9200/kibana_sample_data_flights
```

```bash
{
    "kibana_sample_data_flights": {
        "aliases": {},
        "mappings": {
            "properties": {
                "AvgTicketPrice": {
                    "type": "float"
                },
                "Cancelled": {
                    "type": "boolean"
                },
                "Carrier": {
                    "type": "keyword"
                },
                "DestLocation": {
                    "type": "geo_point"
                },
                "FlightDelay": {
                    "type": "boolean"
                },
                "FlightDelayMin": {
                    "type": "integer"
                },
                "timestamp": {
                    "type": "date"
                }
            }
        },
        "settings": {
            "index": {
                "number_of_shards": "1",
                "auto_expand_replicas": "0-1",
                "blocks": {
                    "read_only_allow_delete": "true"
                },
                "provided_name": "kibana_sample_data_flights",
                "creation_date": "1566271868125",
                "number_of_replicas": "0",
                "uuid": "SfR20UNiSLKJWIpR1bcrzQ",
                "version": {
                    "created": "7020199"
                }
            }
        }
    }
}
```
根据返回结果，我们知道：

 - mappings：定义文档字段的类型
 - settings：定义不同数据分布
 - aliases：定义索引的别名，可以通过别名访问该索引

索引，是逻辑空间概念，每个索引有对那个的 Mapping 定义，对应的就是文档的字段名和字段类型。相比后面会讲到分片，是物理空间概念，索引中存储数据会分散到分片上。

aliases 别名大有作为，比如 myindex 迁移到 myindex_new , 数据迁移后，只需要保持一致的别名配置。那么通过别名访问索引的业务方都不需要修改，直接迁移即可。

### 2.4 索引与mysql比对
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201031191902506.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
## 3. REST API
各种语言很方便调用
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201031192117379.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
如图，Elasticsearch 提供了 REST API，方便，相关索引 API 如下：

```bash
# 查看索引相关信息
GET kibana_sample_data_ecommerce

# 查看索引的文档总数
GET kibana_sample_data_ecommerce/_count

# 查看前10条文档，了解文档格式
POST kibana_sample_data_ecommerce/_search
{
}

# _cat indices API
# 查看indices
GET /_cat/indices/kibana*?v&s=index

# 查看状态为绿的索引
GET /_cat/indices?v&health=green

# 按照文档个数排序
GET /_cat/indices?v&s=docs.count:desc

# 查看具体的字段
GET /_cat/indices/kibana*?pri&v&h=health,index,pri,rep,docs.count,mt

# How much memory is used per index?
GET /_cat/indices?v&h=i,tm&s=tm:desc
```
kibana执行效果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201031192630903.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
参考资料：
极客时间：Elasticsearch核心技术与实战

相关阅读：
[初学elasticsearch入门](https://blog.csdn.net/xixihahalelehehe/article/details/109380768)
[Elasticsearch本地安装与简单配置](https://blog.csdn.net/xixihahalelehehe/article/details/109385145)

