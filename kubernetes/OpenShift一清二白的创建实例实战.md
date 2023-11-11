

##  1. 概述
---
![在这里插入图片描述](https://img-blog.csdnimg.cn/d367fb6c323c4155a53bce19ee89b56b.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/69d95260e11242c98ed563556e14d6d5.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/9f21eaa896cb4cde8ae0f6b0bfaa75ce.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/c0d9bd67993649f1af065b4a3e0552de.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/63e7a7c2a16e4f48b3b24988aa471236.png)



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

![在这里插入图片描述](https://img-blog.csdnimg.cn/2a416b3c064d4d2ba7a7e4139fcd7da0.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/c093342c13154342a14a833e41da1ea3.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/eafd85550fe14ff180e030e466a801c5.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/428ec89ee91c495bacc5561b1befb3d7.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/52c35b9b0b9f4333aa1f9dfc3ae47f6c.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/db9f07d02727404399100aceffdd018b.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/4e3cf27491d3477a943bae9f46af898a.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/06ad29989aa5476e8ed36316aabc89d6.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/82958f1b9e614612924742615c12ea5d.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/4d51d9413d1a40009b643d4dd92ac54c.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/a9715e24d6034e68b9aeed7b5474efb6.png)

```bash
oc  describe pod  mysql-openshift-1-jg6bk
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/0855e4a0819a4bd58882d1f5cccfcd55.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/c2d7db8112a7464caeda8fae6a1527f8.png)


##  2. 创建路由
![在这里插入图片描述](https://img-blog.csdnimg.cn/a724fd925c3a44d78278b16e2382d372.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/ea9ae8b164c44d5281545da99a567270.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/0b5866b9ad5e4f01a95d3fa7931be168.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/1f4b55d3fd0c4f538e9f11b8929c94b3.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/91bd62dc9c724cc0ab14bddd032d765d.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/ed02e250c8ae4be48fa931d994e1cee4.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/87a537b421bf44c5a97be620646b2ced.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/72706b7527af459cb44cef82f7d84a70.png)
##  3. 命令行创建实例application 

![在这里插入图片描述](https://img-blog.csdnimg.cn/181200f4423b4d7ea434a9d304d93070.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/da6bdf1b5bd54a059b56f69a5caaee7a.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/a9127c4e4454484189dad63c9b45f6ae.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/6adc080599414d76b07bb03f3d337334.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/3c10204edca54a4c8aabe5d4251ebe14.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2af06ef359eb468e8ef400f0d02395a4.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/84b06917d165432f9b4109d581740e77.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/1e402fe731be4834807e629a95b9e66d.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/232d40e3f5b74dfd97de5ced1a22980f.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/677397877163466b9b903a25d38a9f1b.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/f283bea815484ad399158dd1c84a9617.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/dc2f6e5b26614d69805c66dc01abb476.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/7ec19ae990244e00aded2c8aa20be482.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/a09c8a1934484288bdaa663e850cc6b3.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/74de598ca33a4637a7b9a9c349b39328.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/8561323985d142848cc5a4ddc071a358.png)

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


![在这里插入图片描述](https://img-blog.csdnimg.cn/bebe0da48e5b4fa1a4afe1889afd607a.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/4aa5d5be9db84c8bafeb72828eaac18f.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/eeda9be0860d40ea9182228a5b18a85d.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/a61e08f8a6994275a0334987d887b05b.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/9888b36fc5174a158b9fa643311e5012.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/bb42701efbc844cbb828249aad3c8635.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/dbcb28fd5d074eea86107acd527b11d9.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/3fb9b9ad70ad41a6abfe60c97efe3923.png)

```bash
oc clone http://service.lab.example.com/php-helloworld
cd php-helloworld/
vim index.php
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/5974d9e76e5c47dfacfd319ed0cd4557.png)

```bash
oc commit -am "add a print something!"
git push
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/db9c599f6c0c4691902c59c1b3257db5.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/ac3adfa0d05e4e3db470ff2829b99959.png)

结束。
