在gitlab创建一个java的代码仓库项目

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fe4264f29604d721fbf53d8a5716d577.png#pic_center)

```c
# pwd
/data/hello-world/hello-world
# git init
# git remote add origin git@192.168.137.100:xmlgrg_test/java.git
# git add .
# git commit -m "Initial commit v0.0.1"
# git push -u origin master
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9d6d90754e413459ccd555db9de99189.png#pic_center)

修改上面创建的hello-word，job项目
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5e212b2f3fe64d1f2676e978afc93132.png#pic_center)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5cda97955e330bb65f070abc498b92e0.png#pic_center)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e8b2c155386913d7f3f63a40821a374f.png#pic_center)

源码管理配置从git仓库获取helloword仓库的代码
配置要执行的maven命令、保存配置、返回到job主界面。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d4bc6292f4b108375c841bd82929264a.png#pic_center)
执行构建
成功后、在工作空间，可以看到构建后的产物
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e75286613c8d5fd2ce144c2ca90c1fea.png#pic_center)



fatal: 远程 origin 已经存在。
此时只需要将远程配置删除，重新添加即可；

```c
# git remote rm origin
```

