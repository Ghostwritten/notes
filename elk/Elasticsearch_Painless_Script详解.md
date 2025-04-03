

---
## 1. Painless 简介
自 ES 5.x 后引入，专门为 ES 设置，扩展了 Java 的语法
6.0 开始，ES 只支持 Painless。Grooby ,JavaScript 和 Python 都不在支持
Painless 支持所有的 Java 的数据类型及 Java API 子集
Painless Script 具备以下特性

 - 高性能 、 安全
 - 支持显示类型或者动态定义类型


##Painless 的用途
 Painless 可以对文档字段进行加工处理

 - 更新或者删除字段，处理数据聚合操作
 - Script Field： 对返回的字段提前进行计算
 - Function Score：对文档的算分进行处理

在 Ingest Pipeline 中执行脚本
在 Reindex API，Update By Query 时，对数据进行处理

 - 脚本编写的语言，默认为painless。
 - 脚本本身可以指定为内联脚本的source或存储脚本的id。
 - 应传递给脚本的任何命名参数。



## 2. 参数
lang

 - 指定编写脚本的语言，默认为painless。

source，id

 - 指定脚本的来源，inline脚本是指定`source`，，存储的脚本是指定的id，并从群集状态中检索（请参阅存储的脚本）。

params

 - 指定作为变量传递到脚本的任何命名参数。


## 3. 首选参数
Elasticsearch第一次看到一个新脚本，它会编译它并将编译后的版本存储在缓存中，编译可能是一个繁重的过程。

如果需要将变量传递给脚本，则应将它们作为命名参数传递给脚本本身而不是硬编码值，例如，如果你希望能够将字段值乘以不同的乘数，请不要将乘数硬编码到脚本中：

```bash
"source": "doc['my_field'] * 2"
```

相反，将其作为命名参数传递：

```bash
  "source": "doc['my_field'] * multiplier",
  "params": {
    "multiplier": 2
  }
```

第一个版本每次乘数改变时都必须重新编译，第二个版本只编译一次。

如果你在很短的时间内编译了太多独特的脚本，Elasticsearch将使用`circuit_breaking_exception`错误拒绝新的动态脚本。默认情况下，每分钟将编译最多15个内联脚本，你可以通过设置`script.max_compilations_rate`动态更改此设置。


## 4. 简短脚本形式
可以使用简短脚本形式来简化，在简短形式中，script由字符串而不是对象表示，该字符串包含脚本的源。

简写：

```bash
"script": "ctx._source.likes++"
```

正常形式的相同脚本：

```bash
  "script": {
    "source": "ctx._source.likes++"
  }
```


## 5. 通过 Painless 脚本访问字段
| 上线文                  | 语法                     |
|----------------------|------------------------|
| Ingestion            | ctx.field_name         |
| Update               | ctx._source.field_name |
| Search & Aggregation | doc{“field_name”]      |



## 6. 示例
### 6.1 案例 1：Script Processsor
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0fe1bcd8c0957b364e296a19d4a98af1.png)

```bash
# 增加一个 Script Prcessor
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "to split blog tags",
    "processors": [
      {
        "split": {
          "field": "tags",
          "separator": ","
        }
      },
      {
        "script": {
          "source": """
          if(ctx.containsKey("content")){
            ctx.content_length = ctx.content.length();
          }else{
            ctx.content_length=0;
          }
        """
        }
      },
      {
        "set": {
          "field": "views",
          "value": 0
        }
      }
    ]
  },
  "docs": [
    {
      "_index": "index",
      "_id": "id",
      "_source": {
        "title": "Introducing big data......",
        "tags": "hadoop,elasticsearch,spark",
        "content": "You konw, for big data"
      }
    },
    {
      "_index": "index",
      "_id": "idxx",
      "_source": {
        "title": "Introducing cloud computering",
        "tags": "openstack,k8s",
        "content": "You konw, for cloud"
      }
    }
  ]
}
```

### 6.2 案例 2：文档更新计数
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0e36d26d87f28adf033bfeed3a48950f.png)

```bash
DELETE tech_blogs
PUT tech_blogs/_doc/1
{
  "title":"Introducing big data......",
  "tags":"hadoop,elasticsearch,spark",
  "content":"You konw, for big data",
  "views":0
}

POST tech_blogs/_update/1
{
  "script": {
    "source": "ctx._source.views += params.new_views",
    "params": {
      "new_views":100
    }
  }
}

# 查看views计数
POST tech_blogs/_search
```

### 6.3 案例 3：搜索时的 Script 字段
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/381147e1a2ddcab3b4c54d4dd1ca7640.png)

```bash
GET tech_blogs/_search
{
  "script_fields": {
    "rnd_views": {
      "script": {
        "lang": "painless",
        "source": """
          java.util.Random rnd = new Random();
          doc['views'].value+rnd.nextInt(1000);
        """
      }
    }
  },
  "query": {
    "match_all": {}
  }
}
```

### 6.4 Script :Inline v.s Stored
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f60076cbeb4630c471943f9d772642ff.png)

```bash
#保存脚本在 Cluster State
POST _scripts/update_views
{
  "script":{
    "lang": "painless",
    "source": "ctx._source.views += params.new_views"
  }
}

POST tech_blogs/_update/1
{
  "script": {
    "id": "update_views",
    "params": {
      "new_views":1000
    }
  }
}
```

### 6.5 示例4
首先，在集群状态下创建名为calculate-score的脚本：

```bash
POST _scripts/calculate-score
{
  "script": {
    "lang": "painless",
    "source": "Math.log(_score * 2) + params.my_modifier"
  }
}
```

可以使用以下命令检索相同的脚本：

```bash
GET _scripts/calculate-score
```

可以通过指定id参数来使用存储的脚本，如下所示：

```bash
GET _search
{
  "query": {
    "script": {
      "script": {
        "id": "calculate-score",
        "params": {
          "my_modifier": 2
        }
      }
    }
  }
}
```

删除：

```bash
DELETE _scripts/calculate-score
```


## 3. 本缓存
编译的开销相较大
Elasticsearch 会将甲苯编译后缓存在 Cache 中

 - Inline scripts 和 Stored Scripts 都会被缓存
 - 默认缓存 100 个脚本
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7318cda6aa0d692023d4164b9f30b8f4.png)



