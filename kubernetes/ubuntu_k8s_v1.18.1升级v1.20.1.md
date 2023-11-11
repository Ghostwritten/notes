组件版本支持查询
[https://kubernetes.io/docs/setup/release/notes/](https://kubernetes.io/docs/setup/release/notes/)
查看版本

```bash
apt-cache madison kubelet
apt-cache madison kubeadm
apt-cache madison kubectl
```

升级v1.20.0-00版本

```bash
apt-get update && apt-get install -y --allow-change-held-packages kubeadm=1.20.1-00
apt-get install -y --allow-change-held-packages kubectl=1.20.1-00
apt-get install -y --allow-change-held-packages kubelet=1.20.1-00
```
显示镜像版本列表

```bash
root@master:~# kubeadm config images list
k8s.gcr.io/kube-apiserver:v1.20.1
k8s.gcr.io/kube-controller-manager:v1.20.1
k8s.gcr.io/kube-scheduler:v1.20.1
k8s.gcr.io/kube-proxy:v1.20.1
k8s.gcr.io/pause:3.2
k8s.gcr.io/etcd:3.4.13-0
k8s.gcr.io/coredns:1.7.0
```
生成默认kubeadm.conf文件

```bash
kubeadm config print init-defaults > kubeadm.conf
```
绕过墙下载镜像方法

```bash
sed -i "s/imageRepository: .*/imageRepository: registry.aliyuncs.com\/google_containers/g" kubeadm.conf
```
下载需要用到的镜像

```bash
kubeadm config images pull --config kubeadm.conf
```

> 注意： 要停止所有kube-system的pod，否则下载镜像会自动删除

master初始化
```bash
kubeadm init --kubernetes-version=v1.20.1 --pod-network-cidr=172.22.0.0/16 --apiserver-advertise-address=192.168.211.40
kdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
scp /root/.kube/config root@192.168.211.41:/root/.kube/config
scp /root/.kube/config root@192.168.211.42:/root/.kube/config
```
node1，node2初始化

```bash
kubeadm reset
kubeadm join 192.168.211.40:6443 --token blduo0.676tzd2jndviqpeq \
>     --discovery-token-ca-cert-hash sha256:8012b7c0a3d9fd13d5263ab97df3d49eff46afb1c10de413cc7322c8f9e00247 
```
配置calico网络

```bash
kubectl apply -f https://docs.projectcalico.org/v3.16/manifests/calico.yaml
```

```bash
$ kubectl get nodes
NAME     STATUS   ROLES                  AGE    VERSION
master   Ready    control-plane,master   158m   v1.20.1
node1    Ready    <none>                 153m   v1.20.1
node2    Ready    <none>                 153m   v1.20.1

kubectl get pods -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-57fc9c76cc-cknk9   1/1     Running   0          97m
kube-system   calico-node-4b49f                          1/1     Running   0          97m
kube-system   calico-node-m5hjk                          1/1     Running   0          97m
kube-system   calico-node-t8k2g                          1/1     Running   0          97m
kube-system   coredns-74ff55c5b-9k9pd                    1/1     Running   0          158m
kube-system   coredns-74ff55c5b-jwm7b                    1/1     Running   0          158m
kube-system   etcd-master                                1/1     Running   0          158m
kube-system   kube-apiserver-master                      1/1     Running   0          158m
kube-system   kube-controller-manager-master             1/1     Running   0          5m56s
kube-system   kube-proxy-2prxw                           1/1     Running   0          153m
kube-system   kube-proxy-q9n5c                           1/1     Running   0          158m
kube-system   kube-proxy-rwbqz                           1/1     Running   0          153m
kube-system   kube-scheduler-master                      1/1     Running   0          10m
```

