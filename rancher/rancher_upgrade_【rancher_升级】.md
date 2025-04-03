![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/ed3355ec38a348a4bdc516cc65e761e3.jpeg#pic_center)



# 1. 背景

rancher v2.8.2 升级 v2.9.1



# 2. 下载

下载charts
```bash
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
helm repo update
helm fetch rancher-latest/rancher --version=v2.9.1
```


下载镜像

- 请参考：[Helm Deploy Online Rancher v2.9.1](https://blog.csdn.net/xixihahalelehehe/article/details/141889301)


# 3. 安装

```bash
helm upgrade rancher ./rancher-2.9.1.tgz \
    --namespace cattle-system \
    --set hostname=rancheruat.demo.com.cn \
    --set rancherImage=harbor.bsgchina.com/rancher \
    --set ingress.tls.source=secret \
    --set privateCA=true \
    --set systemDefaultRegistry=harbor.bsgchina.com \
    --set useBundledSystemChart=true
```


# 4. 检查
pod状态

```bash
$ for i in `kubectl get ns |grep cattle | awk '{print $1}'` ;do kubectl get pod -n $i ;done
No resources found in cattle-fleet-clusters-system namespace.
NAME                          READY   STATUS    RESTARTS   AGE
fleet-agent-0                 2/2     Running   0          99m
fleet-agent-fd575fbd5-xzl9l   1/1     Running   0          19h
NAME                                READY   STATUS    RESTARTS   AGE
fleet-controller-56648b754c-kj9gz   3/3     Running   0          99m
gitjob-5c499c5c7f-nskm5             1/1     Running   0          99m
No resources found in cattle-global-data namespace.
No resources found in cattle-global-nt namespace.
No resources found in cattle-impersonation-system namespace.
NAME                                       READY   STATUS      RESTARTS   AGE
capi-controller-manager-5f5f4fff9d-g48qk   1/1     Running     0          101m
rancher-provisioning-capi-patch-sa-9fsk8   0/1     Completed   0          107s
NAME                              READY   STATUS    RESTARTS   AGE
rancher-backup-787bd4b98b-dwff8   1/1     Running   0          5h11m
NAME                               READY   STATUS    RESTARTS       AGE
rancher-779dff5dc9-bzrb8           1/1     Running   0              103m
rancher-779dff5dc9-f42mt           1/1     Running   0              118m
rancher-779dff5dc9-jsl5p           1/1     Running   1 (104m ago)   118m
rancher-webhook-749d6bd65d-jmqdt   1/1     Running   0              100m
No resources found in cattle-ui-plugin-system namespace.
```



**登陆界面,检查未升级之前创建的用户与导入的集群相关数据都在。确认数据没有丢失。**

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/49447669a1894437866b61ae78d6fe4f.png)

查询注册的集群cluster01 的 pod 状态。

```bash
$ kubectl get pod -A
NAMESPACE             NAME                                                    READY   STATUS      RESTARTS      AGE
cattle-fleet-system   fleet-agent-0                                           0/2     Init:0/1    0             11m
cattle-fleet-system   fleet-agent-56f48d899f-8wpw9                            1/1     Running     0             101m
cattle-system         cattle-cluster-agent-855f9ffddf-nhrls                   1/1     Running     0             101m
cattle-system         helm-operation-8sdhg                                    0/2     Completed   0             30m
cattle-system         rancher-webhook-7b44dcb98-hhlp4                         1/1     Running     0             30m
```

查询镜像版本,并未所有相关注册镜像的pod 更新。

```bash
$ for i in `kubectl get ns |grep cattle | awk '{print $1}'` ;do kubectl get pod -n $i -oyaml |grep image: | awk '{print $2}' | sort -r | uniq;done
image:
harbor.bsgchina.com/rancher/fleet-agent:v0.9.0
harbor.bsgchina.com/rancher/fleet-agent:v0.10.1
harbor.bsgchina.com/rancher/shell:v0.2.1
harbor.bsgchina.com/rancher/rancher-webhook:v0.5.1
harbor.bsgchina.com/rancher/rancher-agent:v2.9.1
```

# 5. 测试

## 5.1 创建项目
创建test2
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/78166f29f9604a2a924b62cd4f70c3e4.png)



## 5.2 创建应用

但并未影响在注册的集群创建应用。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a1958dfeba604770960e0a4684e3456b.png)

## 5.3 删除集群

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/5d385c06589748e9b6101acaa93d3c72.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/6aaa47cdb18744dda86a834963a3b328.png)


```bash
$ kubectl get pod -A
NAMESPACE     NAME                                                    READY   STATUS      RESTARTS      AGE
default       nginx                                                   1/1     Running     0             17h
kube-system   cloud-controller-manager-rke-master01                   1/1     Running     3 (37h ago)   218d
kube-system   etcd-rke-master01                                       1/1     Running     4             218d
kube-system   helm-install-rke2-canal-wvpzk                           0/1     Completed   0             218d
kube-system   helm-install-rke2-coredns-t6mqj                         0/1     Completed   0             218d
kube-system   helm-install-rke2-ingress-nginx-pcqjv                   0/1     Completed   0             218d
kube-system   helm-install-rke2-metrics-server-wnnmr                  0/1     Completed   0             218d
kube-system   helm-install-rke2-snapshot-controller-crd-kfxdx         0/1     Completed   0             218d
kube-system   helm-install-rke2-snapshot-controller-qgz6b             0/1     Completed   1             218d
kube-system   helm-install-rke2-snapshot-validation-webhook-kw97t     0/1     Completed   0             218d
kube-system   kube-apiserver-rke-master01                             1/1     Running     1             218d
kube-system   kube-controller-manager-rke-master01                    1/1     Running     3 (37h ago)   218d
kube-system   kube-proxy-rke-master01                                 1/1     Running     2 (37h ago)   37h
kube-system   kube-scheduler-rke-master01                             1/1     Running     1 (37h ago)   218d
kube-system   rke2-canal-47k9j                                        2/2     Running     2 (37h ago)   218d
kube-system   rke2-coredns-rke2-coredns-565dfc7d75-8b2qk              1/1     Running     1 (37h ago)   218d
kube-system   rke2-coredns-rke2-coredns-autoscaler-6c48c95bf9-xzv5w   1/1     Running     1 (37h ago)   218d
kube-system   rke2-ingress-nginx-controller-ljx7t                     1/1     Running     1 (37h ago)   218d
kube-system   rke2-metrics-server-c9c78bd66-zslfq                     1/1     Running     1 (37h ago)   218d
kube-system   rke2-snapshot-controller-6f7bbb497d-x8d8t               1/1     Running     1 (37h ago)   218d
kube-system   rke2-snapshot-validation-webhook-65b5675d5c-v9mlx       1/1     Running     1 (37h ago)   218d
```

## 5.4 注册集群


![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/548f6e959ec54adab1ca6e1a5a138e93.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/158f724ee56144e0bb701feeb974b223.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/46ad27342baa4a1c851cf226c516810f.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/80d8785ab0084ea6bc604af095ecad80.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/55ff0f64be9c437b9ef9a0902e2e06e3.png)

```bash
$ curl --insecure -sfL https://rancheruat.demo.com.cn/v3/import/lppzthqddjmx427dhd8ggdv6ddfc2gzrrg9r2rf7bk66hjlkmg22m5_c-m-8m97ngwv.yaml | kubectl apply -f -
$ kubectl edit deploy cattle-cluster-agent -n cattle-system
…..
      dnsPolicy: ClusterFirst
      hostAliases:
      - hostnames:
        - rancheruat.demo.com.cn
        ip: 192.168.23.79
      restartPolicy: Always
      schedulerName: default-scheduler
…..
```

集群重新加入成功。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/c06b5972f9ab491988d4dc58e40fe0be.png)

