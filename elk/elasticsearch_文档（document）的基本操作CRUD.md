
## 1. 文档CURD基本操作
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201101212743544.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
### 1.1 元数据

 - _index：文档所属的索引名
 - _type：文档所属的type
 - _id：文档的唯一ID

有了这三个，我们就可以唯一确定一个document了，当然，7.0版本以后我们已经不需要_type了。接下来我们再来看看其他的一些元数据

 - _source：文档的原始JSON数据
 - _field_names：该字段用于索引文档中值不为null的字段名，主要用于exists请求查找指定字段是否为空
 - _ignore：这个字段用于索引和存储文档中每个由于异常（开启了ignore_malformed）而被忽略的字段的名称
 - _meta：该字段用于存储一些自定义的元数据信息
 - _routing：用来指定数据落在哪个分片上，默认值是Id
 - _version：文档的版本信息
 - _score：相关性打分
### 1.2 创建文档
创建文档有以下4种方法：

```csharp
 - PUT /<index>/_doc/<_id>   # 如果文档的ID不存在，则创建新的文档。若有相同的ID，先删除现有文档，然后再创建新的文档，同时版本会增加。
 - POST /<index>/_doc/  # 若不指定文档ID，创建文档时会自动生成。（自动生成文档ID的方式）
 - PUT /<index>/_create/<_id>  #创建新的文档，但是如果ID已经存在，会失败。
 - POST /<index>/_create/<_id>
```

这四种方法的区别是，如果不指定id，则Elasticsearch会自动生成一个id。如果使用_create的方法，则必须保证文档不存在，而使用_doc方法的话，既可以创建新的文档，也可以更新已存在的文档。
在创建文档时，还可以选择一些参数。

请求参数

 - if_seq_no：当文档的序列号是指定值时才更新
 - if_primary_term：当文档的primary term是指定值时才更新
 - op_type：如果设置为create则指定id的文档必须不存在，否则操作失败。有效值为index或create，默认为index
 - op_type：指定预处理的管道id
 - refresh：如果设置为true，则立即刷新受影响的分片。如果是wait_for，则会等到刷新分片后，此次操作才对搜索可见。如果是false，则不会刷新分片。默认值为false
 - routing：指定路由到的主分片
 - timeout：指定响应时间，默认是30秒
 - master_timeout：连接主节点的响应时长，默认是30秒
 - version：显式的指定版本号
 - version_type：指定版本号类型：internal、 external、external_gte、force
 - wait_for_active_shards：处理操作之前，必须保持活跃的分片副本数量，可以设置为all或者任意正整数。默认是1，即只需要主分片活跃。

响应包体

 - _shards：提供分片的信息
 - _shards.total：创建了文档的总分片数量
 - _shards.successful：成功创建文档分片的数量
 - _shards.failed：创建文档失败的分片数量
 - _index：文档所属索引
 - _type：文档所属type，目前只支持_doc
 - _id：文档的id
 - _version：文档的版本号
 - _seq_no：文档的序列号
 - _primary_term：文档的主要术语
 - result：索引的结果，created或者updated

我们在创建文档时，如果指定的索引不存在，则ES会自动为我们创建索引。这一操作是可以通过设置中的`action.auto_create_index`字段来控制的，默认是true。你可以修改这个字段，实现指定某些索引可以自动创建或者所有索引都不能自动创建的目的。

### 1.3 更新文档

```csharp
PUT /<index>/_doc/<id>  #_doc方法是先删除原有的文档，再创建新的
POST /<index>/_update/<_id> #_update方法则是增量更新，它的更新过程是先检索到文档，然后运行指定脚本，最后重新索引。
```
区别就是_update方法支持使用脚本更新，默认的语言是`painless`，你可以通过参数`lang`来进行设置。在请求参数方面，_update相较于_doc多了以下几个参数：

 - lang：指定脚本语言
 - retry_on_conflict：发生冲突时重试次数，默认是0
 - _source：设置为false，则不返回任何检索字段
 - _source_excludes：指定要从检索结果排除的source字段
 - _source_includes：指定要返回的检索source字段

#### 1.3.1 _update
```csharp
curl -X POST "localhost:9200/test/_update/1?pretty" -H 'Content-Type: application/json' -d'
{
    "script" : {
        "source": "ctx._source.counter += params.count",
        "lang": "painless",
        "params" : {
            "count" : 4
        }
    }
}
```

#### 1.3.2 Upsert

```csharp
curl -X POST "localhost:9200/test/_update/1?pretty" -H 'Content-Type: application/json' -d'
{
    "script" : {
        "source": "ctx._source.counter += params.count",
        "lang": "painless",
        "params" : {
            "count" : 4
        }
    },
    "upsert" : {
        "counter" : 1
    }
}
```
#### 1.3.3 doc_as_upsert
当指定的文档不存在时，可以使用upsert参数，创建一个新的文档，而当指定的文档存在时，该请求会执行script中的脚本。如果不想使用脚本，而只想新增/更新文档的话，可以使用`doc_as_upsert`。
curl -X POST "localhost:9200/test/_update/1?pretty" -H 'Content-Type: application/json' -d'
{
    "doc" : {
        "name" : "new_name"
    },
    "doc_as_upsert" : true
}
'
#### 1.3.4 update by query
这个API是用于批量更新检索出的文档的，具体可以通过一个例子来了解。

```csharp
curl -X POST "localhost:9200/twitter/_update_by_query?pretty" -H 'Content-Type: application/json' -d'
{
  "script": {
    "source": "ctx._source.likes++",
    "lang": "painless"
  },
  "query": {
    "term": {
      "user": "kimchy"
    }
  }
}
```
### 1.4 获取文档
ES获取文档用的是GET API，请求的格式是：

```csharp
GET /<index>/_doc/<_id>
```

它会返回文档的数据和一些元数据，如果你只想要文档的内容而不需要元数据时，可以使用

```csharp
GET /<index>/_source/<_id>
```
### 1.5 删除文档
CURD操作只剩下最后一个D了，下面我们就一起来看看ES中如何删除一个文档。
删除指定id使用的请求是

```csharp
DELETE /<index>/_doc/<_id>
```

在并发量比较大的情况下，我们在删除时通常会指定版本，以确定删除的文档是我们真正想要删除的文档。
#### 1.5.1 delete by query
类似于update，delete也有一个delete by query的API。

```csharp
POST /<index>/_delete_by_query
```

它也是要先按照条件来查询匹配的文档，然后删除这些文档。在执行查询之前，Elasticsearch会先为指定索引做一个快照，如果在执行删除过程中，要索引发生改变，则会导致操作冲突，同时返回删除失败。
如果删除的文档比较多，也可以使这个请求异步执行，只需要设置`wait_for_completion=false`即可。
这个API的refresh与delete API的refresh参数有所不同，delete中的refresh参数是设置操作是否立即可见，即只刷新一个分片，而这个API中的refresh参数则是需要刷新受影响的所有分片。

## 2. 批量操作
### 2.1 Bulk API（批量操作）
批量操作的API。
Bulk API的作用：在访问网络API时，每一次的访问都需要重新建立网络开销，因此是非常损耗性能的。 而Bulk API的核心思想就是在一次Rest请求中，对不同索引执行多次操作。它支持四种操作类型：Index、Create、Update、Delete。

```csharp
POST /_bulk
POST /<index>/_bulk
```
在这个请求中，你可以任意使用之前的CRUD请求的组合。
格式：
```csharp
curl -X POST "localhost:9200/_bulk?pretty" -H 'Content-Type: application/json' -d'
{ "index" : { "_index" : "test", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_id" : "2" } }
{ "create" : { "_index" : "test", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }
```
实例1

```csharp
post _bulk
{ "index":{"_index":"users","_id":"1"}}
{"field1":"value1"}
{"delete":{"_index":"users","_id":"2"}}
{"update":{"_index":"users","_id":"1"}}
{"doc":{"field2":"value2"}}
{"create":{"_index":"shops","_id":"1"}}
{"field1":"value1"}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201101215649982.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201101215714691.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
通过上图中实例操作，可以看出：

 - 1）对于索引“users”执行index操作，返回成功；
 - 2）对于索引"users"中，文档ID为2的文档信息进行删除，返回状态是404，结果是not_found，说明在索引“users”中并没有文档ID=2的文档信息；
 - 3）对于索引"users"中，文档ID为2的文档信息进行更新，新增字段field2；
 - 4）对于索引"shops"中，创建文档ID为1的文档信息；

在Bulk API操作中，若有单条操作失败，并不会影响其他操作。同时，返回结果包括了每一条操作执行的结果。

### 2.2 mget（批量读取）
`mget`与`Bulk API`的思路是一样的，都是为了减少网络连接所产生的开销，以提高性能。通过提供一系列的文档ID，在一次API请求中，就可以将所有的文档信息返回回来。

```bash
get /_mget
{
  "docs":[
    {
      "_index":"users",
      "_id":"1"
    },
    {
      "_index":"users",
      "_id":"101"
    },
    {
      "_index":"shops",
      "_id":"1"
    }]
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201101220247464.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
上图中，我们通过mget操作访问索引“users”中文档ID为“1”、“101”的文档信息，访问索引“shops”中文档ID为“1”的文档信息。其中两条均返回成功，而文档ID=101的文档信息没有找到。
### 2.3 msearch（批量查询）
msearch通过一次Rest访问，对不同的索引进行不同的查询。

```bash
post users/_msearch
{}
{"query":{"match_all":{}},"from":1,"size":10}
{}
{"query":{"match_all":{}}}
{"index":"shops"}
{"query":{"match_all":{}}}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020110122135396.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201101221539263.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201101221556458.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
通过上图中可以看出，此次批量查询一共执行了三段查询操作，第一次是针对索引users，查询文档ID大于等于1的文档信息，一共查询10条；第二次是查询索引users中所有的文档信息；第三条是查询索引shops中所有的文档信息。
## 3. 常见错误返回说明及注意事项 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201101222150144.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
## 4. 注意

 - 1、对于Bulk  API、mget、msearch等批量操作的API，通过调用它们可以很好的提高性能，但是在调用时也不要过多的发送数据，否则也会容易导致ES集群过大的压力，造成性能的下降。那么过多的数据一般控制在多少为好呢？一般建议是`1000-5000`个文档，如果文档很大，可以适当减少队列，大小建议是5-15M，默认不能超过100M，否则会报错。
 - 2、虽我们在执行CU操作，或者批量执行CU操作时，动态的向索引更新或者创建了字段。此时并没有对索引预先做mapping定义，但是ES也会根据文档类型进行类型推断，将新增的字段定义在mapping中。在生产环境中，建议做mapping设定后再写入数据。
 - 3、**mget与msearch的区别：mget是通过文档ID列表得到文档信息，msearch是根据查询条件，搜索到相关文档。**
 - 4、自创建文档ID时，需要考虑ID的均衡性，避免产生分配不均衡的问题。


参考资料：
极客时间：Elasticsearch核心技术与实战

相关阅读：
[初学elasticsearch入门](https://blog.csdn.net/xixihahalelehehe/article/details/109380768)
[Elasticsearch本地安装与简单配置](https://blog.csdn.net/xixihahalelehehe/article/details/109385145)
[docker-compose安装elasticsearch集群](https://blog.csdn.net/xixihahalelehehe/article/details/109389780)
[Elasticsearch 7.X之文档、索引、REST API详解](https://blog.csdn.net/xixihahalelehehe/article/details/109406518)
[Elasticsearch节点，集群，分片及副本详解](https://blog.csdn.net/xixihahalelehehe/article/details/109406875)
