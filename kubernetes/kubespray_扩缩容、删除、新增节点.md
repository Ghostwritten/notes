
## 1. 扩容节点
配置 inventory.ini

```bash
cat /inventory/inventory.ini 
[all]
kube-master01 ansible_host=192.168.10.41 
kube-master02 ansible_host=192.168.10.42
kube-master03 ansible_host=192.168.10.43 
kube-node01 ansible_host=192.168.10.44
kube-node02 ansible_host=192.168.10.45
kube-node03 ansible_host=192.168.10.46
kube-node04 ansible_host=192.168.10.47
kube-node05 ansible_host=192.168.10.48  #添加


[bastion]
bastion ansible_host=192.168.10.40 ansible_user=root

[kube_control_plane]
kube-master01
kube-master02
kube-master03

[etcd]
kube-master01
kube-master02
kube-master03

[kube_node]
kube-node01
kube-node02
kube-node03
kube-node04
kube-node05  #添加

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr

```
执行：
```bash
ansible-playbook -i /inventory/inventory.ini --private-key /root/.ssh/id_rsa scale.yml
```

##  2. 缩容节点

设置节点不可调度

```bash
kubectl drain  kube-node05 --ignore-daemonsets
kubectl drain kube-node05  --ignore-daemonsets --delete-local-data
```
查看

```bash
$ kubectl get node kube-node05
NAME          STATUS                     ROLES           AGE    VERSION
kube-node05   Ready,SchedulingDisabled   <none>          181d   v1.27.5

$ kubectl get pod -A -owide |grep kube-node05
```


```bash
ansible-playbook -i /inventory/inventory.ini --private-key /root/.ssh/id_rsa -e "node=kube-node05" remove-node.yml
```

