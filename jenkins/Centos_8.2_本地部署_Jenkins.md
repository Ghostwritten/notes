
![](https://i-blog.csdnimg.cn/blog_migrate/70819eddac614d89ca32339eab95c29b.png)




关于`Jenkins` 部署上一篇是：[minikube & helm 安装 jenkins](https://blog.csdn.net/xixihahalelehehe/article/details/128166331)
## 1. 简介
[Jenkins](https://www.jenkins.io/doc/book/getting-started/) 是一个 CI/CD 工具。这里CI是指持续集成，CD是指持续交付。Jenkins 也被认为是自动化工具或服务器，它有助于自动化与构建、测试和部署相关的软件开发。


## 2. 准备条件
- 系统：centos 8.2
- cpu：4
- 内存：8G
- 硬盘：40G
- ip: 192.168.10.90
- hostname: `jenkins_master`

## 3. 安装依赖工具
1. 安装依赖
```bash
sudo dnf -y install make autoconf automake cmake perl-CPAN libcurl-devel libtool gcc gcc-c++ glibc-headers zlib-devel git-lfs telnet lrzsz jq expat-devel openssl-devel
```
2. 安装 Git

```bash

cd /tmp
wget --no-check-certificate https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.38.1.tar.gz
tar -xvzf git-2.38.1.tar.gz
cd git-2.38.1/
./configure
make
sudo make install
```

```bash
$ git --version          # 输出 git 版本号，说明安装成功
git version 2.38.1
```

## 4. 配置 jenkins 源 
- [官方 Linux 本地安装 Jenkins](https://www.jenkins.io/doc/book/installing/linux/)

```bash
echo "192.168.10.90 jenkins_master " | sudo tee -a /etc/hosts
```
[dnf 命令](https://blog.csdn.net/xixihahalelehehe/article/details/123168620)软件更新

```bash
sudo dnf update -y
sudo dnf repolist
```
下载 jenkins 源

```bash
sudo dnf install wget -y
sudo wget http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo -O /etc/yum.repos.d/jenkins.repo
```
运行以下 `rpm` 命令以导入 `Jenkins` 包的 GPG 密钥

```bash
sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
```

## 5. 安装 java-17
查询 java 版本

```bash
dnf list java* --showduplicates  | sort -r
Last metadata expiration check: 0:00:38 ago on Tue 06 Dec 2022 12:00:53 PM CST.
javapackages-tools.noarch                    5.3.0-1.module_el8.0.0+11+5b8c10bd appstream
javapackages-filesystem.noarch               5.3.0-1.module_el8.0.0+11+5b8c10bd appstream
java-atk-wrapper.x86_64                      0.33.2-6.el8                       appstream
java-1.8.0-openjdk.x86_64                    1:1.8.0.312.b07-2.el8_5            appstream
java-1.8.0-openjdk.x86_64                    1:1.8.0.312.b07-1.el8_4            appstream
java-1.8.0-openjdk-src.x86_64                1:1.8.0.312.b07-2.el8_5            appstream
java-1.8.0-openjdk-src.x86_64                1:1.8.0.312.b07-1.el8_4            appstream
java-1.8.0-openjdk-slowdebug.x86_64          1:1.8.0.312.b07-2.el8_5            appstream
java-1.8.0-openjdk-slowdebug.x86_64          1:1.8.0.312.b07-1.el8_4            appstream
java-1.8.0-openjdk-javadoc-zip.noarch        1:1.8.0.312.b07-2.el8_5            appstream
java-1.8.0-openjdk-javadoc-zip.noarch        1:1.8.0.312.b07-1.el8_4            appstream
java-1.8.0-openjdk-javadoc.noarch            1:1.8.0.312.b07-2.el8_5            appstream
java-1.8.0-openjdk-javadoc.noarch            1:1.8.0.312.b07-1.el8_4            appstream
java-1.8.0-openjdk-headless.x86_64           1:1.8.0.312.b07-2.el8_5            appstream
java-1.8.0-openjdk-headless.x86_64           1:1.8.0.312.b07-1.el8_4            appstream
java-1.8.0-openjdk-headless-slowdebug.x86_64 1:1.8.0.312.b07-2.el8_5            appstream
java-1.8.0-openjdk-headless-slowdebug.x86_64 1:1.8.0.312.b07-1.el8_4            appstream
java-1.8.0-openjdk-devel.x86_64              1:1.8.0.312.b07-2.el8_5            appstream
java-1.8.0-openjdk-devel.x86_64              1:1.8.0.312.b07-1.el8_4            appstream
java-1.8.0-openjdk-demo.x86_64               1:1.8.0.312.b07-2.el8_5            appstream
java-1.8.0-openjdk-demo.x86_64               1:1.8.0.312.b07-1.el8_4            appstream
java-1.8.0-openjdk-accessibility.x86_64      1:1.8.0.312.b07-2.el8_5            appstream
java-1.8.0-openjdk-accessibility.x86_64      1:1.8.0.312.b07-1.el8_4            appstream
java-17-openjdk.x86_64                       1:17.0.1.0.12-2.el8_5              appstream
java-17-openjdk.x86_64                       1:17.0.0.0.35-4.el8                appstream
java-17-openjdk-static-libs.x86_64           1:17.0.1.0.12-2.el8_5              appstream
java-17-openjdk-static-libs.x86_64           1:17.0.0.0.35-4.el8                appstream
java-17-openjdk-src.x86_64                   1:17.0.1.0.12-2.el8_5              appstream
java-17-openjdk-src.x86_64                   1:17.0.0.0.35-4.el8                appstream
java-17-openjdk-jmods.x86_64                 1:17.0.1.0.12-2.el8_5              appstream
java-17-openjdk-jmods.x86_64                 1:17.0.0.0.35-4.el8                appstream
java-17-openjdk-javadoc-zip.x86_64           1:17.0.1.0.12-2.el8_5              appstream
java-17-openjdk-javadoc-zip.x86_64           1:17.0.0.0.35-4.el8                appstream
java-17-openjdk-javadoc.x86_64               1:17.0.1.0.12-2.el8_5              appstream
java-17-openjdk-javadoc.x86_64               1:17.0.0.0.35-4.el8                appstream
java-17-openjdk-headless.x86_64              1:17.0.1.0.12-2.el8_5              appstream
java-17-openjdk-headless.x86_64              1:17.0.0.0.35-4.el8                appstream
java-17-openjdk-devel.x86_64                 1:17.0.1.0.12-2.el8_5              appstream
java-17-openjdk-devel.x86_64                 1:17.0.0.0.35-4.el8                appstream
java-17-openjdk-demo.x86_64                  1:17.0.1.0.12-2.el8_5              appstream
java-17-openjdk-demo.x86_64                  1:17.0.0.0.35-4.el8                appstream
java-11-openjdk.x86_64                       1:11.0.13.0.8-4.el8_5              appstream
java-11-openjdk.x86_64                       1:11.0.13.0.8-3.el8_5              appstream
java-11-openjdk.x86_64                       1:11.0.13.0.8-1.el8_4              appstream
java-11-openjdk-static-libs.x86_64           1:11.0.13.0.8-4.el8_5              appstream
java-11-openjdk-static-libs.x86_64           1:11.0.13.0.8-3.el8_5              appstream
java-11-openjdk-static-libs.x86_64           1:11.0.13.0.8-1.el8_4              appstream
java-11-openjdk-src.x86_64                   1:11.0.13.0.8-4.el8_5              appstream
java-11-openjdk-src.x86_64                   1:11.0.13.0.8-3.el8_5              appstream
java-11-openjdk-src.x86_64                   1:11.0.13.0.8-1.el8_4              appstream
java-11-openjdk-jmods.x86_64                 1:11.0.13.0.8-4.el8_5              appstream
java-11-openjdk-jmods.x86_64                 1:11.0.13.0.8-3.el8_5              appstream
java-11-openjdk-jmods.x86_64                 1:11.0.13.0.8-1.el8_4              appstream
java-11-openjdk-javadoc-zip.x86_64           1:11.0.13.0.8-4.el8_5              appstream
java-11-openjdk-javadoc-zip.x86_64           1:11.0.13.0.8-3.el8_5              appstream
java-11-openjdk-javadoc-zip.x86_64           1:11.0.13.0.8-1.el8_4              appstream
java-11-openjdk-javadoc.x86_64               1:11.0.13.0.8-4.el8_5              appstream
java-11-openjdk-javadoc.x86_64               1:11.0.13.0.8-3.el8_5              appstream
java-11-openjdk-javadoc.x86_64               1:11.0.13.0.8-1.el8_4              appstream
java-11-openjdk-headless.x86_64              1:11.0.13.0.8-4.el8_5              appstream
java-11-openjdk-headless.x86_64              1:11.0.13.0.8-3.el8_5              appstream
java-11-openjdk-headless.x86_64              1:11.0.13.0.8-1.el8_4              appstream
java-11-openjdk-devel.x86_64                 1:11.0.13.0.8-4.el8_5              appstream
java-11-openjdk-devel.x86_64                 1:11.0.13.0.8-3.el8_5              appstream
java-11-openjdk-devel.x86_64                 1:11.0.13.0.8-1.el8_4              appstream
java-11-openjdk-demo.x86_64                  1:11.0.13.0.8-4.el8_5              appstream
java-11-openjdk-demo.x86_64                  1:11.0.13.0.8-3.el8_5              appstream
java-11-openjdk-demo.x86_64                  1:11.0.13.0.8-1.el8_4              appstream
Available Packages
```
自 Jenkins 2.357 和 LTS 2.361.1 以来，Jenkins 需要 Java 11 或 17。

```bash
$ dnf list java-17-openjdk-devel --showduplicates  | sort -r
Last metadata expiration check: 0:08:00 ago on Tue 06 Dec 2022 12:06:28 PM CST.
java-17-openjdk-devel.x86_64           1:17.0.1.0.12-2.el8_5           appstream
java-17-openjdk-devel.x86_64           1:17.0.0.0.35-4.el8             appstream
```

Java 是 Jenkins 的必备条件之一，所以运行下面的 dnf 命令来安装 java

```bash
sudo dnf install -y java-17-openjdk-devel
```
查看 java 版本
```bash
$ java -version
openjdk version "17.0.1" 2021-10-19 LTS
OpenJDK Runtime Environment 21.9 (build 17.0.1+12-LTS)
OpenJDK 64-Bit Server VM 21.9 (build 17.0.1+12-LTS, mixed mode, sharing)
```
设置Java环境

```bash
$ sudo vi /etc/profile.d/java.sh
export JAVA_HOME=$(dirname $(dirname $(readlink $(readlink $(which javac)))))
export PATH=$PATH:$JAVA_HOME/bin
export JRE_HOME=/usr/lib/jvm/jre
export CLASSPATH=.:$JAVA_HOME/jre/lib:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```
生效

```bash
source /etc/profile.d/java.sh
```
检查

```bash
$ echo $JAVA_HOME
/usr/lib/jvm/java-17-openjdk-17.0.1.0.12-2.el8_5.x86_64

$ env |grep java
JAVA_HOME=/usr/lib/jvm/java-17-openjdk-17.0.1.0.12-2.el8_5.x86_64
CLASSPATH=.:/usr/lib/jvm/java-17-openjdk-17.0.1.0.12-2.el8_5.x86_64/jre/lib:/usr/lib/jvm/java-17-openjdk-17.0.1.0.12-2.el8_5.x86_64/lib:/usr/lib/jvm/java-17-openjdk-17.0.1.0.12-2.el8_5.x86_64/lib/tools.jar
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:/usr/lib/jvm/java-17-openjdk-17.0.1.0.12-2.el8_5.x86_64/bin
```
## 6. 安装 Jenkins
查询最新 Jenkins 版本
```bash
$ dnf list jenkins --showduplicates  | sort -r
Last metadata expiration check: 0:16:30 ago on Tue 06 Dec 2022 12:06:28 PM CST.
jenkins.noarch                        2.89.4-1.1                         jenkins
jenkins.noarch                        2.89.3-1.1                         jenkins
jenkins.noarch                        2.89.2-1.1                         jenkins
jenkins.noarch                        2.89.1-1.1                         jenkins
jenkins.noarch                        2.7.4-1.1                          jenkins
jenkins.noarch                        2.73.3-1.1                         jenkins
jenkins.noarch                        2.73.2-1.1                         jenkins
......
```
安装

```bash
sudo dnf -y install jenkins

```
查询默认安装版本

```bash
$ rpm -q jenkins
jenkins-2.375.1-1.1.noarch
```

启动

```bash
sudo systemctl enable jenkins && sudo systemctl start jenkins
```
```bash
$ sudo systemctl status jenkins
● jenkins.service - Jenkins Continuous Integration Server
   Loaded: loaded (/usr/lib/systemd/system/jenkins.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2022-12-06 12:50:29 CST; 12s ago
 Main PID: 2516 (java)
    Tasks: 52 (limit: 49495)
   Memory: 1.7G
   CGroup: /system.slice/jenkins.service
           └─2516 /usr/bin/java -Djava.awt.headless=true -jar /usr/share/java/jenkins.war --webroot=/var/cache/jenkins/>

Dec 06 12:49:53 jenkins_master jenkins[2516]: Jenkins initial setup is required. An admin user has been created and a p>
Dec 06 12:49:53 jenkins_master jenkins[2516]: Please use the following password to proceed to installation:
Dec 06 12:49:53 jenkins_master jenkins[2516]: 92b6f311ba9b433e894b5242cd4ab23c
Dec 06 12:49:53 jenkins_master jenkins[2516]: This may also be found at: /var/lib/jenkins/secrets/initialAdminPassword
Dec 06 12:49:53 jenkins_master jenkins[2516]: *************************************************************
Dec 06 12:50:29 jenkins_master jenkins[2516]: 2022-12-06 04:50:29.241+0000 [id=34]        INFO        jenkins.InitReact>
Dec 06 12:50:29 jenkins_master jenkins[2516]: 2022-12-06 04:50:29.269+0000 [id=25]        INFO        hudson.lifecycle.>
Dec 06 12:50:29 jenkins_master systemd[1]: Started Jenkins Continuous Integration Server.
Dec 06 12:50:30 jenkins_master jenkins[2516]: 2022-12-06 04:50:30.302+0000 [id=53]        INFO        h.m.DownloadServi>
Dec 06 12:50:30 jenkins_master jenkins[2516]: 2022-12-06 04:50:30.303+0000 [id=53]        INFO        hudson.util.Retri>
```
如果 firewalld 启动：

```bash
YOURPORT=8080
PERM="--permanent"
SERV="$PERM --service=jenkins"

firewall-cmd $PERM --new-service=jenkins
firewall-cmd $SERV --set-short="Jenkins ports"
firewall-cmd $SERV --set-description="Jenkins port exceptions"
firewall-cmd $SERV --add-port=$YOURPORT/tcp
firewall-cmd $PERM --add-service=jenkins
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --reload
```
或者关闭firewalld

```bash
systemctl stop firewalld.service
systemctl disable firewalld.service
```

## 7. 登陆
访问：`http://192.168.10.90:8080

![](https://i-blog.csdnimg.cn/blog_migrate/bc52c0cbf7a1ab58b6dc48918850b30a.png)
获取自动生成的密码

```bash
$ sudo  cat /var/lib/jenkins/secrets/initialAdminPassword
92b6f311ba9b433e894b5242cd4ab23c
```
安装插件
![](https://i-blog.csdnimg.cn/blog_migrate/760a3f6ded2a2e925483154f8e8e0fb3.png)
![](https://i-blog.csdnimg.cn/blog_migrate/02ae589992af64959ef264fd3d6ce5d9.png)
设置管理用户
![](https://i-blog.csdnimg.cn/blog_migrate/f1d26ed0849be2487bbde0dbda49ce01.png)
![](https://i-blog.csdnimg.cn/blog_migrate/71d00779eebd22d23b329f1f968b6efe.png)
![](https://i-blog.csdnimg.cn/blog_migrate/26a45b7e53a423dc39a760ea685fbb50.png)
界面
![](https://i-blog.csdnimg.cn/blog_migrate/9d8cacfdc2daa0bde0f8002fdfbca736.png)
用户状态
![](https://i-blog.csdnimg.cn/blog_migrate/33528cbef675fbf2b34e4ef9b952c677.png)
![](https://i-blog.csdnimg.cn/blog_migrate/d8ddfe48a549f8714ce227addd0933a4.png)

## 8. 安装插件
### 8.1 kubernets 插件
![](https://i-blog.csdnimg.cn/blog_migrate/73698fecb62bd4d5e5c294fadd88bfd5.png)

### 8.2 git 插件
![](https://i-blog.csdnimg.cn/blog_migrate/d63839ad7246fc124ce8c73909bd0477.png)
### 8.3 docker 插件
![](https://i-blog.csdnimg.cn/blog_migrate/a3ef06065107f337e80e75c85e86cbb0.png)
安装完插件进行重启
![](https://i-blog.csdnimg.cn/blog_migrate/f0db71e65e0ca6e94096469db07c2231.png)

## 9. 创建 pipeline job

![](https://i-blog.csdnimg.cn/blog_migrate/66f0dd9c4ccce4375ab1be8391a7866c.png)
![](https://i-blog.csdnimg.cn/blog_migrate/0e1d8649d52677bf23e17de5c051b167.png)
### 9.1 加载本地 Jenkinsfile 构建
如果SCM 选择 `None`
![](https://i-blog.csdnimg.cn/blog_migrate/04871737409cb2ac3914e65f541f5a14.png)
保存后，点击`build`构建。
console output：
![](https://i-blog.csdnimg.cn/blog_migrate/3abd0946623008c2840c27cfd103baf2.png)

```bash
/var/lib/jenkins/workspace/hello@script/bc6d5aaac091397cf6a4e48610337eabdad7c6ec8ff38cf8699e4e8a0aaaa1c8/Jenkinsfile
```
我们尝试创建该文件，并编写一个`jenkinsfile`

```bash
$ vim /var/lib/jenkins/workspace/hello@script/bc6d5aaac091397cf6a4e48610337eabdad7c6ec8ff38cf8699e4e8a0aaaa1c8/Jenkinsfile
```

```bash
pipeline {
  agent any
  stages {
    stage('hello') {
      steps {
        sh 'echo "Hello World"'
      }
    }
  }
}
```
保存后再次构建
![](https://i-blog.csdnimg.cn/blog_migrate/d6dd973bd7f35982e765add0a946ad49.png)
### 9.2 git 构建
仓库地址：[https://github.com/Ghostwritten/jenkins-example-private-repo.git](https://github.com/Ghostwritten/jenkins-example-private-repo.git)
![](https://i-blog.csdnimg.cn/blog_migrate/bc4654a3981a4e4271e298a6613e8fd3.png)

![](https://i-blog.csdnimg.cn/blog_migrate/a2d3e4ef9ae2fb4f9d78a7e5c316747b.png)
获取 `github Token`
![](https://i-blog.csdnimg.cn/blog_migrate/42bdf36337c22b41394b6578a41613cb.png)
加载`Credentials`，选择好分支，确认 `Script path`，保存。
![](https://i-blog.csdnimg.cn/blog_migrate/2a715f01f8ef8371cb8c3285030f3ce8.png)

点击构建，查看`console output`：

```bash
Started by user ghostwritten
Obtained Jenkinsfile from git https://github.com/Ghostwritten/jenkins-example-private-repo.git
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins in /var/lib/jenkins/workspace/hello
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Declarative: Checkout SCM)
[Pipeline] checkout
Selected Git installation does not exist. Using Default
The recommended git tool is: NONE
using credential github-token
Cloning the remote Git repository
Cloning repository https://github.com/Ghostwritten/jenkins-example-private-repo.git
 > git init /var/lib/jenkins/workspace/hello # timeout=10
Fetching upstream changes from https://github.com/Ghostwritten/jenkins-example-private-repo.git
 > git --version # timeout=10
 > git --version # 'git version 2.38.1'
using GIT_ASKPASS to set credentials github-token
 > git fetch --tags --force --progress -- https://github.com/Ghostwritten/jenkins-example-private-repo.git +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git config remote.origin.url https://github.com/Ghostwritten/jenkins-example-private-repo.git # timeout=10
 > git config --add remote.origin.fetch +refs/heads/*:refs/remotes/origin/* # timeout=10
Avoid second fetch
 > git rev-parse refs/remotes/origin/main^{commit} # timeout=10
Checking out Revision edd6932bd314bd2ad81d7e8bc6a312f18a93431b (refs/remotes/origin/main)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f edd6932bd314bd2ad81d7e8bc6a312f18a93431b # timeout=10
Commit message: "add link"
First time build. Skipping changelog.
[Pipeline] }
[Pipeline] // stage
[Pipeline] withEnv
[Pipeline] {
[Pipeline] stage
[Pipeline] { (hello)
[Pipeline] sh
+ echo 'Hello World'
Hello World
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS
```

## 10. 问题
加载下载插件失败：
![](https://i-blog.csdnimg.cn/blog_migrate/71eb13fadd77a7ce8eecb27099631886.png)
尝试 `systemctl restart jenkins`

参考：
- [Java requirements](https://www.jenkins.io/doc/administration/requirements/java/)
- [Linux install Jenkins](https://www.jenkins.io/doc/book/installing/linux/)
- [How to Add Git Credentials in Jenkins](https://www.youtube.com/watch?v=HSA_mZoADSw)
