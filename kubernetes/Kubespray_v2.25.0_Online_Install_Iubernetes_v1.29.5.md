![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b106261419b84f418cd7d5d3262f44ec.png)



# 基础配置

- [Rocky 8.9 & Kubespray v2.24.1 在线安装 kubernetes v1.28.6 集群](https://blog.csdn.net/xixihahalelehehe/article/details/138581657)

# 运行 kubespray
```bash
docker run -it --net=host  -v "${HOME}"/.ssh/id_rsa:/root/.ssh/id_rsa --name kubespray \
  quay.io/kubespray/kubespray:v2.25.0 bash
```

# 配置 inventory.ini
```bash
$ vim  inventory/sample/inventory.ini
[all]
kube-master01 ansible_host=192.168.23.31 ip=192.168.23.31 etcd_member_name=etcd1
kube-node01 ansible_host=192.168.23.32 ip=192.168.23.32
kube-node02 ansible_host=192.168.23.33 ip=192.168.23.33
kube-node03 ansible_host=192.168.23.34 ip=192.168.23.34
kube-node04 ansible_host=192.168.23.35 ip=192.168.23.35

[kube_control_plane]
kube-master01


[etcd]
kube-master01

[kube_node]
kube-node01
kube-node02
kube-node03
kube-node04

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr
```

# 配置代理
```bash
$ vim inventory/sample/group_vars/all/all.yml
http_proxy: "http://192.168.21.101:7890"
https_proxy: "http://192.168.21.101:7890"
no_proxy: "localhost,127.0.0.0/8,169.0.0.0/8,10.0.0.0/8,192.168.0.0/16,*.coding.net,*.tencentyun.com,*.myqcloud.com"
```

# 安装集群

```bash
ansible-playbook -i inventory/sample/inventory.ini --private-key /root/.ssh/id_rsa cluster.yml
```

# 检查集群

```bash
$ kubectl get node
NAME            STATUS   ROLES           AGE     VERSION
kube-master01   Ready    control-plane   8m21s   v1.29.5
kube-node01     Ready    <none>          7m22s   v1.29.5
kube-node02     Ready    <none>          7m22s   v1.29.5
kube-node03     Ready    <none>          7m21s   v1.29.5
kube-node04     Ready    <none>          7m21s   v1.29.5

$ kubectl get pod -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS       AGE
kube-system   calico-kube-controllers-68485cbf9c-q29gp   1/1     Running   0              5m50s
kube-system   calico-node-5s29t                          1/1     Running   0              6m48s
kube-system   calico-node-694l7                          1/1     Running   0              6m48s
kube-system   calico-node-d54lx                          1/1     Running   0              6m48s
kube-system   calico-node-qdh9p                          1/1     Running   0              6m48s
kube-system   calico-node-zrpz9                          1/1     Running   0              6m48s
kube-system   coredns-69db55dd76-p9h4n                   1/1     Running   0              5m4s
kube-system   coredns-69db55dd76-qskrl                   1/1     Running   0              5m12s
kube-system   dns-autoscaler-6f4b597d8c-f4gsq            1/1     Running   0              5m5s
kube-system   kube-apiserver-kube-master01               1/1     Running   0              8m55s
kube-system   kube-controller-manager-kube-master01      1/1     Running   1              8m55s
kube-system   kube-proxy-28wsx                           1/1     Running   0              7m52s
kube-system   kube-proxy-2vz95                           1/1     Running   0              7m53s
kube-system   kube-proxy-cz27r                           1/1     Running   0              7m53s
kube-system   kube-proxy-jlkjw                           1/1     Running   0              7m53s
kube-system   kube-proxy-lbbfd                           1/1     Running   0              7m52s
kube-system   kube-scheduler-kube-master01               1/1     Running   1              8m56s
kube-system   nginx-proxy-kube-node01                    1/1     Running   0              7m59s
kube-system   nginx-proxy-kube-node02                    1/1     Running   0              7m58s
kube-system   nginx-proxy-kube-node03                    1/1     Running   0              7m57s
kube-system   nginx-proxy-kube-node04                    1/1     Running   0              7m57s
kube-system   nodelocaldns-7b7wx                         1/1     Running   0              5m4s
kube-system   nodelocaldns-l2qh8                         1/1     Running   0              5m4s
kube-system   nodelocaldns-pm82m                         1/1     Running   0              5m4s
kube-system   nodelocaldns-t6t6m                         1/1     Running   1 (5m2s ago)   5m4s
kube-system   nodelocaldns-zmg6j                         1/1     Running   0              5m4s
```

参考：

- [https://github.com/kubernetes-sigs/kubespray](https://github.com/kubernetes-sigs/kubespray)
