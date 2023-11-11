


---
## 1. 介绍
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210425185449694.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210425185621584.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210425185743944.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210425185845510.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

## 2. Practice - Anonymous Access
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210425185956297.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210425190008318.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
root@master:~/cks/serviceaccount# curl https://localhost:6443
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.haxx.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
root@master:~/cks/serviceaccount# curl https://localhost:6443 -k
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {
    
  },
  "status": "Failure",
  "message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
  "reason": "Forbidden",
  "details": {
    
  },
  "code": 403
}root@master:~/cks/serviceaccount# vim /etc/kubernetes/manifests/kube-apiserver.yaml 
...
    - kube-apiserver
    - --anonymous-auth=false
    - --advertise-address=192.168.211.40
....
root@master:~/cks/serviceaccount# k get pods | grep api
The connection to the server 192.168.211.40:6443 was refused - did you specify the right host or port?
root@master:~/images# k get pods -n kube-system | grep api
kube-apiserver-master                      1/1     Running   0          8m3s
root@master:~/images# k get pods -n kube-system | grep api
kube-apiserver-master                      1/1     Running   0          3s

```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210425194247517.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
root@master:~/cks/serviceaccount# vim /etc/kubernetes/manifests/kube-apiserver.yaml 
...
    - kube-apiserver
    - --anonymous-auth=true #默认其实为true
    - --advertise-address=192.168.211.40
....


root@master:~/cks/serviceaccount# curl https://localhost:6443 -k
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {
    
  },
  "status": "Failure",
  "message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
  "reason": "Forbidden",
  "details": {
    
  },
  "code": 403
}
```
## 3. Practice - Insecure Access
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210425194521961.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210425194710926.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210425194805223.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
root@master:~/cks/serviceaccount# curl https://localhost:6443 -k
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {
    
  },
  "status": "Failure",
  "message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
  "reason": "Forbidden",
  "details": {
    
  },
  "code": 403
}

root@master:~# vim /etc/kubernetes/manifests/kube-apiserver.yaml
...
    - kube-apiserver
    - --advertise-address=192.168.211.40
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --insecure-port=8080  #0改成8080
  .....


root@master:~/cks/serviceaccount# k get pods | grep api
The connection to the server 192.168.211.40:6443 was refused - did you specify the right host or port?

root@master:~/images# k get pods -n kube-system | grep api
kube-apiserver-master                      1/1     Running   0          3s


root@master:~# curl http://localhost:8080
```

## 4. Practice - Manual API Request
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210426141859678.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
root@master:/etc/kubernetes/pki# curl https://192.168.211.40:6443 --cacert ca --cert  ca.crt --key ca.key
```
## 5. Practice - External Apiserver Access
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210426143209717.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
root@master:/etc/kubernetes/pki# k edit svc
....
  type: NodePort
....

root@master:/etc/kubernetes/pki# k get svc
NAME         TYPE       CLUSTER-IP   EXTERNAL-IP   PORT(S)         AGE
kubernetes   NodePort   10.96.0.1    <none>        443:30300/TCP   19h



root@master:/etc/kubernetes/pki# curl https://192.168.211.40:30300 -l
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.haxx.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
root@master:/etc/kubernetes/pki# curl https://192.168.211.40:30300 -k
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {
    
  },
  "status": "Failure",
  "message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
  "reason": "Forbidden",
  "details": {
    
  },
  "code": 403
}
```

```bash
root@master:~/cks/apiserver# k config view --raw >config
root@master:~/cks/apiserver# k --kubeconfig config get ns
NAME              STATUS   AGE
default           Active   19h
kube-node-lease   Active   19h
kube-public       Active   19h
kube-system       Active   19h
root@master:~/cks/apiserver# vim config
.....
   server: https://192.168.211.40:30300  #6443改为30300
.... 
root@master:~/cks/apiserver# k --kubeconfig config get ns
NAME              STATUS   AGE
default           Active   19h
kube-node-lease   Active   19h
kube-public       Active   19h
kube-system       Active   19h
```
## 6. NodeRestriction AdmissionController
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210426150839360.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210426151051581.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 7. Practice - Verify NodeRestriction
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210426151124766.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
root@master:~/cks/apiserver# vim /etc/kubernetes/manifests/kube-apiserver.yaml 
....
- --enable-admission-plugins=NodeRestriction
...

root@master:~/cks/apiserver# k get ns
Error from server (Forbidden): namespaces is forbidden: User "system:node:master" cannot list resource "namespaces" in API group "" at the cluster scope

root@master:~/cks/apiserver# export KUBECONFIG=/etc/kubernetes/kubelet.conf.

root@master:~/cks/apiserver# k label node master cks/test=yes
node/master labeled
root@master:~/cks/apiserver# k label node node1 cks/test=yes
Error from server (Forbidden): nodes "node1" is forbidden: node "master" is not allowed to modify node "node1"

```

