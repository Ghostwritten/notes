阿里云

```bash
helm repo add ali-incubator     https://aliacs-app-catalog.oss-cn-hangzhou.aliyuncs.com/charts-incubator/  
helm repo add ali-stable    https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts  
```
Git Pages 镜像

```bash
helm repo add stable https://burdenbear.github.io/kube-charts-mirror/
```
官方镜像源

```bash
helm repo add stable https://charts.helm.sh/stable
```
微软库

```bash
helm repo add stable http://mirror.azure.cn/kubernetes/charts/
```

google仓库

```bash
helm repo add stable https://kubernetes-charts-incubator.storage.googleapis.com/
```

微软仓库

```bash
helm repo add stable http://mirror.azure.cn/kubernetes/charts/
helm repo add incubator http://mirror.azure.cn/kubernetes/charts-incubator/
```

#更新仓库

```bash
helm repo update
```

查看库

```bash
helm repo list
helm search repo stable
```
删除库
删除存储库：

```bash
helm repo remove aliyu
```

