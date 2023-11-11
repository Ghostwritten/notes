

-----
## 1. 同步数据库数据到 Elasticsearch
● 需求 – 将数据库中的数据同步到 ES，借助 ES 的全文搜索，提高搜索速度
○ 需要把新增用户信息同步到 Elasticsearch 中
○ 用户信息 Update 后，需要能被更新到 Elasticsearch
○ 支持增量更新
○ 用户注销后，不能被 ES所搜索到

## 2. JDBC Input Plugin & 设计实现思路
● 支持通过 JDBC Input Plugin 将数·据从数据库从读到 Logstash
○ 需要自己提供所需的 JDBC Driver
● Scheduling
○ 语法来自 Rufus-scheduler
○ 扩展了 Cron，支持时区
● State
○ Tracking_column / sql_last_value

```bash
input {
  jdbc {
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    jdbc_connection_string => "jdbc:mysql://localhost:3306/db_example"
    jdbc_user => root
    jdbc_password => ymruan123
    #启用追踪，如果为true，则需要指定tracking_column
    use_column_value => true
    #指定追踪的字段，
    tracking_column => "last_updated"
    #追踪字段的类型，目前只有数字(numeric)和时间类型(timestamp)，默认是数字类型
    tracking_column_type => "numeric"
    #记录最后一次运行的结果
    record_last_run => true
    #上面运行结果的保存位置
    last_run_metadata_path => "jdbc-position.txt"
    statement => "SELECT * FROM user where last_updated >:sql_last_value;"
    schedule => " * * * * * *"
  }
}
output {
  elasticsearch {
    document_id => "%{id}"
    document_type => "_doc"
    index => "users"
    hosts => ["http://localhost:9200"]
  }
  stdout{
    codec => rubydebug
  }
}
```

## 3. Demo
● [https://spring.io/guides/gs/accessing-data-mysql/](https://spring.io/guides/gs/accessing-data-mysql/)
● [logstash导入数据](https://blog.csdn.net/xixihahalelehehe/article/details/109385145)
● 支持 新增 / 更新 / 删除 三种 API
● User 字段包含了一个 last update 的字段
● User 表包含了一个 is deleted 字段
