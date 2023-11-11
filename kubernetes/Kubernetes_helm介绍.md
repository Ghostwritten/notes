## 简介
Helm 是 Kubernetes 的包管理器
两个大版本
```bash
v3.5.0
v2.17.0
```

## 安装
v3... [安装Helm](https://helm.sh/zh/docs/intro/quickstart/)
v2... [https://v2.helm.sh/docs/using_helm/#installing-helm](https://v2.helm.sh/docs/using_helm/#installing-helm)

## 配置国内镜像源

```bash
helm repo add stable https://burdenbear.github.io/kube-charts-mirror/
或者
helm repo add stable https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
```
查看helm源添加情况

```bash
helm repo list
helm search repo stable
```
更新

```bash
 helm repo update  
```
哪些chart被发布

```bash
helm ls
```

