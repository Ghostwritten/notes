##  运行不是想要的runners执行器

发现一个问题，我的flask_web1注册的runner明明是flask_demo，为什么真正运行的runner却是demo呢。
![在这里插入图片描述](https://img-blog.csdnimg.cn/fccec6ace56c46d9a91e88afbae24cee.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/70b1e46266f84ad29f7a1f6d3d532fb8.png)
我的runners有三个
![在这里插入图片描述](https://img-blog.csdnimg.cn/3e7ff102be0d4a53a0969801d3ada6d8.png)
看看每个demo runners与fask_demo runners有什么区别
![在这里插入图片描述](https://img-blog.csdnimg.cn/f8c3ab47622f4a8596c67fc90dcebcad.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/9f5cc45e3c8c4f068cf87f15211e339e.png)

原来runners如果没有tag标签，就不会锁定当前的项目。所以demo跨项目接手了flask_demo的CI任务。

因此，我们只需把此runners停掉即可。
![在这里插入图片描述](https://img-blog.csdnimg.cn/7bb633c2f85245b589fa3629f93aa69e.png)
##  项目选择出runners
![在这里插入图片描述](https://img-blog.csdnimg.cn/16556c2f198648c6948ff5c9fd9b50ae.png)
我们的runners 注册是正常的
![在这里插入图片描述](https://img-blog.csdnimg.cn/2ff81c4bc51c494d9a53e001c8b8c62c.png)

我们回到flask_demo runners的配置
![在这里插入图片描述](https://img-blog.csdnimg.cn/afbd518b3c484013a54a82050ace1975.png)
v
