

-----
## 1. Index Pattern
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1be200c0305b3c2d93807e95e4d53f15.png)

## 2. 插入数据

```bash
PUT /logstash-2015.05.18
{
  "mappings": {
    "properties": {
      "geo": {
        "properties": {
          "coordinates": {
            "type": "geo_point"
          }
        }
      }
    }
  }
}



PUT /logstash-2015.05.19
{
  "mappings": {
    "properties": {
      "geo": {
        "properties": {
          "coordinates": {
            "type": "geo_point"
          }
        }
      }
    }
  }
}


PUT /logstash-2015.05.20
{
  "mappings": {
    "properties": {
      "geo": {
        "properties": {
          "coordinates": {
            "type": "geo_point"
          }
        }
      }
    }
  }
}
```

## 3. 导入数据

```bash
# For Mac & Windows
curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/_bulk?pretty' --data-binary @logs.jsonl

curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/bank/account/_bulk?pretty' --data-binary @accounts.json
```
## 4. 打开kibana创建第一个index pattern
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6a1082a7e71750a5088f28ff48657f57.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9b7ffac8a3cf07480858839ec4aa837f.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0ca463ef315fb449887a0e0bf15ea817.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5eeadc54376b381be1fd76e5f4ab0bdd.png)

## 5. 创建第二个index pattern
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/522a749ae4703d11397af2ad77c5310c.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0ac6a1ae60fa838d6afdd053d74ffddb.png)

