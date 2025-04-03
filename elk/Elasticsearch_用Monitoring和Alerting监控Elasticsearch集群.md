

----
## 1. X-Pack Monitoring
● X-Pack 提供了免费集群监控的功能
● 使用 Elasticsearch 监控 Elasticsearch
○ `Xpack.monitoring.collection.interval` 默认设置 10 秒
● 在生产环境中，建议搭建 `dedicated` 集群用于 ES 集群的监控。有以下几个好处

 - ○ 减少负载和数据
 - ○ 当被监控集群出现问题，还能看到监控相关的数据

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d0637d6905b494751e659261599a86c4.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d705619915e68687c82f893ec6c04b51.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7eb4ff8fa37a2ce1966646110725fb21.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/21a5d44aff21dd17f0a3357af1e7af9c.png)
因为我需要钱来购买，因此选择30天试用即可
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/68ba4a214bb6d04ef2aca7bb3f58ede8.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f2af588b3f23bbb88f5702f39e79a8e4.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b61dcf817662d04c3d3241093aa9d5cc.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/926563d0542d227d2a3fc69680824fb8.png)


## 2. 配置 Monitoring
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/78e09544e67b309f7926ab488867fa1b.png)

## 3. Overall
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4aaca8a3dc66bc0d3ba946676aafc301.png)

## 4. Nodes
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/61a6bbb1c6bca7f5291b211ec6375a36.png)

## 5. Watcher for Alerting
● 需要 Gold 账户
● 一个 Watcher 由 5个部分组成
○ `Trigger` – 多久被触发一次（例如：5 分钟触发一次）
○ `Input` – 查询条件（在所有日志索引中查看 “ERROR” 相关）
○ `Condition` –查询是否满足条件（例如：大于 1000 条返回）
○ Actions – 执行相关操作（例如：发送
### 5.1 创建阈值告警
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cc358d69c0834a127895ac096736a879.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/73791dc95ecd8c4bd9de9437c61bb9e5.png)


### 5.2 创建高级监视
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6fa26527ac21acdc36e75bcae627ff09.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f72b0ad940ca73e2be417d14e0d48433.png)

