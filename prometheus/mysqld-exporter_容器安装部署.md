# mysqld-exporter 容器安装部署
tags: exporters
<!-- catalog: ~mysqld-exporter~ -->

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/560715f7a1765bde038e74809b17726f.png)





##  1. 部署
数据库创建mysqld_exporter连接用户并配置密码以及赋权。

```bash
create user 'exporter'@'%' identified by '123456' with MAX_USER_CONNECTIONS 3 ;
grant process,replication client,select on *.* to 'exporter'@'%';
```
运行容器

```bash
docker run -d -p 9104:9104  -e DATA_SOURCE_NAME="exporter:123456@(128.196.0.98:3306)/" --name mysqld_exporter prom/mysqld-exporter:latest
```

## 2. prometheus配置mysql-exporter metrics

```bash
scrape_configs:
  # 添加作业并命名
  - job_name: 'mysql'
    # 静态添加node
    static_configs:
    # 指定监控端
    - targets: ['47.98.138.176:9104']
```
## 3. 检查并重启prometheus
检查并重启服务

```bash
$ promtool check config prometheus.yml 
$ curl -X POST http://localhost:9090/-/reloa
```

