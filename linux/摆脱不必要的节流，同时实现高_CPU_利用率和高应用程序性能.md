CPU Burst：摆脱不必要的节流，同时实现高 CPU 利用率和高应用程序性能 | CPU Burst: Getting Rid of Unnecessary Throttling, Achieving High CPU Utilization and Application Performance at the Same Time - Huaixin Chang & Tianchen Ding, Alibaba

![在这里插入图片描述](https://img-blog.csdnimg.cn/c6d7d5eb0f724e578497f28f3c59a524.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)


长期以来，CPU 节流一直是一个令人头疼的问题。即使当 pods 的 CPU 利用率远低于其 CPU 限制时，仍然存在许多长尾情况。因此，开发人员很难选择合适的硬限制。到目前为止，对于这一问题的答案始终是增加 CPU 硬限制或关闭 CPU 硬限制。然而，这一方法带来了一些新问题，如潜在的 难应付的性能影响、低 CPU 利用率和高 TCO（总拥有成本）。最近，融入到 Linux 5.14 中的 CPU Burst 的特性，成为了彻底解决不必要的 CPU 节流问题的一个新选择。注意到 CPU 节流是由 100 毫秒级的突发 CPU 使用引起的后，CPU Burst 的特性可允许平均 CPU 利用率低于 CPU 限制情况下可能的突发使用。应用 CPU Burst 后，用户可以同时获得高 CPU 利用率和高应用程序性能。在本会话中，我们将介绍执行 CPU 限制的内核机制，CPU Burst 更改的内容及其影响，以及如何评估此种更改。在本会话结束时，您可确切地了解到是否在您的 pods 上使用 CPU Burst。

![在这里插入图片描述](https://img-blog.csdnimg.cn/77e17ad78df24edb84cf28322aae9c27.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/86e83e761c2a44e38b7ee28c0d0625e5.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/8ce20c4d97084ae29d04bf109b1742f9.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/654045cf0556435d9b80838ffc70e62a.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/45126defb30b4d2fbafeb433ec94bcbc.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/4e4ee1dd28554de4ac8798b08121482f.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/584d43a83b024754b2c41868605c7ffb.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/38c75bdfaab645fd9f71b00e40e66ecb.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/875d3bfeaffa41baa6bab7e3126bbe66.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/0526da690197422f9713c84b0f28097c.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/99165be070e94926990cf4eb6593b15e.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/8a3c70c8c9d4402d9315db62060127fa.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/ee86380b2d2740dbaddd7b8a52024980.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/f197b92d71c94307b956b7bad54e4b92.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/912db93f5677431f8778b181c69e8ac8.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/c5812c3de0d04907a95d7b4723dce8e0.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/1684d4e9f7204ae1824fcde3519d6bad.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/ec2bdd1ee1ce4ccca820ce38201df804.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)



![在这里插入图片描述](https://img-blog.csdnimg.cn/0402de6aa46a48379b7f5ebff56f4c74.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/484d099b97a141d3b05e8f55431839c4.png?shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)

