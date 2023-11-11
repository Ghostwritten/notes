# couchbase-exporter 容器安装部署
tags: exporters
<!-- catalog: ~couchbase-exporter~ -->

![在这里插入图片描述](https://img-blog.csdnimg.cn/bc529ae6d81649918d61a9eae812c28b.png)




## 1. 部署

```bash
docker run -d -ti -p 9420:9420 -e COUCHBASE_HOST=128.196.0.100 -e COUCHBASE_PORT=8091 totvslabs/couchbase-exporter  
 --couchbase.username=root --couchbase.password=222222 --couchbase.url='128.196.0.98:8091'
```
参数说明

```bash
Name Description Default value
COUCHBASE_HOST    Couchbase host address    127.0.0.1
COUCHBASE_PORT    Couchbase port address    8091
COUCHBASE_USERNAME Couchbase username       xxx
COUCHBASE_PASSWORD Couchbase password       xxx
PROMETHEUS_PORT  Prometheus port to listen   9420
```

