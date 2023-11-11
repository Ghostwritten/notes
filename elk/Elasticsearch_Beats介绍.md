


------
## 1. 什么是 Beats
● Light weight data shippers
○ 以搜集数据为主
○ 支持与 Logstash 或 ES 集成
● 全品类 / 轻量级 / 开箱即用 / 可插拔
/ 可扩展 / 可视化
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210317152957563.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 2. Metricbeat
● 用来定期搜集操作系统，软件的指标数据
○ Metric v.s Logs
■ Metric – 可聚合的数据，定期搜集
■ Logs 文本数据，随机搜集
● 指标存储在 Elasticsearch 中，可以通过 Kibana 进行实时的数据分析

下载地址：
[https://www.elastic.co/cn/downloads/beats/metricbeat](https://www.elastic.co/cn/downloads/beats/metricbeat)

### 2.1 Metricbeat 组成
● Module
○ 搜集的指标对象，例如不同的操作系统，不同的数据库，不同的应用系统
● Metricset
○ 一个 Module可以有多个 metricset
○ 具体的指标集合。以减少调用次数为原则进行划分
■ 不同的 metricset 可以设置不同的抓取时长


### 2.2 Module
● Metricbeat 提供了大量的开箱即用的 Module
○ [https://www.elastic.co/guide/en/beats/metricbeat/7.1/index.html](https://www.elastic.co/guide/en/beats/metricbeat/7.1/index.html)
● 通过执行 metricbeat module list 查看
● 通过执行 metricbeat moudle enable module_name 定制


### 2.3 metricbeat实战
```bash
[root@master metricbeat-7.3.1-linux-x86_64]# ./metricbeat  modules list
Enabled:
system

Disabled:
aerospike
apache
aws
beat
beat-xpack
ceph
cockroachdb
consul
coredns
couchbase
couchdb
docker
dropwizard
elasticsearch
elasticsearch-xpack
envoyproxy
etcd
golang
graphite
haproxy
http
jolokia
kafka
kibana
kibana-xpack
kubernetes
kvm
logstash
logstash-xpack
memcached
mongodb
mssql
munin
mysql
nats
nginx
oracle
php_fpm
postgresql
prometheus
rabbitmq
redis
traefik
uwsgi
vsphere
windows
zookeeper

[root@master metricbeat-7.3.1-linux-x86_64]# ./metricbeat modules enable mysql
Enabled mysql
[root@master metricbeat-7.3.1-linux-x86_64]# ./metricbeat modules enable kibana
Enabled kibana
[root@master metricbeat-7.3.1-linux-x86_64]# ./metricbeat modules enable elasticsearch
Enabled elasticsearch
[root@master metricbeat-7.3.1-linux-x86_64]# ./metricbeat  modules list
Enabled:
elasticsearch
kibana
mysql--
system

```
运行metricbeats
```bash
[root@master metricbeat-7.3.1-linux-x86_64]# ./metricbeat setup --dashboards
Loading dashboards (Kibana must be running and reachable)
Loaded dashboards
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210318161826797.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210318161859670.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
打开这个面板。发现没有数据，因为`metricbeats`没有运行
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210318161943204.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
配置mysql面板

```bash
[root@master metricbeat-7.3.1-linux-x86_64]# vim modules.d/mysql.yml
  username: root
  password: 123456
```

运行metricbeats

```bash
[root@master metricbeat-7.3.1-linux-x86_64]# ./metricbeat 
```

运行一个测试app对mysql进行读写插入等操作


```bash
# 安装mysql
create database db_example
use db_example;
show tables;
select * from user

curl localhost:8080/demo/add -d name=Mike -d email=mike@xyz.com -d tags=Elasticsearch,IntelliJ
curl localhost:8080/demo/add -d name=Jack -d email=jack@xyz.com -d tags=Mysql,IntelliJ
curl localhost:8080/demo/add -d name=Bob -d email=bob@xyz.com -d tags=Mysql,IntelliJ

curl 'localhost:8080/demo/all'
```
效果图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210318162807496.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)



### 2.4 Metricsets
● 每个 Module 都有自己的 metricsets，以 System Module 为例
○ core
○ CPU
○ disk IO
○ filesystem
○ load
○ memory

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210318162942633.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)



### 2.5 Metricbeat Event

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210317153856839.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

## 3. Packetbeat
下载地址：[https://www.elastic.co/cn/downloads/beats/packetbeat](https://www.elastic.co/cn/downloads/beats/packetbeat)
● Packetbeat - 实时网络数据分析，监控应用服务器之间的网络流量
○ 常见抓包工具 - Tcpdump /wireshark
○ 常见抓包配置 - Pcap 基于 libpcap，跨平台 / Af_packet 仅支持 Linux，基于内存映射嗅探，高性
能
● Packetbeat 支持的协议
○ ICMP / DHCP / DNS / HTTP / Cassandra / Mysql / PostgresSQL / Redis / MongoDB / Memcache /
TLS
● Network flows：抓取记录网络流量数据，不涉及协议解析

### 3.1 packetbeat实战
```bash
#确保mysql监控打开，一般默认都是开的 
[root@master packetbeat-7.11.2-linux-x86_64]# vim packetbeat.yml
  ports: [80, 8080, 8000, 5000, 8002]

- type: memcache
  # Configure the ports where to listen for memcache traffic. You can disable
  # the Memcache protocol by commenting out the list of ports.
  ports: [11211]

- type: mysql
  # Configure the ports where to listen for MySQL traffic. You can disable
  # the MySQL protocol by commenting out the list of ports.
  ports: [3306,3307]

- type: pgsql
  # Configure the ports where to listen for Pgsql traffic. You can disable
  # the Pgsql protocol by commenting out the list of ports.
  ports: [5432]

- type: redis
  # Configure the ports where to listen for Redis traffic. You can disable
  # the Redis protocol by commenting out the list of ports.
  ports: [6379]

- type: thrift
  # Configure the ports where to listen for Thrift-RPC traffic. You can disable
  # the Thrift-RPC protocol by commenting out the list of ports.
  ports: [9090]

- type: mongodb
  # Configure the ports where to listen for MongoDB traffic. You can disable
  # the MongoDB protocol by commenting out the list of ports.
  ports: [27017]

- type: nfs
  # Configure the ports where to listen for NFS traffic. You can disable
  # the NFS protocol by commenting out the list of ports.
  ports: [2049]

- type: tls
  # Configure the ports where to listen for TLS traffic. You can disable
  # the TLS protocol by commenting out the list of ports.
  ports:
    - 443   # HTTPS
    - 993   # IMAPS
    - 995   # POP3S
    - 5223  # XMPP over SSL
    - 8443
    - 8883  # Secure MQTT
    - 9243  # Elasticsearch

- type: sip
  # Configure the ports where to listen for SIP traffic. You can disable the SIP protocol by commenting out the list of ports.
  ports: [5060]

[root@master packetbeat-7.11.2-linux-x86_64]# ./packetbeat setup --dashboards
Loading dashboards (Kibana must be running and reachable)
Loaded dashboards
[root@master packetbeat-7.11.2-linux-x86_64]# ./packetbeat 
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210318163849492.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
查看一些监控数据
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021031816393065.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

