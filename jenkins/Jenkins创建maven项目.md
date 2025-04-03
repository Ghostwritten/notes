


----
##  1. maven简介
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3d05e26feb88cc2f166531403fa83e18.png#pic_center)

[https://mirrors.tuna.tsinghua.edu.cn/apache/maven/](https://mirrors.tuna.tsinghua.edu.cn/apache/maven/)

##  2. 安装mvn
```c
# java -version
java version "1.8.0_211"
Java(TM) SE Runtime Environment (build 1.8.0_211-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.211-b12, mixed mode)
# cd /data/
# wget https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.1/binaries/apache-maven-3.6.1-bin.tar.gz
# tar xvf apache-maven-3.6.1-bin.tar.gz
# mv apache-maven-3.6.1 /usr/local/
# cd /usr/local/
# ln -s /usr/local/apache-maven-3.6.1/ /usr/local/maven
# /usr/local/maven/bin/mvn -v
Apache Maven 3.6.1 (d66c9c0b3152b2e69ee9bac180bb8fcc8e6af555; 2019-04-05T03:00:29+08:00)
Maven home: /usr/local/maven
Java version: 1.8.0_211, vendor: Oracle Corporation, runtime: /usr/java/jdk1.8.0_211-amd64/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-862.el7.x86_64", arch: "amd64", family: "unix"
```
配置maven
编辑`/etc/profile`文件，在末尾添加参数，将`maven`命令加入系统环境变量

```c
# vim /etc/profile
export PATH=/usr/local/maven/bin/:$PATH
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/043af5e67651367595d75e5711a6bcb8.png#pic_center)

```c
# source /etc/profile
# mvn -v
Apache Maven 3.6.1 (d66c9c0b3152b2e69ee9bac180bb8fcc8e6af555; 2019-04-05T03:00:29+08:00)
Maven home: /usr/local/maven
Java version: 1.8.0_211, vendor: Oracle Corporation, runtime: /usr/java/jdk1.8.0_211-amd64/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-862.el7.x86_64", arch: "amd64", family: "unix"
```
###认识maven安装目录
安装完成后。Maven的安装目录结构如下：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7b3df2c10ef867829e359159fac932f4.png#pic_center)



 - `bin`：该目录包含了mvn运行的脚本，这些脚本用来配置java命令，准备好`classpath`和相关的java系统属性，然后执行java命令。其中mvn是基于UNIX平台的脚本，`mvn.bat`是基于Windows平台的脚本。
 - `boot`：该目录只包含一个文件，该文件是一个类加载器框架，Maven使用该框架加载自己的类库。
 - `Conf`：该目录包含了一个非常重要的文件`settings.xml`。用于全局定义Maven的行为。也可以将该文件复制到`~/.m2/`目录下，在用户范围内定制Maven行为。
 - `Lib`：该目录包含了所有Maven运行时需要的java类库。

## 3. Jenkins构建一个maven项目
首先在管理员的全局配置里配置maven的路径信息
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ed75cb754bc3801208375b98eb6c21d8.png#pic_center)



添加maven
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e7120da606feb665e18ea0db1dce0212.png#pic_center)


然后在线安装插件Maven Integration
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/208bce0f24750eb013b3563d2730004a.png#pic_center)


安装完成后。在新建项目的时候，就会多一个构建maven项目的选项
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3719ab01950d2f464c48a346fbe3d2dc.png#pic_center)

[https://www.yiibai.com/maven](https://www.yiibai.com/maven)
使用mvn命令构建一个java项目`hello-world`  失败了 测试命令

```c
# mvn archetype:generate -DgroupId=com.companyname.bank  -DartifactId=hello-world  -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

还是网上下载吧
https://down.51cto.com/data/2462845
解压后目录结构如下
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1162436f0a91be3f83e3733b248bc97e.png#pic_center)


测试常用的maven命令  暂时需要联网
`Mvn clean`命令用于清理项目生产的临时文件，一般是模块下的`target`目录
`Mvn test` 命令用于测试，用于执行`src/test/java/`下的测试用例。使用`-Dmaven.test.skip=true`参数可以跳过测试。
生成`target`目录。但是目录里没有生成jar或者war包
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f273cf95f4fe8f4ab61707d5d6fbcf0d.png#pic_center)


Mvn package 命令用于项目打包，会在模块下的target目录生成jar或者war等文件
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f3cb24ac727f87ba2f502f5a68d4f9d1.png#pic_center)
Mvn install命令用于模块安装，将打包好的`jar/war`文件复制到你的本地仓库中，供其他模块使用
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ee124e6b8ba0f8c935ff111e176a80bc.png#pic_center)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c4b1fedff786db6e86183e71176a5f17.png#pic_center)




