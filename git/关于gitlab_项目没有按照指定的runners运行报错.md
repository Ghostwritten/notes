##  运行不是想要的runners执行器

发现一个问题，我的flask_web1注册的runner明明是flask_demo，为什么真正运行的runner却是demo呢。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7d38170bce23026b96e312f1db6d86ae.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/df6732fd49fee29a7185f798e08da711.png)
我的runners有三个
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dbacb3e94d3c003a7e2ff1769dfdd028.png)
看看每个demo runners与fask_demo runners有什么区别
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1a5020fc1df3e6c9d1bfd428eab76c12.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/34d57bce8508035d65e2be5e3b241446.png)

原来runners如果没有tag标签，就不会锁定当前的项目。所以demo跨项目接手了flask_demo的CI任务。

因此，我们只需把此runners停掉即可。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8f364d41b15429d40bc512bec87f98e3.png)
##  项目选择出runners
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/729fb381d95a980d21d6e2423fae3a9c.png)
我们的runners 注册是正常的
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0240257847e5ac4547a11e391993accd.png)

我们回到flask_demo runners的配置
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/abdc312eb06689cdddaeef15535f5b45.png)
v
