

----
## 1. 日志的重要性
● 为什么重要

 - ○ 运维：医生给病人看病。日志就是病人对自己的陈述
 - ○ 恶意攻击，恶意注册，刷单，恶意密码猜测

● 挑战

 - ○ 关注点很多，任何一个点都有可能引起问题
 - ○ 日志分散在很多机器，出了问题时，才发现日志被删了
 - ○ 很多运维人员是消防员，哪里有问题去哪里

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210322105400341.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 2. 集中化日志管理
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210322105445940.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

## 3. **Filebeat 简介**
● 简介

 - ○ A log data shipper for local files
 - ○ 读取日志文件，Filebeat 不做数据的解析，加工处理

■ 日志是非结构化数据
■ 需要进行处理后，以结构化的方式保存到 Elasticsearch

 - ○ 保证数据至少被读取一次
 - ○ 处理多行数据，解析 JSON 格式 ，简单的过滤
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210322105559877.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 4. Filebeat 执行流程
● 定义数据采集：`Prospector` 配置。通过 `filebeat.yml`
● 建立数据模型： `Index Template`
● 建立数据处理流程：`Ingest Pipeline`
● 存储并提供可视化分析：ES + Kibana Dashboar
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210322105951467.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)



## 5. Modules 开箱即用
● 大量开箱即用的日志模块

 - ○ 简化使用流程
 - ○ 减少开发的投入
 - ○ 最佳参考实践

● 一些命令

 - ○ ./filebeat modules list
 - ○ ./Filebeat modules enable nginx

## 6. demo

```bash
[root@master filebeat-7.3.1-linux-x86_64]# tar -zxvf filebeat-7.3.1-linux-x86_64.tar.gz
[root@master filebeat-7.3.1-linux-x86_64]# cd filebeat-7.3.1-linux-x86_64/
[root@master filebeat-7.3.1-linux-x86_64]# ./filebeat modules list
Enabled:

Disabled:
apache
auditd
cisco
coredns
elasticsearch
envoyproxy
googlecloud
haproxy
icinga
iis
iptables
kafka
kibana
logstash
mongodb
mssql
mysql
nats
netflow
nginx
osquery
panw
postgresql
rabbitmq
redis
santa
suricata
system
traefik
zeek

[root@master filebeat-7.3.1-linux-x86_64]# ./filebeat modules enable nginx
Enabled nginx
[root@master filebeat-7.3.1-linux-x86_64]# ./filebeat modules enable system
Enabled system
[root@master filebeat-7.3.1-linux-x86_64]# ./filebeat modules enable elasticsearch
Enabled elasticsearch
[root@master filebeat-7.3.1-linux-x86_64]# ./filebeat modules list
Enabled:
elasticsearch
nginx
system

```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210322111521417.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210322111737369.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
[root@master filebeat-7.3.1-linux-x86_64]#  ./filebeat setup --dashboards
Loading dashboards (Kibana must be running and reachable)
Loaded dashboards

[root@master filebeat-7.3.1-linux-x86_64]# ./filebeat -e

```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210322112957670.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
查看具体值，显示高亮，设置输出日志类型
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210322113351327.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

