


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
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319175646909.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
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
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319175748759.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

## 9. ES ML：单指标 / ES ML：单指标 / 种群分析
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319175826469.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

## 10. 单指标任务
● Create New Job
● 选择 Index Pattern，选择 Fare quote
● 选择单指标任务

 - ○ Aggregation ○ Field
 - ○ Bucket Span （取决于用户的数据业务 /数据的采样时间 / 波动情况 / 需要预测的频率）
 - ○ Use full dat

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319175839287.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 11. Demo
● Create New Single Metric Job

 - ○ 基于 Index Pattern 和 Full data set

● Create Customer URL

 - ○ Saved Query

● 对数据实现预测
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319181024785.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
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
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319182918897.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)


![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319182528202.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319183035951.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319183101560.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319183546167.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319183748481.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319183825325.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319184028292.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
定制化URL
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319184143902.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319184341998.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319184400881.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319184523415.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319185014130.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319185115351.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
再次创建job
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319185213398.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319185300592.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
获取一个新的数据结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319185325452.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319185444488.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319185515680.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)


