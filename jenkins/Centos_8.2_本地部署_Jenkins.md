
![](https://img-blog.csdnimg.cn/8d6f241cb564459ca229f5d11cb98c83.png)




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

![](https://img-blog.csdnimg.cn/d7e6759e6ef0472bbcb039c10609d999.png)
获取自动生成的密码

```bash
$ sudo  cat /var/lib/jenkins/secrets/initialAdminPassword
92b6f311ba9b433e894b5242cd4ab23c
```
安装插件
![](https://img-blog.csdnimg.cn/94d64e765765419785e921f80264f328.png)
![](https://img-blog.csdnimg.cn/8ef285f2f7f147e0962dc6917ce5eb13.png)
设置管理用户
![](https://img-blog.csdnimg.cn/0aa999f5f3c0418794872e1ffb933d0d.png)
![](https://img-blog.csdnimg.cn/1b0ce9ab2b6a4ae8ab0ec3cb52f163cd.png)
![](https://img-blog.csdnimg.cn/30cfa80d0a94444fb9e9ab29a56d2333.png)
界面
![](https://img-blog.csdnimg.cn/d8c66c6bd987491a9802478ae599b7ee.png)
用户状态
![](https://img-blog.csdnimg.cn/6549d4b3d9d84c70ab34ac82740cb211.png)
![](https://img-blog.csdnimg.cn/ff5cea026e7d463a85c673971f823b97.png)

## 8. 安装插件
### 8.1 kubernets 插件
![](https://img-blog.csdnimg.cn/cf897ce761e04d90b5c17ed94b83e76f.png)

### 8.2 git 插件
![](https://img-blog.csdnimg.cn/4ca9617580bb493a9a0f17c225044cd5.png)
### 8.3 docker 插件
![](https://img-blog.csdnimg.cn/54b3379ecab64049a5e718e8f4b83413.png)
安装完插件进行重启
![](https://img-blog.csdnimg.cn/5a5294e913064cc7b7c3df0f7a071e20.png)

## 9. 创建 pipeline job

![](https://img-blog.csdnimg.cn/a0d00df432ef41f9a25cf6158e479313.png)
![](https://img-blog.csdnimg.cn/5388258204d7480185d69c5fe17c4349.png)
### 9.1 加载本地 Jenkinsfile 构建
如果SCM 选择 `None`
![](https://img-blog.csdnimg.cn/a3779bbf6ead47bdaf529f979efc8ffa.png)
保存后，点击`build`构建。
console output：
![](https://img-blog.csdnimg.cn/9d344eb3156744c89e0333f762e8098c.png)

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
![](https://img-blog.csdnimg.cn/4f02463b9876443ebfda3b2a6ba99914.png)
### 9.2 git 构建
仓库地址：[https://github.com/Ghostwritten/jenkins-example-private-repo.git](https://github.com/Ghostwritten/jenkins-example-private-repo.git)
![](https://img-blog.csdnimg.cn/b62fb0a451fc4c3fb96bdd362b20f1ca.png)

![](https://img-blog.csdnimg.cn/43ed02554fae42cb93046122d7fbb6d2.png)
获取 `github Token`
![](https://img-blog.csdnimg.cn/5bedc51b442d405b9551740a369d5a05.png)
加载`Credentials`，选择好分支，确认 `Script path`，保存。
![](https://img-blog.csdnimg.cn/0bd0b6467bf14d2c82af708ac4f94495.png)

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
![](https://img-blog.csdnimg.cn/918b92aa3dba408e9143eb87e598d84b.png)
尝试 `systemctl restart jenkins`

参考：
- [Java requirements](https://www.jenkins.io/doc/administration/requirements/java/)
- [Linux install Jenkins](https://www.jenkins.io/doc/book/installing/linux/)
- [How to Add Git Credentials in Jenkins](https://www.youtube.com/watch?v=HSA_mZoADSw)
