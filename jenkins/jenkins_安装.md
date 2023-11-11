​


## linux安装

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.reposudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
yum install jenkins
```

## docker安装
1.下载jenkins镜像

```bash
docker run --name donhui_jenkins -p 8080:8080 -v /var/jenkins_home donhui/jenkins
```



