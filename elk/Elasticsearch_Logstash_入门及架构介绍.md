

-----
## 1. Logstash

● ELT 工具 / 数据搜集处理引擎。支持 200 多个插件

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f4255031c2540b7b396db609fa2c1e98.png)
## 2. Logstash Concepts
● Pipeline
○ 包含了 `input-filter-output` 三个阶段的处理流程
○ 插件生命周期管理
○ 队列管理
● Logstash Event
○ 数据在内部流转时的具体表现形式。数据在 input 阶段被转换为 Event，在 output 被转化成目标格式
数据
○ `Event` 其实是一个 `Java Object`，在配置文件中，对 Event 的属性进行增删改查

## 3. Logstash 架构简介

Codec（Code / Decode）：将原始数据 `decode` 成 `Event`；将 Event encode 成目标。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/24ce7e64f37ced9f06850fe677f98026.png)
## 4. Logstash 配置文件结构
● Bin/logstash –f demo.conf
● Pipeline
○ Input / Filter / Output
● Codec
○ Line / js
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c81a04369606a969b136c775bd37c3fd.png)
## 5. Input Plugins
● 一个 `Pipeline` 可以有多个 `input` 插件
○ Stdin / File
○ Beats / Log4J / Elasticsearch / JDBC / Kafka / Rabbitmq / Redis
○ JMX / HTTP / Websocket / UDP / TCP
○ Google Cloud Storage / S3
○ Github 

## 6. Output Plugin
● 将 Event 发送到特定的目的地，是 Pipeline 的最后一个阶段
● 常见 Output Plugins 
○ Elasticsearch
○ Email / Pageduty
○ Influxdb / Kafka / Mongodb / Opentsdb / Zabbix
○ Http / TCP / 

## 7. Codec Plugins
● 将原始数据 decode 成 Event；将 Event encode 成目标数据
● 内置的 Codec Plugins ([https://www.elastic.co/guide/en/logstash/7.1/codec-plugins.html](https://www.elastic.co/guide/en/logstash/7.1/codec-plugins.html))
○ Line / Multiline
○ JSON / Avro / Cef (ArcSight Common Event Format)
○ Dots /

## 8. Filter Plugins
● 处理 Event
● 内置的 Filter Plugins ([https://www.elastic.co/guide/en/logstash/7.1/filter-plugins.html](https://www.elastic.co/guide/en/logstash/7.1/filter-plugins.html))
○ Mutate – 操作 Event 的字段
○ Metrics – Aggregate metrics
○ Ruby – 执行 Ruby

## 9. Queue
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4853e1d0e49e3cb290da37ddd8aa9876.png)
## 10. 多 Pipelines 实例
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bf954c5fd276a26aaaf05361908baacc.png)
● `Pipeline.works`: Pipeline 线程数，默认是 CPU 核数
● `Pipeline.batch.size`：Batcher 一次批量获取等待处理的文档数，默认 125。需结合 jvm.options
调节
● `Pipeline.batch.delay`：Batcher 等待时间

## 11. Logstash Queue
● In Memory Queue
○ 进程 Crash，机器当机，都会引起数据的丢失
● Persistent Queue
○ Queue.type.persisted (默认是 memory)
■ Queue.max_bytes: 4gb
○ 机器当机，数据也不会丢失；数据保证会被消费；可以替代 Kafka 等消息队列缓冲区的作用
○ [https://www.elastic.co/guide/en/logstash/7.1/persistent-queues.html](https://www.elastic.co/guide/en/logstash/7.1/persistent-queues.html)

## 12. Codec Plugin – Single Line

```bash
$ sudo bin/logstash -e "input{stdin{codec=>line}}output{stdout{codec=> rubydebug}}"
hello world
{
       "message" => "hello world",
          "host" => "master",
    "@timestamp" => 2021-03-16T09:20:58.992Z,
      "@version" => "1"
}

$ sudo bin/logstash -e "input{stdin{codec=>json}}output{stdout{codec=> rubydebug}}"
{"hello":"world"}   
{
         "hello" => "world",
          "host" => "master",
    "@timestamp" => 2021-03-16T10:45:38.604Z,
      "@version" => "1"
}

#dots一般处理进度
$ sudo bin/logstash -e "input{stdin{codec=>line}}output{stdout{codec=> dots}}"
hello world
.{hello world}   
.
```
## 13. Codec Plugin - Multiline
● 设置参数
○ Pattern： 设置行匹配的正则表达式
○ What ：如果匹配成功，那么匹配行属于上一个事件还是下一个事件
■ Previous / Next
○ Negate true / false：是否对 pattern 结果取反
■ True 

## 14. Codec Plugin – Multiline(异常日志）
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bb6dfd305841b6a2079bbdefeafbfe6d.png)
```bash
$ cat multiline-exception.conf 
input {
  stdin {
    codec => multiline {
      pattern => "^\s"
      what => "previous"
    }
  }
}


filter {}

output {
  stdout { codec => rubydebug }
}
```


```bash
$ ./bin/logstash -f multiline-exception.conf
.............
hello
{
       "message" => "hello world",
      "@version" => "1",
    "@timestamp" => 2021-03-16T10:52:49.565Z,
          "host" => "master"
}
```

## 15. Input Plugin – File
● 支持从文件中读取数据，如日志文件
● 文件读取需要解决的问题
○ 只被读取一次。重启后需要从上次读取的位置继续（通过 sincedb 实现）
● 读取到文件新内容，发现新文件
● 文件发生归档操作（文档位置发生变化，日志 rotation），不能影响当前的内容读取




## 16. Filter Plugin
● Filter Plugin 可以对 Logstash Event 进行各种处理，例如解析，删除字段，类型转换
○ `Date`：日期解析
○ `Dissect`：分割符解析
○ `Grok`：正则匹配解析
○ `Mutate`：处理字段。重命名，删除，替换
○ Ruby：利用 Ruby 代码来动态修改 Event

## 17. Filter Plugin - Mutate
● 对字段做各种操作
○ Convert 类型转换
○ Gsub 字符串替换
○ Split / Join / Merge 字符串切割，数组合并字符串，数组合并数组
○ Rename 字段重命名
○ Update / Replace 字段内容更新替换
○ Remove_field 字段删除

