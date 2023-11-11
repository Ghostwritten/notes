在gitlab创建一个java的代码仓库项目

![在这里插入图片描述](https://img-blog.csdnimg.cn/6deeb9d46550433c8bf0b0984776481a.png#pic_center)

```c
# pwd
/data/hello-world/hello-world
# git init
# git remote add origin git@192.168.137.100:xmlgrg_test/java.git
# git add .
# git commit -m "Initial commit v0.0.1"
# git push -u origin master
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/439d3ec04c154234abe0cc2a2084e2c1.png#pic_center)

修改上面创建的hello-word，job项目
![在这里插入图片描述](https://img-blog.csdnimg.cn/511c273dda864070908a8bd1a4a155cf.png#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/df3db66df02c49ca881a68feb12441c7.png#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/5cc99c54e49c4f5baa3aaf3531d11f49.png#pic_center)

源码管理配置从git仓库获取helloword仓库的代码
配置要执行的maven命令、保存配置、返回到job主界面。
![在这里插入图片描述](https://img-blog.csdnimg.cn/9eb5a17c7b49496aa5f188292a733063.png#pic_center)
执行构建
成功后、在工作空间，可以看到构建后的产物
![在这里插入图片描述](https://img-blog.csdnimg.cn/c07b8cfa8582491cbd7022e684d7b544.png#pic_center)



fatal: 远程 origin 已经存在。
此时只需要将远程配置删除，重新添加即可；

```c
# git remote rm origin
```

