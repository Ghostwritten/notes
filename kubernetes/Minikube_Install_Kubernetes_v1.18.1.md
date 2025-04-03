![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/3858533580a643e2bec67e1375db2158.jpeg#pic_center)



# 简介

模拟客户环境，测试 kubernetes v1.18.x 是否可以被 rancher v2.9.1 纳管。

# 安装工具

- [docker 安装](https://blog.csdn.net/xixihahalelehehe/article/details/104293170)
- [Install and Set Up kubectl on Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
- - [安装 minikube](https://minikube.sigs.k8s.io/docs/start/?arch=/linux/x86-64/stable/binary%20download)

# 配置代理

- [docker proxy](https://blog.csdn.net/xixihahalelehehe/article/details/134115905)
- [linux proxy](https://blog.csdn.net/xixihahalelehehe/article/details/141957969)


# 运行集群

```bash
$ minikube start  --driver=none --container-runtime=docker  --kubernetes-version=v1.18.1
```
输出：
```bash
😄  minikube v1.18.1 on Rocky 8.10
🆕  Kubernetes 1.20.2 is now available. If you would like to upgrade, specify: --kubernetes-version=v1.20.2
✨  Using the none driver based on existing profile
👍  Starting control plane node minikube in cluster minikube
🔄  Restarting existing none bare metal machine for "minikube" ...
ℹ️  OS release is Rocky Linux 8.10 (Green Obsidian)
🌐  Found network options:
    ▪ HTTP_PROXY=http://192.168.21.101:7890
    ▪ HTTPS_PROXY=http://192.168.21.101:7890
    ▪ NO_PROXY=proxyhost,localhost,*.vsphere.local,*.vm.demo,*.tanzu.demo,192.168.21.101,127.0.0.1/8,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16
    ▪ http_proxy=http://192.168.21.101:7890
    ▪ https_proxy=http://192.168.21.101:7890
    ▪ no_proxy=proxyhost,localhost,*.vsphere.local,*.vm.demo,*.tanzu.demo,192.168.21.101,127.0.0.1/8,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16
🐳  Preparing Kubernetes v1.18.1 on Docker 19.03.15 ...
    ▪ env HTTP_PROXY=http://192.168.21.101:7890
    ▪ env HTTPS_PROXY=http://192.168.21.101:7890
    ▪ env NO_PROXY=proxyhost,localhost,*.vsphere.local,*.vm.demo,*.tanzu.demo,192.168.21.101,127.0.0.1/8,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16

🤹  Configuring local host environment ...

❗  The 'none' driver is designed for experts who need to integrate with an existing VM
💡  Most users should use the newer 'docker' driver instead, which does not require root!
📘  For more information, see: https://minikube.sigs.k8s.io/docs/reference/drivers/none/

❗  kubectl and minikube configuration will be stored in /root
❗  To use kubectl or minikube commands as your own user, you may need to relocate them. For example, to overwrite your own settings, run:

    ▪ sudo mv /root/.kube /root/.minikube $HOME
    ▪ sudo chown -R $USER $HOME/.kube $HOME/.minikube

💡  This can also be done automatically by setting the env var CHANGE_MINIKUBE_NONE_USER=true
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v4
🌟  Enabled addons: storage-provisioner, default-storageclass
❗  The cluster minikube already exists which means the --nodes parameter will be ignored. Use "minikube node add" to add nodes to an existing cluster.
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

# 检查集群

```bash
$ kubectl get node
NAME         STATUS   ROLES    AGE    VERSION
minikube01   Ready    <none>   178m   v1.18.1
$ kubectl get pod -A
NAMESPACE     NAME                                 READY   STATUS    RESTARTS   AGE
kube-system   coredns-66bff467f8-rszjx             1/1     Running   0          176m
kube-system   etcd-minikube01                      1/1     Running   1          177m
kube-system   kube-apiserver-minikube01            1/1     Running   0          177m
kube-system   kube-controller-manager-minikube01   1/1     Running   0          177m
kube-system   kube-proxy-nthcz                     1/1     Running   0          176m
kube-system   kube-scheduler-minikube01            1/1     Running   0          177m
kube-system   storage-provisioner                  1/1     Running   1          176m
```

# 加入rancher

```bash
curl --insecure -sfL https://rancheruat.demo.com.cn/v3/import/gnplffg5965h27mjv4dwbgpvk8qhhpssj896t8vrdnblljhfpx6dhc_c-m-9gpswkpp.yaml | kubectl apply -f -
```

```bash
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

$ kubectl edit  sts  fleet-agent  -n cattle-fleet-system
…..
      dnsPolicy: ClusterFirst
      hostAliases:
      - hostnames:
        - rancheruat.demo.com.cn
        ip: 192.168.23.79
      restartPolicy: Always
      schedulerName: default-scheduler
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/7899644fd3cf40bd97a9dd64b0796439.png)

