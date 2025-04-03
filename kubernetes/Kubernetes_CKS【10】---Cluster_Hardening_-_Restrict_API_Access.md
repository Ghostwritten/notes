


---
## 1. 介绍
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8905820457fb37994edaf01f51110454.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/82ef8eaac1fae71ec994942b439c1184.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/29723f7ca451c4469d563183534c9fb1.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/da2c3a46024b61386c2eeecbf84b214f.png)

## 2. Practice - Anonymous Access
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/255e9803f28062a840a6b6bd9fbf3a82.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/23a1a692dca366f00feefb073098fc16.png)

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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b88c4308f5ad7227fec6ded76fd6e387.png)

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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/46cdee5a40b79e83d7784287ad6bd2f2.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c56d190dfcce9e90e03d2c2692d82533.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/25fa8dcf9a92b2d4596835f2d1d6a3f5.png)

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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a498d3b6b1d5b2977461bf8804e26341.png)

```bash
root@master:/etc/kubernetes/pki# curl https://192.168.211.40:6443 --cacert ca --cert  ca.crt --key ca.key
```
## 5. Practice - External Apiserver Access
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8c7d8598fe7015e03c0073e92b582051.png)

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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/59eedb81eabe46131da66fb742c78d77.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/99e94d6318ef6b05e296e06dc44e47fe.png)
## 7. Practice - Verify NodeRestriction
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/eb8244fc80ac5b4593c183ff55c8d984.png)

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

