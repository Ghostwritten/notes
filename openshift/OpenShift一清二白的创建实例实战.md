

##  1. 概述
---
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/221e747a55d895dc569bd7224446cf88.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/880ca50a109114b78b66d1dd415b37df.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/79ed03edf9f84957b7d01da4d6d772f5.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c358994b2d3c1fe2c26880d62533321e.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f94a57a7b50457898d5faa82fc8fafef.png)



架构组成：

 - pod
 - master node
 - worker node
 - controller
 - services
 - replication controller
 - pv & pvc
 - secret & configmap
 - **deployment config**（dc）
 - **build config**（bc）
 - **routes**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/71731f7e8a8aafdb75951bb070ac1eec.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/09dfacf78f8d7b008c4705e4949e3daf.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/96884236f6c416d295c579512fd389bc.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/092966de1332547e4e1e2a5242203d1d.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a166a4b8599468815e8dd5e41ee2bf04.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ff61737554ef1c58f0165527e04a2dae.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e824c3d39c04515862b4fc82c1dabd16.png)


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5f97bce6f2648fdb8fc52543093ecd58.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/33ada071f4034a39908d6bf1cbf3cf6e.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f6af6199a34a88711d3234583ae5b6a4.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0865fe3914100611d62e69d916cc8ad7.png)

```bash
oc  describe pod  mysql-openshift-1-jg6bk
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/698c8f06cd0f01ac4d2bab1f8a17c614.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3eb185a6cce95546c62d97807f374e52.png)


##  2. 创建路由
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a0aa9135e8ae59bf96e9d9a396f4c5e5.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0b7466a561961787a82d49e48197f439.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3d2be2d23e5f49e9778b6392c4c2cfec.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2358d564d18f8d1c51de5c418bcfef53.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b2f59d72d3ba1bc9803fead8cd6aa5b0.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f0b3a44faa5e1ce4f3fb1f4efe0f5aa7.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/821529a3fd552a3919fdbadc4e414258.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bffecea2f5bbb2a015a8b6a3b3ca8d9f.png)
##  3. 命令行创建实例application 

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/06ef6f4dc6eb60637659128eb2201ddf.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1e0520a27e78c4c6d24f86c1060d2944.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/51dcdb603e4ced237b48889aebbffc74.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4c5e48eb34d76bf5cd4e323c5ecb199b.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b35bb1c94e56533e9e9f9e9cac559d54.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9569ab71a5d1151350d077f705b527ba.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a62b55f930b97b8c5fdd2038a9345df1.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c331e771fcce3e848ba6733d936b4731.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/825e07af981106367abcb3de61a70651.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/70d08d5bb49308f6f87266a18addf8d7.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b9af4714ad48a4d0db6c70e9784e11e9.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/247c048ef9411515931258135d8a52a8.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9453fcd73ab22ccefe097020325a0f1e.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/79b72b779017a62074d37219cb958a18.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2b59c9630178e14e76b835c5498275ca.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4c0d8c61d1ab192dc1c5422277faf7db.png)

命令总结：
```bash
oc status
oc new-app -i php:5.6 --name php-helloworld https://services.lab.example.com/php-helloworld
oc get services
oc expose svc/php-helloworld
oc get route
curl php-helloworld-s2i.apps.cluster.lab.example.com
oc clone http://service.lab.example.com/php-helloworld
cd php-helloworld/
vim index.php
git add .
oc commit -am "add a print something!"
git push
oc start-build php-helloworld
oc get pods -w
curl php-helloworld-s2i.apps.cluster.lab.example.com
<内容更新>
```

## 4. 界面创建实例applitcation


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/26e638b205a482a7270bf7d5bbf14822.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a4e93867a60acda47fdd81c2464e1406.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3e8533b407a526850e9925e24378b595.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cb2aa049f9f30f4abdad4e4cbb5e7d7d.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/519f66899cd15836e197499f534418de.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6bb056425022e860a07f5583ca5b15d8.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2a841e1e068a67cfe17a5abaf695f955.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3bea8447d9b4ef3b48d45c662353529d.png)

```bash
oc clone http://service.lab.example.com/php-helloworld
cd php-helloworld/
vim index.php
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/376d50230a59332b9ece513523c43c92.png)

```bash
oc commit -am "add a print something!"
git push
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f185269b3b0529e44fdff44983288ca7.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/550658c6e298f848cc976e9d76368512.png)

结束。
