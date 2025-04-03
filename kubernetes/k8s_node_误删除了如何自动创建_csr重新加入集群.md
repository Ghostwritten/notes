![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8b3f2630765b701a5b75124d23c26b9f.png)



worker node 节点当部署晚 kubelet、kube-proxy就会加入集群，如何加入呢，

```bash
[root@kube-node01 ssl]# mv kubelet-client-2023-08-13-01-19-00.pem kubelet-client-current.pem kubelet.crt kubelet.key /tmp/kubelet
[root@kube-node01 ssl]# systemctl daemon-reload 
[root@kube-node01 ssl]# systemctl restart kubelet
[root@kube-node01 ssl]# ls -l
total 16
-rw-r--r-- 1 root root 1322 Aug 12 22:02 ca.pem
-rw------- 1 root root  227 Aug 13 02:04 kubelet-client.key.tmp
-rw-r--r-- 1 root root 2271 Aug 13 02:04 kubelet.crt
-rw------- 1 root root 1679 Aug 13 02:04 kubelet.key
```


集群收到新的 csr

```bash
[root@kube-master01 cni]# kubectl get csr
NAME                                                   AGE     SIGNERNAME                                    REQUESTOR           REQUESTEDDURATION   CONDITION
node-csr-aOsrLNUyz5ny6niMrdpytbeJVopJmUvRXFtFryxGr0M   9s      kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   <none>              Pending
node-csr-pfDntvT88QqSwoF4qXlZ7h4aY4HSMmoJvokLwJS0tmo   3h55m   kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   <none>              Approved,Issued

[root@kube-master01 cni]# kubectl get node
NAME            STATUS   ROLES    AGE     VERSION
kube-master01   Ready    <none>   3h57m   v1.23.17
[root@kube-master01 cni]# kubectl certificate approve node-csr-aOsrLNUyz5ny6niMrdpytbeJVopJmUvRXFtFryxGr0M
certificatesigningrequest.certificates.k8s.io/node-csr-aOsrLNUyz5ny6niMrdpytbeJVopJmUvRXFtFryxGr0M approved
[root@kube-master01 cni]# kubectl get csr
NAME                                                   AGE     SIGNERNAME                                    REQUESTOR           REQUESTEDDURATION   CONDITION
node-csr-aOsrLNUyz5ny6niMrdpytbeJVopJmUvRXFtFryxGr0M   4m6s    kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   <none>              Approved,Issued
node-csr-pfDntvT88QqSwoF4qXlZ7h4aY4HSMmoJvokLwJS0tmo   3h59m   kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   <none>              Approved,Issued
[root@kube-master01 cni]# kubectl get node
NAME            STATUS     ROLES    AGE     VERSION
kube-master01   Ready      <none>   3h58m   v1.23.17
kube-node01     NotReady   <none>   6s      v1.23.17
[root@kube-master01 cni]# kubectl get node
NAME            STATUS   ROLES    AGE     VERSION
kube-master01   Ready    <none>   3h58m   v1.23.17
kube-node01     Ready    <none>   31s     v1.23.17
[root@kube-master01 cni]# kubectl label  nodes kube-master01   node-role.kubernetes.io/control-plane=''
node/kube-master01 labeled
[root@kube-master01 cni]# kubectl get node
NAME            STATUS   ROLES           AGE    VERSION
kube-master01   Ready    control-plane   4h     v1.23.17
kube-node01     Ready    <none>          2m2s   v1.23.17
[root@kube-master01 cni]# kubectl get pod -n kube-system
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-5454b755c5-wh5h9   1/1     Running   0          59m
calico-node-2n2vx                          1/1     Running   0          61m
calico-node-vgm4f                          1/1     Running   0          2m16s

```

参考：
- [https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/certificate-signing-requests/](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/certificate-signing-requests/)
- [https://blog.csdn.net/Michaelwubo/article/details/113769391](https://blog.csdn.net/Michaelwubo/article/details/113769391)
