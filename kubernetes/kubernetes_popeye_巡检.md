



## 1. 简介
Popeye是一个实用程序，可以扫描实时Kubernetes集群，并报告部署的资源和配置的潜在问题。它根据部署的内容而不是磁盘上的内容来清理集群。通过扫描您的集群，它可以检测错误配置，并帮助您确保最佳实践到位，从而防止未来的麻烦。它旨在减少在野外操作Kubernetes集群时面临的认知过载。此外，如果您的集群采用了metric-server，它会报告潜在的资源分配过多/不足，并在集群容量不足时尝试警告您。

- [https://github.com/derailed/popeye.git](https://github.com/derailed/popeye.git)


## 2. 安装

下载地址：[https://github.com/derailed/popeye/releases/tag/v0.11.1](https://github.com/derailed/popeye/releases/tag/v0.11.1)

```bash
wget https://github.com/derailed/popeye/releases/download/v0.11.1/popeye_Linux_x86_64.tar.gz
mkdir popeye 
mv popeye_Linux_x86_64.tar.gz popeye
cd popeye 
tar zxvf popeye_Linux_x86_64.tar.gz
cp popeye /usr/local/bin/
```


## 3.  本地

```bash
POPEYE_REPORT_DIR=$(pwd) popeye --save
或者

POPEYE_REPORT_DIR=$(pwd) ./popeye --save --out html --output-file report.html
```


## 4. 容器
你不需要构建和/或安装二进制文件来运行popeye：你可以直接从DockerHub上的官方docker repo运行它。运行docker容器时的默认命令是popeye，所以你只需要将通常传递给popeye的cli参数传递给popeye。要访问集群，请使用-v将本地kube config目录映射到容器中

```bash
  docker run --rm -it \
    -v $HOME/.kube:/root/.kube \
    derailed/popeye --context foo -n bar
```
使用--rm运行上面的docker命令意味着当popeye退出时容器会被删除。当你使用--保存时，它会将其写入容器中的/tmp，然后在popeye退出时删除容器，这意味着你会丢失输出。要解决这个问题，请将/tmp映射到容器的/tmp。注意：您可以通过设置POPEYE_REPORT_DIR env变量来覆盖默认的输出目录位置。

```bash
  docker run --rm -it \
    -v $HOME/.kube:/root/.kube \
    -e POPEYE_REPORT_DIR=/tmp/popeye \
    -v /tmp:/tmp \
    derailed/popeye --context foo -n bar --save --output-file my_report.txt

  # Docker has exited, and the container has been deleted, but the file
  # is in your /tmp directory because you mapped it into the container
  $ cat /tmp/popeye/my_report.txt
    <snip>
```

