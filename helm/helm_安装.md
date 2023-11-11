![](https://img-blog.csdnimg.cn/e97409df002549e7bdf2566cea58692b.png)



官方安装：

 -  [https://helm.sh/docs/intro/install/](https://helm.sh/docs/intro/install/)


## 1. 一键安装


```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```
## 2. 二进制指定版本安装
helm 的每个[版本](https://github.com/helm/helm/releases)都为各种操作系统提供二进制版本

```bash
$ wget https://get.helm.sh/helm-v3.11.0-linux-amd64.tar.gz
$ tar -xzvf helm-v3.11.0-linux-amd64.tar.gz
$ cp linux-amd64/helm /usr/local/bin/
$ helm version
version.BuildInfo{Version:"v3.11.0", GitCommit:"d14138609b01886f544b2025f5000351c9eb092e", GitTreeState:"clean", GoVersion:"go1.17.5"}
```


## 3. helm & kubernetes 版本
- [https://helm.sh/docs/topics/version_skew/](https://helm.sh/docs/topics/version_skew/)

| Helm Version | Supported Kubernetes Versions |
|--------------|-------------------------------|
| 3.12.x       | 1.27.x - 1.24.x               |
| 3.11.x       | 1.26.x - 1.23.x               |
| 3.10.x       | 1.25.x - 1.22.x               |
| 3.9.x        | 1.24.x - 1.21.x               |
| 3.8.x        | 1.23.x - 1.20.x               |
| 3.7.x        | 1.22.x - 1.19.x               |
| 3.6.x        | 1.21.x - 1.18.x               |
| 3.5.x        | 1.20.x - 1.17.x               |
| 3.4.x        | 1.19.x - 1.16.x               |
| 3.3.x        | 1.18.x - 1.15.x               |
| 3.2.x        | 1.18.x - 1.15.x               |
| 3.1.x        | 1.17.x - 1.14.x               |
| 3.0.x        | 1.16.x - 1.13.x               |
| 2.16.x       | 1.16.x - 1.15.x               |
| 2.15.x       | 1.15.x - 1.14.x               |
| 2.14.x       | 1.14.x - 1.13.x               |
| 2.13.x       | 1.13.x - 1.12.x               |
| 2.12.x       | 1.12.x - 1.11.x               |
| 2.11.x       | 1.11.x - 1.10.x               |
| 2.10.x       | 1.10.x - 1.9.x                |
| 2.9.x        | 1.10.x - 1.9.x                |
| 2.8.x        | 1.9.x - 1.8.x                 |
| 2.7.x        | 1.8.x - 1.7.x                 |
| 2.6.x        | 1.7.x - 1.6.x                 |
| 2.5.x        | 1.6.x - 1.5.x                 |
| 2.4.x        | 1.6.x - 1.5.x                 |
| 2.3.x        | 1.5.x - 1.4.x                 |
| 2.2.x        | 1.5.x - 1.4.x                 |
| 2.1.x        | 1.5.x - 1.4.x                 |
| 2.0.x        | 1.4.x - 1.3.x                 |


