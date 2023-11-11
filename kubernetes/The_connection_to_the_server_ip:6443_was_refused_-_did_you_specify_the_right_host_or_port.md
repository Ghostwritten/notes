![在这里插入图片描述](https://img-blog.csdnimg.cn/20201214221320212.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

---

## 解决方法1：

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
export $HOME/.kube/config
```
## 解决方法2：

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf HOME/.kube/config sudo chown (id -u):$(id -g) $HOME/.kube/config
Alternatively you can also export KUBECONFIG variable like this:
export KUBECONFIG=$HOME/.kube/config
```
## 解决方法3：
```bash
#!/bin/bash
swapoff -a
systemctl start kubelet
docker start (docker ps -a -q)
docker start (docker ps -a -q)
```
## 解决方法4：

```bash
sudo systemctl status firewalld #redhat centos
sudo systemctl stop firewalld #redhat, centos
sudo ufw status verbose #ubuntu
sudo ufw disable #ubuntu
```

## 解决方法5：

 - master： 192.168.211.40
 - node1： 192.168.211.41
 - node2： 192.168.211.42
### master
```bash
$ kubeadm reset

$ kubeadm init  --pod-network-cidr=172.22.0.0/16 --apiserver-advertise-address=192.168.211.40 --kubernetes-version=v1.20.1
..........
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.211.40:6443 --token s7apx1.mlxn2jkid6n99fr0 \
    --discovery-token-ca-cert-hash sha256:2fa9da39110d02efaf4f8781aa50dd25cce9be524618dc7ab91a53e81c5c22f8 

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
### node1

```bash
$ kubeadm reset
$ kubeadm join 192.168.211.40:6443 --token s7apx1.mlxn2jkid6n99fr0 \
    --discovery-token-ca-cert-hash sha256:2fa9da39110d02efaf4f8781aa50dd25cce9be524618dc7ab91a53e81c5c22f8 
```

### node1

```bash
$ kubeadm reset
$ kubeadm join 192.168.211.40:6443 --token s7apx1.mlxn2jkid6n99fr0 \
    --discovery-token-ca-cert-hash sha256:2fa9da39110d02efaf4f8781aa50dd25cce9be524618dc7ab91a53e81c5c22f8 
```
### master

```bash
$ kubectl get nodes
NAME     STATUS   ROLES    AGE     VERSION
master   Ready    master   5m18s   v1.18.6
node1    Ready    <none>   81s     v1.18.6
node2    Ready    <none>   43s     v1.18.6
$ scp /root/.kube/config root@192.168.211.41:/root/.kube/config
$ scp /root/.kube/config root@192.168.211.42:/root/.kube/config

$ root@master:~# kubectl get pods -A
NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE
kube-system   coredns-66bff467f8-2xmnt         0/1     Pending   0          21m
kube-system   coredns-66bff467f8-ghj2s         0/1     Pending   0          21m
kube-system   etcd-master                      1/1     Running   0          22m
kube-system   kube-apiserver-master            1/1     Running   0          22m
kube-system   kube-controller-manager-master   1/1     Running   0          22m
kube-system   kube-proxy-dh46z                 1/1     Running   0          7m35s
kube-system   kube-proxy-jq6cb                 1/1     Running   0          21m
kube-system   kube-proxy-z6prp                 1/1     Running   0          9m14s
kube-system   kube-scheduler-master            1/1     Running   0          22m

$ kubectl apply -f https://docs.projectcalico.org/v3.10/manifests/calico.yaml
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
daemonset.apps/calico-node created
serviceaccount/calico-node created
deployment.apps/calico-kube-controllers created
serviceaccount/calico-kube-controllers created

```
参考资料：
[https://discuss.kubernetes.io/t/the-connection-to-the-server-host-6443-was-refused-did-you-specify-the-right-host-or-port/552/45](https://discuss.kubernetes.io/t/the-connection-to-the-server-host-6443-was-refused-did-you-specify-the-right-host-or-port/552/45)
