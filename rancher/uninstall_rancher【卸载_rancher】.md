

```bash
yum -y install git
git clone https://github.com/rancher/rancher-cleanup.git
```

```bash
cd rancher-cleanup

kubectl create -f deploy/rancher-cleanup.yaml

kubectl  -n kube-system logs -l job-name=cleanup-job  -f

kubectl create -f deploy/verify.yaml

kubectl  -n kube-system logs -l job-name=verify-job  -f

kubectl  -n kube-system logs -l job-name=verify-job  -f | grep -v "is deprecated"
```

> 注意：加入是离线私有仓库，需要推送镜像至私有仓库，并修改yaml仓库地址：

```bash
sed -i 's#rancher\/#192.168.128.156\/rancher\/#g' deploy/*.yaml
```

参考：

- https://github.com/rancher/rancher-cleanup.git
