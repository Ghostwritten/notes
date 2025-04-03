![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/fd5dadf9eda64cec87638992f36e386f.jpeg#pic_center)



# 准备
```bash
$ kubectl get node
NAME            STATUS   ROLES           AGE   VERSION
kube-master01   Ready    control-plane   19d   v1.29.5
kube-node01     Ready    <none>          19d   v1.29.5
kube-node02     Ready    <none>          19d   v1.29.5
kube-node03     Ready    <none>          19d   v1.29.5
kube-node04     Ready    <none>          19d   v1.29.5

$ kubectl get pod -n cert-manager
NAME                                     READY   STATUS    RESTARTS      AGE
cert-manager-6fd987499c-6mkkx            1/1     Running   1 (19h ago)   19d
cert-manager-cainjector-5b94bd6f-8phtm   1/1     Running   2 (19h ago)   19d
cert-manager-webhook-575479ff47-7zbzq    1/1     Running   2 (19h ago)   19d
```

# 安装

```bash
helm install rancher rancher-latest/rancher  \
    --version=v2.9.1 \
    --namespace cattle-system \
    --create-namespace \
    --set hostname=rancher.ghostwritten.com \
    --set certmanager.version=1.15.2 \
    --set useBundledSystemChart=true \
    --set bootstrapPassword=admin
```


# 查看

```bash
for i in `kubectl get ns |grep cattle | awk '{print $1}'` ;do kubectl get pod -n $i ;done
```
或者
```bash
$ kubectl get pod -A
NAMESPACE                         NAME                                                         READY   STATUS      RESTARTS         AGE
cattle-fleet-local-system         fleet-agent-0                                                2/2     Running     0                5m21s
cattle-fleet-system               fleet-controller-7b9c4cc46b-nt6zp                            3/3     Running     0                5m52s
cattle-fleet-system               gitjob-7fd56d7b75-4vhk7                                      1/1     Running     0                5m52s
cattle-provisioning-capi-system   capi-controller-manager-5967c7487f-mfshc                     1/1     Running     0                4m5s
cattle-system                     helm-operation-mcrcl                                         0/2     Completed   0                3m36s
cattle-system                     helm-operation-szb99                                         0/2     Completed   0                6m22s
cattle-system                     helm-operation-vt242                                         0/2     Completed   0                6m58s
cattle-system                     helm-operation-xl5z7                                         0/2     Completed   0                5m19s
cattle-system                     helm-operation-xv797                                         0/2     Completed   0                4m10s
cattle-system                     rancher-689b74bc98-9cfw2                                     1/1     Running     0                11m
cattle-system                     rancher-689b74bc98-v98tj                                     1/1     Running     0                11m
cattle-system                     rancher-689b74bc98-zxp6z                                     1/1     Running     0                11m
cattle-system                     rancher-webhook-698d85dfcc-qfn4d                             1/1     Running     0                4m45s
```



# 下载



下载 charts

```bash
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
helm repo update
helm fetch rancher-latest/rancher --version=v2.9.1
```
下载部署镜像，下载可能用到的镜像，请参考[这里](https://github.com/rancher/rancher/releases/tag/v2.9.1)。

```bash
for i in `kubectl get ns |grep cattle | awk '{print $1}'` ;do kubectl get pod -n $i -oyaml |grep 'image:' | awk '{print $2}' | sort -r | uniq;done 
rancher/fleet-agent:v0.10.1
rancher/fleet:v0.10.1
rancher/mirrored-cluster-api-controller:v1.7.3
rancher/shell:v0.2.1
rancher/rancher-webhook:v0.5.1
rancher/rancher:v2.9.1
rancher/rancher-agent:v2.9.1
rancher/kubectl:v1.29.2
```


注册的集群需要拉取哪些镜像

```bash
$ for i in `kubectl get ns |grep cattle | awk '{print $1}'` ;do kubectl get pod -n $i -oyaml |grep image: | awk '{print $2}' | sort -r | uniq;done
rancher/fleet-agent:v0.10.1
rancher/shell:v0.2.1
rancher/rancher-webhook:v0.5.1
rancher/rancher-agent:v2.9.1
```

