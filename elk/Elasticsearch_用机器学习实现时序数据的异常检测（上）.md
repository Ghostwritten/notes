


## 1. 异常检测所解决的问题
● 解决一些基于规则或者 Dashboard 难以实时发现的问题
● IT 运维

 - ○ 如何知道系统正常运行 / 如何调节阈值触发合适的报警 / 如何进行归因分析

● 信息安全

 - ○ 哪些用户构成了内部威胁 / 系统是否感染了病毒

● 物联网 / 数据采集监控

 - ○ 工厂和设备是否正常运营

## 2. 什么是正常

 - ○ 随着时间的推移，某个个体一直表现出一致的行为
 - ○ 某个个体和他的同类比较，一直表现出和其他个体一致的行为


##  3. 什么是异常
○ 和自己比 - 个体的行为发生了急剧的变化
○ 和他人比 - 个体明显区别于其他的个体

## 4. 判定异常需要一定的指导
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/57c7eb36b197aa25754b47a599acf313.png)
## 5. 相关术语
● Elastic 平台的机器学习功能
○ Elastic 的ML，主要针对时序数据的异常检测和预测
● 非监督机器学习
○ 不需要使用人工标签的数据来学习，仅仅依靠历史数据自动学习
● 贝叶斯统计
○ 一种概率计算方法，使用先验结果来计算现值或者预测未来的数值
● 异常检测
○ 异常代表的是不同的，但未必代表的是坏的 / 定义异常需要一些指导，从哪个方面去看


## 6. 如何学习“正常”
● 观察不同的人每天走路的步数，由此预测明天他会走多少步
● 需要观察不同的人，需要观察多久？
○ 一天 / 一周 / 一个月 / 一年 / 十年
● 直觉： 观察的数据多，你的预测越准确
● 使用这些观察来创建一个模型
○ 概率分布函数；使用这个模型找出什么事几乎不可能的事件


## 7. 机器学习帮你自动挑选模型
● 使用成熟的机器学习技术，挑选适合数据的正确的统计模型
● 更好的模型 = 更好的异常检测 = 更少的误报和漏报
● 出现在低概率区域，发现异

## 8. 模型与需要考虑任何的周期
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fd4d63ed941712b3f72a753555d92c58.png)

## 9. ES ML：单指标 / ES ML：单指标 / 种群分析
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/383325c2506479216285260957e96efd.png)

## 10. 单指标任务
● Create New Job
● 选择 Index Pattern，选择 Fare quote
● 选择单指标任务

 - ○ Aggregation ○ Field
 - ○ Bucket Span （取决于用户的数据业务 /数据的采样时间 / 波动情况 / 需要预测的频率）
 - ○ Use full dat

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9bf077460e0e69eba90200de668b11bd.png)
## 11. Demo
● Create New Single Metric Job

 - ○ 基于 Index Pattern 和 Full data set

● Create Customer URL

 - ○ Saved Query

● 对数据实现预测
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cb4fd717f83a990469217ca049e89980.png)
下载数据
[https://download.elasticsearch.org/demos/machine_learning/gettingstarted/server_metrics.tar.gz](https://download.elasticsearch.org/demos/machine_learning/gettingstarted/server_metrics.tar.gz)


```bash
[root@master machine_leanrning]# tar -zxvf server_metrics.tar.gz 
files/
files/server-metrics_16.json
files/server-metrics_6.json
files/server-metrics_20.json
files/server-metrics_7.json
files/server-metrics_17.json
files/create_single_metric.sh
files/create_multi_metric_noauth.sh
files/server-metrics_10.json
files/server-metrics_11.json
files/create_multi_metric.sh
files/server-metrics_1.json
files/README.md
files/server-metrics_2.json
files/server-metrics_12.json
files/server-metrics_13.json
files/server-metrics_3.json
files/._upload_server_metrics_noauth.sh
files/upload_server_metrics_noauth.sh
files/server-metrics_14.json
files/create_single_metric_noauth.sh
files/server-metrics_8.json
files/server-metrics_18.json
files/._upload_server_metrics.sh
files/upload_server_metrics.sh
files/server-metrics_4.json
files/server-metrics_5.json
files/server-metrics_19.json
files/server-metrics_9.json
files/server-metrics_15.json
[root@master machine_leanrning]# ls
files  server_metrics.tar.gz
[root@master machine_leanrning]# cd files/
[root@master files]# ls
create_multi_metric_noauth.sh   create_single_metric.sh  server-metrics_11.json  server-metrics_14.json  server-metrics_17.json  server-metrics_1.json   server-metrics_3.json  server-metrics_6.json  server-metrics_9.json
create_multi_metric.sh          README.md                server-metrics_12.json  server-metrics_15.json  server-metrics_18.json  server-metrics_20.json  server-metrics_4.json  server-metrics_7.json  upload_server_metrics_noauth.sh
create_single_metric_noauth.sh  server-metrics_10.json   server-metrics_13.json  server-metrics_16.json  server-metrics_19.json  server-metrics_2.json   server-metrics_5.json  server-metrics_8.json  upload_server_metrics.sh
[root@master files]# bash upload_server_metrics.sh 
== Script for creating index and uploading data == 
 

== Deleting old index == 

{"error":{"root_cause":[{"type":"index_not_found_exception","reason":"no such index [server-metrics]","resource.type":"index_or_alias","resource.id":"server-metrics","index_uuid":"_na_","index":"server-metrics"}],"type":"index_not_found_exception","reason":"no such index [server-metrics]","resource.type":"index_or_alias","resource.id":"server-metrics","index_uuid":"_na_","index":"server-metrics"},"status":404}
== Creating Index - server-metrics == 

{"error":{"root_cause":[{"type":"mapper_parsing_exception","reason":"Root mapping definition has unsupported parameters:  [metric : {properties={deny={type=long}, total={type=long}, @timestamp={type=date}, response={type=float}, service={type=keyword}, host={type=keyword}, accept={type=long}}}]"}],"type":"mapper_parsing_exception","reason":"Failed to parse mapping [_doc]: Root mapping definition has unsupported parameters:  [metric : {properties={deny={type=long}, total={type=long}, @timestamp={type=date}, response={type=float}, service={type=keyword}, host={type=keyword}, accept={type=long}}}]","caused_by":{"type":"mapper_parsing_exception","reason":"Root mapping definition has unsupported parameters:  [metric : {properties={deny={type=long}, total={type=long}, @timestamp={type=date}, response={type=float}, service={type=keyword}, host={type=keyword}, accept={type=long}}}]"}},"status":400}
== Bulk uploading data to index... 

Server-metrics_1 uploaded
Server-metrics_2 uploaded
Server-metrics_3 uploaded
Server-metrics_4 uploaded
Server-metrics_5 uploaded
Server-metrics_6 uploaded
Server-metrics_7 uploaded
Server-metrics_8 uploaded
Server-metrics_9 uploaded
Server-metrics_10 uploaded
Server-metrics_11 uploaded
Server-metrics_12 uploaded
Server-metrics_13 uploaded
Server-metrics_14 uploaded
Server-metrics_15 uploaded
Server-metrics_16 uploaded
Server-metrics_17 uploaded
Server-metrics_18 uploaded
Server-metrics_19 uploaded
Server-metrics_20 uploaded
 done - output to /dev/null

== Check upload 
health status index          uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   server-metrics nE8SdaN4TTu4D95wvGLIjw   1   1     707995            0       70mb           70mb

 Server-metrics uploaded 

```

创建数据索引
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/76ec81436d223e7439b2e5fc09aadb3f.png)


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1748016d87d07ba6582eab03158d7f20.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b54307fa5e26c019f92359515d23c323.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/02b2ecd3ac89d0064de8803ec68aef72.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1a6a724c4000a6d34eeb1bc803a0fbe3.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9160c5d3b40ea5d370702058882fec99.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9eb5b3a04e5b9b0ff1c59da13b883019.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/05a423f924757b81471a7b01e45313a7.png)
定制化URL
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e6452fb4eb4000037739468fe8d7999d.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8eafe192a9bfb29e3388e970d8a908f3.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/31b454abc9e69a6ea68ee06d027a5198.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a67daf19a0ada2aa6e848c1e064a3f5e.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5a1c02f7f46a539ddaec7615a0e013a1.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3f5ccd592e8fe1ce3c5af756b19cc740.png)
再次创建job
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a2a8af7535342adaef9d2ca70ac5da6a.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/56fc86ff050a7ab2f4c0a2d4347290be.png)
获取一个新的数据结果
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f626d577afdf483eb1df540084d542af.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5e2f01f921b6ea5c2458aca56cd6d1fd.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6862953bac14d2a3a65a15f9265eff30.png)


