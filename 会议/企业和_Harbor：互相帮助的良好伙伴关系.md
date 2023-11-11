## 简介
Ye Liu 和 Ludwig Lim（惠普公司）- Harbor 是一种安全、非常稳定和成熟的开源镜像注册表。惠普研发 IT 部门在部署 Harbor 时，将 Harbor 作为惠普公司的一个 docker 镜像注册表来运行；Ye 和 Ludwig 与 Harbor 团队和社区合作，使 Harbor 应用于惠普的工作。他们想谈一谈企业和 Harbor 如何进行合作，从而互惠互利。本次讨论分为两个小节，1) 企业如何帮助 Harbor 发展；2) 回馈：将 Harbor 调整为企业级的内部 Docker 注册表 -Ye 和 Ludwig 想分享他们在 Harbor 发展工作中的经验，并通过分享他们所学到的使 Harbor 适应企业生态系统的知识来回馈 Harbor 社区。

![在这里插入图片描述](https://img-blog.csdnimg.cn/08ae8e2b56ac44d4a5af62f89e2ed118.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/6ec163337fdf4eb785858421fb28dd53.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/c07c7c2f73534a38af151797bf98a928.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/5cf44dbfc76f4eaba52a60a0804d327b.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/cd9ba8dbe22e4148bbc1bca404941979.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2204ad19f1dc4aacabfb4ae2b04a2ddf.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/9dbe7d1b1dfd4c8b8f37c0d15352afdc.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/5adb49662fc84e66aa8b81fc4f5653ce.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/6c9eb8079a2040f893228c6ac02997e8.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/58b643875b634d36818a5a32f525ab6f.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/70308b6e435b4b76a1cab429bb1d1b50.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/c98b1d6e371c4e5b87815cc37f1bd457.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/c8546d664b0e429fb74a340c8212df85.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/0bcd3503a661476291274f1377cd0270.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/727b8f47ce5f4fbca435f297dd259bd3.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/16396033be184bd399d24f9d5e41a2ae.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b54784223488457ba0dca0592031932b.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)

