
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/eaebfd5e4f914a3ea7bd34f4a3f5260d.png)





# 背景

harbor 的镜像同步至备份harbor 或者同步quay.io。实现镜像备份。

- [centos 7.9 部署 harbor 镜像仓库实践](https://ghostwritten.blog.csdn.net/article/details/127920005)


# Add Harbor Endpoint

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/7670ff322993428fb4119a311a084a8c.png)


![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/7bf77cc73e9f4f99a29714a115d39226.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/4c4cacd44dc04ec892d4a38f321e95f5.png)

# Add Quay.io Endpoint
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b85cf199b16f4333a737526557365fe4.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/8f334170803c4b069fbb3d8621197d4d.png)


## add registry Endpoint

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/f4e7fd1e8eee4489b0196468d28aa9eb.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/5691d2e5bcbd455382af5cb63a1fca25.png)

# Harbor New Replication Rule

## Harbor Manual Push Images To Quay.io

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/46961d022aa24b60aa8315f53b1747fd.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/12f75d9aa3ee4dbab646fdf475f42ded.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/56aab05cdbab43acb241c8f142e438d4.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/562fb6b126f442e7a1ed4813ffb9238b.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/624dd76ece5c4ce0badec10e50bea2c2.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/eded435721d743b7bff8b148f13363e1.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/ef577470b13a413fbc00ff87f583a3ac.png)

## Harbor Scheduled Push Images To Quay.io
每5小时同步一次。

0 0 */5 * * *

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/fc9d0686a7fc42cd933529f0685b738f.png)


![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/8dc455b7638b4f6f902e69a1c5b15c78.png)


参考：

- [https://goharbor.io/docs/1.10/administration/configuring-replication/create-replication-rules/](https://goharbor.io/docs/1.10/administration/configuring-replication/create-replication-rules/)
- [https://goharbor.io/docs/2.3.0/administration/configuring-replication/create-replication-endpoints/](https://goharbor.io/docs/2.3.0/administration/configuring-replication/create-replication-endpoints/)
